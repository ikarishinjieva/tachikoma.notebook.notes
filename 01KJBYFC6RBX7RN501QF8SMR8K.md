---
title: 20211118 - HDTrans 使用
confluence_page_id: 1573105
created_at: 2021-11-18T10:19:08+00:00
updated_at: 2021-12-07T14:37:34+00:00
---

# 摘要

下载路径: <https://srl.cs.jhu.edu/projects/HDTrans/HDTrans-0.4-1.tar.gz>

相关论文: <https://www.usenix.org/legacy/events/vee06/full_papers/p175-sridhar.pdf>

仅对32bit有效, 搭建虚拟机进行测试

编译时会碰到少量报错, 对头文件进行调整即可:

  - 修改 signals.h, 将 struct siginfo 改为 siginfo_t, 将 struct ucontext 改为 ucontext_t
  - 修改 emit.c, 将 #include  改为 #include 
  - 修改 xlcore.h, 删除 #include 
  - 修改 xlcore.c, 删除 #include 

运行时会碰到Segment Fault

# 处理SIGSEGV

进行gdb调试: 

```
set exec-wrapper env LD_PRELOAD=/opt/HDTrans-0.4-1/nvdebug.so
``` 

崩溃位置: 

```
(gdb) disas/r 0xb7872cbc,0xb7872ce0
Dump of assembler code from 0xb7872cbc to 0xb7872ce0:
   0xb7872cbc <theMachine+262396>:	0f b7 c9	movzwl %cx,%ecx
   0xb7872cbf <theMachine+262399>:	8d 0c 4d 08 2c 85 b7	lea    -0x487ad3f8(,%ecx,2),%ecx
   0xb7872cc6 <theMachine+262406>:	ff e1	jmp    *%ecx
=> 0xb7872cc8 <theMachine+262408>:	c7 05 1c 40 fd b7 00 02 00 00	movl   $0x200,0xb7fd401c
   0xb7872cd2 <theMachine+262418>:	68 c0 2b 83 b7	push   $0xb7832bc0
   0xb7872cd7 <theMachine+262423>:	e8 b4 f1 f9 ff	call   0xb7811e90 <xlate_for_sieve>
   0xb7872cdc <theMachine+262428>:	8b 44 24 24	mov    0x24(%esp),%eax
``` 

  - opcode + Mod R/M参考: 
    - <http://www.cs.loyola.edu/~binkley/371/Encoding_Real_x86_Instructions.html>
    - <https://datacadamia.com/intel/modrm>
    - <http://www.c-jump.com/CIS77/CPU/x86/lecture.html#X77_0150_encoding_add_edx_displacement> (有图)
  - 关于SI, DI的说明
    - <https://www.tutorialspoint.com/assembly_programming/assembly_registers.htm>
  - 关于LEA的说明
    - <https://en.wikibooks.org/wiki/X86_Assembly/Data_Transfer#Load_Effective_Address>
  - 如何解析 IA32指令的opcode: <https://blog.csdn.net/Apollon_krj/article/details/77524601>
  - 指令到二进制的对照表: <http://www.mathemainzel.info/files/x86asmref.html#mov>
  - online disassembler: 
    - <https://defuse.ca/online-x86-assembler.htm#disassembly2>
    - <https://onlinedisassembler.com/odaweb/>

读源码得知, 崩溃的二进制代码 由 bb_setup_ret_calls_fast_dispatch_bb 或者 类似函数 生成

# 解析指令二进制码举例

研究崩溃点: 

![image2021-12-4 16:36:23.png](/assets/01KJBYFC6RBX7RN501QF8SMR8K/image2021-12-4%2016%3A36%3A23.png)

```
c7
05 
1c 40 fd b7 
00 02 00 00
``` 

查询x86手册: <https://www.intel.com/content/dam/www/public/us/en/documents/manuals/64-ia-32-architectures-software-developer-instruction-set-reference-manual-325383.pdf>

查询MOV的说明: 

第一位为 C7, 其指令形式如下图

![image2021-12-4 16:40:23.png](/assets/01KJBYFC6RBX7RN501QF8SMR8K/image2021-12-4%2016%3A40%3A23.png)

其opcode指示位是 /0, 在下表2-2中, 找到 第二位 05对应的说明: 

其标明: 第一参数 r/m32 指的是: disp32, 为直接地址

![image2021-12-4 16:39:59.png](/assets/01KJBYFC6RBX7RN501QF8SMR8K/image2021-12-4%2016%3A39%3A59.png)

# 对SIGSEGV的解析

更换崩溃段的代码, 换为noop: 

```
#define BORDER_START  do {                              \
    bb_emit_byte(M, 0x90u);     \
    bb_emit_byte(M, 0x90u);     \
    bb_emit_byte(M, 0x90u);     \
  } while(0)
``` 

仍会发生崩溃, 也就是说在UserEntry最后的jmpl后, 第一个命令就会SIGSEGV:

```
UserEntry:
	subl $32,%esp
	pushl 32(%esp)
	add $40,%esp

	pushf
	pusha

	subl 	$4,%esp
	call 	init_translator
	addl	$4,%esp
	
	movl 	(%eax),%eax
	jmpl	*%eax
``` 

测试

  - gdb断点设置为UserEntry, 达到断点后, 开始测试
  - 测试1:

```
(gdb) info proc mapping
process 12971
Mapped address spaces:

	Start Addr   End Addr       Size     Offset objfile
	 0x8048000  0x8049000     0x1000        0x0 /opt/test-main/a.out
	 0x8049000  0x804a000     0x1000        0x0 /opt/test-main/a.out
	 0x804a000  0x804b000     0x1000     0x1000 /opt/test-main/a.out
	0xb7637000 0xb7638000     0x1000        0x0
	0xb7638000 0xb77e8000   0x1b0000        0x0 /lib/i386-linux-gnu/libc-2.23.so
	0xb77e8000 0xb77e9000     0x1000   0x1b0000 /lib/i386-linux-gnu/libc-2.23.so
	0xb77e9000 0xb77eb000     0x2000   0x1b0000 /lib/i386-linux-gnu/libc-2.23.so
	0xb77eb000 0xb77ec000     0x1000   0x1b2000 /lib/i386-linux-gnu/libc-2.23.so
	0xb77ec000 0xb77ef000     0x3000        0x0
	0xb7805000 0xb7824000    0x1f000        0x0 /opt/HDTrans-0.4-1/nvdebug.so
	0xb7824000 0xb782c000     0x8000    0x1e000 /opt/HDTrans-0.4-1/nvdebug.so
	0xb782c000 0xb782d000     0x1000    0x26000 /opt/HDTrans-0.4-1/nvdebug.so
	0xb782d000 0xb7fd6000   0x7a9000        0x0
	0xb7fd6000 0xb7fd9000     0x3000        0x0 [vvar]
	0xb7fd9000 0xb7fdb000     0x2000        0x0 [vdso]
	0xb7fdb000 0xb7ffe000    0x23000        0x0 /lib/i386-linux-gnu/ld-2.23.so
	0xb7ffe000 0xb7fff000     0x1000    0x22000 /lib/i386-linux-gnu/ld-2.23.so
	0xb7fff000 0xb8000000     0x1000    0x23000 /lib/i386-linux-gnu/ld-2.23.so
	0xbffdf000 0xc0000000    0x21000        0x0 [stack]
 
设置目标地址内容为 0x90 (nop), 然后设置断点, 直接用gdb jump过去, ni执行一个指令
 
(gdb) set *(0xb782d000)=0x90
(gdb) b *0xb782d000
Breakpoint 6 at 0xb782d000
(gdb) jump *0xb782d000
Continuing at 0xb782d000.

Breakpoint 6, 0xb782d000 in thePtState () from /opt/HDTrans-0.4-1/nvdebug.so
(gdb) disas/r 0xb782d000,+10
Dump of assembler code from 0xb782d000 to 0xb782d00a:
=> 0xb782d000 <thePtState+3872>:	90	nop
   0xb782d001 <thePtState+3873>:	00 00	add    %al,(%eax)
   0xb782d003 <thePtState+3875>:	00 00	add    %al,(%eax)
   0xb782d005 <thePtState+3877>:	00 00	add    %al,(%eax)
   0xb782d007 <thePtState+3879>:	00 00	add    %al,(%eax)
   0xb782d009 <thePtState+3881>:	00 00	add    %al,(%eax)
End of assembler dump.
(gdb) ni
0xb782d001 in thePtState () from /opt/HDTrans-0.4-1/nvdebug.so
 
发现 0xb782d000 是可以执行的. 
 
换做崩溃点 0xb7872cba, 也可以执行: 
 
(gdb) set *(0xb7872cba)=0x90
(gdb) b *0xb7872cba
Breakpoint 9 at 0xb7872cba
(gdb) jump *0xb7872cba
Continuing at 0xb7872cba.

Breakpoint 9, 0xb7872cba in theMachine () from /opt/HDTrans-0.4-1/nvdebug.so
(gdb) ni
0xb7872cbb in theMachine () from /opt/HDTrans-0.4-1/nvdebug.so
 
 
``` 

  
  

另外的现象: 如果进入UserEntry后, 直接对崩溃点进行断点, 达到断点后, 会发现代码内容与预期的不同: 

