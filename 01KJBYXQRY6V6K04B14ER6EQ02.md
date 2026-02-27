---
title: 20230109 - 农行crash
confluence_page_id: 2130405
created_at: 2023-01-09T15:26:16+00:00
updated_at: 2023-01-11T08:24:12+00:00
---

# 测试环境

124.71.201.184 root/actionsky2020!

# 第一次: [BEIJ-3252](<https://support.actionsky.com/service_desk/browse/BEIJ-3252>)

<https://support.actionsky.com/service_desk/browse/BEIJ-3252>

![image2023-1-9 22:39:22.png](/assets/01KJBYXQRY6V6K04B14ER6EQ02/image2023-1-9%2022%3A39%3A22.png)

环境为arm, 在arm服务器上, 使用指定版本容器进行调试环境搭建

已确认调试环境与报错中的函数签名和函数地址一致

### 堆栈位置 0x2043d60

mysql-8.0.18/storage/temptable/include/temptable/table.h, line 212.

![image2023-1-9 22:43:33.png](/assets/01KJBYXQRY6V6K04B14ER6EQ02/image2023-1-9%2022%3A43%3A33.png)

### 崩溃位置 0x2046a9c

/usr/include/c++/8/bits/stl_vector.h, line 805

### 参考: <https://tracker.ceph.com/issues/39174>

![image2023-1-9 22:54:0.png](/assets/01KJBYXQRY6V6K04B14ER6EQ02/image2023-1-9%2022%3A54%3A0.png)

其描述了 stl_vector.h:805, 是vector的取数操作符[], 其内部逻辑存在对 "取数位置 < 数组长度"的断言, 如果断言失败, 则触发崩溃

第二次: BEIJ-3260

![image2023-1-9 23:29:8.png](/assets/01KJBYXQRY6V6K04B14ER6EQ02/image2023-1-9%2023%3A29%3A8.png)

# 第二次: BEIJ-3260

![image2023-1-9 23:29:8.png](/assets/01KJBYXQRY6V6K04B14ER6EQ02/image2023-1-9%2023%3A29%3A8.png)

# Coredump分析

获取了coredump, 更新了故障堆栈: 

![image2023-1-10 13:48:47.png](/assets/01KJBYXQRY6V6K04B14ER6EQ02/image2023-1-10%2013%3A48%3A47.png)

frame 10, temptable::handler

```
p *((temptable::Handler*)0xfff0580ff598)
$50 = {<handler> = {_vptr.handler = 0x3386df8 <vtable for temptable::Handler+16>, table_share = 0xfff0581b94f8, table = 0xfff0581b8b88, cached_table_flags = 4303365297,
    estimation_rows_to_insert = 0, ht = 0xfc5f790, ref = 0xfff0580ea798 "0\205\030Y\356\377", dup_ref = 0xfff0580ea7a0 "", stats = {data_file_length = 0,
      max_data_file_length = 0, index_file_length = 0, max_index_file_length = 16777216, delete_length = 0, auto_increment_value = 0, records = 26967, deleted = 0,
      mean_rec_length = 0, create_time = 0, check_time = 0, update_time = 0, block_size = 0, mrr_length_per_rec = 1500607027, table_in_mem_estimate = 1},
    mrr_iter = 0x46636a616d533000, mrr_funcs = {init = 0x2b000c3d4d702b49, next = 0x5532544b78526d6b, skip_record = 0x6b4935000c3d3034,
...
 
 
``` 

![image2023-1-10 14:22:20.png](/assets/01KJBYXQRY6V6K04B14ER6EQ02/image2023-1-10%2014%3A22%3A20.png)

m_opened_table

```
(gdb) p *(((temptable::Handler*)0xfff0580ff598)->m_opened_table)
$56 = {m_allocator = {<temptable::MemoryMonitor> = {static ram = {<std::__atomic_base<unsigned long>> = {static _S_alignment = 8, _M_i = 838860800}, <No data fields>}},
    m_state = std::shared_ptr<temptable::AllocatorState> (use count 4, weak count 0) = {get() = 0xfff2282af460}}, m_rows = {static ALIGN_TO = 8,
    static ELEMENT_FIRST_ON_PAGE = 1 '\001', static ELEMENT_LAST_ON_PAGE = 2 '\002', static ELEMENT_DELETED = 4 '\004', static META_BYTES_PER_ELEMENT = 1,
    static META_BYTES_PER_PAGE = 16, m_allocator = 0xfff05802dbe8, m_element_size = 24, m_bytes_used_per_element = 32, m_number_of_elements_per_page = 2047,
    m_number_of_elements = 26967, m_first_page = 0xfff6d13200a0, m_last_page = 0xffeef88a5560, m_last_element = 0xffeef88a81c8}, m_all_columns_are_fixed_size = false,
  m_indexes_are_enabled = true, m_mysql_row_length = 552, m_index_entries = std::vector of length 0, capacity 0, m_insert_undo = std::vector of length 0, capacity 0,
  m_columns = std::vector of length 7, capacity 7 = {{m_nullable = false, m_is_blob = false, m_null_bitmask = 0 '\000', m_length_bytes_size = 1 '\001', {m_length = 1,
        m_offset = 1}, m_null_byte_offset = 0, m_user_data_offset = 2}, {m_nullable = false, m_is_blob = false, m_null_bitmask = 0 '\000', m_length_bytes_size = 1 '\001', {
        m_length = 130, m_offset = 130}, m_null_byte_offset = 0, m_user_data_offset = 131}, {m_nullable = false, m_is_blob = false, m_null_bitmask = 0 '\000',
      m_length_bytes_size = 1 '\001', {m_length = 259, m_offset = 259}, m_null_byte_offset = 0, m_user_data_offset = 260}, {m_nullable = false, m_is_blob = false,
      m_null_bitmask = 0 '\000', m_length_bytes_size = 0 '\000', {m_length = 5, m_offset = 5}, m_null_byte_offset = 0, m_user_data_offset = 388}, {m_nullable = false,
      m_is_blob = false, m_null_bitmask = 0 '\000', m_length_bytes_size = 1 '\001', {m_length = 393, m_offset = 393}, m_null_byte_offset = 0, m_user_data_offset = 394}, {
      m_nullable = true, m_is_blob = false, m_null_bitmask = 1 '\001', m_length_bytes_size = 1 '\001', {m_length = 402, m_offset = 402}, m_null_byte_offset = 0,
      m_user_data_offset = 403}, {m_nullable = true, m_is_blob = true, m_null_bitmask = 2 '\002', m_length_bytes_size = 4 '\004', {m_length = 531, m_offset = 531},
      m_null_byte_offset = 0, m_user_data_offset = 535}}, m_mysql_table_share = 0xfff0581b94f8}
``` 

