---
title: 20240418 - 对比Mariadb client和obclient的区别
confluence_page_id: 2949291
created_at: 2024-04-18T05:41:39+00:00
updated_at: 2024-04-18T06:47:55+00:00
---

```
# Mariadb
git checkout mariadb-10.4.18
 
# Obclient
git checkout 354b609fad7bfd09c6d859910e08b52bf65df657
 
root@ubuntu:/opt/mariadb# diff -q /opt/mariadb/ /opt/obclient/
Only in /opt/mariadb/: .clang-format
Common subdirectories: /opt/mariadb/.git and /opt/obclient/.git
Only in /opt/mariadb/: .gitattributes
Files /opt/mariadb/.gitignore and /opt/obclient/.gitignore differ
Files /opt/mariadb/.gitmodules and /opt/obclient/.gitmodules differ
Only in /opt/mariadb/: .travis.compiler.sh
Only in /opt/mariadb/: .travis.yml
Common subdirectories: /opt/mariadb/BUILD and /opt/obclient/BUILD
Files /opt/mariadb/CMakeLists.txt and /opt/obclient/CMakeLists.txt differ
Common subdirectories: /opt/mariadb/Docs and /opt/obclient/Docs
Only in /opt/obclient/: INFO_BIN
Only in /opt/obclient/: INFO_SRC
Only in /opt/obclient/: INSTALL-BINARY
Only in /opt/obclient/: README-wsrep
Files /opt/mariadb/README.md and /opt/obclient/README.md differ
Only in /opt/obclient/: VERSION.dep
Only in /opt/mariadb/: appveyor.yml
Only in /opt/obclient/: build.sh
Common subdirectories: /opt/mariadb/client and /opt/obclient/client
Common subdirectories: /opt/mariadb/cmake and /opt/obclient/cmake
Only in /opt/obclient/: cmake_install.cmake
Common subdirectories: /opt/mariadb/dbug and /opt/obclient/dbug
Common subdirectories: /opt/mariadb/debian and /opt/obclient/debian
Common subdirectories: /opt/mariadb/extra and /opt/obclient/extra
Only in /opt/obclient/: import_executables.cmake
Common subdirectories: /opt/mariadb/include and /opt/obclient/include
Only in /opt/obclient/: info_macros.cmake
Only in /opt/mariadb/: libmariadb
Common subdirectories: /opt/mariadb/libmysqld and /opt/obclient/libmysqld
Common subdirectories: /opt/mariadb/libservices and /opt/obclient/libservices
Only in /opt/obclient/: make_dist.cmake
Common subdirectories: /opt/mariadb/man and /opt/obclient/man
Only in /opt/obclient/: myisam.txt
Only in /opt/mariadb/: mysql-test
Only in /opt/obclient/: mysql.info
Common subdirectories: /opt/mariadb/mysys and /opt/obclient/mysys
Common subdirectories: /opt/mariadb/mysys_ssl and /opt/obclient/mysys_ssl
Common subdirectories: /opt/mariadb/pcre and /opt/obclient/pcre
Common subdirectories: /opt/mariadb/plugin and /opt/obclient/plugin
Only in /opt/mariadb/: randgen
Only in /opt/obclient/: rpm
Common subdirectories: /opt/mariadb/scripts and /opt/obclient/scripts
Common subdirectories: /opt/mariadb/sql and /opt/obclient/sql
Only in /opt/mariadb/: sql-bench
Common subdirectories: /opt/mariadb/sql-common and /opt/obclient/sql-common
Common subdirectories: /opt/mariadb/storage and /opt/obclient/storage
Common subdirectories: /opt/mariadb/strings and /opt/obclient/strings
Common subdirectories: /opt/mariadb/support-files and /opt/obclient/support-files
Common subdirectories: /opt/mariadb/tests and /opt/obclient/tests
Common subdirectories: /opt/mariadb/unittest and /opt/obclient/unittest
Common subdirectories: /opt/mariadb/vio and /opt/obclient/vio
Common subdirectories: /opt/mariadb/win and /opt/obclient/win
Common subdirectories: /opt/mariadb/wsrep-lib and /opt/obclient/wsrep-lib
Common subdirectories: /opt/mariadb/zlib and /opt/obclient/zlib
 
整理差异: 

Files /opt/mariadb/.gitignore and /opt/obclient/.gitignore differ
Files /opt/mariadb/.gitmodules and /opt/obclient/.gitmodules differ
Files /opt/mariadb/CMakeLists.txt and /opt/obclient/CMakeLists.txt differ
Files /opt/mariadb/README.md and /opt/obclient/README.md differ
Only in /opt/mariadb/: .clang-format
Only in /opt/mariadb/: .gitattributes
Only in /opt/mariadb/: .travis.compiler.sh
Only in /opt/mariadb/: .travis.yml
Only in /opt/mariadb/: appveyor.yml
Only in /opt/mariadb/: libmariadb
Only in /opt/mariadb/: mysql-test
Only in /opt/mariadb/: randgen
Only in /opt/mariadb/: sql-bench
Only in /opt/obclient/: build.sh
Only in /opt/obclient/: cmake_install.cmake
Only in /opt/obclient/: import_executables.cmake
Only in /opt/obclient/: INFO_BIN
Only in /opt/obclient/: info_macros.cmake
Only in /opt/obclient/: INFO_SRC
Only in /opt/obclient/: INSTALL-BINARY
Only in /opt/obclient/: make_dist.cmake
Only in /opt/obclient/: myisam.txt
Only in /opt/obclient/: mysql.info
Only in /opt/obclient/: README-wsrep
Only in /opt/obclient/: rpm
Only in /opt/obclient/: VERSION.dep
``` 

具体文件夹中, 会有一些代码不一致, 看起来是将高版本文件回溯到低版本中使用, 比如: 

```
root@ubuntu:/opt/mariadb# diff /opt/mariadb/sql/item_cmpfunc.cc /opt/obclient/sql/item_cmpfunc.cc
324c324
<     MY_BITMAP *old_maps[2] = { NULL, NULL };
---
>     my_bitmap_map *old_maps[2] = { NULL, NULL };
330c330
<                                &table->read_set, &table->write_set);
---
>                                table->read_set, table->write_set);
371c371
<       dbug_tmp_restore_column_maps(&table->read_set, &table->write_set, old_maps);
---
>       dbug_tmp_restore_column_maps(table->read_set, table->write_set, old_maps);
1201,1203c1201,1203
<   if (!invisible_mode())
<     return ((Item_in_subselect *)args[1])->is_top_level_item();
<   return false;
---
>   if (invisible_mode())
>     return FALSE;
>   return ((Item_in_subselect *)args[1])->is_top_level_item();
3199c3199
<
 
 
---
 
root@ubuntu:/opt/mariadb# diff /opt/mariadb/storage/connect/mysql-test/connect/r/xml.result /opt/obclient/storage/connect/mysql-test/connect/r/xml.result
377c377,378
< Message	warning about characters outside of iso-8859-1
---
> Message	Com error: Unable to save character to 'iso-8859-1' encoding.
>
 
"Com error: Unable to save character to 'iso-8859-1' encoding." 是 10.3.10被修改成"warning about characters outside of iso-8859-1"的
``` 

目标: 找到一些可以合并进obclient的commit, 沿着mysql.cc找: 

  - <https://github.com/MariaDB/server/commit/560c15c44be665e5d73194d84411c69acf8606a1>
  - <https://github.com/MariaDB/server/commit/6fe882cd85aee487c0534e6399ebd16c1ad2cab4>
  - <https://github.com/MariaDB/server/commit/1a13dbff0175d74a0304d23b0d9e5624a982db74>
