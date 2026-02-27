---
title: 20210724 - MySQL 内存, 除了p_s统计的, 还有哪些
confluence_page_id: 1343537
created_at: 2021-07-24T15:45:49+00:00
updated_at: 2021-08-19T15:10:33+00:00
---

# 概述

需要建立一套通用的观测方法

使用gdb, 自动打印以下两个点的堆栈: 

  1. OS 内存分配: malloc
  2. p_s 的内存统计入口: pfs_memory_alloc_vc

gdb脚本: gdb.init

```
set logging redirect
set pagination off 
set logging on

b pfs_memory_alloc_vc
commands
bt
ignore 1 10
c
end

b malloc
commands
bt
ignore 2 10
c
end

c
``` 

为避免对性能影响过大, 在其中使用ignore语句, 进行了断点的抽样

调用gdb命令: 

```
gdb -x gdb.init -p 16339
``` 

# 测试1

测试: select * from a limit 100;

获取gdb输出文件: [gdb.txt](/assets/01KJBYDV2NWRKK7EW365CFMZG6/gdb.txt)

对输出的处理: 

```
less gdb.txt | egrep '^#|^$' | awk '{if (match($0, /in (.*) \(.*at/, a)) {print $1, a[1]} else {print $1, $2}}' | sed -z 's/\n \n/<double_return>/g;s/\n/<single_return>/g;s/<double_return>/\n \n/g'  | sort | uniq | sed -z 's/\n/\n\n/g' | sed -z 's/<single_return>/\n/g'
``` 

malloc与p_s匹配的记录: 

