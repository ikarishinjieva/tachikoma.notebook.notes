---
title: 20251115 - 为实际机械臂添加摄像头
confluence_page_id: 4359125
created_at: 2025-11-17T03:46:16+00:00
updated_at: 2025-11-28T12:29:18+00:00
---

起始commit: c1056b02940102d156b475745276116ef5f5f10c

在 <https://raw.githubusercontent.com/elephantrobotics/mycobot_ros/refs/heads/noetic/mycobot_description/urdf/mycobot_280_m5/mycobot_280m5_with_camera_flange_pump.urdf> 找到 相机和pump共存的 URDF

让模型根据当前 mycobot_280_m5_adaptive_gripper.urdf 和 mycobot_280m5_with_camera_flange_pump.urdf 生成一个 夹爪和相机共存的URDF

将URDF与之前的urdf合并, 需要注意 gripper_base.stl 的使用 (之前为了避免冲突检测失误, 使用了STL格式), 还需要将 camera_flange 转换成stl格式

修改SRDF, 将新增的相机和机械爪/机械臂 组件, 避免冲突检测

然后在isaac sim中, 新增一个ROS2相机, 参考 ([20251105 - 在仿真环境中, 为倒水添加视觉])

然后在moveit启动脚本中, 需要添加TF的发布话题 (机械臂 -> camera_flange -> camera, 需要发布两段TF话题)

然后在moveit启动脚本中, 增加apriltag组件容器

运行test_real.py后, 出现报错:

```
[WARN] [1763442344.497557912] [test_real_robot]: [Tag] TF变换失败: Lookup would require extrapolation into the past.  Requested time 12.683334 but the earliest data is at time 26482.168048, when looking up transform from frame [camera_optical_frame] to frame [world]
``` 

camera发布的图像话题时间轴不是sim_time, 在graph中, 找到ROS2 Camera Helper和ROS2 Camera Info Helper, 取消 "Reset Simulation Time On Stop"

会碰到夹爪在调度时碰撞崩溃, 没有明显的日志, 并且偶发. 重要技巧:

将urdf中, gripper_base.dae, 转成stl.

但注意: 在将urdf导出成usd时 (isaac sim), 需要保持dae. (使用stl, 那么机械臂的图形会未知错误)

在moveit使用的urdf, 需要全部转成stl. (使用dae, 会出现夹爪在调度时崩溃)

commit: 452bc5212ab5254ed3a006957850936ae2b06e36

  - 对于apriltag, 需要配置抓取位置的偏移, tag_to_object_offset_in_tag, 但其不符合人类习惯
  - 增加了tag_to_object_offset_in_world, 以世界坐标系来调整偏移
  - 但调整完以后, 需要转换到tag_to_object_offset_in_tag中, 之后使用tag_to_object_offset_in_tag. 
  - 否则, 如果坚持使用tag_to_object_offset_in_world作为偏移, 那么如果移动杯子位置, 就会产生误差
  - 另外: 从tag_to_object_offset_in_tag来看, apriltag识别的位置可能有一个固定偏移, tag_to_object_offset_in_tag能正常工作, 但偏移量不符合人类的距离认知 (这个偏移量被配置的偏移抵消)

但发现, 如果移动了杯子的深度, 那么还是会偏移. 从头进行诊断: 

  - 使用 ros2 run tf2_tools view_frames 获取ros2中的层级关系, 由AI整理: 

```
world
  └─ g_base
       └─ joint1 └ joint2 └ joint3 └ joint4 └ joint5 └ joint6 └ joint6_flange
                                                        └ camera_flange
                                                             └ camera_link
                                                                  └ camera_optical_frame
                                                                  ├ tag25h9:0
                                                                  └ tag25h9:31
                                                             └ gripper_base
                                                                ├ gripper_left3 └ gripper_left1
                                                                └ gripper_right3 └ gripper_right1
```

  - 与isaac sim中的层次关系进行对应: 
    -       - ROS world ↔ Isaac /World
      - ROS g_base ↔ Isaac /World/mycobot_280_m5_adaptive_gripper/g_base
      - ROS joint6_flange ↔ Isaac /World/mycobot_280_m5_adaptive_gripper/camera_flange

（在 TF 树里 camera_flange 的父亲就是 joint6_flange，和 USD 里名字一致）

      - ROS camera_flange ↔ Isaac 同名 Prim camera_flange（机械末端法兰）
      - ROS camera_link：相机机身中间 link，父亲是 camera_flange，孩子是 camera_optical_frame \+ Tag
      - ROS camera_optical_frame ↔（概念上）Isaac 的相机光学坐标系
      - 很可能对应 /World/.../CameraOnHand（或 Camera prim 的内部“光学 frame”），而不是机械 camera_flange
  - 定量检查关系
    - ROS g_base ↔ Isaac /World/mycobot_280_m5_adaptive_gripper/g_base
      - ROS: 

```
admin@bot:/workspaces/isaac_ros-dev$ ros2 run tf2_ros tf2_echo world g_base
[INFO] [1763520594.898125938] [tf2_echo]: Waiting for transform world ->  g_base: Invalid frame ID "world" passed to canTransform argument target_frame - frame does not exist
At time 0.0
- Translation: [0.000, 0.000, 0.000]
- Rotation: in Quaternion (xyzw) [0.000, 0.000, 0.000, 1.000]
- Rotation: in RPY (radian) [0.000, -0.000, 0.000]
- Rotation: in RPY (degree) [0.000, -0.000, 0.000]
- Matrix:
  1.000  0.000  0.000  0.000
  0.000  1.000  0.000  0.000
  0.000  0.000  1.000  0.000
  0.000  0.000  0.000  1.000
```

      - isaac sim (使用 debug_apriltag_coordinates.py 脚本)

```
================================================================================
Prim: /World/mycobot_280_m5_adaptive_gripper/g_base
================================================================================
  位置 (m):         x=-0.037602, y=-0.018253, z=+0.034600
  四元数 (w,x,y,z): +1.000000, +0.000000, +0.000000, +0.000000
  四元数 (x,y,z,w): +0.000000, +0.000000, +0.000000, +1.000000
  欧拉角 RPY (deg): roll=+0.00, pitch=+0.00, yaw=+0.00
  缩放 (SVD):       1.000000, 1.000000, 1.000000
  AABB 尺寸 (m):    N/A (不是 Mesh 或无顶点)
```

      - 其原因是: ROS2 默认以机械臂为零点, 所以ROS2的world坐标系, 对应的是isaac sim中的机械臂坐标系
      - 两者不一致, 为了计算方便, 需要将isaac sim中的机械臂在world坐标系中归零, 然后从头开始对齐
    - ROS camera_flange ↔ Isaac 同名 Prim camera_flange

