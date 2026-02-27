---
title: 20251011 - 分析isaac_ros_nvblox的原理 (3D地图建立, 以及机器人路径规划)
confluence_page_id: 4358847
created_at: 2025-10-11T05:24:12+00:00
updated_at: 2025-10-12T14:45:57+00:00
---

实验文档: <https://nvidia-isaac-ros.github.io/concepts/scene_reconstruction/nvblox/tutorials/tutorial_isaac_sim.html>

包文档: <https://nvidia-isaac-ros.github.io/repositories_and_packages/isaac_ros_nvblox/isaac_ros_nvblox/index.html#quickstart>

# 安装

在ROS dev容器中, 下载asset (8G大小)

```
NGC_ORG="nvidia"
NGC_TEAM="isaac"
PACKAGE_NAME="isaac_ros_nvblox"
NGC_RESOURCE="isaac_ros_nvblox_assets"
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

在ROS dev容器中:

```
sudo apt-get install -y ros-humble-isaac-ros-nvblox
rosdep update && \
rosdep install isaac_ros_nvblox
``` 

在桌面上, 执行验证: 

```
ros2 launch nvblox_examples_bringup isaac_sim_example.launch.py rosbag:=${ISAAC_ROS_WS}/isaac_ros_assets/isaac_ros_nvblox/quickstart rosbag_args:="--clock --rate 0.1 --loop" navigation:=False
 
# 使用--rate参数, 放慢rosbag的回放过程
``` 

![image2025-10-11 13:23:11.png](/assets/01KJC03VRF6SMQ2J69DXQ5SBTA/image2025-10-11%2013%3A23%3A11.png)

排错:

  - 如果没有3D图像输出, 检查一下进程, 有遗留进程时 信号会有冲突, 导致有相机图像, 但没有3D图像
  - 使用loop参数时, 第二次播放也会丢失3D图像

# 与Isaac Sim配合使用

打开Isaac Sim, 打开场景: <https://omniverse-content-production.s3-us-west-2.amazonaws.com/Assets/Isaac/4.2/Isaac/Samples/NvBlox/nvblox_sample_scene.usd>

![image2025-10-11 13:58:22.png](/assets/01KJC03VRF6SMQ2J69DXQ5SBTA/image2025-10-11%2013%3A58%3A22.png)

(此处会提示是否运行内置python脚本, 开启后, 会导致Isaac Sim panic, 暂不开启)

进入桌面的容器, 执行: 

```
ros2 launch nvblox_examples_bringup isaac_sim_example.launch.py

