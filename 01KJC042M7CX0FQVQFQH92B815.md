---
title: 20251012 - 分析isaac_ros_visual_slam的原理 (机器人视觉感知)
confluence_page_id: 4358874
created_at: 2025-10-12T15:25:09+00:00
updated_at: 2025-10-13T08:35:10+00:00
---

实验文档: <https://nvidia-isaac-ros.github.io/concepts/visual_slam/cuvslam/tutorial_isaac_sim.html>

包文档: <https://nvidia-isaac-ros.github.io/repositories_and_packages/isaac_ros_visual_slam/isaac_ros_visual_slam/index.html#quickstart>

# 与Nvblox实验的区别

Nvblox中, 其关于机器人在base坐标系中的位置(位姿), 是Issac Sim给出的GT位姿

而在isaac_ros_visual_slam中, 这个位姿需要估算

![image2025-10-12 23:15:49.png](/assets/01KJC042M7CX0FQVQFQH92B815/image2025-10-12%2023%3A15%3A49.png)

# 安装

进入Ros2 DEV容器, 下载assets: 

```
NGC_ORG="nvidia"
NGC_TEAM="isaac"
PACKAGE_NAME="isaac_ros_visual_slam"
NGC_RESOURCE="isaac_ros_visual_slam_assets"
NGC_FILENAME="quickstart.tar.gz"
MAJOR_VERSION=3
MINOR_VERSION=2
VERSION_REQ_URL="https://catalog.ngc.nvidia.com/api/resources/versions?orgName=$NGC_ORG&teamName=$NGC_TEAM&name=$NGC_RESOURCE&isPublic=true&pageNumber=0&pageSize=100&sortOrder=CREATED_DATE_DESC"
AVAILABLE_VERSIONS=$(curl -s \
    -H "Accept: application/json" "$VERSION_REQ_URL")
LATEST_VERSION_ID=$(echo $AVAILABLE_VERSIONS | jq -r "
    .recipeVersions[]
    | .versionId as \$v
    | \$v | select(test(\"^\\\\d+\\\\.\\\\d+\\\\.\\\\d+$\"))
    | split(\".\") | {major: .[0]|tonumber, minor: .[1]|tonumber, patch: .[2]|tonumber}
    | select(.major == $MAJOR_VERSION and .minor <= $MINOR_VERSION)
    | \$v
    " | sort -V | tail -n 1
)
if [ -z "$LATEST_VERSION_ID" ]; then
    echo "No corresponding version found for Isaac ROS $MAJOR_VERSION.$MINOR_VERSION"
    echo "Found versions:"
    echo $AVAILABLE_VERSIONS | jq -r '.recipeVersions[].versionId'
else
    mkdir -p ${ISAAC_ROS_WS}/isaac_ros_assets && \
    FILE_REQ_URL="https://api.ngc.nvidia.com/v2/resources/$NGC_ORG/$NGC_TEAM/$NGC_RESOURCE/\
versions/$LATEST_VERSION_ID/files/$NGC_FILENAME" && \
    curl -LO --request GET "${FILE_REQ_URL}" && \
    tar -xf ${NGC_FILENAME} -C ${ISAAC_ROS_WS}/isaac_ros_assets && \
    rm ${NGC_FILENAME}
fi
``` 

安装包: 

```
sudo apt-get install -y ros-humble-isaac-ros-visual-slam
``` 

使用rosbag进行测试: 