````
ROS2输出: 
```
admin@bot:/workspaces/isaac_ros-dev$ ros2 run tf2_ros tf2_echo world camera_flange
At time 922.350048104
- Translation: [-0.054, -0.001, 0.291]
- Rotation: in Quaternion (xyzw) [-0.235, 0.471, -0.175, 0.832]
- Rotation: in RPY (radian) [-0.896, 0.778, -0.803]
- Rotation: in RPY (degree) [-51.325, 44.556, -45.992]
- Matrix:
  0.495  0.069  0.866 -0.054
 -0.513  0.828  0.227 -0.001
 -0.702 -0.556  0.445  0.291
  0.000  0.000  0.000  1.000
```

isaac sim输出:

```

================================================================================
Prim: /World/mycobot_280_m5_adaptive_gripper/camera_flange
================================================================================
  位置 (m):         x=-0.054290, y=-0.000854, z=+0.291243
  四元数 (w,x,y,z): +0.586641, -0.151705, +0.753748, -0.254364
  四元数 (x,y,z,w): -0.151705, +0.753748, -0.254364, +0.586641
  欧拉角 RPY (deg): roll=-107.99, pitch=+53.82, yaw=-116.75
  缩放 (SVD):       1.000000, 1.000000, 1.000000
  AABB 尺寸 (m):    N/A (不是 Mesh 或无顶点)

```

````

  

      - 两者位置一致, 但旋转差很多, 原因: 

        - 现在 ROS 里看到的 camera_flange 位姿并不是 Isaac 直接“原样发出来”的，而是：Isaac 只发关节角 + 基坐标，真正的 camera_flange TF 是 ROS 端根据 URDF 计算出来的。 如果 URDF 里的 camera_flange 定义和 Isaac 场景里的 camera_flange Prim 姿态有差异，就会出现你现在看到的“位置几乎一样，姿态差很多”

      - 向前检查各个joint的对应关系 (camera_flange TF是由URDF根据前面的关节位置推理出来的, 也就是前面的关节可能有误差)
        - joint6: 
          - 基本一致

````
ROS2输出: 
```
admin@bot:/workspaces/isaac_ros-dev$ ros2 run tf2_ros tf2_echo world joint6
At time 345.366684678
- Translation: [-0.057, -0.050, 0.319]
- Rotation: in Quaternion (xyzw) [-0.248, -0.513, 0.116, 0.813]
- Rotation: in RPY (radian) [-0.981, -0.890, 0.783]
- Rotation: in RPY (degree) [-56.193, -50.992, 44.846]
- Matrix:
  0.446  0.065 -0.893 -0.057
  0.444  0.850  0.284 -0.050
  0.777 -0.523  0.350  0.319
  0.000  0.000  0.000  1.000
```

isaac sim输出:

```

================================================================================
Prim: /World/mycobot_280_m5_adaptive_gripper/joint6
================================================================================
  位置 (m):         x=-0.057349, y=-0.049555, z=+0.319189
  四元数 (w,x,y,z): +0.813381, -0.248109, -0.513147, +0.116333
  四元数 (x,y,z,w): -0.248109, -0.513147, +0.116333, +0.813381
  欧拉角 RPY (deg): roll=-56.19, pitch=-50.99, yaw=+44.84
  缩放 (SVD):       1.000000, 1.000000, 1.000000
  AABB 尺寸 (m):    N/A (不是 Mesh 或无顶点)

```
```` 
        - joint6_flange: 
          - 旋转差异

````
ROS2输出: 
```
At time 365.216685714
- Translation: [-0.054, -0.011, 0.295]
- Rotation: in Quaternion (xyzw) [0.834, 0.257, 0.179, -0.454]
- Rotation: in RPY (radian) [-2.237, -0.562, 0.320]
- Rotation: in RPY (degree) [-128.174, -32.200, 18.329]
- Matrix:
  0.803  0.592  0.065 -0.054
  0.266 -0.455  0.850 -0.011
  0.533 -0.665 -0.523  0.295
  0.000  0.000  0.000  1.000
```

isaac sim输出:

```

================================================================================
Prim: /World/mycobot_280_m5_adaptive_gripper/joint6_flange
================================================================================
  位置 (m):         x=-0.054367, y=-0.010803, z=+0.295340
  四元数 (w,x,y,z): +0.488281, -0.868708, +0.082756, +0.008882
  四元数 (x,y,z,w): -0.868708, +0.082756, +0.008882, +0.488281
  欧拉角 RPY (deg): roll=-121.70, pitch=+5.52, yaw=-7.80
  缩放 (SVD):       1.000000, 1.000000, 1.000000
  AABB 尺寸 (m):    N/A (不是 Mesh 或无顶点)

```

```` 
          - 判断ROS2和ISAAC中, 分别 joint6_flange相对joint6旋转了多少: 

````
ROS2:

```
admin@bot:/workspaces/isaac_ros-dev$ ros2 run tf2_ros tf2_echo world joint6
At time 2205.316781682
- Translation: [-0.057, -0.050, 0.319]
- Rotation: in Quaternion (xyzw) [-0.248, -0.513, 0.116, 0.813]
- Rotation: in RPY (radian) [-0.981, -0.890, 0.783]
- Rotation: in RPY (degree) [-56.193, -50.992, 44.846]
- Matrix:
  0.446  0.065 -0.893 -0.057
  0.444  0.850  0.284 -0.050
  0.777 -0.523  0.350  0.319
  0.000  0.000  0.000  1.000

admin@bot:/workspaces/isaac_ros-dev$ ros2 run tf2_ros tf2_echo world joint6_flange
At time 2210.866781972
- Translation: [-0.054, -0.011, 0.295]
- Rotation: in Quaternion (xyzw) [0.834, 0.257, 0.179, -0.454]
- Rotation: in RPY (radian) [-2.237, -0.562, 0.320]
- Rotation: in RPY (degree) [-128.174, -32.200, 18.329]
- Matrix:
  0.803  0.592  0.065 -0.054
  0.266 -0.455  0.850 -0.011
  0.533 -0.665 -0.523  0.295
  0.000  0.000  0.000  1.000
```

