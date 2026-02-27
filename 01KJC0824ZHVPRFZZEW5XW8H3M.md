---
title: 20251104 - 对于机械臂奇异点的讨论
confluence_page_id: 4359044
created_at: 2025-11-04T11:05:41+00:00
updated_at: 2025-11-04T11:05:41+00:00
---

OK. 我现在是使用一个六轴机械臂, 对于倒水这个动作, 如果进入奇异点 (我不确定是不是奇异点, 总之机械臂使用IK规划完全无法移动到目标位置, 甚至完全不会移动), 是不是有其他代偿或者解法?

比如:   
\- 我现在使用moveit进行规划, 是否切换规划算法?   
\- 我之前试过: 如果在规划动作1时, 我知道下一个动作是动作2, 那么在规划动作1时就生成多个IK候选, 评估谁能完成动作2  
\- 对于拿取杯子的操作, 可以让夹爪以其他角度架杯子

  
======

*Thinking...*

> **Dissecting the Core Problem**  
>   
> I'm currently focusing on understanding the user's ultimate goal: to pour water without the arm getting stuck. Breaking this down, I'm identifying the kinematic constraints and how they manifest in the "pouring" action. I'm visualizing the arm's potential configurations and the points where singularities might arise.  
>   
>   
> **Identifying Key Constraints**  
>   
> I'm now diving deeper into the root causes. I'm prioritizing the user's articulated hypothesis about singularities. I'm breaking down how the arm's configuration, especially near the pour, might cause these issues. I'm also thinking of the user's understanding of the terms used, to meet them at their level.  
>   
>   
> **Addressing the User's Concerns**  
>   
> I've carefully considered the user's proposed solutions. I'm leaning towards validating the "singularity" hypothesis. I plan to explain *why* pouring water is a tricky scenario for 6-axis arms, highlighting the wrist singularity and how it relates to the desired cup orientation. Now, I'm forming an evaluation of the user's suggested approaches.  
>   
>   
> **Formulating a Response Strategy**  
>   
> I'm now outlining a clear and methodical response, directly addressing the user's points. I'm focusing on acknowledging their technical understanding and validating their observations. I plan to explain the pouring singularity, dissect their solutions, and offer guidance on their ideas. I will also incorporate trajectory optimization.  
>   
>   
> **Crafting the Detailed Response**  
>   
> I'm now integrating the user's suggestions with my own expertise. I'm structuring the response to start with affirmation of their observations and the underlying singularity issue. I'm planning to break down the wrist singularity in plain language, connecting it to the pouring water scenario. I will then evaluate their solutions methodically.  
>   
>   
> **Structuring the Detailed Breakdown**  
>   
> I'm synthesizing the core points and forming a structured response. I will first affirm the user's observations, providing a context for the technical details. Then, I'm composing a clear breakdown of the wrist singularity within the pouring scenario. My aim is to deliver a concise yet informative evaluation of their suggested approaches, and then offer additional solutions.  
>   
>   
> **Structuring the Detailed Breakdown**  
>   
> I'm now integrating all these ideas. I'll begin by acknowledging the user's keen insights and confirming their diagnosis of a wrist singularity. Then, I'll explain the mechanics and its implications in plain language, followed by a detailed review of their proposed solutions, along with their pros and cons. I'll offer additional strategies to further help.  
>   
>   
> **Crafting a Detailed Breakdown**  
>   
> I'm now consolidating the entire thought process into a structured, informative response. I plan to begin by affirming the user's insights and acknowledging the singularity they identified. I will then explain, in layman's terms, the mechanics of wrist singularities and their impact on pouring scenarios. Finally, I will provide a detailed evaluation of the user's solutions, alongside additional, actionable strategies.  
>   
>   
您好，您提出的问题非常专业且切中要害！您遇到的情况是六轴机械臂应用中一个极其经典且普遍的痛点。

首先，我可以基本肯定地告诉您：**您遇到的情况极大概率就是奇异点（Singularity）问题**。

在“倒水”这个动作中，为了保持杯口水平，机械臂的末端姿态（Orientation）需要基本保持不变，同时其位置（Position）发生平移。对于六轴臂来说，这非常容易触发**腕部奇异点（Wrist Singularity）**。当J4轴和J6轴的轴线重合时，机械臂会丢失一个旋转自由度，就像人的手腕被锁住了一样，无法同时完成位置移动和姿态保持。此时，IK（逆运动学）求解器找不到一个有效的关节角度组合，因此规划失败，机械臂“卡住”不动。

