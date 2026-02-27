---
title: 20240521 - 使用RAG的方法进行SQLe代码生成
confluence_page_id: 2949477
created_at: 2024-05-21T09:47:12+00:00
updated_at: 2024-05-27T02:18:52+00:00
---

# 问题定义

<https://github.com/actiontech/sqle/tree/ai-demo/sqle/driver/mysql/rule/ai> 中, 39和55是比较疑难的case, 需要长链推导

# 理论参考

  - [20240516 - 阅读论文: REACT: SYNERGIZING REASONING AND ACTING IN LANGUAGE MODELS]
  - [20240520 - 阅读论文: AceCoder: Utilizing Existing Code to Enhance Code Generation]

# 尝试1 - 进行步骤拆解

模仿REACT, 进行步骤拆解

```
你是一个协助人类写代码的AI, 你需要逐步引导人类按照进行思考, 引导人类生成符合要求的代码.

注意: 
1. 你只需要输出思考和行动, 不需要输出结果
2. 先获取同类型数据库的处理框架, 在处理其他部分
3. 对于功能的复合描述, 应当拆解开, 逐一查找样例代码
4. 如果同类型数据库中找不到合适的样例代码, 可以在其他类型数据库中进行查找

举例: 
------
要求: 生成符合如下规则的SQL检查代码: 
<规则>
在 PostgreSQL 中，您应当检查 SQL 是否违反了规则（SQLE00028）：“在表定义中，除了自增列和文本列外，每个列都应该有一个默认值。”

您应当遵循以下逻辑：
1. 对于 “CREATE TABLE ...” 语句，检查每个列，如果其类型不是 “text”，并且不是自增类型，且没有定义 DEFAULT 值，则将该列名添加到违规列表中。
2. 对于 “ALTER TABLE ... ADD COLUMN ...” 语句，检查新列，如果其类型不是 “text”，并且不是自增类型，且没有定义 DEFAULT 值，则将该列名添加到违规列表中。
3. 生成违规消息作为检查结果，包括违反规则的列名（如果有任何违规的话）。
</规则>

思考1: 我应当在同类型数据库中, 查找 `对于 “CREATE TABLE ...” 语句，检查每个列, ...的处理框架` 的样例代码
行动1: FIND("PostgreSQL", `对于 “CREATE TABLE ...” 语句，检查每个列, ...的处理框架`)
结果1: <code>...</code>

思考2: 我应当在同类型数据库中, 查找`对于某一列, 判断其类型不是 “text”`的样例代码
行动2: FIND("PostgreSQL", `对于某一列, 判断其类型不是 “text”`)
结果2: <code>...</code>

思考3: 我应当在同类型数据库中, 查找`对于某一列, 判断其类型不是自增类型`的样例代码
行动3: FIND("PostgreSQL", `对于某一列, 判断其类型不是自增类型`)
结果3: None

思考4: 由于思考3没有找到结果, 我应当在其他类型数据库中, 查找`对于某一列, 判断其类型不是自增类型`的样例代码
行动4: FIND("MySQL", `对于某一列, 判断其类型不是自增类型`)
结果4: <code>...</code>

思考5: 我应当在同类型数据库中, 查找`对于某一列, 判断其类型没有定义 DEFAULT 值`的样例代码
行动5: 找到PostgreSQL中, `对于某一列, 判断其类型没有定义 DEFAULT 值` 的样例代码
结果5: <code>...</code>

思考6: 我应当将样例代码进行整合, 完成`对于 “CREATE TABLE ...” 语句，检查每个列，如果其类型不是 “text”，并且不是自增类型，且没有定义 DEFAULT 值，则将该列名添加到违规列表中`部分
行动6: GENERATE(结果1, 结果2, 结果4, 结果5)
结果6: <code>...</code>

思考7: 我应当在同类型数据库中, 查找 `对于 “ALTER TABLE ... ADD COLUMN ...” 语句，检查新列, ...的处理框架` 的样例代码
行动7: FIND("PostgreSQL", `对于 “ALTER TABLE ... ADD COLUMN ...” 语句，检查新列, ...的处理框架`)
结果7: <code>...</code>

思考8: 我应当将样例代码进行整合, 完成`对于 “ALTER TABLE ... ADD COLUMN ...” 语句，检查新列，如果其类型不是 “text”，并且不是自增类型，且没有定义 DEFAULT 值，则将该列名添加到违规列表中`部分
行动8: GENERATE(结果7, 结果2, 结果4, 结果5)
结果8: <code>...</code>

思考9: 我应当在同类型数据库中, 查找 `生成违规消息作为检查结果，包括违反规则的列名（如果有任何违规的话）` 的样例代码
行动9: FIND("PostgreSQL", `生成违规消息作为检查结果，包括违反规则的列名（如果有任何违规的话）`)
结果9: <code>...</code>

思考10: 我应当生成最终结果
行动10: GENERATE(结果6, 结果8, 结果9)
结果10: <code>...</code>
...
------

要求: 生成符合如下规则的SQL检查代码: 
<规则>
在 PostgreSQL 中，您应该检查 SQL 是否违反了规则（SQLE00052）：“在 DDL 中，主键应该是自增的”。
您应该遵循以下逻辑：

对于 “create table ... primary key ...” 语句，表选项或列上的主键应该是自增的，否则，报告违规。
对于 “alter table ... primary key ...” 语句，表选项或列上的主键应该是自增的，否则，报告违规。
</规则>

思考1: 我应当在同类型数据库中, 查找 `对于 “create table ... primary key ...” 语句, ...的处理框架` 的样例代码
行动1: FIND("PostgreSQL", `对于 “create table ... primary key ...” 语句, ...的处理框架`)
结果1: <code>...</code>

思考2: 我应当在同类型数据库中, 查找`表选项的主键应该是自增的`的样例代码
行动2: FIND("PostgreSQL", `表选项的主键应该是自增的`)
结果2: <code>...</code>

思考3: 我应当在同类型数据库中, 查找`列上的主键应该是自增的`的样例代码
行动3: FIND("PostgreSQL", `列上的主键应该是自增的`)
结果3: None

思考4: 由于思考3没有找到结果, 我应当在其他类型数据库中, 查找`列上的主键应该是自增的`的样例代码
行动4: FIND("MySQL", `列上的主键应该是自增的`)
结果4: <code>...</code>

思考5: 我应当将样例代码进行整合, 完成`对于 “create table ... primary key ...” 语句，表选项或列上的主键应该是自增的，否则，报告违规。`部分
行动5: GENERATE(结果1, 结果2, 结果4)
结果5: <code>...</code>

...

------

要求: 生成符合如下规则的SQL检查代码: 
<规则>
在 PostgreSQL 中，您应检查 SQL 是否违反以下规则：“在 DDL 中，创建表时，表中应包含一个关于创建时间戳的字段，其默认值应为 'CURRENT_TIMESTAMP'”，创建时间戳列的名称是一个参数，默认值为 'CREATE_TIME'。
您应遵循以下逻辑：
对于“create table ...”语句，检查以下条件，如果任何条件被违反，则报告违规：
1. 表应该有一个创建时间戳列，其类型为 datetime 或 timestamp，并且列名称与参数相同
2. 创建时间戳列的默认值应配置为 'CURRENT_TIMESTAMP'
</规则>
``` 

结果: 

```
思考1: 我应当在同类型数据库中, 查找 `对于 “create table ...” 语句, ...的处理框架` 的样例代码
行动1: FIND("PostgreSQL", `对于 “create table ...” 语句, ...的处理框架`)
结果1: <code>...</code>

思考2: 我应当在同类型数据库中, 查找`表应该有一个创建时间戳列，其类型为 datetime 或 timestamp`的样例代码
行动2: FIND("PostgreSQL", `表应该有一个创建时间戳列，其类型为 datetime 或 timestamp`)
结果2: <code>...</code>

思考3: 我应当在同类型数据库中, 查找`列名称与参数相同`的样例代码
行动3: FIND("PostgreSQL", `列名称与参数相同`)
结果3: <code>...</code>

思考4: 我应当在同类型数据库中, 查找`创建时间戳列的默认值应配置为 'CURRENT_TIMESTAMP'`的样例代码
行动4: FIND("PostgreSQL", `创建时间戳列的默认值应配置为 'CURRENT_TIMESTAMP'`)
结果4: <code>...</code>

思考5: 我应当将样例代码进行整合, 完成`对于“create table ...”语句，检查以下条件，如果任何条件被违反，则报告违规`部分
行动5: GENERATE(结果1, 结果2, 结果3, 结果4)
结果5: <code>...</code>
``` 

# 尝试2 - 如何改善召回, 对已有代码进行整理 - 抽取框架

````
我有一段代码: 
```
func RuleSQLE00171(ctx context.Context, rule *driverV2.Rule, astSQL interface{}, nextSQL []string) (string, error) {
	node, ok := astSQL.(*parser.RawStmt)
	if !ok {
		return "", fmt.Errorf("invalid type of ast of rule[%v]", rule.Name)
	}

	// get expected create_time field name in config
	createTimeFieldName := rule.Params.GetParam(DefaultSingleParamKeyName).String()

	switch stmt := node.GetStmt().GetNode().(type) {
	case *parser.Node_CreateStmt:
		// "create table"
		found := false
		for _, elt := range stmt.CreateStmt.TableElts {
			if strings.EqualFold(utilGetColumnName(elt), createTimeFieldName) {
				// the column is create_time column
				if c := utilGetColumnConstraint(elt, parser.ConstrType_CONSTR_DEFAULT); nil != c {
					// the column has "DEFAULT" constraint
					if utilIsConstraintAFuncCall(c, parser.SQLValueFunctionOp_SVFOP_CURRENT_TIMESTAMP) {
						// the "DEFAULT" value is current_timestamp
						found = true
					}
				}
			}
		}
		if !found {
			return RuleHandlerMap[rule.Name].Message, nil
		}
	}
	return "", nil
}

```

你帮我把这段代码中无关的部分去除, 使其匹配描述: "对于 “create table ...” 语句, ...的处理框架"
```` 

结果: 

````
```
func RuleSQLE00171(astSQL interface{}) (bool, error) {
	node, ok := astSQL.(*parser.RawStmt)
	if !ok {
		return false, fmt.Errorf("invalid type of ast") 
	}

	switch stmt := node.GetStmt().GetNode().(type) {
	case *parser.Node_CreateStmt:
		// "create table"
		// 在此处添加对于 "create table ..." 语句的处理逻辑
		// ...

		return true, nil 
	}
	return false, nil 
}
```

**代码解释:**

