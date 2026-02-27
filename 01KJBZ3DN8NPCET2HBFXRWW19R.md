---
title: 20231007 - 测试codellama微调后的推广性
confluence_page_id: 2589220
created_at: 2023-10-07T02:17:44+00:00
updated_at: 2023-10-17T05:18:43+00:00
---

# 测试规则推广性-1

已知PG规则 "Table creation DDL must include a creation time field with a default value of CURRENT_TIMESTAMP", 推广到MySQL, 效果如下: 

其中 isTableHasColumn 和 isColumnHasDefaultValue 是未知的辅助函数. 主体逻辑两个if间的 column关系缺失

```
[CODE]
func ruleMySQLDDLCheckCreateTimeColumn(input *RuleHandlerInput) error {
	switch stmt := input.Node.(type) {
	case *ast.CreateTableStmt:
        if isTableHasColumn(stmt, input.Params[DefaultSingleParamKeyName].String()) {
            // the target table has create_time column
            if !isColumnHasDefaultValue(stmt, input.Params[DefaultSingleParamKeyName].String(), "CURRENT_TIMESTAMP") {
                // create_time column has no default value
                addResult(input.Res, input.Rule, MySQLDDLCheckCreateTimeColumn)
            }
		}
	}
	return nil
}

func isTableHasColumn(tableDefNode *ast.CreateTableStmt, columnName string) bool {
    for _, col := range tableDefNode.Cols {
        if getColumnName(col) == columnName {
            return true
        }
    }
    return false
}

func getColumnName(columnDefNode *ast.ColumnDef) string {
    return columnDefNode.Name.Name.L
}

func isColumnHasDefaultValue(tableDefNode *ast.CreateTableStmt, columnName, expectedDefaultValue string) bool {
    for _, option := range tableDefNode.Options {
        if option.Tp == ast.TableOptionDefaultValue && 
            getColumnName(option.Expr.(*ast.ColumnOption).Expr.(*ast.ColumnOption).Expr) == columnName &&
            getSQLString(option.Expr.(*ast.ColumnOption).Expr.(*ast.ColumnOption).Expr) == expectedDefaultValue {
            return true
        }
    }
    return false
}

func getSQLString(exprNode ast.Node) string {
    switch expr := exprNode.(type) {
    case *ast.ColumnName:
        return expr.Name.L
    case *ast.ValueExpr:
        return expr.StringDef.String()
    }
    return ""
}
[/CODE]
[/INST] 
[TEST]
func TestRuleMySQLDDLCheckCreateTimeColumn(t *testing.T) {
    ruleName := rulepkg.MySQLDDLCheckCreateTimeColumn
    rule := rulepkg.RuleHandlerMap[ruleName].Rule
    //create table, has create_time column
    runSingleRuleInspectCase(rule, t, "create table, has create_time column", DefaultMysqlInspect(),`
    CREATE TABLE  if not exists exist_db.not_exist_tb_1 (
    id bigint unsigned NOT NULL AUTO_INCREMENT COMMENT "unit test",
    CREATE_TIME datetime DEFAULT NULL COMMENT "unit test",
    PRIMARY KEY (id)
    );
    `, newTestResult())

    //create table, has no create_time column
    runSingleRuleInspectCase(rule, t, "create table, has no create_time column", DefaultMysqlInspect(),`
    CREATE TABLE  if not exists exist_db.not_exist_tb_1 (
    id bigint unsigned NOT NULL AUTO_INCREMENT COMMENT "unit test",
    PRIMARY KEY (id)
    );
    `, newTestResult().addResult(ruleName))

    //alter table add column, has create_time column
    runSingleRuleInspectCase(rule, t, "alter table add column, has create_time column", DefaultMysqlInspect(),`
    ALTER TABLE exist_db.exist_tb_1 ADD COLUMN CREATE_TIME datetime DEFAULT NULL COMMENT "unit test";
    `, newTestResult())

    //alter table add column, has no create_time column
    runSingleRuleInspectCase(rule, t, "alter table add column, has no create_time column", DefaultMysqlInspect(),`
    ALTER TABLE exist_db.exist_tb_1 ADD COLUMN NO_CREATE_TIME int DEFAULT NULL COMMENT "unit test";
    `, newTestResult().addResult(ruleName))
}
[/TEST]
``` 

# 测试规则推广性-2

已有训练数据: 

