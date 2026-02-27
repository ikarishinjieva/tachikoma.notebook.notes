---
title: 20230119 - 农行crash分析整理 (20230111续)
confluence_page_id: 2130454
created_at: 2023-01-19T10:38:27+00:00
updated_at: 2023-01-20T09:55:55+00:00
---

# 前继

[20230111 - 农行crash分析整理]

工单:

  - <https://support.actionsky.com/service_desk/browse/BEIJ-3252>
  - <https://support.actionsky.com/service_desk/browse/BEIJ-3260>

调试环境: 

  - [124.71.201.184](<http://124.71.201.184/>) root/actionsky2020!

# 分析1月5日coredump

崩溃现场

![image2023-1-19 19:6:41.png](/assets/01KJBYXQQS6MFV2NVNDXSN404C/image2023-1-19%2019%3A6%3A41.png)

获得THD的地址: 0xfff05801ab90
    
    
     

获得完整的问题SQL

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
```
//问题 Row
 
(gdb) p static_cast<const temptable::Row *>(((temptable::Handler*)0xfff0580ff598)->m_rnd_iterator->m_element)
$3 = (const temptable::Row *) 0xffee59188550
 
//m_ptr 内容为 0xa 非法数据
(gdb) p *((const temptable::Row *) 0xffee59188550)
$6 = {m_allocator = 0xfff05802dbe8, m_data_is_in_mysql_memory = false, m_ptr = 0xffee61500258 '\n' <repeats 200 times>...}
 
//allocator 为 0xfff05802dbe8
 
(gdb) p *((const temptable::Row *) 0xffee59188550)->m_allocator
$11 = {<temptable::MemoryMonitor> = {static ram = {<std::__atomic_base<unsigned long>> = {static _S_alignment = 8, _M_i = 838860800}, <No data fields>}},
  m_state = std::shared_ptr<temptable::AllocatorState> (use count 4, weak count 0) = {get() = 0xfff2282af460}}
 
//allocator->m_state, 查看当前block
 
(gdb) p ((const temptable::Row *) 0xffee59188550)->m_allocator->m_state
$5 = std::shared_ptr<temptable::AllocatorState> (use count 4, weak count 0) = {get() = 0xfff2282af460}
(gdb) p *((const temptable::Row *) 0xffee59188550)->m_allocator->m_state
$6 = {current_block = {<temptable::Header> = {static SIZE = 32, m_offset = 0xffeeec000000 ""}, static ALIGN_TO = 8}, number_of_blocks = 12}
 
//shared block
(gdb) p shared_block
$11 = {<temptable::Header> = {static SIZE = 32, m_offset = 0xfff6d1320000 "\001"}, static ALIGN_TO = 8}
 
//尝试从 row->ptr 推导出 block 的起始位置
 
(gdb) p static_cast<const temptable::Row *>(0xffee5917c8b0 + 1508*32)->m_ptr
$6 = (unsigned char *) 0xffee61489720 "0k\a"
 
//Chunk::Chunk(void *data), 计算 Chunk.m_offset
(gdb) p reinterpret_cast<uint8_t *>(0xffee61489720) - sizeof(Chunk::metadata_type)
$7 = (uint8_t *) 0xffee61489718 "\030\227H\025"
 
//Chunk::block
(gdb) p/x 0xffee61489718 - static_cast<size_t>(*reinterpret_cast<Chunk::metadata_type *>(0xffee61489718))
$10 = 0xffee4c000000
 
//简化脚本: 
set $row=static_cast<const temptable::Row *>(0xffee5917c8b0+1508*32)->m_ptr
set $chunk=reinterpret_cast<uint8_t *>($row) - sizeof(Chunk::metadata_type)
set $block=$chunk - static_cast<size_t>(*reinterpret_cast<Chunk::metadata_type *>($chunk))
set $block_size=static_cast<size_t>(*reinterpret_cast<Header::metadata_type *>($block + 1 * sizeof(Header::metadata_type)))
set $block_end=$block+$block_size
p $row
p $chunk
p $block
p/x $block_size
p/x $block_end
 
//样例: 
(gdb) set $row=static_cast<const temptable::Row *>(0xffee5917c8b0+1508*32)->m_ptr
(gdb) set $chunk=reinterpret_cast<uint8_t *>($row) - sizeof(Chunk::metadata_type)
(gdb) set $block=$chunk - static_cast<size_t>(*reinterpret_cast<Chunk::metadata_type *>($chunk))
(gdb) set $block_size=static_cast<size_t>(*reinterpret_cast<Header::metadata_type *>($block + 1 * sizeof(Header::metadata_type)))
(gdb) set $block_end=$block+$block_size
(gdb) set $block_first_pristine_offset=static_cast<size_t>(*reinterpret_cast<Header::metadata_type *>($block + 3 * sizeof(Header::metadata_type)))
(gdb) p $row
$7 = (unsigned char *) 0xffee61489720 "0k\a"
(gdb) p $chunk
$8 = (uint8_t *) 0xffee61489718 "\030\227H\025"
(gdb) p $block
$9 = (uint8_t *) 0xffee4c000000 ""
(gdb) p/x $block_size
$10 = 0x20000000
(gdb) p/x $block_end
$11 = 0xffee6c000000
(gdb) p/x $block_first_pristine_offset
$12 = 0x1fffd808
 
// 故障block的尾部第一个空闲块

(gdb) p/x $block+$block_first_pristine_offset
$13 = 0xffee6bffd808 
 
``` 

通过之前的分析, 已知: 

  - 造成崩溃的内存地址在 0xffee61500258 附近
  - 确定内存污染范围为: 0xffee61490000 - 0xffee61570000, 共 896kb

需判断内存污染 除了在故障block中, 还在 哪个block里: 

TODO: 需按照 [20230109 - 农行crash] 逐个检查 同表的page和row, 还需要检查不同表的内存地址

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

# 分析1月9日coredump

![image2023-1-19 19:12:16.png](/assets/01KJBYXQQS6MFV2NVNDXSN404C/image2023-1-19%2019%3A12%3A16.png)

获取问题SQL: 

```
(gdb) printf "%s", ((THD*)0xfff0ec00ef00)->m_query_string.str
SELECT COUNT(*) AS rowcou  from (

        SELECT
         n.ID_USER_NEWS, n.ID_USER, n.ID_CONTENT, n.TIME_CREATE, n.TYPE,

        CASE WHEN TYPE like '0%' THEN
         ( SELECT ques.QUESTION_ID from DT_ANSWERS answer
        ,DT_QUESTIONS ques,DT_USERS u
        where
        n.ID_CONTENT=answer.ANSWER_ID and answer.QUESTION_ID=ques.QUESTION_ID
        and u.ID_USER=answer.USER_ID)

        WHEN TYPE = '22' THEN
         ( SELECT blog.ID_BLOG_META from DT_BLOG_META blog,DT_USERS u,DT_BLOG_COMMENT bcm
        where
        n.ID_CONTENT=bcm.ID_BLOG_COMMENT and blog.ID_BLOG_META=bcm.ID_BLOG_META
        and u.ID_USER=blog.ID_USER)
        ELSE '' end as title_id,

        CASE WHEN TYPE like '0%' THEN
         ( SELECT CONCAT(ques.QUESTION_TITLE,'&^%1',answer.ANSWER_CONTENT,'&^%2',
        IFNULL(u.PIC_USER,''),'&^%3',IFNULL(u.NICK_USER,''),'&^%4') from DT_ANSWERS answer
        ,DT_QUESTIONS ques,DT_USERS u
        where
        n.ID_CONTENT=answer.ANSWER_ID and answer.QUESTION_ID=ques.QUESTION_ID
        and u.ID_USER=answer.USER_ID)

        WHEN TYPE like '1%' THEN
         ( SELECT CONCAT(ques.QUESTION_TITLE,'&^%1',ques.QUESTION_DES,'&^%2','&^%3','&^%4') FROM
         DT_QUESTIONS ques
        where n.ID_CONTENT=ques.QUESTION_ID)

        WHEN TYPE = '21' THEN
         ( SELECT CONCAT(blog.TITLE,'&^%1',
        IFNULL(bc.CONTENT,''),'&^%2',IFNULL(u.PIC_USER,''),'&^%3',IFNULL(u.NICK_USER,''),'&^%4') FROM
         DT_BLOG_META blog,DT_BLOG_CONT bc,DT_USERS u
        where
        n.ID_CONTENT=blog.ID_BLOG_META and blog.ID_BLOG_CONT=bc.ID_BLOG_CONT
        and u.ID_USER=blog.ID_USER and blog.IF_BLOCK='0')

        WHEN TYPE = '25' THEN
         ( SELECT CONCAT(blog.TITLE,'&^%1',
        IFNULL(bc.CONTENT,''),'&^%2',IFNULL(u.PIC_USER,''),'&^%3',IFNULL(u.NICK_USER,''),'&^%4') FROM
         DT_BLOG_META blog,DT_BLOG_CONT bc,DT_USERS u
        where
        n.ID_CONTENT=blog.ID_BLOG_META and blog.ID_BLOG_CONT=bc.ID_BLOG_CONT
        and u.ID_USER=blog.ID_USER and blog.IF_BLOCK='0')

        WHEN TYPE = '22' THEN
         ( SELECT CONCAT(blog.TITLE,'&^%1',
        bcm.CONTENT_COMMENT,'&^%2',IFNULL(u.PIC_USER,''),'&^%3',IFNULL(u.NICK_USER,''),'&^%4') FROM
         DT_BLOG_META blog,DT_USERS u,DT_BLOG_COMMENT bcm
        where
        n.ID_CONTENT=bcm.ID_BLOG_COMMENT and blog.ID_BLOG_META=bcm.ID_BLOG_META
        and u.ID_USER=blog.ID_USER and blog.IF_BLOCK='0')

        WHEN TYPE = '23' THEN
         ( SELECT CONCAT(blog.TITLE,'&^%1',
        IFNULL(bc.CONTENT,''),'&^%2',IFNULL(u.PIC_USER,''),'&^%3',IFNULL(u.NICK_USER,''),'&^%4') FROM
         DT_BLOG_META blog,DT_BLOG_CONT bc,DT_USERS u
        where
        n.ID_CONTENT=blog.ID_BLOG_META and blog.ID_BLOG_CONT=bc.ID_BLOG_CONT
        and u.ID_USER=blog.ID_USER and blog.IF_BLOCK='0')

        WHEN TYPE = '24' THEN
         ( SELECT CONCAT(blog.TITLE,'&^%1',
        IFNULL(bc.CONTENT,''),'&^%2',IFNULL(u.PIC_USER,''),'&^%3',IFNULL(u.NICK_USER,''),'&^%4') FROM
         DT_BLOG_META blog,DT_BLOG_CONT bc,DT_USERS u
        where
        n.ID_CONTENT=blog.ID_BLOG_META and blog.ID_BLOG_CONT=bc.ID_BLOG_CONT
        and u.ID_USER=blog.ID_USER and blog.IF_BLOCK='0')

        WHEN TYPE like '3%' THEN
         ( SELECT CONCAT(IFNULL(app.NAM_APP_CN,''),'&^%1',
        '&^%2',IFNULL(app.APP_LOGO_PIC,''),'&^%3','&^%4') from DT_APPS_REGISTER app
        where n.ID_CONTENT=app.APP_ID)

        WHEN TYPE like '4%' THEN
         ( SELECT CONCAT(IFNULL(u.NICK_USER,''),'&^%1',
        '&^%2',IFNULL(u.PIC_USER,''),'&^%3','&^%4') from DT_USERS u
        where
        n.ID_CONTENT=u.ID_USER)

        WHEN TYPE like '5%' THEN
         ( SELECT CONCAT(IFNULL(news.TITLE_NEWS,''),'&^%1',
        IFNULL(news.ABSTRACT_NEWS,''),'&^%2',IFNULL(news.PIC_NEWS,''),
        '&^%3','&^%4') from DT_NEWS
        news
        where n.ID_CONTENT=news.ID_NEWS)

        WHEN TYPE like '6%' THEN
         ( SELECT CONCAT(t.NAME_TAG,'&^%1',
        '&^%2',IFNULL(t.PIC_TAG,''),'&^%3','&^%4') from DT_TAGS t
        where n.ID_CONTENT=t.ID_TAG)

        WHEN TYPE like '7%' THEN
         ( SELECT CONCAT(IFNULL(channel.TITLE_CHANNEL,''),'&^%1',
        '&^%2',IFNULL(channel.PIC_CHANNEL,''),'&^%3','&^%4')
        from DT_CHANNELS channel
        where n.ID_CONTENT=channel.ID_CHANNEL)

        WHEN TYPE like '8%' THEN
         ( SELECT CONCAT(t.NAME_TAG,'&^%1','&^%2',
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

        from DT_USER_NEWS n where
        
        n.ID_USER = 'e9ab3894a71a45c29bbcfb45fb7782ac'
)t
``` 

梳理故障内存前后区域对应的行/列: 

```
//污染位置: 0xffe93d060000 - 0xffe93d070000
 
set $first_page=(((temptable::Handler*)0xfff0ec163a78)->m_opened_table)->m_rows->m_first_page

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
 
 
//找到涉及污染区域的页: 
 
(gdb) p $block
$81 = (uint8_t *) 0xffe935f00000 ""
(gdb) p $block_end
$82 = (uint8_t *) 0xffe955f00000 ""
...
(gdb) p $page
$77 = (temptable::Storage::Page *) 0xffe9397e8450 //页1
...
(gdb) p $page
$83 = (temptable::Storage::Page *) 0xffe9427a5560 //页2
 
//找到行的数据地址, 不断增加row_idx, 找到污染区域
set $page=0xffe9397e8450
set $row_idx=0
set $element_on_page=(static_cast<uint8_t*>($page)+sizeof(Storage::Page*)+ $row_idx*32)
set $row=*static_cast<const temptable::Row *>($element_on_page)
set $cells=reinterpret_cast<Cell *>($row.m_ptr+sizeof(size_t))
p $cells->m_data
 
 
(gdb) p $cells->m_data
// 728行
0xffe93d04a088
//729行
0xffe93d055238, 距离上一行 45488
// 730行
0xffe93d05d268, 距离上一行 32816
// 731行
0xffe93d064ec8 (污染), 距离上一行 31840
// 732行
0xffe93d095840, 跨过了污染区?? , 距离上一行 199032
 
``` 

检查0105的coredump, 验证跨过污染区的结论: 

```
//脚本跟上部分一致
//污染区域: 0xffee61490000 - 0xffee61570000

set $page=0xffee5917c8a8
set $row_idx=1508
set $element_on_page=(static_cast<uint8_t*>($page)+sizeof(Storage::Page*)+ $row_idx*32)
set $row=*static_cast<const temptable::Row *>($element_on_page)
set $cells=reinterpret_cast<Cell *>($row.m_ptr+sizeof(size_t))
p $cells
p $cells->m_data
 
(gdb) p $cells->m_data
//1507行: 0xffee61481ce0
//1508行: 0xffee61489798, 距离上一行 31416
//1509行: 0xffee61500268 (污染), 距离上一行 486096
//1510行: 0xffee61507728 (污染), 距离上一行 29888
//1511行: 0xffee61552f00 (污染), 距离上一行 309208
//1512行: 0xffee61566040 (污染), 距离上一行 78144
//1513行: 0xffee6157eb78, 距离上一行 101176
//1514行: 0xffee61596718
 
``` 

# 线索1

查阅8.0.18以后, 与temptable相关的commit, 找到线索1

```
commit 9d630eea67fa642ab56229c80d9480fe4f6fae70
Author: Aditya A <aditya.a@oracle.com>
Date:   Mon May 11 16:57:17 2020 +0530

    Bug #30562964 8.0.18: PERFORMANCE REGRESSION IN SELECT DISTINCT

    PROBLEM

    1) In temptable engine, memory is allocated in blocks which is of fixed size
       . whenever a request comes for memory we try to allocate it from a block
       by means of chunk.

    2) Chunk within the block contains a fixed size metadata and then the data of requested
       size.

    3) So a block is nothing but a series of chunks. Once the block size is reached
       and we cannot allocate chunks from it ,we go for new block creation.

    4) Each block maintains a metadata which stores the next free offset within the block
       for chunk allocation. It is called the first_pristine_pointer.

    5) While deallcoating memory we pass the chunk to the deallocate function
       where a optimization exists which tries to reuse the memory of chunk
       if it is the rightmost in the block. We do this by reducing the first_pristine_pointer
       by the chunk size

    6) When checking if the given chunk is the rightmost chunk we must know the starting
       offset of the chunk and the size of the chunk,when these two are added we
       should get the offset of first_pristine_pointer

    7) There was a bug in the code where the size of chunk was not calculated correctly
       we only took the data part of the chunk and didn't consider the metadata part,
       because of which it couldn't recognize if the chunk was the rightmost chunk and
       missed the optimization of memory reuse. This causes the Temptable engine to
       allocate more blocks to accommodate the requests which is the cause of this
       performance regression.

``` 

# 线索2

![image2023-1-20 17:53:33.png](/assets/01KJBYXQQS6MFV2NVNDXSN404C/image2023-1-20%2017%3A53%3A33.png)

block 内存分配的最小值为 1MiB, 倍数增加. 

而污染区域都小于1MiB, 有两种可能: 

  - 0xa 区域, 不是由temptable mmap机制写进来的
  - 在block内存使用的过程中, 刚好避开了故障区域 (也就是说, 故障区域其实是未使用区域)
