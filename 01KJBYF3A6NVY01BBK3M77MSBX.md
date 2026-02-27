---
title: 20211013 - 探索 dynimize 原理
confluence_page_id: 1343808
created_at: 2021-10-13T07:11:23+00:00
updated_at: 2021-10-28T14:34:03+00:00
---

# 分析安装脚本

  1. 重点文件: /opt/dynimize/measureDyniMysql
  2. 变更了cron: /etc/cron.weekly/dyni
  3. 重点文件: /usr/sbin/dyni
  4. 常用命令: 
     1. dyni -start
     2. dyni -status
     3. dyni -stop
     4. dyni -undo
     5. dyni -license=start -token=vUPvA1nwWh73MDYE3846

# 通过strace分析

```
strace -f -o /tmp/dyni-redo.strace dyni -redo
``` 

[附件: dyni-redo.strace.zip] 

找到特殊的syscall:

  - process_vm_readv
  - process_vm_writev
  - rt_sigaction
  - execve("/bin/sh", ["sh", "-c", "(echo 99 > /proc/sys/kernel/perf"...], 0x7ffc280f2230 /* 24 vars */) = 0
  - openat(AT_FDCWD, "/proc/sys/kernel/perf_cpu_time_max_percent", O_WRONLY|O_CREAT|O_TRUNC, 0666)
  - openat(AT_FDCWD, "/proc/sys/kernel/perf_event_max_sample_rate", O_WRONLY|O_CREAT|O_TRUNC, 0666)
  - perf_event_open({type=PERF_TYPE_HARDWARE, size=PERF_ATTR_SIZE_VER3, config=PERF_COUNT_HW_CPU_CYCLES, ...}, -1, 0, -1, 0) = -1 ENOENT (No such file or directory)

  - perf_event_open({type=PERF_TYPE_SOFTWARE, size=PERF_ATTR_SIZE_VER3, config=PERF_COUNT_SW_TASK_CLOCK, ...}, -1, 0, -1, 0) = 6

  - ptrace(PTRACE_ATTACH, 31783) = 0
  - ptrace(PTRACE_GETREGS, 31783, NULL, 0x7ff17d765458) = 0

  - ptrace(PTRACE_GETFPREGS, 31783, NULL, 0x7ff17d765930) = 0

  - ptrace(PTRACE_PEEKDATA, 31783, 0x7fee946895b1, [0x5554415541564157]) = 0
  - ptrace(PTRACE_SETREGS, 31783, NULL, 0x7ffc280ebbb0) = 0

  - ptrace(PTRACE_SETFPREGS, 31783, NULL, 0x7ffc280ec088) = 0

  - ptrace(PTRACE_CONT, 31783, NULL, SIG_0) = 0

  - ptrace(PTRACE_POKEDATA, 31783, 0x15360090, 0x480a771739480f75) = 0
  - ptrace(PTRACE_PEEKTEXT, 31783, 0x15224000, [NULL]) = 0  
  

调整dyni自身的nice

  - execve("/bin/sh", ["sh", "-c", "renice -20 17371 > /dev/null 2>&"...], 0x7ffc280f2230 /* 24 vars */) = 0
  - setpriority(PRIO_PROCESS, 17371, -20) = 0

  - getpriority(PRIO_PROCESS, 17371) = 40

# 对ptrace进行分析

将POKE相关的地址分离出来: 

```
less /tmp/dyni-redo.strace | grep ptrace | grep POKE | awk '{print $4}' | sort | uniq | less
``` 

获取进程的内存分布: 

