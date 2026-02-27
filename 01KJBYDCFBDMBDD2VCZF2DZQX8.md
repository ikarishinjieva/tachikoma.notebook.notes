---
title: 20210606 - 从binlog中分离SQL, 进行统计
confluence_page_id: 1146948
created_at: 2021-06-06T09:07:33+00:00
updated_at: 2021-06-07T02:38:54+00:00
---

```
/data/mysql/base/5.7.25/bin/mysqlbinlog -vvvv  mysql-bin.008449 > mysql-bin.008449.parsed
 
//分离出SQL
awk '/Rows_query/,/^# at /' mysql-bin.008449.parsed | less
 
//合并SQL到同一行
awk '/Rows_query/,/^# at /{if ($0 ~ /Rows_query/ || $0 ~ /^# at/) {printf("\n")} else {sub(/^#/, "", $0); printf("%s", $0)}}' mysql-bin.008449.parsed > mysql-bin.008449.parsed2
 
//通过pt解析SQL文件, 得到SQL分布
pt-query-digest --type rawlog mysql-bin.008449.parsed2 > mysql-bin.008449.analyse
```
