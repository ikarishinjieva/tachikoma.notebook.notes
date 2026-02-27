---
title: 20230830 - 随机SQL生成
confluence_page_id: 2589093
created_at: 2023-08-30T15:16:23+00:00
updated_at: 2023-08-31T02:50:32+00:00
---

# 相关项目

<https://github.com/MariaDB/randgen>

# 命令

```
perl runall-new.pl --basedir=/home/user01/mariadb-10.11.5-linux-systemd-x86_64 \
--grammar=conf/mariadb/optimizer.yy \
--threads=4 \
--duration=60 \
--engine=InnoDB \
--vardir=/dev/shm/vardir \
--mysqld='--user=root' \
--queries=100 \
--sqltrace | tee 1.log
``` 

# 原理

通过.yy文件 (语法: <https://github.com/RQG/RQG-Documentation/wiki/RandomQueryGeneratorGrammar>), 随机生成SQL

# 调整规则

稍微调整 conf/mariadb/optimizer.yy 文件, 生成部分恒等条件:

```
degenerate_where_item:
    tinyint[invariant] = tinyint[invariant] |
    tinyint[invariant] > tinyint[invariant] |
    char[invariant] = char[invariant] |
	char[invariant] NOT LIKE char[invariant] |
    string_16[invariant] = string_16[invariant] | 
    string_16[invariant] NOT LIKE string_16[invariant];

tinyint:
    _tinyint;

char:
    _char;

string_16:
    _string(16);
 
 
where_item:
	real_where_item |
	real_where_item |
	real_where_item |
	real_where_item |
	real_where_item |
	degenerate_where_item /*DEGENERATE*/;
``` 

带有DEGENERATE标签, 在sql audit训练是, 可以根据DEGENERATE标签判断结果, 而不需要使用正则表达式的结果
