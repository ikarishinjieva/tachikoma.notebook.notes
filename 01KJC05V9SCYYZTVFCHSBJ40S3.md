---
title: 20251017 - 为机械臂开发环境, 整备4090 48G机器, 并完成Pick and Place任务
confluence_page_id: 4358930
created_at: 2025-10-16T16:41:04+00:00
updated_at: 2025-10-19T16:24:16+00:00
---

# 参考

[20251010 - 整理Isaac Sim和ROS2 demo的环境安装和运行指令]

经验: 更换4090后, 模拟的FPS大幅上升

# 机器

```
ssh ubuntu@117.50.172.161
SMSn6ZK7QFRgAnt2
``` 

# 下载

```
# Isaac Sim 4.2
wget https://download.isaacsim.omniverse.nvidia.com/isaac-sim-standalone%404.2.0-rc.18%2Brelease.16044.3b2ed111.gl.linux-x86_64.release.zip
 
# NICE DCV
wget https://d1uj6qtbmh3dt5.cloudfront.net/2024.0/Servers/nice-dcv-2024.0-19030-ubuntu2204-x86_64.tgz

# ROS2 DEV 容器
在isaac_ros_common项目脚本中自动下载
``` 

# 代理

注意需要设置代理, 由于是ucloud的服务器, 需要增加例外: 

```
export http_proxy=http://127.0.0.1:7890
export https_proxy=http://127.0.0.1:7890
export no_proxy=127.0.0.1,localhost,ucloud.cn
``` 

# 工作区路径

```
mkdir -p  /data/huangyan/isaac_ros-dev/src
echo "export ISAAC_ROS_WS=/data/huangyan/isaac_ros-dev/" >> ~/.bashrc
``` 

# 配置NICE DCV

参考: [20251010 - 整理Isaac Sim和ROS2 demo的环境安装和运行指令]

环境是ubuntu 22.04, 需要稍微修改命令

  - 下载地址
  - proxy地址

# 配置ROS2

参考: [20251010 - 整理Isaac Sim和ROS2 demo的环境安装和运行指令]

配置isaac_ros_common项目时, 需要修改 ./src/isaac_ros_common/scripts/build_image_layers.sh , 修改image的根域名: 

```
BASE_DOCKER_REGISTRY_NAMES=("ngc.nju.edu.cn/nvidia/isaac/ros")
``` 

并且需要执行配置: [20251010 - 整理Isaac Sim和ROS2 demo的环境安装和运行指令] "进入容器配置ROS"

# 跟随案例

[20251015 - 分析ros-humble-isaac-manipulator-bringup的原理 (机械臂复杂流)]

```
sudo apt-get install -y ros-humble-isaac-manipulator-bringup ros-humble-isaac-manipulator-pick-and-place
``` 

按照<https://nvidia-isaac-ros.github.io/reference_workflows/isaac_manipulator/tutorials/tutorial_e2e.html#set-up-perception-deep-learning-models> , 下载相关模型

# 启动Isaac Sim

因为使用了Isaac Sim 4.2, LD的路径需要变更, 并指定GPU: 

```
export http_proxy=http://127.0.0.1:7890
export https_proxy=http://127.0.0.1:7890
export no_proxy=127.0.0.1,localhost,ucloud.cn
export ROS_DOMAIN_ID=99
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
export ROS_DISTRO=humble
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/data/huangyan/isaac-sim/exts/omni.isaac.ros2_bridge/humble/lib/
export CUDA_VISIBLE_DEVICES=7
export __GLX_VENDOR_LIBRARY_NAME=nvidia
export DISPLAY=:0
 
source /workspaces/isaac_ros-dev/install/setup.bash
``` 

# 匹配QoS

修改: /opt/ros/humble/share/isaac_manipulator_pick_and_place/launch/isaac_sim_workflows.launch.py , 设置QoS

```
            #'qos_setting': 'SENSOR_DATA',
            'qos_setting': 'DEFAULT',
``` 

执行: 

