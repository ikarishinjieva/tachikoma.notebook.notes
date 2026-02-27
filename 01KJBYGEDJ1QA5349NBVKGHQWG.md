---
title: 20220129 - HDTrans在32位虚拟机上的使用
confluence_page_id: 1736749
created_at: 2022-01-29T14:33:50+00:00
updated_at: 2022-01-30T14:58:21+00:00
---

环境 root@10.186.17.110

使用aqemu架设虚拟机

也可以使用命令行启动虚拟机: 

```
/usr/bin/qemu-system-x86_64 \
    -cpu qemu32 \
    -M pc \
    -machine accel=tcg \
    -m 2048 \
    -hda /root/.aqemu/Linux_2.6_HDA.img \
    -boot once=c,menu=off \
    -device virtio-net-pci,netdev=usernet \
    -netdev user,id=usernet,hostname=vm1 \
    -rtc base=localtime \
    -name "Linux 2.6" \
    -vnc :2 -daemonize
``` 

开启虚拟机的VNC端口

使用ssh tunnel, 将10.186.17.110上的虚拟机VNC端口5902映射出来: 

```
ssh -L 5902:localhost:5902 root@10.186.17.110
``` 

使用VNC viewer 连接localhost:5902

虚拟机用户名密码: root/root

IP为10.0.2.15

从虚拟机架设反向ssh通道: 

```
ssh -p 22 -qngfNTR 6766:localhost:22 root@10.186.17.110
``` 

在10.186.17.110 的sshd配置中, 开启GatewayPorts, 这样反向端口6766绑定于0.0.0.0

可以通过如下命令连接到虚拟机: 

```
ssh -p 6766 root@10.186.17.110
``` 

# 遇到isupper函数panic

panic信息: 

```
root@ubuntu:/opt/HDTrans-0.4-1# LD_PRELOAD=/opt/HDTrans-0.4-1/nvdebug.so /opt/test-main/a.out
VDebug rules!...
Initial eip   = b6fdcddb

Looking up b6fdcddb Success
bb# 0, 0xb6fdcddb:
VDebug is handling SIGSEGV...
eip: b6e3dfee
Fault code = 14
If(Pagefault), cr2 = c2
BBCache is:
0xb7037c08:	Segmentation fault (core dumped)
 
崩溃位置在isupper函数
``` 

原因未知, 将isupper换成ascII大小判断: 

![image2022-1-30 0:12:47.png](/assets/01KJBYGEDJ1QA5349NBVKGHQWG/image2022-1-30%200%3A12%3A47.png)

可正确跑通

# 未见性能明显改善

使用HDTrans, 比起不使用, overhead不高, 但未见性能改善

使用测试程序: 

```
#include <stdio.h>

int sum(int i,int j) {
return i+j;
}

int main() {
   // printf() displays the string inside quotation
   printf("Hello, World!");
   int i=0;
   for (int j=0; j<=10000; j++)
   for (int n=0; n<=50000; n++) i = sum(i,j);
   printf("%d\n", i);
   return 0;
}
``` 

用gcc -O0 b.c编译

使用HDTrans, 调整DEBUG级别为ALL (debug.h中调整常量), 可得到调试日志: 

[debug.log](/assets/01KJBYGEDJ1QA5349NBVKGHQWG/debug.log)

解析debug.log, 断点设置在 lookup_bb_eip:

  1. 0xb7fea41f 断点位: 

```
(gdb) bt
#0  0xb7fea41f in call_init (l=0xb7fd61b0, argc=argc@entry=1, argv=0xbffff6b4, env=0xbffff6bc)
    at dl-init.c:58
#1  0xb7fea609 in call_init (env=0xbffff6bc, argv=0xbffff6b4, argc=1, l=<optimized out>)
    at dl-init.c:104
#2  _dl_init (main_map=0xb7fff918, argc=1, argv=0xbffff6b4, env=0xbffff6bc) at dl-init.c:87
#3  0xb7fdba5f in _dl_start_user () from /lib/ld-linux.so.2
``` 
  2. Initial eip步骤输出

```
VDebug rules!...
Initial eip   = b7818dbb
 
Looking up b7818dbb Success
bb# 0, 0xb7818dbb:	83 c4 08              addl     $0x8,%esp
Tanslation Path - inline_emit_normal
0: Normal b7818dbb addL!
0xb7873cfa:	83 c4 08              addl     $0x8,%esp

bb# 0, 0xb7818dbe:	5b                    popl     %ebx
Tanslation Path - inline_emit_normal
0: Normal b7818dbe popL!
0xb7873cfd:	5b                    popl     %ebx

bb# 0, 0xb7818dbf:	c3                    retl
Tanslation Path - NOT inline_emit_normal
0: Ret
0xb7873cfe:	ff 25 f4 3e c5 b7     jmpl     0xb7c53ef4
``` 
  3. 查询 0xb7fea421 的代码, 是 0xb7fea41f 返回后的代码

