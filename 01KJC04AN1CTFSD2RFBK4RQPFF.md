---
title: 20251012 - 分析isaac_ros_ess的原理 (ESS神经网络 感知深度)
confluence_page_id: 4358884
created_at: 2025-10-13T10:01:20+00:00
updated_at: 2025-10-13T16:35:36+00:00
---

实验文档: <https://nvidia-isaac-ros.github.io/concepts/stereo_depth/ess/tutorial_isaac_sim.html>

文档: <https://nvidia-isaac-ros.github.io/repositories_and_packages/isaac_ros_dnn_stereo_depth/isaac_ros_ess/index.html#quickstart>

与[20251010 - 分析isaac_ros_stereo_image_proc的原理 (SGM生成点云)]的区别: 

![image2025-10-13 18:0:56.png](/assets/01KJC04AN1CTFSD2RFBK4RQPFF/image2025-10-13%2018%3A0%3A56.png)

# 安装

在ROS2 dev容器中, 下载assets: 

```
NGC_ORG="nvidia"
NGC_TEAM="isaac"
PACKAGE_NAME="isaac_ros_ess"
NGC_RESOURCE="isaac_ros_ess_assets"
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

在ROS2 dev容器中, 安装包: 

```
sudo apt-get install -y ros-humble-isaac-ros-ess && \
   sudo apt-get install -y ros-humble-isaac-ros-ess-models-install
 
# 下载ess模型
ros2 run isaac_ros_ess_models_install install_ess_models.sh --eula
``` 

在ROS2 dev容器中,测试: 

```
sudo apt-get install -y ros-humble-isaac-ros-examples

# 窗口1: 
ros2 launch isaac_ros_examples isaac_ros_examples.launch.py launch_fragments:=ess_disparity \
   engine_file_path:=${ISAAC_ROS_WS:?}/isaac_ros_assets/models/dnn_stereo_disparity/dnn_stereo_disparity_v4.1.0_onnx/ess.engine \
   threshold:=0.0
 
 
# 窗口2: 回放bag: 
ros2 bag play -l ${ISAAC_ROS_WS}/isaac_ros_assets/isaac_ros_ess/rosbags/ess_rosbag \
   --remap /left/camera_info:=/left/camera_info_rect /right/camera_info:=/right/camera_info_rect

``` 

在桌面, 进入容器, 进行可视化: 

```
ros2 run isaac_ros_ess isaac_ros_ess_visualizer.py
``` 

深度图: 

![image2025-10-13 23:16:12.png](/assets/01KJC04AN1CTFSD2RFBK4RQPFF/image2025-10-13%2023%3A16%3A12.png)

将threshold=0.5, 深度图: 

![image2025-10-13 23:16:59.png](/assets/01KJC04AN1CTFSD2RFBK4RQPFF/image2025-10-13%2023%3A16%3A59.png)

# 与Isaac Sim联用

启动Isaac Sim, 加载Sample Scene, 执行Play

在桌面容器中执行: 

```
ros2 launch isaac_ros_ess isaac_ros_ess_isaac_sim.launch.py \
   engine_file_path:=${ISAAC_ROS_WS:?}/isaac_ros_assets/models/dnn_stereo_disparity/dnn_stereo_disparity_v4.1.0_onnx/ess.engine \
   threshold:=0.4
