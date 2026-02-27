---
title: 20251209 - 调试双目摄像头
confluence_page_id: 4359429
created_at: 2025-12-09T11:19:54+00:00
updated_at: 2025-12-17T10:14:50+00:00
---

接入usb

使用命令预览摄像头图像: 

```
ffplay -f v4l2 -framerate 30 -video_size 1280x480 -i /dev/video3
``` 

效果: 

![image2025-12-9 16:43:16.png](/assets/01KJC09C6KBEC83Z6P9EX5Y4F9/image2025-12-9%2016%3A43%3A16.png)

查看能力: 

```
huangyan@bot:/data/huangyan/isaac_ros-dev$ v4l2-ctl -d /dev/video3 --all
Driver Info:
	Driver name      : uvcvideo
	Card type        : USB Camera: USB Camera
	Bus info         : usb-0000:00:14.0-4
	Driver version   : 5.15.189
	Capabilities     : 0x84a00001
		Video Capture
		Metadata Capture
		Streaming
		Extended Pix Format
		Device Capabilities
	Device Caps      : 0x04200001
		Video Capture
		Streaming
		Extended Pix Format
Media Driver Info:
	Driver name      : uvcvideo
	Model            : USB Camera: USB Camera
	Serial           : 01.00.00
	Bus info         : usb-0000:00:14.0-4
	Media version    : 5.15.189
	Hardware revision: 0x00001029 (4137)
	Driver version   : 5.15.189
Interface Info:
	ID               : 0x03000002
	Type             : V4L Video
Entity Info:
	ID               : 0x00000001 (1)
	Name             : USB Camera: USB Camera
	Function         : V4L2 I/O
	Flags            : default
	Pad 0x01000007   : 0: Sink
	  Link 0x02000013: from remote pad 0x100000a of entity 'Extension 4' (Video Pixel Formatter): Data, Enabled, Immutable
Priority: 2
Video input : 0 (Input 1: ok)
Format Video Capture:
	Width/Height      : 1280/480
	Pixel Format      : 'YUYV' (YUYV 4:2:2)
	Field             : None
	Bytes per Line    : 2560
	Size Image        : 1228800
	Colorspace        : sRGB
	Transfer Function : Rec. 709
	YCbCr/HSV Encoding: ITU-R 601
	Quantization      : Default (maps to Limited Range)
	Flags             :
Crop Capability Video Capture:
	Bounds      : Left 0, Top 0, Width 1280, Height 480
	Default     : Left 0, Top 0, Width 1280, Height 480
	Pixel Aspect: 1/1
Selection Video Capture: crop_default, Left 0, Top 0, Width 1280, Height 480, Flags:
Selection Video Capture: crop_bounds, Left 0, Top 0, Width 1280, Height 480, Flags:
Streaming Parameters Video Capture:
	Capabilities     : timeperframe
	Frames per second: 10.000 (10/1)
	Read buffers     : 0

User Controls

                     brightness 0x00980900 (int)    : min=-64 max=64 step=1 default=0 value=0
                       contrast 0x00980901 (int)    : min=0 max=95 step=1 default=0 value=0
                     saturation 0x00980902 (int)    : min=0 max=100 step=1 default=64 value=64
                            hue 0x00980903 (int)    : min=-2000 max=2000 step=1 default=0 value=0
        white_balance_automatic 0x0098090c (bool)   : default=1 value=1
                          gamma 0x00980910 (int)    : min=100 max=300 step=1 default=100 value=100
           power_line_frequency 0x00980918 (menu)   : min=0 max=2 default=2 value=2 (60 Hz)
				0: Disabled
				1: 50 Hz
				2: 60 Hz
      white_balance_temperature 0x0098091a (int)    : min=2800 max=6500 step=1 default=4600 value=4600 flags=inactive
                      sharpness 0x0098091b (int)    : min=1 max=7 step=1 default=2 value=2
         backlight_compensation 0x0098091c (int)    : min=0 max=1 step=1 default=1 value=1

Camera Controls

                  auto_exposure 0x009a0901 (menu)   : min=0 max=3 default=3 value=3 (Aperture Priority Mode)
				1: Manual Mode
				3: Aperture Priority Mode
         exposure_time_absolute 0x009a0902 (int)    : min=3 max=2047 step=1 default=166 value=166 flags=inactive
huangyan@bot:/data/huangyan/isaac_ros-dev$
``` 

主要信息: 

  - Width/Height : 1280/480
  - Pixel Format : 'YUYV' (YUYV 4:2:2)

# 标定

先将相机接入ros2: 

```
sudo apt install ros-humble-v4l2-camera
``` 

运行: 

```
ros2 run v4l2_camera v4l2_camera_node   --ros-args -p video_device:=/dev/video3 -p image_size:="[1280,480]"
``` 

会报错: 

```
Failed mapping device memory
``` 

通过fuser查看谁占用了摄像头: 

```
sudo fuser -v /dev/video3
``` 

清理后, 可以正常启动.

查看ros2图像: 

```
sudo apt install ros-humble-image-tools
ros2 run image_tools showimage --ros-args -r image:=/image_raw
``` 

![image2025-12-9 19:44:2.png](/assets/01KJC09C6KBEC83Z6P9EX5Y4F9/image2025-12-9%2019%3A44%3A2.png)

这个1280 * 480, 是两个640 * 480 拼接而成, 需要切分开

切分脚本: stero_splitter.py

查看切分结果: 

```
ros2 run image_tools showimage --ros-args -r image:=/stereo/left/image_raw
ros2 run image_tools showimage --ros-args -r image:=/stereo/right/image_raw
``` 

查看都没问题

双眼标定: 

```
ros2 run camera_calibration cameracalibrator \
  --approximate 0.1 \
  --size 8x6 \
  --square 0.024 \
  right:=/stereo/right/image_raw \
  left:=/stereo/left/image_raw \
  right_camera:=/stereo/right \
  left_camera:=/stereo/left
``` 

