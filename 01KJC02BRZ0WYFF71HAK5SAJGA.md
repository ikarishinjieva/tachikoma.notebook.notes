---
title: 20251008 - 列出ROS2样例表, 尝试ROS2样例isaac_ros_apriltag, 解决OOM报错
confluence_page_id: 4358780
created_at: 2025-10-07T16:57:34+00:00
updated_at: 2025-10-10T10:46:06+00:00
---

# 背景

经过一个假期折腾, 可以安装好Isaac Sim和ROS2 等, 但不能解决 以下样例的OOM的问题: 

```
# https://nvidia-isaac-ros.github.io/reference_workflows/isaac_manipulator/tutorials/tutorial_isaac_sim.html
ros2 launch isaac_manipulator_pick_and_place isaac_sim_workflows.launch.py workflow_type:=pose_to_pose
``` 

更多样例列表: <https://nvidia-isaac-ros.github.io/getting_started/index.html#isaac-sim-tutorials>

下面给你一条“从易到难”的最小学习路径，只挑简单、上手快、覆盖面广的 Isaac Sim 教程。每步都标注学习目标、预计耗时与容易踩的坑，方便你快速入门并建立成就感。

阶段 0：环境确认（5–10 分钟）

  - 教程入口：Isaac Sim Tutorials（文档已注明最近验证在 Isaac Sim 4.5.0/4.2.0）
  - 目标：确保你能启动 Isaac Sim，并能用 Docker/ROS 2 Humble 的 Isaac ROS 开发容器跑起来。
  - 坑点：显卡驱动/CUDA 版本与容器权限；尽量用官方 Isaac ROS Dev Docker 镜像。

阶段 1：图像与标记的最小闭环（30–45 分钟）

  1. Tutorial for AprilTag Detection with Isaac Sim

  - 为什么先学它：不涉及 DNN 推理，依赖少、反馈直观（能在可视化里看到检测框和位姿）。
  - 学习目标：
    - 认识仿真相机、相机话题与坐标系发布。
    - 会在 RViz/Foxglove 可视化 tag 检测结果和 TF。
  - 常见坑：
    - 相机内参/畸变与仿真配置不一致导致姿态抖动。
    - 话题名和命名空间对不齐。

阶段 2：最简单的 DNN 推理体验（30–60 分钟）  
2) Tutorial for DNN Object Detection with Isaac Sim

  - 为什么：目标检测是最直观的 DNN 例子，推理结果可视化清晰；比分割/深度更易懂。
  - 学习目标：
    - 跑通 TensorRT 模型（或 Triton），理解预处理/后处理节点与 NITROS 数据流。
    - 在仿真场景中识别常见物体，学会调整模型与阈值。
  - 常见坑：
    - 模型引擎不匹配平台（Jetson/x86）或精度（FP16/INT8）转换失败。
    - 图像编码（RGB/BGR）或分辨率与模型输入不匹配。

阶段 3：双目深度的“零门槛”版本（30–45 分钟）  
3) Tutorial for SGM Stereo Disparity with Isaac Sim

  - 为什么：传统 SGM 无需 DNN/模型准备，比 ESS 简单；让你快速感知视差/深度流程。
  - 学习目标：
    - 理解双目相机基线、标定、视差到深度的转换。
    - 观察不同分辨率/基线对深度质量与实时性的影响。
  - 常见坑：
    - 左右目时间戳不同步或畸变参数缺失，导致视差噪声大。
    - 输入图像尺寸需要偶数宽高。

阶段 4：基础建图与可视化（45–90 分钟）  
4) Tutorial for Nvblox with Isaac Sim

  - 为什么：把相机/深度转成 3D TSDF/ESDF，是通向导航/避障的必修课；整体配置比导航栈简单。
  - 学习目标：
    - 输出 TSDF/ESDF 点云/切片，理解距离场在避障中的意义。
    - 学会话题对接（来自 SGM 的深度或仿真深度）与帧对齐。
  - 常见坑：
    - 深度尺度/单位与相机外参不正确导致地图漂移或穿插。
    - 需要合理限制最大深度与更新半径以保证性能。

可选＋：最轻量的定位体验（30–45 分钟）  
5) Tutorial for Visual SLAM with Isaac Sim

  - 为什么：让机器人“动起来”，获得轨迹与地图；稍复杂但回报大。
  - 学习目标：
    - 跑通 cuVSLAM，查看轨迹、回环，了解如何保存/加载地图。
  - 常见坑：
    - 纹理过少或光照不合理会影响跟踪；相机频率/QoS 设置不当会丢帧。

再往后（等你熟练后再碰）

  - DNN Stereo Depth Estimation with Isaac Sim（ESS）：需准备模型，配置略复杂，但深度质量更佳。
  - Freespace Segmentation / Bi3D：偏导航决策，涉及模型或多阈值配置。
  - FoundationPose / cuMotion MoveIt / Manipulator Workflows：机械臂完整感知-规划-控制链路，集成度高，初学者建议后学。
  - NITROS Bridge / Mission Client：系统与任务编排层面的互通与控制，适合作为中后期的工程化目标。

一键清单（按顺序跑）

  - AprilTag Detection → DNN Object Detection → SGM Stereo Disparity → Nvblox → Visual SLAM（可选）
  - 每一步都用 Foxglove 或 RViz2 做可视化，确保话题与坐标系一致。

告诉我你的硬件平台（Jetson Orin 还是 x86 独显）、是否使用 Docker，以及更偏向“移动机器人”还是“机械臂”，我可以把上述每一步再细化成具体命令、示例 launch 文件与常见报错的对症排查清单。

# 尝试isaac_ros_apriltag

文档: <https://nvidia-isaac-ros.github.io/concepts/fiducials/apriltag/tutorial_isaac_sim.html>

参考[20251006 - 尝试将Isaac Sim和WebRTC连通] 的启动过程: 

启动Issac Sim:

```
10.186.16.136:
dcv create-session test-issac
  
使用 DCV viewer 连接到桌面
 
 
桌面窗口1: 启动Issac Sim:
cd /data/huangyan/isaac-sim
export http_proxy=http://10.186.16.136:7890
export https_proxy=http://10.186.16.136:7890
export FASTRTPS_DEFAULT_PROFILES_FILE=/data/huangyan/isaac-sim/rtps_udp_profile.xml
./isaac-sim.sh
``` 

配置dev环境: 

```
10.186.16.136:
export ISAAC_ROS_WS=/data/huangyan/isaac_ros-dev/
cd $ISAAC_ROS_WS && ./src/isaac_ros_common/scripts/run_dev.sh 进容器
 
容器内: 
export http_proxy=http://10.186.16.136:7890
export https_proxy=http://10.186.16.136:7890
export FASTRTPS_DEFAULT_PROFILES_FILE=/usr/local/share/middleware_profiles/rtps_udp_profile.xml
cd ${ISAAC_ROS_WS}
source install/setup.bash
 
下载asset: 
NGC_ORG="nvidia"
NGC_TEAM="isaac"
PACKAGE_NAME="isaac_ros_apriltag"
NGC_RESOURCE="isaac_ros_apriltag_assets"
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
 
# 安装
sudo apt-get install -y ros-humble-isaac-ros-apriltag
``` 

注意: 先不安装jetson: <https://nvidia-isaac-ros.github.io/getting_started/hardware_setup/compute/jetson_vpi.html>

验证: 