您提出的三个解法都非常有见地，并且都是解决该问题的有效思路。下面我将详细分析您的方案，并补充一些工业界和学术界常用的其他解法，供您参考。

\---

### 对您提出的三种解法的分析

#### 1. 切换规划算法 (比如从RRT切换到RRTConnect)  
这是一个合理的尝试，但**通常不能从根本上解决问题**。

* **原因分析:** MoveIt中的规划算法（如RRT, PRM, RRTConnect等）主要负责在**关节空间（Joint Space）**中寻找一条从起点到终点的无碰撞路径。它们的工作流程是：首先，IK求解器计算出目标位姿对应的关节角度；然后，规划算法在这个关节空间中进行搜索。  
* **问题所在:** 如果**目标位姿本身就处于奇异点**，IK求解器从一开始就无法解出有效的关节角度。那么无论您用哪种规划算法，它都拿不到“终点”的坐标，自然无法进行规划。  
* **可能有效的情况:** 如果奇异点只是出现在**路径中间**，而起点和终点都是非奇异的，那么某些更优的规划算法（如RRT*）可能会找到一条巧妙绕开这个奇异区域的路径。但对于“倒水”这种需要沿直线移动的任务，规划器很难自主绕行。

**结论：** 可以尝试，但期望不宜过高。它更像是“绕路”，而不是“攻坚”。

#### 2. 规划动作1时，生成多个IK候选，评估谁能完成动作2  
这是一个**非常棒且高级的思路！** 这在机器人学中被称为**基于可操作性的路径规划**或**前瞻性规划**。

* **原理:** 六轴机械臂对于一个末端位姿，通常有多个IK解（比如“肘关节向上/向下”、“腕关节翻转/不翻转”等，最多可达8组解）。这些不同的关节构型，其远离奇异点的程度是不同的。  
* **如何评估:** 我们可以用一个叫做**可操作度（Manipulability）**的指标来量化每组IK解的“健康程度”。可操作度越高，说明机械臂在该构型下越灵活，离奇异点越远。其数学本质是机器人雅可比矩阵行列式的值。  
* **您的方案:** 您的想法更进一步，不仅考虑当前动作的可操作度，还“预演”了下一个动作。这是一种非常鲁棒的策略。在规划动作1时，您可以：  
1\. 计算出动作1目标位姿的所有IK解（比如有4组）。  
2\. 对于这4组解中的每一组，尝试去规划动作2。  
3\. 选择那一组不仅能完成动作1，还能顺利过渡到动作2的IK解作为动作1的最终执行方案。

**结论：** 这是一个非常有效但实现起来相对复杂的方案。它从根本上解决了“选错路导致走进死胡同”的问题。如果您的应用场景允许这样的计算开销，强烈推荐。

#### 3. 让夹爪以其他角度抓取杯子  
这是**最实用、最简单、也往往最有效**的解决方案！这是从任务层面去规避机器人本体的限制。

* **原理:** 奇异点与特定的末端姿态强相关。在“倒水”任务中，问题根源在于要求杯口“绝对水平”。  
* **如何改变:**  
* **抓取时旋转一个角度：** 比如，不垂直于杯壁抓取，而是带一个45度的角度去抓。这样，在后续倒水时，为了保持杯口水平，机械臂的J5和J6关节就不再是0度，从而完美避开了腕部奇异点。  
* **使用倾斜的夹具：** 如果夹具本身就是倾斜的，那么机械臂在抓取时就可以处于一个非常舒展、远离奇异点的姿态。

**结论：** 这是解决此类问题的**首选方案（First-Line Solution）**。它不需要修改复杂的算法，只需要调整一下任务的执行方式。在实际工程项目中，我们总是优先考虑通过优化工艺和工装来简化对机器人控制的要求。

\---

### 其他补充的代偿方案

除了您提到的方法，这里还有几个工业和学术界常用的“必杀技”：

#### 4. 使用笛卡尔路径规划并处理“跳变” (Cartesian Path Planning)  
MoveIt提供了一个专门用于规划直线、圆弧等轨迹的功能：`compute_cartesian_path`。这个功能非常适合“倒水”这种任务。