m_opened_table->rows()

```
(gdb) p (((temptable::Handler*)0xfff0580ff598)->m_opened_table)->m_rows
$78 = {static ALIGN_TO = 8, static ELEMENT_FIRST_ON_PAGE = 1 '\001', static ELEMENT_LAST_ON_PAGE = 2 '\002', static ELEMENT_DELETED = 4 '\004',
  static META_BYTES_PER_ELEMENT = 1, static META_BYTES_PER_PAGE = 16, m_allocator = 0xfff05802dbe8, m_element_size = 24, m_bytes_used_per_element = 32,
  m_number_of_elements_per_page = 2047, m_number_of_elements = 26967, m_first_page = 0xfff6d13200a0, m_last_page = 0xffeef88a5560, m_last_element = 0xffeef88a81c8}
``` 

m_rnd_iterator, 访问临时表的迭代器

```
(gdb) p ((temptable::Handler*)0xfff0580ff598)->m_rnd_iterator
$57 = {m_storage = 0xfff05802dbf8, m_element = 0xffee59188550}
(gdb)
``` 

frame 9: pos = m_rnd_iterator

![image2023-1-10 14:25:23.png](/assets/01KJBYXQRY6V6K04B14ER6EQ02/image2023-1-10%2014%3A25%3A23.png)

storage_element:

```
(gdb) p ((temptable::Handler*)0xfff0580ff598)->m_rnd_iterator->m_element
$63 = (temptable::Storage::Element *) 0xffee59188550
``` 

row: 

```
(gdb) p *static_cast<const temptable::Row *>(((temptable::Handler*)0xfff0580ff598)->m_rnd_iterator->m_element)
$2 = {m_allocator = 0xfff05802dbe8, m_data_is_in_mysql_memory = false, m_ptr = 0xffee61500258 '\n' <repeats 200 times>...}
``` 

frame 8: 

![image2023-1-10 14:54:16.png](/assets/01KJBYXQRY6V6K04B14ER6EQ02/image2023-1-10%2014%3A54%3A16.png)

![image2023-1-10 18:36:1.png](/assets/01KJBYXQRY6V6K04B14ER6EQ02/image2023-1-10%2018%3A36%3A1.png)

cells()

```
(gdb) p *reinterpret_cast<Cell *>(0xffee61500258+sizeof(size_t))
$8 = {m_is_null = 10, m_data_length = 168430090, m_data = 0xa0a0a0a0a0a0a0a <error: Cannot access memory at address 0xa0a0a0a0a0a0a0a>}
(gdb)
``` 

判断从 m_rnd_iterator 开始, 地址就开始错误.

尝试拿 table.rows.first_page: 

```
//first page
(gdb) p (((temptable::Handler*)0xfff0580ff598)->m_opened_table)->m_rows->m_first_page
$9 = (temptable::Storage::Page *) 0xfff6d13200a0
 
 
//Storage::first_possible_element_on_page
(gdb) p (static_cast<uint8_t*>(0xfff6d13200a0)+sizeof(Storage::Page*))
$12 = (uint8_t *) 0xfff6d13200a8 "\350\333\002X\360\377"
 
//转成row, Table::row
(gdb) p *static_cast<const temptable::Row *>(0xfff6d13200a8)
$13 = {m_allocator = 0xfff05802dbe8, m_data_is_in_mysql_memory = false, m_ptr = 0xfff6d1330098 "c\315\003"}
 
//转成cell, Row::Cells
(gdb) p *reinterpret_cast<Cell *>(0xfff6d1330098+sizeof(size_t))
$14 = {m_is_null = false, m_data_length = 6,
  m_data = 0xfff6d1330110 "585814e9ab3894a71a45c29bbcfb45fb7782ac40365\231\254F\221\270\062\061 \344\272\221\345\216\237\347\224\237\347\275\221\347\273\234\345\210\251\345\231\250--Cilium \346\200\273\350\247\210 &^%1{\"md\":\"<div class=\\\"rich_media_content \\\" id=\\\"js_content\\\" >  <p><strong style=\\\"font-family: mp-quote, -ap"...}
``` 

说明解析方法是没问题的, 可以看到具体数据

查找第二页: 

