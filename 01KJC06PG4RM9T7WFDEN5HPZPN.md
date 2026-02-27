---
title: 20251023 - 在仿真环境中, 完成一次倒水的动作
confluence_page_id: 4358975
created_at: 2025-10-22T16:23:32+00:00
updated_at: 2025-11-04T08:46:09+00:00
---

# 背景信息

用手工模拟的意义: 

你完全正确，手动设定所有路径点确实看起来效率不高，而且似乎违背了我们让AI“自主学习”的初衷。

让我来详细解释一下为什么这个“看似低效”的阶段不仅有意义，而且是**极其关键和必要** 的。我们并不是在否定“试探和演练”的价值，而是在为它做最重要的准备工作。

* * *

### **为什么“导演整个动作”是必要的第一步？**

简单来说，我们这么做主要有四个战略目的：

#### **1\. 解决“有没有”的问题，再谈“好不好”的问题 (Feasibility Check)**

在你让AI开始“盲目”试探之前，你必须首先确认这个任务在你的设定下**是否物理上可行** 。

  - **可达性（Reachability）** ：机械臂的臂长够不够抓到A杯子，再移动到B杯子上方？它的关节会不会在某个姿态下达到极限或者发生自碰撞？
  - **物理稳定性（Physics Stability）** ：你选择的杯子模型，在被夹爪抓住时，物理效果是否稳定？会不会因为重心或摩擦力设置不当而滑落、抖动？液体模拟在倾倒时是否符合预期，还是会直接“爆炸”或穿透？
  - **基础逻辑验证** ：倒水这个动作的基本逻辑——抓取、抬起、移动、倾斜、收回——是否正确？

**手动拖拽和编写脚本来走一遍流程，是回答上述所有问题的最快、最直接、成本最低的方法。** 如果一个经验丰富的人类都无法通过“导演”的方式让机器人完成任务，那么让一个从零开始的AI去探索，其难度将是天文数字。这个阶段就像是为电影拍摄进行“勘景”和“剧本围读”，确保正式开拍时不会遇到颠覆性的问题。

#### **2\. 为模仿学习（Imitation Learning）提供高质量的“专家数据”**

这直接关系到你提到的“让脚本进行不停试探和演练”。AI学习主要有两种范式：

  - **强化学习（Reinforcement Learning, RL）** ：让AI像个婴儿一样，通过海量的、随机的试错去学习。它会自己探索，做对了给奖励，做错了给惩罚。
  - **模仿学习（Imitation Learning, IL）** ：让AI像个学徒，通过观察“专家”（也就是我们手动导演的完美动作）是如何做的，来模仿学习。

**对于“倒水”这种复杂、精细且连续的动作，直接使用强化学习（RL）的效率极低。** 想象一下，在一个三维空间里，让机械臂随机摆动，它需要尝试多少亿次，才能“碰巧”完成一次完美的倒水动作并获得奖励？这个奖励信号太稀疏了（Sparse Reward Problem），AI几乎学不到任何东西。

**因此，在机器人领域，一个非常主流且高效的方法是：先模仿，再强化。**

  - **第一步（我们正在做的）** ：我们作为“专家”，手动创建一个或多个完美的“倒水”演示（Demonstration）。在演示过程中，我们记录下每时每刻机器人的状态（关节角度、速度）和它执行的动作（关节力矩或目标姿态）。这就是**“专家数据”**。
  - **第二步** ：使用模仿学习算法（如 Behavior Cloning），让AI模型（一个神经网络）学习这个“状态-动作”的映射关系。学完后，AI就得到了一个初始策略，它已经知道“在什么情况下，大概应该怎么动”。
  - **第三步（可选但常用）** ：在这个已经不错的初始策略基础上，再使用强化学习进行微调（Fine-tuning），让它在更多样的、未见过的场景下（比如杯子位置稍微变动）做得更好、更鲁棒。

所以，我们手动调试路径的“低效”工作，恰恰是在为AI提供**最高效的学习材料** 。没有这份“教材”，AI连门都入不了。

#### **3\. 作为强化学习（RL）的“奖励函数”设计基准**

即便我们决定直接使用强化学习，我们手动创建的完美轨迹也极其有用。它可以用来设计一个密集的奖励函数（Dense Reward Function），以解决前面提到的奖励稀疏问题。

例如，奖励函数可以这样设计：  
`Reward = - (距离A杯子的距离) - (距离B杯子上方的距离) - (当前姿态与专家轨迹对应姿态的差异) + (成功倒入水的量)`

你看，**“与专家轨迹的差异”** 这一项，极大地指导了AI的探索方向。它每时每刻都知道自己离“正确答案”有多远，学习效率因此大大提升。

#### **4\. 调试和验证仿真环境的真实性 (Sim-to-Real Gap)**

仿真环境终究是“假”的。液体可能不完全符合流体力学，摩擦力可能和真实世界有偏差。

当我们执行一个我们认为是“完美”的动作序列时，如果仿真结果很糟糕（比如水花四溅），这就给了我们一个信号：**问题可能出在仿真环境的参数上，而不是我们的动作逻辑上。** 这时，我们就可以去调整液体的粘度、表面的摩擦系数等，让仿真环境更接近真实世界。

这个过程对于最终将算法部署到真实机器人上（Sim-to-Real）至关重要。如果在一个错误的仿真环境中训练AI，那么训练出的策略在真实世界中几乎必然会失败。

* * *

### **总结：两种路径的对比**

  - **你最初设想的路径（纯探索）** ：

    1. 搭建环境。
    2. 让AI脚本从零开始，在环境中海量试错。
    3. 结果：由于任务复杂、奖励稀疏，AI可能花费几天甚至几周都无法学会倒水，探索过程极其低效，甚至失败。
  - **我建议的路径（先导演，后学习）** ：

    1. **第一阶段：导演与数据采集**
       - 搭建环境。
       - 手动调试出一条完美的“专家轨迹”。（**你现在所处的阶段** ）
       - 验证了任务可行性，并调试了仿真环境。
       - 记录下这条轨迹，生成高质量的专家数据集。
    2. **第二阶段：模仿学习**
       - 使用专家数据集训练一个AI模型。
       - 结果：AI很快学会了一个相当不错的倒水策略。
    3. **第三阶段：强化与泛化**
       - （可选）在模仿学习的基础上，用强化学习进行微调，让AI应对更多变化，变得更强。

**结论：** 我们现在做的“手动导演”，不是最终目的，而是整个AI训练流程中一个**不可或缺的、具有战略意义的奠基步骤** 。它看似“低效”，但从整个项目的角度看，它是保证AI能够**高效、可靠地** 学会复杂技能的关键所在。

你的直觉——让AI自主探索——是最终目标。但我们现在做的是为AI铺设一条能够最快到达那个目标的跑道，而不是让它在荒野中从零开始。

手工模拟的方案: 

我们来聚焦第一阶段，并按照你简化的想法（用普通杯子倒水）来制定一个详细、可执行的学习和实践步骤。

这个阶段的目标是：**在Isaac Sim仿真环境中，利用cuMotion作为运动规划工具，实现机械臂从一个杯子抓水倒入另一个杯子的完整、平滑、可靠的流程。**

这个过程将让你成为这个特定任务的“领域专家”，为你后续无论是部署到真实世界，还是训练AI模型都打下坚实的基础。

* * *

### **第一阶段学习路径：在仿真中成为“倒水专家”**

我们将这个阶段分解为6个关键步骤。请按顺序进行，确保掌握了前一步再进入下一步。

#### **第0步：准备工作与基础知识回顾**

在开始之前，确保你已经准备好并熟悉以下内容。这是地基，地基不牢，后面会很痛苦。

  1. **环境安装** ：确保你的Isaac Sim 2023.1.x 版本已正确安装并能正常运行。
  2. **基础操作复习** ：
     - **UI操作** ：熟悉如何在Isaac Sim中创建、移动、旋转物体（`Prim`）。
     - **Python脚本基础** ：回顾一下如何通过Python脚本与Isaac Sim交互，特别是如何获取`Stage`，如何生成`Prim`，以及如何运行仿真。
     - **独立Python脚本控制** ：复习或学习如何从一个外部的、独立的Python脚本通过`omni.kit.app.get_app_interface().update()`循环来控制仿真。这是推荐的工作流，比在内置的Script Editor里写全部代码更灵活。
  3. **核心工具学习** ：
     - **找到并运行官方示例** ：Isaac Sim自带了大量示例。你的首要任务是找到并成功运行 **`Franka` \+ `cuMotion`** 的相关示例。通常在 `Window -> Demo` 或 `Isaac Examples` 菜单下可以找到。
     - **理解示例代码** ：仔细阅读这些示例的Python代码。理解它是如何初始化cuMotion、设置目标姿态（`target_pose`）、以及执行运动的。这是你接下来所有工作的基础。

#### **第1步：搭建仿真场景 (Scene Setup)**

这是将你的想法可视化的第一步。

  1. **创建基础环境** ：
     - 创建一个`GroundPlane`（地面）或一个简单的立方体作为桌子。
     - 从Isaac Sim的资产库（`NVIDIA/Assets/Isaac/2023.1.1/Isaac/Robots/Franka/`）中，将`franka_alt_fingers.usd`拖入场景。这是你的主角——Franka机械臂。
  2. **添加关键物体** ：
     - 在资产库中搜索杯子。一个很好的来源是YCB数据集，路径通常在 `NVIDIA/Assets/Isaac/2023.1.1/Isaac/Props/YCB/`。找到`025_mug`（马克杯）或`024_bowl`（碗）等合适的模型。
     - 创建**两个** 杯子的实例，分别命名为`source_cup`（源杯）和`target_cup`（目标杯）。
  3. **精确布局** ：
     - 将机械臂、`source_cup`和`target_cup`摆放到合理的位置。用代码或UI精确设置它们的坐标。例如：
       - `source_cup` 放在 `(x=0.5, y=0.2, z=桌子高度)`
       - `target_cup` 放在 `(x=0.5, y=-0.2, z=桌子高度)`
     - **记录下这些坐标** ，你的代码将依赖这些精确的位置。
  4. **设置物理属性** ：
     - 选中两个杯子和桌子，为它们添加`Physics -> Rigid Body`组件。
     - 选中机械臂，确保它已经配置好了`Articulation`（关节驱动）和物理特性。从资产库拖入的机器人模型通常都已预先配置好。

**产出** ：一个包含机械臂和两个杯子的、静态的、物理设置正确的仿真场景。

#### **第2步：任务规划与路径点定义 (Task Planning & Waypoint Definition)**