```
ros2 launch isaac_manipulator_pick_and_place isaac_sim_workflows.launch.py    workflow_type:=pose_to_pose
``` 

有很大概率出现 "ESDF data is empty"

先运行程序, 再执行isaac sim的play, 一定概率能正常运行, 认为是QoS设置仍然有一定问题

修改 /opt/ros/humble/share/isaac_manipulator_bringup/config/nvblox/nvblox_manipulator_base.yaml:

```
    input_qos: "DEFAULT"
``` 

不再出现ESDF的报错, 出现了新的报错: 

```
[cumotion_goal_set_planner_node-4] [INFO] [1760843672.166250752] [cumotion_planner]: warming up cuMotion, wait until ready
[goal_initializer_node-15] [WARN] [1760843673.336770605] [goal_init_node]: Waiting for object pose transform to be available in TF,                                       between base_link and goal_frame. if                                       warning persists, check if object pose is detected and                                       published to tf. Message from TF: Could not find a connection between 'base_link' and 'goal_frame' because they are not part of the same tree.Tf has two or more unconnected trees.
[goal_initializer_node-15] [WARN] [1760843675.253438371] [goal_init_node]: Waiting for object pose transform to be available in TF,                                       between base_link and goal_frame. if                                       warning persists, check if object pose is detected and                                       published to tf. Message from TF: Could not find a connection between 'base_link' and 'goal_frame' because they are not part of the same tree.Tf has two or more unconnected trees.
[robot_segmenter_node-5] [INFO] [1760843675.850973163] [robot_segmenter]: Reading TF from cameras
[goal_initializer_node-15] [WARN] [1760843677.171435332] [goal_init_node]: Waiting for object pose transform to be available in TF,                                       between base_link and goal_frame. if                                       warning persists, check if object pose is detected and                                       published to tf. Message from TF: Could not find a connection between 'base_link' and 'goal_frame' because they are not part of the same tree.Tf has two or more unconnected trees.
[goal_initializer_node-15] [WARN] [1760843679.040556464] [goal_init_node]: Waiting for object pose transform to be available in TF,                                       between base_link and goal_frame. if                                       warning persists, check if object pose is detected and                                       published to tf. Message from TF: Could not find a connection between 'base_link' and 'goal_frame' because they are not part of the same tree.Tf has two or more unconnected trees.
``` 

分析时, 发现一个额外的现象, ros2 topic hz因为qos的原因, 会卡住, 但 ros2 topic echo 会有值, 描述如下: 

```
我确认 : 

admin@10-60-141-108:/opt$ ros2 topic hz /front_stereo_camera/left/camera_info
average rate: 24.285
	min: 0.038s max: 0.052s std dev: 0.00348s window: 26
average rate: 24.273
	min: 0.038s max: 0.052s std dev: 0.00312s window: 51

但

admin@10-60-141-108:/opt$ ros2 topic hz /front_stereo_camera/left/image_raw

会卡住

但

admin@10-60-141-108:/opt$ ros2 topic echo /front_stereo_camera/left/image_raw --field header.stamp
sec: 113
nanosec: 450005916
---
sec: 113
nanosec: 466672584
---
sec: 113
nanosec: 483339251
---
sec: 113
nanosec: 500005919
---
sec: 113
nanosec: 516672587
---
sec: 113
nanosec: 533339254

会有输出. 

``` 

检查步骤: 