```
//Storage::page_next_page_ptr
(gdb) p *reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xfff6d13200a0) + sizeof(Storage::Page *) +32 * 2047)
$29 = (temptable::Storage::Page *) 0xffef253340d0

//Storage::first_possible_element_on_page
(gdb) p (static_cast<uint8_t*>(0xffef253340d0)+sizeof(Storage::Page*))
$30 = (uint8_t *) 0xffef253340d8 "\350\333\002X\360\377"

//转成row, Table::row
(gdb) p *static_cast<const temptable::Row *>(0xffef253340d8)
$33 = {m_allocator = 0xfff05802dbe8, m_data_is_in_mysql_memory = false, m_ptr = 0xffef253440c8 ")\205"}

//转成cell, Row::Cells
(gdb) p *reinterpret_cast<Cell *>(0xffef253440c8+sizeof(size_t))
$34 = {m_is_null = false, m_data_length = 6,
  m_data = 0xffef25344140 "604771e9ab3894a71a45c29bbcfb45fb7782ac44453\231\254\216\232(21 \344\275\240\350\277\230\345\234\250\347\224\250\345\210\206\351\241\265\357\274\237\350\257\225\350\257\225 MyBatis \346\265\201\345\274\217\346\237\245\350\257\242\357\274\214\347\234\237\345\277\203\345\274\272\345\244\247\357\274\201 &^%1{\"md\":\"<div class=\\\"rich_media_content", ' ' <repeats 40 times>...}
``` 

精简查询: 

```
//Storage::page_next_page_ptr

(gdb) p *reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xfff6d13200a0) + sizeof(Storage::Page *) +32 * 2047)
$29 = (temptable::Storage::Page *) 0xffef253340d0
 
 
//合并查询
(gdb) p *reinterpret_cast<Cell *>((static_cast<const temptable::Row *>((static_cast<uint8_t*>(0xffef253340d0)+sizeof(Storage::Page*)))->m_ptr+sizeof(size_t)))
$35 = {m_is_null = false, m_data_length = 6,
  m_data = 0xffef25344140 "604771e9ab3894a71a45c29bbcfb45fb7782ac44453\231\254\216\232(21 \344\275\240\350\277\230\345\234\250\347\224\250\345\210\206\351\241\265\357\274\237\350\257\225\350\257\225 MyBatis \346\265\201\345\274\217\346\237\245\350\257\242\357\274\214\347\234\237\345\277\203\345\274\272\345\244\247\357\274\201 &^%1{\"md\":\"<div class=\\\"rich_media_content", ' ' <repeats 40 times>...}
``` 

计算总页数: 

![image2023-1-11 0:3:5.png](/assets/01KJBYXQRY6V6K04B14ER6EQ02/image2023-1-11%200%3A3%3A5.png)

总页数 = 26967 / 2047 = 13.17

需要测试: 

