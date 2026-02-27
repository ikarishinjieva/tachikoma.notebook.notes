---
title: 20210902 - collation 不同时, _bin collation也可以进行匹配
confluence_page_id: 1343707
created_at: 2021-09-02T16:08:07+00:00
updated_at: 2021-09-02T16:08:07+00:00
---

# 问题: 

```
mysql [localhost:8025] {msandbox} (test) > explain select * from ci where id = _utf8mb4'aaaa' COLLATE utf8mb4_is_0900_as_cs;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | ci    | NULL       | ALL  | idx_b         | NULL | NULL    | NULL |    1 |   100.00 | Using where |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
1 row in set, 3 warnings (0.00 sec)
```
```
mysql [localhost:8025] {msandbox} (test) > explain select * from ci where id = _utf8mb4'aaaa' COLLATE utf8mb4_bin;
+----+-------------+-------+------------+-------+---------------+-------+---------+------+------+----------+-----------------------+
| id | select_type | table | partitions | type  | possible_keys | key   | key_len | ref  | rows | filtered | Extra                 |
+----+-------------+-------+------------+-------+---------------+-------+---------+------+------+----------+-----------------------+
|  1 | SIMPLE      | ci    | NULL       | range | idx_b         | idx_b | 403     | NULL |    1 |   100.00 | Using index condition |
+----+-------------+-------+------------+-------+---------------+-------+---------+------+------+----------+-----------------------+
1 row in set, 2 warnings (0.00 sec)
``` 

表校验集为 utf8mb4_general_ci, 当与utf8mb4_bin的字符串进行比较时, 可以使用索引range, 而与其他字符集进行比较时, 不能使用索引. 

# 分析

获取optimizer trace

```
mysql [localhost:8025] {msandbox} (test) > explain select * from ci where id = _utf8mb4'aaaa' COLLATE utf8mb4_is_0900_as_cs;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | ci    | NULL       | ALL  | idx_b         | NULL | NULL    | NULL |    1 |   100.00 | Using where |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
1 row in set, 3 warnings (0.00 sec)

mysql [localhost:8025] {msandbox} (test) > select * from ci where id = _utf8mb4'aaaa' COLLATE utf8mb4_is_0900_as_cs;
Empty set (0.00 sec)

mysql [localhost:8025] {msandbox} (test) > select trace from information_schema.optimizer_trace\G
*************************** 1. row ***************************
trace: {
  "steps": [
    {
      "join_preparation": {
        "select#": 1,
        "steps": [
          {
            "expanded_query": "/* select#1 */ select `ci`.`id` AS `id`,`ci`.`col2` AS `col2` from `ci` where (`ci`.`id` = (_utf8mb4'aaaa' collate utf8mb4_is_0900_as_cs))"
          }
        ]
      }
    },
    {
      "join_optimization": {
        "select#": 1,
        "steps": [
          {
            "condition_processing": {
              "condition": "WHERE",
              "original_condition": "(`ci`.`id` = (_utf8mb4'aaaa' collate utf8mb4_is_0900_as_cs))",
              "steps": [
                {
                  "transformation": "equality_propagation",
                  "resulting_condition": "(`ci`.`id` = (_utf8mb4'aaaa' collate utf8mb4_is_0900_as_cs))"
                },
                {
                  "transformation": "constant_propagation",
                  "resulting_condition": "(`ci`.`id` = (_utf8mb4'aaaa' collate utf8mb4_is_0900_as_cs))"
                },
                {
                  "transformation": "trivial_condition_removal",
                  "resulting_condition": "(`ci`.`id` = (_utf8mb4'aaaa' collate utf8mb4_is_0900_as_cs))"
                }
              ]
            }
          },
          {
            "substitute_generated_columns": {
            }
          },
          {
            "table_dependencies": [
              {
                "table": "`ci`",
                "row_may_be_null": false,
                "map_bit": 0,
                "depends_on_map_bits": [
                ]
              }
            ]
          },
          {
            "ref_optimizer_key_uses": [
            ]
          },
          {
            "rows_estimation": [
              {
                "table": "`ci`",
                "range_analysis": {
                  "table_scan": {
                    "rows": 1,
                    "cost": 2.45
                  },
                  "potential_range_indexes": [
                    {
                      "index": "idx_b",
                      "usable": true,
                      "key_parts": [
                        "id"
                      ]
                    }
                  ],
                  "setup_range_conditions": [
                  ],
                  "group_index_range": {
                    "chosen": false,
                    "cause": "not_group_by_or_distinct"
                  },
                  "skip_scan_range": {
                    "chosen": false,
                    "cause": "disjuntive_predicate_present"
                  }
                }
              }
            ]
          },
          {
            "considered_execution_plans": [
              {
                "plan_prefix": [
                ],
                "table": "`ci`",
                "best_access_path": {
                  "considered_access_paths": [
                    {
                      "rows_to_scan": 1,
                      "access_type": "scan",
                      "resulting_rows": 1,
                      "cost": 0.35,
                      "chosen": true
                    }
                  ]
                },
                "condition_filtering_pct": 100,
                "rows_for_plan": 1,
                "cost_for_plan": 0.35,
                "chosen": true
              }
            ]
          },
          {
            "attaching_conditions_to_tables": {
              "original_condition": "(`ci`.`id` = (_utf8mb4'aaaa' collate utf8mb4_is_0900_as_cs))",
              "attached_conditions_computation": [
              ],
              "attached_conditions_summary": [
                {
                  "table": "`ci`",
                  "attached": "(`ci`.`id` = (_utf8mb4'aaaa' collate utf8mb4_is_0900_as_cs))"
                }
              ]
            }
          },
          {
            "finalizing_table_conditions": [
              {
                "table": "`ci`",
                "original_table_condition": "(`ci`.`id` = (_utf8mb4'aaaa' collate utf8mb4_is_0900_as_cs))",
                "final_table_condition   ": "(`ci`.`id` = <cache>((_utf8mb4'aaaa' collate utf8mb4_is_0900_as_cs)))"
              }
            ]
          },
          {
            "refine_plan": [
              {
                "table": "`ci`"
              }
            ]
          }
        ]
      }
    },
    {
      "join_execution": {
        "select#": 1,
        "steps": [
        ]
      }
    }
  ]
}
1 row in set (0.00 sec)

mysql [localhost:8025] {msandbox} (test) >
``` 

