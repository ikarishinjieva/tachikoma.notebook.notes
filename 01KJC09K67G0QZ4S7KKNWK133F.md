---
title: 20260104 - 机械臂 moveit+ros2_control+driver的启动链路
confluence_page_id: 4359512
created_at: 2026-01-04T14:28:30+00:00
updated_at: 2026-01-04T14:29:54+00:00
---

```
用户启动：alicia_d_moveit/launch/real_robot.launch.py
└─ 1) 生成 MoveIt 配置 + robot_description（URDF/xacro 展开）
   ├─ 读取：alicia_d_moveit/launch/moveit_config_builder.py
   ├─ xacro 映射参数：port / gripper_type / default_speed_deg_s
   └─ robot_description 内包含 <ros2_control>
      └─ 指定硬件插件：alicia_d_driver/AliciaDHardwareInterface
         └─ 硬件参数：port, gripper_type, default_speed_deg_s

└─ 2) 启动 ros2_control 总管：controller_manager(ros2_control_node)
   ├─ 入参 A：robot_description（决定“硬件是谁、关节/接口有哪些”）
   │  └─ pluginlib 加载硬件：AliciaDHardwareInterface
   │     ├─ on_init/on_configure/on_activate
   │     ├─ 连接串口 SerialCommunicator
   │     ├─ 启动解析线程 AliciaDDataParserControl
   │     └─ torque_control("on") 使能力矩（具备保位能力）
   └─ 入参 B：alicia_d_moveit/config/ros2_controllers.yaml（决定“有哪些 controller”）
      ├─ joint_state_broadcaster -> JointStateBroadcaster
      ├─ Alicia_controller       -> JointTrajectoryController(6轴)
      └─ Gripper_controller      -> JointTrajectoryController(夹爪)

└─ 3) 用 spawner 逐个激活 controller（调用 /controller_manager 的服务）
   ├─ 先：joint_state_broadcaster
   ├─ 再：Alicia_controller
   └─ 再：Gripper_controller

└─ 4) 控制器就绪后启动 MoveIt：move_group
   └─ 读取：alicia_d_moveit/config/moveit_controllers.yaml
      ├─ Alicia_controller  -> FollowJointTrajectory action
      └─ Gripper_controller -> FollowJointTrajectory action

└─ 5) 运行态数据流（链路闭环）
   ├─ MoveIt(move_group) 规划并执行
   │  └─ 发送 FollowJointTrajectory goal
   │     ├─ -> Alicia_controller/follow_joint_trajectory
   │     └─ -> Gripper_controller/follow_joint_trajectory
   ├─ ros2_control 控制循环（controller_manager update_rate=200Hz）
   │  ├─ JointTrajectoryController 插值/跟踪轨迹 -> 写 command_interface
   │  ├─ 硬件 read() -> 更新 state_interface（从串口解析到的状态）
   │  └─ 硬件 write() -> 下发命令到串口
   │     └─ AliciaDHardwareInterface::write()
   │        └─ AliciaDDataParserControl::set_joint_and_gripper()
   │           └─ SerialCommunicator -> 机械臂
   └─ 状态发布
      └─ joint_state_broadcaster 读 state_interface -> 发布 /joint_states
         └─ MoveIt/RViz 用 /joint_states 更新当前姿态
``` 

其中spawner 只接受controller名字, 其实际定义由ros2_control.xml提供
