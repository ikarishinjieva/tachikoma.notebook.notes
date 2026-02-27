---
title: 20251010 - 分析isaac_ros_stereo_image_proc的原理 (SGM生成点云)
confluence_page_id: 4358837
created_at: 2025-10-10T12:21:24+00:00
updated_at: 2025-10-13T09:56:35+00:00
---

# 安装

在Issac DEV容器中, 下载assets

```
NGC_ORG="nvidia"
NGC_TEAM="isaac"
PACKAGE_NAME="isaac_ros_stereo_image_proc"
NGC_RESOURCE="isaac_ros_stereo_image_proc_assets"
NGC_FILENAME="quickstart.tar.gz"
MAJOR_VERSION=3
MINOR_VERSION=2
VERSION_REQ_URL="https://catalog.ngc.nvidia.com/api/resources/versions?orgName=$NGC_ORG&teamName=$NGC_TEAM&name=$NGC_RESOURCE&isPublic=true&pageNumber=0&pageSize=100&sortOrder=CREATED_DATE_DESC"
AVAILABLE_VERSIONS=$(curl -s \
    -H "Accept: application/json" "$VERSION_REQ_URL")
LATEST_VERSION_ID=$(echo $AVAILABLE_VERSIONS | jq -r "
    .recipeVersions[]
    | .versionId as \$v
    | \$v | select(test(\"^\\\\d+\\\\.\\\\d+\\\\.\\\\d+$\"))
    | split(\".\") | {major: .[0]|tonumber, minor: .[1]|tonumber, patch: .[2]|tonumber}
    | select(.major == $MAJOR_VERSION and .minor <= $MINOR_VERSION)
    | \$v
    " | sort -V | tail -n 1
)
if [ -z "$LATEST_VERSION_ID" ]; then
    echo "No corresponding version found for Isaac ROS $MAJOR_VERSION.$MINOR_VERSION"
    echo "Found versions:"
    echo $AVAILABLE_VERSIONS | jq -r '.recipeVersions[].versionId'
else
    mkdir -p ${ISAAC_ROS_WS}/isaac_ros_assets && \
    FILE_REQ_URL="https://api.ngc.nvidia.com/v2/resources/$NGC_ORG/$NGC_TEAM/$NGC_RESOURCE/\
versions/$LATEST_VERSION_ID/files/$NGC_FILENAME" && \
    curl -LO --request GET "${FILE_REQ_URL}" && \
    tar -xf ${NGC_FILENAME} -C ${ISAAC_ROS_WS}/isaac_ros_assets && \
    rm ${NGC_FILENAME}
fi
``` 

安装代码: 

```
sudo apt-get install -y  ros-humble-isaac-ros-image-proc ros-humble-isaac-ros-stereo-image-proc
``` 

