---
title: 20251204 - 使用uhand操作移液枪
confluence_page_id: 4359390
created_at: 2025-12-04T15:51:58+00:00
updated_at: 2025-12-09T07:43:49+00:00
---

# 碰到的问题

五舵机的机械手, 每个手指的轨迹是固定的, 导致无法碰触到移液枪的按钮

我在进行机械手的实验, 发现一些问题, 帮我找一些解决方案:

我想用机械手去抓取移液枪, 然后按下移液枪的按钮, 进行移液.  
现在的问题是, 使用的机械手使用了五个舵机分别控制了五个手指, 所以大拇指的轨迹是固定的, 而移液枪的按钮需要大拇指去按, 但不在这个轨迹上.

我想了一些办法:

  1. 使用六自由度的五指机械手, 但这种灵巧手价格太高
  2. 使用三指灵巧手, 但价格也不低
  3. 改造移液枪的按钮, 比如在上面贴上一个压片, 让拇指可以压到压片一段, 但这样按钮受力不平衡

你帮我想想办法

# 思路列表

  - 改造成气动泵: <https://pubs.rsc.org/en/content/articlehtml/2023/dd/d3dd00115f>
  - 扩大移液枪的按钮的大小 (但离大拇指的轨迹偏差有点大, 受力不均匀, 按压就会不顺畅)
  - 六自由度的五指灵巧手
    - ![image2025-12-4 22:22:47.png](/assets/01KJC09552D537B0DQ1NXPDZRF/image2025-12-4%2022%3A22%3A47.png)
  - 三指灵巧手: 
    - ![image2025-12-4 22:32:27.png](/assets/01KJC09552D537B0DQ1NXPDZRF/image2025-12-4%2022%3A32%3A27.png)

  - 较轻的仿生手: <https://www.brainco.cn/#/product/revo2>
    - 也是6主动自由度, 我理解比上面的灵巧手的区别就是轻一点, 好看
  - 类似的6主动自由度: 
    - <https://www.hitbot.cc/ehand-6/>
    - <https://linkerbot.cn/product?page=O6>
      - 便宜
  - 真仿生手: 12主动自由度
    - <https://dexrobot.feishu.cn/docx/YyEddjwSQovgCvxnhFQcgipInjc>

买了Linker Hand O6, 重量300g+, 可能能在现在的机械臂上使用 (负重有限)

# Linker Hand O6

O6的URDF: <https://github.com/linker-bot/linkerhand-urdf/tree/main/o6/right>

URDF导入的效果: 

![image2025-12-5 17:46:34.png](/assets/01KJC09552D537B0DQ1NXPDZRF/image2025-12-5%2017%3A46%3A34.png)

将O6导入到场景中, 这次使用不同的做法: 

  - 将机械臂和机械手掌作为两个组件导入isaac sim, 将机械手掌作为子Prim挂在机械臂的末端link下, 进行测试
    - 仿真时会报错: 形成了刚体嵌套: 

```
2025-12-05 16:49:22 [25,577,789ms] [Error] [omni.physicsschema.plugin] Rigid Body of (/World/firefighter/joint6_flange/linkerhand_o6_right/hand_base_link) missing xformstack reset when child of rigid body (/World/firefighter/joint6_flange) in hierarchy. Simulation of multiple RigidBodyAPI's in a hierarchy will cause unpredicted results. Please fix the hierarchy or use XformStack reset.
2025-12-05 16:49:22 [25,577,791ms] [Error] [omni.physicsschema.plugin] Rigid Body of (/World/firefighter/joint6_flange/linkerhand_o6_right/index_proximal) missing xformstack reset when child of rigid body (/World/firefighter/joint6_flange) in hierarchy. Simulation of multiple RigidBodyAPI's in a hierarchy will cause unpredicted results. Please fix the hierarchy or use XformStack reset.
```

  - 更换方法, 在URDF xacro中, 将两个组件分别include, 然后用一个fix joint连接. 然后用xacro将合并后的URDF展开: 