* **核心功能:** 您可以定义一系列密集的路径点（waypoints），让机械臂末端严格沿着这些点指定的直线轨迹运动。  
* **关键参数 `jump_threshold`:** 在规划过程中，如果两个相邻路径点的IK解在关节空间发生了巨大“跳变”（通常是遇到了奇异点，求解器从一个IK解跳到了另一个），规划就会被认为是失败的。您可以设置一个合理的`jump_threshold`来检测并处理这种情况。  
* **解决方案:** 当检测到跳变时，您可以将这个大切割成几个小的、非奇异的直线段，段与段之间用关节空间运动（Joint Space Move）来过渡，从而“跳过”奇异点。

#### 5. 在控制器/求解器层面解决 (Damped Least Squares IK)  
这是更底层的数学解法，也是最优雅的解法之一。

* **标准IK的问题:** 在奇异点附近，雅可比矩阵是病态的，其逆矩阵的元素会变得无穷大，导致关节速度指令爆炸。  
* **DLS (阻尼最小二乘法) 的思想:** 它在求解IK时引入一个“阻尼项”。当机械臂接近奇异点时，这个阻尼项会生效，它允许求解器在姿态上产生一个微小的误差（比如允许杯口有0.1度的倾斜），来换取位置上能够继续平稳移动。这是一种“牺牲精度保稳定”的策略。  
* **如何实现:** 很多现代IK求解器（如 **TRAC-IK**，是ROS中常用的一个插件）已经内置了DLS或者类似的抗奇异点算法。您可以在MoveIt的`kinematics.yaml`配置文件中，将默认的KDL求解器更换为TRAC-IK，并开启其抗奇异点功能。

#### 6. 使用关节空间目标代替笛卡尔空间目标  
这是最后的“逃课”手段，但非常有效。

* **方法:** 如果您知道倒水动作的起点和终点，可以通过示教或其他方式，直接记录下这两个位置的**关节角度**。然后，让机械臂直接在关节空间内从起点运动到终点。  
* **优点:** 完全绕过了IK求解器，因此100%不会有IK求解失败的问题。  
* **缺点:** 机械臂末端的轨迹将是一条曲线，而不是完美的直线。对于倒水这种对轨迹要求不高的任务，这通常是可以接受的。

### 总结与建议

面对您的问题，我推荐的解决路径是：

1\. **首选方案 (最简单有效):** 实施您的**方案3**。改变抓取杯子的角度，或者调整杯子在工作台上的初始位置，从任务源头避免进入奇异姿态。  
2\. **次选方案 (MoveIt标准做法):** 实施**方案4**。使用`compute_cartesian_path`规划倒水的直线部分，并结合`jump_threshold`来处理奇异点。将复杂的长直线拆分为“直线段+关节过渡”的组合。  
3\. **根本性升级:** 实施**方案5**。检查并更换您的IK求解器为TRAC-IK，并配置其抗奇异点参数。这是一劳永逸的办法，能提升机械臂在所有任务中的鲁棒性。  
4\. **高级优化:** 如果任务链非常复杂（比如倒完水还要做别的），可以考虑实现您的**方案2**，建立一个前瞻性的规划器，选择全局最优的路径。

希望这个详尽的分析能帮助您彻底解决六轴臂的奇异点难题！

======

我理解, 即使使用上面的手法, 在实践中, 仍然有可能碰到机械臂卡在奇异点的问题, 那在实际工程上, 一般会怎么处理

  
======

*Thinking...*