```
admin@10-60-141-108:/opt$ ./check.sh

======================================
TF断开问题 - 完整系统诊断
======================================
开始时间: Sun Oct 19 03:06:39 PM CST 2025

======================================
阶段1: RT-DETR 数据流
======================================

【1.1 - 相机输出】 等待10秒...
✓ 相机正常 (238 帧)

【1.2 - Padded Image】 等待10秒...
✓ padded_image 正常 (12 帧)

【1.3 - Resize Image】 等待10秒...
✓ resize/image 正常 (10 帧)

【1.4 - RT-DETR Input】 等待10秒...
✓ rtdetr/image_dropped 正常 (10 帧)
✓ 阶段1完成

======================================
阶段2: RT-DETR 推理
======================================

【2.1 - RT-DETR节点】
/rtdetr_decoder
/rtdetr_drop_node
/rtdetr_image_to_tensor_node
/rtdetr_interleaved_to_planar_node
/rtdetr_pad_node
/rtdetr_preprocessor
/rtdetr_reshape_node
/rtdetr_resize_node
/rtdetr_tensor_rt
✓ 找到 9 个节点

【2.2 - RT-DETR检测输出】 等待15秒...
检测结果：
header:
  stamp:
    sec: 20
    nanosec: 900001090
  frame_id: front_stereo_camera_left
results:
- hypothesis:
    class_id: '3'
    score: 0.7639259099960327
  pose:
    pose:
      position:
        x: 0.0
        y: 0.0
        z: 0.0
      orientation:
        x: 0.0
        y: 0.0
        z: 0.0
        w: 1.0
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

检测到 1 个物体
✓ RT-DETR检测正常 (1 个)
类别ID:
    class_id: '3'

【2.3 - 检测过滤】 等待15秒...
过滤后结果：
header:
  stamp:
    sec: 22
    nanosec: 16667814
  frame_id: front_stereo_camera_left
detections:
- header:
    stamp:
      sec: 22
      nanosec: 16667814
    frame_id: front_stereo_camera_left
  results:
  - hypothesis:
      class_id: '3'
      score: 0.7766614556312561
    pose:
      pose:
        position:
          x: 0.0
          y: 0.0
          z: 0.0
        orientation:
          x: 0.0
          y: 0.0
          z: 0.0
          w: 1.0
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

✓ 过滤后有 1 个物体
✓ 阶段2完成

======================================
阶段3: Foundation Pose
======================================

【3.1 - Foundation Pose输入】
✓ 有输入 (1 个)

【3.2 - Foundation Pose输出】 等待15秒...
输出结果:
header:
  stamp:
    sec: 25
    nanosec: 950001353
  frame_id: front_stereo_camera_left
detections:
- header:
    stamp:
      sec: 25
      nanosec: 950001353
    frame_id: front_stereo_camera_left
  results:
  - hypothesis:
      class_id: ''
      score: 0.0
    pose:
      pose:
        position:
          x: -0.01788538694381714
          y: -0.091917484998703
          z: 1.2039997577667236
        orientation:
          x: 0.7426320364795997
          y: 0.20157311556170282
          z: -0.589788500108161
          w: -0.24498053517999488
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

✓ 有位姿输出 (1 个)

【3.3 - TF发布】 等待10秒...
✓ detected_object1 TF正在发布

  child_frame_id: detected_object1
  transform:
    translation:
      x: -0.017855651676654816
      y: -0.09199859201908112
      z: 1.2039573192596436
    rotation:
      x: 0.7420837007388023
      y: 0.20185675899128472
      z: -0.5908098347421145
      w: -0.24394542241930592
---
transforms:
✓ 阶段3完成

======================================
阶段4: TF树连接
======================================

【4.1 - static_transform_publisher】
/static_transform_publisher_fcyY8j4IxYNfURTP
✓ 找到 1 个节点

【4.2 - goal_frame发布】 等待10秒...
✓ goal_frame正在发布

【4.3 - TF链路测试】
测试1: base_link → detected_object1
✗ 不连通
[INFO] [1760857672.559099479] [tf2_echo]: Waiting for transform base_link ->  detected_object1: Invalid frame ID "base_link" passed to canTransform argument target_frame - frame does not exist
At time 30.233334910
- Translation: [-0.567, 0.302, 0.051]
- Rotation: in Quaternion (xyzw) [0.541, -0.450, -0.452, 0.549]
- Rotation: in RPY (radian) [1.561, -0.006, -1.383]

测试2: detected_object1 → goal_frame
✗ 不连通
[INFO] [1760857677.563107350] [tf2_echo]: Waiting for transform detected_object1 ->  goal_frame: Invalid frame ID "detected_object1" passed to canTransform argument target_frame - frame does not exist
At time 0.0
- Translation: [0.043, 0.359, 0.065]
- Rotation: in Quaternion (xyzw) [0.553, 0.475, -0.454, 0.513]
- Rotation: in RPY (radian) [1.999, 1.421, 0.409]

测试3: base_link → goal_frame
✗ 不连通
[INFO] [1760857682.567642073] [tf2_echo]: Waiting for transform base_link ->  goal_frame: Invalid frame ID "base_link" passed to canTransform argument target_frame - frame does not exist
[INFO] [1760857684.556472812] [tf2_echo]: Waiting for transform base_link ->  goal_frame: Could not find a connection between 'base_link' and 'goal_frame' because they are not part of the same tree.Tf has two or more unconnected trees.
[INFO] [1760857686.556486377] [tf2_echo]: Waiting for transform base_link ->  goal_frame: Could not find a connection between 'base_link' and 'goal_frame' because they are not part of the same tree.Tf has two or more unconnected trees.
[INFO] [1760857687.377222515] [rclcpp]: signal_handler(signum=15)

【4.4 - TF树可视化】

======================================
诊断总结
======================================
完成时间: Sun Oct 19 03:08:13 PM CST 2025

诊断未完全通过
TF链路未完全连通
 
 
###### foundationpose_node并不发布TF
 
admin@10-60-141-108:/opt$ ros2 node info /foundationpose_node
/foundationpose_node
  Subscribers:
    /camera_info_segmentation: sensor_msgs/msg/CameraInfo
    /camera_info_segmentation/nitros: negotiated_interfaces/msg/NegotiatedTopicsInfo
    /clock: rosgraph_msgs/msg/Clock
    /front_stereo_camera/depth/ground_truth: sensor_msgs/msg/Image
    /front_stereo_camera/depth/ground_truth/nitros: negotiated_interfaces/msg/NegotiatedTopicsInfo
    /front_stereo_camera/left/image_raw: sensor_msgs/msg/Image
    /front_stereo_camera/left/image_raw/nitros: negotiated_interfaces/msg/NegotiatedTopicsInfo
    /parameter_events: rcl_interfaces/msg/ParameterEvent
    /segmentation: sensor_msgs/msg/Image
    /segmentation/nitros: negotiated_interfaces/msg/NegotiatedTopicsInfo
  Publishers:
    /parameter_events: rcl_interfaces/msg/ParameterEvent
    /pose_estimation/output: vision_msgs/msg/Detection3DArray
    /pose_estimation/output/nitros: negotiated_interfaces/msg/NegotiatedTopicsInfo
    /pose_estimation/pose_matrix_output: isaac_ros_tensor_list_interfaces/msg/TensorList
    /pose_estimation/pose_matrix_output/nitros: negotiated_interfaces/msg/NegotiatedTopicsInfo
    /rosout: rcl_interfaces/msg/Log
  Service Servers:
    /foundationpose_node/describe_parameters: rcl_interfaces/srv/DescribeParameters
    /foundationpose_node/get_parameter_types: rcl_interfaces/srv/GetParameterTypes
    /foundationpose_node/get_parameters: rcl_interfaces/srv/GetParameters
    /foundationpose_node/list_parameters: rcl_interfaces/srv/ListParameters
    /foundationpose_node/set_parameters: rcl_interfaces/srv/SetParameters
    /foundationpose_node/set_parameters_atomically: rcl_interfaces/srv/SetParametersAtomically
  Service Clients:

  Action Servers:

  Action Clients:
 
 
###### TF中缺失有detected_object1
 
admin@10-60-141-108:/opt$ ros2 topic echo /tf_static | grep detected_object1
    frame_id: detected_object1
 
 
###### 整个frames: 
 
admin@10-60-141-108:/opt$ ros2 run tf2_tools view_frames
[INFO] [1760857837.755277428] [view_frames]: Listening to tf data for 5.0 seconds...
[INFO] [1760857842.792634785] [view_frames]: Generating graph in frames.pdf file...
[INFO] [1760857842.794833603] [view_frames]: Result:tf2_msgs.srv.FrameGraph_Response(frame_yaml="base_link: \n  parent: 'odom'\n  broadcaster: 'default_authority'\n  rate: 60.690\n  most_recent_transform: 79.483337\n  oldest_transform: 78.033337\n  buffer_length: 1.450\ngoal_frame: \n  parent: 'detected_object1'\n  broadcaster: 'default_authority'\n  rate: 10000.000\n  most_recent_transform: 0.000000\n  oldest_transform: 0.000000\n  buffer_length: 0.000\ntarget1_frame: \n  parent: 'base_link'\n  broadcaster: 'default_authority'\n  rate: 10000.000\n  most_recent_transform: 0.000000\n  oldest_transform: 0.000000\n  buffer_length: 0.000\ntarget2_frame: \n  parent: 'base_link'\n  broadcaster: 'default_authority'\n  rate: 10000.000\n  most_recent_transform: 0.000000\n  oldest_transform: 0.000000\n  buffer_length: 0.000\nsoup_can: \n  parent: 'base_link'\n  broadcaster: 'default_authority'\n  rate: 10000.000\n  most_recent_transform: 0.000000\n  oldest_transform: 0.000000\n  buffer_length: 0.000\nworld: \n  parent: 'base_link'\n  broadcaster: 'default_authority'\n  rate: 10000.000\n  most_recent_transform: 0.000000\n  oldest_transform: 0.000000\n  buffer_length: 0.000\nfront_stereo_camera_right: \n  parent: 'base_link'\n  broadcaster: 'default_authority'\n  rate: 10000.000\n  most_recent_transform: 0.000000\n  oldest_transform: 0.000000\n  buffer_length: 0.000\nfront_stereo_camera_left: \n  parent: 'base_link'\n  broadcaster: 'default_authority'\n  rate: 10000.000\n  most_recent_transform: 0.000000\n  oldest_transform: 0.000000\n  buffer_length: 0.000\nbase: \n  parent: 'base_link'\n  broadcaster: 'default_authority'\n  rate: 10000.000\n  most_recent_transform: 0.000000\n  oldest_transform: 0.000000\n  buffer_length: 0.000\nbase_link_inertia: \n  parent: 'base_link'\n  broadcaster: 'default_authority'\n  rate: 10000.000\n  most_recent_transform: 0.000000\n  oldest_transform: 0.000000\n  buffer_length: 0.000\ntool0: \n  parent: 'flange'\n  broadcaster: 'default_authority'\n  rate: 10000.000\n  most_recent_transform: 0.000000\n  oldest_transform: 0.000000\n  buffer_length: 0.000\nflange: \n  parent: 'wrist_3_link'\n  broadcaster: 'default_authority'\n  rate: 10000.000\n  most_recent_transform: 0.000000\n  oldest_transform: 0.000000\n  buffer_length: 0.000\ngrasp_frame: \n  parent: 'gripper_frame'\n  broadcaster: 'default_authority'\n  rate: 10000.000\n  most_recent_transform: 0.000000\n  oldest_transform: 0.000000\n  buffer_length: 0.000\ngripper_frame: \n  parent: 'robotiq_base_link'\n  broadcaster: 'default_authority'\n  rate: 10000.000\n  most_recent_transform: 0.000000\n  oldest_transform: 0.000000\n  buffer_length: 0.000\nrobotiq_base_link: \n  parent: 'tool0'\n  broadcaster: 'default_authority'\n  rate: 10000.000\n  most_recent_transform: 0.000000\n  oldest_transform: 0.000000\n  buffer_length: 0.000\nleft_inner_finger_pad: \n  parent: 'left_inner_finger'\n  broadcaster: 'default_authority'\n  rate: 10000.000\n  most_recent_transform: 0.000000\n  oldest_transform: 0.000000\n  buffer_length: 0.000\nleft_inner_finger: \n  parent: 'left_outer_finger'\n  broadcaster: 'default_authority'\n  rate: 20.714\n  most_recent_transform: 79.450004\n  oldest_transform: 78.050004\n  buffer_length: 1.400\nleft_outer_finger: \n  parent: 'left_outer_knuckle'\n  broadcaster: 'default_authority'\n  rate: 10000.000\n  most_recent_transform: 0.000000\n  oldest_transform: 0.000000\n  buffer_length: 0.000\nleft_outer_knuckle: \n  parent: 'robotiq_140_base_link'\n  broadcaster: 'default_authority'\n  rate: 20.714\n  most_recent_transform: 79.450004\n  oldest_transform: 78.050004\n  buffer_length: 1.400\nright_inner_finger_pad: \n  parent: 'right_inner_finger'\n  broadcaster: 'default_authority'\n  rate: 10000.000\n  most_recent_transform: 0.000000\n  oldest_transform: 0.000000\n  buffer_length: 0.000\nright_inner_finger: \n  parent: 'right_outer_finger'\n  broadcaster: 'default_authority'\n  rate: 20.714\n  most_recent_transform: 79.450004\n  oldest_transform: 78.050004\n  buffer_length: 1.400\nright_outer_finger: \n  parent: 'right_outer_knuckle'\n  broadcaster: 'default_authority'\n  rate: 10000.000\n  most_recent_transform: 0.000000\n  oldest_transform: 0.000000\n  buffer_length: 0.000\nright_outer_knuckle: \n  parent: 'robotiq_140_base_link'\n  broadcaster: 'default_authority'\n  rate: 20.714\n  most_recent_transform: 79.450004\n  oldest_transform: 78.050004\n  buffer_length: 1.400\nrobotiq_140_base_link: \n  parent: 'robotiq_base_link'\n  broadcaster: 'default_authority'\n  rate: 10000.000\n  most_recent_transform: 0.000000\n  oldest_transform: 0.000000\n  buffer_length: 0.000\nwrist_3_link: \n  parent: 'wrist_2_link'\n  broadcaster: 'default_authority'\n  rate: 20.714\n  most_recent_transform: 79.450004\n  oldest_transform: 78.050004\n  buffer_length: 1.400\nft_frame: \n  parent: 'wrist_3_link'\n  broadcaster: 'default_authority'\n  rate: 10000.000\n  most_recent_transform: 0.000000\n  oldest_transform: 0.000000\n  buffer_length: 0.000\nforearm_link: \n  parent: 'upper_arm_link'\n  broadcaster: 'default_authority'\n  rate: 20.714\n  most_recent_transform: 79.450004\n  oldest_transform: 78.050004\n  buffer_length: 1.400\nupper_arm_link: \n  parent: 'shoulder_link'\n  broadcaster: 'default_authority'\n  rate: 20.714\n  most_recent_transform: 79.450004\n  oldest_transform: 78.050004\n  buffer_length: 1.400\nleft_inner_knuckle: \n  parent: 'robotiq_140_base_link'\n  broadcaster: 'default_authority'\n  rate: 20.714\n  most_recent_transform: 79.450004\n  oldest_transform: 78.050004\n  buffer_length: 1.400\nright_inner_knuckle: \n  parent: 'robotiq_140_base_link'\n  broadcaster: 'default_authority'\n  rate: 20.714\n  most_recent_transform: 79.450004\n  oldest_transform: 78.050004\n  buffer_length: 1.400\nshoulder_link: \n  parent: 'base_link_inertia'\n  broadcaster: 'default_authority'\n  rate: 20.714\n  most_recent_transform: 79.450004\n  oldest_transform: 78.050004\n  buffer_length: 1.400\nwrist_1_link: \n  parent: 'forearm_link'\n  broadcaster: 'default_authority'\n  rate: 20.714\n  most_recent_transform: 79.450004\n  oldest_transform: 78.050004\n  buffer_length: 1.400\nwrist_2_link: \n  parent: 'wrist_1_link'\n  broadcaster: 'default_authority'\n  rate: 20.714\n  most_recent_transform: 79.450004\n  oldest_transform: 78.050004\n  buffer_length: 1.400\n")
``` 

