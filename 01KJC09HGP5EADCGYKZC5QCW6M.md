---
title: 20251226 - 重新开始机械臂+仿生手 完成移液动作
confluence_page_id: 4359473
created_at: 2025-12-26T03:38:22+00:00
updated_at: 2026-01-07T02:55:45+00:00
---

重新配置环境: [20251225 - 重新梳理笔记本环境使用]

之前使用机械臂 MyCobot 280 + 灵心巧手O6, 机械臂的终端负载不足, 导致扛不住仿生手的动作

购买了新的机械臂, 等待到货.

# 对于灵巧手对移液器的握持动作的讨论

研究了单独手部的动作, 如何拿起移液枪, 最终姿态: 

![image2025-12-26 11:33:12.png](/assets/01KJC09HGP5EADCGYKZC5QCW6M/image2025-12-26%2011%3A33%3A12.png)

握持动作: 

  - 靠近移液枪, 让下两指形成一个空腔
  - 然后将机械手往回拉, 让移液枪进入空腔并贴近
  - 收紧下两指, 增加摩擦力

存在几个问题: 

  - 不能拿取宽面, 因为手指长度不足, 无法用手指对移液枪形成包裹动作
  - 即使在最终姿态下, 也不能完成包裹动作, 因为手指的自由度不足, 一个手指只有一个主动关节, 其手指只能形成指定大小的空腔, 不能更大
  - 即使是6自由度的机械手, 多了一个拇指平移的自由度, 其平移的极限没有人手充足, 按压角度是斜的
  - 另外由于拇指主动自由度只有1, 其按压动作轨迹不可调整, 如果握持位置太高, 会按不住开关 (是指关节下放碰触开关). 只能有 下两指 握住移液器

有用的脚本: [can_controller.py](/assets/01KJC09HGP5EADCGYKZC5QCW6M/can_controller.py) : 用html控制仿生手的6个舵机

#   
对于新机械臂的研究

URDF描述: <https://github.com/Synria-Robotics/Synria-Robot-Descriptions/tree/main/synriard/urdf/Alicia_D_v5_6>

加载其URDF: 

```
# 替换相对路径为绝对路径
sed 's#filename="../../meshes/#filename="file:///workspaces/isaac_ros-dev/robots-ai/moveit_and_isaac_sim/config/urdf/Synria-Robot-Descriptions-main/synriard/meshes/#g' /workspaces/isaac_ros-dev/robots-ai/moveit_and_isaac_sim/config/urdf/Synria-Robot-Descriptions-main/synriard/urdf/Alicia_D_v5_6/Alicia_D_v5_6_leader.urdf > /workspaces/isaac_ros-dev/robots-ai/moveit_and_isaac_sim/config/urdf/Synria-Robot-Descriptions-main/synriard/urdf/Alicia_D_v5_6/Alicia_D_v5_6_leader_absolute.urdf

# 加载URDF到rviz: 
sudo apt install ros-$ROS_DISTRO-urdf-tutorial

ros2 launch urdf_tutorial display.launch.py model:=/workspaces/isaac_ros-dev/robots-ai/moveit_and_isaac_sim/config/urdf/Synria-Robot-Descriptions-main/synriard/urdf/Alicia_D_v5_6/Alicia_D_v5_6_leader_absolute.urdf
``` 

效果: 

![image2025-12-26 14:2:31.png](/assets/01KJC09HGP5EADCGYKZC5QCW6M/image2025-12-26%2014%3A2%3A31.png)

要决定怎么将灵巧手安装在机械臂上, 用MeshLab打开相应组件的三维图进行测量: 

Joint5 组件的连接口: 

螺丝内径2.5mm, 孔间距为2.2cm

![image2025-12-26 14:5:19.png](/assets/01KJC09HGP5EADCGYKZC5QCW6M/image2025-12-26%2014%3A5%3A19.png)![image2025-12-26 14:7:7.png](/assets/01KJC09HGP5EADCGYKZC5QCW6M/image2025-12-26%2014%3A7%3A7.png)

# 设计和灵巧手的接头

# 配置玄雅机械臂 配置玄雅机械臂 

<https://docs.sparklingrobo.com/docs/alicia-d-series/follower/doc_04_ROS2_Humble>

其中: 

```
#rosdep 需要替换: 
 
rosdep install --from-paths src --ignore-src -r -y --skip-keys="warehouse_ros_mongo"

``` 

当卸下夹爪, 最后一级不连接舵机时, 开机初始化会失败, 导致状态失败

使用如下命令, 可以操作机械臂:

```
ros2 topic pub --once /joint_commands sensor_msgs/msg/JointState "{
  header: {stamp: {sec: 0, nanosec: 0}, frame_id: ''},
  name: ['Joint1', 'Joint2', 'Joint3', 'Joint4', 'Joint5', 'Joint6'],
  position: [0.1, 0.0, 0.0, 0.0, 0.0, 1.0],
  velocity: [],
  effort: []
}"
``` 

但使用官方的moveit脚本, 不能操作机械臂, 显示操作成功, 但实际上没有移动

  - 解决: 官方提供的 `ros2 launch alicia_d_driver alicia_d_driver.launch.py` 和 `ros2 launch alicia_d_moveit real_robot.launch.py` 不能同时启动, 否则会重复占用端口.
  - 

