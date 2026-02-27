---
title: 20231214 - 诊断xbcloud crash
confluence_page_id: 2589886
created_at: 2023-12-14T02:33:06+00:00
updated_at: 2023-12-16T16:30:23+00:00
---

# 状况

<http://10.186.18.11/confluence/pages/viewpage.action?pageId=124289596>

现象：客户arm和x86环境从s3上恢复备份时，xbcloud发生异常崩溃。

系统版本： UnionTech OS Server 20 1050e / Kylin Linux release Advanced Server V10 (SP3) /(Lance)-aarch64-ild23/20230324

DMP版本：5.23.01.8/rel

xbcloud版本： 2.4.24

xbcloud测试命令：

./xbcloud get --storage=s3 --s3-endpoint='[obs.cn-east-3.myhuaweicloud.com](<http://obs.cn-east-3.myhuaweicloud.com/>)' --s3-access-key='PPLTXC5MIAU7DWAVMV6Q' --s3-secret-key='EnWMWqKPIDPjlcRpGmJOybWxYcHHzWwP9C0bouKa' --s3-bucket='actionsky-sbb' --parallel=10 --insecure mysql-uos/opt/udb/mysql/backup/manual_mysql-uos_2023_12_08_14_32_29/2023_12_08_14_32_29 > /tmp/test

本地环境：

  - 统信V20: 192.168.20.3 root/123(与客户环境完全一致)，/opt/urman-agent/bin下有 客户现场的core文件和rpm包中的 xbcloud
  - kylinV10镜像: 10.186.60.44 root/123 container_id:9524f6163e

主要堆栈: 

![image2023-12-14 10:30:21.png](/assets/01KJBZ4D06APPE59ND24QFT0WQ/image2023-12-14%2010%3A30%3A21.png)

# 诊断

尝试在本地复现, 但无法复现. 

想要 对比本地环境和客户环境复现堆栈的区别, 但堆栈中openssl的部分无法解析

需要首先编译debug版本的openssl: 

```
下载openssl 1.1.1f源码: https://github.com/openssl/openssl/archive/refs/tags/OpenSSL_1_1_1f.tar.gz, 假设目录为/opt/openssl-OpenSSL_1_1_1f/

cd /opt/openssl-OpenSSL_1_1_1f/

./config shared enable-ssl enable-ssl3-method enable-tls enable-tls1_3 enable-ssl3

make

LD_LIBRARY_PATH=/opt/openssl-OpenSSL_1_1_1f/ ./xbcloud ......
``` 

这样就可以看到崩溃堆栈的细节: 

![image2023-12-17 0:10:41.png](/assets/01KJBZ4D06APPE59ND24QFT0WQ/image2023-12-17%200%3A10%3A41.png)

