---
title: 20251014 - 分析isaac_ros_rtdetr的原理 (RT_DETR 物品检测)
confluence_page_id: 4358896
created_at: 2025-10-14T14:32:20+00:00
updated_at: 2025-10-14T14:32:20+00:00
---

实验文档: <https://nvidia-isaac-ros.github.io/concepts/object_detection/rtdetr/tutorial_isaac_sim.html>

包文档: <https://nvidia-isaac-ros.github.io/repositories_and_packages/isaac_ros_object_detection/isaac_ros_rtdetr/index.html#quickstart>

注意: 

  - 实验中只检测一个特殊的盒子
  - 如果要更换测试对象, 参考<https://nvidia-isaac-ros.github.io/concepts/pose_estimation/foundationpose/tutorial_create_your_own_mesh.html> 用iPhone制作3D模型

# 安装

进入ROS2 dev容器, 下载assets: 

```
NGC_ORG="nvidia"
NGC_TEAM="isaac"
PACKAGE_NAME="isaac_ros_rtdetr"
NGC_RESOURCE="isaac_ros_rtdetr_assets"
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

进入ROS2 dev容器, 下载模型: 

```
mkdir -p ${ISAAC_ROS_WS}/isaac_ros_assets/models/synthetica_detr && \
cd ${ISAAC_ROS_WS}/isaac_ros_assets/models/synthetica_detr && \
   wget 'https://api.ngc.nvidia.com/v2/models/nvidia/isaac/synthetica_detr/versions/1.0.0_onnx/files/sdetr_grasp.onnx'
 
/usr/src/tensorrt/bin/trtexec --onnx=${ISAAC_ROS_WS}/isaac_ros_assets/models/synthetica_detr/sdetr_grasp.onnx --saveEngine=${ISAAC_ROS_WS}/isaac_ros_assets/models/synthetica_detr/sdetr_grasp.plan
``` 

进入ROS2 dev容器, 下载包: 

```
sudo apt-get install -y ros-humble-isaac-ros-rtdetr
``` 

### 测试:

进入ROS2 dev容器:

```
sudo apt-get install -y ros-humble-isaac-ros-examples
 
# 窗口1, 执行脚本
ros2 launch isaac_ros_examples isaac_ros_examples.launch.py launch_fragments:=rtdetr interface_specs_file:=${ISAAC_ROS_WS}/isaac_ros_assets/isaac_ros_rtdetr/quickstart_interface_specs.json engine_file_path:=${ISAAC_ROS_WS}/isaac_ros_assets/models/synthetica_detr/sdetr_grasp.plan
 
# 窗口2, 回放rosbag
ros2 bag play -l ${ISAAC_ROS_WS}/isaac_ros_assets/isaac_ros_rtdetr/quickstart.bag
``` 

检查结果: 进入窗口的容器: 

```
# 窗口3: 将检测的边框, 与图片结合在一起, 给窗口4显示用
ros2 run isaac_ros_rtdetr isaac_ros_rtdetr_visualizer.py
 
 
# 窗口4: 
ros2 run rqt_image_view rqt_image_view /rtdetr_processed_image
``` 

等待1分钟, 可以看到结果: 

![image2025-10-14 16:26:31.png](/assets/01KJC04NBZQ13J4YH4F9SPEV8J/image2025-10-14%2016%3A26%3A31.png)

# 与Isaac Sim联合测试

```
# 开启Isaac Sim, 加载 Sample Scene, 进行Play
 
# 在桌面窗口执行
ros2 launch isaac_ros_rtdetr isaac_ros_rtdetr_isaac_sim.launch.py

会自动打开rqt

``` 

识别结果: 

![image2025-10-14 18:25:0.png](/assets/01KJC04NBZQ13J4YH4F9SPEV8J/image2025-10-14%2018%3A25%3A0.png)

移动盒子位置 或者 翻转, 会出现识别不稳定的情况 (在相同的深度, 识别率高, 一旦将物品放远, 识别率低)

# 代码分析

## 节点功能分类（完整版）

### 图像预处理层 (Image Preprocessing)

  1. resize_left_rt_detr_node \- 图像缩放
  2. pad_node \- 图像填充

作用: 将原始相机图像(1920×1200)调整为模型所需的输入尺寸(640×640)  
特点: 通用图像处理，与具体模型无关

* * *

### 张量转换层 (Tensor Conversion)

  1. image_to_tensor_node \- 图像→张量
  2. interleaved_to_planar_node \- HWC→CHW格式转换
  3. reshape_node \- 添加批次维度 [3,640,640]→[1,3,640,640]

作用: 将图像像素转换为神经网络可接受的张量格式  
核心:  
\- 数据类型转换 (像素→浮点数组)  
\- 颜色通道重排 (交错→平面)  
\- 批次维度封装 (符合NCHW标准)

* * *

### 模型推理层 (Model Inference)

  1. rtdetr_preprocessor_node \- RT-DETR特定预处理
  2. tensor_rt_node \- TensorRT推理引擎

作用:  
\- 预处理器: 准备模型特定输入(`images` \+ `orig_target_sizes`)  
\- TensorRT: 加载优化引擎文件，执行GPU加速推理

输出: 原始张量  
\- `labels` [1, 300] - 类别ID  
\- `boxes` [1, 300, 4] - 边界框坐标  
\- `scores` [1, 300] - 置信度分数

关键: 模型特定预处理必须在TensorRT外完成

* * *

### 结果后处理层 (Post-processing)

  1. rtdetr_decoder_node \- 检测结果解码

作用: 将模型原始输出转换为ROS标准Detection2DArray消息

解码过程:  
1\. 过滤: 去除低置信度检测 (如score < 0.5)  
2\. 坐标转换: 归一化坐标→像素坐标，模型空间→图像空间  
3\. NMS去重: 同一物体的多个框→保留最佳的一个  
4\. 格式封装: 数值张量→结构化ROS消息  
5\. 类别映射: ID数字→文本标签 (0→"person", 1→"car")

输入/输出对比:

  1. `输入: labels[1,300], boxes[1,300,4], scores[1,300] (原始张量)`
  2. ` ↓`
  3. `输出: Detection2DArray (可直接使用的结构化消息)`
  4. ` - 时间戳和坐标系信息`
  5. ` - 只包含有效检测`
  6. ` - 带类别名称和置信度`

* * *

### 可视化层 (Visualization)

  1. isaac_ros_rtdetr_visualizer_node \- 检测结果可视化
  2. rqt_image_view_node \- 图像显示

作用: 在图像上绘制检测框并显示最终结果

* * *

## 核心数据流

  1. `相机图像 → 通用预处理 → 张量转换 → 模型特定预处理 → TensorRT推理 → 解码 → 可视化`
  2. `(1920×1200) (640×640) (NCHW格式) (额外输入准备) (神经网络) (ROS消息) (显示)`
  3. ` [原始张量] ↓过滤+转换`
  4. ` [结构化消息]`

  1. `输入: labels[1,300], boxes[1,300,4], scores[1,300] (原始张量)`
  2. ` ↓`
  3. `输出: Detection2DArray (可直接使用的结构化消息)`
  4. ` - 时间戳和坐标系信息`
  5. ` - 只包含有效检测`
  6. ` - 带类别名称和置信度`
