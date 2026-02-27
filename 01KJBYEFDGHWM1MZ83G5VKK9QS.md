---
title: 20210821 - MySQL double write buffer 相关
confluence_page_id: 1343659
created_at: 2021-08-20T18:44:56+00:00
updated_at: 2021-08-20T18:44:56+00:00
---

# 参考

  1. <https://www.cnblogs.com/geaozhang/p/7276802.html>
  2. <https://www.huaweicloud.com/articles/12596091.html>
  3. <https://dev.mysql.com/doc/refman/8.0/en/innodb-doublewrite-buffer.html>

# [`innodb_doublewrite_files`](<https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_doublewrite_files>)

默认值为2, 为每个 buffer pool instance 生成两个double write file: 

  1. A flush list doublewrite file
     1. 默认大小: the InnoDB page size * doublewrite page bytes ???
  2. an LRU list doublewrite file
     1. 默认大小: InnoDB page size * (doublewrite pages + (512 / the number of buffer pool instances))
     2. 其中doublewrite pages为 [`innodb_doublewrite_pages`](<https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_doublewrite_pages>) 配置, 默认值为 innodb_write_io_threads = 4
