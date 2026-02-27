---
title: 20240228 - 对embedding进行微调 - 增加训练数据
confluence_page_id: 2590248
created_at: 2024-02-28T16:12:52+00:00
updated_at: 2024-03-04T04:42:36+00:00
---

# 原来的数据

[https://actiontech.feishu.cn/base/bascnXrnZzK3pxDPFughpyuo4AY?table=tblGgGyD15pwy5pb&view=vew2CfPxNW&chunked=false](<https://actiontech.feishu.cn/base/bascnXrnZzK3pxDPFughpyuo4AY?table=tblGgGyD15pwy5pb&view=vew2CfPxNW&chunked=false>)

列表: 

```
mysql修改密码后用navicat 登陆报错1862

数据库show engine innodb status中存在大量row locks记录，数据库是否存在异常？

Mysql主从同步时Slave_SQL_Running状态为Yes , 但是Slave_IO_Running状态为Connecting以及NO

主从切换后原主库比新主库多数据

多个从库出现延迟

数据库主从延迟严重

主从复制报错：HA_ERR_KEY_NOT_FOUND

MySQL出现异常连接线程的情况，如何排查

MySQL数据导入到TiDB时报错：Error 8025: entry too large, the max entry size is 6291456。

在MySQL中，发现有INSERT语句执行的比较慢，如何进行排查

MySQL升级，出现连接异常故障，报错：Host '10.136,41.17' is blocked because of many connection errors

应用连接mysql实例一段时间之后总是经常中断

MySQL OOM 故障

MySQL进程崩溃

mysql从库在做Optimize table操作，延时一直在加大，可能的原因是什么

MySQL临时表占用过高，如何排查

MySQL创建全文索引，报错：“Temporary file write failure”

MySQL数据库插入数据慢

数据库多次异常关闭，error log报错：scada5 kernel:XFS (sde1): xfs_log_force: error 5 returned

使用fsc用户访问视图fsc_vfc.v_rsc_card_type报错：SQL Error (1045): Access denied for user 'fsc'@'%' (using password: YES)

关闭MySQL服务时进程卡住导致业务中断

MySQL的table_handles内存暴增，怎么排查

MySQL数据库出现大量的请求不能响应，报错：No operations allowed after statement closed

表添加字段时报主键冲突：ERROR:1062 Duplicate entry '12178350' for key 'PRIMARY'

SQL执行报错：Lock wait timeout exceeded

MySQL服务器负载突然飙升

MySQL启动失败排查

mysql的MyISAM引擎表，show processlist 显示大量线程“Waiting for table level lock”

MySQL内存使用过高

mysql insert写入数据报错：Incorrect string value: 'ð' for column 'name' at row 1

MySQL执行stop slave超时

使用MyISAM引擎的表，显示大量“Waiting for table level lock”，怎么解决

从库的IO线程报错，last_IO_Errot: 1593

MySQL 半同步复制频繁断开：Timeout waiting for reply of binlog

数据库hang，错误日中中包含大量等待信号日志：[Warning] InnoDB: A long semaphore wait:--Thread 140477271320320 has waited at ibuf0ibuf.cc line 3439 for 241.00 seconds the semaphore:S-lock on RW-latch

mysql8.0客户端执行load data local infile命令失败

MySQL当连接数超过一定值(3000-4000)左右，连接MySQL会变得十分卡顿。

从库Seconds_Behind_Master的值出现了规律性的变化

MySQL 从库所在主机故障重启后，sql_thread 线程报错

数据库报错：Can't open file: 'xxx_forums.MYI'. (errno: 145)

主从复制异常中断，Semi-sync slave net_flush() reply failed

MySQL插入数据过程中主键冲突怎么办

MySQL通过xtrabackup备份失败报错： Unable to obtain lock. Please try again later

MySQL从库频繁出现复制断开现象，SQL线程报错：Last_SQL_Errno: 1205，怎么排查

数据库crash，error log里有大量Semaphore wait has lasted > 600 seconds

MySQL数据误删除，如何恢复数据 ***

SQL在升级数据库版本以后效率骤降

主从读写分离环境下，从库出现了 MySQL crash

双主模式中从库报错主键冲突，主从同步出现问题

在数据库重启后触发器无法使用

客户端连接 MySQL失败，报错：ERROR 1130 (HY000): Host '192.168.17.149' is not allowed to connect to this MySQL server

MySQL8.0.30报错ERROR] [MY-013183] [InnoDB] Assertion failure: btr0cur.cc:4024:page

4. 错误与InnoDB存储引擎相关
MySQL复制中断

SQL语句出现性能定时抖动的问题，怎么处理

mysql中的语句无法kill，一直处于killed状态，但进程正常

在从库上执行了flush privilege语句后，主从GTID不一致，为什么

使用MySQL client命令行登录报错：mysql Segmentation fault (core dumped)

数据库运行时，应用写入历史表比较慢（Debug: APP_MODE[1] STORE OBJECTS[108771SED TIME: 91048 (ms) 2022-03-14 02:15:04 2022-03-14），之后提示故障报错

从库IO线程异常，show slave status显示状态为NO

MySQL级联复制日志频繁报错：found a zombie dump thread with the same UUID

MySQL无法启动，报错：“Can't start server: can't create PID file: No such file or directory”

数据库报错：ERROR 1292 (22007): Truncated incorredct DECIMAL value

MySQL主从切换原因分析

MySQL活跃线程过多，大量处于Waiting for table metadata lock状态，无法响应新链接，应该怎么处理

MySQL服务器在运行中突然无法正常响应客户端的请求

数据库重启后触发器无法使用，应该怎么排查？

业务反馈MySQL无法链接，有时候能连，有时候不能，大多数时候都是报错超时？

MySQL 无监听端口

应用连接 MySQL 报错：Public Key Retrieval is not allowed

通过xtrabackup物理备份MySQL数据,出现备份失败

MySQL复制报警，从库复制延时越来越大, show slave status观察gtid一直停留在固定的地方不变

MySQL实例登录报错Too many connections

数据库报错：Lock wait timeout exceeded; try restarting transaction, Error_code: 1205

MySQL用户登录出现无法通过socket登录的问题，怎么解决

MySQL的select查询执行慢

MySQL语句出现阻塞现象

MySQL执行的sql语句一直处于Opening table状态

mysqlrouter 出现5秒自动重启的问题

MySQL链接异常中断，报错：ERROR 2013(HY000): Lost Connection to MySQL server during query

使用socket，远程连接MySQL数据库很慢

MySQL安装时报错Could not open required defaults file

使用case when存储过程报错：You have an error in you SQL syntax

MySQL锁等待超时

MySQL频繁OOM，怎么排查

MySQL登录报错ERROR 1045(28000), Access denied

MySQL主从切换丢失数据

mysql cpu负载高该如何处理？

数据库报错 ERROR 2002  (HY000):   Can't  connect   to  local  MySQL  server  through   socket

MySQL 错误日志报错: Too many open files

MySQL无法启动报错：“File './mysql-bin.index' not found (Errcode:13 - Permission denied)”

MySQL更新数据出现超时

MySQL的cpu利用率突然达到100%如何处理

MySQL 启动异常

undo表空间占用过大且无法被MySQL自动truncate截断，怎么解决

从库复制断开，报错：Last_SQL_Errno=1118

SQL线程故障，

MySQL主从GTID不一致分析

从库出现主从延迟，并且Second Behind Master的值不断增加（问题描述有歧义）

Binlog 太大如何进行解析，找到大事务

``` 

# 增加stackoverflow数据

从stackoverflow中找问题: <https://stackoverflow.com/questions/tagged/mysql+-php+-python+-sql+-node.js>, 按照score排序

```
Would you recommend using a datetime or a timestamp field, and why (using MySQL)?

------

Is there an easy way to run a MySQL query from the Linux command line and output the results in CSV format?
Here's what I'm doing now:
 
mysql -u uid -ppwd -D dbname << EOQ | sed -e 's/        /,/g' | tee list.csv
select id, concat("\"",name,"\"") as name
from students
EOQ
 
It gets messy when there are a lot of columns that need to be surrounded by quotes, or if there are quotes in the results that need to be escaped.
 
------
  
Between utf8_general_ci and utf8_unicode_ci, are there any differences in terms of performance?
 
------
  
I can run this query to get the sizes of all tables in a MySQL database:
 
show table status from myDatabaseName;
 
I would like some help in understanding the results. I am looking for tables with the largest sizes.
Which column should I look at?
 
------
  
How do I quickly rename a MySQL database (change its schema name)?
Usually I just dump a database and re-import it with a new name. This is not an option for very big databases. Apparently RENAME {DATABASE | SCHEMA} db_name TO new_db_name; does bad things, exists only in a handful of versions, and is a bad idea overall.
This needs to work with InnoDB, which stores things very differently than MyISAM.
 
 
------
  
How can I find all the tables in MySQL with specific column names in them?
I have 2-3 different column names that I want to look up in the entire database and list out all tables which have those columns. Is there any easy script?
 
------
  
How do you set a default value for a MySQL Datetime column?
In SQL Server it's getdate(). What is the equivalant for MySQL? I'm using MySQL 5.x if that is a factor.
 
------
While executing an INSERT statement with many rows, I want to skip duplicate entries that would otherwise cause failure. After some research, my options appear to be the use of either:
 
ON DUPLICATE KEY UPDATE which implies an unnecessary update at some cost, or
INSERT IGNORE implies an invitation for other kinds of failure to slip in unannounced.
Am I right in these assumptions? What's the best way to simply skip the rows that might cause duplicates and just continue on to the other rows?
 
------
 Per the MySQL docs, there are four TEXT types:
 
TINYTEXT
TEXT
MEDIUMTEXT
LONGTEXT
What is the maximum length that I can store in a column of each data type assuming the character encoding is UTF-8?
 
------

I've got a messages table in MySQL which records messages between users. Apart from the typical ids and message types (all integer types) I need to save the actual message text as either VARCHAR or TEXT. I'm setting a front-end limit of 3000 characters which means the messages would never be inserted into the db as longer than this.
Is there a rationale for going with either VARCHAR(3000) or TEXT? There's something about just writing VARCHAR(3000) that feels somewhat counter-intuitive. I've been through other similar posts on Stack Overflow but would be good to get views specific to this type of common message storing.
 
------

I accidentally enabled ONLY_FULL_GROUP_BY mode like this:
SET sql_mode = 'ONLY_FULL_GROUP_BY';
How do I disable it?
 
------

 This should be dead simple, but I cannot get it to work for the life of me.
I'm just trying to connect remotely to my MySQL server.
Connecting as:
mysql -u root -h localhost -p
 
works fine, but trying:
mysql -u root -h 'any ip address here' -p
 
fails with the error:
ERROR 1130 (00000): Host 'xxx.xx.xxx.xxx' is not allowed to connect to this MySQL server
 
 
 
In the mysql.user table, there is exactly the same entry for user 'root' with host 'localhost' as another with host '%'.
I'm at my wits' end and have no idea how to proceed. Any ideas are welcome.
 
 
------

 How do I copy or clone or duplicate the data, structure, and indices of a MySQL table to a new one?
This is what I've found so far.
This will copy the data and the structure, but not the indices:
create table {new_table} select * from {old_table};
This will copy the structure and indices, but not the data:
create table {new_table} like {old_table};
 
------

How to get size of a mysql database?
Suppose the target database is called "v3". ------
 In MySQL, how do I get a list of all foreign key constraints pointing to a particular table? a particular column? This is the same thing as this Oracle question, but for MySQL.
 
 
------

I've created database, for example 'mydb'.
CREATE DATABASE mydb CHARACTER SET utf8 COLLATE utf8_bin; CREATE USER 'myuser'@'%' IDENTIFIED BY PASSWORD '*HASH'; GRANT ALL ON mydb.* TO 'myuser'@'%'; GRANT ALL ON mydb TO 'myuser'@'%'; GRANT CREATE ON mydb TO 'myuser'@'%'; FLUSH PRIVILEGES;
Now i can login to database from everywhere, but can't create tables.
How to grant all privileges on that database and (in the future) tables. I can't create tables in 'mydb' database. I always get:
CREATE TABLE t (c CHAR(20) CHARACTER SET utf8 COLLATE utf8_bin); ERROR 1142 (42000): CREATE command denied to user 'myuser'@'...' for table 't'
 
------

 I want to change the data type of multiple columns from float to int. What is the simplest way to do this?
There is no data to worry about, yet.
 
------

 I am connecting MySQL - 8.0 with MySQL Workbench and getting the below error:
Authentication plugin 'caching_sha2_password' cannot be loaded: dlopen(/usr/local/mysql/lib/plugin/caching_sha2_password.so, 2): image not found
 
I have tried with other client tool as well.
Any solution for this?
 
------

 When I executed the following command:
ALTER TABLE `mytable` ADD UNIQUE ( `column1` , `column2` );
I got this error message:
#1071 - Specified key was too long; max key length is 767 bytes
Information about column1 and column2:
column1 varchar(20) utf8_general_ci column2 varchar(500) utf8_general_ci
I think varchar(20) only requires 21 bytes while varchar(500) only requires 501 bytes. So the total bytes are 522, less than 767. So why did I get the error message?
#1071 - Specified key was too long; max key length is 767 bytes

------

What are the differences between PRIMARY, UNIQUE, INDEX and FULLTEXT when creating MySQL tables?

How would I use them?
``` 

![image2024-2-29 0:9:7.png](/assets/01KJBZ6V4HFTNCBMEZSA71YAKB/image2024-2-29%200%3A9%3A7.png)

对以上问题进行翻译, 并进行分析: [question_to_training_data.html](/assets/01KJBZ6V4HFTNCBMEZSA71YAKB/question_to_training_data.html)

# 额外发现: 存在与材料完全无关的问题, 需要流程上特别处理

  1. AI的回答中, 对于不确定的结论, 或者材料不足的结论, 应当承认 "不知道"
  2. 训练数据中, 对于relevance不高的case, 不应当出现在pos(正向)数据集中

# 重新训练

  1. 将上述20个问题的训练数据加入训练集
  2. 将relevance不高的case, 从pos(正向)数据集排除

微调: epoch = 10, loss = 0.78

效果: 

```
当前retriever: 
 
expect_rank = 0, actual_rank = 31, delta = 31
expect_rank = 1, actual_rank = 8, delta = 7
expect_rank = 2, actual_rank = 29, delta = 27
expect_rank = 3, actual_rank = 15, delta = 12
expect_rank = 4, actual_rank = 18, delta = 14
expect_rank = 5, actual_rank = 0, delta = 5
expect_rank = 6, actual_rank = 1, delta = 5
expect_rank = 7, actual_rank = 2, delta = 5
expect_rank = 8, actual_rank = 20, delta = 12
expect_rank = 9, actual_rank = 21, delta = 12
question[0] total delta = 130
expect_rank = 0, actual_rank = 23, delta = 23
expect_rank = 1, actual_rank = 11, delta = 10
expect_rank = 2, actual_rank = 2, delta = 0
expect_rank = 3, actual_rank = 3, delta = 0
expect_rank = 4, actual_rank = 7, delta = 3
expect_rank = 5, actual_rank = 14, delta = 9
expect_rank = 6, actual_rank = 17, delta = 11
expect_rank = 7, actual_rank = 19, delta = 12
expect_rank = 8, actual_rank = 22, delta = 14
expect_rank = 9, actual_rank = 1, delta = 8
question[1] total delta = 90
expect_rank = 0, actual_rank = 7, delta = 7
expect_rank = 1, actual_rank = 0, delta = 1
expect_rank = 2, actual_rank = 4, delta = 2
expect_rank = 3, actual_rank = 1, delta = 2
expect_rank = 4, actual_rank = 6, delta = 2
expect_rank = 5, actual_rank = 18, delta = 13
expect_rank = 6, actual_rank = 22, delta = 16
expect_rank = 7, actual_rank = 24, delta = 17
expect_rank = 8, actual_rank = 26, delta = 18
expect_rank = 9, actual_rank = 29, delta = 20
question[2] total delta = 98
expect_rank = 0, actual_rank = 2, delta = 2
expect_rank = 1, actual_rank = 13, delta = 12
expect_rank = 2, actual_rank = 6, delta = 4
expect_rank = 3, actual_rank = 8, delta = 5
expect_rank = 4, actual_rank = 17, delta = 13
expect_rank = 5, actual_rank = 18, delta = 13
expect_rank = 6, actual_rank = 19, delta = 13
expect_rank = 7, actual_rank = 21, delta = 14
expect_rank = 8, actual_rank = 3, delta = 5
expect_rank = 9, actual_rank = 4, delta = 5
question[3] total delta = 86
expect_rank = 0, actual_rank = 9, delta = 9
expect_rank = 1, actual_rank = 27, delta = 26
expect_rank = 2, actual_rank = 0, delta = 2
expect_rank = 3, actual_rank = 1, delta = 2
expect_rank = 4, actual_rank = 21, delta = 17
expect_rank = 5, actual_rank = 23, delta = 18
expect_rank = 6, actual_rank = 31, delta = 25
expect_rank = 7, actual_rank = 15, delta = 8
actual_rank = not found, delta = 48.0
actual_rank = not found, delta = 48.0
question[4] total delta = 203.0
expect_rank = 0, actual_rank = 3, delta = 3
expect_rank = 1, actual_rank = 13, delta = 12
expect_rank = 2, actual_rank = 11, delta = 9
expect_rank = 3, actual_rank = 6, delta = 3
expect_rank = 4, actual_rank = 0, delta = 4
expect_rank = 5, actual_rank = 4, delta = 1
expect_rank = 6, actual_rank = 17, delta = 11
expect_rank = 7, actual_rank = 7, delta = 0
expect_rank = 8, actual_rank = 8, delta = 0
expect_rank = 9, actual_rank = 33, delta = 24
question[5] total delta = 67
expect_rank = 0, actual_rank = 21, delta = 21
expect_rank = 1, actual_rank = 17, delta = 16
expect_rank = 2, actual_rank = 14, delta = 12
expect_rank = 3, actual_rank = 16, delta = 13
expect_rank = 4, actual_rank = 15, delta = 11
expect_rank = 5, actual_rank = 26, delta = 21
expect_rank = 6, actual_rank = 27, delta = 21
expect_rank = 7, actual_rank = 20, delta = 13
expect_rank = 8, actual_rank = 23, delta = 15
expect_rank = 9, actual_rank = 24, delta = 15
question[6] total delta = 158
expect_rank = 0, actual_rank = 1, delta = 1
expect_rank = 1, actual_rank = 11, delta = 10
expect_rank = 2, actual_rank = 15, delta = 13
expect_rank = 3, actual_rank = 3, delta = 0
expect_rank = 4, actual_rank = 9, delta = 5
expect_rank = 5, actual_rank = 16, delta = 11
expect_rank = 6, actual_rank = 26, delta = 20
expect_rank = 7, actual_rank = 28, delta = 21
expect_rank = 8, actual_rank = 31, delta = 23
expect_rank = 9, actual_rank = 0, delta = 9
question[7] total delta = 113
expect_rank = 0, actual_rank = 4, delta = 4
expect_rank = 1, actual_rank = 19, delta = 18
expect_rank = 2, actual_rank = 16, delta = 14
expect_rank = 3, actual_rank = 18, delta = 15
expect_rank = 4, actual_rank = 13, delta = 9
expect_rank = 5, actual_rank = 28, delta = 23
actual_rank = not found, delta = 48.0
expect_rank = 7, actual_rank = 0, delta = 7
expect_rank = 8, actual_rank = 21, delta = 13
expect_rank = 9, actual_rank = 22, delta = 13
question[8] total delta = 164.0
expect_rank = 0, actual_rank = 4, delta = 4
expect_rank = 1, actual_rank = 29, delta = 28
expect_rank = 2, actual_rank = 20, delta = 18
expect_rank = 3, actual_rank = 1, delta = 2
expect_rank = 4, actual_rank = 31, delta = 27
expect_rank = 5, actual_rank = 2, delta = 3
expect_rank = 6, actual_rank = 16, delta = 10
expect_rank = 7, actual_rank = 7, delta = 0
expect_rank = 8, actual_rank = 9, delta = 1
expect_rank = 9, actual_rank = 21, delta = 12
question[9] total delta = 105
all delta = 1214.0

微调后的bge/m3: 
 
expect_rank = 0, actual_rank = 0, delta = 0
expect_rank = 1, actual_rank = 7, delta = 6
expect_rank = 2, actual_rank = 3, delta = 1
expect_rank = 3, actual_rank = 20, delta = 17
expect_rank = 4, actual_rank = 5, delta = 1
expect_rank = 5, actual_rank = 16, delta = 11
expect_rank = 6, actual_rank = 15, delta = 9
expect_rank = 7, actual_rank = 26, delta = 19
expect_rank = 8, actual_rank = 1, delta = 7
expect_rank = 9, actual_rank = 11, delta = 2
question[0] total delta = 73
expect_rank = 0, actual_rank = 5, delta = 5
expect_rank = 1, actual_rank = 0, delta = 1
expect_rank = 2, actual_rank = 4, delta = 2
expect_rank = 3, actual_rank = 8, delta = 5
expect_rank = 4, actual_rank = 6, delta = 2
expect_rank = 5, actual_rank = 1, delta = 4
expect_rank = 6, actual_rank = 3, delta = 3
expect_rank = 7, actual_rank = 12, delta = 5
expect_rank = 8, actual_rank = 2, delta = 6
expect_rank = 9, actual_rank = 11, delta = 2
question[1] total delta = 35
expect_rank = 0, actual_rank = 12, delta = 12
expect_rank = 1, actual_rank = 20, delta = 19
expect_rank = 2, actual_rank = 1, delta = 1
expect_rank = 3, actual_rank = 5, delta = 2
expect_rank = 4, actual_rank = 2, delta = 2
expect_rank = 5, actual_rank = 0, delta = 5
expect_rank = 6, actual_rank = 7, delta = 1
expect_rank = 7, actual_rank = 6, delta = 1
expect_rank = 8, actual_rank = 23, delta = 15
expect_rank = 9, actual_rank = 15, delta = 6
question[2] total delta = 64
expect_rank = 0, actual_rank = 2, delta = 2
expect_rank = 1, actual_rank = 0, delta = 1
expect_rank = 2, actual_rank = 5, delta = 3
expect_rank = 3, actual_rank = 1, delta = 2
expect_rank = 4, actual_rank = 4, delta = 0
expect_rank = 5, actual_rank = 7, delta = 2
expect_rank = 6, actual_rank = 6, delta = 0
expect_rank = 7, actual_rank = 3, delta = 4
expect_rank = 8, actual_rank = 10, delta = 2
expect_rank = 9, actual_rank = 9, delta = 0
question[3] total delta = 16
expect_rank = 0, actual_rank = 3, delta = 3
expect_rank = 1, actual_rank = 0, delta = 1
expect_rank = 2, actual_rank = 6, delta = 4
expect_rank = 3, actual_rank = 16, delta = 13
expect_rank = 4, actual_rank = 1, delta = 3
expect_rank = 5, actual_rank = 2, delta = 3
expect_rank = 6, actual_rank = 10, delta = 4
expect_rank = 7, actual_rank = 14, delta = 7
expect_rank = 8, actual_rank = 22, delta = 14
expect_rank = 9, actual_rank = 19, delta = 10
question[4] total delta = 62
expect_rank = 0, actual_rank = 1, delta = 1
expect_rank = 1, actual_rank = 0, delta = 1
expect_rank = 2, actual_rank = 4, delta = 2
expect_rank = 3, actual_rank = 3, delta = 0
expect_rank = 4, actual_rank = 5, delta = 1
expect_rank = 5, actual_rank = 2, delta = 3
expect_rank = 6, actual_rank = 14, delta = 8
expect_rank = 7, actual_rank = 20, delta = 13
expect_rank = 8, actual_rank = 8, delta = 0
expect_rank = 9, actual_rank = 6, delta = 3
question[5] total delta = 32
expect_rank = 0, actual_rank = 0, delta = 0
expect_rank = 1, actual_rank = 4, delta = 3
expect_rank = 2, actual_rank = 1, delta = 1
expect_rank = 3, actual_rank = 2, delta = 1
expect_rank = 4, actual_rank = 7, delta = 3
expect_rank = 5, actual_rank = 13, delta = 8
expect_rank = 6, actual_rank = 24, delta = 18
expect_rank = 7, actual_rank = 5, delta = 2
expect_rank = 8, actual_rank = 3, delta = 5
expect_rank = 9, actual_rank = 15, delta = 6
question[6] total delta = 47
expect_rank = 0, actual_rank = 0, delta = 0
expect_rank = 1, actual_rank = 1, delta = 0
expect_rank = 2, actual_rank = 6, delta = 4
expect_rank = 3, actual_rank = 5, delta = 2
expect_rank = 4, actual_rank = 4, delta = 0
expect_rank = 5, actual_rank = 2, delta = 3
expect_rank = 6, actual_rank = 10, delta = 4
expect_rank = 7, actual_rank = 9, delta = 2
expect_rank = 8, actual_rank = 20, delta = 12
expect_rank = 9, actual_rank = 33, delta = 24
question[7] total delta = 51
expect_rank = 0, actual_rank = 2, delta = 2
expect_rank = 1, actual_rank = 17, delta = 16
expect_rank = 2, actual_rank = 1, delta = 1
expect_rank = 3, actual_rank = 0, delta = 3
expect_rank = 4, actual_rank = 7, delta = 3
expect_rank = 5, actual_rank = 5, delta = 0
expect_rank = 6, actual_rank = 4, delta = 2
expect_rank = 7, actual_rank = 6, delta = 1
expect_rank = 8, actual_rank = 12, delta = 4
expect_rank = 9, actual_rank = 3, delta = 6
question[8] total delta = 38
expect_rank = 0, actual_rank = 4, delta = 4
expect_rank = 1, actual_rank = 0, delta = 1
expect_rank = 2, actual_rank = 3, delta = 1
expect_rank = 3, actual_rank = 10, delta = 7
expect_rank = 4, actual_rank = 1, delta = 3
expect_rank = 5, actual_rank = 9, delta = 4
expect_rank = 6, actual_rank = 6, delta = 0
expect_rank = 7, actual_rank = 5, delta = 2
expect_rank = 8, actual_rank = 2, delta = 6
expect_rank = 9, actual_rank = 15, delta = 6
question[9] total delta = 34
all delta = 452
``` 

质疑这组数据: 可能是因为代码错误, 导致的过拟合现象

# 再次增加训练数据

```
为发送私人消息创建表单，并想要将文本区域（textarea）的最大长度（maxlength）值设置为与MySQL数据库表中文本字段的最大长度相匹配。文本字段类型可以存储多少字符？

如果字符数很多，我能像使用varchar那样在数据库的文本类型字段中指定长度吗？

======

更改表以允许列为空的语法是什么？或者，以下操作有什么问题：

ALTER mytable MODIFY mycolumn varchar(255) null;

我按照手册的解释执行上述操作，它应该重新创建列，并允许为空。但服务器告诉我有语法错误。我就是看不出来。

======

根据以下创建的表：

CREATE TABLE tbl_Country
(
  CountryId INT NOT NULL AUTO_INCREMENT,
  IsDeleted bit,
  PRIMARY KEY (CountryId) 
)

我怎样才能删除IsDeleted这一列？

======

在MySQL中，utf8mb4和utf8字符集有什么区别？

我已经知道了ASCII、UTF-8、UTF-16和UTF-32编码；但我很好奇，想知道MySQL Server中定义的utf8mb4编码组与其他编码类型有什么区别。

使用utf8mb4而不是utf8有什么特别的好处/目的吗？

======

我想将现有生产数据库复制到我的本地开发数据库中。有没有办法做到这一点，而不锁定生产数据库？

我目前使用的是：

mysqldump -u root --password=xxx -h xxx my_db1 | mysql -u root --password=xxx -h localhost my_db1

但它在运行时会锁定每个表。

======

在mysql命令行中，如何查看存储过程或存储函数的列表，就像SHOW TABLES; 或SHOW DATABASES; 命令一样。

======

有没有一个MySQL命令可以定位my.cnf配置文件，类似于PHP的phpinfo()定位其php.ini的方式？

======

我有一个非常大的MySQL表，大约有150,000行数据。目前，当我尝试运行

SELECT * FROM table WHERE id = '1';
代码运行正常，因为ID字段是主索引。然而，对于项目中的最近开发，我不得不通过另一个字段来搜索数据库。例如：

SELECT * FROM table WHERE product_id = '1';
这个字段以前没有被索引；然而，我已经添加了一个索引，所以mysql现在索引了这个字段，但当我尝试运行上面的查询时，它运行得非常慢。一个EXPLAIN查询显示，尽管我已经添加了索引，但product_id字段没有索引，结果查询需要20到30分钟才能返回单行数据。

我的完整EXPLAIN结果如下：

| id | select_type | table | type | possible_keys | key | key_len | ref | rows | Extra |
+----+-------------+-------+------+--------------+------+---------+------+-------+------------------+
| 1 | SIMPLE | table | ALL | NULL | NULL | NULL | NULL |157211 | Using where |
+----+-------------+-------+------+--------------+------+---------+------+-------+------------------+
值得注意的是，我刚刚查看了一下，ID字段存储为INT，而PRODUCT_ID字段存储为VARCHAR。这可能是问题的根源吗？

======

我在MySQL上运行这个查询

SELECT ID FROM (
    SELECT ID, msisdn
    FROM (
        SELECT * FROM TT2
    )
);
它报了这个错误：

每个派生表都必须有自己的别名。

是什么原因造成这个错误？

======

我提了一个问题并得到了这个有帮助的回复。

   UPDATE TABLE_A a JOIN TABLE_B b
   ON a.join_col = b.join_col AND a.column_a = b.column_b
   SET a.column_c = a.column_c + 1
现在我想要做的是，如果涉及到三个表，就像这样。

    UPDATE tableC c JOIN tableB b JOIN tableA a

我的问题基本上是... 是否可能在UPDATE语句上做三个表的连接？正确的语法是什么？

我应该这样做吗？

 JOIN tableB, tableA
 JOIN tableB JOIN tableA

======

我需要向表中添加多个列，但要把这些列放在一个名为lastname的列之后。

我尝试过这样做：

ALTER TABLE `users` ADD COLUMN
(
    `count` smallint(6) NOT NULL,
    `log` varchar(12) NOT NULL,
    `status` int(10) unsigned NOT NULL
) 
AFTER `lastname`;
我收到这个错误：

您的SQL语法有错误；请检查与您的MySQL服务器版本对应的手册，了解使用near ') AFTER lastname' at line 7的正确语法。

我该如何在这样的查询中使用AFTER？

======

我的MySQL数据库包含几个使用不同存储引擎的表（具体来说是myisam和innodb）。我如何知道哪些表使用的是哪种引擎？

======

在MySQL中创建一个带有VARCHAR列的表时，我们必须为其设置长度。但对于TEXT类型，我们不需要提供长度。

VARCHAR和TEXT有什么区别？

======

在MySQL控制台中，什么命令可以显示任何给定表的架构？

======

我在本地机器上安装了MySQL社区版5.5，并且我想允许远程连接，以便我可以从外部源进行连接。

我该如何做？

======

我真的很感兴趣MySQL索引是如何工作的，更具体地说，它们是如何在不扫描整个表的情况下返回请求的数据的？

我知道这有点跑题，但如果有人能详细解释给我听，我会非常非常感激的。

======

我如何通过终端用mysql导入一个数据库？

我找不到确切的语法。

======

我该如何设置MySQL的时区？

在一台服务器上，当我运行：

mysql> select now();
+---------------------+
| now()               |
+---------------------+
| 2009-05-30 16:54:29 |
+---------------------+
1 row in set (0.00 sec)

在另一台服务器上：

mysql> select now();
+---------------------+
| now()               |
+---------------------+
| 2009-05-30 20:01:43 |
+---------------------+
1 row in set (0.00 sec)

======

如何使用命令提示符导出mysql数据库？

我有一个相当大的数据库，所以我想使用命令提示符导出它，但我不知道如何操作。

======

我想备份我所有的MySQL数据库。我有超过100个MySQL数据库。我想要同时导出它们所有，并且再次一次性将它们全部导入我的MySQL服务器。我该怎么做？

======
``` 

训练: 

epoch = 2, per_device_train_batch_size=8, loss = 0.86

实际效果: 

```
question[0] total delta = 191.0
question[1] total delta = 81.0
question[2] total delta = 208.0
question[3] total delta = 38
question[4] total delta = 149.0
question[5] total delta = 68
question[6] total delta = 172.0
question[7] total delta = 54
question[8] total delta = 52
question[9] total delta = 29
all delta = 1042.0
``` 

可以看到: 

  1. loss无法体现 all delta的水平, 需要构造校验集? (但在Trainer的参数中, 没有找到校验集的构造参数)
  2. epoch需要提高, epoch过少无法得到较好效果

提高epoch=14, per_device_train_batch_size=8, loss=0.707

效果:

```
question[0] total delta = 184.0
question[1] total delta = 267.0
question[2] total delta = 295.0
question[3] total delta = 148.0
question[4] total delta = 260.0
question[5] total delta = 159.0
question[6] total delta = 357.0
question[7] total delta = 292.0
question[8] total delta = 202.0
question[9] total delta = 316.0
all delta = 2480.0
``` 

epoch提高时, 效果在变差?? 进行ABA测试.

epoch=2, per_device_train_batch_size=1, loss=0.833, 效果持续变差, 超过6000

将新的训练数据全部去掉, 仅使用 15-100 作为训练数据, 进行微调, epoch = 2, per_device_train_batch_size=8, loss=0.99, 效果: 

(可能是因为训练文件夹未清空?)

```
question[0] total delta = 201.0
question[1] total delta = 203.0
question[2] total delta = 232.0
question[3] total delta = 164.0
question[4] total delta = 272.0
question[5] total delta = 117.0
question[6] total delta = 331.0
question[7] total delta = 301.0
question[8] total delta = 181.0
question[9] total delta = 323.0
all delta = 2325.0
``` 

使用 15-150 作为训练数据, 进行微调, epoch = 2, per_device_train_batch_size=8, loss=0.86, 效果: 

```
question[0] total delta = 263.0
question[1] total delta = 175.0
question[2] total delta = 245.0
question[3] total delta = 165.0
question[4] total delta = 277.0
question[5] total delta = 165.0
question[6] total delta = 333.0
question[7] total delta = 359.0
question[8] total delta = 147.0
question[9] total delta = 340.0
all delta = 2469.0
``` 

  
使用 15-150 作为训练数据, 进行微调, epoch = 2, per_device_train_batch_size=1, loss=0.83, 效果:

```
question[0] total delta = 600.0
question[1] total delta = 600.0
question[2] total delta = 600.0
question[3] total delta = 567.0
question[4] total delta = 542.0
question[5] total delta = 600.0
question[6] total delta = 600.0
question[7] total delta = 545.0
question[8] total delta = 600.0
question[9] total delta = 489.0
all delta = 5743.0
``` 

# 对于训练数据的调整

在训练数据101 - 150中, 这些训练问题来自于stackoverflow的问题. 

由于知识库使用的是公司工单库中的局部, 导致这些问题命中的文档不足, 可能对训练效果的贡献非常有限. 

准备用ChatGPT-4生成一些问题, 使得其知识领域与原问题相近.

并且需要增加一个评分系统, 将relevance文档过少的问题先排除掉 (并重新进行问题编号)

提示词: 

```
我有一些关于MySQL的问题样例, 我希望你生成类似的10个问题, 这些问题将用于我训练一个模型. 我希望这些问题:
与我给出的问题, 在MySQL的知识点上是一致的, 或者说在MySQL中的知识领域是一样的
提问的形式变得更多样, 有的简单, 有的复杂
这些问题是真实的, 是现实世界中能出现的问题

问题样例:
mysql修改密码后用navicat 登陆报错1862
数据库show engine innodb status中存在大量row locks记录，数据库是否存在异常？
Mysql主从同步时Slave_SQL_Running状态为Yes , 但是Slave_IO_Running状态为Connecting以及NO
主从切换后原主库比新主库多数据
多个从库出现延迟
数据库主从延迟严重
主从复制报错：HA_ERR_KEY_NOT_FOUND
 
---
 
修改MySQL root用户密码后，无法通过命令行连接，显示'Access denied for user 'root'@'localhost' (using password: YES)'，应如何解决？
如何在MySQL中启用慢查询日志以监控性能低下的查询？
在尝试导入大型SQL文件时，MySQL报错：'max_allowed_packet'大小不足，我应该如何调整这个参数？
MySQL数据库意外崩溃后，有哪些步骤可以恢复数据？
如何在不停机的情况下，对MySQL的表进行在线重建索引操作？
如果发现MySQL的某个表索引丢失，如何恢复或重建该索引？
MySQL备份时提示磁盘空间不足，怎样清理或优化空间以完成备份？
如何在MySQL中配置和管理二进制日志（binlog）的自动清理？
为什么MySQL中有些查询锁定了表，但是SHOW PROCESSLIST看不到锁定的线程？
如何确定MySQL中InnoDB缓冲池的大小是否适合当前的工作负载？
``` 

问题列表: 

```
MySQL备份时出现错误："mysqldump: Got error: 1049: Unknown database"，应如何解决？
在执行MySQL JOIN查询时，响应时间过长，如何优化性能？
如何在MySQL中恢复意外删除的数据？
MySQL服务器频繁重启，如何定位问题根源？
在尝试导入大型SQL文件时，MySQL报错："ERROR 2013 (HY000) at line 55: Lost connection to MySQL server during query"，可能的原因是什么？
如何在MySQL中实施行级锁定以提高并发性能？
当尝试连接到MySQL数据库时，遇到错误："ERROR 2003 (HY000): Can't connect to MySQL server on '127.0.0.1' (61)"，应该如何故障排除？
如何监控MySQL数据库的实时性能并分析慢查询？
尝试使用UTF-8编码存储数据时，MySQL报错："Incorrect string value"，如何解决字符集问题？
在使用存储过程处理数据时，MySQL遇到锁等待超时："ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction"，如何优化事务处理？
当尝试执行事务时，MySQL报错："Deadlock found when trying to get lock; try restarting transaction"，应如何解决？
MySQL的InnoDB表频繁出现行锁争用，如何监控并解决性能瓶颈？
如何处理MySQL中的Too many connections错误？
MySQL数据库在执行大批量数据更新时变得非常缓慢，如何优化？
使用LOAD DATA INFILE导入数据时MySQL报错："ERROR 29 (HY000): File not found / (Errcode: 13)", 如何解决权限问题？
在尝试修改MySQL数据表结构时，遭遇锁表现象，如何减少对业务的影响？
如何诊断并修复MySQL的表碎片化问题？
MySQL从库同步延迟严重，如何实时监控并快速定位问题？
如何恢复MySQL中因操作错误而删除的重要数据行？
MySQL数据库执行批量删除操作时报错："ERROR 1206 (HY000): The total number of locks exceeds the lock table size"，如何调整配置避免此错误？
MySQL中GTID复制出现错误："Error on master: gtid_mode is ON but the master has purged binary logs containing GTIDs that the slave requires"，如何修复？
如何解决MySQL的主从复制延迟问题，当复制延迟超过一定阈值时自动报警？
在大量并发写入操作下，MySQL的InnoDB表出现死锁，如何检测并预防这种情况？
MySQL数据库在执行批量插入操作时出现错误："ERROR 1114 (HY000): The table 'tbl_name' is full"，如何处理？
如何在MySQL中配置和使用慢查询日志来识别低效的SQL语句？
MySQL数据库的表损坏了，显示错误："Table './db/tbl_name' is marked as crashed and should be repaired"，怎么修复？
MySQL实例CPU占用率异常升高，如何定位到具体的查询或操作？
MySQL复制错误："Slave_IO_Running: No"和"Last_IO_Error: error connecting to master"，该如何排查并解决？
当MySQL的binlog文件被意外删除，导致主从复制中断，应如何恢复复制过程？
在尝试对MySQL数据库升级时，服务无法启动并报错："Unknown/unsupported storage engine: InnoDB"，可能的原因及解决方案是什么？
当MySQL主库发生故障，从库如何提升为主库以确保业务连续性？
MySQL数据库出现大量慢查询，如何优化查询性能？
触发MySQL数据库死锁，怎样定位问题并解决？
数据库频繁出现ER_NET_PACKET_TOO_LARGE错误，如何配置以避免这个问题？
发现MySQL定时任务（Event Scheduler）未按预期执行，应如何排查原因？
在尝试连接到MySQL数据库时，报错ERROR 1045 (28000): Access denied for user，应如何解决？
如何检测并修复MySQL InnoDB表的索引损坏问题？
在进行MySQL数据库迁移时遇到数据不一致问题，如何确保数据的完整性和一致性？
如何通过MySQL的优化器跟踪（Optimizer Trace）功能来理解和解决复杂查询计划选择不当的问题？
如果MySQL的GTID复制遇到不一致的GTID序列，如何安全地重新同步数据且不丢失任何事务？
```