修改 slider 和 moveit , commit: a49c6301d8a02a0b5ca7be61dcf389725b52e5d8

手眼标定: 

```
# 启动slider
python3 slider/slider.launch.py --use-sim-time \
	--enable-apriltag \
	--apriltag-size 0.05 \
    --arm-type aliciad \
    --gripper-type aliciad_gripper \
    --aliciad-gripper-size 50mm \
    --aliciad-port /dev/ttyACM0

  
# 查看tag_detections
ros2 topic echo tag_detections
 
# 松开关节
ros2 service call /slider_hw_stack/toggle_demonstration_mode \
    std_srvs/srv/SetBool "{data: true}"

  
# 记录点位
rm -f eye_to_hand.log; python3 collect_eye_to_hand_calibration.py   --base-frame g_base   --flange-frame Link6  --camera-frame camera_optical   --tag-frame "tag36h11:6" |& tee eye_to_hand.log
   
# 计算结果
python3 collect_eye_to_hand_calibration.py   --calibrate eye_to_hand.log   --base-frame g_base   --flange-frame Link6   --camera-frame camera_optical --trim-percent 20

 
# 恢复关节 
ros2 service call /slider_hw_stack/toggle_demonstration_mode \
    std_srvs/srv/SetBool "{data: false}"

``` 

手眼标定结果偏差会比较大: 

```
[INFO] 离群值剔除敏感性（按综合评分排序，剔除最差比例后重新统计）
  说明：综合评分 = trans_err_mm + rot_err_deg*10（1deg≈10mm，仅用于排序）
  - 去掉最差  0%（保留 14/14）: 平移 mean=60.60mm std=23.30mm, 旋转 mean=4.385deg std=2.419deg
  - 去掉最差  5%（保留 14/14）: 平移 mean=60.60mm std=23.30mm, 旋转 mean=4.385deg std=2.419deg
  - 去掉最差 10%（保留 13/14）: 平移 mean=58.04mm std=24.13mm, 旋转 mean=3.682deg std=1.416deg
  - 去掉最差 20%（保留 12/14）: 平移 mean=54.61mm std=23.19mm, 旋转 mean=3.561deg std=1.347deg
  - 去掉最差 30%（保留 10/14）: 平移 mean=47.15mm std=13.65mm, 旋转 mean=3.582deg std=1.090deg
  - 去掉最差 40%（保留 9/14）: 平移 mean=43.19mm std=13.10mm, 旋转 mean=3.598deg std=1.154deg
  - 去掉最差 50%（保留 7/14）: 平移 mean=37.44mm std=11.38mm, 旋转 mean=3.628deg std=1.170deg
``` 

启动命令: 

```
python3 slider/slider.launch.py --use-sim-time \
	--enable-apriltag \
	--apriltag-size 0.02 \
    --arm-type aliciad \
    --gripper-type aliciad_gripper \
    --aliciad-gripper-size 50mm \
    --aliciad-port /dev/ttyACM0
  
python3 moveit/moveit.launch.py --use-sim-time \
    --arm-type aliciad \
    --gripper-type aliciad_gripper \
    --aliciad-gripper-size 50mm \
    --use-rviz 
 
# 测试动作序列: 
python3 stereo_tag_arm_scheduler.py --arm-type aliciad --gripper-type aliciad_gripper --test
 
# 正式运行: 
python3 stereo_tag_arm_scheduler.py   --use-sim-time \
--arm-type aliciad --gripper-type aliciad_gripper \
--tag1-id 6 --tag2-id 7 \
--ee-offset-z -0.2 --ik-timeout-sec 5 \
--hold-on-ik-fail \
--ik-fail-tf-child-frame ik_target_debug \
--ik-fail-tf-rate-hz 10 \
--confirm-before-execute
``` 

操作机械臂的命令: 

```
# 操作机械臂
ros2 action send_goal /Alicia_controller/follow_joint_trajectory \
  control_msgs/action/FollowJointTrajectory \
  "{
    trajectory: {
      joint_names: ['Joint1', 'Joint2', 'Joint3', 'Joint4', 'Joint5', 'Joint6'],
      points: [
        {
          positions: [0.52, 0.3, 0.0, 0.0, 0.0, 0.0],
          time_from_start: {sec: 2, nanosec: 0}
        }
      ]
    }
  }"
 
# 操作夹爪
 
# 关闭
ros2 action send_goal /Gripper_controller/follow_joint_trajectory control_msgs/action/FollowJointTrajectory "{trajectory: {joint_names: ['Gripper'], points: [{positions: [0.0], time_from_start: {sec: 1}}]}}"

# 打开
ros2 action send_goal /Gripper_controller/follow_joint_trajectory control_msgs/action/FollowJointTrajectory "{trajectory: {joint_names: ['Gripper'], points: [{positions: [0.025], time_from_start: {sec: 1}}]}}"

# 半开
ros2 action send_goal /Gripper_controller/follow_joint_trajectory control_msgs/action/FollowJointTrajectory "{trajectory: {joint_names: ['Gripper'], points: [{positions: [0.020], time_from_start: {sec: 1}}]}}"
``` 

正式运行测试: 

commit: e3fe6e8fbe486aa67188447dc65bbdf657cce191