```
# 用demo验证?
sudo apt-get install -y ros-humble-isaac-ros-examples
ros2 launch isaac_ros_examples isaac_ros_examples.launch.py launch_fragments:=apriltag interface_specs_file:=${ISAAC_ROS_WS}/isaac_ros_assets/isaac_ros_apriltag/quickstart_interface_specs.json

## 再开一个docker: 模拟一个image流
ros2 bag play -l ${ISAAC_ROS_WS}/isaac_ros_assets/isaac_ros_apriltag/quickstart.bag --remap image:=image_rect camera_info:=camera_info_rect

## 再桌面上再开一个docker, 查看结果: 
ros2 topic echo /tag_detections
 
# ros2 topic echo /tag_detections 的 结果: 
header:
  stamp:
    sec: 336
    nanosec: 283350871
  frame_id: carter_camera_stereo_left
detections:
- family: tag36h11
  id: 0
  center:
    x: 746.0
    y: 377.0
    z: 0.0
  corners:
  - x: 786.0
    y: 337.0
    z: 0.0
  - x: 786.0
    y: 417.0
    z: 0.0
  - x: 706.0
    y: 417.0
    z: 0.0
  - x: 706.0
    y: 337.0
    z: 0.0
  pose:
    header:
      stamp:
        sec: 0
        nanosec: 0
      frame_id: ''
    pose:
      pose:
        position:
          x: 0.29172515869140625
          y: 0.04671391844749451
          z: 4.034609794616699
        orientation:
          x: 0.0
          y: 0.0
          z: 0.7071067690849304
          w: 0.7071067690849304
      covariance:
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
- family: tag36h11
  id: 2
  center:
    x: 568.0
    y: 377.0
    z: 0.0
  corners:
  - x: 608.0
    y: 337.0
    z: 0.0
  - x: 608.0
    y: 417.0
    z: 0.0
  - x: 528.0
    y: 417.0
    z: 0.0
  - x: 528.0
    y: 337.0
    z: 0.0
  pose:
    header:
      stamp:
        sec: 0
        nanosec: 0
      frame_id: ''
    pose:
      pose:
        position:
          x: -0.19815294444561005
          y: 0.04671391844749451
          z: 4.034609794616699
        orientation:
          x: 0.0
          y: 0.0
          z: 0.7071067690849304
          w: 0.7071067690849304
      covariance:
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
      - 0.0
---

``` 

与Isaac Sim并用: 

在Isaac Sim中, 打开Robotic Examples:

![image2025-10-8 16:37:31.png](/assets/01KJC02BRZ0WYFF71HAK5SAJGA/image2025-10-8%2016%3A37%3A31.png)

选择Sample: 

![image2025-10-8 16:38:16.png](/assets/01KJC02BRZ0WYFF71HAK5SAJGA/image2025-10-8%2016%3A38%3A16.png)

加载完成

![image2025-10-8 16:43:14.png](/assets/01KJC02BRZ0WYFF71HAK5SAJGA/image2025-10-8%2016%3A43%3A14.png)

进行Play

启动ros2任务: 

```
进入容器: 
 
开启pipeline: 
ros2 launch isaac_ros_apriltag isaac_ros_apriltag_isaac_sim_pipeline.launch.py
 
``` 

资源使用: 

![image2025-10-8 16:44:38.png](/assets/01KJC02BRZ0WYFF71HAK5SAJGA/image2025-10-8%2016%3A44%3A38.png)

在桌面, 启动rviz2查看结果: 

```
export DISPLAY=:0
rviz2 -d $(ros2 pkg prefix isaac_ros_apriltag --share)/rviz/default.rviz
``` 

结果: 

![image2025-10-8 16:52:31.png](/assets/01KJC02BRZ0WYFF71HAK5SAJGA/image2025-10-8%2016%3A52%3A31.png)

重新再Isaac Sim中Play: 资源消耗

![image2025-10-8 16:55:22.png](/assets/01KJC02BRZ0WYFF71HAK5SAJGA/image2025-10-8%2016%3A55%3A22.png)

仍然会OOM, 需要调查原因

# 分析isaac_ros_apriltag的原理, 以及OOM的原因

ROS2 sample的场景中, 使用了 Nova Carter 机器人, 参考文档: <https://docs.isaacsim.omniverse.nvidia.com/5.1.0/assets/nova_carter_landing_page.html>

其提供了很多接口

分析isaac_ros_apriltag的启动文件: 

```
admin@dell-136:/workspaces/isaac_ros-dev$ cat /opt/ros/humble/share/isaac_ros_apriltag/launch/isaac_ros_apriltag_isaac_sim_pipeline.launch.py
# SPDX-FileCopyrightText: NVIDIA CORPORATION & AFFILIATES
# Copyright (c) 2021-2024 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
# SPDX-License-Identifier: Apache-2.0

import launch
from launch_ros.actions import ComposableNodeContainer
from launch_ros.descriptions import ComposableNode

def generate_launch_description():
    apriltag_node = ComposableNode(
        package='isaac_ros_apriltag',
        plugin='nvidia::isaac_ros::apriltag::AprilTagNode',
        name='apriltag',
        remappings=[('image', 'front_stereo_camera/left/image_rect_color'),
                    ('camera_info', 'front_stereo_camera/left/camera_info')],
        parameters=[{'size': 0.32,
                     'max_tags': 64,
                     'tile_size': 4}]
    )

    apriltag_container = ComposableNodeContainer(
        package='rclcpp_components',
        name='apriltag_container',
        namespace='',
        executable='component_container_mt',
        composable_node_descriptions=[
            apriltag_node,
        ],
        output='screen'
    )

    return launch.LaunchDescription([apriltag_container])
``` 

列出启动树: 

```
ROS 2 启动结构树 (基于已提供信息)
│
└─ **容器进程 (Container Process)**
   │
   ├─ **启动的可执行文件**: component_container_mt
   │  └─ (来源: 启动脚本中的 `executable` 字段)
   │
   ├─ **容器的ROS节点名**: apriltag_container
   │  └─ (来源: 启动脚本中的 `name` 字段)
   │
   └─ **加载的可组合节点 (Loaded Composable Node)**
      │
      ├─ **节点名**: apriltag
      │  └─ (来源: 启动脚本中 ComposableNode 的 `name` 字段)
      │
      ├─ **节点实现**:
      │  ├─ **功能包 (Package)**: isaac_ros_apriltag
      │  └─ **插件 (Plugin)**: nvidia::isaac_ros::apriltag::AprilTagNode
      │     └─ (来源: 启动脚本中的 `package` 和 `plugin` 字段)
      │
      ├─ **配置的参数 (Parameters)**
      │  ├─ size: 0.32
      │  ├─ max_tags: 64
      │  └─ tile_size: 4
      │     └─ (来源: 启动脚本中的 `parameters` 列表)
      │
      ├─ **订阅的话题 (Subscribed Topics)**
      │  ├─ /front_stereo_camera/left/image_rect_color
      │  │  └─ (来源: 启动脚本 `remappings` 将内部话题 `image` 重定向至此)
      │  │
      │  └─ /front_stereo_camera/left/camera_info
      │     └─ (来源: 启动脚本 `remappings` 将内部话题 `camera_info` 重定向至此。此话题在参考文档的表格中被列出)
      │
      └─ **发布的话题 (Published Topics)**
         └─ **[ 未知 ]**
            └─ (说明: 节点会发布哪些话题是由其内部代码实现决定的。您提供的启动脚本和参考文档中，均未描述 `AprilTagNode` 会发布哪些话题。因此，基于现有信息，其输出是未知的。)
``` 

