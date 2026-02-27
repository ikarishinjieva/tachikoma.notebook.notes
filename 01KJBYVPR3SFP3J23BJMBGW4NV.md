---
title: 20221121 - 麒麟V10编译MySQL 8.0报错
confluence_page_id: 2130194
created_at: 2022-11-21T06:55:06+00:00
updated_at: 2022-11-21T06:55:06+00:00
---

# 报错

![image2022-11-21 10:32:10.png](/assets/01KJBYVPR3SFP3J23BJMBGW4NV/image2022-11-21%2010%3A32%3A10.png)

# 解决

<https://lists.fedoraproject.org/archives/list/devel@lists.fedoraproject.org/thread/BPDQ5MH24ZUFXPPBDQUYHFJCTMTHBRLX/?sort=date>

指向gcc版本问题

安装devtoolset失败, ky10的包会依赖其他ky10的包, 跟官方源的包签名不同

核心问题: 需要将repo中ky10的包洗掉, 使用官方源兼容的包

尝试使用docker: 

```
docker run -it centos:8
 
> cd /etc/yum.repo.d
> sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*
> yum install -y gcc
...
编译MySQL成功
```