1. **函数定义:**  函数 `RuleSQLE00171` 接收抽象语法树 (AST) 作为输入，并返回一个布尔值和一个错误。布尔值表示是否处理了 "create table ..." 语句。
2. **类型断言:**  代码首先检查输入的 AST 是否为 `*parser.RawStmt` 类型。如果不是，则返回错误。
3. **语句类型判断:** 使用类型 switch 语句判断 AST 中的语句类型。
4. **"create table ..." 处理:**  如果语句类型是 `*parser.Node_CreateStmt`，则表示这是一个 "create table ..." 语句。您需要在此处添加具体的处理逻辑。
5. **返回值:**  
    - 如果成功处理了 "create table ..." 语句，则返回 `true` 和 `nil`。
    - 如果输入 AST 不是 "create table ..." 语句，则返回 `false` 和 `nil`。
    - 如果出现错误，则返回 `false` 和相应的错误信息。

**注意:**  此代码框架只保留了与 "create table ..." 语句处理相关的部分，其他无关代码已被移除。您需要根据具体需求完善代码逻辑。
```` 

# 尝试2 - 如何改善召回, 对已有代码进行整理 - 抽取处理逻辑

````
我有一段代码: 
```
func RuleSQLE00055(ctx context.Context, rule *driverV2.Rule, sql string, nextSQL []string) (string, error) {
	node, err := sqlParserFuncV2(sql)
	if err != nil {
		return "", errors.Wrap(err, "parse sql")
	}

	switch stmt := node.GetStmt().GetNode().(type) {
	case *parser.Node_IndexStmt:
		// "create index ..."
		schemaName := stmt.IndexStmt.GetRelation().GetSchemaname()
		tableName := stmt.IndexStmt.GetRelation().GetRelname()

		newIndexColNames := []string{}
		existIndexColNames := [][]string{}

		// get new index column names
		for _, elt := range stmt.IndexStmt.IndexParams {
			newIndexColNames = append(newIndexColNames, utilGetIndexColumnName(elt))
		}

		// get exist index column names
		indexInfo, err := utilGetTableIndexInfo(ctx, schemaName, tableName)
		if err != nil {
			return "", err
		}
		for _, idx := range indexInfo {
			existIndexColNames = append(existIndexColNames, idx.ColumnList)
		}

		// check the index is not duplicated
		for _, source := range calculateNewIndexRedundant(existIndexColNames, [][]string{newIndexColNames}) {
			return fmt.Sprintf(RuleHandlerMap[rule.Name].Message, existIndexColNames[source], newIndexColNames), nil
		}
	}
	return "", nil
}

```

你帮我把这段代码中无关的部分去除, 或者摘取其中代码片段, 使其匹配描述: "检查某个索引与已有索引之间是否重复"
````
````
```
// ... other code ...

		// get new index column names
		for _, elt := range stmt.IndexStmt.IndexParams {
			newIndexColNames = append(newIndexColNames, utilGetIndexColumnName(elt))
		}

		// get exist index column names
		indexInfo, err := utilGetTableIndexInfo(ctx, schemaName, tableName)
		if err != nil {
			return "", err
		}
		for _, idx := range indexInfo {
			existIndexColNames = append(existIndexColNames, idx.ColumnList)
		}

		// check the index is not duplicated
		for _, source := range calculateNewIndexRedundant(existIndexColNames, [][]string{newIndexColNames}) {
			return fmt.Sprintf(RuleHandlerMap[rule.Name].Message, existIndexColNames[source], newIndexColNames), nil
		}

// ... other code ...
```

这段代码片段实现了 "检查某个索引与已有索引之间是否重复" 的功能。

**解释：**

1.  **获取新索引的列名:**
    -   遍历 `stmt.IndexStmt.IndexParams` 获取新索引的每一列。
    -   使用 `utilGetIndexColumnName` 函数获取列名，并将列名添加到 `newIndexColNames` 列表中。

2.  **获取已有索引的列名:**
    -   使用 `utilGetTableIndexInfo` 函数获取指定 schema 和 table 的所有索引信息。
    -   遍历 `indexInfo` 获取每个已有索引的列名列表，并将列名列表添加到 `existIndexColNames` 列表中。

3.  **检查索引是否重复:**
    -   使用 `calculateNewIndexRedundant` 函数比较 `existIndexColNames` 和 `newIndexColNames`，判断新索引是否与已有索引重复。
    -   如果存在重复索引，则返回包含重复索引信息的错误消息。

这段代码片段的核心逻辑在于 `calculateNewIndexRedundant` 函数，该函数用于判断新索引的列组合是否与已有索引的列组合相同，从而判断索引是否重复。
````

```
对以上代码进行一句话的健全的描述, 这个描述将用于搜索这段代码, 再详细一点
```

```
这段代码解析SQL语句，提取新索引的列名，并与数据库中已有的索引进行比较，判断新索引的列组合是否与已有索引重复，以防止创建冗余索引。
``` 

# TODO

  1. 用REACT对已有规则进行拆解
  2. 对于拆解的每一步, 对已有代码进行拆解, 做成待召回的文档
  3. 将待召回的文档, 放在一个召回->生成的流程中, 生成一次PG的规则39

# Rule-39

英文: 

```
In MySQL, you should check if the SQL violate the rule(SQLE00039): "For index, It is suggested that the discrimination of the index field is greater than the percentage threshold. the threshold should be a parameter whose default value is 0.7.".
You should follow the following logic:
1. For the  "CREATE INDEX ..." statements, check if discrimination of the index which on the table option or on the column less than the threshold. If it does, report a violation.
The calculation of index discrimination(eg: select 1- (
                select count(*) as record_count from (select id from t1 limit 50000) group by id order by record_count desc limit 1
             )/ (select count(*) from t1);) needs to be obtained from the online database.
2. For the  "ALTER TABLE ... ADD INDEX ..." statements, perform the same check as above.
3. For SELECT clauses in all DML statements, if the where condition is not always true,
   1. Define set A
   2. Add the condition fields used in the WHERE clause to the set A
   3. Connect to the database and put the indexed fields in set B based on the fields in set A
   4. Calculate the discrimination of each field in set B, if the discrimination is less than the rule variable selectivity value, then report the rule violation.
	 The calculation of field discrimination(eg: select 1- (
                select count(*) as record_count from (select id from t1 limit 50000) group by id order by record_count desc limit 1
             )/ (select count(*) from t1);) needs to be obtained from the online database.
``` 

中文:

````
在 MySQL 中，您应该检查 SQL 是否违反了规则 (SQLE00039)：“对于索引，建议索引字段的区分度大于百分比阈值。该阈值应该是一个参数，默认值为 0.7。” 您应该按照以下逻辑进行检查：

1. 对于 "CREATE INDEX ..." 语句，检查表选项或列上的索引的区分度是否低于阈值。如果是，则报告违规。
   区分度的计算（例如：
   ```sql
   select 1 - (
     select count(*) as record_count from (select id from t1 limit 50000) group by id order by record_count desc limit 1
   ) / (select count(*) from t1);
   ```
   ）需要从在线数据库中获得。

2. 对于 "ALTER TABLE ... ADD INDEX ..." 语句，执行与上述相同的检查。

3. 对于所有 DML 语句中的 SELECT 子句，如果 where 条件不总是为真：
   1. 定义集合 A
   2. 将 WHERE 子句中使用的条件字段添加到集合 A
   3. 连接到数据库，并根据集合 A 中的字段将索引字段放入集合 B
   4. 计算集合 B 中每个字段的区分度，如果区分度低于规则变量选择性值，则报告规则违规。
      字段区分度的计算（例如：
      ```sql
      select 1 - (
        select count(*) as record_count from (select id from t1 limit 50000) group by id order by record_count desc limit 1
      ) / (select count(*) from t1);
      ```
      ）需要从在线数据库中获得。
```` 

### 第一步: 拆解: 

````
你是一个协助人类写代码的AI, 你需要对人类提出的需求 拆解成 多个 子需求, 我会使用这些 子需求 来搜索类似的代码. 

拆解过程需要遵循如下原则: 
1. 每个子需求 是一句独立的描述, 形成一个搜索条件
1. 每个子需求 不应丢失原始文档中的信息, 比如举例或代码等

以下是样例:
<举例>
要求: 生成符合如下规则的SQL检查代码: 
<规则>
在 PostgreSQL 中，您应当检查 SQL 是否违反了规则（SQLE00028）：“在表定义中，除了自增列和文本列外，每个列都应该有一个默认值。”

您应当遵循以下逻辑：
1. 对于 “CREATE TABLE ...” 语句，检查每个列，如果其类型不是 “text”，并且不是自增类型，且没有定义 DEFAULT 值，则将该列名添加到违规列表中。
2. 对于 “ALTER TABLE ... ADD COLUMN ...” 语句，检查新列，如果其类型不是 “text”，并且不是自增类型，且没有定义 DEFAULT 值，则将该列名添加到违规列表中。
3. 生成违规消息作为检查结果，包括违反规则的列名（如果有任何违规的话）。
</规则>

拆解: 
- "对于 “CREATE TABLE ...” 语句，检查每个列, ..."
- "对于 “ALTER TABLE ... ADD COLUMN ...” 语句，检查新列, ..."
- "如果某一列类型不是 “text”, ..."
- "如果某一列不是自增类型, ..."
- "如果某一列没有定义 DEFAULT 值, ..."
- "该列名添加到违规列表中"
- "生成违规消息作为检查结果，包括违反规则的列名"
</举例>

<举例>
在 MySQL 中，您应该检查 SQL 是否违反了规则(SQLE00015): "在 DDL 中，应使用指定的排序规则，指定的排序规则应为一个参数，默认值为 utf8mb4_0900_ai_ci。" 您应遵循以下逻辑：

1. 对于 "create table ... collate= ..." 语句，检查表选项或列上的 collate 应与指定的排序规则相同，否则，报告违规。
2. 对于 "alter table ... collate= ..." 语句，检查表选项或列上的 collate 应与指定的排序规则相同，否则，报告违规。
3. 对于 "create database ... collate= ..." 语句，检查 collate 应与指定的排序规则相同，否则，报告违规。
4. 对于 "alter database ... collate= ..." 语句，检查 collate 应与指定的排序规则相同，否则，报告违规。

拆解: 
- 对于 "create table ... collate= ..." 语句，...
- 对于 "create table ... collate= ..." 语句，...
- 对于 "alter table ... collate= ..." 语句，...
- 对于 "create database ... collate= ..." 语句，...
- 对于 "alter database ... collate= ..." 语句，...
- 检查表选项的collate, 应与指定的排序规则相同，否则，报告违规
- 检查列上的 collate, 应与指定的排序规则相同，否则，报告违规
- 检查 collate, 应与指定的排序规则相同，否则，报告违规
</举例>