```
#0 pfs_memory_alloc_vc
#1 ut_allocator<unsigned char>::allocate_trace
#2 ut_allocator<unsigned char>::allocate
#3 rw_lock_debug_create
#4 rw_lock_add_debug_info
#5 rw_lock_x_lock_low
#6 rw_lock_x_lock_func
#7 pfs_rw_lock_x_lock_func
#8 srv_master_evict_from_table_cache
#9 srv_master_do_idle_tasks
#10 srv_master_main_loop
#11 srv_master_thread
#12 std::__invoke_impl<void, void (*&)()>
#13 std::__invoke<void (*&)()>
#14 std::_Bind<void (*())()>::__call<void>(std::tuple<>&&, std::_Index_tuple<>)
#15 std::_Bind<void (*())()>::operator()<, void>()
#16 Runnable::operator()<void (*)()>
#17 std::__invoke_impl<void, Runnable, void (*)()>
#18 std::__invoke<Runnable, void (*)()>
#19 std::thread::_Invoker<std::tuple<Runnable, void (*)()> >::_M_invoke<0ul, 1ul>
#20 std::thread::_Invoker<std::tuple<Runnable, void (*)()> >::operator()
#21 std::thread::_State_impl<std::thread::_Invoker<std::tuple<Runnable, void (*)()> > >::_M_run
#22 0x000000000534789f
#23 start_thread
#24 clone

#0 __GI___libc_malloc
#1 ut_allocator<unsigned char>::allocate
#2 rw_lock_debug_create
#3 rw_lock_add_debug_info
#4 rw_lock_x_lock_low
#5 rw_lock_x_lock_func
#6 pfs_rw_lock_x_lock_func
#7 srv_master_evict_from_table_cache
#8 srv_master_do_idle_tasks
#9 srv_master_main_loop
#10 srv_master_thread
#11 std::__invoke_impl<void, void (*&)()>
#12 std::__invoke<void (*&)()>
#13 std::_Bind<void (*())()>::__call<void>(std::tuple<>&&, std::_Index_tuple<>)
#14 std::_Bind<void (*())()>::operator()<, void>()
#15 Runnable::operator()<void (*)()>
#16 std::__invoke_impl<void, Runnable, void (*)()>
#17 std::__invoke<Runnable, void (*)()>
#18 std::thread::_Invoker<std::tuple<Runnable, void (*)()> >::_M_invoke<0ul, 1ul>
#19 std::thread::_Invoker<std::tuple<Runnable, void (*)()> >::operator()
#20 std::thread::_State_impl<std::thread::_Invoker<std::tuple<Runnable, void (*)()> > >::_M_run
#21 0x000000000534789f
#22 start_thread
#23 clone

#0 pfs_memory_alloc_vc
#1 ut_allocator<latch_level_t>::allocate_trace
#2 ut_allocator<latch_level_t>::allocate
#3 std::allocator_traits<ut_allocator<latch_level_t> >::allocate
#4 std::_Vector_base<latch_level_t, ut_allocator<latch_level_t> >::_M_allocate
#5 std::vector<latch_level_t, ut_allocator<latch_level_t> >::_M_range_initialize<latch_level_t const*>
#6 std::vector<latch_level_t, ut_allocator<latch_level_t> >::_M_initialize_dispatch<latch_level_t const*>
#7 std::vector<latch_level_t, ut_allocator<latch_level_t> >::vector<latch_level_t const*, void>
#8 sync_allowed_latches::sync_allowed_latches
#9 log_free_check_validate
#10 log_free_check
#11 srv_master_main_loop
#12 srv_master_thread
#13 std::__invoke_impl<void, void (*&)()>
#14 std::__invoke<void (*&)()>
#15 std::_Bind<void (*())()>::__call<void>(std::tuple<>&&, std::_Index_tuple<>)
#16 std::_Bind<void (*())()>::operator()<, void>()
#17 Runnable::operator()<void (*)()>
#18 std::__invoke_impl<void, Runnable, void (*)()>
#19 std::__invoke<Runnable, void (*)()>
#20 std::thread::_Invoker<std::tuple<Runnable, void (*)()> >::_M_invoke<0ul, 1ul>
#21 std::thread::_Invoker<std::tuple<Runnable, void (*)()> >::operator()
#22 std::thread::_State_impl<std::thread::_Invoker<std::tuple<Runnable, void (*)()> > >::_M_run
#23 0x000000000534789f
#24 start_thread
#25 clone

#0 __GI___libc_malloc
#1 ut_allocator<latch_level_t>::allocate
#2 std::allocator_traits<ut_allocator<latch_level_t> >::allocate
#3 std::_Vector_base<latch_level_t, ut_allocator<latch_level_t> >::_M_allocate
#4 std::vector<latch_level_t, ut_allocator<latch_level_t> >::_M_range_initialize<latch_level_t const*>
#5 std::vector<latch_level_t, ut_allocator<latch_level_t> >::_M_initialize_dispatch<latch_level_t const*>
#6 std::vector<latch_level_t, ut_allocator<latch_level_t> >::vector<latch_level_t const*, void>
#7 sync_allowed_latches::sync_allowed_latches
#8 log_free_check_validate
#9 log_free_check
#10 srv_master_main_loop
#11 srv_master_thread
#12 std::__invoke_impl<void, void (*&)()>
#13 std::__invoke<void (*&)()>
#14 std::_Bind<void (*())()>::__call<void>(std::tuple<>&&, std::_Index_tuple<>)
#15 std::_Bind<void (*())()>::operator()<, void>()
#16 Runnable::operator()<void (*)()>
#17 std::__invoke_impl<void, Runnable, void (*)()>
#18 std::__invoke<Runnable, void (*)()>
#19 std::thread::_Invoker<std::tuple<Runnable, void (*)()> >::_M_invoke<0ul, 1ul>
#20 std::thread::_Invoker<std::tuple<Runnable, void (*)()> >::operator()
#21 std::thread::_State_impl<std::thread::_Invoker<std::tuple<Runnable, void (*)()> > >::_M_run
#22 0x000000000534789f
#23 start_thread
#24 clone

#0 pfs_memory_alloc_vc
#1 ut_allocator<innodb_session_t>::allocate_trace
#2 ut_allocator<innodb_session_t>::allocate
#3 thd_to_innodb_session
#4 row_drop_tables_for_mysql_in_background
#5 srv_master_do_idle_tasks
#6 srv_master_main_loop
#7 srv_master_thread
#8 std::__invoke_impl<void, void (*&)()>
#9 std::__invoke<void (*&)()>
#10 std::_Bind<void (*())()>::__call<void>(std::tuple<>&&, std::_Index_tuple<>)
#11 std::_Bind<void (*())()>::operator()<, void>()
#12 Runnable::operator()<void (*)()>
#13 std::__invoke_impl<void, Runnable, void (*)()>
#14 std::__invoke<Runnable, void (*)()>
#15 std::thread::_Invoker<std::tuple<Runnable, void (*)()> >::_M_invoke<0ul, 1ul>
#16 std::thread::_Invoker<std::tuple<Runnable, void (*)()> >::operator()
#17 std::thread::_State_impl<std::thread::_Invoker<std::tuple<Runnable, void (*)()> > >::_M_run
#18 0x000000000534789f
#19 start_thread
#20 clone

#0 __GI___libc_malloc
#1 ut_allocator<innodb_session_t>::allocate
#2 thd_to_innodb_session
#3 row_drop_tables_for_mysql_in_background
#4 srv_master_do_idle_tasks
#5 srv_master_main_loop
#6 srv_master_thread
#7 std::__invoke_impl<void, void (*&)()>
#8 std::__invoke<void (*&)()>
#9 std::_Bind<void (*())()>::__call<void>(std::tuple<>&&, std::_Index_tuple<>)
#10 std::_Bind<void (*())()>::operator()<, void>()
#11 Runnable::operator()<void (*)()>
#12 std::__invoke_impl<void, Runnable, void (*)()>
#13 std::__invoke<Runnable, void (*)()>
#14 std::thread::_Invoker<std::tuple<Runnable, void (*)()> >::_M_invoke<0ul, 1ul>
#15 std::thread::_Invoker<std::tuple<Runnable, void (*)()> >::operator()
#16 std::thread::_State_impl<std::thread::_Invoker<std::tuple<Runnable, void (*)()> > >::_M_run
#17 0x000000000534789f
#18 start_thread
#19 clone

#0 pfs_memory_alloc_vc
#1 my_malloc
#2 Malloc_allocator<std::__detail::_Hash_node_base*>::allocate
#3 std::allocator_traits<Malloc_allocator<std::__detail::_Hash_node_base*> >::allocate
#4 std::__detail::_Hashtable_alloc<Malloc_allocator<std::__detail::_Hash_node<std::pair<binary_log::Uuid const, std::unique_ptr<Sid_map::Node, My_free_deleter> >, true> > >::_M_allocate_buckets
#5 std::_Hashtable<binary_log::Uuid, std::pair<binary_log::Uuid const, std::unique_ptr<Sid_map::Node, My_free_deleter> >, Malloc_allocator<std::pair<binary_log::Uuid const, std::unique_ptr<Sid_map::Node, My_free_deleter> > >, std::__detail::_Select1st, std::equal_to<binary_log::Uuid>, binary_log::Hash_Uuid, std::__detail::_Mod_range_hashing, std::__detail::_Default_ranged_hash, std::__detail::_Prime_rehash_policy, std::__detail::_Hashtable_traits<true, false, true> >::_M_allocate_buckets
#6 std::_Hashtable<binary_log::Uuid, std::pair<binary_log::Uuid const, std::unique_ptr<Sid_map::Node, My_free_deleter> >, Malloc_allocator<std::pair<binary_log::Uuid const, std::unique_ptr<Sid_map::Node, My_free_deleter> > >, std::__detail::_Select1st, std::equal_to<binary_log::Uuid>, binary_log::Hash_Uuid, std::__detail::_Mod_range_hashing, std::__detail::_Default_ranged_hash, std::__detail::_Prime_rehash_policy, std::__detail::_Hashtable_traits<true, false, true> >::_Hashtable
#7 std::_Hashtable<binary_log::Uuid, std::pair<binary_log::Uuid const, std::unique_ptr<Sid_map::Node, My_free_deleter> >, Malloc_allocator<std::pair<binary_log::Uuid const, std::unique_ptr<Sid_map::Node, My_free_deleter> > >, std::__detail::_Select1st, std::equal_to<binary_log::Uuid>, binary_log::Hash_Uuid, std::__detail::_Mod_range_hashing, std::__detail::_Default_ranged_hash, std::__detail::_Prime_rehash_policy, std::__detail::_Hashtable_traits<true, false, true> >::_Hashtable
#8 std::unordered_map<binary_log::Uuid, std::unique_ptr<Sid_map::Node, My_free_deleter>, binary_log::Hash_Uuid, std::equal_to<binary_log::Uuid>, Malloc_allocator<std::pair<binary_log::Uuid const, std::unique_ptr<Sid_map::Node, My_free_deleter> > > >::unordered_map
#9 malloc_unordered_map<binary_log::Uuid, std::unique_ptr<Sid_map::Node, My_free_deleter>, binary_log::Hash_Uuid, std::equal_to<binary_log::Uuid> >::malloc_unordered_map
#10 Sid_map::Sid_map
#11 Clone_persist_gtid::flush_gtids
#12 Clone_persist_gtid::periodic_write
#13 clone_gtid_thread
#14 std::__invoke_impl<void, void (*&)(Clone_persist_gtid*), Clone_persist_gtid*&>
#15 std::__invoke<void (*&)(Clone_persist_gtid*), Clone_persist_gtid*&>
#16 std::_Bind<void (*(Clone_persist_gtid*))(Clone_persist_gtid*)>::__call<void, , 0ul>(std::tuple<>&&, std::_Index_tuple<0ul>)
#17 std::_Bind<void (*(Clone_persist_gtid*))(Clone_persist_gtid*)>::operator()<, void>()
#18 Runnable::operator()<void (*)(Clone_persist_gtid*), Clone_persist_gtid*>
#19 std::__invoke_impl<void, Runnable, void (*)(Clone_persist_gtid*), Clone_persist_gtid*>
#20 std::__invoke<Runnable, void (*)(Clone_persist_gtid*), Clone_persist_gtid*>
#21 std::thread::_Invoker<std::tuple<Runnable, void (*)(Clone_persist_gtid*), Clone_persist_gtid*> >::_M_invoke<0ul, 1ul, 2ul>
#22 std::thread::_Invoker<std::tuple<Runnable, void (*)(Clone_persist_gtid*), Clone_persist_gtid*> >::operator()
#23 std::thread::_State_impl<std::thread::_Invoker<std::tuple<Runnable, void (*)(Clone_persist_gtid*), Clone_persist_gtid*> > >::_M_run
#24 0x000000000534789f
#25 start_thread
#26 clone

#0 __GI___libc_malloc
#1 my_raw_malloc
#2 my_malloc
#3 Malloc_allocator<std::__detail::_Hash_node_base*>::allocate
#4 std::allocator_traits<Malloc_allocator<std::__detail::_Hash_node_base*> >::allocate
#5 std::__detail::_Hashtable_alloc<Malloc_allocator<std::__detail::_Hash_node<std::pair<binary_log::Uuid const, std::unique_ptr<Sid_map::Node, My_free_deleter> >, true> > >::_M_allocate_buckets
#6 std::_Hashtable<binary_log::Uuid, std::pair<binary_log::Uuid const, std::unique_ptr<Sid_map::Node, My_free_deleter> >, Malloc_allocator<std::pair<binary_log::Uuid const, std::unique_ptr<Sid_map::Node, My_free_deleter> > >, std::__detail::_Select1st, std::equal_to<binary_log::Uuid>, binary_log::Hash_Uuid, std::__detail::_Mod_range_hashing, std::__detail::_Default_ranged_hash, std::__detail::_Prime_rehash_policy, std::__detail::_Hashtable_traits<true, false, true> >::_M_allocate_buckets
#7 std::_Hashtable<binary_log::Uuid, std::pair<binary_log::Uuid const, std::unique_ptr<Sid_map::Node, My_free_deleter> >, Malloc_allocator<std::pair<binary_log::Uuid const, std::unique_ptr<Sid_map::Node, My_free_deleter> > >, std::__detail::_Select1st, std::equal_to<binary_log::Uuid>, binary_log::Hash_Uuid, std::__detail::_Mod_range_hashing, std::__detail::_Default_ranged_hash, std::__detail::_Prime_rehash_policy, std::__detail::_Hashtable_traits<true, false, true> >::_Hashtable
#8 std::_Hashtable<binary_log::Uuid, std::pair<binary_log::Uuid const, std::unique_ptr<Sid_map::Node, My_free_deleter> >, Malloc_allocator<std::pair<binary_log::Uuid const, std::unique_ptr<Sid_map::Node, My_free_deleter> > >, std::__detail::_Select1st, std::equal_to<binary_log::Uuid>, binary_log::Hash_Uuid, std::__detail::_Mod_range_hashing, std::__detail::_Default_ranged_hash, std::__detail::_Prime_rehash_policy, std::__detail::_Hashtable_traits<true, false, true> >::_Hashtable
#9 std::unordered_map<binary_log::Uuid, std::unique_ptr<Sid_map::Node, My_free_deleter>, binary_log::Hash_Uuid, std::equal_to<binary_log::Uuid>, Malloc_allocator<std::pair<binary_log::Uuid const, std::unique_ptr<Sid_map::Node, My_free_deleter> > > >::unordered_map
#10 malloc_unordered_map<binary_log::Uuid, std::unique_ptr<Sid_map::Node, My_free_deleter>, binary_log::Hash_Uuid, std::equal_to<binary_log::Uuid> >::malloc_unordered_map
#11 Sid_map::Sid_map
#12 Clone_persist_gtid::flush_gtids
#13 Clone_persist_gtid::periodic_write
#14 clone_gtid_thread
#15 std::__invoke_impl<void, void (*&)(Clone_persist_gtid*), Clone_persist_gtid*&>
#16 std::__invoke<void (*&)(Clone_persist_gtid*), Clone_persist_gtid*&>
#17 std::_Bind<void (*(Clone_persist_gtid*))(Clone_persist_gtid*)>::__call<void, , 0ul>(std::tuple<>&&, std::_Index_tuple<0ul>)
#18 std::_Bind<void (*(Clone_persist_gtid*))(Clone_persist_gtid*)>::operator()<, void>()
#19 Runnable::operator()<void (*)(Clone_persist_gtid*), Clone_persist_gtid*>
#20 std::__invoke_impl<void, Runnable, void (*)(Clone_persist_gtid*), Clone_persist_gtid*>
#21 std::__invoke<Runnable, void (*)(Clone_persist_gtid*), Clone_persist_gtid*>
#22 std::thread::_Invoker<std::tuple<Runnable, void (*)(Clone_persist_gtid*), Clone_persist_gtid*> >::_M_invoke<0ul, 1ul, 2ul>
#23 std::thread::_Invoker<std::tuple<Runnable, void (*)(Clone_persist_gtid*), Clone_persist_gtid*> >::operator()
#24 std::thread::_State_impl<std::thread::_Invoker<std::tuple<Runnable, void (*)(Clone_persist_gtid*), Clone_persist_gtid*> > >::_M_run
#25 0x000000000534789f
#26 start_thread
#27 clone
``` 

