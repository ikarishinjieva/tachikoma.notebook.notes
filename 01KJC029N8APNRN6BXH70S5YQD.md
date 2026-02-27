---
title: 20251004 - 在两个网络间，配置Isaac Sim和ROS2 demo, 进行机械手的教程
confluence_page_id: 4358710
created_at: 2025-10-04T06:23:16+00:00
updated_at: 2025-10-07T10:14:29+00:00
---

# 实验1 - 在Mac和Linux间建立UDP通道

一、服务器端准备（Linux）

  1. 启用 PermitTunnel

  - 编辑 /etc/ssh/sshd_config：  
PermitTunnel yes
  - 重启 sshd：sudo systemctl restart ssh 或等效命令

  1. 开启转发（可选但推荐）

  - 临时：sudo sysctl -w net.ipv4.ip_forward=1
  - 永久：写入 /etc/sysctl.conf 后 sysctl -p

二、建立 SSH TUN 隧道  
在 macOS 上发起 SSH 隧道。建议先用前台测试，确认成功后再 -f -N 后台运行。

  - 测试前台：  
sudo ssh -vv -w 0:0 root[@10.186.16.135](<mailto:user@10.186.16.135>)  
成功后会在两端各自创建一个 TUN 设备（macOS 是 utunX，Linux 是 tun0，若已有占用可能是 tun1 等）。

  - 稳定用法（后台）：  
sudo ssh -f -N -w 0:0 root[@10.186.16.135](<mailto:user@10.186.16.135>)

注意：

  - macOS 上 ssh -w 需要 sudo，否则无法创建 utun。
  - 若报错“Tunnel device open failed”或“tun not permitted”，检查服务器的 PermitTunnel 和本机权限。

三、为隧道接口配置 IP  
给两端的 TUN 接口分配同一网段的点对点地址，例如 10.200.0.1/30 和 10.200.0.2/30。

  1. macOS 端

  - 查看新建的 utun 名称：  
ifconfig | grep -A3 tun  
找到最新出现的 tunX，比如 tun0。

  - 配置地址（点对点语义在 macOS 用 ifconfig 的 tunnel 关键字实现）：  
sudo ifconfig tun0 10.200.0.1 10.200.0.2 up

说明：

  - 这里的语法等价于在 utun0 上设置“本地地址 10.200.0.1，对端地址 10.200.0.2”的点对点链路。
  - 若报权限或参数错误，确认 utun 名称、ssh 隧道已建立。

  1. Linux 服务器端

  - 识别 TUN 设备名（通常是 tun0）：  
ip link
  - 配置地址：  
sudo ip addr add 10.200.0.2/30 peer 10.200.0.1 dev tun0  
sudo ip link set tun0 up

四、连通性验证

  - 从 macOS：  
ping 10.200.0.2
  - 从服务器：  
ping 10.200.0.1

五、在隧道上进行 UDP 测试

  - 服务器上监听：  
sudo nc -u -l -p 11812
  - macOS 上发送：  
echo 'udp test' | nc -u 10.200.0.2 11812

如果需要从服务器发回到 macOS：

  - macOS 上也可以监听：  
nc -u -l 11813
  - 服务器发送：  
echo 'hi' | nc -u 10.200.0.1 11813

# 实验2 - 在Linux和Linux间建立UDP通道

前提

  - 双方都是 Linux，均有 root 或 sudo 权限。
  - Server 的 sshd 允许 TUN。
  - 双方内核支持 TUN（大多数发行版默认支持）。

一、Server 端准备

  1. 启用 TUN 支持与设备节点

  - 如果是模块制：sudo modprobe tun
  - 确保设备节点存在：  
sudo mkdir -p /dev/net  
[ -c /dev/net/tun ] || (sudo mknod /dev/net/tun c 10 200 && sudo chmod 600 /dev/net/tun)

  1. 开启 sshd 对 TUN 的支持

  - 编辑 /etc/ssh/sshd_config，确保有：  
PermitTunnel point-to-point  
AllowTcpForwarding yes
  - 重启 sshd：  
sudo systemctl restart sshd

  1. 可选：开启 IP 转发（若要通过隧道访问对端后面的网段）

  - 临时：  
sudo sysctl -w net.ipv4.ip_forward=1
  - 永久写入：  
echo 'net.ipv4.ip_forward=1' | sudo tee /etc/sysctl.d/99-tun.conf && sudo sysctl -p /etc/sysctl.d/99-tun.conf

二、建立隧道（由 Client 发起）  
在 Client 上执行（必须 sudo，否则无法创建 tun 设备）：

  - 前台调试：  
sudo ssh -vv -w 0:0 user@SERVER_IP
  - 成功后会在两端各出现一个 tun 接口（可能是 tun0，如已占用会是 tun1 等）。确认接口名：
    - Client: ip link | grep -A1 tun
    - Server: ip link | grep -A1 tun

也可直接后台运行：

  - sudo ssh -f -N -w 0:0 user@SERVER_IP

三、为隧道接口配置 IP  
假设两端接口分别是 tun0（若不同请替换为实际名称）：

  - Client：  
sudo ip addr add 10.200.0.1/30 peer 10.200.0.2 dev tun0  
sudo ip link set tun0 up

  - Server：  
sudo ip addr add 10.200.0.2/30 peer 10.200.0.1 dev tun0  
sudo ip link set tun0 up

四、连通性测试

  - 在 Client：  
ping -c 3 10.200.0.2
  - 在 Server：  
ping -c 3 10.200.0.1

五、在隧道上跑你的 UDP/TCP 流量

  - 例如在 Server 监听 UDP 11812：  
sudo nc -u -l -p 11812
  - 在 Client 发送：  
echo 'hello' | nc -u 10.200.0.2 11812

六、可选：通过隧道路由更多网段

  - 让 Client 通过隧道访问 Server 后面的 10.186.16.0/24：  
