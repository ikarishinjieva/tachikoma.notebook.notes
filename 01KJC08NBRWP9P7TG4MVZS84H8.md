---
title: 20251107 - 为机械臂配置笔记本
confluence_page_id: 4359071
created_at: 2025-11-07T07:57:08+00:00
updated_at: 2025-11-17T03:30:03+00:00
---

机械臂已经到货, 使用一台新笔记本, 安装ubuntu 22.04 (IP:192.168.22.12, huangyan/1)

  - 注意: 安装过程中, 配置wifi和mirror地址, 否则安装好后, 需要网络才能安装网卡驱动

安装docker:

```
sudo apt install -y docker.io
``` 

配置代理

按照 [20251010 - 整理Isaac Sim和ROS2 demo的环境安装和运行指令] , 配置"安装 ROS2 DEMO", 在笔记本上配置ROS2.

~~下载驱动:<https://download.elephantrobotics.com/software/drivers/CP210x_VCP_Linux.zip>~~

modinfo cp210x 已经有串口驱动了

将usb线连接机械臂和电脑, 参考 <https://docs.elephantrobotics.com/docs/mycobot_280_m5_cn/3-FunctionsAndApplications/6.developmentGuide/python/1_download.html> 中提供的脚本: 

```
from pymycobot.mycobot280 import MyCobot280
import time
#以上需写在代码开头，意为导入项目包

# MyCobot280 类初始化需要两个参数：串口和波特率
#   第一个是串口字符串， 如：
#       linux： "/dev/ttyUSB0"
#       windows: "COM3"
#   第二个是波特率：
#       M5版本为： 115200
#   以下为如:
#       mycobot-M5:
#           linux:
#              mc = MyCobot280("/dev/ttyUSB0", 115200)
#           windows:
#              mc = MyCobot280("COM3", 115200)

# 初始化一个MyCobot280对象
# 下面为 windows版本创建对象代码
mc = MyCobot280("/dev/ttyACM0", 115200)

i = 7
#循环7次
while i > 0:
    mc.set_color(0,0,255) #蓝灯亮
    time.sleep(2)    #等2秒
    mc.set_color(255,0,0) #红灯亮
    time.sleep(2)    #等2秒
    mc.set_color(0,255,0) #绿灯亮
    time.sleep(2)    #等2秒
    i -= 1
``` 

注意: 设备号/dev/ttyACM0 是插入usb后, 从dmesg中获得. 可以看到机械臂的灯按照代码进行调度

需要安装桌面和NICE: 

````
sudo snap set system proxy.http="http://127.0.0.1:7890"
sudo snap set system proxy.https="http://127.0.0.1:7890"
sudo apt install ubuntu-desktop
sudo apt install gdm3
sudo apt install dkms
sudo dcvusbdriverinstaller
 
reboot -f 
 
wget https://d1uj6qtbmh3dt5.cloudfront.net/2024.0/Servers/nice-dcv-2024.0-19030-ubuntu2204-x86_64.tgz
tar -xvzf nice-dcv-2024.0-19030-ubuntu2204-x86_64.tgz && cd nice-dcv-2024.0-19030-ubuntu2204-x86_64
sudo apt install ./nice-dcv-server_2024.0.19030-1_amd64.ubuntu2204.deb
sudo apt install ./nice-dcv-web-viewer_2024.0.19030-1_amd64.ubuntu2204.deb
sudo apt install ./nice-xdcv_2024.0.654-1_amd64.ubuntu2204.deb
  
sudo systemctl start dcvserver && sudo systemctl enable dcvserver
sudo systemctl status dcvserver
 
 
dcv create-session mysession
dcv list-sessions
# 发现创建session失败
 
 
检查日志: 
```
Nov  8 05:04:39 bot dbus-daemon[919]: [system] Activating via systemd: service name='com.nicesoftware.DcvSessionLauncher' unit='dcvsessionlauncher.service' requested by ':1.175' (uid=998 pid=7051 comm="/usr/lib/x86_64-linux-gnu/dcv/dcvserver --service " label="unconfined")
Nov  8 05:04:39 bot systemd[1]: Starting Amazon DCV session launcher daemon...
Nov  8 05:04:39 bot dbus-daemon[919]: [system] Successfully activated service 'com.nicesoftware.DcvSessionLauncher'
Nov  8 05:04:39 bot systemd[1]: Started Amazon DCV session launcher daemon.
Nov  8 05:04:39 bot systemd[1]: Started Session c6 of User huangyan.
Nov  8 05:04:40 bot gnome-session[7249]: gnome-session-check-accelerated: no X11 display found
Nov  8 05:04:40 bot gnome-session[7275]: gnome-session-check-accelerated: no X11 display found
Nov  8 05:04:40 bot gnome-session[7134]: gnome-session-binary[7134]: WARNING: software acceleration check failed: Child process exited with code 1
Nov  8 05:04:40 bot gnome-session-binary[7134]: WARNING: software acceleration check failed: Child process exited with code 1
Nov  8 05:04:40 bot gnome-session-f[7290]: Cannot open display:
```
 
得先禁用笔记本上的图形界面: sudo systemctl stop gdm3
然后就可以创建session成功
```` 

因为笔记本没有显卡, 需要调整run_dev.sh: 

```
在docker run中, 删除--runtime=nvidia参数
``` 

进入容器: 

```
export ISAAC_ROS_WS=/data/huangyan/isaac_ros-dev/
cd $ISAAC_ROS_WS && ./src/isaac_ros_common/scripts/run_dev.sh
``` 

安装机械臂的ros2包, 以及moveit: 

```
export http_proxy=http://127.0.0.1:7890
export https_proxy=http://127.0.0.1:7890
export no_proxy=127.0.0.1,localhost,ucloud.cn
pip install pymycobot --upgrade
 
sudo apt-get install ros-humble-moveit
sudo apt install ros-humble-ros2-control ros-humble-ros2-controllers ros-humble-joint-trajectory-controller ros-humble-joint-state-broadcaster
 
cd /workspaces/isaac_ros-dev/src
git clone --depth 1 https://github.com/elephantrobotics/mycobot_ros2.git
cd ..
colcon build --symlink-install
source install/setup.bash
``` 

初始化容器环境: 

```
export http_proxy=http://127.0.0.1:7890
export https_proxy=http://127.0.0.1:7890
export no_proxy=127.0.0.1,localhost,ucloud.cn
export ROS_DOMAIN_ID=99
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
export ROS_DISTRO=humble
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/data/huangyan/isaac-sim/exts/omni.isaac.ros2_bridge/humble/lib/
export DISPLAY=:0
 
source install/setup.bash
``` 

进行机械臂连接测试, 在容器内: 

```
sudo chmod 777 /dev/ttyACM0
ros2 launch mycobot_280 slider_control.launch.py port:=/dev/ttyACM0 baud:=115200
``` 

可以看到界面: 

![image2025-11-8 13:15:27.png](/assets/01KJC08NBRWP9P7TG4MVZS84H8/image2025-11-8%2013%3A15%3A27.png)

拖动滑块, 各关节可以实际移动

# 从远端进行调用

在远端服务器, 配置sshd配置: 

```
PermitRootLogin prohibit-password
PermitTunnel yes
``` 

在本地笔记本中生成公钥, 在远端服务器的root中, 将公钥加入authorized_keys

从本地笔记本, 连接远端服务器, 建立tun通道: 

```
sudo ssh -f -w 0:0 root@117.50.172.161 -p 22 -o Tunnel=point-to-point -N
``` 

配置IP: 

```
在客户端（本机）：

ip link show | grep -A2 tun

sudo ip addr add 10.10.10.2/30 dev tun0

sudo ip link set tun0 up

在服务器（远端）：

ip link show | grep -A2 tun

sudo ip addr add 10.10.10.1/30 dev tun0

sudo ip link set tun0 up
``` 

测试UDP:

```
客户端: 
sudo nc -u -l -p 11812

服务器端:
echo 'hello' | nc -u 10.10.10.2 11812
``` 

配置CYCLONEDDS 配置文件: /workspaces/isaac_ros-dev/a.xml

```
在笔记本端: 
 
<?xml version="1.0" encoding="UTF-8"?>

<CycloneDDS>

<Domain id="any">

<General>

<NetworkInterfaceAddress>10.10.10.2</NetworkInterfaceAddress>

</General>

<Discovery>

<Peers>

<Peer address="10.10.10.1"/>

</Peers>

</Discovery>

<Tracing><Verbosity>config</Verbosity></Tracing>

</Domain>

</CycloneDDS>
 
在服务器端: 
 
<?xml version="1.0" encoding="UTF-8"?>

<CycloneDDS>

<Domain id="any">

<General>

<NetworkInterfaceAddress>10.10.10.1</NetworkInterfaceAddress>

</General>

<Discovery>

<Peers>

<Peer address="10.10.10.2"/>

</Peers>

</Discovery>

<Tracing><Verbosity>config</Verbosity></Tracing>

</Domain>

</CycloneDDS>
``` 

在两端引入环境变量, 并重启ros2缓存: 

```
export CYCLONEDDS_URI=/workspaces/isaac_ros-dev/a.xml
 
ros2 daemon stop; ros2 daemon start
``` 

进行测试: 

```
在笔记本端: 
ros2 run demo_nodes_cpp talker

在服务器端:
ros2 node list
# 可以看到chatter节点
``` 

注意, 将Domain ID切换到了98

# 如何控制机械臂

```
ros2 launch mycobot_280 slider_control.launch.py port:=/dev/ttyACM0 baud:=115200
``` 

其中使用了: <https://raw.githubusercontent.com/elephantrobotics/mycobot_ros2/refs/heads/humble/mycobot_280/mycobot_280/mycobot_280/slider_control_adaptive_gripper.py>

这个驱动订阅了/joint_states, 来进行机械臂操作. 这违反了行业惯例.

读取状态: 

```
admin@bot:/workspaces/isaac_ros-dev$ ros2 topic echo /joint_states --once
1762586972.089329 [98]       ros2: config: //CycloneDDS/Domain/General: 'NetworkInterfaceAddress': deprecated element (/workspaces/isaac_ros-dev/a.xml line 9)
header:
  stamp:
    sec: 1762586972
    nanosec: 721529422
  frame_id: ''
name:
- joint2_to_joint1
- joint3_to_joint2
- joint4_to_joint3
- joint5_to_joint4
- joint6_to_joint5
- joint6output_to_joint6
position:
- 0.0
- 0.49668696000000034
- 0.0
- 0.0
- 0.0
- 0.0
velocity: []
effort: []
---
``` 

发布新位置:

```
ros2 topic pub -r 5 /joint_states sensor_msgs/JointState "
header: {stamp: {sec: 0, nanosec: 0}, frame_id: ''}
name:
- joint2_to_joint1
- joint3_to_joint2
- joint4_to_joint3
- joint5_to_joint4
- joint6_to_joint5
- joint6output_to_joint6
position:
- 0
- 0.1
- 0
- 0
- 0
- 0
velocity: []
effort: []
"
``` 

可以看到机械臂来回移动 (脚本要移动位置, rviz要还原位置, 产生冲突)

下一步需要一个简单的脚本, 在笔记本上, 接受控制信号, 从<https://raw.githubusercontent.com/elephantrobotics/mycobot_ros2/refs/heads/humble/mycobot_280/mycobot_280/launch/slider_control.launch.py> 修改, 去掉rviz部分即可: 

```
import os

from ament_index_python import get_package_share_directory
from launch_ros.actions import Node
from launch_ros.parameter_descriptions import ParameterValue

from launch import LaunchDescription
from launch.actions import DeclareLaunchArgument
from launch.conditions import IfCondition, UnlessCondition
from launch.substitutions import Command, LaunchConfiguration

def generate_launch_description():
    res = []

    model_launch_arg = DeclareLaunchArgument(
        "model",
        default_value=os.path.join(
            get_package_share_directory("mycobot_description"),
            "urdf/mycobot_280_m5/mycobot_280_m5.urdf"
        )
    )
    res.append(model_launch_arg)

    gui_launch_arg = DeclareLaunchArgument(
        "gui",
        default_value="true"
    )
    res.append(gui_launch_arg)
    
    serial_port_arg = DeclareLaunchArgument(
        'port',
        default_value='/dev/ttyACM0',
        description='Serial port to use'
    )
    res.append(serial_port_arg)
    baud_rate_arg = DeclareLaunchArgument(
        'baud',
        default_value='115200',
        description='Baud rate to use'
    )
    res.append(baud_rate_arg)

    robot_description = ParameterValue(Command(['xacro ', LaunchConfiguration('model')]),
                                       value_type=str)

    robot_state_publisher_node = Node(
        name="robot_state_publisher",
        package="robot_state_publisher",
        executable="robot_state_publisher",
        parameters=[{'robot_description': robot_description}]
    )
    res.append(robot_state_publisher_node)
    
    slider_control_node = Node(
        package="mycobot_280",
        executable="slider_control",
        parameters=[
            {'port': LaunchConfiguration('port')},
            {'baud': LaunchConfiguration('baud')}
        ],
        name="slider_control",
        output="screen"
    )
    res.append(slider_control_node)

    return LaunchDescription(res)
``` 

放在容器目录中: /workspaces/isaac_ros-dev/install/mycobot_280/share/mycobot_280/launch/280_control.py

启动命令: 

```
ros2 launch mycobot_280 280_control.py
``` 

然后在远端服务器上, 执行命令: 

```
ros2 topic pub -r 5 /joint_states sensor_msgs/JointState "
header: {stamp: {sec: 0, nanosec: 0}, frame_id: ''}
name:
- joint2_to_joint1
- joint3_to_joint2
- joint4_to_joint3
- joint5_to_joint4
- joint6_to_joint5
- joint6output_to_joint6
position:
- 0
- 0.1
- 0
- 0
- 0
- 0
velocity: []
effort: []
"
``` 

可以看到机械臂稳定以后, 而不会发生摆动

使用moveit, 在容器内: 

```
在一个窗口内, 执行控制器: 
ros2 run mycobot_280_moveit2_control sync_plan
 
在另一个窗口内, 执行rviz进行执行: 
ros2 lanuch mycobot_280_moveit2 demo.launch.py
 
可以看到机械臂的移动
``` 

更新方案: 

```
在笔记本上, 运行: 
ros2 run mycobot_280 slider_control --ros-args -p port:=/dev/ttyACM0 -p baud:=115200
 
在服务器上, 运行: (commit: 12cfbf6ddfa7a3a1a256de3cbb05570d09e8b0ac)
ros2 launch huangyan/moveit_and_isaac_sim/mycobot_280_real_moveit.launch.py

使用rviz, 就可以调动机械臂
``` 

# 重新整理环境架设步骤

```
远端服务器: 公网IP: 117.50.172.161
近端笔记本: 内网IP: 192.168.22.12
 
... TODO
``` 

# 物理对齐

使用rviz, 进行逐个关节的验证, 发现: 

![image2025-11-10 13:15:55.png](/assets/01KJC08NBRWP9P7TG4MVZS84H8/image2025-11-10%2013%3A15%3A55.png)

发现:

  - 调整joint2_to_joint1, 机械臂joint2_to_joint1会正确移动
  - 调整joint3_to_joint2, 机械臂并不会移动
  - 调整joint4_to_joint3, 机械臂的joint3_to_joint2会移动
  - 调整joint5_to_joint4, 机械臂的joint4_to_joint3会移动
  - 调整joint6_to_joint5, 机械臂的joint5_to_joint4会移动
  - 调整joint6output, 机械臂对应夹爪连接处角度正常移动

调整: 更新近端笔记本上的脚本: /workspaces/isaac_ros-dev/src/mycobot_ros2/mycobot_280/mycobot_280/mycobot_280/slider_control_adaptive_gripper.py, 增加对参数名字的解析 (原来只是依靠顺序) :