这是“倒水”这个复杂任务的逻辑拆解，是整个流程的“剧本”。你需要将连续的动作分解为一系列关键的**末端执行器（Gripper）目标姿态** 。

  1. **定义“剧本”的关键帧** ：

     - **`Pose_Home`** ：一个安全的、远离所有物体的起始/结束姿态。
     - **`Pose_Pre_Grasp`** ：在`source_cup`正上方约10-15cm处。这是一个安全的接近姿态。
     - **`Pose_Grasp`** ：末端执行器准备好抓取`source_cup`的姿态。手指应该对准杯柄或杯身。
     - **`Pose_Post_Grasp_Lift`** ：抓紧杯子后，垂直向上提起一小段距离（例如5cm），以确保杯底离开桌面。
     - **`Pose_Move_To_Target`** ：将杯子移动到`target_cup`的正上方。
     - **`Pose_Pour_Start`** ：稍微降低高度，让`source_cup`的杯沿靠近`target_cup`的中心。
     - **`Pouring_Sequence`** ：**这不仅仅是一个点，而是一个姿态序列** 。从`Pose_Pour_Start`开始，保持杯沿位置基本不变，以其为轴心，逐步旋转手腕关节，使杯子倾斜。例如，定义一个从倾斜0度到90度的5个中间姿态。
     - **`Pose_Pour_End`** ：倾倒完成后，将手腕转回水平姿态。
     - **`Pose_Return_Home`** ：回到安全的`Pose_Home`。
  2. **获取姿态坐标** ：

     - 在Isaac Sim中，手动将机械臂的末端执行器移动到你设想的每一个路径点位置。
     - 使用Python脚本（或UI中的属性窗口）读取并记录下末端执行器的**位置（Position）和姿态（Orientation, 四元数）** 。将这些数据硬编码或保存在一个配置文件中。

**产出** ：一个定义了倒水流程中所有关键路径点坐标的列表或字典。

#### **第3步：编写运动控制脚本 (Motion Control Scripting)**

现在，我们将使用cuMotion来连接上一步定义的路径点，让机器人动起来。

  1. **脚本结构** ：
     - 创建一个独立的Python脚本。
     - 在脚本开头，初始化Isaac Sim环境并加载你的场景USD文件。
     - 获取机械臂、cuMotion和夹爪的接口。
  2. **实现运动逻辑** ：
     - 编写一个主循环，按顺序遍历你在第2步中定义的路径点列表。
     - 对于每一个路径点：
       - 调用cuMotion的API，将该路径点的姿态设置为`target_pose`。
       - 执行运动规划和移动。
       - **等待** 机器人运动完成。cuMotion通常有API可以查询运动状态是否为`DONE`。
  3. **集成夹爪控制** ：
     - 在运动流程的正确时机插入夹爪（Gripper）控制命令。
     - 在移动到`Pose_Grasp`**之后** ，调用`franka.gripper.close()`。
     - 在未来需要释放杯子时，调用`franka.gripper.open()`。
     - **注意** ：抓取后，需要将杯子“附着”（Attach）到机械臂的夹爪上，这样移动机械臂时杯子才会跟着动。Isaac Sim提供了`UsdPhysics.FilteredPairsAPI` 或更简单的约束API来实现这一点。

**产出** ：一个可以驱动机械臂、按照你设计的路径点、完成“抓取-移动-倾倒（空动作）-返回”完整流程的Python脚本。此时杯子里还没有水。

#### **第4th步：集成液体仿真与迭代调试 (Fluid Simulation & Iteration)**

这是见证奇迹和反复失败的时刻。

  1. **添加液体** ：
     - 在Isaac Sim中，选择`Create -> Physics -> Particle System`。
     - 在属性中，将其类型设置为`Fluid`。
     - 将这个粒子系统的发射器（Emitter）放置在`source_cup`的内部。
     - 调整液体属性，如粒子数量、粘度（viscosity）等，使其看起来像水。
  2. **运行与观察** ：
     - 运行你的完整运动脚本。
     - **仔细观察** ：水是如何被倒出来的？是平稳地流入目标杯，还是溅得到处都是？
  3. **迭代与优化（核心学习环节）** ：
     - **问题** ：水倒得太快，溅出来了。
       - **解决方案** ：回到第2步，调整`Pouring_Sequence`。减慢倾倒时的旋转速度（在cuMotion中设置更低的最大速度），或者将倾倒序列分解为更多、更精细的中间姿态。
     - **问题** ：水没有完全倒入目标杯，偏了。
       - **解决方案** ：回到第2步，微调`Pose_Pour_Start`的位置，确保源杯杯沿的位置是准确的。
     - **问题** ：机械臂的运动看起来很生硬。
       - **解决方案** ：检查cuMotion的配置，可以调整`blending_radius`等参数让路径点之间的过渡更平滑。或者在关键路径点之间增加一些辅助性的中间点。

**这个反复调试、优化路径点的过程，就是你积累“专家知识”的过程。** 目标是找到一组“黄金参数”，使得倒水动作既高效又稳定。

**产出** ：一个经过反复调试、能够完美完成仿真倒水任务的运动脚本和优化后的路径点数据。

#### **第5步：记录专家数据 (Recording Expert Data)**

这是为你未来的AI训练（第三阶段）准备弹药。

  1. **确定要记录的数据** ：
     - **状态（Observation）** ：在仿真的每一个时间步（或以固定频率，如30Hz），你需要记录：
       - 机械臂的关节角度 `joint_positions`
       - 机械臂的关节速度 `joint_velocities`
       - 末端执行器的世界坐标系姿态 `end_effector_pose`
     - **动作（Action）** ：记录你“命令”机器人做的事情。对于使用cuMotion，这稍微复杂。一个好的做法是记录由cuMotion计算出的、发送给底层的**关节速度指令** 。你可以通过API从`ArticulationController`中读取这些指令。
  2. **编写数据记录器** ：
     - 在你的主循环中，添加代码。
     - 在每个`simulation_app.update()`之后，读取上述的状态和动作数据。
     - 将这些数据（例如，`timestamp, q1, ..., q7, q_dot1, ..., q_dot7, action1, ..., action7`）追加到一个文件中（如CSV、Numpy的`.npy`文件或HDF5文件）。
  3. **执行并保存** ：
     - 运行你完美的倒水脚本数次，生成一个包含多次成功演示的数据集。

**产出** ：一个或多个数据文件，其中包含了机器人完成倒水任务时的完整状态-动作序列。这是模仿学习的宝贵食粮。

* * *

完成以上所有步骤后，你就圆满地完成了第一阶段。你不仅掌握了如何使用cuMotion解决一个复杂的机器人任务，还拥有了一个高质量的仿真环境和一份宝贵的专家数据集，为你后续的探索铺平了道路。祝你学习顺利！

更快的仿真方法: 

### **方法三：示教器/遥操作 (Teleoperation / Teach Pendant)**

这是介于纯手动和纯自动之间的一种非常实用且流行的方法，也是工业界和研究界最常用的方法之一。

  - **是什么** ：你使用一个输入设备来“实时”控制仿真环境中的机械臂，就像玩一个精密的视频游戏。你亲手完成一次倒水，系统会记录下你操作的全过程作为专家数据。
  - **常见的输入设备** ：
    - **VR手柄 (如Oculus Quest, HTC Vive)** ：这是目前非常流行的方式。你在VR环境中“看到”机械臂，用VR手柄直接“抓住”机械臂的末端，然后像移动自己的手一样移动它。Isaac Sim对VR遥操作有很好的支持。
    - **专用遥操作设备 (如Geomagic Touch, Franka Panda的遥操作模式)** ：这些设备提供力反馈，当你控制的虚拟机械臂碰到物体时，你的手上也能感觉到力。这使得操作更加直观和精确。
    - **键盘/鼠标** ：一种成本最低但操作最不直观的方式。通过按键来控制机械臂末端在X/Y/Z轴上的移动和旋转。Isaac Sim也支持这种方式。
  - **如何操作** ：
    1. 在Isaac Sim中设置好遥操作接口，连接你的输入设备。
    2. 启动仿真，戴上VR头盔或拿起手柄。
    3. 实时控制机械臂，亲自完成抓取、移动、倾倒、放回的全过程。
    4. 在后台运行一个脚本，记录下你操作过程中机械臂的关节状态和末端执行器姿态。
  - **优点** ：
    - **直观高效** ：比手动拖拽和设定坐标点要快得多，也更符合人类的直觉。一次完美的演示可能只需要一两分钟就能录制好。
    - **高质量数据** ：因为是实时操作，数据是连续且动态的，比静态的关键点能更好地捕捉到任务的精髓。
    - **可扩展性好** ：可以轻松录制大量的、略有不同的演示数据，以增加AI模型的泛化能力。
  - **缺点** ：
    - **需要额外硬件** ：至少需要一个VR设备或键盘/鼠标的熟练操作。力反馈设备则价格不菲。
    - **需要一定的操作技巧** ：就像玩游戏一样，需要一定的练习才能熟练、平稳地控制机械臂。

**结论** ：如果你有VR设备，这**绝对是比手动设置路径点更优越、更高效的方法** 。它是目前生成高质量专家演示数据的“黄金标准”之一。

# 考虑使用大象机器人 (京东可达)

urdf文件在其开源的ros中: <https://github.com/elephantrobotics/mycobot_ros>

问题是为什么其会有一个ros: 

简而言之：**仿真得到的数据非常有价值，它是连接仿真世界和现实世界的桥梁。`mycobot_ros2`组件的角色是作为现实世界中的“执行者”，而不是数据的“创造者”。**

让我们通过一个清晰的流程和比喻来理解这一点。

* * *

### **核心理念：数据与执行的分离**

你可以把整个过程想象成**“制定旅行计划”**和** “实际开车旅行”**。

  1. **制定旅行计划 (在Isaac Sim中生成轨迹)**

     - **工具** ：你使用一个高级的地图软件（比如Google Maps，这里就是**Isaac Sim** ）。
     - **过程** ：你在这个软件里规划了一条从A到B的完美路线。你拖动路线，避开虚拟的拥堵，确保路线最优。
     - **产出** ：你得到了一份详细的路线清单（**轨迹数据** ），它可能是一系列GPS坐标点和对应的时间：`[时间1, 坐标1], [时间2, 坐标2], ...`。
  2. **实际开车旅行 (在真实机械臂上执行)**

     - **工具** ：你坐进你的真实汽车里。你的汽车有一个内置的导航系统（这里就是**`mycobot_ros2`组件**和它背后的固件）。
     - **过程** ：你把之前规划好的路线清单（轨迹数据）输入到汽车的导航系统里。
     - **执行** ：导航系统（`mycobot_ros2`）并不知道这条路线是怎么来的，它只负责一件事：读取清单上的下一个坐标点，然后向汽车的发动机和方向盘发出指令（“加速”、“向左转15度”），让汽车精确地到达那个坐标点。它会一个点一个点地执行，直到完成整个旅程。

