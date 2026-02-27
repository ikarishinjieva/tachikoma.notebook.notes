---
title: 20260204 - 对机械臂/手 进行 AI示教AI 的尝试二 - yolo seg/pose
confluence_page_id: 4620401
created_at: 2026-02-04T03:21:43+00:00
updated_at: 2026-02-09T07:44:31+00:00
---

# 现状

在 [20260112 - 对机械臂/手 进行 AI示教AI 的尝试一] 中: 

  - AI示教, 无法通过图片判断 动作状态 (握住与否)
    - 计划使用压敏组件进行判断
  - 点云匹配
    - 无法获得干净的点云
    - 无法区分 移液枪正反
    - 需要额外特征值

目标场景: 让 灵巧手 能通过尝试学会 握持不同的移液枪

当前子目标: 使用YOLO来进行特征分析, 求得移液枪位姿 (并具有泛化性)

# 用roboflow进行 segment 标记, 测试 yolo-seg (YOLO26)

标记segment, roboflow提供SEM模型进行智能辅助, 标注比较容易.

用7张图片( 加增强一共 24张), 训练后 (一共10张, 其中2张val, 1张test), 测试一个独立的效果: 

![image2026-2-4 13:25:54.png](/assets/01KJC0CANQHRJ7GDJWQAST4EEW/image2026-2-4%2013%3A25%3A54.png)

完全正面看不到 指勾 的效果: 

![image2026-2-4 13:28:22.png](/assets/01KJC0CANQHRJ7GDJWQAST4EEW/image2026-2-4%2013%3A28%3A22.png)

大小移液枪测试 (训练数据只有大的移液枪):

![image2026-2-4 13:29:40.png](/assets/01KJC0CANQHRJ7GDJWQAST4EEW/image2026-2-4%2013%3A29%3A40.png)

小移液枪的正面测试: 

![image2026-2-4 13:30:36.png](/assets/01KJC0CANQHRJ7GDJWQAST4EEW/image2026-2-4%2013%3A30%3A36.png)

效果是比较好的, roboflow地址: <https://app.roboflow.com/project/pipetee-seg/3>

python样例代码: 

```
from inference_sdk import InferenceHTTPClient

CLIENT = InferenceHTTPClient(
    api_url="https://serverless.roboflow.com",
    api_key="LG58D7QWjX6SvUID8c1a"
)

result = CLIENT.infer("image.jpg", model_id="pipetee-seg/3")
print(result)
 
``` 

这里还尝试了用 "通过截面面积判断 指勾 大致方位, 确定轴线方向", 这个效果是可以接受的:

commit: 73ca32da17f28337122734bb234ed5f1b4c49853

  - 脚本说明: /data/huangyan/isaac_ros-dev/robots-ai/ai_mentor/pipetee_6d_v2/[README.md](<http://README.md>)

# 使用yolo-pose 来获取指勾具体2D姿态

发现用 yolo-pose 效果不好. 经过排查, roboflow平台的图片增强, 会导致标记点错乱. 关掉图片增强进行重试.

发现效果仍然不好, 关掉预处理中的stretch (伸缩影响标记), 效果变好, 但仍有误差: 

![image2026-2-5 13:48:50.png](/assets/01KJC0CANQHRJ7GDJWQAST4EEW/image2026-2-5%2013%3A48%3A50.png)

更换成三点标记试试 (四个点钟, 中间那个点hook connector没有明显视觉特征).

训练后效果更差: 

![image2026-2-5 14:44:16.png](/assets/01KJC0CANQHRJ7GDJWQAST4EEW/image2026-2-5%2014%3A44%3A16.png)

只标注一个点, 点会很偏: 

![image2026-2-5 15:45:3.png](/assets/01KJC0CANQHRJ7GDJWQAST4EEW/image2026-2-5%2015%3A45%3A3.png)

搭建本地标记环境: [20260205 - 搭建labelme环境, 进行yolo训练]

进行三点标记, 使用模型: 

  - yolov8n-pose, 验证效果不好: 
    - 训练数据重验 (拟合不上): 
      - ![image2026-2-5 21:44:16.png](/assets/01KJC0CANQHRJ7GDJWQAST4EEW/image2026-2-5%2021%3A44%3A16.png)  

    - 验证数据
      - ![image2026-2-5 21:44:35.png](/assets/01KJC0CANQHRJ7GDJWQAST4EEW/image2026-2-5%2021%3A44%3A35.png)  
  

因为图片很少, 将batch降为1: 

```
yolo pose train \
    project=pipetee_project \
    name=run1 \
    data=/data/huangyan/pipetee/pipetee_pose.yaml \
    model=yolov8n-pose.pt \
    epochs=100 \
    imgsz=640 \
    batch=1 \
    device=0
``` 

  

结果指标: 

```
      Epoch    GPU_mem   box_loss  pose_loss  kobj_loss   cls_loss   dfl_loss  Instances       Size
    100/100     0.385G      1.255      2.628      0.644      1.412      1.526          1        640: 100% ━━━━━━━━━━━━ 13/13 13.0it/s 1.0s
                 Class     Images  Instances      Box(P          R      mAP50  mAP50-95)     Pose(P          R      mAP50  mAP50-95): 100% ━━━━━━━━━━━━ 2/2 31.5it/s 0.1s
                   all          4          4      0.361       0.75      0.516      0.234      0.239        0.5      0.185     0.0542
``` 

  

其中: Box的mAP50=0.516, Pose的mAP50=0.185 很低 (mAP50为及格率, 标记与目标有50%重合)

  

放大epoch=300, best结果: 

```
                 Class     Images  Instances      Box(P          R      mAP50  mAP50-95)     Pose(P          R      mAP50  mAP50-95): 100% ━━━━━━━━━━━━ 2/2 25.2it/s 0.1s
                   all          4          4      0.915       0.75      0.783      0.578      0.727       0.75      0.714      0.364
``` 

  

训练数据的偏差仍不小: 

![image2026-2-5 23:59:53.png](/assets/01KJC0CANQHRJ7GDJWQAST4EEW/image2026-2-5%2023%3A59%3A53.png)

  

放大imgsz=960, 以及增加epoch, 也没有改善

  

  

使用更大的模型 (M模型): 

```
export http_proxy=http://127.0.0.1:7990
export https_proxy=http://127.0.0.1:7990
yolo pose train \
    project=pipetee_project \
    name=run \
    data=/data/huangyan/pipetee/pipetee_pose.yaml \
    model=yolov8m-pose.pt \
    epochs=300 \
    imgsz=640 \
    batch=1 \
    device=0 \
    patience=0
``` 

  

效果变差: 

```
YOLOv8m-pose summary (fused): 102 layers, 26,401,822 parameters, 0 gradients, 80.8 GFLOPs
                 Class     Images  Instances      Box(P          R      mAP50  mAP50-95)     Pose(P          R      mAP50  mAP50-95): 100% ━━━━━━━━━━━━ 2/2 22.3it/s 0.1s
                   all          4          4    0.00245       0.25     0.0031    0.00031      0.599        0.5      0.419      0.146
``` 

  

  

测试YOLO11 (yolo8v的升级版), 在不同规模和不同batch下的性能: 

| 模型 | Batch Size | Box mAP50 | Box mAP50-95 | Pose mAP50 | Pose mAP50-95 |
| --- | --- | --- | --- | --- | --- |
| yolo11n-pose | 1 | 0.512 | 0.353 | 0.448 | 0.381 |
| yolo11n-pose | 2 | 0.895 | 0.625 | 0.753 | 0.635 |
| yolo11n-pose | 4 | 0.945 | 0.837 | 0.648 | 0.603 |
| yolo11n-pose | 8 | 0.912 | 0.604 | 0.828 | 0.673 |
| yolo11n-pose | 16 | 0.808 | 0.699 | 0.775 | 0.727 |
| yolo11s-pose | 1 | 0.292 | 0.189 | 0.29 | 0.264 |
| yolo11s-pose | 2 | 0.658 | 0.407 | 0.309 | 0.273 |
| yolo11s-pose | 4 | 0.912 | 0.668 | 0.745 | 0.626 |
| yolo11s-pose | 8 | 0.995 | 0.732 | 0.764 | 0.688 |
| yolo11s-pose | 16 | 0.761 | 0.687 | 0.586 | 0.497 |
| yolo11m-pose | 1 | 0.297 | 0.233 | 0.545 | 0.455 |
| yolo11m-pose | 2 | 0.695 | 0.301 | 0.731 | 0.432 |
| yolo11m-pose | 4 | 0.807 | 0.48 | 0.661 | 0.536 |
| yolo11m-pose | 8 | 0.822 | 0.595 | 0.684 | 0.51 |
| yolo11m-pose | 16 | 0.695 | 0.529 | 0.601 | 0.46 |
  
  
效果比较好的是: yolo11n-pose, batch=8

测试效果: 

  - 训练数据: 大致方位正确: 
    - ![image2026-2-6 12:56:56.png](/assets/01KJC0CANQHRJ7GDJWQAST4EEW/image2026-2-6%2012%3A56%3A56.png)
  - 校验数据
    - ![image2026-2-6 13:0:45.png](/assets/01KJC0CANQHRJ7GDJWQAST4EEW/image2026-2-6%2013%3A0%3A45.png)
    - 首尾在一定范围内, 指勾点有点偏

  

先尝试完全过拟合, 需要去掉自动的图片增强: 

```
# 需要在pipetee_pose.overfit.yaml中, 将验证集也设置为训练集目录
 
yolo pose train \
    project=pipetee_project \
    name=overfit_test_17imgs \
    data=/data/huangyan/pipetee/pipetee_pose.overfit.yaml \
    model=yolo11n-pose.pt \
    epochs=300 \
    imgsz=640 \
    batch=16 \
    device=0 \
    workers=0 \
    lr0=0.01 \
    lrf=0.01 \
    amp=False \
    \
    degrees=0.0 \
    translate=0.0 \
    scale=0.0 \
    shear=0.0 \
    perspective=0.0 \
    flipud=0.0 \
    fliplr=0.0 \
    mosaic=0.0 \
    mixup=0.0 \
    copy_paste=0.0 \
    auto_augment=randaugment \
    erasing=0.0 \
    crop_fraction=1.0
``` 

  

  

过拟合结果: 

```
                 Class     Images  Instances      Box(P          R      mAP50  mAP50-95)     Pose(P          R      mAP50  mAP50-95): 100% ━━━━━━━━━━━━ 1/1 3.0it/s 0.3s
                   all         13         13      0.996          1      0.995      0.785      0.996          1      0.995      0.991
``` 

  

其中 Box的mAP50-95 比较低, 是因为 Box的标记是手工标记, 包裹物体的box面积不稳定. 认为只要Pose 准就可以. 

  

在这个场景上, 开启部分增强: 

```
yolo pose train     project=pipetee_project     name=aug_test_geom_only     data=/data/huangyan/pipetee/pipetee_pose.overfit.yaml     model=yolo11n-pose.pt     epochs=300     imgsz=640     batch=16     device=0     workers=0         mosaic=0.0     mixup=0.0     copy_paste=0.0         degrees=10.0     translate=0.1     scale=0.5     shear=0.0     perspective=0.0         fliplr=0.0     flipud=0.0
``` 

  

参数调整跟上一次的对比: (关掉了 拼图 mosaic 和 翻转flip)

![image2026-2-6 17:54:59.png](/assets/01KJC0CANQHRJ7GDJWQAST4EEW/image2026-2-6%2017%3A54%3A59.png)

本次过拟合效果良好: 

```
YOLO11n-pose summary (fused): 110 layers, 2,654,326 parameters, 0 gradients, 6.6 GFLOPs
                 Class     Images  Instances      Box(P          R      mAP50  mAP50-95)     Pose(P          R      mAP50  mAP50-95): 100% ━━━━━━━━━━━━ 1/1 3.1it/s 0.3s
                   all         13         13      0.995          1      0.995      0.918      0.995          1      0.995      0.989
``` 

在配置文件中配置 flip_idx, 开启翻转: 

```
# pipetee_pose.yaml

# 1. 数据集路径
path: /data/huangyan/pipetee/dataset_final
train: images/train
val: images/val

# 2. 关键点形状
# 3个点，每个点有3个值 (x, y, visibility)
kpt_shape: [3, 3]

# 3. 对称关系 (可选，但推荐)
# 加上这行，开启左右翻转增强，且ID不互换
flip_idx: [0, 1, 2]

# 4. 物体类别名称 (Class Names)
# 这里定义的是“物体”叫什么，不是“点”叫什么
names:
  0: pipette  # 你的物体叫移液枪
``` 

只禁用拼图: 

```
yolo pose train     project=pipetee_project     name=aug_test_geom_only     data=/data/huangyan/pipetee/pipetee_pose.overfit.yaml     model=yolo11n-pose.pt     epochs=300     imgsz=640     batch=16     device=0     workers=0         mosaic=0.0
``` 

结果也可以过拟合

允许mosaic拼图, 达到最优值很慢 (从 100 epoch 变成300 epoch)

  

仍禁用拼图, 切换真实校验集, 在epoch 204开始过拟合, 触发早停: 

```
yolo pose train     project=pipetee_project     name=aug_test_geom_only     data=/data/huangyan/pipetee/pipetee_pose.yaml     model=yolo11n-pose.pt     epochs=600     imgsz=640     batch=16     device=0     workers=0         mosaic=0.0
 
 
---
 
 

EarlyStopping: Training stopped early as no improvement observed in last 100 epochs. Best results observed at epoch 204, best model saved as best.pt.
To update EarlyStopping(patience=100) pass a new patience value, i.e. `patience=300` or use `patience=0` to disable EarlyStopping.

                 Class     Images  Instances      Box(P          R      mAP50  mAP50-95)     Pose(P          R      mAP50  mAP50-95): 100% ━━━━━━━━━━━━ 1/1 8.5it/s 0.1s
                   all          4          4      0.648       0.75      0.856      0.657      0.648       0.75       0.87      0.674
``` 

  

  

扩展11张图片 (扩增到28张), 再做这个过拟合训练: 

```
极值在160 epoch, 然后早停
 
YOLO11n-pose summary (fused): 110 layers, 2,654,326 parameters, 0 gradients, 6.6 GFLOPs
                 Class     Images  Instances      Box(P          R      mAP50  mAP50-95)     Pose(P          R      mAP50  mAP50-95): 100% ━━━━━━━━━━━━ 1/1 6.3it/s 0.2s
                   all          6          6          1      0.975      0.995      0.802          1      0.975      0.995      0.916
``` 

  

测试效果: 

  - 训练集
    - ![image2026-2-6 20:11:17.png](/assets/01KJC0CANQHRJ7GDJWQAST4EEW/image2026-2-6%2020%3A11%3A17.png)
  - 校验集
    - ![image2026-2-6 20:8:47.png](/assets/01KJC0CANQHRJ7GDJWQAST4EEW/image2026-2-6%2020%3A8%3A47.png)

增加patience=0 禁用早停:

```
epoch=312
 
YOLO11n-pose summary (fused): 110 layers, 2,654,326 parameters, 0 gradients, 6.6 GFLOPs
                 Class     Images  Instances      Box(P          R      mAP50  mAP50-95)     Pose(P          R      mAP50  mAP50-95): 100% ━━━━━━━━━━━━ 1/1 5.5it/s 0.2s
                   all          6          6          1       0.99      0.995      0.889          1       0.99      0.995      0.995
 
 
最后的epoch:
      Epoch    GPU_mem   box_loss  pose_loss  kobj_loss   cls_loss   dfl_loss  Instances       Size
    600/600      2.42G     0.1892     0.1032    0.05545      0.189     0.7779          6        640: 100% ━━━━━━━━━━━━ 2/2 5.4it/s 0.4s
                 Class     Images  Instances      Box(P          R      mAP50  mAP50-95)     Pose(P          R      mAP50  mAP50-95): 100% ━━━━━━━━━━━━ 1/1 6.7it/s 0.1s
                   all          6          6          1          1      0.995      0.863          1          1      0.995      0.977
``` 

分别测试: 

  - best epoch
    - ![image2026-2-6 20:23:52.png](/assets/01KJC0CANQHRJ7GDJWQAST4EEW/image2026-2-6%2020%3A23%3A52.png)![image2026-2-6 20:24:36.png](/assets/01KJC0CANQHRJ7GDJWQAST4EEW/image2026-2-6%2020%3A24%3A36.png)
  - last
    - ![image2026-2-6 20:25:38.png](/assets/01KJC0CANQHRJ7GDJWQAST4EEW/image2026-2-6%2020%3A25%3A38.png)![image2026-2-6 20:26:26.png](/assets/01KJC0CANQHRJ7GDJWQAST4EEW/image2026-2-6%2020%3A26%3A26.png)

增加数据的效果很好, 指标提高, 并且last中的训练数据校验已经接近正确, 校验集数据的指勾仍有偏差

(与大模型训练一样, best和last都得考虑, best的实际性能不一定最好)

因为是细节问题, 考虑放大像素, 同时放大epoch

| run_name | batch | imgsz | box_mAP50 | box_mAP50-95 | pose_mAP50 | pose_mAP50-95 |
| --- | --- | --- | --- | --- | --- | --- |
| aug_test_geom_only_b8_img640 | 8 | 640 | 0.995 | 0.842 | 0.995 | 0.995 |
| aug_test_geom_only_b8_img960 | 8 | 960 | 0.917 | 0.808 | 0.917 | 0.917 |
| aug_test_geom_only_b8_img1280 | 8 | 1280 | 0.995 | 0.55 | 0.995 | 0.938 |
| aug_test_geom_only_b16_img640 | 16 | 640 | 0.995 | 0.873 | 0.995 | 0.985 |
| aug_test_geom_only_b16_img960 | 16 | 960 | 0.886 | 0.737 | 0.972 | 0.955 |
| aug_test_geom_only_b16_img1280 | 16 | 1280 | 0.846 | 0.607 | 0.903 | 0.812 |
  
举例: batch=8, imgsz=640 的校验集: 仍然会差几个像素

(aug_test_geom_only_b8_img6402/val_batch0_pred.jpg)

![image2026-2-7 18:45:8.png](/assets/01KJC0CANQHRJ7GDJWQAST4EEW/image2026-2-7%2018%3A45%3A8.png)

batch=8, imgsz=960 的校验集: 仍然会差几个像素

![image2026-2-7 18:48:54.png](/assets/01KJC0CANQHRJ7GDJWQAST4EEW/image2026-2-7%2018%3A48%3A54.png)

对于陌生集: (顶底 大体位置类似, 但指勾仍有不少差异), 数据不够多

![image2026-2-7 18:31:16.png](/assets/01KJC0CANQHRJ7GDJWQAST4EEW/image2026-2-7%2018%3A31%3A16.png)

# 对现状总结

yolo-seg在很小的数据集中, 就可以形成很好的分割. 通过图像学算法 可以确定位姿. 并且泛化性良好.

yolo-pose 在小数据集 (30个以下), 点位识别 不准, 需要更大的数据集, 且泛化性消失 (需要更大的数据集改善泛化性). 但判断肯定能解决:

  - 比如用分割的移液枪, 放到其他背景中, 合成
  - 用标准模型进行合成
  - 将分割后的移液枪 涂黑, 只认识轮廓?   
  

对现在的判断: 可以使用yolo-seg + 图形学算法 确定位姿, 如果找不到指勾, 就让机械臂换个方向?