``` 

会默认打开rviz: (3个摄像头)

![image2025-10-11 14:0:7.png](/assets/01KJC03VRF6SMQ2J69DXQ5SBTA/image2025-10-11%2014%3A0%3A7.png)

在rviz上, 使用2D Goal Pose, 设定一个终点, 然后程序会自动规划路径: 

![image2025-10-11 14:12:34.png](/assets/01KJC03VRF6SMQ2J69DXQ5SBTA/image2025-10-11%2014%3A12%3A34.png)

绕行路径: 

![image2025-10-11 14:13:5.png](/assets/01KJC03VRF6SMQ2J69DXQ5SBTA/image2025-10-11%2014%3A13%3A5.png)

结合雷达数据: 

```
ros2 launch nvblox_examples_bringup isaac_sim_example.launch.py \
lidar:=True num_cameras:=1
``` 

(只使用相机的话, 探索范围比这个小很多)

![image2025-10-11 14:16:18.png](/assets/01KJC03VRF6SMQ2J69DXQ5SBTA/image2025-10-11%2014%3A16%3A18.png)

使用三个相机: 

```
ros2 launch nvblox_examples_bringup isaac_sim_example.launch.py num_cameras:=3
``` 

![image2025-10-11 14:25:58.png](/assets/01KJC03VRF6SMQ2J69DXQ5SBTA/image2025-10-11%2014%3A25%3A58.png)

在rviz中增加对额外摄像头图像的显示

单个摄像头对比

![image2025-10-11 14:41:56.png](/assets/01KJC03VRF6SMQ2J69DXQ5SBTA/image2025-10-11%2014%3A41%3A56.png)

# 融入人类识别

对应源文档的 Reconstruction With People 章节

  - 将Human显示出来
    - ![image2025-10-12 12:15:23.png](/assets/01KJC03VRF6SMQ2J69DXQ5SBTA/image2025-10-12%2012%3A15%3A23.png)
  - 在桌面上执行脚本: 

```
ros2 launch nvblox_examples_bringup isaac_sim_example.launch.py \
mode:=people_segmentation num_cameras:=1
```

但脚本地址错误: 

![image2025-10-12 12:13:56.png](/assets/01KJC03VRF6SMQ2J69DXQ5SBTA/image2025-10-12%2012%3A13%3A56.png)

修正脚本地址后, 启动模拟器, 人员也不能移动, 只能站在原地, 原因不明.

可能与ActionGraph有关, ActionGraph并没有触发条件: 

![image2025-10-12 12:20:23.png](/assets/01KJC03VRF6SMQ2J69DXQ5SBTA/image2025-10-12%2012%3A20%3A23.png)

先忽略这个case

# 检测动态物体

将 /World/Dynamics 可见出来, 启动模拟器, 发现两个小车在做圆周运动, 但没有相关脚本配置

相关的驱动组件: 

  - /World/Dynamics/forklift_b_01/back_wheel_joints/back_wheel_swivel
  - /World/Dynamics/forklift_b_01/back_wheel_joints/back_wheel_drive

需要获取小车的所有配置, 使用如下方法: 

  - 在isaac sim中, 使用Save Flattern As, 将这个usd导出为usda
  - 使用awk 'length <= 500' a.usda > a.filtered.usda, 将其中过长的行过滤掉 (大部分是图片的表达式), 容易处理
  - 从中找到 后轮的驱动 的配置

```
           def Scope "back_wheel_joints"
            {
                def PhysicsRevoluteJoint "back_wheel_drive" (
                    apiSchemas = ["PhysicsDriveAPI:angular"]
                )
                {
                    float drive:angular:physics:damping = 10000
                    float drive:angular:physics:maxForce = inf
                    float drive:angular:physics:stiffness = 100
                    float drive:angular:physics:targetVelocity = -600
                    uniform token physics:axis = "X"
                    rel physics:body0 = </World/Dynamics/forklift_b_01/back_wheel>
                    rel physics:body1 = </World/Dynamics/forklift_b_01/back_wheel_swivel>
                    float physics:breakForce = inf
                    float physics:breakTorque = inf
                    point3f physics:localPos0 = (0, 0.0000010996208, 0)
                    point3f physics:localPos1 = (0.07109, 0.0023810996, -0.19771929)
                    quatf physics:localRot0 = (0.17364818, -0.9848077, 0, 0)
                    quatf physics:localRot1 = (0.9848077, 0, 0, 0.17364818)
                }

                def PhysicsRevoluteJoint "back_wheel_swivel" (
                    apiSchemas = ["PhysicsDriveAPI:angular"]
                )
                {
                    float drive:angular:physics:damping = 100
                    float drive:angular:physics:maxForce = inf
                    float drive:angular:physics:stiffness = 100000
                    float drive:angular:physics:targetPosition = 0
                    uniform token physics:axis = "Z"
                    rel physics:body0 = </World/Dynamics/forklift_b_01/back_wheel_swivel>
                    rel physics:body1 = </World/Dynamics/forklift_b_01/body>
                    float physics:breakForce = inf
                    float physics:breakTorque = inf
                    point3f physics:localPos0 = (0, 0, 0)
                    point3f physics:localPos1 = (0.57754, 0.07873, 0.35575458)
                    quatf physics:localRot0 = (1, 0, 0, 1.7114271e-8)
                    quatf physics:localRot1 = (0.70710677, 0, 0, -0.70710677)
                    float physics:lowerLimit = -60
                    float physics:upperLimit = 60
                }
            }
