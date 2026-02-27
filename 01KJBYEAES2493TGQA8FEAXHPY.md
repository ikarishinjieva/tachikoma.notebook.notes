---
title: 20210814 - Clickhouse 编译
confluence_page_id: 1343623
created_at: 2021-08-14T15:43:16+00:00
updated_at: 2021-08-14T16:30:12+00:00
---

安装LLVM 11

```
wget https://apt.llvm.org/llvm.sh
chmod +x llvm.sh
sudo ./llvm.sh 11
``` 

```
ALL_PROXY=socks5://mgm.ntidc.actionsky.com:10800 git clone --progress --verbose --recursive https://github.com/ClickHouse/ClickHouse.git
 
git checkout v21.3.10.1-lts
 
ALL_PROXY=socks5://mgm.ntidc.actionsky.com:10800 git submodule update --init --recursive
 
mkdir build
 
/opt/cmake-3.16.9-Linux-x86_64/bin/cmake -DCMAKE_BUILD_TYPE=Debug -DCMAKE_C_COMPILER=$(which clang-11) -DCMAKE_CXX_COMPILER=$(which clang++-11) ..
 
ninja -j4
```