```
root@ubuntu:~# cat /proc/31783/maps
00400000-03874000 r-xp 00000000 fc:01 1039240                            /root/opt/mysql/8.0.25/bin/mysqld
03874000-039e5000 r--p 03473000 fc:01 1039240                            /root/opt/mysql/8.0.25/bin/mysqld
039e5000-03d6b000 rw-p 035e4000 fc:01 1039240                            /root/opt/mysql/8.0.25/bin/mysqld
03d6b000-0428f000 rw-p 00000000 00:00 0
04745000-06f9e000 rw-p 00000000 00:00 0                                  [heap]
06f9e000-090ff000 r-xp 00000000 00:00 0                                  [heap]
090ff000-0bb20000 rw-p 00000000 00:00 0
0bb20000-0dc81000 r-xp 00000000 00:00 0
0dc81000-106a2000 rw-p 00000000 00:00 0
106a2000-12803000 r-xp 00000000 00:00 0
12803000-15224000 rw-p 00000000 00:00 0
15224000-17385000 r-xp 00000000 00:00 0
17385000-19da6000 rw-p 00000000 00:00 0
19da6000-1bf07000 r-xp 00000000 00:00 0
1bf07000-1e928000 rw-p 00000000 00:00 0
1e928000-20a89000 r-xp 00000000 00:00 0
20a89000-234aa000 rw-p 00000000 00:00 0
7fed60000000-7fed60045000 rw-p 00000000 00:00 0
7fed60045000-7fed64000000 ---p 00000000 00:00 0
7fed64000000-7fed64052000 rw-p 00000000 00:00 0
7fed64052000-7fed68000000 ---p 00000000 00:00 0
7fed68000000-7fed6804a000 rw-p 00000000 00:00 0
7fed6804a000-7fed6c000000 ---p 00000000 00:00 0
7fed6c000000-7fed6c040000 rw-p 00000000 00:00 0
7fed6c040000-7fed70000000 ---p 00000000 00:00 0
7fed70000000-7fed70047000 rw-p 00000000 00:00 0
7fed70047000-7fed74000000 ---p 00000000 00:00 0
7fed74000000-7fed74073000 rw-p 00000000 00:00 0
7fed74073000-7fed78000000 ---p 00000000 00:00 0
7fed78000000-7fed7806a000 rw-p 00000000 00:00 0
7fed7806a000-7fed7c000000 ---p 00000000 00:00 0
7fed7c000000-7fed7c06f000 rw-p 00000000 00:00 0
7fed7c06f000-7fed80000000 ---p 00000000 00:00 0
7fed80000000-7fed80085000 rw-p 00000000 00:00 0
7fed80085000-7fed84000000 ---p 00000000 00:00 0
7fed84000000-7fed84072000 rw-p 00000000 00:00 0
7fed84072000-7fed88000000 ---p 00000000 00:00 0
7fed88000000-7fed88077000 rw-p 00000000 00:00 0
7fed88077000-7fed8c000000 ---p 00000000 00:00 0
7fed8c000000-7fed8c073000 rw-p 00000000 00:00 0
7fed8c073000-7fed90000000 ---p 00000000 00:00 0
7fed90000000-7fed9006f000 rw-p 00000000 00:00 0
7fed9006f000-7fed94000000 ---p 00000000 00:00 0
7fed94000000-7fed94073000 rw-p 00000000 00:00 0
7fed94073000-7fed98000000 ---p 00000000 00:00 0
7fed98000000-7fed98082000 rw-p 00000000 00:00 0
7fed98082000-7fed9c000000 ---p 00000000 00:00 0
7fed9c000000-7fed9c06e000 rw-p 00000000 00:00 0
7fed9c06e000-7feda0000000 ---p 00000000 00:00 0
7feda0000000-7feda0075000 rw-p 00000000 00:00 0
7feda0075000-7feda4000000 ---p 00000000 00:00 0
7feda4000000-7feda4a51000 rw-p 00000000 00:00 0
7feda4a51000-7feda8000000 ---p 00000000 00:00 0
7feda8000000-7fedaa6e6000 rw-p 00000000 00:00 0
7fedaa6e6000-7fedac000000 ---p 00000000 00:00 0
7fedac000000-7fedaca4d000 rw-p 00000000 00:00 0
7fedaca4d000-7fedb0000000 ---p 00000000 00:00 0
7fedb0000000-7fedb0071000 rw-p 00000000 00:00 0
7fedb0071000-7fedb4000000 ---p 00000000 00:00 0
7fedb8000000-7fedb806e000 rw-p 00000000 00:00 0
7fedb806e000-7fedbc000000 ---p 00000000 00:00 0
7fedbc000000-7fedbc9bd000 rw-p 00000000 00:00 0
7fedbc9bd000-7fedc0000000 ---p 00000000 00:00 0
7fedc0000000-7fedc0074000 rw-p 00000000 00:00 0
7fedc0074000-7fedc4000000 ---p 00000000 00:00 0
7fedc4000000-7fedc406f000 rw-p 00000000 00:00 0
7fedc406f000-7fedc8000000 ---p 00000000 00:00 0
7fedc8000000-7fedc89e2000 rw-p 00000000 00:00 0
7fedc89e2000-7fedcc000000 ---p 00000000 00:00 0
7fedcc000000-7fedcca53000 rw-p 00000000 00:00 0
7fedcca53000-7fedd0000000 ---p 00000000 00:00 0
7fedd0000000-7fedd007e000 rw-p 00000000 00:00 0
7fedd007e000-7fedd4000000 ---p 00000000 00:00 0
7fedd8000000-7fedd8078000 rw-p 00000000 00:00 0
7fedd8078000-7feddc000000 ---p 00000000 00:00 0
7fede0000000-7fede0087000 rw-p 00000000 00:00 0
7fede0087000-7fede4000000 ---p 00000000 00:00 0
7fede4000000-7fede4070000 rw-p 00000000 00:00 0
7fede4070000-7fede8000000 ---p 00000000 00:00 0
7fede8000000-7fede806f000 rw-p 00000000 00:00 0
7fede806f000-7fedec000000 ---p 00000000 00:00 0
7fedec000000-7fedeca52000 rw-p 00000000 00:00 0
7fedeca52000-7fedf0000000 ---p 00000000 00:00 0
7fedf0000000-7fedf0a5a000 rw-p 00000000 00:00 0
7fedf0a5a000-7fedf4000000 ---p 00000000 00:00 0
7fedf8000000-7fedf8a57000 rw-p 00000000 00:00 0
7fedf8a57000-7fedfc000000 ---p 00000000 00:00 0
7fedfc000000-7fedfc087000 rw-p 00000000 00:00 0
7fedfc087000-7fee00000000 ---p 00000000 00:00 0
7fee00000000-7fee00a52000 rw-p 00000000 00:00 0
7fee00a52000-7fee04000000 ---p 00000000 00:00 0
7fee04000000-7fee04100000 rw-p 00000000 00:00 0
7fee04100000-7fee08000000 ---p 00000000 00:00 0
7fee08000000-7fee0803a000 rw-p 00000000 00:00 0
7fee0803a000-7fee0c000000 ---p 00000000 00:00 0
7fee0c000000-7fee0c074000 rw-p 00000000 00:00 0
7fee0c074000-7fee10000000 ---p 00000000 00:00 0
7fee10000000-7fee10041000 rw-p 00000000 00:00 0
7fee10041000-7fee14000000 ---p 00000000 00:00 0
7fee14000000-7fee1403b000 rw-p 00000000 00:00 0
7fee1403b000-7fee18000000 ---p 00000000 00:00 0
7fee18000000-7fee18c00000 rw-p 00000000 00:00 0
7fee18c00000-7fee1c000000 ---p 00000000 00:00 0
7fee1c000000-7fee1c03a000 rw-p 00000000 00:00 0
7fee1c03a000-7fee20000000 ---p 00000000 00:00 0
7fee20000000-7fee2003a000 rw-p 00000000 00:00 0
7fee2003a000-7fee24000000 ---p 00000000 00:00 0
7fee24000000-7fee24044000 rw-p 00000000 00:00 0
7fee24044000-7fee28000000 ---p 00000000 00:00 0
7fee28000000-7fee2803a000 rw-p 00000000 00:00 0
7fee2803a000-7fee2c000000 ---p 00000000 00:00 0
7fee2c000000-7fee2c03a000 rw-p 00000000 00:00 0
7fee2c03a000-7fee30000000 ---p 00000000 00:00 0
7fee30000000-7fee30044000 rw-p 00000000 00:00 0
7fee30044000-7fee34000000 ---p 00000000 00:00 0
7fee34000000-7fee3403d000 rw-p 00000000 00:00 0
7fee3403d000-7fee38000000 ---p 00000000 00:00 0
7fee38000000-7fee38040000 rw-p 00000000 00:00 0
7fee38040000-7fee3c000000 ---p 00000000 00:00 0
7fee3c000000-7fee3c04b000 rw-p 00000000 00:00 0
7fee3c04b000-7fee40000000 ---p 00000000 00:00 0
7fee40000000-7fee40047000 rw-p 00000000 00:00 0
7fee40047000-7fee44000000 ---p 00000000 00:00 0
7fee44000000-7fee44040000 rw-p 00000000 00:00 0
7fee44040000-7fee48000000 ---p 00000000 00:00 0
7fee48000000-7fee48747000 rw-p 00000000 00:00 0
7fee48747000-7fee4c000000 ---p 00000000 00:00 0
7fee4c000000-7fee4c1a0000 rw-p 00000000 00:00 0
7fee4c1a0000-7fee50000000 ---p 00000000 00:00 0
7fee50000000-7fee5003a000 rw-p 00000000 00:00 0
7fee5003a000-7fee54000000 ---p 00000000 00:00 0
7fee54000000-7fee5403a000 rw-p 00000000 00:00 0
7fee5403a000-7fee58000000 ---p 00000000 00:00 0
7fee58000000-7fee58021000 rw-p 00000000 00:00 0
7fee58021000-7fee5c000000 ---p 00000000 00:00 0
7fee5c000000-7fee5c021000 rw-p 00000000 00:00 0
7fee5c021000-7fee60000000 ---p 00000000 00:00 0
7fee60000000-7fee60021000 rw-p 00000000 00:00 0
7fee60021000-7fee64000000 ---p 00000000 00:00 0
7fee64000000-7fee643d2000 rw-p 00000000 00:00 0
7fee64543000-7fee64544000 ---p 00000000 00:00 0
7fee64544000-7fee65116000 rw-p 00000000 00:00 0
7fee6526d000-7fee6526e000 ---p 00000000 00:00 0
7fee6526e000-7fee652b5000 rw-p 00000000 00:00 0
7fee652b5000-7fee652b6000 ---p 00000000 00:00 0
7fee652b6000-7fee652fd000 rw-p 00000000 00:00 0
7fee652fd000-7fee652fe000 ---p 00000000 00:00 0
7fee652fe000-7fee65345000 rw-p 00000000 00:00 0
7fee65345000-7fee65346000 ---p 00000000 00:00 0
7fee65346000-7fee6538d000 rw-p 00000000 00:00 0
7fee6538d000-7fee6538e000 ---p 00000000 00:00 0
7fee6538e000-7fee653d5000 rw-p 00000000 00:00 0
7fee653d5000-7fee653d6000 ---p 00000000 00:00 0
7fee653d6000-7fee6541d000 rw-p 00000000 00:00 0
7fee6541d000-7fee6541e000 ---p 00000000 00:00 0
7fee6541e000-7fee65465000 rw-p 00000000 00:00 0
7fee65465000-7fee65466000 ---p 00000000 00:00 0
7fee65466000-7fee654ad000 rw-p 00000000 00:00 0
7fee654ad000-7fee654ae000 ---p 00000000 00:00 0
7fee654ae000-7fee654f5000 rw-p 00000000 00:00 0
7fee654f5000-7fee654f6000 ---p 00000000 00:00 0
7fee654f6000-7fee6553d000 rw-p 00000000 00:00 0
7fee6553d000-7fee6553e000 ---p 00000000 00:00 0
7fee6553e000-7fee65d3e000 rw-p 00000000 00:00 0
7fee65d3e000-7fee65d3f000 ---p 00000000 00:00 0
7fee65d3f000-7fee6653f000 rw-p 00000000 00:00 0
7fee6653f000-7fee66540000 ---p 00000000 00:00 0
7fee66540000-7fee66d40000 rw-p 00000000 00:00 0
7fee66d40000-7fee66d41000 ---p 00000000 00:00 0
7fee66d41000-7fee67541000 rw-p 00000000 00:00 0
7fee67541000-7fee67542000 ---p 00000000 00:00 0
7fee67542000-7fee67d42000 rw-p 00000000 00:00 0
7fee67d42000-7fee67d43000 ---p 00000000 00:00 0
7fee67d43000-7fee68543000 rw-p 00000000 00:00 0
7fee68543000-7fee68544000 ---p 00000000 00:00 0
7fee68544000-7fee68d44000 rw-p 00000000 00:00 0
7fee68d44000-7fee68d45000 ---p 00000000 00:00 0
7fee68d45000-7fee69545000 rw-p 00000000 00:00 0
7fee69545000-7fee69546000 ---p 00000000 00:00 0
7fee69546000-7fee69d46000 rw-p 00000000 00:00 0
7fee69d46000-7fee69d47000 ---p 00000000 00:00 0
7fee69d47000-7fee6a547000 rw-p 00000000 00:00 0
7fee6a547000-7fee6a548000 ---p 00000000 00:00 0
7fee6a548000-7fee6ad48000 rw-p 00000000 00:00 0
7fee6ad48000-7fee6ad49000 ---p 00000000 00:00 0
7fee6ad49000-7fee6b549000 rw-p 00000000 00:00 0
7fee6b549000-7fee6b54a000 ---p 00000000 00:00 0
7fee6b54a000-7fee6bd4a000 rw-p 00000000 00:00 0
7fee6bd4a000-7fee6bd4b000 ---p 00000000 00:00 0
7fee6bd4b000-7fee6c54b000 rw-p 00000000 00:00 0
7fee6c54b000-7fee6c54c000 ---p 00000000 00:00 0
7fee6c54c000-7fee6cd4c000 rw-p 00000000 00:00 0
7fee6cd4c000-7fee6cd4d000 ---p 00000000 00:00 0
7fee6cd4d000-7fee6d54d000 rw-p 00000000 00:00 0
7fee6d54d000-7fee6d54e000 ---p 00000000 00:00 0
7fee6d54e000-7fee78000000 rw-p 00000000 00:00 0
7fee78000000-7fee783df000 rw-p 00000000 00:00 0
7fee783df000-7fee7c000000 ---p 00000000 00:00 0
7fee7c019000-7fee7c01a000 ---p 00000000 00:00 0
7fee7c01a000-7fee7c061000 rw-p 00000000 00:00 0
7fee7c061000-7fee7c062000 ---p 00000000 00:00 0
7fee7c062000-7fee7c0a9000 rw-p 00000000 00:00 0
7fee7c0a9000-7fee7c0aa000 ---p 00000000 00:00 0
7fee7c0aa000-7fee7c0f1000 rw-p 00000000 00:00 0
7fee7c0f1000-7fee7c0f2000 ---p 00000000 00:00 0
7fee7c0f2000-7fee7c139000 rw-p 00000000 00:00 0
7fee7c139000-7fee7c13a000 ---p 00000000 00:00 0
7fee7c13a000-7fee7c181000 rw-p 00000000 00:00 0
7fee7c181000-7fee7c182000 ---p 00000000 00:00 0
7fee7c182000-7fee7c1c9000 rw-p 00000000 00:00 0
7fee7c1c9000-7fee7c1ca000 ---p 00000000 00:00 0
7fee7c1ca000-7fee7c211000 rw-p 00000000 00:00 0
7fee7c211000-7fee7c212000 ---p 00000000 00:00 0
7fee7c212000-7fee7c259000 rw-p 00000000 00:00 0
7fee7c259000-7fee7c25a000 ---p 00000000 00:00 0
7fee7c25a000-7fee7c2a1000 rw-p 00000000 00:00 0
7fee7c2a1000-7fee7c2a2000 ---p 00000000 00:00 0
7fee7c2a2000-7fee7c2e9000 rw-p 00000000 00:00 0
7fee7c2e9000-7fee7c2ea000 ---p 00000000 00:00 0
7fee7c2ea000-7fee7c331000 rw-p 00000000 00:00 0
7fee7c331000-7fee7c332000 ---p 00000000 00:00 0
7fee7c332000-7fee7c379000 rw-p 00000000 00:00 0
7fee7c379000-7fee7c37a000 ---p 00000000 00:00 0
7fee7c37a000-7fee7c3c1000 rw-p 00000000 00:00 0
7fee7c3c1000-7fee7c3c2000 ---p 00000000 00:00 0
7fee7c3c2000-7fee7c409000 rw-p 00000000 00:00 0
7fee7c409000-7fee7c40a000 ---p 00000000 00:00 0
7fee7c40a000-7fee7c451000 rw-p 00000000 00:00 0
7fee7c451000-7fee7c452000 ---p 00000000 00:00 0
7fee7c452000-7fee7c499000 rw-p 00000000 00:00 0
7fee7c499000-7fee7c49a000 ---p 00000000 00:00 0
7fee7c49a000-7fee7c4e1000 rw-p 00000000 00:00 0
7fee7c4e1000-7fee7c4e2000 ---p 00000000 00:00 0
7fee7c4e2000-7fee7c529000 rw-p 00000000 00:00 0
7fee7c529000-7fee7c52a000 ---p 00000000 00:00 0
7fee7c52a000-7fee7c571000 rw-p 00000000 00:00 0
7fee7c571000-7fee7c572000 ---p 00000000 00:00 0
7fee7c572000-7fee7c5b9000 rw-p 00000000 00:00 0
7fee7c5b9000-7fee7c5ba000 ---p 00000000 00:00 0
7fee7c5ba000-7fee7c601000 rw-p 00000000 00:00 0
7fee7c601000-7fee7c602000 ---p 00000000 00:00 0
7fee7c602000-7fee7c649000 rw-p 00000000 00:00 0
7fee7c649000-7fee7c64a000 ---p 00000000 00:00 0
7fee7c64a000-7fee7c691000 rw-p 00000000 00:00 0
7fee7c691000-7fee7c692000 ---p 00000000 00:00 0
7fee7c692000-7fee7c6d9000 rw-p 00000000 00:00 0
7fee7c6d9000-7fee7c6da000 ---p 00000000 00:00 0
7fee7c6da000-7fee7c721000 rw-p 00000000 00:00 0
7fee7c721000-7fee7c722000 ---p 00000000 00:00 0
7fee7c722000-7fee7c769000 rw-p 00000000 00:00 0
7fee7c769000-7fee7c76a000 ---p 00000000 00:00 0
7fee7c76a000-7fee7c7b1000 rw-p 00000000 00:00 0
7fee7c7b1000-7fee7c7b2000 ---p 00000000 00:00 0
7fee7c7b2000-7fee7c7f9000 rw-p 00000000 00:00 0
7fee7c7f9000-7fee7c7fa000 ---p 00000000 00:00 0
7fee7c7fa000-7fee7cffa000 rw-p 00000000 00:00 0
7fee7cffa000-7fee7cffb000 ---p 00000000 00:00 0
7fee7cffb000-7fee7d7fb000 rw-p 00000000 00:00 0
7fee7d7fb000-7fee7d7fc000 ---p 00000000 00:00 0
7fee7d7fc000-7fee7dffc000 rw-p 00000000 00:00 0
7fee7dffc000-7fee7dffd000 ---p 00000000 00:00 0
7fee7dffd000-7fee7e7fd000 rw-p 00000000 00:00 0
7fee7e7fd000-7fee7e7fe000 ---p 00000000 00:00 0
7fee7e7fe000-7fee7effe000 rw-p 00000000 00:00 0
7fee7effe000-7fee7efff000 ---p 00000000 00:00 0
7fee7efff000-7fee7f7ff000 rw-p 00000000 00:00 0
7fee7f7ff000-7fee7f800000 ---p 00000000 00:00 0
7fee7f800000-7fee80000000 rw-p 00000000 00:00 0
7fee80000000-7fee82dda000 rw-p 00000000 00:00 0
7fee82dda000-7fee84000000 ---p 00000000 00:00 0
7fee84006000-7fee84007000 ---p 00000000 00:00 0
7fee84007000-7fee8404e000 rw-p 00000000 00:00 0
7fee8404e000-7fee8404f000 ---p 00000000 00:00 0
7fee8404f000-7fee84096000 rw-p 00000000 00:00 0
7fee84096000-7fee84097000 ---p 00000000 00:00 0
7fee84097000-7fee840de000 rw-p 00000000 00:00 0
7fee840de000-7fee840df000 ---p 00000000 00:00 0
7fee840df000-7fee84126000 rw-p 00000000 00:00 0
7fee84126000-7fee84127000 ---p 00000000 00:00 0
7fee84127000-7fee8416e000 rw-p 00000000 00:00 0
7fee8416e000-7fee8416f000 ---p 00000000 00:00 0
7fee8416f000-7fee841b6000 rw-p 00000000 00:00 0
7fee841b6000-7fee841b7000 ---p 00000000 00:00 0
7fee841b7000-7fee841fe000 rw-p 00000000 00:00 0
7fee841fe000-7fee841ff000 ---p 00000000 00:00 0
7fee841ff000-7fee84246000 rw-p 00000000 00:00 0
7fee84246000-7fee84247000 ---p 00000000 00:00 0
7fee84247000-7fee8428e000 rw-p 00000000 00:00 0
7fee8428e000-7fee8428f000 ---p 00000000 00:00 0
7fee8428f000-7fee842d6000 rw-p 00000000 00:00 0
7fee842d6000-7fee842d7000 ---p 00000000 00:00 0
7fee842d7000-7fee8431e000 rw-p 00000000 00:00 0
7fee8431e000-7fee8431f000 ---p 00000000 00:00 0
7fee8431f000-7fee84366000 rw-p 00000000 00:00 0
7fee84366000-7fee84367000 ---p 00000000 00:00 0
7fee84367000-7fee843cf000 rw-p 00000000 00:00 0
7fee843cf000-7fee843d0000 ---p 00000000 00:00 0
7fee843d0000-7fee84bd0000 rw-p 00000000 00:00 0
7fee84bd0000-7fee84bd1000 ---p 00000000 00:00 0
7fee84bd1000-7fee853d1000 rw-p 00000000 00:00 0
7fee853d1000-7fee853d2000 ---p 00000000 00:00 0
7fee853d2000-7fee861d6000 rw-p 00000000 00:00 0
7fee861d8000-7fee861d9000 rw-p 00000000 00:00 0
7fee861d9000-7fee861da000 r-xp 00000000 00:00 0
7fee861da000-7fee861db000 rw-p 00000000 00:00 0
7fee861db000-7fee861dc000 r-xp 00000000 00:00 0
7fee861dc000-7fee861dd000 rw-p 00000000 00:00 0
7fee861dd000-7fee861de000 r-xp 00000000 00:00 0
7fee861de000-7fee861df000 rw-p 00000000 00:00 0
7fee861df000-7fee861e0000 r-xp 00000000 00:00 0
7fee861e0000-7fee861e1000 rw-p 00000000 00:00 0
7fee861e1000-7fee861e2000 r-xp 00000000 00:00 0
7fee861e2000-7fee861e3000 rw-p 00000000 00:00 0
7fee861e3000-7fee861e4000 r-xp 00000000 00:00 0
7fee861e4000-7fee861e5000 ---p 00000000 00:00 0
7fee861e5000-7fee8622b000 rw-p 00000000 00:00 0
7fee8622b000-7fee8622c000 ---p 00000000 00:00 0
7fee8622c000-7fee86272000 rw-p 00000000 00:00 0
7fee86272000-7fee86273000 ---p 00000000 00:00 0
7fee86273000-7fee862ba000 rw-p 00000000 00:00 0
7fee862ba000-7fee862bb000 ---p 00000000 00:00 0
7fee862bb000-7fee86302000 rw-p 00000000 00:00 0
7fee86302000-7fee86303000 ---p 00000000 00:00 0
7fee86303000-7fee86349000 rw-p 00000000 00:00 0
7fee86349000-7fee8634a000 ---p 00000000 00:00 0
7fee8634a000-7fee86390000 rw-p 00000000 00:00 0
7fee86390000-7fee86391000 ---p 00000000 00:00 0
7fee86391000-7fee8663b000 rw-p 00000000 00:00 0
7fee8663b000-7fee8663c000 ---p 00000000 00:00 0
7fee8663c000-7fee86e3c000 rw-p 00000000 00:00 0
7fee86e3c000-7fee86e41000 rw-s 00000000 00:11 2973619                    /[aio] (deleted)
7fee86e41000-7fee86e46000 rw-s 00000000 00:11 2973618                    /[aio] (deleted)
7fee86e46000-7fee86e4b000 rw-s 00000000 00:11 2973617                    /[aio] (deleted)
7fee86e4b000-7fee86e50000 rw-s 00000000 00:11 2973616                    /[aio] (deleted)
7fee86e50000-7fee8791c000 rw-p 00000000 00:00 0
7fee8791c000-7fee8791d000 ---p 00000000 00:00 0
7fee8791d000-7fee8891e000 rw-p 00000000 00:00 0
7fee8891e000-7fee88925000 r-xp 00000000 fc:01 1039289                    /root/opt/mysql/8.0.25/lib/plugin/component_reference_cache.so
7fee88925000-7fee88b24000 ---p 00007000 fc:01 1039289                    /root/opt/mysql/8.0.25/lib/plugin/component_reference_cache.so
7fee88b24000-7fee88b25000 r--p 00006000 fc:01 1039289                    /root/opt/mysql/8.0.25/lib/plugin/component_reference_cache.so
7fee88b25000-7fee88b26000 rw-p 00007000 fc:01 1039289                    /root/opt/mysql/8.0.25/lib/plugin/component_reference_cache.so
7fee88b26000-7fee88b31000 r-xp 00000000 fc:01 10625                      /lib/x86_64-linux-gnu/libnss_files-2.27.so
7fee88b31000-7fee88d30000 ---p 0000b000 fc:01 10625                      /lib/x86_64-linux-gnu/libnss_files-2.27.so
7fee88d30000-7fee88d31000 r--p 0000a000 fc:01 10625                      /lib/x86_64-linux-gnu/libnss_files-2.27.so
7fee88d31000-7fee88d32000 rw-p 0000b000 fc:01 10625                      /lib/x86_64-linux-gnu/libnss_files-2.27.so
7fee88d32000-7fee88d38000 rw-p 00000000 00:00 0
7fee88d38000-7fee88d4f000 r-xp 00000000 fc:01 10622                      /lib/x86_64-linux-gnu/libnsl-2.27.so
7fee88d4f000-7fee88f4e000 ---p 00017000 fc:01 10622                      /lib/x86_64-linux-gnu/libnsl-2.27.so
7fee88f4e000-7fee88f4f000 r--p 00016000 fc:01 10622                      /lib/x86_64-linux-gnu/libnsl-2.27.so
7fee88f4f000-7fee88f50000 rw-p 00017000 fc:01 10622                      /lib/x86_64-linux-gnu/libnsl-2.27.so
7fee88f50000-7fee88f52000 rw-p 00000000 00:00 0
7fee88f52000-7fee88f5d000 r-xp 00000000 fc:01 10629                      /lib/x86_64-linux-gnu/libnss_nis-2.27.so
7fee88f5d000-7fee8915c000 ---p 0000b000 fc:01 10629                      /lib/x86_64-linux-gnu/libnss_nis-2.27.so
7fee8915c000-7fee8915d000 r--p 0000a000 fc:01 10629                      /lib/x86_64-linux-gnu/libnss_nis-2.27.so
7fee8915d000-7fee8915e000 rw-p 0000b000 fc:01 10629                      /lib/x86_64-linux-gnu/libnss_nis-2.27.so
7fee8915e000-7fee89166000 r-xp 00000000 fc:01 10623                      /lib/x86_64-linux-gnu/libnss_compat-2.27.so
7fee89166000-7fee89366000 ---p 00008000 fc:01 10623                      /lib/x86_64-linux-gnu/libnss_compat-2.27.so
7fee89366000-7fee89367000 r--p 00008000 fc:01 10623                      /lib/x86_64-linux-gnu/libnss_compat-2.27.so
7fee89367000-7fee89368000 rw-p 00009000 fc:01 10623                      /lib/x86_64-linux-gnu/libnss_compat-2.27.so
7fee89368000-7fee945ef000 rw-p 00000000 00:00 0
7fee945ef000-7fee947d6000 r-xp 00000000 fc:01 10614                      /lib/x86_64-linux-gnu/libc-2.27.so
7fee947d6000-7fee949d6000 ---p 001e7000 fc:01 10614                      /lib/x86_64-linux-gnu/libc-2.27.so
7fee949d6000-7fee949da000 r--p 001e7000 fc:01 10614                      /lib/x86_64-linux-gnu/libc-2.27.so
7fee949da000-7fee949dc000 rw-p 001eb000 fc:01 10614                      /lib/x86_64-linux-gnu/libc-2.27.so
7fee949dc000-7fee949e0000 rw-p 00000000 00:00 0
7fee949e0000-7fee949f7000 r-xp 00000000 fc:01 6530                       /lib/x86_64-linux-gnu/libgcc_s.so.1
7fee949f7000-7fee94bf6000 ---p 00017000 fc:01 6530                       /lib/x86_64-linux-gnu/libgcc_s.so.1
7fee94bf6000-7fee94bf7000 r--p 00016000 fc:01 6530                       /lib/x86_64-linux-gnu/libgcc_s.so.1
7fee94bf7000-7fee94bf8000 rw-p 00017000 fc:01 6530                       /lib/x86_64-linux-gnu/libgcc_s.so.1
7fee94bf8000-7fee94d95000 r-xp 00000000 fc:01 10619                      /lib/x86_64-linux-gnu/libm-2.27.so
7fee94d95000-7fee94f94000 ---p 0019d000 fc:01 10619                      /lib/x86_64-linux-gnu/libm-2.27.so
7fee94f94000-7fee94f95000 r--p 0019c000 fc:01 10619                      /lib/x86_64-linux-gnu/libm-2.27.so
7fee94f95000-7fee94f96000 rw-p 0019d000 fc:01 10619                      /lib/x86_64-linux-gnu/libm-2.27.so
7fee94f96000-7fee9510f000 r-xp 00000000 fc:01 8625                       /usr/lib/x86_64-linux-gnu/libstdc++.so.6.0.25
7fee9510f000-7fee9530f000 ---p 00179000 fc:01 8625                       /usr/lib/x86_64-linux-gnu/libstdc++.so.6.0.25
7fee9530f000-7fee95319000 r--p 00179000 fc:01 8625                       /usr/lib/x86_64-linux-gnu/libstdc++.so.6.0.25
7fee95319000-7fee9531b000 rw-p 00183000 fc:01 8625                       /usr/lib/x86_64-linux-gnu/libstdc++.so.6.0.25
7fee9531b000-7fee9531f000 rw-p 00000000 00:00 0
7fee9531f000-7fee95329000 r-xp 00000000 fc:01 7691                       /usr/lib/x86_64-linux-gnu/libnuma.so.1.0.0
7fee95329000-7fee95528000 ---p 0000a000 fc:01 7691                       /usr/lib/x86_64-linux-gnu/libnuma.so.1.0.0
7fee95528000-7fee95529000 r--p 00009000 fc:01 7691                       /usr/lib/x86_64-linux-gnu/libnuma.so.1.0.0
7fee95529000-7fee9552a000 rw-p 0000a000 fc:01 7691                       /usr/lib/x86_64-linux-gnu/libnuma.so.1.0.0
7fee9552a000-7fee9552b000 r-xp 00000000 fc:01 10979                      /lib/x86_64-linux-gnu/libaio.so.1.0.1
7fee9552b000-7fee9572a000 ---p 00001000 fc:01 10979                      /lib/x86_64-linux-gnu/libaio.so.1.0.1
7fee9572a000-7fee9572b000 r--p 00000000 fc:01 10979                      /lib/x86_64-linux-gnu/libaio.so.1.0.1
7fee9572b000-7fee9572c000 rw-p 00000000 00:00 0
7fee9572c000-7fee957ab000 r-xp 00000000 fc:01 1039499                    /root/opt/mysql/8.0.25/lib/private/libssl.so.1.1
7fee957ab000-7fee959aa000 ---p 0007f000 fc:01 1039499                    /root/opt/mysql/8.0.25/lib/private/libssl.so.1.1
7fee959aa000-7fee959b7000 rw-p 0007e000 fc:01 1039499                    /root/opt/mysql/8.0.25/lib/private/libssl.so.1.1
7fee959b7000-7fee959bc000 rw-p 000a2000 fc:01 1039499                    /root/opt/mysql/8.0.25/lib/private/libssl.so.1.1
7fee959bc000-7fee95c27000 r-xp 00000000 fc:01 1039474                    /root/opt/mysql/8.0.25/lib/private/libcrypto.so.1.1
7fee95c27000-7fee95e26000 ---p 0026b000 fc:01 1039474                    /root/opt/mysql/8.0.25/lib/private/libcrypto.so.1.1
7fee95e26000-7fee95e54000 rw-p 0026a000 fc:01 1039474                    /root/opt/mysql/8.0.25/lib/private/libcrypto.so.1.1
7fee95e54000-7fee95e58000 rw-p 00000000 00:00 0
7fee95e58000-7fee95e6e000 rw-p 002f0000 fc:01 1039474                    /root/opt/mysql/8.0.25/lib/private/libcrypto.so.1.1
7fee95e6e000-7fee95e75000 r-xp 00000000 fc:01 10635                      /lib/x86_64-linux-gnu/librt-2.27.so
7fee95e75000-7fee96074000 ---p 00007000 fc:01 10635                      /lib/x86_64-linux-gnu/librt-2.27.so
7fee96074000-7fee96075000 r--p 00006000 fc:01 10635                      /lib/x86_64-linux-gnu/librt-2.27.so
7fee96075000-7fee96076000 rw-p 00007000 fc:01 10635                      /lib/x86_64-linux-gnu/librt-2.27.so
7fee96076000-7fee96079000 r-xp 00000000 fc:01 10617                      /lib/x86_64-linux-gnu/libdl-2.27.so
7fee96079000-7fee96278000 ---p 00003000 fc:01 10617                      /lib/x86_64-linux-gnu/libdl-2.27.so
7fee96278000-7fee96279000 r--p 00002000 fc:01 10617                      /lib/x86_64-linux-gnu/libdl-2.27.so
7fee96279000-7fee9627a000 rw-p 00003000 fc:01 10617                      /lib/x86_64-linux-gnu/libdl-2.27.so
7fee9627a000-7fee96283000 r-xp 00000000 fc:01 10616                      /lib/x86_64-linux-gnu/libcrypt-2.27.so
7fee96283000-7fee96482000 ---p 00009000 fc:01 10616                      /lib/x86_64-linux-gnu/libcrypt-2.27.so
7fee96482000-7fee96483000 r--p 00008000 fc:01 10616                      /lib/x86_64-linux-gnu/libcrypt-2.27.so
7fee96483000-7fee96484000 rw-p 00009000 fc:01 10616                      /lib/x86_64-linux-gnu/libcrypt-2.27.so
7fee96484000-7fee964b2000 rw-p 00000000 00:00 0
7fee964b2000-7fee96540000 r-xp 00000000 fc:01 1039493                    /root/opt/mysql/8.0.25/lib/private/libprotobuf-lite.so.3.11.4
7fee96540000-7fee9673f000 ---p 0008e000 fc:01 1039493                    /root/opt/mysql/8.0.25/lib/private/libprotobuf-lite.so.3.11.4
7fee9673f000-7fee96741000 r--p 0008d000 fc:01 1039493                    /root/opt/mysql/8.0.25/lib/private/libprotobuf-lite.so.3.11.4
7fee96741000-7fee96742000 rw-p 0008f000 fc:01 1039493                    /root/opt/mysql/8.0.25/lib/private/libprotobuf-lite.so.3.11.4
7fee96742000-7fee96743000 rw-p 00000000 00:00 0
7fee96743000-7fee9675d000 r-xp 00000000 fc:01 10632                      /lib/x86_64-linux-gnu/libpthread-2.27.so
7fee9675d000-7fee9695c000 ---p 0001a000 fc:01 10632                      /lib/x86_64-linux-gnu/libpthread-2.27.so
7fee9695c000-7fee9695d000 r--p 00019000 fc:01 10632                      /lib/x86_64-linux-gnu/libpthread-2.27.so
7fee9695d000-7fee9695e000 rw-p 0001a000 fc:01 10632                      /lib/x86_64-linux-gnu/libpthread-2.27.so
7fee9695e000-7fee96962000 rw-p 00000000 00:00 0
7fee96962000-7fee9698b000 r-xp 00000000 fc:01 2681                       /lib/x86_64-linux-gnu/ld-2.27.so
7fee9698b000-7fee96990000 rw-s 00000000 00:11 2973615                    /[aio] (deleted)
7fee96990000-7fee96995000 rw-s 00000000 00:11 2973614                    /[aio] (deleted)
7fee96995000-7fee9699a000 rw-s 00000000 00:11 2973613                    /[aio] (deleted)
7fee9699a000-7fee9699f000 rw-s 00000000 00:11 2973612                    /[aio] (deleted)
7fee9699f000-7fee96b7c000 rw-p 00000000 00:00 0
7fee96b7c000-7fee96b7e000 rw-s 00000000 00:11 2973620                    /[aio] (deleted)
7fee96b7e000-7fee96b83000 rw-s 00000000 00:11 2973611                    /[aio] (deleted)
7fee96b83000-7fee96b88000 rw-s 00000000 00:11 2973610                    /[aio] (deleted)
7fee96b88000-7fee96b89000 rw-s 00000000 00:11 2973609                    /[aio] (deleted)
7fee96b89000-7fee96b8b000 rw-p 00000000 00:00 0
7fee96b8b000-7fee96b8c000 r--p 00029000 fc:01 2681                       /lib/x86_64-linux-gnu/ld-2.27.so
7fee96b8c000-7fee96b8d000 rw-p 0002a000 fc:01 2681                       /lib/x86_64-linux-gnu/ld-2.27.so
7fee96b8d000-7fee96f60000 rw-p 00000000 00:00 0
7ffddd0fb000-7ffdddc71000 rw-p 00000000 00:00 0
7ffdddc71000-7ffdddc92000 rw-p 00000000 00:00 0                          [stack]
7ffdddd31000-7ffdddd34000 r--p 00000000 00:00 0                          [vvar]
7ffdddd34000-7ffdddd36000 r-xp 00000000 00:00 0                          [vdso]
ffffffffff600000-ffffffffff601000 r-xp 00000000 00:00 0                  [vsyscall]
root@ubuntu:~#
``` 