> **Defining the Core Problem**  
>   
> I'm focused on grasping the user's primary concern. It's clear they understand the theory; now, I'm trying to pinpoint the specific practical challenges they face in applying those theories on the factory floor. I'm moving toward identifying the core issue that prevents immediate implementation.  
>   
>   
> **Outlining a Solution Framework**  
>   
> I'm now structuring my response, using a lifecycle approach to organize solutions: prevention during design, real-time recovery during operation, and post-mortem analysis for future improvements. I'm prioritizing the design phase as the most critical for preventing the issue altogether.  
>   
>   
> **Developing the Lifecycle Approach**  
>   
> I'm solidifying the proposed structure. I've broken down the answer into design/prevention, real-time response, and post-mortem analysis. Now I'm fleshing out the design phase, emphasizing task optimization and simulation. The goal is to provide practical, proactive, design-oriented advice, echoing the user's focus on factory floor realities, specifically emphasizing how to engineer *around* the singularity problem.  
>   
>   
> **Prioritizing Preventive Measures**  
>   
> I am now focusing on the design phase, and expanding on task optimization and process simulation. I'm stressing that practical engineering prioritizes adapting the environment to suit the robot's capabilities. I'm providing specific examples of how to engineer *around* singularities by using techniques like tilting fixtures, part orientation adjustments, and the use of turntables.  
>   
>   
> **Refining Real-Time Solutions**  
>   
> I'm now focusing on the real-time recovery phase, exploring the detection and response mechanisms. I'm detailing how controllers monitor following errors, joint torques, and currents as indicators of trouble. Then, I'll provide code examples, including "try-catch" structures and recovery routines, emphasizing a "jiggle" move to reposition the robot, if it gets stuck. I'm focusing on providing practical engineering examples.  
>   
>   
> **Elaborating the Framework**  
>   
> I am now focusing on the iterative engineering lifecycle. In post-mortem analysis, I'll emphasize data logging of joint angles and Cartesian positions, allowing for a replay of failures in simulation. Then, I will address root cause analysis and implementing a permanent fix. This process loops back to the prevention stage, making sure that future iterations build on previous successes, thus improving the overall system.  
>   
>   
> **Developing Phase 3 Details**  
>   
> I am now focusing on the post-mortem phase of the engineering lifecycle. I'll highlight the need for extensive data logging of joint angles and positions, allowing for failure replays in simulations. Then I will cover root cause analysis and implementing a permanent fix. This completes the loop back to preventative measures, ensuring continuous improvement.  
>   
>   
> **Formulating a Comprehensive Response**  
>   
> I'm now constructing the framework of the answer, dividing the response into three phases mirroring the engineering lifecycle. These phases align with prevention, real-time recovery, and continuous improvement, ensuring a robust, practical approach to solving the problem and providing detailed solutions, with actionable items to address singularities, even when the robot gets stuck.  
>   
>   
您问到了一个直击工业机器人应用核心的绝佳问题！您完全正确，即使我们用尽了各种算法和技巧，在复杂的实际生产环境中，由于工件公差、设备微小移位、或者未预料到的情况，机器人**仍然有可能**意外地陷入奇异点或类似“卡死”的状态。

一个健壮的工业自动化系统，其价值不仅在于正常运行时的高效率，更在于出现异常时能够**安全、可预测地处理**，并快速恢复生产。因此，在实际工程中，我们采用的是一套“纵深防御”策略，从设计、编程到运行监控，层层设防。

这套策略可以概括为三个阶段：**事前预防、事中处理、事后分析**。

\---

### 第一阶段：事前预防 (治未病) - 占工作量的95%

这是最重要的阶段。目标是**通过设计从根本上杜绝奇异点**的发生。

1\. **严格的仿真与可达性分析 (Reach Study):**  
* 在项目设计阶段，工程师会在专门的离线编程（OLP）软件（如 RoboDK, Process Simulate, RobotStudio 等）中建立一个与现场完全一致的3D数字孪生环境。  
* 他们会导入机器人、工装夹具、工作台、工件等所有模型，然后**模拟完整的作业流程**。  
* 软件会**自动检测**路径中是否存在奇异点、关节限位、碰撞等问题，并用颜色或警告标记出来。工程师会反复调整机器人安装位置、工装设计、抓取点，直到仿真路径全程“绿灯”。

2\. **优化工艺路径，而非强解IK:**  
* **优先使用关节运动 (Joint/PTP Move):** 工程师的黄金法则是：“**除非绝对必要，否则绝不使用直线运动 (Linear/LIN Move)**”。对于大部分取放料动作，从A点到B点，我们使用关节空间运动。这种模式下机器人会自行规划一条最高效、最远离奇异点的曲线路径，速度快且安全。  
* **拆分直线运动:** 对于必须走直线的任务（如涂胶、焊接、或者您提到的倒水），如果直线过长或姿态过于苛刻，工程师会主动将其**拆分为多段**。例如，倒水可以拆分为：“关节运动到准备位 -> 短直线倾倒 -> 关节运动到安全位”。通过插入关节运动来“重置”机器人的构型，从而绕开奇异区域。