```
//找到下一页
 
(gdb) p *reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xfff6d13200a0) + sizeof(Storage::Page *) +32 * 2047)
$74 = (temptable::Storage::Page *) 0xffef253340d0
 
//找到页的cells范围
(gdb) p static_cast<const temptable::Row *>((static_cast<uint8_t*>(*reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffef253340d0) + sizeof(Storage::Page *) +32 * 2047))+sizeof(Storage::Page*)))->m_ptr
$90 = (unsigned char *) 0xffeedea92260 "\033\274\001"
 
出问题的迭代器地址: 
(gdb) p ((temptable::Handler*)0xfff0580ff598)->m_rnd_iterator->m_element
$38 = (temptable::Storage::Element *) 0xffee59188550
 
需确定在哪个页上, 有何特征
 
---
 
命令: 

p *reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffeef88a5560) + sizeof(Storage::Page *) +32 * 2047)
p static_cast<const temptable::Row *>((static_cast<uint8_t*>(*reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffeef88a5560) + sizeof(Storage::Page *) +32 * 2047))+sizeof(Storage::Page*)))->m_ptr

第1页: 

(gdb) p *reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xfff6d13200a0) + sizeof(Storage::Page *) +32 * 2047)
$1 = (temptable::Storage::Page *) 0xffef253340d0
(gdb) p static_cast<const temptable::Row *>((static_cast<uint8_t*>(*reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xfff6d13200a0) + sizeof(Storage::Page *) +32 * 2047))+sizeof(Storage::Page*)))->m_ptr
$2 = (unsigned char *) 0xffef253440c8 ")\205"

第2页: 
(gdb) p *reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffef253340d0) + sizeof(Storage::Page *) +32 * 2047)
$8 = (temptable::Storage::Page *) 0xffeedea82268
(gdb) p static_cast<const temptable::Row *>((static_cast<uint8_t*>(*reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffef253340d0) + sizeof(Storage::Page *) +32 * 2047))+sizeof(Storage::Page*)))->m_ptr
$9 = (unsigned char *) 0xffeedea92260 "\033\274\001"

第3页: 
(gdb) p *reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffeedea82268) + sizeof(Storage::Page *) +32 * 2047)
$11 = (temptable::Storage::Page *) 0xffeee4b26b10
(gdb) p static_cast<const temptable::Row *>((static_cast<uint8_t*>(*reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffeedea82268) + sizeof(Storage::Page *) +32 * 2047))+sizeof(Storage::Page*)))->m_ptr
$12 = (unsigned char *) 0xffeee4b36b08 "\333"

第4页: 
(gdb) p *reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffeee4b26b10) + sizeof(Storage::Page *) +32 * 2047)
$13 = (temptable::Storage::Page *) 0xffeee5562d80
(gdb) p static_cast<const temptable::Row *>((static_cast<uint8_t*>(*reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffeee4b26b10) + sizeof(Storage::Page *) +32 * 2047))+sizeof(Storage::Page*)))->m_ptr
$14 = (unsigned char *) 0xffeee5572d78 "\333"

第5页: 
(gdb) p *reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffeee5562d80) + sizeof(Storage::Page *) +32 * 2047)
$15 = (temptable::Storage::Page *) 0xffeee8a85188
(gdb) p static_cast<const temptable::Row *>((static_cast<uint8_t*>(*reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffeee5562d80) + sizeof(Storage::Page *) +32 * 2047))+sizeof(Storage::Page*)))->m_ptr
$16 = (unsigned char *) 0xffeee8a95180 "q?\001"

第6页: 
(gdb) p *reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffeee8a85188) + sizeof(Storage::Page *) +32 * 2047)
$17 = (temptable::Storage::Page *) 0xffee4edd4cc8
(gdb) p static_cast<const temptable::Row *>((static_cast<uint8_t*>(*reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffeee8a85188) + sizeof(Storage::Page *) +32 * 2047))+sizeof(Storage::Page*)))->m_ptr
$18 = (unsigned char *) 0xffee4ede4cc0 "\021\347"

第7页: 
(gdb) p *reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffee4edd4cc8) + sizeof(Storage::Page *) +32 * 2047)
$19 = (temptable::Storage::Page *) 0xffee5917c8a8
(gdb) p static_cast<const temptable::Row *>((static_cast<uint8_t*>(*reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffee4edd4cc8) + sizeof(Storage::Page *) +32 * 2047))+sizeof(Storage::Page*)))->m_ptr
$20 = (unsigned char *) 0xffee5918c8a0 "\276R"

第8页: 
(gdb) p *reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffee5917c8a8) + sizeof(Storage::Page *) +32 * 2047)
$21 = (temptable::Storage::Page *) 0xffee64af2998
(gdb) p static_cast<const temptable::Row *>((static_cast<uint8_t*>(*reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffee5917c8a8) + sizeof(Storage::Page *) +32 * 2047))+sizeof(Storage::Page*)))->m_ptr
$22 = (unsigned char *) 0xffee64b02990 "\303\267"

第9页: 
(gdb) p *reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffee64af2998) + sizeof(Storage::Page *) +32 * 2047)
$23 = (temptable::Storage::Page *) 0xffededb0c4a8
(gdb) p static_cast<const temptable::Row *>((static_cast<uint8_t*>(*reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffee64af2998) + sizeof(Storage::Page *) +32 * 2047))+sizeof(Storage::Page*)))->m_ptr
$24 = (unsigned char *) 0xffededb1c4a0 "-9"

第10页: 
(gdb) p *reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffededb0c4a8) + sizeof(Storage::Page *) +32 * 2047)
$25 = (temptable::Storage::Page *) 0xffedf717bb20
(gdb) p static_cast<const temptable::Row *>((static_cast<uint8_t*>(*reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffededb0c4a8) + sizeof(Storage::Page *) +32 * 2047))+sizeof(Storage::Page*)))->m_ptr
$26 = (unsigned char *) 0xffedf718bb18 "\315\251\002"

第11页:
(gdb) p *reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffedf717bb20) + sizeof(Storage::Page *) +32 * 2047)
$27 = (temptable::Storage::Page *) 0xffee015a97f0
(gdb) p static_cast<const temptable::Row *>((static_cast<uint8_t*>(*reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffedf717bb20) + sizeof(Storage::Page *) +32 * 2047))+sizeof(Storage::Page*)))->m_ptr
$28 = (unsigned char *) 0xffee015b97e8 ":\217\001"

第12页: 
(gdb) p *reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffee015a97f0) + sizeof(Storage::Page *) +32 * 2047)
$29 = (temptable::Storage::Page *) 0xffeeef8e8450
(gdb) p static_cast<const temptable::Row *>((static_cast<uint8_t*>(*reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffee015a97f0) + sizeof(Storage::Page *) +32 * 2047))+sizeof(Storage::Page*)))->m_ptr
$30 = (unsigned char *) 0xffeeef8f8448 "\271a\003"

第13页: 
(gdb) p *reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffeeef8e8450) + sizeof(Storage::Page *) +32 * 2047)
$31 = (temptable::Storage::Page *) 0xffeef88a5560
(gdb) p static_cast<const temptable::Row *>((static_cast<uint8_t*>(*reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffeeef8e8450) + sizeof(Storage::Page *) +32 * 2047))+sizeof(Storage::Page*)))->m_ptr
$32 = (unsigned char *) 0xffeef88b5558 "\333"

第14页报错: 
(gdb) p *reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffeef88a5560) + sizeof(Storage::Page *) +32 * 2047)
$33 = (temptable::Storage::Page *) 0x0
(gdb) p static_cast<const temptable::Row *>((static_cast<uint8_t*>(*reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffeef88a5560) + sizeof(Storage::Page *) +32 * 2047))+sizeof(Storage::Page*)))->m_ptr
Cannot access memory at address 0x18
 
匹配 总页数
``` 

问题地址 0xffee59188550 在第8页:

```
第7页地址: 0xffee5917c8a8
 
页头记录: 
 
(gdb) p (static_cast<uint8_t *>(0xffee5917c8a8) + sizeof(Storage::Page *))
$41 = (uint8_t *) 0xffee5917c8b0 "\350\333\002X\360\377"
 
(gdb) p *reinterpret_cast<Cell *>(static_cast<const temptable::Row *>(0xffee5917c8b0 + 0*32)->m_ptr+sizeof(size_t))
$44 = {m_is_null = false, m_data_length = 6,
  m_data = 0xffee5918c918 "672178e9ab3894a71a45c29bbcfb45fb7782ac56665\231\255\240\232\315\062\061 flink sql \347\237\245\345\205\266\346\211\200\344\273\245\347\204\266\357\274\210\345\215\201\344\270\203\357\274\211\357\274\232flink sql \345\274\200\345\217\221\345\210\251\345\231\250\344\271\213 Zeppelin &^%1{\"md\":\"<div class=\\\"rich_media_content", ' ' <repeats 32 times>...}
 
 
问题地址的记录序号: 
(0xffee59188550-0xffee5917c8b0) / 32 = 1509
 
(gdb) p/x 0xffee5917c8b0 + 1509*32
$48 = 0xffee59188550
 
即第1510条记录
 
检查前后数据: 
 
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

先当与 1510 - 1513 记录在temptable中受损, 内存为 0xa0

观察内存数据, 找到0xa0开始污染的位置: 

![image2023-1-11 0:31:37.png](/assets/01KJBYXQRY6V6K04B14ER6EQ02/image2023-1-11%200%3A31%3A37.png)

污染起始地址为 0xffee61490000

污染下界: 0xffee61570000

![image2023-1-11 11:40:4.png](/assets/01KJBYXQRY6V6K04B14ER6EQ02/image2023-1-11%2011%3A40%3A4.png)

污染大小: 896k

```
(gdb) p *reinterpret_cast<Cell *>(static_cast<const temptable::Row *>(0xffee5917c8b0 + 1508*32)->m_ptr+sizeof(size_t))
$71 = {m_is_null = false, m_data_length = 6,
  m_data = 0xffee61489798 "680934e9ab3894a71a45c29bbcfb45fb7782ac59127\231\255\277\003\263\062\061 \344\270\207\345\255\227\351\225\277\346\226\207 | \344\275\277\347\224\250 RBAC \351\231\220\345\210\266\345\257\271 Kubernetes \350\265\204\346\272\220\347\232\204\350\256\277\351\227\256 &^%1{\"md\":\"<div class=\\\"rich_media_content", ' ' <repeats 39 times>, "\\\" i"...}
(gdb) p 0xffee61490000-0xffee61489798
$72 = 26728
 
//TODO: 需检查此数字
 
 
//检查1509记录的m_data段, 发现之后是一段HTML, 直到污染地址位置
(gdb) p *reinterpret_cast<Cell *>(static_cast<const temptable::Row *>(0xffee5917c8b0 + 1508*32)->m_ptr+sizeof(size_t))
$81 = {m_is_null = false, m_data_length = 6,
  m_data = 0xffee61489798 "680934e9ab3894a71a45c29bbcfb45fb7782ac59127\231\255\277\003\263\062\061 \344\270\207\345\255\227\351\225\277\346\226\207 | \344\275\277\347\224\250 RBAC \351\231\220\345\210\266\345\257\271 Kubernetes \350\265\204\346\272\220\347\232\204\350\256\277\351\227\256 &^%1{\"md\":\"<div class=\\\"rich_media_content", ' ' <repeats 39 times>, "\\\" i"...}
(gdb) print "%s", 0xffee61489798
$82 = 281399299446680
(gdb) printf "%s", 0xffee61489798
680934e9ab3894a71a45c29bbcfb45fb7782ac59127����21 万字长文 | 使用 RBAC 限制对 Kubernetes 资源的访问 &^%1{"md":"<div class=\"rich_media_content                                       \" id=\"js_content\" > <h2 style=\"margin: 0px 8px;padding: 0px;outline: 0px;font-weight: 400;font-size: 16px;max-width: 100%;box-sizing: border-box !important;overflow-wrap: break-word !important;color: rgb(34, 34, 34);font-family: -apple-system, BlinkMacSystemFont, &quot;Helvetica Neue&quot;, &quot;PingFang SC&quot;, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei UI&quot;, &quot;Microsoft YaHei&quot;, Arial, sans-serif;font-style: normal;font-variant-ligatures: normal;font-variant-caps: normal;letter-spacing: 0．544px;orphans: 2;text-align: justify;text-indent: 0px;text-transform: none;white-space: normal;widows: 2;word-spacing: 0px;-webkit-text-stroke-width: 0px;text-decoration-thickness: initial;text-decoration-style: initial;text-decoration-color: initial;background-color: rgb(255, 255, 255);visibility: visible;\"><img class=\"rich_pages wxw-img __bg_gif\" data-fileid=\"100008505\" data-ratio=\"0．16625\" data-s=\"300,640\" data-type=\"gif\" data-w=\"800\" style=\"margin: 0px;padding: 0px;outline: 0px;max-width: 100%;vertical-align: bottom;color: rgb(62, 62, 62);font-size: 12px;letter-spacing: 0．5px;text-align: center;box-sizing: border-box !important;overflow-wrap: break-word !important;visibility: visible !important;width: 677px !important;height: auto !important;\" src=\"http://dt．abc/diting_back/api/rest/outservice/ufile/bb61954bc89149e8b78aa4db612acddc9900008820220831\"></h2> <section style=\"margin: 0px 0px 0em;padding: 0px 10px;outline: 0px;max-width: 100%;box-sizing: border-box;font-family: -apple-system, BlinkMacSystemFont, &quot;Helvetica Neue&quot;, &quot;PingFang SC&quot;, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei UI&quot;, &quot;Microsoft YaHei&quot;, Arial, sans-serif;font-style: normal;font-variant-ligatures: normal;font-variant-caps: normal;font-weight: 400;orphans: 2;text-indent: 0px;text-transform: none;white-space: normal;widows: 2;word-spacing: 0px;-webkit-text-stroke-width: 0px;text-decoration-thickness: initial;text-decoration-style: initial;text-decoration-color: initial;font-size: 15px;color: rgb(62, 62, 62);letter-spacing: 0．5px;text-align: center;background-color: rgb(255, 255, 255);line-height: 2;visibility: visible;overflow-wrap: break-word !important;\">  <section powered-by=\"xiumi．us\" style=\"margin: 0px;padding: 0px;outline: 0px;max-width: 100%;box-sizing: border-box;font-size: 12px;visibility: visible;overflow-wrap: break-word !important;\">   <section style=\"margin: 0px;padding: 0px 10px;outline: 0px;max-width: 100%;box-sizing: border-box;font-size: 15px;line-height: 2;visibility: visible;overflow-wrap: break-word !important;\">    <section style=\"margin: 0px;padding: 0px 10px;outline: 0px;max-width: 100%;box-sizing: border-box;line-height: 2;visibility: visible;overflow-wrap: break-word !important;\">     <section powered-by=\"xiumi．us\" style=\"margin: 0px;padding: 0px;outline: 0px;max-width: 100%;box-sizing: border-box;font-size: 12px;visibility: visible;overflow-wrap: break-word !important;\">
...
 
``` 

# 检查另一份coredump的污染范围

```
//bt, 获取temptable::Handler地址
 
#10 temptable::Handler::rnd_next (this=0xfff0ec163a78,
    mysql_row=0xfff0ec16b818 "\374\n738869")
    at /usr/src/debug/mysql-community-8.0.18-1.el8.aarch64/mysql-8.0.18/storage/temptable/src/handler.cc:279
 