```
Thread 1 "xbcloud" received signal SIGSEGV, Segmentation fault.
0x00007ffff7f9c842 in pthread_rwlock_wrlock () from /usr/lib64/libpthread.so.0
(gdb) bt
#0  0x00007ffff7f9c842 in pthread_rwlock_wrlock () from /usr/lib64/libpthread.so.0
#1  0x00007ffff7c087c9 in CRYPTO_THREAD_write_lock (lock=<optimized out>) at crypto/threads_pthread.c:78
#2  0x00007ffff7bc6c23 in RAND_get_rand_method () at crypto/rand/rand_lib.c:849
#3  0x00007ffff7bc6f10 in RAND_bytes (buf=0x7f4fe8 " \022l\367\377\177", num=16) at crypto/rand/rand_lib.c:936
#4  0x00007ffff7d22d8e in tls1_enc (s=0x8b4050, recs=0x7fffffffd580, n_recs=1, sending=1) at ssl/record/ssl3_record.c:985
#5  0x00007ffff7d1ffe0 in do_ssl3_write (s=s@entry=0x8b4050, type=type@entry=21, buf=0x8b59e0 "\001", pipelens=pipelens@entry=0x7fffffffdfd0,
    numpipes=numpipes@entry=1, create_empty_fragment=create_empty_fragment@entry=0, written=0x7fffffffdfd8) at ssl/record/rec_layer_s3.c:1011
#6  0x00007ffff7d2aa99 in ssl3_dispatch_alert (s=0x8b4050) at ssl/s3_msg.c:78
#7  0x00007ffff7d28aa5 in ssl3_shutdown (s=0x8b4050) at ssl/s3_lib.c:4418
#8  0x00007ffff7d35613 in SSL_shutdown (s=0x8b4050) at ssl/ssl_lib.c:2083
#9  0x00007ffff7e03312 in ?? () from /usr/lib64/libcurl.so.4
#10 0x00007ffff7e03371 in ?? () from /usr/lib64/libcurl.so.4
#11 0x00007ffff7dc1408 in ?? () from /usr/lib64/libcurl.so.4
#12 0x00007ffff7daeefb in ?? () from /usr/lib64/libcurl.so.4
#13 0x00007ffff7ddd8c8 in curl_multi_cleanup () from /usr/lib64/libcurl.so.4
#14 0x00007ffff7dc179f in ?? () from /usr/lib64/libcurl.so.4
#15 0x00007ffff7db8619 in curl_easy_cleanup () from /usr/lib64/libcurl.so.4
#16 0x0000000000409866 in std::unique_ptr<void, void (*)(void*)>::~unique_ptr (this=0x7b2900 <http_client+96>, __in_chrg=<optimized out>)
    at /usr/include/c++/7.3.0/bits/unique_ptr.h:268
#17 xbcloud::Http_client::~Http_client (this=0x7b28a0 <http_client>, __in_chrg=<optimized out>)
    at /root/percona-xtrabackup/storage/innobase/xtrabackup/src/xbcloud/http.h:336
#18 0x00007ffff754adf1 in ?? () from /usr/lib64/libc.so.6
#19 0x00007ffff754aeea in exit () from /usr/lib64/libc.so.6
#20 0x00007ffff7534aee in __libc_start_main () from /usr/lib64/libc.so.6
#21 0x000000000040956a in _start ()
``` 

在本地无法复现的场景中, ssl3_shutdown不会调用RAND_bytes, 相关代码: 

![image2023-12-17 0:12:21.png](/assets/01KJBZ4D06APPE59ND24QFT0WQ/image2023-12-17%200%3A12%3A21.png)

其逻辑与加密算法是否是CBC类型的加密算法有关. 确实用户环境上的加密算法是: DHE-RSA-AES256-SHA, 而本地环境不是CBC类型算法

![image2023-12-17 0:13:1.png](/assets/01KJBZ4D06APPE59ND24QFT0WQ/image2023-12-17%200%3A13%3A1.png)

小知识: 关于如何确认算法是否是CBC算法: 需要获取算法全称: <https://ciphersuite.info/cs/TLS_DHE_RSA_WITH_AES_256_CBC_SHA/>

![image2023-12-17 0:13:58.png](/assets/01KJBZ4D06APPE59ND24QFT0WQ/image2023-12-17%200%3A13%3A58.png)

下一步需要模拟客户环境, 使用CBC类型算法

按照 [20231215 - 配置minio, 将s3 API的加密套件配置为TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA], 配置minio+nginx, 接受ssh连接, 只使用指定算法

(但由于秘钥配置原因, 无法完成验证, 所以xbcloud需要使用–insecure来跳过ssh验证, 但这样需要强制指定xbcloud的算法, 才能指定算法)

修改xbcloud代码: 

![image2023-12-17 0:16:37.png](/assets/01KJBZ4D06APPE59ND24QFT0WQ/image2023-12-17%200%3A16%3A37.png)

  - 修改此处, 设置参数 CURLOPT_SSLVERSION 将最大版本设置成TLS1.2, 他会自动使用 ECDHE-RSA-AES256-SHA 密码套件 也是CBC的 就复现了上面的问题

自此得到了复现环境. 复现命令: 

```
LD_LIBRARY_PATH=/root/openssl-OpenSSL_1_1_1f ./xbcloud get --cbc --verbose --storage=s3 --s3-endpoint='10.186.16.126:9443' --s3-access-key='orkWZnHN0kV04P9uMmHW' --s3-secret-key='0onGmVjiSHYrplq21iYvvXIvGAJqUy4nNlCU8d7w' --s3-bucket='test' --parallel=10 --insecure test
``` 

诊断崩溃原因, 是 变量 rand_meth_lock 变为了空, 导致崩溃. 因此用gdb watchpoint监控变量: 

