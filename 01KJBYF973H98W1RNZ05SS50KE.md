---
title: 20211109 - opt4db 实现
confluence_page_id: 1573017
created_at: 2021-11-09T14:08:30+00:00
updated_at: 2021-11-18T06:19:09+00:00
---

# 背景

根据 [20211013 - 探索 dynimize 原理] 中的"整体原理论文"附章中的论文, 实现类似的机制, 项目名为 opt4db

# 问题1

有retdec生成的ll (LLVM IR) 文件, 编译后会报 SEGSIGV, 经调查, 认为其IR文件生成的代码有问题

换用reopt生成 ll 文件, 命令如下: 

```
reopt --export-llvm /tmp/a.ll /tmp/a
``` 

可以正确进行

# 问题2

使用reopt, 生成MySQL的ll文件, 发现会有不少函数丢失. reopt输出如下: 

```
以MYSQLparse为例: 
Discovering _Z10MYSQLparseP3THDPP15Parse_tree_root.cold.385(0xd2f603)
  Complete.
Discovering _Z10MYSQLparseP3THDPP15Parse_tree_root(0x1072700)
  Block 0x10728f3: Unclassified control flow transfer.
  Incomplete.
  reopt__Z10MYSQLparseP3THDPP15Parse_tree_root.cold.385_0_0xd2f603(0xd2f603): Could not determine signature at callsite 0xd2f628:
    Unknown arguments to _Unwind_Resume.
``` 

# 问题3

换用 llvm-mctoll

解析mysqlbinlog会报错: 

```
root@ubuntu:/opt# C_INCLUDE_PATH=:/usr/include/:/usr/include/x86_64-linux-gnu/:/usr/lib/gcc/x86_64-linux-gnu/7/include/ CPLUS_INCLUDE_PATH=:/usr/include/c++/7/:/usr/include/x86_64-linux-gnu/c++/7/ /opt/llvm-build/bin/llvm-mctoll --include-files="/opt/1.h,/usr/lib/gcc/x86_64-linux-gnu/7/include/unwind.h,/usr/include/pthread.h,/usr/include/string.h,/usr/include/time.h,/usr/include/sys/stat.h,/usr/include/stdio.h,/usr/include/stdlib.h" -d /root/opt/mysql/8.0.25/bin/mysqlbinlog
llvm-mctoll: /opt/llvm-project/llvm/tools/llvm-mctoll/X86/X86AdditionalInstrInfo.h:80: mctoll::InstructionKind mctoll::getInstructionKind(unsigned int): Assertion `Iter != mctoll::X86AddlInstrInfo.end() && "Unknown opcode"' failed.
 #0 0x0000556f9eaa7524 llvm::sys::PrintStackTrace(llvm::raw_ostream&, int) [clone .localalias.0] /opt/llvm-project/llvm/lib/Support/Unix/Signals.inc:565:0
 #1 0x0000556f9eaa75db PrintStackTraceSignalHandler(void*) /opt/llvm-project/llvm/lib/Support/Unix/Signals.inc:632:0
 #2 0x0000556f9eaa528f llvm::sys::RunSignalHandlers() [clone .localalias.3] /opt/llvm-project/llvm/lib/Support/Signals.cpp:97:0
 #3 0x0000556f9eaa6ea5 SignalHandler(int) /opt/llvm-project/llvm/lib/Support/Unix/Signals.inc:407:0
 #4 0x00007fcca2e3e980 __restore_rt (/lib/x86_64-linux-gnu/libpthread.so.0+0x12980)
 #5 0x00007fcca1cf3fb7 gsignal /build/glibc-S9d2JN/glibc-2.27/signal/../sysdeps/unix/sysv/linux/raise.c:51:0
 #6 0x00007fcca1cf5921 abort /build/glibc-S9d2JN/glibc-2.27/stdlib/abort.c:81:0
 #7 0x00007fcca1ce548a __assert_fail_base /build/glibc-S9d2JN/glibc-2.27/assert/assert.c:89:0
 #8 0x00007fcca1ce5502 (/lib/x86_64-linux-gnu/libc.so.6+0x30502)
 #9 0x0000556f9eaf4bb5 mctoll::getInstructionKind(unsigned int) /opt/llvm-project/llvm/tools/llvm-mctoll/X86/X86AdditionalInstrInfo.h:80:0