``` 
    
    
    back_wheel_swivel 的根配置:

```
            def Xform "back_wheel_swivel" (
                apiSchemas = ["PhysicsRigidBodyAPI", "PhysxRigidBodyAPI"]
            )
            {
                vector3f physics:angularVelocity = (0, 0, 0)
                bool physics:kinematicEnabled = 0
                bool physics:rigidBodyEnabled = 1
                vector3f physics:velocity = (0, 0, 0)
                float physxRigidBody:maxDepenetrationVelocity = 0.049999997
                float physxRigidBody:maxLinearVelocity = inf
                float physxRigidBody:sleepThreshold = 5e-7
                float physxRigidBody:stabilizationThreshold = 1e-9
                quatd xformOp:orient = (0.7071067811865476, 0, 0, -0.7071067811865475)
                double3 xformOp:scale = (1, 1, 1)
                double3 xformOp:translate = (0.57754, 0.07873, 0.35575459204826504)
                uniform token[] xformOpOrder = ["xformOp:translate", "xformOp:orient", "xformOp:scale"]

                def Mesh "SM_Forklift_BackWheelbase_B01_01" (
                    apiSchemas = ["PhysxCookedDataAPI:convexDecomposition", "NavMeshExcludeAPI", "ShadowAPI", "PhysicsCollisionAPI", "PhysxCollisionAPI", "PhysxConvexHullCollisionAPI", "PhysicsMeshCollisionAPI", "PhysxConvexDecompositionCollisionAPI", "MaterialBindingAPI"]
                )
                {
                    float3[] extent = [(-14.09821, -0.856953, 7.15626), (2.926806, 14.06144, 36.16684)]
                    rel material:binding = </World/Dynamics/forklift_b/Looks/M_Forklift_B1_Body> (
                        bindMaterialAs = "weakerThanDescendants"
                    )
                        interpolation = "faceVarying"
                    )
                    uniform token orientation = "rightHanded"
                    uniform token physics:approximation = "convexDecomposition"
                    bool physics:collisionEnabled = 0
                    float physxCollision:contactOffset = -inf
                    float physxCollision:restOffset = -inf
                    float physxConvexDecompositionCollision:minThickness = 0.000010000001
                    float physxConvexHullCollision:minThickness = 0.000010000001
                    color3f[] primvars:displayColor = [(0.8000001, 0.8000001, 0.8000001)]
                        interpolation = "faceVarying"
                    )
                    uniform token subdivisionScheme = "none"
                    float3 xformOp:rotateXYZ = (0, 0, 0)
                    float3 xformOp:scale = (0.01, 0.01, 0.01)
                    double3 xformOp:translate = (0.07827444927582031, -0.06602240977916696, -0.3616883916592866)
                    double3 xformOp:translate:pivot = (0, 0, 0)
                    uniform token[] xformOpOrder = ["xformOp:translate", "xformOp:translate:pivot", "xformOp:rotateXYZ", "xformOp:scale", "!invert!xformOp:translate:pivot"]
                }
            }
``` 

小车能动的原因: 

  - 在back_wheel_drive中, 配置了角动量: targetVelocity = -600
  - 在back_wheel_swivel 根配置中, 配置了旋转: orient = (0.7071067811865476, 0, 0, -0.7071067811865475)  
  

执行命令: 

```
ros2 launch nvblox_examples_bringup isaac_sim_example.launch.py \
mode:=dynamic
``` 

对动态物体的检测: 

![image2025-10-12 14:39:56.png](/assets/01KJC03VRF6SMQ2J69DXQ5SBTA/image2025-10-12%2014%3A39%3A56.png)

如果不使用动态物体检测: 

![image2025-10-12 14:44:10.png](/assets/01KJC03VRF6SMQ2J69DXQ5SBTA/image2025-10-12%2014%3A44%3A10.png)

# 代码分析

<https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_nvblox/tree/main/nvblox_examples/nvblox_examples_bringup>

```
════════════════════════════════════════════════════════════════════════════════
                    按数据流向顺序的组件分析
════════════════════════════════════════════════════════════════════════════════

┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃                        数据源层（起点）                               ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛

[数据源A] Isaac Sim 仿真环境 (默认数据源)
──────────────────────────────────────────────────────────────────
输入: 无（数据生成源）

输出话题:
   ├─ /front_stereo_camera/left/image_raw          (RGB图像 640x480 @30Hz)
   ├─ /front_stereo_camera/depth/ground_truth      (深度图 640x480 @30Hz)
   ├─ /front_stereo_camera/left/camera_info        (相机标定参数)
   ├─ /front_stereo_camera/semantic                (语义标签 @30Hz)
   │  └─ 说明: 每个像素的类别ID (uint32)
   │     示例: 像素值15=人员, 3=地面, 8=墙壁
   │     用途: people_segmentation模式用于区分人员/环境
   │
   ├─ /left_stereo_camera/...                      (左相机数据 - 如启用)
   ├─ /right_stereo_camera/...                     (右相机数据 - 如启用)
   ├─ /front_3d_lidar/point_cloud                  (激光点云 @10Hz - 如启用)
   │
   ├─ /odom                                         (里程计 @50Hz)
   │  └─ 包含两部分:
   │     ├─ pose: 机器人累积位置 (通过速度积分得到, 长期会漂移)
   │     └─ twist: 当前瞬时速度 (轮式编码器直接测量, 短期精确)
   │
   ├─ /tf                                           (动态坐标变换 @100Hz)
   │  └─ 说明: 随时间变化的变换 (如 odom->base_link, 机器人运动)
   │
   └─ /tf_static                                    (静态坐标变换)
      └─ 说明: 固定不变的变换 (如 base_link->camera, 机械安装位置)