```
In MySQL, for DDL statement: Table creation DDL must include an update time field with a default value of CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
In MySQL, for DDL statement: Table creation DDL must include a creation time field with a default value of CURRENT_TIMESTAMP
In MySQL, for DDL statement: When the length of CHAR is greater than 20, VARCHAR type must be used
In MySQL, for DDL statement: TIMESTAMP columns must have a default value
In PostgreSQL(PG), for DDL statement: When the length of CHAR is greater than 20, VARCHAR type must be used
In PostgreSQL(PG), for DDL statement: TIMESTAMP columns must have a default value
In PostgreSQL(PG), for DDL statement: Table creation DDL must include a creation time field with a default value of CURRENT_TIMESTAMP
In PostgreSQL(PG), for DDL statement: The field constraint must have a default value when it is set to NOT NULL
In PostgreSQL(PG), for DDL statement: column whose type is varchar must have a default value
``` 

生成训练数据中没有的规则: 

```
[RULE]"In MySQL, for DDL statement: It is not recommended to set BLOB and TEXT fields as NOT NULL"[/RULE]
[META]
{
    Rule: driverV2.Rule{
        Name:       MySQLDDLCheckColumnBlobWithNotNull,
        Desc:       "对于MySQL的DDL, BLOB 和 TEXT 类型的字段不建议设置为 NOT NULL",
        Annotation: "BLOB 和 TEXT 类型的字段无法指定默认值，如插入数据不指定字段默认为NULL，如果添加了 NOT NULL 限制，写入数据时又未对该字段指定值会导致写入失败",
        Level:      driverV2.RuleLevelError,
        Category:   RuleTypeDDLConvention,
    },
    Message:      "对于MySQL的DDL, BLOB 和 TEXT 类型的字段不建议设置为 NOT NULL",
    AllowOffline: true,
    Func:         ruleMySQLDDLCheckColumnBlobWithNotNull,
}
[/META]
``` 

效果: 

测试用例错误, 报错会多出一个参数

```
[CODE]
func ruleMySQLDDLCheckColumnBlobWithNotNull(input *RuleHandlerInput) error {
    switch stmt := input.Node.(type) {
    case *ast.CreateTableStmt:
        for _, col := range stmt.Cols {
            if isColumnTypeIs(col, mysql.TypeBlob) || isColumnTypeIs(col, mysql.TypeText) {
                //the column type is blob or text
                if isColumnHasConstraint(col, ast.ColumnOptionNotNull) {
                    //the column has "NOT NULL" constraint
                    addResult(input.Res, input.Rule, MySQLDDLCheckColumnBlobWithNotNull, col.Name)
                    return nil
                }
            }
        }
    case *ast.AlterTableStmt:
        for _, spec := range getAlterTableCommandsByTypes(stmt, ast.AlterTableAddColumns, ast.AlterTableChangeColumn, ast.AlterTableModifyColumn) {
            for _, col := range spec.NewColumns {
                if isColumnTypeIs(col, mysql.TypeBlob) || isColumnTypeIs(col, mysql.TypeText) {
                    //the column type is blob or text
                    if isColumnHasConstraint(col, ast.ColumnOptionNotNull) {
                        //the column has "NOT NULL" constraint
                        addResult(input.Res, input.Rule, MySQLDDLCheckColumnBlobWithNotNull, col.Name)
                        return nil
                    }
                }
            }
        }
    }
    return nil
}

func isColumnHasConstraint(columnDef *ast.ColumnDef, targetConstraint ast.ColumnOptionType) bool {
    for _, option := range columnDef.Options {
        if option.Tp == targetConstraint {
            return true
        }
    }
    return false
}

func isColumnTypeIs(columnDef *ast.ColumnDef, targetType byte) bool {
    return columnDef.Tp.Tp == targetType
}

func getAlterTableCommandsByTypes(alterTableStmt *ast.AlterTableStmt, ts ...ast.AlterTableType) []*ast.AlterTableSpec {
    s := []*ast.AlterTableSpec{}
    
    if alterTableStmt == nil || alterTableStmt.Specs == nil {
        return nil
    }
    
	for _, spec := range alterTableStmt.Specs {
		for _, tp := range ts {
			if spec.Tp == tp {
				s = append(s, spec)
			}
		}
	}
	return s
}
[/CODE]
[TEST]
func TestRuleMySQLDDLCheckColumnBlobWithNotNull(t *testing.T) {
    ruleName := rulepkg.MySQLDDLCheckColumnBlobWithNotNull
    rule := rulepkg.RuleHandlerMap[ruleName].Rule
    //create table, no blob/text column, no problem
    runSingleRuleInspectCase(rule, t, "create table, no blob/text column, no problem", DefaultMysqlInspect(),`
    CREATE TABLE  if not exists exist_db.not_exist_tb_1 (
    id bigint unsigned NOT NULL AUTO_INCREMENT COMMENT "unit test",
    PRIMARY KEY (id)
    );
    `, newTestResult())
    
    //create table, with blob/text column, no problem
    runSingleRuleInspectCase(rule, t, "create table, with blob/text column, no problem", DefaultMysqlInspect(),`
    CREATE TABLE  if not exists exist_db.not_exist_tb_1 (
    id bigint unsigned NOT NULL AUTO_INCREMENT COMMENT "unit test",
    CONSTRAINT c1 COMPONENT "field1" NOT NULL
    );
    `, newTestResult())
    
    //create table, with blob/text column and NOT NULL constraint
    runSingleRuleInspectCase(rule, t, "create table, with blob/text column and NOT NULL constraint", DefaultMysqlInspect(),`
    CREATE TABLE  if not exists exist_db.not_exist_tb_1 (
    id bigint unsigned NOT NULL AUTO_INCREMENT COMMENT "unit test",
    CONSTRAINT c1 COMPONENT "field1" NOT NULL
    );
    `, newTestResult().addResult(ruleName))
    
    //alter table add column, no problem
    runSingleRuleInspectCase(rule, t, "alter table add column, no problem", DefaultMysqlInspect(),`
    ALTER TABLE exist_db.exist_tb_1 ADD COLUMN v1 varchar(100);
    `, newTestResult())
    
    //alter table add column, with violate constraint
    runSingleRuleInspectCase(rule, t, "alter table add column, with violate constraint", DefaultMysqlInspect(),`
    ALTER TABLE exist_db.exist_tb_1 ADD COLUMN v1 varchar(100);
    `, newTestResult().addResult(ruleName))
}
[/TEST]
``` 