```
Looking up b7fea421 Failed
Entering New translation $#COLD$# on b7fea421::b7873d1f
bb# 1, 0xb7fea421:	8b 85 84 00 00 00     movl     0x84(%ebp),%eax
Tanslation Path - inline_emit_normal
0: Normal b7fea421 movL!
0xb7873d1f:	8b 85 84 00 00 00     movl     0x84(%ebp),%eax

bb# 1, 0xb7fea427:	83 c4 10              addl     $0x10,%esp
Tanslation Path - inline_emit_normal
0: Normal b7fea427 addL!
0xb7873d25:	83 c4 10              addl     $0x10,%esp

bb# 1, 0xb7fea42a:	85 c0                 testl    %eax,%eax
Tanslation Path - inline_emit_normal
0: Normal b7fea42a testL!
0xb7873d28:	85 c0                 testl    %eax,%eax

bb# 1, 0xb7fea42c:	74 43                 je       b7fea471
Tanslation Path - NOT inline_emit_normal
0: Jcond
```
```
Looking up b7fea42e Failed
0xb7873d2a:	0f 84 00 00 00 00     je       b7873d30

bb# 2, 0xb7fea42e:	8b 95 8c 00 00 00     movl     0x8c(%ebp),%edx
Tanslation Path - inline_emit_normal
0: Normal b7fea42e movL!
0xb7873d30:	8b 95 8c 00 00 00     movl     0x8c(%ebp),%edx

bb# 2, 0xb7fea434:	8b 6d 00              movl     0x0(%ebp),%ebp
Tanslation Path - inline_emit_normal
0: Normal b7fea434 movL!
0xb7873d36:	8b 6d 00              movl     0x0(%ebp),%ebp

bb# 2, 0xb7fea437:	03 68 04              addl     0x4(%eax),%ebp
Tanslation Path - inline_emit_normal
0: Normal b7fea437 addL!
0xb7873d39:	03 68 04              addl     0x4(%eax),%ebp

bb# 2, 0xb7fea43a:	8b 7a 04              movl     0x4(%edx),%edi
Tanslation Path - inline_emit_normal
0: Normal b7fea43a movL!
0xb7873d3c:	8b 7a 04              movl     0x4(%edx),%edi

bb# 2, 0xb7fea43d:	c1 ef 02              shrl     $0x2,%edi
Tanslation Path - inline_emit_normal
0: Normal b7fea43d shrL!
0xb7873d3f:	c1 ef 02              shrl     $0x2,%edi

bb# 2, 0xb7fea440:	85 ff                 testl    %edi,%edi
Tanslation Path - inline_emit_normal
0: Normal b7fea440 testL!
0xb7873d42:	85 ff                 testl    %edi,%edi

bb# 2, 0xb7fea442:	89 7c 24 0c           movl     %edi,0xc(%esp,,1)
Tanslation Path - inline_emit_normal
0: Normal b7fea442 movL!
0xb7873d44:	89 7c 24 0c           movl     %edi,0xc(%esp,,1)

bb# 2, 0xb7fea446:	74 29                 je       b7fea471
Tanslation Path - NOT inline_emit_normal
0: Jcond
``` 

HDTrans以Basic Block为单位, 一段一段"翻译"二进制码  
  

         
         "Entering New translation $#COLD$# on b7fea421::b7873d1f" 的意思是: 将0xb7fea421的内容翻译到0xb7873d1f的空间上

结论: HDTrans 的定位是提供了一个低消耗的翻译机制, 不包括运行优化, 运行优化得额外增加代码处理

# 翻译过程

举一个典型的翻译堆栈: 

```
(gdb) bt
#0  bb_emit_byte (M=0xb7833bc0 <theMachine>, c=255 '\377') at emit-inline.c:47
#1  0xb781cb01 in emit_ret (M=0xb7833bc0 <theMachine>, d=0xbffff4dc) at emit.c:2002
#2  0xb781dd94 in translate_instr (M=0xb7833bc0 <theMachine>, ds=0xbffff4dc) at xlcore.c:833
#3  0xb781e0d5 in xlate_bb (M=0xb7833bc0 <theMachine>) at xlcore.c:1101
#4  0xb781d41f in xlate_for_sieve (M=0xb7833bc0 <theMachine>) at xlcore.c:147
#5  0xb7873cdc in theMachine () from /opt/HDTrans-0.4-1/nvdebug.so
Backtrace stopped: previous frame inner to this frame (corrupt stack?)
``` 

命令调用哪个emit函数, 是由 nopbyte0表 和相关表 进行声明的: 

![image2022-1-30 20:33:30.png](/assets/01KJBYGEDJ1QA5349NBVKGHQWG/image2022-1-30%2020%3A33%3A30.png)

# 线索

  - <https://dl101.zlibcdn.com/dtoken/de21b1e01a72ca54f56fc07cbd95357a>
  - <https://github.com/DynamoRIO/dynamorio/blob/master/api/samples/inline.c>[  
](<https://github.com/DynamoRIO/dynamorio/blob/master/api/samples/inline.c>)

下一步: 尝试DynamoRIO
