---
title: 20230111 - 农行crash分析整理
confluence_page_id: 2130442
created_at: 2023-01-11T09:56:22+00:00
updated_at: 2023-01-11T09:56:22+00:00
---

# 分析材料

获取了 1月5日 和 1月9日两份coredump

# 分析1月5日coredump

崩溃堆栈: 

![image](http://8.134.54.170:8330/download/attachments/2130405/image2023-1-10%2013%3A48%3A47.png?version=1&modificationDate=1673329728055&api=v2)

堆栈显示崩溃逻辑是:

  - temptable::Handler::rnd_next: 访问temptable表数据
  - temptable::Row::copy_to_mysql_row: 将temptable中的行数据 转换为 MySQL输出的数据格式
  - temptable::Column::write_std_user_data: 将temptable中的行中的列数据 转换为 MySQL输出的数据格式
  - 此处memcpy内存访问发生失败

查看扫描到temptable的哪一行: 

```
//0xfff0580ff598 是从堆栈中获取的handler地址
 
(gdb) p ((temptable::Handler*)0xfff0580ff598)->m_rnd_iterator->m_element
$38 = (temptable::Storage::Element *) 0xffee59188550
 
//0xffee61500258 是当前行的迭代器地址
 
//查看该行的列信息
(gdb) p *static_cast<const temptable::Row *>(((temptable::Handler*)0xfff0580ff598)->m_rnd_iterator->m_element)
$2 = {m_allocator = 0xfff05802dbe8, m_data_is_in_mysql_memory = false, m_ptr = 0xffee61500258 '\n' <repeats 200 times>...}
 
(gdb) p *reinterpret_cast<Cell *>(0xffee61500258+sizeof(size_t))
$8 = {m_is_null = 10, m_data_length = 168430090, m_data = 0xa0a0a0a0a0a0a0a <error: Cannot access memory at address 0xa0a0a0a0a0a0a0a>}

//列的数据 m_data 指向非法地址 0xa0a0a0a0a0a0a0a
//访问该数据会产生MySQL崩溃
``` 

遍历temptable所有页和行, 找到问题行所在位置, 此处省略过程

问题行所在位置为 temptable的第7页第1510行

检查该行前后的行: 

```
(gdb) p *reinterpret_cast<Cell *>(static_cast<const temptable::Row *>(0xffee5917c8b0 + 1508*32)->m_ptr+sizeof(size_t))
$50 = {m_is_null = false, m_data_length = 6,
  m_data = 0xffee61489798 "680934e9ab3894a71a45c29bbcfb45fb7782ac59127\231\255\277\003\263\062\061 \344\270\207\345\255\227\351\225\277\346\226\207 | \344\275\277\347\224\250 RBAC \351\231\220\345\210\266\345\257\271 Kubernetes \350\265\204\346\272\220\347\232\204\350\256\277\351\227\256 &^%1{\"md\":\"<div class=\\\"rich_media_content", ' ' <repeats 39 times>, "\\\" i"...}
  
(gdb) p *reinterpret_cast<Cell *>(static_cast<const temptable::Row *>(0xffee5917c8b0 + 1509*32)->m_ptr+sizeof(size_t))
$49 = {m_is_null = 10, m_data_length = 168430090, m_data = 0xa0a0a0a0a0a0a0a <error: Cannot access memory at address 0xa0a0a0a0a0a0a0a>}
(gdb) p *reinterpret_cast<Cell *>(static_cast<const temptable::Row *>(0xffee5917c8b0 + 1510*32)->m_ptr+sizeof(size_t))
$51 = {m_is_null = 10, m_data_length = 168430090, m_data = 0xa0a0a0a0a0a0a0a <error: Cannot access memory at address 0xa0a0a0a0a0a0a0a>}
(gdb) p *reinterpret_cast<Cell *>(static_cast<const temptable::Row *>(0xffee5917c8b0 + 1511*32)->m_ptr+sizeof(size_t))
$52 = {m_is_null = 10, m_data_length = 168430090, m_data = 0xa0a0a0a0a0a0a0a <error: Cannot access memory at address 0xa0a0a0a0a0a0a0a>}
(gdb) p *reinterpret_cast<Cell *>(static_cast<const temptable::Row *>(0xffee5917c8b0 + 1512*32)->m_ptr+sizeof(size_t))
$53 = {m_is_null = 10, m_data_length = 168430090, m_data = 0xa0a0a0a0a0a0a0a <error: Cannot access memory at address 0xa0a0a0a0a0a0a0a>}
(gdb) p *reinterpret_cast<Cell *>(static_cast<const temptable::Row *>(0xffee5917c8b0 + 1513*32)->m_ptr+sizeof(size_t))
$54 = {m_is_null = false, m_data_length = 6,
  m_data = 0xffee6157ebe0 "680959e9ab3894a71a45c29bbcfb45fb7782ac58951\231\255\276\340^21 \351\233\266\345\237\272\347\241\200\345\205\245\351\227\250\342\224\202\345\270\246\344\275\240\347\220\206\350\247\243Kubernetes &^%1{\"md\":\"<div class=\\\"rich_media_content", ' ' <repeats 39 times>, "\\\" id=\\\"js_content\\\" > <sec"...}
(gdb) p *reinterpret_cast<Cell *>(static_cast<const temptable::Row *>(0xffee5917c8b0 + 1514*32)->m_ptr+sizeof(size_t))
$55 = {m_is_null = false, m_data_length = 6,
  m_data = 0xffee61596780 "680973e9ab3894a71a45c29bbcfb45fb7782ac58952\231\255\277'\266\062\061 XRP\357\274\232\347\224\250eBPF\344\274\230\345\214\226\345\206\205\345\255\230\345\255\230\345\202\250\345\212\237\350\203\275 &^%1{\"md\":\"<div class=\\\"rich_media_content", ' ' <repeats 39 times>, "\\\" id=\\\"js_content\\\" > <sectio"...}
(gdb) p *reinterpret_cast<Cell *>(static_cast<const temptable::Row *>(0xffee5917c8b0 + 1515*32)->m_ptr+sizeof(size_t))
$56 = {m_is_null = false, m_data_length = 6,
  m_data = 0xffee615ac930 "680974e9ab3894a71a45c29bbcfb45fb7782ac58953\231\255\277E\023\062\061 \345\243\256\345\256\236\345\255\246\346\225\260\346\215\256\346\212\200\346\234\257\060\064\357\274\232ETL &^%1{\"md\":\"<div class=\\\"rich_media_content", ' ' <repeats 39 times>, "\\\" id=\\\"js_content\\\" > <section powere"...}
``` 

其中: 1509-1512记录都出现了内存异常, 非法内存地址均为 0xa0a0a0a0a0a0a0a

观察内存异常的位置:

![image](http://8.134.54.170:8330/download/attachments/2130405/image2023-1-11%200%3A31%3A37.png?version=1&modificationDate=1673368304135&api=v2)

确定数据: 

![image2023-1-11 17:23:16.png](/assets/01KJBYXQZ8AJVT5AG419NGRPC7/image2023-1-11%2017%3A23%3A16.png)

可以看到数据是正常业务数据 (HTML片段), 到内存异常位置 0xffee61490000 被截断

确定内存异常位置的下界 (0xa 持续到哪里)

![image2023-1-11 17:24:44.png](/assets/01KJBYXQZ8AJVT5AG419NGRPC7/image2023-1-11%2017%3A24%3A44.png)

超过异常位置下界后, 数据恢复正常, 是另一端截断的HTML代码

确定内存污染范围为: 0xffee61490000 - 0xffee61570000, 共 896kb

查看内存地址映射: 

```
//coredump 0105 :
//污染位置: 0xffee61490000 - 0xffee61570000
//mmap信息:
  
(gdb) info proc all
exe = '/usr/sbin/mysqld'
Mapped address spaces:
 
          Start Addr           End Addr       Size     Offset objfile
            0x400000          0x3240000  0x2e40000        0x0 /usr/sbin/mysqld
           0x3240000          0x33a0000   0x160000  0x2e30000 /usr/sbin/mysqld
           0x33a0000          0x3720000   0x380000  0x2f90000 /usr/sbin/mysqld
      0xffedcc000000     0xffedec000000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1612185994 (deleted)
      0xffedec000000     0xffee0c000000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1610612923 (deleted)
      0xffee0c000000     0xffee2c000000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1610612884 (deleted)
      0xffee2c000000     0xffee4c000000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1610612920 (deleted)
      0xffee4c000000     0xffee6c000000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1610612913 (deleted)
      0xffeec0000000     0xffeec4000000  0x4000000        0x0 /mysqldata/mysql_3306/tmp/#1612186005 (deleted)
      0xffeec4000000     0xffeed4000000 0x10000000        0x0 /mysqldata/mysql_3306/tmp/#1610612911 (deleted)
      0xffeedc000000     0xffeeec000000 0x10000000        0x0 /mysqldata/mysql_3306/tmp/#1610612898 (deleted)
      0xffeeec000000     0xffef0c000000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1612186019 (deleted)
      0xffef106a0000     0xffef126a0000  0x2000000        0x0 /mysqldata/mysql_3306/tmp/#1612186012 (deleted)
      0xffef126a0000     0xffef136a0000  0x1000000        0x0 /mysqldata/mysql_3306/tmp/#1612185998 (deleted)
      0xffef3c000000     0xffef44000000  0x8000000        0x0 /mysqldata/mysql_3306/tmp/#1612186020 (deleted)
      0xffef8c3c0000     0xffef8cbc0000   0x800000        0x0 /mysqldata/mysql_3306/tmp/#1612185988 (deleted)
      0xffef94250000     0xffef94650000   0x400000        0x0 /mysqldata/mysql_3306/tmp/#1612185986 (deleted)
      0xfff9f8380000     0xfff9f83d0000    0x50000        0x0 /usr/lib64/libblkid.so.1.1.0
      0xfff9f83d0000     0xfff9f83e0000    0x10000    0x40000 /usr/lib64/libblkid.so.1.1.0
      0xfff9f83e0000     0xfff9f83f0000    0x10000    0x50000 /usr/lib64/libblkid.so.1.1.0
      0xfff9f83f0000     0xfff9f8450000    0x60000        0x0 /usr/lib64/libmount.so.1.1.0
  

``` 
    
    
    污染位置对应的映射项: /mysqldata/mysql_3306/tmp/#1610612913

# 分析1月9日coredump

确定崩溃原理跟 1月5日的coredump相同

使用相同的方法, 确定内存污染范围

上界: 0xffe93d060000

![image](http://8.134.54.170:8330/download/attachments/2130405/image2023-1-11%2011%3A12%3A9.png?version=1&modificationDate=1673406736000&api=v2)

下界: 0xffe93d070000

![image2023-1-11 17:36:24.png](/assets/01KJBYXQZ8AJVT5AG419NGRPC7/image2023-1-11%2017%3A36%3A24.png)

污染区域: 0xffe93d060000 - 0xffe93d070000, 共64kb

  

```
//coredump 0109 :
//污染位置: 0xffe93d060000 - 0xffe93d070000
//mmap信息:
  
(gdb) info proc all
exe = '/usr/sbin/mysqld'
Mapped address spaces:
 
          Start Addr           End Addr       Size     Offset objfile
            0x400000          0x3240000  0x2e40000        0x0 /usr/sbin/mysqld
           0x3240000          0x33a0000   0x160000  0x2e30000 /usr/sbin/mysqld
           0x33a0000          0x3720000   0x380000  0x2f90000 /usr/sbin/mysqld
      0xffe4b7800000     0xffe4d7800000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1613321953 (deleted)
      0xffe4d7800000     0xffe4f7800000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1613321955 (deleted)
      0xffe4f7800000     0xffe517800000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1613321950 (deleted)
      0xffe517800000     0xffe537800000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1613321949 (deleted)
      0xffe537800000     0xffe557800000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1610669282 (deleted)
      0xffe557800000     0xffe577800000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1610616893 (deleted)
      0xffe577800000     0xffe597800000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1610616866 (deleted)
      0xffe597800000     0xffe5b7800000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1610616872 (deleted)
      0xffe5b7800000     0xffe5d7800000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1610616862 (deleted)
      0xffe5f7800000     0xffe617800000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1610616858 (deleted)
      0xffe617800000     0xffe637800000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1610616856 (deleted)
      0xffe637800000     0xffe657800000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1610613240 (deleted)
      0xffe657800000     0xffe677800000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1613321946 (deleted)
      0xffe677800000     0xffe697800000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1610613190 (deleted)
      0xffe697800000     0xffe6b7800000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1610617110 (deleted)
      0xffe6b7800000     0xffe6d7800000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1610616854 (deleted)
      0xffe6d7800000     0xffe6e7800000 0x10000000        0x0 /mysqldata/mysql_3306/tmp/#1613321931 (deleted)
      0xffe6e7800000     0xffe707800000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1610669294 (deleted)
      0xffe707800000     0xffe727800000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1611676640 (deleted)
      0xffe727800000     0xffe72f800000  0x8000000        0x0 /mysqldata/mysql_3306/tmp/#1613321927 (deleted)
      0xffe72f800000     0xffe74f800000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1611676633 (deleted)
      0xffe74f800000     0xffe76f800000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1610616857 (deleted)
      0xffe76f800000     0xffe77f800000 0x10000000        0x0 /mysqldata/mysql_3306/tmp/#1611676659 (deleted)
      0xffe77f800000     0xffe787800000  0x8000000        0x0 /mysqldata/mysql_3306/tmp/#1613321937 (deleted)
      0xffe787800000     0xffe7a7800000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1611293950 (deleted)
      0xffe7a7800000     0xffe7c7800000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1611676628 (deleted)
      0xffe7c7800000     0xffe7cb800000  0x4000000        0x0 /mysqldata/mysql_3306/tmp/#1610613158 (deleted)
      0xffe7cb800000     0xffe7db800000 0x10000000        0x0 /mysqldata/mysql_3306/tmp/#1610613234 (deleted)
      0xffe7db800000     0xffe7eb800000 0x10000000        0x0 /mysqldata/mysql_3306/tmp/#1611293889 (deleted)
      0xffe7f3f00000     0xffe7fbf00000  0x8000000        0x0 /mysqldata/mysql_3306/tmp/#1611676666 (deleted)
      0xffe7fbf00000     0xffe7fff00000  0x4000000        0x0 /mysqldata/mysql_3306/tmp/#1613321936 (deleted)
--Type <RET> for more, q to quit, c to continue without paging--
      0xffe7fff00000     0xffe801f00000  0x2000000        0x0 /mysqldata/mysql_3306/tmp/#1613321935 (deleted)
      0xffe801f00000     0xffe821f00000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1611293929 (deleted)
      0xffe821f00000     0xffe831f00000 0x10000000        0x0 /mysqldata/mysql_3306/tmp/#1611676643 (deleted)
      0xffe831f00000     0xffe832f00000  0x1000000        0x0 /mysqldata/mysql_3306/tmp/#1613321938 (deleted)
      0xffe832f00000     0xffe83af00000  0x8000000        0x0 /mysqldata/mysql_3306/tmp/#1611676656 (deleted)
      0xffe83af00000     0xffe83b700000   0x800000        0x0 /mysqldata/mysql_3306/tmp/#1613321934 (deleted)
      0xffe83b700000     0xffe83d700000  0x2000000        0x0 /mysqldata/mysql_3306/tmp/#1613321928 (deleted)
      0xffe83df00000     0xffe841f00000  0x4000000        0x0 /mysqldata/mysql_3306/tmp/#1611676661 (deleted)
      0xffe841f00000     0xffe861f00000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1611293918 (deleted)
      0xffe861f00000     0xffe862f00000  0x1000000        0x0 /mysqldata/mysql_3306/tmp/#1611676660 (deleted)
      0xffe862f00000     0xffe872f00000 0x10000000        0x0 /mysqldata/mysql_3306/tmp/#1611676641 (deleted)
      0xffe872f00000     0xffe876f00000  0x4000000        0x0 /mysqldata/mysql_3306/tmp/#1611676654 (deleted)
      0xffe876f00000     0xffe886f00000 0x10000000        0x0 /mysqldata/mysql_3306/tmp/#1611676634 (deleted)
      0xffe886f00000     0xffe888f00000  0x2000000        0x0 /mysqldata/mysql_3306/tmp/#1611676658 (deleted)
      0xffe888f00000     0xffe890f00000  0x8000000        0x0 /mysqldata/mysql_3306/tmp/#1611293901 (deleted)
      0xffe890f00000     0xffe891f00000  0x1000000        0x0 /mysqldata/mysql_3306/tmp/#1610616891 (deleted)
      0xffe891f00000     0xffe8a1f00000 0x10000000        0x0 /mysqldata/mysql_3306/tmp/#1610616863 (deleted)
      0xffe8a1f00000     0xffe8b1f00000 0x10000000        0x0 /mysqldata/mysql_3306/tmp/#1610616870 (deleted)
      0xffe8b1f00000     0xffe8c1f00000 0x10000000        0x0 /mysqldata/mysql_3306/tmp/#1611676629 (deleted)
      0xffe8c1f00000     0xffe8c5f00000  0x4000000        0x0 /mysqldata/mysql_3306/tmp/#1610616892 (deleted)
      0xffe8e5f00000     0xffe8f5f00000 0x10000000        0x0 /mysqldata/mysql_3306/tmp/#1611293932 (deleted)
      0xffe8f5f00000     0xffe915f00000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1610669286 (deleted)
      0xffe915f00000     0xffe935f00000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1613321932 (deleted)
      0xffe935f00000     0xffe955f00000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1610616875 (deleted)
      0xffe955f00000     0xffe965f00000 0x10000000        0x0 /mysqldata/mysql_3306/tmp/#1611293933 (deleted)
      0xffe965f00000     0xffe96df00000  0x8000000        0x0 /mysqldata/mysql_3306/tmp/#1611676630 (deleted)
      0xffe96df00000     0xffe98df00000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1613321952 (deleted)
      0xffe98df00000     0xffe99df00000 0x10000000        0x0 /mysqldata/mysql_3306/tmp/#1611293926 (deleted)
      0xffe99df00000     0xffe99e300000   0x400000        0x0 /mysqldata/mysql_3306/tmp/#1613321933 (deleted)
      0xffe99e700000     0xffe9a6700000  0x8000000        0x0 /mysqldata/mysql_3306/tmp/#1610669295 (deleted)
      0xffe9a6700000     0xffe9a7700000  0x1000000        0x0 /mysqldata/mysql_3306/tmp/#1611676639 (deleted)
      0xffe9a7800000     0xffe9a7a00000   0x200000        0x0 /mysqldata/mysql_3306/tmp/#1613321930 (deleted)
      0xffe9a7a00000     0xffe9a7b00000   0x100000        0x0 /mysqldata/mysql_3306/tmp/#1613321929 (deleted)
      0xffe9a7e00000     0xffe9afe00000  0x8000000        0x0 /mysqldata/mysql_3306/tmp/#1611293946 (deleted)
      0xffe9afe00000     0xffe9b1e00000  0x2000000        0x0 /mysqldata/mysql_3306/tmp/#1611293948 (deleted)
      0xffe9b1e00000     0xffe9b5e00000  0x4000000        0x0 /mysqldata/mysql_3306/tmp/#1611293943 (deleted)
      0xffe9b5e00000     0xffe9bde00000  0x8000000        0x0 /mysqldata/mysql_3306/tmp/#1611293941 (deleted)
      0xffe9bde00000     0xffe9c5e00000  0x8000000        0x0 /mysqldata/mysql_3306/tmp/#1611293942 (deleted)
--Type <RET> for more, q to quit, c to continue without paging--
      0xffe9c5e00000     0xffe9e5e00000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1610639285 (deleted)
      0xffe9e5e00000     0xffe9e6e00000  0x1000000        0x0 /mysqldata/mysql_3306/tmp/#1610613220 (deleted)
      0xffe9e6e00000     0xffea06e00000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1610616867 (deleted)
      0xffea06e00000     0xffea16e00000 0x10000000        0x0 /mysqldata/mysql_3306/tmp/#1610669300 (deleted)
      0xffea16e00000     0xffea1ae00000  0x4000000        0x0 /mysqldata/mysql_3306/tmp/#1611293936 (deleted)
      0xffea1ae00000     0xffea1ce00000  0x2000000        0x0 /mysqldata/mysql_3306/tmp/#1611293947 (deleted)
      0xffea1ce00000     0xffea1de00000  0x1000000        0x0 /mysqldata/mysql_3306/tmp/#1611293944 (deleted)
      0xffea1de00000     0xffea25e00000  0x8000000        0x0 /mysqldata/mysql_3306/tmp/#1611293914 (deleted)
      0xffea25e00000     0xffea35e00000 0x10000000        0x0 /mysqldata/mysql_3306/tmp/#1610669287 (deleted)
      0xffea35e00000     0xffea3de00000  0x8000000        0x0 /mysqldata/mysql_3306/tmp/#1611293898 (deleted)
      0xffea3de00000     0xffea3e600000   0x800000        0x0 /mysqldata/mysql_3306/tmp/#1611676638 (deleted)
      0xffea3e600000     0xffea3ee00000   0x800000        0x0 /mysqldata/mysql_3306/tmp/#1611293951 (deleted)
      0xffea3ee00000     0xffea42e00000  0x4000000        0x0 /mysqldata/mysql_3306/tmp/#1611293922 (deleted)
      0xffea42e00000     0xffea44e00000  0x2000000        0x0 /mysqldata/mysql_3306/tmp/#1611676652 (deleted)
      0xffea44e00000     0xffea46e00000  0x2000000        0x0 /mysqldata/mysql_3306/tmp/#1611676650 (deleted)
      0xffea46e00000     0xffea47200000   0x400000        0x0 /mysqldata/mysql_3306/tmp/#1611676637 (deleted)
      0xffea47200000     0xffea47400000   0x200000        0x0 /mysqldata/mysql_3306/tmp/#1611676636 (deleted)
      0xffea47400000     0xffea47800000   0x400000        0x0 /mysqldata/mysql_3306/tmp/#1611293949 (deleted)
      0xffea47800000     0xffea47a00000   0x200000        0x0 /mysqldata/mysql_3306/tmp/#1611293945 (deleted)
      0xffea47a00000     0xffea67a00000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1610617097 (deleted)
      0xffea67a00000     0xffea67b00000   0x100000        0x0 /mysqldata/mysql_3306/tmp/#1611676635 (deleted)
      0xffea67b00000     0xffea67c00000   0x100000        0x0 /mysqldata/mysql_3306/tmp/#1611293934 (deleted)
      0xffea67c00000     0xffea69c00000  0x2000000        0x0 /mysqldata/mysql_3306/tmp/#1611293924 (deleted)
      0xffea69c00000     0xffea6a400000   0x800000        0x0 /mysqldata/mysql_3306/tmp/#1611293931 (deleted)
      0xffea6a400000     0xffea6a800000   0x400000        0x0 /mysqldata/mysql_3306/tmp/#1611293930 (deleted)
      0xffea6a800000     0xffea6e800000  0x4000000        0x0 /mysqldata/mysql_3306/tmp/#1611293909 (deleted)
      0xffea6e800000     0xffea70800000  0x2000000        0x0 /mysqldata/mysql_3306/tmp/#1611293919 (deleted)
      0xffea70800000     0xffea78800000  0x8000000        0x0 /mysqldata/mysql_3306/tmp/#1610669306 (deleted)
      0xffea78800000     0xffea79800000  0x1000000        0x0 /mysqldata/mysql_3306/tmp/#1611293920 (deleted)
      0xffea79800000     0xffea89800000 0x10000000        0x0 /mysqldata/mysql_3306/tmp/#1610669264 (deleted)
      0xffea89800000     0xffea8b800000  0x2000000        0x0 /mysqldata/mysql_3306/tmp/#1611293912 (deleted)
      0xffea8b800000     0xffea8c000000   0x800000        0x0 /mysqldata/mysql_3306/tmp/#1611293917 (deleted)
      0xffea8c000000     0xffea8c800000   0x800000        0x0 /mysqldata/mysql_3306/tmp/#1611676663 (deleted)
      0xffea8c800000     0xffea8d800000  0x1000000        0x0 /mysqldata/mysql_3306/tmp/#1611293899 (deleted)
      0xffea8d800000     0xffea8e000000   0x800000        0x0 /mysqldata/mysql_3306/tmp/#1611676655 (deleted)
      0xffea8e000000     0xffea92000000  0x4000000        0x0 /mysqldata/mysql_3306/tmp/#1611676632 (deleted)
      0xffea92000000     0xffea93000000  0x1000000        0x0 /mysqldata/mysql_3306/tmp/#1611293908 (deleted)
      0xffea93000000     0xffea9b000000  0x8000000        0x0 /mysqldata/mysql_3306/tmp/#1610669292 (deleted)
--Type <RET> for more, q to quit, c to continue without paging--
      0xffea9b000000     0xffea9b400000   0x400000        0x0 /mysqldata/mysql_3306/tmp/#1611293916 (deleted)
      0xffea9b400000     0xffea9f400000  0x4000000        0x0 /mysqldata/mysql_3306/tmp/#1611293888 (deleted)
      0xffea9f400000     0xffea9fc00000   0x800000        0x0 /mysqldata/mysql_3306/tmp/#1611293911 (deleted)
      0xffea9fc00000     0xffeaafc00000 0x10000000        0x0 /mysqldata/mysql_3306/tmp/#1610669249 (deleted)
      0xffeaafc00000     0xffeab0c00000  0x1000000        0x0 /mysqldata/mysql_3306/tmp/#1611293906 (deleted)
      0xffeab0c00000     0xffeab2c00000  0x2000000        0x0 /mysqldata/mysql_3306/tmp/#1611293892 (deleted)
      0xffeab2c00000     0xffeab3000000   0x400000        0x0 /mysqldata/mysql_3306/tmp/#1611293905 (deleted)
      0xffeab3000000     0xffeab3200000   0x200000        0x0 /mysqldata/mysql_3306/tmp/#1611293928 (deleted)
      0xffeab3200000     0xffeab4200000  0x1000000        0x0 /mysqldata/mysql_3306/tmp/#1611293910 (deleted)
      0xffeab4200000     0xffeab4400000   0x200000        0x0 /mysqldata/mysql_3306/tmp/#1611293915 (deleted)
      0xffeab4400000     0xffeab4c00000   0x800000        0x0 /mysqldata/mysql_3306/tmp/#1611293896 (deleted)
      0xffeab4c00000     0xffeab5400000   0x800000        0x0 /mysqldata/mysql_3306/tmp/#1611676646 (deleted)
      0xffeab5400000     0xffeab5600000   0x200000        0x0 /mysqldata/mysql_3306/tmp/#1611293907 (deleted)
      0xffeab5600000     0xffeab5a00000   0x400000        0x0 /mysqldata/mysql_3306/tmp/#1611676668 (deleted)
      0xffeab5a00000     0xffeab5e00000   0x400000        0x0 /mysqldata/mysql_3306/tmp/#1611293897 (deleted)
      0xffeab5e00000     0xffeac5e00000 0x10000000        0x0 /mysqldata/mysql_3306/tmp/#1610639291 (deleted)
      0xffeac5e00000     0xffeac6000000   0x200000        0x0 /mysqldata/mysql_3306/tmp/#1611293895 (deleted)
      0xffeac6000000     0xffeac7000000  0x1000000        0x0 /mysqldata/mysql_3306/tmp/#1610669309 (deleted)
      0xffeac7000000     0xffeac9000000  0x2000000        0x0 /mysqldata/mysql_3306/tmp/#1611676647 (deleted)
      0xffeac9000000     0xffead1000000  0x8000000        0x0 /mysqldata/mysql_3306/tmp/#1610669281 (deleted)
      0xffead1000000     0xffead3000000  0x2000000        0x0 /mysqldata/mysql_3306/tmp/#1610669304 (deleted)
      0xffead3000000     0xffead3800000   0x800000        0x0 /mysqldata/mysql_3306/tmp/#1611293891 (deleted)
      0xffead3800000     0xffead3c00000   0x400000        0x0 /mysqldata/mysql_3306/tmp/#1610669311 (deleted)
      0xffead3c00000     0xffead7c00000  0x4000000        0x0 /mysqldata/mysql_3306/tmp/#1610669285 (deleted)
      0xffead7c00000     0xffead7e00000   0x200000        0x0 /mysqldata/mysql_3306/tmp/#1611676665 (deleted)
      0xffead7e00000     0xffead8000000   0x200000        0x0 /mysqldata/mysql_3306/tmp/#1610669308 (deleted)
      0xffead8000000     0xffead9000000  0x1000000        0x0 /mysqldata/mysql_3306/tmp/#1611293938 (deleted)
      0xffead9000000     0xffeada000000  0x1000000        0x0 /mysqldata/mysql_3306/tmp/#1610669296 (deleted)
      0xffeada000000     0xffeade000000  0x4000000        0x0 /mysqldata/mysql_3306/tmp/#1610669284 (deleted)
      0xffeade000000     0xffeae6000000  0x8000000        0x0 /mysqldata/mysql_3306/tmp/#1610669265 (deleted)
      0xffeae6000000     0xffeae6800000   0x800000        0x0 /mysqldata/mysql_3306/tmp/#1610669303 (deleted)
      0xffeae6800000     0xffeaf6800000 0x10000000        0x0 /mysqldata/mysql_3306/tmp/#1610639281 (deleted)
      0xffeaf6800000     0xffeaf6c00000   0x400000        0x0 /mysqldata/mysql_3306/tmp/#1610669299 (deleted)
      0xffeaf6c00000     0xffeaf7400000   0x800000        0x0 /mysqldata/mysql_3306/tmp/#1610669291 (deleted)
      0xffeaf7400000     0xffeaf7800000   0x400000        0x0 /mysqldata/mysql_3306/tmp/#1610669290 (deleted)
      0xffeaf7800000     0xffeaf9800000  0x2000000        0x0 /mysqldata/mysql_3306/tmp/#1610669280 (deleted)
      0xffeaf9800000     0xffeb19800000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1610617141 (deleted)
      0xffeb19800000     0xffeb1d800000  0x4000000        0x0 /mysqldata/mysql_3306/tmp/#1610669272 (deleted)
--Type <RET> for more, q to quit, c to continue without paging--
      0xffeb1d800000     0xffeb1f800000  0x2000000        0x0 /mysqldata/mysql_3306/tmp/#1610669278 (deleted)
      0xffeb1f800000     0xffeb2f800000 0x10000000        0x0 /mysqldata/mysql_3306/tmp/#1610639271 (deleted)
      0xffeb2f800000     0xffeb30800000  0x1000000        0x0 /mysqldata/mysql_3306/tmp/#1610669279 (deleted)
      0xffeb30800000     0xffeb34800000  0x4000000        0x0 /mysqldata/mysql_3306/tmp/#1610669255 (deleted)
      0xffeb34800000     0xffeb35000000   0x800000        0x0 /mysqldata/mysql_3306/tmp/#1610669277 (deleted)
      0xffeb35000000     0xffeb36000000  0x1000000        0x0 /mysqldata/mysql_3306/tmp/#1610669275 (deleted)
      0xffeb36000000     0xffeb36400000   0x400000        0x0 /mysqldata/mysql_3306/tmp/#1610669276 (deleted)
      0xffeb36400000     0xffeb36c00000   0x800000        0x0 /mysqldata/mysql_3306/tmp/#1610669274 (deleted)
      0xffeb36c00000     0xffeb37000000   0x400000        0x0 /mysqldata/mysql_3306/tmp/#1610669270 (deleted)
      0xffeb37000000     0xffeb39000000  0x2000000        0x0 /mysqldata/mysql_3306/tmp/#1610669253 (deleted)
      0xffeb39000000     0xffeb41000000  0x8000000        0x0 /mysqldata/mysql_3306/tmp/#1610639282 (deleted)
      0xffeb41000000     0xffeb43000000  0x2000000        0x0 /mysqldata/mysql_3306/tmp/#1610648344 (deleted)
      0xffeb43000000     0xffeb47000000  0x4000000        0x0 /mysqldata/mysql_3306/tmp/#1610639286 (deleted)
      0xffeb47000000     0xffeb57000000 0x10000000        0x0 /mysqldata/mysql_3306/tmp/#1610639255 (deleted)
      0xffeb57000000     0xffeb58000000  0x1000000        0x0 /mysqldata/mysql_3306/tmp/#1610669251 (deleted)
      0xffeb58000000     0xffeb58800000   0x800000        0x0 /mysqldata/mysql_3306/tmp/#1610669248 (deleted)
      0xffeb58800000     0xffeb60800000  0x8000000        0x0 /mysqldata/mysql_3306/tmp/#1610639265 (deleted)
      0xffeb60800000     0xffeb60c00000   0x400000        0x0 /mysqldata/mysql_3306/tmp/#1610639295 (deleted)
      0xffeb60c00000     0xffeb61c00000  0x1000000        0x0 /mysqldata/mysql_3306/tmp/#1610639292 (deleted)
      0xffeb61c00000     0xffeb65c00000  0x4000000        0x0 /mysqldata/mysql_3306/tmp/#1610639280 (deleted)
      0xffeb65c00000     0xffeb66400000   0x800000        0x0 /mysqldata/mysql_3306/tmp/#1610639290 (deleted)
      0xffeb66400000     0xffeb66800000   0x400000        0x0 /mysqldata/mysql_3306/tmp/#1610639289 (deleted)
      0xffeb66800000     0xffeb68800000  0x2000000        0x0 /mysqldata/mysql_3306/tmp/#1610639283 (deleted)
      0xffeb68800000     0xffeb88800000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1611676631 (deleted)
      0xffeb88800000     0xffeb89800000  0x1000000        0x0 /mysqldata/mysql_3306/tmp/#1610639279 (deleted)
      0xffeb89800000     0xffeb8d800000  0x4000000        0x0 /mysqldata/mysql_3306/tmp/#1610639270 (deleted)
      0xffeb8d800000     0xffebad800000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1610617107 (deleted)
      0xffebad800000     0xffebaf800000  0x2000000        0x0 /mysqldata/mysql_3306/tmp/#1610639268 (deleted)
      0xffebaf800000     0xffebb0000000   0x800000        0x0 /mysqldata/mysql_3306/tmp/#1610639278 (deleted)
      0xffebb0000000     0xffebb8000000  0x8000000        0x0 /mysqldata/mysql_3306/tmp/#1610639257 (deleted)
      0xffebb8000000     0xffebb9000000  0x1000000        0x0 /mysqldata/mysql_3306/tmp/#1610639266 (deleted)
      0xffebb9000000     0xffebc9000000 0x10000000        0x0 /mysqldata/mysql_3306/tmp/#1610617146 (deleted)
      0xffebc9000000     0xffebd1000000  0x8000000        0x0 /mysqldata/mysql_3306/tmp/#1610639252 (deleted)
      0xffebd1000000     0xffebd5000000  0x4000000        0x0 /mysqldata/mysql_3306/tmp/#1610639262 (deleted)
      0xffebd5000000     0xffebd5400000   0x400000        0x0 /mysqldata/mysql_3306/tmp/#1610639275 (deleted)
      0xffebd5400000     0xffebd7400000  0x2000000        0x0 /mysqldata/mysql_3306/tmp/#1610639267 (deleted)
      0xffebd7400000     0xffebdf400000  0x8000000        0x0 /mysqldata/mysql_3306/tmp/#1610639239 (deleted)
      0xffebdf400000     0xffebdf800000   0x400000        0x0 /mysqldata/mysql_3306/tmp/#1610639276 (deleted)
--Type <RET> for more, q to quit, c to continue without paging--
      0xffebdf800000     0xffebe0000000   0x800000        0x0 /mysqldata/mysql_3306/tmp/#1610639277 (deleted)
      0xffebe0000000     0xffebe1000000  0x1000000        0x0 /mysqldata/mysql_3306/tmp/#1610639263 (deleted)
      0xffebe1000000     0xffebe1800000   0x800000        0x0 /mysqldata/mysql_3306/tmp/#1610639261 (deleted)
      0xffebe1800000     0xffebe3800000  0x2000000        0x0 /mysqldata/mysql_3306/tmp/#1610639256 (deleted)
      0xffebe3800000     0xffebe3c00000   0x400000        0x0 /mysqldata/mysql_3306/tmp/#1610639260 (deleted)
      0xffebe3c00000     0xffebebc00000  0x8000000        0x0 /mysqldata/mysql_3306/tmp/#1610617139 (deleted)
      0xffebebc00000     0xffebefc00000  0x4000000        0x0 /mysqldata/mysql_3306/tmp/#1610637041 (deleted)
      0xffebefc00000     0xffebf0c00000  0x1000000        0x0 /mysqldata/mysql_3306/tmp/#1610639254 (deleted)
      0xffebf0c00000     0xffebf1400000   0x800000        0x0 /mysqldata/mysql_3306/tmp/#1610639253 (deleted)
      0xffebf1400000     0xffebf5400000  0x4000000        0x0 /mysqldata/mysql_3306/tmp/#1610617145 (deleted)
      0xffebf5400000     0xffebf9400000  0x4000000        0x0 /mysqldata/mysql_3306/tmp/#1610617137 (deleted)
      0xffebf9400000     0xffec01400000  0x8000000        0x0 /mysqldata/mysql_3306/tmp/#1610617129 (deleted)
      0xffec01400000     0xffec03400000  0x2000000        0x0 /mysqldata/mysql_3306/tmp/#1610617148 (deleted)
      0xffec03400000     0xffec13400000 0x10000000        0x0 /mysqldata/mysql_3306/tmp/#1610617108 (deleted)
      0xffec13400000     0xffec14400000  0x1000000        0x0 /mysqldata/mysql_3306/tmp/#1610617142 (deleted)
      0xffec14400000     0xffec16400000  0x2000000        0x0 /mysqldata/mysql_3306/tmp/#1610617150 (deleted)
      0xffec16400000     0xffec36400000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1613321944 (deleted)
      0xffec36400000     0xffec38400000  0x2000000        0x0 /mysqldata/mysql_3306/tmp/#1610617134 (deleted)
      0xffec38400000     0xffec3c400000  0x4000000        0x0 /mysqldata/mysql_3306/tmp/#1610617130 (deleted)
      0xffec3c400000     0xffec3c800000   0x400000        0x0 /mysqldata/mysql_3306/tmp/#1610637028 (deleted)
      0xffec3c800000     0xffec5c800000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1610616881 (deleted)
      0xffec5c800000     0xffec5d800000  0x1000000        0x0 /mysqldata/mysql_3306/tmp/#1610617147 (deleted)
      0xffec5d800000     0xffec5e000000   0x800000        0x0 /mysqldata/mysql_3306/tmp/#1610637029 (deleted)
      0xffec5e000000     0xffec5e800000   0x800000        0x0 /mysqldata/mysql_3306/tmp/#1610617138 (deleted)
      0xffec5e800000     0xffec5ec00000   0x400000        0x0 /mysqldata/mysql_3306/tmp/#1610639249 (deleted)
      0xffec5ec00000     0xffec5fc00000  0x1000000        0x0 /mysqldata/mysql_3306/tmp/#1610617144 (deleted)
      0xffec5fc00000     0xffec67c00000  0x8000000        0x0 /mysqldata/mysql_3306/tmp/#1610617118 (deleted)
      0xffec67c00000     0xffec68000000   0x400000        0x0 /mysqldata/mysql_3306/tmp/#1610617143 (deleted)
      0xffec68000000     0xffec68800000   0x800000        0x0 /mysqldata/mysql_3306/tmp/#1610617135 (deleted)
      0xffec68800000     0xffec6a800000  0x2000000        0x0 /mysqldata/mysql_3306/tmp/#1610617126 (deleted)
      0xffec6a800000     0xffec8a800000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1613321939 (deleted)
      0xffec8a800000     0xffec8e800000  0x4000000        0x0 /mysqldata/mysql_3306/tmp/#1610617121 (deleted)
      0xffec8e800000     0xffecae800000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1611676657 (deleted)
      0xffecae800000     0xffecaf800000  0x1000000        0x0 /mysqldata/mysql_3306/tmp/#1610617125 (deleted)
      0xffecaf800000     0xffecb0000000   0x800000        0x0 /mysqldata/mysql_3306/tmp/#1610617127 (deleted)
      0xffecb0000000     0xffecd0000000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1610669283 (deleted)
      0xffecd0000000     0xffecf0000000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1610616868 (deleted)
      0xffecf0000000     0xffed00000000 0x10000000        0x0 /mysqldata/mysql_3306/tmp/#1610617092 (deleted)
--Type <RET> for more, q to quit, c to continue without paging--
      0xffed00000000     0xffed02000000  0x2000000        0x0 /mysqldata/mysql_3306/tmp/#1610617103 (deleted)
      0xffed02000000     0xffed0a000000  0x8000000        0x0 /mysqldata/mysql_3306/tmp/#1610617100 (deleted)
      0xffed0a000000     0xffed2a000000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1610617090 (deleted)
      0xffed2a000000     0xffed32000000  0x8000000        0x0 /mysqldata/mysql_3306/tmp/#1610617096 (deleted)
      0xffed32000000     0xffed36000000  0x4000000        0x0 /mysqldata/mysql_3306/tmp/#1610617095 (deleted)
      0xffed36000000     0xffed46000000 0x10000000        0x0 /mysqldata/mysql_3306/tmp/#1610613230 (deleted)
      0xffed46000000     0xffed4a000000  0x4000000        0x0 /mysqldata/mysql_3306/tmp/#1611293893 (deleted)
      0xffed4a000000     0xffed52000000  0x8000000        0x0 /mysqldata/mysql_3306/tmp/#1610613227 (deleted)
      0xffed52000000     0xffed72000000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1610613160 (deleted)
      0xffed82000000     0xffed8a000000  0x8000000        0x0 /mysqldata/mysql_3306/tmp/#1610613225 (deleted)
      0xffed8a000000     0xffed9a000000 0x10000000        0x0 /mysqldata/mysql_3306/tmp/#1610669305 (deleted)
      0xffeda0000000     0xffedb0000000 0x10000000        0x0 /mysqldata/mysql_3306/tmp/#1610613232 (deleted)
      0xffedb0000000     0xffedc0000000 0x10000000        0x0 /mysqldata/mysql_3306/tmp/#1610613163 (deleted)
      0xffedc0000000     0xffedc8000000  0x8000000        0x0 /mysqldata/mysql_3306/tmp/#1610613218 (deleted)
      0xffedc8000000     0xffedd8000000 0x10000000        0x0 /mysqldata/mysql_3306/tmp/#1613321948 (deleted)
      0xffedd8000000     0xffedf8000000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1611676648 (deleted)
      0xffedfc000000     0xffee00000000  0x4000000        0x0 /mysqldata/mysql_3306/tmp/#1613321951 (deleted)
      0xffee00000000     0xffee10000000 0x10000000        0x0 /mysqldata/mysql_3306/tmp/#1613321942 (deleted)
      0xffee10000000     0xffee30000000 0x20000000        0x0 /mysqldata/mysql_3306/tmp/#1611676644 (deleted)
      0xffee30000000     0xffee30400000   0x400000        0x0 /mysqldata/mysql_3306/tmp/#1610617133 (deleted)
      0xffee30400000     0xffee30800000   0x400000        0x0 /mysqldata/mysql_3306/tmp/#1610617124 (deleted)
      0xffee30800000     0xffee32800000  0x2000000        0x0 /mysqldata/mysql_3306/tmp/#1610617116 (deleted)
      0xffee32800000     0xffee3a800000  0x8000000        0x0 /mysqldata/mysql_3306/tmp/#1610616853 (deleted)
      0xffee3a800000     0xffee3e800000  0x4000000        0x0 /mysqldata/mysql_3306/tmp/#1610617109 (deleted)
      0xffee3e800000     0xffee3f800000  0x1000000        0x0 /mysqldata/mysql_3306/tmp/#1610617120 (deleted)
      0xffee3f800000     0xffee40000000   0x800000        0x0 /mysqldata/mysql_3306/tmp/#1610617119 (deleted)
      0xffee50000000     0xffee58000000  0x8000000        0x0 /mysqldata/mysql_3306/tmp/#1613321941 (deleted)
      0xffee58000000     0xffee58400000   0x400000        0x0 /mysqldata/mysql_3306/tmp/#1610617117 (deleted)
      0xffee58400000     0xffee5a400000  0x2000000        0x0 /mysqldata/mysql_3306/tmp/#1610617106 (deleted)
      0xffee5a400000     0xffee5b400000  0x1000000        0x0 /mysqldata/mysql_3306/tmp/#1610617105 (deleted)
      0xffee5b400000     0xffee5bc00000   0x800000        0x0 /mysqldata/mysql_3306/tmp/#1610613239 (deleted)
      0xffee5bc00000     0xffee5fc00000  0x4000000        0x0 /mysqldata/mysql_3306/tmp/#1610617104 (deleted)
      0xffee5fc00000     0xffee60000000   0x400000        0x0 /mysqldata/mysql_3306/tmp/#1610616847 (deleted)
      0xffee60000000     0xffee61000000  0x1000000        0x0 /mysqldata/mysql_3306/tmp/#1610617102 (deleted)
      0xffee61000000     0xffee65000000  0x4000000        0x0 /mysqldata/mysql_3306/tmp/#1610616887 (deleted)
      0xffee6c000000     0xffee74000000  0x8000000        0x0 /mysqldata/mysql_3306/tmp/#1610616860 (deleted)
      0xffee78000000     0xffee88000000 0x10000000        0x0 /mysqldata/mysql_3306/tmp/#1611676669 (deleted)
      0xffee88000000     0xffee90000000  0x8000000        0x0 /mysqldata/mysql_3306/tmp/#1613321945 (deleted)
--Type <RET> for more, q to quit, c to continue without paging--
      0xffeea0000000     0xffeea4000000  0x4000000        0x0 /mysqldata/mysql_3306/tmp/#1610616865 (deleted)
      0xffeeac000000     0xffeebc000000 0x10000000        0x0 /mysqldata/mysql_3306/tmp/#1610616883 (deleted)
      0xffeebc000000     0xffeec4000000  0x8000000        0x0 /mysqldata/mysql_3306/tmp/#1610616882 (deleted)
      0xffeedc000000     0xffeee4000000  0x8000000        0x0 /mysqldata/mysql_3306/tmp/#1610613226 (deleted)
      0xffeeec000000     0xffeeec800000   0x800000        0x0 /mysqldata/mysql_3306/tmp/#1610617101 (deleted)
      0xffeeec800000     0xffeeee800000  0x2000000        0x0 /mysqldata/mysql_3306/tmp/#1610617094 (deleted)
      0xffeeee800000     0xffeeef800000  0x1000000        0x0 /mysqldata/mysql_3306/tmp/#1610617091 (deleted)
      0xffeeef800000     0xffeef0000000   0x800000        0x0 /mysqldata/mysql_3306/tmp/#1610617089 (deleted)
      0xffeef6000000     0xffeef8000000  0x2000000        0x0 /mysqldata/mysql_3306/tmp/#1611676642 (deleted)
      0xfff03c000000     0xfff040000000  0x4000000        0x0 /mysqldata/mysql_3306/tmp/#1610613236 (deleted)
      0xfff044000000     0xfff044200000   0x200000        0x0 /mysqldata/mysql_3306/tmp/#1610669297 (deleted)
      0xfff044200000     0xfff044600000   0x400000        0x0 /mysqldata/mysql_3306/tmp/#1610617099 (deleted)
      0xfff04c000000     0xfff04c200000   0x200000        0x0 /mysqldata/mysql_3306/tmp/#1610669289 (deleted)
      0xfff04e400000     0xfff04e600000   0x200000        0x0 /mysqldata/mysql_3306/tmp/#1610669273 (deleted)
      0xfff0541a0000     0xfff0543a0000   0x200000        0x0 /mysqldata/mysql_3306/tmp/#1610669269 (deleted)
      0xfff054ca0000     0xfff0550a0000   0x400000        0x0 /mysqldata/mysql_3306/tmp/#1611676645 (deleted)
      0xfff0550a0000     0xfff0552a0000   0x200000        0x0 /mysqldata/mysql_3306/tmp/#1611293935 (deleted)
      0xfff0552a0000     0xfff0553a0000   0x100000        0x0 /mysqldata/mysql_3306/tmp/#1611293921 (deleted)
      0xfff0553a0000     0xfff055ba0000   0x800000        0x0 /mysqldata/mysql_3306/tmp/#1610616889 (deleted)
      0xfff055ca0000     0xfff0560a0000   0x400000        0x0 /mysqldata/mysql_3306/tmp/#1610617088 (deleted)
      0xfff0569d0000     0xfff056bd0000   0x200000        0x0 /mysqldata/mysql_3306/tmp/#1610639294 (deleted)
      0xfff057250000     0xfff057350000   0x100000        0x0 /mysqldata/mysql_3306/tmp/#1611293927 (deleted)
      0xfff0573f0000     0xfff0575f0000   0x200000        0x0 /mysqldata/mysql_3306/tmp/#1610639288 (deleted)
      0xfff057e00000     0xfff058000000   0x200000        0x0 /mysqldata/mysql_3306/tmp/#1610639274 (deleted)
      0xfff05c0e0000     0xfff05c1e0000   0x100000        0x0 /mysqldata/mysql_3306/tmp/#1611293913 (deleted)
      0xfff05c250000     0xfff05c350000   0x100000        0x0 /mysqldata/mysql_3306/tmp/#1611293900 (deleted)
      0xfff05c350000     0xfff05c550000   0x200000        0x0 /mysqldata/mysql_3306/tmp/#1610639272 (deleted)
      0xfff05ce50000     0xfff05d050000   0x200000        0x0 /mysqldata/mysql_3306/tmp/#1610639259 (deleted)
      0xfff05d110000     0xfff05d210000   0x100000        0x0 /mysqldata/mysql_3306/tmp/#1611293894 (deleted)
      0xfff05d400000     0xfff05d600000   0x200000        0x0 /mysqldata/mysql_3306/tmp/#1610617151 (deleted)
      0xfff05d7f0000     0xfff05d9f0000   0x200000        0x0 /mysqldata/mysql_3306/tmp/#1610639248 (deleted)
      0xfff05da70000     0xfff05dc70000   0x200000        0x0 /mysqldata/mysql_3306/tmp/#1610617140 (deleted)
      0xfff05ea50000     0xfff05ec50000   0x200000        0x0 /mysqldata/mysql_3306/tmp/#1610617132 (deleted)
      0xfff05eeb0000     0xfff05f0b0000   0x200000        0x0 /mysqldata/mysql_3306/tmp/#1610617123 (deleted)
      0xfff0940d0000     0xfff0941d0000   0x100000        0x0 /mysqldata/mysql_3306/tmp/#1610669307 (deleted)
      0xfff095ac0000     0xfff095bc0000   0x100000        0x0 /mysqldata/mysql_3306/tmp/#1610669293 (deleted)
      0xfff0963f0000     0xfff0964f0000   0x100000        0x0 /mysqldata/mysql_3306/tmp/#1610669288 (deleted)
      0xfff097420000     0xfff097620000   0x200000        0x0 /mysqldata/mysql_3306/tmp/#1610617115 (deleted)
--Type <RET> for more, q to quit, c to continue without paging--
      0xfff097a90000     0xfff097c90000   0x200000        0x0 /mysqldata/mysql_3306/tmp/#1610613238 (deleted)
      0xfff097e00000     0xfff098000000   0x200000        0x0 /mysqldata/mysql_3306/tmp/#1610617098 (deleted)
      0xfff09c100000     0xfff09c200000   0x100000        0x0 /mysqldata/mysql_3306/tmp/#1611676664 (deleted)
      0xfff09c200000     0xfff09c600000   0x400000        0x0 /mysqldata/mysql_3306/tmp/#1611676653 (deleted)
      0xfff09c600000     0xfff09c800000   0x200000        0x0 /mysqldata/mysql_3306/tmp/#1611676651 (deleted)
      0xfff09c800000     0xfff09c900000   0x100000        0x0 /mysqldata/mysql_3306/tmp/#1611293890 (deleted)
      0xfff09c900000     0xfff09cd00000   0x400000        0x0 /mysqldata/mysql_3306/tmp/#1610613235 (deleted)
      0xfff09ddf0000     0xfff09dff0000   0x200000        0x0 /mysqldata/mysql_3306/tmp/#1610616894 (deleted)
      0xfff0bc500000     0xfff0bc600000   0x100000        0x0 /mysqldata/mysql_3306/tmp/#1610669271 (deleted)
      0xfff0bccd0000     0xfff0bcdd0000   0x100000        0x0 /mysqldata/mysql_3306/tmp/#1610669268 (deleted)
      0xfff0bd000000     0xfff0bd100000   0x100000        0x0 /mysqldata/mysql_3306/tmp/#1610639293 (deleted)
      0xfff0bd8f0000     0xfff0bd9f0000   0x100000        0x0 /mysqldata/mysql_3306/tmp/#1610639287 (deleted)
      0xfff0be900000     0xfff0bea00000   0x100000        0x0 /mysqldata/mysql_3306/tmp/#1610639273 (deleted)
      0xfff0bec00000     0xfff0bed00000   0x100000        0x0 /mysqldata/mysql_3306/tmp/#1610639269 (deleted)
      0xfff0bf300000     0xfff0bf400000   0x100000        0x0 /mysqldata/mysql_3306/tmp/#1610639258 (deleted)
      0xfff0bf5f0000     0xfff0bf6f0000   0x100000        0x0 /mysqldata/mysql_3306/tmp/#1610617149 (deleted)
      0xfff0c48a0000     0xfff0c49a0000   0x100000        0x0 /mysqldata/mysql_3306/tmp/#1610639241 (deleted)
      0xfff0c50a0000     0xfff0c51a0000   0x100000        0x0 /mysqldata/mysql_3306/tmp/#1610617136 (deleted)
      0xfff0c54f0000     0xfff0c55f0000   0x100000        0x0 /mysqldata/mysql_3306/tmp/#1610617131 (deleted)
      0xfff0c6300000     0xfff0c6400000   0x100000        0x0 /mysqldata/mysql_3306/tmp/#1610617122 (deleted)
      0xfff0c7390000     0xfff0c7490000   0x100000        0x0 /mysqldata/mysql_3306/tmp/#1610617111 (deleted)
      0xfff0c7a60000     0xfff0c7b60000   0x100000        0x0 /mysqldata/mysql_3306/tmp/#1610613237 (deleted)
  

``` 
    
    
    污染位置对应的内存映射项为: /mysqldata/mysql_3306/tmp/#1610616875

# 分析

故障原理: 

  1. temptable的行表中, 有一部分内存被污染成了 0xa
  2. 污染位置对应的内存映射项 均为 通过mmap映射将文件到内存 (均不是匿名映射)
  3. 污染区域均为整块 (8kb倍数)

    
    
     

检查temptable中与mmap相关的代码, 发现: 

![image2023-1-11 17:44:13.png](/assets/01KJBYXQZ8AJVT5AG419NGRPC7/image2023-1-11%2017%3A44%3A13.png)

在temptable机制中, 当开启temptable_use_mmap时, 会生成临时文件, 用0xa充填 临时文件, 并将临时文件通过mmap映射到内存地址

与故障原理匹配: 

  1. 某个 临时表 使用mmap映射文件时, 污染到了其他临时表的内存中
  2. 临时表 访问 行数据时, 访问到了 被污染的内存, 根据被污染的内存 进行指针跳转时, 会访问非法地址, 导致崩溃

目前触发内存污染的动作 尚未复现

# 运维建议

可以关闭 temptable_use_mmap 参数, 直接使用 磁盘临时表 机制 处理相关SQL

关闭 temptable_use_mmap 参数后, temptable不会走到mmap机制, 不会用0xa充填临时文件, 即不会触发相关内存污染

实施变更前: 需要对相关SQL进行性能比较, 性能差异不大即可变更
