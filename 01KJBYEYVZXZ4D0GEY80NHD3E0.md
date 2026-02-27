---
title: 20210928 - tcmalloc 不同so包性能有差异
confluence_page_id: 1343763
created_at: 2021-09-28T07:01:39+00:00
updated_at: 2021-10-08T10:28:42+00:00
---

# 现象

libtcmalloc_minimal.so 与 libtcmalloc.so 性能差很多

tcmalloc版本: gperftools 2.9.1

启动MySQL命令: 

```
LD_PRELOAD=/opt/gperftools/libtcmalloc_minimal.so HEAPPROFILE=/data/liukaiyang/mysql-profile-insert HEAP_PROFILE_INUSE_INTERVAL=0 HEAP_PROFILE_ALLOCATION_INTERVAL=0 HEAPPROFILESIGNAL=21 HEAP_PROFILE_MMAP=true HEAPCHECK=minimal nohup /data/liukaiyang/mysql/8.0.19/bin/mysqld --defaults-file=/data/liukaiyang/sandboxes/msb_8_0_19/my.sandbox.cnf --basedir=/data/liukaiyang/sandboxes/msb_8_0_19 --datadir=/data/liukaiyang/sandboxes/msb_8_0_19/data --plugin-dir=/usr/local/mysql/lib/plugin --user=root --log-error=/data/liukaiyang/sandboxes/msb_8_0_19/data/msandbox.err --open-files-limit=65535 --pid-file=/data/liukaiyang/sandboxes/msb_8_0_19/data/mysql_sandbox8019.pid --socket=/tmp/mysql_sandbox8019.sock --port=8019 &
``` 

sysbench命令: 

```
/usr/local/sysbench/bin/sysbench /usr/local/share/sysbench/oltp_point_select.lua --mysql-host=127.0.0.1 --mysql-port=8019 --mysql-user=root --mysql-password=123 --mysql-db=test --tables=10 --table-size=1000000 --threads=10  --events=0 --time=300 --report-interval=3 --db-ps-mode=disable run
``` 

性能: 

libtcmalloc_minimal: tps 10624.81 起步, 逐渐上升

libtcmalloc: tps 5.32 起步, 逐渐上升

# 探索

观察编译日志: 

libtcmalloc_minimal

```
libtool: link: g++ -std=gnu++11  -fPIC -DPIC -shared -nostdlib /usr/lib/gcc/x86_64-redhat-linux/4.8.5/../../../../lib64/crti.o /usr/lib/gcc/x86_64-redhat-linux/4.8.5/crtbeginS.o  src/.libs/libtcmalloc_minimal_la-tcmalloc.o  -Wl,--whole-archive ./.libs/libtcmalloc_minimal_internal.a -Wl,--no-whole-archive  -L/usr/lib/gcc/x86_64-redhat-linux/4.8.5 -L/usr/lib/gcc/x86_64-redhat-linux/4.8.5/../../../../lib64 -L/lib/../lib64 -L/usr/lib/../lib64 -L/usr/lib/gcc/x86_64-redhat-linux/4.8.5/../../.. -lstdc++ -lm -lc -lgcc_s /usr/lib/gcc/x86_64-redhat-linux/4.8.5/crtendS.o /usr/lib/gcc/x86_64-redhat-linux/4.8.5/../../../../lib64/crtn.o  -pthread -g -O2   -pthread -Wl,-soname -Wl,libtcmalloc_minimal.so.4 -o .libs/libtcmalloc_minimal.so.4.5.9
libtool: link: (cd ".libs" && rm -f "libtcmalloc_minimal.so.4" && ln -s "libtcmalloc_minimal.so.4.5.9" "libtcmalloc_minimal.so.4")
libtool: link: (cd ".libs" && rm -f "libtcmalloc_minimal.so" && ln -s "libtcmalloc_minimal.so.4.5.9" "libtcmalloc_minimal.so")
``` 

libtcmalloc

