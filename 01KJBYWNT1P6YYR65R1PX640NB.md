---
title: 20221221 - clickhouse 获取堆栈
confluence_page_id: 2130359
created_at: 2022-12-21T06:44:00+00:00
updated_at: 2022-12-21T06:44:00+00:00
---

```
select thread_number, query_id, arrayMap(x -> demangle(addressToSymbol(x)), trace) from stack_trace;
```