在这个比喻中：

  - **轨迹数据 (路线清单)** 是通用的、抽象的。它只描述了“去哪里”，而不关心“怎么开”。
  - **Isaac Sim (地图软件)** 是一个理想化的环境，用于生成这个“去哪里”的计划。
  - **`mycobot_ros2` (汽车导航系统)** 是一个执行引擎，负责将“去哪里”的抽象指令，翻译成真实硬件能懂的底层命令（“转动电机”、“设置电流”）。

所以，你在Isaac Sim中生成数据时**不需要**`mycobot_ros2`，因为你只是在“规划路线”。当你需要在真实硬件上**复现** 这条路线时，你才需要`mycobot_ros2`来“驾驶汽车”。

* * *

### **具体的技术工作流程**

下面是这个过程在技术上是如何实现的：

**阶段一：在Isaac Sim中生成并保存轨迹数据**

  1. **环境搭建** ：你在Isaac Sim中搭建场景，导入myCobot的URDF模型，放置目标物体。
  2. **任务执行与数据记录** ：你使用Isaac Sim的Python API编写脚本，控制仿真机器人完成任务（例如，从A点抓取一个方块移动到B点）。在机器人运动的每一步（或每一帧），你记录下它所有关节的角度值。
  3. **数据输出** ：你将记录下的数据保存到一个文件中，比如CSV或JSON格式。这个文件看起来可能像这样：
         
         timestamp, joint_1, joint_2, joint_3, joint_4, joint_5, joint_6
         0.00,      0.0,     0.0,     0.0,     0.0,     0.0,     0.0
         0.02,      0.01,    -0.005,  0.02,    0.0,     0.008,   0.0
         0.04,      0.02,    -0.010,  0.04,    0.0,     0.016,   0.0
         ...        ...      ...      ...      ...      ...      ...
         

这份文件就是你的宝贵资产——**与平台无关的关节空间轨迹** 。

**阶段二：在真实机器人上复现轨迹**

  1. **环境准备** ：现在你切换到控制真实myCobot的电脑上。这台电脑上安装了ROS2，并且已经按照官方文档正确安装和配置了`mycobot_ros2`软件包。
  2. **编写“轨迹播放器”节点** ：你需要编写一个新的、简单的ROS2 Python节点。我们称之为`trajectory_player`。
     - 这个节点的功能是：  
a. 读取你之前保存的`trajectory.csv`文件。  
b. 创建一个ROS2发布者（Publisher），它将发布标准的ROS2消息，比如 `trajectory_msgs/msg/JointTrajectory` 或更简单的 `sensor_msgs/msg/JointState`。  
c. 循环读取CSV文件的每一行数据。对于每一行，它将关节角度填充到一个ROS2消息中，然后通过发布者将这个消息发布出去。  
d. 你需要控制发布的频率，使其与仿真时记录数据的时间步长（`timestamp`）相匹配。
  3. **启动与执行** ：
     - 在一个终端中，你启动`mycobot_ros2`提供的核心节点。这个节点会启动与真实机械臂的通信，并**订阅（Subscribe）**一个用于接收关节指令的话题（Topic），例如 `/joint_trajectory_controller/joint_trajectory`。
     - 在另一个终端中，你运行你编写的`trajectory_player`节点。
     - 你的`trajectory_player`节点开始发布轨迹点 -> `mycobot_ros2`的节点接收到这些轨迹点 -> `mycobot_ros2`将其翻译成串口指令发送给机械臂 -> **真实机械臂开始运动，复现你在仿真中看到的动作！**

### **“仿真与现实的差距 (Sim2Real Gap)”**

需要特别注意的是，这个过程并非100%完美。直接将在仿真中生成的轨迹在真实世界中回放，可能会遇到问题，这就是所谓的“Sim2Real Gap”。原因包括：

  - **动力学差异** ：仿真的摩擦力、重力、惯性可能与真实世界不完全一致。
  - **模型误差** ：URDF模型是对真实机器人的近似。真实的连杆长度、关节位置可能有微小的制造误差。
  - **控制延迟** ：真实世界的通信和电机响应存在延迟。

**解决方案** ：

  - **使用反馈控制** ：`mycobot_ros2`通常会集成`ros2_control`框架。这意味着你发布的轨迹点是**目标（Goal）** ，而底层的控制器会根据真实编码器的读数不断调整电机指令来**追踪** 这个目标，而不是盲目执行。这能修正很多小误差。
  - **领域随机化 (Domain Randomization)** ：在仿真时，故意随机化模型的物理参数（如质量、摩擦系数），让生成的轨迹对这些变化更具鲁棒性。
  - **强化学习 (Reinforcement Learning)** ：更高级的方法是，在仿真中训练一个能根据传感器输入做出决策的“策略网络”，而不是一个固定的轨迹。这个策略通常比固定轨迹更容易迁移到现实世界。

**总结：**

仿真得到的数据，其核心作用是**定义了一个与实现方式无关的、理想化的运动计划** 。`mycobot_ros2`的作用是**将这个理想计划在充满物理限制的现实世界中尽可能精确地执行出来** 。理解了这种“计划与执行”的分离，你就掌握了机器人仿真和Sim2Real的核心思想。

