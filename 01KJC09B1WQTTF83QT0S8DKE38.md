---
title: 20251211 - 仿真实验室演示记录
confluence_page_id: 4359441
created_at: 2025-12-11T08:34:03+00:00
updated_at: 2025-12-11T08:34:03+00:00
---

# 多器具陈列

lab-077

# 有缓存的实验室场景

034

040

085

023

# 环境变量

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

# 任务

```
cd /data/huangyan/lab/lab-code
PYTHONPATH=$(pwd):$PYTHONPATH /data/huangyan/isaac-sim-4-2/python.sh main.py --config-name level3_HeatLiquid --no-video --backend gpu
```
```
cd /data/huangyan/lab/lab-code
PYTHONPATH=$(pwd):$PYTHONPATH /data/huangyan/isaac-sim-4-2/python.sh main.py --config-name level4_LiquidMixing --no-video --backend gpu
```