p_s独有的记录

```
#0 pfs_memory_alloc_vc
#1 my_malloc
#2 create_acl_cache_hash_key
#3 Acl_cache::checkout_acl_map
#4 Security_context::checkout_access_maps
#5 THD::reset_for_next_command
#6 mysql_reset_thd_for_next_command
#7 dispatch_sql_command
#8 dispatch_command
#9 do_command
#10 handle_connection
#11 pfs_spawn_thread
#12 start_thread
#13 clone

#0 pfs_memory_alloc_vc
#1 my_malloc
#2 my_openssl_malloc
#3 CRYPTO_malloc
#4 CRYPTO_zalloc
#5 EVP_DigestInit_ex
#6 SHA_EVP256
#7 compute_digest_hash
#8 pfs_digest_end_v2
#9 inline_mysql_digest_end
#10 parse_sql
#11 dispatch_sql_command
#12 dispatch_command
#13 do_command
#14 handle_connection
#15 pfs_spawn_thread
#16 start_thread
#17 clone

#0 pfs_memory_alloc_vc
#1 ut_allocator<unsigned char>::allocate_trace
#2 ut_allocator<unsigned char>::allocate
#3 rw_lock_debug_create
#4 rw_lock_add_debug_info
#5 rw_lock_s_lock_low
#6 rw_lock_s_lock_func
#7 pfs_rw_lock_s_lock_func
#8 Buf_fetch<Buf_fetch_normal>::lookup
#9 Buf_fetch_normal::get
#10 Buf_fetch<Buf_fetch_normal>::single_page
#11 buf_page_get_gen
#12 btr_cur_open_at_index_side_func
#13 btr_pcur_t::open_at_side
#14 row_search_mvcc
#15 ha_innobase::index_read
#16 ha_innobase::index_first
#17 handler::ha_index_first
#18 IndexScanIterator<false>::Read
#19 LimitOffsetIterator::Read
#20 Query_expression::ExecuteIteratorQuery
#21 Query_expression::execute
#22 Sql_cmd_dml::execute_inner
#23 Sql_cmd_dml::execute
#24 mysql_execute_command
#25 dispatch_sql_command
#26 dispatch_command
#27 do_command
#28 handle_connection
#29 pfs_spawn_thread
#30 start_thread
#31 clone 
``` 

malloc独有的记录: 

```
#0 __GI___libc_malloc
#1 my_raw_malloc
#2 my_malloc
#3 Malloc_allocator<std::__detail::_Hash_node_base*>::allocate
#4 std::allocator_traits<Malloc_allocator<std::__detail::_Hash_node_base*> >::allocate
#5 std::__detail::_Hashtable_alloc<Malloc_allocator<std::__detail::_Hash_node<std::string, true> > >::_M_allocate_buckets
#6 std::_Hashtable<std::string, std::string, Malloc_allocator<std::string>, std::__detail::_Identity, std::equal_to<std::string>, std::hash<std::string>, std::__detail::_Mod_range_hashing, std::__detail::_Default_ranged_hash, std::__detail::_Prime_rehash_policy, std::__detail::_Hashtable_traits<true, true, true> >::_M_allocate_buckets
#7 std::_Hashtable<std::string, std::string, Malloc_allocator<std::string>, std::__detail::_Identity, std::equal_to<std::string>, std::hash<std::string>, std::__detail::_Mod_range_hashing, std::__detail::_Default_ranged_hash, std::__detail::_Prime_rehash_policy, std::__detail::_Hashtable_traits<true, true, true> >::_Hashtable
#8 std::_Hashtable<std::string, std::string, Malloc_allocator<std::string>, std::__detail::_Identity, std::equal_to<std::string>, std::hash<std::string>, std::__detail::_Mod_range_hashing, std::__detail::_Default_ranged_hash, std::__detail::_Prime_rehash_policy, std::__detail::_Hashtable_traits<true, true, true> >::_Hashtable
#9 std::unordered_set<std::string, std::hash<std::string>, std::equal_to<std::string>, Malloc_allocator<std::string> >::unordered_set
#10 malloc_unordered_set<std::string, std::hash<std::string>, std::equal_to<std::string> >::malloc_unordered_set
#11 get_and_lock_tablespace_names
#12 lock_table_names
#13 open_tables
#14 open_tables_for_query
#15 Sql_cmd_dml::prepare
#16 Sql_cmd_dml::execute
#17 mysql_execute_command
#18 dispatch_sql_command
#19 dispatch_command
#20 do_command
#21 handle_connection
#22 pfs_spawn_thread
#23 start_thread
#24 clone

#0 __GI___libc_malloc
#1 ut_allocator<unsigned char>::allocate
#2 rw_lock_debug_create
#3 rw_lock_add_debug_info
#4 rw_lock_s_lock_low
#5 rw_lock_s_lock_func
#6 pfs_rw_lock_s_lock_func
#7 Buf_fetch<Buf_fetch_normal>::mtr_add_page
#8 Buf_fetch<Buf_fetch_normal>::single_page
#9 buf_page_get_gen
#10 btr_block_get_func
#11 btr_cur_latch_leaves
#12 btr_cur_open_at_index_side_func
#13 btr_pcur_t::open_at_side
#14 row_search_mvcc
#15 ha_innobase::index_read
#16 ha_innobase::index_first
#17 handler::ha_index_first
#18 IndexScanIterator<false>::Read
#19 LimitOffsetIterator::Read
#20 Query_expression::ExecuteIteratorQuery
#21 Query_expression::execute
#22 Sql_cmd_dml::execute_inner
#23 Sql_cmd_dml::execute
#24 mysql_execute_command
#25 dispatch_sql_command
#26 dispatch_command
#27 do_command
#28 handle_connection
#29 pfs_spawn_thread
#30 start_thread
#31 clone
``` 

经代码确认, 怀疑是采样缺失

# 测试 2

延续测试1, 将采样率调整成全采样

输出文件: [gdb.txt.zip](/assets/01KJBYDV2NWRKK7EW365CFMZG6/gdb.txt.zip)

处理命令: 

```
less gdb.txt | egrep '^#|^$' | awk '{if (match($0, /in (.*) \(.*(at|from|$)/, a)) {print a[1]} else {print $2}}' | sed -z 's/\n \n/<double_return>/g;s/\n/<single_return>/g;s/<double_return>/\n \n/g'  | sort | uniq | sed -z 's/\n/\n\n/g' | sed -z 's/<single_return>/\n/g' | sed 's+<.*>++g' > gdb.fixed.txt
``` 

使用python脚本进行分析, 将malloc和p_s相同的堆栈抵消, 得到仅在malloc或者p_s出现的堆栈: 

python脚本: 

```
#!/bin/python3
segments = []
segment = []
for l in open('gdb.fixed.txt', 'r'):
    if l == "\n":
        segments.append("".join(segment))
        segment = []
    else:
        #print(l)
        segment.append(l)

uniq_segments = []
for idx in range(len(segments)):
    uniq = True
    for idx_dup in range(len(segments)):
        if (idx > idx_dup and segments[idx] == segments[idx_dup]):
            uniq = False
    if uniq:
        uniq_segments.append(segments[idx])

for a in uniq_segments:
    paired = False
    analyzed = False
    for b in uniq_segments:
        if '__GI___libc_malloc' in a and a.replace('__GI___libc_malloc', 'pfs_memory_alloc_vc\nut_allocator::allocate_trace') == b:
            paired = True
        if '__GI___libc_malloc' in b and b.replace('__GI___libc_malloc', 'pfs_memory_alloc_vc\nut_allocator::allocate_trace') == a:
            paired = True
        if 'pfs_memory_alloc_vc' in a and a.replace('pfs_memory_alloc_vc', '__GI___libc_malloc\nmy_raw_malloc') == b:
            paired = True
        if 'pfs_memory_alloc_vc' in b and b.replace('pfs_memory_alloc_vc', '__GI___libc_malloc\nmy_raw_malloc') == a:
            paired = True

    if ('ib::logger::msg' in a) or \
        ('ib::logger::operator' in a) or \
        ('ib::logger::~logger' in a) or \
        ('std::basic_string const&)\ncheck_access' in a) or \
        ('std::basic_string const&)\ncheck_grant' in a) or \
        ('operator new(unsigned long, std::nothrow_t const&)\nMDL_ticket::create' in a) or \
        ('std::basic_string const&)\nTable_cache::get_table' in a) or \
        ('std::basic_string const&)\nget_table_share' in a) or \
        ('operator new(unsigned long)\ndd::cache::Object_registry::create_map' in a) or \
        ('std::basic_string const&)\ninnodb_session_t::lookup_table_handler' in a) or \
        ('std::basic_string const&)\nCreateTracker::File::File' in a) or \
        ('std::basic_string const&)\nha_innobase::open' in a) or \
        ('dict_name::get_table' in a) or \
        ('Table_cache::add_used_table' in a) or \
        ('rw_lock_get_debug_info' in a) or \
        ('rw_lock_add_debug_info' in a) or \
        ('to_string\ntransaction_cache_delete' in a) or \
        ('AutoDebugTrace::AutoDebugTrace' in a) or \
        False:
            analyzed = True

    if not paired and not analyzed:
        print(a)
``` 