ISAAC:
```
================================================================================
Prim: /World/mycobot_280_m5_adaptive_gripper/joint6
================================================================================
  位置 (m):         x=-0.057349, y=-0.049555, z=+0.319189
  四元数 (w,x,y,z): +0.813381, -0.248109, -0.513147, +0.116333
  四元数 (x,y,z,w): -0.248109, -0.513147, +0.116333, +0.813381
  欧拉角 RPY (deg): roll=-56.19, pitch=-50.99, yaw=+44.84
  缩放 (SVD):       1.000000, 1.000000, 1.000000
  AABB 尺寸 (m):    N/A (不是 Mesh 或无顶点)

================================================================================
Prim: /World/mycobot_280_m5_adaptive_gripper/joint6_flange
================================================================================
  位置 (m):         x=-0.054367, y=-0.010803, z=+0.295340
  四元数 (w,x,y,z): +0.488281, -0.868708, +0.082756, +0.008882
  四元数 (x,y,z,w): -0.868708, +0.082756, +0.008882, +0.488281
  欧拉角 RPY (deg): roll=-121.70, pitch=+5.52, yaw=-7.80
  缩放 (SVD):       1.000000, 1.000000, 1.000000
  AABB 尺寸 (m):    N/A (不是 Mesh 或无顶点)
  ```

帮我算一下这两组的角度差
````

            - 局部旋转 joint6 → joint6_flange：ROS 和 Isaac 之间差了大约一个绕 Z 轴的 45° 旋转。
            - 再次修正 URDF -> USD的过程 (认为是之前修改成stl, 导致了什么偏差? )
            - 新的验证: 

```
ROS2:
admin@bot:/workspaces/isaac_ros-dev$ ros2 run tf2_ros tf2_echo world joint6_flange
1763530403.971573 [99]   tf2_echo: config: //CycloneDDS/Domain/General: 'NetworkInterfaceAddress': deprecated element (/workspaces/isaac_ros-dev/a.xml line 9)
[INFO] [1763530403.977053957] [tf2_echo]: Waiting for transform world ->  joint6_flange: Invalid frame ID "world" passed to canTransform argument target_frame - frame does not exist
At time 173.33342357
- Translation: [-0.059, -0.010, 0.300]
- Rotation: in Quaternion (xyzw) [0.848, 0.238, 0.146, -0.450]
- Rotation: in RPY (radian) [-2.244, -0.481, 0.312]
- Rotation: in RPY (degree) [-128.574, -27.559, 17.903]
- Matrix:
  0.844  0.536  0.034 -0.059
  0.273 -0.482  0.833 -0.010
  0.463 -0.693 -0.553  0.300
  0.000  0.000  0.000  1.000

ISAAC: `
================================================================================
Prim: /World/mycobot_280_m5_adaptive_gripper/joint6_flange
================================================================================
  位置 (m):         x=-0.059046, y=-0.009649, z=+0.300114
  四元数 (w,x,y,z): +0.449844, -0.848684, -0.236922, -0.145752
  四元数 (x,y,z,w): -0.848684, -0.236922, -0.145752, +0.449844
  欧拉角 RPY (deg): roll=-128.52, pitch=-27.42, yaw=+17.78
  缩放 (SVD):       1.000000, 1.000000, 1.000000
  AABB 尺寸 (m):    N/A (不是 Mesh 或无顶点)
```

              - joint6_flange的位置也一致
            - 检查camera_flange的位姿也一致
          - 因为重新生成了USD, 所以 CameraOnHand 的偏移在Isaac Sim中也需要重新调整
          - 调整后: Isaac中的/World/mycobot_280_m5_adaptive_gripper/camera_flange/CameraOnHand 与 camera_link的位姿 变为一致
          - 另外需要调整 moveit启动脚本中, camera_link的发布 (根据CameraOnHand的偏移关系, 发布camera_link的TF): 

```
    # ================= 相机相关 TF（camera_flange -> camera_link -> camera_optical_frame） =================
    # 1) camera_flange -> camera_link
    #
    # 数值来源: Isaac Sim USDA 中 CameraOnHand 对应的 Local Transform:
    #   xformOp:translate = (0.03914059164164335,
    #                        0.0058398539872201935,
    #                        0.0032736610252634968)
    #   xformOp:orient   = (0.562030414882409,
    #                        0.46297452056840194,
    #                       -0.5364195132694431,
    #                        0.4266503390779614)
    #   注意: USD 四元数格式为 (w,x,y,z)，static_transform_publisher 需要 (x,y,z,w)
    #
    camera_mount_tf_node = Node(
        package="tf2_ros",
        executable="static_transform_publisher",
        name="static_tf_camera_flange_to_camera_link",
        output="screen",
        arguments=[
            # 平移 tx ty tz
            "0.03914059164164335",
            "0.0058398539872201935",
            "0.0032736610252634968",
            # 四元数 qx qy qz qw（从 USD 的 w,x,y,z 转换而来）
            "0.46297452056840194",
            "-0.5364195132694431",
            "0.4266503390779614",
            "0.562030414882409",
            # 父 / 子坐标系
            "camera_flange", "camera_link",
        ],
        parameters=[use_sim_time_param],
    )