" BLOB 和 TEXT 类型的字段不建议设置为 NOT NULL" 在PostgreSQL中不再合理. 根据AI解答: 

```
在MySQL中, 有一条SQL审核规则"BLOB 和 TEXT 类型的字段不建议设置为 NOT NULL", 作用是: BLOB 和 TEXT 类型的字段无法指定默认值，如插入数据不指定字段默认为NULL，如果添加了 NOT NULL 限制，写入数据时又未对该字段指定值会导致写入失败.
在PostgreSQL中, 该规则是否仍然有效
 
---
 
在PostgreSQL中,该规则已经不再完全适用了。
PostgreSQL支持为BLOB和TEXT类型的字段指定默认值,也支持这些类型的字段设置为NOT NULL。

CREATE TABLE t (
  id SERIAL PRIMARY KEY,
  text_field TEXT NOT NULL DEFAULT 'default value',
  bytea_field BYTEA NOT NULL DEFAULT '\xDEADBEEF'::bytea 
);
 
上面创建了一个表,text_field为TEXT类型,设置为NOT NULL并指定了默认值。bytea_field为BYTEA(二进制)类型,也设置为NOT NULL并指定了默认值。
在PostgreSQL中写入数据时,如果不指定这两个字段的值,会使用默认值而不是报错。
所以在PostgreSQL中,允许BLOB和TEXT类型的字段设置为NOT NULL,只要它们在创建表时指定了默认值即可。
该审核规则在MySQL中是合理的,但在PostgreSQL中已经可以规避其潜在问题了。

``` 

# 测试规则推广性-3

当前训练数据: 

```
In MySQL, for DDL statement: It is not recommended to set BLOB and TEXT fields as NOT NULL
In MySQL, for DDL statement: Table creation DDL must include an update time field with a default value of CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
In MySQL, for DDL statement: Table creation DDL must include a creation time field with a default value of CURRENT_TIMESTAMP
In MySQL, for DDL statement: When the length of CHAR is greater than 20, VARCHAR type must be used
In MySQL, for DDL statement: TIMESTAMP columns must have a default value
In PostgreSQL(PG), for DDL statement: When the length of CHAR is greater than 20, VARCHAR type must be used
In PostgreSQL(PG), for DDL statement: TIMESTAMP columns must have a default value
In PostgreSQL(PG), for DDL statement: Table creation DDL must include a creation time field with a default value of CURRENT_TIMESTAMP
In PostgreSQL(PG), for DDL statement: The field constraint must have a default value when it is set to NOT NULL
In PostgreSQL(PG), for DDL statement: column whose type is varchar must have a default value
``` 

生成MySQL规则: 

