---
title: 20251006 - 尝试将Isaac Sim和WebRTC连通
confluence_page_id: 4358758
created_at: 2025-10-06T17:22:36+00:00
updated_at: 2025-10-07T12:06:59+00:00
---

# 放在同一个wifi子网测试

发现是可以连通的

网络诊断: 

```
┌────[tachikoma@miao]───────[23:40:46]───────[~]─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
└──> sudo tcpdump -i any -w capture_for_analysis.pcap host 192.168.88.214
tcpdump: data link type PKTAP
tcpdump: listening on any, link-type PKTAP (Apple DLT_PKTAP), capture size 262144 bytes
^C1104 packets captured
1567 packets received by filter
0 packets dropped by kernel
┌────[tachikoma@miao]───────[23:40:52]───────[~]─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
└──> tshark -r capture_for_analysis.pcap -q -z conv,tcp
tshark: Error loading table 'TLS Decrypt': ssl_keys:2: File '/Users/tachikoma/Downloads/server.key' does not exist or access is denied.
================================================================================
TCP Conversations
Filter:<No Filter>
                                                           |       <-      | |       ->      | |     Total     |    Relative    |   Duration   |
                                                           | Frames  Bytes | | Frames  Bytes | | Frames  Bytes |      Start     |              |
192.168.88.235:55621       <-> 192.168.88.214:49100            12 839bytes       18 19kB           30 20kB          0.626113000         2.4481
================================================================================
┌────[tachikoma@miao]───────[23:41:04]───────[~]─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
└──> tshark -r capture_for_analysis.pcap -q -z conv,udp
tshark: Error loading table 'TLS Decrypt': ssl_keys:2: File '/Users/tachikoma/Downloads/server.key' does not exist or access is denied.
================================================================================
UDP Conversations
Filter:<No Filter>
                                                           |       <-      | |       ->      | |     Total     |    Relative    |   Duration   |
                                                           | Frames  Bytes | | Frames  Bytes | | Frames  Bytes |      Start     |              |
192.168.88.235:61785       <-> 192.168.88.214:47998           990 229kB          84 8,446bytes    1074 237kB         0.000000000         4.0765
================================================================================
``` 

# 将Isaac Sim运行在公司服务器上, 通过VPN连入

WebRTC client连接后, 在Isaac Sim端报错:

```
2025-10-06T16:02:39Z [51,214ms] [Info] [carb.livestream-rtc.plugin] Stream server: connected stream 0x62ec65bffbc0 on connection 0x74fd6c021580
2025-10-06T16:02:39Z [51,252ms] [Warning] [gpu.foundation.plugin] Invalid sync scope for buffer resource 'shared swapchain buffer'. Create resource with valid sync scope for lifetime tracking or use kResourceUsageFlagNoSyncScope.
2025-10-06T16:02:39Z [51,253ms] [Warning] [gpu.foundation.plugin] Invalid sync scope for buffer resource 'shared swapchain buffer'. Create resource with valid sync scope for lifetime tracking or use kResourceUsageFlagNoSyncScope.
2025-10-06T16:02:39Z [51,253ms] [Warning] [gpu.foundation.plugin] Invalid sync scope for buffer resource 'shared swapchain buffer'. Create resource with valid sync scope for lifetime tracking or use kResourceUsageFlagNoSyncScope.
2025-10-06T16:02:40Z [52,368ms] [Warning] [carb.livestream-rtc.plugin] onClientEventRaised: NVST_CCE_DISCONNECTED when m_connectionCount 0 != 1
2025-10-06T16:02:40Z [52,542ms] [Info] [carb.livestream-rtc.plugin] Stream Server: stream0 0x74fd6c021580 (type 1) stopped
2025-10-06T16:02:40Z [52,542ms] [Info] [carb.livestream-rtc.plugin] Stream Server: stream1 0x74fd6c065f40 (type 4) stopped
2025-10-06T16:02:40Z [52,542ms] [Info] [carb.livestream-rtc.plugin] Stream Server: stream2 0x74fd6c04e5a0 (type 2) stopped
2025-10-06T16:02:40Z [52,542ms] [Warning] [carb.livestream-rtc.plugin] onClientEventRaised: NVST_CCE_DISCONNECTED when m_connectionCount 4294967295 != 1
``` 

抓包统计: 