输出: 

```
root@ubuntu:/tmp/gdb# python process.py

__GI___libc_malloc
operator new(unsigned long)
std::string::_Rep::_S_create(unsigned long, unsigned long, std::allocator const&)
std::string::_S_construct
std::basic_string const&)
check_access
check_table_access
Sql_cmd_select::precheck
Sql_cmd_dml::prepare
Sql_cmd_dml::execute
mysql_execute_command
dispatch_sql_command
dispatch_command
do_command
handle_connection
pfs_spawn_thread
start_thread
clone

__GI___libc_malloc
operator new(unsigned long)
std::string::_Rep::_S_create(unsigned long, unsigned long, std::allocator const&)
std::string::_S_construct
std::basic_string const&)
check_grant
check_table_access
Sql_cmd_select::precheck
Sql_cmd_dml::prepare
Sql_cmd_dml::execute
mysql_execute_command
dispatch_sql_command
dispatch_command
do_command
handle_connection
pfs_spawn_thread
start_thread
clone

__GI___libc_malloc
operator new(unsigned long, std::nothrow_t const&)
MDL_ticket::create
MDL_context::try_acquire_lock_impl
MDL_context::acquire_lock
open_table_get_mdl_lock
open_table
open_and_process_table
open_tables
open_tables_for_query
Sql_cmd_dml::prepare
Sql_cmd_dml::execute
mysql_execute_command
dispatch_sql_command
dispatch_command
do_command
handle_connection
pfs_spawn_thread
start_thread
clone

__GI___libc_malloc
operator new(unsigned long)
std::string::_Rep::_S_create(unsigned long, unsigned long, std::allocator const&)
std::string::_S_construct
std::basic_string const&)
Table_cache::get_table
open_table
open_and_process_table
open_tables
open_tables_for_query
Sql_cmd_dml::prepare
Sql_cmd_dml::execute
mysql_execute_command
dispatch_sql_command
dispatch_command
do_command
handle_connection
pfs_spawn_thread
start_thread
clone

__GI___libc_malloc
operator new(unsigned long)
std::string::_Rep::_S_create(unsigned long, unsigned long, std::allocator const&)
std::string::_S_construct
std::basic_string const&)
get_table_share
get_table_share_with_discover
open_table
open_and_process_table
open_tables
open_tables_for_query
Sql_cmd_dml::prepare
Sql_cmd_dml::execute
mysql_execute_command
dispatch_sql_command
dispatch_command
do_command
handle_connection
pfs_spawn_thread
start_thread
clone

__GI___libc_malloc
operator new(unsigned long, std::nothrow_t const&)
MDL_ticket::create
MDL_context::try_acquire_lock_impl
MDL_context::acquire_lock
dd::mdl_lock_schema
dd::Schema_MDL_locker::ensure_locked
dd::cache::Dictionary_client::acquire
open_table
open_and_process_table
open_tables
open_tables_for_query
Sql_cmd_dml::prepare
Sql_cmd_dml::execute
mysql_execute_command
dispatch_sql_command
dispatch_command
do_command
handle_connection
pfs_spawn_thread
start_thread
clone

__GI___libc_malloc
operator new(unsigned long)
dd::cache::Object_registry::create_map
dd::cache::Object_registry::create_map_if_needed
dd::cache::Object_registry::m_map
dd::cache::Object_registry::m_map
dd::cache::Object_registry::put
dd::cache::Dictionary_client::acquire
dd::cache::Dictionary_client::acquire
dd::cache::Dictionary_client::acquire
open_table
open_and_process_table
open_tables
open_tables_for_query
Sql_cmd_dml::prepare
Sql_cmd_dml::execute
mysql_execute_command
dispatch_sql_command
dispatch_command
do_command
handle_connection
pfs_spawn_thread
start_thread
clone

__GI___libc_malloc
operator new(unsigned long)
dd::cache::Object_registry::create_map
dd::cache::Object_registry::create_map_if_needed
dd::cache::Object_registry::m_map
dd::cache::Object_registry::m_map
dd::cache::Object_registry::put
dd::cache::Dictionary_client::Auto_releaser::auto_release
dd::cache::Dictionary_client::acquire
dd::cache::Dictionary_client::acquire
dd::cache::Dictionary_client::acquire
open_table
open_and_process_table
open_tables
open_tables_for_query
Sql_cmd_dml::prepare
Sql_cmd_dml::execute
mysql_execute_command
dispatch_sql_command
dispatch_command
do_command
handle_connection
pfs_spawn_thread
start_thread
clone

__GI___libc_malloc
operator new(unsigned long)
dd::cache::Object_registry::create_map
dd::cache::Object_registry::create_map_if_needed
dd::cache::Object_registry::m_map
dd::cache::Object_registry::m_map
dd::cache::Object_registry::put
dd::cache::Dictionary_client::Auto_releaser::auto_release
dd::cache::Dictionary_client::Auto_releaser::transfer_release
dd::cache::Dictionary_client::acquire
dd::cache::Dictionary_client::acquire
open_table
open_and_process_table
open_tables
open_tables_for_query
Sql_cmd_dml::prepare
Sql_cmd_dml::execute
mysql_execute_command
dispatch_sql_command
dispatch_command
do_command
handle_connection
pfs_spawn_thread
start_thread
clone

__GI___libc_malloc
operator new(unsigned long)
dd::cache::Object_registry::create_map
dd::cache::Object_registry::create_map_if_needed
dd::cache::Object_registry::m_map
dd::cache::Object_registry::m_map
dd::cache::Object_registry::begin
dd::cache::Dictionary_client::release
dd::cache::Dictionary_client::Auto_releaser::~Auto_releaser
dd::cache::Dictionary_client::acquire
dd::cache::Dictionary_client::acquire
open_table
open_and_process_table
open_tables
open_tables_for_query
Sql_cmd_dml::prepare
Sql_cmd_dml::execute
mysql_execute_command
dispatch_sql_command
dispatch_command
do_command
handle_connection
pfs_spawn_thread
start_thread
clone

__GI___libc_malloc
operator new(unsigned long)
dd::cache::Object_registry::create_map
dd::cache::Object_registry::create_map_if_needed
dd::cache::Object_registry::m_map
dd::cache::Object_registry::m_map
dd::cache::Object_registry::put
dd::cache::Dictionary_client::acquire
dd::cache::Dictionary_client::acquire
open_table
open_and_process_table
open_tables
open_tables_for_query
Sql_cmd_dml::prepare
Sql_cmd_dml::execute
mysql_execute_command
dispatch_sql_command
dispatch_command
do_command
handle_connection
pfs_spawn_thread
start_thread
clone

__GI___libc_malloc
operator new(unsigned long)
dd::cache::Object_registry::create_map
dd::cache::Object_registry::create_map_if_needed
dd::cache::Object_registry::m_map
dd::cache::Object_registry::m_map
dd::cache::Object_registry::put
dd::cache::Dictionary_client::Auto_releaser::auto_release
dd::cache::Dictionary_client::acquire
dd::cache::Dictionary_client::acquire
open_table
open_and_process_table
open_tables
open_tables_for_query
Sql_cmd_dml::prepare
Sql_cmd_dml::execute
mysql_execute_command
dispatch_sql_command
dispatch_command
do_command
handle_connection
pfs_spawn_thread
start_thread
clone

__GI___libc_malloc
operator new(unsigned long)
dd::cache::Object_registry::create_map
dd::cache::Object_registry::create_map_if_needed
dd::cache::Object_registry::m_map
dd::cache::Object_registry::m_map
dd::cache::Object_registry::begin
dd::cache::Dictionary_client::release
dd::cache::Dictionary_client::Auto_releaser::~Auto_releaser
dd::cache::Dictionary_client::acquire
(anonymous namespace)::MDL_checker::is_locked
(anonymous namespace)::MDL_checker::is_read_locked
dd::cache::Dictionary_client::acquire
dd::cache::Dictionary_client::acquire
open_table
open_and_process_table
open_tables
open_tables_for_query
Sql_cmd_dml::prepare
Sql_cmd_dml::execute
mysql_execute_command
dispatch_sql_command
dispatch_command
do_command
handle_connection
pfs_spawn_thread
start_thread
clone

__GI___libc_malloc
operator new(unsigned long)
dd::cache::Object_registry::create_map
dd::cache::Object_registry::create_map_if_needed
dd::cache::Object_registry::m_map
dd::cache::Object_registry::m_map
dd::cache::Object_registry::begin
dd::cache::Dictionary_client::release
dd::cache::Dictionary_client::Auto_releaser::~Auto_releaser
(anonymous namespace)::MDL_checker::is_locked
(anonymous namespace)::MDL_checker::is_read_locked
dd::cache::Dictionary_client::acquire
dd::cache::Dictionary_client::acquire
open_table
open_and_process_table
open_tables
open_tables_for_query
Sql_cmd_dml::prepare
Sql_cmd_dml::execute
mysql_execute_command
dispatch_sql_command
dispatch_command
do_command
handle_connection
pfs_spawn_thread
start_thread
clone

__GI___libc_malloc
operator new(unsigned long)
dd::cache::Object_registry::create_map
dd::cache::Object_registry::create_map_if_needed
dd::cache::Object_registry::m_map
dd::cache::Object_registry::m_map
dd::cache::Object_registry::put
dd::cache::Dictionary_client::Auto_releaser::auto_release
dd::cache::Dictionary_client::Auto_releaser::transfer_release
dd::cache::Dictionary_client::acquire
open_table
open_and_process_table
open_tables
open_tables_for_query
Sql_cmd_dml::prepare
Sql_cmd_dml::execute
mysql_execute_command
dispatch_sql_command
dispatch_command
do_command
handle_connection
pfs_spawn_thread
start_thread
clone

__GI___libc_malloc
operator new(unsigned long)
dd::cache::Object_registry::create_map
dd::cache::Object_registry::create_map_if_needed
dd::cache::Object_registry::m_map
dd::cache::Object_registry::m_map
dd::cache::Object_registry::begin
dd::cache::Dictionary_client::release
dd::cache::Dictionary_client::Auto_releaser::~Auto_releaser
dd::cache::Dictionary_client::acquire
open_table
open_and_process_table
open_tables
open_tables_for_query
Sql_cmd_dml::prepare
Sql_cmd_dml::execute
mysql_execute_command
dispatch_sql_command
dispatch_command
do_command
handle_connection
pfs_spawn_thread
start_thread
clone

__GI___libc_malloc
operator new(unsigned long)
std::string::_Rep::_S_create(unsigned long, unsigned long, std::allocator const&)
std::string::_S_construct
std::basic_string const&)
innodb_session_t::lookup_table_handler
ha_innobase::open
handler::ha_open
open_table_from_share
open_table
open_and_process_table
open_tables
open_tables_for_query
Sql_cmd_dml::prepare
Sql_cmd_dml::execute
mysql_execute_command
dispatch_sql_command
dispatch_command
do_command
handle_connection
pfs_spawn_thread
start_thread
clone

__GI___libc_malloc
operator new(unsigned long)
std::string::_Rep::_S_create(unsigned long, unsigned long, std::allocator const&)
std::string::_S_construct
std::basic_string const&)
CreateTracker::File::File
CreateTracker::register_latch
sync_file_created_register
GenericPolicy::init
TTASEventMutex::init
PolicyMutex::init
mutex_init
dict_table_mutex_alloc
os_once::do_or_wait_for_done
dict_table_t::lock
dict_table_t::acquire_with_lock
ha_innobase::open
handler::ha_open
open_table_from_share
open_table
open_and_process_table
open_tables
open_tables_for_query
Sql_cmd_dml::prepare
Sql_cmd_dml::execute
mysql_execute_command
dispatch_sql_command
dispatch_command
do_command
handle_connection
pfs_spawn_thread
start_thread
clone

__GI___libc_malloc
operator new(unsigned long)
std::string::_Rep::_S_create(unsigned long, unsigned long, std::allocator const&)
std::string::_S_construct
std::basic_string const&)
ha_innobase::open
handler::ha_open
open_table_from_share
open_table
open_and_process_table
open_tables
open_tables_for_query
Sql_cmd_dml::prepare
Sql_cmd_dml::execute
mysql_execute_command
dispatch_sql_command
dispatch_command
do_command
handle_connection
pfs_spawn_thread
start_thread
clone

__GI___libc_malloc
operator new(unsigned long)
std::string::_Rep::_S_create(unsigned long, unsigned long, std::allocator const&)
std::string::_S_construct
std::basic_string::basic_string(std::string const&, unsigned long, unsigned long)
std::string::substr(unsigned long, unsigned long) const
dict_name::get_table
dict_name::get_table
ha_innobase::open
handler::ha_open
open_table_from_share
open_table
open_and_process_table
open_tables
open_tables_for_query
Sql_cmd_dml::prepare
Sql_cmd_dml::execute
mysql_execute_command
dispatch_sql_command
dispatch_command
do_command
handle_connection
pfs_spawn_thread
start_thread
clone

__GI___libc_malloc
operator new(unsigned long)
std::string::_Rep::_S_create(unsigned long, unsigned long, std::allocator const&)
std::string::_S_construct
std::basic_string const&)
dict_name::check_partition
dict_name::get_table
dict_name::get_table
ha_innobase::open
handler::ha_open
open_table_from_share
open_table
open_and_process_table
open_tables
open_tables_for_query
Sql_cmd_dml::prepare
Sql_cmd_dml::execute
mysql_execute_command
dispatch_sql_command
dispatch_command
do_command
handle_connection
pfs_spawn_thread
start_thread
clone

__GI___libc_malloc
operator new(unsigned long)
std::string::_Rep::_S_create(unsigned long, unsigned long, std::allocator const&)
std::string::_S_construct
std::basic_string const&)
dict_name::check_partition
dict_name::check_tmp
dict_name::get_table
dict_name::get_table
ha_innobase::open
handler::ha_open
open_table_from_share
open_table
open_and_process_table
open_tables
open_tables_for_query
Sql_cmd_dml::prepare
Sql_cmd_dml::execute
mysql_execute_command
dispatch_sql_command
dispatch_command
do_command
handle_connection
pfs_spawn_thread
start_thread
clone

__GI___libc_malloc
operator new(unsigned long)
dd::cache::Object_registry::create_map
dd::cache::Object_registry::create_map_if_needed
dd::cache::Object_registry::m_map
dd::cache::Object_registry::m_map
dd::cache::Object_registry::begin
dd::cache::Dictionary_client::release
dd::cache::Dictionary_client::Auto_releaser::~Auto_releaser
open_table_from_share
open_table
open_and_process_table
open_tables
open_tables_for_query
Sql_cmd_dml::prepare
Sql_cmd_dml::execute
mysql_execute_command
dispatch_sql_command
dispatch_command
do_command
handle_connection
pfs_spawn_thread
start_thread
clone

__GI___libc_malloc
operator new(unsigned long)
dd::cache::Object_registry::create_map
dd::cache::Object_registry::create_map_if_needed
dd::cache::Object_registry::m_map
dd::cache::Object_registry::m_map
dd::cache::Object_registry::begin
dd::cache::Dictionary_client::release
dd::cache::Dictionary_client::Auto_releaser::~Auto_releaser
open_table
open_and_process_table
open_tables
open_tables_for_query
Sql_cmd_dml::prepare
Sql_cmd_dml::execute
mysql_execute_command
dispatch_sql_command
dispatch_command
do_command
handle_connection
pfs_spawn_thread
start_thread
clone

__GI___libc_malloc
operator new(unsigned long)
dd::cache::Object_registry::create_map
dd::cache::Object_registry::create_map_if_needed
dd::cache::Object_registry::m_map
dd::cache::Object_registry::m_map
dd::cache::Object_registry::erase
dd::cache::Object_registry::erase_all
dd::cache::Dictionary_client::Auto_releaser::~Auto_releaser
open_table
open_and_process_table
open_tables
open_tables_for_query
Sql_cmd_dml::prepare
Sql_cmd_dml::execute
mysql_execute_command
dispatch_sql_command
dispatch_command
do_command
handle_connection
pfs_spawn_thread
start_thread
clone

__GI___libc_malloc
operator new(unsigned long)
std::string::_Rep::_S_create(unsigned long, unsigned long, std::allocator const&)
std::string::_S_construct
std::basic_string const&)
Table_cache::add_used_table
open_table
open_and_process_table
open_tables
open_tables_for_query
Sql_cmd_dml::prepare
Sql_cmd_dml::execute
mysql_execute_command
dispatch_sql_command
dispatch_command
do_command
handle_connection
pfs_spawn_thread
start_thread
clone

__GI___libc_malloc
operator new(unsigned long)
Table_cache::add_used_table
open_table
open_and_process_table
open_tables
open_tables_for_query
Sql_cmd_dml::prepare
Sql_cmd_dml::execute
mysql_execute_command
dispatch_sql_command
dispatch_command
do_command
handle_connection
pfs_spawn_thread
start_thread
clone

__GI___libc_malloc
operator new(unsigned long)
__gnu_cxx::new_allocator::allocate
std::allocator_traits::allocate
std::__detail::_Hashtable_alloc
std::_Hashtable
std::_Hashtable
std::unordered_map
Table_cache::add_used_table
open_table
open_and_process_table
open_tables
open_tables_for_query
Sql_cmd_dml::prepare
Sql_cmd_dml::execute
mysql_execute_command
dispatch_sql_command
dispatch_command
do_command
handle_connection
pfs_spawn_thread
start_thread
clone

__GI___libc_malloc
operator new(unsigned long)
__gnu_cxx::new_allocator::allocate
std::allocator_traits::allocate
std::__detail::_Hashtable_alloc::_M_allocate_buckets
std::_Hashtable::_M_allocate_buckets
std::_Hashtable::_M_rehash_aux
std::_Hashtable::_M_rehash
std::_Hashtable::_M_insert_unique_node
std::_Hashtable
std::_Hashtable
std::unordered_map
Table_cache::add_used_table
open_table
open_and_process_table
open_tables
open_tables_for_query
Sql_cmd_dml::prepare
Sql_cmd_dml::execute
mysql_execute_command
dispatch_sql_command
dispatch_command
do_command
handle_connection
pfs_spawn_thread
start_thread
clone

__GI___libc_malloc
operator new(unsigned long)
__gnu_cxx::new_allocator::allocate
std::allocator_traits::allocate
std::_Vector_base::_M_allocate
std::vector
std::vector::push_back
rw_lock_get_debug_info
rw_lock_own_flagged
buf_block_get_modify_clock
btr_pcur_t::store_position
row_search_mvcc
ha_innobase::index_read
ha_innobase::index_first
handler::ha_index_first
IndexScanIterator::Read
LimitOffsetIterator::Read
Query_expression::ExecuteIteratorQuery
Query_expression::execute
Sql_cmd_dml::execute_inner
Sql_cmd_dml::execute
mysql_execute_command
dispatch_sql_command
dispatch_command
do_command
handle_connection
pfs_spawn_thread
start_thread
clone

__GI___libc_malloc
operator new(unsigned long)
std::string::_Rep::_S_create(unsigned long, unsigned long, std::allocator const&)
std::string::_S_construct
std::basic_string const&)
to_string
transaction_cache_delete
THD::cleanup
THD::release_resources
handle_connection
pfs_spawn_thread
start_thread
clone

__GI___libc_malloc
operator new(unsigned long)
dd::cache::Object_registry::create_map
dd::cache::Object_registry::create_map_if_needed
dd::cache::Object_registry::m_map
dd::cache::Object_registry::m_map
dd::cache::Object_registry::begin
dd::cache::Dictionary_client::release
dd::cache::Dictionary_client::release
dd::cache::Dictionary_client::~Dictionary_client
std::default_delete::operator()
std::unique_ptr::~unique_ptr
THD::~THD
THD::~THD
handle_connection
pfs_spawn_thread
start_thread
clone

__GI___libc_malloc
DbugMalloc
code_state
_db_enter_
AutoDebugTrace::AutoDebugTrace
my_stat
open_error_log
reopen_error_log
handle_reload_request
signal_hand
pfs_spawn_thread
start_thread
clone

root@ubuntu:/tmp/gdb#
``` 

