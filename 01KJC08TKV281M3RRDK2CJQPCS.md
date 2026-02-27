---
title: 20251125 - 重新梳理机械臂的运行环境初始化命令
confluence_page_id: 4359335
created_at: 2025-11-25T05:08:48+00:00
updated_at: 2025-11-25T06:10:26+00:00
---

# slider机器

登录slider机器:

```
ssh huangyan@192.168.22.12
``` 

进入容器: 

```
export ISAAC_ROS_WS=/data/huangyan/isaac_ros-dev/
cd $ISAAC_ROS_WS && ./src/isaac_ros_common/scripts/run_dev.sh
``` 

初始化环境: 

```
export http_proxy=http://127.0.0.1:7890
export https_proxy=http://127.0.0.1:7890
export no_proxy=127.0.0.1,localhost,ucloud.cn
export ROS_DOMAIN_ID=98
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
export ROS_DISTRO=humble
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/data/huangyan/isaac-sim/exts/omni.isaac.ros2_bridge/humble/lib/
export CUDA_VISIBLE_DEVICES=7
export __GLX_VENDOR_LIBRARY_NAME=nvidia
export DISPLAY=:0
export CYCLONEDDS_URI=/workspaces/isaac_ros-dev/a.xml
source install/setup.bash
``` 

开启slider: 

```
ros2 run mycobot_280 slider_control_adaptive_gripper use_sim_time:=true
``` 

# moveit机器

登录机器: 

```
ssh ubuntu@117.50.172.161
``` 

进入容器: 

```
export ISAAC_ROS_WS=/data/huangyan/isaac_ros-dev/
cd $ISAAC_ROS_WS && ./src/isaac_ros_common/scripts/run_dev.sh
``` 

初始化环境: 

```
export http_proxy=http://127.0.0.1:7890
export https_proxy=http://127.0.0.1:7890
export no_proxy=127.0.0.1,localhost,ucloud.cn
export ROS_DOMAIN_ID=98
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
export ROS_DISTRO=humble
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/data/huangyan/isaac-sim/exts/omni.isaac.ros2_bridge/humble/lib/
export CUDA_VISIBLE_DEVICES=7
export __GLX_VENDOR_LIBRARY_NAME=nvidia
export DISPLAY=:0
export CYCLONEDDS_URI=/workspaces/isaac_ros-dev/a.xml
source install/setup.bash
``` 

启动moveit脚本: 

```
ros2 launch huangyan/moveit_and_isaac_sim/mycobot_280_real_moveit.launch.py use_rviz:=true use_sim_time:=true enable_apriltag:=true
``` 

启动操作测试脚本: 

```
cd huangyan/moveit_and_isaac_sim/
python test_real.py --use-sim
```