```
python3 stereo_tag_arm_scheduler.py   --use-sim-time \
--arm-type aliciad --gripper-type aliciad_gripper \
--tag1-id 6 --tag2-id 7 \
--ee-offset-z -0.2 --ik-timeout-sec 5 \
--hold-on-ik-fail \
--ik-fail-tf-child-frame ik_target_debug \
--ik-fail-tf-rate-hz 10 \
--confirm-before-execute
``` 

commit: e7a19809b712b7424c1c219efea15244015637b6

  - 增加了微调模式, 来进行手工操作: 

```
--enable-fine-tune
```

发现新的问题, 因为机械手变长, 目标与相机之间的距离变长, 虽然增大了apriltag的大小到 20mm, 但标签位置识别出来仍然不准

# 尝试使用yolo + 深度, 来进行位姿标记

先增加 进行 相机 + YOLO 识别的脚本, 测试效果

commit: b775cc95dc8de2b8a1ab9562ea3291fe738ab8c4

使用方法: 

```
窗口1: YOLO接入相机
python3 yolo_pipette_detector_node.py --ros-args -p detection_interval_sec:=10.0
 
窗口2: 查看结果
ros2 run rqt_image_view rqt_image_view
``` 

发现: 之前在 [20251216 - 尝试YOLO] 中, 使用的图片太干净. 在真实相机中, 因为背景 + 光照原因, 无法识别.

处于实验考虑, 考虑就用真实场景进行标注测试

# 尝试使用Qwen-Image-edit 进行识别

commit: 2348d4f7febaf47920c83d4955c58dbed91d5a69

使用slider + 脚本, 运行命令: 

```
运行slider: ...
 
运行脚本: python3 qwen_pipette_locator.py
``` 

可以获得比较好的结果: 

![image2025-12-31 14:41:51.png](/assets/01KJC09HGP5EADCGYKZC5QCW6M/image2025-12-31%2014%3A41%3A51.png)

日志中可以获得红色圆心坐标: 

```
移液器尖端坐标: (316, 254)
红色圆点半径: 6 像素
``` 

增加点云识别: 

commit: 2d0b8966759d46a5c7713bdbc35d2c4801f8a235

执行命令: 

```
# 启动slider, 其中会启动深度+点云生成
 
# 启动识别
python3 qwen_pipette_locator.py
``` 

发现问题: 在50cm左右的深度, 点云分布很稀疏, 查询不到有效的深度

将相机与移液器调整到76cm左右, 才能有概率查询到有效深度. 但这样拍出来的移液器比较小, 导致大模型标记时出现了偏差 (在50cm深度时, 偏差很少)

还需要在移液器上做彩带标记, 和背景色分开, 才能增加深度有效点:

![image2025-12-31 16:38:57.png](/assets/01KJC09HGP5EADCGYKZC5QCW6M/image2025-12-31%2016%3A38%3A57.png)

![image2025-12-31 16:41:25.png](/assets/01KJC09HGP5EADCGYKZC5QCW6M/image2025-12-31%2016%3A41%3A25.png)

考虑直接在移液枪上进行彩色圆形标记, 直接使用圆形识别算法, 这样就跳过大模型标记这个环节, 作为demo应该够用

# 重新设计接口

![image2025-12-29 13:16:16.png](/assets/01KJC09HGP5EADCGYKZC5QCW6M/image2025-12-29%2013%3A16%3A16.png)

![image2025-12-29 13:18:33.png](/assets/01KJC09HGP5EADCGYKZC5QCW6M/image2025-12-29%2013%3A18%3A33.png)

<https://docs.sparklingrobo.com/docs/alicia-d-series/follower/doc_04_ROS2_Humble>

其中: 

```
#rosdep 需要替换: 
 
rosdep install --from-paths src --ignore-src -r -y --skip-keys="warehouse_ros_mongo"

``` 

当卸下夹爪, 最后一级不连接舵机时, 开机初始化会失败, 导致状态失败

使用如下命令, 可以操作机械臂:

```
ros2 topic pub --once /joint_commands sensor_msgs/msg/JointState "{
  header: {stamp: {sec: 0, nanosec: 0}, frame_id: ''},
  name: ['Joint1', 'Joint2', 'Joint3', 'Joint4', 'Joint5', 'Joint6'],
  position: [0.1, 0.0, 0.0, 0.0, 0.0, 1.0],
  velocity: [],
  effort: []
}"
``` 

但使用官方的moveit脚本, 不能操作机械臂, 显示操作成功, 但实际上没有移动

  - 解决: 官方提供的 `ros2 launch alicia_d_driver alicia_d_driver.launch.py` 和 `ros2 launch alicia_d_moveit real_robot.launch.py` 不能同时启动, 否则会重复占用端口.
  - 

修改 slider 和 moveit , commit: a49c6301d8a02a0b5ca7be61dcf389725b52e5d8

手眼标定: 

