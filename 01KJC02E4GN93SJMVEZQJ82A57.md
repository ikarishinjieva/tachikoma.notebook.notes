---
title: 20251010 - 整理Isaac Sim和ROS2 demo的环境安装和运行指令
confluence_page_id: 4358810
created_at: 2025-10-10T07:09:24+00:00
updated_at: 2025-11-07T08:50:01+00:00
---

# 目标

使用公司服务器, 安装Isaac Sim和ROS2 demo, 本地笔记本远程连接到公司服务器

# 配置远程连接 NICE DCV

原文档: [20251006 - 尝试将Isaac Sim和WebRTC连通]

"虚拟会话（Virtual session）+ DCV GL（GPU 共享）":

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

或可以指定驱动: 

```
报错日志: 
 
huangyan@dell-136:~$ DISPLAY=:0 glxinfo -B 
name of display: :0
E Oct 15 05:07:36:933014 1249320:1249320 DCVGL (char* findSystemGLXVendor()): Trying to open Xdcv (:0.0) as local display, bailing out
E Oct 15 05:07:36:933082 1249320:1249320 DCVGL (int __glx_Main(uint32_t, const __GLXapiExports*, __GLXvendorInfo*, __GLXapiImports*)): No supported glVND vendor found on local display
display: :0  screen: 0
direct rendering: Yes
Extended renderer info (GLX_MESA_query_renderer):
    Vendor: Mesa (0xffffffff)
    Device: llvmpipe (LLVM 20.1.2, 256 bits) (0xffffffff)
    Version: 25.0.7
    Accelerated: no
    Video memory: 257347MB
    Unified memory: yes
    Preferred profile: core (0x1)
    Max core profile version: 4.5
    Max compat profile version: 4.5
    Max GLES1 profile version: 1.1
    Max GLES[23] profile version: 3.2
Memory info (GL_ATI_meminfo):
    VBO free memory - total: 0 MB, largest block: 0 MB
    VBO free aux. memory - total: 249992 MB, largest block: 249992 MB
    Texture free memory - total: 0 MB, largest block: 0 MB
    Texture free aux. memory - total: 249992 MB, largest block: 249992 MB
    Renderbuffer free memory - total: 0 MB, largest block: 0 MB
    Renderbuffer free aux. memory - total: 249992 MB, largest block: 249992 MB
Memory info (GL_NVX_gpu_memory_info):
    Dedicated video memory: 0 MB
    Total available memory: 257347 MB
    Currently available dedicated video memory: 0 MB
OpenGL vendor string: Mesa
OpenGL renderer string: llvmpipe (LLVM 20.1.2, 256 bits)
OpenGL core profile version string: 4.5 (Core Profile) Mesa 25.0.7-0ubuntu0.24.04.2
OpenGL core profile shading language version string: 4.50
OpenGL core profile context flags: (none)
OpenGL core profile profile mask: core profile

OpenGL version string: 4.5 (Compatibility Profile) Mesa 25.0.7-0ubuntu0.24.04.2
OpenGL shading language version string: 4.50
OpenGL context flags: (none)
OpenGL profile mask: compatibility profile

OpenGL ES profile version string: OpenGL ES 3.2 Mesa 25.0.7-0ubuntu0.24.04.2
OpenGL ES profile shading language version string: OpenGL ES GLSL ES 3.20

 
通过 __GLX_VENDOR_LIBRARY_NAME 指定驱动
 
huangyan@dell-136:~$ export __GLX_VENDOR_LIBRARY_NAME=nvidia
huangyan@dell-136:~$ DISPLAY=:0 glxinfo -B 
name of display: :0
display: :0  screen: 0
direct rendering: Yes
Memory info (GL_NVX_gpu_memory_info):
    Dedicated video memory: 16376 MB
    Total available memory: 16376 MB
    Currently available dedicated video memory: 15737 MB
OpenGL vendor string: NVIDIA Corporation
OpenGL renderer string: NVIDIA RTX A4000/PCIe/SSE2
OpenGL core profile version string: 4.6.0 NVIDIA 565.57.01
OpenGL core profile shading language version string: 4.60 NVIDIA
OpenGL core profile context flags: (none)
OpenGL core profile profile mask: core profile

OpenGL version string: 4.6.0 NVIDIA 565.57.01
OpenGL shading language version string: 4.60 NVIDIA
OpenGL context flags: (none)
OpenGL profile mask: (none)

OpenGL ES profile version string: OpenGL ES 3.2 NVIDIA 565.57.01
OpenGL ES profile shading language version string: OpenGL ES GLSL ES 3.20

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

# 安装Isaac Sim

原文档: [20251003 - 重新配置isaac和demo的笔记本]

在服务器上启动图形会话: 

```
10.186.16.136:
dcv create-session test-issac
  
