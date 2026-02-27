---
title: 20221102 - 更新内核后, 服务器网卡驱动丢失
confluence_page_id: 2129979
created_at: 2022-11-02T03:04:56+00:00
updated_at: 2022-11-02T05:42:22+00:00
---

<http://10.186.18.11/confluence/pages/viewpage.action?pageId=39623044>

通过网卡型号, 找到Bcom NXE驱动: 

[https://dl.dell.com/FOLDER06231824M/1/Bcom_LAN_216.0.293.8_NXE_Linux_Source_216.0.293.8.tar.gz?uid=4a870f10-87a5-4596-0b77-bf29b9ae4df3&fn=Bcom_LAN_216.0.293.8_NXE_Linux_Source_216.0.293.8.tar.gz](<https://dl.dell.com/FOLDER06231824M/1/Bcom_LAN_216.0.293.8_NXE_Linux_Source_216.0.293.8.tar.gz?uid=4a870f10-87a5-4596-0b77-bf29b9ae4df3&fn=Bcom_LAN_216.0.293.8_NXE_Linux_Source_216.0.293.8.tar.gz>)

# 报错1

编译在 4.x内核通过, 在5.x内核报错. 报错为define语法错误, 认为是GCC版本过低, 不支持define 第二参数中 带空格的语法

![image2022-11-2 10:29:39.png](/assets/01KJBYSC9DM9XPHKKGY9HX1V4E/image2022-11-2%2010%3A29%3A39.png)

更换GCC版本: 

make install CC=gcc-10 CXX=g++-10

可解决当前报错

# 报错2

![image2022-11-2 11:4:53.png](/assets/01KJBYSC9DM9XPHKKGY9HX1V4E/image2022-11-2%2011%3A4%3A53.png)

报错为ib_umem_get函数使用与签名不匹配

查找Linux源码, 发现ib_umem_get函数 在 4.x->5.x 的过程中, 函数定义有变更

修改驱动代码, 满足该变更: 

![image2022-11-2 11:20:31.png](/assets/01KJBYSC9DM9XPHKKGY9HX1V4E/image2022-11-2%2011%3A20%3A31.png)

编译成功: 

![image2022-11-2 11:23:51.png](/assets/01KJBYSC9DM9XPHKKGY9HX1V4E/image2022-11-2%2011%3A23%3A51.png)

通过insmod添加模块, 重启后可正常使用