```
在窗口1, 进入容器后: 
ros2 launch isaac_ros_examples isaac_ros_examples.launch.py launch_fragments:=visual_slam \
interface_specs_file:=${ISAAC_ROS_WS}/isaac_ros_assets/isaac_ros_visual_slam/quickstart_interface_specs.json \
rectified_images:=false
 
在窗口2, 进入容器后, 回放bag: 
ros2 bag play ${ISAAC_ROS_WS}/isaac_ros_assets/isaac_ros_visual_slam/quickstart_bag --remap  \
/front_stereo_camera/left/image_raw:=/left/image_rect \
/front_stereo_camera/left/camera_info:=/left/camera_info_rect \
/front_stereo_camera/right/image_raw:=/right/image_rect \
/front_stereo_camera/right/camera_info:=/right/camera_info_rect \
/back_stereo_camera/left/image_raw:=/rear_left/image_rect \
/back_stereo_camera/left/camera_info:=/rear_left/camera_info_rect \
/back_stereo_camera/right/image_raw:=/rear_right/image_rect \
/back_stereo_camera/right/camera_info:=/rear_right/camera_info_rect
 
在桌面进入容器后, 运行rviz: 
rviz2 -d $(ros2 pkg prefix isaac_ros_visual_slam --share)/rviz/default.cfg.rviz
``` 

能显示少量点云图 (跟教程上一样): 

![image2025-10-12 23:36:11.png](/assets/01KJC042M7CX0FQVQFQH92B815/image2025-10-12%2023%3A36%3A11.png)

# 与Isaac Sim一起实验

在Isaac Sim中, 加载ROS2的sample scene, 进行play
    
    
    在桌面进入容器后, 运行rviz: 

```
rviz2 -d $(ros2 pkg prefix isaac_ros_visual_slam --share)/rviz/isaac_sim.cfg.rviz
``` 

在容器内, 使用ROS2命令, 让机器人运动:

```
ros2 topic pub --once /cmd_vel geometry_msgs/msg/Twist "{linear: {x: 0.2, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 0.2}}"
``` 

发现rviz中, 有时能成功, 有时不能显示点云.

成功的情况: 

![image2025-10-13 16:22:41.png](/assets/01KJC042M7CX0FQVQFQH92B815/image2025-10-13%2016%3A22%3A41.png)

对失败的情况的排查: 

  - 发现是QoS问题, 在 /opt/ros/humble/share/isaac_ros_visual_slam/launch/isaac_ros_visual_slam_isaac_sim.launch.py 中修改代码, 可以解决问题: 
    - ![image2025-10-13 16:25:10.png](/assets/01KJC042M7CX0FQVQFQH92B815/image2025-10-13%2016%3A25%3A10.png)

这里看到的点云是稀疏的, 是因为SLAM的主要作用是帮助机器人定位到自己的全局位置, 而不是为了避障, 所以SLAM只反应了全局的特征点 (很少个), 而不是构建整体地图: 

好的，这是对以上信息的简要总结：

  1. **点云稀疏的原因** ：

     - 您使用的`cuVSLAM`是一种**稀疏VSLAM** 方法，它只提取和跟踪环境中几百到几千个最稳定、最易识别的**特征点（地标）** ，而不是重建整个场景。因此，您看到的点云是这些地标的集合，自然是稀疏的。
  2. **稀疏点云的核心作用** ：

     - **定位与轨迹生成** ：这个稀疏点云是系统的**“路标地图”**。机器人通过持续匹配当前视野中的特征点与地图中的地标，来精确计算自身的实时位置和姿态（轨迹）。它是精确轨迹的** 基础**，而非结果。
     - **消除误差** ：它被用于回环检测和全局优化，能大幅消除长时间运行时累积的定位误差，确保全局地图的一致性。
  3. **功能与局限** ：

     - **不能直接用于避障** ：稀疏点云点与点之间存在大量未知区域，无法提供足够的环境表面信息来判断障碍物，因此不能直接用于路径规划和避障。
     - **为避障提供定位支持** ：它提供的高精度全局定位，是整个导航系统（如Nav2）的基石。导航系统需要结合`cuVSLAM`提供的“我在哪”信息，以及**其他传感器（如深度相机、激光雷达）**提供的“障碍物在哪”的稠密信息，才能实现安全有效的避障和导航。

**一句话概括：稀疏点云是`cuVSLAM`用来实现高精度自我定位的“骨架地图”，它负责解决“我在哪”的问题，但不负责解决“路在哪”的避障问题。**

实验中还有 保存地图点云 和 读取地图点云 的部分, 先忽略
