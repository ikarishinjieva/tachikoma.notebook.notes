---
title: 20211117 - retdec 会导致SIGSEGV的探索
confluence_page_id: 1573074
created_at: 2021-11-17T07:40:43+00:00
updated_at: 2021-11-17T07:40:43+00:00
---

# 背景

尝试 mcsema/reopt 等工具失败后, 重新探索retdec

# 测试

对于 opt4db/test 中的测试程序进行retdec反编译: 

```
/opt/retdec/bin/retdec-decompiler.py --stop-after bin2llvmir -o a.ll a.out
``` 

[a.ll](/assets/01KJBYFBDSR1SNE0X711Z0TGXR/a.ll)

在进行anvill反编译: 

```
使用IDAPro获取 ANVILL spec file
 
anvill-decompile-json-12 --spec program.json --ir_out a.ll.2
``` 

[a.ll.2](/assets/01KJBYFBDSR1SNE0X711Z0TGXR/a.ll.2)

发现两套ll文件没有可比性, anvill生成的ll文件明显更长, 可读性更差