```
# 启动slider
python3 slider/slider.launch.py --use-sim-time \
	--enable-apriltag \
	--apriltag-size 0.05 \
    --arm-type aliciad \
    --gripper-type aliciad_gripper \
    --aliciad-gripper-size 50mm \
    --aliciad-port /dev/ttyACM0

  
# 查看tag_detections
ros2 topic echo /tag_detections
 
# 松开关节
ros2 service call /slider_hw_stack/toggle_demonstration_mode \
    std_srvs/srv/SetBool "{data: true}"

  
# 记录点位
rm -f eye_to_hand.log; python3 collect_eye_to_hand_calibration.py   --base-frame g_base   --flange-frame Link6  --camera-frame camera_optical   --tag-frame "tag36h11:6" |& tee eye_to_hand.log
   
# 计算结果
python3 collect_eye_to_hand_calibration.py   --calibrate eye_to_hand.log   --base-frame g_base   --flange-frame Link6   --camera-frame camera_optical --trim-percent 20

 
# 恢复关节 
ros2 service call /slider_hw_stack/toggle_demonstration_mode \
    std_srvs/srv/SetBool "{data: false}"

``` 

手眼标定结果偏差会比较大: 

```
[INFO] 离群值剔除敏感性（按综合评分排序，剔除最差比例后重新统计）
  说明：综合评分 = trans_err_mm + rot_err_deg*10（1deg≈10mm，仅用于排序）
  - 去掉最差  0%（保留 14/14）: 平移 mean=60.60mm std=23.30mm, 旋转 mean=4.385deg std=2.419deg
  - 去掉最差  5%（保留 14/14）: 平移 mean=60.60mm std=23.30mm, 旋转 mean=4.385deg std=2.419deg
  - 去掉最差 10%（保留 13/14）: 平移 mean=58.04mm std=24.13mm, 旋转 mean=3.682deg std=1.416deg
  - 去掉最差 20%（保留 12/14）: 平移 mean=54.61mm std=23.19mm, 旋转 mean=3.561deg std=1.347deg
  - 去掉最差 30%（保留 10/14）: 平移 mean=47.15mm std=13.65mm, 旋转 mean=3.582deg std=1.090deg
  - 去掉最差 40%（保留 9/14）: 平移 mean=43.19mm std=13.10mm, 旋转 mean=3.598deg std=1.154deg
  - 去掉最差 50%（保留 7/14）: 平移 mean=37.44mm std=11.38mm, 旋转 mean=3.628deg std=1.170deg
``` 

启动命令: 

```
python3 slider/slider.launch.py --use-sim-time \
	--enable-apriltag \
	--apriltag-size 0.02 \
    --arm-type aliciad \
    --gripper-type aliciad_gripper \
    --aliciad-gripper-size 50mm \
    --aliciad-port /dev/ttyACM0
  
python3 moveit/moveit.launch.py --use-sim-time \
    --arm-type aliciad \
    --gripper-type aliciad_gripper \
    --aliciad-gripper-size 50mm \
    --use-rviz 
 
# 测试动作序列: 
python3 stereo_tag_arm_scheduler.py --arm-type aliciad --gripper-type aliciad_gripper --test
 
# 正式运行: 
python3 stereo_tag_arm_scheduler.py   --use-sim-time \
--arm-type aliciad --gripper-type aliciad_gripper \
--tag1-id 6 --tag2-id 7 \
--ee-offset-z -0.2 --ik-timeout-sec 5 \
--hold-on-ik-fail \
--ik-fail-tf-child-frame ik_target_debug \
--ik-fail-tf-rate-hz 10 \
--confirm-before-execute
``` 

操作机械臂的命令: 

```
# 操作机械臂
ros2 action send_goal /Alicia_controller/follow_joint_trajectory \
  control_msgs/action/FollowJointTrajectory \
  "{
    trajectory: {
      joint_names: ['Joint1', 'Joint2', 'Joint3', 'Joint4', 'Joint5', 'Joint6'],
      points: [
        {
          positions: [0.52, 0.3, 0.0, 0.0, 0.0, 0.0],
          time_from_start: {sec: 2, nanosec: 0}
        }
      ]
    }
  }"
 
# 操作夹爪
 
# 关闭
ros2 action send_goal /Gripper_controller/follow_joint_trajectory control_msgs/action/FollowJointTrajectory "{trajectory: {joint_names: ['Gripper'], points: [{positions: [0.0], time_from_start: {sec: 1}}]}}"

# 打开
ros2 action send_goal /Gripper_controller/follow_joint_trajectory control_msgs/action/FollowJointTrajectory "{trajectory: {joint_names: ['Gripper'], points: [{positions: [0.025], time_from_start: {sec: 1}}]}}"

# 半开
ros2 action send_goal /Gripper_controller/follow_joint_trajectory control_msgs/action/FollowJointTrajectory "{trajectory: {joint_names: ['Gripper'], points: [{positions: [0.020], time_from_start: {sec: 1}}]}}"
``` 

正式运行测试: 

commit: e3fe6e8fbe486aa67188447dc65bbdf657cce191

```
python3 stereo_tag_arm_scheduler.py   --use-sim-time \
--arm-type aliciad --gripper-type aliciad_gripper \
--tag1-id 6 --tag2-id 7 \
--ee-offset-z -0.2 --ik-timeout-sec 5 \
--hold-on-ik-fail \
--ik-fail-tf-child-frame ik_target_debug \
--ik-fail-tf-rate-hz 10 \
--confirm-before-execute
``` 

commit: e7a19809b712b7424c1c219efea15244015637b6

  - 增加了微调模式, 来进行手工操作: 

```
--enable-fine-tune
```