Client 上添加：  
sudo ip route add 10.186.16.0/24 via 10.200.0.2 dev tun0
  - 让 Server 通过隧道访问 Client 后面的 172.16.169.0/24：  
Server 上添加：  
sudo ip route add 172.16.169.0/24 via 10.200.0.1 dev tun0
  - 确保双方开启了 IP 转发，并在各自的防火墙上放行相应流量。

# 实验3 - 验证ROS连接

````
### 详细步骤

#### 第1步：在机器 A 上准备 Docker 环境

如果您还没有安装 Docker，请先安装。

```bash
# 安装 Docker
sudo apt update
sudo apt install docker.io

# （推荐）将当前用户添加到 docker 组，这样就不用每次都加 sudo
# 注意：执行后需要重新登录或重启终端才能生效
sudo usermod -aG docker $USER
newgrp docker # 立即在当前终端生效
```
接下来，拉取 ROS 2 的官方 Docker 镜像（我们以 Humble 为例）：
```bash
docker pull ros:humble
```

#### 第2步：在机器 A 上启动发现服务器（在 Docker 中）

1.  在机器 A 上打开一个**新的终端（服务器终端）**。
2.  运行以下 `docker run` 命令。

    ```bash
    # 解释：
    # --rm: 容器停止后自动删除，保持系统干净。
    # -it:  以交互模式运行，这样我们能看到服务器的日志。
    # --net=host: 关键！让容器共享主机的网络。
    # ros:humble: 使用的镜像。
    # bash -c "...": 在容器内执行的命令。
    docker run --rm -it --net=host ros:humble \
    bash -c "source /opt/ros/humble/setup.bash && fastdds discovery -i 0 -l 192.168.1.10 -p 11811"
    ```
    *   **重要提示**：请将 `192.168.1.10` 替换为**机器 A 的实际 IP 地址**。

3.  执行后，您会看到服务器启动的日志。**请保持这个终端开启**。
    ```
    Server listening on 192.168.1.10:11811
    ```
    由于使用了 `--net=host`，这个服务现在就在 `192.168.1.10:11811` 上对整个局域网可见。

#### 第3步：在机器 A 上启动发布者（在另一个 Docker 中）

1.  在机器 A 上打开**另一个新的终端（发布者终端）**。
2.  运行以下 `docker run` 命令来启动 `talker` 节点。

    ```bash
    # 解释：
    # -e ROS_DISCOVERY_SERVER=... : 通过 -e 参数将环境变量传递到容器内部。
    docker run --rm -it --net=host \
    -e ROS_DISCOVERY_SERVER="192.168.1.10:11811" \
    ros:humble \
    bash -c "source /opt/ros/humble/setup.bash && ros2 run demo_nodes_cpp talker"
    ```
    *   同样，请确保这里的 IP 地址是机器 A 的正确 IP。

3.  您会看到 `talker` 节点开始发布消息的日志。

#### 第4步：在机器 B 上启动订阅者（原生环境）

现在，我们去机器 B 上验证连接。

1.  在机器 B 上打开一个终端。
2.  Source 您的原生 ROS 2 环境。
3.  设置 `ROS_DISCOVERY_SERVER` 环境变量，指向**机器 A 的 IP 地址**。
4.  启动 `listener` 节点。

    ```bash
    # Source 原生 ROS 2 环境
    source /opt/ros/humble/setup.bash

    # 设置发现服务器地址，指向机器 A！
    export ROS_DISCOVERY_SERVER="192.168.1.10:11811"

    # 运行 listener 节点
    ros2 run demo_nodes_cpp listener
    ```

#### 第5步：验证最终结果

*   **在机器 B 的终端**，您应该能清晰地看到从机器 A 的 Docker 容器中发出的消息：
    ```
    [INFO] [listener]: I heard: [Hello World: 1]
    [INFO] [listener]: I heard: [Hello World: 2]
    ...
    ```
*   **在机器 A 的服务器终端**，您会看到有两个客户端连接的日志，一个来自 `192.168.1.10`（发布者容器），另一个来自 `192.168.1.20`（机器 B）。

**如果这个流程成功，那么您已经完美地验证了：机器 A 上运行的发现服务器是有效的，并且机器 B 可以通过网络成功连接到它，并在此基础上建立 ROS 2 通信。**

```` 

在运行talker/listener时, ros2 node list 仍然显示为空, 需要执行: 

```
ros2 node list --spin-time 10 --no-daemon 
``` 

可以获取node

# 实验4 - 给Isaac Sim配置ROS2

根据文档， 安装ROS2： <https://docs.isaacsim.omniverse.nvidia.com/latest/installation/install_ros.html#isaac-sim-app-install-native-ros-default>

```
locale  # check for UTF-8

sudo apt update && sudo apt install locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8

locale  # verify settings

sudo apt install software-properties-common
sudo add-apt-repository universe

sudo apt update && sudo apt install curl -y
export ROS_APT_SOURCE_VERSION=$(curl -s https://api.github.com/repos/ros-infrastructure/ros-apt-source/releases/latest | grep -F "tag_name" | awk -F\" '{print $4}')
curl -L -o /tmp/ros2-apt-source.deb "https://github.com/ros-infrastructure/ros-apt-source/releases/download/${ROS_APT_SOURCE_VERSION}/ros2-apt-source_${ROS_APT_SOURCE_VERSION}.$(. /etc/os-release && echo ${UBUNTU_CODENAME:-${VERSION_CODENAME}})_all.deb"
sudo dpkg -i /tmp/ros2-apt-source.deb

sudo apt update && sudo apt install -y \
  python3-flake8-blind-except \
  python3-flake8-class-newline \
  python3-flake8-deprecated \
  python3-mypy \
  python3-pip \
  python3-pytest \
  python3-pytest-cov \
  python3-pytest-mock \
  python3-pytest-repeat \
  python3-pytest-rerunfailures \
  python3-pytest-runner \
  python3-pytest-timeout \
  ros-dev-tools

mkdir -p /home/tachikoma/Code/ros2_jazzy/src
cd /home/tachikoma/Code/ros2_jazzy
vcs import --input https://raw.githubusercontent.com/ros2/ros2/jazzy/ros2.repos src

sudo apt update

sudo rosdep init
rosdep update
rosdep install --from-paths src --ignore-src -y --skip-keys "fastcdr rti-connext-dds-6.0.1 urdfdom_headers"

cd /home/tachikoma/Code/ros2_jazzy/
colcon build --symlink-install

source /home/tachikoma/Code/ros2_jazzy/install/local_setup.bash
``` 