#10 0x0000556f9eaf4f6f X86MachineInstructionRaiser::raiseMachineJumpTable() /opt/llvm-project/llvm/tools/llvm-mctoll/X86/X86JumpTables.cpp:43:0
#11 0x0000556f9eb06505 X86MachineInstructionRaiser::getRaisedFunctionPrototype() /opt/llvm-project/llvm/tools/llvm-mctoll/X86/X86FuncPrototypeDiscovery.cpp:192:0
#12 0x0000556f9d7565b5 ModuleRaiser::runMachineFunctionPasses() /opt/llvm-project/llvm/tools/llvm-mctoll/ModuleRaiser.cpp:108:0
#13 0x0000556f9d61f8ef DisassembleObject(llvm::object::ObjectFile const*, bool) /opt/llvm-project/llvm/tools/llvm-mctoll/llvm-mctoll.cpp:1418:0
#14 0x0000556f9d6203b2 DumpObject(llvm::object::ObjectFile*, llvm::object::Archive const*) /opt/llvm-project/llvm/tools/llvm-mctoll/llvm-mctoll.cpp:1488:0
#15 0x0000556f9d6209e5 DumpInput(llvm::StringRef) /opt/llvm-project/llvm/tools/llvm-mctoll/llvm-mctoll.cpp:1548:0
#16 0x0000556f9d632efe void (*std::for_each<__gnu_cxx::__normal_iterator<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >*, std::vector<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >, std::allocator<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > > > >, void (*)(llvm::StringRef)>(__gnu_cxx::__normal_iterator<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >*, std::vector<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >, std::allocator<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > > > >, __gnu_cxx::__normal_iterator<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >*, std::vector<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >, std::allocator<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > > > >, void (*)(llvm::StringRef)))(llvm::StringRef) /usr/include/c++/7/bits/stl_algo.h:3883:0
#17 0x0000556f9d620e63 main /opt/llvm-project/llvm/tools/llvm-mctoll/llvm-mctoll.cpp:1621:0
#18 0x00007fcca1cd6bf7 __libc_start_main /build/glibc-S9d2JN/glibc-2.27/csu/../csu/libc-start.c:344:0
#19 0x0000556f9d619a9a _start (/opt/llvm-build/bin/llvm-mctoll+0x90ca9a)

*** Please submit an issue at https://github.com/microsoft/llvm-mctoll
*** along with a back trace and a reproducer, if possible.
Stack dump:
0.	Program arguments: /opt/llvm-build/bin/llvm-mctoll --include-files=/opt/1.h,/usr/lib/gcc/x86_64-linux-gnu/7/include/unwind.h,/usr/include/pthread.h,/usr/include/string.h,/usr/include/time.h,/usr/include/sys/stat.h,/usr/include/stdio.h,/usr/include/stdlib.h -d /root/opt/mysql/8.0.25/bin/mysqlbinlog
Aborted (core dumped)
root@ubuntu:/opt#

``` 

# 问题4

使用 IDApro + anvill, anvill-decompile-json-12 会报错: 

```
F1117 03:12:48.131911 16914 DeclLifter.cpp:156] Check failed: native_val->getType() == decl.type (0xc870fd0 vs. 0xc870ff0)
*** Check failure stack trace: ***
    @          0x2c0017c  google::LogMessageFatal::~LogMessageFatal()
    @           0x6d2c6f  anvill::StoreNativeValue()
    @           0x6db19d  anvill::FunctionLifter::CallLiftedFunctionFromNativeFunction()
    @           0x6dcae8  anvill::FunctionLifter::LiftFunction()
    @           0x6dce95  anvill::EntityLifter::LiftEntity()
    @           0x6aa22f  std::_Function_handler<>::_M_invoke()
    @           0x6b905b  anvill::Program::ForEachFunction()
    @           0x6a3010  main
    @     0x7f20a180bbf7  __libc_start_main
    @           0x6a221a  _start
    @              (nil)  (unknown)