发现新的问题, 因为机械手变长, 目标与相机之间的距离变长, 虽然增大了apriltag的大小到 20mm, 但标签位置识别出来仍然不准

# 尝试使用yolo + 深度, 来进行位姿标记

先增加 进行 相机 + YOLO 识别的脚本, 测试效果

commit: b775cc95dc8de2b8a1ab9562ea3291fe738ab8c4

使用方法: 

```
窗口1: YOLO接入相机
python3 yolo_pipette_detector_node.py --ros-args -p detection_interval_sec:=10.0
 
窗口2: 查看结果
ros2 run rqt_image_view rqt_image_view
``` 

发现: 之前在 [20251216 - 尝试YOLO] 中, 使用的图片太干净. 在真实相机中, 因为背景 + 光照原因, 无法识别.

处于实验考虑, 考虑就用真实场景进行标注测试

# 尝试使用Qwen-Image-edit 进行识别

commit: 2348d4f7febaf47920c83d4955c58dbed91d5a69

使用slider + 脚本, 运行命令: 

```
运行slider: ...
 
运行脚本: python3 qwen_pipette_locator.py
``` 

可以获得比较好的结果: 

![image2025-12-31 14:41:51.png](/assets/01KJC09HGP5EADCGYKZC5QCW6M/image2025-12-31%2014%3A41%3A51.png)

日志中可以获得红色圆心坐标: 

```
移液器尖端坐标: (316, 254)
红色圆点半径: 6 像素
``` 

增加点云识别: 

commit: 2d0b8966759d46a5c7713bdbc35d2c4801f8a235

执行命令: 

```
# 启动slider, 其中会启动深度+点云生成
 
# 启动识别
python3 qwen_pipette_locator.py
``` 

发现问题: 在50cm左右的深度, 点云分布很稀疏, 查询不到有效的深度

将相机与移液器调整到76cm左右, 才能有概率查询到有效深度. 但这样拍出来的移液器比较小, 导致大模型标记时出现了偏差 (在50cm深度时, 偏差很少)

还需要在移液器上做彩带标记, 和背景色分开, 才能增加深度有效点:

![image2025-12-31 16:38:57.png](/assets/01KJC09HGP5EADCGYKZC5QCW6M/image2025-12-31%2016%3A38%3A57.png)

![image2025-12-31 16:41:25.png](/assets/01KJC09HGP5EADCGYKZC5QCW6M/image2025-12-31%2016%3A41%3A25.png)

考虑直接在移液枪上进行彩色圆形标记, 直接使用圆形识别算法, 这样就跳过大模型标记这个环节, 作为demo应该够用

# 简化深度检测

在移液器上贴一个蓝色块, 直接识别蓝色块的中心, 然后进行深度查询

commit: ef41bc543b044bf3efc7276421154556cfd913f0

命令: 

```
python3 blue_rectangle_locator.py --ros-args -p use_sim_time:=true
``` 

在深度 70-90 cm 深度检测没问题. 当缩小到50cm, 深度仍然检测为90cm, 场景示意: 

![image2026-1-2 13:33:13.png](/assets/01KJC09HGP5EADCGYKZC5QCW6M/image2026-1-2%2013%3A33%3A13.png)![image2026-1-2 13:33:34.png](/assets/01KJC09HGP5EADCGYKZC5QCW6M/image2026-1-2%2013%3A33%3A34.png)

即使增加了多点的深度查询矫正, 深度仍然错误为90cm: 

```
[3] 查询3D坐标...

  深度一致性检查：查询5个点...
    邻域搜索: 中心=(329, 343), 半径=3
    邻域搜索失败：未找到有效点
    邻域搜索: 中心=(333, 317), 半径=3
    邻域搜索成功: 找到4个有效点, 深度=0.8984m
    邻域搜索: 中心=(346, 319), 半径=3
    邻域搜索失败：未找到有效点
    邻域搜索: 中心=(341, 345), 半径=3
    邻域搜索失败：未找到有效点
    中心(337,331): Z=0.9077m ✓
    角点1(329,343): 无效 ✗
    角点2(333,317): Z=0.8984m ✓
    角点3(346,319): 无效 ✗
    角点4(341,345): 无效 ✗

  深度统计:
    有效点数: 2/5
    平均深度: 0.9030m
    标准差: 0.0047m
    深度范围: [0.8984m, 0.9077m]
    最大偏差: 0.0093m (0.9cm)
    ✓ 深度一致性检查通过（偏差 ≤ 5cm）
``` 

当真实距离 > 65cm, 深度测量是正确的. 当 <50cm, 深度就错误为90cm. 

解决: 在深度检测节点, 增加参数 "disparity_range:=128", 缩短近距匹配的阈值, 得到结果: 

```
  深度一致性检查：查询5个点...
    中心(313,348): Z=0.5058m ✓
    角点1(303,361): Z=0.8703m ✓
    角点2(308,333): Z=0.5062m ✓
    角点3(322,336): Z=0.5042m ✓
    角点4(317,363): Z=0.5093m ✓

  深度统计:
    有效点数: 5/5
    平均深度: 0.5791m
    标准差: 0.1456m
    深度范围: [0.5042m, 0.8703m]
    最大偏差: 0.3661m (36.6cm)
    ⚠️  深度一致性检查失败（偏差 > 5cm）
    ⚠️  可能原因：矩形边缘跨越深度不连续区域、遮挡、点云噪声
``` 

