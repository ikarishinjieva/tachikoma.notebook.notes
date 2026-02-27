---
title: 20231031 - mac osx 监听文件变更并同步到服务器
confluence_page_id: 2589347
created_at: 2023-10-31T07:15:55+00:00
updated_at: 2023-10-31T07:15:55+00:00
---

```
fswatch log_from_ui_code.py | xargs -n1 -I{} scp {} root@10.186.62.73:/tmp/log_from_ui_code.py
```