要求: 生成符合如下规则的SQL检查代码: 
<规则>
在 MySQL 中，您应该检查 SQL 是否违反了规则 (SQLE00039)：“对于索引，建议索引字段的区分度大于百分比阈值。该阈值应该是一个参数，默认值为 0.7。” 您应该按照以下逻辑进行检查：

1. 对于 "CREATE INDEX ..." 语句，检查表选项或列上的索引的区分度是否低于阈值。如果是，则报告违规。
   区分度的计算（例如：
   ```sql
   select 1 - (
     select count(*) as record_count from (select id from t1 limit 50000) group by id order by record_count desc limit 1
   ) / (select count(*) from t1);
   ```
   ）需要从在线数据库中获得。

2. 对于 "ALTER TABLE ... ADD INDEX ..." 语句，执行与上述相同的检查。

3. 对于所有 DML 语句中的 SELECT 子句，如果 where 条件不总是为真：
   1. 定义集合 A
   2. 将 WHERE 子句中使用的条件字段添加到集合 A
   3. 连接到数据库，并根据集合 A 中的字段将索引字段放入集合 B
   4. 计算集合 B 中每个字段的区分度，如果区分度低于规则变量选择性值，则报告规则违规。
      字段区分度的计算（例如：
      ```sql
      select 1 - (
        select count(*) as record_count from (select id from t1 limit 50000) group by id order by record_count desc limit 1
      ) / (select count(*) from t1);
      ```
      ）需要从在线数据库中获得。
</规则>

拆解:
````

```
拆解: 
- 对于 "CREATE INDEX ..." 语句，...
- 对于 "ALTER TABLE ... ADD INDEX ..." 语句，...
- 检查表选项或列上的索引的区分度是否低于阈值, 如果是，则报告违规
- 对于所有 DML 语句中的 SELECT 子句，如果 where 条件不总是为真：...
- 将 WHERE 子句中使用的条件字段添加到集合 A
- 连接到数据库，并根据集合 A 中的字段将索引字段放入集合 B
- 计算集合 B 中每个字段的区分度，如果区分度低于规则变量选择性值，则报告规则违规
- 区分度的计算需要从在线数据库中获得 (提供区分度计算的SQL代码示例) 
``` 

### 拆解规则39的源代码:

````
我有一段代码: 
```
func RuleSQLE00039(input *rulepkg.RuleHandlerInput) error {
	// get expected param value
	param := input.Rule.Params.GetParam(rulepkg.DefaultSingleParamKeyName)
	if param == nil {
		return fmt.Errorf("param %s not found", rulepkg.DefaultSingleParamKeyName)
	}
	threshold := param.Float64()
	if threshold > 1 || threshold <= 0 {
		return fmt.Errorf("param %s should be in range (0, 1]", rulepkg.DefaultSingleParamKeyName)
	}

	var tableName *ast.TableName
	indexColumns := make([]string, 0)
	switch stmt := input.Node.(type) {
	case *ast.CreateIndexStmt:
		// "create index..."

		tableName = stmt.Table
		for _, col := range stmt.IndexPartSpecifications {
			//"create index... column..."
			indexColumns = append(indexColumns, util.GetIndexColName(col))
		}

		if len(indexColumns) == 0 {
			// the table has no index
			return nil
		}

		discrimination, err := util.CalculateIndexDiscrimination(input.Ctx, tableName, indexColumns)
		if err != nil {
			log.NewEntry().Errorf("get index discrimination failed, sqle: %v, error: %v", input.Node.Text(), err)
			return nil
		}

		// the table has index, check the discrimination
		for col, d := range discrimination {
			if d < threshold {
				rulepkg.AddResult(input.Res, input.Rule, SQLE00039, col, threshold)
			}
		}
	case *ast.AlterTableStmt:
		// "alter table"
		tableName = stmt.Table

		for _, spec := range util.GetAlterTableCommandsByTypes(stmt, ast.AlterTableAddConstraint) {
			// "alter table... add index..."
			constraints := util.GetTableConstraints([]*ast.Constraint{spec.Constraint}, util.GetIndexConstraintTypes()...)

			for _, constraint := range constraints {
				for _, col := range constraint.Keys {
					indexColumns = append(indexColumns, util.GetIndexColName(col))
				}
			}

		}
		if len(indexColumns) == 0 {
			// the table has no index
			return nil
		}

		discrimination, err := util.CalculateIndexDiscrimination(input.Ctx, tableName, indexColumns)
		if err != nil {
			log.NewEntry().Errorf("get index discrimination failed, sqle: %v, error: %v", input.Node.Text(), err)
			return nil
		}

		// the table has index, check the discrimination
		for col, d := range discrimination {
			if d < threshold {
				rulepkg.AddResult(input.Res, input.Rule, SQLE00039, col, threshold)
			}
		}
	case *ast.SelectStmt, *ast.UpdateStmt, *ast.DeleteStmt, *ast.InsertStmt:
		// "select..."

		var defaultTable string
		var alias []*util.TableAliasInfo
		getTableName := func(col *ast.ColumnNameExpr) string {
			if col.Name.Table.L != "" {
				for _, a := range alias {
					if a.TableAliasName == col.Name.Table.String() {
						return a.TableName
					}
				}
				return col.Name.Table.L
			}
			return defaultTable
		}

		for _, selectStmt := range util.GetSelectStmt(stmt) {

			// get default table name
			if t := util.GetDefaultTable(selectStmt); t != nil {
				defaultTable = t.Name.L
			}

			// get table alias info
			if selectStmt.From != nil && selectStmt.From.TableRefs != nil {
				alias = util.GetTableAliasInfoFromJoin(selectStmt.From.TableRefs)
			}

			var (
				table2colNames = map[string] /*table name*/ []*ast.ColumnName /*col names*/ {}
			)

			// get column names in where condition
			for _, col := range util.GetColumnNameInExpr(selectStmt.Where) {
				table2colNames[getTableName(col)] = append(table2colNames[getTableName(col)], col.Name)
			}

			for table, colNames := range table2colNames {
				// get index of the table
				indexesInfo, err := util.GetTableIndexes(input.Ctx, table, colNames[0].Schema.L)
				if err != nil {
					log.NewEntry().Errorf("get table indexes failed, sqle: %v, error: %v", input.Node.Text(), err)
					return nil
				}

				// get index columns
				indexColumns := make([]string, 0)
				for _, colName := range colNames {

					for _, index := range indexesInfo {
						// check if the column is index
						if colName.Name.String() == index.ColumnName {
							indexColumns = append(indexColumns, colName.Name.String())
						}
					}
				}

				if len(indexColumns) == 0 {
					// no index
					return nil
				}

				tableName = &ast.TableName{
					Schema: colNames[0].Schema,
					Name:   model.NewCIStr(table),
				}
				discrimination, err := util.CalculateIndexDiscrimination(input.Ctx, tableName, indexColumns)
				if err != nil {
					log.NewEntry().Errorf("get index discrimination failed, sqle: %v, error: %v", input.Node.Text(), err)
					return nil
				}

				// has index, check the discrimination
				for col, d := range discrimination {
					if d < threshold {
						rulepkg.AddResult(input.Res, input.Rule, SQLE00039, col, threshold)
					}
				}

			}
		}

	}

	return nil
}

```

你需要: 
1. 保留函数的签名和框架, 把这段代码中无关的部分去除, 或者摘取其中代码片段, 或者摘取代码框架, 使其匹配描述.
2. 描述中的"..."代表此处将来需要填写其他代码. 
3. 不要修改或增加源代码中的内容, 仅做删除或摘取
4. 如果你找不到符合描述的结果, 可以输出"我没办法"

描述: `对于 "CREATE INDEX ..." 语句，...`
代码:
````

````
```go
func RuleSQLE00039(input *rulepkg.RuleHandlerInput) error {
	// ...

	switch stmt := input.Node.(type) {
	case *ast.CreateIndexStmt:
		// "create index..."
		// ...
	case // ...
		// ...
	}
	// ...
	return nil
}
```
```` 

其他拆解: 

````
对于 "ALTER TABLE ... ADD INDEX ..." 语句，...
 
---
 
```go
func RuleSQLE00039(input *rulepkg.RuleHandlerInput) error {
	// ...

	switch stmt := input.Node.(type) {
	// ...
	case *ast.AlterTableStmt:
		// "alter table"
		tableName = stmt.Table

		for _, spec := range util.GetAlterTableCommandsByTypes(stmt, ast.AlterTableAddConstraint) {
			// "alter table... add index..."
			// ...
		}
		// ...
	// ...
	}
	// ...
}
```

这段代码片段展示了 `RuleSQLE00039` 函数中处理 `ALTER TABLE ... ADD INDEX ...` 语句的部分。

1. **`switch stmt := input.Node.(type)`**:  这段代码使用类型断言，根据 `input.Node` 的具体类型选择不同的处理分支。
2. **`case *ast.AlterTableStmt`**:  如果 `input.Node` 是 `*ast.AlterTableStmt` 类型，说明当前语句是 `ALTER TABLE` 语句，进入该分支进行处理。
3. **`for _, spec := range util.GetAlterTableCommandsByTypes(stmt, ast.AlterTableAddConstraint)`**:  这行代码遍历 `ALTER TABLE` 语句中所有类型为 `ast.AlterTableAddConstraint` 的操作，也就是添加约束的操作。 `ADD INDEX` 语句会被解析成添加索引约束。
4. **`// "alter table... add index..."`**:  这里省略了具体的处理逻辑，你可以在这里添加针对 `ADD INDEX` 语句的处理代码。 
   
   你需要补充的代码应该完成以下功能：
   * 从 `spec` 中获取索引的信息，例如索引名、索引列等。
   * 根据索引信息计算索引区分度。
   * 判断索引区分度是否低于阈值，如果低于则记录违规信息。 
   
   你可以参考 `case *ast.CreateIndexStmt` 分支的代码实现类似的逻辑。
5. **// ...**:  这里省略了其他分支和后续的处理逻辑。
````

````
检查表选项或列上的索引的区分度是否低于阈值, 如果是，则报告违规
 
---
 
```go
func RuleSQLE00039(input *rulepkg.RuleHandlerInput) error {
	// get expected param value
	param := input.Rule.Params.GetParam(rulepkg.DefaultSingleParamKeyName)
	if param == nil {
		return fmt.Errorf("param %s not found", rulepkg.DefaultSingleParamKeyName)
	}
	threshold := param.Float64()
	if threshold > 1 || threshold <= 0 {
		return fmt.Errorf("param %s should be in range (0, 1]", rulepkg.DefaultSingleParamKeyName)
	}

	var tableName *ast.TableName
	indexColumns := make([]string, 0)

	// ... 获取表名和索引列 ...

	if len(indexColumns) == 0 {
		// the table has no index
		return nil
	}

	discrimination, err := util.CalculateIndexDiscrimination(input.Ctx, tableName, indexColumns)
	if err != nil {
		log.NewEntry().Errorf("get index discrimination failed, sqle: %v, error: %v", input.Node.Text(), err)
		return nil
	}

	// the table has index, check the discrimination
	for col, d := range discrimination {
		if d < threshold {
			rulepkg.AddResult(input.Res, input.Rule, SQLE00039, col, threshold)
		}
	}

	return nil
}
```