//出问题的迭代器地址
(gdb) p ((temptable::Handler*)0xfff0ec163a78)->m_rnd_iterator->m_element
$3 = (temptable::Storage::Element *) 0xffe9397edfb8
 
// 获取 m_opend_table
(gdb) p *(((temptable::Handler*)0xfff0ec163a78)->m_opened_table)
$2 = {m_allocator = {<temptable::MemoryMonitor> = {
      static ram = {<std::__atomic_base<unsigned long>> = {static _S_alignment = 8,
          _M_i = 1071644672}, <No data fields>}},
    m_state = std::shared_ptr<temptable::AllocatorState> (use count 4, weak count 0) = {
      get() = 0xfff0ec1538b0}}, m_rows = {static ALIGN_TO = 8,
    static ELEMENT_FIRST_ON_PAGE = 1 '\001', static ELEMENT_LAST_ON_PAGE = 2 '\002',
    static ELEMENT_DELETED = 4 '\004', static META_BYTES_PER_ELEMENT = 1,
    static META_BYTES_PER_PAGE = 16, m_allocator = 0xfff0ec027d38, m_element_size = 24,
    m_bytes_used_per_element = 32, m_number_of_elements_per_page = 2047,
    m_number_of_elements = 26967, m_first_page = 0xfff445e600a0,
    m_last_page = 0xffe9427a5560, m_last_element = 0xffe9427a81c8},
  m_all_columns_are_fixed_size = false, m_indexes_are_enabled = true,
  m_mysql_row_length = 552, m_index_entries = std::vector of length 0, capacity 0,
  m_insert_undo = std::vector of length 0, capacity 0,
  m_columns = std::vector of length 7, capacity 7 = {{m_nullable = false, m_is_blob = false,
      m_null_bitmask = 0 '\000', m_length_bytes_size = 1 '\001', {m_length = 1,
        m_offset = 1}, m_null_byte_offset = 0, m_user_data_offset = 2}, {m_nullable = false,
      m_is_blob = false, m_null_bitmask = 0 '\000', m_length_bytes_size = 1 '\001', {
        m_length = 130, m_offset = 130}, m_null_byte_offset = 0, m_user_data_offset = 131}, {
      m_nullable = false, m_is_blob = false, m_null_bitmask = 0 '\000',
      m_length_bytes_size = 1 '\001', {m_length = 259, m_offset = 259},
      m_null_byte_offset = 0, m_user_data_offset = 260}, {m_nullable = false,
      m_is_blob = false, m_null_bitmask = 0 '\000', m_length_bytes_size = 0 '\000', {
        m_length = 5, m_offset = 5}, m_null_byte_offset = 0, m_user_data_offset = 388}, {
      m_nullable = false, m_is_blob = false, m_null_bitmask = 0 '\000',
      m_length_bytes_size = 1 '\001', {m_length = 393, m_offset = 393},
      m_null_byte_offset = 0, m_user_data_offset = 394}, {m_nullable = true,
      m_is_blob = false, m_null_bitmask = 1 '\001', m_length_bytes_size = 1 '\001', {
        m_length = 402, m_offset = 402}, m_null_byte_offset = 0, m_user_data_offset = 403}, {
      m_nullable = true, m_is_blob = true, m_null_bitmask = 2 '\002',
      m_length_bytes_size = 4 '\004', {m_length = 531, m_offset = 531},
      m_null_byte_offset = 0, m_user_data_offset = 535}},
  m_mysql_table_share = 0xfff0ec05d4b8}
 
 
//遍历页, 查看出问题的地址在哪个页, 第一页地址为 0xfff445e600a0
 
//第一页
(gdb) p *reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xfff445e600a0) + sizeof(Storage::Page *) +32 * 2047)
$1 = (temptable::Storage::Page *) 0xffed833340d0
(gdb) p static_cast<const temptable::Row *>((static_cast<uint8_t*>(*reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xfff445e600a0) + sizeof(Storage::Page *) +32 * 2047))+sizeof(Storage::Page*)))->m_ptr
$2 = (unsigned char *) 0xffed833440c8 ")\205"
 
//第二页
(gdb) p *reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffed833340d0) + sizeof(Storage::Page *) +32 * 2047)
$4 = (temptable::Storage::Page *) 0xffed38a82268
(gdb) p static_cast<const temptable::Row *>((static_cast<uint8_t*>(*reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffed833340d0) + sizeof(Storage::Page *) +32 * 2047))+sizeof(Storage::Page*)))->m_ptr
$5 = (unsigned char *) 0xffed38a92260 "\033\274\001"
 
//第三页
(gdb) p *reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffed38a82268) + sizeof(Storage::Page *) +32 * 2047)
$7 = (temptable::Storage::Page *) 0xffed3eb26b40
(gdb) p static_cast<const temptable::Row *>((static_cast<uint8_t*>(*reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffed38a82268) + sizeof(Storage::Page *) +32 * 2047))+sizeof(Storage::Page*)))->m_ptr
$8 = (unsigned char *) 0xffed3eb36b38 "\333"
 