对输出进行分析: 

  1. ib_logger相关的堆栈, 举例:   
  

```
Thread 12 "mysqld-debug" hit Breakpoint 2, __GI___libc_malloc (bytes=41) at malloc.c:303
8
3038    in malloc.c
#0  __GI___libc_malloc (bytes=41) at malloc.c:3038
#1  0x00007f7199256298 in operator new(unsigned long) () from /usr/lib/x86_64-linux-gnu/
libstdc++.so.6
#2  0x00007f7199296c29 in std::string::_Rep::_S_create(unsigned long, unsigned long, std
::allocator<char> const&) () from /usr/lib/x86_64-linux-gnu/libstdc++.so.6
#3  0x0000000002fff2ab in std::string::_S_construct<char const*> (__beg=0x680fa8b "-+ #0
123456789.*", __end=0x680fa9b "", __a=...) at /opt/rh/devtoolset-8/root/usr/include/c++/
8/bits/basic_string.tcc:578
#4  0x00007f7199298b50 in std::basic_string<char, std::char_traits<char>, std::allocator
<char> >::basic_string(char const*, std::allocator<char> const&) () from /usr/lib/x86_64
-linux-gnu/libstdc++.so.6
#5  0x000000000487672f in ib::verify_fmt_match<char const (&) [1]> (fmt=0x7f718d5dba5e "
%s", head=...) at ../../../mysql-8.0.25/storage/innobase/include/ut0ut.h:414
#6  0x000000000488aae7 in ib::logger::msg<char const (&) [1]> (err=11953, args#0=...) at ../../../mysql-8.0.25/storage/innobase/include/ut0ut.h:585
#7  0x000000000487da8c in ib::logger::logger (this=0x7f7170a688c0, level=INFORMATION_LEVEL, err=11953) at ../../../mysql-8.0.25/storage/innobase/include/ut0ut.h:622
#8  0x000000000488c839 in ib::info::info<>(int) (this=0x7f7170a688c0, err=11953) at ../../../mysql-8.0.25/storage/innobase/include/ut0ut.h:665
#9  0x0000000004c7d8fe in buf_flush_page_coordinator_thread (n_page_cleaners=1) at ../../../mysql-8.0.25/storage/innobase/buf/buf0flu.cc:3310
#10 0x0000000004b1adf4 in std::__invoke_impl<void, void (*&)(unsigned long), unsigned long&> (__f=@0x7f7170a68be0: 0x4c7d3ee <buf_flush_page_coordinator_thread(size_t)>, __args#0=@0x7f7170a68be8: 1) at /opt/rh/devtoolset-8/root/usr/include/c++/8/bits/invoke.h:60
#11 0x0000000004b1ad69 in std::__invoke<void (*&)(unsigned long), unsigned long&> (__fn=@0x7f7170a68be0: 0x4c7d3ee <buf_flush_page_coordinator_thread(size_t)>, __args#0=@0x7f7170a68be8: 1) at /opt/rh/devtoolset-8/root/usr/include/c++/8/bits/invoke.h:95
#12 0x0000000004b1ac7c in std::_Bind<void (*(unsigned long))(unsigned long)>::__call<void, , 0ul>(std::tuple<>&&, std::_Index_tuple<0ul>) (this=0x7f7170a68be0, __args=empty std::tuple) at /opt/rh/devtoolset-8/root/usr/include/c++/8/functional:400
#13 0x0000000004b1aad0 in std::_Bind<void (*(unsigned long))(unsigned long)>::operator()<, void>() (this=0x7f7170a68be0) at /opt/rh/devtoolset-8/root/usr/include/c++/8/functional:484
#14 0x0000000004b1a853 in Runnable::operator()<void (*)(unsigned long), unsigned long> (this=0x7f71843e1418, f=@0x7f71843e1410: 0x4c7d3ee <buf_flush_page_coordinator_thread(size_t)>, args#0=@0x7f71843e1408: 1) at ../../../mysql-8.0.25/storage/innobase/include/os0thread-create.h:101
#15 0x0000000004b19909 in std::__invoke_impl<void, Runnable, void (*)(unsigned long), unsigned long> (__f=..., __args#0=@0x7f71843e1410: 0x4c7d3ee <buf_flush_page_coordinator_thread(size_t)>, __args#1=@0x7f71843e1408: 1) at /opt/rh/devtoolset-8/root/usr/include/c++/8/bits/invoke.h:60
#16 0x0000000004b16f36 in std::__invoke<Runnable, void (*)(unsigned long), unsigned long> (__fn=..., __args#0=@0x7f71843e1410: 0x4c7d3ee <buf_flush_page_coordinator_thread(size_t)>, __args#1=@0x7f71843e1408: 1) at /opt/rh/devtoolset-8/root/usr/include/c++/8/bits/invoke.h:95
#17 0x0000000004b1aeff in std::thread::_Invoker<std::tuple<Runnable, void (*)(unsigned long), unsigned long> >::_M_invoke<0ul, 1ul, 2ul> (this=0x7f71843e1408) at /opt/rh/devtoolset-8/root/usr/include/c++/8/thread:244
#18 0x0000000004b1ae9e in std::thread::_Invoker<std::tuple<Runnable, void (*)(unsigned long), unsigned long> >::operator() (this=0x7f71843e1408) at /opt/rh/devtoolset-8/root/usr/include/c++/8/thread:253
#19 0x0000000004b1ae82 in std::thread::_State_impl<std::thread::_Invoker<std::tuple<Runnable, void (*)(unsigned long), unsigned long> > >::_M_run (this=0x7f71843e1400) at /opt/rh/devtoolset-8/root/usr/include/c++/8/thread:196
#20 0x000000000534789f in execute_native_thread_routine ()
#21 0x00007f719a9776db in start_thread (arg=0x7f7170a69700) at pthread_create.c:463
#22 0x00007f719893d71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95
[Switching to Thread 0x7f7140ff9700 (LWP 23410)]
``` 
  2. check_access/check_grant 获取库名, 举例:  
  