将POKE的地址对应到内存区, 发现POKE涉及如下区域: 

  - 00400000-03874000 r-xp 00000000 fc:01 1039240 /root/opt/mysql/8.0.25/bin/mysqld

  - 03874000-039e5000 r--p 03473000 fc:01 1039240 /root/opt/mysql/8.0.25/bin/mysqld

  - 039e5000-03d6b000 rw-p 035e4000 fc:01 1039240 /root/opt/mysql/8.0.25/bin/mysqld

  - 15224000-17385000 r-xp 00000000 00:00 0 (大量)

  - 7fee861dd000-7fee861de000 r-xp 00000000 00:00 0

  - 7fee945ef000-7fee947d6000 r-xp 00000000 fc:01 10614 /lib/x86_64-linux-gnu/[libc-2.27.so](<http://libc-2.27.so>)

  - 7fee949da000-7fee949dc000 rw-p 001eb000 fc:01 10614 /lib/x86_64-linux-gnu/[libc-2.27.so](<http://libc-2.27.so>)

  - 7fee94f96000-7fee9510f000 r-xp 00000000 fc:01 8625 /usr/lib/x86_64-linux-gnu/libstdc++.so.6.0.25

  - 7fee95319000-7fee9531b000 rw-p 00183000 fc:01 8625 /usr/lib/x86_64-linux-gnu/libstdc++.so.6.0.25

  - 7fee9572c000-7fee957ab000 r-xp 00000000 fc:01 1039499 /root/opt/mysql/8.0.25/lib/private/[libssl.so](<http://libssl.so>).1.1

  - 7fee959aa000-7fee959b7000 rw-p 0007e000 fc:01 1039499 /root/opt/mysql/8.0.25/lib/private/[libssl.so](<http://libssl.so>).1.1

  - 7fee959bc000-7fee95c27000 r-xp 00000000 fc:01 1039474 /root/opt/mysql/8.0.25/lib/private/[libcrypto.so](<http://libcrypto.so>).1.1

  - 7fee96743000-7fee9675d000 r-xp 00000000 fc:01 10632 /lib/x86_64-linux-gnu/[libpthread-2.27.so](<http://libpthread-2.27.so>)

  - 7fee9675d000-7fee9695c000 ---p 0001a000 fc:01 10632 /lib/x86_64-linux-gnu/[libpthread-2.27.so](<http://libpthread-2.27.so>)

  - 7ffdddc71000-7ffdddc92000 rw-p 00000000 00:00 0 [stack]

  - 7ffdddd34000-7ffdddd36000 r-xp 00000000 00:00 0 [vdso]

dyni通过PTRACE_POKEDATA, 向代码区注入了代码

# 注入了什么代码

以 0x23d76a8 地址为例, 在dyni -redo后, 使用gdb观察内存, 否则dyni每次修改的内存位置会不同 (原理未知)

dyni对此区域的变更: 

```
29374 19:43:23 ptrace(PTRACE_PEEKTEXT, 29183, 0x23d76a8, [0x90fffff751e9ffd6]) = 0
29374 19:43:23 ptrace(PTRACE_POKEDATA, 29183, 0x23d76a8, 0x90fffff751e90514) = 0
``` 

通过 gdb 看到地址对应的代码位: 

```
(gdb) b *0x23d76a8
Breakpoint 2 at 0x23d76a8: file ../../../mysql-8.0.25/storage/innobase/buf/buf0flu.cc, line 1925.
``` 

变更后的代码段: 反编译: 
    
    
    0x90fffff751e90514

```
(gdb) disass/r 0x23d76a8
...
   0x00000000023d769e <+4254>:	49 8d bf 28 23 00 00	lea    0x2328(%r15),%rdi
   0x00000000023d76a5 <+4261>:	e8 26 a4 14 05	callq  0x7521ad0 (dyni修改了此处)
   0x00000000023d76aa <+4266>:	e9 51 f7 ff ff	jmpq   0x23d6e00 <buf_flush_do_batch(buf_pool_t*, buf_flush_t, unsigned long, unsigned long, unsigned long*)+2048>
``` 

解析 0x7521ad0: 

```
(gdb) x/20i 0x7521ad0
   0x7521ad0:	push   %rbp
   0x7521ad1:	mov    %rsp,%rbp
   0x7521ad4:	push   %rbx
   0x7521ad5:	mov    %rdi,%rbx
   0x7521ad8:	sub    $0x8,%rsp
   0x7521adc:	mov    0x28(%rdi),%rdi
   0x7521ae0:	test   %rdi,%rdi
   0x7521ae3:	je     0x7521aee
   0x7521ae5:	cmpb   $0x0,(%rdi)
   0x7521ae8:	jne    0x7eb438b
   0x7521aee:	movb   $0x0,(%rbx)
   0x7521af1:	mfence
   0x7521af4:	movzbl 0x1(%rbx),%eax
   0x7521af8:	test   %al,%al
   0x7521afa:	jne    0x7eb4370
   0x7521b00:	add    $0x8,%rsp
   0x7521b04:	pop    %rbx
   0x7521b05:	pop    %rbp
   0x7521b06:	retq
   0x7521b07:	add    %al,(%rax)
``` 

源代码段:

0x90fffff751e9ffd6

```
   0x00000000023d769e <+4254>:	49 8d bf 28 23 00 00	lea    0x2328(%r15),%rdi
   0x00000000023d76a5 <+4261>:	e8 d6 a1 d6 ff	callq  0x2141880 <PolicyMutex<TTASEventMutex<GenericPolicy> >::exit()>
   0x00000000023d76aa <+4266>:	e9 51 f7 ff ff	jmpq   0x23d6e00 <buf_flush_do_batch(buf_pool_t*, buf_flush_t, unsigned long, unsigned long, unsigned long*)+2048>
``` 

解析0x2750800:

```
(gdb) x/20i 0x2141880
   0x2141880 <PolicyMutex<TTASEventMutex<GenericPolicy> >::exit()>:	push   %rbp
   0x2141881 <PolicyMutex<TTASEventMutex<GenericPolicy> >::exit()+1>:	mov    %rsp,%rbp
   0x2141884 <PolicyMutex<TTASEventMutex<GenericPolicy> >::exit()+4>:	push   %rbx
   0x2141885 <PolicyMutex<TTASEventMutex<GenericPolicy> >::exit()+5>:	mov    %rdi,%rbx
   0x2141888 <PolicyMutex<TTASEventMutex<GenericPolicy> >::exit()+8>:	sub    $0x8,%rsp
   0x214188c <PolicyMutex<TTASEventMutex<GenericPolicy> >::exit()+12>:	mov    0x28(%rdi),%rdi
   0x2141890 <PolicyMutex<TTASEventMutex<GenericPolicy> >::exit()+16>:	test   %rdi,%rdi
   0x2141893 <PolicyMutex<TTASEventMutex<GenericPolicy> >::exit()+19>:	je     0x214189a <PolicyMutex<TTASEventMutex<GenericPolicy> >::exit()+26>
   0x2141895 <PolicyMutex<TTASEventMutex<GenericPolicy> >::exit()+21>:	cmpb   $0x0,(%rdi)
   0x2141898 <PolicyMutex<TTASEventMutex<GenericPolicy> >::exit()+24>:	jne    0x21418b0 <PolicyMutex<TTASEventMutex<GenericPolicy> >::exit()+48>
   0x214189a <PolicyMutex<TTASEventMutex<GenericPolicy> >::exit()+26>:	movb   $0x0,(%rbx)
   0x214189d <PolicyMutex<TTASEventMutex<GenericPolicy> >::exit()+29>:	mfence
   0x21418a0 <PolicyMutex<TTASEventMutex<GenericPolicy> >::exit()+32>:	movzbl 0x1(%rbx),%eax
   0x21418a4 <PolicyMutex<TTASEventMutex<GenericPolicy> >::exit()+36>:	test   %al,%al
   0x21418a6 <PolicyMutex<TTASEventMutex<GenericPolicy> >::exit()+38>:	jne    0x21418c0 <PolicyMutex<TTASEventMutex<GenericPolicy> >::exit()+64>
   0x21418a8 <PolicyMutex<TTASEventMutex<GenericPolicy> >::exit()+40>:	add    $0x8,%rsp
   0x21418ac <PolicyMutex<TTASEventMutex<GenericPolicy> >::exit()+44>:	pop    %rbx
   0x21418ad <PolicyMutex<TTASEventMutex<GenericPolicy> >::exit()+45>:	pop    %rbp
   0x21418ae <PolicyMutex<TTASEventMutex<GenericPolicy> >::exit()+46>:	retq
   0x21418af <PolicyMutex<TTASEventMutex<GenericPolicy> >::exit()+47>:	nop
``` 

修改前后代码一致??

# 探索dyni修改代码的逻辑

## 尝试-1

找到strace中最早修改mysqld代码段的地址: 

```
> less /tmp/dyni-redo.strace  | grep POKEDATA | awk '{print $5}' | grep -v 0x7 | less
 
0x239a500,
0x2397110,
0x23ada90,
0x23ada98,
0x235d960,
0x23ba1c0,
0x225f6e0,
0x225eca0,
0x223b830,
0x2242140,
...
``` 

strace相关记录: 

```
# 读取 0x235d960 内容
26763 09:09:50 ptrace(PTRACE_PEEKTEXT, 25654, 0x235d960, [0x89495741e5894855]) = 0
 
# 将 0x235d960 地址计入 0x9193080
26763 09:09:50 ptrace(PTRACE_POKEDATA, 25654, 0x9193080, 0x235d960) = 0
 
# 将 0x5 写入 0x9193080 ??
26763 09:09:50 ptrace(PTRACE_POKEDATA, 25654, 0x9193088, 0x5) = 0
# 将 0x9193090 的原内容 写入 0x9193080
26763 09:09:50 ptrace(PTRACE_POKEDATA, 25654, 0x9193090, 0x89495741e5894855) = 0
 
# ?
26763 09:09:50 ptrace(PTRACE_POKEDATA, 25654, 0x9208370, 0x9b830d0) = 0
 
# 将 0x235d960 地址计入 0x9208378
26763 09:09:50 ptrace(PTRACE_POKEDATA, 25654, 0x9208378, 0x235d960) = 0
 
# 更新 0x235d960 的内容
26763 09:09:50 ptrace(PTRACE_PEEKTEXT, 25654, 0x235d960, [0x89495741e5894855]) = 0
26763 09:09:50 ptrace(PTRACE_POKEDATA, 25654, 0x235d960, 0x8949570782576be9) = 0
 
...
 
# 登记 0x94d30d0
26763 09:09:55 ptrace(PTRACE_POKEDATA, 25654, 0x920bd80, 0x94d30d0) = 0
26763 09:09:55 ptrace(PTRACE_POKEDATA, 25654, 0x920bd88, 0x235d960) = 0
 
# 再次更新 0x235d960 的内容, 将0x235d960跳转到0x920bd88
26763 09:09:55 ptrace(PTRACE_PEEKTEXT, 25654, 0x235d960, [0x8949570782576be9]) = 0
26763 09:09:55 ptrace(PTRACE_POKEDATA, 25654, 0x235d960, 0x8949570717576be9) = 0
 
# 下一段被替换的代码
26763 09:09:55 ptrace(PTRACE_POKEDATA, 25654, 0x920bd90, 0x94d3340) = 0
26763 09:09:55 ptrace(PTRACE_POKEDATA, 25654, 0x920bd98, 0x223c040) = 0
26763 09:09:55 ptrace(PTRACE_PEEKTEXT, 25654, 0x223c040, [0x894957079574bbe9]) = 0
``` 
    
    
    0x9193080, 像是一个变更函数的登记表
    
    
    [0x94d30d0 - 0x94d3340) 段的内容 (长度为0x270 = ), 应是 0x235d960 被替换的内容
    
    
    0x235d960 原内容: 

```
(gdb) x/156x 0x235d960
0x235d960 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)>:	0xe5894855	0x89495741	0x415641ff	0x53544155
0x235d970 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+16>:	0x28ec8348	0x20468b48	0xfd4fb60f	0x015040f6
0x235d980 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+32>:	0x00ca840f	0xe1830000	0x01f98007	0x040e840f
0x235d990 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+48>:	0xc9840000	0x02d6840f	0xf9800000	0xad860f03
0x235d9a0 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+64>:	0x49000002	0xffffc3c7	0xb70fffff	0x8d4d3c46
0x235d9b0 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+80>:	0xc931fa6f	0x0001ba41	0x894d0000	0x25ff31ee
0x235d9c0 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+96>:	0x000003ff	0xc107c083	0x984803f8	0x31c62949
0x235d9d0 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+112>:	0x0f3aebc0	0x0000441f	0x10408b45	0x0ce8c141
0x235d9e0 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+128>:	0xe0814166	0x840f03ff	0x0000021c	0xc0b70f45
0x235d9f0 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+144>:	0x49c7014c	0x894cf889	0x4818c244	0x4818c183
0x235da00 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+160>:	0x4801c083	0x0f08423b	0x0001c383	0xd8394c00
0x235da10 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+176>:	0x01e2840f	0x8b4c0000	0x01495046	0x088b4dc8
0x235da20 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+192>:	0x0941f641	0x45b17501	0x840fd284	0x00000400
0x235da30 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+208>:	0x4dd3894c	0x0f45d201	0x490065b6	0x9874dc85
0x235da40 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+224>:	0x0000b841	0x09498000	0x0fabebf8	0x0000441f
0x235da50 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+240>:	0xfc47b60f	0xc1f9b60f	0xf80908e0	0x35e0c148
0x235da60 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+256>:	0x36e8c148	0x0f01e183	0x0000c385	0x448d4800
0x235da70 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+272>:	0x8d4d0600	0xc931f847	0xffffba49	0xffff3fff
0x235da80 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+288>:	0x8948ffff	0xbb411042	0xffffffff	0x0000b941
0x235da90 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+304>:	0x47eb8000	0x00401f0f	0x00b70f41	0x08c0c166
0x235daa0 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+320>:	0xf6c0b70f	0x067480c4	0x4c7fe480	0xc4f6c809
0x235dab0 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+336>:	0x48117440	0x00104a81	0x80400000	0x0d48bfe4
0x235dac0 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+352>:	0x40000000	0xca448948	0xe8834918	0xc1834802
0x235dad0 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+368>:	0x4a3b4801	0x08830f08	0x80000001	0x79003d7e
0x235dae0 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+384>:	0xb60f41b7	0x0f41fc7f	0xc1fd5fb6	0xdf0908e7
0x235daf0 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+400>:	0x35e7c148	0x36efc148	0x72f93948	0x5e8b489b
0x235db00 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+416>:	0x3c8d4850	0xd0214c49	0xfb3c8d48	0x483f8b48
0x235db10 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+432>:	0x85483f8b	0x4c0a74ff	0x0f105f39	0x00032f84
0x235db20 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+448>:	0x000d4800	0xe9200000	0xffffff77	0x00401f0f
0x235db30 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+464>:	0x06c08348	0xb949c931	0x7fffffff	0xffffffff
0x235db40 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+480>:	0xffffba41	0x8948ffff	0xb8411042	0x80000000
0x235db50 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+496>:	0x0f662aeb	0x0000441f	0x48c88948	0x0f41d8f7
0x235db60 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+512>:	0xf90744b6	0x057480a8	0x094c7f24	0x448948c0
0x235db70 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+528>:	0x834818ca	0x3b4801c1	0x6773084a	0x003d7e80
0x235db80 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+544>:	0x0f41d679	0x45fc7fb6	0xfd5fb60f	0x4408e7c1
0x235db90 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+560>:	0xc148df09	0xc14835e7	0x394836ef	0x4cb972f9
0x235dba0 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+576>:	0x4850668b	0x4c493c8d	0x8d49c821	0x8b48fc3c
0x235dbb0 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+592>:	0x3f8b483f	0x74ff8548	0x57394c0a	0x85840f10
0x235dbc0 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+608>:	0x48000002	0x0000000d	0x0f99eb20	0x0000441f
(gdb)
``` 
    
    
    0x235d960 修改后的内容: 

```
(gdb) x/20i 0x235d960
   0x235d960 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)>:
    jmpq   0x94d30d0
   0x235d965 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+5>:
    push   %rdi
   0x235d966 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+6>:
    mov    %rdi,%r15
   0x235d969 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+9>:
    push   %r14
   0x235d96b <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+11>:
    push   %r13
   0x235d96d <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+13>:
    push   %r12
   0x235d96f <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+15>:
    push   %rbx
   0x235d970 <rec_init_offsets(unsigned char const*, dict_index_t const*, unsigned long*)+16>:
    sub    $0x28,%rsp
``` 
    
    
    0x94d5c50的内容: 

```
(gdb) x/156x 0x94d30d0
0x94d30d0:	0xe5894855	0x89495741	0x415641ff	0x53544155
0x94d30e0:	0x28ec8348	0x20468b48	0xfd4fb60f	0x015040f6
0x94d30f0:	0xa139840f	0xe1830098	0x01f98007	0x013e840f
0x94d3100:	0xc9840000	0xa03b850f	0x4e8b0098	0x5f8d4838
0x94d3110:	0x4e8b4cfa	0x5d894850	0x0ae9c1c8	0x66c88941
0x94d3120:	0xffe08141	0x3d7e8003	0x58880f00	0x0f00989f
0x94d3130:	0x443a4eb7	0xc166c789	0x816604e9	0x0f03ffe1
0x94d3140:	0x8b4cc1b7	0xba41c845	0x00000001	0xc083f631
0x94d3150:	0x41c93107	0xffffffbe	0x7d894cff	0x03f8c1b8
0x94d3160:	0xc045c748	0x00000000	0x29499848	0x4cc031c0
0x94d3170:	0x4f401c8d	0x4dd91c8d	0x39662b8b	0xe6830ff9
0x94d3180:	0x4100989e	0x010945f6	0x9e9f840f	0x8b450098
0x94d3190:	0xc141105b	0x41660ceb	0x03ffe381	0x0f455274
0x94d31a0:	0x014cdbb7	0xf38949de	0x4c01c183	0x18c25c89
0x94d31b0:	0x48c1b70f	0x7208423b	0x7d8b4cb6	0xc08349b8
0x94d31c0:	0xf8894c01	0x48c0294c	0x48c0450b	0x1fe8ba0f
0x94d31d0:	0x10428948	0x28c48348	0x415c415b	0x415e415d
0x94d31e0:	0x0fc35d5f	0x6600401f	0x00841f0f	0x00000000
0x94d31f0:	0x18b60f41	0x7d814166	0x4d00ff0d	0x49ff608d
0x94d3200:	0x1c77db89	0x6db60f45	0x7d8d450c	0xe78141f2
0x94d3210:	0x000000fd	0x80410a74	0x0d7505fd	0x00401f0f
0x94d3220:	0x0fdb8445	0x989dc788	0xde014800	0x49e0894d
0x94d3230:	0x71e9f389	0x66ffffff	0x00841f0f	0x00000000
0x94d3240:	0x3046b70f	0x0001bb41	0xc1660000	0x40a806e8
0x94d3250:	0x01a81775	0x00c6850f	0x8b440000	0xc141385e
0x94d3260:	0x81410aeb	0x0003ffe3	0x46b70f00	0x6f8d4d3c
0x94d3270:	0x41c931fa	0x000001ba	0xee894d00	0xff25ff31
0x94d3280:	0x83000003	0xf8c107c0	0x49984803	0xc031c629
0x94d3290:	0x0fd8394c	0x00007784	0x468b4c00	0xc8014950
0x94d32a0:	0x41088b4d	0x010941f6	0x9f29840f	0x8b450098
0x94d32b0:	0xc1411040	0x41660ce8	0x03ffe081	0x9e98840f
0x94d32c0:	0x0f450098	0x014cc0b7	0xf88949c7	0xc244894c
0x94d32d0:	0xc1834818	0xc0834818	0x423b4801	0x49b17208
0x94d32e0:	0x4c01c683	0x294cff89	0xba0f48f7	0x89481fef
0x94d32f0:	0x8348107a	0x415b28c4	0x415d415c	0x5d5f415e
0x94d3300:	0x1f0f66c3	0x66000044	0x00841f0f	0x00000000
0x94d3310:	0x04c78348	0xebf88949	0x801f0fb3	0x00000000
0x94d3320:	0x345e8b44	0x0debc141	0xffe38141	0xe9000003
0x94d3330:	0xffffff35	0x00000000	0x00000000	0x00000000
``` 

比较 0x94d5c50 的内容与0x235d960的变更前的内容

![image2021-10-24 14:9:31.png](/assets/01KJBYF3A6NVY01BBK3M77MSBX/image2021-10-24%2014%3A9%3A31.png)

  - 红色部分更换了跳转地址
  - 蓝色部分不一致
    - 探索 0x235dc70  
  
![image2021-10-24 16:56:12.png](/assets/01KJBYF3A6NVY01BBK3M77MSBX/image2021-10-24%2016%3A56%3A12.png)
    - 蓝色部分与0x235dc70的代码一致, 相当于在此处展开了0x235dc70的代码  

      - <https://dynimize.com/blog/discussions/dynimize-the-big-picture/> dynimize的设计文档中说明了: 其使用了inline expansion

# 相关论文

  - <https://hal.inria.fr/hal-01597880/document> : Dynamic Function Specialization
  - [Robust Practical Binary Optimization ](<https://github.com/caps-tum/dbrew>)at Run-time using LLVM: <https://llvm-hpc-2020-workshop.github.io/presentations/llvmhpc2020-engelke.pdf>
    - <https://github.com/caps-tum/dbrew>
      - <https://github.com/aengelke/binopt/blob/master/rewriter/dbll/dbll.cc#L318:28> 是用LLVM搭建的optimizer, 需要研究其输入和输出, 是否为二进制代码, 是否可以跨进程使用
  - 调查LLVM的lifting特性: 
    - <https://adalogics.com/blog/binary-to-llvm-comparison>
  - 整体原理论文: <https://gaps.heig-vd.ch/public/diplome/rapports.php?id=4199>
    - [附件: Elisei Lucas.pdf]