goal_frame 以 detected_object1 为父节点, 但没有更高的父节点

通过以下命令监听谁发布了 转换: 

```
ros2 topic echo /tf --field transforms 2>/dev/null | grep -B 10 -A 10 "detected_object1"
 
例如: 
[geometry_msgs.msg.TransformStamped(header=std_msgs.msg.Header(stamp=builtin_interfaces.msg.Time(sec=281, nanosec=200014665), frame_id='front_stereo_camera_left'), child_frame_id='detected_object1', transform=geometry_msgs.msg.Transform(translation=geometry_msgs.msg.Vector3(x=-0.017812518402934074, y=-0.09190164506435394, z=1.2037791013717651), rotation=geometry_msgs.msg.Quaternion(x=0.7413282698552143, y=0.20171556547319422, z=-0.5913169249186127, w=-0.24512756120864881)))]
``` 

判断确实是检测不到物品, 在rviz中检查镜头, 让物品靠近镜头后, 可以检测: 

![image2025-10-19 21:44:16.png](/assets/01KJC05V9SCYYZTVFCHSBJ40S3/image2025-10-19%2021%3A44%3A16.png)

但有时候能正常识别, 以及跟踪, 以及调整物品位置后跟随, 但有时候不行. 

报错是: 

```
[cumotion_goal_set_planner_node-4] [INFO] [1760881520.446443213] [cumotion_planner]: ESDF  req = geometry_msgs.msg.Point(x=-1.5, y=-1.0, z=-0.2), geometry_msgs.msg.Vector3(x=2.0, y=2.0, z=1.0)
[cumotion_goal_set_planner_node-4] [ERROR] [1760881520.701339261] [cumotion_planner]: ESDF data is empty, try again after few seconds.
[cumotion_goal_set_planner_node-4] [ERROR] [1760881520.703400428] [cumotion_planner]: World update failed.
[move_group-7] [INFO] [1760881520.704936465] [move_group]: Received result
[move_group-7] [INFO] [1760881520.707332999] [move_group]: Received trajectory result
[move_group-7] [ERROR] [1760881520.707384550] [move_group]: No trajectory
[move_group-7] [INFO] [1760881520.707476391] [moveit_move_group_default_capabilities.move_action_capability]: No motion plan found. No execution attempted.
[goal_initializer_node-15] [ERROR] [1760881520.713090896] [goal_init_node]: Planning failed!
``` 