```

          - commit: 9e8db03ce76e4d9e37b980f5b1242e742456e9fb
            - ros2识别出的tag, 和isaac的tag位置一致
    - 总结经验: 
      - ROS2 只有关节位姿, 其他部分是通过关节位姿 + URDF中的组件关系 进行推理得出
      - ROS2 发布的TF, 需要和 Isaac Sim中的组件位置 对账, 两者一致, 才认为配置正确
      - ROS2 相机, 需要使用光学坐标系, 而并不是相机坐标系 (ROS2相机的坐标系定义, 和apriltag检测组件的坐标系定义不同). 所以moveit脚本中, 需要发布两个坐标系的转换关系. 
      - 从URDF到USD文件的生成, 各组件需要使用dae, 否则外观和衔接尺寸会出错. 
      - 但ROS2使用的URDF, 需要对关键组件使用stl, 否则会有碰撞
        - TODO: 这里的碰撞会有点奇怪, 会产生机械臂和摄像头的碰撞, 需要后续处理

  

# 额外的技巧 - 在Isaac sim中获取绝对位姿

除了使用 debug_apriltag_coordinates.py 这个脚本, 还可以使用isaac sim的工具: simulation data visualizer 

![image2025-11-19 23:54:16.png](/assets/01KJC08QZJFY6ZC78Y6TJ7R5VV/image2025-11-19%2023%3A54%3A16.png)

  

  

# 额外的技巧 - 在isaac sim中显示碰撞体

![image2025-11-19 23:56:43.png](/assets/01KJC08QZJFY6ZC78Y6TJ7R5VV/image2025-11-19%2023%3A56%3A43.png)

  

  

# 诊断为什么有时会计算崩溃

打开碰撞体的显示, 发现: 

![image2025-11-20 0:3:56.png](/assets/01KJC08QZJFY6ZC78Y6TJ7R5VV/image2025-11-20%200%3A3%3A56.png)

  

确实会发生碰撞, 曲臂(比如joint4)的碰撞体比看上去的大 (包含了曲臂中间的中空部分)

  

尝试在srdf中, 体现真实的碰撞, 而不是将camera_flange与其他所有组件都"避免碰撞"

  

  

发现新的问题, 在Isaac Sim中, 相机姿态正确, 但在rviz中 (可能因为urdf中更换了stl), 所以导致相机位姿异常:

![image2025-11-20 10:57:1.png](/assets/01KJC08QZJFY6ZC78Y6TJ7R5VV/image2025-11-20%2010%3A57%3A1.png)

回到为什么要使用stl问题:

  - 来自于gripper_base, 如果不使用stl, 那么机械臂移动就会报错, 发现了虚假的碰撞 (视觉上没有碰撞)
    - 认为是dae是层次定义, 碰撞检测器对于各个层次上的缩放参数等处理不佳, 导致冲突检测异常
    - stl是将层次定义预计算成碰撞三角
    - 但使用stl, 会导致模型视觉上这部分组件异常
      - 发现了组件视觉异常的原因, 有一个stl文件丢失, 需要重新生成
  - 尝试使用obj文件, 使用vhacd拆解

```
python3 -c "import pybullet as p, trimesh, os; trimesh.load('gripper_base.dae', force='mesh').export('temp.obj'); p.vhacd('temp.obj', 'gripper_base.obj', 'log.txt'); os.remove('temp.obj')"

```

    - 与使用dae一样, 也会发生冲突异常
  - 还是考虑使用stl文件: 
    - stl有默认的轴, dae是将轴写在文件中自洽, 所以本实验中, dae会比stl多一个旋转, 对stl的配置需要修正这个旋转:

```
    <collision>
      <geometry>
        <mesh filename="package://mycobot_description/urdf/adaptive_gripper/gripper_base.stl"/>
      </geometry>
      <origin xyz = "0.0 0 -0.012 " rpy = " 1.570795 0 0"/>
    </collision>
    <!-- <collision>
      <geometry>
        <mesh filename="package://mycobot_description/urdf/adaptive_gripper/gripper_base.dae"/>
      </geometry>
      <origin xyz = "0.0 0 -0.012 " rpy = " 0 0 0"/>
    </collision> -->
```

    - 将相机到机械臂的srdf中碰撞避免规则进行修改, 只保留相邻组件的规则: 

```
<!--
    <disable_collisions link1="g_base" link2="camera_flange" reason="Never"/>
    <disable_collisions link1="joint1" link2="camera_flange" reason="Never"/>
    <disable_collisions link1="joint2" link2="camera_flange" reason="Never"/>
    <disable_collisions link1="joint3" link2="camera_flange" reason="Never"/>
    <disable_collisions link1="joint4" link2="camera_flange" reason="Never"/>
    <disable_collisions link1="joint5" link2="camera_flange" reason="Never"/>
    <disable_collisions link1="joint6" link2="camera_flange" reason="Never"/>
-->

    <disable_collisions link1="joint6_flange" link2="camera_flange" reason="Adjacent"/>
```

      - 修改后, "计算崩溃"的问题可以解决
    - 将更多的srdf中不必要的碰撞避免规则, 进行去除, 发现会产生碰撞, 好像与g_base底座 <-> 夹爪有关: 
      - ![image2025-11-20 13:42:34.png](/assets/01KJC08QZJFY6ZC78Y6TJ7R5VV/image2025-11-20%2013%3A42%3A34.png)
      - 测试确实与这一段有关: 

```
    <disable_collisions link1="g_base" link2="gripper_left1" reason="Never"/>
    <disable_collisions link1="g_base" link2="gripper_left2" reason="Never"/>
    <disable_collisions link1="g_base" link2="gripper_left3" reason="Never"/>
    <disable_collisions link1="g_base" link2="gripper_right1" reason="Never"/>
    <disable_collisions link1="g_base" link2="gripper_right2" reason="Never"/>
    <disable_collisions link1="g_base" link2="gripper_right3" reason="Never"/>

```

      - 将这部分夹爪也更换成stl, 可以修复这个问题
      - 现在在srdf中, 只有相邻组件和永远不会碰撞的组件 才会配置碰撞避免
        - commit: 8813f5dfdab9b58df3963770f9cef5f5225a15b2

进一步做一些调整: 

  - commit: 9a1a112d2550d4c9fccd7486a81bb08f572a7d06
  - 已经能正确检查到tag的位置, 需要设定好 抓取位置的偏移
  - 杯子的位置很重要: 很多位置是机械臂的奇异点, 达不到
  - 剩下的问题: 倾斜角修正(approach_tilt_deg) 是个问题, 其不是绝对的位姿, 而是一个相对的偏移. TODO 建议是在IK阶段, 将倾斜角设定为一个范围内

  

  

# 接入真实相机

在容器内安装:

```
sudo apt install ros-humble-usb-cam
``` 

在mycobot_280_real_moveit.launch.py中, 加入: 

  - 相机相关的 TF 发布（camera_flange → camera_link → camera_optical_frame）

  - AprilTag 检测容器

在slider中, 加入 usb_cam_node

在笔记本上运行slider: 

```
sudo chmod 666 /dev/video0
python3 slider_control_adaptive_gripper.fix_seq.py
``` 

插入usb摄像头, 发现容器内看不到, 需要特殊处理: 

```
在宿主机上插入摄像头, v4l2-ctl --list-devices 确定设备好: 
 
USB 2.0 Camera: USB 2.0 Camera (usb-0000:00:14.0-3):
	/dev/video3
	/dev/video4
	/dev/media1
 
