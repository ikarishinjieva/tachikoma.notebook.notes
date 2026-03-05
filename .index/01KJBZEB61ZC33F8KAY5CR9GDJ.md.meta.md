---
note: 01KJBZEB61ZC33F8KAY5CR9GDJ.md
title: 20240814 - 对AutoPrompt进行日志和流程分析
indexed_at: 2026-03-05T10:32:48.868583+00:00
---

## 标签
AutoPrompt, 日志分析, Prompt 优化, LLM 标注, 对抗样本，分类任务

## 摘要
分析 AutoPrompt 运行日志，梳理其工作流程包括初始化训练数据生成、LLM 标注配置、对抗样本生成等步骤。记录了各阶段使用的提示词模板、API 请求参数和标注配置细节。

## 关键概念
- 初始化训练数据: 数据集为空时由 LLM 生成平衡的挑战性样本
- LLM 批量标注: 使用多指令配置和存在聚合模式进行自动标注
- 对抗样本: 生成能导致分类器出错的多样化测试样本
- 提示词模板: AutoPrompt 各阶段使用的标准化 prompt 文件

## 关联笔记
- 01KJBZE545BYHRCYTWBJMQ6PJS.md: 本笔记分析的日志来源，记录 AutoPrompt 试用过程
- 01KJBZDYTJF2N82HT70Z06VNQN.md: AutoPrompt 架构分析，与本笔记的流程分析互为补充
