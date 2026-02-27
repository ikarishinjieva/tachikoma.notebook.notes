---
title: 20241029 - 配置新的gpu服务器
confluence_page_id: 3342844
created_at: 2024-10-28T17:18:37+00:00
updated_at: 2024-10-29T06:28:44+00:00
---

安装 Anacoda:

```
wget https://mirrors.ustc.edu.cn/anaconda/archive/Anaconda3-2024.10-1-Linux-x86_64.sh
 
bash Anaconda3-2024.10-1-Linux-x86_64.sh
``` 

创建并激活conda环境: 

```
conda create -n huangyan-llama-factory python=3.12
conda activate /root/miniconda3/envs/huangyan-llama-factory/
``` 

用本地代理映射到服务器: 

```
ssh -p 22199 -R 7890:localhost:7890 user01@gpu888.cn
``` 

配置Llama-factory:

数据根目录: /opt/huangyan

ALL_PROXY=<http://127.0.0.1:7890> git clone --depth 1 <https://github.com/hiyouga/LLaMA-Factory.git>

cd LlaMA-Factory

pip install -e ".[torch,metrics]"

配置clash: 

```
ALL_PROXY=http://127.0.0.1:7890 git clone https://github.com/wnlen/clash-for-linux.git
 
编辑.env
配置clash URL为 https://neofeed.org/files/BpUVoFckc4/clash
 
bash start.sh
 
---
 
Clash Dashboard 访问地址: http://<ip>:9090/ui
Secret: 386de2d9d8d827facd7388b5a5d504d67b1510ea33f4157d44e173f0b45a036d

请执行以下命令加载环境变量: source /etc/profile.d/clash.sh

请执行以下命令开启系统代理: proxy_on

若要临时关闭系统代理，请执行: proxy_off
``` 

代理端口: 127.0.0.1:7890