//第四页
(gdb) p *reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffed3eb26b40) + sizeof(Storage::Page *) +32 * 2047)
$10 = (temptable::Storage::Page *) 0xffed3f562db0
(gdb) p static_cast<const temptable::Row *>((static_cast<uint8_t*>(*reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffed3eb26b40) + sizeof(Storage::Page *) +32 * 2047))+sizeof(Storage::Page*)))->m_ptr
$11 = (unsigned char *) 0xffed3f572da8 "\333"
 
//第五页
(gdb) p *reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffed3f562db0) + sizeof(Storage::Page *) +32 * 2047)
$13 = (temptable::Storage::Page *) 0xffed42a851b8
(gdb) p static_cast<const temptable::Row *>((static_cast<uint8_t*>(*reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffed3f562db0) + sizeof(Storage::Page *) +32 * 2047))+sizeof(Storage::Page*)))->m_ptr
$14 = (unsigned char *) 0xffed42a951b0 "q?\001"
 
//第六页
(gdb) p *reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffed42a851b8) + sizeof(Storage::Page *) +32 * 2047)
$16 = (temptable::Storage::Page *) 0xffeb905d4cc8
(gdb) p static_cast<const temptable::Row *>((static_cast<uint8_t*>(*reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffed42a851b8) + sizeof(Storage::Page *) +32 * 2047))+sizeof(Storage::Page*)))->m_ptr
$17 = (unsigned char *) 0xffeb905e4cc0 "\021\347"
 
//第七页
(gdb) p *reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffeb905d4cc8) + sizeof(Storage::Page *) +32 * 2047)
$18 = (temptable::Storage::Page *) 0xffeb9a97c8a8
(gdb) p static_cast<const temptable::Row *>((static_cast<uint8_t*>(*reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffeb905d4cc8) + sizeof(Storage::Page *) +32 * 2047))+sizeof(Storage::Page*)))->m_ptr
$19 = (unsigned char *) 0xffeb9a98c8a0 "\276R"
 
//第八页
(gdb) p *reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffeb9a97c8a8) + sizeof(Storage::Page *) +32 * 2047)
$20 = (temptable::Storage::Page *) 0xffeba62f2998
(gdb) p static_cast<const temptable::Row *>((static_cast<uint8_t*>(*reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffeb9a97c8a8) + sizeof(Storage::Page *) +32 * 2047))+sizeof(Storage::Page*)))->m_ptr
$21 = (unsigned char *) 0xffeba6302990 "\303\267"
 
//第九页
(gdb) p *reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffeba62f2998) + sizeof(Storage::Page *) +32 * 2047)
$22 = (temptable::Storage::Page *) 0xffe78930c4a8
(gdb) p static_cast<const temptable::Row *>((static_cast<uint8_t*>(*reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffeba62f2998) + sizeof(Storage::Page *) +32 * 2047))+sizeof(Storage::Page*)))->m_ptr
$23 = (unsigned char *) 0xffe78931c4a0 "-9"
 
//第十页
(gdb) p *reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffe78930c4a8) + sizeof(Storage::Page *) +32 * 2047)
$25 = (temptable::Storage::Page *) 0xffe79297bb20
(gdb) p static_cast<const temptable::Row *>((static_cast<uint8_t*>(*reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffe78930c4a8) + sizeof(Storage::Page *) +32 * 2047))+sizeof(Storage::Page*)))->m_ptr
$26 = (unsigned char *) 0xffe79298bb18 "\315\251\002"
 
//第十一页
(gdb) p *reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffe79297bb20) + sizeof(Storage::Page *) +32 * 2047)
$27 = (temptable::Storage::Page *) 0xffe79cda97f0
(gdb) p static_cast<const temptable::Row *>((static_cast<uint8_t*>(*reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffe79297bb20) + sizeof(Storage::Page *) +32 * 2047))+sizeof(Storage::Page*)))->m_ptr
$28 = (unsigned char *) 0xffe79cdb97e8 ":\217\001"
 
//第十二页
(gdb) p *reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffe79cda97f0) + sizeof(Storage::Page *) +32 * 2047)
$29 = (temptable::Storage::Page *) 0xffe9397e8450
(gdb) p static_cast<const temptable::Row *>((static_cast<uint8_t*>(*reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffe79cda97f0) + sizeof(Storage::Page *) +32 * 2047))+sizeof(Storage::Page*)))->m_ptr
$30 = (unsigned char *) 0xffe9397f8448 "\271a\003"
 
//第十三页
(gdb) p *reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffe9397e8450) + sizeof(Storage::Page *) +32 * 2047)
$31 = (temptable::Storage::Page *) 0xffe9427a5560
(gdb) p static_cast<const temptable::Row *>((static_cast<uint8_t*>(*reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffe9397e8450) + sizeof(Storage::Page *) +32 * 2047))+sizeof(Storage::Page*)))->m_ptr
$32 = (unsigned char *) 0xffe9427b5558 "\333"
 
//第十四页 报错, 跟之前判断一致
(gdb) p *reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffe9427a5560) + sizeof(Storage::Page *) +32 * 2047)
$33 = (temptable::Storage::Page *) 0x0
(gdb) p static_cast<const temptable::Row *>((static_cast<uint8_t*>(*reinterpret_cast<Storage::Page **>(static_cast<uint8_t *>(0xffe9427a5560) + sizeof(Storage::Page *) +32 * 2047))+sizeof(Storage::Page*)))->m_ptr
Cannot access memory at address 0x18
 
 
泄露地址出现在第12页, 页地址为 0xffe9397e8450
 