```
import rclpy
from rclpy.node import Node
from sensor_msgs.msg import JointState
import time
import math
import pymycobot
from packaging import version

# min low version require
MIN_REQUIRE_VERSION = '4.0.0'
current_verison = pymycobot.__version__
print('current pymycobot library version: {}'.format(current_verison))
if version.parse(current_verison) < version.parse(MIN_REQUIRE_VERSION):
    raise RuntimeError(
        'The version of pymycobot library must be greater than {} or higher. '
        'The current version is {}. Please upgrade the library version.'.format(
            MIN_REQUIRE_VERSION, current_verison))
else:
    print('pymycobot library version meets the requirements!')
    from pymycobot import MyCobot280

class SliderSubscriberByName(Node):
    def __init__(self):
        super().__init__("control_slider_by_name")

        # 可覆盖：串口与波特率
        self.declare_parameter('port', '/dev/ttyUSB0')
        self.declare_parameter('baud', 115200)

        # 可覆盖：机械臂6关节的“期望顺序的关节名”（与物理J1..J6一致）
        default_arm_joint_names = [
            'joint2_to_joint1',
            'joint3_to_joint2',
            'joint4_to_joint3',
            'joint5_to_joint4',
            'joint6_to_joint5',
            'joint6output_to_joint6',
        ]
        self.declare_parameter('arm_joint_names', default_arm_joint_names)

        # 可覆盖：夹爪关节名与数值映射区间
        self.declare_parameter('gripper_joint_name', 'gripper_controller')
        self.declare_parameter('gripper_min', -0.74)  # fully closed
        self.declare_parameter('gripper_max', 0.15)   # fully open

        port = self.get_parameter('port').get_parameter_value().string_value
        baud = self.get_parameter('baud').get_parameter_value().integer_value
        arm_joint_names_param = self.get_parameter('arm_joint_names').get_parameter_value().string_array_value
        self.arm_joint_names = list(arm_joint_names_param) if arm_joint_names_param else default_arm_joint_names
        self.gripper_joint_name = self.get_parameter('gripper_joint_name').get_parameter_value().string_value
        self.gripper_min = float(self.get_parameter('gripper_min').get_parameter_value().double_value)
        self.gripper_max = float(self.get_parameter('gripper_max').get_parameter_value().double_value)

        if len(self.arm_joint_names) != 6:
            raise ValueError("arm_joint_names 必须包含且仅包含 6 个关节名。当前为: {}".format(self.arm_joint_names))

        self.get_logger().info("port:%s, baud:%d" % (port, baud))
        self.get_logger().info("arm_joint_names: {}".format(self.arm_joint_names))
        self.get_logger().info("gripper_joint_name: {}, range[{}, {}]".format(
            self.gripper_joint_name, self.gripper_min, self.gripper_max))

        self.mc = MyCobot280(port, baud)
        time.sleep(0.05)
        self.mc.set_fresh_mode(1)
        time.sleep(0.05)

        self.subscription = self.create_subscription(
            JointState,
            "joint_states",
            self.listener_callback,
            10
        )
        self.subscription

        self._printed_missing_warn = False

    def _map_gripper_value(self, value: float) -> int:
        # 将 [-0.74, 0.15] 映射到 [0, 100]（可通过参数覆盖）
        min_val = self.gripper_min
        max_val = self.gripper_max
        if max_val == min_val:
            return 0
        # 夹爪数值越大越张开：保持与原脚本一致
        mapped = (value - min_val) / (max_val - min_val) * 100.0
        if mapped < 0.0:
            mapped = 0.0
        if mapped > 100.0:
            mapped = 100.0
        return int(round(mapped))

    def listener_callback(self, msg: JointState):
        if not msg.name or not msg.position:
            return

        # 构建 name->index 映射
        name_to_index = {n: i for i, n in enumerate(msg.name)}

        # 逐名提取6轴角度（单位：度）
        data_list = []
        missing_names = []
        for joint_name in self.arm_joint_names:
            idx = name_to_index.get(joint_name, None)
            if idx is None or idx >= len(msg.position):
                missing_names.append(joint_name)
                continue
            radians_val = msg.position[idx]
            degrees_val = round(math.degrees(radians_val), 2)
            data_list.append(degrees_val)

        # 若有缺失，跳过本帧（避免错位），并适度告警（只打印一次，避免刷屏）
        if missing_names or len(data_list) != 6:
            if not self._printed_missing_warn:
                self.get_logger().warn(
                    "JointState 中缺失关节或长度不符，跳过发送。缺失: {}，收到的name: {}".format(
                        missing_names, list(msg.name)))
                self._printed_missing_warn = True
            return
        else:
            # 一旦成功一帧，允许后续再次打印缺失告警
            self._printed_missing_warn = False

        # 提取夹爪值（可选）
        gripper_value = 0
        if self.gripper_joint_name in name_to_index:
            g_idx = name_to_index[self.gripper_joint_name]
            if g_idx < len(msg.position):
                gripper_value = self._map_gripper_value(msg.position[g_idx])

        # 下发
        # self.get_logger().info('data_list(deg): {} gripper_value: {}'.format(data_list, gripper_value))
        self.mc.send_angles(data_list, 25, _async=True)
        self.mc.set_gripper_value(gripper_value, 80, gripper_type=1)

def main(args=None):
    rclpy.init(args=args)
    node = SliderSubscriberByName()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()

if __name__ == "__main__":
    main()
``` 

commit: b9b1f9ab3b3586c518dba43d17750b1a8c4ceaf8 (代码库中的文件名: slider_control_adaptive_gripper.fix_seq.py)

可以恢复正常

# 安装机械爪