认为是脚本启动时, 需要ESDF时, 还没有计算完成. 但目标计算已经完成, 所以脚本就认为目标并没有改变, 不需要重新计算. 

当移动物品后, 重新触发计算, 可以恢复正常. 

只要离摄像头比较近, 保证 foundationpose 能检测到, 那么机械臂就能正确工作

启动抓取任务: 

```
ros2 launch isaac_manipulator_pick_and_place isaac_sim_workflows.launch.py \
   workflow_type:=pick_and_place object_attachment_type:=sphere use_pose_from_rviz:=True
``` 

在另一个窗口 (注意初始化), 获取对象, 并指定抓取任务: 

```
ros2 action send_goal /get_objects isaac_manipulator_interfaces/action/GetObjects {}
 
(在rviz中, 得指定抓取任务后, 物品识别才会标记出来)
ros2 action send_goal /pick_and_place isaac_manipulator_interfaces/action/PickAndPlace \
"{object_id : 0, place_pose : { position : { x : -0.35, y : 0.2, z : 0.4}, orientation : { w : 0.034, x : 0.996, y : 0.066, z : 0.042}}}"
``` 

![image2025-10-20 0:0:6.png](/assets/01KJC05V9SCYYZTVFCHSBJ40S3/image2025-10-20%200%3A0%3A6.png)

![image2025-10-20 0:0:9.png](/assets/01KJC05V9SCYYZTVFCHSBJ40S3/image2025-10-20%200%3A0%3A9.png)

最终会扔到地上

# 下一步

<https://poe.com/s/6OVKKn9ju4ZP6TWodhHo>