连接到桌面:
- 使用浏览器: 10.186.16.136:8443
- 或在本地使用 DCV viewer
``` 

运行: 

```
# 参考文档: https://docs.isaacsim.omniverse.nvidia.com/latest/installation/install_workstation.html
  
wget https://download.isaacsim.omniverse.nvidia.com/isaac-sim-standalone-5.0.0-linux-x86_64.zip

# 解压

sudo apt-get install -y libxt6 libglu1-mesa libxrandr2
  
./post_install.sh
bash isaac-sim.sh
``` 

运行时报错:

```
An input-output memory management unit (IOMMU) appears to be enabled on this system.
On bare-metal Linux systems, CUDA and the display driver do not support IOMMU-enabled PCIe peer to peer memory copy.
If you are on a bare-metal Linux system, please disable the IOMMU. Otherwise you risk image corruption and program instability.
This typically can be controlled via BIOS settings (Intel Virtualization Technology for Directed I/O (VT-d) or AMD I/O Virtualization Technology (AMD-Vi)) and kernel parameters (iommu, intel_iommu, amd_iommu).
Note that in virtual machines with GPU pass-through (vGPU) the IOMMU needs to be enabled.
Since we can not reliably detect whether this system is bare-metal or a virtual machine, we show this warning in any case when an IOMMU appears to be enabled.
``` 

在os中, 编辑/etc/default/grub, 修改: 

```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash intel_iommu=off"
``` 

重启后， 通过./isaac-sim.sh 可以正确启动

# 安装 ROS2 DEMO

原文档: [20251003 - 重新配置isaac和demo的笔记本](<http://8.134.54.170:8330/pages/viewpage.action?pageId=4358700>) 中的"安装Demo"章节

在服务器上: 

```
# 下载dev容器 (具体版本, 需查看下面的run_dev.sh脚本, 或者直接执行脚本, 会自动pull)
docker pull nvcr.io/nvidia/isaac/ros:x86_64-ros2_humble_6f2a6bddf70fcd928f08e874635efe43

# 安装nvidia  container toolkit
 
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
 
 
sudo apt-get update
 
export NVIDIA_CONTAINER_TOOLKIT_VERSION=1.17.8-1
  sudo apt-get install -y \
      nvidia-container-toolkit=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
      nvidia-container-toolkit-base=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
      libnvidia-container-tools=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
      libnvidia-container1=${NVIDIA_CONTAINER_TOOLKIT_VERSION}

sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl daemon-reload && sudo systemctl restart docker

# 安装 git lfs
sudo apt-get install git-lfs
git lfs install --skip-repo
 

# 创建ROS工作区
mkdir -p  ~/Code/isaac_ros-dev/src
echo "export ISAAC_ROS_WS=${HOME}/Code/isaac_ros-dev/" >> ~/.bashrc
source ~/.bashrc

# 准备 isaac ROS common

cd ${ISAAC_ROS_WS}/src && \
  git clone -b release-3.2 https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_common.git isaac_ros_common
``` 

修改文件./src/isaac_ros_common/scripts/run_dev.sh:

```
删除 docker run -it --rm 中的 --rm
将docker rm 改成 docker start
``` 

进入容器:

```
cd $ISAAC_ROS_WS && ./src/isaac_ros_common/scripts/run_dev.sh
``` 

使用最小验证程序: 

```
#include <iostream>
#include <cuda_runtime.h>

