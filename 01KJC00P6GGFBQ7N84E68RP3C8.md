---
title: 20250926 - 搭建Issac环境
confluence_page_id: 4358648
created_at: 2025-09-26T16:05:05+00:00
updated_at: 2025-09-28T07:59:40+00:00
---

配置nvidia docker:

  - 参考文档: <https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html>

```
mkdir /etc/apt/sources.list.d

curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

apt-get install -y nvidia-container-toolkit

export NVIDIA_CONTAINER_TOOLKIT_VERSION=1.17.8-1
  sudo apt-get install -y \
      nvidia-container-toolkit=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
      nvidia-container-toolkit-base=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
      libnvidia-container-tools=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
      libnvidia-container1=${NVIDIA_CONTAINER_TOOLKIT_VERSION}

nvidia-ctk runtime configure --runtime=docker

systemctl restart docker

nvidia-ctk config --set nvidia-container-cli.no-cgroups --in-place
``` 

参考文档: <https://docs.isaacsim.omniverse.nvidia.com/latest/installation/install_container.html#isaac-sim-setup-remote-headless-container>

启动容器

```
docker pull nvcr.io/nvidia/isaac-sim:5.0.0   docker run --name isaac-sim --entrypoint bash -it --runtime=nvidia --gpus device=0 -e "ACCEPT_EULA=Y" --network=host --privileged \ -e "PRIVACY_CONSENT=Y" \ -v /data/huangyan/isaac-sim/cache/kit:/isaac-sim/kit/cache:rw \ -v /data/huangyan/isaac-sim/cache/ov:/root/.cache/ov:rw \ -v /data/huangyan/isaac-sim/cache/pip:/root/.cache/pip:rw \ -v /data/huangyan/isaac-sim/cache/glcache:/root/.cache/nvidia/GLCache:rw \ -v /data/huangyan/isaac-sim/cache/computecache:/root/.nv/ComputeCache:rw \ -v /data/huangyan/isaac-sim/logs:/root/.nvidia-omniverse/logs:rw \ -v /data/huangyan/isaac-sim/data:/root/.local/share/ov/data:rw \ -v /data/huangyan/isaac-sim/documents:/root/Documents:rw \ nvcr.io/nvidia/isaac-sim:5.0.0   在容器内: ./runheadless.sh -v   出现日志: [109.653s] Isaac Sim Full Streaming App is loaded.
``` 

在mac上下载客户端

将远程机器的49100和47998端口暴露出来 (用cursor)

用客户端连接127.0.0.1

发现失败, 找到了文档: 

  - Livestreaming is not supported when Isaac Sim is run on the A100 GPU. NVENC (NVIDIA Encoder) is required for livestreaming and is not included in the A100 GPU

更换了4090显卡, 仍然会报错: 

```
onClientEventRaised: NVST_CCE_DISCONNECTED when m_connectionCount 4294967291 != 1
``` 

![image2025-9-27 22:43:18.png](/assets/01KJC00P6GGFBQ7N84E68RP3C8/image2025-9-27%2022%3A43%3A18.png)

转而安装workstation版本: 

```
# 参考文档: https://docs.isaacsim.omniverse.nvidia.com/latest/installation/install_workstation.html
 
apt-get install -y libxt6 libglu1-mesa libxrandr2
 
wget https://download.isaacsim.omniverse.nvidia.com/isaac-sim-standalone-5.0.0-linux-x86_64.zip
 
mkdir /data/huangyan/isaac-sim
cd /data/huangyan/isaac-sim
unzip ...
 
./post_install.sh
bash isaac-sim.streaming.sh --allow-root
 
``` 

会报错, 一些cuda的c++库的版本问题

unset LD_LIBRARY_PATH 后可以避开报错. 但仍然无法远程连接

TODO: 参考: <https://cloud.tencent.com/developer/article/2547244>

![image2025-9-28 1:43:1.png](/assets/01KJC00P6GGFBQ7N84E68RP3C8/image2025-9-28%201%3A43%3A1.png)

最终使用了阿里云的 无影云电脑, 安装issac sim进行使用
