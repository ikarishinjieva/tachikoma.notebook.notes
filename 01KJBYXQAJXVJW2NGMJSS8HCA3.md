---
title: 20230128 - 农行crash信息整理并移交
confluence_page_id: 2130479
created_at: 2023-01-28T01:47:25+00:00
updated_at: 2023-01-28T01:47:25+00:00
---

# 基本信息

工单:

  - <https://support.actionsky.com/service_desk/browse/BEIJ-3252>
  - <https://support.actionsky.com/service_desk/browse/BEIJ-3260>

调试环境: 

  - [124.71.201.184](<http://124.71.201.184/>) root/actionsky2020!

  - docker exec -it huangyan2 bash

    - source /opt/rh/gcc-toolset-11/enable

    - gdb /usr/sbin/mysqld /opt/huangyan/0105/core-mysqld.log

    - gdb /usr/sbin/mysqld /opt/huangyan/0109/core-d01-01091518.log
  - 生产环境为arm环境, 调试环境已经与生产环境一致

有两次崩溃, 1月5日和1月9日, 分别由coredump

# 分析1月5日coredump

![image](http://8.134.54.170:8330/download/attachments/2130454/image2023-1-19%2019%3A6%3A41.png?version=1&modificationDate=1674126434000&api=v2)

获得THD的地址: 0xfff05801ab90

获取完整的问题SQL: 

```
(gdb) printf "%s", ((THD*)0xfff05801ab90)->m_query_string.str
SELECT t.ID_USER_NEWS,
        t.ID_USER, t.ID_CONTENT, t.TIME_CREATE, t.TYPE ,t.TITLE_ID,
        SUBSTRING(t.c,1,locate('&^%1',t.c)-1) title,
        SUBSTRING(t.c,locate('&^%1',t.c)+4,locate('&^%2',t.c)-locate('&^%1',t.c)-4) content,
        SUBSTRING(t.c,locate('&^%2',t.c)+4,locate('&^%3',t.c)-locate('&^%2',t.c)-4) portraid_id,
        SUBSTRING(t.c,locate('&^%3',t.c)+4,locate('&^%4',t.c)-locate('&^%3',t.c)-4) nick_name,
        SUBSTRING(t.c,locate('&^%4',t.c)+4) source_id
from (
        select
        n.ID_USER_NEWS, n.ID_USER, n.ID_CONTENT, n.TIME_CREATE, n.TYPE,
 
        CASE WHEN TYPE like '0%' THEN
        (select ques.QUESTION_ID from DT_ANSWERS answer
        ,DT_QUESTIONS ques,DT_USERS u
        where
        n.ID_CONTENT=answer.ANSWER_ID and answer.QUESTION_ID=ques.QUESTION_ID
        and u.ID_USER=answer.USER_ID)
 
        WHEN TYPE = '22' THEN
        (select blog.ID_BLOG_META from DT_BLOG_META blog,DT_USERS u,DT_BLOG_COMMENT bcm
        where
        n.ID_CONTENT=bcm.ID_BLOG_COMMENT and blog.ID_BLOG_META=bcm.ID_BLOG_META
        and u.ID_USER=blog.ID_USER)
        ELSE '' end as title_id,
 
        CASE WHEN TYPE like '0%' THEN
        (select CONCAT(ques.QUESTION_TITLE,'&^%1',answer.ANSWER_CONTENT,'&^%2',
        IFNULL(u.PIC_USER,''),'&^%3',IFNULL(u.NICK_USER,''),'&^%4') from DT_ANSWERS answer
        ,DT_QUESTIONS ques,DT_USERS u
        where
        n.ID_CONTENT=answer.ANSWER_ID and answer.QUESTION_ID=ques.QUESTION_ID
        and u.ID_USER=answer.USER_ID)
 
        WHEN TYPE like '1%' THEN
        (select CONCAT(ques.QUESTION_TITLE,'&^%1',ques.QUESTION_DES,'&^%2','&^%3','&^%4') from
        DT_QUESTIONS ques
        where n.ID_CONTENT=ques.QUESTION_ID)
 
 
        WHEN TYPE = '21' THEN
        (select CONCAT(blog.TITLE,'&^%1',
        IFNULL(bc.CONTENT,''),'&^%2',IFNULL(u.PIC_USER,''),'&^%3',IFNULL(u.NICK_USER,''),'&^%4') from
        DT_BLOG_META blog,DT_BLOG_CONT bc,DT_USERS u
        where
        n.ID_CONTENT=blog.ID_BLOG_META and blog.ID_BLOG_CONT=bc.ID_BLOG_CONT
        and u.ID_USER=blog.ID_USER and blog.IF_BLOCK='0')
 
        WHEN TYPE = '25' THEN
        (select CONCAT(blog.TITLE,'&^%1',
        IFNULL(bc.CONTENT,''),'&^%2',IFNULL(u.PIC_USER,''),'&^%3',IFNULL(u.NICK_USER,''),'&^%4') from
        DT_BLOG_META blog,DT_BLOG_CONT bc,DT_USERS u
        where
        n.ID_CONTENT=blog.ID_BLOG_META and blog.ID_BLOG_CONT=bc.ID_BLOG_CONT
        and u.ID_USER=blog.ID_USER and blog.IF_BLOCK='0')
 
        WHEN TYPE = '22' THEN
        (select CONCAT(blog.TITLE,'&^%1',
        bcm.CONTENT_COMMENT,'&^%2',IFNULL(u.PIC_USER,''),'&^%3',IFNULL(u.NICK_USER,''),'&^%4') from
        DT_BLOG_META blog,DT_USERS u,DT_BLOG_COMMENT bcm
        where
        n.ID_CONTENT=bcm.ID_BLOG_COMMENT and blog.ID_BLOG_META=bcm.ID_BLOG_META
        and u.ID_USER=blog.ID_USER and blog.IF_BLOCK='0')
 
        WHEN TYPE = '23' THEN
        (select CONCAT(blog.TITLE,'&^%1',
        IFNULL(bc.CONTENT,''),'&^%2',IFNULL(u.PIC_USER,''),'&^%3',IFNULL(u.NICK_USER,''),'&^%4') from
        DT_BLOG_META blog,DT_BLOG_CONT bc,DT_USERS u
        where
        n.ID_CONTENT=blog.ID_BLOG_META and blog.ID_BLOG_CONT=bc.ID_BLOG_CONT
        and u.ID_USER=blog.ID_USER and blog.IF_BLOCK='0')
 
        WHEN TYPE = '24' THEN
        (select CONCAT(blog.TITLE,'&^%1',
        IFNULL(bc.CONTENT,''),'&^%2',IFNULL(u.PIC_USER,''),'&^%3',IFNULL(u.NICK_USER,''),'&^%4') from
        DT_BLOG_META blog,DT_BLOG_CONT bc,DT_USERS u
        where
        n.ID_CONTENT=blog.ID_BLOG_META and blog.ID_BLOG_CONT=bc.ID_BLOG_CONT
        and u.ID_USER=blog.ID_USER and blog.IF_BLOCK='0')
 
        WHEN TYPE like '3%' THEN
        (select CONCAT(IFNULL(app.NAM_APP_CN,''),'&^%1',
        '&^%2',IFNULL(app.APP_LOGO_PIC,''),'&^%3','&^%4') from DT_APPS_REGISTER app
        where n.ID_CONTENT=app.APP_ID)
 
        WHEN TYPE like '4%' THEN
        (select CONCAT(IFNULL(u.NICK_USER,''),'&^%1',
        '&^%2',IFNULL(u.PIC_USER,''),'&^%3','&^%4') from DT_USERS u
        where
        n.ID_CONTENT=u.ID_USER)
 
        WHEN TYPE like '5%' THEN
        (select CONCAT(IFNULL(news.TITLE_NEWS,''),'&^%1',
        IFNULL(news.ABSTRACT_NEWS,''),'&^%2',IFNULL(news.PIC_NEWS,''),
        '&^%3','&^%4') from DT_NEWS
        news
        where n.ID_CONTENT=news.ID_NEWS)
 
        WHEN TYPE like '6%' THEN
        (select CONCAT(t.NAME_TAG,'&^%1',
        '&^%2',IFNULL(t.PIC_TAG,''),'&^%3','&^%4') from DT_TAGS t
        where n.ID_CONTENT=t.ID_TAG)
 
        WHEN TYPE like '7%' THEN
        (SELECT CONCAT(IFNULL(channel.TITLE_CHANNEL,''),'&^%1',
        '&^%2',IFNULL(channel.PIC_CHANNEL,''),'&^%3','&^%4')
        from DT_CHANNELS channel
        where n.ID_CONTENT=channel.ID_CHANNEL)
 
        WHEN TYPE like '8%' THEN
        (select CONCAT(t.NAME_TAG,'&^%1','&^%2',
        IFNULL(t.PIC_TAG,''),'&^%3','&^%4') from DT_TAGS t
        where n.ID_CONTENT=t.ID_TAG)
 
        WHEN TYPE LIKE '9%'
        THEN
            (
                SELECT
                    CONCAT('发表了想法','&^%1',idea.CONTENT,'&^%2','&^%3','&^%4',IFNULL(idea.SOURCE_ID,''))
                FROM
                    DT_IDEA idea
                WHERE
                    n.ID_CONTENT = idea.ID_IDEA AND idea.IF_BLOCK='0'
            )
        end as c
    from DT_USER_NEWS n
 
    where
        n.ID_USER = 'e9ab3894a71a45c29bbcfb45fb7782ac'
)t
 
ORDER BY t.TIME_CREATE DESC limit 0 , 10
``` 

TODO: 联系启超安排复现

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

遍历temptable所有页和行, 找到问题行所在位置

```
//遍历页
  
//第一页
set $first_page=(((temptable::Handler*)0xfff0580ff598)->m_opened_table)->m_rows->m_first_page
  
set $page=$first_page
  
//以下部分不断循环, 遍历每一页的第一个row和最后一个row的block
set $first_possible_element_on_page=(static_cast<uint8_t*>($page)+sizeof(Storage::Page*))
set $first_row=*static_cast<const temptable::Row *>($first_possible_element_on_page)
set $last_possible_element_on_page=(static_cast<uint8_t*>($page)+sizeof(Storage::Page*)+32*2046)
set $last_row=*static_cast<const temptable::Row *>($first_possible_element_on_page)
  
set $row=$first_row
set $chunk=reinterpret_cast<uint8_t *>($row.m_ptr) - sizeof(Chunk::metadata_type)
set $block=$chunk - static_cast<size_t>(*reinterpret_cast<Chunk::metadata_type *>($chunk))
set $block_size=static_cast<size_t>(*reinterpret_cast<Header::metadata_type *>($block + 1 * sizeof(Header::metadata_type)))
set $block_end=$block+$block_size
p $block
p $block_end
  
set $row=$last_row
set $chunk=reinterpret_cast<uint8_t *>($row.m_ptr) - sizeof(Chunk::metadata_type)
set $block=$chunk - static_cast<size_t>(*reinterpret_cast<Chunk::metadata_type *>($chunk))
set $block_size=static_cast<size_t>(*reinterpret_cast<Header::metadata_type *>($block + 1 * sizeof(Header::metadata_type)))
set $block_end=$block+$block_size
p $block
p $block_end
set $next_page=*reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>($page) + sizeof(Storage::Page *) +32 * 2047)
set $page=$next_page
``` 

问题行所在位置为 temptable的第7页第1509行

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

![image](http://8.134.54.170:8330/download/attachments/2130442/image2023-1-11%2017%3A23%3A16.png?version=1&modificationDate=1673430982000&api=v2)

可以看到数据是正常业务数据 (HTML片段), 到内存异常位置 0xffee61490000 被截断

确定内存异常位置的下界 (0xa 持续到哪里)

![image](http://8.134.54.170:8330/download/attachments/2130442/image2023-1-11%2017%3A24%3A44.png?version=1&modificationDate=1673430982000&api=v2)

超过异常位置下界后, 数据恢复正常, 是另一端截断的HTML代码

确定内存污染范围为: 0xffee61490000 - 0xffee61570000, 共 896kb

(1月9日coredump的污染区域: 0xffe93d060000 - 0xffe93d070000, 共64kb)

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

//污染位置对应的映射项: /mysqldata/mysql_3306/tmp/#1610612913
``` 

# 判断

0xa的区域, 疑似是 temptable_use_mmap 的作用 (也有可能是误判), 临时表mmap时会用0xa初始化内存: 

![image2023-1-28 9:43:29.png](/assets/01KJBYXQAJXVJW2NGMJSS8HCA3/image2023-1-28%209%3A43%3A29.png)

但temptable_use_mmap的初始化最小块为1M

而污染区域都小于1MiB, 有两种可能: 

  - 0xa 区域, 不是由temptable mmap机制写进来的
  - 在block内存使用的过程中, 刚好避开了故障区域 (也就是说, 故障区域其实是未使用区域)