**代码解释:**

1. **获取阈值**:
   - 从 `input.Rule.Params` 中获取名为 `rulepkg.DefaultSingleParamKeyName` 的参数值。
   - 检查参数值是否存在，并将其转换为 `float64` 类型的阈值。
   - 验证阈值是否在 (0, 1] 范围内。

2. **获取表名和索引列**:
   - 这部分代码需要根据具体的 SQL 语句类型进行分析，提取出表名和索引列。
   - 在原始代码中，使用了 `switch` 语句处理了 `CreateIndexStmt`、`AlterTableStmt` 和 `SelectStmt` 等多种情况。

3. **计算索引区分度**:
   - 如果索引列不为空，调用 `util.CalculateIndexDiscrimination` 函数计算索引区分度。
   - 如果计算过程中出现错误，记录错误日志并返回。

4. **检查区分度并报告违规**:
   - 遍历索引区分度结果，如果某个列的区分度低于阈值，则调用 `rulepkg.AddResult` 函数报告违规。

**需要补充的代码:**

需要根据具体的 SQL 语句类型，补充获取表名和索引列的代码。例如，对于 `CREATE INDEX` 语句，可以使用以下代码：

```go
case *ast.CreateIndexStmt:
	tableName = stmt.Table
	for _, col := range stmt.IndexPartSpecifications {
		indexColumns = append(indexColumns, util.GetIndexColName(col))
	}
```
````

````
对于所有 DML 语句中的 SELECT 子句，如果 where 条件不是永真式：...
 
------
 
```go
func RuleSQLE00039(input *rulepkg.RuleHandlerInput) error {
	// ...

	switch stmt := input.Node.(type) {
	// ...
	case *ast.SelectStmt, *ast.UpdateStmt, *ast.DeleteStmt, *ast.InsertStmt:
		// "select..."

		for _, selectStmt := range util.GetSelectStmt(stmt) {
			// ...

			// 如果 where 条件不是永真式
			if !util.IsWhereClauseAlwaysTrue(selectStmt.Where) {
				// ...
			}

			// ...
		}

	}

	return nil
}
```

**代码解释:**

1. **`...`**:  代表代码中被省略的部分，保留了原函数的签名和框架。
2. **`switch stmt := input.Node.(type)`**:  根据输入语句类型进行判断。
3. **`case *ast.SelectStmt, *ast.UpdateStmt, *ast.DeleteStmt, *ast.InsertStmt`**: 处理所有包含 SELECT 子句的 DML 语句，包括 SELECT, UPDATE, DELETE 和 INSERT。
4. **`for _, selectStmt := range util.GetSelectStmt(stmt)`**:  遍历语句中所有 SELECT 子句。
5. **`util.IsWhereClauseAlwaysTrue(selectStmt.Where)`**: 判断 WHERE 条件是否是永真式，例如 `WHERE 1=1`。
6. **`if !util.IsWhereClauseAlwaysTrue(selectStmt.Where)`**: 如果 WHERE 条件不是永真式，则执行 `...` 中的代码。

**需要补充的部分:**

你需要在 `if !util.IsWhereClauseAlwaysTrue(selectStmt.Where)` 语句块中添加代码，实现针对非永真 WHERE 条件的处理逻辑。
````

````
将 WHERE 子句中使用的条件字段添加到集合 A
 
------
 
```go
func RuleSQLE00039(input *rulepkg.RuleHandlerInput) error {
	// ...

	case *ast.SelectStmt, *ast.UpdateStmt, *ast.DeleteStmt, *ast.InsertStmt:
		// ...

		for _, selectStmt := range util.GetSelectStmt(stmt) {
			// ...

			var (
				table2colNames = map[string] /*table name*/ []*ast.ColumnName /*col names*/ {}
			)

			// 将 WHERE 子句中使用的条件字段添加到集合 A
			for _, col := range util.GetColumnNameInExpr(selectStmt.Where) { 
				table2colNames[getTableName(col)] = append(table2colNames[getTableName(col)], col.Name)
			}

			// ...
		}

	}

	return nil
}
```

**代码解释:**

1. **`for _, selectStmt := range util.GetSelectStmt(stmt)`**:  遍历获取所有 `SelectStmt` 语句，因为 `Update`, `Delete`, `Insert` 语句内部也可能包含 `SelectStmt`。
2. **`table2colNames := map[string][]*ast.ColumnName{}`**:  定义一个 map 用于存储 表名 到 字段列表 的映射关系。
3. **`for _, col := range util.GetColumnNameInExpr(selectStmt.Where)`**: 遍历 `WHERE` 子句中使用的所有字段。
4. **`table2colNames[getTableName(col)] = append(table2colNames[getTableName(col)], col.Name)`**: 将当前字段名添加到对应的表名下的字段列表中。 `getTableName(col)` 函数用于获取字段所属的表名。

**总结:**

这段代码片段的功能是从 `WHERE` 子句中提取所有使用的字段，并将它们按照所属表名进行分组存储。
````

````
从集合A(WHERE 子句中使用的条件字段)中, 筛选出数据库中索引使用到的列, 放在集合B中
 
------
 
```go
func RuleSQLE00039(input *rulepkg.RuleHandlerInput) error {
	...
	case *ast.SelectStmt, *ast.UpdateStmt, *ast.DeleteStmt, *ast.InsertStmt:
		// "select..."

		...

		for _, selectStmt := range util.GetSelectStmt(stmt) {

			...

			var (
				table2colNames = map[string] /*table name*/ []*ast.ColumnName /*col names*/ {}
			)

			// get column names in where condition
			for _, col := range util.GetColumnNameInExpr(selectStmt.Where) {
				table2colNames[getTableName(col)] = append(table2colNames[getTableName(col)], col.Name)
			}

			for table, colNames := range table2colNames {
				...
				for _, colName := range colNames {

					for _, index := range indexesInfo {
						// check if the column is index
						if colName.Name.String() == index.ColumnName {
							indexColumns = append(indexColumns, colName.Name.String())
						}
					}
				}

				...
			}
		}

	}

	return nil
}
```
````

````
计算集合(数据库中索引使用到的列) 中每个字段的区分度，如果区分度低于规则变量选择性值，则报告规则违规
 
------
 
```go
func RuleSQLE00039(input *rulepkg.RuleHandlerInput) error {
	// ...

	threshold := param.Float64()
	// ...
	
	// ... 

		discrimination, err := util.CalculateIndexDiscrimination(input.Ctx, tableName, indexColumns)
		if err != nil {
			// ...
			return nil
		}

		// the table has index, check the discrimination
		for col, d := range discrimination {
			if d < threshold {
				rulepkg.AddResult(input.Res, input.Rule, SQLE00039, col, threshold)
			}
		}
	// ...
	return nil
}
```
````

```
区分度的计算需要从在线数据库中获得
 
------
 
func RuleSQLE00039(input *rulepkg.RuleHandlerInput) error {
	// ...

		discrimination, err := util.CalculateIndexDiscrimination(input.Ctx, tableName, indexColumns)
		if err != nil {
			log.NewEntry().Errorf("get index discrimination failed, sqle: %v, error: %v", input.Node.Text(), err)
			return nil
		}

	// ...
}
``` 

### 查找一些PG的规则, 找到相关片段

```
PG规则43
 
---
 
**翻译：**
在 PostgreSQL 中，你应该检查 SQL 是否违反了规则(SQLE00043)：“对于创建表和创建索引的语句，同一列上的索引数量应该在阈值之内，该阈值应该是一个参数，默认值为2。”
你应该遵循以下逻辑：
1. 对于“CREATE INDEX...”语句，连接到数据库，获取目标表中的索引信息，结合新的索引，统计每个索引中的字段，当同一字段在不同索引中出现的次数超过阈值时，将报告违反规则。
 
---
 
对于“CREATE INDEX...”语句，...
 
---
 
func RuleSQLE00043(ctx context.Context, rule *driverV2.Rule, sql string, nextSQL []string) (string, error) {
	node, err := sqlParserFuncV2(sql)
	if err != nil {
		errors.Wrap(err, "parse sql")
	}

	...

	switch stmt := node.GetStmt().GetNode().(type) {
	case *parser.Node_IndexStmt:
		// "create index ..."
		...
	}
	return "", nil
}
```

````
pg旧规则15
------
 
对于 "ALTER TABLE ... ADD INDEX ..." 语句，...
 
------
 
```
func rule15(ctx context.Context, rule *driverV2.Rule, astSQL interface{}, nextSQL []string) (string, error) {
	..., ok := astSQL.(*parser.RawStmt)
	if !ok {
		...
	}

	...
	switch stmt := ...; ... {
	case ...: //...
		...
	case ...: //...
		...
	case ...: //...
		...
	case ...: //...
		...
	case ...: //...
		...
	case ...: //...
		for _, cmd := range stmt.AlterTableStmt.GetCmds() {
			alterTableCmd := cmd.GetAlterTableCmd()
			if alterTableCmd.GetSubtype() == parser.AlterTableType_AT_AddColumn {
				columnDef := alterTableCmd.GetDef().GetColumnDef()
				if columnDef.GetColname() == "index" {
					//ALTER TABLE tablename ADD INDEX [indexname] ...;
					...
				} else {
					...
				}
			} ...
		}
	case ...: //...
		...
	}
	...
	return "", nil
}
```
````

````
PG规则 08
 
------
**翻译：**
在 PostgreSQL 中，您应检查 SQL 是否违反了规则（SQLE00008）：“在表定义中，必须存在主键。”
您应遵循以下逻辑：
1. 对于“create table ... primary key...”语句，如果表选项或列上不存在主键，则报告违规情况。
 
------
 
拆解: 
- "对于 “create table ... primary key...” 语句，..."
- "如果表选项上不存在主键，则报告违规情况"
- "如果列上不存在主键，则报告违规情况"
 
------
 
如果表选项上不存在主键，则报告违规情况
 
```
func RuleSQLE00008(ctx context.Context, rule *driverV2.Rule, astSQL interface{}, nextSQL []string) (string, error) {
	...
	case *parser.Node_CreateStmt:

		...

		found := false
		// check primary key in column definition
		for _, elt := range stmt.CreateStmt.TableElts {
			if utilIsColumnHasConstraint(elt, parser.ConstrType_CONSTR_PRIMARY) {
				found = true
				break
			}
		}

		// check primary key in table constraint
		if utilIsTableHasConstraint(stmt.CreateStmt.TableElts, parser.ConstrType_CONSTR_PRIMARY) {
			found = true
		}

		if !found {
			return RuleHandlerMap[rule.Name].Message, nil
		}
	...
}
```
 
