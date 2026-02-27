---
title: 20210915 - 编译gperftools / tcmalloc
confluence_page_id: 1343731
created_at: 2021-09-15T03:54:08+00:00
updated_at: 2021-09-15T07:11:46+00:00
---

准备centos 7 虚拟机

yum install -y rpm-build

wget <https://github.com/gperftools/gperftools/releases/download/gperftools-2.9.1/gperftools-2.9.1.tar.gz>

解压后, 在解压目录下再放一份 gperftools-2.9.1.tar.gz 的压缩包

cd 解压目录/packages

yum install -y gcc gcc-c++

修改 packages/rpm/rpm.spec文件的两行: 

![image2021-9-15 15:11:30.png](/assets/01KJBYEXCGC2N27HDWQWTJZJ9D/image2021-9-15%2015%3A11%3A30.png)

bash rpm.sh gperftools 2.9.1
