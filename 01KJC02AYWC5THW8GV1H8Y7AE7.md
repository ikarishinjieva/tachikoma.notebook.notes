---
title: 20251007 - 诊断Isaac Sim和ros2 launch运行时发生OOM
confluence_page_id: 4358776
created_at: 2025-10-07T15:36:59+00:00
updated_at: 2025-10-07T16:02:17+00:00
---

# 步骤

[20251006 - 尝试将Isaac Sim和WebRTC连通] 中的"使用NICE DCV 可以将Isaac Sim和ros2 launch放在一台服务器上运行, 启动过程":

```
10.186.16.136:
dcv create-session test-issac
  
连接到桌面:
- 使用浏览器: 10.186.16.136:8443
- 使用 DCV viewer
 
 
桌面窗口1:
 
cd /data/huangyan/isaac-sim
export http_proxy=http://10.186.16.136:7890
export https_proxy=http://10.186.16.136:7890
export FASTRTPS_DEFAULT_PROFILES_FILE=/data/huangyan/isaac-sim/rtps_udp_profile.xml
export ROS_DOMAIN_ID=77
export RMW_IMPLEMENTATION=rmw_fastrtps_cpp
./isaac-sim.sh
 
打开 https://omniverse-content-production.s3-us-west-2.amazonaws.com/Assets/Isaac/4.2/Isaac/Samples/ROS2/Scenario/isaac_manipulator_ur10e_robotiq_2f_140.usd
开始Play
 
 
桌面窗口2:
export ISAAC_ROS_WS=/data/huangyan/isaac_ros-dev/
cd $ISAAC_ROS_WS && ./src/isaac_ros_common/scripts/run_dev.sh 
进容器
 
容器内
export http_proxy=http://10.186.16.136:7890
export https_proxy=http://10.186.16.136:7890
export FASTRTPS_DEFAULT_PROFILES_FILE=/usr/local/share/middleware_profiles/rtps_udp_profile.xml
cd ${ISAAC_ROS_WS}
export ROS_DOMAIN_ID=77
export RMW_IMPLEMENTATION=rmw_fastrtps_cpp
source install/setup.bash
ros2 launch isaac_manipulator_pick_and_place isaac_sim_workflows.launch.py \
   workflow_type:=object_following pose_estimation_type:=foundationpose
``` 

会发生OOM (服务器200G+的内存都会被占用):

![image2025-10-7 20:10:38.png](/assets/01KJC02AYWC5THW8GV1H8Y7AE7/image2025-10-7%2020%3A10%3A38.png)

运行简单任务: 

```
ros2 launch isaac_manipulator_pick_and_place isaac_sim_workflows.launch.py workflow_type:=pose_to_pose
``` 

仍然会OOM:

![image2025-10-7 20:21:23.png](/assets/01KJC02AYWC5THW8GV1H8Y7AE7/image2025-10-7%2020%3A21%3A23.png)

明确禁用部分组件: 

```
ros2 launch isaac_manipulator_pick_and_place isaac_sim_workflows.launch.py workflow_type:=pose_to_pose enable_segmentation:=False enable_attachment:=False enable_reconstruction:=False
``` 

仍然会发生OOM:

![image2025-10-7 20:30:32.png](/assets/01KJC02AYWC5THW8GV1H8Y7AE7/image2025-10-7%2020%3A30%3A32.png)

增加环境变量export ROS_DOMAIN_ID=77, 并清理/dev/shm, 仍然会OOM, 没有作用: 

![image2025-10-7 20:40:32.png](/assets/01KJC02AYWC5THW8GV1H8Y7AE7/image2025-10-7%2020%3A40%3A32.png)

Isaac Sim使用空文件加载, 内存不会有问题, top: 

![image2025-10-7 21:8:24.png](/assets/01KJC02AYWC5THW8GV1H8Y7AE7/image2025-10-7%2021%3A8%3A24.png)

```
admin@dell-136:/workspaces/isaac_ros-dev$ ros2 node list
/controller_manager
/joint_state_broadcaster
/joint_state_publisher
/launch_ros_17564
/object_attachment
/pose_to_pose_node
/robot_segmenter
/robot_state_publisher
/topic_based_ros2_control_ur_ros2_control_sim
admin@dell-136:/workspaces/isaac_ros-dev$ ros2 topic list
/attached_collision_object
/clock
/clustered_point_cloud
/collision_object
/cumotion/camera_1/robot_mask
/cumotion/camera_1/world_depth
/cumotion/robot_segmenter/robot_spheres
/display_contacts
/display_planned_path
/front_stereo_camera/depth/camera_info
/front_stereo_camera/depth/ground_truth
/isaac_joint_commands
/isaac_joint_states
/isaac_parsed_joint_states
/joint_state_broadcaster/transition_event
/joint_states
/monitored_planning_scene
/motion_plan_request
/nearby_point_cloud
/object_origin_frame
/object_point_cloud
/parameter_events
/planning_scene
/planning_scene_world
/robot_description
/robot_sphere_markers
/rosout
/tf
/tf_static
/trajectory_execution_event
/viz_all_spheres/segmenter_attach_object
``` 

修改场景: 将两个camera关掉一个, 剩下的一个是640*480, 并关掉对应的render, 也没有作用

修改场景: 禁用掉如下组件

  - /World/transform_tree_odometry_ur_nvblox
  - 名字里含 nvblox/odometry/transform_tree
  - /World/front_hawk_manipulator
  - /World/hawk_manipulator
  - /World/small_KLT

没有作用, 仍然会OOM: 

![image2025-10-7 21:42:55.png](/assets/01KJC02AYWC5THW8GV1H8Y7AE7/image2025-10-7%2021%3A42%3A55.png)

禁用各个动态组件, 仍然会OOM

使用GT 视觉, 来代替真实视觉, 不会OOM: 

```
ros2 launch isaac_manipulator_pick_and_place isaac_sim_workflows.launch.py workflow_type:=pose_to_pose use_ground_truth_pose_in_sim:=true log_level:=info
``` 

但启用了动态组件后, 即使使用GT视觉, 也会OOM

逐步开启动态组件, 开启到front hawk manipulator时, OOM

禁用后, 再执行, 也会OOM

好像OOM是因为某一个不定期触发的机制有关