拟选择型号 280 M5, 不带计算单元 (比如Jetson Nano) (<https://item.jd.com/10078192728948.html#switch-sku>)

URDF定义: <https://github.com/elephantrobotics/mycobot_ros2/tree/humble/mycobot_description/urdf/mycobot_280_m5>

先按照URDF中的路径, 重新组织资源文件: 

![image2025-10-23 15:21:10.png](/assets/01KJC06PG4RM9T7WFDEN5HPZPN/image2025-10-23%2015%3A21%3A10.png)

导入URDF: 

![image2025-10-23 15:5:26.png](/assets/01KJC06PG4RM9T7WFDEN5HPZPN/image2025-10-23%2015%3A5%3A26.png)

![image2025-10-23 15:6:51.png](/assets/01KJC06PG4RM9T7WFDEN5HPZPN/image2025-10-23%2015%3A6%3A51.png)

导入后, 生成USD文件, 并可以看到机械臂影像: 

![image2025-10-23 15:25:50.png](/assets/01KJC06PG4RM9T7WFDEN5HPZPN/image2025-10-23%2015%3A25%3A50.png)

设置物理属性, 先找到PDI的设置: 

但只找到了夹爪的配置文件 (<https://github.com/elephantrobotics/mycobot_ros/blob/879f86c160d1410e5db1670242da23de8b985574/mycobot_280/mycobot_280m5_gazebo_gripper/config/gazebo_controllers.yaml#L10>), P:D:I = 100:1:1, 看上去不一定合理

但先按照这个PDI配置, 关节的物理初始值设置为: Stiffness = 100, Damping = 1. 目前暂不设置

# 因为4.2的相关教程缺失, 换用Isaac Sim 5.1.0.

重新import: 

![image2025-10-23 22:59:35.png](/assets/01KJC06PG4RM9T7WFDEN5HPZPN/image2025-10-23%2022%3A59%3A35.png)

会导致Isaac Sim崩溃, 只能先用4.2对USDF进行import成USD. 然后使用5.1.0加载USD.

开启新的Stage, 将机械臂USD加入场景.

按照教程, 应当给机械臂设置articulation root, 但在机械臂的根节点, Add -> Physics 中没有找到articulation root.

在Stage中筛选已经存在的articulation root, 发现已经自动放在了: root_joint上

![image2025-10-23 23:54:16.png](/assets/01KJC06PG4RM9T7WFDEN5HPZPN/image2025-10-23%2023%3A54%3A16.png)![image2025-10-23 23:55:48.png](/assets/01KJC06PG4RM9T7WFDEN5HPZPN/image2025-10-23%2023%3A55%3A48.png)

同理, 在要添加drive时, 发现已经自动放在了 joint上, 先不调整参数: 

![image2025-10-24 0:6:31.png](/assets/01KJC06PG4RM9T7WFDEN5HPZPN/image2025-10-24%200%3A6%3A31.png)

在播放模式下, 可以使用Physics Inspector来驱动机械臂

注意: 如果在编辑模式下, 使用Physics Inspector后, 可能无法复原

# 重新整理步骤

先使用Isaac Sim 4.2.0, 将 mycobot_280_m5_adaptive_gripper.urdf 转成 USD. 这个机械臂带夹爪.

继续使用Isaac Sim 4.2.0: (使用5.1.0, 会偶尔crash)

  - 添加机械臂
  - 添加一个Cube, 作为桌子
  - 添加一个Cylinder, 作为杯子
  - 将桌子和杯子 添加 Physics -> Rigid Body with Colliders Preset, 对于桌子, Disable Gravity. 
  - 进行一下模拟查看效果
    - 杯子可以落在桌子上, 但机械臂会悬空
      - 机械臂只能被放置, 并设置各组件的物理特性, 但不能整体设置
      - 另外: 如果我删除机械臂的root_joint, 那么就可以为机械臂单独添加 Articulation Root, 但这样的话, 机械臂的各组件会瞬间垮塌, 需要重新设置各组件的参数

下一步: 让机械臂动起来. 

但使用Isaac Sim自己的运动规划器, API总是用不对. 于是考虑是否使用moveit: 

我整理一下现在的情况:   
1\. 我在Isaac Sim中, 已经添加了机械臂, 现在想通过Python代码让其完成"倒水"这个动作  
2\. 其要完成的第一个动作是要移动到水杯上方  
3\. 移动这个动作, 需要使用Isaac Sim提供的运动规划器  
4\. 这里使用的是Lula RMPflow  
5\. 但在机械臂的ROS2 模块中, 使用的是moveit作为运动规划器  
6\. 为什么Isaac Sim不能使用moveit作为运动规划器?

==========

你把情况整理得非常清晰，而且提出了一个非常核心且专业的问题！你的理解完全正确。现在我来回答你最关键的问题：为什么 Isaac Sim 不能直接使用 MoveIt 作为运动规划器？

* * *

### 答案：因为它们是两个生活在不同“世界”的独立系统

想象一下：

  - Isaac Sim 是一个完整的数字孪生世界。它有自己的物理引擎、自己的场景描述（USD）、自己的时间循环和自己的机器人控制API（Python）。它就像一个自给自足的虚拟宇宙。

  - MoveIt 是 ROS 世界里的“大脑”。它是一个运动规划框架，生活在 ROS 的生态系统中。它通过 ROS 的通信机制（Topics, Services, Actions）来获取世界信息并发送控制指令。

它们不能直接对话，因为它们说的“语言”和感知世界的方式完全不同：

| 特性 | Isaac Sim (内部规划器, 如 RMPflow) | ROS + MoveIt |
| --- | --- | --- |
| 如何“看”世界 | 直接读取 USD 场景图。能瞬间知道场景中每个物体的精确位置、形状和物理属性。 | 通过订阅 ROS Topics (/tf, /scan, /camera/depth/image_raw 等) 来构建一个规划场景 (PlanningScene)。 |
| 如何“知道”机器人状态 | 直接从物理引擎读取关节的精确角度和速度。 | 订阅 ROS 的 /joint_states 话题来获取机器人关节状态。 |
| 如何“控制”机器人 | 通过 Python API 直接向物理引擎施加关节力矩或设置目标位置 (robot.apply_action())。 | 发布一个规划好的轨迹到一个 ROS Action Server (/follow_joint_trajectory)。 |
| 运行环境 | 在 Isaac Sim 应用内部的 Python 脚本环境中运行。 | 在独立的 ROS 节点 (Nodes) 中运行，可以和 Isaac Sim 在不同机器上。 |
  
* * *

### 那么，如何让它们一起工作呢？ -> 通过“ROS Bridge”翻译官

虽然它们不能直接对话，但我们可以雇一个“翻译官”，这就是 Isaac Sim 的 ROS Bridge (omni.isaac.ros2_bridge)。它的工作流程是这样的：

  1. Isaac Sim -> ROS (状态发布):

  - ROS Bridge 会实时读取 Isaac Sim 物理引擎中的机器人关节状态。

  - 然后，它将这些状态打包成标准的 ROS 消息，并发布到 /joint_states 话题上。

  - MoveIt 监听这个话题，于是就知道了虚拟机器人的当前姿态。

  1. ROS -> Isaac Sim (指令接收):

  - 你在 ROS 中运行 MoveIt，它规划出一条轨迹。

  - MoveIt 将这条轨迹发送给一个 Action Server (通常是 ros2_control)。

  - ROS Bridge 订阅这个控制指令，将 ROS 的轨迹消息翻译成 Isaac Sim 能理解的 Python API 调用 (robot.apply_action())。

  - 最终，虚拟机器人就在 Isaac Sim 中执行了 MoveIt 规划的动作。

这个过程如下图所示：

+--------------------------------+ +---------\---------+ +---------------------------------+

| Isaac Sim World | | ROS Bridge | | ROS / MoveIt World |

| | | (翻译官)  | | |

| [虚拟机器人 (USD Prim)] | |  | | [MoveIt 规划节点] |

| [物理引擎] | |  | | [规划场景] |

| | |  | | |

| ↓ (直接读取状态) | |  | | ↑ (订阅 /joint_states) |

| |----->| 发布/joint_states |----->|  |

| ↑ (直接执行动作) | |  | | ↓ (发布轨迹指令) |

| |<\-----| 订阅控制指令  |<\-----| |

+--------------------------------+ +---------\---------+ +---------------------------------+

* * *

### 结论：你应该选择哪种方式？

  - 如果你只是想在 Isaac Sim 中完成任务（比如动画、仿真测试、数据生成）：

  - 最佳选择是使用内部规划器（如 Lula RMPflow）。

  - 原因：它更直接、性能更高（没有通信开销），配置也更简单（纯 Python，无需 ROS 环境）。我们当前遇到的问题只是配置文件格式的细节问题。

  - 如果你的目标是开发将来要部署到真实机器人上的软件（Sim-to-Real）：

  - 最佳选择是使用 MoveIt + ROS Bridge。

  - 原因：你可以确保在仿真中调试的 MoveIt 代码，无需修改就能在真实的 MyCobot 机器人上运行。

# 使用ros2

![image2025-10-25 0:39:2.png](/assets/01KJC06PG4RM9T7WFDEN5HPZPN/image2025-10-25%200%3A39%3A2.png)

![image2025-10-25 0:39:55.png](/assets/01KJC06PG4RM9T7WFDEN5HPZPN/image2025-10-25%200%3A39%3A55.png)

不论是配置 /World/firefighter 还是 /World/firefighter/root_joint 都会报错: Prim is not an articulation

修复:

  - 将/World/firefighter/root_joint属性中的 Articulation Root章节删掉, 这样就可以在/World/firefighter中增加 Add->Physical->Articulation Root
  - 重新使用 ROS2 JointState parameters功能, 就可以添加新的Graph
    - ![image2025-10-25 1:2:16.png](/assets/01KJC06PG4RM9T7WFDEN5HPZPN/image2025-10-25%201%3A2%3A16.png)
  - 重新Play后, 使用ros2 topic list就可以看到 /joint_command和/joint_states
  - 使用 ros2 topic echo /joint_states 可以看到结果

但/joint_states的结果值都是nan, 并且能看到warning: 

```
2025-10-24 17:19:23 [208,201ms] [Warning] [omni.physx.plugin] Invalid PhysX transform detected for /World/firefighter/joint2.
2025-10-24 17:19:23 [208,201ms] [Warning] [omni.physx.plugin] Invalid PhysX transform detected for /World/firefighter/joint3.
2025-10-24 17:19:23 [208,201ms] [Warning] [omni.physx.plugin] Invalid PhysX transform detected for /World/firefighter/joint4.
2025-10-24 17:19:23 [208,201ms] [Warning] [omni.physx.plugin] Invalid PhysX transform detected for /World/firefighter/joint5.
2025-10-24 17:19:23 [208,201ms] [Warning] [omni.physx.plugin] Invalid PhysX transform detected for /World/firefighter/joint6.
2025-10-24 17:19:23 [208,201ms] [Warning] [omni.physx.plugin] Invalid PhysX transform detected for /World/firefighter/joint6_flange.
2025-10-24 17:19:23 [208,201ms] [Warning] [omni.physx.plugin] Invalid PhysX transform detected for /World/firefighter/gripper_base.
2025-10-24 17:19:23 [208,201ms] [Warning] [omni.physx.plugin] Invalid PhysX transform detected for /World/firefighter/gripper_left2.
2025-10-24 17:19:23 [208,201ms] [Warning] [omni.physx.plugin] Invalid PhysX transform detected for /World/firefighter/gripper_right2.
2025-10-24 17:19:23 [208,201ms] [Warning] [omni.physx.plugin] Invalid PhysX transform detected for /World/firefighter/gripper_right3.
2025-10-24 17:19:23 [208,201ms] [Warning] [omni.physx.plugin] Invalid PhysX transform detected for /World/firefighter/gripper_left3.
2025-10-24 17:19:23 [208,201ms] [Warning] [omni.physx.plugin] Invalid PhysX transform detected for /World/firefighter/gripper_right1.
2025-10-24 17:19:23 [208,201ms] [Warning] [omni.physx.plugin] Invalid PhysX transform detected for /World/firefighter/gripper_left1.

``` 

通过Python脚本检查各组件的物理属性, 并集中设置, 并不能解决问题. 采用分离诊断, 重建一个新场景, 导入机械臂, 然后只启用机械臂的部分, 进行诊断

先处理之前忽略的一个Error: 

```
2025-10-25 04:53:44 [329,445ms] [Error] [omni.physx.plugin] Usd Physics: PhysxMimicJointAPI at /World/mycobot_280_m5_adaptive_gripper/gripper_base/gripper_base_to_gripper_left2 must have exactly 1 "referenceJoint" relationship defined.

2025-10-25 04:53:44 [329,445ms] [Error] [omni.physx.plugin] Usd Physics: PhysxMimicJointAPI at /World/mycobot_280_m5_adaptive_gripper/gripper_base/gripper_base_to_gripper_right2 must have exactly 1 "referenceJoint" relationship defined.

2025-10-25 04:53:44 [329,445ms] [Error] [omni.physx.plugin] Usd Physics: PhysxMimicJointAPI at /World/mycobot_280_m5_adaptive_gripper/gripper_base/gripper_base_to_gripper_right3 must have exactly 1 "referenceJoint" relationship defined.

2025-10-25 04:53:44 [329,445ms] [Error] [omni.physx.plugin] Usd Physics: PhysxMimicJointAPI at /World/mycobot_280_m5_adaptive_gripper/gripper_right3/gripper_right3_to_gripper_right1 must have exactly 1 "referenceJoint" relationship defined.

``` 

这是关于模仿关节 (MimicJoint) 缺少 参考关节的问题 (不知道该模仿谁), 查看报错关节的USDA描述: 

```
            def PhysicsRevoluteJoint "gripper_base_to_gripper_left2" (
                apiSchemas = ["PhysxJointAPI", "PhysicsJointStateAPI:angular", "PhysxMimicJointAPI:rotZ"]
            )
            {
                uniform token physics:axis = "Z"
                rel physics:body0 = </World/mycobot_280_m5_adaptive_gripper/gripper_base>
                rel physics:body1 = </World/mycobot_280_m5_adaptive_gripper/gripper_left2>
                float physics:breakForce = 3.4028235e38
                float physics:breakTorque = 3.4028235e38
                point3f physics:localPos0 = (-0.005, 0.027, 0)
                point3f physics:localPos1 = (0, 0, 0)
                quatf physics:localRot0 = (1, 0, 0, 0)
                quatf physics:localRot1 = (1, 0, 0, 0)
                float physics:lowerLimit = -45.83662
                float physics:upperLimit = 28.647888
                float physxJoint:maxJointVelocity = 3.4028235e38
                float physxMimicJoint:rotZ:gearing = -1
                float physxMimicJoint:rotZ:offset = 0
                float state:angular:physics:position = 0
                float state:angular:physics:velocity = 0
            }
``` 

其中 :

  - `gearing = -1`：这明确告诉我们，`gripper_left2` 这个关节的运动方向将与主关节**完全相反** 。这完美地模拟了夹爪的对称开合。
  - `offset = 0`：这表示当主关节在零点位置（比如完全闭合或完全张开的某个基准点）时，这个从关节也在它的零点位置。

通过对夹爪结构的分析:

  - 判断前三个夹爪 (/World/mycobot_280_m5_adaptive_gripper/gripper_base/gripper_base_to_gripper_left2 , /World/mycobot_280_m5_adaptive_gripper/gripper_base/gripper_base_to_gripper_right2 , /World/mycobot_280_m5_adaptive_gripper/gripper_base/gripper_base_to_gripper_right3) 的参考夹爪是 /World/mycobot_280_m5_adaptive_gripper/gripper_base/gripper_controller
  - /World/mycobot_280_m5_adaptive_gripper/gripper_right3/gripper_right3_to_gripper_right1 的参考夹爪, 模仿/World/mycobot_280_m5_adaptive_gripper/gripper_left3的情况狂, 也是gripper_controller

仿真Play一下, 没有Error了. 

重新设置机械臂的Articulation Root.

仿真Play一下, 没有报错.

添加ROS2的接口, 通过ros2 topic echo /joint_states检查, 就没有问题了: 

```
header:
  stamp:
    sec: 138
    nanosec: 133340537
  frame_id: ''
name:
- joint2_to_joint1
- joint3_to_joint2
- joint4_to_joint3
- joint5_to_joint4
- joint6_to_joint5
- joint6output_to_joint6
- gripper_base_to_gripper_left2
- gripper_base_to_gripper_right2
- gripper_base_to_gripper_right3
- gripper_controller
- gripper_right3_to_gripper_right1
- gripper_left3_to_gripper_left1
position:
- -0.0012
- -0.0007
- 0.001
- -0.0005
- -0.0011
- 0.0002
- 0.0059
- -0.004
- -0.15
- -0.0012
- -0.1306
- 0.0115
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
effort:
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
``` 

(不知道之前设置错了什么, 导致出现了问题, 现在已经正常)

添加一个地面(Ground Plane), 再添加一个圆柱体, Play没有报错, 但目前没有重力

给圆柱体添加 Rigid body with Collidars presets, Play没有报错, 已经有重力

使用ros2简单的调度脚本: [ros2.py](/assets/01KJC06PG4RM9T7WFDEN5HPZPN/ros2.py) : 

```
admin@10-60-141-108:/workspaces/isaac_ros-dev/huangyan$ python3 ros2.py joint2_to_joint1 1.0
[INFO] [1761372428.961113119] [single_joint_controller]: Single Joint Controller Ready.
[INFO] [1761372429.962809844] [single_joint_controller]: Publishing command for joint "joint2_to_joint1": 0.0175 radians
admin@10-60-141-108:/workspaces/isaac_ros-dev/huangyan$ python3 ros2.py joint2_to_joint1 5.0
[INFO] [1761372442.774760988] [single_joint_controller]: Single Joint Controller Ready.
[INFO] [1761372443.776416787] [single_joint_controller]: Publishing command for joint "joint2_to_joint1": 0.0873 radians
admin@10-60-141-108:/workspaces/isaac_ros-dev/huangyan$ python3 ros2.py joint2_to_joint1 10.0
[INFO] [1761372455.907690268] [single_joint_controller]: Single Joint Controller Ready.
[INFO] [1761372456.909705628] [single_joint_controller]: Publishing command for joint "joint2_to_joint1": 0.1745 radians
admin@10-60-141-108:/workspaces/isaac_ros-dev/huangyan$ python3 ros2.py joint2_to_joint1 15.0
[INFO] [1761372466.993639599] [single_joint_controller]: Single Joint Controller Ready.
[INFO] [1761372467.995163554] [single_joint_controller]: Publishing command for joint "joint2_to_joint1": 0.2618 radians
admin@10-60-141-108:/workspaces/isaac_ros-dev/huangyan$ python3 ros2.py joint2_to_joint1 20.0
[INFO] [1761372472.924174844] [single_joint_controller]: Single Joint Controller Ready.
[INFO] [1761372473.926010539] [single_joint_controller]: Publishing command for joint "joint2_to_joint1": 0.3491 radians
admin@10-60-141-108:/workspaces/isaac_ros-dev/huangyan$ python3 ros2.py joint2_to_joint1 25.0
[INFO] [1761372477.336519154] [single_joint_controller]: Single Joint Controller Ready.
[INFO] [1761372478.338269814] [single_joint_controller]: Publishing command for joint "joint2_to_joint1": 0.4363 radians
admin@10-60-141-108:/workspaces/isaac_ros-dev/huangyan$ python3 ros2.py joint2_to_joint1 30.0
[INFO] [1761372481.921241711] [single_joint_controller]: Single Joint Controller Ready.
[INFO] [1761372482.922894960] [single_joint_controller]: Publishing command for joint "joint2_to_joint1": 0.5236 radians
admin@10-60-141-108:/workspaces/isaac_ros-dev/huangyan$ python3 ros2.py joint2_to_joint1 40.0
[INFO] [1761372487.478248882] [single_joint_controller]: Single Joint Controller Ready.
[INFO] [1761372488.479357526] [single_joint_controller]: Publishing command for joint "joint2_to_joint1": 0.6981 radians
admin@10-60-141-108:/workspaces/isaac_ros-dev/huangyan$ python3 ros2.py joint2_to_joint1 60.0
[INFO] [1761372494.417968702] [single_joint_controller]: Single Joint Controller Ready.
[INFO] [1761372495.419837231] [single_joint_controller]: Publishing command for joint "joint2_to_joint1": 1.0472 radians
admin@10-60-141-108:/workspaces/isaac_ros-dev/huangyan$ python3 ros2.py joint2_to_joint1 90.0
[INFO] [1761372502.347498341] [single_joint_controller]: Single Joint Controller Ready.
[INFO] [1761372503.349350950] [single_joint_controller]: Publishing command for joint "joint2_to_joint1": 1.5708 radians
(此处Isaac Sim会报 Invalid PhysX transform detected for /World/mycobot_280_m5_adaptive_gripper/...)
``` 

认为是角度设定过大, 并没有使用运动规划, 所以瞬间扭矩过大, 导致崩溃

# 使用运动规划

安装机械臂的ros2和moveIt使用: 

<https://github.com/elephantrobotics/mycobot_ros2?tab=readme-ov-file>

```
pip install pymycobot --user

cd /workspaces/isaac_ros-dev/src
git clone -b humble --depth 1 https://github.com/elephantrobotics/mycobot_ros2.git

cd ..
colcon build
``` 

执行demo: 

```
ros2 launch mycobot_280_moveit2 demo.launch.py
``` 

效果: 

![image2025-10-26 14:15:39.png](/assets/01KJC06PG4RM9T7WFDEN5HPZPN/image2025-10-26%2014%3A15%3A39.png)

连接moveit和isaac sim: 

直接使用<https://github.com/elephantrobotics/mycobot_ros2> , 参考其安装步骤, 将其安装到ros2容器内/workspaces/isaac_ros-dev 

```
pip install pymycobot --user
cd /workspaces/isaac_ros-dev/src
git clone -b humble --depth 1 https://github.com/elephantrobotics/mycobot_ros2.git
cd /workspaces/isaac_ros-dev/
colcon build
source /workspaces/isaac_ros-dev/install/setup.bash
``` 

需要定制跟Isaac Sim互动的脚本:

代码: <https://github.com/ikarishinjieva/robots-ai/tree/main/moveit_and_isaac_sim> (9106b7542fb5d69c6e25052470e51addffec84b9)

使用deploy.sh, 将其软链到mycobot_ros2的各部分. 解释其作用: 

```
### **1. RViz用户交互**
- **定义**: `moveit.rviz` 配置文件加载 `moveit_rviz_plugin/MotionPlanning` 插件，提供交互界面。
- **启动**: `rviz_node` 在launch文件中启动。
- **主要作用**: 用户在RViz中拖动机械臂的交互式标记，定义一个目标位姿。

### **2. RViz插件处理**
- **定义**: `moveit_rviz_plugin` 的C++代码内部硬编码了标准的MoveIt Action接口。
- **启动**: 作为RViz的一部分被加载。
- **主要作用**: 捕获交互标记的目标位姿，并通过ROS 2 Action将规划请求发送到 `/move_action` 接口。

### **3. MoveIt接收请求**
- **定义**: `move_group` 节点是MoveIt框架的核心。
- **启动**: `move_group_node` 在launch文件中启动。
- **主要作用**: 作为 `/move_action` 的Action Server，接收来自RViz的规划请求。

### **4. 路径规划**
- **定义**: `MoveItConfigsBuilder` 在launch文件中整合了URDF, SRDF, `kinematics.yaml`, `joint_limits.yaml` 等配置文件。
- **启动**: `move_group_node` 启动时加载这些配置。
- **主要作用**: `move_group` 使用运动规划器（如OMPL）计算从当前状态到目标位姿的无碰撞关节轨迹。

### **5. 发送轨迹到控制器**
- **定义**: `moveit_controllers.yaml` 文件定义了MoveIt应与哪个控制器接口通信。
- **启动**: 此配置由 `MoveItConfigsBuilder` 加载并传递给 `move_group_node`。
- **主要作用**: 路径规划成功后，`move_group` 将生成的轨迹通过Action发送到 `arm_group_controller` 的 `/follow_joint_trajectory` 接口。

### **6. 控制器执行轨迹**
- **定义**: `ros2_controllers.yaml` 定义了 `arm_group_controller` 的类型 (`joint_trajectory_controller/JointTrajectoryController`) 和其需要控制的关节列表。
- **启动**: `spawner` 节点通过向 `controller_manager` 发送服务请求来加载并启动此控制器。
- **主要作用**: `arm_group_controller` 作为一个Action Server接收轨迹，并根据时间戳逐点计算要发送给硬件的关节指令。

### **7. 控制器与硬件的连接 (匹配机制)**
- **定义**:
    1.  **硬件接口**在 `.ros2_control.xacro` 中声明其**提供**的关节和接口类型（如 `position`, `velocity`）。
    2.  **控制器**在 `ros2_controllers.yaml` 中声明其**需要**的关节和接口类型。
- **启动**: `ros2_control_node` 启动时，`controller_manager` 被实例化。
- **主要作用**: `controller_manager` 充当中间人。当 `spawner` 请求加载控制器时，`controller_manager` 通过**关节名称和接口类型**匹配硬件接口提供的资源和控制器需要的资源。
- **重要说明**: 匹配成功后，`controller_manager` 将硬件接口的内存引用直接注入控制器。这是一个在**同一进程内**完成的**插件加载和依赖注入**过程，不涉及ROS网络通信。

### **8. 硬件接口执行**
- **定义**: `.ros2_control.xacro` 定义了硬件接口插件 (`TopicBasedSystem`) 及其参数（如ROS话题名称）。
- **启动**: `ros2_control_node` 加载 `robot_description` 时被实例化。
- **主要作用**: 硬件接口从 `arm_group_controller` 写入的内存中读取指令，并将其转换为具体的硬件命令（在此例中是发布到 `/joint_command` 话题）。

### **9. 仿真或物理执行**
- **定义**: Isaac Sim的Action Graph或真实机器人的驱动程序。
- **启动**: 在Isaac Sim或物理机器人启动时运行。
- **主要作用**: 订阅 `/joint_command` 话题，驱动仿真模型或物理机械臂的关节运动。

### **10. 状态反馈闭环**
- **定义**: Isaac Sim或真实机器人的编码器/驱动程序配置。
- **启动**: 持续运行。
- **主要作用**: 发布当前关节的实际状态（位置、速度）到 `/joint_states` 话题。

### **11. 状态更新**
- **定义**: 硬件接口 (`TopicBasedSystem`) 配置为订阅 `/joint_states`。`joint_state_broadcaster` 在 `ros2_controllers.yaml` 中定义。
- **启动**: `ros2_control_node` 和相应的 `spawner` 启动。
- **主要作用**: 硬件接口读取 `/joint_states` 更新内部状态。`joint_state_broadcaster` 则将这些状态广播给整个ROS系统，供RViz和MoveIt等节点使用以更新机器人的当前姿态。
``` 

其中: 

  - ros2_control.xacro中, 是从mycobot_ros2中复制过来, 修改了hardware部分, 支持了Isaac Sim的接口
  - srdf文件, 增加了夹爪的逻辑定义
  - urdf.xacro, 是从mycobot_ros2中复制过来, 更换成了带夹爪的urdf

示意图: 

![image2025-10-27 14:16:23.png](/assets/01KJC06PG4RM9T7WFDEN5HPZPN/image2025-10-27%2014%3A16%3A23.png)

添加圆柱体到moveit中: (手工添加常量, 实际使用时, 应当使用视觉来检测)

  - 在启动脚本中, 增加robot_state_publisher和static_transform_publisher, 增加世界坐标系
  - 增加一个scene_updater.py脚本, 将圆柱体作为CollisionObject, 添加到moveit
  - 运行scene_updater.py后, 在rviz中, 可以看到圆柱体
  - 还需要在脚本中增加use_sim_time, 在场景中增加ROS2 clock, 否则时间对不起, TF发布会报错
  - 代码commit: 794c118eaedf04ede9fbea8a678ac7fd373ec52c

![image2025-10-27 16:0:50.png](/assets/01KJC06PG4RM9T7WFDEN5HPZPN/image2025-10-27%2016%3A0%3A50.png)

在规划圆柱体的位置时, 发现一个问题, 夹爪的基座会和机器人的基座在规划时发生"碰撞" (但实际没有碰撞), 需要检查

整理一个步骤, 从rviz的碰撞的日志, 进行排查, 找到碰撞点, 并标记在rviz上: 

````
### 从报错到在 RViz 标记碰撞点（超简步骤）

- 记录碰撞帧索引
  - 在 `move_group` 日志中找到 “Invalid states at index locations: [ k ]”，记下 k（0 基）。

- 导出规划轨迹并取第 k 帧关节角
```bash
ros2 topic echo --once /display_planned_path > /tmp/traj.yaml
# 打开 /tmp/traj.yaml，取 joint_trajectory.joint_names 和 points[k].positions
```

- 用第 k 帧做有效性检查，拿到接触点与法向
```bash
ros2 service call /check_state_validity moveit_msgs/srv/GetStateValidity "{
  robot_state: { joint_state: { name: [替换为 joint_names...],
                                position: [替换为 points[k].positions...] } },
  group_name: 'arm_group',
  constraints: {}
}"
# 关注 response.valid、contacts[].position、contacts[].normal、contacts[].depth
```

- 在 RViz 显示第 k 帧机器人姿态（便于对齐场景）
```bash
ros2 topic pub --once /display_robot_state moveit_msgs/msg/DisplayRobotState "{
  state: { joint_state: { name: [joint_names...], position: [points[k].positions...] } }
}"
```

- 在 RViz 标记接触点（手动 Marker，必看得到）
  - 在 RViz：Add → Marker，Topic 设 `/collision_contact`；再加一个 Marker，Topic 设 `/collision_normal`；Fixed Frame 设为 `contacts.header.frame_id`（如 `g_base`）。
```bash
# 红色小球（接触点）
ros2 topic pub --once /collision_contact visualization_msgs/msg/Marker "{
  header: {frame_id: 'g_base'},
  ns: 'contacts', id: 1, type: 2, action: 0,
  pose: {position: {x: Px, y: Py, z: Pz}, orientation: {w: 1.0}},
  scale: {x: 0.01, y: 0.01, z: 0.01},
  color: {r: 1.0, g: 0.0, b: 0.0, a: 1.0}
}"
# 绿色箭头（法向，长度 5cm）
ros2 topic pub --once /collision_normal visualization_msgs/msg/Marker "{
  header: {frame_id: 'g_base'},
  ns: 'contacts', id: 2, type: 0, action: 0,
  points: [{x: Px, y: Py, z: Pz},
           {x: Px+Nx*0.05, y: Py+Ny*0.05, z: Pz+Nz*0.05}],
  scale: {x: 0.005, y: 0.01, z: 0.0},
  color: {r: 0.0, g: 1.0, b: 0.0, a: 1.0}
}"
```

- 可选：让 MoveIt 自动发接触标记
```bash
ros2 param set /move_group planning_scene_monitor.publish_contacts true
ros2 param set /move_group planning_scene_monitor.max_contacts 50
ros2 param set /move_group planning_scene_monitor.contact_distance 0.01
# 在 RViz 添加 MarkerArray，订阅 /display_contacts
```
```` 

举例: 

````
- 监听规划轨迹
```bash
ros2 topic echo --once /display_planned_path > /tmp/traj.yaml
```

 
进行规划, 得到错误日志: 
 
[move_group-6] [ERROR] [1761641385.780217940] [moveit.ros_planning.planning_pipeline]: Computed path is not valid. Invalid states at index locations: [ 31 ] out of 48. Explanations follow in command line. Contacts are published on /display_contacts
[move_group-6] [INFO] [1761641385.780260503] [moveit_collision_detection_fcl.collision_common]: Found a contact between 'cylinder' (type 'Object') and 'gripper_base' (type 'Robot link'), which constitutes a collision. Contact information is not stored.

 
碰撞帧 为 31
 
在/tmp/traj.yaml找到第32组position: 
 
- positions:
      - 0.07952340662408887
      - -0.29056189983870245
      - -0.8710307866416351
      - 0.941456360574789
      - -1.0707026551684633
      - -1.3457829191740336
      velocities:
      - -0.18056278662687766
      - -0.4420670841813961
      - -0.49413800187309337
      - 0.43867136229386783
      - -0.4001284111447548
      - -0.9999999999999658
      accelerations:
      - 1.534953245727744e-11
      - 3.7444599357663346e-11
      - 4.198751030914456e-11
      - -3.7169271421209944e-11
      - 3.386533618376906e-11
      - 8.480100442764934e-11
      effort: []
      time_from_start:
        sec: 3
        nanosec: 100000000
 
验证碰撞:
admin@10-60-141-108:/workspaces/isaac_ros-dev$ ros2 service call /check_state_validity moveit_msgs/srv/GetStateValidity "{
  robot_state: { joint_state: { name: ['joint2_to_joint1','joint3_to_joint2','joint4_to_joint3','joint5_to_joint4','joint6_to_joint5','joint6output_to_joint6'],
                                position: [0.07952340662408887,-0.29056189983870245,-0.8710307866416351,0.941456360574789,-1.0707026551684633,-1.3457829191740336] } },
  group_name: 'arm_group',
  constraints: {}
}"
requester: making request: moveit_msgs.srv.GetStateValidity_Request(robot_state=moveit_msgs.msg.RobotState(joint_state=sensor_msgs.msg.JointState(header=std_msgs.msg.Header(stamp=builtin_interfaces.msg.Time(sec=0, nanosec=0), frame_id=''), name=['joint2_to_joint1', 'joint3_to_joint2', 'joint4_to_joint3', 'joint5_to_joint4', 'joint6_to_joint5', 'joint6output_to_joint6'], position=[0.07952340662408887, -0.29056189983870245, -0.8710307866416351, 0.941456360574789, -1.0707026551684633, -1.3457829191740336], velocity=[], effort=[]), multi_dof_joint_state=sensor_msgs.msg.MultiDOFJointState(header=std_msgs.msg.Header(stamp=builtin_interfaces.msg.Time(sec=0, nanosec=0), frame_id=''), joint_names=[], transforms=[], twist=[], wrench=[]), attached_collision_objects=[], is_diff=False), group_name='arm_group', constraints=moveit_msgs.msg.Constraints(name='', joint_constraints=[], position_constraints=[], orientation_constraints=[], visibility_constraints=[]))

response:
moveit_msgs.srv.GetStateValidity_Response(valid=False, contacts=[moveit_msgs.msg.ContactInformation(header=std_msgs.msg.Header(stamp=builtin_interfaces.msg.Time(sec=64409, nanosec=520025884), frame_id='g_base'), position=geometry_msgs.msg.Point(x=0.18594412587526585, y=-0.12727449298898244, z=0.004250506023412053), normal=geometry_msgs.msg.Vector3(x=0.12426727719272147, y=-0.18705414347173877, z=0.9744579986993598), depth=0.08153682634578885, contact_body_1='gripper_base', body_type_1=0, contact_body_2='cylinder', body_type_2=1)], cost_sources=[moveit_msgs.msg.CostSource(cost_density=1.0, aabb_min=geometry_msgs.msg.Vector3(x=0.18105584976820163, y=-0.1459217614678571, z=-5.268740067409378e-07), aabb_max=geometry_msgs.msg.Vector3(x=0.2110559880957205, y=-0.11592081718166378, z=0.030000384642423147))], constraint_result=[])

 
在rviz中, 添加RobotState显示, 然后使用如下命令, 将机械臂状态固定到碰撞位置: 
 
ros2 topic pub --once /display_robot_state moveit_msgs/msg/DisplayRobotState "{
  state: { joint_state: { name: ['joint2_to_joint1','joint3_to_joint2','joint4_to_joint3','joint5_to_joint4','joint6_to_joint5','joint6output_to_joint6'],
                                position: [0.07952340662408887,-0.29056189983870245,-0.8710307866416351,0.941456360574789,-1.0707026551684633,-1.3457829191740336] } }
}"
 
 
- 在 RViz 标记接触点（手动 Marker，必看得到）
  - 在 RViz：Add → Marker，Topic 设 `/collision_contact`；再加一个 Marker，Topic 设 `/collision_normal`；Fixed Frame 设为 `contacts.header.frame_id`（如 `g_base`）。