```
[RULE]"In MySQL, for DDL statement: The default value for BLOB and TEXT fields, if any, should be NULL"[/RULE]
[META]
{
    Rule: driverV2.Rule{
        Name:       MySQLDDLCheckColumnBlobDefaultValueIfAnyShouldBeNull,
        Desc:       "对于MySQL的DDL, BLOB 和 TEXT 类型的字段如果定义了默认值, 那默认值应为NULL",
        Annotation: "在SQL_MODE严格模式下BLOB 和 TEXT 类型无法设置默认值，如插入数据不指定值，字段会被设置为NULL",
        Level:      driverV2.RuleLevelError,
        Category:   RuleTypeDDLConvention,
    },
    Message:      "对于MySQL的DDL, BLOB 和 TEXT 类型的字段如果定义了默认值, 那默认值应为NULL",
    AllowOffline: true,
    Func:         ruleMySQLDDLCheckColumnBlobDefaultValueIfAnyShouldBeNull,
}
[/META]
``` 

效果: 

逻辑正确, 对NULL的判断方式有误

```
[CODE]
func ruleMySQLDDLCheckColumnBlobDefaultValueIfAnyShouldBeNull(input *RuleHandlerInput) error {
	switch stmt := input.Node.(type) {
	case *ast.CreateTableStmt:
		for _, col := range stmt.Cols {
			if isColumnTypeIs(col, mysql.TypeBlob) || isColumnTypeIs(col, mysql.TypeMediumBlob) || 
            isColumnTypeIs(col, mysql.TypeTinyBlob) || isColumnTypeIs(col, mysql.TypeLongBlob) {
				//the column type is blob or text
				if isColumnHasConstraint(col, ast.ColumnOptionDefaultValue) {
					//the column has "DEFAULT" constraint
					value := getColumnConstraint(col, ast.ColumnOptionDefaultValue).Expr
    
					//the "DEFAULT" value is not NULL
    				if !isColumnHasConstraint(value, ast.ColumnOptionNull) {
						addResult(input.Res, input.Rule, MySQLDDLCheckColumnBlobDefaultValueIfAnyShouldBeNull)
						return nil
					}
				}
			}
		}
	case *ast.AlterTableStmt:
		for _, spec := range getAlterTableCommandsByTypes(stmt, ast.AlterTableAddColumns, ast.AlterTableChangeColumn, ast.AlterTableModifyColumn) {
			for _, col := range spec.NewColumns {
				if isColumnTypeIs(col, mysql.TypeBlob) || isColumnTypeIs(col, mysql.TypeMediumBlob) || 
                isColumnTypeIs(col, mysql.TypeTinyBlob) || isColumnTypeIs(col, mysql.TypeLongBlob) {
					//the column type is blob or text
					if isColumnHasConstraint(col, ast.ColumnOptionDefaultValue) {
						//the column has "DEFAULT" constraint
						value := getColumnConstraint(col, ast.ColumnOptionDefaultValue).Expr
    
						//the "DEFAULT" value is not NULL
    					if !isColumnHasConstraint(value, ast.ColumnOptionNull) {
							addResult(input.Res, input.Rule, MySQLDDLCheckColumnBlobDefaultValueIfAnyShouldBeNull)
							return nil
						}
					}
				}
			}
		}
	}
	return nil
}

func isColumnHasConstraint(columnDef *ast.ColumnDef, targetConstraint ast.ColumnOptionType) bool {
	for _, option := range columnDef.Options {
		if option.Tp == targetConstraint {
			return true
		}
	}
	return false
}

func getColumnConstraint(columnDef *ast.ColumnDef, targetConstraint ast.ColumnOptionType) *ast.ColumnOption {
	for _, option := range columnDef.Options {
		if option.Tp == targetConstraint {
			return option
		}
	}
	return nil
}

func isColumnTypeIs(columnDef *ast.ColumnDef, targetType byte) bool {
	return columnDef.Tp.Tp == targetType
}

func getAlterTableCommandsByTypes(alterTableStmt *ast.AlterTableStmt, ts ...ast.AlterTableType) []*ast.AlterTableSpec {
	s := []*ast.AlterTableSpec{}
    
	if alterTableStmt == nil || alterTableStmt.Specs == nil {
		return nil
	}
    
	for _, spec := range alterTableStmt.Specs {
		for _, tp := range ts {
			if spec.Tp == tp {
				s = append(s, spec)
			}
		}
	}
	return s
}
[/CODE]
[TEST]
func TestRuleMySQLDDLCheckColumnBlobDefaultValueIfAnyShouldBeNull(t *testing.T) {
	rule := rulepkg.RuleHandlerMap[rulepkg.MySQLDDLCheckColumnBlobDefaultValueIfAnyShouldBeNull].Rule
	//create table, no blob/text column
	runSingleRuleInspectCase(rule, t, "create table, no blob/text column", DefaultMysqlInspect(), `
    CREATE TABLE  if not exists exist_db.not_exist_tb_1 (
    id bigint unsigned NOT NULL AUTO_INCREMENT COMMENT "unit test"
    );
    `, newTestResult())
    
	//create table, with blob/text column without DEFAULT
	runSingleRuleInspectCase(rule, t, "create table, with blob/text column without DEFAULT", DefaultMysqlInspect(), `
    CREATE TABLE  if not exists exist_db.not_exist_tb_1 (
    id bigint unsigned NOT NULL AUTO_INCREMENT COMMENT "unit test",
    b blob
    );
    `, newTestResult())
    
	//create table, with blob/text column with DEFAULT, the DEFAULT value is not NULL
	runSingleRuleInspectCase(rule, t, "create table, with blob/text column with DEFAULT", DefaultMysqlInspect(), `
    CREATE TABLE  if not exists exist_db.not_exist_tb_1 (
    id bigint unsigned NOT NULL AUTO_INCREMENT COMMENT "unit test",
    b blob default 0
    );
    `, newTestResult().addResult(rulepkg.MySQLDDLCheckColumnBlobDefaultValueIfAnyShouldBeNull))
    
	//alter table add column, no blob/text column
	runSingleRuleInspectCase(rule, t, "alter table add column, no blob/text column", DefaultMysqlInspect(), `
    ALTER TABLE exist_db.exist_tb_1 ADD COLUMN b bigint unsigned NOT NULL COMMENT "unit test";
    `, newTestResult())
    
	//alter table add column, with blob/text column without DEFAULT
	runSingleRuleInspectCase(rule, t, "alter table add column, with blob/text column without DEFAULT", DefaultMysqlInspect(), `
    ALTER TABLE exist_db.exist_tb_1 ADD COLUMN b blob COMMENT "unit test";
    `, newTestResult())
    
	//alter table add column, with blob/text column with DEFAULT, the DEFAULT value is not NULL
	runSingleRuleInspectCase(rule, t, "alter table add column, with blob/text column with DEFAULT", DefaultMysqlInspect(), `
    ALTER TABLE exist_db.exist_tb_1 ADD COLUMN b blob default 0 COMMENT "unit test";
    `, newTestResult().addResult(rulepkg.MySQLDDLCheckColumnBlobDefaultValueIfAnyShouldBeNull))
    
	//alter table change column, no blob/text column
	runSingleRuleInspectCase(rule, t, "alter table change column, no blob/text column", DefaultMysqlInspect(), `
    ALTER TABLE exist_db.exist_tb_1 CHANGE COLUMN b1 b2 int unsigned NOT NULL COMMENT "unit test";
    `, newTestResult())
    
	//alter table change column, with blob/text column without DEFAULT
	runSingleRuleInspectCase(rule, t, "alter table change column, with blob/text column without DEFAULT", DefaultMysqlInspect(), `
    ALTER TABLE exist_db.exist_tb_1 CHANGE COLUMN b1 b2 blob COMMENT "unit test";
    `, newTestResult())
    
	//alter table change column, with blob/text column with DEFAULT, the DEFAULT value is not NULL
	runSingleRuleInspectCase(rule, t, "alter table change column, with blob/text column with DEFAULT", DefaultMysqlInspect(), `
    ALTER TABLE exist_db.exist_tb_1 CHANGE COLUMN b1 b2 blob default 0 COMMENT "unit test";
    `, newTestResult().addResult(rulepkg.MySQLDDLCheckColumnBlobDefaultValueIfAnyShouldBeNull))
}
[/TEST]
``` 