```
┌────[tachikoma@miao]───────[00:04:09]───────[~]─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
└──> sudo tcpdump -i any -w capture_for_analysis.pcap host 10.186.16.135
Password:
tcpdump: data link type PKTAP
tcpdump: listening on any, link-type PKTAP (Apple DLT_PKTAP), capture size 262144 bytes
^C201 packets captured
1742 packets received by filter
0 packets dropped by kernel
┌────[tachikoma@miao]───────[00:04:45]───────[~]─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
└──> tshark -r capture_for_analysis.pcap -q -z conv,udp
tshark: Error loading table 'TLS Decrypt': ssl_keys:2: File '/Users/tachikoma/Downloads/server.key' does not exist or access is denied.
================================================================================
UDP Conversations
Filter:<No Filter>
                                                           |       <-      | |       ->      | |     Total     |    Relative    |   Duration   |
                                                           | Frames  Bytes | | Frames  Bytes | | Frames  Bytes |      Start     |              |
172.16.169.1:55806         <-> 10.186.16.135:47998             27 3,376bytes      30 4,204bytes      57 7,580bytes     0.640770000         0.8031
192.168.88.235:61003       <-> 10.186.16.135:47998              0 0bytes          3 414bytes        3 414bytes      0.327819000         0.8745
================================================================================
┌────[tachikoma@miao]───────[00:04:58]───────[~]─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
└──> tshark -r capture_for_analysis.pcap -q -z conv,tcp
tshark: Error loading table 'TLS Decrypt': ssl_keys:2: File '/Users/tachikoma/Downloads/server.key' does not exist or access is denied.
================================================================================
TCP Conversations
Filter:<No Filter>
                                                           |       <-      | |       ->      | |     Total     |    Relative    |   Duration   |
                                                           | Frames  Bytes | | Frames  Bytes | | Frames  Bytes |      Start     |              |
172.16.169.1:58552         <-> 10.186.16.135:49100             55 45kB           54 8,955bytes     109 54kB          0.000000000         0.7614
172.16.169.1:58272         <-> 10.186.16.135:22                 9 1,924bytes       8 448bytes       17 2,372bytes     0.082731000         1.4884
172.16.169.1:58553         <-> 10.186.16.135:49100              5 512bytes        7 2,994bytes      12 3,506bytes     0.730800000         0.7131
================================================================================
┌────[tachikoma@miao]───────[00:05:06]───────[~]─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
└──>

``` 

使用isaac sim参数: `--/app/livestream/publicEndpointAddress=10.186.16.135 --/app/livestream/port=49100` 限制isaac sim的宣告IP. 不能解决问题. 

(出问题的是webrtc的宣告IP)

使用ssh tun + 路由配置, 仍然无法解决问题

```
# Mac
sudo ssh -vv -w 0:0 root@10.186.16.135
sudo ifconfig tun0 10.10.10.1 10.10.10.2 up

# Linux
sudo ip addr add 10.10.10.2/30 peer 10.10.10.1 dev tun0
sudo ip link set tun0 up

# Mac
sudo route add -host 10.186.16.135 192.168.88.2
sudo route delete default
sudo route add default 10.10.10.2

# Linux
sudo iptables -t nat -A INPUT -i tun0 -s 10.10.10.1 -j SNAT --to-source 127.0.0.1

``` 

# 尝试使用NICE DCV

在Ubuntu server, 没有物理显示器的情况下, 使用NICE DCV , 实现桌面远程显示

有两种模式: 

  - 控制台会话：依赖正在运行且配置正确的本机 X 服务器。无物理显示器时通常需要安装 XDummy 虚拟显卡驱动来让 X 服务器工作。
  - 虚拟会话：不依赖系统的 X 服务器（除非你启用 DCV GL 做 GPU 共享），适合无显示器环境，部署更简单。

先尝试 "虚拟会话（Virtual session）+ DCV GL（GPU 共享）":