```bash
# 红色小球（接触点）
ros2 topic pub --once /collision_contact visualization_msgs/msg/Marker "{
  header: {frame_id: 'g_base'},
  ns: 'contacts', id: 1, type: 2, action: 0,
  pose: {position: {x: Px, y: Py, z: Pz}, orientation: {w: 1.0}},
  scale: {x: 0.01, y: 0.01, z: 0.01},
  color: {r: 1.0, g: 0.0, b: 0.0, a: 1.0}
}"
# 绿色箭头（法向，长度 5cm）
ros2 topic pub --once /collision_normal visualization_msgs/msg/Marker "{
  header: {frame_id: 'g_base'},
  ns: 'contacts', id: 2, type: 0, action: 0,
  points: [{x: Px, y: Py, z: Pz},
           {x: Px+Nx*0.05, y: Py+Ny*0.05, z: Pz+Nz*0.05}],
  scale: {x: 0.005, y: 0.01, z: 0.0},
  color: {r: 0.0, g: 1.0, b: 0.0, a: 1.0}
}"
 
 
ros2 topic pub --once /collision_contact visualization_msgs/msg/Marker "{
  header: {frame_id: 'g_base'},
  ns: 'contacts', id: 1, type: 2, action: 0,
  pose: {position: {x: 0.18594412587526585, y: -0.12727449298898244, z: 0.004250506023412053}, orientation: {w: 1.0}},
  scale: {x: 0.01, y: 0.01, z: 0.01},
  color: {r: 1.0, g: 0.0, b: 0.0, a: 1.0}
}"
```` 

碰撞点在圆柱体上, 接近地平线, 但离机械臂很远: 

![image2025-10-28 17:56:48.png](/assets/01KJC06PG4RM9T7WFDEN5HPZPN/image2025-10-28%2017%3A56%3A48.png)

在URDF中, 将gridder_base的碰撞模型换成简单的box, 就可以正常工作

![image2025-10-29 10:23:21.png](/assets/01KJC06PG4RM9T7WFDEN5HPZPN/image2025-10-29%2010%3A23%3A21.png)

发现使用STL文件作为collision mesh文件, (替换gripper_base.dae), 可以正常工作: 

```
转换命令: 
sudo apt-get install -y assimp-utils
assimp export ./install/mycobot_description/share/mycobot_description/urdf/adaptive_gripper/gripper_base.dae gripper_base.stl
 