```
rand_meth_lock 赋值: 
 
(gdb) bt
#0  do_rand_init () at crypto/rand/rand_lib.c:315
#1  do_rand_init_ossl_ () at crypto/rand/rand_lib.c:306
#2  0x00007ffff7f9f647 in ?? () from /usr/lib64/libpthread.so.0
#3  0x00007ffff7c08839 in CRYPTO_THREAD_run_once (once=once@entry=0x7ffff7cf8dc8 <rand_init>, init=init@entry=0x7ffff7bc5c60 <do_rand_init_ossl_>)
    at crypto/threads_pthread.c:118
#4  0x00007ffff7bc6c09 in RAND_get_rand_method () at crypto/rand/rand_lib.c:846
#5  0x00007ffff7bc7029 in RAND_status () at crypto/rand/rand_lib.c:958
#6  0x00007ffff7e03529 in ?? () from /usr/lib64/libcurl.so.4
#7  0x00007ffff7e0620d in ?? () from /usr/lib64/libcurl.so.4
#8  0x00007ffff7e082f0 in ?? () from /usr/lib64/libcurl.so.4
#9  0x00007ffff7e093a7 in ?? () from /usr/lib64/libcurl.so.4
#10 0x00007ffff7dc9342 in ?? () from /usr/lib64/libcurl.so.4
#11 0x00007ffff7dca99b in ?? () from /usr/lib64/libcurl.so.4
#12 0x00007ffff7ddde8a in ?? () from /usr/lib64/libcurl.so.4
#13 0x00007ffff7ddf203 in curl_multi_perform () from /usr/lib64/libcurl.so.4
#14 0x00007ffff7db8563 in curl_easy_perform () from /usr/lib64/libcurl.so.4
#15 0x000000000041eff8 in xbcloud::Http_client::make_request (this=0x7b28a0 <http_client>, request=..., response=...)
    at /root/percona-xtrabackup/storage/innobase/xtrabackup/src/xbcloud/http.cc:561
#16 0x000000000042189f in xbcloud::S3_client::bucket_exists (this=this@entry=0x7e4d58, name="test", exists=@0x7fffffffe308: 4)
    at /root/percona-xtrabackup/storage/innobase/xtrabackup/src/xbcloud/s3.cc:485
#17 0x0000000000421e7f in xbcloud::S3_client::probe_api_version_and_lookup (this=this@entry=0x7e4d58, bucket="test")
    at /root/percona-xtrabackup/storage/innobase/xtrabackup/src/xbcloud/s3.cc:459
#18 0x000000000040836d in xbcloud::S3_object_store::probe_api_version_and_lookup (bucket="test", this=<optimized out>)
    at /root/percona-xtrabackup/storage/innobase/xtrabackup/src/xbcloud/s3.h:283
#19 main (argc=<optimized out>, argv=<optimized out>) at /root/percona-xtrabackup/storage/innobase/xtrabackup/src/xbcloud/xbcloud.cc:1206
```
```
rand_meth_lock 清空
 
Thread 1 "xbcloud" hit Hardware watchpoint 2: rand_meth_lock

Old value = (CRYPTO_RWLOCK *) 0x7ef1f0
New value = (CRYPTO_RWLOCK *) 0x0
rand_cleanup_int () at crypto/rand/rand_lib.c:359
359	    CRYPTO_THREAD_lock_free(rand_nonce_lock);
(gdb) bt
#0  rand_cleanup_int () at crypto/rand/rand_lib.c:359
#1  0x00007ffff7b92f6b in OPENSSL_cleanup () at crypto/init.c:597
#2  0x00007ffff754adf1 in ?? () from /usr/lib64/libc.so.6
#3  0x00007ffff754aeea in exit () from /usr/lib64/libc.so.6
#4  0x00007ffff7534aee in __libc_start_main () from /usr/lib64/libc.so.6
#5  0x000000000040956a in _start ()
``` 

判断原因: 

  - OPENSSL_cleanup 是使用atexit 注册到程序退出的钩子中
  - 程序退出时, 先调用了OPENSSL_cleanup, 然后调用各对象的析构函数, (Http_client的析构函数中, 会清理其中的curl对象), 因此会触发崩溃的堆栈

解决方案: 

  - 手工将HTTP_client和curl对象脱钩, 这样析构Http_client时, 就不会调用curl对象的析构, 不触发崩溃

代码: 

![image2023-12-17 0:23:43.png](/assets/01KJBZ4D06APPE59ND24QFT0WQ/image2023-12-17%200%3A23%3A43.png)
