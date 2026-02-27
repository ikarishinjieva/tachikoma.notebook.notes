---
title: 20251008 - 关于Isaac Sim 和 ROS2 demo的文章总结
confluence_page_id: 4358782
created_at: 2025-10-08T07:55:40+00:00
updated_at: 2025-10-10T07:14:56+00:00
---

# 结论

  - [20251010 - 整理Isaac Sim和ROS2 demo的环境安装和运行指令]
    - 完整的安装+使用

# 探索过程, 以及有用的信息

  - nvidia-container-toolkit, 用于在docker容器中, 能正确使用nvidia显卡 ([20250926 - 搭建Issac环境])
  - 本来想使用 Isaac sim 的 headless模式, 在本地使用Client连接显示器. 但一直会报错
    - NVENC (NVIDIA Encoder), 这个功能是用于在显示中进行视频流编码, 用于流传输, 但A100显卡不包括这个功能
    - 更换显卡后, 会一直报连接错误 (这个错误一直卡了7天)
  - 使用了阿里云的 无影云电脑, 可以安装Isaac Sim, 直接显示 (而不是使用headless模式)
    - 使用时, 发现在调用Nvidia高级API时, 发生报错 "operation not supported" ([20251001 - 在Isaac Sim中, 进行机械手的教程])
      - 大模型给出了验证脚本, 脱离了复杂的Isaac Sim环境, 进行了直接API调用验证
      - 主要原因是 无影云的显卡是虚拟机映射, 其中没有映射复杂API
    - 只能退回到自建模式
  - 使用带显卡的笔记本电脑进行尝试, 正确安装Isaam Sim以及ROS2 demo, 但会发生GPU OOM ([20251003 - 重新配置isaac和demo的笔记本])
    - 笔记本的显卡(4060桌面版), 显存确实满足不了文档上的要求
  - 在笔记本上安装Isaac Sim (需要显示器), 在服务器上安装ROS2 demo (需要更大的GPU), 难点是如何将两者连接起来
    - 探索了连接过程 ([20251004 - 在两个网络间，配置Isaac Sim和ROS2 demo, 进行机械手的教程])
      - Isaac Sim和ROS2 之间, 使用了fastdds进行了服务发现. 如何正确进行服务发现, 这里探索了很久, 坎坷的主要原因: 
        - Nvidia内置的fastdds是v2版本, 而大模型都是最新版本v3, 两者配置差很多
        - 其使用了TCP和UDP协议, 刚开始忽略了 ssh -L 端口转发只能转发TCP协议
        - 最终使用SSH TUN通道 + fastdds单播发现服务器 这种结构, 将本地笔记本和VPN服务器连接在一起 ([20251004 - 在两个网络间，配置Isaac Sim和ROS2 demo, 进行机械手的教程], "重新整理所有步骤"章节)
          - 这里有用的经验: 将每部分拆开进行测试
    - 在服务器上安装ROS2 demo后, ROS2 demo会出现内存OOM (而不是显存OOM)
  - 考虑: 还是消除本地笔记本这个变量, 那么就需要在服务器上使用Issac Sim的headless模式, 在服务器上安装Isaac Sim ([20251006 - 尝试将Isaac Sim和WebRTC连通])
    - 在同一个wifi子网内, 测试 Isaac Sim和WebRTC, 是可以连通的
    - 但跨子网的实验均失败 (两边都需要对方的宣告IP, 而NAT满足不了这个架构, 使用一些拦截服务也没办法正常进行)
  - 改为将服务器的显示器整个都映射出来, 也就是不使用Isaac Sim的headless模式, 而是将整个服务器都显示映射
    - 使用NICE DCV ([20251006 - 尝试将Isaac Sim和WebRTC连通])
      - 安装DCV GL 花了很久, Nvidia的GL驱动和Mesa的GL驱动冲突, 将Mesa显卡禁用, 并重装nvidia driver  

  - 将Isaac Sim和ROS2 demo放在同一个服务器上, 仍然会不停OOM ([20251007 - 诊断Isaac Sim和ros2 launch运行时发生OOM])
    - 尝试缩小变量, 包括限制ROS2的任务参数来开启局部, 以及对Isaac Sim的stage中的动态元素进行禁用. 但在ABA测试中, 都会出现OOM, 仿佛是一个不定期触发的机制导致OOM
  - 至此陷入僵局
    - 思考: 核心目标是尝试 机械手控制 + Isaac Sim环境, 那么先放弃部分目标
      - 只测试Isaac Sim环境, 找到其他可测场景 ([20251008 - 列出ROS2样例表, 尝试ROS2样例isaac_ros_apriltag, 解决OOM报错])
      - 只关注机械手控制, 对ROS2的manipulator这个项目启动代码进行理解, 理解了其是由各组件进程 + 设置交换消息的topic, 来进行各组件的交互. 
        - 那么可以尝试将一些组件进行单独的测试, 在 ([20251008 - 列出ROS2样例表, 尝试ROS2样例isaac_ros_apriltag, 解决OOM报错]) 中涉及了部分
  - 突破: ([20251008 - 列出ROS2样例表, 尝试ROS2样例isaac_ros_apriltag, 解决OOM报错])
    - 在使用简单案例时, 也发现了OOM, 通过各种观察工具诊断, 发现了几个问题: 
      - fastdds 网络回环 
      - Isaac Sim缓存
      - 对于错误包, fastdds会OOM, 切换到cyclonedds
      - Isaac Sim ROS2版本需要切换成hamble
    - 解决了报错