# 运行
    
    
    (开启Issac Sim, 并加载Sample Scene, 参考: http://8.134.54.170:8330/pages/viewpage.action?pageId=4358780)

在桌面上, 进入容器, 并运行: 

```
export ISAAC_ROS_WS=/data/huangyan/isaac_ros-dev/
cd $ISAAC_ROS_WS && ./src/isaac_ros_common/scripts/run_dev.sh
   
# 容器内
export http_proxy=http://10.186.16.136:7890
export https_proxy=http://10.186.16.136:7890
cd ${ISAAC_ROS_WS}
source install/setup.bash
export ROS_DOMAIN_ID=99
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
export DISPLAY=:0
 
# 运行
ros2 launch isaac_ros_stereo_image_proc isaac_ros_stereo_image_pipeline_isaac_sim.launch.py

# 自动弹出rviz
``` 

# 排错

报错: 

```
[rviz2-2] [INFO] [1760097138.501866262] [rviz]: Message Filter dropping message: frame 'front_stereo_camera_left_optical' at time 2.483 for reason 'discarding message because the queue is full'
[component_container_mt-1] 2025-10-10 11:52:18.532 WARN  gems/pose_tree/pose_tree.cpp@503: No pose frame found with UID 0
[component_container_mt-1] 2025-10-10 11:52:18.648 WARN  gems/pose_tree/pose_tree.cpp@503: No pose frame found with UID 0
[component_container_mt-1] 2025-10-10 11:52:18.774 WARN  gems/pose_tree/pose_tree.cpp@503: No pose frame found with UID 0

``` 

无法显示点云

诊断过程: 

### 问题根源分析

您执行的两个命令揭示了所有信息：

  1. **`ros2 topic echo /front_stereo_camera/left/camera_info | grep "frame_id"`**

     - **输出** : `frame_id: front_stereo_camera_left_optical`
     - **含义** : 您的相机数据被标记为属于一个名为 `front_stereo_camera_left_optical` 的坐标系。这是完全正常的。
  2. **`ros2 run tf2_ros tf2_echo chassis_link front_stereo_camera_left_optical`**

     - **输出** : `Invalid frame ID "chassis_link" passed to canTransform argument target_frame - frame does not exist`
     - **含义** : 这是问题的关键所在！`tf2_echo` 工具报告说，在整个TF坐标树中，**根本就不存在一个叫做 "chassis_link" 的坐标系** 。

**结论：**  
`PointCloudNode` 节点之所以持续报错 `No pose frame found`，是因为它被配置为将点云转换到 `chassis_link` 坐标系下，但是这个目标坐标系在您的仿真环境中**根本不存在** ！节点不知道 "chassis_link" 在哪里，所以它无法完成从 `front_stereo_camera_left_optical` 到 `chassis_link` 的坐标变换。

修复: 需要在rviz中, 指定坐标系为base_link: 

![image2025-10-10 20:18:18.png](/assets/01KJC03F2DVR3X6WTK4RPEFBY8/image2025-10-10%2020%3A18%3A18.png)

BTW: 并不能解决日志中的WARN, 但点云工作正常了

# 代码分析

代码: <https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_image_pipeline/blob/19c241cf45f04b3b096779b8e572f3f3af491f9e/isaac_ros_stereo_image_proc/launch/isaac_ros_stereo_image_pipeline_isaac_sim.launch.py>

```
Isaac ROS 立体视觉流水线
│
├─── Launch文件
│    └─ isaac_ros_stereo_image_pipeline_isaac_sim.launch.py
│       └─ 启动2个组件: container + rviz_node
│
├─── 【容器进程】ComposableNodeContainer
│    │
│    ├─ 容器信息
│    │  ├─ 名称: disparity_container
│    │  ├─ 命名空间: disparity
│    │  ├─ 类型: 多线程容器 (component_container_mt)
│    │  └─ 进程: 单个进程运行2个节点
│    │
│    ├─── 节点1: DisparityNode (视差计算)
│    │    │
│    │    ├─ 节点名: disparity
│    │    ├─ 完整路径: /disparity/disparity
│    │    │
│    │    ├─ 输入 (4路)
│    │    │  ├─ 左相机图像 ← Isaac Sim
│    │    │  ├─ 左相机参数 ← Isaac Sim
│    │    │  ├─ 右相机图像 ← Isaac Sim
│    │    │  └─ 右相机参数 ← Isaac Sim
│    │    │
│    │    ├─ 关键参数
│    │    │  ├─ 后端: CUDA (GPU加速)
│    │    │  └─ 最大视差: 64像素
│    │    │
│    │    ├─ 工作流程
│    │    │  ├─ 步骤1: 接收左右图像
│    │    │  ├─ 步骤2: SGM立体匹配算法
│    │    │  │   ├─ 在右图中找左图每个像素的对应点
│    │    │  │   └─ GPU并行处理
│    │    │  ├─ 步骤3: 计算视差值（像素位移）
│    │    │  └─ 步骤4: 输出视差图
│    │    │
│    │    ├─ 输出
│    │    │  └─ 视差图 → 发布到容器内的PointCloudNode
│    │    │     └─ 特点: 零拷贝传输（GPU内存共享）
│    │    │
│    │    ├─ 核心算法
│    │    │  └─ SGM (半全局匹配)
│    │    │     ├─ 作用: 找到左右图的对应点
│    │    │     └─ 输出: 每个像素的视差值
│    │    │
│    │    └─ 性能
│    │       └─ 处理时间: 约10-15ms/帧
│    │
│    └─── 节点2: PointCloudNode (点云生成)
│         │
│         ├─ 节点名: (自动生成)
│         ├─ 完整路径: /disparity/point_cloud
│         │
│         ├─ 输入 (4路)
│         │  ├─ 视差图 ← DisparityNode (容器内零拷贝)
│         │  ├─ 彩色图像 ← Isaac Sim
│         │  ├─ 左相机参数 ← Isaac Sim
│         │  └─ 右相机参数 ← Isaac Sim
│         │
│         ├─ 关键参数
│         │  ├─ 使用颜色: 是 (生成彩色点云)
│         │  └─ 单位缩放: 1.0 (米)
│         │
│         ├─ 工作流程
│         │  ├─ 步骤1: 同步4路输入（等待所有数据到齐）
│         │  ├─ 步骤2: 视差转深度
│         │  │   └─ 深度 = (焦距 × 基线) / 视差
│         │  ├─ 步骤3: 2D转3D (针孔相机反投影)
│         │  │   ├─ 对每个像素计算3D坐标
│         │  │   ├─ X = (像素u - 主点x) × 深度 / 焦距x
│         │  │   ├─ Y = (像素v - 主点y) × 深度 / 焦距y
│         │  │   └─ Z = 深度
│         │  ├─ 步骤4: 添加颜色信息
│         │  │   └─ 从彩色图像提取RGB值
│         │  └─ 步骤5: 组装点云
│         │     └─ 每个点: (X, Y, Z, R, G, B)
│         │
│         ├─ 输出
│         │  └─ 3D彩色点云 → 发布到容器外的RViz2
│         │     └─ 话题: /disparity/points2
│         │
│         ├─ 核心算法
│         │  └─ 针孔相机反投影
│         │     ├─ 作用: 将2D+深度转为3D坐标
│         │     └─ 实现: CUDA并行计算
│         │
│         └─ 性能
│            └─ 处理时间: 约5-10ms/帧
│
│
├─── 【独立进程】RViz2 (可视化)
│    │
│    ├─ 进程: 独立运行，不在容器内
│    ├─ 包: rviz2
│    ├─ 配置: isaac_ros_stereo_image_proc_isaac_sim.rviz
│    │
│    ├─ 订阅话题
│    │  ├─ /disparity/points2 ← PointCloudNode (容器外通信)
│    │  ├─ /disparity/disparity ← DisparityNode (容器外通信)
│    │  └─ /front_stereo_camera/* ← Isaac Sim
│    │
│    ├─ 工作
│    │  ├─ 接收点云数据
│    │  ├─ 3D渲染
│    │  └─ 显示图像和视差图
│    │
│    └─ 输出
│       └─ 图形界面显示
│
│
└─── 完整系统架构
     │
     ├─ 进程架构
     │  │
     │  ├─ 进程1: disparity_container
     │  │  ├─ DisparityNode (视差计算)
     │  │  └─ PointCloudNode (点云生成)
     │  │     └─ 进程内零拷贝通信 (超快)
     │  │
     │  └─ 进程2: rviz2
     │     └─ RViz2 (可视化)
     │        └─ 跨进程通信 (ROS话题)
     │
     ├─ 通信方式
     │  │
     │  ├─ Isaac Sim → Container
     │  │  └─ ROS 2话题 (跨进程)
     │  │
     │  ├─ DisparityNode → PointCloudNode
     │  │  └─ 容器内零拷贝 (GPU内存共享) [超快]
     │  │
     │  └─ Container → RViz2
     │     └─ ROS 2话题 (跨进程)
     │
     ├─ 数据流向
     │  │
     │  ┌──────────────┐
     │  │  Isaac Sim   │ 仿真器
     │  └──────┬───────┘
     │         │ ROS话题
     │         ↓
     │  ┌─────────────────────────────┐
     │  │  Container (单个进程)        │
     │  │  ┌─────────────────────┐    │
     │  │  │  DisparityNode      │    │
     │  │  │  (立体匹配)          │    │
     │  │  └─────────┬───────────┘    │
     │  │            │ 零拷贝 [快]    │
     │  │            ↓                 │
     │  │  ┌─────────────────────┐    │
     │  │  │  PointCloudNode     │    │
     │  │  │  (3D重建)           │    │
     │  │  └─────────┬───────────┘    │
     │  └────────────┼─────────────────┘
     │               │ ROS话题
     │               ↓
     │  ┌──────────────┐
     │  │    RViz2     │ 可视化
     │  └──────────────┘
     │
     ├─ 性能优势
     │  │
     │  ├─ 容器内通信
     │  │  ├─ 零拷贝 (不复制数据)
     │  │  ├─ GPU内存直接共享
     │  │  └─ 延迟 < 1ms
     │  │
     │  └─ 独立RViz
     │     ├─ 不影响核心处理
     │     ├─ 可随时启动/关闭
     │     └─ 崩溃不影响数据流
     │
     └─ 整体性能
        ├─ 帧率: 约30fps
        ├─ 端到端延迟: 约20ms
        ├─ CPU占用: 低
        └─ GPU占用: 中等
``` 

对Isaac-ROS库的理解: 

```
Isaac ROS 完整架构
│
├─── ROS 2节点包装层
│    └─ 作用: 提供标准ROS接口 (话题、参数、服务)
│    └─ 示例: DisparityNode, PointCloudNode
│       ↓
│
├─── NITROS/GXF框架层 ← 解决"数据传输"问题
│    │
│    ├─ NITROS: GPU零拷贝通信
│    │  └─ 避免GPU⇄CPU拷贝，数据始终在GPU
│    │
│    └─ GXF: 计算图引擎
│       └─ 节点间共享GPU内存，高效调度
│       ↓
│
├─── GPU加速算法层 ← 解决"计算执行"问题
│    │
│    ├─ VPI库 (视觉算法加速)
│    │  ├─ 使用节点: DisparityNode, RectifyNode
│    │  └─ 作用: 提供优化的视觉算法 (SGM、重映射等)
│    │
│    ├─ CUDA (自定义并行计算)
│    │  ├─ 使用节点: PointCloudNode, NormalizeNode
│    │  └─ 作用: 自己编写的GPU并行算法
│    │
│    └─ TensorRT (深度学习加速)
│       ├─ 使用节点: DNN推理节点
│       └─ 作用: 神经网络推理优化
│       ↓
│
└─── NVIDIA GPU硬件层
     ├─ CUDA核心 (通用并行计算)
     └─ 专用加速器 (Xavier DLA/PVA, Orin PVA 2.0)
```