```
ros2 run xacro xacro   /workspaces/isaac_ros-dev/huangyan/moveit_and_isaac_sim/config/isaac_sim/mycobot_280_isaac_sim.urdf.xacro   initial_positions_file:=/workspaces/isaac_ros-dev/huangyan/moveit_and_isaac_sim/config/isaac_sim/mycobot_280_isaac_sim.initial_positions.yaml   urdf_dir:=/data/huangyan/isaac_ros-dev/huangyan/moveit_and_isaac_sim/config/urdf   > mycobot_280_isaac_sim_linker_o6.urdf
```

    - 在isaac sim中, 导入合并后的URDF, 仿真时机械臂会崩溃. 
      - 经过分析, 机械手掌带有惯量配置 (inertial), 但手臂没有惯量配置, 所以计算错误. 修复后正常. 
      - commit: 49c1d305d6db37f1de5b4de4ce7918366ab707f6
    - 在场景中
      - 将articulation root 从root_joint移到机械臂上
      - 添加 ros2 jointstates 和 ros2 clock, 开启仿真, 测试状态: 

```
admin@10-60-141-108:/workspaces/isaac_ros-dev/huangyan/moveit_and_isaac_sim$ ros2 topic echo /joint_states --once
1764992236.844378 [99]       ros2: config: //CycloneDDS/Domain/General: 'NetworkInterfaceAddress': deprecated element (/workspaces/isaac_ros-dev/a.xml line 9)
header:
  stamp:
    sec: 1239
    nanosec: 400064639
  frame_id: ''
name:
- joint2_to_joint1
- joint3_to_joint2
- joint4_to_joint3
- joint5_to_joint4
- joint6_to_joint5
- joint6output_to_joint6
- index_mcp_pitch
- middle_mcp_pitch
- pinky_mcp_pitch
- ring_mcp_pitch
- thumb_cmc_yaw
- index_dip
- middle_dip
- pinky_dip
- ring_dip
- thumb_cmc_pitch
- thumb_ip
position:
- 0.0
- 0.0
- 0.0
- 0.0
- 0.0
- 0.0
- -0.0002
- 0.0
- 0.0
- 0.0
- 0.0
- 0.0
- 0.0
- 0.0
- -0.0001
- 0.0
- 0.0001
velocity:
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
effort:
- 0.0
- 0.1523
- 0.1523
- 0.1523
- 0.0
- -0.0066
- 0.0
- 0.0
- 0.0
- 0.0
- 0.0048
- 0.0
- 0.0
- 0.0
- 0.0
- 0.0028
- 0.0004
---
```

    - 让AI进行ros2相关文件修改, 方案为: "新机械手修改方案.md"
    - 运行moveit: 

```
ros2 launch huangyan/moveit_and_isaac_sim/launch/mycobot_280_isaac_sim_moveit.launch.py
```

      - 可以看到机械臂和机械手臂, 并能进行路线规划

# 解决路线规划的崩溃

之前增加了机械臂的惯量, 让机械臂和机械掌在开始仿真时不再崩溃

现在的问题是: 在某些路线规划时, 会发生崩溃. 但过程很快, 很难诊断原因.

增加一个 run_isaac_sim_with_monitor.py 脚本, 启动isaac sim, 并在关节异常时, 能暂停isaac sim

  - 命令: /data/huangyan/isaac-sim-4-2/python.sh run_isaac_sim_with_monitor.py
  - 脚本启动有一定概率崩溃, 目前无解, 多启动几次就可以运行

第一次崩溃: 

```
2025-12-06 14:33:42 [72,832ms] [Warning] [__main__] [MonitorUSD] Abnormal joint velocity detected at step 0.01666666753590107. Threshold = 50.0 rad/s.
2025-12-06 14:33:42 [72,832ms] [Warning] [__main__]   -> Joint [8] 'pinky_mcp_pitch': vel = -125.9622 rad/s
2025-12-06 14:33:42 [72,843ms] [Warning] [__main__] [MonitorUSD] World paused due to abnormal joint velocity
``` 

![image2025-12-6 22:43:29.png](/assets/01KJC09552D537B0DQ1NXPDZRF/image2025-12-6%2022%3A43%3A29.png)

第二次崩溃: 

```
1765035053.892513 [99] pt_main_th: config: //CycloneDDS/Domain/General: 'NetworkInterfaceAddress': deprecated element (/data/huangyan/isaac_ros-dev/a.xml line 9)
2025-12-06 15:31:41 [63,241ms] [Warning] [__main__] [MonitorUSD] Abnormal joint velocity detected at step 0.01666666753590107. Threshold = 50.0 rad/s.
2025-12-06 15:31:41 [63,242ms] [Warning] [__main__]   -> Joint [7] 'middle_mcp_pitch': vel = -56.4761 rad/s
2025-12-06 15:31:41 [63,242ms] [Warning] [__main__]   -> Joint [8] 'pinky_mcp_pitch': vel = -167.2476 rad/s
``` 

