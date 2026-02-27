---
title: 20231105 - 判断Oracle某个SQL hang的原因
confluence_page_id: 2589352
created_at: 2023-11-05T02:22:57+00:00
updated_at: 2023-11-05T02:22:57+00:00
---

```
select * from dba_blockers;

select serial# from v$session s where s.sid = 221

alter system kill session '221,73' immediate;
```