# 解决: 无法输出TEST端

生成以下规则的PG版本: 

```
case = {
    "Desc_en":  "In MySQL, for DDL statement: The default value for BLOB and TEXT fields, if any, should be NULL",
    "RuleHandler": """
{
	Rule: driverV2.Rule{
		Name:       MySQLDDLCheckColumnBlobDefaultValueIfAnyShouldBeNull,
		Desc:       "对于MySQL的DDL, BLOB 和 TEXT 类型的字段如果定义了默认值, 那默认值应为NULL",
		Annotation: "在SQL_MODE严格模式下BLOB 和 TEXT 类型无法设置默认值，如插入数据不指定值，字段会被设置为NULL",
		Level:      driverV2.RuleLevelError,
		Category:   RuleTypeDDLConvention,
	},
	Message:      "对于MySQL的DDL, BLOB 和 TEXT 类型的字段如果定义了默认值, 那默认值应为NULL",
	AllowOffline: true,
	Func:         ruleMySQLDDLCheckColumnBlobDefaultValueIfAnyShouldBeNull,
}
``` 

问题: 增加MySQL版本进行训练后, 模型输出完整的CODE, 但无法输出TEST段, 怀疑是训练数据超过了长度限制

使用代码诊断训练数据的token数量, 某组训练数据的长度超过了2048:

```
from transformers import AutoTokenizer
tokenizer = AutoTokenizer.from_pretrained("codellama/CodeLlama-7b-hf") 
max_tokens = 0
for prompt in prompts:
    tokens = tokenizer.tokenize(prompt["prompt"])
    print(len(tokens))
    if len(tokens) > max_tokens: 
        max_tokens = len(tokens)

print(max_tokens)
``` 

解决: 我们自己定制的csv_dataset.py中的chunk size, 原来定义是1000 token, 这样切分, 刚好会把CODE和TEST段分在两个chunk中, 上下文损失.

将chunk size调整为2000, 可正常产出. 但如果调到3000, GPU内存会不够, 可能需要调小batch size. 或者将CODE和TEST端分在两个训练数据中.

新问题: TEST代码, 不是PG类型, 而是MySQL类型

# 增加DB2支持

将PG规则完善后, 增加DB2相关的支持

目前的规则状态如下: 

```
In MySQL, for DDL statement: Table fields must have a NOT NULL constraint
In MySQL, for DDL statement: Renaming or changing table and column names is prohibited
In MySQL, for DDL statement: It is recommended to set the initial AUTO_INCREMENT value of a table to 0
In MySQL, for DDL statement: It is not recommended for the number of columns in a primary key to exceed the threshold, when creating tables
In MySQL, for DDL statement: It is not recommended to set the length of VARCHAR greater than the threshold
In MySQL, for DDL statement: Do not specify charset or collation to column-level, use table-level instead
In MySQL, for DDL statement: It is not recommended for a table to exceed the threshold number of columns when creating table
In MySQL, for DDL statement: Using full-width Chinese quotation marks in DDL statements is not recommended
In MySQL, for DDL statement: Deleting foreign keys is prohibited
In MySQL, for DDL statement: Dropping primary key is prohibited
In MySQL, for DDL statement: Deleting columns is prohibited
In MySQL, for DDL statement: It is not recommended to modify the default character set of a table
In MySQL, for DDL statement: Using TIMESTAMP-type fields is prohibited
In MySQL, for DDL statement: BLOB or TEXT types are not recommended
In MySQL, for DDL statement: SET type is not recommended
In MySQL, for DDL statement: ENUM type is not recommended
In MySQL, for DDL statement: BIGINT type is recommended instead of DECIMAL
In MySQL, for DDL statement: High-precision floating-point type is recommended for floating-point-type numbers
In MySQL, for DDL statement: Only one auto-increment field can be set when creating a table
In MySQL, for DDL statement: Every column, except auto-incrementing columns and blob/text columns, should have a default value
In MySQL, for DDL statement: The default value for BLOB and TEXT fields, if any, should be NULL
In MySQL, for DDL statement: It is not recommended to set BLOB and TEXT fields as NOT NULL
In MySQL, for DDL statement: Table creation DDL must include an update time field with a default value of CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
In MySQL, for DDL statement: Table creation DDL must include a creation time field with a default value of CURRENT_TIMESTAMP
In MySQL, for DDL statement: When the length of CHAR is greater than 20, VARCHAR type must be used
In MySQL, for DDL statement: time-type columns must have a default value
In PostgreSQL(PG), for DDL statement: Table fields must have a NOT NULL constraint
In PostgreSQL(PG), for DDL statement: Renaming or changing table and column names is prohibited
In PostgreSQL(PG), for DDL statement: It is not recommended for the number of columns in a primary key to exceed the threshold, when creating tables
In PostgreSQL(PG), for DDL statement: It is not recommended to set the length of VARCHAR greater than the threshold
In PostgreSQL(PG), for DDL statement: Do not specify charset or collation to column-level, use table-level instead
In PostgreSQL(PG), for DDL statement: It is not recommended for a table to exceed the threshold number of columns when creating table
In PostgreSQL(PG), for DDL statement: Using full-width Chinese quotation marks in DDL statements is not recommended
In PostgreSQL(PG), for DDL statement: Deleting foreign keys is prohibited
In PostgreSQL(PG), for DDL statement: Dropping primary key is prohibited
In PostgreSQL(PG), for DDL statement: Deleting columns is prohibited
In PostgreSQL(PG), for DDL statement: It is not recommended to modify the default character set of a table
In PostgreSQL(PG), for DDL statement: BLOB or TEXT types are not recommended
In PostgreSQL(PG), for DDL statement: ENUM type is not recommended
In PostgreSQL(PG), for DDL statement: High-precision floating-point type is recommended for floating-point-type numbers
In PostgreSQL(PG), for DDL statement: Only one auto-increment field can be set when creating a table
In PostgreSQL(PG), for DDL statement: Every column, except auto-incrementing columns and blob/text columns, should have a default value
In PostgreSQL(PG), for DDL statement: The default value for BLOB and TEXT fields, if any, should be NULL
In PostgreSQL(PG), for DDL statement: When the length of CHAR is greater than 20, VARCHAR type must be used
In PostgreSQL(PG), for DDL statement: TIMESTAMP columns must have a default value
In PostgreSQL(PG), for DDL statement: Table creation DDL must include a creation time field with a default value of CURRENT_TIMESTAMP
In PostgreSQL(PG), for DDL statement: The field constraint must have a default value when it is set to NOT NULL
In PostgreSQL(PG), for DDL statement: column whose type is varchar must have a default value
In DB2, for DDL statement: Deleting foreign keys is prohibited
In DB2, for DDL statement: Deleting foreign keys is prohibited
In DB2, for DDL statement: Deleting foreign keys is prohibited
In DB2, for DDL statement: Deleting foreign keys is prohibited
In DB2, for DDL statement: Deleting foreign keys is prohibited
``` 