```
__GI___libc_malloc
operator new(unsigned long)
std::string::_Rep::_S_create(unsigned long, unsigned long, std::allocator const&)
std::string::_S_construct
std::basic_string const&)
check_access
check_table_access
Sql_cmd_select::precheck
Sql_cmd_dml::prepare
Sql_cmd_dml::execute
mysql_execute_command
dispatch_sql_command
dispatch_command
do_command
handle_connection
pfs_spawn_thread
start_thread
clone
``` 
  3. MDL_ticket, 举例:   
  

```
__GI___libc_malloc
operator new(unsigned long, std::nothrow_t const&)
MDL_ticket::create
MDL_context::try_acquire_lock_impl
MDL_context::acquire_lock
open_table_get_mdl_lock
open_table
open_and_process_table
open_tables
open_tables_for_query
Sql_cmd_dml::prepare
Sql_cmd_dml::execute
mysql_execute_command
dispatch_sql_command
dispatch_command
do_command
handle_connection
pfs_spawn_thread
start_thread
clone
``` 
  4. Table_cache中, 构造查询的key:   
  

```
__GI___libc_malloc
operator new(unsigned long)
std::string::_Rep::_S_create(unsigned long, unsigned long, std::allocator const&)
std::string::_S_construct
std::basic_string const&)
Table_cache::get_table
open_table
open_and_process_table
open_tables
open_tables_for_query
Sql_cmd_dml::prepare
Sql_cmd_dml::execute
mysql_execute_command
dispatch_sql_command
dispatch_command
do_command
handle_connection
pfs_spawn_thread
start_thread
clone
``` 
  5. get_table_share中, 构造查询的key:   
  

