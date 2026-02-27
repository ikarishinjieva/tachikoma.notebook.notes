---
title: 20231119 - Oracle技巧
confluence_page_id: 2589510
created_at: 2023-11-19T04:09:34+00:00
updated_at: 2023-11-23T15:57:08+00:00
---

# 清理oracle的archive log

运行rman: 

```
RMAN> connect target;

RMAN> DELETE ARCHIVELOG ALL COMPLETED BEFORE 'sysdate-1';
``` 

重启Oracle:

sqlplus / as sysdba

```
SHUTDOWN IMMEDIATE
STARTUP
``` 

# 查看用户有多少张表

```
SELECT table_name FROM all_tables WHERE owner = UPPER('OBTEST');
``` 

# 查看用户有多少对象

```
select * from all_objects where OWNER = 'OBTEST';
``` 

# 查看表的DDL

```
SET LONG 10000
SET LONGCHUNKSIZE 1000
SET LINESIZE 1000
SET PAGESIZE 0
SET TRIMOUT ON
SET TRIMSPOOL ON
SELECT dbms_metadata.get_ddl('TABLE', 'ITEM', 'OBTEST') AS definition FROM dual;
```
