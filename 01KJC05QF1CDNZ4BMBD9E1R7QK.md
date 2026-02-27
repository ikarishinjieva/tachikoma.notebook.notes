---
title: 20251015 - 分析ros-humble-isaac-manipulator-bringup的原理 (机械臂复杂流)
confluence_page_id: 4358914
created_at: 2025-10-15T10:02:03+00:00
updated_at: 2025-10-20T06:38:58+00:00
---

实验文档: <https://nvidia-isaac-ros.github.io/reference_workflows/isaac_manipulator/tutorials/tutorial_isaac_sim.html>

其中有手工变异cuMotion的一段内容, 但另一个文档(<https://nvidia-isaac-ros.github.io/concepts/manipulation/cumotion_moveit/tutorial_isaac_sim.html>)中使用的apt install.

先尝试apt install的方案

# 安装

在ROS2 dev容器中, 安装: 

```
sudo apt-get install -y ros-humble-isaac-manipulator-bringup ros-humble-isaac-manipulator-pick-and-place
``` 

在桌面上, 启动Isaac Sim, 加载 <https://omniverse-content-production.s3-us-west-2.amazonaws.com/Assets/Isaac/4.2/Isaac/Samples/ROS2/Scenario/isaac_manipulator_ur10e_robotiq_2f_140.usd>, 并Play

# 测试pose-to-pose

在桌面的容器中: 

```
ros2 launch isaac_manipulator_pick_and_place isaac_sim_workflows.launch.py \
   workflow_type:=pose_to_pose
``` 

自动弹出rviz, 并且机械臂能轻微活动: 

![image2025-10-15 18:1:35.png](/assets/01KJC05QF1CDNZ4BMBD9E1R7QK/image2025-10-15%2018%3A1%3A35.png)

但尚不能解释 从哪个姿势到哪个姿势 是如何设定的

阅读代码得知: pose_to_pose默认情况下, 末端执行器是 wrist_3_link, 目标帧是 ["target1_frame", "target2_frame"], 其目标是"控制机器人使 wrist_3_link 到达 target1_frame 或 target2_frame 的位置"

在rviz中标记出这几个元素: 

![image2025-10-15 23:7:33.png](/assets/01KJC05QF1CDNZ4BMBD9E1R7QK/image2025-10-15%2023%3A7%3A33.png)

发现规划并没有成功

进行诊断, Isaac Sim报错: 

```
2025-10-16T02:19:18Z [35,376,636ms] [Error] [isaacsim.ros2.bridge.ogn.python.nodes.OgnROS2CameraHelper] type is not supported
``` 

尝试: 

  - 加载5.0版本的模型: [https://omniverse-content-production.s3-us-west-2.amazonaws.com/Assets/Isaac/5.0/Isaac/Samples/ROS2/Scenario/isaac_manipulator_ur10e_robotiq_2f_140.usd](<https://omniverse-content-production.s3-us-west-2.amazonaws.com/Assets/Isaac/4.2/Isaac/Samples/ROS2/Scenario/isaac_manipulator_ur10e_robotiq_2f_140.usd>)
    - 失败, 问题依旧
  - 切换Isaac Sim版本到4.2, 报错消失

重新运行, 机械臂可以在 target1_frame和target2_frame间交替进行: 

![image2025-10-16 10:51:39.png](/assets/01KJC05QF1CDNZ4BMBD9E1R7QK/image2025-10-16%2010%3A51%3A39.png)

正确的日志: 

```
@dell-136:/workspaces/isaac_ros-dev$ ros2 launch isaac_manipulator_pick_and_place isaac_sim_workflows.launch.py    workflow_type:=pose_to_pose
[INFO] [launch]: All log files can be found below /home/admin/.ros/log/2025-10-16-02-52-36-503796-dell-136-1003638
[INFO] [launch]: Default logging verbosity is set to INFO
WARNING:root:Cannot infer URDF from `/opt/ros/humble/share/isaac_manipulator_pick_and_place`. -- using config/ur10e.urdf
WARNING:root:Cannot infer SRDF from `/opt/ros/humble/share/isaac_manipulator_pick_and_place`. -- using config/ur10e.srdf
WARNING:root:"File /opt/ros/humble/share/isaac_manipulator_pick_and_place/config/ur10e.urdf doesn't exist"
WARNING:root:The robot description will be loaded from /robot_description topic 
[INFO] [launch.user]: Loading the 'sim_test_bench' workspace. Ignoring the grid_center_m / grid_size_m parameters of cumotion and esdf visualizer.
[INFO] [launch.user]: Adding nodes [ /nvblox_node ] to container manipulator_container.
[INFO] [launch.user]: Enabling nvblox for '1' 'isaac_sim' cameras configured for the 'sim_test_bench' workspace.
[INFO] [component_container_mt-1]: process started with pid [1003713]
[INFO] [robot_state_publisher-2]: process started with pid [1003715]
[INFO] [joint_state_publisher-3]: process started with pid [1003717]
[INFO] [cumotion_goal_set_planner_node-4]: process started with pid [1003719]
[INFO] [robot_segmenter_node-5]: process started with pid [1003721]
[INFO] [attach_object_server_node-6]: process started with pid [1003723]
[INFO] [move_group-7]: process started with pid [1003725]
[INFO] [rviz2-8]: process started with pid [1003727]
[INFO] [joint_parser_node.py-9]: process started with pid [1003729]
[INFO] [ros2_control_node-10]: process started with pid [1003731]
[INFO] [spawner-11]: process started with pid [1003733]
[INFO] [isaac_sim_gripper_action_server.py-12]: process started with pid [1003737]
[INFO] [spawner-13]: process started with pid [1003739]
[INFO] [pose_to_pose_node-14]: process started with pid [1003748]
[robot_state_publisher-2] [INFO] [1760583157.174810116] [robot_state_publisher]: got segment base
[robot_state_publisher-2] [INFO] [1760583157.174863067] [robot_state_publisher]: got segment base_link
[robot_state_publisher-2] [INFO] [1760583157.174870625] [robot_state_publisher]: got segment base_link_inertia
[robot_state_publisher-2] [INFO] [1760583157.174875441] [robot_state_publisher]: got segment flange
[robot_state_publisher-2] [INFO] [1760583157.174879902] [robot_state_publisher]: got segment forearm_link
[robot_state_publisher-2] [INFO] [1760583157.174884383] [robot_state_publisher]: got segment ft_frame
[robot_state_publisher-2] [INFO] [1760583157.174888665] [robot_state_publisher]: got segment grasp_frame
[robot_state_publisher-2] [INFO] [1760583157.174893009] [robot_state_publisher]: got segment gripper_frame
[robot_state_publisher-2] [INFO] [1760583157.174897621] [robot_state_publisher]: got segment left_inner_finger
[robot_state_publisher-2] [INFO] [1760583157.174901789] [robot_state_publisher]: got segment left_inner_finger_pad
[robot_state_publisher-2] [INFO] [1760583157.174906265] [robot_state_publisher]: got segment left_inner_knuckle
[robot_state_publisher-2] [INFO] [1760583157.174910876] [robot_state_publisher]: got segment left_outer_finger
[robot_state_publisher-2] [INFO] [1760583157.174915174] [robot_state_publisher]: got segment left_outer_knuckle
[robot_state_publisher-2] [INFO] [1760583157.174919395] [robot_state_publisher]: got segment right_inner_finger
[robot_state_publisher-2] [INFO] [1760583157.174923630] [robot_state_publisher]: got segment right_inner_finger_pad
[robot_state_publisher-2] [INFO] [1760583157.174928026] [robot_state_publisher]: got segment right_inner_knuckle
[robot_state_publisher-2] [INFO] [1760583157.174932327] [robot_state_publisher]: got segment right_outer_finger
[robot_state_publisher-2] [INFO] [1760583157.174936479] [robot_state_publisher]: got segment right_outer_knuckle
[robot_state_publisher-2] [INFO] [1760583157.174940634] [robot_state_publisher]: got segment robotiq_140_base_link
[robot_state_publisher-2] [INFO] [1760583157.174945034] [robot_state_publisher]: got segment robotiq_base_link
[robot_state_publisher-2] [INFO] [1760583157.174949273] [robot_state_publisher]: got segment shoulder_link
[robot_state_publisher-2] [INFO] [1760583157.174953564] [robot_state_publisher]: got segment tool0
[robot_state_publisher-2] [INFO] [1760583157.174957753] [robot_state_publisher]: got segment upper_arm_link
[robot_state_publisher-2] [INFO] [1760583157.174962075] [robot_state_publisher]: got segment world
[robot_state_publisher-2] [INFO] [1760583157.174966319] [robot_state_publisher]: got segment wrist_1_link
[robot_state_publisher-2] [INFO] [1760583157.174970463] [robot_state_publisher]: got segment wrist_2_link
[robot_state_publisher-2] [INFO] [1760583157.174974575] [robot_state_publisher]: got segment wrist_3_link
[ros2_control_node-10] [INFO] [1760583157.183172015] [controller_manager]: Subscribing to '~/robot_description' topic for robot description file.
[ros2_control_node-10] [INFO] [1760583157.183559705] [controller_manager]: update rate is 100 Hz
[ros2_control_node-10] [INFO] [1760583157.183577733] [controller_manager]: Spawning controller_manager RT thread with scheduler priority: 50
[ros2_control_node-10] [WARN] [1760583157.183702506] [controller_manager]: No real-time kernel detected on this system. See [https://control.ros.org/master/doc/ros2_control/controller_manager/doc/userdoc.html] for details on how to enable realtime scheduling.
[rviz2-8] QStandardPaths: XDG_RUNTIME_DIR not set, defaulting to '/tmp/runtime-admin'
[move_group-7] [INFO] [1760583157.194826401] [moveit_rdf_loader.rdf_loader]: Loaded robot model in 0 seconds
[move_group-7] [INFO] [1760583157.194874622] [moveit_robot_model.robot_model]: Loading robot model 'robotiq_gripper'...
[move_group-7] [INFO] [1760583157.194881432] [moveit_robot_model.robot_model]: No root/virtual joint specified in SRDF. Assuming fixed joint
[ros2_control_node-10] [INFO] [1760583157.197091110] [controller_manager]: Received robot description file.
[ros2_control_node-10] [INFO] [1760583157.197419876] [resource_manager]: Loading hardware 'ur_ros2_control_sim' 
[ros2_control_node-10] [INFO] [1760583157.198336044] [resource_manager]: Initialize hardware 'ur_ros2_control_sim' 
[ros2_control_node-10] [INFO] [1760583157.201418194] [resource_manager]: Successful initialization of hardware 'ur_ros2_control_sim'
[ros2_control_node-10] [INFO] [1760583157.201488259] [resource_manager]: 'configure' hardware 'ur_ros2_control_sim' 
[ros2_control_node-10] [INFO] [1760583157.201495623] [resource_manager]: Successful 'configure' of hardware 'ur_ros2_control_sim'
[ros2_control_node-10] [INFO] [1760583157.201502809] [resource_manager]: 'activate' hardware 'ur_ros2_control_sim' 
[ros2_control_node-10] [INFO] [1760583157.201507273] [resource_manager]: Successful 'activate' of hardware 'ur_ros2_control_sim'
[move_group-7] [INFO] [1760583157.265561713] [moveit_ros.planning_scene_monitor.planning_scene_monitor]: Publishing maintained planning scene on 'monitored_planning_scene'
[move_group-7] [INFO] [1760583157.265746228] [moveit.ros_planning_interface.moveit_cpp]: Listening to 'joint_states' for joint states
[move_group-7] [INFO] [1760583157.267230677] [moveit_ros.current_state_monitor]: Listening to joint states on topic 'joint_states'
[move_group-7] [INFO] [1760583157.267857075] [moveit_ros.planning_scene_monitor.planning_scene_monitor]: Listening to '/attached_collision_object' for attached collision objects
[move_group-7] [INFO] [1760583157.267874635] [moveit_ros.planning_scene_monitor.planning_scene_monitor]: Starting planning scene monitor
[move_group-7] [INFO] [1760583157.269627810] [moveit_ros.planning_scene_monitor.planning_scene_monitor]: Listening to '/planning_scene'
[move_group-7] [INFO] [1760583157.269651122] [moveit_ros.planning_scene_monitor.planning_scene_monitor]: Starting world geometry update monitor for collision objects, attached objects, octomap updates.
[move_group-7] [INFO] [1760583157.270648450] [moveit_ros.planning_scene_monitor.planning_scene_monitor]: Listening to 'collision_object'
[move_group-7] [INFO] [1760583157.271391800] [moveit_ros.planning_scene_monitor.planning_scene_monitor]: Listening to 'planning_scene_world' for planning scene world geometry
[move_group-7] [WARN] [1760583157.271810647] [moveit.ros.occupancy_map_monitor.middleware_handle]: Resolution not specified for Octomap. Assuming resolution = 0.1 instead
[move_group-7] [ERROR] [1760583157.271827353] [moveit.ros.occupancy_map_monitor.middleware_handle]: No 3D sensor plugin(s) defined for octomap updates
[move_group-7] [INFO] [1760583157.289540416] [moveit.ros_planning_interface.moveit_cpp]: Loading planning pipeline 'isaac_ros_cumotion'
[move_group-7] [WARN] [1760583157.290993595] [pluginlib.ClassLoader]: given plugin name 'libisaac_ros_cumotion_moveit' should be 'isaac_ros_cumotion_moveit' for better portability
[move_group-7] [INFO] [1760583157.322392456] [moveit.ros_planning.planning_pipeline]: Using planning interface 'Generate minimum-jerk trajectories using NVIDIA Isaac ROS cuMotion'
[move_group-7] [INFO] [1760583157.325175483] [moveit_ros.fix_workspace_bounds]: Param 'isaac_ros_cumotion.default_workspace_bounds' was not set. Using default value: 10.000000
[move_group-7] [INFO] [1760583157.325210284] [moveit_ros.fix_start_state_bounds]: Param 'isaac_ros_cumotion.start_state_max_bounds_error' was set to 0.100000
[move_group-7] [INFO] [1760583157.325217085] [moveit_ros.fix_start_state_bounds]: Param 'isaac_ros_cumotion.start_state_max_dt' was not set. Using default value: 0.500000
[move_group-7] [INFO] [1760583157.325230046] [moveit_ros.fix_start_state_collision]: Param 'isaac_ros_cumotion.start_state_max_dt' was not set. Using default value: 0.500000
[move_group-7] [INFO] [1760583157.325236745] [moveit_ros.fix_start_state_collision]: Param 'isaac_ros_cumotion.jiggle_fraction' was not set. Using default value: 0.020000
[move_group-7] [INFO] [1760583157.325241904] [moveit_ros.fix_start_state_collision]: Param 'isaac_ros_cumotion.max_sampling_attempts' was not set. Using default value: 100
[move_group-7] [INFO] [1760583157.325267484] [moveit.ros_planning.planning_pipeline]: Using planning request adapter 'Fix Workspace Bounds'
[move_group-7] [INFO] [1760583157.325273576] [moveit.ros_planning.planning_pipeline]: Using planning request adapter 'Fix Start State Bounds'
[move_group-7] [INFO] [1760583157.325277406] [moveit.ros_planning.planning_pipeline]: Using planning request adapter 'Fix Start State In Collision'
[move_group-7] [INFO] [1760583157.325281734] [moveit.ros_planning.planning_pipeline]: Using planning request adapter 'Fix Start State Path Constraints'
[move_group-7] [INFO] [1760583157.327362561] [moveit.ros_planning_interface.moveit_cpp]: Loading planning pipeline 'ompl'
[move_group-7] [INFO] [1760583157.328475494] [moveit.ros_planning.planning_pipeline]: Multiple planning plugins available. You should specify the '~planning_plugin' parameter. Using 'chomp_interface/CHOMPPlanner' for now.
[move_group-7] [INFO] [1760583157.330105224] [moveit.ros_planning.planning_pipeline]: Using planning interface 'CHOMP'
[move_group-7] [WARN] [1760583157.331108325] [moveit.ros_planning_interface.moveit_cpp]: Skipping duplicate entry for planning pipeline 'ompl'.
[rviz2-8] [INFO] [1760583157.356601936] [rviz2]: Stereo is NOT SUPPORTED
[rviz2-8] [INFO] [1760583157.356755912] [rviz2]: OpenGl version: 4.5 (GLSL 4.5)
[move_group-7] [INFO] [1760583157.359995624] [moveit.plugins.moveit_simple_controller_manager]: Added FollowJointTrajectory controller for scaled_joint_trajectory_controller
[move_group-7] [INFO] [1760583157.361538806] [moveit.plugins.moveit_simple_controller_manager]: Added FollowJointTrajectory controller for joint_trajectory_controller
[move_group-7] [INFO] [1760583157.361712521] [moveit.plugins.moveit_simple_controller_manager]: Returned 2 controllers in list
[move_group-7] [INFO] [1760583157.361736975] [moveit.plugins.moveit_simple_controller_manager]: Returned 2 controllers in list
[move_group-7] [INFO] [1760583157.362083208] [moveit_ros.trajectory_execution_manager]: Trajectory execution is managing controllers
[move_group-7] [INFO] [1760583157.362100325] [move_group.move_group]: MoveGroup debug mode is ON
[rviz2-8] [INFO] [1760583157.376343878] [rviz2]: Stereo is NOT SUPPORTED
[move_group-7] [INFO] [1760583157.385543823] [move_group.move_group]: 
[move_group-7] 
[move_group-7] ********************************************************
[move_group-7] * MoveGroup using: 
[move_group-7] *     - ApplyPlanningSceneService
[move_group-7] *     - ClearOctomapService
[move_group-7] *     - ExecuteTaskSolution
[move_group-7] *     - CartesianPathService
[move_group-7] *     - ExecuteTrajectoryAction
[move_group-7] *     - GetPlanningSceneService
[move_group-7] *     - KinematicsService
[move_group-7] *     - MoveAction
[move_group-7] *     - MotionPlanService
[move_group-7] *     - QueryPlannersService
[move_group-7] *     - StateValidationService
[move_group-7] ********************************************************
[move_group-7] 
[move_group-7] [INFO] [1760583157.386355876] [moveit_move_group_capabilities_base.move_group_context]: MoveGroup context using planning plugin isaac_ros_cumotion_moveit/CumotionPlanner
[move_group-7] [INFO] [1760583157.386369852] [moveit_move_group_capabilities_base.move_group_context]: MoveGroup context initialization complete
[move_group-7] Loading 'move_group/ApplyPlanningSceneService'...
[move_group-7] Loading 'move_group/ClearOctomapService'...
[move_group-7] Loading 'move_group/ExecuteTaskSolutionCapability'...
[move_group-7] Loading 'move_group/MoveGroupCartesianPathService'...
[move_group-7] Loading 'move_group/MoveGroupExecuteTrajectoryAction'...
[move_group-7] Loading 'move_group/MoveGroupGetPlanningSceneService'...
[move_group-7] Loading 'move_group/MoveGroupKinematicsService'...
[move_group-7] Loading 'move_group/MoveGroupMoveAction'...
[move_group-7] Loading 'move_group/MoveGroupPlanService'...
[move_group-7] Loading 'move_group/MoveGroupQueryPlannersService'...
[move_group-7] Loading 'move_group/MoveGroupStateValidationService'...
[move_group-7] 
[move_group-7] You can start planning now!
[move_group-7] 
[rviz2-8] [INFO] [1760583157.446339510] [rviz2]: Connected on namespace: /end_effector_marker
[rviz2-8] [INFO] [1760583157.521756996] [rviz2]: Stereo is NOT SUPPORTED
[ros2_control_node-10] [INFO] [1760583157.562234239] [controller_manager]: Loading controller 'joint_state_broadcaster'
[joint_state_publisher-3] [INFO] [1760583157.582633872] [joint_state_publisher]: Waiting for robot_description to be published on the robot_description topic...
[ros2_control_node-10] [INFO] [1760583157.586671478] [controller_manager]: Loading controller 'scaled_joint_trajectory_controller'
[ros2_control_node-10] [WARN] [1760583157.593014715] [scaled_joint_trajectory_controller]: [Deprecated]: "allow_nonzero_velocity_at_trajectory_end" is set to true. The default behavior will change to false.
[spawner-13] [INFO] [1760583157.596211580] [spawner_joint_state_broadcaster]: Loaded joint_state_broadcaster
[component_container_mt-1] WARNING: Logging before InitGoogleLogging() is written to STDERR
[component_container_mt-1] I1016 02:52:37.609588 1003836 block_memory_pool_impl.h:65] Expanding the memory pool with 2048 blocks. Number of allocated blocks: 2048
[ros2_control_node-10] [INFO] [1760583157.638381542] [controller_manager]: Configuring controller 'joint_state_broadcaster'
[ros2_control_node-10] [INFO] [1760583157.638539608] [joint_state_broadcaster]: 'joints' or 'interfaces' parameter is empty. All available state interfaces will be published
[component_container_mt-1] I1016 02:52:37.640517 1003836 gpu_hash_interface_impl.cuh:83] Creating a GPUHashImpl with requested capacity of 4096 blocks. Real capacity: 4096
[component_container_mt-1] I1016 02:52:37.646073 1003836 layer_cake_impl.h:32] Adding Layer with type: N6nvblox15VoxelBlockLayerINS_9TsdfVoxelEEE, voxel_size: 0.01, and memory_type: kDevice to LayerCake.
[spawner-11] [INFO] [1760583157.651950519] [spawner_scaled_joint_trajectory_controller]: Loaded scaled_joint_trajectory_controller
[component_container_mt-1] I1016 02:52:37.654018 1003836 block_memory_pool_impl.h:65] Expanding the memory pool with 2048 blocks. Number of allocated blocks: 2048
[component_container_mt-1] I1016 02:52:37.667923 1003836 gpu_hash_interface_impl.cuh:83] Creating a GPUHashImpl with requested capacity of 4096 blocks. Real capacity: 4096
[component_container_mt-1] I1016 02:52:37.672168 1003836 layer_cake_impl.h:32] Adding Layer with type: N6nvblox15VoxelBlockLayerINS_10ColorVoxelEEE, voxel_size: 0.01, and memory_type: kDevice to LayerCake.
[component_container_mt-1] I1016 02:52:37.679105 1003836 block_memory_pool_impl.h:65] Expanding the memory pool with 2048 blocks. Number of allocated blocks: 2048
[component_container_mt-1] I1016 02:52:37.684180 1003836 gpu_hash_interface_impl.cuh:83] Creating a GPUHashImpl with requested capacity of 4096 blocks. Real capacity: 4096
[component_container_mt-1] I1016 02:52:37.684262 1003836 layer_cake_impl.h:32] Adding Layer with type: N6nvblox15VoxelBlockLayerINS_14FreespaceVoxelEEE, voxel_size: 0.01, and memory_type: kDevice to LayerCake.
[component_container_mt-1] I1016 02:52:37.690315 1003836 block_memory_pool_impl.h:65] Expanding the memory pool with 2048 blocks. Number of allocated blocks: 2048
[component_container_mt-1] I1016 02:52:37.690606 1003836 gpu_hash_interface_impl.cuh:83] Creating a GPUHashImpl with requested capacity of 4096 blocks. Real capacity: 4096
[component_container_mt-1] I1016 02:52:37.693437 1003836 layer_cake_impl.h:32] Adding Layer with type: N6nvblox15VoxelBlockLayerINS_14OccupancyVoxelEEE, voxel_size: 0.01, and memory_type: kDevice to LayerCake.
[component_container_mt-1] I1016 02:52:37.701021 1003836 block_memory_pool_impl.h:65] Expanding the memory pool with 2048 blocks. Number of allocated blocks: 2048
[ros2_control_node-10] [INFO] [1760583157.725394489] [controller_manager]: Configuring controller 'scaled_joint_trajectory_controller'
[ros2_control_node-10] [INFO] [1760583157.725637743] [scaled_joint_trajectory_controller]: No specific joint names are used for command interfaces. Using 'joints' parameter.
[ros2_control_node-10] [INFO] [1760583157.725667149] [scaled_joint_trajectory_controller]: Command interfaces are [position] and state interfaces are [position velocity].
[ros2_control_node-10] [INFO] [1760583157.725692241] [scaled_joint_trajectory_controller]: Using 'splines' interpolation method.
[ros2_control_node-10] [INFO] [1760583157.726085310] [scaled_joint_trajectory_controller]: Controller state will be published at 50.00 Hz.
[component_container_mt-1] I1016 02:52:37.726173 1003836 gpu_hash_interface_impl.cuh:83] Creating a GPUHashImpl with requested capacity of 4096 blocks. Real capacity: 4096
[component_container_mt-1] I1016 02:52:37.727583 1003836 layer_cake_impl.h:32] Adding Layer with type: N6nvblox15VoxelBlockLayerINS_9EsdfVoxelEEE, voxel_size: 0.01, and memory_type: kDevice to LayerCake.
[ros2_control_node-10] [INFO] [1760583157.728146866] [scaled_joint_trajectory_controller]: Action status changes will be monitored at 20.00 Hz.
[component_container_mt-1] I1016 02:52:37.728235 1003836 block_memory_pool_impl.h:65] Expanding the memory pool with 2048 blocks. Number of allocated blocks: 2048
[component_container_mt-1] I1016 02:52:37.740311 1003836 gpu_hash_interface_impl.cuh:83] Creating a GPUHashImpl with requested capacity of 4096 blocks. Real capacity: 4096
[component_container_mt-1] I1016 02:52:37.741556 1003836 layer_cake_impl.h:32] Adding Layer with type: N6nvblox10BlockLayerINS_9MeshBlockEEE, voxel_size: 0.01, and memory_type: kDevice to LayerCake.
[component_container_mt-1] I1016 02:52:37.741575 1003836 layer_cake_streamer_impl.h:79] Adding Streamer with type: N6nvblox25LayerStreamerOldestBlocksINS_15VoxelBlockLayerINS_9TsdfVoxelEEEEE to LayerCake.
[component_container_mt-1] I1016 02:52:37.741577 1003836 layer_cake_streamer_impl.h:79] Adding Streamer with type: N6nvblox25LayerStreamerOldestBlocksINS_15VoxelBlockLayerINS_10ColorVoxelEEEEE to LayerCake.
[component_container_mt-1] I1016 02:52:37.741580 1003836 layer_cake_streamer_impl.h:79] Adding Streamer with type: N6nvblox25LayerStreamerOldestBlocksINS_15VoxelBlockLayerINS_14FreespaceVoxelEEEEE to LayerCake.
[component_container_mt-1] I1016 02:52:37.741583 1003836 layer_cake_streamer_impl.h:79] Adding Streamer with type: N6nvblox25LayerStreamerOldestBlocksINS_15VoxelBlockLayerINS_14OccupancyVoxelEEEEE to LayerCake.
[component_container_mt-1] I1016 02:52:37.741585 1003836 layer_cake_streamer_impl.h:79] Adding Streamer with type: N6nvblox25LayerStreamerOldestBlocksINS_15VoxelBlockLayerINS_9EsdfVoxelEEEEE to LayerCake.
[component_container_mt-1] I1016 02:52:37.741598 1003836 layer_cake_streamer_impl.h:79] Adding Streamer with type: N6nvblox25LayerStreamerOldestBlocksINS_10BlockLayerINS_9MeshBlockEEEEE to LayerCake.
[component_container_mt-1] I1016 02:52:37.749877 1003836 block_memory_pool_impl.h:65] Expanding the memory pool with 2048 blocks. Number of allocated blocks: 2048
[component_container_mt-1] I1016 02:52:37.763895 1003836 gpu_hash_interface_impl.cuh:83] Creating a GPUHashImpl with requested capacity of 4096 blocks. Real capacity: 4096
[component_container_mt-1] I1016 02:52:37.765136 1003836 layer_cake_impl.h:32] Adding Layer with type: N6nvblox15VoxelBlockLayerINS_9TsdfVoxelEEE, voxel_size: 0.01, and memory_type: kDevice to LayerCake.
[component_container_mt-1] I1016 02:52:37.772014 1003836 block_memory_pool_impl.h:65] Expanding the memory pool with 2048 blocks. Number of allocated blocks: 2048
[component_container_mt-1] I1016 02:52:37.772502 1003836 gpu_hash_interface_impl.cuh:83] Creating a GPUHashImpl with requested capacity of 4096 blocks. Real capacity: 4096
[component_container_mt-1] I1016 02:52:37.772548 1003836 layer_cake_impl.h:32] Adding Layer with type: N6nvblox15VoxelBlockLayerINS_10ColorVoxelEEE, voxel_size: 0.01, and memory_type: kDevice to LayerCake.
[move_group-7] [INFO] [1760583157.775502893] [moveit_move_group_default_capabilities.move_action_capability]: Received request
[move_group-7] [INFO] [1760583157.775858383] [moveit_move_group_default_capabilities.move_action_capability]: executing..
[component_container_mt-1] I1016 02:52:37.780476 1003836 block_memory_pool_impl.h:65] Expanding the memory pool with 2048 blocks. Number of allocated blocks: 2048
[pose_to_pose_node-14] [INFO] [1760583157.798196072] [pose_to_pose_node]: Planning Goal request accepted!
[component_container_mt-1] I1016 02:52:37.802702 1003836 gpu_hash_interface_impl.cuh:83] Creating a GPUHashImpl with requested capacity of 4096 blocks. Real capacity: 4096
[component_container_mt-1] I1016 02:52:37.804117 1003836 layer_cake_impl.h:32] Adding Layer with type: N6nvblox15VoxelBlockLayerINS_14FreespaceVoxelEEE, voxel_size: 0.01, and memory_type: kDevice to LayerCake.
[component_container_mt-1] I1016 02:52:37.809901 1003836 block_memory_pool_impl.h:65] Expanding the memory pool with 2048 blocks. Number of allocated blocks: 2048
[component_container_mt-1] I1016 02:52:37.824155 1003836 gpu_hash_interface_impl.cuh:83] Creating a GPUHashImpl with requested capacity of 4096 blocks. Real capacity: 4096
[component_container_mt-1] I1016 02:52:37.825392 1003836 layer_cake_impl.h:32] Adding Layer with type: N6nvblox15VoxelBlockLayerINS_14OccupancyVoxelEEE, voxel_size: 0.01, and memory_type: kDevice to LayerCake.
[component_container_mt-1] I1016 02:52:37.832242 1003836 block_memory_pool_impl.h:65] Expanding the memory pool with 2048 blocks. Number of allocated blocks: 2048
[component_container_mt-1] I1016 02:52:37.845543 1003836 gpu_hash_interface_impl.cuh:83] Creating a GPUHashImpl with requested capacity of 4096 blocks. Real capacity: 4096
[component_container_mt-1] I1016 02:52:37.847054 1003836 layer_cake_impl.h:32] Adding Layer with type: N6nvblox15VoxelBlockLayerINS_9EsdfVoxelEEE, voxel_size: 0.01, and memory_type: kDevice to LayerCake.
[component_container_mt-1] I1016 02:52:37.847498 1003836 block_memory_pool_impl.h:65] Expanding the memory pool with 2048 blocks. Number of allocated blocks: 2048
[component_container_mt-1] I1016 02:52:37.852486 1003836 gpu_hash_interface_impl.cuh:83] Creating a GPUHashImpl with requested capacity of 4096 blocks. Real capacity: 4096
[component_container_mt-1] I1016 02:52:37.852571 1003836 layer_cake_impl.h:32] Adding Layer with type: N6nvblox10BlockLayerINS_9MeshBlockEEE, voxel_size: 0.01, and memory_type: kDevice to LayerCake.
[component_container_mt-1] I1016 02:52:37.852586 1003836 layer_cake_streamer_impl.h:79] Adding Streamer with type: N6nvblox25LayerStreamerOldestBlocksINS_15VoxelBlockLayerINS_9TsdfVoxelEEEEE to LayerCake.
[component_container_mt-1] I1016 02:52:37.852595 1003836 layer_cake_streamer_impl.h:79] Adding Streamer with type: N6nvblox25LayerStreamerOldestBlocksINS_15VoxelBlockLayerINS_10ColorVoxelEEEEE to LayerCake.
[component_container_mt-1] I1016 02:52:37.852598 1003836 layer_cake_streamer_impl.h:79] Adding Streamer with type: N6nvblox25LayerStreamerOldestBlocksINS_15VoxelBlockLayerINS_14FreespaceVoxelEEEEE to LayerCake.
[component_container_mt-1] I1016 02:52:37.852602 1003836 layer_cake_streamer_impl.h:79] Adding Streamer with type: N6nvblox25LayerStreamerOldestBlocksINS_15VoxelBlockLayerINS_14OccupancyVoxelEEEEE to LayerCake.
[component_container_mt-1] I1016 02:52:37.852604 1003836 layer_cake_streamer_impl.h:79] Adding Streamer with type: N6nvblox25LayerStreamerOldestBlocksINS_15VoxelBlockLayerINS_9EsdfVoxelEEEEE to LayerCake.
[component_container_mt-1] I1016 02:52:37.852608 1003836 layer_cake_streamer_impl.h:79] Adding Streamer with type: N6nvblox25LayerStreamerOldestBlocksINS_10BlockLayerINS_9MeshBlockEEEEE to LayerCake.
[INFO] [launch_ros.actions.load_composable_nodes]: Loaded node '/nvblox_node' in container 'manipulator_container'
[move_group-7] [INFO] [1760583157.976960159] [moveit_move_group_default_capabilities.move_action_capability]: Combined planning and execution request received for MoveGroup action. Forwarding to planning and execution pipeline.
[move_group-7] [WARN] [1760583157.977051115] [moveit_move_group_capabilities_base.move_group_capability]: Execution of motions should always start at the robot's current state. Ignoring the state supplied as start state in the motion planning request
[move_group-7] [WARN] [1760583157.977071367] [moveit_move_group_capabilities_base.move_group_capability]: Execution of motions should always start at the robot's current state. Ignoring the state supplied as difference in the planning scene diff
[move_group-7] [INFO] [1760583157.977102671] [moveit_ros.plan_execution]: Planning attempt 1 of at most 1
[move_group-7] [INFO] [1760583157.977128161] [moveit_move_group_capabilities_base.move_group_capability]: Using planning pipeline 'isaac_ros_cumotion'
[spawner-13] [INFO] [1760583157.977490389] [spawner_joint_state_broadcaster]: Configured and activated joint_state_broadcaster
[move_group-7] [INFO] [1760583157.978624724] [move_group]: Planning trajectory
[spawner-11] [INFO] [1760583158.117338996] [spawner_scaled_joint_trajectory_controller]: Configured and activated scaled_joint_trajectory_controller
[INFO] [spawner-13]: process has finished cleanly [pid 1003739]
[INFO] [spawner-11]: process has finished cleanly [pid 1003733]
[robot_segmenter_node-5] kinematics_fused_cu not found, JIT compiling...
[cumotion_goal_set_planner_node-4] kinematics_fused_cu not found, JIT compiling...
[attach_object_server_node-6] kinematics_fused_cu not found, JIT compiling...
[robot_segmenter_node-5] /usr/local/lib/python3.10/dist-packages/torch/utils/cpp_extension.py:2356: UserWarning: TORCH_CUDA_ARCH_LIST is not set, all archs for visible cards are included for compilation. 
[robot_segmenter_node-5] If this is not desired, please set os.environ['TORCH_CUDA_ARCH_LIST'].
[robot_segmenter_node-5]   warnings.warn(
[cumotion_goal_set_planner_node-4] /usr/local/lib/python3.10/dist-packages/torch/utils/cpp_extension.py:2356: UserWarning: TORCH_CUDA_ARCH_LIST is not set, all archs for visible cards are included for compilation. 
[cumotion_goal_set_planner_node-4] If this is not desired, please set os.environ['TORCH_CUDA_ARCH_LIST'].
[cumotion_goal_set_planner_node-4]   warnings.warn(
[attach_object_server_node-6] /usr/local/lib/python3.10/dist-packages/torch/utils/cpp_extension.py:2356: UserWarning: TORCH_CUDA_ARCH_LIST is not set, all archs for visible cards are included for compilation. 
[attach_object_server_node-6] If this is not desired, please set os.environ['TORCH_CUDA_ARCH_LIST'].
[attach_object_server_node-6]   warnings.warn(
[robot_segmenter_node-5] geom_cu binary not found, jit compiling...
[robot_segmenter_node-5] /usr/local/lib/python3.10/dist-packages/torch/utils/cpp_extension.py:2356: UserWarning: TORCH_CUDA_ARCH_LIST is not set, all archs for visible cards are included for compilation. 
[robot_segmenter_node-5] If this is not desired, please set os.environ['TORCH_CUDA_ARCH_LIST'].
[robot_segmenter_node-5]   warnings.warn(
[cumotion_goal_set_planner_node-4] geom_cu binary not found, jit compiling...
[cumotion_goal_set_planner_node-4] /usr/local/lib/python3.10/dist-packages/torch/utils/cpp_extension.py:2356: UserWarning: TORCH_CUDA_ARCH_LIST is not set, all archs for visible cards are included for compilation. 
[cumotion_goal_set_planner_node-4] If this is not desired, please set os.environ['TORCH_CUDA_ARCH_LIST'].
[cumotion_goal_set_planner_node-4]   warnings.warn(
[robot_segmenter_node-5] tensor_step_cu not found, jit compiling...
[robot_segmenter_node-5] /usr/local/lib/python3.10/dist-packages/torch/utils/cpp_extension.py:2356: UserWarning: TORCH_CUDA_ARCH_LIST is not set, all archs for visible cards are included for compilation. 
[robot_segmenter_node-5] If this is not desired, please set os.environ['TORCH_CUDA_ARCH_LIST'].
[robot_segmenter_node-5]   warnings.warn(
[cumotion_goal_set_planner_node-4] tensor_step_cu not found, jit compiling...
[cumotion_goal_set_planner_node-4] /usr/local/lib/python3.10/dist-packages/torch/utils/cpp_extension.py:2356: UserWarning: TORCH_CUDA_ARCH_LIST is not set, all archs for visible cards are included for compilation. 
[cumotion_goal_set_planner_node-4] If this is not desired, please set os.environ['TORCH_CUDA_ARCH_LIST'].
[cumotion_goal_set_planner_node-4]   warnings.warn(
[robot_segmenter_node-5] lbfgs_step_cu not found, JIT compiling...
[robot_segmenter_node-5] /usr/local/lib/python3.10/dist-packages/torch/utils/cpp_extension.py:2356: UserWarning: TORCH_CUDA_ARCH_LIST is not set, all archs for visible cards are included for compilation. 
[robot_segmenter_node-5] If this is not desired, please set os.environ['TORCH_CUDA_ARCH_LIST'].
[robot_segmenter_node-5]   warnings.warn(
[robot_segmenter_node-5] line_search_cu not found, JIT compiling...
[robot_segmenter_node-5] /usr/local/lib/python3.10/dist-packages/torch/utils/cpp_extension.py:2356: UserWarning: TORCH_CUDA_ARCH_LIST is not set, all archs for visible cards are included for compilation. 
[robot_segmenter_node-5] If this is not desired, please set os.environ['TORCH_CUDA_ARCH_LIST'].
[robot_segmenter_node-5]   warnings.warn(
[cumotion_goal_set_planner_node-4] lbfgs_step_cu not found, JIT compiling...
[cumotion_goal_set_planner_node-4] /usr/local/lib/python3.10/dist-packages/torch/utils/cpp_extension.py:2356: UserWarning: TORCH_CUDA_ARCH_LIST is not set, all archs for visible cards are included for compilation. 
[cumotion_goal_set_planner_node-4] If this is not desired, please set os.environ['TORCH_CUDA_ARCH_LIST'].
[cumotion_goal_set_planner_node-4]   warnings.warn(
[cumotion_goal_set_planner_node-4] line_search_cu not found, JIT compiling...
[cumotion_goal_set_planner_node-4] /usr/local/lib/python3.10/dist-packages/torch/utils/cpp_extension.py:2356: UserWarning: TORCH_CUDA_ARCH_LIST is not set, all archs for visible cards are included for compilation. 
[cumotion_goal_set_planner_node-4] If this is not desired, please set os.environ['TORCH_CUDA_ARCH_LIST'].
[cumotion_goal_set_planner_node-4]   warnings.warn(
[attach_object_server_node-6] geom_cu binary not found, jit compiling...
[attach_object_server_node-6] /usr/local/lib/python3.10/dist-packages/torch/utils/cpp_extension.py:2356: UserWarning: TORCH_CUDA_ARCH_LIST is not set, all archs for visible cards are included for compilation. 
[attach_object_server_node-6] If this is not desired, please set os.environ['TORCH_CUDA_ARCH_LIST'].
[attach_object_server_node-6]   warnings.warn(
[attach_object_server_node-6] tensor_step_cu not found, jit compiling...
[attach_object_server_node-6] /usr/local/lib/python3.10/dist-packages/torch/utils/cpp_extension.py:2356: UserWarning: TORCH_CUDA_ARCH_LIST is not set, all archs for visible cards are included for compilation. 
[attach_object_server_node-6] If this is not desired, please set os.environ['TORCH_CUDA_ARCH_LIST'].
[attach_object_server_node-6]   warnings.warn(
[cumotion_goal_set_planner_node-4] [INFO] [1760583161.854844911] [cumotion_planner]: Loading grid center and dims from workspace file: /opt/ros/humble/share/isaac_manipulator_bringup/config/nvblox/workspace_bounds/sim_test_bench.yaml.
[cumotion_goal_set_planner_node-4] [INFO] [1760583161.857167501] [cumotion_planner]: Loaded grid dims: [2.0, 2.0, 1.0], voxel size: 0.01
[attach_object_server_node-6] lbfgs_step_cu not found, JIT compiling...
[attach_object_server_node-6] /usr/local/lib/python3.10/dist-packages/torch/utils/cpp_extension.py:2356: UserWarning: TORCH_CUDA_ARCH_LIST is not set, all archs for visible cards are included for compilation. 
[attach_object_server_node-6] If this is not desired, please set os.environ['TORCH_CUDA_ARCH_LIST'].
[attach_object_server_node-6]   warnings.warn(
[attach_object_server_node-6] line_search_cu not found, JIT compiling...
[attach_object_server_node-6] /usr/local/lib/python3.10/dist-packages/torch/utils/cpp_extension.py:2356: UserWarning: TORCH_CUDA_ARCH_LIST is not set, all archs for visible cards are included for compilation. 
[attach_object_server_node-6] If this is not desired, please set os.environ['TORCH_CUDA_ARCH_LIST'].
[attach_object_server_node-6]   warnings.warn(
[robot_segmenter_node-5] [INFO] [1760583162.697008791] [robot_segmenter]: Node initialized with 1 cameras
[robot_segmenter_node-5] [INFO] [1760583162.697317580] [robot_segmenter]: Starting CumotionRobotSegmenter node
[robot_segmenter_node-5] [INFO] [1760583162.716935539] [robot_segmenter]: Reading TF from cameras
[attach_object_server_node-6] [INFO] [1760583163.586612281] [object_attachment]: Node initialized with 1 cameras
[cumotion_goal_set_planner_node-4] [INFO] [1760583164.817094628] [cumotion_planner]: warming up cuMotion, wait until ready
[robot_segmenter_node-5] [INFO] [1760583167.800131712] [robot_segmenter]: Reading TF from cameras
[robot_segmenter_node-5] [INFO] [1760583167.801938268] [robot_segmenter]: Received TF from cameras to robot
[robot_segmenter_node-5] [INFO] [1760583167.832892812] [robot_segmenter]: Updated Projection Matrices
[component_container_mt-1] I1016 02:52:48.927479 1003814 block_memory_pool_impl.h:65] Expanding the memory pool with 4096 blocks. Number of allocated blocks: 6144
[component_container_mt-1] I1016 02:52:49.004042 1003814 block_memory_pool_impl.h:65] Expanding the memory pool with 12288 blocks. Number of allocated blocks: 18432
[component_container_mt-1] I1016 02:52:49.015676 1003814 gpu_layer_view_impl.cuh:100] Resizing GPU hash capacity from 4096 to 12710 in order to accomodate space for 6355 new elements.
[component_container_mt-1] I1016 02:52:49.031841 1003814 gpu_hash_interface_impl.cuh:83] Creating a GPUHashImpl with requested capacity of 12710 blocks. Real capacity: 16384
[component_container_mt-1] I1016 02:52:49.036064 1003814 mapper.cpp:304] Allocating space for last depth image
[cumotion_goal_set_planner_node-4] [INFO] [1760583174.566968289] [cumotion_planner]: cuMotion is ready for planning queries!
[move_group-7] [INFO] [1760583174.599090804] [move_group]: Sending goal
[move_group-7] [INFO] [1760583174.606098558] [move_group]: Goal accepted by server, waiting for result
[cumotion_goal_set_planner_node-4] [INFO] [1760583174.609358122] [cumotion_planner]: Executing goal...
[cumotion_goal_set_planner_node-4] [INFO] [1760583174.609830472] [cumotion_planner]: Planning with time_dilation_factor: 1.0
[cumotion_goal_set_planner_node-4] [INFO] [1760583174.610372028] [cumotion_planner]: Calling ESDF service
[cumotion_goal_set_planner_node-4] [INFO] [1760583174.610823403] [cumotion_planner]: ESDF  req = geometry_msgs.msg.Point(x=-1.5, y=-1.0, z=-0.2), geometry_msgs.msg.Vector3(x=2.0, y=2.0, z=1.0)
[component_container_mt-1] I1016 02:52:54.670656 1003854 block_memory_pool_impl.h:65] Expanding the memory pool with 4096 blocks. Number of allocated blocks: 6144
[component_container_mt-1] I1016 02:52:54.742401 1003854 block_memory_pool_impl.h:65] Expanding the memory pool with 12288 blocks. Number of allocated blocks: 18432
[component_container_mt-1] I1016 02:52:54.744249 1003854 gpu_layer_view_impl.cuh:100] Resizing GPU hash capacity from 4096 to 12710 in order to accomodate space for 6355 new elements.
[component_container_mt-1] I1016 02:52:54.752557 1003854 gpu_hash_interface_impl.cuh:83] Creating a GPUHashImpl with requested capacity of 12710 blocks. Real capacity: 16384
[cumotion_goal_set_planner_node-4] [INFO] [1760583175.032405661] [cumotion_planner]: Updated ESDF grid
[cumotion_goal_set_planner_node-4] [INFO] [1760583175.036744095] [cumotion_planner]: PlanRequest start state was empty, reading current joint state
[cumotion_goal_set_planner_node-4] [INFO] [1760583175.039520041] [cumotion_planner]: Using goal from Pose
[cumotion_goal_set_planner_node-4] [INFO] [1760583175.450307727] [cumotion_planner]: returned planning result (query, success, failure_status): 0 True None
[move_group-7] [INFO] [1760583175.454721262] [move_group]: Received result
[move_group-7] [INFO] [1760583175.454741723] [move_group]: Checking results
[move_group-7] [INFO] [1760583175.454934483] [move_group]: Success
[move_group-7] [INFO] [1760583175.460036684] [move_group]: Received trajectory result
[move_group-7] [INFO] [1760583175.460064054] [move_group]: Trajectory success!
[move_group-7] [INFO] [1760583175.462239041] [moveit.plugins.moveit_simple_controller_manager]: Returned 2 controllers in list
[move_group-7] [INFO] [1760583175.462276888] [moveit.plugins.moveit_simple_controller_manager]: Returned 2 controllers in list
[move_group-7] [INFO] [1760583175.462548441] [moveit_ros.trajectory_execution_manager]: Starting trajectory execution ...
[move_group-7] [INFO] [1760583175.462682243] [moveit.plugins.moveit_simple_controller_manager]: Returned 2 controllers in list
[move_group-7] [INFO] [1760583175.462743313] [moveit.plugins.moveit_simple_controller_manager]: Returned 2 controllers in list
[move_group-7] [INFO] [1760583175.463065539] [moveit.simple_controller_manager.follow_joint_trajectory_controller_handle]: sending trajectory to scaled_joint_trajectory_controller
[ros2_control_node-10] [INFO] [1760583175.463927284] [scaled_joint_trajectory_controller]: Received new action goal
[ros2_control_node-10] [INFO] [1760583175.464071544] [scaled_joint_trajectory_controller]: Accepted new action goal
[move_group-7] [INFO] [1760583175.464385572] [moveit.simple_controller_manager.follow_joint_trajectory_controller_handle]: scaled_joint_trajectory_controller started execution
[move_group-7] [INFO] [1760583175.464459619] [moveit.simple_controller_manager.follow_joint_trajectory_controller_handle]: Goal request accepted!
[ros2_control_node-10] [INFO] [1760583180.421836006] [scaled_joint_trajectory_controller]: Goal reached, success!
[move_group-7] [INFO] [1760583180.465068910] [moveit.simple_controller_manager.follow_joint_trajectory_controller_handle]: Controller 'scaled_joint_trajectory_controller' successfully finished
[move_group-7] [INFO] [1760583180.465188159] [moveit_ros.trajectory_execution_manager]: Not waiting for trajectory completion
[move_group-7] [INFO] [1760583180.465213609] [moveit_ros.trajectory_execution_manager]: Completed trajectory execution with status SUCCEEDED ...
[move_group-7] [INFO] [1760583180.472824770] [moveit_move_group_default_capabilities.move_action_capability]: Solution was found and executed.
[pose_to_pose_node-14] [INFO] [1760583180.484559462] [pose_to_pose_node]: Motion planning succeeded.
[move_group-7] [INFO] [1760583180.501058995] [moveit_move_group_default_capabilities.move_action_capability]: Received request
[move_group-7] [INFO] [1760583180.501359400] [moveit_move_group_default_capabilities.move_action_capability]: executing..
[move_group-7] [INFO] [1760583180.501529848] [moveit_move_group_default_capabilities.move_action_capability]: Combined planning and execution request received for MoveGroup action. Forwarding to planning and execution pipeline.
[move_group-7] [WARN] [1760583180.501564011] [moveit_move_group_capabilities_base.move_group_capability]: Execution of motions should always start at the robot's current state. Ignoring the state supplied as start state in the motion planning request
[move_group-7] [WARN] [1760583180.501579248] [moveit_move_group_capabilities_base.move_group_capability]: Execution of motions should always start at the robot's current state. Ignoring the state supplied as difference in the planning scene diff
[move_group-7] [INFO] [1760583180.501593190] [moveit_ros.plan_execution]: Planning attempt 1 of at most 1
[move_group-7] [INFO] [1760583180.501606719] [moveit_move_group_capabilities_base.move_group_capability]: Using planning pipeline 'isaac_ros_cumotion'
[move_group-7] [INFO] [1760583180.502444289] [move_group]: Planning trajectory
[move_group-7] [INFO] [1760583180.502938986] [move_group]: Sending goal
[pose_to_pose_node-14] [INFO] [1760583180.505263354] [pose_to_pose_node]: Planning Goal request accepted!

``` 

另外: Isaac Sim 4.2 加载场景进行Play时, 有报错如下, 但不影响实验: 

```
2025-10-16 02:52:30 [833,305ms] [Error] [omni.graph.core.plugin] /World/ur10e_robotiq2f_140_ROS/ur10e_robotiq2f_140/Robotiq_2F_140_config/ActionGraph/get_array_index: [/World/ur10e_robotiq2f_140_ROS/ur10e_robotiq2f_140/Robotiq_2F_140_config/ActionGraph] inputs:index 0 is out of range for inputs:array of size 0
``` 

# 测试DOPE

生成onnx:

```
从https://drive.google.com/drive/folders/1DfoA3m_Bm0fW8tOWXGVxi4ETlLEAgmcg 下载soup-60.pth
放到${ISAAC_ROS_WS}/isaac_ros_assets/
 
mkdir -p ${ISAAC_ROS_WS}/isaac_ros_assets/models/dope/
 
ros2 run isaac_ros_dope dope_converter.py --format onnx \
   --input ${ISAAC_ROS_WS}/isaac_ros_assets/soup_60.pth \
   --output ${ISAAC_ROS_WS}/isaac_ros_assets/models/dope/soup_can.onnx \
   --row 1200 --col 1920
``` 

在桌面容器中: 

```
ros2 launch isaac_manipulator_pick_and_place isaac_sim_workflows.launch.py \
   workflow_type:=object_following pose_estimation_type:=dope
``` 

碰到问题, 没有作用, 分析: 

````
### 数据流图
```
Isaac Sim (RGB/深度) 
    ↓
cuMotion机器人分割节点 (/robot_segmenter)
    ↓
nvblox (3D重建)
    ↓
ESDF (距离场)
    ↓
cuMotion规划器 (/cumotion_planner)
    ↓
机械臂运动
```

---

## 各阶段检查状态

### 阶段1: Isaac Sim数据输出
**状态：✅ 正常**

| 话题 | 检查命令 | 结果 |
| --- | --- | --- |
| RGB图像 | `ros2 topic echo /front_stereo_camera/left/image_raw --once` | ✅ 有数据输出 |
| 深度图像 | `ros2 topic echo /front_stereo_camera/depth/ground_truth --once` | ✅ 有数据输出 |
| RGB相机信息 | `ros2 topic echo /front_stereo_camera/left/camera_info --once` | ✅ 有数据输出 |
| 深度相机信息 | `ros2 topic echo /front_stereo_camera/depth/camera_info --once` | ✅ 有数据输出 |
| 关节状态 | `ros2 topic echo /isaac_joint_states --once` | ✅ 有数据输出 |

**频率检查：**
- RGB图像: 9.904 Hz ✅
- 深度图像: 12.021 Hz ✅

---

### 阶段2: cuMotion机器人分割节点
**状态：❌ 节点启动但无输出**

| 检查项 | 检查命令 | 结果 |  |
| --- | --- | --- | --- |
| 节点存在 | `ros2 node list \ | grep robot_segmenter` | ✅ `/robot_segmenter` 存在 |
| 节点信息 | `ros2 node info /robot_segmenter` | ✅ 正常订阅和发布话题 |  |
| 输入-深度数据 | 订阅 `/front_stereo_camera/depth/ground_truth` | ✅ 正在订阅 |  |
| 输入-关节状态 | 订阅 `/isaac_joint_states` | ✅ 正在订阅 |  |
| 输入-TF | 订阅 `/tf` 和 `/tf_static` | ✅ 正在订阅 |  |
| **输出-处理后深度** | `ros2 topic echo /cumotion/camera_1/world_depth --once` | ❌ **卡住无输出** |  |
| **输出-机器人掩码** | `ros2 topic echo /cumotion/camera_1/robot_mask --once` | ❌ **卡住无输出** |  |
| 输出-机器人球体 | 发布 `/cumotion/robot_segmenter/robot_spheres` | ✅ 话题存在 |  |

**话题列表确认：**
- `/cumotion/camera_1/robot_mask` ✅ 存在
- `/cumotion/camera_1/world_depth` ✅ 存在
- `/cumotion/robot_segmenter/robot_spheres` ✅ 存在

**问题：节点声称在发布数据，但实际无数据输出**

---

### 阶段3: nvblox节点
**状态：❌ 节点运行但无有效数据**

| 检查项 | 检查命令 | 结果 |  |
| --- | --- | --- | --- |
| 节点存在 | `ros2 node list \ | grep nvblox` | ✅ `/nvblox_node` 存在 |
| 节点信息 | `ros2 node info /nvblox_node` | ✅ 正常订阅和发布话题 |  |
| 输入-RGB数据 | 订阅 `/front_stereo_camera/left/image_raw` | ✅ 正在订阅 |  |
| **输入-深度数据** | 订阅 `/cumotion/camera_1/world_depth` | ❌ **无数据（上游问题）** |  |
| 输入-相机信息 | 订阅相机info话题 | ✅ 正在订阅 |  |
| **输出-TSDF** | `ros2 topic echo /nvblox_node/tsdf_layer --once` | ❌ **卡住无输出** |  |
| **输出-ESDF点云** | `ros2 topic echo /nvblox_node/static_esdf_pointcloud --once` | ❌ **卡住无输出** |  |
| 输出-工作空间边界 | `ros2 topic echo /nvblox_node/workspace_bounds --once` | ✅ 有数据输出 |  |
| ESDF服务 | `/nvblox_node/get_esdf_and_gradient` | ✅ 服务存在 |  |

**工作空间边界配置：**
- X: -1.5 到 0.5
- Y: -1.0 到 1.0  
- Z: -0.2 到 0.8

**问题：nvblox无法获取深度数据，导致无法构建3D环境地图**

---

### 阶段4: cuMotion规划器
**状态：❌ 运行正常但ESDF数据为空**

| 检查项 | 检查命令 | 结果 |  |
| --- | --- | --- | --- |
| 节点存在 | `ros2 node list \ | grep cumotion_planner` | ✅ `/cumotion_planner` 存在 |
| 规划流程 | 从日志查看 | ✅ 规划流程正常启动 |  |
| ESDF服务调用 | 从日志查看 | ✅ 正常调用服务 |  |
| **ESDF数据** | 从日志查看 | ❌ **ESDF data is empty** |  |
| 规划结果 | 从日志查看 | ❌ **World update failed, No trajectory** |  |

**日志流程：**
```
Planning trajectory 
→ Sending goal 
→ Goal accepted 
→ Executing goal 
→ Calling ESDF service 
→ ❌ ESDF data is empty 
→ ❌ World update failed 
→ ❌ No trajectory 
→ ❌ Planning failed
```

---

### 阶段5: 目标设置节点
**状态：✅ 运行正常但目标变化太小**

| 检查项 | 检查命令 | 结果 |  |
| --- | --- | --- | --- |
| 节点存在 | `ros2 node list \ | grep goal_init` | ✅ `/goal_initializer_node` 存在 |
| 目标检测 | 从日志查看 | ✅ 检测到目标位置变化 |  |
| 阈值检查 | 从日志查看 | ⚠️ 变化太小（< 0.1m），不触发新规划 |  |

**日志信息：**
```
New goal position is within 0.1 meters at 0.002001090312389788
→ 目标变化约2mm，低于阈值100mm
```
```` 

认为 robot_segmenter的运作有问题. 

检查后, 发现QoS不匹配: 

```
admin@dell-136:/workspaces/isaac_ros-dev$ ros2 topic info /front_stereo_camera/depth/ground_truth -v
Type: sensor_msgs/msg/Image

Publisher count: 1

Node name: _Render_PostProcess_SDGPipeline_Replicator_01_NodeWriterWriter_01
Node namespace: /front_stereo_camera/depth
Topic type: sensor_msgs/msg/Image
Endpoint type: PUBLISHER
GID: 01.10.bc.87.c9.9f.0f.a3.6f.ec.d4.ca.00.00.0a.03.00.00.00.00.00.00.00.00
QoS profile:
  Reliability: RELIABLE
  History (Depth): KEEP_LAST (10)
  Durability: VOLATILE
  Lifespan: Infinite
  Deadline: Infinite
  Liveliness: AUTOMATIC
  Liveliness lease duration: Infinite

Subscription count: 1

Node name: robot_segmenter
Node namespace: /
Topic type: sensor_msgs/msg/Image
Endpoint type: SUBSCRIPTION
GID: 01.10.d2.ad.90.c2.6d.5d.dd.4a.c3.c5.00.00.15.04.00.00.00.00.00.00.00.00
QoS profile:
  Reliability: BEST_EFFORT
  History (Depth): KEEP_LAST (5)
  Durability: VOLATILE
  Lifespan: Infinite
  Deadline: Infinite
  Liveliness: AUTOMATIC
  Liveliness lease duration: Infinite

admin@dell-136:/workspaces/isaac_ros-dev$ ros2 topic info /isaac_joint_states -v
Type: sensor_msgs/msg/JointState

Publisher count: 1

Node name: _World_ur10e_robotiq2f_140_ROS_ur_action_graph_ros2_publish_joint_state
Node namespace: /
Topic type: sensor_msgs/msg/JointState
Endpoint type: PUBLISHER
GID: 01.10.bc.87.71.9d.c2.d7.dc.2f.53.d3.00.00.07.03.00.00.00.00.00.00.00.00
QoS profile:
  Reliability: RELIABLE
  History (Depth): KEEP_LAST (10)
  Durability: VOLATILE
  Lifespan: Infinite
  Deadline: Infinite
  Liveliness: AUTOMATIC
  Liveliness lease duration: Infinite

Subscription count: 5

Node name: cumotion_planner
Node namespace: /
Topic type: sensor_msgs/msg/JointState
Endpoint type: SUBSCRIPTION
GID: 01.10.2b.92.7f.ca.80.a2.fe.72.f0.83.00.00.18.04.00.00.00.00.00.00.00.00
QoS profile:
  Reliability: RELIABLE
  History (Depth): KEEP_LAST (10)
  Durability: VOLATILE
  Lifespan: Infinite
  Deadline: Infinite
  Liveliness: AUTOMATIC
  Liveliness lease duration: Infinite

Node name: topic_based_ros2_control_ur_ros2_control_sim
Node namespace: /
Topic type: sensor_msgs/msg/JointState
Endpoint type: SUBSCRIPTION
GID: 01.10.56.0d.64.6b.8e.52.6f.53.57.8e.00.00.28.04.00.00.00.00.00.00.00.00
QoS profile:
  Reliability: BEST_EFFORT
  History (Depth): KEEP_LAST (5)
  Durability: VOLATILE
  Lifespan: Infinite
  Deadline: Infinite
  Liveliness: AUTOMATIC
  Liveliness lease duration: Infinite

Node name: joint_parser
Node namespace: /
Topic type: sensor_msgs/msg/JointState
Endpoint type: SUBSCRIPTION
GID: 01.10.5c.ef.5a.ff.26.d4.24.e0.dc.0b.00.00.15.04.00.00.00.00.00.00.00.00
QoS profile:
  Reliability: RELIABLE
  History (Depth): KEEP_LAST (10)
  Durability: VOLATILE
  Lifespan: Infinite
  Deadline: Infinite
  Liveliness: AUTOMATIC
  Liveliness lease duration: Infinite

Node name: object_attachment
Node namespace: /
Topic type: sensor_msgs/msg/JointState
Endpoint type: SUBSCRIPTION
GID: 01.10.a4.60.62.49.f4.cd.a9.dc.d4.bc.00.00.16.04.00.00.00.00.00.00.00.00
QoS profile:
  Reliability: RELIABLE
  History (Depth): KEEP_LAST (10)
  Durability: VOLATILE
  Lifespan: Infinite
  Deadline: Infinite
  Liveliness: AUTOMATIC
  Liveliness lease duration: Infinite

Node name: robot_segmenter
Node namespace: /
Topic type: sensor_msgs/msg/JointState
Endpoint type: SUBSCRIPTION
GID: 01.10.d2.ad.90.c2.6d.5d.dd.4a.c3.c5.00.00.16.04.00.00.00.00.00.00.00.00
QoS profile:
  Reliability: RELIABLE
  History (Depth): KEEP_LAST (10)
  Durability: VOLATILE
  Lifespan: Infinite
  Deadline: Infinite
  Liveliness: AUTOMATIC
  Liveliness lease duration: Infinite
``` 

(robot_segmenter 有一个topic是BEST_EFFORT)

修改文件: sudo vi /opt/ros/humble/share/isaac_manipulator_pick_and_place/launch/isaac_sim_workflows.launch.py , 将SENSOR_DATA改成DEFAULT: 

![image2025-10-16 14:30:57.png](/assets/01KJC05QF1CDNZ4BMBD9E1R7QK/image2025-10-16%2014%3A30%3A57.png)

可以正常工作: 

![image2025-10-16 14:31:14.png](/assets/01KJC05QF1CDNZ4BMBD9E1R7QK/image2025-10-16%2014%3A31%3A14.png)

![image2025-10-16 14:31:35.png](/assets/01KJC05QF1CDNZ4BMBD9E1R7QK/image2025-10-16%2014%3A31%3A35.png)

调整罐子的位置, 机械臂会跟随, 但有时会GPU OOM: 

```
[component_container_mt-1] 2025-10-16 07:10:50.335 ERROR gxf/std/unbounded_allocator.cpp@58: Failure in cudaMalloc. cuda_error: cudaErrorMemoryAllocation, error_str: out of memory
[component_container_mt-1] 2025-10-16 07:10:50.335 ERROR external/com_nvidia_gxf/gxf/std/memory_buffer.hpp@79: unbounded_allocator Failed to allocate 9216000 size of memory of type 1. Error code: GXF_OUT_OF_MEMORY

``` 

关键信息: 

  - DOPE模型, 只能识别一个物体, 而且要经过单独的训练. 对于不同物体, 需要进行不同训练, 加载各自的模型
  - 夹爪会悬停在物体上面, 其行为定义是在 isaac_manipulator_bringup/launch/include/static_transforms.launch.py : 

```
'object_to_grasp_frame': {
    'parent_frame': 'detected_object1',
    'child_frame': 'goal_frame',
    'translation': [0.043, 0.359, 0.065],  # ← 悬停偏移量（米）
    'rotation': [0.553, 0.475, -0.454, 0.513],  # ← 夹爪姿态（四元数）
},
```

# 对DOPE的代码分析

```
═══════════════════════════════════════════════════════════════
循环开始: 环境认知阶段 (感知当前状态)
═══════════════════════════════════════════════════════════════

第一步: 传感器数据采集
│
├─── Isaac Sim Camera
│    ├─ 输出: RGB图像 (1920x1080)
│    │       深度图像 (原始)
│    │       相机标定信息
│    └─ 话题: /front_stereo_camera/left/image_raw
│              /front_stereo_camera/depth/ground_truth

第二步: 机器人状态解析
│
├─── Joint Parser Node
│    ├─ 输入: /joint_states (Isaac Sim)
│    ├─ 输出: /isaac_parsed_joint_states
│    └─ 作用: 解析关节状态
│
└─── Robot State Publisher
     ├─ 输入: /isaac_parsed_joint_states + URDF
     ├─ 输出: TF树 (机器人所有连杆的空间位置)
     └─ 作用: 计算机器人姿态

第三步: 物体位姿感知
│
└─── DOPE Pipeline
     ├─ 处理: 降帧 → 预处理 → 推理 → 解码 → 滤波
     ├─ 输入: RGB图像
     ├─ 输出: detected_object1 (TF frame)
     └─ 作用: 识别物体的6D姿态

第四步: 环境深度处理
│
└─── Robot Segmenter
     ├─ 输入: 原始深度图 + 关节状态 + 机器人模型
     ├─ 处理: 
     │   ├─ 计算机器人各连杆位置
     │   ├─ 生成碰撞球体
     │   └─ 从深度图中移除机器人像素
     ├─ 输出: /cumotion/camera_1/world_depth (纯环境深度)
     └─ 作用: 获取不含机器人的环境深度

第五步: 3D环境建模
│
└─── nvblox
     ├─ 输入: RGB图像 + 环境深度 (world_depth)
     ├─ 处理:
     │   ├─ TSDF融合 (体素化场景)
     │   └─ ESDF生成 (距离场计算)
     ├─ 输出: ESDF (通过服务提供查询)
     └─ 作用: 维护实时3D环境地图

第六步: 物体抓取状态检测
│
└─── Object Attachment
     ├─ 输入: 环境深度 + 夹爪TF + 关节状态
     ├─ 处理:
     │   ├─ 在夹爪周围搜索深度点
     │   ├─ 聚类检测物体
     │   └─ 生成物体碰撞表示
     ├─ 输出: 附着状态 + 碰撞模型更新
     └─ 作用: 检测是否抓住物体

环境认知完成
↓
当前世界状态:
  - 物体位置: detected_object1
  - 机器人位姿: TF树
  - 环境障碍物: ESDF
  - 抓取状态: Object Attachment状态

═══════════════════════════════════════════════════════════════
任务决策阶段 (根据环境状态决定下一步动作)
═══════════════════════════════════════════════════════════════

第七步: 目标位姿计算
│
└─── Static Transform Publisher
     ├─ 输入: detected_object1 (物体当前位置)
     ├─ 计算: 物体位置 + 偏移量 [0.043, 0.359, 0.065]
     ├─ 输出: goal_frame (期望的悬停位姿)
     └─ 作用: 定义机器人应该去的目标位置

第八步: 任务触发判断
│
└─── Goal Initializer Node
     ├─ 监听: goal_frame的位姿变化
     ├─ 判断逻辑:
     │   ├─ 定时检查 (每0.5秒)
     │   ├─ 计算位移量
     │   └─ 若位移 > 0.1m → 触发新任务
     ├─ 输出: MotionPlanRequest (规划请求)
     │   ├─ 起点: 当前关节状态
     │   ├─ 终点: goal_frame位姿
     │   └─ 规划器: 'isaac_ros_cumotion'
     └─ 作用: 决策是否需要重新规划运动

任务决策完成
↓
决策结果: 需要移动到 goal_frame

═══════════════════════════════════════════════════════════════
任务执行阶段 (规划路径并执行运动)
═══════════════════════════════════════════════════════════════

第九步: 规划请求协调
│
└─── MoveIt2 Move Group
     ├─ 接收: MotionPlanRequest
     ├─ 准备: 
     │   ├─ 更新Planning Scene (当前机器人状态)
     │   ├─ 获取SRDF配置 (运动约束)
     │   └─ 选择规划器: isaac_ros_cumotion
     ├─ 调用: cuMotion Planner服务
     └─ 作用: 协调规划流程

第十步: 轨迹规划计算
│
└─── cuMotion Planner (响应规划请求)
     ├─ 输入:
     │   ├─ MotionPlanRequest (起点终点)
     │   ├─ ESDF (从nvblox查询)
     │   ├─ 当前关节状态
     │   └─ 附着物体碰撞模型
     ├─ 计算 (GPU并行):
     │   ├─ 生成候选轨迹
     │   ├─ 碰撞检测 (基于ESDF快速查询)
     │   ├─ 轨迹优化
     │   └─ 选择最优解
     ├─ 输出: MotionPlanResponse
     │   ├─ trajectory: 关节空间轨迹 [(t0,q0), (t1,q1), ...]
     │   ├─ planning_time: 规划耗时
     │   └─ error_code: SUCCESS/FAILURE
     └─ 作用: 计算无碰撞轨迹

第十一步: 规划结果验证
│
└─── MoveIt2 Move Group (继续处理)
     ├─ 接收: MotionPlanResponse
     ├─ 验证:
     │   ├─ 检查error_code
     │   ├─ 轨迹合法性验证
     │   └─ 时间参数化调整
     ├─ 输出: ExecuteTrajectory命令 → ROS2 Control
     └─ 作用: 准备执行轨迹

第十二步: 轨迹执行控制
│
├─── ROS2 Control Manager
│    └─ 作用: 接收执行命令，分发给控制器
│
└─── Scaled Joint Trajectory Controller
     ├─ 输入: 轨迹序列 [(t0,q0), (t1,q1), ...]
     ├─ 处理:
     │   ├─ 时间插值 (生成细密控制点)
     │   ├─ PID控制 (跟踪期望轨迹)
     │   └─ 速度缩放 (根据负载调整)
     ├─ 输出: 关节位置命令 → Isaac Sim
     └─ 作用: 实时控制机器人运动

第十三步: 物理仿真执行
│
└─── Isaac Sim 机器人接口
     ├─ 输入: 关节位置命令
     ├─ 处理: 物理引擎仿真
     ├─ 输出:
     │   ├─ 新的关节状态 → /joint_states
     │   ├─ 新的RGB图像 → /front_stereo_camera/left/image_raw
     │   └─ 新的深度图像 → /front_stereo_camera/depth/ground_truth
     └─ 作用: 驱动机器人运动，产生新的环境状态

任务执行完成
↓
机器人位置已更新
``` 

# 测试FoundationPose

在 [20251017 - 为机械臂开发环境, 整备4090 48G机器, 并完成Pick and Place任务] 中完成, 重点: 

  - 修改QoS
  - 注意镜头位置, 目标物体要与镜头足够近, 远离的话识别会不稳定, 导致流程无法进行

# 对Pick and Place任务进行代码梳理

[附件: pick_and_place_workflow_visualization.html] 

重点: 

  - 以Orchestrator为中心轴, 对各组件进行调用
  - 使用三种不同的调用手段 (已经在图中标注): 
    - Topic（主题）：
      - 作用：单向、连续广播数据，像“公告栏”——一个节点发布信息，其他节点订阅监听。没有响应机制，适合实时流式数据（如传感器读数）。
      - 优势：高效、低开销，支持多对多通信，不阻塞发送者。
      - 何时用：当数据需要持续更新，且不需要立即反馈时。
    - Service（服务）：
      - 作用：请求-响应模式，像“客服电话”——客户端发送请求，服务器处理后返回结果。同步阻塞（客户端等待响应）。
      - 优势：简单、可靠，适合一次性查询或计算任务。
      - 何时用：需要明确输入输出、短期任务时。
    - Action（动作）：
      - 作用：扩展的请求-响应，带反馈和取消，像“任务委托”——客户端发送目标，服务器执行长时间任务，同时提供进度反馈、结果或取消选项。异步非阻塞。
      - 优势：处理复杂、耗时操作，支持中断和监控。
      - 何时用：任务可能长时间运行、需要中间反馈时。