Aborted (core dumped)
``` 

# 问题5 

使用dagger, 反编译 opt4db/test, 会报错: 

```
root@ubuntu:/opt/dagger/build# ./bin/llvm-dec /opt/opt4db/test/a.out
Cannot translate instruction:
    LEAVE64: <MCInst 1313>
Couldn't translate instruction

UNREACHABLE executed at /opt/dagger/lib/DC/DCTranslator.cpp:144!
#0 0x000055ba273e79f5 llvm::sys::PrintStackTrace(llvm::raw_ostream&) /opt/dagger/lib/Support/Unix/Signals.inc:398:0
#1 0x000055ba273e7a88 PrintStackTraceSignalHandler(void*) /opt/dagger/lib/Support/Unix/Signals.inc:462:0
#2 0x000055ba273e5cbb llvm::sys::RunSignalHandlers() /opt/dagger/lib/Support/Signals.cpp:49:0
#3 0x000055ba273e7261 SignalHandler(int) /opt/dagger/lib/Support/Unix/Signals.inc:252:0
#4 0x00007f4798d27980 __restore_rt (/lib/x86_64-linux-gnu/libpthread.so.0+0x12980)
#5 0x00007f47979d8fb7 gsignal /build/glibc-S9d2JN/glibc-2.27/signal/../sysdeps/unix/sysv/linux/raise.c:51:0
#6 0x00007f47979da921 abort /build/glibc-S9d2JN/glibc-2.27/stdlib/abort.c:81:0
#7 0x000055ba2736edeb bindingsErrorHandler(void*, std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const&, bool) /opt/dagger/lib/Support/ErrorHandling.cpp:127:0
#8 0x000055ba270ac2ab llvm::DCTranslator::translateFunction(llvm::MCFunction const&) /opt/dagger/lib/DC/DCTranslator.cpp:139:0
#9 0x000055ba270b2bb2 llvm::translateRecursivelyAt(llvm::ArrayRef<unsigned long>, llvm::DCTranslator&, llvm::MCModule&, llvm::MCObjectDisassembler*, llvm::MCObjectSymbolizer*) /opt/dagger/lib/DC/DCTranslatorUtils.cpp:79:0
#10 0x000055ba26a5d59d main /opt/dagger/tools/llvm-dec/llvm-dec.cpp:219:0
#11 0x00007f47979bbbf7 __libc_start_main /build/glibc-S9d2JN/glibc-2.27/csu/../csu/libc-start.c:344:0
#12 0x000055ba26a5c2ca _start (/opt/dagger/build/bin/llvm-dec+0x2372ca)
Stack dump:
0.	Program arguments: ./bin/llvm-dec /opt/opt4db/test/a.out
1.	DC: Translating Function at address AFA
2.	DC: Translating Basic Block at address CF2
3.	DC: Translating instruction LEAVE64 at address CF2
Aborted
``` 

# 尝试

用clang编译MySQL

```
wget https://downloads.mysql.com/archives/get/p/23/file/mysql-8.0.25.tar.gz
wget https://udomain.dl.sourceforge.net/project/boost/boost/1.73.0/boost_1_73_0.tar.gz

export CC=/usr/bin/clang-12
export CXX=/usr/bin/clang++-12

cmake -DWITH_NUMA=1 -DDOWNLOAD_BOOST=0 -DWITH_BOOST=/opt/boost_1_73_0 -DCMAKE_BUILD_TYPE=RelWithDebInfo ..

cmake时不指定环境变量
先编译一次
再指定环境变量, 重新编译: make -B --dry-run > make.log, 记录所有命令.

将其中 .o 改成 .ll
cat make.log | egrep '(clang\+?\+?-12|cmake_echo_color)' | sed -e 's/\.c\.o/.c.ll/g' | sed -e 's/\.cc\.o/.cc.ll/g' | sed -e 's/\.cpp\.o/.cpp.ll/g' | sed -e 's/^/cd \/opt\/mysql-8.0.25\/bld \&\& /g' | sed -e 's/ -DNDEBUG / -DNDEBUG -S -emit-llvm /g' > make.2.log
执行改过的语句

