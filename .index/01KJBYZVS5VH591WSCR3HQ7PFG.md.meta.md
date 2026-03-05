---
note: 01KJBYZVS5VH591WSCR3HQ7PFG.md
title: 20230507 - 架设私有chatgpt
indexed_at: 2026-03-05T08:42:24.478599+00:00
---

## 摘要
记录使用 Docker 部署私有 chatbot-ui 的命令行配置，通过 Azure OpenAI API 接入 GPT-3.5-turbo 模型。指定了 API 密钥、API 类型、API 版本、部署 ID 等关键参数，端口映射为 3000。

## 关键概念
- chatbot-ui: 开源的 ChatGPT Web UI 界面项目，支持自定义 API 端点
- Azure OpenAI: 微软 Azure 云提供的 OpenAI API 服务，需配置部署 ID 和版本号
- Docker 容器部署: 通过环境变量传递配置，运行无状态服务

## 关联笔记
- 01KJBYZ4TEGRYF95QXFSF9JFZ4.md: 使用相同的 Azure OpenAI API 配置（API KEY、端点、版本）开发私有运维知识库 demo