![image2025-12-9 19:59:10.png](/assets/01KJC09C6KBEC83Z6P9EX5Y4F9/image2025-12-9%2019%3A59%3A10.png)

标定数据: 放在切分脚本中

简单总结: 

  - 需要用 ros2 run v4l2_camera v4l2_camera_node 将相机接入ros2
  - 需要用分割脚本, 将视频流切成两个左右流
  - 用两个流进行双眼标定

# 生成深度图

重新启动环境的过程: 

```
窗口1: 将摄像头接入ros2体系
 
ros2 run v4l2_camera v4l2_camera_node   --ros-args -p video_device:=/dev/video3 -p image_size:="[1280,480]"
 
窗口2: 对左右眼图像进行分割
python StereoSplitter.py
 
 
窗口3: 图像校正 (左)
sudo apt install ros-humble-image-pipeline
ros2 run image_proc image_proc   --ros-args   \
-r __ns:=/stereo/left   \
-r image_raw:=image_raw   \
-r image:=image_color   \
-r camera_info:=camera_info   \
-r image_rect:=image_rect   \
-r image_rect_color:=image_rect_color
 
窗口4: 图像校正 (右)
ros2 run image_proc image_proc   --ros-args   \
-r __ns:=/stereo/right   \
-r image_raw:=image_raw   \
-r image:=image_color   \
-r camera_info:=camera_info   \
-r image_rect:=image_rect   \
-r image_rect_color:=image_rect_color
 
窗口5: 深度计算
ros2 run stereo_image_proc disparity_node   --ros-args   \
-r left/image_rect:=/stereo/left/image_rect   \
-r right/image_rect:=/stereo/right/image_rect   \
-r left/camera_info:=/stereo/left/camera_info   \
-r right/camera_info:=/stereo/right/camera_info
 
# 检查输出话题: 
ros2 topic echo /disparity

 
窗口6: 点云计算
ros2 run stereo_image_proc point_cloud_node --ros-args \
-r left/camera_info:=/stereo/left/camera_info \
-r right/camera_info:=/stereo/right/camera_info \
-r left/image_rect_color:=/stereo/left/image_rect \
-r disparity:=/disparity
 
# 检查输出话题
ros2 topic echo /points2
 
窗口7: TF发布: 
ros2 run tf2_ros static_transform_publisher 0 0 1 -1.57 0 -1.57 world camera
 
窗口8: rviz:
 
``` 