剔除异常点即可. 这样, 在低分辨率的双目相机下, 也可以基本准确的检测深度, 并且不需要使用大模型进行位置识别

# 重新设计接口

![image2025-12-29 13:16:16.png](/assets/01KJC09HGP5EADCGYKZC5QCW6M/image2025-12-29%2013%3A16%3A16.png)

![image2025-12-29 13:18:33.png](/assets/01KJC09HGP5EADCGYKZC5QCW6M/image2025-12-29%2013%3A18%3A33.png)

用机械臂提供的stl文件, 在淘宝上, 找人修改, 然后找3D打印工厂打印了接口适配器, 进行组装, 效果: 

![image2026-1-2 11:27:43.png](/assets/01KJC09HGP5EADCGYKZC5QCW6M/image2026-1-2%2011%3A27%3A43.png)

主要解决的问题: 

  - 仿生手的末端接口, 接线需要一个空间, 所以用长螺丝支起了这个空间
  - 适配器(白色) 需要同时向两端拧螺丝, 这一点在设计时有所疏忽, 勉强够用. 重新设计连接器时, 需要考虑解决

# 遇到机械臂无法调度的问题

启动slider, 用手工测试命令: 

```
admin@bot:/workspaces/isaac_ros-dev/robots-ai/moveit_and_isaac_sim/linkerhand_and_stereo_camera$ ros2 action send_goal /Alicia_controller/follow_joint_trajectory   control_msgs/action/FollowJointTrajectory   "{trajectory: {joint_names: ['Joint1', 'Joint2', 'Joint3', 'Joint4', 'Joint5', 'Joint6'], \
   points: [{positions: [1.4, 0.0, 0.0, 0.0, 0.0, 1.0], time_from_start: {sec: 3, nanosec: 0}}]}}"
``` 

无法移动机械臂, slider日志正常: 

```
[python3-16] [INFO] [1767513376.494146320] [slider_hw_stack]: slider_hw_stack 启动完成 [arm=aliciad, gripper=none]
[ros2_control_node-3] [INFO] [1767513380.716205774] [Alicia_controller]: Received new action goal
[ros2_control_node-3] [INFO] [1767513380.716262611] [Alicia_controller]: Accepted new action goal
[ros2_control_node-3] [INFO] [1767513383.740414585] [Alicia_controller]: Goal reached, success!
[ros2_control_node-3] [INFO] [1767513396.655940372] [AliciaDHardwareInterface]: Sending joint command with speed: 20.0 deg/s
[ros2_control_node-3] [INFO] [1767513422.421880623] [AliciaDHardwareInterface]: Sending joint command with speed: 20.0 deg/s
``` 

考虑重启机械臂, USB连接有问题, 但没有采纳反馈信号, 于是无法判断是否正确

# 继续进行移液器抓取

commit: c47f03eb71d87b02481099086eb446b674bfac77

只调度机械臂, 使用蓝色色块作为定位方案, 执行命令: 

```
python3 slider/slider.launch.py --use-sim-time      --enable-apriltag       --apriltag-size 0.05     --arm-type aliciad     --gripper-type none     --aliciad-port /dev/ttyACM0
 
python3 moveit/moveit.launch.py --use-sim-time     --arm-type aliciad     --gripper-type none     --use-rviz
 
python3 blue_rectangle_arm_scheduler.py --skip-gripper-preset   --confirm-before-execute   --gripper-type none  --ee-offset-x -0.1 --ee-offset-y 0 --ee-offset-z -0.15 --enable-fine-tune
``` 

发现对蓝色色块的定位不准: 

```
第一次: 

[INFO] [1767590101.983866832] [blue_rectangle_arm_scheduler]:   目标位置: (+0.110, +0.267, +0.206)
[INFO] [1767590101.984622086] [blue_rectangle_arm_scheduler]:   目标姿态: (+0.074, +0.672, +0.684, +0.274)

第二次: 
[INFO] [1767590220.380376204] [blue_rectangle_arm_scheduler]:   目标位置: (+0.104, +0.287, +0.237)
[INFO] [1767590220.381055587] [blue_rectangle_arm_scheduler]:   目标姿态: (+0.062, +0.737, +0.612, +0.279)
``` 

都会偏离几公分到十几公分. 不确定是因为手眼标定时的偏差, 还是因为相机精度不足

更换gemini max相机进行测试

# gemini max 相机

教程: <https://www.yahboom.com/build/id/10300/cid/656>

提取码: gmax

~~安装 (旧的尝试, 废弃):~~

````
sudo apt install libgflags-dev nlohmann-json3-dev libgoogle-glog-dev ros-humble-image-transport ros-humble-image-publisher  ros-humble-pangolin ros-humble-image-transport-plugins
 
# 从教程上, 下载ros2源码, src目录放到 ~/orbbec_ws/src 中
 
cd ~/orbbec_ws/src
git clone https://github.com/shanpenghui/ORB_SLAM2_fixed.git ORB_SLAM2
cd ORB_SLAM2
 
修改CmakeLists.txt
==========
 