int main() {
    void* d_ptr = nullptr;
    cudaStream_t stream;

    // 1. 创建一个CUDA流
    cudaError_t stream_err = cudaStreamCreate(&stream);
    if (stream_err != cudaSuccess) {
        std::cerr << "Failed to create CUDA stream: " << cudaGetErrorString(stream_err) << std::endl;
        return -1;
    }
    std::cout << "CUDA stream created successfully." << std::endl;

    // 2. 尝试调用 cudaMallocAsync
    std::cout << "Attempting to call cudaMallocAsync..." << std::endl;
    cudaError_t err = cudaMallocAsync(&d_ptr, 1024, stream);

    // 3. 检查结果
    if (err == cudaSuccess) {
        std::cout << "SUCCESS: cudaMallocAsync executed without error." << std::endl;
        cudaFreeAsync(d_ptr, stream);
    } else {
        std::cerr << "FAILURE: cudaMallocAsync returned error code " << err << ": " << cudaGetErrorString(err) << std::endl;
    }

    cudaStreamDestroy(stream);
    return 0;
}
``` 

编译: nvcc test_cuda.cpp -o test_cuda

运行后正确输出: 

```
admin@tachikoma-ThinkBook-16p-G5-IRX:/workspaces/test$  nvcc test_cuda.cpp -o test_cuda
admin@tachikoma-ThinkBook-16p-G5-IRX:/workspaces/test$ ./test_cuda 
CUDA stream created successfully.
Attempting to call cudaMallocAsync...
SUCCESS: cudaMallocAsync executed without error.
``` 

注意: 下面的"需要解决nvidia-560的问题", 需要升级宿主机器的nvidia driver > 550

进入容器配置ROS: 

```
# clone manipulator
cd ${ISAAC_ROS_WS}/src && \
  git clone --recursive -b release-3.2 https://github.com/NVIDIA-ISAAC-ROS/isaac_manipulator.git isaac_manipulator
 
# Clone the Isaac ROS fork of ros2_robotiq_gripper and tylerjw/serial
cd ${ISAAC_ROS_WS}/src && \
  git clone --recursive https://github.com/NVIDIA-ISAAC-ROS/ros2_robotiq_gripper && \
  git clone -b ros2 https://github.com/tylerjw/serial
 
# Building dependencies:
cd ${ISAAC_ROS_WS}
colcon build --symlink-install --packages-select-regex robotiq* serial --cmake-args "-DBUILD_TESTING=OFF" && \
source install/setup.bash

# 检查AMENT_PREFIX_PATH, 应编译目录优先, 例如: /workspaces/isaac_ros-dev/install/robotiq_hardware_tests:/workspaces/isaac_ros-dev/install/robotiq_driver:/workspaces/isaac_ros-dev/install/serial:/workspaces/isaac_ros-dev/install/robotiq_description:/workspaces/isaac_ros-dev/install/robotiq_controllers:/opt/ros/humble

# 设置环境变量
export FASTRTPS_DEFAULT_PROFILES_FILE=/usr/local/share/middleware_profiles/rtps_udp_profile.xml

# 安装 DEMO

(配置好国外代理, 否则会报错, nvidia的国内站点文件有问题)

sudo apt-get update
 
sudo apt-get install -y ros-humble-isaac-manipulator-bringup ros-humble-isaac-manipulator-pick-and-place
 
(注意, sudo会掉环境变量, 需要sudo bash后, 重新设置代理的环境变量, 再执行sudo命令)

# 需要解决nvidia-560的问题
以上apt会引入nvidia-560的包， 需要将宿主机的nvidia版本升级到560以上 （实际是570），容器内才能正确运行 
``` 
    
    
      
      
    配置视觉模型

```
 
