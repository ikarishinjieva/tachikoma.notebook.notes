---
note: 01KJBZJ3GVHGXNXSFNN2B05Q2V.md
title: 20241029 - 配置新的gpu服务器
indexed_at: 2026-03-05T10:53:18.710235+00:00
---

## 摘要
记录在远程 GPU 服务器上配置深度学习环境的完整流程，包括 Anaconda 安装、conda 环境创建及 Llama Factory 部署。详细说明通过 SSH 反向隧道映射本地代理、Clash 代理服务的配置方法与访问凭证。

## 关键概念
- SSH 反向隧道：通过 `-R` 参数将本地代理端口映射到远程服务器，实现远程访问本地代理服务
- Llama Factory：大语言模型微调框架，提供高效的 SFT、RLHF 等训练功能
- Clash for Linux：命令行代理工具，提供系统级代理和 Dashboard 管理界面
- Conda 环境：Python 虚拟环境管理工具，用于隔离项目依赖

## 关联笔记
- 01KJC08NBRWP9P7TG4MVZS84H8.md: 同样涉及服务器代理配置和远程开发环境搭建
- 01KJC0BHKEH1VHK2YCAFH1G8CJ.md: 记录另一台服务器的搭建过程，包含类似的环境变量和代理配置
- 01KJC0CY3E5Z18ZPRZ0X69SH1F.md: 同样使用 conda 创建深度学习环境，涉及 GPU 训练配置