在用admesh转成二进制, 否则rviz会报错: 
mv gripper_base.stl gripper_base.stl.org
 
sudo apt-get install -y admesh
admesh \
  gripper_base.stl.orig \
  --write-binary \
  gripper_base.stl
``` 

STL和DAE的区别: (DAE 像“工程装配图/架构图”; STL 像“定格的快照”或“烘焙后的实体模型”)

下面这份科普，尽量用“家常话”把 DAE 和 STL 讲清楚，并结合你在 Isaac Sim + MoveIt 的碰撞问题，解释为什么 STL 在你的场景下更“听话”。

一、两种文件到底是什么？

  - STL（Stereolithography）

    - 最初为3D打印/快速成型设计的格式。
    - 本质：一堆三角形面片坐标（几何表面）。
    - 没有单位、没有层级、没有材质、没有骨骼动画……就是“定死的三角网格”。
    - 优点：简单、可预测、解析器很少踩坑。
    - 缺点：信息少（比如颜色、材质、层级关系都没有）。
  - DAE（COLLADA）

    - 数字资产交换格式，功能很全。
    - 本质：一个“场景”描述，可以包含网格、层级（父子节点）、变换（平移/旋转/缩放）、单位（）、坐标系（Y-Up/Z-Up）、材质、动画、相机、灯光等。
    - 优点：表达丰富，适合 DCC（Maya/Blender）与引擎之间交换。
    - 缺点：太灵活，解析器要做很多决定；不同软件对单位、轴系、缩放、层级的处理可能不一致，容易“出岔子”。

二、几个关键概念（用直白比喻）

  - 顶点坐标（Vertices）

    - 就是网格上每个点的三维位置。STL 直接存最终位置；DAE 可能存“原始网格 + 节点变换”。
  - 三角化（Triangulation）

    - 复杂多边形拆成三角形。碰撞库（如 FCL）更擅长处理三角形。STL 生来就是三角；DAE 可能含四边形、NGon，依赖导出/导入时再三角化，过程存在差异。
  - 单位与缩放（Units & Scale）

    - DAE 可以在文件里声明“1单位=1厘米/1米”，还可以给任意节点加缩放（scale）。如果导入方处理不一致，就会出现“东西被放大/缩小两次”或“单位被误解”的问题。
    - STL 没有单位，数值是多少就是多少，通常大家默认“这些数值就是米或按URDF里统一的单位来解释”，更少歧义。
  - 轴系（Up Axis）

    - 有的软件认为“Z向上”，有的软件认为“Y向上”。DAE 会标注，也可能通过节点变换“纠正”。如果处理不一致，模型会被“侧躺/倒置/偏移”。STL 通常在导出时已经“烘焙好”，不再需要额外纠正。
  - 层级与变换（Hierarchy & Transforms）

    - DAE 支持父子节点，子节点会继承父节点的平移/旋转/缩放。多个层级叠加，容易在导入时某一层“多算/少算”的问题。
    - STL 没有层级，所有顶点坐标都是“绝对结果”。
  - 包围盒（AABB）

    - 碰撞检测的第一步常用“轴对齐包围盒”。如果一个模型包含“隐藏/很薄/偏远”的子网格（DAE 里很常见，比如辅助线、标记），AABB 会被拉大，带来“明明看起来没碰，却先过了粗检”的错觉。

三、为什么你的场景下，STL 更准确？

  - 避免单位与缩放的歧义

    - DAE：文件里可能有 ，节点还有 scale=0.001/100 之类。URDF/Isaac/MoveIt 的读取链路如果对这些缩放理解不完全一致，就会出现“明明看起来远，算法却以为近”的现象。
    - STL：没有单位也没有层级缩放，导出时就把一切变换“烘焙”进顶点坐标，后端只需按点坐标做碰撞，出错面小得多。
  - 扁平化坐标系与层级

    - DAE：可能是 Y-Up 或 Z-Up，再叠加父子节点的 transform/matrix。稍有差错就出现整体偏移或倾斜。
    - STL：导出后就是固定坐标，无需再应用层级变换，所以更稳定。
  - 清理“隐形几何”和退化面

    - DAE：可以包含隐藏子网格、极薄面、辅助对象。很多导入器在“粗检阶段”会把这些也算上，导致 AABB 膨胀或面级误判。
    - STL：常见导出流程会只保留选中的可渲染网格；同时会把网格三角化、去退化面，FCL 等库在三角网格上更稳。
  - 与碰撞库（FCL）的匹配度更高

    - FCL 针对三角网格的稳定性较好。DAE 如果带有多边形、重合面、法线/拓扑异常，会带来数值上的“近距即碰”假阳性；STL 导出往往已经三角化并去掉异常。

四、这不代表 DAE 一定“不准确”

  - DAE 完全可以很准确，但需要“严格的导出和导入规范”：

    - 导出前“应用/烘焙”所有变换（Apply Transform）。
    - 统一单位（比如全部以米），并在导出器和导入器中确认单位一致。
    - 统一 Up 轴（例如全部 Z-Up），并在导出器里做轴系转换，导入时禁用再次转换。
    - 删除辅助/隐藏/极薄几何，只保留用于碰撞的干净网格。
    - 三角化并去退化面，检查法线和重复顶点。
    - 在单独工具里验证 AABB 与体素化碰撞近似，确保和视觉一致。
  - 但对机器人碰撞来说，我们更强调“确定性与简洁”。STL 恰好是一种“把复杂性提前在内容制作阶段解决”的格式，落地时几乎没有可变因素，所以在你的流水线里显得更“准确”。

五、给你的实操建议（面向初学者）

  - 如果目的是“碰撞几何”：

    - 优先用 STL。它简单、稳定、可复现，避免单位/轴系/层级坑。
    - 使用低到中等分辨率的 STL（减少计算量），但确保形状关键特征不被过度简化。
    - 如果视觉渲染用 DAE/GLTF，碰撞仍可用 STL（在 URDF/XACRO 中分开指定 visual 和 collision）。
  - 如果必须用 DAE：

    - 在 Blender/Maya 中：
      - 应用所有变换（位置/旋转/缩放全部“烘焙”到网格）。
      - 统一单位为米；检查导出设置里的 = meter。
      - 统一坐标系，建议最终导出为 Z-Up，与目标引擎预期一致。
      - 删除隐藏/辅助网格；只保留碰撞需要的简化几何。
      - Triangulate（全三角化）、Remove Doubles（焊点）、Merge by Distance、清理非流形/退化面。
    - 在导入侧（Isaac/URDF/MoveIt）：
      - 禁用重复的单位/轴系转换（防“双重处理”）。
      - 检查可视与碰撞的 mesh 是否对齐（在同一坐标基准下）。
      - 用工具打印网格的实际包围盒尺寸，确认与预期一致。
  - 快速自检清单（出问题时先看这 6 点）

    1. 网格单位：这个模型的一条已知尺寸（比如 10 cm 边）导入后是不是 0.1 m？
    2. 轴向：模型朝向与世界坐标一致吗（没有侧躺/倒置）？
    3. 缩放：有没有残留 scale ≠ 1 的节点？
    4. 层级：是否有父节点还在施加变换？
    5. 几何：是否存在超薄片、孤立小片、远离主体的残片？
    6. 三角化：是否已三角化并去除退化面/重合面？

六、一句话总结

  - STL 就像“烤熟的几何”，简单直接，碰撞库吃起来不挑食。
  - DAE 像“带调味和摆盘的菜”，表达力强，但端到端的处理链要完全一致，否则容易“盐放重了”。
  - 在机器人碰撞场景里，为了减少歧义与不确定性，STL 往往更稳、更准。

在rviz中, 调整机械臂姿态到目标为止, 然后Plan, Plan完成后, 使用MotionPlanning - Trajectory Slider, 将轨迹帧定位到最后一帧, 并在MotionPlanning -> PlannedPath -> Links中, 找到gripper_base, 获取定位信息 (注意: 整个rviz的参考坐标系需要改成world): 

![image2025-10-29 16:13:18.png](/assets/01KJC06PG4RM9T7WFDEN5HPZPN/image2025-10-29%2016%3A13%3A18.png)

验证, 使用如下命令发布一个TF, 并在rviz中, 显示这个TF (gripper_tcp), 验证位置正确: 

```
ros2 run tf2_ros static_transform_publisher \
  --x 0.12443 --y -0.12745 --z 0.10985 \
  --qx -0.016581 --qy -0.021837 --qz -0.70438 --qw 0.70929 \
  --frame-id world \
  --child-frame-id gripper_tcp
