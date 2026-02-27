---
title: 20251204 - 运行LabUtopia
confluence_page_id: 4359385
created_at: 2025-12-04T10:56:16+00:00
updated_at: 2025-12-04T10:56:16+00:00
---

数据集: <https://huggingface.co/datasets/Ruinwalker/LabUtopia-Dataset/tree/main>

代码包: <https://github.com/Rui-li023/LabUtopia/tree/main>

环境变量: 

```
export http_proxy=http://127.0.0.1:7890
export https_proxy=http://127.0.0.1:7890
export no_proxy=127.0.0.1,localhost,ucloud.cn
export ROS_DOMAIN_ID=99
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
export ROS_DISTRO=humble
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/data/huangyan/isaac-sim-4-2/exts/omni.isaac.ros2_bridge/humble/lib/
export CUDA_VISIBLE_DEVICES=7
export __GLX_VENDOR_LIBRARY_NAME=nvidia
export DISPLAY=:0
export CYCLONEDDS_URI=/data/huangyan/isaac_ros-dev/a.xml
``` 

安装依赖: 

```
/data/huangyan/isaac-sim-4-2/python.sh -m pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
``` 

utils包与isaac sim内置cv2包冲突, 更换名字: 

```
grep -rl 'from utils' . --include='*.py' | xargs sed -i 's/from utils/from lab_utils/g'
``` 

运行: 

```
cd /data/huangyan/lab/lab-code
PYTHONPATH=$(pwd):$PYTHONPATH /data/huangyan/isaac-sim-4-2/python.sh main.py --config-name level1_pick --no-video --backend gpu
```
```
cd /data/huangyan/lab/lab-code
PYTHONPATH=$(pwd):$PYTHONPATH /data/huangyan/isaac-sim-4-2/python.sh main.py --config-name level2_PourLiquid --no-video --backend gpu
```

```
cd /data/huangyan/lab/lab-code
PYTHONPATH=$(pwd):$PYTHONPATH /data/huangyan/isaac-sim-4-2/python.sh main.py --config-name level3_HeatLiquid --no-video --backend gpu
```

```
cd /data/huangyan/lab/lab-code
PYTHONPATH=$(pwd):$PYTHONPATH /data/huangyan/isaac-sim-4-2/python.sh main.py --config-name level4_LiquidMixing --no-video --backend gpu
```