```
__GI___libc_malloc
operator new(unsigned long)
std::string::_Rep::_S_create(unsigned long, unsigned long, std::allocator const&)
std::string::_S_construct
std::basic_string const&)
get_table_share
get_table_share_with_discover
open_table
open_and_process_table
open_tables
open_tables_for_query
Sql_cmd_dml::prepare
Sql_cmd_dml::execute
mysql_execute_command
dispatch_sql_command
dispatch_command
do_command
handle_connection
pfs_spawn_thread
start_thread
clone
``` 
  6. DD创建map, 举例: 

```
__GI___libc_malloc
operator new(unsigned long)
dd::cache::Object_registry::create_map
dd::cache::Object_registry::create_map_if_needed
dd::cache::Object_registry::m_map
dd::cache::Object_registry::m_map
dd::cache::Object_registry::put
dd::cache::Dictionary_client::acquire
dd::cache::Dictionary_client::acquire
dd::cache::Dictionary_client::acquire
open_table
open_and_process_table
open_tables
open_tables_for_query
Sql_cmd_dml::prepare
Sql_cmd_dml::execute
mysql_execute_command
dispatch_sql_command
dispatch_command
do_command
handle_connection
pfs_spawn_thread
start_thread
clone
``` 
  7. lookup_table_handler, 查找handler, 构造查找键: 

```
__GI___libc_malloc
operator new(unsigned long)
std::string::_Rep::_S_create(unsigned long, unsigned long, std::allocator const&)
std::string::_S_construct
std::basic_string const&)
innodb_session_t::lookup_table_handler
ha_innobase::open
handler::ha_open
open_table_from_share
open_table
open_and_process_table
open_tables
open_tables_for_query
Sql_cmd_dml::prepare
Sql_cmd_dml::execute
mysql_execute_command
dispatch_sql_command
dispatch_command
do_command
handle_connection
pfs_spawn_thread
start_thread
clone
``` 
  8. 用于跟踪mutex创建的位置, 构造文件名  
  

```
__GI___libc_malloc
operator new(unsigned long)
std::string::_Rep::_S_create(unsigned long, unsigned long, std::allocator const&)
std::string::_S_construct
std::basic_string const&)
CreateTracker::File::File
CreateTracker::register_latch
sync_file_created_register
GenericPolicy::init
TTASEventMutex::init
PolicyMutex::init
mutex_init
dict_table_mutex_alloc
os_once::do_or_wait_for_done
dict_table_t::lock
dict_table_t::acquire_with_lock
ha_innobase::open
handler::ha_open
open_table_from_share
open_table
open_and_process_table
open_tables
open_tables_for_query
Sql_cmd_dml::prepare
Sql_cmd_dml::execute
mysql_execute_command
dispatch_sql_command
dispatch_command
do_command
handle_connection
pfs_spawn_thread
start_thread
clone
``` 
  9. 构造在DD中的查找键  
  

```
__GI___libc_malloc
operator new(unsigned long)
std::string::_Rep::_S_create(unsigned long, unsigned long, std::allocator const&)
std::string::_S_construct
std::basic_string const&)
ha_innobase::open
handler::ha_open
open_table_from_share
open_table
open_and_process_table
open_tables
open_tables_for_query
Sql_cmd_dml::prepare
Sql_cmd_dml::execute
mysql_execute_command
dispatch_sql_command
dispatch_command
do_command
handle_connection
pfs_spawn_thread
start_thread
clone
``` 
  10. dict_name::get_table, 对 DD name的处理
  11. Table_cache::add_used_table, 增加Table cache中项的处理
  12. rw_lock_get_debug_info, 增加 thread debug info
  13. 在transaction_cache查找xid时, 构造查询键:   
  

```
__GI___libc_malloc
operator new(unsigned long)
std::string::_Rep::_S_create(unsigned long, unsigned long, std::allocator const&)
std::string::_S_construct
std::basic_string const&)
to_string
transaction_cache_delete
THD::cleanup
THD::release_resources
handle_connection
pfs_spawn_thread
start_thread
clone
``` 
  14. AutoDebugTrace 自动打印DEBUG信息 (DBUG_TRACE), 用于生成相关日志

```
__GI___libc_malloc
DbugMalloc
code_state
_db_enter_
AutoDebugTrace::AutoDebugTrace
my_stat
open_error_log
reopen_error_log
handle_reload_request
signal_hand
pfs_spawn_thread
start_thread
clone
``` 

# 测试3

使用 select a+5 from test.aa group by a+5 测试, 获取gdb.txt文件: [gdb.txt.zip](/assets/01KJBYDV2NWRKK7EW365CFMZG6/gdb.txt.zip)

分析独立的堆栈: 

  1. buf_page_t::is_io_fix_write, buf_page_t::is_io_fix_read_as_opposed_to_write, 构建参数set

```
__GI___libc_malloc
operator new(unsigned long)
__gnu_cxx::new_allocator::allocate
std::allocator_traits::allocate
std::_Rb_tree::_M_get_node
std::_Rb_tree
std::_Rb_tree
std::_Rb_tree
std::_Rb_tree
std::_Rb_tree
std::set::set
buf_page_t::is_io_fix_write
buf_flush_write_block_low
buf_flush_page
buf_flush_try_neighbors
buf_flush_page_and_try_neighbors
buf_do_flush_list_batch
buf_flush_batch
buf_flush_do_batch
buf_flush_lists
buf_flush_page_coordinator_thread
std::__invoke_impl
std::__invoke
std::_Bind)
std::_Bind()
Runnable::operator()
std::__invoke_impl
std::__invoke
std::thread::_Invoker
std::thread::_Invoker::operator()
std::thread::_State_impl::_M_run
execute_native_thread_routine
start_thread
clone
``` 
  2. 创建 temptable 需要的kv对 (table_name -> 临时表空间)   
  

```
__GI___libc_malloc
operator new(unsigned long)
__gnu_cxx::new_allocator::allocate
std::allocator_traits::allocate
std::__detail::_Hashtable_alloc
std::_Hashtable
std::_Hashtable
std::unordered_map
temptable::Key_value_store
temptable::Handler::create
create_tmp_table_with_fallback
instantiate_tmp_table
MaterializeIterator::Init
Query_expression::ExecuteIteratorQuery
Query_expression::execute
Sql_cmd_dml::execute_inner
Sql_cmd_dml::execute
mysql_execute_command
dispatch_sql_command
dispatch_command
do_command
handle_connection
pfs_spawn_thread
start_thread
clone
``` 
  3. temptable::Handler::open, temptable::Handler::delete_table, 构造在temptable 容器里的查找键 (table_name), 例:   
  

```
__GI___libc_malloc
operator new(unsigned long)
std::string::_Rep::_S_create(unsigned long, unsigned long, std::allocator const&)
std::string::_S_construct
std::basic_string const&)
temptable::Handler::open
handler::ha_open
open_tmp_table
instantiate_tmp_table
MaterializeIterator::Init
Query_expression::ExecuteIteratorQuery
Query_expression::execute
Sql_cmd_dml::execute_inner
Sql_cmd_dml::execute
mysql_execute_command
dispatch_sql_command
dispatch_command
do_command
handle_connection
pfs_spawn_thread
start_thread
clone
``` 

修订测试程序: 

```
root@ubuntu:/tmp/gdb# cat process.py
#!/bin/python3
segments = []
segment = []
for l in open('gdb.fixed.txt', 'r'):
    if l == "\n":
        segments.append("".join(segment))
        segment = []
    else:
        #print(l)
        segment.append(l)

uniq_segments = []
for idx in range(len(segments)):
    uniq = True
    for idx_dup in range(len(segments)):
        if (idx > idx_dup and segments[idx] == segments[idx_dup]):
            uniq = False
    if uniq:
        uniq_segments.append(segments[idx])

for a in uniq_segments:
    paired = False
    analyzed = False
    for b in uniq_segments:
        if '__GI___libc_malloc' in a and a.replace('__GI___libc_malloc', 'pfs_memory_alloc_vc\nut_allocator::allocate_trace') == b:
            paired = True
        if '__GI___libc_malloc' in b and b.replace('__GI___libc_malloc', 'pfs_memory_alloc_vc\nut_allocator::allocate_trace') == a:
            paired = True
        if 'pfs_memory_alloc_vc' in a and a.replace('pfs_memory_alloc_vc', '__GI___libc_malloc\nmy_raw_malloc') == b:
            paired = True
        if 'pfs_memory_alloc_vc' in b and b.replace('pfs_memory_alloc_vc', '__GI___libc_malloc\nmy_raw_malloc') == a:
            paired = True

    if ('ib::logger::msg' in a) or \
        ('ib::logger::operator' in a) or \
        ('ib::logger::~logger' in a) or \
        ('std::basic_string const&)\ncheck_access' in a) or \
        ('std::basic_string const&)\ncheck_grant' in a) or \
        ('operator new(unsigned long, std::nothrow_t const&)\nMDL_ticket::create' in a) or \
        ('std::basic_string const&)\nTable_cache::get_table' in a) or \
        ('std::basic_string const&)\nget_table_share' in a) or \
        ('operator new(unsigned long)\ndd::cache::Object_registry::create_map' in a) or \
        ('std::basic_string const&)\ninnodb_session_t::lookup_table_handler' in a) or \
        ('std::basic_string const&)\nCreateTracker::File::File' in a) or \
        ('std::basic_string const&)\nha_innobase::open' in a) or \
        ('dict_name::get_table' in a) or \
        ('Table_cache::add_used_table' in a) or \
        ('rw_lock_get_debug_info' in a) or \
        ('rw_lock_add_debug_info' in a) or \
        ('to_string\ntransaction_cache_delete' in a) or \
        ('AutoDebugTrace::AutoDebugTrace' in a) or \
        ('buf_page_t::is_io_fix_write' in a) or \
        ('buf_page_t::is_io_fix_read_as_opposed_to_write' in a) or \
        ('temptable::Key_value_store\ntemptable::Handler::create' in a) or \
        ('std::basic_string const&)\ntemptable::Handler::open' in a) or \
        ('std::basic_string const&)\ntemptable::Handler::delete_table' in a) or \
        False:
            analyzed = True

    if not paired and not analyzed:
        print(a)
root@ubuntu:/tmp/gdb#
``` 