其中, 发布的话题, 需要通过ros2 node info /apriltag来获取

阅读包的文档: <https://nvidia-isaac-ros.github.io/repositories_and_packages/isaac_ros_apriltag/isaac_ros_apriltag/index.html#package-name>, 提取重要信息: 

```
isaac_ros_apriltag 核心知识树
│
├──  核心功能: 利用NVIDIA GPU加速检测AprilTag，并将其位姿集成到ROS坐标系中
│
├──  运行与使用 (How to Run)
│   │
│   ├── 1. 启动检测节点 (Launch Node)
│   │   │   └── ros2 launch isaac_ros_examples isaac_ros_examples.launch.py launch_fragments:=<INPUT>,apriltag
│   │       │
│   │       └── <INPUT> 可替换为:
│   │           ├── rosbag (使用录制好的数据文件)
│   │           ├── realsense_mono_rect (使用RealSense相机)
│   │           ├── argus_mono,rectify_mono (使用Hawk相机)
│   │           └── zed_mono_rect (使用ZED相机)
│   │
│   └── 2. 提供输入数据 (Provide Input)
│       │
│       ├── 方式一: 播放Rosbag (模拟相机)
│       │   └── ros2 bag play <your_bag_file>.bag --remap image:=image_rect camera_info:=camera_info_rect
│       │
│       └── 方式二: 运行真实相机驱动 (如RealSense, ZED等)
│
├── 可配置参数 (ROS Parameters)
│   │
│   ├── size: 标签的物理边长 (单位: 米), 用于计算3D位姿
│   ├── tag_family: 标签的家族 (如 'tag36h11')
│   └── backends: 计算后端 ('CUDA', 'CPU', 'PVA'), 'CUDA'为GPU加速
│
└── 节点接口 (Topics & TF)
    │
    ├── 输入 (Subscribed Topics)
    │   │
    │   ├── /image: 图像流 (sensor_msgs/Image)
    │   │   └── 来源: 真实相机 或 Rosbag回放
    │   │
    │   └── /camera_info: 相机内参 (sensor_msgs/CameraInfo)
    │       └── 作用: 用于将2D像素坐标转换为3D空间坐标
    │
    └── 输出 (Published Topics & TF)
        │
        ├── /tag_detections (isaac_ros_apriltag_interfaces/AprilTagDetectionArray)
        │   │
        │   ├── 内容: 详细的检测结果数组 (ID, 像素角点, 位姿等)
        │   └── 坐标系: 位姿(pose)是相对于【相机坐标系 (camera_link)】的
        │   └── 好比: "本地寻宝图" - 从相机出发如何找到标签
        │
        └── /tf (tf2_msgs/TFMessage)
            │
            ├── 内容: 发布一个从【相机坐标系 (camera_link)】到【标签坐标系 (tag_X)】的坐标变换
            ├── 作用: 将标签的位置广播到整个ROS的全局坐标变换树中
            └── 好比: "更新GPS系统" - 报告新地标相对于已知地标(相机)的位置
            │
            └── 最终效果 (通过查询TF系统实现):
                └── 任何节点都可以轻松查询到【标签】相对于【机器人底盘 (base_link)】或其他任何坐标系的位姿，TF系统会自动完成 `base_link` -> `camera_link` -> `tag_X` 的计算。
``` 

分析相关代码: <https://raw.githubusercontent.com/NVIDIA-ISAAC-ROS/isaac_ros_apriltag/refs/heads/main/isaac_ros_apriltag/src/apriltag_node.cpp>

  - 这段代码主要是对cuAprilTagsDetect算法的输入/输出进行包装, 核心算法已经由nvidia实现

```
AprilTag 节点代码执行流程树
│
└─1. 节点启动与初始化 (在 `ros2 run` 或 `ros2 launch` 时发生)
    │
    ├─ a. 构造函数被调用 (`AprilTagNode::AprilTagNode`)
    │   │
    │   ├─ ① **加载配置参数**: 从ROS参数服务器读取用户配置。
    │   │   └── **知识点**: 这是节点灵活性的来源。用户可以通过launch文件或命令行轻松更改标签尺寸(`size`)、家族(`tag_family`)和性能设置(`max_tags`, `backends`)，而无需重新编译代码。
    │   │
    │   ├─ ② **选择算法引擎**: 根据`backends`参数的值，决定使用哪个底层实现。
    │   │   ├── 如果 `backends` 仅为 "CUDA" -> 选择 `CUAprilTagImpl` (专用库)
    │   │   └── 否则 -> 选择 `VPIAprilTagImpl` (通用库)
    │   │   └── **知识点 (策略模式)**: 这是一个非常优雅的设计。主节点逻辑与具体算法实现解耦。未来如果想支持第三种检测库，只需再写一个`Impl`结构体，在这里加一个判断分支即可，完全不影响其他代码。
    │   │
    │   └─ ③ **设置消息接口**: 准备好与ROS世界沟通的通道。
    │       ├── **创建发布者**: 建立 `/tag_detections` 和 `/tf` 两个话题的发布通道。
    │       └── **设置同步订阅者**:
    │           ├── 目的：为了确保收到的 `/image` (图像) 和 `/camera_info` (相机参数) 具有**完全相同的时间戳**。
    │           └── **知识点 (Message Filters)**: 这是处理多传感器数据的关键。3D位姿计算必须使用与图像帧完全对应的相机参数，否则结果会出错。`message_filters`库通过缓存和匹配时间戳来保证这种严格的同步。
    │
    └─ b. 节点进入“自旋”状态，等待消息...
.
.
.
└─2. 核心处理循环 (当一对同步的消息到达时触发)
    │
    ├─  a. 回调函数被触发 (`AprilTagNode::CameraImageCallback`)
    │   │
    │   ├─ ① **首次运行检查**: 判断算法引擎是否已经初始化。
    │   │   └── 如果未初始化 -> 调用 `impl_->Initialize(...)`
    │   │       └──  **知识点 (延迟初始化)**: 初始化过程需要知道图像的宽度、高度等信息，而这些信息只有在收到第一帧数据后才能确定。因此，初始化被推迟到第一次回调时执行。
    │   │
    │   └─ ② **任务委派**: 将图像和相机数据交给在步骤 `1.a.②` 中选定的算法引擎实例去处理。
    │       └── `impl_->OnCameraFrame(...)`
    │       └──  **知识点 (职责分离)**: `AprilTagNode` 类本身不关心如何检测，它只负责ROS通信和任务调度。实际的计算工作全部由 `impl_` 对象完成。
    │
    └─  b. 算法引擎执行检测 (以 `CUAprilTagImpl::OnCameraFrame` 为例)
        │
        ├─ ① **准备输入**: 将ROS的图像消息(`NitrosImageView`)适配成 `cuAprilTags` 库能识别的格式 (`cuAprilTagsImageInput_t`)。
        │   └── **知识点 (数据包装)**: 这是典型的“胶水代码”，它的作用是连接两个不同系统（ROS 和 cuAprilTags库）之间的数据结构。
        │
        ├─ ② **调用核心库**: 执行 `cuAprilTagsDetect(...)` 函数。
        │   └──  **知识点 (调用外部库)**: 所有的GPU加速、图像分析、几何计算等复杂工作都在这个函数内部完成。我们的C++代码只是这个强大“黑盒”的使用者。
        │
        ├─ ③ **处理输出结果**: 核心库返回了检测到的标签数量和包含所有信息的数组。
        │   │
        │   └─ **进入循环，遍历每个检测到的标签**:
        │       │
        │       ├─ **数据解包**: 从 `cuAprilTagsID_t` 结构体中提取ID、角点、平移、旋转矩阵等原始数据。
        │       │
        │       ├─ **格式转换**:
        │       │   ├── **位姿转换**: 调用 `ToTransformMsg()` 将库返回的旋转矩阵 (`Eigen::ColMajor`) 转换为ROS标准的四元数 (`geometry_msgs::msg::Quaternion`)。
        │       │   └──  **知识点**: 不同库和系统间对坐标和姿态的表示方式可能不同（如矩阵的行/列主序，四元数的w,x,y,z顺序等），这种转换是保证系统兼容性的必要步骤。
        │       │
        │       └─ **构建两种ROS消息**: 用转换好的数据填充两种不同的消息。
        │           ├── **消息A (`AprilTagDetection`)**: 包含ID、像素坐标、中心点，以及相对于【相机】的位姿。用于详细分析。
        │           └── **消息B (`TransformStamped`)**: 只包含从【相机坐标系】到【标签坐标系】的坐标变换。用于融入全局TF树。
        │
        └─ ④ **发布最终结果**:
            ├── `node.detections_pub_->publish(...)`: 将所有标签的“详细报告”打包成一个数组一次性发布。
            └── `node.tf_pub_->publish(...)`: 将所有标签的“坐标变换”打包成一个消息一次性发布。
            └──  **知识点 (批量发布)**: 在单次回调中将同一帧的所有结果打包发布，比逐个发布更高效，可以减少网络负载和消息处理的开销。
``` 