# SyntheticaDETR
mkdir -p ${ISAAC_ROS_WS}/isaac_ros_assets/models/synthetica_detr && \
cd ${ISAAC_ROS_WS}/isaac_ros_assets/models/synthetica_detr && \
wget 'https://api.ngc.nvidia.com/v2/models/nvidia/isaac/synthetica_detr/versions/1.0.0_onnx/files/sdetr_grasp.onnx'
 
/usr/src/tensorrt/bin/trtexec --onnx=${ISAAC_ROS_WS}/isaac_ros_assets/models/synthetica_detr/sdetr_grasp.onnx --saveEngine=${ISAAC_ROS_WS}/isaac_ros_assets/models/synthetica_detr/sdetr_grasp.plan
 
# FoundationPose
 
mkdir -p ${ISAAC_ROS_WS}/isaac_ros_assets/models/foundationpose && \
cd ${ISAAC_ROS_WS}/isaac_ros_assets/models/foundationpose && \
wget 'https://api.ngc.nvidia.com/v2/models/nvidia/isaac/foundationpose/versions/1.0.0_onnx/files/refine_model.onnx' -O refine_model.onnx && \
wget 'https://api.ngc.nvidia.com/v2/models/nvidia/isaac/foundationpose/versions/1.0.0_onnx/files/score_model.onnx' -O score_model.onnx
 
/usr/src/tensorrt/bin/trtexec --onnx=${ISAAC_ROS_WS}/isaac_ros_assets/models/foundationpose/refine_model.onnx --saveEngine=${ISAAC_ROS_WS}/isaac_ros_assets/models/foundationpose/refine_trt_engine.plan --minShapes=input1:1x160x160x6,input2:1x160x160x6 --optShapes=input1:1x160x160x6,input2:1x160x160x6 --maxShapes=input1:42x160x160x6,input2:42x160x160x6
 
/usr/src/tensorrt/bin/trtexec --onnx=${ISAAC_ROS_WS}/isaac_ros_assets/models/foundationpose/score_model.onnx --saveEngine=${ISAAC_ROS_WS}/isaac_ros_assets/models/foundationpose/score_trt_engine.plan --minShapes=input1:1x160x160x6,input2:1x160x160x6 --optShapes=input1:1x160x160x6,input2:1x160x160x6 --maxShapes=input1:252x160x160x6,input2:252x160x160x6
 
 
# Mac and Cheese texture and mesh
 
sudo apt-get install -y curl jq tar
 
NGC_ORG="nvidia"
NGC_TEAM="isaac"
PACKAGE_NAME="isaac_ros_foundationpose"
NGC_RESOURCE="isaac_ros_foundationpose_assets"
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
 
(本命令因为网络不一定稳定, 需要检查最后的输出文件正确)
``` 

# 运行

在10.186.16.136服务器上: 

```
10.186.16.136:
dcv create-session test-issac
   
使用 DCV viewer 连接到桌面
``` 

在桌面上启动Isaac Sim: 

```
cd /data/huangyan/isaac-sim
export http_proxy=http://10.186.16.136:7890
export https_proxy=http://10.186.16.136:7890
export ROS_DOMAIN_ID=99
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
export ROS_DISTRO=humble
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/data/huangyan/isaac-sim/exts/isaacsim.ros2.bridge/humble/lib
export __GLX_VENDOR_LIBRARY_NAME=nvidia
export DISPLAY=:0
./isaac-sim.sh
``` 

在服务器上进入容器: 

```
export ISAAC_ROS_WS=/data/huangyan/isaac_ros-dev/
cd $ISAAC_ROS_WS && ./src/isaac_ros_common/scripts/run_dev.sh
``` 

在容器内, 准备ROS2 demo的运行环境: 

```
export http_proxy=http://10.186.16.136:7890
export https_proxy=http://10.186.16.136:7890
cd ${ISAAC_ROS_WS}
source install/setup.bash
export ROS_DOMAIN_ID=99
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
 
# 此处可以执行ROS2 demo的具体命令
ros2 node list --no-daemon
``` 

在桌面上进入容器, 运行rviz2:

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
rviz2 -d $(ros2 pkg prefix isaac_ros_apriltag --share)/rviz/default.rviz
```
