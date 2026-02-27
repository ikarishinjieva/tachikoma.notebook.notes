---
title: 20240108 - langchain 使用 LCEL
confluence_page_id: 2589958
created_at: 2024-01-08T06:47:08+00:00
updated_at: 2024-01-08T06:47:31+00:00
---

# LCEL如何开启verbose

<https://github.com/langchain-ai/langchain/issues/12659>

使用: ConsoleCallbackHandler

```
from langchain.callbacks.tracers import ConsoleCallbackHandler
chain_with_history.invoke(
    {"input": "项目例会是下周一"}, 
    {'configurable': {'session_id': '1'}, 'callbacks': [ConsoleCallbackHandler()]},
)
```
