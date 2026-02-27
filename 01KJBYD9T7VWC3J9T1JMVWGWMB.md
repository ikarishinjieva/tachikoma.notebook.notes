---
title: 20210330 - performance_schema的日常使用
confluence_page_id: 753739
created_at: 2021-03-30T06:34:25+00:00
updated_at: 2021-03-30T06:34:25+00:00
---

```
call sys.ps_setup_enable_instrument('');
call sys.ps_setup_enable_consumer('');
call sys.ps_setup_disable_background_threads();
```