```
mysql [localhost:8025] {msandbox} (test) > explain select * from ci where id = _utf8mb4'aaaa' COLLATE utf8mb4_bin;
+----+-------------+-------+------------+-------+---------------+-------+---------+------+------+----------+-----------------------+
| id | select_type | table | partitions | type  | possible_keys | key   | key_len | ref  | rows | filtered | Extra                 |
+----+-------------+-------+------------+-------+---------------+-------+---------+------+------+----------+-----------------------+
|  1 | SIMPLE      | ci    | NULL       | range | idx_b         | idx_b | 403     | NULL |    1 |   100.00 | Using index condition |
+----+-------------+-------+------------+-------+---------------+-------+---------+------+------+----------+-----------------------+
1 row in set, 2 warnings (0.00 sec)

mysql [localhost:8025] {msandbox} (test) > select * from ci where id = _utf8mb4'aaaa' COLLATE utf8mb4_bin;
Empty set (0.00 sec)

mysql [localhost:8025] {msandbox} (test) > select trace from information_schema.optimizer_trace\G
*************************** 1. row ***************************
trace: {
  "steps": [
    {
      "join_preparation": {
        "select#": 1,
        "steps": [
          {
            "expanded_query": "/* select#1 */ select `ci`.`id` AS `id`,`ci`.`col2` AS `col2` from `ci` where (`ci`.`id` = (_utf8mb4'aaaa' collate utf8mb4_bin))"
          }
        ]
      }
    },
    {
      "join_optimization": {
        "select#": 1,
        "steps": [
          {
            "condition_processing": {
              "condition": "WHERE",
              "original_condition": "(`ci`.`id` = (_utf8mb4'aaaa' collate utf8mb4_bin))",
              "steps": [
                {
                  "transformation": "equality_propagation",
                  "resulting_condition": "(`ci`.`id` = (_utf8mb4'aaaa' collate utf8mb4_bin))"
                },
                {
                  "transformation": "constant_propagation",
                  "resulting_condition": "(`ci`.`id` = (_utf8mb4'aaaa' collate utf8mb4_bin))"
                },
                {
                  "transformation": "trivial_condition_removal",
                  "resulting_condition": "(`ci`.`id` = (_utf8mb4'aaaa' collate utf8mb4_bin))"
                }
              ]
            }
          },
          {
            "substitute_generated_columns": {
            }
          },
          {
            "table_dependencies": [
              {
                "table": "`ci`",
                "row_may_be_null": false,
                "map_bit": 0,
                "depends_on_map_bits": [
                ]
              }
            ]
          },
          {
            "ref_optimizer_key_uses": [
            ]
          },
          {
            "rows_estimation": [
              {
                "table": "`ci`",
                "range_analysis": {
                  "table_scan": {
                    "rows": 1,
                    "cost": 2.45
                  },
                  "potential_range_indexes": [
                    {
                      "index": "idx_b",
                      "usable": true,
                      "key_parts": [
                        "id"
                      ]
                    }
                  ],
                  "setup_range_conditions": [
                  ],
                  "group_index_range": {
                    "chosen": false,
                    "cause": "not_group_by_or_distinct"
                  },
                  "skip_scan_range": {
                    "potential_skip_scan_indexes": [
                      {
                        "index": "idx_b",
                        "usable": false,
                        "cause": "query_references_nonkey_column"
                      }
                    ]
                  },
                  "analyzing_range_alternatives": {
                    "range_scan_alternatives": [
                      {
                        "index": "idx_b",
                        "ranges": [
                          "aaaa <= id <= aaaa"
                        ],
                        "index_dives_for_eq_ranges": true,
                        "rowid_ordered": true,
                        "using_mrr": false,
                        "index_only": false,
                        "rows": 1,
                        "cost": 0.61,
                        "chosen": true
                      }
                    ],
                    "analyzing_roworder_intersect": {
                      "usable": false,
                      "cause": "too_few_roworder_scans"
                    }
                  },
                  "chosen_range_access_summary": {
                    "range_access_plan": {
                      "type": "range_scan",
                      "index": "idx_b",
                      "rows": 1,
                      "ranges": [
                        "aaaa <= id <= aaaa"
                      ]
                    },
                    "rows_for_plan": 1,
                    "cost_for_plan": 0.61,
                    "chosen": true
                  }
                }
              }
            ]
          },
          {
            "considered_execution_plans": [
              {
                "plan_prefix": [
                ],
                "table": "`ci`",
                "best_access_path": {
                  "considered_access_paths": [
                    {
                      "rows_to_scan": 1,
                      "access_type": "range",
                      "range_details": {
                        "used_index": "idx_b"
                      },
                      "resulting_rows": 1,
                      "cost": 0.71,
                      "chosen": true
                    }
                  ]
                },
                "condition_filtering_pct": 100,
                "rows_for_plan": 1,
                "cost_for_plan": 0.71,
                "chosen": true
              }
            ]
          },
          {
            "attaching_conditions_to_tables": {
              "original_condition": "(`ci`.`id` = (_utf8mb4'aaaa' collate utf8mb4_bin))",
              "attached_conditions_computation": [
              ],
              "attached_conditions_summary": [
                {
                  "table": "`ci`",
                  "attached": "(`ci`.`id` = (_utf8mb4'aaaa' collate utf8mb4_bin))"
                }
              ]
            }
          },
          {
            "finalizing_table_conditions": [
              {
                "table": "`ci`",
                "original_table_condition": "(`ci`.`id` = (_utf8mb4'aaaa' collate utf8mb4_bin))",
                "final_table_condition   ": "(`ci`.`id` = <cache>((_utf8mb4'aaaa' collate utf8mb4_bin)))"
              }
            ]
          },
          {
            "refine_plan": [
              {
                "table": "`ci`",
                "pushed_index_condition": "(`ci`.`id` = <cache>((_utf8mb4'aaaa' collate utf8mb4_bin)))",
                "table_condition_attached": null
              }
            ]
          }
        ]
      }
    },
    {
      "join_execution": {
        "select#": 1,
        "steps": [
        ]
      }
    }
  ]
}
1 row in set (0.00 sec)

mysql [localhost:8025] {msandbox} (test) >
``` 

差异在于 skip_scan_range 这一步: 

全表扫描: 

```
"range_analysis": {
                  "skip_scan_range": {
                    "chosen": false,
                    "cause": "disjuntive_predicate_present"
                  }
	没有后续步骤
}
``` 

range扫描: 

```
"range_analysis": {
                  "skip_scan_range": {
                    "potential_skip_scan_indexes": [
                      {
                        "index": "idx_b",
                        "usable": false,
                        "cause": "query_references_nonkey_column"
                      }
                    ]
                  },
                  "analyzing_range_alternatives": {
					...
				}
				...
}
``` 

按代码排查, 为什么collation会影响 disjuntive_predicate_present 的判断: 

![image2021-9-3 0:5:9.png](/assets/01KJBYEM2FNRZBZA6CDYKTGYH3/image2021-9-3%200%3A5%3A9.png)

原理: 部分字符集中, ci 与 bin 的顺序是一致的, 于是可以直接比较