```
# 安装server: 
 
sudo snap set system proxy.http="http://10.186.16.135:7890"
sudo snap set system proxy.https="http://10.186.16.135:7890"
sudo apt install ubuntu-desktop
sudo apt install gdm3
 
验证默认显示管理器：cat /etc/X11/default-display-manager
期望输出：/usr/sbin/gdm3
如非 gdm3：sudo dpkg-reconfigure gdm3

 
#重启完成桌面初始化
reboot -f 
 
wget https://d1uj6qtbmh3dt5.cloudfront.net/NICE-GPG-KEY
gpg --import NICE-GPG-KEY
 
 
wget https://d1uj6qtbmh3dt5.cloudfront.net/2024.0/Servers/nice-dcv-2024.0-19030-ubuntu2404-x86_64.tgz
tar -xvzf nice-dcv-2024.0-19030-ubuntu2404-x86_64.tgz && cd nice-dcv-2024.0-19030-ubuntu2404-x86_64
sudo apt install ./nice-dcv-server_2024.0.19030-1_amd64.ubuntu2404.deb
sudo apt install ./nice-dcv-web-viewer_2024.0.19030-1_amd64.ubuntu2404.deb
sudo apt install ./nice-xdcv_2024.0.654-1_amd64.ubuntu2404.deb
 
sudo systemctl start dcvserver && sudo systemctl enable dcvserver
sudo systemctl status dcvserver
 
# 检查端口
sudo ss -lntp | grep 8443
``` 

创建会话: 

```
dcv create-session mysession
dcv list-sessions
 
访问 https://10.186.16.136:8443/, 输入操作系统的用户名密码, 可以登录桌面
``` 

根据<https://docs.amazonaws.cn/dcv/latest/userguide/client-mac.html>, 安装客户端, 可连接服务器

另外安装DCV GL: 

```
 sudo apt install ./nice-dcv-gl_2024.0.1096-1_amd64.ubuntu2404.deb
sudo apt install mesa-utils
检查状态: sudo dcvgladmin status
 
 
 sudo nvidia-xconfig --preserve-busid --enable-all-gpus
sudo systemctl isolate multi-user.target
sudo systemctl isolate graphical.target
 
连接到服务器会话, 在会话内运行: DISPLAY=:0 glxinfo | grep -i "opengl.*version"
	期望输出含 “NVIDIA” 字样，例如：OpenGL version string: 4.x.y NVIDIA nnn.nn
	如果看到 “Mesa” 而不是 NVIDIA，说明会话里没有走硬件加速
``` 

如果试用了Mesa, 通过禁用内置显卡来修复: 

```
echo "blacklist mgag200" | sudo tee /etc/modprobe.d/blacklist-matrox.conf
sudo update-initramfs -u
sudo reboot -f 
``` 

样例输出: 

```
huangyan@dell-136:~$ DISPLAY=:0 XAUTHORITY=$(ps aux | grep "X.*\-auth" | grep -v Xdcv | grep -v grep | sed -n 's/.*-auth \([^ ]\+\).*/\1/p') glxinfo | grep -i "opengl.*version"
OpenGL core profile version string: 4.6.0 NVIDIA 565.57.01
OpenGL core profile shading language version string: 4.60 NVIDIA
OpenGL version string: 4.6.0 NVIDIA 565.57.01
OpenGL shading language version string: 4.60 NVIDIA
OpenGL ES profile version string: OpenGL ES 3.2 NVIDIA 565.57.01
OpenGL ES profile shading language version string: OpenGL ES GLSL ES 3.20
OpenGL core profile version string: 4.6.0 NVIDIA 565.57.01
OpenGL core profile shading language version string: 4.60 NVIDIA
OpenGL version string: 4.6.0 NVIDIA 565.57.01
OpenGL shading language version string: 4.60 NVIDIA
OpenGL ES profile version string: OpenGL ES 3.2 NVIDIA 565.57.01
OpenGL ES profile shading language version string: OpenGL ES GLSL ES 3.20

``` 

使用NICE DCV 可以将Isaac Sim和ros2 launch放在一台服务器上运行, 启动过程: 

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
./isaac-sim.sh

打开 https://omniverse-content-production.s3-us-west-2.amazonaws.com/Assets/Isaac/4.2/Isaac/Samples/ROS2/Scenario/isaac_manipulator_ur10e_robotiq_2f_140.usd
开始Play

桌面窗口2: 
export ISAAC_ROS_WS=/data/huangyan/isaac_ros-dev/
cd $ISAAC_ROS_WS && ./src/isaac_ros_common/scripts/run_dev.sh 进容器

容器内
export http_proxy=http://10.186.16.136:7890
export https_proxy=http://10.186.16.136:7890
export FASTRTPS_DEFAULT_PROFILES_FILE=/usr/local/share/middleware_profiles/rtps_udp_profile.xml
cd ${ISAAC_ROS_WS}
source install/setup.bash
ros2 launch isaac_manipulator_pick_and_place isaac_sim_workflows.launch.py \
   workflow_type:=object_following pose_estimation_type:=foundationpose
```