![image2025-12-6 23:32:21.png](/assets/01KJC09552D537B0DQ1NXPDZRF/image2025-12-6%2023%3A32%3A21.png)

缩小范围, 修改以下文件, 将joint逐个放出: 

  - linkerhand_o6_right.urdf.xacro
  - mycobot_280_isaac_sim.srdf
  - mycobot_280_isaac_sim.ros2_controllers.yaml
  - mycobot_280_isaac_sim.ros2_control.xacro
  - mycobot_280_isaac_sim.moveit_controllers.yaml
  - mycobot_280_isaac_sim.initial_positions.yaml

只开启手掌, 移动都没问题: 

![image2025-12-7 14:21:7.png](/assets/01KJC09552D537B0DQ1NXPDZRF/image2025-12-7%2014%3A21%3A7.png)

开启大拇指根部, 也没有问题: 

![image2025-12-7 14:28:45.png](/assets/01KJC09552D537B0DQ1NXPDZRF/image2025-12-7%2014%3A28%3A45.png)

增加大拇指尖部, 则仿真会崩溃: 

![image2025-12-7 14:31:45.png](/assets/01KJC09552D537B0DQ1NXPDZRF/image2025-12-7%2014%3A31%3A45.png)

通过脚本停留在崩溃位置: 

![image2025-12-7 15:1:11.png](/assets/01KJC09552D537B0DQ1NXPDZRF/image2025-12-7%2015%3A1%3A11.png) 

隐藏掉手掌, 拇指会向内扣到与手掌内部相碰: 

![image2025-12-7 15:1:53.png](/assets/01KJC09552D537B0DQ1NXPDZRF/image2025-12-7%2015%3A1%3A53.png)

将大拇指尖部的mimic属性去掉, 不再崩溃 (经过ABA测试, 确实是mimic属性导致)

去掉mimic时, 关节thumb_ip的属性: 

![image2025-12-7 17:46:34.png](/assets/01KJC09552D537B0DQ1NXPDZRF/image2025-12-7%2017%3A46%3A34.png)![image2025-12-7 17:46:40.png](/assets/01KJC09552D537B0DQ1NXPDZRF/image2025-12-7%2017%3A46%3A40.png)

加上mimic时, 属性对比: 

![image2025-12-7 17:48:11.png](/assets/01KJC09552D537B0DQ1NXPDZRF/image2025-12-7%2017%3A48%3A11.png)![image2025-12-7 17:48:15.png](/assets/01KJC09552D537B0DQ1NXPDZRF/image2025-12-7%2017%3A48%3A15.png)![image2025-12-7 17:48:19.png](/assets/01KJC09552D537B0DQ1NXPDZRF/image2025-12-7%2017%3A48%3A19.png)

mimic joint的gearing是 -1.86, 与URDF中的定义不符. 猜测是 isaac sim的推断错误. 将其改为 +1.86后, 崩溃现象也消失: 

在 Isaac Sim 的几个版本里（2022/2023/2024），NVIDIA 自己的文档和论坛都提到过：

  - 对 URDF 的 `` 支持是“部分支持”，
  - 对非平凡的关节布局（比如有 `rpy` 旋转、手指这类小关节链）时，  
自动推断 mimic 的 axis / 符号有时会出错，需要**在导入后自己看一眼 / 手动修** 。

加入其他指头, 修正mimic关节的gearing, 恢复正常.

  - commit: bd4862aea46f26f2caa61e8c4eb9ab2cca3c9f03

用moveit调度手指, 发现除了大拇指横向关节, 其他手指的纵向关节都不移动

  - 经过诊断, 仍然是mimic标签, isaac sim这个版本不能正常驱动.
  - 准备去掉mimic空间, 将各关节关系梳理如下, 之后用单独的controller (isaac sim专属) 来维护
    - thumb_cmc_pitch -> thumb_ip , 倍数: 1.86
    - index_mcp_pitch -> index_dip , 倍数: 0.89
    - middle_mcp_pitch -> middle_dip , 倍数: 0.89
    - ring_mcp_pitch -> ring_dip , 倍数: 0.89
    - pinky_mcp_pitch -> pinky_dip , 倍数: 0.89

