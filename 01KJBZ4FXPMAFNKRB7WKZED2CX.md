---
title: 20231217 - 进行oceanbase编译
confluence_page_id: 2589910
created_at: 2023-12-17T14:56:34+00:00
updated_at: 2023-12-25T10:08:12+00:00
---

clone代码: 

```
git clone http://10.186.18.21/huangyan/actiondb-audit.git
``` 

```
docker login reg.actiontech.com
 
#在弹出的输入框中输入如下信息：
# username:robot$actiontech-ci-agent
# password:xi3mdE1NZEfGaT3NgnbR7dQ1Kkwlw0bW
  
#拉取ob-dev:latest镜像
docker pull reg.actiontech.com/actiontech-dev/ob-dev:latest
 
启动容器
docker run -it --name huangyan-oceanbase-compiler --privileged -p 33301:3301 -p 22202:3302 -v /data/huangyan/actiondb-audit:/oceanbase -e TZ=Asia/Shanghai reg.actiontech.com/actiontech-dev/ob-dev:latest
``` 

在容器内: 

```
./build.sh init
./build.sh debug -DCMAKE_EXPORT_COMPILE_COMMANDS=1
make -j40 observer
``` 

初始化目录: 

```
cd /data/ob_log/single_ob1
mkdir clog
mkdir etc2
mkdir etc3
mkdir ilog
mkdir oob_clog
mkdir slog
mkdir sort_dir
mkdir sstable
rm -rf */*
``` 

启动: 

```
cd /oceanbase/single_ob1 && /oceanbase/build_debug/src/observer/observer -i eth0  -p 33301 -P 22202 -z dbg_z1 -d /oceanbase/store/single_ob1 -r "172.17.0.12:22202:33301" -c 6661018 -n single_ob1 -o "memory_limit=16G,cache_wash_threshold=1G,system_memory=4G,memory_chunk_cache_size=128M,cpu_count=4,net_thread_count=4,datafile_size=40G,stack_size=1536K,__min_full_resource_pool_memory=1073741824,max_syslog_file_count=5,enable_syslog_recycle=1,log_disk_size=50G,config_additional_dir=/data/ob_data/single_ob1/etc3;/data/ob_log/single_ob1/etc2"
``` 

初始化数据库: 

```
mysql -h127.0.0.1 -uroot -P33301

> SET SESSION ob_query_timeout = 1200000000;
> ALTER SYSTEM BOOTSTRAP ZONE 'dbg_z1' SERVER '172.17.0.12:22202';
``` 

检查输出: 

```
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| LBACSYS            |
| mysql              |
| oceanbase          |
| ORAAUDITOR         |
| SYS                |
| test               |
+--------------------+
7 rows in set (0.03 sec)
```