初期发现, 要生成DB2的规则, 由于DB2只有一个规则, 则: 

  1. 需要对DB2的规则进行增殖 (增殖成5个)
  2. 发现生成的规则会近似于MySQL的规则

为了解决规则近似于MySQL规则的问题, 在提示词中, 增加关于编程语言的提示 (其中lang和test_lang的部分): 

```
<<SYS>> 
Given a SQL audit rule(wrapped by [RULE][/RULE]), and rule metadata (wrapped by [META][/META]), you should: 
1. write a rule function in {lang} to check if SQL ast matches the SQL audit rule or not, and helper functions if needed, wrapped by [CODE][/CODE]
2. write unit test in {test_lang} to test the rule function, wrapped by [TEST][/TEST]
<</SYS>> 
``` 

多次生成后 (初始数据量低), 可以获得不错的效果: 

```
[CODE]
@Data
@Slf4j
@EqualsAndHashCode(callSuper = false)
public class Rule56 extends AbstractRuleHandler {
    /**
     * 支持的解析器类型列表
     */
    public List<ParserTypeEnum> parserTypeEnumList = Arrays.asList(ParserTypeEnum.ANTLR);
    /**
     * 支持的SQL类型列表
     */
    public List<SqlTypeEnum> sqlTypeEnumList = Arrays.asList(SqlTypeEnum.DDL);
    String name = "DB2DDLDisableDropColumn";
    String desc = "对于DB2的DDL, 禁止进行删除列的操作";
    String annotation = "业务逻辑与删除列依赖未完全消除，列被删除后可能导致程序异常（无法正常读写）的情况；开启该规则，SQLE将提醒删除列为高危操作";
    String category = RuleConstant.RULE_TYPE_DDL_CONVENTION;
    String level = RuleConstant.ERROR;

    @Override
    public void maintenanceParams() {

    }

    @Override
    public Response handle(String sql, List<String> sqlList) throws Throwable {
        if (isSqlDropColumn(sql)) {
            return new Response(level, desc);
        } else {
            return new Response();
        }
    }

    private boolean isSqlDropColumn(String sql) throws Throwable {
        DB2Parser parser = null;
        try {
            parser = Db2PluginGrpcServer.pool.borrowParser(sql, 5, TimeUnit.SECONDS);
            ParseTreeWalker walker = new ParseTreeWalker();
            AstWalkerIsSqlDropColumn listener = new AstWalkerIsSqlDropColumn();
            walker.walk(listener, parser.sqlStatement());
            return listener.isSqlDropColumn();
        } catch (Throwable e) {
            throw e;
        } finally {
            if (parser != null) {
                Db2PluginGrpcServer.pool.releaseParser(parser);
            }
        }
    }

    @Data
    static class AstWalkerIsSqlDropColumn extends DB2ParserBaseListener {
        boolean flag = false;

        @Override
        public void enterAlterTableOptionList(DB2Parser.AlterTableOptionListContext ctx) {
            if (ctx.DROP() != null && ctx.COLUMN() != null) {
                flag = true;
            }
        }

        public boolean isSqlDropColumn() {
            return flag;
        }
    }
[/CODE]
[TEST]
<rule>
    <ruleName>Db2_056</ruleName>
    <cases>
        <case>
            <!-- alter table, no drop column -->
            <sql desc="规则匹配" isMatchRule="true">alter table table56_1 ADD COLUMN new_column_name integer;</sql>
        </case>
        <case>
            <!-- alter table, with drop column -->
            <sql desc="规则匹配" isMatchRule="true">alter table table56_1 drop column v1;</sql>
        </case>
    </cases>
</rule>
[/TEST]
``` 