---
 
如果列上不存在主键，则报告违规情况
 
```
func RuleSQLE00008(ctx context.Context, rule *driverV2.Rule, astSQL interface{}, nextSQL []string) (string, error) {
	...
	case *parser.Node_CreateStmt:

		...

		found := false
		// check primary key in column definition
		for _, elt := range stmt.CreateStmt.TableElts {
			if utilIsColumnHasConstraint(elt, parser.ConstrType_CONSTR_PRIMARY) {
				found = true
				break
			}
		}

		...

		if !found {
			return RuleHandlerMap[rule.Name].Message, nil
		}
	...
	return "", nil
}
```
````

````
PG规则98
---
 
在 PostgreSQL 中，您应当检查 SQL 是否违反了规则（SQLE00098）：“对于 DML，避免在单条 SQL 语句中多次连接或查询同一个表”。阈值应该是一个参数，默认值为 3。
您应当遵循以下逻辑：
1. 对于所有 DML 语句中的 SELECT 子句，检查 SELECT 中的 FROM 子句以获取涉及的表，记录同一个表出现的次数，如果次数大于或等于阈值，则报告规则违规。
 
---
 
拆解:
- 对于所有 DML 语句中的 SELECT 子句，...
- 检查 SELECT 中的 FROM 子句以获取涉及的表，...
- 记录同一个表出现的次数，...
- 如果次数大于或等于阈值，则报告规则违规。
 
---
 
- 对于所有 DML 语句中的 SELECT 子句，...
 
---
 
```
func RuleSQLE00098(ctx context.Context, rule *driverV2.Rule, astSQL interface{}, nextSQL []string) (string, error) {
	..., ok := astSQL.(*parser.RawStmt)
	...
	
	...
	case *parser.Node_SelectStmt, ..., *parser.Node_InsertStmt, *parser.Node_UpdateStmt, *parser.Node_DeleteStmt:
		...
		for _, selectStmt := range utilGetSelectStmt(node.GetStmt()) {
			...
		}
		...
	}
	...
}
```
```` 

拼接:

````
你是一个生成代码的AI, 需要实现以下需求的代码.

<需求> 
要求: 生成符合如下规则的SQL检查代码:
<规则>
在 PostgreSQL 中，您应该检查 SQL 是否违反了规则 (SQLE00039)：“对于索引，建议索引字段的区分度大于百分比阈值。该阈值应该是一个参数，默认值为 0.7。” 您应该按照以下逻辑进行检查：

1. 对于 "CREATE INDEX ..." 语句，检查表选项或列上的索引的区分度是否低于阈值。如果是，则报告违规。
   区分度的计算（例如：
   ```sql
   select 1 - (
     select count(*) as record_count from (select id from t1 limit 50000) group by id order by record_count desc limit 1
   ) / (select count(*) from t1);
   ```
   ）需要从在线数据库中获得。

2. 对于 "ALTER TABLE ... ADD INDEX ..." 语句，执行与上述相同的检查。

3. 对于所有 DML 语句中的 SELECT 子句，如果 where 条件不总是为真：
   1. 定义集合 A
   2. 将 WHERE 子句中使用的条件字段添加到集合 A
   3. 连接到数据库，并根据集合 A 中的字段将索引字段放入集合 B
   4. 计算集合 B 中每个字段的区分度，如果区分度低于规则变量选择性值，则报告规则违规。
      字段区分度的计算（例如：
      ```sql
      select 1 - (
        select count(*) as record_count from (select id from t1 limit 50000) group by id order by record_count desc limit 1
      ) / (select count(*) from t1);
      ```
      ）需要从在线数据库中获得。
</规则>

</需求> 

根据这个需求, 你检索了已有的代码库, 找到了一些相关的代码片段和这些片段的作用, 注意其中有一些代码片段属于不同的数据库类型, 不同数据库类型的语法树不一样 (对ast的处理会不同).

你可以参考这些代码片段来实现需求.

<代码片段> 

MySQL中, 检查表选项或列上的索引的区分度是否低于阈值, 如果是，则报告违规

```go
func RuleSQLE00039(input *rulepkg.RuleHandlerInput) error {
    // get expected param value
    param := input.Rule.Params.GetParam(rulepkg.DefaultSingleParamKeyName)
    if param == nil {
        return fmt.Errorf("param %s not found", rulepkg.DefaultSingleParamKeyName)
    }
    threshold := param.Float64()
    if threshold > 1 || threshold <= 0 {
        return fmt.Errorf("param %s should be in range (0, 1]", rulepkg.DefaultSingleParamKeyName)
    }

    var tableName *ast.TableName
    indexColumns := make([]string, 0)

    // ... 获取表名和索引列 ...

    if len(indexColumns) == 0 {
        // the table has no index
        return nil
    }

    discrimination, err := util.CalculateIndexDiscrimination(input.Ctx, tableName, indexColumns)
    if err != nil {
        log.NewEntry().Errorf("get index discrimination failed, sqle: %v, error: %v", input.Node.Text(), err)
        return nil
    }

    // the table has index, check the discrimination
    for col, d := range discrimination {
        if d < threshold {
            rulepkg.AddResult(input.Res, input.Rule, SQLE00039, col, threshold)
        }
    }

    return nil
}
```

------

MySQL中, 对于所有 DML 语句中的 SELECT 子句，如果 where 条件不是永真式：...

```go
func RuleSQLE00039(input *rulepkg.RuleHandlerInput) error {
    // ...

    switch stmt := input.Node.(type) {
    // ...
    case *ast.SelectStmt, *ast.UpdateStmt, *ast.DeleteStmt, *ast.InsertStmt:
        // "select..."

        for _, selectStmt := range util.GetSelectStmt(stmt) {
            // ...

            // 如果 where 条件不是永真式
            if !util.IsWhereClauseAlwaysTrue(selectStmt.Where) {
                // ...
            }

            // ...
        }

    }

    return nil
}
```

------

MySQL中, 将 WHERE 子句中使用的条件字段添加到集合 A

```go
func RuleSQLE00039(input *rulepkg.RuleHandlerInput) error {
    // ...

    case *ast.SelectStmt, *ast.UpdateStmt, *ast.DeleteStmt, *ast.InsertStmt:
        // ...

        for _, selectStmt := range util.GetSelectStmt(stmt) {
            // ...

            var (
                table2colNames = map[string] /*table name*/ []*ast.ColumnName /*col names*/ {}
            )

            // 将 WHERE 子句中使用的条件字段添加到集合 A
            for _, col := range util.GetColumnNameInExpr(selectStmt.Where) {
                table2colNames[getTableName(col)] = append(table2colNames[getTableName(col)], col.Name)
            }

            // ...
        }

    }

    return nil
}
```

------

MySQL中, 从集合A(WHERE 子句中使用的条件字段)中, 筛选出数据库中索引使用到的列, 放在集合B中

```go
func RuleSQLE00039(input *rulepkg.RuleHandlerInput) error {
    ...
    case *ast.SelectStmt, *ast.UpdateStmt, *ast.DeleteStmt, *ast.InsertStmt:
        // "select..."

        ...

        for _, selectStmt := range util.GetSelectStmt(stmt) {

            ...

            var (
                table2colNames = map[string] /*table name*/ []*ast.ColumnName /*col names*/ {}
            )

            // get column names in where condition
            for _, col := range util.GetColumnNameInExpr(selectStmt.Where) {
                table2colNames[getTableName(col)] = append(table2colNames[getTableName(col)], col.Name)
            }

            for table, colNames := range table2colNames {
                ...
                for _, colName := range colNames {

                    for _, index := range indexesInfo {
                        // check if the column is index
                        if colName.Name.String() == index.ColumnName {
                            indexColumns = append(indexColumns, colName.Name.String())
                        }
                    }
                }

                ...
            }
        }

    }

    return nil
}
```

------

MySQL中, 计算集合(数据库中索引使用到的列) 中每个字段的区分度，如果区分度低于规则变量选择性值，则报告规则违规

```go
func RuleSQLE00039(input *rulepkg.RuleHandlerInput) error {
    // ...

    threshold := param.Float64()
    // ...

    // ...

        discrimination, err := util.CalculateIndexDiscrimination(input.Ctx, tableName, indexColumns)
        if err != nil {
            // ...
            return nil
        }

        // the table has index, check the discrimination
        for col, d := range discrimination {
            if d < threshold {
                rulepkg.AddResult(input.Res, input.Rule, SQLE00039, col, threshold)
            }
        }
    // ...
    return nil
}
```

------

MySQL中, 区分度的计算需要从在线数据库中获得

func RuleSQLE00039(input *rulepkg.RuleHandlerInput) error {
    // ...

        discrimination, err := util.CalculateIndexDiscrimination(input.Ctx, tableName, indexColumns)
        if err != nil {
            log.NewEntry().Errorf("get index discrimination failed, sqle: %v, error: %v", input.Node.Text(), err)
            return nil
        }

    // ...
}

------

PostgreSQL中, 对于“CREATE INDEX...”语句，...

```
func RuleSQLE00043(ctx context.Context, rule *driverV2.Rule, sql string, nextSQL []string) (string, error) {
    node, err := sqlParserFuncV2(sql)
    if err != nil {
        errors.Wrap(err, "parse sql")
    }

    ...

    switch stmt := node.GetStmt().GetNode().(type) {
    case *parser.Node_IndexStmt:
        // "create index ..."
        ...
    }
    return "", nil
}
```

------

PostgreSQL中, 对于 "ALTER TABLE ... ADD INDEX ..." 语句，...

```
func rule15(ctx context.Context, rule *driverV2.Rule, astSQL interface{}, nextSQL []string) (string, error) {
    ..., ok := astSQL.(*parser.RawStmt)
    if !ok {
        ...
    }

    ...
    switch stmt := ...; ... {
    case ...: //...
        ...
    case ...: //...
        ...
    case ...: //...
        ...
    case ...: //...
        ...
    case ...: //...
        ...
    case ...: //...
        for _, cmd := range stmt.AlterTableStmt.GetCmds() {
            alterTableCmd := cmd.GetAlterTableCmd()
            if alterTableCmd.GetSubtype() == parser.AlterTableType_AT_AddColumn {
                columnDef := alterTableCmd.GetDef().GetColumnDef()
                if columnDef.GetColname() == "index" {
                    //ALTER TABLE tablename ADD INDEX [indexname] ...;
                    ...
                } else {
                    ...
                }
            } ...
        }
    case ...: //...
        ...
    }
    ...
    return "", nil
}
```

------

