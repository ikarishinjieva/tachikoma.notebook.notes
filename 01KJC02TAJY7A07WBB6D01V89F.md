---
title: 20251010 - 分析isaac_ros_detectnet的原理 (DNN 检测 人员)
confluence_page_id: 4358827
created_at: 2025-10-10T10:35:24+00:00
updated_at: 2025-10-13T09:55:01+00:00
---

# 使用

文档: <https://nvidia-isaac-ros.github.io/concepts/object_detection/detectnet/tutorial_isaac_sim.html>
    
    
    (开启Issac Sim, 并加载Sample Scene, 参考: http://8.134.54.170:8330/pages/viewpage.action?pageId=4358780)

```
在 Isaac DEV 容器中: 
 
下载模型文件并配置:
ros2 run isaac_ros_detectnet setup_model.sh --config-file peoplenet_config.pbtxt

 
运行: 
ros2 launch isaac_ros_detectnet isaac_ros_detectnet_isaac_sim.launch.py
``` 

这个项目没有配置rviz, 通过以下命令使用rqt进行结果查看: 

```
在 Isaac DEV 容器中: 
 
ros2 run rqt_image_view rqt_image_view /detectnet_processed_image
``` 

# 排错

运行会报错: 

```
[rqt_image_view-3] could not connect to display
[rqt_image_view-3] This application failed to start because no Qt platform plugin could be initialized. Reinstalling the application may fix this problem.
[rqt_image_view-3]
[rqt_image_view-3] Available platform plugins are: eglfs, linuxfb, minimal, minimalegl, offscreen, vnc, xcb.
[rqt_image_view-3]
[ERROR] [rqt_image_view-3]: process has died [pid 40907, exit code 1, cmd '/opt/ros/humble/lib/rqt_image_view/rqt_image_view /detectnet_processed_image --ros-args -r __node:=image_view --params-file /tmp/launch_params_cz486r2s'].
``` 

转到桌面上运行: 

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
 
ros2 launch isaac_ros_detectnet isaac_ros_detectnet_isaac_sim.launch.py
 
# 会自动弹出rqt
``` 

会自动打开rqt显示结果, 但识别失败: 

![image2025-10-10 18:20:17.png](/assets/01KJC02TAJY7A07WBB6D01V89F/image2025-10-10%2018%3A20%3A17.png)

错误日志:

```
admin@dell-136:/workspaces/isaac_ros-dev$ ros2 launch isaac_ros_detectnet isaac_ros_detectnet_isaac_sim.launch.py
[INFO] [launch]: All log files can be found below /home/admin/.ros/log/2025-10-10-10-23-01-779922-dell-136-43386
[INFO] [launch]: Default logging verbosity is set to INFO
[INFO] [component_container_mt-1]: process started with pid [43399]
[INFO] [isaac_ros_detectnet_visualizer.py-2]: process started with pid [43401]
[INFO] [rqt_image_view-3]: process started with pid [43403]
[rqt_image_view-3] QStandardPaths: XDG_RUNTIME_DIR not set, defaulting to '/tmp/runtime-admin'
[component_container_mt-1] [INFO] [1760091782.120767132] [detectnet_container.detectnet_container]: Load Library: /opt/ros/humble/lib/libresize_node.so
[component_container_mt-1] [INFO] [1760091782.141865583] [detectnet_container.detectnet_container]: Found class: rclcpp_components::NodeFactoryTemplate<nvidia::isaac_ros::image_proc::ResizeNode>
[component_container_mt-1] [INFO] [1760091782.141924728] [detectnet_container.detectnet_container]: Instantiate class: rclcpp_components::NodeFactoryTemplate<nvidia::isaac_ros::image_proc::ResizeNode>
[component_container_mt-1] [INFO] [1760091782.169826988] [image_resize_node_left]: [NitrosNode] Initializing NitrosNode
[component_container_mt-1] [INFO] [1760091782.170210202] [NitrosContext]: [NitrosContext] Loading extension: gxf/lib/std/libgxf_std.so
[component_container_mt-1] [INFO] [1760091782.172319106] [NitrosContext]: [NitrosContext] Loading extension: gxf/lib/libgxf_isaac_gxf_helpers.so
[component_container_mt-1] [INFO] [1760091782.176139920] [NitrosContext]: [NitrosContext] Loading extension: gxf/lib/libgxf_isaac_sight.so
[component_container_mt-1] [INFO] [1760091782.180242375] [NitrosContext]: [NitrosContext] Loading extension: gxf/lib/libgxf_isaac_atlas.so
[component_container_mt-1] 2025-10-10 10:23:02.191 WARN  gxf/std/program.cpp@538: No GXF scheduler specified.
[component_container_mt-1] [INFO] [1760091782.192299222] [image_resize_node_left]: [ResizeNode] Set output data format to: "nitros_image_rgb8"
[component_container_mt-1] [INFO] [1760091782.192874298] [image_resize_node_left]: [NitrosNode] Starting NitrosNode
[component_container_mt-1] [INFO] [1760091782.207883543] [image_resize_node_left]: [NitrosNode] Loading extensions
[component_container_mt-1] [INFO] [1760091782.208484722] [image_resize_node_left]: [NitrosContext] Loading extension: gxf/lib/multimedia/libgxf_multimedia.so
[component_container_mt-1] [INFO] [1760091782.211549491] [image_resize_node_left]: [NitrosContext] Loading extension: gxf/lib/libgxf_isaac_message_compositor.so
[component_container_mt-1] [INFO] [1760091782.212470777] [image_resize_node_left]: [NitrosContext] Loading extension: gxf/lib/cuda/libgxf_cuda.so
[component_container_mt-1] [INFO] [1760091782.215600384] [image_resize_node_left]: [NitrosContext] Loading extension: gxf/lib/libgxf_isaac_tensorops.so
[component_container_mt-1] [INFO] [1760091782.220174630] [image_resize_node_left]: [NitrosNode] Loading graph to the optimizer
[component_container_mt-1] [INFO] [1760091782.225178382] [image_resize_node_left]: [NitrosNode] Running optimization
[component_container_mt-1] [INFO] [1760091782.396124361] [image_resize_node_left]: [NitrosNode] Obtaining graph IO group info from the optimizer
[component_container_mt-1] [INFO] [1760091782.398961312] [image_resize_node_left]: [NitrosPublisherSubscriberGroup] Pinning the component "image_sink/sink" (type="nvidia::isaac_ros::MessageRelay") to use its compatible format only: "nitros_image_rgb8"
[component_container_mt-1] [INFO] [1760091782.404976061] [image_resize_node_left]: [NitrosNode] Starting negotiation...
[component_container_mt-1] [INFO] [1760091782.405782312] [detectnet_container.detectnet_container]: Found class: rclcpp_components::NodeFactoryTemplate<nvidia::isaac_ros::image_proc::ResizeNode>
[component_container_mt-1] [INFO] [1760091782.405817493] [detectnet_container.detectnet_container]: Instantiate class: rclcpp_components::NodeFactoryTemplate<nvidia::isaac_ros::image_proc::ResizeNode>
[INFO] [launch_ros.actions.load_composable_nodes]: Loaded node '/image_resize_node_left' in container '/detectnet_container/detectnet_container'
[component_container_mt-1] [INFO] [1760091782.408488578] [detectnet_encoder.resize_node]: [NitrosNode] Initializing NitrosNode
[component_container_mt-1] [INFO] [1760091782.409210753] [detectnet_encoder.resize_node]: [ResizeNode] Set output data format to: "nitros_image_rgb8"
[component_container_mt-1] [INFO] [1760091782.409304042] [detectnet_encoder.resize_node]: [NitrosNode] Starting NitrosNode
[component_container_mt-1] [INFO] [1760091782.424996731] [detectnet_encoder.resize_node]: [NitrosNode] Loading extensions
[component_container_mt-1] [INFO] [1760091782.425301223] [detectnet_encoder.resize_node]: [NitrosNode] Loading graph to the optimizer
[component_container_mt-1] [INFO] [1760091782.430519824] [detectnet_encoder.resize_node]: [NitrosNode] Running optimization
[component_container_mt-1] [INFO] [1760091782.643177008] [detectnet_encoder.resize_node]: [NitrosNode] Obtaining graph IO group info from the optimizer
[component_container_mt-1] [INFO] [1760091782.645601613] [detectnet_encoder.resize_node]: [NitrosPublisherSubscriberGroup] Pinning the component "image_sink/sink" (type="nvidia::isaac_ros::MessageRelay") to use its compatible format only: "nitros_image_rgb8"
[component_container_mt-1] [INFO] [1760091782.649540026] [detectnet_encoder.resize_node]: [NitrosNode] Starting negotiation...
[component_container_mt-1] [INFO] [1760091782.650038891] [image_resize_node_left]: Negotiating
[component_container_mt-1] [INFO] [1760091782.650628423] [image_resize_node_left]: Negotiating
[INFO] [launch_ros.actions.load_composable_nodes]: Loaded node '/detectnet_encoder/resize_node' in container 'detectnet_container/detectnet_container'
[component_container_mt-1] [INFO] [1760091782.650782242] [detectnet_container.detectnet_container]: Load Library: /opt/ros/humble/lib/libtriton_node.so
[component_container_mt-1] [INFO] [1760091782.666631498] [detectnet_container.detectnet_container]: Found class: rclcpp_components::NodeFactoryTemplate<nvidia::isaac_ros::dnn_inference::TritonNode>
[component_container_mt-1] [INFO] [1760091782.666710404] [detectnet_container.detectnet_container]: Instantiate class: rclcpp_components::NodeFactoryTemplate<nvidia::isaac_ros::dnn_inference::TritonNode>
[component_container_mt-1] [INFO] [1760091782.672524587] [triton_node]: [NitrosNode] Initializing NitrosNode
[component_container_mt-1] [INFO] [1760091782.674794006] [triton_node]: [TritonNode] Set input data format to: "nitros_tensor_list_nchw_rgb_f32"
[component_container_mt-1] [INFO] [1760091782.674972826] [triton_node]: [TritonNode] Set output data format to: "nitros_tensor_list_nhwc_rgb_f32"
[component_container_mt-1] [INFO] [1760091782.675396736] [triton_node]: [NitrosNode] Starting NitrosNode
[component_container_mt-1] [INFO] [1760091782.689640676] [triton_node]: [NitrosNode] Loading extensions
[component_container_mt-1] [INFO] [1760091782.690091432] [triton_node]: [NitrosContext] Loading extension: gxf/lib/serialization/libgxf_serialization.so
[component_container_mt-1] [INFO] [1760091782.691742163] [triton_node]: [NitrosContext] Loading extension: gxf/lib/libgxf_isaac_triton.so
[component_container_mt-1] [INFO] [1760091782.757661432] [triton_node]: [NitrosNode] Loading graph to the optimizer
[component_container_mt-1] [INFO] [1760091782.759606384] [triton_node]: [NitrosNode] Running optimization
[component_container_mt-1] [INFO] [1760091782.895349466] [triton_node]: [NitrosNode] Obtaining graph IO group info from the optimizer
[component_container_mt-1] [INFO] [1760091782.903389349] [triton_node]: [NitrosPublisherSubscriberGroup] Pinning the component "triton_request/input" (type="nvidia::gxf::DoubleBufferReceiver") to use its compatible format only: "nitros_tensor_list_nchw_rgb_f32"
[component_container_mt-1] [INFO] [1760091782.905359079] [triton_node]: [NitrosPublisherSubscriberGroup] Pinning the component "sink/sink" (type="nvidia::isaac_ros::MessageRelay") to use its compatible format only: "nitros_tensor_list_nhwc_rgb_f32"
[component_container_mt-1] [INFO] [1760091782.905659873] [triton_node]: [NitrosNode] Starting negotiation...
[component_container_mt-1] [INFO] [1760091782.906635111] [detectnet_container.detectnet_container]: Load Library: /opt/ros/humble/lib/libimage_format_converter_node.so
[INFO] [launch_ros.actions.load_composable_nodes]: Loaded node '/triton_node' in container '/detectnet_container/detectnet_container'
[component_container_mt-1] [INFO] [1760091782.910712606] [detectnet_container.detectnet_container]: Found class: rclcpp_components::NodeFactoryTemplate<nvidia::isaac_ros::image_proc::ImageFormatConverterNode>
[component_container_mt-1] [INFO] [1760091782.910779630] [detectnet_container.detectnet_container]: Instantiate class: rclcpp_components::NodeFactoryTemplate<nvidia::isaac_ros::image_proc::ImageFormatConverterNode>
[component_container_mt-1] [INFO] [1760091782.917730224] [detectnet_encoder.image_format_converter_node]: [NitrosNode] Initializing NitrosNode
[component_container_mt-1] [INFO] [1760091782.918889546] [detectnet_encoder.image_format_converter_node]: [ImageFormatConverterNode] Set output data format to: "nitros_image_rgb8"
[component_container_mt-1] [INFO] [1760091782.919078607] [detectnet_encoder.image_format_converter_node]: [NitrosNode] Starting NitrosNode
[component_container_mt-1] [INFO] [1760091782.955430959] [detectnet_encoder.image_format_converter_node]: [NitrosNode] Loading extensions
[component_container_mt-1] [INFO] [1760091782.955962300] [detectnet_encoder.image_format_converter_node]: [NitrosNode] Loading graph to the optimizer
[component_container_mt-1] [INFO] [1760091782.960552646] [detectnet_encoder.image_format_converter_node]: [NitrosNode] Running optimization
[component_container_mt-1] [INFO] [1760091783.033125482] [detectnet_encoder.image_format_converter_node]: [NitrosNode] Obtaining graph IO group info from the optimizer
[component_container_mt-1] [INFO] [1760091783.039236787] [detectnet_encoder.image_format_converter_node]: [NitrosPublisherSubscriberGroup] Pinning the component "sink/sink" (type="nvidia::isaac_ros::MessageRelay") to use its compatible format only: "nitros_image_rgb8"
[component_container_mt-1] [INFO] [1760091783.040131934] [detectnet_encoder.image_format_converter_node]: [NitrosNode] Starting negotiation...
[component_container_mt-1] [INFO] [1760091783.040367392] [detectnet_encoder.resize_node]: Negotiating
[component_container_mt-1] [INFO] [1760091783.040678457] [image_resize_node_left]: Negotiating
[component_container_mt-1] [INFO] [1760091783.041370061] [detectnet_container.detectnet_container]: Load Library: /opt/ros/humble/lib/libdetectnet_decoder_node.so
[component_container_mt-1] [INFO] [1760091783.041467201] [image_resize_node_left]: Negotiating
[INFO] [launch_ros.actions.load_composable_nodes]: Loaded node '/detectnet_encoder/image_format_converter_node' in container 'detectnet_container/detectnet_container'
[component_container_mt-1] [INFO] [1760091783.047242808] [detectnet_container.detectnet_container]: Found class: rclcpp_components::NodeFactoryTemplate<nvidia::isaac_ros::detectnet::DetectNetDecoderNode>
[component_container_mt-1] [INFO] [1760091783.047452571] [detectnet_container.detectnet_container]: Instantiate class: rclcpp_components::NodeFactoryTemplate<nvidia::isaac_ros::detectnet::DetectNetDecoderNode>
[component_container_mt-1] [INFO] [1760091783.063654156] [detectnet_decoder_node]: [NitrosNode] Initializing NitrosNode
[component_container_mt-1] [INFO] [1760091783.066636479] [detectnet_decoder_node]: [NitrosNode] Starting NitrosNode
[component_container_mt-1] [INFO] [1760091783.090064194] [detectnet_decoder_node]: [NitrosNode] Loading extensions
[component_container_mt-1] [INFO] [1760091783.091488944] [detectnet_decoder_node]: [NitrosContext] Loading extension: gxf/lib/libgxf_isaac_detectnet.so
[component_container_mt-1] [INFO] [1760091783.094230403] [detectnet_decoder_node]: [NitrosNode] Loading graph to the optimizer
[component_container_mt-1] [INFO] [1760091783.100968376] [detectnet_decoder_node]: [NitrosNode] Running optimization
[component_container_mt-1] [INFO] [1760091783.104329810] [detectnet_decoder_node]: [NitrosNode] Obtaining graph IO group info from the optimizer
[component_container_mt-1] [INFO] [1760091783.106608594] [detectnet_decoder_node]: [NitrosNode] Starting negotiation...
[component_container_mt-1] [INFO] [1760091783.106931106] [triton_node]: Negotiating
[INFO] [launch_ros.actions.load_composable_nodes]: Loaded node '/detectnet_decoder_node' in container '/detectnet_container/detectnet_container'
[component_container_mt-1] [INFO] [1760091783.106978780] [triton_node]: Could not negotiate
[component_container_mt-1] [INFO] [1760091783.107613347] [detectnet_container.detectnet_container]: Load Library: /opt/ros/humble/lib/libcrop_node.so
[component_container_mt-1] [INFO] [1760091783.109846031] [detectnet_container.detectnet_container]: Found class: rclcpp_components::NodeFactoryTemplate<nvidia::isaac_ros::image_proc::CropNode>
[component_container_mt-1] [INFO] [1760091783.109876043] [detectnet_container.detectnet_container]: Instantiate class: rclcpp_components::NodeFactoryTemplate<nvidia::isaac_ros::image_proc::CropNode>
[component_container_mt-1] [INFO] [1760091783.114178258] [detectnet_encoder.crop_node]: [NitrosNode] Initializing NitrosNode
[component_container_mt-1] [INFO] [1760091783.114837413] [detectnet_encoder.crop_node]: [NitrosNode] Starting NitrosNode
[component_container_mt-1] [INFO] [1760091783.132143502] [detectnet_encoder.crop_node]: [NitrosNode] Loading extensions
[component_container_mt-1] [INFO] [1760091783.132441378] [detectnet_encoder.crop_node]: [NitrosNode] Loading graph to the optimizer
[component_container_mt-1] [INFO] [1760091783.136275891] [detectnet_encoder.crop_node]: [NitrosNode] Running optimization
[component_container_mt-1] [INFO] [1760091783.171249659] [detectnet_encoder.crop_node]: [NitrosNode] Obtaining graph IO group info from the optimizer
[component_container_mt-1] [INFO] [1760091783.174803681] [detectnet_encoder.crop_node]: [NitrosNode] Starting negotiation...
[component_container_mt-1] [INFO] [1760091783.175401684] [detectnet_encoder.image_format_converter_node]: Negotiating
[INFO] [launch_ros.actions.load_composable_nodes]: Loaded node '/detectnet_encoder/crop_node' in container 'detectnet_container/detectnet_container'
[component_container_mt-1] [INFO] [1760091783.175470059] [detectnet_encoder.resize_node]: Negotiating
[component_container_mt-1] [INFO] [1760091783.176555605] [detectnet_encoder.resize_node]: Negotiating
[component_container_mt-1] [INFO] [1760091783.177154421] [detectnet_encoder.crop_node]: Negotiating
[component_container_mt-1] [INFO] [1760091783.177221387] [detectnet_encoder.crop_node]: Could not negotiate
[component_container_mt-1] [INFO] [1760091783.177281580] [detectnet_encoder.crop_node]: Negotiating
[component_container_mt-1] [INFO] [1760091783.177307331] [detectnet_encoder.crop_node]: Could not negotiate
[component_container_mt-1] [INFO] [1760091783.180508851] [detectnet_container.detectnet_container]: Load Library: /opt/ros/humble/lib/libimage_to_tensor_node.so
[component_container_mt-1] [INFO] [1760091783.279773453] [detectnet_container.detectnet_container]: Found class: rclcpp_components::NodeFactoryTemplate<nvidia::isaac_ros::dnn_inference::ImageToTensorNode>
[component_container_mt-1] [INFO] [1760091783.279824079] [detectnet_container.detectnet_container]: Instantiate class: rclcpp_components::NodeFactoryTemplate<nvidia::isaac_ros::dnn_inference::ImageToTensorNode>
[component_container_mt-1] [INFO] [1760091783.282973657] [detectnet_encoder.image_to_tensor.ManagedNitrosSubscriber]: Starting Managed Nitros Subscriber
[component_container_mt-1] [INFO] [1760091783.283394236] [detectnet_encoder.crop_node]: Negotiating
[component_container_mt-1] [INFO] [1760091783.283558915] [detectnet_encoder.image_to_tensor.ManagedNitrosPublisher]: Starting Managed Nitros Publisher
[INFO] [launch_ros.actions.load_composable_nodes]: Loaded node '/detectnet_encoder/image_to_tensor' in container 'detectnet_container/detectnet_container'
[component_container_mt-1] [INFO] [1760091783.283599449] [detectnet_encoder.image_format_converter_node]: Negotiating
[component_container_mt-1] [INFO] [1760091783.284004357] [detectnet_encoder.resize_node]: Negotiating
[component_container_mt-1] [INFO] [1760091783.288524421] [detectnet_container.detectnet_container]: Load Library: /opt/ros/humble/lib/libimage_tensor_normalize_node.so
[component_container_mt-1] [INFO] [1760091783.289860550] [detectnet_container.detectnet_container]: Found class: rclcpp_components::NodeFactoryTemplate<nvidia::isaac_ros::dnn_inference::ImageTensorNormalizeNode>
[component_container_mt-1] [INFO] [1760091783.289923848] [detectnet_container.detectnet_container]: Instantiate class: rclcpp_components::NodeFactoryTemplate<nvidia::isaac_ros::dnn_inference::ImageTensorNormalizeNode>
[component_container_mt-1] [INFO] [1760091783.292696431] [detectnet_encoder.normalize_node.ManagedNitrosSubscriber]: Starting Managed Nitros Subscriber
[component_container_mt-1] [INFO] [1760091783.293011564] [detectnet_encoder.image_to_tensor]: Negotiating
[component_container_mt-1] [INFO] [1760091783.293115566] [detectnet_encoder.image_to_tensor]: Could not negotiate
[component_container_mt-1] [INFO] [1760091783.293291144] [detectnet_encoder.normalize_node.ManagedNitrosPublisher]: Starting Managed Nitros Publisher
[INFO] [launch_ros.actions.load_composable_nodes]: Loaded node '/detectnet_encoder/normalize_node' in container 'detectnet_container/detectnet_container'
[component_container_mt-1] [INFO] [1760091783.302331185] [detectnet_container.detectnet_container]: Load Library: /opt/ros/humble/lib/libinterleaved_to_planar_node.so
[component_container_mt-1] [INFO] [1760091783.305992249] [detectnet_container.detectnet_container]: Found class: rclcpp_components::NodeFactoryTemplate<nvidia::isaac_ros::dnn_inference::InterleavedToPlanarNode>
[component_container_mt-1] [INFO] [1760091783.306062507] [detectnet_container.detectnet_container]: Instantiate class: rclcpp_components::NodeFactoryTemplate<nvidia::isaac_ros::dnn_inference::InterleavedToPlanarNode>
[component_container_mt-1] [INFO] [1760091783.315128080] [detectnet_encoder.interleaved_to_planar_node]: [NitrosNode] Initializing NitrosNode
[component_container_mt-1] [INFO] [1760091783.316576685] [detectnet_encoder.interleaved_to_planar_node]: [NitrosNode] Starting NitrosNode
[component_container_mt-1] [INFO] [1760091783.360725241] [detectnet_encoder.interleaved_to_planar_node]: [NitrosNode] Loading extensions
[component_container_mt-1] [INFO] [1760091783.362223589] [detectnet_encoder.interleaved_to_planar_node]: [NitrosNode] Loading graph to the optimizer
[component_container_mt-1] [INFO] [1760091783.365639185] [detectnet_encoder.interleaved_to_planar_node]: [NitrosNode] Running optimization
[component_container_mt-1] [INFO] [1760091783.374090058] [detectnet_encoder.interleaved_to_planar_node]: [NitrosNode] Obtaining graph IO group info from the optimizer
[component_container_mt-1] [INFO] [1760091783.378136847] [detectnet_encoder.interleaved_to_planar_node]: [NitrosNode] Starting negotiation...
[component_container_mt-1] [INFO] [1760091783.378495778] [detectnet_encoder.normalize_node]: Negotiating
[INFO] [launch_ros.actions.load_composable_nodes]: Loaded node '/detectnet_encoder/interleaved_to_planar_node' in container 'detectnet_container/detectnet_container'
[component_container_mt-1] [INFO] [1760091783.383288207] [detectnet_container.detectnet_container]: Load Library: /opt/ros/humble/lib/libreshape_node.so
[component_container_mt-1] [INFO] [1760091783.387186155] [detectnet_container.detectnet_container]: Found class: rclcpp_components::NodeFactoryTemplate<nvidia::isaac_ros::dnn_inference::ReshapeNode>
[component_container_mt-1] [INFO] [1760091783.387269377] [detectnet_container.detectnet_container]: Instantiate class: rclcpp_components::NodeFactoryTemplate<nvidia::isaac_ros::dnn_inference::ReshapeNode>
[component_container_mt-1] [INFO] [1760091783.396366060] [detectnet_encoder.reshape_node]: [NitrosNode] Initializing NitrosNode
[component_container_mt-1] [INFO] [1760091783.397974068] [detectnet_encoder.reshape_node]: [NitrosNode] Starting NitrosNode
[component_container_mt-1] [INFO] [1760091783.406697159] [image_resize_node_left]: [NitrosNode] Starting post negotiation setup
[component_container_mt-1] [INFO] [1760091783.406764365] [image_resize_node_left]: [NitrosNode] Getting data format negotiation results
[component_container_mt-1] [INFO] [1760091783.406815014] [image_resize_node_left]: [NitrosPublisher] Use the negotiated data format: "nitros_image_rgb8"
[component_container_mt-1] [INFO] [1760091783.406840498] [image_resize_node_left]: [NitrosPublisher] Use the negotiated data format: "nitros_camera_info"
[component_container_mt-1] [INFO] [1760091783.406863007] [image_resize_node_left]: [NitrosSubscriber] Negotiation ended with no results
[component_container_mt-1] [INFO] [1760091783.406881314] [image_resize_node_left]: [NitrosSubscriber] Use the compatible subscriber: topic_name="/front_stereo_camera/left/image_rect_color", data_format="nitros_image_rgb8"
[component_container_mt-1] [INFO] [1760091783.406901121] [image_resize_node_left]: [NitrosSubscriber] Negotiation ended with no results
[component_container_mt-1] [INFO] [1760091783.406918032] [image_resize_node_left]: [NitrosSubscriber] Use the compatible subscriber: topic_name="/front_stereo_camera/left/camera_info", data_format="nitros_camera_info"
[component_container_mt-1] [INFO] [1760091783.407251724] [image_resize_node_left]: [NitrosNode] Exporting the final graph based on the negotiation results
[component_container_mt-1] [INFO] [1760091783.413074666] [detectnet_encoder.reshape_node]: [NitrosNode] Loading extensions
[component_container_mt-1] [INFO] [1760091783.413651010] [detectnet_encoder.reshape_node]: [NitrosNode] Loading graph to the optimizer
[component_container_mt-1] [INFO] [1760091783.417548554] [detectnet_encoder.reshape_node]: [NitrosNode] Running optimization
[component_container_mt-1] [INFO] [1760091783.427049992] [image_resize_node_left]: [NitrosNode] Wrote the final top level YAML graph to "/tmp/isaac_ros_nitros/graphs/YAFEXEXNVT/YAFEXEXNVT.yaml"
[component_container_mt-1] [INFO] [1760091783.427105379] [image_resize_node_left]: [NitrosNode] Loading application
[component_container_mt-1] [INFO] [1760091783.431415745] [detectnet_encoder.reshape_node]: [NitrosNode] Obtaining graph IO group info from the optimizer
[component_container_mt-1] [INFO] [1760091783.432139499] [image_resize_node_left]: [ResizeNode] postLoadGraphCallback().
[component_container_mt-1] [INFO] [1760091783.432241094] [image_resize_node_left]: [NitrosNode] Initializing and running GXF graph
[component_container_mt-1] [INFO] [1760091783.435992293] [detectnet_encoder.reshape_node]: [NitrosNode] Starting negotiation...
[component_container_mt-1] [INFO] [1760091783.436726359] [detectnet_encoder.interleaved_to_planar_node]: Negotiating
[component_container_mt-1] [INFO] [1760091783.437356367] [detectnet_encoder.normalize_node]: Negotiating
[component_container_mt-1] [INFO] [1760091783.437439544] [detectnet_encoder.reshape_node]: Negotiating
[INFO] [launch_ros.actions.load_composable_nodes]: Loaded node '/detectnet_encoder/reshape_node' in container 'detectnet_container/detectnet_container'
[component_container_mt-1] [INFO] [1760091783.439041649] [detectnet_encoder.interleaved_to_planar_node]: Negotiating
[component_container_mt-1] [INFO] [1760091783.439455271] [detectnet_encoder.normalize_node]: Negotiating
[component_container_mt-1] [INFO] [1760091783.568482051] [image_resize_node_left]: [NitrosNode] Node was started
[component_container_mt-1] [INFO] [1760091783.650671620] [detectnet_encoder.resize_node]: [NitrosNode] Starting post negotiation setup
[component_container_mt-1] [INFO] [1760091783.650730706] [detectnet_encoder.resize_node]: [NitrosNode] Getting data format negotiation results
[component_container_mt-1] [INFO] [1760091783.650744089] [detectnet_encoder.resize_node]: [NitrosPublisher] Use the negotiated data format: "nitros_image_rgb8"
[component_container_mt-1] [INFO] [1760091783.650756271] [detectnet_encoder.resize_node]: [NitrosPublisher] Use the negotiated data format: "nitros_camera_info"
[component_container_mt-1] [INFO] [1760091783.650765598] [detectnet_encoder.resize_node]: [NitrosSubscriber] Use the negotiated data format: "nitros_image_rgb8"
[component_container_mt-1] [INFO] [1760091783.650777026] [detectnet_encoder.resize_node]: [NitrosSubscriber] Use the negotiated data format: "nitros_camera_info"
[component_container_mt-1] [INFO] [1760091783.650931186] [detectnet_encoder.resize_node]: [NitrosNode] Exporting the final graph based on the negotiation results
[component_container_mt-1] [INFO] [1760091783.664535780] [detectnet_encoder.resize_node]: [NitrosNode] Wrote the final top level YAML graph to "/tmp/isaac_ros_nitros/graphs/KTKXRRSZYV/KTKXRRSZYV.yaml"
[component_container_mt-1] [INFO] [1760091783.664602120] [detectnet_encoder.resize_node]: [NitrosNode] Loading application
[component_container_mt-1] [INFO] [1760091783.668303459] [detectnet_encoder.resize_node]: [ResizeNode] postLoadGraphCallback().
[component_container_mt-1] [INFO] [1760091783.668369159] [detectnet_encoder.resize_node]: [NitrosNode] Initializing and running GXF graph
[component_container_mt-1] [INFO] [1760091783.671145349] [detectnet_encoder.resize_node]: [NitrosNode] Node was started
[component_container_mt-1] [INFO] [1760091783.908873730] [triton_node]: [NitrosNode] Starting post negotiation setup
[component_container_mt-1] [INFO] [1760091783.909008828] [triton_node]: [NitrosNode] Getting data format negotiation results
[component_container_mt-1] [INFO] [1760091783.909057586] [triton_node]: [NitrosSubscriber] Use the negotiated data format: "nitros_tensor_list_nchw_rgb_f32"
[component_container_mt-1] [INFO] [1760091783.909121846] [triton_node]: [NitrosPublisher] Negotiation ended with no results
[component_container_mt-1] [INFO] [1760091783.909160824] [triton_node]: [NitrosPublisher] Use only the compatible publisher: topic_name="/tensor_sub", data_format="nitros_tensor_list_nhwc_rgb_f32"
[component_container_mt-1] [INFO] [1760091783.909565558] [triton_node]: [NitrosNode] Exporting the final graph based on the negotiation results
[component_container_mt-1] [INFO] [1760091783.948073533] [triton_node]: [NitrosNode] Wrote the final top level YAML graph to "/tmp/isaac_ros_nitros/graphs/WCMQZQGOXT/WCMQZQGOXT.yaml"
[component_container_mt-1] [INFO] [1760091783.948128689] [triton_node]: [NitrosNode] Loading application
[component_container_mt-1] 2025-10-10 10:23:03.950 WARN  gxf/std/yaml_file_loader.cpp@1076: Using unregistered parameter 'dummy_rx' in component 'requester'.
[component_container_mt-1] [INFO] [1760091783.951176003] [triton_node]: In TritonNode postLoadGraphCallback().
[component_container_mt-1] [INFO] [1760091783.951323710] [triton_node]: [NitrosNode] Initializing and running GXF graph
[component_container_mt-1] [INFO] [1760091784.065417912] [detectnet_encoder.image_format_converter_node]: [NitrosNode] Starting post negotiation setup
[component_container_mt-1] [INFO] [1760091784.065544343] [detectnet_encoder.image_format_converter_node]: [NitrosNode] Getting data format negotiation results
[component_container_mt-1] [INFO] [1760091784.065632767] [detectnet_encoder.image_format_converter_node]: [NitrosPublisher] Use the negotiated data format: "nitros_image_rgb8"
[component_container_mt-1] [INFO] [1760091784.065673489] [detectnet_encoder.image_format_converter_node]: [NitrosSubscriber] Use the negotiated data format: "nitros_image_rgb8"
[component_container_mt-1] [INFO] [1760091784.065961854] [detectnet_encoder.image_format_converter_node]: [NitrosNode] Exporting the final graph based on the negotiation results
[component_container_mt-1] WARNING: infer_trtis_server.cpp:1217 NvDsTritonServerInit suggest to set model_control_mode:none. otherwise may cause unknow issues.
[component_container_mt-1] I1010 10:23:04.070078 43399 pinned_memory_manager.cc:277] "Pinned memory pool is created at '0x725f9a000000' with size 268435456"
[component_container_mt-1] I1010 10:23:04.075819 43399 cuda_memory_manager.cc:107] "CUDA memory pool is created on device 0 with size 67108864"
[component_container_mt-1] I1010 10:23:04.075828 43399 cuda_memory_manager.cc:107] "CUDA memory pool is created on device 1 with size 67108864"
[component_container_mt-1] [INFO] [1760091784.106845849] [detectnet_encoder.image_format_converter_node]: [NitrosNode] Wrote the final top level YAML graph to "/tmp/isaac_ros_nitros/graphs/SQBAFMMZQG/SQBAFMMZQG.yaml"
[component_container_mt-1] [INFO] [1760091784.107034720] [detectnet_encoder.image_format_converter_node]: [NitrosNode] Loading application
[component_container_mt-1] [INFO] [1760091784.107520256] [detectnet_decoder_node]: [NitrosNode] Starting post negotiation setup
[component_container_mt-1] [INFO] [1760091784.107622465] [detectnet_decoder_node]: [NitrosNode] Getting data format negotiation results
[component_container_mt-1] [INFO] [1760091784.107659411] [detectnet_decoder_node]: [NitrosPublisher] Negotiation ended with no results
[component_container_mt-1] [INFO] [1760091784.107692705] [detectnet_decoder_node]: [NitrosPublisher] Use only the compatible publisher: topic_name="/detectnet/detections", data_format="nitros_detection2_d_array"
[component_container_mt-1] [INFO] [1760091784.107726721] [detectnet_decoder_node]: [NitrosSubscriber] Negotiation ended with no results
[component_container_mt-1] [INFO] [1760091784.107755415] [detectnet_decoder_node]: [NitrosSubscriber] Use the compatible subscriber: topic_name="/tensor_sub", data_format="nitros_tensor_list_nchw_rgb_f32"
[component_container_mt-1] [INFO] [1760091784.108069304] [detectnet_decoder_node]: [NitrosNode] Exporting the final graph based on the negotiation results
[component_container_mt-1] [INFO] [1760091784.118385803] [detectnet_decoder_node]: [NitrosNode] Wrote the final top level YAML graph to "/tmp/isaac_ros_nitros/graphs/MWZZIEARZA/MWZZIEARZA.yaml"
[component_container_mt-1] [INFO] [1760091784.118489402] [detectnet_decoder_node]: [NitrosNode] Loading application
[component_container_mt-1] [INFO] [1760091784.176632388] [detectnet_encoder.crop_node]: [NitrosNode] Starting post negotiation setup
[component_container_mt-1] [INFO] [1760091784.176703214] [detectnet_encoder.crop_node]: [NitrosNode] Getting data format negotiation results
[component_container_mt-1] [INFO] [1760091784.176725003] [detectnet_encoder.crop_node]: [NitrosPublisher] Use the negotiated data format: "nitros_image_rgb8"
[component_container_mt-1] [INFO] [1760091784.176744080] [detectnet_encoder.crop_node]: [NitrosPublisher] Negotiation ended with no results
[component_container_mt-1] [INFO] [1760091784.176759714] [detectnet_encoder.crop_node]: [NitrosPublisher] Use only the compatible publisher: topic_name="/detectnet_encoder/crop/camera_info", data_format="nitros_camera_info"
[component_container_mt-1] [INFO] [1760091784.176776950] [detectnet_encoder.crop_node]: [NitrosSubscriber] Use the negotiated data format: "nitros_image_rgb8"
[component_container_mt-1] [INFO] [1760091784.176825607] [detectnet_encoder.crop_node]: [NitrosSubscriber] Use the negotiated data format: "nitros_camera_info"
[component_container_mt-1] [INFO] [1760091784.177035937] [detectnet_encoder.crop_node]: [NitrosNode] Exporting the final graph based on the negotiation results
[component_container_mt-1] [INFO] [1760091784.231542447] [detectnet_encoder.crop_node]: [NitrosNode] Wrote the final top level YAML graph to "/tmp/isaac_ros_nitros/graphs/GRITWKBGHR/GRITWKBGHR.yaml"
[component_container_mt-1] [INFO] [1760091784.231603767] [detectnet_encoder.crop_node]: [NitrosNode] Loading application
[component_container_mt-1] I1010 10:23:04.285076 43399 server.cc:604] 
[component_container_mt-1] +------------------+------+
[component_container_mt-1] | Repository Agent | Path |
[component_container_mt-1] +------------------+------+
[component_container_mt-1] +------------------+------+
[component_container_mt-1] 
[component_container_mt-1] I1010 10:23:04.285103 43399 server.cc:631] 
[component_container_mt-1] +---------+------+--------+
[component_container_mt-1] | Backend | Path | Config |
[component_container_mt-1] +---------+------+--------+
[component_container_mt-1] +---------+------+--------+
[component_container_mt-1] 
[component_container_mt-1] I1010 10:23:04.285111 43399 server.cc:674] 
[component_container_mt-1] +-------+---------+--------+
[component_container_mt-1] | Model | Version | Status |
[component_container_mt-1] +-------+---------+--------+
[component_container_mt-1] +-------+---------+--------+
[component_container_mt-1] 
[component_container_mt-1] I1010 10:23:04.354750 43399 metrics.cc:877] "Collecting metrics for GPU 0: NVIDIA RTX A4000"
[component_container_mt-1] I1010 10:23:04.354773 43399 metrics.cc:877] "Collecting metrics for GPU 1: NVIDIA RTX A4000"
[component_container_mt-1] I1010 10:23:04.367429 43399 metrics.cc:770] "Collecting CPU metrics"
[component_container_mt-1] I1010 10:23:04.367570 43399 tritonserver.cc:2598] 
[component_container_mt-1] +----------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
[component_container_mt-1] | Option                           | Value                                                                                                                                                                                                           |
[component_container_mt-1] +----------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
[component_container_mt-1] | server_id                        | triton                                                                                                                                                                                                          |
[component_container_mt-1] | server_version                   | 2.49.0                                                                                                                                                                                                          |
[component_container_mt-1] | server_extensions                | classification sequence model_repository model_repository(unload_dependents) schedule_policy model_configuration system_shared_memory cuda_shared_memory binary_tensor_data parameters statistics trace logging |
[component_container_mt-1] | model_repository_path[0]         | /workspaces/isaac_ros-dev/isaac_ros_assets/models                                                                                                                                                               |
[component_container_mt-1] | model_control_mode               | MODE_EXPLICIT                                                                                                                                                                                                   |
[component_container_mt-1] | strict_model_config              | 1                                                                                                                                                                                                               |
[component_container_mt-1] | model_config_name                |                                                                                                                                                                                                                 |
[component_container_mt-1] | rate_limit                       | OFF                                                                                                                                                                                                             |
[component_container_mt-1] | pinned_memory_pool_byte_size     | 268435456                                                                                                                                                                                                       |
[component_container_mt-1] | cuda_memory_pool_byte_size{0}    | 67108864                                                                                                                                                                                                        |
[component_container_mt-1] | cuda_memory_pool_byte_size{1}    | 67108864                                                                                                                                                                                                        |
[component_container_mt-1] | min_supported_compute_capability | 6.0                                                                                                                                                                                                             |
[component_container_mt-1] | strict_readiness                 | 1                                                                                                                                                                                                               |
[component_container_mt-1] | exit_timeout                     | 30                                                                                                                                                                                                              |
[component_container_mt-1] | cache_enabled                    | 0                                                                                                                                                                                                               |
[component_container_mt-1] +----------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
[component_container_mt-1] 
[component_container_mt-1] [INFO] [1760091784.368452496] [triton_node]: [NitrosNode] Node was started
[component_container_mt-1] [INFO] [1760091784.379012024] [detectnet_encoder.interleaved_to_planar_node]: [NitrosNode] Starting post negotiation setup
[component_container_mt-1] [INFO] [1760091784.379174413] [detectnet_encoder.interleaved_to_planar_node]: [NitrosNode] Getting data format negotiation results
[component_container_mt-1] I1010 10:23:04.379288 43399 model_lifecycle.cc:472] "loading: peoplenet:1"
[component_container_mt-1] [INFO] [1760091784.379246896] [detectnet_encoder.interleaved_to_planar_node]: [NitrosPublisher] Use the negotiated data format: "nitros_tensor_list_nchw_rgb_f32"
[component_container_mt-1] [INFO] [1760091784.380148044] [detectnet_encoder.interleaved_to_planar_node]: [NitrosSubscriber] Use the negotiated data format: "nitros_tensor_list_nhwc_rgb_f32"
[component_container_mt-1] [INFO] [1760091784.380481248] [detectnet_encoder.image_format_converter_node]: [ImageFormatConverterNode] postLoadGraphCallback().
[component_container_mt-1] [INFO] [1760091784.380529036] [detectnet_encoder.interleaved_to_planar_node]: [NitrosNode] Exporting the final graph based on the negotiation results
[component_container_mt-1] [INFO] [1760091784.394153076] [detectnet_encoder.image_format_converter_node]: [NitrosNode] Initializing and running GXF graph
[component_container_mt-1] [INFO] [1760091784.398819978] [detectnet_encoder.interleaved_to_planar_node]: [NitrosNode] Wrote the final top level YAML graph to "/tmp/isaac_ros_nitros/graphs/KUMAAIHRSB/KUMAAIHRSB.yaml"
[component_container_mt-1] [INFO] [1760091784.398892301] [detectnet_encoder.interleaved_to_planar_node]: [NitrosNode] Loading application
[component_container_mt-1] I1010 10:23:04.401774 43399 tensorrt.cc:65] "TRITONBACKEND_Initialize: tensorrt"
[component_container_mt-1] I1010 10:23:04.401792 43399 tensorrt.cc:75] "Triton TRITONBACKEND API version: 1.19"
[component_container_mt-1] I1010 10:23:04.401798 43399 tensorrt.cc:81] "'tensorrt' TRITONBACKEND API version: 1.19"
[component_container_mt-1] I1010 10:23:04.401803 43399 tensorrt.cc:105] "backend configuration:\n{\"cmdline\":{\"auto-complete-config\":\"false\",\"backend-directory\":\"/opt/tritonserver/backends\",\"min-compute-capability\":\"6.000000\",\"default-max-batch-size\":\"4\"}}"
[component_container_mt-1] [INFO] [1760091784.402684278] [detectnet_encoder.image_format_converter_node]: [NitrosNode] Node was started
[component_container_mt-1] [INFO] [1760091784.404251276] [detectnet_encoder.interleaved_to_planar_node]: In InterleavedToPlanarNode postLoadGraphCallback().
[component_container_mt-1] [INFO] [1760091784.404349078] [detectnet_decoder_node]: [NitrosNode] Initializing and running GXF graph
[component_container_mt-1] [INFO] [1760091784.404356002] [detectnet_encoder.interleaved_to_planar_node]: [NitrosNode] Initializing and running GXF graph
[component_container_mt-1] [INFO] [1760091784.404366576] [detectnet_encoder.crop_node]: [CropNode] postLoadGraphCallback().
[component_container_mt-1] [INFO] [1760091784.405597651] [detectnet_decoder_node]: [NitrosNode] Node was started
[component_container_mt-1] [INFO] [1760091784.409335377] [detectnet_encoder.interleaved_to_planar_node]: [NitrosNode] Node was started
[component_container_mt-1] [INFO] [1760091784.409380334] [detectnet_encoder.crop_node]: [NitrosNode] Initializing and running GXF graph
[component_container_mt-1] I1010 10:23:04.410006 43399 tensorrt.cc:231] "TRITONBACKEND_ModelInitialize: peoplenet (version 1)"
[component_container_mt-1] [INFO] [1760091784.413001149] [detectnet_encoder.crop_node]: [NitrosNode] Node was started
[component_container_mt-1] I1010 10:23:04.423953 43399 tensorrt.cc:297] "TRITONBACKEND_ModelInstanceInitialize: peoplenet_0 (GPU device 0)"
[component_container_mt-1] [INFO] [1760091784.437221677] [detectnet_encoder.reshape_node]: [NitrosNode] Starting post negotiation setup
[component_container_mt-1] [INFO] [1760091784.437260751] [detectnet_encoder.reshape_node]: [NitrosNode] Getting data format negotiation results
[component_container_mt-1] [INFO] [1760091784.437276675] [detectnet_encoder.reshape_node]: [NitrosPublisher] Use the negotiated data format: "nitros_tensor_list_nchw_rgb_f32"
[component_container_mt-1] [INFO] [1760091784.437289024] [detectnet_encoder.reshape_node]: [NitrosSubscriber] Use the negotiated data format: "nitros_tensor_list_nchw_rgb_f32"
[component_container_mt-1] [INFO] [1760091784.437446525] [detectnet_encoder.reshape_node]: [NitrosNode] Exporting the final graph based on the negotiation results
[component_container_mt-1] [INFO] [1760091784.439780919] [detectnet_encoder.reshape_node]: [NitrosNode] Wrote the final top level YAML graph to "/tmp/isaac_ros_nitros/graphs/VZPFGOAFBM/VZPFGOAFBM.yaml"
[component_container_mt-1] [INFO] [1760091784.439808764] [detectnet_encoder.reshape_node]: [NitrosNode] Loading application
[component_container_mt-1] [INFO] [1760091784.440821786] [detectnet_encoder.reshape_node]: In ReshapeNode postLoadGraphCallback().
[component_container_mt-1] [INFO] [1760091784.440874957] [detectnet_encoder.reshape_node]: [NitrosNode] Initializing and running GXF graph
[component_container_mt-1] [INFO] [1760091784.442027220] [detectnet_encoder.reshape_node]: [NitrosNode] Node was started
[component_container_mt-1] I1010 10:23:04.535379 43399 logging.cc:46] "Loaded engine size: 86 MiB"
[component_container_mt-1] I1010 10:23:04.701156 43399 logging.cc:46] "[MemUsageChange] TensorRT-managed allocation in IExecutionContext creation: CPU +0, GPU +797, now: CPU 0, GPU 881 (MiB)"
[component_container_mt-1] I1010 10:23:04.708760 43399 instance_state.cc:186] "Created instance peoplenet_0 on GPU 0 with stream priority 0 and optimization profile default[0];"
[component_container_mt-1] I1010 10:23:04.721677 43399 tensorrt.cc:297] "TRITONBACKEND_ModelInstanceInitialize: peoplenet_0 (GPU device 1)"
[component_container_mt-1] I1010 10:23:04.837268 43399 logging.cc:46] "Loaded engine size: 86 MiB"
[component_container_mt-1] I1010 10:23:04.864184 43399 logging.cc:46] "[MemUsageChange] TensorRT-managed allocation in IExecutionContext creation: CPU +0, GPU +797, now: CPU 0, GPU 1763 (MiB)"
[component_container_mt-1] I1010 10:23:04.864527 43399 instance_state.cc:186] "Created instance peoplenet_0 on GPU 1 with stream priority 0 and optimization profile default[0];"
[component_container_mt-1] I1010 10:23:04.864743 43399 model_lifecycle.cc:839] "successfully loaded 'peoplenet'"
[component_container_mt-1] INFO: infer_simple_runtime.cpp:71 TrtISBackend id:323 initialized model: peoplenet
[component_container_mt-1] 2025-10-10 10:23:04.924 WARN  extensions/triton/inferencers/triton_inferencer_impl.cpp@508: Incomplete inference; response appeared out of order; invalid: index = 0
[component_container_mt-1] 2025-10-10 10:23:04.924 WARN  extensions/triton/inferencers/triton_inferencer_impl.cpp@510: Inference appeared out of order
[ERROR] [component_container_mt-1]: process has died [pid 43399, exit code -6, cmd '/opt/ros/humble/lib/rclcpp_components/component_container_mt --ros-args -r __node:=detectnet_container -r __ns:=/detectnet_container'].

