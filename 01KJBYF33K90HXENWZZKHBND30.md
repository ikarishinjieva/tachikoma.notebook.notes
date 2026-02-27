---
title: 20211028 - retdec 试用
confluence_page_id: 1572920
created_at: 2021-10-28T09:55:01+00:00
updated_at: 2021-10-28T13:24:32+00:00
---

# 目的

探索是否有针对 binary code 的优化技术

retdec的目的: binary code -> LLVM IR -> 优化的binary code

# 第一次试用

日志: [retdec.output.txt](/assets/01KJBYF33K90HXENWZZKHBND30/retdec.output.txt)

各个phase的初步阶段: 

```
Running phase: Initialization ( 0.01s )
Running phase: LLVM ( 0.02s )
Running phase: Providers initialization ( 0.02s )
Running phase: Input binary to LLVM IR decoding ( 0.91s )
Running phase: LLVM ( 5.45s )
Running phase: x86 address spaces optimization ( 6.13s )
Running phase: x87 fpu register analysis ( 6.38s )
Running phase: Main function identification optimization ( 6.82s )
Running phase: Libgcc idioms optimization ( 6.83s )
Running phase: LLVM instruction optimization ( 6.83s )
Running phase: Conditional branch optimization ( 7.09s )
Running phase: Syscalls optimization ( 9.23s )
Running phase: Stack optimization ( 9.23s )
Running phase: Constants optimization ( 15.25s )
Running phase: Function parameters and returns optimization ( 26.98s )
Running phase: LLVM instruction optimization using RDA ( 36.56s )
Running phase: LLVM instruction optimization ( 52.22s )
Running phase: Simple types recovery optimization ( 52.42s )
Running phase: Disassembly generation ( 55.73s )
Running phase: Assembly mapping instruction removal ( 59.08s )
...
``` 

可参考其中从binary生成LLVM IR, 以及后续的优化步骤

生成的mysqlbinlog.bc 是什么文件: 

  - 查日志, 发现bc文件是通过bin2llvmir生成的
  - bin2llvmir 的说明是: "library of LLVM passes for translating binaries into LLVM IR modules.", 生成的是LLVM IR modules
  - <https://llvm.org/docs/BitCodeFormat.html#abstract> : bc文件应当是LLVM IR的存储形式
  - 如何解析: <https://llvm.org/docs/CommandGuide/llvm-bcanalyzer.html>
  - 解析样例: 

```
root@ubuntu:/opt/retdec# llvm-bcanalyzer-11 /root/opt/mysql/8.0.25/bin/mysqlbinlog.bc
Summary of /root/opt/mysql/8.0.25/bin/mysqlbinlog.bc:
         Total size: 29655712b/3706964.00B/926741W
        Stream type: LLVM IR
  # Toplevel Blocks: 4

Per-block Summary:
  Block ID #0 (BLOCKINFO_BLOCK):
      Num Instances: 1
         Total Size: 768b/96.00B/24W
    Percent of file: 0.0026%
      Num SubBlocks: 0
        Num Abbrevs: 18
        Num Records: 3
    Percent Abbrevs: 0.0000%

	Record Histogram:
		  Count    # Bits     b/Rec   % Abv  Record Kind
		      3        60      20.0          SETBID
...
...
```

其生成的mysqlbinlog.c举例: 

![image2021-10-28 17:54:57.png](/assets/01KJBYF33K90HXENWZZKHBND30/image2021-10-28%2017%3A54%3A57.png)
