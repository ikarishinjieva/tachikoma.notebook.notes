---
title: 20220627 - p_s 中 io 统计次数的来源
confluence_page_id: 1933360
created_at: 2022-06-27T07:55:52+00:00
updated_at: 2022-06-28T12:04:34+00:00
---

以 io_global_by_file_by_latency 为起点

找到 mysql-8.0.21/scripts/sys_schema/views/p_s/io_global_by_file_by_latency.sql, 找到主表: performance_schema.file_summary_by_instance

在代码中全文搜索, 找到主表的接口: mysql-8.0.21/storage/perfschema/[table_file_summary_by_instance.cc](<http://table_file_summary_by_instance.cc>)

找到行访问函数: table_file_summary_by_instance::read_row_values, 找到写次数的出处

```
 case 14: /* COUNT_WRITE */
          set_field_ulonglong(f, m_row.m_io_stat.m_write.m_waits.m_count);
          break;
        case 15: /* SUM_TIMER_WRITE */
          set_field_ulonglong(f, m_row.m_io_stat.m_write.m_waits.m_sum);
          break;
        case 16: /* MIN_TIMER_WRITE */
          set_field_ulonglong(f, m_row.m_io_stat.m_write.m_waits.m_min);
          break;
        case 17: /* AVG_TIMER_WRITE */
          set_field_ulonglong(f, m_row.m_io_stat.m_write.m_waits.m_avg);
          break;
        case 18: /* MAX_TIMER_WRITE */
          set_field_ulonglong(f, m_row.m_io_stat.m_write.m_waits.m_max);
          break;
        case 19: /* SUM_NUMBER_OF_BYTES_WRITE */
          set_field_ulonglong(f, m_row.m_io_stat.m_write.m_bytes);
          break;
``` 

全文搜索: m_io_stat.m_write

定位到: pfs_end_file_wait_vc 函数, 内容: 

```
  switch (state->m_operation) {
    /* Group read operations */
    case PSI_FILE_READ:
      byte_stat = &file_stat->m_io_stat.m_read;
      break;
    /* Group write operations */
    case PSI_FILE_WRITE:
      byte_stat = &file_stat->m_io_stat.m_write;
      break;
...
  }
``` 

观察函数头: 

```
/**
  Implementation of the file instrumentation interface.
  @sa PSI_v1::end_file_wait.
*/
void pfs_end_file_wait_vc(PSI_file_locker *locker, size_t byte_count) {
	...
}
``` 

全文搜索接口使用 PSI_v1::end_file_wait, 找到使用典型方式: 

```
static inline size_t inline_mysql_file_write(
#ifdef HAVE_PSI_FILE_INTERFACE
    const char *src_file, uint src_line,
#endif
    File file, const uchar *buffer, size_t count, myf flags) {
  size_t result;
#ifdef HAVE_PSI_FILE_INTERFACE
  struct PSI_file_locker *locker;
  PSI_file_locker_state state;
  size_t bytes_written;
  locker = PSI_FILE_CALL(get_thread_file_descriptor_locker)(&state, file,
                                                            PSI_FILE_WRITE);
  if (likely(locker != nullptr)) {
    PSI_FILE_CALL(start_file_wait)(locker, count, src_file, src_line);
    result = my_write(file, buffer, count, flags);
    if (flags & (MY_NABP | MY_FNABP)) {
      bytes_written = (result == 0) ? count : 0;
    } else {
      bytes_written = (result != MY_FILE_ERROR) ? result : 0;
    }
    PSI_FILE_CALL(end_file_wait)(locker, bytes_written);
    return result;
  }
#endif

  result = my_write(file, buffer, count, flags);
  return result;
}
``` 

TODO: 找到 触发了P_S但没有触发syscall的机制

通过gdb.init: 

```
set pagination off
set logging file gdb.txt
set logging on
 
br pfs_end_file_wait_vc
commands
    bt
    continue
end
 
 
catch syscall write pwrite64 fsync fdatasync read pread64
commands
    bt
    continue
end

continue
``` 

监测mysqld: 

```
gdb -p 2797 -x gdb.init 2>&1 > gdb.out
``` 

获取文件: [gdb.txt](/assets/01KJBYJWFJJDFZH16Q7JWFXN4W/gdb.txt)

按线程梳理:

![image2022-6-28 20:3:49.png](/assets/01KJBYJWFJJDFZH16Q7JWFXN4W/image2022-6-28%2020%3A3%3A49.png)

TODO: 找到 触发了P_S但没有触发syscall的 堆栈

目前: 使用简单的insert, 找不到 触发了P_S但没有触发syscall的 堆栈
