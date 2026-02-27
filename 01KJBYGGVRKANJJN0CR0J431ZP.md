---
title: 20220130 - 尝试DynamoRIO
confluence_page_id: 1736768
created_at: 2022-01-30T15:01:46+00:00
updated_at: 2022-02-05T17:17:39+00:00
---

# 执行

下载 DynamoRIO-Linux-9.0.0

尝试执行: 

```
time ./bin64/drrun -c samples/bin64/libcbr.so -- /opt/test-main/a.out
``` 

# DynamoRIO的使用介绍

  - <https://www.researchgate.net/profile/Derek-Bruening/publication/4010247_An_infrastructure_for_adaptive_dynamic_optimization/links/00b7d53039701163ee000000/An-infrastructure-for-adaptive-dynamic-optimization.pdf?origin=publication_detail>

# 线索

  - samples/cbr.c
    - CBr: Conditional Branch
    - 在CBR处, 插入诊断代码, 打印分支命中状况, 然后删除诊断代码
    - 即: 只在第一次执行后, 就可放弃追踪
  - samples/inline.c
    - 将inline的指令聚成trace ??
  - inc2add.c
    - The sample [inc2add.c](<https://github.com/DynamoRIO/dynamorio/tree/master/api/samples/inc2add.c>) performs a dynamic optimization: it converts the "inc" instruction to "add 1" without perturbing the target application's behavior.

  - hot_bbcount.c
    - The sample [hot_bbcount.c](<https://github.com/DynamoRIO/dynamorio/tree/master/api/samples/hot_bbcount.c>) uses the drbbdup extension to count the execution of hot basic blocks which have exceeded a given hit threshold.

    - 演示了bb dup的用法
  - <https://dynamorio.org/page_drwrap.html>
    - Function Wrapping and Replacing Extension

  - TLS: thread-local storage

  - <https://github.com/JonathanSalwan/Triton>
    - dynamic binary analysis framework
  - Dynamic Control-Flow Graph with DynamoRIO

    - <https://tpiazza.me/posts/2016-11-04-dynamorio_cfg.html>  
  

#   
目标1

调转 条件jmp 的两部分 

参考: <https://github.com/toshipiazza/drcfg/blob/cd5d19a19889c61a63b803e49f895ccf3bec1aad/cbr.cpp>

测试代码a: 

```
root@ubuntu:/opt/test-main# cat a.c
#include <stdio.h>

int main() {
	printf("Hello, World!\n");
	int n=0;
	for (int i=0; i<=50000; i++)
		for (int j=0; j<=50000; j++)
			if (i+j <= 99000) {
				n += i + j + 200;
			} else {
				n += i * j + 100;
			}
	printf("%d\n", n);
	return 0;
}
``` 

编译: gcc -o a.out -O0 a.c

反编译: 标志位: 0x64 为100, 0xc8为200

```
(gdb) disas/r main
Dump of assembler code for function main:
   0x000000000000068a <+0>:	55	push   %rbp
   0x000000000000068b <+1>:	48 89 e5	mov    %rsp,%rbp
   0x000000000000068e <+4>:	48 83 ec 10	sub    $0x10,%rsp
   0x0000000000000692 <+8>:	48 8d 3d 0b 01 00 00	lea    0x10b(%rip),%rdi        # 0x7a4
   0x0000000000000699 <+15>:	e8 b2 fe ff ff	call   0x550 <puts@plt>
   0x000000000000069e <+20>:	c7 45 f4 00 00 00 00	movl   $0x0,-0xc(%rbp)
   0x00000000000006a5 <+27>:	c7 45 f8 00 00 00 00	movl   $0x0,-0x8(%rbp)
   0x00000000000006ac <+34>:	eb 48	jmp    0x6f6 <main+108>
   0x00000000000006ae <+36>:	c7 45 fc 00 00 00 00	movl   $0x0,-0x4(%rbp)
   0x00000000000006b5 <+43>:	eb 32	jmp    0x6e9 <main+95>
   0x00000000000006b7 <+45>:	8b 55 f8	mov    -0x8(%rbp),%edx
   0x00000000000006ba <+48>:	8b 45 fc	mov    -0x4(%rbp),%eax
   0x00000000000006bd <+51>:	01 d0	add    %edx,%eax
   0x00000000000006bf <+53>:	3d b8 82 01 00	cmp    $0x182b8,%eax
   0x00000000000006c4 <+58>:	7f 12	jg     0x6d8 <main+78>

   0x00000000000006c6 <+60>:	8b 55 f8	mov    -0x8(%rbp),%edx
   0x00000000000006c9 <+63>:	8b 45 fc	mov    -0x4(%rbp),%eax
   0x00000000000006cc <+66>:	01 d0	add    %edx,%eax
   0x00000000000006ce <+68>:	05 c8 00 00 00	add    $0xc8,%eax
   0x00000000000006d3 <+73>:	01 45 f4	add    %eax,-0xc(%rbp)
   0x00000000000006d6 <+76>:	eb 0d	jmp    0x6e5 <main+91>

   0x00000000000006d8 <+78>:	8b 45 f8	mov    -0x8(%rbp),%eax
   0x00000000000006db <+81>:	0f af 45 fc	imul   -0x4(%rbp),%eax
   0x00000000000006df <+85>:	83 c0 64	add    $0x64,%eax
   0x00000000000006e2 <+88>:	01 45 f4	add    %eax,-0xc(%rbp)

   0x00000000000006e5 <+91>:	83 45 fc 01	addl   $0x1,-0x4(%rbp)
   0x00000000000006e9 <+95>:	81 7d fc 50 c3 00 00	cmpl   $0xc350,-0x4(%rbp)
   0x00000000000006f0 <+102>:	7e c5	jle    0x6b7 <main+45>
   0x00000000000006f2 <+104>:	83 45 f8 01	addl   $0x1,-0x8(%rbp)
   0x00000000000006f6 <+108>:	81 7d f8 50 c3 00 00	cmpl   $0xc350,-0x8(%rbp)
   0x00000000000006fd <+115>:	7e af	jle    0x6ae <main+36>
   0x00000000000006ff <+117>:	8b 45 f4	mov    -0xc(%rbp),%eax
   0x0000000000000702 <+120>:	89 c6	mov    %eax,%esi
   0x0000000000000704 <+122>:	48 8d 3d a7 00 00 00	lea    0xa7(%rip),%rdi        # 0x7b2
   0x000000000000070b <+129>:	b8 00 00 00 00	mov    $0x0,%eax
   0x0000000000000710 <+134>:	e8 4b fe ff ff	call   0x560 <printf@plt>
   0x0000000000000715 <+139>:	b8 00 00 00 00	mov    $0x0,%eax
   0x000000000000071a <+144>:	c9	leave
   0x000000000000071b <+145>:	c3	ret
End of assembler dump.
``` 

性能: 

```
root@ubuntu:/opt/test-main# time ./a.out
Hello, World!
1482935222

real	0m8.017s
user	0m8.009s
sys	0m0.000s
``` 

测试代码b: 

```
root@ubuntu:/opt/test-main# cat b.c
#include <stdio.h>

int main() {
	printf("Hello, World!\n");
	int n=0;
	for (int i=0; i<=50000; i++)
		for (int j=0; j<=50000; j++)
			if (i+j > 99000) {
				n += i * j + 100;
			} else {
				n += i + j + 200;
			}
	printf("%d\n", n);
	return 0;
}
``` 

编译: gcc -o b.out -O0 b.c

反编译: 标志位: 0x64 为100, 0xc8为200

```
(gdb) disas/r main
Dump of assembler code for function main:
   0x000000000000068a <+0>:	55	push   %rbp
   0x000000000000068b <+1>:	48 89 e5	mov    %rsp,%rbp
   0x000000000000068e <+4>:	48 83 ec 10	sub    $0x10,%rsp
   0x0000000000000692 <+8>:	48 8d 3d 0b 01 00 00	lea    0x10b(%rip),%rdi        # 0x7a4
   0x0000000000000699 <+15>:	e8 b2 fe ff ff	call   0x550 <puts@plt>
   0x000000000000069e <+20>:	c7 45 f4 00 00 00 00	movl   $0x0,-0xc(%rbp)
   0x00000000000006a5 <+27>:	c7 45 f8 00 00 00 00	movl   $0x0,-0x8(%rbp)
   0x00000000000006ac <+34>:	eb 48	jmp    0x6f6 <main+108>
   0x00000000000006ae <+36>:	c7 45 fc 00 00 00 00	movl   $0x0,-0x4(%rbp)
   0x00000000000006b5 <+43>:	eb 32	jmp    0x6e9 <main+95>
   0x00000000000006b7 <+45>:	8b 55 f8	mov    -0x8(%rbp),%edx
   0x00000000000006ba <+48>:	8b 45 fc	mov    -0x4(%rbp),%eax
   0x00000000000006bd <+51>:	01 d0	add    %edx,%eax
   0x00000000000006bf <+53>:	3d b8 82 01 00	cmp    $0x182b8,%eax
   0x00000000000006c4 <+58>:	7e 0f	jle    0x6d5 <main+75>

   0x00000000000006c6 <+60>:	8b 45 f8	mov    -0x8(%rbp),%eax
   0x00000000000006c9 <+63>:	0f af 45 fc	imul   -0x4(%rbp),%eax
   0x00000000000006cd <+67>:	83 c0 64	add    $0x64,%eax
   0x00000000000006d0 <+70>:	01 45 f4	add    %eax,-0xc(%rbp)
   0x00000000000006d3 <+73>:	eb 10	jmp    0x6e5 <main+91>

   0x00000000000006d5 <+75>:	8b 55 f8	mov    -0x8(%rbp),%edx
   0x00000000000006d8 <+78>:	8b 45 fc	mov    -0x4(%rbp),%eax
   0x00000000000006db <+81>:	01 d0	add    %edx,%eax
   0x00000000000006dd <+83>:	05 c8 00 00 00	add    $0xc8,%eax
   0x00000000000006e2 <+88>:	01 45 f4	add    %eax,-0xc(%rbp)

   0x00000000000006e5 <+91>:	83 45 fc 01	addl   $0x1,-0x4(%rbp)
   0x00000000000006e9 <+95>:	81 7d fc 50 c3 00 00	cmpl   $0xc350,-0x4(%rbp)
   0x00000000000006f0 <+102>:	7e c5	jle    0x6b7 <main+45>
   0x00000000000006f2 <+104>:	83 45 f8 01	addl   $0x1,-0x8(%rbp)
   0x00000000000006f6 <+108>:	81 7d f8 50 c3 00 00	cmpl   $0xc350,-0x8(%rbp)
   0x00000000000006fd <+115>:	7e af	jle    0x6ae <main+36>
   0x00000000000006ff <+117>:	8b 45 f4	mov    -0xc(%rbp),%eax
   0x0000000000000702 <+120>:	89 c6	mov    %eax,%esi
   0x0000000000000704 <+122>:	48 8d 3d a7 00 00 00	lea    0xa7(%rip),%rdi        # 0x7b2
   0x000000000000070b <+129>:	b8 00 00 00 00	mov    $0x0,%eax
   0x0000000000000710 <+134>:	e8 4b fe ff ff	call   0x560 <printf@plt>
   0x0000000000000715 <+139>:	b8 00 00 00 00	mov    $0x0,%eax
   0x000000000000071a <+144>:	c9	leave
   0x000000000000071b <+145>:	c3	ret
End of assembler dump.
(gdb)
``` 

性能: 

```
root@ubuntu:/opt/test-main# time ./b.out
Hello, World!
1482935222

real	0m7.786s
user	0m7.771s
sys	0m0.000s
``` 

性能差异: 

`(+``58) 行, jg和jle都必须要执行, 而高频分支 (200所在的分支), 在b.out中, 不会涉及jmp, 所以执行会更快`

使用 [cbr.c](/assets/01KJBYGGVRKANJJN0CR0J431ZP/cbr.c) 变更a.out, 执行命令如下: 

```
/opt/DynamoRIO-Linux-9.0.0/bin64/drrun -debug -loglevel 3 -c bin/libcbr.so -- /opt/test-main/a.out
``` 

查看日志, 可发现代码生效的痕迹: 

![image2022-1-31 20:49:8.png](/assets/01KJBYGGVRKANJJN0CR0J431ZP/image2022-1-31%2020%3A49%3A8.png)

对应的代码: 

![image2022-1-31 20:52:36.png](/assets/01KJBYGGVRKANJJN0CR0J431ZP/image2022-1-31%2020%3A52%3A36.png)

尝试进行简单编程: 

![image2022-1-31 21:23:37.png](/assets/01KJBYGGVRKANJJN0CR0J431ZP/image2022-1-31%2021%3A23%3A37.png)

这段代码进行以下代码转换: 

```
jle targ
fall:
targ:
 
转换为: 
jg fall
jmp targ
fall
targ
``` 

以上代码进行了简单的指令替换, 插入

# 目标2

上面使用调换jmp的逻辑, 使用了jmp targ作为指针

下面尝试将jmp的两组bb调换位置: 将jmp后的代码续到当前的block中

问题: 某一个BB, 只能容下 一些ubr(unconditional branch, JMP) 和一个cbr (conditional branch, JG).

如果要完成目标, 需要写成: 

```
BB1:
jg fall
 
BB1.fall_though_bb:
jmp targ
 
BB2:
fall
 
BB3:
targ
``` 

不能确保BB2一定紧跟在BB1后面, 所以 BB1.fall_though_bb的逻辑 生成的 jmp targ 不能少, 这也就意味着转换以后, 并没有性能提升

已向社区提出问题: <https://groups.google.com/g/dynamorio-users/c/jnavZXvfjEk>

推广思考: 

我们想进行运行时优化, 而框架并不能保证BB2一定紧跟BB1, 那么对于cbr的调整, 优化的方向就不是节省跳转, 而是其他方向

可能的尝试: 

TODO: 将代码放在额外的内存中, 不通过BB的机制生成代码

  - 内存分配函数: <https://dynamorio.org/dr__tools_8h.html#a9333e0dee46cc854228efab156ce98c5>

# 尝试Trace

读DynamoRIO的样例: instrace_x86.c

在每个指令前, 增加一段指令, 意为: 

```
    /* The following assembly performs the following instructions
     * buf_ptr->pc = pc;
     * buf_ptr->opcode = opcode;
     * buf_ptr++;
     * if (buf_ptr >= buf_end_ptr)
     *    clean_call();
     */
``` 

将pc和opcode存储在TLS (Thread-Local Storage)中, 在clean_call中写入文件

instrace_simple.c中有一个技巧, 关于内存申请: 

```
    /* The TLS field provided by DR cannot be directly accessed from the code cache.
     * For better performance, we allocate raw TLS so that we can directly
     * access and update it with a single instruction.
     */
    if (!dr_raw_tls_calloc(&tls_seg, &tls_offs, INSTRACE_TLS_COUNT, 0))
        DR_ASSERT(false);
``` 

# 尝试申请独立的内存, 将代码放在其中

可以申请独立的内存, 将代码放入

![image2022-2-2 23:37:22.png](/assets/01KJBYGGVRKANJJN0CR0J431ZP/image2022-2-2%2023%3A37%3A22.png)

这段代码用一个JLE转向独立的内存, 内存中含有JMP 0x0死循环

DynamoRIO, 会将这片内存认作是BB进行解析

完整代码: [cbr.cpp.mem.cpp](/assets/01KJBYGGVRKANJJN0CR0J431ZP/cbr.cpp.mem.cpp)

可以将Fall和Target两个部分逻辑换过来, 要点:

  - 注意每一个BB的跳出目标位置, 需要单独增加JMP, 在BB位置变更时, 补全其跳出目标位置
  - BB位置变更时, 短跳转会变成长跳转, DynamoRIO需要将指令先标记成meta指令, 才能进行地址转换

但性能没有提升, 原因: 

即使用了内存作为代码段, 执行时, 仍会被DynamoRIO解析, 并制作成BB, 每个BB在code cache中会添加JMP出口, 导致指令数没有减少

# TODO: 整理DynamoRIO对于hot code/trace的逻辑

目的: 找到hot code的处理, 是否能包括了优化的可能

```
/* This routine maintains the statistics that identify hot code
 * regions, and it controls the building and installation of trace
 * fragments.
 */
fragment_t *
monitor_cache_enter(dcontext_t *dcontext, fragment_t *f)
``` 

  - <https://en.wikipedia.org/wiki/Granularity> 说明了 coarse-grained 和 fine-grained 的含义: 
    - Coarse-grained materials or systems have fewer, larger discrete components than fine-grained materials or systems.

      - A **coarse-grained** description of a system regards large subcomponents. 粗粒度

      - A **fine-grained** description regards smaller components of which the larger ones are composed. 细粒度

  - 解读 monitor_cache_enter
    - 函数说明中的两个作用: 
      - 维护hot code的统计信息 (maintains the statistics that identify hot code regions)

        - 需探索构成trace的条件
        - 需探索构成hot code的条件
      - 创建trace (it controls the building and installation of trace fragments)

    - 函数逻辑
      - trace building/trace selection mode (md->trace_tag != NULL)
        - 检查 trace end 条件, 其中会融入 custom trace 的逻辑 (由用户自行决定trace)
        - 根据tag找到trace head
          - 按照tag (内存地址) 从 trace cache中查找
          - 如果找不到, 再从 bb cache中查找
          - 否则, 创建一个trace head
        - 根据内存是否有空闲, 以及参数max_trace_bbs, 再次计算trace end条件
        - 如果 trace end
          - end_and_emit_trace
            - 其中有 optimize_trace 逻辑, 需解读 ??
              - 参数 -optimize
            - 将 trace fragment 进行emit (emit_fragment)
        - 否则
          - internal_extend_trace
            - 将 新代码BB的指令 加到 trace的指令中
            - 对 新代码中的会访问的地址, 扩展到 vm_area中 (vm_area_add_to_list)
      - (md->trace_tag == NULL)
        - searching for a hot trace head 
          - 判断是否应转换为trace head
            - /* Dynamic marking of trace heads for:  
* - indirect exits  
* - an exit from a trace that ends just before a SYSENTER.  
* - private secondary trace heads targeted by shared traces

            - check_for_trace_head
              - 成为trace head的条件
        - 增加trace head计数: Found a trace head, increment its counter
        - 当计数 > trace_threshold 时
          - internal_extend_trace
    - 猜想  

      - fine to coarse: 从BB构建trace?

# OPTIMIZE_OPTION

Dynamize 自带一些对于trace的优化算法, 搜索OPTIMIZE_OPTION可看到所有的相关参数

其中包括: 

  - prefetch
  - rlr
  - vectorize
  - unroll_loops
  - instr_counts
  - stack_adjust
  - loads_to_const
  - safe_loads_to_const
  - remove_dead_code
  - constant_prop
  - call_return_matching
  - remove_unnecessary_zeroing
  - peephole

使用上面的测试程序:

![image2022-2-6 0:50:50.png](/assets/01KJBYGGVRKANJJN0CR0J431ZP/image2022-2-6%200%3A50%3A50.png)

各种优化算法中, unrool_loops的效果最好, 加上code cache引入的优化, 总共能提升10%左右

# 测试OPTIMIZE call_return_matching的效果

TODO: 需要一个程序, 测试call_return_matching进行inline的效果

# attach测试

对测试程序可以进行attach, 但对于MySQL会导致进程崩溃: 

```
<Application /root/opt/mysql/8.0.25/bin/mysqld (18491).  DynamoRIO internal crash at PC 0x000000007114bbad.  Please report this at http://dynamorio.org/issues/.  Program aborted.
Received SIGSEGV at pc 0x000000007114bbad in thread 18491
Base: 0x0000000071000000
Registers:eax=0x0000000000000000 ebx=0x00007f6060094990 ecx=0x0000000000000001 edx=0x0000000000000000
        esi=0x00007fff424469b8 edi=0x0000000000000000 esp=0x00007fff42446970 ebp=0x00007f62d1e5e000
        r8 =0x00007fff424469a8 r9 =0x0000000000000000 r10=0x0000000000000000 r11=0x00007f60600949d0
        r12=0x0000000000000001 r13=0x0000000000000000 r14=0x00007fff424469b8 r15=0x00007fff424469b0
        eflags=0x0000000000010297
version 9.0.19027, custom build
-no_dynamic_options -code_api -stack_size 56K -signal_stack_size 32K -max_elide_jmp 0 -max_elide_call 0 -no_inline_ignored_syscalls -native_exec_default_list '' -no_native_exec_managed_code -no_indcall2direct
0x00007f62d1e5e000 0x0000000000000000>
``` 

# JIT optimize

TODO: 

  - <http://dl.acm.org/citation.cfm?id=2738610>
  - <https://github.com/DynamoRIO/dynamorio/blob/dcbaca81b9816a40fddc2b0211475a7c5a9220a3/api/docs/jitopt.dox>

# 接下来的方向

  - 考虑针对hot trace进行优化
    - <http://groups.csail.mit.edu/commit/papers/03/garnett-meng-thesis.pdf>
      - Call Return Matching Optimization
        - It looks for methods that have been completely inlined into a trace such that both the call instruction and the return instruction from the method are in the trace
        - 查找call和return都在trace中的函数, 可以进行inline
    - trace -> IR -> bb
    - ![image](https://dynimize.com/images/underTheHood.svg)
  - 解开CALL
  - 找到另一个变更指令的框架工具

# 杂项

<https://pdfs.semanticscholar.org/351a/23cb045924e90e86d13793ec161c10253f75.pdf>

  - ![image2022-2-2 1:6:28.png](/assets/01KJBYGGVRKANJJN0CR0J431ZP/image2022-2-2%201%3A6%3A28.png)
  - ![image2022-2-2 1:6:59.png](/assets/01KJBYGGVRKANJJN0CR0J431ZP/image2022-2-2%201%3A6%3A59.png)
  - DynamoRIO曾经是个optimization框架, 后来好像是放弃了