# 测试4

复制结构, 一主一从, 新起数据库, 测试create table:

增加如下堆栈: 

  1. NGS初始化:   
  

```
__GI___libc_malloc
operator new(unsigned long)
__gnu_cxx::new_allocator::allocate
std::allocator_traits<std::allocator<std::_List_node<unsigned
std::_List_base<unsigned
std::list<unsigned
std::list<unsigned
std::list<unsigned
ngs::Scheduler_dynamic::lock_list<unsigned
ngs::Scheduler_dynamic::worker()
pfs_spawn_thread
start_thread
clone
``` 
  2. lock_table_names, 构造table对应的schema的集合

```
__GI___libc_malloc
operator new(unsigned long)
std::string::_Rep::_S_create(unsigned long, unsigned long, std::allocator const&)
std::string::_S_construct
std::string::_S_construct<char
std::basic_string const&)
(anonymous namespace)::schema_hash::operator()
std::__detail::_Hash_code_base<TABLE_LIST*,
std::_Hashtable<TABLE_LIST*,
std::__detail::_Insert_base<TABLE_LIST*,
std::unordered_set<TABLE_LIST*,
lock_table_names(THD*,
open_tables
open_tables
mysql_create_table(THD*,
Sql_cmd_create_table::execute(THD*)
mysql_execute_command(THD*, bool)
dispatch_sql_command
dispatch_command(THD*, COM_DATA const*, enum_server_command)
do_command
handle_connection
pfs_spawn_thread
start_thread
clone
``` 
  3. begin_attachable_ro_transaction, 构造Attachable_trx对象  
  

```
__GI___libc_malloc
operator new(unsigned long)
THD::begin_attachable_ro_transaction()
dd::Transaction_ro::Transaction_ro
dd::cache::Storage_adapter::get
dd::cache::Shared_dictionary_cache::get_uncached
dd::cache::Shared_dictionary_cache::get
dd::cache::Dictionary_client::acquire
dd::cache::Dictionary_client::acquire
dd::table_exists(dd::cache::Dictionary_client*, char const*, char const*, bool*)
check_if_table_exists
open_table(THD*,
open_and_process_table
open_tables
open_tables
mysql_create_table(THD*,
Sql_cmd_create_table::execute(THD*)
mysql_execute_command(THD*, bool)
dispatch_sql_command
dispatch_command(THD*, COM_DATA const*, enum_server_command)
do_command
handle_connection
pfs_spawn_thread
start_thread
clone
``` 
  4. 向 dd 添加表  
  

```
__GI___libc_malloc
operator new(unsigned long)
__gnu_cxx::new_allocator::allocate
std::allocator_traits<std::allocator<std::_Rb_tree_node<std::pair<std::basic_string<char,
std::_Rb_tree<std::basic_string<char,
std::_Rb_tree<std::basic_string<char,
std::_Rb_tree<std::basic_string<char,
std::map::operator[]
dd::Open_dictionary_tables_ctx::add_table(std::basic_string<char,
dd::Open_dictionary_tables_ctx::add_table
dd::Table_impl::register_tables
dd::Open_dictionary_tables_ctx::register_tables
dd::Abstract_table_impl::register_tables
dd::Open_dictionary_tables_ctx::register_tables
dd::cache::Storage_adapter::get<dd::Item_name_key,
dd::cache::Shared_dictionary_cache::get_uncached
dd::cache::Shared_dictionary_cache::get
dd::cache::Dictionary_client::acquire
dd::cache::Dictionary_client::acquire
dd::table_exists(dd::cache::Dictionary_client*, char const*, char const*, bool*)
check_if_table_exists
open_table(THD*,
open_and_process_table
open_tables
open_tables
mysql_create_table(THD*,
Sql_cmd_create_table::execute(THD*)
mysql_execute_command(THD*, bool)
dispatch_sql_command
dispatch_command(THD*, COM_DATA const*, enum_server_command)
do_command
handle_connection
pfs_spawn_thread
start_thread
clone
``` 
  5. mem_heap_create_block_func 创建事务对应的内存堆  
  

```
__GI___libc_malloc
ut_allocator::allocate
mem_heap_create_block_func(mem_block_info_t*,
mem_heap_create_func
trx_create_low
trx_allocate_for_background()
trx_allocate_for_mysql
innobase_trx_allocate
check_trx_exists
ha_innobase::update_thd(THD*)
ha_innobase::update_thd
ha_innobase::extra
open_table(THD*, TABLE_LIST*, Open_table_context*)
open_and_process_table
open_tables
open_tables
dd::Open_dictionary_tables_ctx::open_tables
dd::cache::Storage_adapter::get
dd::cache::Shared_dictionary_cache::get_uncached
dd::cache::Shared_dictionary_cache::get
dd::cache::Dictionary_client::acquire
dd::cache::Dictionary_client::acquire
dd::table_exists(dd::cache::Dictionary_client*, char const*, char const*, bool*)
check_if_table_exists
open_table(THD*,
open_and_process_table
open_tables
open_tables
mysql_create_table(THD*,
Sql_cmd_create_table::execute(THD*)
mysql_execute_command(THD*, bool)
dispatch_sql_command
dispatch_command(THD*, COM_DATA const*, enum_server_command)
do_command
handle_connection
pfs_spawn_thread
start_thread
clone
``` 
  6. 构造访问DD的键  
  

```
__GI___libc_malloc
operator new(unsigned long, std::nothrow_t const&)
dd::Item_name_key::create_access_key
dd::Raw_table::find_record(dd::Object_key const&, std::unique_ptr&)
dd::cache::Storage_adapter::get
dd::cache::Shared_dictionary_cache::get_uncached
dd::cache::Shared_dictionary_cache::get
dd::cache::Dictionary_client::acquire
dd::cache::Dictionary_client::acquire
dd::table_exists(dd::cache::Dictionary_client*, char const*, char const*, bool*)
check_if_table_exists
open_table(THD*,
open_and_process_table
open_tables
open_tables
mysql_create_table(THD*,
Sql_cmd_create_table::execute(THD*)
mysql_execute_command(THD*, bool)
dispatch_sql_command
dispatch_command(THD*, COM_DATA const*, enum_server_command)
do_command
handle_connection
pfs_spawn_thread
start_thread
clone
``` 
  7. dd::Properties_impl, 构造DD的属性set  
  

```
__GI___libc_malloc
operator new(unsigned long)
__gnu_cxx::new_allocator::allocate
std::allocator_traits<std::allocator<std::_Rb_tree_node<std::basic_string<char,
std::_Rb_tree<std::basic_string<char,
std::_Rb_tree<std::basic_string<char,
std::_Rb_tree<std::basic_string<char,
std::_Rb_tree<std::basic_string<char,
std::_Rb_tree<std::basic_string<char,
std::_Rb_tree
std::_Rb_tree<std::basic_string<char,
std::_Rb_tree<std::basic_string<char,
std::set<std::basic_string<char,
dd::Properties_impl::Properties_impl
dd::Abstract_table_impl::Abstract_table_impl(dd::Abstract_table_impl
dd::Table_impl::Table_impl(dd::Table_impl const&)
covariant return thunk to dd::Table_impl::clone() const
dd::cache::Dictionary_client::acquire_for_modification
rea_create_base_table
create_table_impl(THD*,
mysql_create_table_no_lock(THD*, char const*, char const*, HA_CREATE_INFO*, Alter_info*, unsigned int, bool, bool*, handlerton**)
mysql_create_table(THD*, TABLE_LIST*, HA_CREATE_INFO*, Alter_info*)
Sql_cmd_create_table::execute(THD*)
mysql_execute_command(THD*, bool)
dispatch_sql_command
dispatch_command(THD*, COM_DATA const*, enum_server_command)
do_command
handle_connection
pfs_spawn_thread
start_thread
clone
``` 

# 结论

此方法穷举很繁琐, 需要更好的方法