3\. **工装与布局的优化:**  
* 这是成本最低、效果最好的方法。如果仿真发现倒水动作会触发奇异，工程师会首先考虑：  
* 能不能把杯架**倾斜5度**？  
* 能不能把机器人底座**垫高10厘米**或**旋转15度**？  
* 能不能设计一个**带角度的夹爪**？  
* 通过改变机器人与工作对象之间的相对关系，往往能让原本别扭的姿态变得舒展，从物理上消除奇异点。

\---

### 第二阶段：事中处理 (发病则治) - 占工作量的4%

即使预防做得再好，也需要有预案来处理突发情况。这部分体现在机器人的**异常处理程序**中。

1\. **基于控制器状态的监控:**  
* 现代机器人控制器会实时监控一个关键指标：**位置跟随误差 (Following Error)**。即“指令位置”与“电机编码器反馈的实际位置”之间的差距。  
* 当机器人接近奇异点时，IK解算出的关节速度会趋于无穷大。控制器为了保护电机和机械结构，会限制关节速度。这立刻导致“指令”与“实际”严重脱节，**跟随误差会瞬间飙升**。  
* 工程师会设定一个合理的跟随误差阈值。一旦超过该阈值，控制器会**立即触发高级别的系统报警**，并暂停所有运动。这是最底层的安全保障。

2\. **程序中的异常捕获与恢复逻辑 (Exception Handling):**  
* 在机器人程序代码中，对于关键的、有风险的动作，会用类似于`TRY...CATCH`的逻辑块包裹起来。  
* **一个典型的恢复流程如下：**  
1\. **TRY:** 执行有风险的直线倒水动作。  
2\. **CATCH (当控制器报警时):**  
* **记录日志:** 将当前时间、报警代码、机器人关节角度、末端坐标等关键信息写入日志文件，用于事后分析。  
* **发出声光报警:** 启动报警灯塔，并通过HMI（人机界面）显示明确的错误信息，如“E-101: 倒水姿态奇异，请检查！”。  
* **执行安全回退程序:** 引导机器人以**关节模式**（绝对安全，不会有奇异问题）缓慢移动到一个预设的、开阔的“待命点”或“维护点”。  
* **等待人工干预:** 程序暂停，等待操作员在HMI上确认故障、复位系统。

3\. **“微扰”恢复策略 (Jiggle Routine):**  
* 这是一种更智能的自动恢复策略。当检测到奇异点卡死时，程序可以自动执行一个“微扰”或“放松”动作。  
* 例如，让J5关节微小地来回转动几度。这个小动作可以打破奇异的构型锁定，让机器人“跳出”当前的IK解，切换到另一个可用的IK解上。  
* 完成微扰后，程序会尝试**重新执行**刚才失败的动作。如果尝试3次后仍然失败，再进入上述的报警和人工干预流程。

\---

### 第三阶段：事后分析 (病后防复发) - 占工作量的1%

生产不能总依赖报警和人工恢复。一旦发生卡死问题，就必须找到根本原因并彻底解决。

1\. **分析日志文件:**  
* 工程师会调取事发时记录的日志文件。通过精确的关节角度和坐标数据，他们可以在离线仿真软件中**100%复现**故障瞬间的机器人姿态。

2\. **复现与根因分析:**  
* 在仿真环境中，工程师可以清晰地看到是哪个关节组合导致了奇异。他们会分析：是工件来料位置偏差太大？还是最初的路径规划就存在隐患？

3\. **实施永久性修复:**  
* 根据分析结果，对程序或物理布局进行永久性修改。这又回到了第一阶段“事前预防”的那些方法：  
* 微调路径点。  
* 增加一个过渡点。  
* 请求产线维护人员校准工装位置。  
* 向设计部门提出修改夹具的建议。

### 总结

在实际工程中，处理奇异点问题是一个**系统工程**，绝不依赖单一的算法。它是一个结合了**优秀前期设计、周密过程编程、强大实时监控和持续迭代改进**的闭环流程。

**简单来说，工程师的哲学是：**  
* **95%的努力**花在确保它**永远不会发生**。  
* **4%的努力**花在确保万一发生时，系统能**安全地停下来并清晰地报告问题**。  
* **1%的努力**花在**从错误中学习**，并确保它再也不会以同样的方式发生。