``` 

在启动命令中, 限制只使用1个GPU: 

```
CUDA_VISIBLE_DEVICES=1 ros2 launch isaac_ros_detectnet isaac_ros_detectnet_isaac_sim.launch.py
``` 

日志恢复正常

调整机器人角度后, 可以正确显示: 

![image2025-10-10 18:34:12.png](/assets/01KJC02TAJY7A07WBB6D01V89F/image2025-10-10%2018%3A34%3A12.png)

额外: 

当让机器人动起来后, 插件默认打开的rqt会不显示结果. 

可以重新打开一个rqt, 可以正确显示

# 代码分析

代码: <https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_object_detection/tree/main>

流程: 

```
Isaac Sim仿真相机输出
│
├─ /front_stereo_camera/left/image_rect_color ────┐
│                                                  │
└─ /front_stereo_camera/left/camera_info ─────────┤
                                                   ↓
                                    ┌──────────────────────────────┐
                                    │ ① 图像缩放节点                │
                                    │   image_resize_node_left     │
                                    │   (ResizeNode)               │
                                    ├──────────────────────────────┤
                                    │ 参数:                        │
                                    │  • output_width: 960         │
                                    │  • output_height: 544        │
                                    │  • encoding: 'rgb8'          │
                                    └──────────────────────────────┘
                                                   │
                    ┌──────────────────────────────┴──────────────────────────────┐
                    ↓                                                             ↓
    /front_stereo_camera/left/image_resize                  /front_stereo_camera/left/camera_info_resize
                    │                                                             │
                    └──────────────────────────────┬──────────────────────────────┘
                                                   ↓
                                    ┌──────────────────────────────┐
                                    │ ② DNN图像编码器              │
                                    │   detectnet_encoder          │
                                    │   (DNN Image Encoder)        │
                                    ├──────────────────────────────┤
                                    │ 参数:                        │
                                    │  • input_size: 960×544       │
                                    │  • network_size: 960×544     │
                                    │  • image_mean: [0,0,0]       │
                                    │  • image_stddev: [1,1,1]     │
                                    │  • enable_padding: False     │
                                    │  • format: NCHW RGB F32      │
                                    └──────────────────────────────┘
                                                   │
                                                   ↓
                                            /tensor_pub
                                          (tensor消息)
                                                   │
                                                   ↓
                                    ┌──────────────────────────────┐
                                    │ ③ Triton推理节点             │
                                    │   triton_node                │
                                    │   (TritonNode)               │
                                    ├──────────────────────────────┤
                                    │ 参数:                        │
                                    │  • model: 'peoplenet'        │
                                    │  • input_binding:            │
                                    │    'input_1:0'               │
                                    │  • output_bindings:          │
                                    │    - output_cov/Sigmoid:0    │
                                    │    - output_bbox/BiasAdd:0   │
                                    │  • input_format: NCHW        │
                                    │  • output_format: NHWC       │
                                    └──────────────────────────────┘
                                                   │
                    ┌──────────────────────────────┴──────────────────────────────┐
                    ↓                                                             ↓
            output_cov tensor                                            output_bbox tensor
            [3, 34, 60]                                                  [12, 34, 60]
            (置信度图)                                                    (边界框坐标)
                    │                                                             │
                    └──────────────────────────────┬──────────────────────────────┘
                                                   ↓
                                            tensor_sub
                                          (tensorlist)
                                                   │
                                                   ↓
                                    ┌──────────────────────────────┐
                                    │ ④ DetectNet解码器            │
                                    │   detectnet_decoder_node     │
                                    │   (DetectNetDecoderNode)     │
                                    ├──────────────────────────────┤
                                    │ 参数 (params_isaac_sim.yaml):│
                                    │  • model: 'detectnet'        │
                                    │  • bbox_scale: 35.0          │
                                    │  • bbox_offset: 0.0          │
                                    │  • confidence_threshold: 0.35│
                                    │  • min_bbox_area: 10000.0    │
                                    │  • dbscan_clustering: true   │
                                    │  • dbscan_eps: 0.7           │
                                    │  • label_list:               │
                                    │    [person, bag, face]       │
                                    └──────────────────────────────┘
                                                   │
                                                   ↓
                                    /detectnet/detections
                                    (Detection2DArray消息)
                                                   │
                                                   ↓
                                    ┌──────────────────────────────┐
                                    │ ⑤ 可视化节点                 │
                                    │   detectnet_visualizer       │
                                    │   (Python脚本)               │
                                    ├──────────────────────────────┤
                                    │ 输入:                        │
                        ┌───────────┤  • image: detectnet_encoder/ │
                        │           │    resize/image              │
/detectnet_encoder/     │           │  • detections: detectnet/    │
resize/image ───────────┘           │    detections                │
(原始图像)                          │ 功能:                        │
                                    │  • 在图像上绘制检测框        │
                                    │  • 使用OpenCV rectangle      │
                                    │  • 颜色: 绿色 (0,255,0)      │
                                    └──────────────────────────────┘
                                                   │
                                                   ↓
                                    /detectnet_processed_image
                                    (带检测框的图像)
                                                   │
                                                   ↓
                                    ┌──────────────────────────────┐
                                    │ ⑥ RQT图像查看器              │
                                    │   rqt_image_view             │
                                    │   (GUI窗口)                  │
                                    ├──────────────────────────────┤
                                    │ 参数:                        │
                                    │  • topic:                    │
                                    │    /detectnet_processed_image│
                                    │  • encoding: 'rgb8'          │
                                    └──────────────────────────────┘
                                                   │
                                                   ↓
                                            显示到屏幕
                                            (用户可见)
```