```
libtool: link: g++ -std=gnu++11  -fPIC -DPIC -shared -nostdlib /usr/lib/gcc/x86_64-redhat-linux/4.8.5/../../../../lib64/crti.o /usr/lib/gcc/x86_64-redhat-linux/4.8.5/crtbeginS.o  src/.libs/libtcmalloc_la-tcmalloc.o src/base/.libs/thread_lister.o src/base/.libs/libtcmalloc_la-linuxthreads.o src/.libs/libtcmalloc_la-heap-checker.o src/.libs/libtcmalloc_la-heap-checker-bcad.o  -Wl,--whole-archive ./.libs/libtcmalloc_internal.a ./.libs/libmaybe_threads.a -Wl,--no-whole-archive  -lpthread -L/usr/lib/gcc/x86_64-redhat-linux/4.8.5 -L/usr/lib/gcc/x86_64-redhat-linux/4.8.5/../../../../lib64 -L/lib/../lib64 -L/usr/lib/../lib64 -L/usr/lib/gcc/x86_64-redhat-linux/4.8.5/../../.. -lstdc++ -lm -lc -lgcc_s /usr/lib/gcc/x86_64-redhat-linux/4.8.5/crtendS.o /usr/lib/gcc/x86_64-redhat-linux/4.8.5/../../../../lib64/crtn.o  -pthread -g -O2 -pthread   -pthread -Wl,-soname -Wl,libtcmalloc.so.4 -o .libs/libtcmalloc.so.4.5.9
libtool: link: (cd ".libs" && rm -f "libtcmalloc.so.4" && ln -s "libtcmalloc.so.4.5.9" "libtcmalloc.so.4")
libtool: link: (cd ".libs" && rm -f "libtcmalloc.so" && ln -s "libtcmalloc.so.4.5.9" "libtcmalloc.so")
``` 

比较编译命令差异: 

![image2021-9-28 15:9:2.png](/assets/01KJBYEYVZXZ4D0GEY80NHD3E0/image2021-9-28%2015%3A9%3A2.png)

libtcmalloc_la-tcmalloc.o

```
libtool: compile:  g++ -std=gnu++11 -DHAVE_CONFIG_H -I. -I./src -I./src -pthread -DNDEBUG -Wall -Wwrite-strings -Woverloaded-virtual -Wno-sign-compare -Wno-unused-result -DNO_F
RAME_POINTER -DENABLE_EMERGENCY_MALLOC -g -O2 -MT src/libtcmalloc_la-tcmalloc.lo -MD -MP -MF src/.deps/libtcmalloc_la-tcmalloc.Tpo -c src/tcmalloc.cc  -fPIC -DPIC -o src/.libs/
libtcmalloc_la-tcmalloc.o
libtool: compile:  g++ -std=gnu++11 -DHAVE_CONFIG_H -I. -I./src -I./src -pthread -DNDEBUG -Wall -Wwrite-strings -Woverloaded-virtual -Wno-sign-compare -Wno-unused-result -DNO_F
RAME_POINTER -DENABLE_EMERGENCY_MALLOC -g -O2 -MT src/libtcmalloc_la-tcmalloc.lo -MD -MP -MF src/.deps/libtcmalloc_la-tcmalloc.Tpo -c src/tcmalloc.cc -o src/libtcmalloc_la-tcma
lloc.o >/dev/null 2>&1
``` 

libtcmalloc_minimal_la-tcmalloc.o

```
libtool: compile:  g++ -std=gnu++11 -DHAVE_CONFIG_H -I. -I./src -I./src -DNO_TCMALLOC_SAMPLES -pthread -DNDEBUG -Wall -Wwrite-strings -Woverloaded-virtual -Wno-sign-compare -Wn
o-unused-result -DNO_FRAME_POINTER -g -O2 -MT src/libtcmalloc_minimal_la-tcmalloc.lo -MD -MP -MF src/.deps/libtcmalloc_minimal_la-tcmalloc.Tpo -c src/tcmalloc.cc  -fPIC -DPIC -
o src/.libs/libtcmalloc_minimal_la-tcmalloc.o
libtool: compile:  g++ -std=gnu++11 -DHAVE_CONFIG_H -I. -I./src -I./src -DNO_TCMALLOC_SAMPLES -pthread -DNDEBUG -Wall -Wwrite-strings -Woverloaded-virtual -Wno-sign-compare -Wn
o-unused-result -DNO_FRAME_POINTER -g -O2 -MT src/libtcmalloc_minimal_la-tcmalloc.lo -MD -MP -MF src/.deps/libtcmalloc_minimal_la-tcmalloc.Tpo -c src/tcmalloc.cc -o src/libtcma
lloc_minimal_la-tcmalloc.o >/dev/null 2>&1
``` 

比较编译差异: 

![image2021-9-28 15:22:45.png](/assets/01KJBYEYVZXZ4D0GEY80NHD3E0/image2021-9-28%2015%3A22%3A45.png)

发现: 去掉HEAPPROFILE后, libtcmalloc.so 性能恢复正常

# 最终结论: 

见 [20211001 - 为什么tcmalloc会慢]
