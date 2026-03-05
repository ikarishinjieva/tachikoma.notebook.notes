---
note: 01KJBZJ2B4WEX7J5JS6W6BXGYH.md
title: 20241028 - 阅读论文: Qwen2-VL: Enhancing Vision-Language Model’s Perception of the World at Any Resolution
indexed_at: 2026-03-05T10:53:00.969826+00:00
---

## 摘要
Qwen2-VL 是阿里推出的视觉语言模型系列，核心创新是动态分辨率处理和 M-RoPE 位置编码，可处理任意分辨率的图像和视频。采用三阶段训练（ViT 训练→多模态训练→指令微调），在通用视觉问答、OCR、视频理解等任务上达到 SOTA 水平。

## 关键概念
- 动态分辨率: 将任意分辨率图像动态转换为数量可变的视觉 token，避免固定缩放导致的信息丢失
- M-RoPE: 多模态旋转位置嵌入，将位置信息分解为时间、高度、宽度三维度进行编码
- 2D-RoPE: 二维旋转位置嵌入，用相对位置关系替代绝对位置编码，支持图像缩放不变性
- ViT: 视觉 Transformer，移除绝对位置嵌入后引入 2D-RoPE 以适应动态分辨率
- 视觉 token 压缩: 通过 MLP 层将相邻 2x2 个 token 压缩为一个，减少进入 LLM 的 token 数量

## 关联笔记
- 01KJBZY9ADDRRK6F59FBPBYDT1.md: Qwen-Image 技术报告，继承 Qwen2.5-VL 架构用于图像生成任务
- 01KJBZRNQFY8AKXNBQH5QW7RKY.md: 使用 ms-swift 对 Qwen2-VL 进行微调的系列实验记录
- 01KJBZJ4ZJYX6CE5DC0AKEKGHH.md: Qwen2-VL-7B-Instruct 在 GPU 服务器上的分布式训练配置