确定设备号: 
huangyan@bot:/data/huangyan/isaac_ros-dev$ ls -alh /dev/video3
crw-rw----+ 1 root video 81, 4 Nov 20 12:16 /dev/video3
huangyan@bot:/data/huangyan/isaac_ros-dev$ ls -alh /dev/video4
crw-rw----+ 1 root video 81, 5 Nov 20 12:16 /dev/video4
 
在容器内: 
sudo mknod /dev/video3 c 81 4
sudo mknod /dev/video4 c 81 5
sudo chmod 666 /dev/video*
``` 

在rviz中, 可以看到效果: 

![image2025-11-20 20:25:45.png](/assets/01KJC08QZJFY6ZC78Y6TJ7R5VV/image2025-11-20%2020%3A25%3A45.png)

下一个问题: 路径规划均失败, 重复规划就可以成功, 认为是规划算法/参数 鲁棒性有问题

切换ompl的算法:

  - commit: 2696662fd5708cf7b32e3dc094e662478e4d868a

规划效果好像好一些

新的问题: 调度有可能失败: 

```
[move_group-3] [INFO] [1763691387.591306480] [moveit_ros.current_state_monitor]: Didn't receive robot state (joint angles) with recent timestamp within 1.000000 seconds. Requested time 1763691386.591231, but latest received state has time 1763691383.618110.
[move_group-3] Check clock synchronization if your are running ROS across multiple machines!
[move_group-3] [WARN] [1763691387.591892837] [moveit_ros.trajectory_execution_manager]: Failed to validate trajectory: couldn't receive full current joint state within 1s

``` 

在slider和moveit启动脚本上, 都开启 use_sim_time, 来拉齐时间差异

新的问题: 相机图像传输很慢: 

  - 在slider端, ros2 topic hz /camera/rgb 接近设置的30Hz, 但在moveit端, 没有结果
  - 降低分辨率和fps, 另外将compressed话题也映射好: 

```
        self.declare_parameter('camera_width', 320)
        self.declare_parameter('camera_height', 240)
        self.declare_parameter('camera_fps', 10.0)
 
...
 
 
            usb_cam_cmd.extend([
                '--remap', '__ns:=/usb_cam_0',
                '-r', '/usb_cam_0/image_raw:=/camera/rgb',
                '-r', '/usb_cam_0/image_raw/compressed:=/camera/rgb/compressed',
                '-r', '/usb_cam_0/camera_info:=/camera/camera_info'
            ])

```

  - ros2 topic hz /camera/rgb 在moveit端也接近10Hz, 在rviz中相机很流畅

新的问题: ros版本的apriltag, 无法识别yuv422_yuy2格式

```
[component_container_mt-4] [INFO] [1763705531.204004341] [apriltag_detector.ManagedNitrosSubscriber]: Starting Managed Nitros Subscriber
[INFO] [launch_ros.actions.load_composable_nodes]: Loaded node '/apriltag_detector' in container '/apriltag_container'
[component_container_mt-4] [ERROR] [1763705531.222318296] [NitrosImage]: [convert_to_custom] Unsupported encoding from ROS [yuv422_yuy2].
[component_container_mt-4] terminate called after throwing an instance of 'std::runtime_error'
[component_container_mt-4]   what():  [convert_to_custom] Unsupported encoding from ROS.
[ERROR] [component_container_mt-4]: process has died [pid 1417517, exit code -6, cmd '/opt/ros/humble/lib/rclcpp_components/component_container_mt --ros-args -r __node:=apriltag_container -r __ns:=/ --params-file /tmp/launch_params_ph5qnna_'].

``` 

更换成社区版, 然后先设置

```
'pose_estimation_method': '',
``` 

然后使用 ros2 topic echo /detections, 来测试相机是否能识别apriltag

上一步骤成功后, 需要进行相机标定, 才能使用pose_estimation_method, 然后apriltag才会发布 从tag到相机的TF变换

但3D位姿需要自己计算

还是尝试使用isaac_ros_apriltag, 但需要处理图像编码转换: yuv422_yuy2 -> RGB:

  - 增加编码转换后, apriltag识别报错: 

```
[component_container_mt-4] Fatal assertion error
[component_container_mt-4] #0 /opt/nvidia/vpi3/lib/x86_64-linux-gnu/libnvvpi.so.3(+0x436875) [0x7fc60a875875]
[component_container_mt-4] #1 /opt/nvidia/vpi3/lib/x86_64-linux-gnu/libnvvpi.so.3(+0x41bf04) [0x7fc60a85af04]
[component_container_mt-4] #2 /opt/nvidia/vpi3/lib/x86_64-linux-gnu/libnvvpi.so.3(+0x113bd6d) [0x7fc60b57ad6d]
[component_container_mt-4] #3 /opt/nvidia/vpi3/lib/x86_64-linux-gnu/libnvvpi.so.3(+0x113bef9) [0x7fc60b57aef9]

```

    - 认为是因为camera_info中, 对相机的标定都为空, 所以apriltag识别服务报错
    - 先生成一组假的标定数据, 发布到/camera/camera_info_fake 进行测试, 可以成功
  - 进行真实相机标定: 
    - 从[https://wiki.ros.org/camera_calibration/Tutorials/MonocularCalibration?action=AttachFile&do=view&target=check-108.pdf](<https://wiki.ros.org/camera_calibration/Tutorials/MonocularCalibration?action=AttachFile&do=view&target=check-108.pdf>) 下载棋盘格打印
      - 参考 <https://wiki.ros.org/camera_calibration/Tutorials/MonocularCalibration> , 9*7 的棋盘格, 参数应当设置为内部订单数 8*6
    - 标定命令:   

```
ros2 run camera_calibration cameracalibrator \
  --size 8x6 --square 0.024 \
  image:=/camera/rgb_rgb8 camera:=/camera
```

    - 不停移动棋盘, 直到采样数足够, 保存结果
      - 结果举例: [calibrationdata.tar.gz](/assets/01KJC08QZJFY6ZC78Y6TJ7R5VV/calibrationdata.tar.gz)
      - [ost.yaml](/assets/01KJC08QZJFY6ZC78Y6TJ7R5VV/ost.yaml)
    - 将其放在slider中, 作为camera_info发布, 可完成正确运行 apriltag识别
    - commit: 3f37d0930845e7fe1a5fc2ea7cf04de03446bb81

检测到的apriltag的位置, 和实际的位置差异过大, 使用debug_hand_eye.py不停打印坐标, 一方面打印检测到的apriltag的坐标, 另一方面将joint6拖放到tag所在位置, 打印joint6_flange的坐标, 将两个坐标对比: 

```
第一次: 