``` 

commit: fb7dafffe0d19fe0cb1ad61c6ceede153f190520

已经能正确将夹爪移动到圆柱体上方

commit: d6417fe583e8c1ab1f670b3f8f5ee4315af24ef4

在夹爪移动到圆柱体上方后, 使用笛卡尔直线, 让夹爪下降, 然后用attach让夹爪拿起圆柱体. 

现在的问题是: 使用笛卡尔直线时, 夹爪只能下降一定比例后就停止了. 

  - 即使将avoid_collisions=False, 夹爪也不能完整下降, 也就是说夹爪停止下降并不是因为碰撞, 而是因为关节到极限了. 
  - 将夹爪移动到圆柱体上方时, 高度放低, 让其离圆柱体尽量近, 这样夹爪下降时, 机械臂关节不容易到极限. 
  - 将来需要解决: 如何避免机械臂关节到极限

下一步: 

  - 将avoid_collisions=True, 夹爪又无法下降了, 需要检查碰撞
  - 进行反复规划时, 有一定概率夹爪无法下降 (机械臂移动规划时, 不是每次的动作都相同, 有些姿势容易到极限)
    - 将来需要解决: 需要更新算法
  - Isaac Sim的仿真中, 夹爪可以移动到圆柱体上方, 但对于夹爪动作, 就完全没有仿真

让夹爪进行仿真: 

  - 添加Physical Material, 并配置摩擦系数, 并将该Material应用到 夹爪和圆柱体的材料 (见下面"整个场景的USDA的重要参数分析")
    - Material只能应用在夹爪最外两个手指, 应用到其他手指, 会导致计算崩溃 (手指与基座间的摩擦系数变了, 导致计算崩溃? )
  - 发现新的问题: 夹爪一旦碰到圆柱体, 就会产生崩溃

诊断过程: (结论: 对夹爪内部进行分组, 组间不计算碰撞, 计算可恢复正常)

好的，这是更新后的版本，其中包含了你执行的具体操作步骤。

### 关键进展与发现（按你的操作顺序）

1\. **确认问题与接触相关**  
\- **你的操作**：在场景中移走或禁用圆柱体的碰撞，然后执行夹爪闭合。  
\- **你的发现**：“我确认没有圆柱体时, 夹爪能正确关闭”  
\- **结论**：排除了夹爪自身驱动/模型问题，锁定为“接触”是崩溃的触发条件。

2\. **确认问题不在于“被抓物体”**  
\- **你的操作**：  
1\. 将你的圆柱体 `Rigid Body` 组件的 `Enabled` 勾选去掉，使其变为 `Static`。  
2\. 隐藏或删除你的圆柱体，在相同位置 `Create -> Physics -> Cylinder` 创建一个内置的 `Primitive Cylinder`。  
\- **你的发现**：“将圆柱体设置为static, 仍然会爆”，以及后续“用内置 Primitive Cylinder, 仍然会爆”  
\- **结论**：排除了圆柱体资产的物理属性是根本原因，进一步将问题收敛到“机器人侧”。

3\. **确认问题根源是“指尖碰撞”**  
\- **你的操作**：在 Stage 视图中选中指尖 `prim`（如 `gripper_left1` 等），在 `Property` 面板中找到 `Collider` 组件并取消勾选 `Collision Enabled`。  
\- **你的发现**：“我禁用了指尖的碰撞, 计算就正常了”  
\- **结论**：这是最关键的一步，它将问题范围从整个机器人缩小到了“指尖的碰撞几何/参数”上。

4\. **确认不稳定的源头是“自碰撞”**  
\- **你的操作**：选中机器人的 `Articulation Root`，在 `Property` 面板中找到 `PhysX Articulation Root` 属性，并取消勾选 `Enable Self Collisions`。  
\- **你的发现**：“关闭整机自碰撞以后, 计算就正常了”  
\- **结论**：在“指尖碰撞”的基础上，进一步精确定位到是“指尖与其他部分（包括指尖之间或与手掌）的自碰撞”导致了数值发散。

5\. **通过局部屏蔽自碰撞获得初步稳定**  
\- **你的操作**：创建了三个 `Physics Collision Group`，将左指、右指、手掌的 `prim` 分别加入，并在每个组的 `Filtered Groups` 关系中互相指定，实现两两屏蔽。  
\- **你的发现**：机械臂可以与圆柱体轻微接触而不立即崩溃。  
\- **结论**：找到了一个工程上可行的解决方案，即通过精确屏蔽内部碰撞对，使得“指尖与外部物体的接触”不再立刻引发崩溃，验证了诊断方向的正确性。

现在可以正确拿起圆柱体:

  - commit: 1b054462b4df450a439822472606fe2fe345c948
  - ![image2025-10-31 14:18:34.png](/assets/01KJC06PG4RM9T7WFDEN5HPZPN/image2025-10-31%2014%3A18%3A34.png)
  - 并且已经将脚本中改回avoid_collisions=True, 因为夹爪的分组策略设置了不检测碰撞, 所以这里已经可以正确运行
  - 发现的问题: 
    - 机械臂进入预定位置后, 不一定能继续下降 (在同一个z轴上, 关节达到极限)
    - 机械臂抓起圆柱体后, 不一定能继续上升 (在同一个z轴上, 关节达到极限)
    - 如果使用路径规划 (而不是笛卡尔曲线), 那么机械臂调整姿态时, 会将夹爪翻转过来, 导致圆柱体翻转
  - 判断你需要解决前两个问题, 否则之后的实验的成功概率低于25%

整个场景的USDA的重要参数分析: 

### 物理相关关键信息（按层级归纳）

\- 材质 `/Item_00/World/PhysicsMaterial`  
\- physics:staticFriction=1.2  
\- physics:dynamicFriction=1.5  
\- physxMaterial:frictionCombineMode=max  
\- physxMaterial:restitutionCombineMode=max  
\- 绑定关系：  
\- 绑定到：`/World/Cylinder`、`gripper_left1.collisions`、`gripper_right1.collisions`  
\- 其余指尖碰撞处显式解绑 `material:binding=None`

\- 机械臂根 `/Item_00/World/mycobot_280_m5_adaptive_gripper`  
\- physxArticulation:enabledSelfCollisions=1

\- 链路覆盖（多个 link 上有同名覆盖）  
\- 多个链接含 `float physics:mass = 0`：`joint1`、`joint2`、`joint3`、`joint4`、`joint5`、`joint6`、`joint6_flange`、`gripper_base` 等  
\- 多处 `physics:collisionEnabled = 1`（各段 `over ".../collisions"`）

\- 场景地面  
\- `GroundPlane.CollisionPlane`：PhysicsCollisionAPI（静态碰撞平面，Z 轴）

\- 圆柱体（作为被抓物）`/Item_00/World/Cylinder`（Mesh）  
\- apiSchemas：PhysicsRigidBodyAPI, PhysxRigidBodyAPI, PhysicsCollisionAPI, PhysxCollisionAPI, PhysxConvexHullCollisionAPI, PhysicsMeshCollisionAPI, MaterialBindingAPI  
\- physics:rigidBodyEnabled=1  
\- physics:kinematicEnabled=0  
\- physics:collisionEnabled=1  
\- physics:approximation=convexHull  
\- material:binding=PhysicsMaterial  
\- 注意：同时存在 ConvexHull 与 MeshCollision API（动态体不应使用三角网格碰撞）

\- 圆柱体（备用）`/Item_00/World/Cylinder_01`（Cylinder primitive）  
\- active=false（未启用）  
\- physics:rigidBodyEnabled=1  
\- physics:collisionEnabled=1  
\- physxRigidBody:disableGravity=0

\- 碰撞组（已定义并互斥）  
\- `CollisionGroup_left_fingers` includes：left1/left2/left3；filteredGroups：palm、right_fingers  
\- `CollisionGroup_right_fingers` includes：right1/right2/right3；filteredGroups：palm、left_fingers  
\- `CollisionGroup_palm` includes：`joint6_flange`、`gripper_base`；filteredGroups：left_fingers、right_fingers

\- 关节从属/联动  
\- 多处 `physxMimicJoint:rotZ:referenceJoint` 指向 `gripper_base/gripper_controller`（用于指尖联动）

使用多IK候选: 

  - commit: b5ce0b3156a23748498047a5acfa7a6260d52a40
    - 说明在代码中: 智能IK选择说明.md
  - 现在的路径规划, 输入是(各关节的初始配置 + 末端位姿), 输出是各关节配置
  - 多IK规划的算法: 
    -       - (随机生成一组关节配置 + 末端位姿) 作为输入, 求得一组输出. 目的是对于同一末端位姿, 拿到多组可能的关节配置
        - 随机生成时, 避免关节处于极限位置
        - 对于生成的关节配置, 需要去重 (去重逻辑是 关节角度差异总和小于阈值)
    - 对于每组输出, 进行笛卡尔下降算法评估, 选择效果最好的
    - 进行实际执行

能正确完成下降/抓取/上举操作. 

想让机械臂举着水杯做一定移动, 但找不到合法的目标位置, 在保持杯口向上的状态下, 所有IK规划都失败

  - 6轴机器人的活动度受限?
  - 先放弃这一步的探索

进行倾倒动作: 

  - 原本是想让机械爪围着世界坐标系的一根轴 (将来替换成杯坐标系的一根轴), 进行一定角度的分步的旋转, 求得IK路径, 经过一天尝试, IK规划是失败的. 
    - IK的输入是每一步的最终姿态, IK规划失败, 无法完成
  - 也尝试过生成一条多点轨迹, 让IK进行规划, 仍然失败
  - 最终使用commit: 49216cbcfe9ea34b4c2c178f9859876da9c9dfe2
    - 指定旋转机械臂最后一节的机械轴到某个角度 (分步进行, 以关节目标状态作为IK的输入)

下一步创建粒子系统来模拟倒水

  - 整个场景需要调整到正常尺寸 (现在缩放为0.03)
    - 尺寸测量工具: <https://docs.omniverse.nvidia.com/extensions/latest/ext_measure-tool.html>
  - 在仿真中, 增加Physical -> ParticleSystem, 需要修改 Particle Contact Offset = 0.005, 否则粒子半径太大
  - 添加一个Mesh (Cube), 针对这个Mesh, Add -> Physical -> Particle Sampler, 会多出一个ParticleSet (将cube转成了粒子群)
  - 一个新问题: 粒子群放在杯子里, play后, 不会进入杯子, 而是和杯子碰撞分离
    - 将杯子的碰撞模型, 改成 Collider -> Approximation -> Convex Decomposition
  - commit: c19d1d3631373221b44fc38ea40bc0b006a67e2c
    - 已经能正确用粒子实现了倒水动作

# 整理发现的问题

  - 六轴机械臂, 其很容易达到锁死点 (奇异点?). 尤其是拿起杯子以后, 需要保持杯口向上, 于是其活动度受限很厉害
  - 夹爪的压力, 并不得控制, 在现实使用中, 可能会夹碎杯子
  - 有一定概率, 在杯子上升/倾倒的过程中, 杯子被夹爪挤出来
  - 使用粒子系统模拟倒水的动作, 其杯中重力变化, 并没有被仿真出来
  - 使用路径规划完成一个小的移动, 会发生绕一圈的情况 (来解除活动限制). 使用笛卡尔曲线, 又会受到活动限制

# 下一步

  - 搞清楚机械臂的轴数和活动度的影响, 购买实际机械臂, 进行对齐
  - 在仿真环境, 接入位姿感应, 从而替换写死的杯子位置

# 参考

![image2025-10-24 14:54:16.png](/assets/01KJC06PG4RM9T7WFDEN5HPZPN/image2025-10-24%2014%3A54%3A16.png)

SRDF和URDF的区别: 

![image2025-10-27 0:47:46.png](/assets/01KJC06PG4RM9T7WFDEN5HPZPN/image2025-10-27%200%3A47%3A46.png)
