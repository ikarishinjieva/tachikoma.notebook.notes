---
title: 20231018 - SQL规则代码生成 的 训练数据重新设计
confluence_page_id: 2589246
created_at: 2023-10-18T16:27:49+00:00
updated_at: 2023-10-23T03:08:18+00:00
---

# 目标

重新设计规则的描述数据, 为多种数据库设计统一的描述语言, 对于某种数据库, 进行主流程描述, 使得生成的代码逻辑性更准确

# 规则元信息

用以下信息, 描述规则库 (元信息库不开源, 由元信息库生成的各代码文件/参考文件, 按目前策略开源)

  1.      1. 规则编号: 需要新的编号系统 (DB2有编号系统)
     2. 规则种类: DDL/DML/... (遵循现在的分类, 将来需要扩充分类, 方便归纳和查找)
     3. 规则是: 
        1. 当定义表字段时, 表字段都应包括NOT NULL约束
        2. 参数列表: xxx
        3. 适用范围: 
           1. MySQL
           2. PostgreSQL
           3. DB2 不适用
     4. 该规则的主要目的是: 当在表中插入记录时, 防止插入空值, 提升查询准确性
        1. 正例举例: ... (高亮关键部分)
        2. 反例举例: ... (高亮关键部分)
           1. 危害说明: 当出现什么情况时, 会发生什么. 可链接外部知识
     5. 参与检查的语句类型: (每种数据库不同)
        1. MySQL
           1. create table ... (脱机检查)
           2. alter table add column (脱机检查)
           3. alter table modify column (脱机检查)
           4. alter table change column (脱机检查)
           5. (联机检查) 当xxx情况下, 会从数据库中获取元数据...
        2. PostgreSQL  

           1. create table ... (脱机检查)
           2. ...
        3. DB2
           1. ...  
  

     6. 检查流程描述 (给AI用, 每种数据库不同)
        1. MySQL
           1. ...

```
In MySQL, you should check if the SQL violate the rule: "Table fields must have a NOT NULL constraint". You need to write rule code by Golang and unit test code using Xml. 
You should follow the following logic:
1. For "create table" statement, check every column if it has NOT NULL constraint, otherwise, add the column name to violation-list
2. For "alter table add column" statement, check the column if it has NOT NULL constraint, otherwise, add the column name to violation-list
3. For "alter table modify column" statement, check the modified column definition if it has NOT NULL constraint, otherwise, add the column name to violation-list
4. For "alter table change column" statement, check the new column's definition if it has NOT NULL constraint, otherwise, add the column name to violation-list
5. Generate a violation message as the checking result, including column names which violate the rule, if there is any violations
``` 
        2. PostgreSQL
           1. ...
        3. DB2
           1. ...
     7. 规则已知缺陷
        1. ...
     8. TODO: SQL优化方案:  

        1. 方案1/2/3: 对于优化流程的描述 (给AI用, 每种数据库不同)
        2. 不同方案的评估比较 (针对DML)
     9. TODO: 针对DDL, 是否支持在线对象扫描 (非SQL扫描)

# 规则的开发流程

  1. 新增一个规则的元信息描述
  2. 根据元信息描述, 生成各种数据库种类的 代码文件: 
     1. rule代码
     2. 单元测试代码
     3. 界面资料库元数据
     4. ???

# 其他功能调整

  1. 报错信息中, 应带有某种结构, 用于标记错误区域, 以及将来的优化区域
  2. PG规则: 
     1. 两类Handler的使用: AstSQLHandler/RawSQLHandler 的必要性? 
     2. 将原始SQL传入AstSQLHandler中
  3. DB2规则: 
     1. DB2规则: name必须为Db2_060形式, 不合理. 使用统一编号系统?
     2. throw的错误处理太宽泛
     3. 多个SQL, create table xxx, ...(其他SQL), 会报xxx表不存在
     4. 测试: mock是按case进行的, 可以用一个统一的mock环境
  4. 多库规则的不一致
     1. rule内的报错形式不统一: RuleHandlerMap[rule.Name].Message
     2. DefaultSingleParamKeyName 的定义
     3. 把存在性检查剥离出单元测试 (测试目标是rule, 而不是mock数据的完备性)
     4. 单元测试数据格式: 统一为xml方式
     5. 单元测试: 对于联机部分和脱机部分是否需要单独测试
     6. rule的函数签名和返回值均不同
  5. 辅助函数的位置, 是否跟随主代码, 如何切割
  6. "表的列数不建议超过阈值" 需要联机检测