TODO: 继续调试仿真环境

# 接入物理手掌

插入CAN 转 USB 接口, 系统日志: 

```
[Mon Dec  8 09:25:44 2025] usb 1-4: new full-speed USB device number 33 using xhci_hcd
[Mon Dec  8 09:25:44 2025] usb 1-4: New USB device found, idVendor=0c72, idProduct=000c, bcdDevice=54.ff
[Mon Dec  8 09:25:44 2025] usb 1-4: New USB device strings: Mfr=10, Product=4, SerialNumber=0
[Mon Dec  8 09:25:44 2025] usb 1-4: Product: XCAN-USB
[Mon Dec  8 09:25:44 2025] usb 1-4: Manufacturer:
[Mon Dec  8 09:25:44 2025] CAN device driver interface
[Mon Dec  8 09:25:44 2025] peak_usb 1-4:1.0: PEAK-System PCAN-USB adapter hwrev 84 serial FFFFFFFF (1 channel)
[Mon Dec  8 09:25:44 2025] peak_usb 1-4:1.0 can0: attached to PCAN-USB channel 0 (device 255)
[Mon Dec  8 09:25:44 2025] usbcore: registered new interface driver peak_usb
[Mon Dec  8 09:26:28 2025] rtw_8821ce 0000:05:00.0: firmware failed to leave lps state
[Mon Dec  8 09:26:30 2025] rtw_8821ce 0000:05:00.0: firmware failed to leave lps state
[Mon Dec  8 09:26:31 2025] ucsi_acpi USBC000:00: ucsi_handle_connector_change: ACK failed (-110)
[Mon Dec  8 09:26:41 2025] ucsi_acpi USBC000:00: ucsi_handle_connector_change: ACK failed (-110)
[Mon Dec  8 09:27:20 2025] rtw_8821ce 0000:05:00.0: firmware failed to leave lps state
[Mon Dec  8 09:30:20 2025] rtw_8821ce 0000:05:00.0: firmware failed to leave lps state
``` 

查看设备:

```
root@bot:/home/huangyan# ip -details link show type can
8: can0: <NOARP,ECHO> mtu 16 qdisc noop state DOWN mode DEFAULT group default qlen 10
    link/can  promiscuity 0 minmtu 0 maxmtu 0
    can state STOPPED (berr-counter tx 0 rx 0) restart-ms 0
	  pcan_usb: tseg1 1..16 tseg2 1..8 sjw 1..4 brp 1..64 brp-inc 1
	  clock 8000000 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 parentbus usb parentdev 1-4:1.0
``` 

开启can0: 

```
sudo ip link set can0 up type can bitrate 1000000
 
查看状态:  (ERROR-ACTIVE 不是有错误的意思) 
admin@bot:/workspaces/isaac_ros-dev$ ip -details link show type can
10: can0: <NOARP,UP,LOWER_UP,ECHO> mtu 16 qdisc pfifo_fast state UP mode DEFAULT group default qlen 10
    link/can  promiscuity 0 minmtu 0 maxmtu 0
    can state ERROR-ACTIVE (berr-counter tx 0 rx 0) restart-ms 0
	  bitrate 1000000 sample-point 0.750
	  tq 125 prop-seg 2 phase-seg1 3 phase-seg2 2 sjw 1
	  pcan_usb: tseg1 1..16 tseg2 1..8 sjw 1..4 brp 1..64 brp-inc 1
	  clock 8000000 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 parentbus usb parentdev 1-4:1.0
``` 

安装python-can: 

```
pip install python-can -i https://pypi.tuna.tsinghua.edu.cn/simple
 
 
huangyan@bot:~$ python3 -c "import can; print('✓ python-can 已安装')"
✓ python-can 已安装
``` 

测试: 

```
窗口1: cansend can0 027#01
 
窗口2: 监听candump: 
 
admin@bot:/workspaces/isaac_ros-dev$ candump -x can0
  can0  TX - -  027   [1]  01
  can0  RX - -  027   [7]  01 95 96 96 96 96 95
 
进行动作: 
cansend can0 027#01969696969696
 
动作复原: 
cansend can0 027#01FAFAFAFAFAFA
```
