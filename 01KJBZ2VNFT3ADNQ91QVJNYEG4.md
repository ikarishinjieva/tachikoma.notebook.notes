---
title: 20230828 - Oceanbase redo log (clog) 阅读
confluence_page_id: 2589077
created_at: 2023-08-28T06:16:35+00:00
updated_at: 2023-08-28T07:57:04+00:00
---

# 样例文件

[clog日志.zip](/assets/01KJBZ2VNFT3ADNQ91QVJNYEG4/clog%E6%97%A5%E5%BF%97.zip)

# 日志流 1006-1 阅读

![image2023-8-28 14:9:50.png](/assets/01KJBZ2VNFT3ADNQ91QVJNYEG4/image2023-8-28%2014%3A9%3A50.png)

表列表示例: [table_locations结果.zip](/assets/01KJBZ2VNFT3ADNQ91QVJNYEG4/table_locations%E7%BB%93%E6%9E%9C.zip)

![image2023-8-28 14:30:4.png](/assets/01KJBZ2VNFT3ADNQ91QVJNYEG4/image2023-8-28%2014%3A30%3A4.png)

对于tablet_id = 1的表, 定位为 __all_core_table:

  - 作用: <http://www.oceanbase.wiki/concept/multi-tenant-architecture/user-tenants/>
  - 结构: K-V结构, <https://blog.csdn.net/OceanBaseGFBK/article/details/124635221>
    - __all_core_table 的 rowkey 包含三个：table_name、row_id、column_name ，每组 key 对应一个column_value。可以理解为将正常的二维关系表拆分成一维进行存储。
  - 表定义:

```
def_table_schema(
    owner = 'yanmu.ztl',
    table_name    = '__all_core_table',
    table_id      = '1',
    table_type = 'SYSTEM_TABLE',
    gm_columns = ['gmt_create', 'gmt_modified'],
    rowkey_columns = [
        ('table_name', 'varchar:OB_MAX_CORE_TALBE_NAME_LENGTH'),
        ('row_id', 'int'),
        ('column_name', 'varchar:OB_MAX_COLUMN_NAME_LENGTH'),
    ],
    in_tenant_space = True,

  normal_columns = [
      ('column_value', 'varchar:OB_OLD_MAX_VARCHAR_LENGTH', 'true'),
  ],
)
```

  

  - redo log

```
 "RowHeader": "{mutator_type_str_:\"MUTATOR_ROW\", mutator_type_:0, tablet_id_:{id:1}}",
            "NORMAL_ROW": {
                "RowKey": "{\"VARCHAR\":\"__all_table\", collation:\"utf8mb4_general_ci\"},{\"BIGINT\":1},{\"VARCHAR\":\"database_id\", collation:\"utf8mb4_general_ci\"}",
                "TableVersion": 0,
                "NewRow Cols": {
                    "0": "{len: 11, flag: 0, null: 0, ptr: 0x7f299e6c2501, hex: 5F5F616C6C5F7461626C65, cstr: __all_table}",
                    "1": "{len: 8, flag: 0, null: 0, ptr: 0x7ffdc748cd08, hex: 0100000000000000, int: 1, num_digit0: 0}",
                    "2": "{len: 11, flag: 0, null: 0, ptr: 0x7f299e6c250d, hex: 64617461626173655F6964, cstr: database_id}",
                    "3": "{len: 8, flag: 0, null: 0, ptr: 0x7f299e6c2518, hex: DCC3BD28BC030600, int: 1692956532523996}",
                    "4": "{len: 8, flag: 0, null: 0, ptr: 0x7f299e6c2520, hex: DCC3BD28BC030600, int: 1692956532523996}",
                    "5": "{len: 6, flag: 0, null: 0, ptr: 0x7f299e6c2528, hex: 323031303031, cstr: 201001}"
                },
                "OldRow Cols": {},
                "DmlFlag": "INSERT",
                "ModifyCount": 0,
                "AccChecksum": 1787322885,
                "Version": 1692956532270201,
                "Flag": 0,
                "SeqNo": 1692956532492849,
                "NewRowSize": 73,
                "OldRowSize": 0
            },
```

  

    - 其中第3/4列, 应该为'gmt_create', 'gmt_modified', 第5列为value

  

  

梳理snapshot_gc_scn的UPDATE事件: 

  