================================================================================
AprilTag 检测:
   局部坐标 (camera_optical_frame): [+0.080914, -0.002286, +0.257829]
   世界坐标 (world):                [-0.039278, +0.239001, +0.122004]
================================================================================

================================================================================
Flange 位置:
   世界坐标 (world): [-0.098570, +0.231891, +0.061878]
================================================================================

第二次: 

================================================================================
AprilTag 检测:
   局部坐标 (camera_optical_frame): [+0.023700, +0.010736, +0.209222]
   世界坐标 (world):                [-0.185845, +0.140360, +0.027447]
================================================================================

================================================================================
Flange 位置:
   世界坐标 (world): [-0.233112, +0.090893, +0.069256]
================================================================================

``` 

偏差并不稳定, 偏差的总距离 相近 + 偏差向量方向差异巨大 = 典型的旋转误差特征

安装visp_hand2eye_calibration进行手眼标定: 

```
sudo apt install libvisp-dev
 
cd /workspaces/isaac_ros-dev/src
git clone https://github.com/lagadic/vision_visp.git -b rolling
 
cd..
colcon build --packages-up-to vision_visp
 
 
``` 
    
    
    忽略以下内容: 

````
 
cd /workspaces/isaac_ros-dev/install/visp_tracker/share/bag
rosbags-convert --src tutorial-static-box-ros2.bag --dst ./tutorial-static-box-ros2-converted

vi /workspaces/isaac_ros-dev/src/vision_visp/visp_tracker/launch/tutorial_launch.xml
修改: 
```
  <executable cmd="ros2 bag play install/visp_tracker/share/bag/tutorial-static-box-ros2-converted --loop"/>
```
```` 

使用: 

```
mkdir /tmp/unknown
窗口1: 
ros2 run visp_hand2eye_calibration visp_hand2eye_calibration_calibrator
 
窗口2: 
ros2 run visp_hand2eye_calibration visp_hand2eye_calibration_client --ros-args -r reset:=/reset_service
``` 

会发生报错: 

```
admin@10-60-141-108:/workspaces/isaac_ros-dev$ ros2 run visp_hand2eye_calibration visp_hand2eye_calibration_calibrator
1764000347.352427 [98] visp_hand2: config: //CycloneDDS/Domain/General: 'NetworkInterfaceAddress': deprecated element (/workspaces/isaac_ros-dev/a.xml line 9)
[INFO] [1764000350.709005621] [rclcpp]: reseting...
[INFO] [1764000351.710032641] [rclcpp]: computing...

Rotation and translation after VVS
Distance theta between rMo(0) and mean (deg) = 6.75356e-15
Distance theta between rMo(1) and mean (deg) = 1.01949e-14
Distance theta between rMo(2) and mean (deg) = 4.07387e-15
Distance theta between rMo(3) and mean (deg) = 7.99352e-15
Distance theta between rMo(4) and mean (deg) = 8.24477e-15
Distance theta between rMo(5) and mean (deg) = 8.82803e-15
Mean residual rMo(6) - rotation (deg) = 7.91569e-15
Distance d between rMo(0) and mean (m) = 6.39734e-17
Distance d between rMo(1) and mean (m) = 1.28884e-16
Distance d between rMo(2) and mean (m) = 8.58294e-17
Distance d between rMo(3) and mean (m) = 1.03154e-16
Distance d between rMo(4) and mean (m) = 1.16317e-16
Distance d between rMo(5) and mean (m) = 3.23162e-16
Mean residual rMo(6) - translation (m) = 1.61593e-16
Mean residual rMo(6) - global = 1.50332e-16
[INFO] [1764000351.711875195] [rclcpp]: computing 2 values...

 Problem in solving Hand-Eye Rotation by Old Tsai method 

 Problem in solving Hand-Eye Rotation by Tsai method 

 Problem in solving Hand-Eye Rotation by Procrustes method 

 Problem in solving Hand-Eye Calibration by VVS 
^C[INFO] [1764000428.088742252] [rclcpp]: signal_handler(signum=2)
admin@10-60-141-108:/workspaces/isaac_ros-dev$ ros2 run visp_hand2eye_calibration visp_hand2eye_calibration_calibrator
1764000435.754285 [98] visp_hand2: config: //CycloneDDS/Domain/General: 'NetworkInterfaceAddress': deprecated element (/workspaces/isaac_ros-dev/a.xml line 9)
[INFO] [1764000442.168021427] [rclcpp]: reseting...
[INFO] [1764000443.169414876] [rclcpp]: computing...

Rotation and translation after VVS
Distance theta between rMo(0) and mean (deg) = 6.75356e-15
Distance theta between rMo(1) and mean (deg) = 1.01949e-14
Distance theta between rMo(2) and mean (deg) = 4.07387e-15
Distance theta between rMo(3) and mean (deg) = 7.99352e-15
Distance theta between rMo(4) and mean (deg) = 8.24477e-15
Distance theta between rMo(5) and mean (deg) = 8.82803e-15
Mean residual rMo(6) - rotation (deg) = 7.91569e-15
Distance d between rMo(0) and mean (m) = 6.39734e-17
Distance d between rMo(1) and mean (m) = 1.28884e-16
Distance d between rMo(2) and mean (m) = 8.58294e-17
Distance d between rMo(3) and mean (m) = 1.03154e-16
Distance d between rMo(4) and mean (m) = 1.16317e-16
Distance d between rMo(5) and mean (m) = 3.23162e-16
Mean residual rMo(6) - translation (m) = 1.61593e-16
Mean residual rMo(6) - global = 1.50332e-16
[ERROR] [1764000443.171151988] [rclcpp]: transformation vectors have different sizes or contain too few elements

``` 

修改代码: /workspaces/isaac_ros-dev/src/vision_visp/visp_hand2eye_calibration/src/client.cpp

![image2025-11-25 0:21:12.png](/assets/01KJC08QZJFY6ZC78Y6TJ7R5VV/image2025-11-25%200%3A21%3A12.png)

再执行, 可以看到日志: 

```
admin@10-60-141-108:/workspaces/isaac_ros-dev$ ros2 run visp_hand2eye_calibration visp_hand2eye_calibration_calibrator
1764001302.715767 [98] visp_hand2: config: //CycloneDDS/Domain/General: 'NetworkInterfaceAddress': deprecated element (/workspaces/isaac_ros-dev/a.xml line 9)
[INFO] [1764001306.773943462] [rclcpp]: reseting...
[INFO] [1764001313.775800102] [rclcpp]: computing...

