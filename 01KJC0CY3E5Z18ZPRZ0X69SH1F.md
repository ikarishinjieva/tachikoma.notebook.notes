---
title: 20260205 - 搭建labelme环境, 进行yolo训练
confluence_page_id: 4620419
created_at: 2026-02-05T11:25:44+00:00
updated_at: 2026-02-06T12:36:37+00:00
---

```
conda create -n yolo_pose python=3.9 -y
conda activate yolo_pose
pip install labelme -i https://pypi.tuna.tsinghua.edu.cn/simple

# 移除可能冲突的 opencv-python
pip uninstall opencv-python -y
# 安装不带 GUI 库的 opencv
pip install opencv-python-headless
 
apt-get install -y libxkbcommon-x11-0 libxcb-xinerama0
然后在桌面上打开终端, 启动labelme (如果在ssh命令行里启动, 会报错)
``` 

如何连接桌面: 将终端映射出来: ssh -p 24922 -L 8443:localhost:8443 [root@183.196.130.56](<mailto:root@183.196.130.56>) , 用DCV连本地, 

用labelme进行标注, 图片目录在/data/huangyan/pipetee/labelme_images

配置训练环境:

```
pip install torch torchvision torchaudio -i https://pypi.tuna.tsinghua.edu.cn/simple
pip install ultralytics -i https://pypi.tuna.tsinghua.edu.cn/simple
 
``` 

创建脚本, 将labelme转成yolo标记: 

```
import json
import os
import glob

# ================= 配置路径 =================
INPUT_IMG_DIR = "/data/huangyan/pipetee/labelme_images"  # JSON 和图片都在这里
OUTPUT_LABEL_DIR = "/data/huangyan/pipetee/yolo_labels"  # 生成的 TXT 放这里
KEYPOINTS_ORDER = ["top", "bottom", "hook"]              # 你的3个点
# ===========================================

def convert():
    if not os.path.exists(OUTPUT_LABEL_DIR):
        os.makedirs(OUTPUT_LABEL_DIR)
    
    json_files = glob.glob(os.path.join(INPUT_IMG_DIR, "*.json"))
    print(f"找到 {len(json_files)} 个 JSON 文件，开始转换...")
    
    for json_path in json_files:
        with open(json_path, 'r', encoding='utf-8') as f:
            data = json.load(f)

        img_w = data['imageWidth']
        img_h = data['imageHeight']
        filename = os.path.basename(json_path).replace('.json', '.txt')
        
        # 提取关键点
        kpts_dict = {}
        for shape in data['shapes']:
            if shape['shape_type'] == 'point' and shape['label'] in KEYPOINTS_ORDER:
                kpts_dict[shape['label']] = shape['points'][0]

        # 自动生成框 (如果没有框)
        if not kpts_dict:
            continue # 空数据跳过
            
        all_x = [p[0] for p in kpts_dict.values()]
        all_y = [p[1] for p in kpts_dict.values()]
        pad = 20 # 稍微大一点的padding
        x1, y1 = max(0, min(all_x)-pad), max(0, min(all_y)-pad)
        x2, y2 = min(img_w, max(all_x)+pad), min(img_h, max(all_y)+pad)

        # 归一化计算
        dw, dh = 1./img_w, 1./img_h
        w, h = x2-x1, y2-y1
        cx, cy = x1 + w/2.0, y1 + h/2.0
        
        # 拼接关键点数据
        kpts_data = []
        for key in KEYPOINTS_ORDER:
            if key in kpts_dict:
                px, py = kpts_dict[key]
                kpts_data.extend([px*dw, py*dh, 2]) # 2=可见
            else:
                kpts_data.extend([0, 0, 0])

        # 写入 (Class ID = 0)
        line = f"0 {cx*dw:.6f} {cy*dh:.6f} {w*dw:.6f} {h*dh:.6f} " + " ".join([f"{x:.6f}" for x in kpts_data])
        
        with open(os.path.join(OUTPUT_LABEL_DIR, filename), 'w') as f_out:
            f_out.write(line + '\n')

    print("转换完成。")

if __name__ == "__main__":
    convert()
``` 

创建脚本, 将数据集进行划分: 

```
import os
import shutil
import random
import glob

# ================= 配置 =================
# 原始数据位置
IMG_SRC = "/data/huangyan/pipetee/labelme_images"
LABEL_SRC = "/data/huangyan/pipetee/yolo_labels"

# 最终训练数据存放位置 (会自动创建)
DATASET_ROOT = "/data/huangyan/pipetee/dataset_final"

# 划分比例 (验证集占 20%)
VAL_RATIO = 0.2
# =======================================

def split_data():
    # 1. 创建目录结构
    for split in ['train', 'val']:
        os.makedirs(os.path.join(DATASET_ROOT, 'images', split), exist_ok=True)
        os.makedirs(os.path.join(DATASET_ROOT, 'labels', split), exist_ok=True)

    # 2. 获取所有生成的 txt 文件名 (作为基准，防止有图片没标签)
    label_files = glob.glob(os.path.join(LABEL_SRC, "*.txt"))
    base_names = [os.path.basename(f).replace('.txt', '') for f in label_files]
    
    # 打乱顺序
    random.shuffle(base_names)
    
    # 计算分割点
    split_idx = int(len(base_names) * (1 - VAL_RATIO))
    train_names = base_names[:split_idx]
    val_names = base_names[split_idx:]
    
    print(f"总数据: {len(base_names)}, 训练集: {len(train_names)}, 验证集: {len(val_names)}")

    # 3. 复制文件
    def copy_files(names, split_type):
        for name in names:
            # 复制图片 (尝试 jpg 和 png)
            img_path = os.path.join(IMG_SRC, name + ".jpg")
            if not os.path.exists(img_path):
                img_path = os.path.join(IMG_SRC, name + ".png")
            
            if os.path.exists(img_path):
                shutil.copy(img_path, os.path.join(DATASET_ROOT, 'images', split_type, os.path.basename(img_path)))
            
            # 复制标签
            lbl_path = os.path.join(LABEL_SRC, name + ".txt")
            shutil.copy(lbl_path, os.path.join(DATASET_ROOT, 'labels', split_type, name + ".txt"))

    print("正在构建训练集...")
    copy_files(train_names, 'train')
    print("正在构建验证集...")
    copy_files(val_names, 'val')
    print(f"数据准备完毕！存放于: {DATASET_ROOT}")

if __name__ == "__main__":
    split_data()
``` 

编写配置文件: 

```
# pipetee_pose.yaml

# 数据集路径 (建议使用绝对路径)
path: /data/huangyan/pipetee/dataset_final
train: images/train
val: images/val

# 关键点配置
# [关键点数量, 维度] 
# 维度为3: (x, y, visibility)
kpt_shape: [3, 3]

# 类别名称
names:
  0: pipetee
``` 

开始训练: 

```
export http_proxy=http://127.0.0.1:7990
export https_proxy=http://127.0.0.1:7990
 
yolo pose train \
    project=pipetee_project \
    name=run1 \
    data=/data/huangyan/pipetee/pipetee_pose.yaml \
    model=yolov8n-pose.pt \
    epochs=100 \
    imgsz=640 \
    batch=16 \
    device=0
``` 

查看预测结果: 

```
yolo pose predict   \
model=/data/huangyan/pipetee/runs/pose/pipetee_project/run13/weights/best.pt \ source=/data/huangyan/pipetee/labelme_images/val.jpg  \
save=True max_det=1
``` 

查看保存的结果文件