变更各个link.txt文件
find . -name link.txt | xargs -L1 bash /opt/replace.link.txt

手工修改/opt/mysql-8.0.25/bld/sql/CMakeFiles/mysqld.dir/link.txt
将 "libmaster.a.ll" 改为 --override=libmaster.a.ll
 
执行cmake中的link过程: 
cat make.log | grep cmake_link_script | bash -x 2>&1 | tee link.log

会有其他报错, 但mysqld.ll可以正常完成
 
link的回滚命令: 
find . -name link.txt | xargs -L1 -I '{}' mv '{}.raw' '{}'
 
``` 

/opt/replace.link.txt内容: 

```
#!/bin/bash
file=$1
set -e
cp $file $file.raw
if grep --quiet '/usr/bin/ar' $file; then
    cat $file | sed -E 's/\/usr\/bin\/ar qc ([^ ]+) (.*)/llvm-link-12 -S -v -o \1.ll \2/g' | sed -e 's/\.c\.o/.c.ll/g' | sed -e 's/\.cc\.o/.cc.ll/g' | sed -e 's/\.cpp\.o/.cpp.ll/g' | sed -e 's/\/usr\/bin\/ranlib .*//g' | awk '{for (i=1;i<=NF;i++) if (!a[$i]++) printf("%s%s",$i,FS)}{printf("\n")}' | awk '{for (i=1;i<=NF;i++) if (!($i ~ /^\//)) printf("%s%s",$i,FS)}{printf("\n")}' > $file.replace
elif grep --quiet '/usr/bin/clang' $file; then
    cat $file | sed -E 's/ \-[^o]([^ ]*)/ /g' | sed -e 's/\.c\.o/.c.ll/g' | sed -e 's/\.cc\.o/.cc.ll/g' | sed -e 's/\.cpp\.o/.cpp.ll/g' | sed -e 's/\.so/.so.ll/g' | sed -e 's/\.a/.a.ll/g' | sed -E 's/^\/usr\/bin\/clang[^ ]+/llvm-link-12 -S -v/g' | awk '{for (i=1;i<=NF;i++) if (!a[$i]++) printf("%s%s",$i,FS)}{printf("\n")}' | awk '{for (i=1;i<=NF;i++) if (!($i ~ /^\//)) printf("%s%s",$i,FS)}{printf("\n")}'> $file.replace
fi
mv $file.replace $file
``` 

生成mysqld.ll后, 再次加载需要大量内存, 无法运行

  - <https://github.com/SRI-CSL/whole-program-llvm>
  - <https://stackoverflow.com/questions/9148890/how-to-make-clang-compile-to-llvm-ir>
    - link them into a single one

# 其他线索

  - binary rewriting
    - <https://github.com/GJDuck/e9patch>
    - <https://www.comp.nus.edu.sg/~gregory/papers/e9patch.pdf> 第七节, 有相关技术的描述
  - basic block reordering 
    - <https://arxiv.org/pdf/1809.04676.pdf>[  
](<https://arxiv.org/pdf/1809.04676.pdf>)
  - [b](<https://arxiv.org/pdf/1809.04676.pdf>)inary layout reordering 
    - <https://github.com/facebookincubator/BOLT>
  - DPP
    - <https://www.hpl.hp.com/techreports/Compaq-DEC/WRL-98-5.pdf>
  - <https://github.com/analysis-tools-dev/dynamic-analysis#binary>
  - dynamic binary instrumentation
    - <https://www.cs.tufts.edu/comp/150PAT/tools/caliper/epic1.pdf>
  - 使用DynamoRIO, 进行优化
    - [https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.222.6188&rep=rep1&type=pdf](<https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.222.6188&rep=rep1&type=pdf>)
      - 其中包括Inlining各种技术
    - <https://dynamorio.org/page_drwrap.html>
  - Intel Pin
