---
title: 20230507 - 架设私有chatgpt
confluence_page_id: 2130875
created_at: 2023-05-07T06:20:27+00:00
updated_at: 2023-05-07T06:22:12+00:00
---

```
docker run -e OPENAI_API_KEY=2de3b03c4ebd4f1f8681ce7d86ef475d -e OPENAI_API_TYPE=azure -e OPENAI_API_HOST=https://tachikoma.openai.azure.com/ -e OPENAI_API_VERSION=2023-03-15-preview -e AZURE_DEPLOYMENT_ID=gpt-35-turbo-0301 -p 3000:3000 -d --name chatgpt ghcr.nju.edu.cn/mckaywrigley/chatbot-ui:main
```