将 add_definitions(-std=c++11) 改成 add_definitions(-std=c++14)
修改: 
```
target_link_libraries(${PROJECT_NAME}
${OpenCV_LIBS}
${EIGEN3_LIBS}
${Pangolin_LIBRARIES}
${PROJECT_SOURCE_DIR}/Thirdparty/DBoW2/lib/libDBoW2.so
${PROJECT_SOURCE_DIR}/Thirdparty/g2o/lib/libg2o.so
)
```
删除 "${EIGEN3_LIBS}" 这一行
 
并增加补丁: 
```
find_package(Eigen3 3.1.0 REQUIRED)

# ================== 新增补丁开始 ==================
# 修复 Pangolin 导致的 Eigen3::Eigen 链接错误
if(NOT TARGET Eigen3::Eigen)
    message(STATUS "Eigen3::Eigen target not found. Creating dummy interface target.")
    add_library(Eigen3::Eigen INTERFACE IMPORTED)
    set_target_properties(Eigen3::Eigen PROPERTIES
        INTERFACE_INCLUDE_DIRECTORIES "${EIGEN3_INCLUDE_DIR}"
    )
endif()
# ================== 新增补丁结束 ==================

find_package(Pangolin REQUIRED)
```
 
==========
 
grep -rl "CV_LOAD_IMAGE_UNCHANGED" Examples/ | xargs sed -i 's/CV_LOAD_IMAGE_UNCHANGED/cv::IMREAD_UNCHANGED/g'
 
bash build.sh
 
cd ~/orbbec_ws
export ORB_SLAM2_ROOT_DIR=/home/admin/orbbec_ws/src/ORB_SLAM2
find /home/admin/orbbec_ws/src/ORB_SLAM2/include -name "*.h" -print0 | xargs -0 sed -i 's|#include <opencv/cv.h>|#include <opencv2/opencv.hpp>|g'
colcon build
 
```` 

编译SDK:

```
sudo apt install libgflags-dev nlohmann-json3-dev libgoogle-glog-dev ros-humble-image-transport ros-humble-image-publisher  ros-humble-pangolin ros-humble-image-transport-plugins
 
# 从教程上, 下载ros2源码, src目录放到 ~/orbbec_ws/src 中, 只保留 OrbbecSDK_ROS2, 删除其他文件夹
 
cd ~/orbbec_ws/src
colcon build
``` 

将 99-obsensor-libusb.rules 的内容, 放到 宿主机的 /etc/udev/rules.d/99-obsensor-libusb.rules

重新插入USB摄像头, 可以发现: /dev/gemini设备

检查指向设备: 

```
huangyan@bot:~$ ls -alh /dev/g*
lrwxrwxrwx 1 root root 15 Jan  5 07:42 /dev/gemini -> bus/usb/001/049
lrwxrwxrwx 1 root root 15 Jan  5 07:42 /dev/gemini_rgb -> bus/usb/001/048
huangyan@bot:~$ ls -alh /dev/bus/usb/001/049
crw-rw-rw- 1 root video 189, 48 Jan  5 07:42 /dev/bus/usb/001/049
huangyan@bot:~$ ls -alh /dev/bus/usb/001/048
crw-rw-rw- 1 root video 189, 47 Jan  5 07:42 /dev/bus/usb/001/048
``` 

将其在容器中创建: 

```
sudo mknod /dev/bus/usb/001/049 c 189 48
sudo chmod 666 /dev/bus/usb/001/049
sudo mknod /dev/bus/usb/001/048 c 189 47
sudo chmod 666 /dev/bus/usb/001/048
``` 

运行: 

```
ros2 launch orbbec_camera gemini.launch.xml
``` 

可以看到话题: 

```
admin@bot:/workspaces/isaac_ros-dev/robots-ai/moveit_and_isaac_sim/linkerhand_and_stereo_camera$ ros2 topic list
1767599201.860842 [98]       ros2: config: //CycloneDDS/Domain/General: 'NetworkInterfaceAddress': deprecated element (/workspaces/isaac_ros-dev/b.xml line 9)
/camera/color/camera_info
/camera/color/image_raw
/camera/color/image_raw/compressed
/camera/color/image_raw/compressedDepth
/camera/color/image_raw/theora
/camera/depth/camera_info
/camera/depth/image_raw
/camera/depth/image_raw/compressed
/camera/depth/image_raw/compressedDepth
/camera/depth/image_raw/theora
/camera/depth/points
/camera/ir/camera_info
/camera/ir/image_raw
/camera/ir/image_raw/compressed
/camera/ir/image_raw/compressedDepth
/camera/ir/image_raw/theora
/parameter_events
/rosout
/tf
/tf_static
``` 

点云显示: 

![image2026-1-5 15:54:1.png](/assets/01KJC09HGP5EADCGYKZC5QCW6M/image2026-1-5%2015%3A54%3A1.png)

与[20251209 - 调试双目摄像头] 中的点云比起来, 点云密度很高

注意: gemini max发布的 camera_info 中时间戳为0, apriltag识别不出来, 使用 camera_info_fixer.py 进行修复

  - commit: bafeaf33756e6b9275b532f8a1b56761a51f35b7

开始使用gemini-max: 

  - 内参标定: 不需要内参标定, 其SDK中显示, 内参是相机出厂内置
  - 坐标系确认: 相机SDK会自己发布 camera_link 和 camera_color_optical_frame 坐标系, 通过ros2命令验证两个坐标系的关系是标准变换无误: 