接入夹爪后, 需要修改代码:

  - commit: f6b2e51db95e6f224bfac11ba03e9f5640e927fc
  - 这样在rviz中才能控制夹爪 (夹爪使用一个单独的规划组, 并且转成JointTrajectoryController方式, 替换在仿真环境中使用的ForwardCommandController

夹爪因为乐高接口原因, 是斜45°安装的, 所以需要再URDF中, 将夹爪的初始位置进行倾斜, 与真实情况相同

另外将 mycobot_280_real_moveit.launch.py 进行修改, 默认情况不开启rviz

commit: bf01082f29de9a52aa5d89df3a3507055dff6ac0

# 将仿真和现实结合在一起

将仿真场景中的碰撞体情况导出, 放在现实规划中

commit: 0b712837538cdce3a294dac6b316f9d468eda47b

使用isaac_sim_exporter.py, 在isaac sim中脚本窗口中执行, 能导出env/scene.yaml

然后修改 test_real.py, 读取env/scene.yaml, 发布到/collision_object

在rviz中, 可以看到碰撞体

让test_real.py也能运行于仿真环境

commit: 801fbef217cba285ba117a0184be531c8fae03ee

使用方法: 

```
<真实机器人模式>
 
# 终端1
ros2 launch mycobot_280_real_moveit.launch.py

# 终端2
python3 test_real.py
 
 
 
<Isaac Sim 仿真模式>
 
# 终端1: Isaac Sim GUI（加载场景并 Play）

# 终端2
ros2 launch mycobot_280_isaac_sim_moveit.launch.py

# 终端3
python3 test_real.py --use-sim

``` 

# 规划新的路径算法

倒向算法, 从目标位置开始, 移动关节, 倒序规划路径

另外设定了QoS

commit: 3bae53dd3358cceb05bdb3840e9acc5e1ff22cb5

# 减少抖动

机械臂会在保位位置发生抖动

commit: 02045d8dc8ebd3e85b52e5ba6a9f84331cf68eb2

调整mycobot_280_real.ros2_controllers.yaml中的刷新率到20Hz. 

只能减轻现象, 但有时仍然会发生重新规划和抖动

# 整理现在的脚本流程

我现在对机械臂的调度有两种流程:   
1\. 仿真流程:   
\- 启动isaac sim  
\- 启动@mycobot_280_isaac_sim_moveit.launch.py   
\- 启动 test_real.py --use_sim  
  
2\. 真实流程:   
\- 在机械臂一端, 启动@slider_control_adaptive_gripper.fix_seq.py   
\- 启动 @mycobot_280_real_moveit.launch.py   
\- 启动 test_real.py

# 用真实反馈替代“命令即状态”

还需要解决: 

用真实反馈替代“命令即状态”

现在 fake hardware 把“命令”当“状态”广播。改为使用真实 /joint_states 反馈给 MoveIt，避免外层“完美跟踪”的假象和不必要的保位刷新。

commit: bba428214ab694e0ec9c7580c57bc3427aa2c44e

主要修改在文件: 分离状态和控制.md 中

发现问题: 在slider端, 引入了时间处理, 现在的问题是: 任何一个关节指令发给机械臂, 都需要500ms以上开始响应, 但moveit的指令间隔为100ms, 所以永远会超时. 

  - 引入的原因是: 
    - 机械臂ros2包中默认的slider, 是不处理时间序列的, 只接受一次关节信息, 然后发给串口
    - 我们改造的slider, 可以接受一个动作序列. 那么每个动作在前一个动作过多久发给串口, 就是一个问题. 发送慢了, 动作卡顿, 发送快了, 发生覆盖.
  - 找到的一些线索: 
    - mc.set_fresh_mode, 有一个队列串行模式, 可以不覆盖
      - <https://github.com/elephantrobotics/pymycobot/blob/40394ba8915764e6b0441b38d88fed1c2bedbf39/pymycobot/mycobot280.py#L280>
      - ![image2025-11-13 22:10:16.png](/assets/01KJC08NBRWP9P7TG4MVZS84H8/image2025-11-13%2022%3A10%3A16.png)
    - mc.sync_send_angles, 是一个同步方法, 在发送控制信号后, 通过is_in_position等待机械臂达到位置
      - is_in_position: 是比较了角度值是否相等
        - <https://github.com/elephantrobotics/pymycobot/blob/40394ba8915764e6b0441b38d88fed1c2bedbf39/pymycobot/elephantrobot.py#L412>
    - 关于moveit调用方的超时原因查询: 
      - \- 具体机制：

\- MoveIt 会用规划轨迹的预计时长 t，计算允许执行上限时间  
T_allowed = t × trajectory_execution.allowed_execution_duration_scaling + trajectory_execution.allowed_goal_duration_margin  
\- 超过这个上限，TrajectoryExecutionManager 会停止轨迹

对照一个包含超时参数的示例（同仓库里另一套配置）：

```1:6:/data/huangyan/isaac_ros-dev/src/isaac_manipulator/isaac_manipulator_pick_and_place/config/moveit_controllers.yaml  
trajectory_execution:  
execution_duration_monitoring: False  
allowed_execution_duration_scaling: 1.2  
allowed_goal_duration_margin: 0.5  
allowed_start_tolerance: 0.01  
```

\- 要点结论  
\- “这个超时是谁设置的”：由 MoveIt 的 TrajectoryExecutionManager 按上述两个参数计算（若未配置则用 MoveIt 默认值）。  
\- 如需调整，给你的配置文件补上 `trajectory_execution:` 段（或在 `move_group` 的参数里覆盖这两个参数），即可改变超时上限。

      - 使用sync_send_angles + 调大trajectory_execution的超时时间, 可以正确完成轨迹, 但感觉一卡一卡
        - commit: 499ed7d6d05e182e3407a69935abb975e7db0820  

  - slider重构, 以解决碰到的问题: 
    - commit: 939fd8b35e62b498b25866eb5b1be2de993b71a5
    - 目前使用的slider, 模式为: 
      - mc.set_fresh_mode(1), 优先遵从最新指令, 而不是使用硬件队列
        - 使用mc.set_fresh_mode(0)时, 更容易出现以下情况: 
          - 当设备上接到的命令发生饱和时, 后续的指令下发耗时就会增加 (但机械臂会一卡一卡, 好像是在等队列进入新的指令)

  
[INFO] [1763115876.645026492] [slider_action_controller]: 收到新轨迹目标  
[INFO] [1763115876.647292490] [slider_action_controller]: 开始执行手臂轨迹  
[INFO] [1763115876.760730647] [slider_action_controller]: ARM_SEND point 0/40 (async) 耗时: 0.111251s  
[INFO] [1763115876.773954081] [slider_action_controller]: ARM_SEND point 1/40 (async) 耗时: 0.011001s  
[INFO] [1763115876.776514710] [slider_action_controller]: ARM_SEND point 2/40 (async) 耗时: 0.000501s  
[INFO] [1763115876.778256334] [slider_action_controller]: ARM_SEND point 3/40 (async) 耗时: 0.000579s  
[INFO] [1763115876.779368330] [slider_action_controller]: ARM_SEND point 4/40 (async) 耗时: 0.000160s  
[INFO] [1763115876.780499899] [slider_action_controller]: ARM_SEND point 5/40 (async) 耗时: 0.000181s  
[INFO] [1763115876.781598143] [slider_action_controller]: ARM_SEND point 6/40 (async) 耗时: 0.000176s  
[INFO] [1763115876.782697143] [slider_action_controller]: ARM_SEND point 7/40 (async) 耗时: 0.000177s  
[INFO] [1763115876.783997938] [slider_action_controller]: ARM_SEND point 8/40 (async) 耗时: 0.000507s  
[INFO] [1763115876.785404513] [slider_action_controller]: ARM_SEND point 9/40 (async) 耗时: 0.000791s  
[INFO] [1763115876.888589329] [slider_action_controller]: ARM_SEND point 10/40 (async) 耗时: 0.101116s  
[INFO] [1763115876.897097307] [slider_action_controller]: ARM_SEND point 11/40 (async) 耗时: 0.000447s  
[INFO] [1763115876.899701581] [slider_action_controller]: ARM_SEND point 12/40 (async) 耗时: 0.000555s  
[INFO] [1763115876.903027934] [slider_action_controller]: ARM_SEND point 13/40 (async) 耗时: 0.000374s  
[INFO] [1763115876.906525468] [slider_action_controller]: ARM_SEND point 14/40 (async) 耗时: 0.000560s  
[INFO] [1763115876.909069413] [slider_action_controller]: ARM_SEND point 15/40 (async) 耗时: 0.000376s  
[INFO] [1763115876.912172410] [slider_action_controller]: ARM_SEND point 16/40 (async) 耗时: 0.000399s  
[INFO] [1763115876.914543468] [slider_action_controller]: ARM_SEND point 17/40 (async) 耗时: 0.000465s  
[INFO] [1763115876.916931356] [slider_action_controller]: ARM_SEND point 18/40 (async) 耗时: 0.000379s  
[INFO] [1763115876.919200616] [slider_action_controller]: ARM_SEND point 19/40 (async) 耗时: 0.000373s  
[INFO] [1763115877.022017177] [slider_action_controller]: ARM_SEND point 20/40 (async) 耗时: 0.100662s  
[INFO] [1763115879.966880988] [slider_action_controller]: ARM_SEND point 21/40 (async) 耗时: 2.941700s  
[INFO] [1763115881.629502294] [slider_action_controller]: ARM_SEND point 22/40 (async) 耗时: 1.659866s  
[INFO] [1763115883.424510148] [slider_action_controller]: ARM_SEND point 23/40 (async) 耗时: 1.791838s  
[INFO] [1763115883.702434855] [slider_action_controller]: ARM_SEND point 24/40 (async) 耗时: 0.274763s  
[INFO] [1763115884.313156909] [slider_action_controller]: ARM_SEND point 25/40 (async) 耗时: 0.607874s  
[INFO] [1763115884.866282111] [slider_action_controller]: ARM_SEND point 26/40 (async) 耗时: 0.550010s  
[INFO] [1763115885.556616648] [slider_action_controller]: ARM_SEND point 27/40 (async) 耗时: 0.687338s  
[INFO] [1763115886.164179170] [slider_action_controller]: ARM_SEND point 28/40 (async) 耗时: 0.604597s  
[INFO] [1763115886.807976106] [slider_action_controller]: ARM_SEND point 29/40 (async) 耗时: 0.640965s  
[INFO] [1763115887.509485380] [slider_action_controller]: ARM_SEND point 30/40 (async) 耗时: 0.700362s

      - mc.send_angles(angles_deg, 35, _async=True), 使用异步发送
        - 如果使用同步发送, 那么每一步的延迟都不低, 机械臂会一卡一卡 (等待下一个命令?):
          -   
[INFO] [1763116039.337169913] [slider_action_controller]: 收到新轨迹目标  
[INFO] [1763116039.339061464] [slider_action_controller]: 开始执行手臂轨迹  
[INFO] [1763116039.477214350] [slider_action_controller]: ARM_SEND point 0/40 (async) 耗时: 0.132718s  
[INFO] [1763116039.499342998] [slider_action_controller]: ARM_SEND point 1/40 (async) 耗时: 0.019501s  
[INFO] [1763116039.524056141] [slider_action_controller]: ARM_SEND point 2/40 (async) 耗时: 0.021804s  
[INFO] [1763116039.554251936] [slider_action_controller]: ARM_SEND point 3/40 (async) 耗时: 0.027499s  
[INFO] [1763116042.072194503] [slider_action_controller]: ARM_SEND point 4/40 (async) 耗时: 2.514734s  
[INFO] [1763116042.192306127] [slider_action_controller]: ARM_SEND point 5/40 (async) 耗时: 0.117239s  
[INFO] [1763116042.808298478] [slider_action_controller]: ARM_SEND point 6/40 (async) 耗时: 0.613114s  
[INFO] [1763116043.360680761] [slider_action_controller]: ARM_SEND point 7/40 (async) 耗时: 0.549314s  
[INFO] [1763116043.975331599] [slider_action_controller]: ARM_SEND point 8/40 (async) 耗时: 0.611724s  
[INFO] [1763116044.578423115] [slider_action_controller]: ARM_SEND point 9/40 (async) 耗时: 0.600333s  
[INFO] [1763116045.274806252] [slider_action_controller]: ARM_SEND point 10/40 (async) 耗时: 0.694155s  
[INFO] [1763116045.451011358] [slider_action_controller]: ARM_SEND point 11/40 (async) 耗时: 0.173511s  
[INFO] [1763116048.110982810] [slider_action_controller]: ARM_SEND point 12/40 (async) 耗时: 2.657333s  
[INFO] [1763116048.771077258] [slider_action_controller]: ARM_SEND point 13/40 (async) 耗时: 0.657355s  
[INFO] [1763116049.389016082] [slider_action_controller]: ARM_SEND point 14/40 (async) 耗时: 0.615027s  
[INFO] [1763116050.489133379] [slider_action_controller]: ARM_SEND point 15/40 (async) 耗时: 1.097379s  
[INFO] [1763116051.109014402] [slider_action_controller]: ARM_SEND point 16/40 (async) 耗时: 0.617012s  
[INFO] [1763116051.301475803] [slider_action_controller]: ARM_SEND point 17/40 (async) 耗时: 0.189567s  
[INFO] [1763116051.943024312] [slider_action_controller]: ARM_SEND point 18/40 (async) 耗时: 0.638938s  
[INFO] [1763116052.652137584] [slider_action_controller]: ARM_SEND point 19/40 (async) 耗时: 0.705437s  
[INFO] [1763116053.410942995] [slider_action_controller]: ARM_SEND point 20/40 (async) 耗时: 0.756411s

          -       - 使用 优先遵从最新指令 + 异步发送指令, 认为可能路径不得遵循, 在路径中会偶发延迟升高: 
        -   
[INFO] [1763116248.923443518] [slider_action_controller]: 开始执行手臂轨迹  
[INFO] [1763116249.030289577] [slider_action_controller]: ARM_SEND point 0/39 (async) 耗时: 0.104433s  
[INFO] [1763116249.073925174] [slider_action_controller]: ARM_SEND point 1/39 (async) 耗时: 0.040924s  
[INFO] [1763116249.112723433] [slider_action_controller]: ARM_SEND point 2/39 (async) 耗时: 0.033744s  
[INFO] [1763116249.116908243] [slider_action_controller]: ARM_SEND point 3/39 (async) 耗时: 0.001975s  
[INFO] [1763116249.120235429] [slider_action_controller]: ARM_SEND point 4/39 (async) 耗时: 0.000508s  
[INFO] [1763116249.122691273] [slider_action_controller]: ARM_SEND point 5/39 (async) 耗时: 0.000481s  
[INFO] [1763116249.125144233] [slider_action_controller]: ARM_SEND point 6/39 (async) 耗时: 0.000430s  
[INFO] [1763116249.127524570] [slider_action_controller]: ARM_SEND point 7/39 (async) 耗时: 0.000431s  
[INFO] [1763116249.129963214] [slider_action_controller]: ARM_SEND point 8/39 (async) 耗时: 0.000516s  
[INFO] [1763116249.132755871] [slider_action_controller]: ARM_SEND point 9/39 (async) 耗时: 0.000404s  
[INFO] [1763116249.235577099] [slider_action_controller]: ARM_SEND point 10/39 (async) 耗时: 0.100641s  
[INFO] [1763116252.183823320] [slider_action_controller]: ARM_SEND point 11/39 (async) 耗时: 2.945030s  
[INFO] [1763116253.287028756] [slider_action_controller]: ARM_SEND point 12/39 (async) 耗时: 1.100646s  
[INFO] [1763116253.882929731] [slider_action_controller]: ARM_SEND point 13/39 (async) 耗时: 0.593925s  
[INFO] [1763116253.991271056] [slider_action_controller]: ARM_SEND point 14/39 (async) 耗时: 0.105686s  
[INFO] [1763116254.136831101] [slider_action_controller]: ARM_SEND point 15/39 (async) 耗时: 0.142941s  
[INFO] [1763116254.716920708] [slider_action_controller]: ARM_SEND point 16/39 (async) 耗时: 0.577362s  
[INFO] [1763116254.814427970] [slider_action_controller]: ARM_SEND point 17/39 (async) 耗时: 0.094597s  
[INFO] [1763116255.395885763] [slider_action_controller]: ARM_SEND point 18/39 (async) 耗时: 0.578685s  
[INFO] [1763116255.518966445] [slider_action_controller]: ARM_SEND point 19/39 (async) 耗时: 0.120331s  
[INFO] [1763116255.711429621] [slider_action_controller]: ARM_SEND point 20/39 (async) 耗时: 0.187635s  
[INFO] [1763116255.731201841] [slider_action_controller]: ARM_SEND point 21/39 (async) 耗时: 0.016904s  
[INFO] [1763116255.740527352] [slider_action_controller]: ARM_SEND point 22/39 (async) 耗时: 0.006550s  
[INFO] [1763116255.743038569] [slider_action_controller]: ARM_SEND point 23/39 (async) 耗时: 0.000485s  
[INFO] [1763116255.745727120] [slider_action_controller]: ARM_SEND point 24/39 (async) 耗时: 0.000744s  
[INFO] [1763116255.748156109] [slider_action_controller]: ARM_SEND point 25/39 (async) 耗时: 0.000447s  
[INFO] [1763116255.752662221] [slider_action_controller]: ARM_SEND point 26/39 (async) 耗时: 0.002474s  
[INFO] [1763116255.755060593] [slider_action_controller]: ARM_SEND point 27/39 (async) 耗时: 0.000450s  
[INFO] [1763116255.762087073] [slider_action_controller]: ARM_SEND point 28/39 (async) 耗时: 0.000417s  
[INFO] [1763116255.764876205] [slider_action_controller]: ARM_SEND point 29/39 (async) 耗时: 0.000628s  
[INFO] [1763116255.867855890] [slider_action_controller]: ARM_SEND point 30/39 (async) 耗时: 0.100703s  
[INFO] [1763116256.382857067] [slider_action_controller]: ARM_SEND point 31/39 (async) 耗时: 0.512097s  
[INFO] [1763116256.987656139] [slider_action_controller]: ARM_SEND point 32/39 (async) 耗时: 0.601895s  
[INFO] [1763116257.129736960] [slider_action_controller]: ARM_SEND point 33/39 (async) 耗时: 0.138410s  
[INFO] [1763116257.687249442] [slider_action_controller]: ARM_SEND point 34/39 (async) 耗时: 0.554747s  
[INFO] [1763116262.271026791] [slider_action_controller]: ARM_SEND point 35/39 (async) 耗时: 4.581195s  
[INFO] [1763116262.331250810] [slider_action_controller]: ARM_SEND point 36/39 (async) 耗时: 0.057522s  
[INFO] [1763116262.908388542] [slider_action_controller]: ARM_SEND point 37/39 (async) 耗时: 0.575135s

      - 另外: 编写了sync_send_angles_with_tolerance, 机械臂自带的sync_send_angles, 其不容差, 导致会一直等待
        - 在最后一个节点, 进行路径等待, 需要重试+容差, 确保位置

一些实验想法: 

  - 降低波特率
  - (mycobot280-M5通讯频率是10-20Hz), 需要穷举变量: 发送命令的间隔时间, send_angles是否是异步的, set_fresh_mode取值

穷举测试: 

  - -p fresh_mode:=1 -p arm_point_sleep_interval:=0.1 -p arm_async_send:=True

```
[INFO] [1763179363.245223492] [slider_action_controller]: === 初始化完成，等待 MoveIt 连接 ===
[INFO] [1763179385.141310737] [slider_action_controller]: 收到新轨迹目标
[INFO] [1763179385.151019433] [slider_action_controller]: 开始执行手臂轨迹
[INFO] [1763179385.262324919] [slider_action_controller]: ARM_SEND point 0/23 (async) 耗时: 0.108665s
[INFO] [1763179385.378634593] [slider_action_controller]: ARM_SEND point 1/23 (async) 耗时: 0.114050s
[INFO] [1763179385.508302057] [slider_action_controller]: ARM_SEND point 2/23 (async) 耗时: 0.127244s
[INFO] [1763179385.613133688] [slider_action_controller]: ARM_SEND point 3/23 (async) 耗时: 0.102015s
[INFO] [1763179385.728791166] [slider_action_controller]: ARM_SEND point 4/23 (async) 耗时: 0.112951s
[INFO] [1763179387.310260611] [slider_action_controller]: ARM_SEND point 5/23 (async) 耗时: 1.578215s
[INFO] [1763179392.391706127] [slider_action_controller]: ARM_SEND point 6/23 (async) 耗时: 5.079107s
[INFO] [1763179395.492881651] [slider_action_controller]: ARM_SEND point 7/23 (async) 耗时: 3.098783s
[INFO] [1763179396.094277561] [slider_action_controller]: ARM_SEND point 8/23 (async) 耗时: 0.598785s
[INFO] [1763179396.706826034] [slider_action_controller]: ARM_SEND point 9/23 (async) 耗时: 0.609995s
[INFO] [1763179398.310130565] [slider_action_controller]: ARM_SEND point 10/23 (async) 耗时: 1.601007s
[INFO] [1763179398.964789867] [slider_action_controller]: ARM_SEND point 11/23 (async) 耗时: 0.652256s
[INFO] [1763179399.577833869] [slider_action_controller]: ARM_SEND point 12/23 (async) 耗时: 0.610760s
[INFO] [1763179400.194165094] [slider_action_controller]: ARM_SEND point 13/23 (async) 耗时: 0.613428s
[INFO] [1763179400.381503868] [slider_action_controller]: ARM_SEND point 14/23 (async) 耗时: 0.184933s
[INFO] [1763179400.967464988] [slider_action_controller]: ARM_SEND point 15/23 (async) 耗时: 0.583736s
[INFO] [1763179401.167108927] [slider_action_controller]: ARM_SEND point 16/23 (async) 耗时: 0.197381s
[INFO] [1763179401.829075431] [slider_action_controller]: ARM_SEND point 17/23 (async) 耗时: 0.659532s
[INFO] [1763179402.461795712] [slider_action_controller]: ARM_SEND point 18/23 (async) 耗时: 0.630110s
[INFO] [1763179403.073802889] [slider_action_controller]: ARM_SEND point 19/23 (async) 耗时: 0.609572s
[INFO] [1763179403.687531205] [slider_action_controller]: ARM_SEND point 20/23 (async) 耗时: 0.611387s
[INFO] [1763179404.285840862] [slider_action_controller]: ARM_SEND point 21/23 (async) 耗时: 0.595958s
[INFO] [1763179404.891002977] [slider_action_controller]: ARM_SYNC point 22/23: 成功到位 (尝试次数: 1)
[INFO] [1763179405.005620100] [slider_action_controller]: ARM_SYNC point 22/23: 目标=[63.0, 42.2, 46.2, -83.6, -48.8, -104.4], 实际=[63.0, 42.4, 46.3, -84.1, -49.0, -103.7], 差值=[0.06, 0.19, 0.13, -0.5, -0.28, 0.69], 最大差=0.69°
[INFO] [1763179405.109906696] [slider_action_controller]: ARM_SEND point 22/23 (sync, result=0) 耗时: 0.819845s
[INFO] [1763179405.112277665] [slider_action_controller]: 手臂轨迹发送完成，总耗时: 19.958750s
```

```
[INFO] [1763179439.843906514] [slider_action_controller]: 收到新轨迹目标
[INFO] [1763179439.852094233] [slider_action_controller]: 开始执行手臂轨迹
[INFO] [1763179439.962663399] [slider_action_controller]: ARM_SEND point 0/29 (async) 耗时: 0.108315s
[INFO] [1763179440.076134103] [slider_action_controller]: ARM_SEND point 1/29 (async) 耗时: 0.111234s
[INFO] [1763179440.179204073] [slider_action_controller]: ARM_SEND point 2/29 (async) 耗时: 0.100766s
[INFO] [1763179440.310696112] [slider_action_controller]: ARM_SEND point 3/29 (async) 耗时: 0.127242s
[INFO] [1763179440.426481160] [slider_action_controller]: ARM_SEND point 4/29 (async) 耗时: 0.113445s
[INFO] [1763179440.529891765] [slider_action_controller]: ARM_SEND point 5/29 (async) 耗时: 0.101090s
[INFO] [1763179440.633011351] [slider_action_controller]: ARM_SEND point 6/29 (async) 耗时: 0.100665s
[INFO] [1763179440.736198628] [slider_action_controller]: ARM_SEND point 7/29 (async) 耗时: 0.100823s
[INFO] [1763179440.839471803] [slider_action_controller]: ARM_SEND point 8/29 (async) 耗时: 0.100855s
[INFO] [1763179440.958413178] [slider_action_controller]: ARM_SEND point 9/29 (async) 耗时: 0.116313s
[INFO] [1763179441.508597507] [slider_action_controller]: ARM_SEND point 10/29 (async) 耗时: 0.547405s
[INFO] [1763179441.721330564] [slider_action_controller]: ARM_SEND point 11/29 (async) 耗时: 0.210007s
[INFO] [1763179442.313296572] [slider_action_controller]: ARM_SEND point 12/29 (async) 耗时: 0.589563s
[INFO] [1763179442.919405323] [slider_action_controller]: ARM_SEND point 13/29 (async) 耗时: 0.603542s
[INFO] [1763179443.522185734] [slider_action_controller]: ARM_SEND point 14/29 (async) 耗时: 0.600542s
[INFO] [1763179444.127438056] [slider_action_controller]: ARM_SEND point 15/29 (async) 耗时: 0.602979s
[INFO] [1763179444.719496343] [slider_action_controller]: ARM_SEND point 16/29 (async) 耗时: 0.589369s
[INFO] [1763179445.324282823] [slider_action_controller]: ARM_SEND point 17/29 (async) 耗时: 0.602407s
[INFO] [1763179445.470112234] [slider_action_controller]: ARM_SEND point 18/29 (async) 耗时: 0.143553s
[INFO] [1763179446.044407178] [slider_action_controller]: ARM_SEND point 19/29 (async) 耗时: 0.571930s
[INFO] [1763179446.665457175] [slider_action_controller]: ARM_SEND point 20/29 (async) 耗时: 0.618684s
[INFO] [1763179448.812293360] [slider_action_controller]: ARM_SEND point 21/29 (async) 耗时: 2.144540s
[INFO] [1763179449.417892561] [slider_action_controller]: ARM_SEND point 22/29 (async) 耗时: 0.604429s
[INFO] [1763179449.585314800] [slider_action_controller]: ARM_SEND point 23/29 (async) 耗时: 0.165111s
[INFO] [1763179450.149196796] [slider_action_controller]: ARM_SEND point 24/29 (async) 耗时: 0.561579s
[INFO] [1763179450.341211600] [slider_action_controller]: ARM_SEND point 25/29 (async) 耗时: 0.189746s
[INFO] [1763179450.464145939] [slider_action_controller]: ARM_SEND point 26/29 (async) 耗时: 0.120660s
[INFO] [1763179451.053697357] [slider_action_controller]: ARM_SEND point 27/29 (async) 耗时: 0.587218s
[INFO] [1763179452.123632104] [slider_action_controller]: ARM_SYNC point 28/29: 成功到位 (尝试次数: 1)
[INFO] [1763179452.230156655] [slider_action_controller]: ARM_SYNC point 28/29: 目标=[63.0, 42.4, 46.3, -84.0, -49.0, -103.7], 实际=[62.7, 42.0, 46.0, -84.3, -49.0, -103.0], 差值=[-0.35, -0.35, -0.35, -0.26, 0.1, 0.71], 最大差=0.71°
[INFO] [1763179452.332922721] [slider_action_controller]: ARM_SEND point 28/29 (sync, result=0) 耗时: 1.276845s
[INFO] [1763179452.333967315] [slider_action_controller]: 手臂轨迹发送完成，总耗时: 12.480819s
```

    - 感觉卡顿比较明显

  - -p fresh_mode:=0 -p arm_point_sleep_interval:=0.1 -p arm_async_send:=True

```
[INFO] [1763179613.126385882] [slider_action_controller]: === 初始化完成，等待 MoveIt 连接 ===
[INFO] [1763179620.774184107] [slider_action_controller]: 收到新轨迹目标
[INFO] [1763179620.781737382] [slider_action_controller]: 开始执行手臂轨迹
[INFO] [1763179620.891996470] [slider_action_controller]: ARM_SEND point 0/29 (async) 耗时: 0.107624s
[INFO] [1763179621.007582566] [slider_action_controller]: ARM_SEND point 1/29 (async) 耗时: 0.112553s
[INFO] [1763179621.110804458] [slider_action_controller]: ARM_SEND point 2/29 (async) 耗时: 0.100959s
[INFO] [1763179621.241789863] [slider_action_controller]: ARM_SEND point 3/29 (async) 耗时: 0.126676s
[INFO] [1763179621.357746476] [slider_action_controller]: ARM_SEND point 4/29 (async) 耗时: 0.113660s
[INFO] [1763179621.461146217] [slider_action_controller]: ARM_SEND point 5/29 (async) 耗时: 0.100984s
[INFO] [1763179622.059473088] [slider_action_controller]: ARM_SEND point 6/29 (async) 耗时: 0.595794s
[INFO] [1763179622.680422684] [slider_action_controller]: ARM_SEND point 7/29 (async) 耗时: 0.618305s
[INFO] [1763179623.289218301] [slider_action_controller]: ARM_SEND point 8/29 (async) 耗时: 0.606411s
[INFO] [1763179623.953779613] [slider_action_controller]: ARM_SEND point 9/29 (async) 耗时: 0.661954s
[INFO] [1763179624.556736878] [slider_action_controller]: ARM_SEND point 10/29 (async) 耗时: 0.600622s
[INFO] [1763179625.166048204] [slider_action_controller]: ARM_SEND point 11/29 (async) 耗时: 0.606760s
[INFO] [1763179625.766433619] [slider_action_controller]: ARM_SEND point 12/29 (async) 耗时: 0.597754s
[INFO] [1763179626.370309979] [slider_action_controller]: ARM_SEND point 13/29 (async) 耗时: 0.601436s
[INFO] [1763179627.087994888] [slider_action_controller]: ARM_SEND point 14/29 (async) 耗时: 0.715170s
[INFO] [1763179627.699355770] [slider_action_controller]: ARM_SEND point 15/29 (async) 耗时: 0.609111s
[INFO] [1763179628.346621636] [slider_action_controller]: ARM_SEND point 16/29 (async) 耗时: 0.645006s
[INFO] [1763179628.957746971] [slider_action_controller]: ARM_SEND point 17/29 (async) 耗时: 0.608637s
[INFO] [1763179629.630698541] [slider_action_controller]: ARM_SEND point 18/29 (async) 耗时: 0.667512s
[INFO] [1763179630.234861501] [slider_action_controller]: ARM_SEND point 19/29 (async) 耗时: 0.601684s
[INFO] [1763179630.845777702] [slider_action_controller]: ARM_SEND point 20/29 (async) 耗时: 0.608296s
[INFO] [1763179631.451566561] [slider_action_controller]: ARM_SEND point 21/29 (async) 耗时: 0.603514s
[INFO] [1763179632.080866351] [slider_action_controller]: ARM_SEND point 22/29 (async) 耗时: 0.626678s
[INFO] [1763179632.702833141] [slider_action_controller]: ARM_SEND point 23/29 (async) 耗时: 0.619338s
[INFO] [1763179633.320067586] [slider_action_controller]: ARM_SEND point 24/29 (async) 耗时: 0.614509s
[INFO] [1763179633.944622884] [slider_action_controller]: ARM_SEND point 25/29 (async) 耗时: 0.622228s
[INFO] [1763179634.623963832] [slider_action_controller]: ARM_SEND point 26/29 (async) 耗时: 0.677015s
[INFO] [1763179634.788040184] [slider_action_controller]: ARM_SEND point 27/29 (async) 耗时: 0.161506s
[WARN] [1763179635.791358424] [slider_action_controller]: ARM_SYNC point 28/29: 第 1 次重试
[INFO] [1763179636.494237632] [slider_action_controller]: ARM_SYNC point 28/29: 成功到位 (尝试次数: 2)
[INFO] [1763179636.586341128] [slider_action_controller]: ARM_SYNC point 28/29: 目标=[63.3, 41.8, 45.5, -84.6, -49.1, -102.4], 实际=[63.8, 41.5, 45.6, -84.9, -49.1, -101.7], 差值=[0.52, -0.35, 0.09, -0.27, -0.01, 0.71], 最大差=0.71°
[INFO] [1763179636.689076415] [slider_action_controller]: ARM_SEND point 28/29 (sync, result=0) 耗时: 1.898705s
```

```
[INFO] [1763179654.169509449] [slider_action_controller]: 收到新轨迹目标
[INFO] [1763179654.177568161] [slider_action_controller]: 开始执行手臂轨迹
[INFO] [1763179654.292653890] [slider_action_controller]: ARM_SEND point 0/29 (async) 耗时: 0.112368s
[INFO] [1763179654.834618925] [slider_action_controller]: ARM_SEND point 1/29 (async) 耗时: 0.539635s
[INFO] [1763179655.023148602] [slider_action_controller]: ARM_SEND point 2/29 (async) 耗时: 0.186011s
[INFO] [1763179655.619193339] [slider_action_controller]: ARM_SEND point 3/29 (async) 耗时: 0.593515s
[INFO] [1763179656.234967569] [slider_action_controller]: ARM_SEND point 4/29 (async) 耗时: 0.613236s
[INFO] [1763179656.883208314] [slider_action_controller]: ARM_SEND point 5/29 (async) 耗时: 0.645651s
[INFO] [1763179657.491531385] [slider_action_controller]: ARM_SEND point 6/29 (async) 耗时: 0.605333s
[INFO] [1763179658.087588366] [slider_action_controller]: ARM_SEND point 7/29 (async) 耗时: 0.593757s
[INFO] [1763179658.700304928] [slider_action_controller]: ARM_SEND point 8/29 (async) 耗时: 0.610362s
[INFO] [1763179659.363657581] [slider_action_controller]: ARM_SEND point 9/29 (async) 耗时: 0.661124s
[INFO] [1763179660.010253469] [slider_action_controller]: ARM_SEND point 10/29 (async) 耗时: 0.643990s
[INFO] [1763179660.631700160] [slider_action_controller]: ARM_SEND point 11/29 (async) 耗时: 0.619180s
[INFO] [1763179661.296653497] [slider_action_controller]: ARM_SEND point 12/29 (async) 耗时: 0.662557s
[INFO] [1763179661.924423942] [slider_action_controller]: ARM_SEND point 13/29 (async) 耗时: 0.625264s
[INFO] [1763179663.515788499] [slider_action_controller]: ARM_SEND point 14/29 (async) 耗时: 1.589031s
[INFO] [1763179663.666139224] [slider_action_controller]: ARM_SEND point 15/29 (async) 耗时: 0.147913s
[INFO] [1763179664.730560717] [slider_action_controller]: ARM_SEND point 16/29 (async) 耗时: 1.062166s
[INFO] [1763179665.347459199] [slider_action_controller]: ARM_SEND point 17/29 (async) 耗时: 0.614326s
[INFO] [1763179666.150206055] [slider_action_controller]: ARM_SEND point 18/29 (async) 耗时: 0.800086s
[INFO] [1763179667.712027308] [slider_action_controller]: ARM_SEND point 19/29 (async) 耗时: 1.559208s
[INFO] [1763179668.319708744] [slider_action_controller]: ARM_SEND point 20/29 (async) 耗时: 0.604729s
[INFO] [1763179668.917988293] [slider_action_controller]: ARM_SEND point 21/29 (async) 耗时: 0.595603s
[INFO] [1763179670.997680647] [slider_action_controller]: ARM_SEND point 22/29 (async) 耗时: 2.076712s
[INFO] [1763179675.583173883] [slider_action_controller]: ARM_SEND point 23/29 (async) 耗时: 4.582510s
[INFO] [1763179676.180866219] [slider_action_controller]: ARM_SEND point 24/29 (async) 耗时: 0.594791s
[INFO] [1763179676.783425922] [slider_action_controller]: ARM_SEND point 25/29 (async) 耗时: 0.599406s
[INFO] [1763179677.365661290] [slider_action_controller]: ARM_SEND point 26/29 (async) 耗时: 0.579264s
[INFO] [1763179677.965232167] [slider_action_controller]: ARM_SEND point 27/29 (async) 耗时: 0.596903s
[INFO] [1763179678.474230366] [slider_action_controller]: ARM_SYNC point 28/29: 成功到位 (尝试次数: 1)
[INFO] [1763179679.085892560] [slider_action_controller]: ARM_SYNC point 28/29: 目标=[0.0, -0.0, 0.0, -0.0, -0.0, -0.0], 实际=[0.1, -0.2, -0.3, -0.3, -0.3, -0.6], 差值=[0.07, -0.17, -0.26, -0.35, -0.26, -0.6], 最大差=0.60°
[INFO] [1763179679.188638453] [slider_action_controller]: ARM_SEND point 28/29 (sync, result=0) 耗时: 1.221005s
[INFO] [1763179679.193865213] [slider_action_controller]: 手臂轨迹发送完成，总耗时: 25.011008s
```

    - 卡顿比较平均, 每一步都卡顿  
  

  - -p fresh_mode:=1 -p arm_point_sleep_interval:=0.1 -p arm_async_send:=False

```
[INFO] [1763179769.393224291] [slider_action_controller]: 收到新轨迹目标
[INFO] [1763179769.403952906] [slider_action_controller]: 开始执行手臂轨迹
[INFO] [1763179769.521540313] [slider_action_controller]: ARM_SEND point 0/29 (async) 耗时: 0.115072s
[INFO] [1763179770.074134515] [slider_action_controller]: ARM_SEND point 1/29 (async) 耗时: 0.549984s
[INFO] [1763179770.699190301] [slider_action_controller]: ARM_SEND point 2/29 (async) 耗时: 0.621338s
[INFO] [1763179771.307801960] [slider_action_controller]: ARM_SEND point 3/29 (async) 耗时: 0.606026s
[INFO] [1763179775.414482378] [slider_action_controller]: ARM_SEND point 4/29 (async) 耗时: 4.104493s
[INFO] [1763179776.039770424] [slider_action_controller]: ARM_SEND point 5/29 (async) 耗时: 0.622972s
[INFO] [1763179776.228154282] [slider_action_controller]: ARM_SEND point 6/29 (async) 耗时: 0.185762s
[INFO] [1763179776.864490740] [slider_action_controller]: ARM_SEND point 7/29 (async) 耗时: 0.630409s
[INFO] [1763179777.059316531] [slider_action_controller]: ARM_SEND point 8/29 (async) 耗时: 0.191828s
[INFO] [1763179777.672491157] [slider_action_controller]: ARM_SEND point 9/29 (async) 耗时: 0.610777s
[INFO] [1763179778.301535922] [slider_action_controller]: ARM_SEND point 10/29 (async) 耗时: 0.626519s
[INFO] [1763179778.915022245] [slider_action_controller]: ARM_SEND point 11/29 (async) 耗时: 0.610854s
[INFO] [1763179779.556012187] [slider_action_controller]: ARM_SEND point 12/29 (async) 耗时: 0.636297s
[INFO] [1763179779.751878567] [slider_action_controller]: ARM_SEND point 13/29 (async) 耗时: 0.193369s
[INFO] [1763179779.896875680] [slider_action_controller]: ARM_SEND point 14/29 (async) 耗时: 0.141442s
[INFO] [1763179780.468944411] [slider_action_controller]: ARM_SEND point 15/29 (async) 耗时: 0.569533s
[INFO] [1763179781.079623598] [slider_action_controller]: ARM_SEND point 16/29 (async) 耗时: 0.607840s
[INFO] [1763179781.689205617] [slider_action_controller]: ARM_SEND point 17/29 (async) 耗时: 0.606884s
[INFO] [1763179782.301794262] [slider_action_controller]: ARM_SEND point 18/29 (async) 耗时: 0.609683s
[INFO] [1763179782.918852686] [slider_action_controller]: ARM_SEND point 19/29 (async) 耗时: 0.614248s
[INFO] [1763179783.528759370] [slider_action_controller]: ARM_SEND point 20/29 (async) 耗时: 0.607732s
[INFO] [1763179784.209196859] [slider_action_controller]: ARM_SEND point 21/29 (async) 耗时: 0.677571s
[INFO] [1763179784.823428695] [slider_action_controller]: ARM_SEND point 22/29 (async) 耗时: 0.611264s
[INFO] [1763179785.433708417] [slider_action_controller]: ARM_SEND point 23/29 (async) 耗时: 0.607723s
[INFO] [1763179786.060080977] [slider_action_controller]: ARM_SEND point 24/29 (async) 耗时: 0.623994s
[INFO] [1763179786.230140683] [slider_action_controller]: ARM_SEND point 25/29 (async) 耗时: 0.167701s
[INFO] [1763179786.349091290] [slider_action_controller]: ARM_SEND point 26/29 (async) 耗时: 0.113081s
[INFO] [1763179786.913234090] [slider_action_controller]: ARM_SEND point 27/29 (async) 耗时: 0.561312s
[WARN] [1763179789.424718567] [slider_action_controller]: ARM_SYNC point 28/29: 第 1 次重试
[INFO] [1763179790.052400693] [slider_action_controller]: ARM_SYNC point 28/29: 成功到位 (尝试次数: 2)
[INFO] [1763179790.667774997] [slider_action_controller]: ARM_SYNC point 28/29: 目标=[63.8, 41.7, 45.6, -84.9, -49.0, -101.7], 实际=[63.8, 42.2, 45.3, -84.5, -48.9, -101.0], 差值=[-0.0, 0.52, -0.35, 0.35, 0.18, 0.7], 最大差=0.70°
[INFO] [1763179790.770114347] [slider_action_controller]: ARM_SEND point 28/29 (sync, result=0) 耗时: 3.854610s
[INFO] [1763179790.772236395] [slider_action_controller]: 手臂轨迹发送完成，总耗时: 21.366087s
```

```
[INFO] [1763179832.542831355] [slider_action_controller]: 收到新轨迹目标
[INFO] [1763179832.551247217] [slider_action_controller]: 开始执行手臂轨迹
[INFO] [1763179832.670266730] [slider_action_controller]: ARM_SEND point 0/29 (async) 耗时: 0.116566s
[INFO] [1763179833.222432510] [slider_action_controller]: ARM_SEND point 1/29 (async) 耗时: 0.549315s
[INFO] [1763179833.826876883] [slider_action_controller]: ARM_SEND point 2/29 (async) 耗时: 0.601558s
[INFO] [1763179834.435382963] [slider_action_controller]: ARM_SEND point 3/29 (async) 耗时: 0.605990s
[INFO] [1763179835.041390026] [slider_action_controller]: ARM_SEND point 4/29 (async) 耗时: 0.603133s
[INFO] [1763179835.653099392] [slider_action_controller]: ARM_SEND point 5/29 (async) 耗时: 0.603512s
[INFO] [1763179835.857265600] [slider_action_controller]: ARM_SEND point 6/29 (async) 耗时: 0.201931s
[INFO] [1763179836.016567910] [slider_action_controller]: ARM_SEND point 7/29 (async) 耗时: 0.156446s
[INFO] [1763179836.610163488] [slider_action_controller]: ARM_SEND point 8/29 (async) 耗时: 0.590678s
[INFO] [1763179837.216559240] [slider_action_controller]: ARM_SEND point 9/29 (async) 耗时: 0.603778s
[INFO] [1763179837.362978381] [slider_action_controller]: ARM_SEND point 10/29 (async) 耗时: 0.143564s
[INFO] [1763179837.938943668] [slider_action_controller]: ARM_SEND point 11/29 (async) 耗时: 0.573682s
[INFO] [1763179838.132773250] [slider_action_controller]: ARM_SEND point 12/29 (async) 耗时: 0.191262s
[INFO] [1763179838.763057994] [slider_action_controller]: ARM_SEND point 13/29 (async) 耗时: 0.627866s
[INFO] [1763179839.430171307] [slider_action_controller]: ARM_SEND point 14/29 (async) 耗时: 0.664917s
[INFO] [1763179839.590654460] [slider_action_controller]: ARM_SEND point 15/29 (async) 耗时: 0.157901s
[INFO] [1763179840.167202885] [slider_action_controller]: ARM_SEND point 16/29 (async) 耗时: 0.574021s
[INFO] [1763179840.793452940] [slider_action_controller]: ARM_SEND point 17/29 (async) 耗时: 0.623933s
[INFO] [1763179841.403460742] [slider_action_controller]: ARM_SEND point 18/29 (async) 耗时: 0.607441s
[INFO] [1763179842.029407941] [slider_action_controller]: ARM_SEND point 19/29 (async) 耗时: 0.623492s
[INFO] [1763179842.186848767] [slider_action_controller]: ARM_SEND point 20/29 (async) 耗时: 0.154661s
[INFO] [1763179842.764760798] [slider_action_controller]: ARM_SEND point 21/29 (async) 耗时: 0.575205s
[INFO] [1763179843.413711231] [slider_action_controller]: ARM_SEND point 22/29 (async) 耗时: 0.644651s
[INFO] [1763179843.577063147] [slider_action_controller]: ARM_SEND point 23/29 (async) 耗时: 0.160856s
[INFO] [1763179844.168091998] [slider_action_controller]: ARM_SEND point 24/29 (async) 耗时: 0.588794s
[INFO] [1763179844.353874525] [slider_action_controller]: ARM_SEND point 25/29 (async) 耗时: 0.183501s
[INFO] [1763179844.514576463] [slider_action_controller]: ARM_SEND point 26/29 (async) 耗时: 0.158261s
[INFO] [1763179844.637230767] [slider_action_controller]: ARM_SEND point 27/29 (async) 耗时: 0.120155s
[INFO] [1763179844.678108123] [slider_action_controller]: ARM_SYNC point 28/29: 成功到位 (尝试次数: 1)
[INFO] [1763179845.208181532] [slider_action_controller]: ARM_SYNC point 28/29: 目标=[63.8, 42.2, 45.3, -84.5, -48.9, -101.0], 实际=[63.6, 42.0, 45.3, -85.0, -48.9, -100.3], 差值=[-0.17, -0.17, 0.0, -0.44, -0.01, 0.69], 最大差=0.69°
[INFO] [1763179845.314130565] [slider_action_controller]: ARM_SEND point 28/29 (sync, result=0) 耗时: 0.671272s
[INFO] [1763179845.316340973] [slider_action_controller]: 手臂轨迹发送完成，总耗时: 12.762832s
[INFO] [1763179845.409188189] [slider_action_controller]: 手臂最终状态对比:
  当前关节位置(弧度): [1.1105530030439918, 0.7332128187628179, 0.7899360194526335, -1.4833553312699805, -0.8527678725244294, -1.7502161738999138]
  目标关节位置(弧度): [1.1134785501484408, 0.7362580203027449, 0.7898498812509889, -1.4756013330229825, -0.8526739269828707, -1.7623341958261218]
  差值(当前-目标)(弧度): [-0.0029255471044489223, -0.003045201539927045, 8.613820164460328e-05, -0.007753998246998073, -9.394554155872648e-05, 0.012118021926208034]
[INFO] [1763179845.412570619] [slider_action_controller]: 手臂 FollowJointTrajectory 标记为成功
```

    - 卡顿明显且平均
  - -p fresh_mode:=0 -p arm_point_sleep_interval:=0.1 -p arm_async_send:=False

```
[INFO] [1763180963.662825929] [slider_action_controller]: 开始执行手臂轨迹
[INFO] [1763180963.782810142] [slider_action_controller]: ARM_SEND point 0/28 (async) 耗时: 0.117550s
[INFO] [1763180964.333359728] [slider_action_controller]: ARM_SEND point 1/28 (async) 耗时: 0.547995s
[INFO] [1763180965.003114910] [slider_action_controller]: ARM_SEND point 2/28 (async) 耗时: 0.667556s
[INFO] [1763180965.152594147] [slider_action_controller]: ARM_SEND point 3/28 (async) 耗时: 0.146855s
[INFO] [1763180965.703258639] [slider_action_controller]: ARM_SEND point 4/28 (async) 耗时: 0.548126s
[INFO] [1763180966.423884147] [slider_action_controller]: ARM_SEND point 5/28 (async) 耗时: 0.718314s
[INFO] [1763180967.089308647] [slider_action_controller]: ARM_SEND point 6/28 (async) 耗时: 0.662827s
[INFO] [1763180970.187180905] [slider_action_controller]: ARM_SEND point 7/28 (async) 耗时: 3.095347s
[INFO] [1763180970.789089269] [slider_action_controller]: ARM_SEND point 8/28 (async) 耗时: 0.600277s
[INFO] [1763180971.398652629] [slider_action_controller]: ARM_SEND point 9/28 (async) 耗时: 0.607378s
[INFO] [1763180972.190493582] [slider_action_controller]: ARM_SEND point 10/28 (async) 耗时: 0.789326s
[INFO] [1763180972.811038926] [slider_action_controller]: ARM_SEND point 11/28 (async) 耗时: 0.616128s
[INFO] [1763180973.417432252] [slider_action_controller]: ARM_SEND point 12/28 (async) 耗时: 0.604121s
[INFO] [1763180974.043156097] [slider_action_controller]: ARM_SEND point 13/28 (async) 耗时: 0.623179s
[INFO] [1763180974.646888090] [slider_action_controller]: ARM_SEND point 14/28 (async) 耗时: 0.601181s
[INFO] [1763180975.252383571] [slider_action_controller]: ARM_SEND point 15/28 (async) 耗时: 0.602555s
[INFO] [1763180975.856966943] [slider_action_controller]: ARM_SEND point 16/28 (async) 耗时: 0.602282s
[INFO] [1763180976.481663641] [slider_action_controller]: ARM_SEND point 17/28 (async) 耗时: 0.621858s
[INFO] [1763180977.086154027] [slider_action_controller]: ARM_SEND point 18/28 (async) 耗时: 0.602219s
[INFO] [1763180977.766410335] [slider_action_controller]: ARM_SEND point 19/28 (async) 耗时: 0.677421s
[INFO] [1763180978.454139023] [slider_action_controller]: ARM_SEND point 20/28 (async) 耗时: 0.685202s
[INFO] [1763180979.034432372] [slider_action_controller]: ARM_SEND point 21/28 (async) 耗时: 0.577421s
[INFO] [1763180979.640714495] [slider_action_controller]: ARM_SEND point 22/28 (async) 耗时: 0.603951s
[INFO] [1763180980.293002083] [slider_action_controller]: ARM_SEND point 23/28 (async) 耗时: 0.649898s
[INFO] [1763180980.987473318] [slider_action_controller]: ARM_SEND point 24/28 (async) 耗时: 0.691640s
[INFO] [1763180981.592517730] [slider_action_controller]: ARM_SEND point 25/28 (async) 耗时: 0.599302s
[INFO] [1763180981.754645357] [slider_action_controller]: ARM_SEND point 26/28 (async) 耗时: 0.159402s
[INFO] [1763180982.247307573] [slider_action_controller]: ARM_SYNC point 27/28: 成功到位 (尝试次数: 1)
[INFO] [1763180982.354528711] [slider_action_controller]: ARM_SYNC point 27/28: 目标=[63.5, 42.0, 44.9, -84.6, -48.4, -97.6], 实际=[64.0, 42.3, 44.6, -84.8, -48.5, -96.9], 差值=[0.43, 0.27, -0.36, -0.18, -0.09, 0.7], 最大差=0.70°
[INFO] [1763180982.458389415] [slider_action_controller]: ARM_SEND point 27/28 (sync, result=0) 耗时: 0.700128s
[INFO] [1763180982.464894977] [slider_action_controller]: 手臂轨迹发送完成，总耗时: 18.795509s
```

```
[INFO] [1763181049.856000151] [slider_action_controller]: 收到新轨迹目标
[INFO] [1763181049.863654568] [slider_action_controller]: 开始执行手臂轨迹
[INFO] [1763181049.982518615] [slider_action_controller]: ARM_SEND point 0/28 (async) 耗时: 0.116521s
[INFO] [1763181050.093628318] [slider_action_controller]: ARM_SEND point 1/28 (async) 耗时: 0.108607s
[INFO] [1763181050.646298412] [slider_action_controller]: ARM_SEND point 2/28 (async) 耗时: 0.549876s
[INFO] [1763181050.848800536] [slider_action_controller]: ARM_SEND point 3/28 (async) 耗时: 0.200093s
[INFO] [1763181051.447173391] [slider_action_controller]: ARM_SEND point 4/28 (async) 耗时: 0.595772s
[INFO] [1763181051.781485590] [slider_action_controller]: ARM_SEND point 5/28 (async) 耗时: 0.331452s
[INFO] [1763181052.391894155] [slider_action_controller]: ARM_SEND point 6/28 (async) 耗时: 0.608094s
[INFO] [1763181053.015601215] [slider_action_controller]: ARM_SEND point 7/28 (async) 耗时: 0.621129s
[INFO] [1763181053.626169732] [slider_action_controller]: ARM_SEND point 8/28 (async) 耗时: 0.607978s
[INFO] [1763181054.239254304] [slider_action_controller]: ARM_SEND point 9/28 (async) 耗时: 0.610856s
[INFO] [1763181055.024202678] [slider_action_controller]: ARM_SEND point 10/28 (async) 耗时: 0.782082s
[INFO] [1763181057.633575894] [slider_action_controller]: ARM_SEND point 11/28 (async) 耗时: 2.607139s
[INFO] [1763181059.248615370] [slider_action_controller]: ARM_SEND point 12/28 (async) 耗时: 1.612369s
[INFO] [1763181059.980101681] [slider_action_controller]: ARM_SEND point 13/28 (async) 耗时: 0.728978s
[INFO] [1763181060.630976352] [slider_action_controller]: ARM_SEND point 14/28 (async) 耗时: 0.648600s
[INFO] [1763181061.283030284] [slider_action_controller]: ARM_SEND point 15/28 (async) 耗时: 0.649549s
[INFO] [1763181062.883970392] [slider_action_controller]: ARM_SEND point 16/28 (async) 耗时: 1.598660s
[INFO] [1763181063.639600074] [slider_action_controller]: ARM_SEND point 17/28 (async) 耗时: 0.754110s
[INFO] [1763181064.289754739] [slider_action_controller]: ARM_SEND point 18/28 (async) 耗时: 0.648099s
[INFO] [1763181064.613641604] [slider_action_controller]: ARM_SEND point 19/28 (async) 耗时: 0.321104s
[INFO] [1763181065.223964722] [slider_action_controller]: ARM_SEND point 20/28 (async) 耗时: 0.608030s
[INFO] [1763181065.875515489] [slider_action_controller]: ARM_SEND point 21/28 (async) 耗时: 0.649294s
[INFO] [1763181066.577597033] [slider_action_controller]: ARM_SEND point 22/28 (async) 耗时: 0.699271s
[INFO] [1763181067.188277778] [slider_action_controller]: ARM_SEND point 23/28 (async) 耗时: 0.607912s
[INFO] [1763181067.798196413] [slider_action_controller]: ARM_SEND point 24/28 (async) 耗时: 0.607529s
[INFO] [1763181068.413921533] [slider_action_controller]: ARM_SEND point 25/28 (async) 耗时: 0.611619s
[INFO] [1763181068.593564191] [slider_action_controller]: ARM_SEND point 26/28 (async) 耗时: 0.176992s
[INFO] [1763181069.116599433] [slider_action_controller]: ARM_SYNC point 27/28: 成功到位 (尝试次数: 1)
[INFO] [1763181069.732576142] [slider_action_controller]: ARM_SYNC point 27/28: 目标=[64.0, 42.4, 44.6, -84.9, -48.5, -96.9], 实际=[64.3, 42.5, 44.5, -85.2, -48.3, -96.2], 差值=[0.35, 0.17, -0.09, -0.26, 0.18, 0.69], 最大差=0.69°
[INFO] [1763181069.834815728] [slider_action_controller]: ARM_SEND point 27/28 (sync, result=0) 耗时: 1.239031s
[INFO] [1763181069.837049539] [slider_action_controller]: 手臂轨迹发送完成，总耗时: 19.971085s
```

    - 卡顿很平均  
  

  - -p fresh_mode:=1 -p arm_point_sleep_interval:=0.05 -p arm_async_send:=True

```
[INFO] [1763179928.902355381] [slider_action_controller]: 开始执行手臂轨迹
[INFO] [1763179928.959572755] [slider_action_controller]: ARM_SEND point 0/29 (async) 耗时: 0.055039s
[INFO] [1763179929.012783012] [slider_action_controller]: ARM_SEND point 1/29 (async) 耗时: 0.050866s
[INFO] [1763179929.076789773] [slider_action_controller]: ARM_SEND point 2/29 (async) 耗时: 0.061692s
[INFO] [1763179929.129782205] [slider_action_controller]: ARM_SEND point 3/29 (async) 耗时: 0.050723s
[INFO] [1763179929.182694044] [slider_action_controller]: ARM_SEND point 4/29 (async) 耗时: 0.050669s
[INFO] [1763179929.235511681] [slider_action_controller]: ARM_SEND point 5/29 (async) 耗时: 0.050631s
[INFO] [1763179929.288665582] [slider_action_controller]: ARM_SEND point 6/29 (async) 耗时: 0.050802s
[INFO] [1763179929.341820366] [slider_action_controller]: ARM_SEND point 7/29 (async) 耗时: 0.050887s
[INFO] [1763179929.398670446] [slider_action_controller]: ARM_SEND point 8/29 (async) 耗时: 0.053175s
[INFO] [1763179929.507563842] [slider_action_controller]: ARM_SEND point 9/29 (async) 耗时: 0.106717s
[INFO] [1763179929.607098189] [slider_action_controller]: ARM_SEND point 10/29 (async) 耗时: 0.097265s
[INFO] [1763179929.710423811] [slider_action_controller]: ARM_SEND point 11/29 (async) 耗时: 0.097849s
[INFO] [1763179929.763802965] [slider_action_controller]: ARM_SEND point 12/29 (async) 耗时: 0.050928s
[INFO] [1763179929.816842896] [slider_action_controller]: ARM_SEND point 13/29 (async) 耗时: 0.050622s
[INFO] [1763179929.870033755] [slider_action_controller]: ARM_SEND point 14/29 (async) 耗时: 0.050903s
[INFO] [1763179929.923027523] [slider_action_controller]: ARM_SEND point 15/29 (async) 耗时: 0.050632s
[INFO] [1763179930.006828716] [slider_action_controller]: ARM_SEND point 16/29 (async) 耗时: 0.081519s
[INFO] [1763179931.557220203] [slider_action_controller]: ARM_SEND point 17/29 (async) 耗时: 1.547746s
[INFO] [1763179932.661231070] [slider_action_controller]: ARM_SEND point 18/29 (async) 耗时: 1.101269s
[INFO] [1763179932.822079867] [slider_action_controller]: ARM_SEND point 19/29 (async) 耗时: 0.158635s
[INFO] [1763179932.914320736] [slider_action_controller]: ARM_SEND point 20/29 (async) 耗时: 0.089900s
[INFO] [1763179932.995654916] [slider_action_controller]: ARM_SEND point 21/29 (async) 耗时: 0.076396s
[INFO] [1763179933.645924894] [slider_action_controller]: ARM_SEND point 22/29 (async) 耗时: 0.647833s
[INFO] [1763179937.756593703] [slider_action_controller]: ARM_SEND point 23/29 (async) 耗时: 4.108355s
[INFO] [1763179938.339393391] [slider_action_controller]: ARM_SEND point 24/29 (async) 耗时: 0.580520s
[INFO] [1763179940.980462530] [slider_action_controller]: ARM_SEND point 25/29 (async) 耗时: 2.639844s
[INFO] [1763179941.579528773] [slider_action_controller]: ARM_SEND point 26/29 (async) 耗时: 0.596600s
[INFO] [1763179942.179326830] [slider_action_controller]: ARM_SEND point 27/29 (async) 耗时: 0.597449s
[INFO] [1763179942.747927293] [slider_action_controller]: ARM_SYNC point 28/29: 成功到位 (尝试次数: 1)
[INFO] [1763179942.908370848] [slider_action_controller]: ARM_SYNC point 28/29: 目标=[63.6, 42.0, 45.3, -84.9, -48.9, -100.3], 实际=[63.8, 41.8, 44.9, -85.2, -48.4, -99.6], 差值=[0.17, -0.18, -0.35, -0.26, 0.44, 0.7], 最大差=0.70°
[INFO] [1763179942.961083295] [slider_action_controller]: ARM_SEND point 28/29 (sync, result=0) 耗时: 0.779376s
[INFO] [1763179942.962076157] [slider_action_controller]: 手臂轨迹发送完成，总耗时: 14.058711s
```

```
[INFO] [1763179974.296617403] [slider_action_controller]: 开始执行手臂轨迹
[INFO] [1763179974.359925889] [slider_action_controller]: ARM_SEND point 0/29 (async) 耗时: 0.060798s
[INFO] [1763179974.425057290] [slider_action_controller]: ARM_SEND point 1/29 (async) 耗时: 0.062801s
[INFO] [1763179974.478120128] [slider_action_controller]: ARM_SEND point 2/29 (async) 耗时: 0.050771s
[INFO] [1763179974.531299163] [slider_action_controller]: ARM_SEND point 3/29 (async) 耗时: 0.050897s
[INFO] [1763179974.584744823] [slider_action_controller]: ARM_SEND point 4/29 (async) 耗时: 0.050997s
[INFO] [1763179974.638153977] [slider_action_controller]: ARM_SEND point 5/29 (async) 耗时: 0.050996s
[INFO] [1763179974.691027116] [slider_action_controller]: ARM_SEND point 6/29 (async) 耗时: 0.050572s
[INFO] [1763179974.747606579] [slider_action_controller]: ARM_SEND point 7/29 (async) 耗时: 0.051077s
[INFO] [1763179976.310236915] [slider_action_controller]: ARM_SEND point 8/29 (async) 耗时: 1.559989s
[INFO] [1763179978.904540772] [slider_action_controller]: ARM_SEND point 9/29 (async) 耗时: 2.591537s
[INFO] [1763179979.524887647] [slider_action_controller]: ARM_SEND point 10/29 (async) 耗时: 0.617959s
[INFO] [1763179980.134653902] [slider_action_controller]: ARM_SEND point 11/29 (async) 耗时: 0.607444s
[INFO] [1763179980.319188380] [slider_action_controller]: ARM_SEND point 12/29 (async) 耗时: 0.181933s
[INFO] [1763179980.930776607] [slider_action_controller]: ARM_SEND point 13/29 (async) 耗时: 0.609293s
[INFO] [1763179981.070654489] [slider_action_controller]: ARM_SEND point 14/29 (async) 耗时: 0.137603s
[INFO] [1763179981.199880924] [slider_action_controller]: ARM_SEND point 15/29 (async) 耗时: 0.126897s
[INFO] [1763179981.767581458] [slider_action_controller]: ARM_SEND point 16/29 (async) 耗时: 0.565409s
[INFO] [1763179982.402574636] [slider_action_controller]: ARM_SEND point 17/29 (async) 耗时: 0.632636s
[INFO] [1763179982.602013530] [slider_action_controller]: ARM_SEND point 18/29 (async) 耗时: 0.197134s
[INFO] [1763179983.213594924] [slider_action_controller]: ARM_SEND point 19/29 (async) 耗时: 0.609249s
[INFO] [1763179983.355651920] [slider_action_controller]: ARM_SEND point 20/29 (async) 耗时: 0.139752s
[INFO] [1763179983.940695800] [slider_action_controller]: ARM_SEND point 21/29 (async) 耗时: 0.582667s
[INFO] [1763179984.622815106] [slider_action_controller]: ARM_SEND point 22/29 (async) 耗时: 0.679823s
[INFO] [1763179985.191957696] [slider_action_controller]: ARM_SEND point 23/29 (async) 耗时: 0.566753s
[INFO] [1763179985.786672377] [slider_action_controller]: ARM_SEND point 24/29 (async) 耗时: 0.592384s
[INFO] [1763179985.885574435] [slider_action_controller]: ARM_SEND point 25/29 (async) 耗时: 0.096533s
[INFO] [1763179985.942150159] [slider_action_controller]: ARM_SEND point 26/29 (async) 耗时: 0.054262s
[INFO] [1763179985.996817014] [slider_action_controller]: ARM_SEND point 27/29 (async) 耗时: 0.052405s
[INFO] [1763179986.036935186] [slider_action_controller]: ARM_SYNC point 28/29: 成功到位 (尝试次数: 1)
[INFO] [1763179986.568616877] [slider_action_controller]: ARM_SYNC point 28/29: 目标=[63.8, 41.8, 44.9, -85.2, -48.4, -99.6], 实际=[63.8, 42.0, 44.9, -85.5, -48.2, -98.9], 差值=[0.0, 0.18, 0.0, -0.35, 0.26, 0.71], 最大差=0.71°
[INFO] [1763179986.620931549] [slider_action_controller]: ARM_SEND point 28/29 (sync, result=0) 耗时: 0.621844s
[INFO] [1763179986.622939208] [slider_action_controller]: 手臂轨迹发送完成，总耗时: 12.324283s
```

    - 存在小段的连续段, 其他段仍然卡顿
  - -p fresh_mode:=0 -p arm_point_sleep_interval:=0.05 -p arm_async_send:=True

```
[INFO] [1763180193.777095753] [slider_action_controller]: 收到新轨迹目标
[INFO] [1763180193.780675543] [slider_action_controller]: 开始执行手臂轨迹
[INFO] [1763180193.858470940] [slider_action_controller]: ARM_SEND point 0/29 (async) 耗时: 0.075658s
[INFO] [1763180193.911576757] [slider_action_controller]: ARM_SEND point 1/29 (async) 耗时: 0.050850s
[INFO] [1763180193.964925274] [slider_action_controller]: ARM_SEND point 2/29 (async) 耗时: 0.050955s
[INFO] [1763180194.042763837] [slider_action_controller]: ARM_SEND point 3/29 (async) 耗时: 0.075599s
[INFO] [1763180194.600678410] [slider_action_controller]: ARM_SEND point 4/29 (async) 耗时: 0.555612s
[INFO] [1763180195.226722413] [slider_action_controller]: ARM_SEND point 5/29 (async) 耗时: 0.623819s
[INFO] [1763180195.849067617] [slider_action_controller]: ARM_SEND point 6/29 (async) 耗时: 0.619933s
[INFO] [1763180196.442139856] [slider_action_controller]: ARM_SEND point 7/29 (async) 耗时: 0.590586s
[INFO] [1763180197.048702346] [slider_action_controller]: ARM_SEND point 8/29 (async) 耗时: 0.604240s
[INFO] [1763180197.641940146] [slider_action_controller]: ARM_SEND point 9/29 (async) 耗时: 0.590962s
[INFO] [1763180198.335057688] [slider_action_controller]: ARM_SEND point 10/29 (async) 耗时: 0.690875s
[INFO] [1763180198.942689754] [slider_action_controller]: ARM_SEND point 11/29 (async) 耗时: 0.605348s
[INFO] [1763180199.602450885] [slider_action_controller]: ARM_SEND point 12/29 (async) 耗时: 0.657454s
[INFO] [1763180200.212046741] [slider_action_controller]: ARM_SEND point 13/29 (async) 耗时: 0.607158s
[INFO] [1763180200.826432151] [slider_action_controller]: ARM_SEND point 14/29 (async) 耗时: 0.612124s
[INFO] [1763180201.445608542] [slider_action_controller]: ARM_SEND point 15/29 (async) 耗时: 0.616830s
[INFO] [1763180202.047052592] [slider_action_controller]: ARM_SEND point 16/29 (async) 耗时: 0.598918s
[INFO] [1763180202.688394575] [slider_action_controller]: ARM_SEND point 17/29 (async) 耗时: 0.638940s
[INFO] [1763180203.283084400] [slider_action_controller]: ARM_SEND point 18/29 (async) 耗时: 0.592504s
[INFO] [1763180203.897089786] [slider_action_controller]: ARM_SEND point 19/29 (async) 耗时: 0.611727s
[INFO] [1763180204.536085324] [slider_action_controller]: ARM_SEND point 20/29 (async) 耗时: 0.636643s
[INFO] [1763180205.139375424] [slider_action_controller]: ARM_SEND point 21/29 (async) 耗时: 0.601069s
[INFO] [1763180205.801276559] [slider_action_controller]: ARM_SEND point 22/29 (async) 耗时: 0.659561s
[INFO] [1763180206.583613518] [slider_action_controller]: ARM_SEND point 23/29 (async) 耗时: 0.780099s
[INFO] [1763180207.223302971] [slider_action_controller]: ARM_SEND point 24/29 (async) 耗时: 0.637365s
[INFO] [1763180207.916771302] [slider_action_controller]: ARM_SEND point 25/29 (async) 耗时: 0.690196s
[INFO] [1763180208.508031994] [slider_action_controller]: ARM_SEND point 26/29 (async) 耗时: 0.588945s
[INFO] [1763180209.618325076] [slider_action_controller]: ARM_SEND point 27/29 (async) 耗时: 1.107695s
[INFO] [1763180210.250634489] [slider_action_controller]: ARM_SYNC point 28/29: 成功到位 (尝试次数: 1)
[INFO] [1763180210.345917423] [slider_action_controller]: ARM_SYNC point 28/29: 目标=[63.8, 42.0, 44.9, -85.5, -48.2, -98.9], 实际=[63.2, 41.9, 44.8, -84.8, -48.2, -98.3], 差值=[-0.61, -0.08, -0.09, 0.7, -0.09, 0.53], 最大差=0.70°
[INFO] [1763180210.400370106] [slider_action_controller]: ARM_SEND point 28/29 (sync, result=0) 耗时: 0.777618s
[INFO] [1763180210.402616671] [slider_action_controller]: 手臂轨迹发送完成，总耗时: 16.619764s
```

```
[INFO] [1763180276.683453116] [slider_action_controller]: 开始执行手臂轨迹
[INFO] [1763180276.745282670] [slider_action_controller]: ARM_SEND point 0/29 (async) 耗时: 0.059422s
[INFO] [1763180277.276012305] [slider_action_controller]: ARM_SEND point 1/29 (async) 耗时: 0.528334s
[INFO] [1763180277.874894378] [slider_action_controller]: ARM_SEND point 2/29 (async) 耗时: 0.596280s
[INFO] [1763180278.477490234] [slider_action_controller]: ARM_SEND point 3/29 (async) 耗时: 0.600227s
[INFO] [1763180279.130621303] [slider_action_controller]: ARM_SEND point 4/29 (async) 耗时: 0.649471s
[INFO] [1763180279.731933083] [slider_action_controller]: ARM_SEND point 5/29 (async) 耗时: 0.596649s
[INFO] [1763180280.327774318] [slider_action_controller]: ARM_SEND point 6/29 (async) 耗时: 0.593534s
[INFO] [1763180280.927763470] [slider_action_controller]: ARM_SEND point 7/29 (async) 耗时: 0.597658s
[INFO] [1763180281.570121787] [slider_action_controller]: ARM_SEND point 8/29 (async) 耗时: 0.639646s
[INFO] [1763180282.165780908] [slider_action_controller]: ARM_SEND point 9/29 (async) 耗时: 0.593287s
[INFO] [1763180282.771215193] [slider_action_controller]: ARM_SEND point 10/29 (async) 耗时: 0.603137s
[INFO] [1763180283.329550883] [slider_action_controller]: ARM_SEND point 11/29 (async) 耗时: 0.555904s
[INFO] [1763180284.107302968] [slider_action_controller]: ARM_SEND point 12/29 (async) 耗时: 0.775495s
[INFO] [1763180284.705881293] [slider_action_controller]: ARM_SEND point 13/29 (async) 耗时: 0.596237s
[INFO] [1763180286.303207249] [slider_action_controller]: ARM_SEND point 14/29 (async) 耗时: 1.594864s
[INFO] [1763180289.365637217] [slider_action_controller]: ARM_SEND point 15/29 (async) 耗时: 3.059609s
[INFO] [1763180289.612099110] [slider_action_controller]: ARM_SEND point 16/29 (async) 耗时: 0.244137s
[INFO] [1763180290.198577627] [slider_action_controller]: ARM_SEND point 17/29 (async) 耗时: 0.584191s
[INFO] [1763180290.906965020] [slider_action_controller]: ARM_SEND point 18/29 (async) 耗时: 0.705722s
[INFO] [1763180299.972569626] [slider_action_controller]: ARM_SEND point 19/29 (async) 耗时: 9.062982s
[INFO] [1763180300.575088774] [slider_action_controller]: ARM_SEND point 20/29 (async) 耗时: 0.600206s
[INFO] [1763180301.299631455] [slider_action_controller]: ARM_SEND point 21/29 (async) 耗时: 0.722266s
[INFO] [1763180301.920614113] [slider_action_controller]: ARM_SEND point 22/29 (async) 耗时: 0.618285s
[INFO] [1763180302.531849041] [slider_action_controller]: ARM_SEND point 23/29 (async) 耗时: 0.606264s
[INFO] [1763180305.638257248] [slider_action_controller]: ARM_SEND point 24/29 (async) 耗时: 3.103818s
[INFO] [1763180306.282194021] [slider_action_controller]: ARM_SEND point 25/29 (async) 耗时: 0.638481s
[INFO] [1763180306.890287496] [slider_action_controller]: ARM_SEND point 26/29 (async) 耗时: 0.605498s
[INFO] [1763180309.490215323] [slider_action_controller]: ARM_SEND point 27/29 (async) 耗时: 2.597658s
[INFO] [1763180310.133694177] [slider_action_controller]: ARM_SYNC point 28/29: 成功到位 (尝试次数: 1)
[INFO] [1763180310.249151986] [slider_action_controller]: ARM_SYNC point 28/29: 目标=[63.3, 42.2, 45.3, -85.0, -48.4, -98.3], 实际=[62.9, 41.8, 44.9, -84.6, -48.0, -97.6], 差值=[-0.36, -0.35, -0.35, 0.36, 0.44, 0.7], 最大差=0.70°
[INFO] [1763180310.301424658] [slider_action_controller]: ARM_SEND point 28/29 (sync, result=0) 耗时: 0.808990s
[INFO] [1763180310.305146786] [slider_action_controller]: 手臂轨迹发送完成，总耗时: 33.617893s
```

    - 卡顿很平均
  - -p fresh_mode:=1 -p arm_point_sleep_interval:=0.01 -p arm_async_send:=True

```
[INFO] [1763181149.533114068] [slider_action_controller]: 收到新轨迹目标
[INFO] [1763181149.542298971] [slider_action_controller]: 开始执行手臂轨迹
[INFO] [1763181149.564530676] [slider_action_controller]: ARM_SEND point 0/28 (async) 耗时: 0.019890s
[INFO] [1763181149.577218167] [slider_action_controller]: ARM_SEND point 1/28 (async) 耗时: 0.010566s
[INFO] [1763181149.594148791] [slider_action_controller]: ARM_SEND point 2/28 (async) 耗时: 0.012093s
[INFO] [1763181149.606853395] [slider_action_controller]: ARM_SEND point 3/28 (async) 耗时: 0.010641s
[INFO] [1763181149.619810378] [slider_action_controller]: ARM_SEND point 4/28 (async) 耗时: 0.010739s
[INFO] [1763181149.632601230] [slider_action_controller]: ARM_SEND point 5/28 (async) 耗时: 0.010630s
[INFO] [1763181149.645263355] [slider_action_controller]: ARM_SEND point 6/28 (async) 耗时: 0.010566s
[INFO] [1763181149.658978009] [slider_action_controller]: ARM_SEND point 7/28 (async) 耗时: 0.010556s
[INFO] [1763181149.672199835] [slider_action_controller]: ARM_SEND point 8/28 (async) 耗时: 0.010916s
[INFO] [1763181149.684962335] [slider_action_controller]: ARM_SEND point 9/28 (async) 耗时: 0.010650s
[INFO] [1763181149.697631685] [slider_action_controller]: ARM_SEND point 10/28 (async) 耗时: 0.010546s
[INFO] [1763181149.710533844] [slider_action_controller]: ARM_SEND point 11/28 (async) 耗时: 0.010659s
[INFO] [1763181149.723620051] [slider_action_controller]: ARM_SEND point 12/28 (async) 耗时: 0.010857s
[INFO] [1763181149.736574735] [slider_action_controller]: ARM_SEND point 13/28 (async) 耗时: 0.010780s
[INFO] [1763181149.749252088] [slider_action_controller]: ARM_SEND point 14/28 (async) 耗时: 0.010580s
[INFO] [1763181149.762193859] [slider_action_controller]: ARM_SEND point 15/28 (async) 耗时: 0.010763s
[INFO] [1763181149.775117573] [slider_action_controller]: ARM_SEND point 16/28 (async) 耗时: 0.010649s
[INFO] [1763181149.792536911] [slider_action_controller]: ARM_SEND point 17/28 (async) 耗时: 0.010705s
[INFO] [1763181149.806139199] [slider_action_controller]: ARM_SEND point 18/28 (async) 耗时: 0.011363s
[INFO] [1763181149.819055329] [slider_action_controller]: ARM_SEND point 19/28 (async) 耗时: 0.010653s
[INFO] [1763181149.832241255] [slider_action_controller]: ARM_SEND point 20/28 (async) 耗时: 0.010906s
[INFO] [1763181149.844968204] [slider_action_controller]: ARM_SEND point 21/28 (async) 耗时: 0.010626s
[INFO] [1763181149.858068556] [slider_action_controller]: ARM_SEND point 22/28 (async) 耗时: 0.010935s
[INFO] [1763181149.871059555] [slider_action_controller]: ARM_SEND point 23/28 (async) 耗时: 0.010747s
[INFO] [1763181149.884568161] [slider_action_controller]: ARM_SEND point 24/28 (async) 耗时: 0.010924s
[INFO] [1763181149.897531550] [slider_action_controller]: ARM_SEND point 25/28 (async) 耗时: 0.010661s
[INFO] [1763181151.412788034] [slider_action_controller]: ARM_SEND point 26/28 (async) 耗时: 1.512736s
[WARN] [1763181154.006398002] [slider_action_controller]: ARM_SYNC point 27/28: 第 1 次重试
[WARN] [1763181155.226140252] [slider_action_controller]: ARM_SYNC point 27/28: 第 2 次重试
[INFO] [1763181155.841471406] [slider_action_controller]: ARM_SYNC point 27/28: 成功到位 (尝试次数: 3)
[INFO] [1763181155.943861747] [slider_action_controller]: ARM_SYNC point 27/28: 目标=[64.3, 42.5, 44.5, -85.2, -48.3, -96.2], 实际=[64.7, 43.4, 44.9, -84.7, -48.0, -95.6], 差值=[0.35, 0.88, 0.44, 0.44, 0.35, 0.62], 最大差=0.88°
[INFO] [1763181155.960534617] [slider_action_controller]: ARM_SEND point 27/28 (sync, result=0) 耗时: 4.541236s
[INFO] [1763181155.962561884] [slider_action_controller]: 手臂轨迹发送完成，总耗时: 6.418167s
```

```
[INFO] [1763181232.095163270] [slider_action_controller]: 开始执行手臂轨迹
[INFO] [1763181232.116790554] [slider_action_controller]: ARM_SEND point 0/28 (async) 耗时: 0.019294s
[INFO] [1763181233.627317623] [slider_action_controller]: ARM_SEND point 1/28 (async) 耗时: 1.508275s
[INFO] [1763181244.201892500] [slider_action_controller]: ARM_SEND point 2/28 (async) 耗时: 10.571879s
[INFO] [1763181244.296986260] [slider_action_controller]: ARM_SEND point 3/28 (async) 耗时: 0.092803s
[INFO] [1763181244.977756598] [slider_action_controller]: ARM_SEND point 4/28 (async) 耗时: 0.678644s
[INFO] [1763181245.553900394] [slider_action_controller]: ARM_SEND point 5/28 (async) 耗时: 0.573617s
[INFO] [1763181245.687092421] [slider_action_controller]: ARM_SEND point 6/28 (async) 耗时: 0.130967s
[INFO] [1763181247.822284197] [slider_action_controller]: ARM_SEND point 7/28 (async) 耗时: 2.132789s
[INFO] [1763181247.904186024] [slider_action_controller]: ARM_SEND point 8/28 (async) 耗时: 0.079551s
[INFO] [1763181248.042668987] [slider_action_controller]: ARM_SEND point 9/28 (async) 耗时: 0.136319s
[INFO] [1763181248.630630656] [slider_action_controller]: ARM_SEND point 10/28 (async) 耗时: 0.586209s
[INFO] [1763181249.230436734] [slider_action_controller]: ARM_SEND point 11/28 (async) 耗时: 0.597549s
[INFO] [1763181249.833496652] [slider_action_controller]: ARM_SEND point 12/28 (async) 耗时: 0.600730s
[INFO] [1763181250.398543058] [slider_action_controller]: ARM_SEND point 13/28 (async) 耗时: 0.563472s
[INFO] [1763181250.991196572] [slider_action_controller]: ARM_SEND point 14/28 (async) 耗时: 0.590555s
[INFO] [1763181251.095958773] [slider_action_controller]: ARM_SEND point 15/28 (async) 耗时: 0.102576s
[INFO] [1763181255.161944831] [slider_action_controller]: ARM_SEND point 16/28 (async) 耗时: 4.063369s
[INFO] [1763181261.279565357] [slider_action_controller]: ARM_SEND point 17/28 (async) 耗时: 6.114981s
[INFO] [1763181261.396454115] [slider_action_controller]: ARM_SEND point 18/28 (async) 耗时: 0.114547s
[INFO] [1763181261.494306206] [slider_action_controller]: ARM_SEND point 19/28 (async) 耗时: 0.095633s
[INFO] [1763181261.594292326] [slider_action_controller]: ARM_SEND point 20/28 (async) 耗时: 0.097839s
[INFO] [1763181262.154008252] [slider_action_controller]: ARM_SEND point 21/28 (async) 耗时: 0.557117s
[INFO] [1763181262.261666146] [slider_action_controller]: ARM_SEND point 22/28 (async) 耗时: 0.105373s
[INFO] [1763181262.360604848] [slider_action_controller]: ARM_SEND point 23/28 (async) 耗时: 0.096713s
[INFO] [1763181262.915649851] [slider_action_controller]: ARM_SEND point 24/28 (async) 耗时: 0.552619s
[INFO] [1763181265.514167078] [slider_action_controller]: ARM_SEND point 25/28 (async) 耗时: 2.596288s
[INFO] [1763181265.604242294] [slider_action_controller]: ARM_SEND point 26/28 (async) 耗时: 0.087818s
[WARN] [1763181269.671336581] [slider_action_controller]: ARM_SYNC point 27/28: 第 1 次重试
[WARN] [1763181275.809515796] [slider_action_controller]: ARM_SYNC point 27/28: 第 2 次重试
[INFO] [1763181276.938556661] [slider_action_controller]: ARM_SYNC point 27/28: 成功到位 (尝试次数: 3)
[INFO] [1763181277.056319208] [slider_action_controller]: ARM_SYNC point 27/28: 目标=[64.7, 43.4, 44.9, -84.8, -48.0, -95.6], 实际=[64.5, 43.2, 44.6, -84.5, -47.9, -94.9], 差值=[-0.17, -0.17, -0.35, 0.34, 0.08, 0.7], 最大差=0.70°
[INFO] [1763181277.072263933] [slider_action_controller]: ARM_SEND point 27/28 (sync, result=0) 耗时: 11.462195s
[INFO] [1763181277.075198459] [slider_action_controller]: 手臂轨迹发送完成，总耗时: 44.977062s
```

    - 两次效果差异很大, 一次很平滑, 另外一次卡顿很厉害
  - -p fresh_mode:=0 -p arm_point_sleep_interval:=0.01 -p arm_async_send:=True

```
[INFO] [1763181362.336581001] [slider_action_controller]: 收到新轨迹目标
[INFO] [1763181362.342000269] [slider_action_controller]: 开始执行手臂轨迹
[INFO] [1763181362.361730192] [slider_action_controller]: ARM_SEND point 0/28 (async) 耗时: 0.017499s
[INFO] [1763181362.375408986] [slider_action_controller]: ARM_SEND point 1/28 (async) 耗时: 0.011523s
[INFO] [1763181362.388751097] [slider_action_controller]: ARM_SEND point 2/28 (async) 耗时: 0.011033s
[INFO] [1763181362.402054817] [slider_action_controller]: ARM_SEND point 3/28 (async) 耗时: 0.011024s
[INFO] [1763181362.420714724] [slider_action_controller]: ARM_SEND point 4/28 (async) 耗时: 0.010766s
[INFO] [1763181362.433481854] [slider_action_controller]: ARM_SEND point 5/28 (async) 耗时: 0.010687s
[INFO] [1763181362.446427621] [slider_action_controller]: ARM_SEND point 6/28 (async) 耗时: 0.010754s
[INFO] [1763181362.459461126] [slider_action_controller]: ARM_SEND point 7/28 (async) 耗时: 0.010692s
[INFO] [1763181362.472458867] [slider_action_controller]: ARM_SEND point 8/28 (async) 耗时: 0.010653s
[INFO] [1763181362.485192715] [slider_action_controller]: ARM_SEND point 9/28 (async) 耗时: 0.010666s
[INFO] [1763181362.498003578] [slider_action_controller]: ARM_SEND point 10/28 (async) 耗时: 0.010630s
[INFO] [1763181362.510970366] [slider_action_controller]: ARM_SEND point 11/28 (async) 耗时: 0.010692s
[INFO] [1763181362.524202986] [slider_action_controller]: ARM_SEND point 12/28 (async) 耗时: 0.010919s
[INFO] [1763181362.536982566] [slider_action_controller]: ARM_SEND point 13/28 (async) 耗时: 0.010619s
[INFO] [1763181362.550157630] [slider_action_controller]: ARM_SEND point 14/28 (async) 耗时: 0.010906s
[INFO] [1763181362.563370446] [slider_action_controller]: ARM_SEND point 15/28 (async) 耗时: 0.010957s
[INFO] [1763181362.577613795] [slider_action_controller]: ARM_SEND point 16/28 (async) 耗时: 0.011901s
[INFO] [1763181362.590525110] [slider_action_controller]: ARM_SEND point 17/28 (async) 耗时: 0.010745s
[INFO] [1763181362.603681491] [slider_action_controller]: ARM_SEND point 18/28 (async) 耗时: 0.010936s
[INFO] [1763181362.618642723] [slider_action_controller]: ARM_SEND point 19/28 (async) 耗时: 0.011511s
[INFO] [1763181364.139692470] [slider_action_controller]: ARM_SEND point 20/28 (async) 耗时: 1.518444s
[INFO] [1763181368.168543671] [slider_action_controller]: ARM_SEND point 21/28 (async) 耗时: 4.022229s
[INFO] [1763181369.777924095] [slider_action_controller]: ARM_SEND point 22/28 (async) 耗时: 1.608465s
[INFO] [1763181370.507398952] [slider_action_controller]: ARM_SEND point 23/28 (async) 耗时: 0.727418s
[INFO] [1763181370.595232558] [slider_action_controller]: ARM_SEND point 24/28 (async) 耗时: 0.085641s
[INFO] [1763181371.125868601] [slider_action_controller]: ARM_SEND point 25/28 (async) 耗时: 0.528142s
[INFO] [1763181371.724415059] [slider_action_controller]: ARM_SEND point 26/28 (async) 耗时: 0.596105s
[INFO] [1763181372.337574127] [slider_action_controller]: ARM_SYNC point 27/28: 成功到位 (尝试次数: 1)
[INFO] [1763181372.946637976] [slider_action_controller]: ARM_SYNC point 27/28: 目标=[64.5, 43.2, 44.6, -84.5, -47.9, -94.9], 实际=[64.3, 43.4, 44.2, -84.8, -48.2, -94.3], 差值=[-0.18, 0.17, -0.36, -0.35, -0.35, 0.62], 最大差=0.62°
[INFO] [1763181372.958857818] [slider_action_controller]: ARM_SEND point 27/28 (sync, result=0) 耗时: 1.232215s
[INFO] [1763181372.959954279] [slider_action_controller]: 手臂轨迹发送完成，总耗时: 10.616705s
```

怀疑是USB接口问题, 进行抓包: 

  - lsusb, 找到usb设备的设备号: Bus 001 Device 015: ID 1a86:55d4 QinHeng Electronics USB Single Serial
  - 打开wireshark, 监听usbmon1 (bus 001对应)
  - 使用表达式来过滤掉不重要的包: 
    - 有效段的数据内容参考 <https://docs.elephantrobotics.com/docs/acc-cn/18-communication/18-communication.html>

```
usb.bus_id == 1 && usb.device_address == 15 && usb.capdata[0:4] != fe:fe:0e:20 && usb.capdata[0:4] != fe:fe:03:65 && usb.capdata[0:4] != fe:fe:03:65 && usb.capdata[0:4] != fe:fe:02:20
``` 
  - 总结命令序号表
    - 机械臂控制与状态指令

      - **0x10** : 机械臂上电
      - **0x11** : 机械臂掉电并断开连接
      - **0x12** : Atom状态查询
      - **0x13** : 机械臂仅掉电
      - **0x14** : 机器人系统检测正常
      - **0x16** : 指令刷新模式开关（设置插补/刷新运动模式）
      - **0x1A** : 机器人自由模式 (关闭所有扭力输出)
      - **0x1B** : 检查是否是自由模式

### 运动控制指令 (角度与坐标)

      - **0x20** : 读取角度（读取所有关节的角度信息）
      - **0x21** : 发送单独角度（控制单个关节运动到指定角度）
      - **0x22** : 发送全部角度（同时控制六个关节运动到指定角度）
      - **0x23** : 读取全部坐标（读取末端 x, y, z, rx, ry, rz 坐标）
      - **0x24** : 发送单独坐标参数（控制末端单个坐标轴运动）
      - **0x25** : 发送全部坐标参数（控制末端运动到指定坐标点）
      - **0x2A** : 判断是否达到指定点位（角度或坐标）
      - **0x2B** : 机械臂运动状态检测

### 程序流程控制指令

      - **0x26** : 程序暂停
      - **0x27** : 查询程序是否暂停
      - **0x28** : 程序恢复
      - **0x29** : 程序停止

### JOG 模式指令

      - **0x30** : JOG - 关节方向运动（控制单个关节持续转动）
      - **0x31** : JOG - 绝对控制（控制单个关节运动到指定角度，同 0x21）
      - **0x32** : JOG - 坐标方向运动（控制末端沿指定坐标轴持续移动）
      - **0x33** : JOG - 步进模式（控制单个关节在当前角度上增加/减少指定角度）

### 舵机底层控制指令 (电位值)

      - **0x3A** : 发送单个电位值
      - **0x3B** : 获取单个电位值
      - **0x3C** : 发送六个舵机的电位值
      - **0x3D** : 读取六个舵机的电位值

### 参数设置与读取指令

      - **0x41** : 设置速度
      - **0x4A** : 读取关节最小角度
      - **0x4B** : 读取关节最大角度
      - **0x4C** : 设置关节最小角度
      - **0x4D** : 设置关节最大角度

### 舵机状态与伺服指令

      - **0x50** : 查看单个舵机连接状态
      - **0x51** : 查看舵机是否全部上电
      - **0x52** : 设置舵机伺服参数（如 PID、最小启动力等）
      - **0x53** : 读取舵机伺服参数
      - **0x54** : 设置舵机当前位置为零点
      - **0x55** : 刹车单个电机
      - **0x56** : 单个电机掉电
      - **0x57** : 单个电机上电

### Atom IO 及 RGB 控制指令

      - **0x60** : 设置 Atom 引脚模式（输入/输出）
      - **0x61** : 设置 Atom IO 输出电平 (setDigitalOutput)
      - **0x62** : 读取 Atom IO 输入电平 (getDigitalInput)
      - **0x6A** : 设定 Atom 屏幕 RGB 灯的颜色

### 夹爪控制指令

      - **0x65** : 读取夹爪角度
      - **0x66** : 设置夹爪模式（张开/收拢）
      - **0x67** : 设置夹爪角度（张开幅度）
      - **0x68** : 夹爪设置零点
      - **0x69** : 检测夹爪是否运动

### 坐标系指令

      - **0x81** : 设置工具坐标系
      - **0x82** : 获取工具坐标系
      - **0x83** : 设置世界坐标系
      - **0x84** : 获取世界坐标系
      - **0x85** : 设置基坐标系（在基座坐标系和世界坐标系中切换）
      - **0x86** : 获取当前基坐标系
      - **0x89** : 设置末端坐标系（在法兰坐标系和工具坐标系中切换）
      - **0x8A** : 获取当前末端坐标系

### 底座 IO 指令

      - **0xA0** : 设置底座 IO 输出
      - **0xA1** : 读取底座 IO 输出

### 网络通信指令

      - **0xB1** : 获取 WiFi 账号和密码
      - **0xB2** : 设置 TCP/IP 端口号
    - 找到指定包: fe:fe:0f:22 (对应文档中: 发送全部角度)
    - 对抓包进行分析: <https://poe.com/s/Dlzib0ksfKRNBUjgbGc7>
    - 使用 MyCobot280(port, baud, debug=True) , 对python库进行debug, 获得结果: 

```
06:05:18.781 DEBU [pymycobot.generate] _read : fe fe 3 65 19 fa
[INFO] [1763186718.802102117] [slider_action_controller]: 收到新轨迹目标
06:05:18.805 DEBU [pymycobot.generate] _write: fe fe 02 20 fa
[INFO] [1763186718.811042910] [slider_action_controller]: 开始执行手臂轨迹
06:05:18.820 DEBU [pymycobot.generate] _read : fe fe e 20 0 8 0 11 ff ef 0 11 0 0 0 11 fa
06:05:18.821 DEBU [pymycobot.generate] _write: fe fe 0f 22 00 08 00 11 ff ef 00 11 00 00 00 11 23 fa
06:05:18.822 DEBU [pymycobot.generate] _write: fe fe 03 65 01 fa
06:05:18.829 DEBU [pymycobot.generate] _read : fe fe 3 22 1 fa
[INFO] [1763186718.834550250] [slider_action_controller]: ARM_SEND point 0/19 (async) 耗时: 0.021002s
06:05:18.835 DEBU [pymycobot.generate] _write: fe fe 0f 22 00 07 00 2d ff f0 00 10 00 00 00 10 23 fa
[INFO] [1763186718.847672579] [slider_action_controller]: ARM_SEND point 1/19 (async) 耗时: 0.011065s
06:05:18.848 DEBU [pymycobot.generate] _write: fe fe 0f 22 00 07 00 83 ff f0 00 10 ff ff 00 10 23 fa
06:05:18.853 DEBU [pymycobot.generate] _write: fe fe 02 20 fa
06:05:18.855 DEBU [pymycobot.generate] _read : fe fe 3 22 1 fa
[INFO] [1763186718.866157659] [slider_action_controller]: ARM_SEND point 2/19 (async) 耗时: 0.010928s
06:05:18.866 DEBU [pymycobot.generate] _write: fe fe 0f 22 00 07 01 12 ff f0 00 10 ff fd 00 10 23 fa
[INFO] [1763186718.879340511] [slider_action_controller]: ARM_SEND point 3/19 (async) 耗时: 0.011120s
06:05:18.879 DEBU [pymycobot.generate] _write: fe fe 0f 22 00 07 01 db ff f1 00 0f ff fa 00 10 23 fa
[INFO] [1763186718.892643994] [slider_action_controller]: ARM_SEND point 4/19 (async) 耗时: 0.011166s
06:05:18.893 DEBU [pymycobot.generate] _write: fe fe 0f 22 00 07 02 dd ff f2 00 0e ff f7 00 10 23 fa
06:05:18.903 DEBU [pymycobot.generate] _write: fe fe 02 20 fa
[INFO] [1763186718.907639247] [slider_action_controller]: ARM_SEND point 5/19 (async) 耗时: 0.011558s
06:05:19.406 DEBU [pymycobot.generate] _read :
06:05:19.407 DEBU [pymycobot.generate] _write: fe fe 02 20 fa
06:05:19.909 DEBU [pymycobot.generate] _read :
06:05:19.910 DEBU [pymycobot.generate] _write: fe fe 02 20 fa
06:05:20.412 DEBU [pymycobot.generate] _read :
06:05:20.413 DEBU [pymycobot.generate] _write: fe fe 0f 22 00 07 04 18 ff f3 00 0d ff f3 00 10 23 fa
06:05:20.416 DEBU [pymycobot.generate] _write: fe fe 02 20 fa
[INFO] [1763186720.428713194] [slider_action_controller]: ARM_SEND point 6/19 (async) 耗时: 1.518673s
06:05:20.919 DEBU [pymycobot.generate] _read :
06:05:20.920 DEBU [pymycobot.generate] _write: fe fe 02 20 fa
06:05:21.423 DEBU [pymycobot.generate] _read :
06:05:21.423 DEBU [pymycobot.generate] _write: fe fe 02 20 fa
06:05:21.926 DEBU [pymycobot.generate] _read :
06:05:21.927 DEBU [pymycobot.generate] _write: fe fe 02 20 fa
06:05:22.432 DEBU [pymycobot.generate] _read :
06:05:22.433 DEBU [pymycobot.generate] _write: fe fe 02 20 fa
06:05:22.934 DEBU [pymycobot.generate] _read :
06:05:22.934 DEBU [pymycobot.generate] _write: fe fe 02 20 fa
06:05:22.947 DEBU [pymycobot.generate] _read : fe fe e 20 0 8 0 11 ff ef 0 11 0 0 0 11 fa
06:05:22.948 DEBU [pymycobot.generate] _write: fe fe 02 20 fa
06:05:22.960 DEBU [pymycobot.generate] _read : fe fe e 20 0 8 0 11 ff ef 0 11 0 0 0 11 fa
06:05:22.961 DEBU [pymycobot.generate] _write: fe fe 02 20 fa
06:05:22.973 DEBU [pymycobot.generate] _read : fe fe e 20 0 8 0 11 ff ef 0 11 0 0 0 11 fa
06:05:22.974 DEBU [pymycobot.generate] _write: fe fe 02 20 fa
06:05:22.988 DEBU [pymycobot.generate] _read : fe fe e 20 0 8 0 11 ff ef 0 11 0 0 0 11 fa
06:05:22.989 DEBU [pymycobot.generate] _write: fe fe 02 20 fa
06:05:23.001 DEBU [pymycobot.generate] _read : fe fe e 20 0 8 0 11 ff ef 0 11 0 0 0 11 fa
06:05:23.002 DEBU [pymycobot.generate] _write: fe fe 02 20 fa
06:05:23.015 DEBU [pymycobot.generate] _read : fe fe e 20 0 8 0 11 ff ef 0 11 0 0 0 11 fa
06:05:23.016 DEBU [pymycobot.generate] _write: fe fe 0f 22 00 07 05 8c ff f5 00 0b ff ee 00 10 23 fa
06:05:23.017 DEBU [pymycobot.generate] _write: fe fe 02 20 fa
06:05:23.024 DEBU [pymycobot.generate] _read : fe fe 3 22 1 fa
06:05:23.025 DEBU [pymycobot.generate] _write: fe fe 03 65 01 fa
[INFO] [1763186723.030455930] [slider_action_controller]: ARM_SEND point 7/19 (async) 耗时: 2.598129s
06:05:23.086 DEBU [pymycobot.generate] _read : fe fe 3 65 19 fa
06:05:23.088 DEBU [pymycobot.generate] _write: fe fe 03 65 01 fa
06:05:23.592 DEBU [pymycobot.generate] _read :
06:05:23.593 DEBU [pymycobot.generate] _write: fe fe 03 65 01 fa
06:05:24.095 DEBU [pymycobot.generate] _read :
06:05:24.095 DEBU [pymycobot.generate] _write: fe fe 03 65 01 fa
06:05:24.598 DEBU [pymycobot.generate] _read :
06:05:24.600 DEBU [pymycobot.generate] _write: fe fe 03 65 01 fa
06:05:25.104 DEBU [pymycobot.generate] _read :
06:05:25.105 DEBU [pymycobot.generate] _write: fe fe 03 65 01 fa
06:05:25.114 DEBU [pymycobot.generate] _read : fe fe 3 65 19 fa
06:05:25.116 DEBU [pymycobot.generate] _write: fe fe 03 65 01 fa
06:05:25.177 DEBU [pymycobot.generate] _read : fe fe 3 65 63 fa
06:05:25.179 DEBU [pymycobot.generate] _write: fe fe 03 65 01 fa
06:05:25.196 DEBU [pymycobot.generate] _read : fe fe 3 65 19 fa
06:05:25.198 DEBU [pymycobot.generate] _write: fe fe 03 65 01 fa
06:05:25.701 DEBU [pymycobot.generate] _read :
06:05:25.702 DEBU [pymycobot.generate] _write: fe fe 03 65 01 fa
06:05:25.714 DEBU [pymycobot.generate] _read : fe fe 3 65 19 fa
06:05:25.716 DEBU [pymycobot.generate] _write: fe fe 02 20 fa
06:05:25.731 DEBU [pymycobot.generate] _read : fe fe e 20 ff ef 5 c4 0 3d 0 69 0 0 0 11 fa
06:05:25.731 DEBU [pymycobot.generate] _write: fe fe 0f 22 00 07 07 3a ff f6 00 09 ff e8 00 10 23 fa
06:05:25.732 DEBU [pymycobot.generate] _write: fe fe 02 20 fa
06:05:25.740 DEBU [pymycobot.generate] _read : fe fe 3 22 1 fa
06:05:25.741 DEBU [pymycobot.generate] _write: fe fe 02 20 fa
[INFO] [1763186725.745922318] [slider_action_controller]: ARM_SEND point 8/19 (async) 耗时: 2.712773s
```

      - 感觉上是状态发布 与 指令下发 之间在设备上有冲突
      - 将状态发布的 publish_rate降低, 发现机械臂调度开始顺滑
  - 将 状态发布和命令发布 互斥
    - commit: f80f40a9989e669890673f13cd86f7301bca61a3
  - 重新测试: 
    - -p fresh_mode:=0 -p arm_point_sleep_interval:=0.10 -p arm_async_send:=True -p publish_rate:=10.0
      - 轻微卡顿
    - -p fresh_mode:=0 -p arm_point_sleep_interval:=0.05 -p arm_async_send:=True -p publish_rate:=10.0
      - 轻微卡顿
    - -p fresh_mode:=0 -p arm_point_sleep_interval:=0.01 -p arm_async_send:=True -p publish_rate:=10.0
      - 轻微卡顿
    - -p fresh_mode:=0 -p arm_point_sleep_interval:=0.001 -p arm_async_send:=True -p publish_rate:=10.0
      - 轻微卡顿, 分成明显几段
    - -p fresh_mode:=0 -p arm_point_sleep_interval:=0.001 -p arm_async_send:=False -p publish_rate:=10.0
      - 轻微卡顿
    - -p fresh_mode:=1 -p arm_point_sleep_interval:=0.01 -p arm_async_send:=True -p publish_rate:=10.0
      - 非常顺滑
    - -p fresh_mode:=1 -p arm_point_sleep_interval:=0.10 -p arm_async_send:=True -p publish_rate:=10.0
      - 非常顺滑
    - -p fresh_mode:=1 -p arm_point_sleep_interval:=0.20 -p arm_async_send:=True -p publish_rate:=10.0
      - 产生卡顿
    - -p fresh_mode:=1 -p arm_point_sleep_interval:=0.10 -p arm_async_send:=False -p publish_rate:=10.0
      - 顺滑, 但略慢
    - -p fresh_mode:=1 -p arm_point_sleep_interval:=0.05 -p arm_async_send:=False -p publish_rate:=10.0
      - 非常顺滑
  - 测试结论: 
    - 设备的处理信息的频率大概是 10-20 Hz
    - arm_async_send, 测试上只是增加了 客户端的延迟, 与arm_point_sleep_interval的效果差不多
    - 同步模式 (fresh_mode = 0), 一定会产生轻微卡顿, 猜测是因为 "必须要达到某点, 所以得在该点减速", 所以每一个点都有点卡顿
    - 异步模式 (fresh_mode = 1), 会比较顺滑, 但需要调整 arm_point_sleep_interval 来调整发布频次, 来减少路径点被跳过的风险

# 总结经验

机械臂的调度逻辑: 

  - rviz界面或者其他调度脚本 -> moveit 服务 -> ros2网络 -> slider脚本, 提供ros2状态话题和控制服务 -> 机械臂的python库 (pymycobot) -> 串口 -> 机械臂
  - 状态话题 和 控制服务 需要分开, 之前将两者都放到 /joint_states 话题里, 产生了冲突
  - 状态话题 和 控制服务 需要互斥, 否则机械臂的内部处理上, 因为状态话题的轮询, 会导致控制服务产生很大卡顿