PostgreSQL中, 如果表选项上不存在主键，则报告违规情况

```
func RuleSQLE00008(ctx context.Context, rule *driverV2.Rule, astSQL interface{}, nextSQL []string) (string, error) {
    ...
    case *parser.Node_CreateStmt:

        ...

        found := false
        // check primary key in column definition
        for _, elt := range stmt.CreateStmt.TableElts {
            if utilIsColumnHasConstraint(elt, parser.ConstrType_CONSTR_PRIMARY) {
                found = true
                break
            }
        }

        // check primary key in table constraint
        if utilIsTableHasConstraint(stmt.CreateStmt.TableElts, parser.ConstrType_CONSTR_PRIMARY) {
            found = true
        }

        if !found {
            return RuleHandlerMap[rule.Name].Message, nil
        }
    ...
}
```

------

PostgreSQL中, 如果列上不存在主键，则报告违规情况

```
func RuleSQLE00008(ctx context.Context, rule *driverV2.Rule, astSQL interface{}, nextSQL []string) (string, error) {
    ...
    case *parser.Node_CreateStmt:

        ...

        found := false
        // check primary key in column definition
        for _, elt := range stmt.CreateStmt.TableElts {
            if utilIsColumnHasConstraint(elt, parser.ConstrType_CONSTR_PRIMARY) {
                found = true
                break
            }
        }

        ...

        if !found {
            return RuleHandlerMap[rule.Name].Message, nil
        }
    ...
    return "", nil
}
```

------

PostgreSQL中, 对于所有 DML 语句中的 SELECT 子句，...

```
func RuleSQLE00098(ctx context.Context, rule *driverV2.Rule, astSQL interface{}, nextSQL []string) (string, error) {
    ..., ok := astSQL.(*parser.RawStmt)
    ...

    ...
    case *parser.Node_SelectStmt, ..., *parser.Node_InsertStmt, *parser.Node_UpdateStmt, *parser.Node_DeleteStmt:
        ...
        for _, selectStmt := range utilGetSelectStmt(node.GetStmt()) {
            ...
        }
        ...
    }
    ...
}
```

</代码片段>
```` 

结果: 

```
package rules

import (
	"context"
	"fmt"

	"github.com/sqlparser/parser"
	"github.com/XiaoMiTech/sql-performance-advisor/pkg/util"
	"github.com/XiaoMiTech/sql-performance-advisor/pkg/rule/driverV2"
)

func RuleSQLE00039(ctx context.Context, rule *driverV2.Rule, astSQL interface{}, nextSQL []string) (string, error) {
	param := rule.Params.GetParam(driverV2.DefaultSingleParamKeyName)
	if param == nil {
		return "", fmt.Errorf("param %s not found", driverV2.DefaultSingleParamKeyName)
	}
	threshold := param.Float64()
	if threshold > 1 || threshold <= 0 {
		return "", fmt.Errorf("param %s should be in range (0, 1]", driverV2.DefaultSingleParamKeyName)
	}

	node, ok := astSQL.(*parser.RawStmt)
	if !ok {
		return "", fmt.Errorf("astSQL is not *parser.RawStmt")
	}

	switch stmt := node.GetStmt().GetNode().(type) {
	case *parser.Node_IndexStmt:
		// "CREATE INDEX ..."
		tableName := stmt.IndexStmt.Relation.Relname.String()
		indexColumns := make([]string, 0)
		for _, col := range stmt.IndexStmt.IndexParams {
			indexColumns = append(indexColumns, col.Columnname.String())
		}

		if len(indexColumns) == 0 {
			// the table has no index
			return "", nil
		}

		discrimination, err := util.CalculateIndexDiscrimination(ctx, tableName, indexColumns)
		if err != nil {
			// ...
			return "", nil
		}

		// the table has index, check the discrimination
		for col, d := range discrimination {
			if d < threshold {
				return rule.Message, nil
			}
		}

	case *parser.Node_AlterTableStmt:
		// "ALTER TABLE ... ADD INDEX ..."
		for _, cmd := range stmt.AlterTableStmt.GetCmds() {
			alterTableCmd := cmd.GetAlterTableCmd()
			if alterTableCmd.GetSubtype() == parser.AlterTableType_AT_AddColumn {
				columnDef := alterTableCmd.GetDef().GetColumnDef()
				if columnDef.GetColname() == "index" {
					//ALTER TABLE tablename ADD INDEX [indexname] ...;
					tableName := stmt.AlterTableStmt.Relation.Relname.String()
					indexColumns := make([]string, 0)
					for _, col := range columnDef.Constraints {
						indexColumns = append(indexColumns, col.GetConstraint().GetRawExpr().String())
					}

					if len(indexColumns) == 0 {
						// the table has no index
						return "", nil
					}

					discrimination, err := util.CalculateIndexDiscrimination(ctx, tableName, indexColumns)
					if err != nil {
						// ...
						return "", nil
					}

					// the table has index, check the discrimination
					for col, d := range discrimination {
						if d < threshold {
							return rule.Message, nil
						}
					}
				} else {
					// ...
				}
			} else {
				// ...
			}
		}
	case *parser.Node_SelectStmt, *parser.Node_InsertStmt, *parser.Node_UpdateStmt, *parser.Node_DeleteStmt:
		// "select..."
		for _, selectStmt := range util.GetSelectStmt(node.GetStmt()) {
			// 如果 where 条件不是永真式
			if !util.IsWhereClauseAlwaysTrue(selectStmt.WhereClause) {
				// ...
				var (
					table2colNames = map[string] /*table name*/ []string /*col names*/ {}
				)

				// 将 WHERE 子句中使用的条件字段添加到集合 A
				for _, col := range util.GetColumnNameInExpr(selectStmt.WhereClause) {
					table2colNames[col.GetTable()] = append(table2colNames[col.GetTable()], col.GetColumn())
				}

				for table, colNames := range table2colNames {
					// ...
					indexesInfo, err := util.GetIndexesInfo(ctx, table)
					if err != nil {
						// ...
						return "", nil
					}

					indexColumns := make([]string, 0)
					for _, colName := range colNames {

						for _, index := range indexesInfo {
							// check if the column is index
							if colName == index.ColumnName {
								indexColumns = append(indexColumns, colName)
							}
						}
					}

					if len(indexColumns) == 0 {
						// the table has no index
						return "", nil
					}

					discrimination, err := util.CalculateIndexDiscrimination(ctx, table, indexColumns)
					if err != nil {
						// ...
						return "", nil
					}

					// the table has index, check the discrimination
					for col, d := range discrimination {
						if d < threshold {
							return rule.Message, nil
						}
					}
				}
			}

			// ...
		}

	}

	return "", nil
}
``` 

存在的问题: 

  - 有一些函数的用法错误, 比如col.Columnname在PG中不是这样使用的
  - 缺少: Select处理中的别名处理逻辑, 这部分在提示词中没有, 也就没有呈现

# 尝试 - 修正Rule39的方法调用

````
我有一段由AI生成的代码: 
```
package rules
 
import (
    "context"
    "fmt"
 
    "github.com/sqlparser/parser"
    "github.com/XiaoMiTech/sql-performance-advisor/pkg/util"
    "github.com/XiaoMiTech/sql-performance-advisor/pkg/rule/driverV2"
)
 
func RuleSQLE00039(ctx context.Context, rule *driverV2.Rule, astSQL interface{}, nextSQL []string) (string, error) {
    param := rule.Params.GetParam(driverV2.DefaultSingleParamKeyName)
    if param == nil {
        return "", fmt.Errorf("param %s not found", driverV2.DefaultSingleParamKeyName)
    }
    threshold := param.Float64()
    if threshold > 1 || threshold <= 0 {
        return "", fmt.Errorf("param %s should be in range (0, 1]", driverV2.DefaultSingleParamKeyName)
    }
 
    node, ok := astSQL.(*parser.RawStmt)
    if !ok {
        return "", fmt.Errorf("astSQL is not *parser.RawStmt")
    }
 
    switch stmt := node.GetStmt().GetNode().(type) {
    case *parser.Node_IndexStmt:
        // "CREATE INDEX ..."
        tableName := stmt.IndexStmt.Relation.Relname.String()
        indexColumns := make([]string, 0)
        for _, col := range stmt.IndexStmt.IndexParams {
            indexColumns = append(indexColumns, col.Columnname.String())
        }
 
        if len(indexColumns) == 0 {
            // the table has no index
            return "", nil
        }
 
        discrimination, err := util.CalculateIndexDiscrimination(ctx, tableName, indexColumns)
        if err != nil {
            // ...
            return "", nil
        }
 
        // the table has index, check the discrimination
        for col, d := range discrimination {
            if d < threshold {
                return rule.Message, nil
            }
        }
    }
 
    return "", nil
}
```

其中可能有一些函数调用或者一些对象的获取方法是错误的. 

我找到了一些人类写的正确的代码 (这个代码实现的功能可能跟AI的代码的功能不同, 你需要学习其中的函数调用和对象的获取方法): 

```
func RuleSQLE00043(ctx context.Context, rule *driverV2.Rule, sql string, nextSQL []string) (string, error) {
	node, err := sqlParserFuncV2(sql)
	if err != nil {
		return "", errors.Wrap(err, "parse sql")
	}

	// get expected indexMaxCount
	indexMaxCount := rule.Params.GetParam(DefaultSingleParamKeyName).Int()
	if indexMaxCount <= 0 {
		return "", fmt.Errorf("invalid size of indexMaxCount, size should > 0")
	}

	indexCounter := map[string] /*col name*/ int{}

	switch stmt := node.GetStmt().GetNode().(type) {
	case *parser.Node_IndexStmt:
		// "create index ..."
		schemaName := stmt.IndexStmt.GetRelation().GetSchemaname()
		tableName := stmt.IndexStmt.GetRelation().GetRelname()

		// get exist index information in the target table
		indexInfo, err := utilGetTableIndexInfo(ctx, schemaName, tableName)
		if err != nil {
			return "", errors.Wrap(err, "get table index info")
		}
		// count the columns in each index
		for _, idx := range indexInfo {
			for _, col := range idx.ColumnList {
				indexCounter[col]++
			}
		}

		// get the new index information
		for _, elt := range stmt.IndexStmt.IndexParams {
			indexCounter[utilGetIndexColumnName(elt)]++
		}

		// check if the same field appears in different indexes more than the threshold
		for col, count := range indexCounter {
			if count > indexMaxCount {
				return fmt.Sprintf(RuleHandlerMap[rule.Name].Message, col, indexMaxCount), nil
			}
		}
	}
	return "", nil
}

```

AI代码中的一些函数和对象属性很有可能是不存在, 我希望你学习人类代码中的函数和对象属性, 用人类代码中的片段来修正AI代码
```` 