作用:
   └─ 提供完整的机器人传感器仿真数据流

[数据源B] ROS2 Bag Player (替代数据源)
──────────────────────────────────────────────────────────────────
启动条件: 提供rosbag参数且不为'None'
代码位置: 行 96-100

输入:
   └─ rosbag文件路径 (包含录制的传感器数据)

输出话题:
   └─ 回放rosbag中的所有话题（通常与Isaac Sim输出相同）

作用:
   ├─ 替代Isaac Sim实时仿真
   ├─ 用于离线数据分析和算法测试
   └─ 可重复播放相同数据进行调试

        数据向下游传播

┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃                    预处理层（可选）                                   ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛

[组件1] Semantic Label Conversion (语义标签转换)
──────────────────────────────────────────────────────────────────
启动条件: mode='people_segmentation'
代码位置: 行 76-80
子节点数量: 2个

说明: people_segmentation模式使用语义标签识别人员
      dynamic模式使用几何检测(深度不一致性), 无需语义标签

    [子组件1.1] semantic_label_stamper
    ────────────────────────────────────────────
    输入:
       └─ /front_stereo_camera/semantic (uint32多类别语义标签)
    
    输出:
       └─ 内部话题: 时间戳对齐的语义数据
    
    作用:
       ├─ 为语义数据添加准确时间戳
       └─ 与相机RGB-D数据进行时间同步

    [子组件1.2] semantic_label_converter
    ────────────────────────────────────────────
    输入:
       └─ 来自stamper的时间戳对齐语义数据
    
    输出:
       └─ /semantic_conversion/front_stereo_camera/semantic_mono8
          (uint8二值掩码: 0=背景, 255=人员)
    
    作用:
       ├─ 将多类别语义ID映射为二值掩码
       ├─ 提取"人员"类别，忽略其他物体
       └─ 转换为nvblox可用的掩码格式

        预处理完成，进入核心处理层

┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃                    核心处理层（3D重建）                               ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛

[组件2] Nvblox Node (3D重建与建图核心)
──────────────────────────────────────────────────────────────────
启动条件: 总是启动
代码位置: 行 83-93
运行环境: NVBLOX_CONTAINER_NAME容器中（ComposableNode）
节点类型: nvblox::NvbloxNode

输入话题（经过话题映射）:
   │
   ├─ 相机数据 (1-3个相机):
   │  ├─ camera_0/depth/image         <- /front_stereo_camera/depth/ground_truth
   │  ├─ camera_0/color/image         <- /front_stereo_camera/left/image_raw
   │  ├─ camera_0/depth/camera_info   <- /front_stereo_camera/left/camera_info
   │  ├─ camera_0/color/camera_info   <- /front_stereo_camera/left/camera_info
   │  │
   │  ├─ camera_1/depth/image         <- /left_stereo_camera/... (如num_cameras>=2)
   │  ├─ camera_1/color/image         <- ...
   │  │
   │  └─ camera_2/depth/image         <- /right_stereo_camera/... (如num_cameras=3)
   │     └─ camera_2/color/image      <- ...
   │
   ├─ 语义掩码 (仅people_segmentation模式):
   │  └─ camera_0/mask/image          <- /semantic_conversion/.../semantic_mono8
   │
   ├─ 激光雷达 (如lidar=True):
   │  └─ pointcloud                   <- /front_3d_lidar/point_cloud
   │
   └─ 坐标变换:
      ├─ /tf                          (动态变换: odom->base_link等)
      │  └─ 查询机器人实时位姿，用于传感器数据的空间配准
      └─ /tf_static                   (静态变换: base_link->相机/激光雷达)
         └─ 获取传感器安装位置，计算传感器在世界坐标系的位姿

