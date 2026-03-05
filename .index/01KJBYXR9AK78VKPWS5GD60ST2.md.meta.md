---
note: 01KJBYXR9AK78VKPWS5GD60ST2.md
title: 20230403 - 测试 google Flan T5 XL模型
indexed_at: 2026-03-05T08:31:30.808458+00:00
---

## 摘要
记录从 OpenAI API 切换到 Google Flan T5 XL 模型的迁移过程，在物理机上通过 Docker 容器部署测试环境。包含完整的依赖安装、Jupyter Notebook 配置及 Hugging Face 缓存迁移步骤。

## 关键概念
- Flan T5 XL: Google 开源的指令微调语言模型，作为 OpenAI API 的替代方案
- Hugging Face Transformers: 提供预训练模型加载和推理的 Python 库
- Jupyter Notebook: 交互式开发环境，用于模型测试和实验

## 关联笔记
- 01KJBYXQY4P60F03GGHAEDA2SW.md: 同样使用 Docker 配置 llama-index 和 Jupyter Notebook 环境，日期相近
- 01KJBYZ4TEGRYF95QXFSF9JFZ4.md: 使用 llama-index + OpenAI API，是本笔记迁移前的方案
- 01KJC08NBRWP9P7TG4MVZS84H8.md: Docker 和 nvidia-docker 配置相关记录