//页头记录: 
(gdb) p (static_cast<uint8_t *>(0xffe9397e8450) + sizeof(Storage::Page *))
$1 = (uint8_t *) 0xffe9397e8458 "8}\002\354\360\377"
 
//问题记录是该页的第731个记录
(gdb) p (0xffe9397edfb8-0xffe9397e8458) / 32
$4 = 731
 
//确定影响了哪些记录: 
 
(gdb) p *reinterpret_cast<Cell *>(static_cast<const temptable::Row *>(0xffe9397e8458 + 729*32)->m_ptr+sizeof(size_t))
$11 = {m_is_null = false, m_data_length = 6,
  m_data = 0xffe93d055238 "738868e9ab3894a71a45c29bbcfb45fb7782ac73767\231\256\207\065K21 \344\273\216\351\233\266\345\274\200\345\247\213\345\255\246\345\244\247\346\225\260\346\215\256\061\064 \345\270\246\344\275\240\350\201\212\344\270\200\350\201\212\346\240\207\345\207\206\345\214\226\346\225\260\346\215\256\346\214\226\346\216\230\345\205\250\346\265\201\347\250\213 &^%1{\"md\":\"<div class=\\\"rich_media_content js_underline_content", ' ' <repeats 13 times>...}
(gdb) p *reinterpret_cast<Cell *>(static_cast<const temptable::Row *>(0xffe9397e8458 + 730*32)->m_ptr+sizeof(size_t))
$12 = {m_is_null = false, m_data_length = 6,
  m_data = 0xffe93d05d268 "738869e9ab3894a71a45c29bbcfb45fb7782ac73768\231\256\207\065z21 \344\273\216\351\233\266\345\274\200\345\247\213\345\255\246\345\244\247\346\225\260\346\215\256\062\062 \346\225\260\346\215\256\344\270\255\345\217\260\357\274\232\347\224\250\345\244\247\346\225\260\346\215\256\350\265\213\350\203\275\344\270\232\345\212\241 &^%1{\"md\":\"<div class=\\\"rich_media_content js_underline_content", ' ' <repeats 13 times>, "\\\" id="...}
(gdb) p *reinterpret_cast<Cell *>(static_cast<const temptable::Row *>(0xffe9397e8458 + 731*32)->m_ptr+sizeof(size_t))
$13 = {m_is_null = 10, m_data_length = 168430090,
  m_data = 0xa0a0a0a0a0a0a0a <error: Cannot access memory at address 0xa0a0a0a0a0a0a0a>}
(gdb) p *reinterpret_cast<Cell *>(static_cast<const temptable::Row *>(0xffe9397e8458 + 732*32)->m_ptr+sizeof(size_t))
$14 = {m_is_null = false, m_data_length = 6,
  m_data = 0xffe93d095840 "738871e9ab3894a71a45c29bbcfb45fb7782ac73770\231\256\207\065\246\062\061 Pandas \345\255\246\344\271\240\357\274\232\345\237\272\347\241\200\345\205\245\351\227\250 &^%1{\"md\":\"<div class=\\\"rich_media_content js_underline_content", ' ' <repeats 13 times>, "\\\" id=\\\"js_content\\\" > <pre data-offset-key="...}
(gdb) p *reinterpret_cast<Cell *>(static_cast<const temptable::Row *>(0xffe9397e8458 + 733*32)->m_ptr+sizeof(size_t))
$15 = {m_is_null = false, m_data_length = 6,
  m_data = 0xffe93d0cbf40 "738872e9ab3894a71a45c29bbcfb45fb7782ac73771\231\256\207\066,21 \347\273\237\350\256\241\345\255\246\345\237\272\347\241\200\346\246\202\345\277\265\357\274\232\346\217\217\350\277\260\346\200\247\347\273\237\350\256\241 &^%1{\"md\":\"<div class=\\\"rich_media_content js_underline_content", ' ' <repeats 13 times>, "autoTypeSetting24psection        "...}
(gdb)

//只影响了第731行记录, 确定地址
 
(gdb) p static_cast<const temptable::Row *>(0xffe9397e8458 + 731*32)->m_ptr+sizeof(size_t)
$18 = (unsigned char *) 0xffe93d064ec0 '\n' <repeats 200 times>...
``` 

往上确定受影响区边界: 0xffe93d060000

![image2023-1-11 11:12:9.png](/assets/01KJBYXQRY6V6K04B14ER6EQ02/image2023-1-11%2011%3A12%3A9.png)

下界: 0xffe93d070000

![image2023-1-11 11:30:7.png](/assets/01KJBYXQRY6V6K04B14ER6EQ02/image2023-1-11%2011%3A30%3A7.png)

污染区域大小: 64k

```
(gdb) p 0xffe93d070000 - 0xffe93d060000
$3 = 65536
``` 

污染区域前:

打印字符串, 跟之前一样, 是HTML

![image2023-1-11 11:13:53.png](/assets/01KJBYXQRY6V6K04B14ER6EQ02/image2023-1-11%2011%3A13%3A53.png)

找到HTML的头, 地址: 0xffe93d05d29a

![image2023-1-11 11:25:8.png](/assets/01KJBYXQRY6V6K04B14ER6EQ02/image2023-1-11%2011%3A25%3A8.png)

截断前HTML的长度: 

```
(gdb) p 0xffe93d060000 - 0xffe93d05d29a
$1 = 11622
``` 

污染区域后:

是被截断的HTML

![image2023-1-11 11:32:52.png](/assets/01KJBYXQRY6V6K04B14ER6EQ02/image2023-1-11%2011%3A32%3A52.png)

怀疑与mmap映射文件有关: 

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
 
映射位置: 
/mysqldata/mysql_3306/tmp/#1610612913
``` 

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
 
映射位置: 
/mysqldata/mysql_3306/tmp/#1610616875
```
