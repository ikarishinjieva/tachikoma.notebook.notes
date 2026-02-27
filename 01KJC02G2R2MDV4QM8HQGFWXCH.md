---
title: 20251010 - 分析isaac_ros_apriltag的原理 (AprilTag 识别demo)
confluence_page_id: 4358817
created_at: 2025-10-10T08:49:53+00:00
updated_at: 2025-10-10T11:44:28+00:00
---

官方文档: <https://nvidia-isaac-ros.github.io/concepts/fiducials/apriltag/tutorial_isaac_sim.html>

安装文档在 [20251008 - 列出ROS2样例表, 尝试ROS2样例isaac_ros_apriltag, 解决OOM报错] 中

在 [20251008 - 列出ROS2样例表, 尝试ROS2样例isaac_ros_apriltag, 解决OOM报错] 中进行了一些分析, 复制过来: 

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

通过让机器人旋转, 在rviz中可以看到动态效果: 

```
# 旋转
ros2 topic pub --rate 10 /cmd_vel geometry_msgs/msg/Twist "{linear: {x: 0.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 0.5}}"
 
# 前进
ros2 topic pub --rate 10 /cmd_vel geometry_msgs/msg/Twist "{linear: {x: 0.2, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 0.0}}"
 
# 停止
ros2 topic pub --once /cmd_vel geometry_msgs/msg/Twist "{linear: {x: 0.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 0.0}}"
``` 

![image2025-10-10 17:33:37.png](/assets/01KJC02G2R2MDV4QM8HQGFWXCH/image2025-10-10%2017%3A33%3A37.png)

关于视觉的topic: 

| Topic 名称 | 用途 | 消息类型 | 关键特性与目标用户 |
| --- | --- | --- | --- |
| `/front_stereo_camera/left/image_rect_color` | 左相机的校正后彩色图像 | `sensor_msgs/Image` | **像素数据** 。用于所有标准的 ROS 视觉节点。 |
| `/front_stereo_camera/left/camera_info` | 左相机的参数信息（内参、畸变等） | `sensor_msgs/CameraInfo` | **元数据** 。与图像配对使用，用于所有需要进行 3D 几何计算的节点。 |
| `/front_stereo_camera/right/image_rect_color` | 右相机的校正后彩色图像 | `sensor_msgs/Image` | **像素数据** 。与左图像一起用于立体视觉算法。 |
| `/front_stereo_camera/right/camera_info` | 右相机的参数信息 | `sensor_msgs/CameraInfo` | **元数据** 。与右图像配对使用，用于立体视觉算法。 |
| `/front_stereo_camera/left/image_rect_color/nitros` | `image_rect_color` 的 **高性能加速版本** | `sensor_msgs/Image` (Adapted) |  |
  
查看图片流的格式: 

```
admin@dell-136:/workspaces/isaac_ros-dev$ ros2 topic echo --once /front_stereo_camera/left/image_rect_color
header:
  stamp:
    sec: 450
    nanosec: 533356830
  frame_id: front_stereo_camera_left_optical
height: 1200
width: 1920
encoding: rgb8
is_bigendian: 0
step: 5760
data:
- 48
- 41
- 37
- 51
- 44
- 40
- 52
- 45
- 41
 
...
``` 

查看图片的发布频率: 

```
admin@dell-136:/workspaces/isaac_ros-dev$ ros2 topic hz /front_stereo_camera/left/image_rect_color
average rate: 7.609
	min: 0.115s max: 0.153s std dev: 0.01195s window: 9
average rate: 7.673
	min: 0.104s max: 0.154s std dev: 0.01432s window: 17
average rate: 7.602
	min: 0.103s max: 0.167s std dev: 0.01666s window: 25
average rate: 7.199
	min: 0.100s max: 0.401s std dev: 0.05050s window: 31
average rate: 7.248
	min: 0.100s max: 0.401s std dev: 0.04575s window: 39
```