```
admin@bot:/workspaces/isaac_ros-dev/robots-ai/moveit_and_isaac_sim/linkerhand_and_stereo_camera$ ros2 run tf2_ros tf2_echo camera_color_optical_frame camera_link

At time 1767668011.72768865
- Translation: [0.000, 0.000, 0.000]
- Rotation: in Quaternion (xyzw) [0.500, -0.500, 0.500, 0.500]
- Rotation: in RPY (radian) [0.785, -1.571, 0.000]
- Rotation: in RPY (degree) [45.000, -90.000, 0.000]
- Matrix:
  0.000 -1.000  0.000  0.000
  0.000  0.000 -1.000  0.000
  1.000  0.000  0.000  0.000
  0.000  0.000  0.000  1.000
```

  - 手眼标定: 

```
# 启动slider
python3 slider/slider.launch.py --use-sim-time \
 --enable-apriltag \
 --apriltag-size 0.05 \
 --arm-type aliciad \
 --gripper-type none \
 --aliciad-port /dev/ttyACM0

# 查看tag_detections
ros2 topic echo /tag_detections

# 松开关节
ros2 service call /slider_hw_stack/toggle_demonstration_mode \
    std_srvs/srv/SetBool "{data: true}"

# 记录点位
rm -f eye_to_hand.log; python3 collect_eye_to_hand_calibration.py   --base-frame g_base   --flange-frame Link6  --camera-frame camera_color_optical_frame --tag-frame "tag36h11:6" |& tee eye_to_hand.log

# 计算结果
python3 collect_eye_to_hand_calibration.py   --calibrate eye_to_hand.log   --base-frame g_base   --flange-frame Link6   --camera-frame camera_color_optical_frame --trim-percent 20

# 恢复关节
ros2 service call /slider_hw_stack/toggle_demonstration_mode \
    std_srvs/srv/SetBool "{data: false}"
```

    - 标定偏差仍然很大: 

```
[INFO] 离群值剔除敏感性（按综合评分排序，剔除最差比例后重新统计）
  说明：综合评分 = trans_err_mm + rot_err_deg*10（1deg≈10mm，仅用于排序）
  - 去掉最差  0%（保留 17/17）: 平移 mean=104.96mm std=43.46mm, 旋转 mean=11.024deg std=2.286deg
  - 去掉最差  5%（保留 17/17）: 平移 mean=104.96mm std=43.46mm, 旋转 mean=11.024deg std=2.286deg
  - 去掉最差 10%（保留 16/17）: 平移 mean=96.72mm std=40.08mm, 旋转 mean=10.886deg std=2.479deg
  - 去掉最差 20%（保留 14/17）: 平移 mean=90.22mm std=28.93mm, 旋转 mean=10.529deg std=1.694deg
  - 去掉最差 30%（保留 12/17）: 平移 mean=82.53mm std=22.60mm, 旋转 mean=9.808deg std=2.509deg
  - 去掉最差 40%（保留 11/17）: 平移 mean=75.02mm std=19.83mm, 旋转 mean=9.314deg std=3.191deg
  - 去掉最差 50%（保留 9/17）: 平移 mean=69.25mm std=27.02mm, 旋转 mean=6.695deg std=3.067deg
```

    - 关于查询点云的逻辑: gemini-max发布的点云是非组织化的点云, 使用起来麻烦, 直接从深度图进行查询. 但其深度图大小为 640*400, 需要开启 "depth_registration:=true"参数, 让硬件进行深度对齐
    - 发现: 深度与实际情况偏差很大

一些可能的尝试:

更换官方的包: 

```
# https://github.com/orbbec/OrbbecSDK_ROS2
apt install ros-humble-orbbec-camera ros-humble-orbbec-description
更换udev的文件: /opt/ros/$ROS_DISTRO/share/orbbec_camera/udev/99-obsensor-libusb.rules
 
sudo udevadm control --reload-rules && sudo udevadm trigger

ros2 run orbbec_camera list_devices_node 
没有任何输出

失败
``` 

使用命令指定分辨率: 

```
ros2 launch orbbec_camera gemini.launch.xml \
  color_width:=640 \
  color_height:=480 \
  depth_width:=640 \
  depth_height:=400 \
  enable_ir:=false \
  depth_registration:=true \
  enable_d2c_viewer:=true
 
这个情况下, depth_registration 可以帮忙对齐两张图像
 
--
 
 
ros2 launch orbbec_camera gemini.launch.xml \
  color_width:=1280 \
  color_height:=720 \
  depth_width:=1280 \
  depth_height:=800 \
  enable_ir:=false

这个情况下, depth_registration 不能对齐图像了, 原因不详
 
另外, enable_ir=false 的原因是 USB 2.0的线宽不够, 暂时禁用, 换用 USB 3.0接口后, 需要再次启用IR
``` 

对于分辨率不同的情况, 得考虑深度查询时如何处理

# 使用官方查看工具

![image2026-1-7 10:55:12.png](/assets/01KJC09HGP5EADCGYKZC5QCW6M/image2026-1-7%2010%3A55%3A12.png)

官方工具查询出的深度等都是正确的, 而且 640*480 在蓝色标签位置的深度均匀且偏差很小