# 发现ros2 topic info -v "/tag_detections" --no-daemon 会 OOM

现象: 

  - ROS_DOMAIN_ID=0时, ros2 topic info -v "/tag_detections" --no-daemon 会OOM
  - ROS_DOMAIN_ID=其他值时, 命令可以正确处理

分析: 

使用strace获取系统调用: 

```
strace -o trace.log -f ros2 topic info -v "/tag_detections" --no-daemon
``` 

关注mmap和brk调用, 分别列出其调用, 交给大模型判断, 锁定在mmap调用上: 

证据分析：一场有预谋、有组织的内存攻击

这份日志可以清晰地分为三个阶段：

#### 阶段一：前奏 - 零星但频繁的分配 (日志开头)
    
    
    56165 mmap(NULL, 1048576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7067050a9000
    ...
    56165 mmap(NULL, 1048576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x706704f47000
    ...
    56165 mmap(NULL, 1048576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x706704e22000
    

  - **行为** : 主线程 (`56165`) 在不断地申请大小为 `1048576` 字节（**刚好 1 MiB** ）的内存块。
  - **解读** : 这是内存泄漏的典型特征。某个循环被触发，在循环的每一轮，程序都在申请一个固定大小的大内存块。这可能是为某个数据结构（比如缓存、历史记录等）分配空间，但由于 Bug，这个分配行为没有上限，永不停止。

#### 阶段二：爆发 - 多线程协同的疯狂分配 (日志中段)
    
    
    56165 mmap(NULL, 8392704, PROT_NONE, MAP_PRIVATE|MAP_ANONYMOUS|MAP_STACK, -1, 0 
    56166 mmap(NULL, 33554432, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0 
    
    56165 mmap(NULL, 8392704, PROT_NONE, MAP_PRIVATE|MAP_ANONYMOUS|MAP_STACK, -1, 0 
    56167 mmap(NULL, 33554432, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0 
    
    56165 mmap(NULL, 8392704, PROT_NONE, MAP_PRIVATE|MAP_ANONYMOUS|MAP_STACK, -1, 0 
    56168 mmap(NULL, 33554432, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0 
    

这是最震撼的部分！

  - **行为** :

    1. 主线程 (`56165`) 在疯狂地创建新线程。我们怎么知道？`mmap` 调用中出现了 `MAP_STACK` 标志，这是在为新线程的**调用栈** 分配内存。每次分配 `8392704` 字节（**刚好 8 MiB** ），这是 Linux 上线程栈的典型默认大小。
    2. 每一个新创建的线程（`56166`, `56167`, `56168`, ... 一直到 `56228`）**刚一出生，立刻就开始申请巨额内存** ！每个线程都在申请 `33554432` 字节（**刚好 32 MiB** ）的内存块。
  - **解读** : 这描绘了一个恐怖的场景：Fast DDS 在收到网络上的发现数据后，它的响应逻辑**不是** 在一个循环里分配内存，而是**在一个循环里创建新的线程，并且每个新线程都会立即申请一大块内存** 。

这就像一个细胞不是自己癌变变大，而是癌变后开始疯狂分裂，每个分裂出的新细胞都是一个巨大的肿瘤。这种模式的内存增长速度是指数级的，比单线程循环分配要快得多得多！

#### 阶段三：终局 - 天文数字级别的内存预留 (日志末尾)
    
    
    56237 mmap(NULL, 37803397120, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0 
    56237 mmap(NULL, 46455255040, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0 
    

这已经超出了“泄漏”的范畴，这是程序在试图“吞噬宇宙”。

  - **行为** : 某个线程 (`56237`) 试图一次性申请**天文数字** 的内存。

    - `37,803,397,120` 字节 = **35.2 GiB**
    - `46,455,255,040` 字节 = **43.2 GiB**
  - **解读** : 为什么会出现这么离谱的数字？这几乎可以肯定是**内存计算溢出** 或**指针错误** 的结果。

    - **场景推测** : 某个负责计算需要分配多少内存的变量（比如一个 `int` 或 `size_t`），因为之前疯狂的分配和混乱的状态，它的值变成了一个不可思议的、巨大的垃圾值。
    - 程序拿着这个垃圾值去调用 `mmap`，试图向操作系统申请几十个 GB 的内存。
    - 当然，这个请求会立刻失败（`strace` 显示为 ``，很可能最终返回 `ENOMEM` 错误），但这个行为本身就宣告了程序的逻辑已经彻底崩溃。

分离出某一个大mmap调用的上下文: 

```
56233 recvfrom(11, "RTPS\2\2\1\17\1\17F7Y\300\342]\4\0\1\0\7\1\34\0\0\0\3\307\0\0\3\302"..., 65500, 0, {sa_family=AF_INET, sin_port=htons(58901), sin_addr=inet_addr("10.186.16.136")
}, [28 => 16]) = 112
56233 recvfrom(11,  <unfinished ...>
56233 <... recvfrom resumed>"RTPS\2\2\1\17\1\17F7Y\300\342]\4\0\1\0\16\1\f\0\1\17F7e\333\233;"..., 65500, 0, {sa_family=AF_INET, sin_port=htons(58901), sin_addr=inet_addr("10.186.16
.136")}, [28 => 16]) = 632
56233 mprotect(0x706644034000, 4096, PROT_READ|PROT_WRITE <unfinished ...>
56235 <... recvfrom resumed>"RTPS\2\2\1\17\1\17F7Y\300\342]\1\0\1\0\16\1\f\0\1\17F7e\333\233;"..., 65500, 0, {sa_family=AF_INET, sin_port=htons(46847), sin_addr=inet_addr("10.186.16
.136")}, [28 => 16]) = 256
56233 <... mprotect resumed>)           = 0
56233 recvfrom(11,  <unfinished ...>
56233 <... recvfrom resumed>"RTPS\2\2\1\17\1\17F7Y\300\342]\4\0\1\0\7\1\34\0\0\0\3\307\0\0\3\302"..., 65500, 0, {sa_family=AF_INET, sin_port=htons(58901), sin_addr=inet_addr("10.186
.16.136")}, [28 => 16]) = 112
56233 recvfrom(11,  <unfinished ...>
56233 <... recvfrom resumed>"RTPS\2\2\1\17\1\17F7Y\300\342]\4\0\1\0\16\1\f\0\1\17F7e\333\233;"..., 65500, 0, {sa_family=AF_INET, sin_port=htons(58901), sin_addr=inet_addr("10.186.16
.136")}, [28 => 16]) = 624
56231 setsockopt(14, SOL_SOCKET, SO_SNDTIMEO_OLD, "\0\0\0\0\0\0\0\0\376_\327\35\24\0\0\0", 16 <unfinished ...>
56233 mprotect(0x706644035000, 4096, PROT_READ|PROT_WRITE <unfinished ...>
56231 <... setsockopt resumed>)         = -1 EDOM (Numerical argument out of domain)
56233 <... mprotect resumed>)           = 0
56231 sendto(14, "RTPS\2\3\1\17\1\17F7e\333\233;\0\0\0\0\16\1\f\0\1\17F7Y\300\342]"..., 64, MSG_NOSIGNAL, {sa_family=AF_INET, sin_port=htons(7418), sin_addr=inet_addr("127.0.0.1")},
 16 <unfinished ...>
56237 mmap(NULL, 37803397120, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0 <unfinished ...>
56231 <... sendto resumed>)             = 64
56237 <... mmap resumed>)               = 0x705d6aa00000
56233 setsockopt(14, SOL_SOCKET, SO_SNDTIMEO_OLD, "\0\0\0\0\0\0\0\0\376_\327\35\24\0\0\0", 16) = -1 EDOM (Numerical argument out of domain)
56233 sendto(14, "RTPS\2\3\1\17\1\17F7e\333\233;\0\0\0\0\16\1\f\0\1\17F7Y\300\342]"..., 68, MSG_NOSIGNAL, {sa_family=AF_INET, sin_port=htons(7004), sin_addr=inet_addr("127.0.0.1")}, 16) = 68
56233 recvfrom(11,  <unfinished ...>
56233 <... recvfrom resumed>"RTPS\2\2\1\17\1\17F7Y\300\342]\4\0\1\0\7\1\34\0\0\0\4\307\0\0\4\302"..., 65500, 0, {sa_family=AF_INET, sin_port=htons(58901), sin_addr=inet_addr("10.186.16.136")}, [28 => 16]) = 112
56235 recvfrom(13,  <unfinished ...>
56235 <... recvfrom resumed>"RTPS\2\2\1\17\1\17F7Y\300\342]\1\0\1\0\7\1\34\0\0\0\2\4\0\0\1\3"..., 65500, 0, {sa_family=AF_INET, sin_port=htons(46847), sin_addr=inet_addr("10.186.16.136")}, [28 => 16]) = 112
56233 recvfrom(11,  <unfinished ...>
56233 <... recvfrom resumed>"RTPS\2\2\1\17\1\17F7Y\300\342]\0\0\1\0\16\1\f\0\1\17F7e\333\233;"..., 65500, 0, {sa_family=AF_INET, sin_port=htons(50205), sin_addr=inet_addr("10.186.16.136")}, [28 => 16]) = 624
56233 mprotect(0x706644036000, 4096, PROT_READ|PROT_WRITE <unfinished ...>
56233 <... mprotect resumed>)           = 0
56236 setsockopt(14, SOL_SOCKET, SO_SNDTIMEO_OLD, "\0\0\0\0\0\0\0\0\377_\327\35\24\0\0\0", 16 <unfinished ...>
56236 <... setsockopt resumed>)         = -1 EDOM (Numerical argument out of domain)
56236 sendto(14, "RTPS\2\3\1\17\1\17F7e\333\233;\0\0\0\0\t\1\10\0V\205\346h\246U!\267"..., 148, MSG_NOSIGNAL, {sa_family=AF_INET, sin_port=htons(7007), sin_addr=inet_addr("127.0.0.1")}, 16 <unfinished ...>
56236 <... sendto resumed>)             = 148
56165 newfstatat(AT_FDCWD, "/opt/ros/humble/local/lib/python3.10/dist-packages/rosidl_parser",  <unfinished ...>
56236 setsockopt(14, SOL_SOCKET, SO_SNDTIMEO_OLD, "\0\0\0\0\0\0\0\0\377_\327\35\24\0\0\0", 16 <unfinished ...>
56165 <... newfstatat resumed>{st_mode=S_IFDIR|0755, st_size=4096, ...}, 0) = 0
56236 <... setsockopt resumed>)         = -1 EDOM (Numerical argument out of domain)
56236 sendto(14, "RTPS\2\3\1\17\1\17F7e\333\233;\0\0\0\0\t\1\10\0V\205\346h\246U!\267"..., 148, MSG_NOSIGNAL, {sa_family=AF_INET, sin_port=htons(7001), sin_addr=inet_addr("127.0.0.1")}, 16 <unfinished ...>
56165 newfstatat(AT_FDCWD, "/opt/ros/humble/local/lib/python3.10/dist-packages/rosidl_parser/parser.py",  <unfinished ...>
``` 

其在发出127.0.0.1的包, 收10.186.16.136的包

感觉是自循环了

修改配置文件, 增加interfaceWhiteList, 指定为127.0.0.1: (参数参考文档: <https://fast-dds.docs.eprosima.com/en/v2.6.10/fastdds/xml_configuration/transports.html>)

```
<?xml version="1.0" encoding="UTF-8" ?>

<!--
Copyright (c) 2022, NVIDIA CORPORATION.  All rights reserved.

NVIDIA CORPORATION and its licensors retain all intellectual property
and proprietary rights in and to this software, related documentation
and any modifications thereto.  Any use, reproduction, disclosure or
distribution of this software and related documentation without an express
license agreement from NVIDIA CORPORATION is strictly prohibited.
-->

<license>NVIDIA Isaac ROS Software License</license>
<profiles xmlns="http://www.eprosima.com/XMLSchemas/fastRTPS_Profiles" >
    <transport_descriptors>
        <transport_descriptor>
            <transport_id>UdpTransport</transport_id>
            <type>UDPv4</type>
            <maxInitialPeersRange>400</maxInitialPeersRange>
            <interfaceWhiteList>
                <address>127.0.0.1</address>
            </interfaceWhiteList>
        </transport_descriptor>
    </transport_descriptors>

    <participant profile_name="udp_transport_profile" is_default_profile="true">
        <rtps>
            <userTransports>
                <transport_id>UdpTransport</transport_id>
            </userTransports>
            <useBuiltinTransports>false</useBuiltinTransports>
        </rtps>
    </participant>
</profiles>
``` 

export FASTRTPS_DEFAULT_PROFILES_FILE=/workspaces/isaac_ros-dev/a.xml

export ROS_DOMAIN_ID=0

ros2 topic info -v "/tag_detections" --no-daemon

不再OOM

总结: 

  - ROS_DOMAIN_ID=0时, ROS会出现发包和收包的IP地址不同, 导致不认识自己发出的包, 形成自循环. 
  - 当ROS_DOMAIN_ID不为0时, ROS能排除自己发的包, 说明发出的包是ROS_DOMAIN_ID=0的包, 是什么机制尚不清楚, 可能是一个多播信号

# 以相同的配置, 测试demo, demo仍然会OOM. 使用其他 ROS_DOMAIN_ID, 仍然会OOM

仍然使用strace来跟踪内存分配: 

```
strace -o trace.log -f ros2 launch isaac_ros_apriltag isaac_ros_apriltag_isaac_sim_pipeline.launch.py
``` 

让模型分析mmap+clone构成的线程关系: 

```
Process (Root PID: 58634)
│
├── Thread 58634 (Main Thread)
│   │
│   └── Creates Thread 58646
│       ├── mmap(STACK): 8 MB (为 58646 分配栈空间)
│       └── clone3() => 58646
│
├── Thread 58646
│   └── mmap(NORESERVE): 64 MB (启动后预留虚拟内存)
│
├── Thread 58645 [ THE INCUBATOR - 孵化器]
│   │   (此线程陷入失控循环，疯狂创建下面的所有子线程)
│   │
│   ├── Creates Thread 58640 (父子关系推断)
│   │   └── mmap(NORESERVE): 128 MB
│   │
│   ├── Creates Thread 58642 [ THE DETONATOR - 灾难制造者]
│   │   │   (父子关系根据行为模式高度推断，clone3日志缺失)
│   │   ├── mmap(FATAL): 40.4 GB (43,368,583,168 bytes)
│   │   └── mmap(FATAL): 43.26 GB (46,455,255,040 bytes)
│   │
│   ├── Creates Thread 58647
│   │   ├── mmap(STACK): 8 MB (由 58645 分配)
│   │   ├── clone3() => 58647
│   │   └── mmap(NORESERVE): 128 MB
│   │
│   ├── Creates Thread 58648
│   │   ├── mmap(STACK): 8 MB (由 58645 分配)
│   │   ├── clone3() => 58648
│   │   └── mmap(NORESERVE): 128 MB
│   │
│   ├── Creates Thread 58649
│   │   ├── mmap(STACK): 8 MB (由 58645 分配)
│   │   └── clone3() => 58649
│   │
│   ├── Creates Thread 58650
│   │   ├── mmap(STACK): 8 MB (由 58645 分配)
│   │   ├── clone3() => 58650
│   │   └── mmap(NORESERVE): 64 MB
│   │
│   ├── Creates Thread 58651
│   │   ├── mmap(STACK): 8 MB (由 58645 分配)
│   │   ├── clone3() => 58651
│   │   └── mmap(NORESERVE): 128 MB
│   │
│   ├── Creates Thread 58652
│   │   ├── mmap(STACK): 8 MB (由 58645 分配)
│   │   ├── clone3() => 58652
│   │   └── mmap(NORESERVE): 64 MB + 128 MB
│   │
│   ├── Creates Thread 58654 [THE DETONATOR - 灾难制造者]
│   │   ├── mmap(STACK): 8 MB (由 58645 分配)
│   │   ├── clone3() => 58654
│   │   ├── mmap(NORESERVE): 128 MB
│   │   ├── mmap(FATAL): 40.4 GB (43,368,583,168 bytes)
│   │   └── mmap(FATAL): 43.26 GB (46,455,255,040 bytes)
│   │
│   └── Creates Threads 58655 to 58781+ (日志中可见的大量线程)
│       ├── mmap(STACK): 8 MB (由 58645 为每个线程分配)
│       ├── clone3() => 58655, 58656, 58657, ...
│       └── (部分线程启动后也进行了 mmap(NORESERVE) 操作)
│
└── ... (其他正常线程)
``` 

重新编译ros2, 增加调试符号: 

```
容器内: 
 
cd ${ISAAC_ROS_WS}
colcon build --symlink-install --packages-select-regex robotiq* serial --cmake-args " -DBUILD_TESTING=OFF -DCMAKE_BUILD_TYPE=Debug -DCMAKE_CXX_FLAGS='-g -fno-omit-frame-pointer'"
``` 

在容器内进行bpf: 

```
容器内
mount -t debugfs none /sys/kernel/debug
sudo bpftrace -p $(pgrep -f ros2) -e 'tracepoint:syscalls:sys_enter_mmap /args->len > 100000000/ { printf("========== LARGE MMAP DETECTED ==========\n"); printf("Comm: %s, PID: %d\n", comm, pid); printf("Requested Size: %llu bytes (~%llu GB)\n", args->len, args->len / 1024 / 1024 / 1024); ustack; printf("=========================================\n\n"); }'

``` 

但无法获取ustack

换用py_spy: 

```
容器内: 
pip install py-spy
 
获取火焰图
py-spy record -o profile.svg --native -- ros2 launch isaac_ros_apriltag isaac_ros_apriltag_isaac_sim_pipeline.launch.py
 
火焰图中只获取了一些import lib的操作
 
需要跟踪ros2的子进程
 
获取子进程列表: 
pstree -p $(pgrep -f ros2) -T
 
查询子进程启动命令: 
ps -p $(pgrep -f component_container) -o args
 
启动命令为: 
/opt/ros/humble/lib/rclcpp_components/component_container_mt --ros-args -r __node:=apriltag_container -r __ns:=/
 
component_container_mt是一个c程序
 
使用gdb来调试这个程序 (在root下运行): 

gdb --args /opt/ros/humble/lib/rclcpp_components/component_container_mt --ros-args -r __node:=apriltag_container -r __ns:=/
 
gdb中: 
start, 然后ctrl+c进入调试, 输入如下命令 增加断点: 
break mmap
condition 2 $rsi > 1000000000
commands 2
  echo "\n\n!!! Large mmap call detected !!!\n"
  printf "Requested Size: %llu bytes (~%llu MB)\n", $rsi, $rsi / 1024 / 1024
  echo "--- C++ Call Stack ---\n"
  bt
  echo "--- End of Stack Trace ---\n\n"
  continue
end
 
继续运行后, 断点触发: 
 
(gdb) bt
#0  __GI___mmap64 (addr=addr@entry=0x0, len=len@entry=66743717888, prot=prot@entry=3, flags=flags@entry=34, fd=fd@entry=-1,
    offset=offset@entry=0) at ../sysdeps/unix/sysv/linux/mmap64.c:47
#1  0x00007ffff79804b8 in sysmalloc_mmap (nb=nb@entry=66743717216, pagesize=pagesize@entry=4096, extra_flags=extra_flags@entry=0,
    av=0x7fffe4000030) at ./malloc/malloc.c:2441
#2  0x00007ffff798128c in sysmalloc (nb=nb@entry=66743717216, av=av@entry=0x7fffe4000030) at ./malloc/malloc.c:2594
#3  0x00007ffff79828dd in _int_malloc (av=av@entry=0x7fffe4000030, bytes=bytes@entry=66743717208) at ./malloc/malloc.c:4407
#4  0x00007ffff7983139 in __GI___libc_malloc (bytes=66743717208) at ./malloc/malloc.c:3329
#5  0x00007ffff7bd798c in operator new(unsigned long) () from /lib/x86_64-linux-gnu/libstdc++.so.6
#6  0x00007ffff6d13915 in ?? () from /opt/ros/humble/lib/librmw_dds_common__rosidl_typesupport_fastrtps_cpp.so
#7  0x00007ffff6d13eb7 in rmw_dds_common::msg::typesupport_fastrtps_cpp::cdr_deserialize(eprosima::fastcdr::Cdr&, rmw_dds_common::msg::NodeEntitiesInfo_<std::allocator<void> >&) () from /opt/ros/humble/lib/librmw_dds_common__rosidl_typesupport_fastrtps_cpp.so
#8  0x00007ffff6d141f7 in rmw_dds_common::msg::typesupport_fastrtps_cpp::cdr_deserialize(eprosima::fastcdr::Cdr&, rmw_dds_common::msg::ParticipantEntitiesInfo_<std::allocator<void> >&) ()
   from /opt/ros/humble/lib/librmw_dds_common__rosidl_typesupport_fastrtps_cpp.so
#9  0x00007ffff75fdbb9 in ?? () from /opt/ros/humble/lib/librmw_fastrtps_cpp.so
#10 0x00007ffff75ae156 in rmw_fastrtps_shared_cpp::TypeSupport::deserialize(eprosima::fastrtps::rtps::SerializedPayload_t*, void*)
    () from /opt/ros/humble/lib/librmw_fastrtps_shared_cpp.so
#11 0x00007ffff73b2c4a in ?? () from /opt/ros/humble/lib/libfastrtps.so.2.6
#12 0x00007ffff7039392 in eprosima::fastdds::dds::DataReaderImpl::read_or_take(eprosima::fastdds::dds::LoanableCollection&, eprosima::fastdds::dds::LoanableSequence<eprosima::fastdds::dds::SampleInfo, std::integral_constant<bool, true> >&, int, eprosima::fastrtps::rtps::InstanceHandle_t const&, unsigned short, unsigned short, unsigned short, bool, bool, bool) ()
   from /opt/ros/humble/lib/libfastrtps.so.2.6
#13 0x00007ffff703953a in eprosima::fastdds::dds::DataReaderImpl::take(eprosima::fastdds::dds::LoanableCollection&, eprosima::fastdds::dds::LoanableSequence<eprosima::fastdds::dds::SampleInfo, std::integral_constant<bool, true> >&, int, unsigned short, unsigned short, unsigned short) () from /opt/ros/humble/lib/libfastrtps.so.2.6
#14 0x00007ffff75a57c6 in rmw_fastrtps_shared_cpp::_take(char const*, rmw_subscription_s const*, void*, bool*, rmw_message_info_s*, rmw_subscription_allocation_s*) () from /opt/ros/humble/lib/librmw_fastrtps_shared_cpp.so
#15 0x00007ffff759457f in ?? () from /opt/ros/humble/lib/librmw_fastrtps_shared_cpp.so
#16 0x00007ffff7c05253 in ?? () from /lib/x86_64-linux-gnu/libstdc++.so.6
--Type <RET> for more, q to quit, c to continue without paging--
#17 0x00007ffff7972ac3 in start_thread (arg=<optimized out>) at ./nptl/pthread_create.c:442
#18 0x00007ffff7a048c0 in clone3 () at ../sysdeps/unix/sysv/linux/x86_64/clone3.S:81
``` 

切换DDS的实现: export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp

会报错: 

```
root@dell-136:/workspaces/isaac_ros-dev# /opt/ros/humble/lib/rclcpp_components/component_container_mt --ros-args -r __node:=apriltag_container -r __ns:=/

>>> [rcutils|error_handling.c:108] rcutils_set_error_state()
This error state is being overwritten:

  'invalid data size, at ./src/serdata.cpp:384'

with this new error message:

  'string data is not null-terminated, at ./src/serdata.cpp:384'

rcutils_reset_error() should be called after error handling to avoid this.
<<<

>>> [rcutils|error_handling.c:108] rcutils_set_error_state()
This error state is being overwritten:

  'string data is not null-terminated, at ./src/serdata.cpp:384'

with this new error message:

  'invalid data size, at ./src/serdata.cpp:384'

rcutils_reset_error() should be called after error handling to avoid this.
<<<
``` 

需要判断是DDS的问题, 还是真的数据包的大小有问题: 

  - 将Isaac Sim和Demo都换成 rmw_cyclonedds_cpp, 发现仍然报错
  - 换成rmw_cyclonedds_cpp后, 跑之前的/apriltag的验证程序, 能正常运行

结论是rmw_cyclonedds_cpp正常, 数据包真的有问题

清理Isaac Sim的缓存 (./clear_caches.sh 脚本)后, 一切恢复正常

### 结论: 

  - 猜测, 之前是因为 FASTRTPS_DEFAULT_PROFILES_FILE=/workspaces/isaac_ros-dev/a.xml 中的IP配置问题, 产生了一些错误的消息, 导致OOM
  - 这些消息有一些进入了Isaac Sim的缓存
  - 之后再启动服务, 会导致ROS2消费错误的消息, 内存OOM
  - 清理缓存后恢复

还有一个问题: ros2 node list会报错: 

```
admin@dell-136:/workspaces/isaac_ros-dev$ ros2 node list
Traceback (most recent call last):
  File "/opt/ros/humble/bin/ros2", line 33, in <module>
    sys.exit(load_entry_point('ros2cli==0.18.12', 'console_scripts', 'ros2')())
  File "/opt/ros/humble/lib/python3.10/site-packages/ros2cli/cli.py", line 91, in main
    rc = extension.main(parser=parser, args=args)
  File "/opt/ros/humble/lib/python3.10/site-packages/ros2node/command/node.py", line 37, in main
    return extension.main(args=args)
  File "/opt/ros/humble/lib/python3.10/site-packages/ros2node/verb/list.py", line 38, in main
    node_names = get_node_names(node=node, include_hidden_nodes=args.all)
  File "/opt/ros/humble/lib/python3.10/site-packages/ros2node/api/__init__.py", line 60, in get_node_names
    node_names_and_namespaces = node.get_node_names_and_namespaces()
  File "/usr/lib/python3.10/xmlrpc/client.py", line 1122, in __call__
    return self.__send(self.__name, args)
  File "/usr/lib/python3.10/xmlrpc/client.py", line 1464, in __request
    response = self.__transport.request(
  File "/usr/lib/python3.10/xmlrpc/client.py", line 1166, in request
    return self.single_request(host, handler, request_body, verbose)
  File "/usr/lib/python3.10/xmlrpc/client.py", line 1182, in single_request
    return self.parse_response(resp)
  File "/usr/lib/python3.10/xmlrpc/client.py", line 1354, in parse_response
    return u.close()
  File "/usr/lib/python3.10/xmlrpc/client.py", line 668, in close
    raise Fault(**self._stack[0])
xmlrpc.client.Fault: <Fault 1: "<class 'rclpy._rclpy_pybind11.RCLError'>:Failed to get node names: empty node name returned by the RMW layer, at ./src/rcl/graph.c:360">
``` 

诊断:

  - ros2 daemon stop, 关掉daemon (daemon是ROS2的节点发现的缓存)
  - 执行ros2 node list, 可以获取空
  - 开启ros2 demo任务, 执行ros2 node list, 可以正确获取节点: 

```
admin@dell-136:/workspaces/isaac_ros-dev$ ros2 node list
/apriltag
/apriltag_container
/launch_ros_23651
```

  - 开启Isaac Sim play, 执行ros2 node list, 报错: 

```
admin@dell-136:/workspaces/isaac_ros-dev$ ros2 node list
Traceback (most recent call last):
  File "/opt/ros/humble/bin/ros2", line 33, in <module>
    sys.exit(load_entry_point('ros2cli==0.18.12', 'console_scripts', 'ros2')())
  File "/opt/ros/humble/lib/python3.10/site-packages/ros2cli/cli.py", line 91, in main
    rc = extension.main(parser=parser, args=args)
  File "/opt/ros/humble/lib/python3.10/site-packages/ros2node/command/node.py", line 37, in main
    return extension.main(args=args)
  File "/opt/ros/humble/lib/python3.10/site-packages/ros2node/verb/list.py", line 38, in main
    node_names = get_node_names(node=node, include_hidden_nodes=args.all)
  File "/opt/ros/humble/lib/python3.10/site-packages/ros2node/api/__init__.py", line 60, in get_node_names
    node_names_and_namespaces = node.get_node_names_and_namespaces()
  File "/usr/lib/python3.10/xmlrpc/client.py", line 1122, in __call__
    return self.__send(self.__name, args)
  File "/usr/lib/python3.10/xmlrpc/client.py", line 1464, in __request
    response = self.__transport.request(
  File "/usr/lib/python3.10/xmlrpc/client.py", line 1166, in request
    return self.single_request(host, handler, request_body, verbose)
  File "/usr/lib/python3.10/xmlrpc/client.py", line 1182, in single_request
    return self.parse_response(resp)
  File "/usr/lib/python3.10/xmlrpc/client.py", line 1354, in parse_response
    return u.close()
  File "/usr/lib/python3.10/xmlrpc/client.py", line 668, in close
    raise Fault(**self._stack[0])
xmlrpc.client.Fault: <Fault 1: "<class 'rclpy._rclpy_pybind11.RCLError'>:Failed to get node names: empty node name returned by the RMW layer, at ./src/rcl/graph.c:360">
```

  - 使用 --no-daemon: 

```
admin@dell-136:/workspaces/isaac_ros-dev$ ros2 node list --no-daemon

>>> [rcutils|error_handling.c:108] rcutils_set_error_state()
This error state is being overwritten:

  'invalid data size, at ./src/serdata.cpp:384'

with this new error message:

  'string data is not null-terminated, at ./src/serdata.cpp:384'

rcutils_reset_error() should be called after error handling to avoid this.
<<<
Failed to get node names: empty node name returned by the RMW layer, at ./src/rcl/graph.c:360
admin@dell-136:/workspaces/isaac_ros-dev$
```

    - Issac-sim端报错: 

```
[WARN] [1760067411.187940102] [rmw_cyclonedds_cpp]: Failed to parse type hash for topic 'ros_discovery_info' with type 'rmw_dds_common::msg::dds_::ParticipantEntitiesInfo_' from USER_DATA '(null)'.
[WARN] [1760067411.188174809] [rmw_cyclonedds_cpp]: Failed to parse type hash for topic 'ros_discovery_info' with type 'rmw_dds_common::msg::dds_::ParticipantEntitiesInfo_' from USER_DATA '(null)'.

```

  - 关闭Isaac Sim play, 执行ros2 node list, 报错: 

```
admin@dell-136:/workspaces/isaac_ros-dev$ ros2 node list
Traceback (most recent call last):
  File "/opt/ros/humble/bin/ros2", line 33, in <module>
    sys.exit(load_entry_point('ros2cli==0.18.12', 'console_scripts', 'ros2')())
  File "/opt/ros/humble/lib/python3.10/site-packages/ros2cli/cli.py", line 91, in main
    rc = extension.main(parser=parser, args=args)
  File "/opt/ros/humble/lib/python3.10/site-packages/ros2node/command/node.py", line 37, in main
    return extension.main(args=args)
  File "/opt/ros/humble/lib/python3.10/site-packages/ros2node/verb/list.py", line 38, in main
    node_names = get_node_names(node=node, include_hidden_nodes=args.all)
  File "/opt/ros/humble/lib/python3.10/site-packages/ros2node/api/__init__.py", line 60, in get_node_names
    node_names_and_namespaces = node.get_node_names_and_namespaces()
  File "/usr/lib/python3.10/xmlrpc/client.py", line 1122, in __call__
    return self.__send(self.__name, args)
  File "/usr/lib/python3.10/xmlrpc/client.py", line 1464, in __request
    response = self.__transport.request(
  File "/usr/lib/python3.10/xmlrpc/client.py", line 1166, in request
    return self.single_request(host, handler, request_body, verbose)
  File "/usr/lib/python3.10/xmlrpc/client.py", line 1182, in single_request
    return self.parse_response(resp)
  File "/usr/lib/python3.10/xmlrpc/client.py", line 1354, in parse_response
    return u.close()
  File "/usr/lib/python3.10/xmlrpc/client.py", line 668, in close
    raise Fault(**self._stack[0])
xmlrpc.client.Fault: <Fault 1: "<class 'rclpy._rclpy_pybind11.RCLError'>:Failed to get node names: empty node name returned by the RMW layer, at ./src/rcl/graph.c:360">
```

  - 使用--no-daemon, 可正确返回结果:

```
admin@dell-136:/workspaces/isaac_ros-dev$ ros2 node list --no-daemon
/apriltag
/apriltag_container
/launch_ros_23651
```

  - 使用ros2 daemon stop关闭daemon, 执行ros2 node list, 可正确返回结果
  - 结论: 
    - Isaac Sim的node名有问题, 需要诊断原因
    - 报错会被daemon记录下来, 一直干扰后面的结果
    - 可以通过ros2 daemon stop来重置daemon
    - 参考: <https://github.com/eclipse-zenoh/zenoh-plugin-ros2dds/issues/290>

  - Isaac Sim指定NodeName, 仍然会报错: 

```
./isaac-sim.sh --/exts/omni.isaac.ros_bridge/nodeName="NewNodeName"
```

  - 将两边还原成rmw_fastrtps_cpp, 内存会OOM  

    - 也就是说核心问题在于 ros2 node list 的报错, 跟Isaac Sim内的DDS有关, 一旦产生错误的包, rmw_fastrtps_cpp会OOM, rmw_cyclonedds_cpp会报错但能继续进行
  - 参考: <https://github.com/eclipse-zenoh/zenoh-plugin-ros2dds/issues/290>
    - 考虑是Isaac Sim内部的ROS2版本过低
      - 检查Isaac Sim内部的ROS2版本, 在windows/script editor中输入脚本检查: 
        - ![image2025-10-10 13:40:12.png](/assets/01KJC02BRZ0WYFF71HAK5SAJGA/image2025-10-10%2013%3A40%3A12.png)
      - 版本为jazzy, 但dev环境中的ros2版本为humble, 需要升级, 但dev容器只有humble版本. 所以考虑将isaac sim使用的ROS2指定为humble, 使用环境变量: 

```
export ROS_DISTRO=humble
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/data/huangyan/isaac-sim/exts/isaacsim.ros2.bridge/humble/lib
```

    - 报错消失

至此, 解决了所有报错, 跑通了isaac_ros_apriltag的样例. 

TODO: 需要整理环境的安装步骤, 以及运行步骤