```
Breakpoint 1, UserEntry () at UserEntry.s:12
12		subl $32,%esp
(gdb) b *0xb7872cba
Breakpoint 32 at 0xb7872cba
(gdb) c
Continuing.
 
(gdb) disas/r 0xb7872cba,+50
Dump of assembler code from 0xb7872cba to 0xb7872cec:
=> 0xb7872cba <theMachine+262394>:	00 90 90 68 c0 2b	add    %dl,0x2bc06890(%eax)
   0xb7872cc0 <theMachine+262400>:	83 b7 e8 c9 f1 f9 ff	xorl   $0xffffffff,-0x60e3618(%edi)
   0xb7872cc7 <theMachine+262407>:	8b 44 24 24	mov    0x24(%esp),%eax
   0xb7872ccb <theMachine+262411>:	89 04 24	mov    %eax,(%esp)
   0xb7872cce <theMachine+262414>:	8b 05 20 40 fd b7	mov    0xb7fd4020,%eax
   0xb7872cd4 <theMachine+262420>:	89 44 24 24	mov    %eax,0x24(%esp)
   0xb7872cd8 <theMachine+262424>:	c7 05 1c 40 fd b7 00 00 00 00	movl   $0x0,0xb7fd401c
   0xb7872ce2 <theMachine+262434>:	9d	popf
   0xb7872ce3 <theMachine+262435>:	61	popa
   0xb7872ce4 <theMachine+262436>:	c3	ret
   0xb7872ce5 <theMachine+262437>:	00 00	add    %al,(%eax)
   0xb7872ce7 <theMachine+262439>:	00 00	add    %al,(%eax)
   0xb7872ce9 <theMachine+262441>:	00 00	add    %al,(%eax)
   0xb7872ceb <theMachine+262443>:	00 00	add    %al,(%eax)
 
正常的0xb7872cba: 
(gdb) disas/r 0xb7872cba,+50
Dump of assembler code from 0xb7872cba to 0xb7872cec:
   0xb7872cba <theMachine+262394>:	90	nop
   0xb7872cbb <theMachine+262395>:	90	nop
   0xb7872cbc <theMachine+262396>:	90	nop
   0xb7872cbd <theMachine+262397>:	68 c0 2b 83 b7	push   $0xb7832bc0
   0xb7872cc2 <theMachine+262402>:	e8 c9 f1 f9 ff	call   0xb7811e90 <xlate_for_sieve>
   0xb7872cc7 <theMachine+262407>:	8b 44 24 24	mov    0x24(%esp),%eax
   0xb7872ccb <theMachine+262411>:	89 04 24	mov    %eax,(%esp)
   0xb7872cce <theMachine+262414>:	8b 05 20 40 fd b7	mov    0xb7fd4020,%eax
   0xb7872cd4 <theMachine+262420>:	89 44 24 24	mov    %eax,0x24(%esp)
   0xb7872cd8 <theMachine+262424>:	c7 05 1c 40 fd b7 00 00 00 00	movl   $0x0,0xb7fd401c
   0xb7872ce2 <theMachine+262434>:	9d	popf
   0xb7872ce3 <theMachine+262435>:	61	popa
   0xb7872ce4 <theMachine+262436>:	c3	ret
   0xb7872ce5 <theMachine+262437>:	00 00	add    %al,(%eax)
   0xb7872ce7 <theMachine+262439>:	00 00	add    %al,(%eax)
   0xb7872ce9 <theMachine+262441>:	00 00	add    %al,(%eax)
   0xb7872ceb <theMachine+262443>:	00 00	add    %al,(%eax)
End of assembler dump.
 
发现 0xb7872cba 被修改为 0x0 ??
 
怀疑是虚拟机环境问题, 但在腾讯云申请 32bit服务器, 获得的结果一致, 都是内存地址会被改变
 
结论: 经测试, 设置断点处的内存值不可修改
``` 

# 线索

<https://coderedirect.com/questions/286720/x86-memory-access-segmentation-fault>

```
The answer is it isn't. x86-64 doesn't have RIP-relative addressing in 32-bit emulation mode (this should be obvious because RIP doesn't exist in 32-bit). What's happening is that nasm is compiling you some lovely 32-bit opcodes that you're trying to run as 64-bit. GDB is disassembling your 32-bit opcodes as 64-bit, and telling you that in 64-bit, those bytes mean a RIP-relative mov. 64-bit and 32-bit opcodes on the x86-64 overlap a lot to make use of common decoding logic in the silicon, and you're getting confused because the code that GDB is disassembling looks similar to the 32-bit code you wrote, but in reality you're just throwing garbage bytes at the processor.
``` 

使用-fno-pic好像没有作用

# 进展

  - 怀疑是 64位CPU模拟32位指令时的不兼容
  - 使用aQEMU, 更换CPU架构为32bit  
  
![image2021-12-7 22:35:30.png](/assets/01KJBYFC6RBX7RN501QF8SMR8K/image2021-12-7%2022%3A35%3A30.png)
  - 原来报错的位置已不再报错, 出现新位置的报错, 已经走进了xlate_for_sieve逻辑中  
  

# 读代码

bb_setup_startup_slow_dispatch_bb

  - BORDER_START

断点设置

  - UserEntry