在apt install时， 会遇到以来版本低， 当前安装版本高的问题, 需要手工设置版本

# 实验5 - 启动Isaac Sim时， 使用编译的ROS

```
source /home/tachikoma/Code/ros2_jazzy/install/local_setup.bash

export FASTRTPS_DEFAULT_PROFILES_FILE=~/.ros/fastdds.xml

export isaac_sim_package_path=/home/tachikoma/Code/isaacsim

export ROS_DISTRO=jazzy

export RMW_IMPLEMENTATION=rmw_fastrtps_cpp

# Can only be set once per terminal.
# Setting this command multiple times will append the internal library path again potentially leading to conflicts
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$isaac_sim_package_path/exts/isaacsim.ros2.bridge/jazzy/lib

# Run Isaac Sim
$isaac_sim_package_path/isaac-sim.sh
``` 

# 实验6 - 使用配置文件, 配置ROS2节点发现

fastdds_client.xml 

```
<?xml version="1.0" encoding="UTF-8" ?>
 <dds>
     <profiles xmlns="http://www.eprosima.com/XMLSchemas/fastRTPS_Profiles">
         <participant profile_name="super_client_profile" is_default_profile="true">
             <rtps>
                 <builtin>
                     <discovery_config>
                         <discoveryProtocol>SUPER_CLIENT</discoveryProtocol>
                         <discoveryServersList>
                             <RemoteServer prefix="44.53.00.5f.45.50.52.4f.53.49.4d.41">
                                 <metatrafficUnicastLocatorList>
                                     <locator>
                                         <udpv4>
                                             <address>127.0.0.1</address>
                                             <port>11811</port>
                                         </udpv4>
                                     </locator>
                                 </metatrafficUnicastLocatorList>
                             </RemoteServer>
                         </discoveryServersList>
                     </discovery_config>
                 </builtin>
             </rtps>
         </participant>
     </profiles>
 </dds>
``` 

以上配置可以完成talker/listener实验: 

```
# 启动server
fastdds discovery -i 0 -l 10.200.0.1 -p 11811

# 指定配置文件
export FASTRTPS_DEFAULT_PROFILES_FILE=~/.ros/fastdds.xml
 
# 启动talker
ros2 run demo_nodes_cpp talker
 
# 启动listener
ros2 run demo_nodes_cpp listener
 
 
# 如果关闭server, 则两个节点无法互相通信
``` 

# 重新整理所有步骤

背景:

在 [20250928 - isaac 机器人定义](<http://8.134.54.170:8330/pages/viewpage.action?pageId=4358667>) 中 "在 Isaac Sim 虚拟环境中使用 Isaac Manipulator 参考工作流" 章节进行了初步分析, 本文中进行实现

  - <https://nvidia-isaac-ros.github.io/reference_workflows/isaac_manipulator/tutorials/tutorial_isaac_sim.html>

目标： 

  - 在本地笔记本安装Isaac Sim
  - 在公司服务器 10.186.16.135 安装 ROS2 DEMO
  - 本地笔记本使用VPN连接公司服务器, 公司服务器不能ping本地笔记本
  - 目标: 使用公司服务器的ROS2 DEMO, 调度本地笔记本的Isaac Sim

在本地笔记本安装Isaac Sim:

  - 参考: [20251003 - 重新配置isaac和demo的笔记本] 中的"安装Isaac"章节

在本地笔记本安装ROS2:

  - 参考本文档的"实验4 - 给Isaac Sim配置ROS2"章节

在公司服务器 10.186.16.135 安装 ROS2 DEMO:

  - 参考: [20251003 - 重新配置isaac和demo的笔记本] 中的"安装Demo"章节

配置两台机器的ssh连接 (TUN转发)

  - 背景参考: 本文的"实验2 - 在Linux和Linux间建立UDP通道"

```
# 从笔记本发起ssh TUN转发连接
sudo ssh -vv -w 0:0 root@10.186.16.135

# (新起session) 在笔记本上, 配置IP
 sudo ip addr add 10.200.0.1/30 peer 10.200.0.2 dev tun0
 sudo ip link set tun0 up

# 在服务器上, 配置IP
 sudo ip addr add 10.200.0.2/30 peer 10.200.0.1 dev tun0
 sudo ip link set tun0 up

# 测试UDP转发
在一端
 sudo nc -u -l -p 11812
在另一端
 echo 'hello' | nc -u 10.200.0.2 11812
```

在笔记本上, 启动FASTDDS发现服务器:

```
source /home/tachikoma/Code/ros2_jazzy/install/local_setup.bash

# 启动server
fastdds discovery -i 0 -l 10.200.0.1 -p 11811
``` 

在服务器上, 测试ROS2通信:

  - 在笔记本和服务器上, 都增加fastdds_client.xml文件:

```
 <?xml version="1.0" encoding="UTF-8" ?>
 <dds>
     <profiles xmlns="http://www.eprosima.com/XMLSchemas/fastRTPS_Profiles">
         <participant profile_name="super_client_profile" is_default_profile="true">
             <rtps>
                 <builtin>
                     <discovery_config>
                         <discoveryProtocol>SUPER_CLIENT</discoveryProtocol>
                         <discoveryServersList>
                             <RemoteServer prefix="44.53.00.5f.45.50.52.4f.53.49.4d.41">
                                 <metatrafficUnicastLocatorList>
                                     <locator>
                                         <udpv4>
                                             <address>10.200.0.1</address>
                                             <port>11811</port>
                                         </udpv4>
                                     </locator>
                                 </metatrafficUnicastLocatorList>
                             </RemoteServer>
                         </discoveryServersList>
                     </discovery_config>
                 </builtin>
             </rtps>
         </participant>
     </profiles>
 </dds>
```

  - 在服务器上, 启动ROS2环境, 并启动talker来测试

```
export ISAAC_ROS_WS=/data/huangyan/isaac_ros-dev
cd $ISAAC_ROS_WS && ./src/isaac_ros_common/scripts/run_dev.sh

# 进入容器
export FASTRTPS_DEFAULT_PROFILES_FILE=/workspaces/isaac_ros-dev/fastdds_client.xml
ros2 run demo_nodes_cpp talker

```

  - 在笔记本上, 启动ROS2环境, 并启动listener来获取测试结果

```
source /home/tachikoma/Code/ros2_jazzy/install/local_setup.bash
export FASTRTPS_DEFAULT_PROFILES_FILE=/home/tachikoma/Code/fastdds_client.xml
ros2 run demo_nodes_cpp listener

# 应当能看到如下日志
[INFO] [1759670865.206302163] [listener]: I heard: [Hello World: 155]
[INFO] [1759670866.203287395] [listener]: I heard: [Hello World: 156]
[INFO] [1759670867.203603028] [listener]: I heard: [Hello World: 157]
[INFO] [1759670868.246075837] [listener]: I heard: [Hello World: 158]
```

  - 在笔记本上, 启动Isaac Sim

```
source /home/tachikoma/Code/ros2_jazzy/install/local_setup.bash
export FASTRTPS_DEFAULT_PROFILES_FILE=/home/tachikoma/Code/fastdds_client.xml

export isaac_sim_package_path=/home/tachikoma/Code/isaac-sim

export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$isaac_sim_package_path/exts/isaacsim.ros2.bridge/jazzy/lib

$isaac_sim_package_path/isaac-sim.sh
```

    - 在Isaac Sim中, 打开 <https://omniverse-content-production.s3-us-west-2.amazonaws.com/Assets/Isaac/4.2/Isaac/Samples/ROS2/Scenario/isaac_manipulator_ur10e_robotiq_2f_140.usd>
    - 并点击播放按钮
  - 在服务器上, 启动ROS2 Demo

```
export ISAAC_ROS_WS=/data/huangyan/isaac_ros-dev
cd $ISAAC_ROS_WS && ./src/isaac_ros_common/scripts/run_dev.sh

# 进入容器
export FASTRTPS_DEFAULT_PROFILES_FILE=/workspaces/isaac_ros-dev/fastdds_client.xml
cd ${ISAAC_ROS_WS}
source install/setup.bash
ros2 launch isaac_manipulator_pick_and_place isaac_sim_workflows.launch.py \
   workflow_type:=object_following pose_estimation_type:=foundationpose
```

  - 可以看到, 两端已有通信, 在Isaac Sim中, 机械臂有时能动一下
  - 在服务器上, 会直接OOM, 并且报错: 需要检查原因 

```
[component_container_mt-1] 2025-10-05 13:34:51.065 ERROR ./gxf/foundationpose/mesh_storage.cpp@172: [MeshStorage] Mesh file does not exist at path: /workspaces/isaac_ros-dev/isaac_ros_assets/isaac_ros_foundationpose/soup_can/soup_can.obj
[component_container_mt-1] 2025-10-05 13:34:51.065 ERROR ./gxf/foundationpose/mesh_storage.cpp@75: [MeshStorage] Failed to load mesh file
[component_container_mt-1] 2025-10-05 13:34:51.065 ERROR gxf/std/entity_warden.cpp@548: Failed to initialize component 00512 (mesh_storage)
[component_container_mt-1] 2025-10-05 13:34:51.065 ERROR gxf/core/runtime.cpp@742: Could not initialize entity 'MPCBOSUCKK_utils' (E507): GXF_FAILURE
[component_container_mt-1] 2025-10-05 13:34:51.065 ERROR gxf/std/program.cpp@289: Failed to activate entity 00507 named MPCBOSUCKK_utils: GXF_FAILURE
[component_container_mt-1] 2025-10-05 13:34:51.065 ERROR gxf/std/program.cpp@291: Deactivating...
[component_container_mt-1] 2025-10-05 13:34:51.065 ERROR gxf/core/runtime.cpp@777: Could not unschedule entity 'MPCBOSUCKK_utils' (E507) from execution: GXF_ENTITY_COMPONENT_NOT_FOUND
[component_container_mt-1] 2025-10-05 13:34:51.065 ERROR gxf/std/program.cpp@293: Deactivation failed.
[component_container_mt-1] 2025-10-05 13:34:51.065 ERROR gxf/core/runtime.cpp@1625: Graph activation failed with error: GXF_FAILURE
[component_container_mt-1] [ERROR] [1759671291.065955765] [foundationpose_node]: [NitrosContext] GxfGraphActivate Error: GXF_FAILURE
[component_container_mt-1] [ERROR] [1759671291.066006741] [foundationpose_node]: [NitrosNode] runGraphAsync Error: GXF_FAILURE
[component_container_mt-1] terminate called after throwing an instance of 'std::runtime_error'

```

      - isaac_ros_assets 确实不存在, 重新执行 <http://8.134.54.170:8330/pages/viewpage.action?pageId=4358693> 中 配置视觉模型的部分步骤

  - 修复后仍有报错 (需要先启动 ROS2 DEMO, 在启动Isaac Sim, 否则容易OOM)

```
[component_container_mt-1] Could not open file model.onnx
[component_container_mt-1] 2025-10-05 16:13:13.082 ERROR ./gxf/extensions/tensor_rt/tensor_rt_inference.cpp@154: TRT ERROR: ModelImporter.cpp:914: Failed to parse ONNX model from file: model.onnx!
[component_container_mt-1] 2025-10-05 16:13:13.082 ERROR ./gxf/extensions/tensor_rt/tensor_rt_inference.cpp@442: Failed to parse ONNX file model.onnx
[component_container_mt-1] 2025-10-05 16:13:13.082 ERROR ./gxf/extensions/tensor_rt/tensor_rt_inference.cpp@290: Failed to create engine plan for model model.onnx.
[component_container_mt-1] 2025-10-05 16:13:13.082 ERROR gxf/std/entity_executor.cpp@630: Entity [SZPENJMWLT_inference] must be in Started, Tick Pending, Ticking or Idle stage before stopping. Current state is StartPending
[component_container_mt-1] Could not open file /tmp/refine_model.onnx
[component_container_mt-1] Could not open file /tmp/score_model.onnx
[component_container_mt-1] 2025-10-05 16:13:13.083 ERROR ./gxf/extensions/tensor_rt/tensor_rt_inference.cpp@154: TRT ERROR: ModelImporter.cpp:914: Failed to parse ONNX model from file: /tmp/score_model.onnx!
[component_container_mt-1] 2025-10-05 16:13:13.083 ERROR ./gxf/extensions/tensor_rt/tensor_rt_inference.cpp@442: Failed to parse ONNX file /tmp/score_model.onnx
[component_container_mt-1] 2025-10-05 16:13:13.083 ERROR ./gxf/extensions/tensor_rt/tensor_rt_inference.cpp@154: TRT ERROR: ModelImporter.cpp:914: Failed to parse ONNX model from file: /tmp/refine_model.onnx!
[component_container_mt-1] 2025-10-05 16:13:13.083 ERROR ./gxf/extensions/tensor_rt/tensor_rt_inference.cpp@290: Failed to create engine plan for model /tmp/score_model.onnx.
[component_container_mt-1] 2025-10-05 16:13:13.083 ERROR ./gxf/extensions/tensor_rt/tensor_rt_inference.cpp@442: Failed to parse ONNX file /tmp/refine_model.onnx
[component_container_mt-1] 2025-10-05 16:13:13.126 ERROR gxf/std/entity_executor.cpp@212: Entity with eid 224 not found!
[component_container_mt-1] 2025-10-05 16:13:13.227 ERROR gxf/core/runtime.cpp@1641: Graph interrupt failed with error: GXF_FAILURE
[component_container_mt-1] [ERROR] [1759680793.227281575] [rtdetr_tensor_rt]: [NitrosContext] GxfGraphInterrupt Error: GXF_FAILURE
[component_container_mt-1] 2025-10-05 16:13:13.227 ERROR gxf/std/program.cpp@580: wait failed. Deactivating...
[component_container_mt-1] 2025-10-05 16:13:13.293 ERROR gxf/core/runtime.cpp@791: Could not deinitialize entity 'SZPENJMWLT_utils' (E245): GXF_FAILURE
[component_container_mt-1] 2025-10-05 16:13:13.293 ERROR gxf/std/program.cpp@582: Deactivation failed.
[component_container_mt-1] 2025-10-05 16:13:13.293 ERROR gxf/core/runtime.cpp@1649: Graph wait failed with error: GXF_FAILURE
[component_container_mt-1] [ERROR] [1759680793.293985369] [rtdetr_tensor_rt]: [NitrosContext] GxfGraphWait Error: GXF_FAILURE
[component_container_mt-1] 2025-10-05 16:13:13.330 ERROR ./gxf/extensions/tensor_rt/tensor_rt_inference.cpp@290: Failed to create engine plan for model /tmp/refine_model.onnx.
[component_container_mt-1] 2025-10-05 16:13:13.371 ERROR gxf/std/entity_executor.cpp@630: Entity [EFCMUEZZMT_refine_inference] must be in Started, Tick Pending, Ticking or Idle stage before stopping. Current state is StartPending
[component_container_mt-1] 2025-10-05 16:13:13.371 ERROR gxf/std/entity_executor.cpp@630: Entity [EFCMUEZZMT_score_inference] must be in Started, Tick Pending, Ticking or Idle stage before stopping. Current state is StartPending
[component_container_mt-1] 2025-10-05 16:13:13.448 ERROR gxf/core/runtime.cpp@1641: Graph interrupt failed with error: GXF_FAILURE
[component_container_mt-1] [ERROR] [1759680793.448431795] [foundationpose_node]: [NitrosContext] GxfGraphInterrupt Error: GXF_FAILURE
[component_container_mt-1] 2025-10-05 16:13:13.448 ERROR gxf/std/program.cpp@580: wait failed. Deactivating...
[component_container_mt-1] 2025-10-05 16:13:13.494 ERROR gxf/core/runtime.cpp@791: Could not deinitialize entity 'EFCMUEZZMT_utils' (E507): GXF_FAILURE
[component_container_mt-1] 2025-10-05 16:13:13.494 ERROR gxf/std/program.cpp@582: Deactivation failed.
[component_container_mt-1] 2025-10-05 16:13:13.494 ERROR gxf/core/runtime.cpp@1649: Graph wait failed with error: GXF_FAILURE
[component_container_mt-1] [ERROR] [1759680793.494570738] [foundationpose_node]: [NitrosContext] GxfGraphWait Error: GXF_FAILURE

```

    - 跟上一个报错一样, 需要重新执行 [20251001 - 在Isaac Sim中, 进行机械手的教程] 中 配置视觉模型的**全部步骤**
  - 仍然会OOM， 占用内存的大户： 

```
admin@r750-135:/workspaces/isaac_ros-dev$ ps aux | grep 19255
admin      19255 99.9 77.7 283505864 204837948 pts/6 Sl 16:08   2:50 /usr/bin/python3 /opt/ros/humble/lib/isaac_ros_cumotion_object_attachment/attach_object_server_node --ros-args -r __node:=object_attachment -r __ns:=/ -p use_sim_time:=True -p sub_qos:=DEFAULT -p pub_qos:=DEFAULT --params-file /tmp/launch_params__kqw4i6_

```

    - 目前是先启动ROS2 demo, 然后再启动Isaac Sim, 有概率能运行
  - 运行一段时间, 机械臂会移动, 然后报错: 
    - 额外发现: 笔记本的网络流量很高, 大概是6M/s
    - 报错内容: 

```
[attach_object_server_node-6] [INFO] [1759682895.219527552] [object_attachment]: Service(nvblox_node/get_esdf_and_gradient) not available, waiting again...
[cumotion_goal_set_planner_node-4] [INFO] [1759682895.455610663] [cumotion_planner]: Service(/nvblox_node/get_esdf_and_gradient) not available, waiting again...
[attach_object_server_node-6] [INFO] [1759682896.221834895] [object_attachment]: Service(nvblox_node/get_esdf_and_gradient) not available, waiting again...
[cumotion_goal_set_planner_node-4] [INFO] [1759682896.457966808] [cumotion_planner]: Service(/nvblox_node/get_esdf_and_gradient) not available, waiting again...
[attach_object_server_node-6] [INFO] [1759682897.224117899] [object_attachment]: Service(nvblox_node/get_esdf_and_gradient) not available, waiting again...
[cumotion_goal_set_planner_node-4] [INFO] [1759682897.460295918] [cumotion_planner]: Service(/nvblox_node/get_esdf_and_gradient) not available, waiting again...
[attach_object_server_node-6] [INFO] [1759682898.226394484] [object_attachment]: Service(nvblox_node/get_esdf_and_gradient) not available, waiting again...
[cumotion_goal_set_planner_node-4] [INFO] [1759682898.462610576] [cumotion_planner]: Service(/nvblox_node/get_esdf_and_gradient) not available, waiting again...
[attach_object_server_node-6] [INFO] [1759682899.228662456] [object_attachment]: Service(nvblox_node/get_esdf_and_gradient) not available, waiting again...
[cumotion_goal_set_planner_node-4] [INFO] [1759682899.464877334] [cumotion_planner]: Service(/nvblox_node/get_esdf_and_gradient) not available, waiting again...
[attach_object_server_node-6] [INFO] [1759682900.230934609] [object_attachment]: Service(nvblox_node/get_esdf_and_gradient) not available, waiting again...
[cumotion_goal_set_planner_node-4] [INFO] [1759682900.467162653] [cumotion_planner]: Service(/nvblox_node/get_esdf_and_gradient) not available, waiting again...
[attach_object_server_node-6] [INFO] [1759682901.233173354] [object_attachment]: Service(nvblox_node/get_esdf_and_gradient) not available, waiting again...
[cumotion_goal_set_planner_node-4] [INFO] [1759682901.469428001] [cumotion_planner]: Service(/nvblox_node/get_esdf_and_gradient) not available, waiting again...
[attach_object_server_node-6] [INFO] [1759682902.235661945] [object_attachment]: Service(nvblox_node/get_esdf_and_gradient) not available, waiting again...
[cumotion_goal_set_planner_node-4] [INFO] [1759682902.471708637] [cumotion_planner]: Service(/nvblox_node/get_esdf_and_gradient) not available, waiting again...
[attach_object_server_node-6] [INFO] [1759682903.238081160] [object_attachment]: Service(nvblox_node/get_esdf_and_gradient) not available, waiting again...
[cumotion_goal_set_planner_node-4] [INFO] [1759682903.473987225] [cumotion_planner]: Service(/nvblox_node/get_esdf_and_gradient) not available, waiting again...
[attach_object_server_node-6] [INFO] [1759682903.740339719] [object_attachment]: Node initialized with 1 cameras
[cumotion_goal_set_planner_node-4] [INFO] [1759682905.948926621] [cumotion_planner]: warming up cuMotion, wait until ready
[spawner-13] [WARN] [1759682913.675309484] [spawner_joint_state_broadcaster]: Failed getting a result from calling /controller_manager/list_controllers in 10.0. (Attempt 1 of 3.)
[spawner-11] [WARN] [1759682913.675888174] [spawner_scaled_joint_trajectory_controller]: Failed getting a result from calling /controller_manager/list_controllers in 10.0. (Attempt 1 of 3.)
[cumotion_goal_set_planner_node-4] [INFO] [1759682915.182871494] [cumotion_planner]: cuMotion is ready for planning queries!
[spawner-13] [WARN] [1759682923.677295323] [spawner_joint_state_broadcaster]: Failed getting a result from calling /controller_manager/list_controllers in 10.0. (Attempt 2 of 3.)
[spawner-11] [WARN] [1759682923.677538141] [spawner_scaled_joint_trajectory_controller]: Failed getting a result from calling /controller_manager/list_controllers in 10.0. (Attempt 2 of 3.)
[spawner-11] [WARN] [1759682933.679255109] [spawner_scaled_joint_trajectory_controller]: Failed getting a result from calling /controller_manager/list_controllers in 10.0. (Attempt 3 of 3.)
[spawner-13] [WARN] [1759682933.679281584] [spawner_joint_state_broadcaster]: Failed getting a result from calling /controller_manager/list_controllers in 10.0. (Attempt 3 of 3.)
[spawner-11] Traceback (most recent call last):
[spawner-11]   File "/opt/ros/humble/lib/controller_manager/spawner", line 33, in <module>
[spawner-11]     sys.exit(load_entry_point('controller-manager==2.51.0', 'console_scripts', 'spawner')())
[spawner-11]   File "/opt/ros/humble/local/lib/python3.10/dist-packages/controller_manager/spawner.py", line 195, in main
[spawner-11]     if is_controller_loaded(
[spawner-11]   File "/opt/ros/humble/local/lib/python3.10/dist-packages/controller_manager/spawner.py", line 67, in is_controller_loaded
[spawner-11]     controllers = list_controllers(
[spawner-11]   File "/opt/ros/humble/local/lib/python3.10/dist-packages/controller_manager/controller_manager_services.py", line 172, in list_controllers
[spawner-11]     return service_caller(
[spawner-11]   File "/opt/ros/humble/local/lib/python3.10/dist-packages/controller_manager/controller_manager_services.py", line 150, in service_caller
[spawner-11]     raise RuntimeError(
[spawner-11] RuntimeError: Could not successfully call service /controller_manager/list_controllers after 3 attempts.
[spawner-13] Traceback (most recent call last):
[spawner-13]   File "/opt/ros/humble/lib/controller_manager/spawner", line 33, in <module>
[spawner-13]     sys.exit(load_entry_point('controller-manager==2.51.0', 'console_scripts', 'spawner')())
[spawner-13]   File "/opt/ros/humble/local/lib/python3.10/dist-packages/controller_manager/spawner.py", line 195, in main
[spawner-13]     if is_controller_loaded(
[spawner-13]   File "/opt/ros/humble/local/lib/python3.10/dist-packages/controller_manager/spawner.py", line 67, in is_controller_loaded
[spawner-13]     controllers = list_controllers(
[spawner-13]   File "/opt/ros/humble/local/lib/python3.10/dist-packages/controller_manager/controller_manager_services.py", line 172, in list_controllers
[spawner-13]     return service_caller(
[spawner-13]   File "/opt/ros/humble/local/lib/python3.10/dist-packages/controller_manager/controller_manager_services.py", line 150, in service_caller
[spawner-13]     raise RuntimeError(
[spawner-13] RuntimeError: Could not successfully call service /controller_manager/list_controllers after 3 attempts.
[ERROR] [spawner-13]: process has died [pid 31583, exit code 1, cmd '/opt/ros/humble/lib/controller_manager/spawner joint_state_broadcaster --controller-manager /controller_manager --ros-args -p use_sim_time:=True -p sub_qos:=DEFAULT -p pub_qos:=DEFAULT'].
[ERROR] [spawner-11]: process has died [pid 31579, exit code 1, cmd '/opt/ros/humble/lib/controller_manager/spawner scaled_joint_trajectory_controller -c /controller_manager --ros-args -p use_sim_time:=True -p sub_qos:=DEFAULT -p pub_qos:=DEFAULT'].
[goal_initializer_node-15] [WARN] [1759683021.306103874] [goal_init_node]: Waiting for object pose transform to be available in TF,                                       between base_link and goal_frame. if                                       warning persists, check if object pose is detected and                                       published to tf. Message from TF: "goal_frame" passed to lookupTransform argument source_frame does not exist. 
[goal_initializer_node-15] [WARN] [1759683036.016825130] [goal_init_node]: Waiting for object pose transform to be available in TF,                                       between base_link and goal_frame. if                                       warning persists, check if object pose is detected and                                       published to tf. Message from TF: "goal_frame" passed to lookupTransform argument source_frame does not exist. 
[goal_initializer_node-15] [WARN] [1759683072.255025074] [goal_init_node]: Waiting for object pose transform to be available in TF,                                       between base_link and goal_frame. if                                       warning persists, check if object pose is detected and                                       published to tf. Message from TF: "goal_frame" passed to lookupTransform argument source_frame does not exist. 
[goal_initializer_node-15] [WARN] [1759683073.557075644] [goal_init_node]: Waiting for object pose transform to be available in TF,                                       between base_link and goal_frame. if                                       warning persists, check if object pose is detected and                                       published to tf. Message from TF: "goal_frame" passed to lookupTransform argument source_frame does not exist. 
[goal_initializer_node-15] [WARN] [1759683082.990974687] [goal_init_node]: Waiting for object pose transform to be available in TF,                                       between base_link and goal_frame. if                                       warning persists, check if object pose is detected and                                       published to tf. Message from TF: "goal_frame" passed to lookupTransform argument source_frame does not exist. 
[goal_initializer_node-15] [WARN] [1759683088.489634328] [goal_init_node]: Waiting for object pose transform to be available in TF,                                       between base_link and goal_frame. if                                       warning persists, check if object pose is detected and                                       published to tf. Message from TF: "goal_frame" passed to lookupTransform argument source_frame does not exist. 
[ERROR] [component_container_mt-1]: process has died [pid 31559, exit code -9, cmd '/opt/ros/humble/lib/rclcpp_components/component_container_mt --ros-args --log-level error --ros-args -r __node:=manipulator_container -p use_sim_time:=True -p sub_qos:=DEFAULT -p pub_qos:=DEFAULT --params-file /tmp/launch_params_nydf1rbf'].
[ERROR] [attach_object_server_node-6]: process has died [pid 31569, exit code -9, cmd '/opt/ros/humble/lib/isaac_ros_cumotion_object_attachment/attach_object_server_node --ros-args -r __node:=object_attachment -r __ns:=/ -p use_sim_time:=True -p sub_qos:=DEFAULT -p pub_qos:=DEFAULT --params-file /tmp/launch_params_45di_82o'].
[goal_initializer_node-15] [WARN] [1759683109.260482280] [goal_init_node]: Waiting for object pose transform to be available in TF,                                       between base_link and goal_frame. if                                       warning persists, check if object pose is detected and                                       published to tf. Message from TF: "goal_frame" passed to lookupTransform argument source_frame does not exist. 
[goal_initializer_node-15] [WARN] [1759683109.265462800] [goal_init_node]: Waiting for object pose transform to be available in TF,                                       between base_link and goal_frame. if                                       warning persists, check if object pose is detected and                                       published to tf. Message from TF: "goal_frame" passed to lookupTransform argument source_frame does not exist. 
[ERROR] [static_transform_publisher-14]: process has died [pid 31586, exit code -9, cmd '/opt/ros/humble/lib/tf2_ros/static_transform_publisher 0.043 0.359 0.065 0.553 0.475 -0.454 0.513 detected_object1 goal_frame --ros-args -p use_sim_time:=True -p sub_qos:=DEFAULT -p pub_qos:=DEFAULT'].
[ERROR] [robot_state_publisher-2]: process has died [pid 31561, exit code -9, cmd '/opt/ros/humble/lib/robot_state_publisher/robot_state_publisher --ros-args -r __node:=robot_state_publisher -p use_sim_time:=True -p sub_qos:=DEFAULT -p pub_qos:=DEFAULT --params-file /tmp/launch_params_3i5mub2k -r /joint_states:=/isaac_parsed_joint_states'].
[ERROR] [isaac_sim_gripper_action_server.py-12]: process has died [pid 31581, exit code -9, cmd '/opt/ros/humble/lib/isaac_manipulator_pick_and_place/isaac_sim_gripper_action_server.py --ros-args -p use_sim_time:=True -p sub_qos:=DEFAULT -p pub_qos:=DEFAULT --params-file /tmp/launch_params_zlcz7_8b'].
[goal_initializer_node-15] [WARN] [1759683155.859719587] [goal_init_node]: Waiting for object pose transform to be available in TF,                                       between base_link and goal_frame. if                                       warning persists, check if object pose is detected and                                       published to tf. Message from TF: "goal_frame" passed to lookupTransform argument source_frame does not exist. 
[goal_initializer_node-15] [WARN] [1759683166.044522684] [goal_init_node]: Waiting for object pose transform to be available in TF,                                       between base_link and goal_frame. if                                       warning persists, check if object pose is detected and                                       published to tf. Message from TF: "goal_frame" passed to lookupTransform argument source_frame does not exist. 
[ERROR] [ros2_control_node-10]: process has died [pid 31577, exit code -9, cmd '/opt/ros/humble/lib/controller_manager/ros2_control_node --ros-args -p use_sim_time:=True -p sub_qos:=DEFAULT -p pub_qos:=DEFAULT --params-file /opt/ros/humble/share/isaac_manipulator_pick_and_place/config/controllers_sim.yaml --params-file /tmp/launch_params_ca8m68yc -r /controller_manager/robot_description:=/robot_description'].
[goal_initializer_node-15] [WARN] [1759683171.288808734] [goal_init_node]: Waiting for object pose transform to be available in TF,                                       between base_link and goal_frame. if                                       warning persists, check if object pose is detected and                                       published to tf. Message from TF: "goal_frame" passed to lookupTransform argument source_frame does not exist. 
[goal_initializer_node-15] [WARN] [1759683174.953623776] [goal_init_node]: Waiting for object pose transform to be available in TF,                                       between base_link and goal_frame. if                                       warning persists, check if object pose is detected and                                       published to tf. Message from TF: "goal_frame" passed to lookupTransform argument source_frame does not exist. 
[goal_initializer_node-15] [WARN] [1759683176.733728597] [goal_init_node]: Waiting for object pose transform to be available in TF,                                       between base_link and goal_frame. if                                       warning persists, check if object pose is detected and                                       published to tf. Message from TF: "goal_frame" passed to lookupTransform argument source_frame does not exist. 
[goal_initializer_node-15] [WARN] [1759683180.139412541] [goal_init_node]: Waiting for object pose transform to be available in TF,                                       between base_link and goal_frame. if                                       warning persists, check if object pose is detected and                                       published to tf. Message from TF: "goal_frame" passed to lookupTransform argument source_frame does not exist. 
[goal_initializer_node-15] [WARN] [1759683192.237859600] [goal_init_node]: Waiting for object pose transform to be available in TF,                                       between base_link and goal_frame. if                                       warning persists, check if object pose is detected and                                       published to tf. Message from TF: "goal_frame" passed to lookupTransform argument source_frame does not exist. 
[goal_initializer_node-15] [WARN] [1759683207.082160627] [goal_init_node]: Waiting for object pose transform to be available in TF,                                       between base_link and goal_frame. if                                       warning persists, check if object pose is detected and                                       published to tf. Message from TF: "goal_frame" passed to lookupTransform argument source_frame does not exist. 
[ERROR] [cumotion_goal_set_planner_node-4]: process has died [pid 31565, exit code -9, cmd '/opt/ros/humble/lib/isaac_ros_cumotion/cumotion_goal_set_planner_node --ros-args -r __node:=cumotion_planner -r __ns:=/ -p use_sim_time:=True -p sub_qos:=DEFAULT -p pub_qos:=DEFAULT --params-file /tmp/launch_params_xrxqiq0x'].
[goal_initializer_node-15] [WARN] [1759683251.519691951] [goal_init_node]: Waiting for object pose transform to be available in TF,                                       between base_link and goal_frame. if                                       warning persists, check if object pose is detected and                                       published to tf. Message from TF: "goal_frame" passed to lookupTransform argument source_frame does not exist. 
[goal_initializer_node-15] [WARN] [1759683265.921307776] [goal_init_node]: Waiting for object pose transform to be available in TF,                                       between base_link and goal_frame. if                                       warning persists, check if object pose is detected and                                       published to tf. Message from TF: "goal_frame" passed to lookupTransform argument source_frame does not exist. 
[goal_initializer_node-15] [WARN] [1759683270.823380963] [goal_init_node]: Waiting for object pose transform to be available in TF,                                       between base_link and goal_frame. if                                       warning persists, check if object pose is detected and                                       published to tf. Message from TF: "goal_frame" passed to lookupTransform argument source_frame does not exist. 
[goal_initializer_node-15] [WARN] [1759683280.443694064] [goal_init_node]: Waiting for object pose transform to be available in TF,                                       between base_link and goal_frame. if                                       warning persists, check if object pose is detected and                                       published to tf. Message from TF: "goal_frame" passed to lookupTransform argument source_frame does not exist. 
```