Rotation and translation after VVS
Distance theta between rMo(0) and mean (deg) = 6.75356e-15
Distance theta between rMo(1) and mean (deg) = 1.01949e-14
Distance theta between rMo(2) and mean (deg) = 4.07387e-15
Distance theta between rMo(3) and mean (deg) = 7.99352e-15
Distance theta between rMo(4) and mean (deg) = 8.24477e-15
Distance theta between rMo(5) and mean (deg) = 8.82803e-15
Mean residual rMo(6) - rotation (deg) = 7.91569e-15
Distance d between rMo(0) and mean (m) = 6.39734e-17
Distance d between rMo(1) and mean (m) = 1.28884e-16
Distance d between rMo(2) and mean (m) = 8.58294e-17
Distance d between rMo(3) and mean (m) = 1.03154e-16
Distance d between rMo(4) and mean (m) = 1.16317e-16
Distance d between rMo(5) and mean (m) = 3.23162e-16
Mean residual rMo(6) - translation (m) = 1.61593e-16
Mean residual rMo(6) - global = 1.50332e-16
[INFO] [1764001313.777786403] [rclcpp]: computing 6 values...

Rotation and translation after VVS
Distance theta between rMo(0) and mean (deg) = 6.75356e-15
Distance theta between rMo(1) and mean (deg) = 1.01949e-14
Distance theta between rMo(2) and mean (deg) = 4.07387e-15
Distance theta between rMo(3) and mean (deg) = 7.99352e-15
Distance theta between rMo(4) and mean (deg) = 8.24477e-15
Distance theta between rMo(5) and mean (deg) = 8.82803e-15
Mean residual rMo(6) - rotation (deg) = 7.91569e-15
Distance d between rMo(0) and mean (m) = 6.39734e-17
Distance d between rMo(1) and mean (m) = 1.28884e-16
Distance d between rMo(2) and mean (m) = 8.58294e-17
Distance d between rMo(3) and mean (m) = 1.03154e-16
Distance d between rMo(4) and mean (m) = 1.16317e-16
Distance d between rMo(5) and mean (m) = 3.23162e-16
Mean residual rMo(6) - translation (m) = 1.61593e-16
Mean residual rMo(6) - global = 1.50332e-16

```
```
admin@10-60-141-108:/workspaces/isaac_ros-dev$ ros2 run visp_hand2eye_calibration visp_hand2eye_calibration_client --ros-args -r reset:=/reset_service
1764001305.765789 [98] visp_hand2: config: //CycloneDDS/Domain/General: 'NetworkInterfaceAddress': deprecated element (/workspaces/isaac_ros-dev/a.xml line 9)
[INFO] [1764001305.773438676] [rclcpp]: Waiting for topics...
[INFO] [1764001306.774305881] [rclcpp]: 1) GROUND TRUTH:
[INFO] [1764001313.775244776] [rclcpp]: 2) QUICK SERVICE:
[INFO] [1764001313.777619804] [rclcpp]: hand_camera: 

[INFO] [1764001313.777643674] [rclcpp]: 3) TOPIC STREAM:
[INFO] [1764001313.779103528] [rclcpp]: hand_camera: 

``` 

client没有具体数值的原因, 是源码注释掉了这部分: 

![image2025-11-25 0:27:27.png](/assets/01KJC08QZJFY6ZC78Y6TJ7R5VV/image2025-11-25%200%3A27%3A27.png)

effector_camera的类型如何字符串化, 需要调研, 暂不处理. 通过ros2命令获取结果: 

```
admin@10-60-141-108:/workspaces/isaac_ros-dev$ ros2 service call /compute_effector_camera visp_hand2eye_calibration/srv/ComputeEffectorCamera
requester: making request: visp_hand2eye_calibration.srv.ComputeEffectorCamera_Request()

response:
visp_hand2eye_calibration.srv.ComputeEffectorCamera_Response(effector_camera=geometry_msgs.msg.Transform(translation=geometry_msgs.msg.Vector3(x=0.10000000000000002, y=0.20000000000000015, z=0.3), rotation=geometry_msgs.msg.Quaternion(x=0.08635554218827568, y=-0.0863555421882755, z=0.21588885547068906, w=0.9687504543226257)))
``` 

数值与文档中相同: <https://wiki.ros.org/visp_hand2eye_calibration>

对标定的理解: 

```
我理解一下标定的流程: 
1. 固定apriltag的位置
2. 不停移动机械臂 (实际上是使用不同的相机的位姿), 将位姿 和 检测到的tag的位姿采集下来
3. 通过这些位姿的对应数据, 来计算 相机的一个修正矩阵, 相当于对相机的位姿进行一个变换, 在这个变换下, 由相机计算出的tag位置更接近于现在的事实
``` 

准备进行实际标定, 先分析识别路径: 

识别路径:

\- 机械臂上, 在 @mycobot_280_m5_adaptive_gripper_ab.urdf 中定义了 camera_flange

\- 在 @slider_control_adaptive_gripper.fix_seq.py 中, 定义了 camera_flange -> camera_link 的变化

\- 在 @slider_control_adaptive_gripper.fix_seq.py 中, 发布了 camera_link -> camera_optical_frame 的变化

\- 在 @test_real.py 中, 识别了apriltag, 强制使用camera_optical_frame坐标系 (不要质疑这一步, 这一步经过严格的测试)

步骤: 

  1. 确定标定的目标: 求得 从 camera_flange 到 camera_optical_frame 的映射. 
     1. 将各个中间矩阵重置, 并且在标定前, 需要将 camera_tf_rotation/camera_tf_translation 重置
     2. 使用145mm的tag进行标定
     3. 标定脚本 (使用opencv算法) 为 collect_calibration_data.py, 使用方法: 

```
python collect_calibration_data.py
# 不断调整摄像头位置, 进行采集
 
# 将输出结果存入collect_calibration_data.log, 然后用以下命令进行分析

python collect_calibration_data.py --calibrate collect_calibration_data.log

...
============================================================
 标定结果 (方法: TSAI)
============================================================