```
                "NewRow Cols": {
                    "0": "{len: 17, flag: 0, null: 0, ptr: 0x7f299e6c1ef6, hex: 5F5F616C6C5F676C6F62616C5F73746174, cstr: __all_global_stat}",
                    "1": "{len: 8, flag: 0, null: 0, ptr: 0x7ffdc748cd08, hex: 0100000000000000, int: 1, num_digit0: 0}",
                    "2": "{len: 15, flag: 0, null: 0, ptr: 0x7f299e6c1f08, hex: 736E617073686F745F67635F73636E, cstr: snapshot_gc_scn}",
                    "3": "{len: 8, flag: 0, null: 0, ptr: 0x7f299e6c1f17, hex: 00A1BD28BC030600, int: 1692956532515072}",
                    "4": "{len: 8, flag: 0, null: 0, ptr: 0x7f299e6c1f1f, hex: 00A1BD28BC030600, int: 1692956532515072}",
                    "5": "{len: 1, flag: 0, null: 0, ptr: 0x7f299e6c1f27, hex: 30, cstr: 0}"
                },

---

                "NewRow Cols": {
                    "0": "{len: 17, flag: 0, null: 0, ptr: 0x7f299ed2015b, hex: 5F5F616C6C5F676C6F62616C5F73746174, cstr: __all_global_stat}",
                    "1": "{len: 8, flag: 0, null: 0, ptr: 0x7ffdc748cd08, hex: 0100000000000000, int: 1, num_digit0: 0}",
                    "2": "{len: 15, flag: 0, null: 0, ptr: 0x7f299ed2016d, hex: 736E617073686F745F67635F73636E, cstr: snapshot_gc_scn}",
                    "3": "NOP",
                    "4": "{len: 8, flag: 0, null: 0, ptr: 0x7f299ed2017c, hex: C701E129BC030600, int: 1692956551610823}",
                    "5": "{len: 19, flag: 0, null: 0, ptr: 0x7f299ed20184, hex: 31363932393536353436363039353931343032, cstr: 1692956546609591402}"
                },
                "OldRow Cols": {
                    "0": "{len: 17, flag: 0, null: 0, ptr: 0x7f299ed201b4, hex: 5F5F616C6C5F676C6F62616C5F73746174, cstr: __all_global_stat}",
                    "1": "{len: 8, flag: 0, null: 0, ptr: 0x7ffdc748cd08, hex: 0100000000000000, int: 1, num_digit0: 0}",
                    "2": "{len: 15, flag: 0, null: 0, ptr: 0x7f299ed201c6, hex: 736E617073686F745F67635F73636E, cstr: snapshot_gc_scn}",
                    "3": "{len: 8, flag: 0, null: 0, ptr: 0x7f299ed201d5, hex: 00A1BD28BC030600, int: 1692956532515072}",
                    "4": "{len: 8, flag: 0, null: 0, ptr: 0x7f299ed201dd, hex: 00A1BD28BC030600, int: 1692956532515072}",
                    "5": "{len: 1, flag: 0, null: 0, ptr: 0x7f299ed201e5, hex: 30, cstr: 0}"
                },

---
                "NewRow Cols": {
                    "0": "{len: 17, flag: 0, null: 0, ptr: 0x7f299ed23605, hex: 5F5F616C6C5F676C6F62616C5F73746174, cstr: __all_global_stat}",
                    "1": "{len: 8, flag: 0, null: 0, ptr: 0x7ffdc748cd08, hex: 0100000000000000, int: 1, num_digit0: 0}",
                    "2": "{len: 15, flag: 0, null: 0, ptr: 0x7f299ed23617, hex: 736E617073686F745F67635F73636E, cstr: snapshot_gc_scn}",
                    "3": "NOP",
                    "4": "{len: 8, flag: 0, null: 0, ptr: 0x7f299ed23626, hex: DFEE792ABC030600, int: 1692956561632991}",
                    "5": "{len: 19, flag: 0, null: 0, ptr: 0x7f299ed2362e, hex: 31363932393536353536363333313336373133, cstr: 1692956556633136713}"
                },
                "OldRow Cols": {
                    "0": "{len: 17, flag: 0, null: 0, ptr: 0x7f299ed2365e, hex: 5F5F616C6C5F676C6F62616C5F73746174, cstr: __all_global_stat}",
                    "1": "{len: 8, flag: 0, null: 0, ptr: 0x7ffdc748cd08, hex: 0100000000000000, int: 1, num_digit0: 0}",
                    "2": "{len: 15, flag: 0, null: 0, ptr: 0x7f299ed23670, hex: 736E617073686F745F67635F73636E, cstr: snapshot_gc_scn}",
                    "3": "{len: 8, flag: 0, null: 0, ptr: 0x7f299ed2367f, hex: 00A1BD28BC030600, int: 1692956532515072}",
                    "4": "{len: 8, flag: 0, null: 0, ptr: 0x7f299ed23687, hex: C701E129BC030600, int: 1692956551610823}",
                    "5": "{len: 19, flag: 0, null: 0, ptr: 0x7f299ed2368f, hex: 31363932393536353436363039353931343032, cstr: 1692956546609591402}"
                },

---
                "NewRow Cols": {
                    "0": "{len: 17, flag: 0, null: 0, ptr: 0x7f299ed26b3c, hex: 5F5F616C6C5F676C6F62616C5F73746174, cstr: __all_global_stat}",
                    "1": "{len: 8, flag: 0, null: 0, ptr: 0x7ffdc748cd08, hex: 0100000000000000, int: 1, num_digit0: 0}",
                    "2": "{len: 15, flag: 0, null: 0, ptr: 0x7f299ed26b4e, hex: 736E617073686F745F67635F73636E, cstr: snapshot_gc_scn}",
                    "3": "NOP",
                    "4": "{len: 8, flag: 0, null: 0, ptr: 0x7f299ed26b5d, hex: 55B7122BBC030600, int: 1692956571645781}",
                    "5": "{len: 19, flag: 0, null: 0, ptr: 0x7f299ed26b65, hex: 31363932393536353636363436333136303035, cstr: 1692956566646316005}"
                },
                "OldRow Cols": {
                    "0": "{len: 17, flag: 0, null: 0, ptr: 0x7f299ed26b95, hex: 5F5F616C6C5F676C6F62616C5F73746174, cstr: __all_global_stat}",
                    "1": "{len: 8, flag: 0, null: 0, ptr: 0x7ffdc748cd08, hex: 0100000000000000, int: 1, num_digit0: 0}",
                    "2": "{len: 15, flag: 0, null: 0, ptr: 0x7f299ed26ba7, hex: 736E617073686F745F67635F73636E, cstr: snapshot_gc_scn}",
                    "3": "{len: 8, flag: 0, null: 0, ptr: 0x7f299ed26bb6, hex: 00A1BD28BC030600, int: 1692956532515072}",
                    "4": "{len: 8, flag: 0, null: 0, ptr: 0x7f299ed26bbe, hex: DFEE792ABC030600, int: 1692956561632991}",
                    "5": "{len: 19, flag: 0, null: 0, ptr: 0x7f299ed26bc6, hex: 31363932393536353536363333313336373133, cstr: 1692956556633136713}"
                },

---
                "NewRow Cols": {
                    "0": "{len: 17, flag: 0, null: 0, ptr: 0x7f299ed29ff9, hex: 5F5F616C6C5F676C6F62616C5F73746174, cstr: __all_global_stat}",
                    "1": "{len: 8, flag: 0, null: 0, ptr: 0x7ffdc748cd08, hex: 0100000000000000, int: 1, num_digit0: 0}",
                    "2": "{len: 15, flag: 0, null: 0, ptr: 0x7f299ed2a00b, hex: 736E617073686F745F67635F73636E, cstr: snapshot_gc_scn}",
                    "3": "NOP",
                    "4": "{len: 8, flag: 0, null: 0, ptr: 0x7f299ed2a01a, hex: 9577AB2BBC030600, int: 1692956581656469}",
                    "5": "{len: 19, flag: 0, null: 0, ptr: 0x7f299ed2a022, hex: 31363932393536353736363536393630343837, cstr: 1692956576656960487}"
                },
                "OldRow Cols": {
                    "0": "{len: 17, flag: 0, null: 0, ptr: 0x7f299ed2a052, hex: 5F5F616C6C5F676C6F62616C5F73746174, cstr: __all_global_stat}",
                    "1": "{len: 8, flag: 0, null: 0, ptr: 0x7ffdc748cd08, hex: 0100000000000000, int: 1, num_digit0: 0}",
                    "2": "{len: 15, flag: 0, null: 0, ptr: 0x7f299ed2a064, hex: 736E617073686F745F67635F73636E, cstr: snapshot_gc_scn}",
                    "3": "{len: 8, flag: 0, null: 0, ptr: 0x7f299ed2a073, hex: 00A1BD28BC030600, int: 1692956532515072}",
                    "4": "{len: 8, flag: 0, null: 0, ptr: 0x7f299ed2a07b, hex: 55B7122BBC030600, int: 1692956571645781}",
                    "5": "{len: 19, flag: 0, null: 0, ptr: 0x7f299ed2a083, hex: 31363932393536353636363436333136303035, cstr: 1692956566646316005}"
                },

---

                "NewRow Cols": {
                    "0": "{len: 17, flag: 0, null: 0, ptr: 0x7f299ed2d43c, hex: 5F5F616C6C5F676C6F62616C5F73746174, cstr: __all_global_stat}",
                    "1": "{len: 8, flag: 0, null: 0, ptr: 0x7ffdc748cd08, hex: 0100000000000000, int: 1, num_digit0: 0}",
                    "2": "{len: 15, flag: 0, null: 0, ptr: 0x7f299ed2d44e, hex: 736E617073686F745F67635F73636E, cstr: snapshot_gc_scn}",
                    "3": "NOP",
                    "4": "{len: 8, flag: 0, null: 0, ptr: 0x7f299ed2d45d, hex: 4E3E442CBC030600, int: 1692956591668814}",
                    "5": "{len: 19, flag: 0, null: 0, ptr: 0x7f299ed2d465, hex: 31363932393536353836363638363435353639, cstr: 1692956586668645569}"
                },
                "OldRow Cols": {
                    "0": "{len: 17, flag: 0, null: 0, ptr: 0x7f299ed2d495, hex: 5F5F616C6C5F676C6F62616C5F73746174, cstr: __all_global_stat}",
                    "1": "{len: 8, flag: 0, null: 0, ptr: 0x7ffdc748cd08, hex: 0100000000000000, int: 1, num_digit0: 0}",
                    "2": "{len: 15, flag: 0, null: 0, ptr: 0x7f299ed2d4a7, hex: 736E617073686F745F67635F73636E, cstr: snapshot_gc_scn}",
                    "3": "{len: 8, flag: 0, null: 0, ptr: 0x7f299ed2d4b6, hex: 00A1BD28BC030600, int: 1692956532515072}",
                    "4": "{len: 8, flag: 0, null: 0, ptr: 0x7f299ed2d4be, hex: 9577AB2BBC030600, int: 1692956581656469}",
                    "5": "{len: 19, flag: 0, null: 0, ptr: 0x7f299ed2d4c6, hex: 31363932393536353736363536393630343837, cstr: 1692956576656960487}"
                },

---

                "NewRow Cols": {
                    "0": "{len: 17, flag: 0, null: 0, ptr: 0x7f299ed3087f, hex: 5F5F616C6C5F676C6F62616C5F73746174, cstr: __all_global_stat}",
                    "1": "{len: 8, flag: 0, null: 0, ptr: 0x7ffdc748cd08, hex: 0100000000000000, int: 1, num_digit0: 0}",
                    "2": "{len: 15, flag: 0, null: 0, ptr: 0x7f299ed30891, hex: 736E617073686F745F67635F73636E, cstr: snapshot_gc_scn}",
                    "3": "NOP",
                    "4": "{len: 8, flag: 0, null: 0, ptr: 0x7f299ed308a0, hex: 6905DD2CBC030600, int: 1692956601681257}",
                    "5": "{len: 19, flag: 0, null: 0, ptr: 0x7f299ed308a8, hex: 31363932393536353936363831323335333636, cstr: 1692956596681235366}"
                },
                "OldRow Cols": {
                    "0": "{len: 17, flag: 0, null: 0, ptr: 0x7f299ed308d8, hex: 5F5F616C6C5F676C6F62616C5F73746174, cstr: __all_global_stat}",
                    "1": "{len: 8, flag: 0, null: 0, ptr: 0x7ffdc748cd08, hex: 0100000000000000, int: 1, num_digit0: 0}",
                    "2": "{len: 15, flag: 0, null: 0, ptr: 0x7f299ed308ea, hex: 736E617073686F745F67635F73636E, cstr: snapshot_gc_scn}",
                    "3": "{len: 8, flag: 0, null: 0, ptr: 0x7f299ed308f9, hex: 00A1BD28BC030600, int: 1692956532515072}",
                    "4": "{len: 8, flag: 0, null: 0, ptr: 0x7f299ed30901, hex: 4E3E442CBC030600, int: 1692956591668814}",
                    "5": "{len: 19, flag: 0, null: 0, ptr: 0x7f299ed30909, hex: 31363932393536353836363638363435353639, cstr: 1692956586668645569}"
                },
``` 

前后均能衔接上, NOP代表该列没有更新

  

UPDATE前会带有 该行的LOCK时间. 也会存在单独的LOCK时间, 猜测是因为该行没有更新, 形成了空事务 (此处不应当产生空事务?)

除了snapshot_gc_scn会被大量更新 (92/108采样), 还有少量其他表会被更新

snapshot_gc_scn会由ObFreezeInfoDetector::run3()进行轮询更新, 该线程会获取集群信息并计算freeze条件, snapshot_gc_scn应该是其集群快照的SCN