结果: 

````
我理解你的需求，你想让我学习两段代码，并用人类代码中正确的函数调用和对象属性来修正 AI 生成的代码。以下是修正后的代码以及解释：

```go
package rules

import (
	"context"
	"fmt"

	"github.com/sqlparser/parser"
	"github.com/XiaoMiTech/sql-performance-advisor/pkg/util"
	"github.com/XiaoMiTech/sql-performance-advisor/pkg/rule/driverV2"
	"errors"
)

func RuleSQLE00039(ctx context.Context, rule *driverV2.Rule, sql string, nextSQL []string) (string, error) { // 修改：添加 sql 参数
	param := rule.Params.GetParam(driverV2.DefaultSingleParamKeyName)
	if param == nil {
		return "", fmt.Errorf("param %s not found", driverV2.DefaultSingleParamKeyName)
	}
	threshold := param.Float64()
	if threshold > 1 || threshold <= 0 {
		return "", fmt.Errorf("param %s should be in range (0, 1]", driverV2.DefaultSingleParamKeyName)
	}

	// 修改：使用 sqlParserFuncV2 解析 SQL 语句
	node, err := sqlParserFuncV2(sql) // 使用人类代码中的函数
	if err != nil {
		return "", errors.Wrap(err, "parse sql")
	}

	switch stmt := node.GetStmt().GetNode().(type) {
	case *parser.Node_IndexStmt:
		// "CREATE INDEX ..."
		// 修改：使用 GetSchemaname() 和 GetRelname() 获取 schema 和 table 名
		schemaName := stmt.IndexStmt.GetRelation().GetSchemaname() // 使用人类代码中的方法
		tableName := stmt.IndexStmt.GetRelation().GetRelname()   // 使用人类代码中的方法

		indexColumns := make([]string, 0)
		for _, col := range stmt.IndexStmt.IndexParams {
			// 修改：使用 utilGetIndexColumnName() 获取列名
			indexColumns = append(indexColumns, utilGetIndexColumnName(col)) // 使用人类代码中的函数
		}

		if len(indexColumns) == 0 {
			// the table has no index
			return "", nil
		}

		// 修改：使用 schemaName 和 tableName 作为参数
		discrimination, err := util.CalculateIndexDiscrimination(ctx, schemaName, tableName, indexColumns) // 使用正确的参数
		if err != nil {
			// ...
			return "", nil
		}

		// the table has index, check the discrimination
		for col, d := range discrimination {
			if d < threshold {
				return rule.Message, nil
			}
		}
	}

	return "", nil
}

```

**修改说明:**

1. **添加 `sql` 参数:**  AI 代码缺少 `sql` 字符串作为输入，这在处理 SQL 语句时是必须的。
2. **使用 `sqlParserFuncV2` 解析 SQL:**  将 `astSQL.(*parser.RawStmt)` 替换为 `sqlParserFuncV2(sql)`，使用人类代码中提供的函数解析 SQL。
3. **使用 `GetSchemaname()` 和 `GetRelname()`:** 使用 `stmt.IndexStmt.GetRelation().GetSchemaname()` 和 `stmt.IndexStmt.GetRelation().GetRelname()` 获取 schema 和 table 名，与人类代码保持一致。
4. **使用 `utilGetIndexColumnName()` 获取列名:** 将 `col.Columnname.String()` 替换为 `utilGetIndexColumnName(col)`，使用人类代码中提供的函数获取列名。
5. **修正 `CalculateIndexDiscrimination` 参数:** 为 `CalculateIndexDiscrimination` 函数添加 `schemaName` 和 `tableName` 参数，使其与函数定义相符。

**注意：** 

*  以上代码中假设 `sqlParserFuncV2` 和 `utilGetIndexColumnName` 函数已经存在于你的项目中，并且功能与人类代码中的一致。 
*  `util.CalculateIndexDiscrimination` 函数的实现需要根据你的项目逻辑进行调整，确保其能够接受正确的参数并返回预期的结果。 

通过学习人类代码，我能够识别 AI 代码中可能存在的错误，并进行相应的修正。这体现了结合人类知识和 AI 代码生成能力的优势，可以更高效地开发和维护代码。 
```` 

# 用 haystack 进行自动化

[code_preprocess.html](/assets/01KJBZ9G97AWEPERJRVQZTYGW3/code_preprocess.html)

规则描述: 

```
In PostgreSQL, check if the SQL violate the rule(SQLE00039): "For index, It is suggested that the discrimination of the index field is greater than the percentage threshold. the threshold should be a parameter whose default value is 0.7.".
follow the following logic:
1. For the "CREATE INDEX ..." statements, check if discrimination of the index less than the threshold. If it does, report a violation.
The calculation of index discrimination(eg: select 1- (
 select count(*) as record_count from (select id from t1 limit 50000) group by id order by record_count desc limit 1
 )/ (select count(*) from t1 limit 50000);) needs to be obtained from the online database.
2. For SELECT clauses in all DML statements, if the where condition is not always true,
 1. Define set A
 2. Add the condition fields used in the WHERE clause to the set A
 3. Connect to the database and put the indexed fields in set B based on the fields in set A
 4. Calculate the discrimination of each field in set B, if the discrimination is less than the rule variable selectivity value, then report the rule violation.
 The calculation of field discrimination(eg: select 1- (
 select count(*) as record_count from (select column_name from table_name limit 50000) group by column_name order by record_count desc limit 1
 )/ (select count(*) from table_name limit 50000);) needs to be obtained from the online database.
``` 

生成的代码: 

```
func RuleSQLExxxxx(ctx context.Context, rule *driverV2.Rule, sql string, nextSQL []string) (string, error) {
	node, err := sqlParserFuncV2(sql)
	if err != nil {
		return "", errors.Wrap(err, "parse sql")
	}
	threshold := utilGetRuleParamFloat(rule, DefaultSingleParamKeyName, DefaultSQLE00039Threshold)

	switch stmt := node.GetStmt().GetNode().(type) {
	case *parser.Node_IndexStmt:
		// "CREATE INDEX ..."
		tableName := stmt.IndexStmt.Relation.Relname
		schemaName := stmt.IndexStmt.Relation.Schemaname
		indexColumns := make([]string, 0)
		for _, col := range stmt.IndexStmt.IndexParams {
			indexColumns = append(indexColumns, utilGetIndexColumnName(col))
		}

		if len(indexColumns) == 0 {
			return "", nil
		}
		discrimination, err := utilCalculateIndexDiscrimination(ctx, schemaName, tableName, indexColumns)
		if err != nil {
			return "", errors.Wrap(err, "get index discrimination")
		}
		// check the discrimination
		for col, d := range discrimination {
			if d < threshold {
				return fmt.Sprintf(RuleHandlerMap[rule.Name].Message, col, threshold), nil
			}
		}
	case *parser.Node_SelectStmt, *parser.Node_InsertStmt, *parser.Node_UpdateStmt, *parser.Node_DeleteStmt:
		// "SELECT" in DML
		for _, selectStmt := range utilGetSelectStmt(node.GetStmt()) {
			if selectStmt.Where == nil || util.IsWhereConditionAlwaysTrue(selectStmt.Where) {
				continue
			}

			tables := make([]string, 0)
			table2Column := make(map[string][]string)

			// extract tables from select statement
			for _, from := range selectStmt.FromClause {
				switch r := from.(type) {
				case *parser.RangeVar:
					tables = append(tables, r.Relname)
					table2Column[r.Relname] = append(table2Column[r.Relname], utilGetColumnNameInExpr(selectStmt.Where)...)
				default:
					return "", fmt.Errorf("not support from clause type")
				}
			}
			if len(tables) == 0 {
				continue
			}

			// calculate discrimination for each table
			for _, table := range tables {
				indexColumns := make([]string, 0)
				// get index columns
				indexInfo, err := utilGetTableIndexInfo(ctx, "", table)
				if err != nil {
					return "", errors.Wrap(err, "get table index info")
				}
				for _, idx := range indexInfo {
					for _, col := range idx.ColumnList {
						// check if index column in where clause
						for _, column := range table2Column[table] {
							if column == col {
								indexColumns = append(indexColumns, col)
								break
							}
						}
					}
				}
				if len(indexColumns) == 0 {
					continue
				}
				discrimination, err := utilCalculateIndexDiscrimination(ctx, "", table, indexColumns)
				if err != nil {
					return "", errors.Wrap(err, "get index discrimination")
				}
				// check the discrimination
				for col, d := range discrimination {
					if d < threshold {
						return fmt.Sprintf(RuleHandlerMap[rule.Name].Message, col, threshold), nil
					}
				}
			}
		}
	}
	return "", nil
} 
``` 

refine后的代码: 

```
 func RuleSQLExxxxx(ctx context.Context, rule *driverV2.Rule, sql string, nextSQL []string) (string, error) {
	node, err := sqlParserFuncV2(sql)
	if err != nil {
		return "", errors.Wrap(err, "parse sql")
	}
	threshold := rule.Params.GetParam(DefaultSingleParamKeyName).Float()

	switch stmt := node.GetStmt().GetNode().(type) {
	case *parser.Node_IndexStmt:
		// "CREATE INDEX ..."
		tableName := stmt.IndexStmt.GetTable().GetRelname()
		schemaName := stmt.IndexStmt.GetTable().GetSchema()
		indexColumns := make([]string, 0)
		for _, col := range stmt.IndexStmt.IndexCols {
			indexColumns = append(indexColumns, col.Column.GetName())
		}

		if len(indexColumns) == 0 {
			return "", nil
		}
		discrimination, err := utilCalculateIndexDiscrimination(ctx, schemaName, tableName, indexColumns)
		if err != nil {
			return "", errors.Wrap(err, "get index discrimination")
		}
		// check the discrimination
		for col, d := range discrimination {
			if d < threshold {
				return fmt.Sprintf(RuleHandlerMap[rule.Name].Message, col, threshold), nil
			}
		}
	case *parser.Node_SelectStmt, *parser.Node_InsertStmt, *parser.Node_UpdateStmt, *parser.Node_DeleteStmt:
		// "SELECT" in DML
		for _, selectStmt := range utilGetSelectStmt(node.GetStmt()) {
			if selectStmt.Where == nil || util.IsWhereConditionAlwaysTrue(selectStmt.Where) {
				continue
			}

			tables := make([]string, 0)
			table2Column := make(map[string][]string)

			// extract tables from select statement
			for _, from := range selectStmt.From {
				switch r := from.(type) {
				case *parser.TableSource_Table:
					tables = append(tables, r.Table.GetRelname())
					table2Column[r.Table.GetRelname()] = append(table2Column[r.Table.GetRelname()], utilGetColumnNameInExpr(selectStmt.Where)...)
				default:
					return "", fmt.Errorf("not support from clause type")
				}
			}
			if len(tables) == 0 {
				continue
			}

			// calculate discrimination for each table
			for _, table := range tables {
				indexColumns := make([]string, 0)
				// get index columns
				indexInfo, err := utilGetTableIndexInfo(ctx, "", table)
				if err != nil {
					return "", errors.Wrap(err, "get table index info")
				}
				for _, idx := range indexInfo {
					for _, col := range idx.Columns {
						// check if index column in where clause
						for _, column := range table2Column[table] {
							if column == col {
								indexColumns = append(indexColumns, col)
								break
							}
						}
					}
				}
				if len(indexColumns) == 0 {
					continue
				}
				discrimination, err := utilCalculateIndexDiscrimination(ctx, "", table, indexColumns)
				if err != nil {
					return "", errors.Wrap(err, "get index discrimination")
				}
				// check the discrimination
				for col, d := range discrimination {
					if d < threshold {
						return fmt.Sprintf(RuleHandlerMap[rule.Name].Message, col, threshold), nil
					}
				}
			}
		}
	}
	return "", nil
}
``` 