输出话题:
   ├─ /nvblox_node/mesh                      (3D三角网格 - 用于可视化)
   ├─ /nvblox_node/mesh_marker               (RViz Marker格式网格)
   │
   ├─ /nvblox_node/static_map_slice          (2D地图切片 - 仅静态障碍物)
   ├─ /nvblox_node/combined_map_slice        (2D地图切片 - 静态+动态)
   │  └─ 说明: 每次发布完整地图(40m×40m), 非增量更新
   │     数据量: ~625KB/帧 @10Hz, 按需发布优化
   │     下游使用: Nav2订阅后覆盖式更新内部缓冲区
   │
   ├─ /nvblox_node/dynamic_points            (动态物体点云)
   ├─ /nvblox_node/esdf_pointcloud           (ESDF距离场点云 - 调试用)
   ├─ /nvblox_node/tsdf_layer                (TSDF体素层)
   └─ /nvblox_node/freespace                 (自由空间标记)

作用:
   ├─ [核心功能1] TSDF融合
   │  └─ 将多帧RGB-D数据融合到3D体素网格中
   │
   ├─ [核心功能2] ESDF计算
   │  └─ 计算到最近障碍物的距离场（用于路径规划）
   │
   ├─ [核心功能3] 网格重建
   │  └─ 从TSDF提取彩色三角网格（用于可视化）
   │
   ├─ [核心功能4] 2D地图切片生成
   │  └─ 从ESDF切割水平切片，生成2D占用栅格地图
   │
   └─ [核心功能5] 动态物体处理
      ├─ people_segmentation模式: 使用语义掩码标记人员区域
      ├─ dynamic模式: 通过几何不一致性检测动态物体
      │  └─ 原理: 比较观测深度 vs TSDF预期深度，差异大则为动态
      └─ 动态物体时间衰减机制 (离开后恢复为静态/删除)

配置参数:
   ├─ num_cameras: 1/3        (相机数量)
   ├─ use_lidar: True/False   (是否使用激光雷达)
   ├─ mode: static/dynamic/people_segmentation
   └─ 加载配置文件:
      ├─ nvblox_base.yaml     (基础配置)
      ├─ nvblox_sim.yaml      (Isaac Sim适配)
      └─ nvblox_*.yaml        (模式特定配置)

        3D重建完成，分流到导航和可视化

┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃                    应用层A（导航规划）                                ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛

[组件3] Navigation Stack (Nav2自主导航)
──────────────────────────────────────────────────────────────────
启动条件: navigation=True (默认启用)
代码位置: 行 60-68
运行环境: NVBLOX_CONTAINER_NAME容器中（可组合节点）
子组件数量: ~15个Nav2节点

    [子系统3.1] Costmap 2D (代价地图系统)
    ────────────────────────────────────────────
    包含: Local Costmap + Global Costmap
    
    输入:
       ├─ 地图数据:
       │  ├─ /nvblox_node/static_map_slice        (static模式)
       │  └─ /nvblox_node/combined_map_slice      (dynamic/people_seg模式)
       │
       ├─ 机器人状态:
       │  ├─ /odom                                 (里程计)
       │  └─ /tf                                   (坐标变换)
       │
       └─ 配置:
          └─ config/navigation/carter_nav2.yaml
    
    输出:
       ├─ /local_costmap/costmap                  (5m×5m 局部代价地图)
       │  └─ 说明: 从全局地图提取以机器人为中心的滚动窗口
       │     用途: 给局部规划器，高频更新(10Hz)
       │
       ├─ /global_costmap/costmap                 (40m×40m 全局代价地图)
       │  └─ 说明: 覆盖整个已知区域
       │     用途: 给全局规划器，低频更新(1Hz)
       │
       └─ /local_costmap/costmap_updates          (增量更新)
    
    作用:
       ├─ 融合nvblox地图到代价地图
       ├─ 应用膨胀层（机器人安全距离）
       └─ 提供统一的代价地图接口给规划器
    
    插件配置:
       ├─ nvblox_layer      (主要障碍物层)
       └─ inflation_layer   (膨胀安全距离)

    [子系统3.2] Planner Server (全局路径规划器)
    ────────────────────────────────────────────
    输入:
       ├─ /goal_pose                              (目标位姿)
       ├─ /global_costmap/costmap                 (全局代价地图)
       └─ 当前机器人位姿 (从/tf查询)
    
    输出:
       └─ /plan                                    (全局路径点序列)
    
    作用:
       ├─ 使用A*或Dijkstra算法规划全局路径
       ├─ 战略层规划: 覆盖大范围，考虑静态障碍
       └─ 提供从当前位置到目标的完整路径
    
    更新频率: 1 Hz 或目标/地图变化时

    [子系统3.3] Controller Server (局部路径规划器)
    ────────────────────────────────────────────
    输入:
       ├─ /plan                                    (全局路径)
       ├─ /local_costmap/costmap                  (局部代价地图)
       ├─ /odom                                    (当前速度和位姿)
       └─ /tf                                      (实时位姿)
    
    输出:
       ├─ /cmd_vel                                 (速度控制指令)
       └─ /local_plan                              (局部轨迹)
    
    作用:
       ├─ 战术层执行: 跟随全局路径，实时避障
       ├─ 动态窗口法(DWB): 生成候选轨迹(~800条)
       ├─ 多目标评分: 路径对齐+目标接近+速度偏好+避障
       │  └─ 加权组合选择最优轨迹
       └─ 生成平滑的速度指令
    
    控制频率: 10-20 Hz
    
    说明: 全局规划提供战略方向，局部规划处理实时避障

    [子系统3.4] Behavior Tree Navigator (任务协调器)
    ────────────────────────────────────────────
    输入:
       ├─ /navigate_to_pose (导航目标动作)
       └─ 各服务器状态反馈
    
    输出:
       └─ 任务状态和进度反馈
    
    作用:
       ├─ 任务管理: 分解导航任务为子步骤
       ├─ 服务调度: 通过ROS2 Action调用Planner/Controller
       ├─ 状态监控: 检测执行成功/失败
       └─ 异常恢复: 触发恢复行为(清理地图/旋转/后退)
    
    说明: 不直接做规划或控制，而是协调其他服务器工作

    [子系统3.5] Static Transform Publisher
    ────────────────────────────────────────────
    输入: 无
    
    输出:
       └─ /tf_static: map -> odom 变换
    
    作用:
       └─ 提供map和odom坐标系之间的固定变换

        导航规划完成，发送控制指令

┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃                    执行层（控制闭环）                                 ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛

[执行端] Isaac Sim (接收控制指令)
──────────────────────────────────────────────────────────────────
输入:
   └─ /cmd_vel (geometry_msgs::Twist)
      ├─ linear.x: 前进速度 (m/s)
      ├─ linear.y: 侧向速度 (通常为0，差速驱动)
      └─ angular.z: 旋转速度 (rad/s)

输出:
   ├─ /odom                  (更新后的里程计)
   │  └─ pose: 通过速度×时间步长积分更新位置
   │     twist: 当前执行的速度
   │
   └─ /tf                    (更新后的机器人位姿)

作用:
   ├─ 将速度指令转换为轮速
   ├─ 执行物理仿真
   ├─ 更新机器人状态
   └─ 闭环反馈到导航系统

        形成闭环：新状态 -> 传感器数据 -> Nvblox -> Nav2 -> 控制

┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃                    应用层B（可视化监控）                              ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛

[组件4] Visualization (可视化系统)
──────────────────────────────────────────────────────────────────
启动条件: 总是启动
代码位置: 行 103-110

    [子组件4.1] RViz2 (本地3D可视化)
    ────────────────────────────────────────────
    启动条件: run_rviz=True (默认启用)
    
    订阅话题（多源汇总）:
       │
       ├─ 来自Nvblox:
       │  ├─ /nvblox_node/mesh                    (3D重建网格)
       │  ├─ /nvblox_node/mesh_marker             (网格标记)
       │  ├─ /nvblox_node/static_map_slice        (地图切片)
       │  ├─ /nvblox_node/dynamic_points          (动态物体)
       │  └─ /nvblox_node/esdf_pointcloud         (距离场)
       │
       ├─ 来自Nav2:
       │  ├─ /plan                                 (全局路径)
       │  ├─ /local_plan                           (局部轨迹)
       │  ├─ /local_costmap/costmap               (局部代价地图)
       │  └─ /global_costmap/costmap              (全局代价地图)
       │
       ├─ 来自Isaac Sim:
       │  ├─ /front_stereo_camera/left/image_raw  (相机图像)
       │  ├─ /front_3d_lidar/point_cloud          (激光点云)
       │  └─ /odom                                 (机器人位姿)
       │
       └─ 坐标变换:
          ├─ /tf                                   (动态变换树)
          └─ /tf_static                            (静态变换树)
    
    输出:
       └─ 无（纯展示）
    
    作用:
       ├─ [显示1] 3D场景
       │  ├─ 彩色重建网格
       │  ├─ 机器人模型（基于URDF+TF）
       │  ├─ 相机视锥
       │  └─ 激光点云
       │
       ├─ [显示2] 地图视图
       │  ├─ 占用栅格地图
       │  ├─ 代价地图热力图
       │  └─ 动态物体标记
       │
       ├─ [显示3] 路径规划
       │  ├─ 全局路径（绿色曲线）
       │  ├─ 局部轨迹（红色曲线）
       │  └─ 目标点标记
       │
       └─ [显示4] 传感器数据
          ├─ 相机图像窗口
          └─ 语义掩码叠加
    
    配置文件:
       └─ config/visualization/*.rviz (根据mode选择)

    [子组件4.2] Foxglove Bridge (Web可视化)
    ────────────────────────────────────────────
    启动条件: run_foxglove=True (默认禁用)
    
    订阅话题:
       └─ 同RViz2（可配置白名单过滤）
    
    输出:
       └─ WebSocket服务 (ws://localhost:8765)
    
    作用:
       ├─ 提供Web端可视化界面
       ├─ 支持远程监控
       ├─ 多客户端并发访问
       └─ 时间轴回放控制

┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃                    基础设施层（支撑服务）                             ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛

[组件5] Component Container (进程容器)
──────────────────────────────────────────────────────────────────
启动时机: 在Navigation和Nvblox之前
代码位置: 行 71-73
容器名称: NVBLOX_CONTAINER_NAME
容器类型: isolated (隔离容器)

输入:
   └─ 无（基础设施）

输出:
   └─ 提供容器环境供其他节点加载

作用:
   ├─ 创建独立的进程环境
   ├─ 加载可组合节点 (ComposableNode)
   ├─ 提供进程内通信（零拷贝）
   └─ 降低多节点通信延迟

加载的节点:
   ├─ Nvblox Node (1个)
   └─ Nav2所有节点 (~15个)

[组件6] Global Parameter (全局参数设置)
──────────────────────────────────────────────────────────────────
启动时机: 最早（行56）
代码位置: 行 56

输入:
   └─ 无

输出:
   └─ 全局ROS参数: use_sim_time = True

作用:
   └─ 配置所有节点使用仿真时间而非系统时间

════════════════════════════════════════════════════════════════════════════════
                        关键概念速查
════════════════════════════════════════════════════════════════════════════════

[1] 语义标签 (Semantic Label)
    定义: 图像中每个像素的类别标注 (如像素值15=人员)
    来源: Isaac Sim渲染器提供GT，真实世界需分割模型预测
    用途: people_segmentation模式用于区分动态人员

[2] 里程计 (Odometry)
    组成: pose(位置) + twist(速度)
    精度: 速度直接测量(高精度)，位置积分计算(会漂移)
    原理: 轮式编码器测速 → 速度×时间积分 → 累积位置

[3] TF坐标变换
    静态: 机械结构固定关系 (base_link→camera)，发布一次
    动态: 随运动变化关系 (odom→base_link)，高频更新
    用途: 统一管理机器人所有坐标系，支持空间数据转换

[4] 地图发布机制
    方式: 每次发布完整地图(非增量)，覆盖式更新
    优化: 按需发布(地图变化时才发)，可选压缩
    下游: 订阅者维护内部缓冲区，用最新数据替换

[5] 路径规划分层
    全局: 战略规划，大范围路径，低频(1Hz)，考虑静态障碍
    局部: 战术执行，实时避障，高频(10-20Hz)，考虑动态障碍
    原因: 计算复杂度、更新需求、信息范围不同

[6] 动态物体检测
    people_seg模式: 使用语义标签(需外部模型/GT)
    dynamic模式: 几何不一致性检测(仅需深度图，自主推断)

[7] BT Navigator角色
    定位: 任务协调器，非执行者
    职责: 调用服务器(Action接口)、监控状态、异常恢复
    不做: 不做路径规划、不发控制指令、不直接访问传感器
``` 

### 重点信息

  - 在人体检测模式下, 是将Isaac Sim对 People 的标签 融入了信息流中, 相当于用GT来生成训练数据
  - 在动态物品检测模式下, 没有使用GT标签, 而是用视觉识别来识别动态物品
