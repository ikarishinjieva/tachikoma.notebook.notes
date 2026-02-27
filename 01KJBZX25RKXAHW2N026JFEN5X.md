---
title: 20250809 - A100 8卡物理机 安装
confluence_page_id: 4161940
created_at: 2025-08-09T04:56:25+00:00
updated_at: 2025-08-09T06:37:45+00:00
---

# 8卡配置

8卡A100, NVlink

```
root@jtkj003:~# nvidia-smi topo -m
	GPU0	GPU1	GPU2	GPU3	GPU4	GPU5	GPU6	GPU7	NIC0	NIC1	NIC2	NIC3	CPU Affinity	NUMA Affinity	GPU NUMA ID
GPU0	 X 	NV12	NV12	NV12	NV12	NV12	NV12	NV12	PXB	PXB	SYS	SYS	0-31,64-95	0		N/A
GPU1	NV12	 X 	NV12	NV12	NV12	NV12	NV12	NV12	PXB	PXB	SYS	SYS	0-31,64-95	0		N/A
GPU2	NV12	NV12	 X 	NV12	NV12	NV12	NV12	NV12	SYS	SYS	PXB	PXB	0-31,64-95	0		N/A
GPU3	NV12	NV12	NV12	 X 	NV12	NV12	NV12	NV12	SYS	SYS	PXB	PXB	0-31,64-95	0		N/A
GPU4	NV12	NV12	NV12	NV12	 X 	NV12	NV12	NV12	SYS	SYS	SYS	SYS	32-63,96-127	1		N/A
GPU5	NV12	NV12	NV12	NV12	NV12	 X 	NV12	NV12	SYS	SYS	SYS	SYS	32-63,96-127	1		N/A
GPU6	NV12	NV12	NV12	NV12	NV12	NV12	 X 	NV12	SYS	SYS	SYS	SYS	32-63,96-127	1		N/A
GPU7	NV12	NV12	NV12	NV12	NV12	NV12	NV12	 X 	SYS	SYS	SYS	SYS	32-63,96-127	1		N/A
NIC0	PXB	PXB	SYS	SYS	SYS	SYS	SYS	SYS	 X 	PXB	SYS	SYS
NIC1	PXB	PXB	SYS	SYS	SYS	SYS	SYS	SYS	PXB	 X 	SYS	SYS
NIC2	SYS	SYS	PXB	PXB	SYS	SYS	SYS	SYS	SYS	SYS	 X 	PIX
NIC3	SYS	SYS	PXB	PXB	SYS	SYS	SYS	SYS	SYS	SYS	PIX	 X

Legend:

  X    = Self
  SYS  = Connection traversing PCIe as well as the SMP interconnect between NUMA nodes (e.g., QPI/UPI)
  NODE = Connection traversing PCIe as well as the interconnect between PCIe Host Bridges within a NUMA node
  PHB  = Connection traversing PCIe as well as a PCIe Host Bridge (typically the CPU)
  PXB  = Connection traversing multiple PCIe bridges (without traversing the PCIe Host Bridge)
  PIX  = Connection traversing at most a single PCIe bridge
  NV#  = Connection traversing a bonded set of # NVLinks

NIC Legend:

  NIC0: mlx5_0
  NIC1: mlx5_1
  NIC2: mlx5_2
  NIC3: mlx5_3
``` 

# 配置环境

配置磁盘

```
mkfs.ext4 /dev/nvme0n1
mkdir /data
 
编辑/etc/fstab, 增加: 
UUID=cbcfec6f-213a-440f-a599-8b795bdc5610 /data ext4 defaults 0 0

mount -a 
``` 

从之前的服务器上, scp conda 的安装脚本, 然后安装conda环境: 

```
conda create -n huangyan python=3.10.18
conda activate huangyan
``` 

安装clash-for-linux (从其他机器复制)

安装ms-swift (按手册)

配置git ssh key

下载代码: HTTPS_PROXY=<http://127.0.0.1:7890> git clone [git@github.com](<mailto:git@github.com>):ikarishinjieva/sql-ai-2.git

下载模型: 

```
pip install modelscope -i https://pypi.tuna.tsinghua.edu.cn/simple
modelscope download --model Qwen/Qwen3-8B
``` 

设置sshd, 配置:

```
TCPKeepAlive yes
ClientAliveInterval 30
ClientAliveCountMax 3
``` 

单独安装一些依赖:

```
pip install flashinfer-python==0.2.9 -i https://pypi.tuna.tsinghua.edu.cn/simple
pip install vllm -i https://pypi.tuna.tsinghua.edu.cn/simple
pip install math_verify==0.5.2 -i https://pypi.tuna.tsinghua.edu.cn/simple
pip install flash-attn --no-build-isolation -i https://pypi.tuna.tsinghua.edu.cn/simple
``` 

可以开始训练

# 性能对比

使用 [20250801 - 进行SQL优化的GRPO - 缩小训练效果和校验效果的差异] 的尝试11场景

使用 4090 (48G) * 8卡:

  - 4块GPU作为rollout server
    - 参数: DP=2, TP=2
  - 4块GPU用于训练
    - \--vllm_gpu_memory_utilization 0.80

    - \--deepspeed zero2

    - \--num_generations 8
    - \--per_device_train_batch_size 1

    - \--per_device_eval_batch_size 2

    - \--gradient_accumulation_steps 8

  - 在之前的实验中, 与2块GPU作为rollout server, 6块GPU用于训练, 时间差不多
  - 训练时长:   

    - "epoch": 20.0, "global_step/max_steps": "1120/1120", "percentage": "100.00%", "elapsed_time": "21h 35m 49s", "remaining_time": "0s"

使用A100 * 8卡, rollout模式:

  - 2块GPU作为rollout server
    - 参数: DP=2, TP=1
  - 6块GPU用于训练
    - \--vllm_gpu_memory_utilization 0.90
    - 没有使用deepspeed
    - \--num_generations 6 
    - \--per_device_train_batch_size 2

    - \--per_device_eval_batch_size 1

    - \--gradient_accumulation_steps 4

  - 训练时长: 
    - "global_step/max_steps": "10/560", "percentage": "1.79%", "elapsed_time": "16m 46s", "remaining_time": "15h 22m 48s"
    - "epoch": 16.60714286, "global_step/max_steps": "465/560", "percentage": "83.04%", "elapsed_time": "8h 27m 41s", "remaining_time": "1h 43m 43s"

使用A100 * 8卡, colocate模式: 

  - 参数

    - \--vllm_gpu_memory_utilization 0.90 

    - \--offload_model true 

    - \--sleep_level 1

    - \--num_generations 8 

    - \--per_device_train_batch_size 2 

    - \--per_device_eval_batch_size 1 

    - \--gradient_accumulation_steps 4 

  - 训练时长
    - "global_step/max_steps": "10/560", "percentage": "1.79%", "elapsed_time": "14m 55s", "remaining_time": "13h 40m 25s"