在 窗口3: 图像校正, 结果为全黑, 判定是标定参数有误 (gemini-3-pro能对标定结果做判断: <https://poe.com/s/5YMsB1uAbZzyg0Lt7RPi>)

重新生成标定参数, 看上去正常了 (需要移动棋盘的远近)

在rviz中显示点云: 

![image2025-12-10 11:28:6.png](/assets/01KJC09C6KBEC83Z6P9EX5Y4F9/image2025-12-10%2011%3A28%3A6.png)

深度比较正确了. 但能识别的物体都在2m左右.

将移液枪放在50cm左右, 手工调整摄像头焦距, 使其清晰. 然后重新标定.

commit: ce1e65fe9619286c1ca28723b7fd652ca69bd127

# 使用模板匹配

获取模板图:

```
ros2 run rqt_image_view rqt_image_view
``` 

进行裁剪

使用 [pic_picker.html](/assets/01KJC09C6KBEC83Z6P9EX5Y4F9/pic_picker.html) , 选择移液器头/握持位/尾部的坐标: 

  - 62/15

  - 62/110

  - 62/435

使用模板匹配: pipette_template_matcher.py

发现: ![image2025-12-11 19:11:11.png](/assets/01KJC09C6KBEC83Z6P9EX5Y4F9/image2025-12-11%2019%3A11%3A11.png)

  - 对于移液器, 遮住白色的部分, 其匹配度也很高 0.65. 不遮住, 也是0.65
  - 在白色背景下, 稍微倾斜, 匹配度就会降低为 0.35
  - 去掉白色背景, 不倾斜, 匹配度就会降低为 0.35

模板方法的适应度很低, 需要想其他办法

# 重新设计slider/moveit方案

让AI重写slider和moveit, 融合机械臂/机械手/双目摄像头的调度

测试slider: 

```
# 先测试摄像头
 
python slider/slider.launch.py --disable-all
 
# 可以查看到图像
ros2 run rqt_image_view rqt_image_view 
 
# 再测试机械臂
sudo chmod 777 /dev/ttyACM0
python slider/slider.launch.py --disable-linkerhand
 
# 机械臂可以归为
ros2 action send_goal /arm_group_controller/follow_joint_trajectory control_msgs/action/FollowJointTrajectory "{trajectory: {joint_names: ['joint2_to_joint1','joint3_to_joint2','joint4_to_joint3','joint5_to_joint4','joint6_to_joint5','joint6output_to_joint6'], points: [{positions: [0.0,0.0,0.0,0.0,0.0,0.0], time_from_start: {sec: 5, nanosec: 0}}]}}"
 
# 最后测试机械手
ip -details link show type can # 查看can0

sudo ip link set can0 up type can bitrate 1000000 #开启can0
 
cansend can0 027#01969696969696 # 简单动作
 
python slider/slider.launch.py # 启动slider
 
ros2 action send_goal /linkerhand_controller/follow_joint_trajectory control_msgs/action/FollowJointTrajectory \
"{trajectory: {joint_names: ['thumb_cmc_pitch','thumb_cmc_yaw','index_mcp_pitch','middle_mcp_pitch','ring_mcp_pitch','pinky_mcp_pitch'], points: [{positions: [0.3,0.6,0.8,0.8,0.8,0.8], time_from_start: {sec: 2, nanosec: 0}}]}}" \
--feedback
``` 

测试完成, commit: da3ff9d511508381d36ab840cbd017ce1ce35b26

测试slider的Apriltag功能

  - python slider/slider.launch.py --enable-apriltag --disable-all
  - 因为slider机器上没有显卡, 所以从 isaac_ros_apriltag切换到官方apriltag
  - 需要注意: 官方apriltag的 /tag_detections 消息中, 没有三维坐标, 所以需要从TF中获取, 方案: 
    - 方案 A（最推荐）：用检测作为触发，用它的时间戳查 TF  
当 /tag_detections 来一条消息时，取 msg.header.stamp  
用这个时间戳去 lookup_transform(world, tag_frame, stamp)

  - commit: 38f4c24d7ca9e42994c764cfe793f3fb6f2f03de

调试moveit: 

  - 需要将URDF/SRDF/slider中的 灵巧手的关节名统一
  - 对于mimic关节, 需要在slider中计算其数值, 并发布到joint_states中, 而不依赖于mimic机制
  - 花了很多时间, 处理 rviz中, 显示不出调度用的marker, 最终的结论: rviz需要放在脚本中启动, 脚本中会传入各个配置文件
  - commit: ee45c9304f9c59dc42c8f583ab70e24257627c17

进行手眼标定: 

```
# 使用50mm的tag, 贴在机械手的背侧, 进行标定
 
# 启动slider
python slider/slider.launch.py --enable-apriltag --apriltag-size 0.05 --disable-linkerhand
 
# 启动 moveit
python3 moveit/moveit.launch.py --use-rviz
 
# 松开关节
ros2 service call /slider_hw_stack/release_all_servos std_srvs/srv/Trigger "{}"
 
# 查看tag_detections
ros2 topic echo tag_detections
 
# 记录点位
rm -f eye_to_hand.log; python3 huangyan/moveit_and_isaac_sim/linkerhand_and_stereo_camera/collect_eye_to_hand_calibration.py   --base-frame g_base   --flange-frame joint6_flange   --camera-frame camera   --tag-frame "tag36h11:6" |& tee eye_to_hand.log
 
# 计算结果
python3 huangyan/moveit_and_isaac_sim/linkerhand_and_stereo_camera/collect_eye_to_hand_calibration.py   --calibrate eye_to_hand.log   --base-frame g_base   --flange-frame joint6_flange   --camera-frame camera
 
 
``` 

让tag完全不动, 采集静止姿态的结果, 误差比较小, 比较稳定: 

```
======================================================================
 Eye-to-Hand 标定结果（外置相机）
======================================================================
 输出静态TF: g_base -> camera
  平移 xyz: [-0.151387, 0.729697, -0.181173]
  旋转 xyzw: [0.171410, -0.524213, 0.800281, -0.235306]
  旋转 RPY(deg): [-66.93, -1.58, -146.18]

 一致性误差（每帧回代的 base->camera 相对平均值）
  平移误差: mean=0.81 mm, std=0.45 mm
  旋转误差: mean=0.279 deg, std=0.143 deg

 离群值剔除敏感性（按综合评分排序，剔除最差比例后重新统计）
  说明：综合评分 = trans_err_mm + rot_err_deg*10（1deg≈10mm，仅用于排序）
  - 去掉最差  0%（保留 21/21）: 平移 mean=0.81mm std=0.45mm, 旋转 mean=0.279deg std=0.143deg
  - 去掉最差  5%（保留 20/21）: 平移 mean=0.74mm std=0.49mm, 旋转 mean=0.258deg std=0.154deg
  - 去掉最差 10%（保留 19/21）: 平移 mean=0.66mm std=0.51mm, 旋转 mean=0.230deg std=0.160deg
  - 去掉最差 20%（保留 17/21）: 平移 mean=0.42mm std=0.49mm, 旋转 mean=0.162deg std=0.148deg
  - 去掉最差 30%（保留 15/21）: 平移 mean=0.28mm std=0.39mm, 旋转 mean=0.122deg std=0.116deg
  - 去掉最差 40%（保留 13/21）: 平移 mean=0.14mm std=0.12mm, 旋转 mean=0.083deg std=0.027deg
  - 去掉最差 50%（保留 11/21）: 平移 mean=0.12mm std=0.10mm, 旋转 mean=0.081deg std=0.025deg

 可直接复制执行：
  ros2 run tf2_ros static_transform_publisher -0.1513867192762065 0.7296971135832212 -0.18117320919597155 0.17141019288087506 -0.5242128821218952 0.8002814620641969 -0.23530614414075252 g_base camera
======================================================================
``` 

移动tag, 进行两次采样, 结果如下, 平移偏差比较大

```
admin@10-60-141-108:/workspaces/isaac_ros-dev$ python3 huangyan/moveit_and_isaac_sim/linkerhand_and_stereo_camera/collect_eye_to_hand_calibration.py   --calibrate eye_to_hand.log   --base-frame g_base   --flange-frame joint6_flange   --camera-frame camera

======================================================================
 Eye-to-Hand 标定结果（外置相机）
======================================================================
 输出静态TF: g_base -> camera
  平移 xyz: [-0.057631, -0.116447, 0.031682]
  旋转 xyzw: [0.348393, 0.691332, 0.580158, 0.253177]
  旋转 RPY(deg): [101.47, -3.11, 129.05]

 一致性误差（每帧回代的 base->camera 相对平均值）
  平移误差: mean=61.97 mm, std=29.08 mm
  旋转误差: mean=18.376 deg, std=12.849 deg

 离群值剔除敏感性（按综合评分排序，剔除最差比例后重新统计）
  说明：综合评分 = trans_err_mm + rot_err_deg*10（1deg≈10mm，仅用于排序）
  - 去掉最差  0%（保留 19/19）: 平移 mean=61.97mm std=29.08mm, 旋转 mean=18.376deg std=12.849deg
  - 去掉最差  5%（保留 19/19）: 平移 mean=61.97mm std=29.08mm, 旋转 mean=18.376deg std=12.849deg
  - 去掉最差 10%（保留 18/19）: 平移 mean=60.11mm std=27.15mm, 旋转 mean=15.781deg std=11.446deg
  - 去掉最差 20%（保留 16/19）: 平移 mean=55.77mm std=24.61mm, 旋转 mean=12.159deg std=8.754deg
  - 去掉最差 30%（保留 14/19）: 平移 mean=55.31mm std=24.94mm, 旋转 mean=9.768deg std=5.559deg
  - 去掉最差 40%（保留 12/19）: 平移 mean=51.47mm std=23.14mm, 旋转 mean=8.058deg std=4.086deg
  - 去掉最差 50%（保留 10/19）: 平移 mean=45.46mm std=19.47mm, 旋转 mean=7.007deg std=3.324deg

 可直接复制执行：
  ros2 run tf2_ros static_transform_publisher -0.05763089453419537 -0.11644679887598353 0.03168249406921706 0.34839330645470196 0.6913319975734916 0.580158153982487 0.2531771899603882 g_base camera
======================================================================

---

======================================================================
 Eye-to-Hand 标定结果（外置相机）
======================================================================
 输出静态TF: g_base -> camera
  平移 xyz: [-0.079677, -0.169097, 0.050531]
  旋转 xyzw: [0.366441, 0.620612, 0.689541, 0.071379]
  旋转 RPY(deg): [92.45, -24.63, 142.51]

 一致性误差（每帧回代的 base->camera 相对平均值）
  平移误差: mean=90.22 mm, std=35.89 mm
  旋转误差: mean=19.441 deg, std=10.305 deg

 离群值剔除敏感性（按综合评分排序，剔除最差比例后重新统计）
  说明：综合评分 = trans_err_mm + rot_err_deg*10（1deg≈10mm，仅用于排序）
  - 去掉最差  0%（保留 28/28）: 平移 mean=90.22mm std=35.89mm, 旋转 mean=19.441deg std=10.305deg
  - 去掉最差  5%（保留 27/28）: 平移 mean=88.60mm std=36.16mm, 旋转 mean=18.416deg std=9.652deg
  - 去掉最差 10%（保留 26/28）: 平移 mean=86.42mm std=37.03mm, 旋转 mean=17.255deg std=8.562deg
  - 去掉最差 20%（保留 23/28）: 平移 mean=81.10mm std=33.04mm, 旋转 mean=15.174deg std=4.849deg
  - 去掉最差 30%（保留 20/28）: 平移 mean=74.71mm std=30.94mm, 旋转 mean=13.950deg std=4.872deg
  - 去掉最差 40%（保留 17/28）: 平移 mean=75.23mm std=31.61mm, 旋转 mean=12.164deg std=4.636deg
  - 去掉最差 50%（保留 14/28）: 平移 mean=61.69mm std=22.87mm, 旋转 mean=11.047deg std=5.113deg

 可直接复制执行：
  ros2 run tf2_ros static_transform_publisher -0.07967671050448015 -0.16909657404321263 0.05053131333827949 0.3664413022345084 0.6206120035710933 0.6895408499516693 0.07137877337192422 g_base camera
======================================================================

``` 

发现手眼标定偏差很大, 进行逐步排查: 

先检查 手眼标定得出的光学坐标系: 

```
## 1. 光学坐标系正确性验证方法 (无需代码)

此方法用于确认底层的 `v4l2_camera` 驱动与 `apriltag_ros` 算法生成的坐标是否符合 ROS 标准光学坐标系定义 (**X-右, Y-下, Z-前**)。

**步骤：**
1. **环境准备**：启动系统 `slider.launch.py`，将相机固定，AprilTag 标签竖直放置（确保标签正立且在相机视野内）。
2. **可视化**：启动 `rviz2`，设置 `Fixed Frame` 为 `camera` (或 `camera_optical_frame`)。
3. **观察**：订阅 `/tag_detections` 或显示 TF。
4. **验证动作**（移动 Tag 标签，从相机镜头视角描述方向）：
   - **Z轴检查**：`camera` 的 Z 轴（蓝色）应指向标签方向。
   - **X轴检查 (水平移动)**：
     - 站在相机后方，从镜头向外看，将标签向 **右侧** 移动。
     - 在 RViz 中，标签应向相机的 X 轴 **正** 方向移动。
   - **Y轴检查 (垂直移动)**：
     - 站在相机后方，从镜头向外看，将标签向 **上方** 移动（远离桌面）。
     - 在 RViz 中，标签应向相机的 Y 轴 **负** 方向移动。

**判定**：若上述移动方向一致，则证明底层的"像素->光学坐标"映射符合标准。

**注意**：光学坐标系 Y 轴向下，因此物理上向上移动对应 Y 坐标减小（负方向）。

``` 

经过检查, 手眼标定后的camera坐标系, 是符合光学坐标系特征的

修改代码, 让坐标系命名统一成: camera_link和camera_optical, 并明确 标定的产出为 camera_link.

修改代码, 从/tag_detections中获得更新时间戳后, 等待TF进行更新, 拿到3D位姿 (这一步很重要, 明确可以看到 两者间更新会有明确的延迟)

更新标定命令: 

```
# ==========================================================
# 步骤 1: 交互式采集点位
# ==========================================================
# 说明：
# 1. 启动 slider.launch.py (确保它发布了 camera_optical)
# 2. 运行此命令，移动机械臂并在不同位置按 Enter 记录
# 3. camera-frame 必须改为 "camera_optical"
# ==========================================================
rm -f eye_to_hand.log
python3 huangyan/moveit_and_isaac_sim/linkerhand_and_stereo_camera/collect_eye_to_hand_calibration.py \
  --base-frame g_base \
  --flange-frame joint6_flange \
  --camera-frame camera_optical \
  --tag-frame "tag36h11:6" \
  2>&1 | tee eye_to_hand.log

# ==========================================================
# 步骤 2: 离线计算标定结果
# ==========================================================
# 说明：
# 计算脚本将输出两组结果：
# 1. Base -> Camera Optical (直接用于验证)
# 2. Base -> Camera Link (用于填入 slider.launch.py)
# ==========================================================

python3 huangyan/moveit_and_isaac_sim/linkerhand_and_stereo_camera/collect_eye_to_hand_calibration.py \
  --calibrate eye_to_hand.log \
  --base-frame g_base \
  --flange-frame joint6_flange \
  --camera-frame camera_optical

``` 

结果: 误差变得很小, 但位置感觉差很多

```
admin@10-60-141-108:/workspaces/isaac_ros-dev$ python3 huangyan/moveit_and_isaac_sim/linkerhand_and_stereo_camera/collect_eye_to_hand_calibration.py \
  --calibrate eye_to_hand.log \
  --base-frame g_base \
  --flange-frame joint6_flange \
  --camera-frame camera_optical

======================================================================
 Eye-to-Hand 标定结果（外置相机）
======================================================================
 输出静态TF: g_base -> camera_optical
  平移 xyz: [-0.029513, -0.047371, 0.093512]
  旋转 xyzw: [0.292632, 0.610399, 0.671565, 0.301297]
  旋转 RPY(deg): [85.21, -1.45, 130.34]

 一致性误差（每帧回代的 base->camera 相对平均值）
  平移误差: mean=18.46 mm, std=7.93 mm
  旋转误差: mean=4.390 deg, std=2.052 deg

 离群值剔除敏感性（按综合评分排序，剔除最差比例后重新统计）
  说明：综合评分 = trans_err_mm + rot_err_deg*10（1deg≈10mm，仅用于排序）
  - 去掉最差  0%（保留 12/12）: 平移 mean=18.46mm std=7.93mm, 旋转 mean=4.390deg std=2.052deg
  - 去掉最差  5%（保留 12/12）: 平移 mean=18.46mm std=7.93mm, 旋转 mean=4.390deg std=2.052deg
  - 去掉最差 10%（保留 11/12）: 平移 mean=18.27mm std=7.95mm, 旋转 mean=4.035deg std=1.922deg
  - 去掉最差 20%（保留 10/12）: 平移 mean=16.68mm std=5.34mm, 旋转 mean=3.790deg std=1.824deg
  - 去掉最差 30%（保留 9/12）: 平移 mean=17.38mm std=5.13mm, 旋转 mean=3.531deg std=1.725deg
  - 去掉最差 40%（保留 8/12）: 平移 mean=17.77mm std=4.85mm, 旋转 mean=3.353deg std=1.832deg
  - 去掉最差 50%（保留 6/12）: 平移 mean=19.50mm std=3.83mm, 旋转 mean=2.757deg std=1.598deg

 可直接复制执行（Base -> Camera Optical）：
  ros2 run tf2_ros static_transform_publisher -0.029512546784313257 -0.04737050769335483 0.09351188674062372 0.29263203728236786 0.6103988666669531 0.6715650981646178 0.30129725065924073 g_base camera_optical

======================================================================
  物理坐标标定结果 (Base -> Camera Link)
======================================================================
  计算说明：Base->Link = (Base->Optical) * (Optical->Link)
  其中 Optical->Link 旋转为 quat=[-0.5, 0.5, -0.5, 0.5] (ROS标准)

 物理输出静态TF: g_base -> camera_link
  平移 xyz: [-0.029513, -0.047371, 0.093512]
  旋转 xyzw: [0.645315, -0.266382, -0.636649, -0.327548]
  旋转 RPY(deg): [-73.21, 84.99, 57.08]

 用于 slider.launch.py 的静态 TF 参数 (x y z qx qy qz qw):
"-0.02951255",
"-0.04737051",
"0.09351189",
"0.64531459",
"-0.26638153",
"-0.63664938",
"-0.32754776"
======================================================================
``` 

增加输出, 求解flange -> tag的关系, 发现两者位置偏离很远: 

```
同时输出求解到的固定关系（用于诊断）：flange -> tag
  平移 xyz: [0.442789, 0.403699, 0.159648]  |  |t|=0.620 m
  旋转 xyzw: [-0.230723, -0.634268, 0.715668, 0.179695]
  旋转 RPY(deg): [-84.87, 5.87, 146.44]
``` 

由AI判断, 这通常意味着TF方向取反了, 对采样脚本的方向进行修正: 

  - ![image2025-12-16 13:6:22.png](/assets/01KJC09C6KBEC83Z6P9EX5Y4F9/image2025-12-16%2013%3A6%3A22.png)

用少数点重新采样的结果输出, 看上去 flange -> tag的距离已经正确, g_base -> camera_link的偏移也恢复正常范围: 

```
admin@10-60-141-108:/workspaces/isaac_ros-dev$ python3 huangyan/moveit_and_isaac_sim/linkerhand_and_stereo_camera/collect_eye_to_hand_calibration.py   --calibrate eye_to_hand.log   --base-frame g_base   --flange-frame joint6_flange   --camera-frame camera_optical

======================================================================
 Eye-to-Hand 标定结果（外置相机）
======================================================================
 输出静态TF: g_base -> camera_optical
  平移 xyz: [0.433770, 0.372144, 0.122283]
  旋转 xyzw: [-0.222481, -0.607995, 0.742728, 0.170877]
  旋转 RPY(deg): [-80.62, 7.05, 148.11]

 同时输出求解到的固定关系（用于诊断）：flange -> tag
  平移 xyz: [-0.002737, 0.012103, 0.079460]  |  |t|=0.080 m
  旋转 xyzw: [0.256537, 0.611347, 0.662256, 0.349084]
  旋转 RPY(deg): [83.03, 4.99, 128.83]

 一致性误差（每帧回代的 base->camera 相对平均值）
  平移误差: mean=42.83 mm, std=14.84 mm
  旋转误差: mean=4.757 deg, std=1.525 deg

 离群值剔除敏感性（按综合评分排序，剔除最差比例后重新统计）
  说明：综合评分 = trans_err_mm + rot_err_deg*10（1deg≈10mm，仅用于排序）
  - 去掉最差  0%（保留 5/5）: 平移 mean=42.83mm std=14.84mm, 旋转 mean=4.757deg std=1.525deg
  - 去掉最差  5%（保留 5/5）: 平移 mean=42.83mm std=14.84mm, 旋转 mean=4.757deg std=1.525deg
  - 去掉最差 10%（保留 5/5）: 平移 mean=42.83mm std=14.84mm, 旋转 mean=4.757deg std=1.525deg
  - 去掉最差 20%（保留 4/5）: 平移 mean=30.52mm std=12.10mm, 旋转 mean=3.376deg std=1.036deg
  - 去掉最差 30%（保留 4/5）: 平移 mean=30.52mm std=12.10mm, 旋转 mean=3.376deg std=1.036deg
  - 去掉最差 40%（保留 3/5）: 平移 mean=27.36mm std=5.83mm, 旋转 mean=3.129deg std=0.967deg
  - 去掉最差 50%（保留 3/5）: 平移 mean=27.36mm std=5.83mm, 旋转 mean=3.129deg std=0.967deg

 可直接复制执行（Base -> Camera Optical）：
  ros2 run tf2_ros static_transform_publisher 0.43377003161104916 0.37214385548262374 0.12228285702895156 -0.2224808290300101 -0.6079950852249072 0.7427283728161873 0.1708766258746232 g_base camera_optical

======================================================================
  物理坐标标定结果 (Base -> Camera Link)
======================================================================
  计算说明：Base->Link = (Base->Optical) * (Optical->Link)
  其中 Optical->Link 旋转为 quat=[-0.5, 0.5, -0.5, 0.5] (ROS标准)

 物理输出静态TF: g_base -> camera_link
  平移 xyz: [0.433770, 0.372144, 0.122283]
  旋转 xyzw: [0.264045, 0.701164, 0.129312, -0.649560]
  旋转 RPY(deg): [-127.19, -78.29, 94.72]

 用于 slider.launch.py 的静态 TF 参数 (x y z qx qy qz qw):
"0.43377003",
"0.37214386",
"0.12228286",
"0.26404537",
"0.70116383",
"0.12931208",
"-0.64955963"
======================================================================
``` 

使用多个点采样: 

```
admin@10-60-141-108:/workspaces/isaac_ros-dev$ python3 huangyan/moveit_and_isaac_sim/linkerhand_and_stereo_camera/collect_eye_to_hand_calibration.py   --calibrate eye_to_hand.log   --base-frame g_base   --flange-frame joint6_flange   --camera-frame camera_optical

======================================================================
[OK] Eye-to-Hand 标定结果（外置相机）
======================================================================
[INFO] 输出静态TF: g_base -> camera_optical
  平移 xyz: [0.405194, 0.397920, 0.094698]
  旋转 xyzw: [-0.102422, -0.630266, 0.722079, 0.266227]
  旋转 RPY(deg): [-79.17, -10.82, 148.47]

[INFO] 同时输出求解到的固定关系（用于诊断）：flange -> tag
  平移 xyz: [0.039340, -0.012546, 0.096091]  |  |t|=0.105 m
  旋转 xyzw: [0.332046, 0.600613, 0.691524, 0.225400]
  旋转 RPY(deg): [86.61, -10.86, 133.65]

[INFO] 一致性误差（每帧回代的 base->camera 相对平均值）
  平移误差: mean=63.81 mm, std=31.14 mm
  旋转误差: mean=5.493 deg, std=3.092 deg

[INFO] 离群值剔除敏感性（按综合评分排序，剔除最差比例后重新统计）
  说明：综合评分 = trans_err_mm + rot_err_deg*10（1deg≈10mm，仅用于排序）
  - 去掉最差  0%（保留 17/17）: 平移 mean=63.81mm std=31.14mm, 旋转 mean=5.493deg std=3.092deg
  - 去掉最差  5%（保留 17/17）: 平移 mean=63.81mm std=31.14mm, 旋转 mean=5.493deg std=3.092deg
  - 去掉最差 10%（保留 16/17）: 平移 mean=58.36mm std=21.64mm, 旋转 mean=4.678deg std=1.900deg
  - 去掉最差 20%（保留 14/17）: 平移 mean=53.13mm std=16.80mm, 旋转 mean=4.218deg std=1.809deg
  - 去掉最差 30%（保留 12/17）: 平移 mean=47.54mm std=9.90mm, 旋转 mean=3.825deg std=1.472deg
  - 去掉最差 40%（保留 11/17）: 平移 mean=44.88mm std=7.99mm, 旋转 mean=3.916deg std=1.440deg
  - 去掉最差 50%（保留 9/17）: 平移 mean=43.74mm std=7.95mm, 旋转 mean=3.648deg std=1.097deg

[INFO] 可直接复制执行（Base -> Camera Optical）：
  ros2 run tf2_ros static_transform_publisher 0.4051941006529292 0.3979197205193096 0.09469803869075048 -0.10242188464815023 -0.6302658027127083 0.7220787382800112 0.2662274801743307 g_base camera_optical

======================================================================
[INFO] 物理坐标标定结果 (Base -> Camera Link)
======================================================================
  计算说明：Base->Link = (Base->Optical) * (Optical->Link)
  其中 Optical->Link 旋转为 quat=[-0.5, 0.5, -0.5, 0.5] (ROS标准)

[INFO] 物理输出静态TF: g_base -> camera_link
  平移 xyz: [0.405194, 0.397920, 0.094698]
  旋转 xyzw: [-0.230231, -0.594269, -0.138418, 0.758075]
  旋转 RPY(deg): [-44.52, -74.74, 14.02]

[INFO] 用于 slider.launch.py 的静态 TF 参数 (x y z qx qy qz qw):
"0.40519410",
"0.39791972",
"0.09469804",
"-0.23023115",
"-0.59426947",
"-0.13841821",
"0.75807507"
======================================================================

``` 

提出极值30%, 求稳定: 

```
admin@10-60-141-108:/workspaces/isaac_ros-dev$ python3 huangyan/moveit_and_isaac_sim/linkerhand_and_stereo_camera/collect_eye_to_hand_calibration.py   --calibrate eye_to_hand.log   --base-frame g_base   --flange-frame joint6_flange   --camera-frame camera_optical --trim-percent 30

======================================================================
[OK] Eye-to-Hand 标定结果（外置相机）
======================================================================
[INFO] 输出静态TF: g_base -> camera_optical
  平移 xyz: [0.405194, 0.397920, 0.094698]
  旋转 xyzw: [-0.102422, -0.630266, 0.722079, 0.266227]
  旋转 RPY(deg): [-79.17, -10.82, 148.47]

[INFO] 同时输出求解到的固定关系（用于诊断）：flange -> tag
  平移 xyz: [0.039340, -0.012546, 0.096091]  |  |t|=0.105 m
  旋转 xyzw: [0.332046, 0.600613, 0.691524, 0.225400]
  旋转 RPY(deg): [86.61, -10.86, 133.65]

[INFO] 一致性误差（每帧回代的 base->camera 相对平均值）
  平移误差: mean=63.81 mm, std=31.14 mm
  旋转误差: mean=5.493 deg, std=3.092 deg

[INFO] 离群值剔除敏感性（按综合评分排序，剔除最差比例后重新统计）
  说明：综合评分 = trans_err_mm + rot_err_deg*10（1deg≈10mm，仅用于排序）
  - 去掉最差  0%（保留 17/17）: 平移 mean=63.81mm std=31.14mm, 旋转 mean=5.493deg std=3.092deg
  - 去掉最差  5%（保留 17/17）: 平移 mean=63.81mm std=31.14mm, 旋转 mean=5.493deg std=3.092deg
  - 去掉最差 10%（保留 16/17）: 平移 mean=58.36mm std=21.64mm, 旋转 mean=4.678deg std=1.900deg
  - 去掉最差 20%（保留 14/17）: 平移 mean=53.13mm std=16.80mm, 旋转 mean=4.218deg std=1.809deg
  - 去掉最差 30%（保留 12/17）: 平移 mean=47.54mm std=9.90mm, 旋转 mean=3.825deg std=1.472deg
  - 去掉最差 40%（保留 11/17）: 平移 mean=44.88mm std=7.99mm, 旋转 mean=3.916deg std=1.440deg
  - 去掉最差 50%（保留 9/17）: 平移 mean=43.74mm std=7.95mm, 旋转 mean=3.648deg std=1.097deg

[INFO] 可直接复制执行（Base -> Camera Optical）：
  ros2 run tf2_ros static_transform_publisher 0.4051941006529292 0.3979197205193096 0.09469803869075048 -0.10242188464815023 -0.6302658027127083 0.7220787382800112 0.2662274801743307 g_base camera_optical

======================================================================
[INFO] 剔除最差 30% 后的标定结果（按综合评分排序）
======================================================================
[INFO] 输出静态TF: g_base -> camera_optical   (kept=12/17)
  平移 xyz: [0.402062, 0.399173, 0.081343]
  旋转 xyzw: [-0.103086, -0.618799, 0.730926, 0.268715]
  旋转 RPY(deg): [-77.49, -10.48, 148.05]

[INFO] 一致性误差（剔除后样本相对剔除后平均值）
  平移误差: mean=47.54 mm, std=9.90 mm
  旋转误差: mean=3.825 deg, std=1.472 deg

[INFO] 可直接复制执行（Trimmed Base -> Camera Optical）：
  ros2 run tf2_ros static_transform_publisher 0.40206172525228956 0.3991725631405305 0.08134329493998836 -0.10308637635653334 -0.6187994099512606 0.7309260825248878 0.2687146276972505 g_base camera_optical

[INFO] 物理坐标标定结果 (Trimmed Base -> Camera Link)
[INFO] 物理输出静态TF: g_base -> camera_link
  平移 xyz: [0.402062, 0.399173, 0.081343]
  旋转 xyzw: [-0.241964, -0.592049, -0.129837, 0.757677]
  旋转 RPY(deg): [-49.50, -73.74, 18.70]

[INFO] 用于 slider.launch.py 的静态 TF 参数 (x y z qx qy qz qw):
"0.40206173",
"0.39917256",
"0.08134329",
"-0.24196384",
"-0.59204862",
"-0.12983717",
"0.75767687"
======================================================================

======================================================================
[INFO] 物理坐标标定结果 (Base -> Camera Link)
======================================================================
  计算说明：Base->Link = (Base->Optical) * (Optical->Link)
  其中 Optical->Link 旋转为 quat=[-0.5, 0.5, -0.5, 0.5] (ROS标准)

[INFO] 物理输出静态TF: g_base -> camera_link
  平移 xyz: [0.405194, 0.397920, 0.094698]
  旋转 xyzw: [-0.230231, -0.594269, -0.138418, 0.758075]
  旋转 RPY(deg): [-44.52, -74.74, 14.02]

[INFO] 用于 slider.launch.py 的静态 TF 参数 (x y z qx qy qz qw):
"0.40519410",
"0.39791972",
"0.09469804",
"-0.23023115",
"-0.59426947",
"-0.13841821",
"0.75807507"
======================================================================

admin@10-60-141-108:/workspaces/isaac_ros-dev$
``` 

观察结果: 

![image2025-12-16 14:7:37.png](/assets/01KJC09C6KBEC83Z6P9EX5Y4F9/image2025-12-16%2014%3A7%3A37.png)

标定正确

让相机靠近机械臂 (视野更好), 重新标定, 需要去除20%的异常值: 

```
admin@10-60-141-108:/workspaces/isaac_ros-dev$ python3 huangyan/moveit_and_isaac_sim/linkerhand_and_stereo_camera/collect_eye_to_hand_calibration.py   --calibrate eye_to_hand.log   --base-frame g_base   --flange-frame joint6_flange   --camera-frame camera_optical --trim-percent 20

======================================================================
[OK] Eye-to-Hand 标定结果（外置相机）
======================================================================
[INFO] 输出静态TF: g_base -> camera_optical
  平移 xyz: [0.339317, 0.233086, 0.119179]
  旋转 xyzw: [-0.129105, -0.666021, 0.699616, 0.224245]
  旋转 RPY(deg): [-85.41, -6.78, 150.71]

[INFO] 同时输出求解到的固定关系（用于诊断）：flange -> tag
  平移 xyz: [0.082516, 0.026327, 0.077161]  |  |t|=0.116 m
  旋转 xyzw: [-0.607026, 0.303858, -0.240872, 0.693665]
  旋转 RPY(deg): [-85.47, 7.42, -45.15]

[INFO] 一致性误差（每帧回代的 base->camera 相对平均值）
  平移误差: mean=79.70 mm, std=44.11 mm
  旋转误差: mean=9.040 deg, std=5.805 deg

[INFO] 离群值剔除敏感性（按综合评分排序，剔除最差比例后重新统计）
  说明：综合评分 = trans_err_mm + rot_err_deg*10（1deg≈10mm，仅用于排序）
  - 去掉最差  0%（保留 18/18）: 平移 mean=79.70mm std=44.11mm, 旋转 mean=9.040deg std=5.805deg
  - 去掉最差  5%（保留 18/18）: 平移 mean=79.70mm std=44.11mm, 旋转 mean=9.040deg std=5.805deg
  - 去掉最差 10%（保留 17/18）: 平移 mean=71.42mm std=40.03mm, 旋转 mean=7.568deg std=3.927deg
  - 去掉最差 20%（保留 15/18）: 平移 mean=57.12mm std=24.11mm, 旋转 mean=5.924deg std=1.811deg
  - 去掉最差 30%（保留 13/18）: 平移 mean=52.31mm std=22.46mm, 旋转 mean=5.943deg std=1.873deg
  - 去掉最差 40%（保留 11/18）: 平移 mean=46.04mm std=17.79mm, 旋转 mean=5.678deg std=1.784deg
  - 去掉最差 50%（保留 9/18）: 平移 mean=40.64mm std=14.68mm, 旋转 mean=5.108deg std=1.388deg

[INFO] 可直接复制执行（Base -> Camera Optical）：
  ros2 run tf2_ros static_transform_publisher 0.3393167922569984 0.23308604076923867 0.11917862331790634 -0.1291047146790727 -0.666021171182566 0.6996156566516707 0.22424474386725324 g_base camera_optical

======================================================================
[INFO] 剔除最差 20% 后的标定结果（按综合评分排序）
======================================================================
[INFO] 输出静态TF: g_base -> camera_optical   (kept=15/18)
  平移 xyz: [0.370395, 0.231721, 0.125061]
  旋转 xyzw: [-0.152242, -0.666084, 0.687818, 0.245073]
  旋转 RPY(deg): [-86.17, -6.72, 147.07]

[INFO] 一致性误差（剔除后样本相对剔除后平均值）
  平移误差: mean=57.12 mm, std=24.11 mm
  旋转误差: mean=5.924 deg, std=1.811 deg

[INFO] 可直接复制执行（Trimmed Base -> Camera Optical）：
  ros2 run tf2_ros static_transform_publisher 0.3703952996774681 0.23172149084059343 0.1250614425508913 -0.1522424085788888 -0.6660841374664568 0.6878178779810229 0.2450729229746678 g_base camera_optical

[INFO] 物理坐标标定结果 (Trimmed Base -> Camera Link)
[INFO] 物理输出静态TF: g_base -> camera_link
  平移 xyz: [0.370395, 0.231721, 0.125061]
  旋转 xyzw: [-0.209525, -0.630536, -0.187791, 0.723366]
  旋转 RPY(deg): [-29.53, -82.27, -3.18]

[INFO] 用于 slider.launch.py 的静态 TF 参数 (x y z qx qy qz qw):
"0.37039530",
"0.23172149",
"0.12506144",
"-0.20952454",
"-0.63053575",
"-0.18779080",
"0.72336626"
======================================================================

======================================================================
[INFO] 物理坐标标定结果 (Base -> Camera Link)
======================================================================
  计算说明：Base->Link = (Base->Optical) * (Optical->Link)
  其中 Optical->Link 旋转为 quat=[-0.5, 0.5, -0.5, 0.5] (ROS标准)

[INFO] 物理输出静态TF: g_base -> camera_link
  平移 xyz: [0.339317, 0.233086, 0.119179]
  旋转 xyzw: [-0.193472, -0.635248, -0.159877, 0.730388]
  旋转 RPY(deg): [-33.96, -81.82, 4.94]

[INFO] 用于 slider.launch.py 的静态 TF 参数 (x y z qx qy qz qw):
"0.33931679",
"0.23308604",
"0.11917862",
"-0.19347197",
"-0.63524840",
"-0.15987749",
"0.73038843"
======================================================================
``` 

commit: f9cb99b2695e64afbdb9cafbe2ece640c9fe4f5f

增加sim-time的使用, 增加诊断用的IK和机械臂姿态: 

```
python3 slider/slider.launch.py --use-sim-time --enable-apriltag --apriltag-size 0.016
 
python3 moveit/moveit.launch.py --use-rviz --use-sim-time
 
python3 stereo_tag_arm_scheduler.py   --use-sim-time   --target-tag-id 6   --ee-offset-z -0.2 --ik-timeout-sec 5   --hold-on-ik-fail   --ik-fail-tf-child-frame ik_target_debug   --ik-fail-tf-rate-hz 10 --confirm-before-execute
``` 

现在的问题是: apriltag的位姿不稳定, 有时向上, 有时向下 (主要是摄像头精度不够), 考虑用写死的某种方式来暂时规避