``` 

会自动打开rviz, 展示点云图, 但需要修改一下默认坐标系为base_link: 

![image2025-10-14 0:28:23.png](/assets/01KJC04AN1CTFSD2RFBK4RQPFF/image2025-10-14%200%3A28%3A23.png)

在桌面容器中执行可视化: 

```
ros2 run isaac_ros_ess isaac_ros_ess_visualizer.py
``` 

能正确显示深度图: 

![image2025-10-13 23:39:28.png](/assets/01KJC04AN1CTFSD2RFBK4RQPFF/image2025-10-13%2023%3A39%3A28.png)

深度图和点云图的关系: 

  - ESS只能生成深度图
  - rviz中展示的彩色点云图, 使用左相机图像 + 深度图, 进行重新整合形成的. 目的是用来展示深度图的作用
    - 从侧面看, 已经可以看到景深: 
      - ![image2025-10-14 0:34:1.png](/assets/01KJC04AN1CTFSD2RFBK4RQPFF/image2025-10-14%200%3A34%3A1.png)

# 代码分析

## **整体架构**

这个启动器实现了一个完整的立体视觉深度估计流水线，主要包含4个处理节点和1个可视化节点。

## **数据流动顺序及组件分析**

### **1. 图像输入与预处理阶段**

#### **左相机图像调整节点** (`image_resize_node_left`)  
\- **功能**: 将左相机图像调整到模型所需的输入尺寸  
\- **输入**:  
\- `front_stereo_camera/left/camera_info` - 左相机标定信息  
\- `front_stereo_camera/left/image_rect_color` - 左相机校正后的彩色图像  
\- **输出**:  
\- `front_stereo_camera/left/camera_info_resize` - 调整后的相机标定信息  
\- `front_stereo_camera/left/image_resize` - 调整到 960×576 的 RGB8 图像

#### **右相机图像调整节点** (`image_resize_node_right`)  
\- **功能**: 将右相机图像调整到模型所需的输入尺寸  
\- **输入**:  
\- `front_stereo_camera/right/camera_info` - 右相机标定信息  
\- `front_stereo_camera/right/image_rect_color` - 右相机校正后的彩色图像  
\- **输出**:  
\- `front_stereo_camera/right/camera_info_resize` - 调整后的相机标定信息  
\- `front_stereo_camera/right/image_resize` - 调整到 960×576 的 RGB8 图像

\---

### **2. 深度估计阶段**

#### **视差估计节点** (`disparity_node` - ESSDisparityNode)  
\- **功能**: 使用 ESS 深度学习模型计算立体图像对的视差图  
\- **输入**:  
\- `front_stereo_camera/left/image_resize` - 左相机调整后的图像  
\- `front_stereo_camera/left/camera_info_resize` - 左相机调整后的标定信息  
\- `front_stereo_camera/right/image_resize` - 右相机调整后的图像  
\- `front_stereo_camera/right/camera_info_resize` - 右相机调整后的标定信息  
\- **输出**:  
\- `disparity` - 视差图（默认输出话题）  
\- **关键参数**:  
\- `engine_file_path`: TensorRT 引擎文件路径  
\- `threshold`: 置信度过滤阈值（0.0-1.0）  
\- `input_layer_width`: 960  
\- `input_layer_height`: 576

\---

### **3. 3D重建阶段**

#### **点云生成节点** (`pointcloud_node` - PointCloudNode)  
\- **功能**: 将视差图转换为3D点云数据  
\- **输入**:  
\- `disparity` - 来自视差估计节点的视差图（隐式连接）  
\- `front_stereo_camera/left/image_resize` - 用于点云着色的彩色图像  
\- `front_stereo_camera/left/camera_info_resize` - 左相机标定信息  
\- `front_stereo_camera/right/camera_info_resize` - 右相机标定信息  
\- **输出**:  
\- `points2` - 彩色3D点云（默认输出话题）  
\- **关键参数**:  
\- `use_color: True` - 生成彩色点云  
\- `unit_scaling: 1.0` - 单位缩放比例

\---

### **4. 可视化阶段**

#### **RViz 可视化节点** (`rviz_node`)  
\- **功能**: 提供交互式3D可视化界面  
\- **配置文件**: `isaac_ros_ess_isaac_sim.rviz`  
\- **可视化内容**: 点云、图像、视差图等

\---

## **启动参数**

1\. **`engine_file_path`**: ESS TensorRT 引擎文件的绝对路径（必需）  
2\. **`threshold`**: 视差置信度过滤阈值，范围 0.0-1.0（默认 0.0）

## *容器配置**

所有处理节点运行在一个多线程组件容器 (`component_container_mt`) 中，实现高效的进程内通信，减少延迟。

\---

## **总结**

这个启动器实现了从立体图像到3D点云的完整流水线：  
**原始立体图像 → 图像调整 → 深度学习视差估计 → 3D点云重建 → 可视化**