Refine的diff:

![image2024-5-25 23:30:30.png](/assets/01KJBZ9G97AWEPERJRVQZTYGW3/image2024-5-25%2023%3A30%3A30.png)

# 其他可改进:

  - 分析 "我没办法" 的情况: 子规则和代码的不匹配
  - 生成一个规则时, 将 其他数据库的同ID规则 放在提示词中
  - 根据代码内容, 重新生成规则描述, 让子规则的拆解更合理
  - 有一些子规则 的描述, 在处理时拆解的有问题, 需要排查
  - 在refine中, 将代码撕成chunk, 按chunk再匹配
  - 对于缺失代码的容忍

# 通过代码修订描述

````
我有一段代码: 
```
func RuleSQLE00039(input *rulepkg.RuleHandlerInput) error {
	// get expected param value
	param := input.Rule.Params.GetParam(rulepkg.DefaultSingleParamKeyName)
	if param == nil {
		return fmt.Errorf("param %s not found", rulepkg.DefaultSingleParamKeyName)
	}
	threshold := param.Float64()
	if threshold > 1 || threshold <= 0 {
		return fmt.Errorf("param %s should be in range (0, 1]", rulepkg.DefaultSingleParamKeyName)
	}

	var tableName *ast.TableName
	indexColumns := make([]string, 0)
	switch stmt := input.Node.(type) {
	case *ast.CreateIndexStmt:
		// "create index..."

		tableName = stmt.Table
		for _, col := range stmt.IndexPartSpecifications {
			//"create index... column..."
			indexColumns = append(indexColumns, util.GetIndexColName(col))
		}

		if len(indexColumns) == 0 {
			// the table has no index
			return nil
		}

		discrimination, err := util.CalculateIndexDiscrimination(input.Ctx, tableName, indexColumns)
		if err != nil {
			log.NewEntry().Errorf("get index discrimination failed, sqle: %v, error: %v", input.Node.Text(), err)
			return nil
		}

		// the table has index, check the discrimination
		for col, d := range discrimination {
			if d < threshold {
				rulepkg.AddResult(input.Res, input.Rule, SQLE00039, col, threshold)
			}
		}
	case *ast.AlterTableStmt:
		// "alter table"
		tableName = stmt.Table

		for _, spec := range util.GetAlterTableCommandsByTypes(stmt, ast.AlterTableAddConstraint) {
			// "alter table... add index..."
			constraints := util.GetTableConstraints([]*ast.Constraint{spec.Constraint}, util.GetIndexConstraintTypes()...)

			for _, constraint := range constraints {
				for _, col := range constraint.Keys {
					indexColumns = append(indexColumns, util.GetIndexColName(col))
				}
			}

		}
		if len(indexColumns) == 0 {
			// the table has no index
			return nil
		}

		discrimination, err := util.CalculateIndexDiscrimination(input.Ctx, tableName, indexColumns)
		if err != nil {
			log.NewEntry().Errorf("get index discrimination failed, sqle: %v, error: %v", input.Node.Text(), err)
			return nil
		}

		// the table has index, check the discrimination
		for col, d := range discrimination {
			if d < threshold {
				rulepkg.AddResult(input.Res, input.Rule, SQLE00039, col, threshold)
			}
		}
	case *ast.SelectStmt, *ast.UpdateStmt, *ast.DeleteStmt, *ast.InsertStmt:
		// "select..."

		var defaultTable string
		var alias []*util.TableAliasInfo
		getTableName := func(col *ast.ColumnNameExpr) string {
			if col.Name.Table.L != "" {
				for _, a := range alias {
					if a.TableAliasName == col.Name.Table.String() {
						return a.TableName
					}
				}
				return col.Name.Table.L
			}
			return defaultTable
		}

		for _, selectStmt := range util.GetSelectStmt(stmt) {

			// get default table name
			if t := util.GetDefaultTable(selectStmt); t != nil {
				defaultTable = t.Name.L
			}

			// get table alias info
			if selectStmt.From != nil && selectStmt.From.TableRefs != nil {
				alias = util.GetTableAliasInfoFromJoin(selectStmt.From.TableRefs)
			}

			var (
				table2colNames = map[string] /*table name*/ []*ast.ColumnName /*col names*/ {}
			)

			// get column names in where condition
			for _, col := range util.GetColumnNameInExpr(selectStmt.Where) {
				table2colNames[getTableName(col)] = append(table2colNames[getTableName(col)], col.Name)
			}

			for table, colNames := range table2colNames {
				// get index of the table
				indexesInfo, err := util.GetTableIndexes(input.Ctx, table, colNames[0].Schema.L)
				if err != nil {
					log.NewEntry().Errorf("get table indexes failed, sqle: %v, error: %v", input.Node.Text(), err)
					return nil
				}

				// get index columns
				indexColumns := make([]string, 0)
				for _, colName := range colNames {

					for _, index := range indexesInfo {
						// check if the column is index
						if colName.Name.String() == index.ColumnName {
							indexColumns = append(indexColumns, colName.Name.String())
						}
					}
				}

				if len(indexColumns) == 0 {
					// no index
					return nil
				}

				tableName = &ast.TableName{
					Schema: colNames[0].Schema,
					Name:   model.NewCIStr(table),
				}
				discrimination, err := util.CalculateIndexDiscrimination(input.Ctx, tableName, indexColumns)
				if err != nil {
					log.NewEntry().Errorf("get index discrimination failed, sqle: %v, error: %v", input.Node.Text(), err)
					return nil
				}

				// has index, check the discrimination
				for col, d := range discrimination {
					if d < threshold {
						rulepkg.AddResult(input.Res, input.Rule, SQLE00039, col, threshold)
					}
				}

			}
		}

	}

	return nil
}
```

这段代码是按照以下描述实现功能的: 
---
在 MySQL 中，您需要检查 SQL 语句是否违反以下规则 (SQLE00039)： “对于索引，建议索引字段的区分度大于阈值百分比。该阈值应该是一个参数，其默认值为 0.7”。

您应该遵循以下逻辑：

1. 对于 "CREATE INDEX ..." 语句，检查表选项或列上的索引区分度是否小于阈值。如果是，则报告违规。
   索引区分度的计算（例如：`select 1- (select count(*) as record_count from (select id from t1 limit 50000) group by id order by record_count desc limit 1) / (select count(*) from t1);`）需要从在线数据库中获取。

2. 对于 "ALTER TABLE ... ADD INDEX ..." 语句，执行与上述相同的检查。

3. 对于所有 DML 语句中的 SELECT 子句，如果 WHERE 条件不总是为真，则：
   1. 定义集合 A
   2. 将 WHERE 子句中使用的条件字段添加到集合 A 中
   3. 连接数据库，并根据集合 A 中的字段将索引字段放入集合 B 中
   4. 计算集合 B 中每个字段的区分度，如果区分度小于规则变量的选择性值，则报告规则违规。
      字段区分度的计算（例如：`select 1- (select count(*) as record_count from (select id from t1 limit 50000) group by id order by record_count desc limit 1) / (select count(*) from t1);`）需要从在线数据库中获取。
---

但我认为这个代码描述和代码之间有偏差, 需要你帮我修订代码描述
在修订时, 我希望你能保持原有的描述的风格, 描述中的重大偏差和逻辑缺失进行修正 
```` 

结果: 

```
你说的没错，代码描述和实际代码实现之间确实存在偏差。主要体现在以下几个方面：

1. **代码描述中对于 DML 语句的处理逻辑过于简略，**  只是简单提到了 “如果 WHERE 条件不总是为真”， 并没有详细说明如何判断 WHERE 条件的真假，以及如何处理不同的 DML 语句（SELECT，UPDATE，DELETE，INSERT）。
2. **代码实现中对于 DML 语句的处理逻辑更加复杂，**  针对每一种 DML 语句，代码都进行了不同的处理，例如：获取默认表名，获取表别名信息，获取 WHERE 条件中的列名等等。
3. **代码描述中没有提到如何处理表别名，** 而代码实现中则针对表别名进行了特殊处理。

以下是根据代码实现修订后的描述，保持了原有描述的风格，并对重大偏差和逻辑缺失进行了修正：

---

在 MySQL 中，您需要检查 SQL 语句是否违反以下规则 (SQLE00039)： “对于索引，建议索引字段的区分度大于阈值百分比。该阈值应该是一个参数，其默认值为 0.7”。

您应该遵循以下逻辑：

1. **对于 "CREATE INDEX ..." 语句，** 检查索引所对应的表的列上的索引区分度是否小于阈值。如果是，则报告违规。
   索引区分度的计算（例如：`select 1- (select count(*) as record_count from (select id from t1 limit 50000) group by id order by record_count desc limit 1) / (select count(*) from t1);`）需要从在线数据库中获取。

2. **对于 "ALTER TABLE ... ADD INDEX ..." 语句，** 执行与上述相同的检查。

3. **对于所有 DML 语句 (SELECT, UPDATE, DELETE, INSERT) ，**  
   1. **分析语句中的 WHERE 条件，** 获取 WHERE 条件中使用的所有列名。
   2. **对于每个包含 WHERE 条件中出现的列的表，** 获取该表上所有索引的信息，并提取出包含在 WHERE 条件中的索引列。
   3. **计算每个索引列的区分度，** 如果区分度小于规则变量的选择性值，则报告规则违规。
      字段区分度的计算（例如：`select 1- (select count(*) as record_count from (select id from t1 limit 50000) group by id order by record_count desc limit 1) / (select count(*) from t1);`）需要从在线数据库中获取。
   4. **在处理过程中，需要特别注意表别名的情况，**  确保正确获取到列所属的表名。

---

希望以上修订能够更好地描述代码的实际功能。 
```