effector -> camera 变换:
   平移 (xyz): [0.032619, 0.099084, 0.007167]
   旋转 (xyzw): [-0.690836, 0.033444, 0.037600, 0.721258]
   旋转 (RPY度): [-87.51, 5.75, 0.46]

可复制到 slider_control_adaptive_gripper.fix_seq.py 的参数:
   camera_tf_translation: [0.03261888128857074, 0.09908350559950316, 0.00716661984637855]
   camera_tf_rotation: [-0.6908360122396046, 0.033443512316134794, 0.03759976817780211, 0.7212582014159818]

camera -> effector 变换 (逆变换):
   平移 (xyz): [-0.032531, 0.006173, -0.099179]
   旋转 (xyzw): [0.690836, -0.033444, -0.037600, 0.721258]

============================================================
提示:
   - 如果结果不理想，尝试使用 --try-all 比较不同算法
   - 确保采集时机械臂有足够的旋转变化
   - 建议采集 10-20 个样本，覆盖不同姿态
============================================================

admin@10-60-141-108:/workspaces/isaac_ros-dev/huangyan/moveit_and_isaac_sim$
```

     4. 将得到的位置放到 slider中的 camera_tf_translation/camera_tf_rotation 中
     5. 但测量的结果, 和实际的结果有偏差, 看上去是围绕机械臂的z轴偏转了90度
  2. 尝试直接标定 joint6_flange 到 camera_optical_frame, ( 怀疑joint6 到 camera_flange 的数据不准), 然后发布camera_optical_frame也通过joint6_flange进行
     1. 发现camera_optical_frame偏离很大
        1. ![image2025-11-27 19:25:27.png](/assets/01KJC08QZJFY6ZC78Y6TJ7R5VV/image2025-11-27%2019%3A25%3A27.png)
     2. 因为usbcam启动时, 配置了 camera_frame_id:=camera_link, 所以得保留camera_link  

        1. 怀疑是: usbcam 将图像放到 camera_link 中, 但图像的光学性质是 camera_optical_frame, 这是usbcam内置的行为
        2. 外部使用时, 也需要尊重这个行为, 否则会有错乱
        3. 重新标定后, 大致方位正确了, 但距离仍有问题: 
           1. ![image2025-11-27 19:39:11.png](/assets/01KJC08QZJFY6ZC78Y6TJ7R5VV/image2025-11-27%2019%3A39%3A11.png)
        4. 经验: camera_link -> camera_optical_frame, 在ROS相机中的默认规则, 是必须遵守的行为
        5. 发现: 
           1. 标定 joint6_flange 到 camera_optical_frame, 但将标志数值应用于 joint6_flange -> camera_link, 方向是准的
           2. 标定 joint6_flange 到 camera_link, 将标志数值应用于 joint6_flange -> camera_link, 方向不太准
        6. 在采集标定数据的脚本中, 订阅的是 /tag_detections, 其应当是始终标定光学坐标系, 所以获取了joint6_flange -> camera_optical_frame  

           1. 为了保持 camera_link到camera_optical_frame的语义, 应当将获取的标定数据, 再转换会 joint6_flange -> camera_link
  3. 将slider中, usbcam的frame_id设置为camera_optical_frame, 在rviz中可以看到识别出的tag, 发现重新标定后, 识别出的tag位置是正确的. 但运行test_real.py时, 抓取位姿错误  

     1. 修正test_real.py中的偏移设定, 恢复正常. 
     2. commit: 9a06e67aff120e3a527929123923659766716b12

# 额外 - 使用rviz检查URDF

```
sudo apt install ros-humble-urdf-tutorial 

ros2 launch urdf_tutorial display.launch.py model:=/workspaces/isaac_ros-dev/huangyan/mycobot_280_m5_urdf/mycobot_description/urdf/mycobot_280_m5/mycobot_280_m5_adaptive_gripper_ab.urdf

``` 

# 总结

commit: 68d652db74c4117796327e7144c90005cfc80c67

总结经验: 

  - 对于相机的使用, 需要进行内标定和手眼标定
    - 内标定: 使用 ros2 run camera_calibration cameracalibrator ... 命令进行标定, 没有太大争议, 注意棋盘格的描述参数即可 (参数是棋盘格的交叉点个数)
      - 结果放到 _embedded_camera_info_yaml 常量中
    - 手眼标定: 使用 collect_calibration_data.py 进行手眼标定 (先采集数据, 然后调用opencv对应的库进行计算)
  - 手眼标定碰到很多问题  

    - 需要使用足够大的Apriltag进行标定, 太小的Apriltag, 导致机械臂移动范围受限, 标定不准
    - 注意标定的坐标系: camera_link (相机物理坐标系) 到 camera_optical_frame (相机光学坐标系), 这两个坐标系之间的转换关系是 usbcam 中内定的 (行业约定?), 所以外面使用的地方都使用这个内定关系, 否则会匹配有误
      - 转换常量: CAMERA_LINK_TO_OPTICAL_QUATERNION/CAMERA_LINK_TO_OPTICAL_TRANSLATION
    - 标定的范围是从 joint6_flange 到 camera_optical_frame
    - usbcam中设置的frame_id 也是 camera_optical_frame, 这样其发布的消息的frame_id正确, 在rviz中能正确显示
    - 对标定结果的使用: 用标定结果 (乘以 内定关系的逆), 推算出 joint6_flange 到 camera_link 的转换, 进行发布
    - 注意: 在rviz中观察tag的位置, 其与实际情况匹配, 说明标定正确
      - (在此坑了很久, 主要是并没有分步诊断, 而是使用最终移动的结果来判断标定是否正确, 忽视了移动中的偏移设置)
  - 移动的偏移: 
    - 是以 joint6_flange 或者 gripper_base 的坐标系来生成偏移, 这样偏移比较稳定
  - 现在的效果: 
    - 可以正确移动并抓取麻将牌
  - 现在的问题: 
    - 有时并不能抓取麻将牌
      - 现在Apriltag是单目相机识别, 晚上需要补光才能识别出来, 导致对Apriltag的位置识别可能有偏差
      - 夹爪接近tag的角度不同 (更倾向于是直线接近), 会导致有一定角度偏差, 导致位置偏差
      - 机械臂移动后, 会有轻微漂移 (臂力量不稳)
    - 只有一次, 机械臂移动时, 忽略了相机的尺寸, 让相机和臂碰撞且不能解