# 为DB2扩充规则

已有的DB2规则仅有两类: 

```
In DB2, for DDL statement: Deleting columns is prohibited
In DB2, for DDL statement: Deleting columns is prohibited
In DB2, for DDL statement: Deleting columns is prohibited
In DB2, for DDL statement: Deleting columns is prohibited
In DB2, for DDL statement: Deleting columns is prohibited
In DB2, for DDL statement: Deleting foreign keys is prohibited
In DB2, for DDL statement: Deleting foreign keys is prohibited
In DB2, for DDL statement: Deleting foreign keys is prohibited
In DB2, for DDL statement: Deleting foreign keys is prohibited
In DB2, for DDL statement: Deleting foreign keys is prohibited
``` 

多次生成后, 取较好的结果: 

```
public class Rule60 extends AbstractRuleHandler {
    /**
     * 支持的解析器类型列表
     */
    public List<ParserTypeEnum> parserTypeEnumList = Arrays.asList(ParserTypeEnum.ANTLR);
    /**
     * 支持的SQL类型列表
     */
    public List<SqlTypeEnum> sqlTypeEnumList = Arrays.asList(SqlTypeEnum.DDL);
    String name = "Db2_060";
    String desc = "对于DB2的DDL, 表字段必须有NOT NULL约束";
    String annotation = "表字段必须有 NOT NULL 约束可确保数据的完整性，防止插入空值，提升查询准确性";
    String category = RuleConstant.RULE_TYPE_DDL_CONVENTION;
    String level = RuleConstant.ERROR;

    @Override
    public void maintenanceParams() {

    }

    @Override
    public Response handle(String sql, List<String> sqlList) throws Throwable {
        if (!isSqlDDL(sql)) {
            return new Response(level, desc);
        }

        DB2Parser parser = null;
        try {
            parser = Db2PluginGrpcServer.pool.borrowParser(sql, 5, TimeUnit.SECONDS);
            ParseTreeWalker walker = new ParseTreeWalker();
            AstWalkerIsSqlDDL listener = new AstWalkerIsSqlDDL();
            walker.walk(listener, parser.sqlStatement());
            if (!listener.isSqlDDL()) {
                return new Response(level, desc);
            }

            ArrayList<ParseTree> violateList = new ArrayList<>();
            for (ParseTree child : parser.sqlStatement().children) {
                if (child instanceof DB2Parser.CreateTableOptionListContext) {
                    if (!isTableOption((DB2Parser.CreateTableOptionListContext) child, DB2Parser.TableOptionTypes.TPT_NOTNULL.toString())) {
                        violateList.add(child);
                    }
                }
            }
            if (violateList.size() > 0) {
                return new Response(level, desc);
            } else {
                return new Response();
            }
        } catch (Throwable e) {
            throw e;
        } finally {
            if (parser != null) {
                Db2PluginGrpcServer.pool.releaseParser(parser);
            }
        }
    }

    private boolean isTableOption(DB2Parser.CreateTableOptionListContext ctx, String targetOption) {
        for (DB2Parser.TableOptionContext option : ctx.tableOption()) {
            if (option.STRING().getText().equalsIgnoreCase(targetOption)) {
                return true;
            }
        }
        return false;
    }

    @Data
    static class AstWalkerIsSqlDDL extends DB2ParserBaseListener {
        boolean flag = false;

        @Override
        public void enterSqlStatement(DB2Parser.SqlStatementContext ctx) {
            flag = true;
        }

        @Override
        public void exitSqlStatement(DB2Parser.SqlStatementContext ctx) {
            if (flag) {
                System.out.println("flag: " + flag);
            }
            flag = false;
        }

        public boolean isSqlDDL() {
            return flag;
        }
    }
}
``` 

增加到第三个DB2规则后, train_loss在10 epoch训练中, 会先降后升, 最终为0.2+左右. 比起之前的0.006差得比较多

调小LR到1e-3, 可恢复效果, epoch 10的效果是0.0051
