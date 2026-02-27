---
title: 20230923 - 测试用codellama, 进行SQLe规则代码的微调 - 跑通单元测试
confluence_page_id: 2589177
created_at: 2023-09-27T05:48:13+00:00
updated_at: 2023-10-07T00:26:29+00:00
---

# 经验1 - 对代码的作用描述 有影响的因素

case描述: 

```
{    
	"Desc_en":  "In MySQL, for DDL statement: varchar columns must have a default value",
    "RuleHandler": """
{
    Rule: driverV2.Rule{
        Name:       MySQLDDLCheckColumnVarcharWithoutDefault,
        Desc:       "对于MySQL的DDL, varchar 类型的列必须添加默认值",
        Annotation: "如存在NOT NULL且不带默认值的字段，INSERT时不包含该字段，会导致插入报错",
        Level:      driverV2.RuleLevelWarn,
        Category:   RuleTypeDDLConvention,
    },
    Message:      "对于MySQL的DDL, varchar 类型的列必须添加默认值. 以下字段不规范:%v",
    AllowOffline: true,
    Func:         ruleMySQLDDLCheckColumnVarcharWithoutDefault,
}
""",
}
``` 

英文描述, 中文描述, 函数名, 三者对代码的作用描述都有影响

# 经验2 - 增加同一case的多种描述, 训练更有效

对于同一组case代码, 增加 多种同义的 英文描述和中文描述, 生成多个case

训练效果会显著增加

# TODO

  - MySQL和PG的代码形式 不同, 比如单元测试的 辅助函数签名不同, rule描述的结构体不同, 等
  - 有一些PG的case, 单元测试代码过长
  - case的描述, 不够准确. 比如: 对于varchar的规则, 同时也会检查char字段

# 问题

使用codellama-13b-hf, 训练数据中有 ""对于PostgreSQL的DDL, varchar字段必须定义默认值"" 的数据: 

```
    "UnitTest": """
func TestRulePgDDLTableVarcharFieldMustHaveDefaultValue(t *testing.T) {
	testSingleSqlAudit(PgDDLTableVarcharFieldMustHaveDefaultValue, t, `
        CREATE TABLE t1(
            id int primary key,
            id2 varchar default 'unit test'
        );
    `, newTestResults())

	testSingleSqlAudit(PgDDLTableVarcharFieldMustHaveDefaultValue, t, `
        CREATE TABLE t1(
            id int primary key,
            id2 varchar
        );
    `, newTestResults().add(PgDDLTableVarcharFieldMustHaveDefaultValue))

	testSingleSqlAudit(PgDDLTableVarcharFieldMustHaveDefaultValue, t, `
        ALTER TABLE exist_tb_1
        ADD COLUMN v3 varchar default 'unit test';
    `, newTestResults())

	testSingleSqlAudit(PgDDLTableVarcharFieldMustHaveDefaultValue, t, `
        ALTER TABLE exist_tb_1
        ADD COLUMN v3 varchar;
    `, newTestResults().add(PgDDLTableVarcharFieldMustHaveDefaultValue))
}
""",
    "Code": """
func rulePgDDLTableVarcharFieldMustHaveDefaultValue(ctx context.Context, rule *driverV2.Rule, astSQL interface{}, nextSQL []string) (string, error) {
	node, ok := astSQL.(*parser.RawStmt)
	if !ok {
		return "", fmt.Errorf("invalid type of ast of rule[%v]", rule.Name)
	}

	switch stmt := node.GetStmt().GetNode().(type) {
	case *parser.Node_CreateStmt:
		// Create Table
		for _, elt := range stmt.CreateStmt.TableElts {
			switch def := elt.GetNode().(type) {
			case *parser.Node_ColumnDef:
                types := def.ColumnDef.TypeName.Names
                if (types[len(types)-1].GetString_().Str != "varchar") {
                    continue
                }
        		columnHasDefault := false
				for _, constraint := range def.ColumnDef.Constraints {
					c := constraint.GetConstraint()
					if c != nil && c.Contype == parser.ConstrType_CONSTR_DEFAULT {
						columnHasDefault = true
						break
					}
				}
				if !columnHasDefault {
					return RuleHandlerMap[rule.Name].Message, nil
				}
			}
		}
	case *parser.Node_AlterTableStmt:
		// Alter Table
		cmds := stmt.AlterTableStmt.GetCmds()
		for _, cmd := range cmds {
            alterTableCmd := cmd.GetAlterTableCmd()
			if alterTableCmd.GetSubtype() != parser.AlterTableType_AT_AddColumn {
				continue
			}
			columnDef := alterTableCmd.Def.GetColumnDef()
			if columnDef == nil {
				continue
			}
            
            types := columnDef.TypeName.Names
            if (types[len(types)-1].GetString_().Str != "varchar") {
                continue
            }
            columnHasDefault := false
            for _, constraint := range columnDef.Constraints {
                c := constraint.GetConstraint()
                if c != nil && c.Contype == parser.ConstrType_CONSTR_DEFAULT {
                    columnHasDefault = true
                    break
                }
            }
            if !columnHasDefault {
                return RuleHandlerMap[rule.Name].Message, nil
            }
		}
	}
	return "", nil
}
"""
}
``` 

但对于测试数据 "对于PostgreSQL的DDL, integer字段必须定义默认值", 效果并不好: 

```
[CODE]
func rulePgDDLTableIntegerFieldMustHaveDefaultValue(ctx context.Context, rule *driverV2.Rule, astSQL interface{}, nextSQL []string) (string, error) {
    node, ok := astSQL.(*parser.RawStmt)
    if !ok {
        return "", fmt.Errorf("invalid type of ast of rule[%v]", rule.Name)
    }

    switch stmt := node.GetStmt().GetNode().(type) {
    case *parser.Node_CreateStmt:
        hasPkColumn := false
        for _, elt := range stmt.CreateStmt.TableElts {
            switch def := elt.GetNode().(type) {
            case *parser.Node_ColumnDef:
                for _, constraint := range def.ColumnDef.Constraints {
                    c := constraint.GetConstraint()
                    if c != nil && c.Contype == parser.ConstrType_CONSTR_DEFAULT {
                        hasPkColumn = true
                        break
                    }
                }
            case *parser.Node_Constraint:
                if def.Constraint.Contype == parser.ConstrType_CONSTR_DEFAULT { 
                    hasPkColumn = true
                    break
                }
            }
        }
        if len(stmt.CreateStmt.TableElts) > 0 && !hasPkColumn {
            return RuleHandlerMap[rule.Name].Message, nil
        }
    }
    return "", nil
}
[/CODE]
[TEST]
func TestRulePgDDLTableIntegerFieldMustHaveDefaultValue(t *testing.T) {
    testSingleSqlAudit(PgDDLTableIntegerFieldMustHaveDefaultValue, t, `
        CREATE TABLE t1(
            id int primary key,
            id2 int,
            id3 int,
            id4 int,
            id5 int,
            id6 int,
            id7 int,
            id8 int,
            id9 int,
            id10 int
        );
        `, newTestResults())
    testSingleSqlAudit(PgDDLTableIntegerFieldMustHaveDefaultValue, t, `
        CREATE TABLE t1(
            id int primary key,
            id2 int,
            id3 int,
            id4 int,
            id5 int,
            id6 int,
            id7 int,
            id8 int,
            id9 int,
            id10 int,
            id11 int,
            id2 int,
            id3 int,
            id4 int,
            id5 int,
            id6 int,
            id7 int,
            id8 int,
            id9 int,
            id10 int,
            id11 int,
            id2 int,
            id3 int,
            id4 int,
            id5 int,
            id6 int,
            id7 int,
            id8 int,
            id9 int,
            id10 int
        );
    `, newTestResults().add(PgDDLTableFields, 9, newTestResults().add(PgDDLTableFields))
}
[/TEST]

``` 

代码和测试并不正确. 

尝试对于训练数据, 增加注释, 并增加逐步骤的代码片段

先简化训练数据, 将训练数据增殖 (一个case使用多种描述) 关闭, 使用codellama-7b测试效果: 

```
[CODE]
func rulePgDDLTableIntegerFieldMustHaveDefaultValue(ctx context.Context, rule *driverV2.Rule, astSQL interface{}, nextSQL []string) (string, error) {
    node, ok := astSQL.(*parser.RawStmt)
    if !ok {
        return "", fmt.Errorf("invalid type of ast of rule[%v]", rule.Name)
    }

    switch stmt := node.GetStmt().GetNode().(type) {
    case *parser.Node_CreateStmt:
        // Create Table
        for _, elt := range stmt.CreateStmt.TableElts {
            switch def := elt.GetNode().(type) {
            case *parser.Node_ColumnDef:
                // Column Definition
                types := def.ColumnDef.TypeName.Names
                if len(types) > 0 && types[0].GetString_().Str == "integer" {
                    for _, constraint := range def.ColumnDef.Constraints {
                        c := constraint.GetConstraint()
                        if c != nil && c.Contype == parser.ConstrType_CONSTR_NOTNULL {
                            continue
                        }
                        if c != nil && c.Contype == parser.ConstrType_CONSTR_DEFAULT {
                            // has default value
                            continue
                        }
                        return RuleHandlerMap[rule.Name].Message, nil
                    }
                }
            }
        }
    }
    return "", nil
}
[/CODE]
[TEST]
func TestRulePgDDLTableIntegerFieldMustHaveDefaultValue(t *testing.T) {
    testSingleSqlAudit(PgDDLTableIntegerFieldMustHaveDefaultValue, t, "CREATE TABLE t1(id integer);", newTestResults().add(PgDDLTableIntegerFieldMustHaveDefaultValue))
    testSingleSqlAudit(PgDDLTableIntegerFieldMustHaveDefaultValue, t, "CREATE TABLE t1(id integer default 0);", newTestResults())
    testSingleSqlAudit(PgDDLTableIntegerFieldMustHaveDefaultValue, t, "CREATE TABLE t1(id integer not null);", newTestResults().add(PgDDLTableIntegerFieldMustHaveDefaultValue))
    testSingleSqlAudit(PgDDLTableIntegerFieldMustHaveDefaultValue, t, "CREATE TABLE t1(id integer default 0 not null);", newTestResults())
}
[/TEST]
``` 

混入了一些NOT NULL相关的检查代码

\---

对代码和测试代码增加必要注释, 抽取辅助函数, 并增加单独的辅助函数的训练. 但效果不好: 

温度=0.6

```
func rulePgDDLTableIntegerFieldMustHaveDefaultValue(ctx context.Context, rule *driverV2.Rule, astSQL interface{}, nextSQL []string) (string, error) {
    node, ok := astSQL.(*parser.RawStmt)
    if !ok {
        return "", fmt.Errorf("invalid type of ast of rule[%v]", rule.Name)
    }

    switch stmt := node.GetStmt().GetNode().(type) {
    case *parser.Node_CreateStmt:
        // Create Table
        for _, elt := range stmt.CreateStmt.TableElts {
            if columnDef := elt.GetColumnDef(); nil != columnDef {
                if isIntegerColumnWithoutDefaultValue(columnDef) {
                    return RuleHandlerMap[rule.Name].Message, nil
                }
            }
        }
    case *parser.Node_AlterTableStmt:
        // Alter Table
        for _, cmd := range stmt.AlterTableStmt.GetCmds() {
            alterTableCmd := cmd.GetAlterTableCmd()
            if alterTableCmd.GetSubtype() != parser.AlterTableType_AT_AddColumn {
                // 仅对ALTER TABLE ADD COLUMN进行校验
                continue
            }
            columnDef := alterTableCmd.Def.GetColumnDef()
            if nil == columnDef {
                // 不是ADD COLUMN的ALTER TABLE命令
                continue
            }
            if isIntegerColumnWithoutDefaultValue(columnDef) {
                return RuleHandlerMap[rule.Name].Message, nil
            }
        }
    }
    return "", nil
}

func isIntegerColumnWithoutDefaultValue(columnDef *parser.ColumnDef) bool {
    types := columnDef.TypeName.Names
    if nil == types || len(types) != 2 {
        // 不是定义了类型的COLUMN
        return false
    }
    if !parser.TypeNameType_TYPNAME_TYPE && types[0].GetString_().Str != "pg_catalog" {
        // 不是pg_catalog中的类型
        return false
    }
    if parser.TypeNameType_TYPNAME_TYPE {
        // CREATE TABLE中的COLUMN
        if types[1].GetString_().Str == "int4" {
            // int4是PostgreSQL的INTEGER类型
            isInt := true
            for _, def := range columnDef.TypeName.Typmods {
                if !checkColDefIntegerTypeMod(def) {
                    isInt = false
                    break
                }
            }
            if isInt {
                return true
            }
        }
    } else {
        // ALTER TABLE ADD COLUMN中的COLUMN
        if types[1].GetString_().Str == "integer" {
            // integer是PostgreSQL的INTEGER类型
            return checkColDefIntegerTypeMod(columnDef.TypeName.Typmods[0])
        }
    }
    return false
}

func checkColDefIntegerTypeMod(def *parser.Node) bool {
    if !parser.NodeTagIs(def.GetNode(), parser.NodeTagInteger) {
        // 不是定义了TYPE MODIFIER的COLUMN
        return false
    }
    node := def.GetInteger()
    if node.GetIval() != 0 {
        // INTEGER类型不允许指定默认值
        return false
    }
    return true
}
``` 

问题=0.2

```
[CODE]
func rulePgDDLTableIntegerFieldMustHaveDefaultValue(ctx context.Context, rule *driverV2.Rule, astSQL interface{}, nextSQL []string) (string, error) {
    node, ok := astSQL.(*parser.RawStmt)
    if !ok {
        return "", fmt.Errorf("invalid type of ast of rule[%v]", rule.Name)
    }

    switch stmt := node.GetStmt().GetNode().(type) {
    case *parser.Node_CreateStmt:
        // Create Table
        for _, elt := range stmt.CreateStmt.TableElts {
            if columnDef := elt.GetColumnDef(); nil != columnDef {
                if isIntegerColumnWithoutDefaultValue(columnDef) {
                    return RuleHandlerMap[rule.Name].Message, nil
                }
            }
        }
    case *parser.Node_AlterTableStmt:
        // Alter Table
        for _, cmd := range stmt.AlterTableStmt.GetCmds() {
            alterTableCmd := cmd.GetAlterTableCmd()
            if alterTableCmd.GetSubtype() != parser.AlterTableType_AT_AddColumn {
                // 仅对ALTER TABLE ADD COLUMN进行检查
                continue
            }
            columnDef := alterTableCmd.Def.GetColumnDef()
            if nil == columnDef {
                // 不是COLUMN_DEF的ALTER TABLE命令，跳过
                continue
            }
            if isIntegerColumnWithoutDefaultValue(columnDef) {
                return RuleHandlerMap[rule.Name].Message, nil
            }
        }
    }
    return "", nil
}

func isIntegerColumnWithoutDefaultValue(columnDef *parser.ColumnDef) bool {
    types := columnDef.TypeName.Names
    if (types[len(types)-1].GetString_().Str != "int4") {
        return false
    }

    for _, def := range columnDef.TypeName.Typmods {
        if def.GetAConst() == nil {
            continue
        }
        if def.GetAConst().Val.GetInteger().GetIval() == int32(0) {
            return true
        }
    }
    return false
}
[/CODE]
[TEST]
func TestRulePgDDLTableIntegerFieldMustHaveDefaultValue(t *testing.T) {
    testSingleSqlAudit(PgDDLTableIntegerFieldMustHaveDefaultValue, t, `
        CREATE TABLE t1(
            id int primary key,
            id2 integer
        );
        `, newTestResults())
    testSingleSqlAudit(PgDDLTableIntegerFieldMustHaveDefaultValue, t, `
        CREATE TABLE t1(
            id int primary key,
            id2 integer default 0
        );
        `, newTestResults())
    testSingleSqlAudit(PgDDLTableIntegerFieldMustHaveDefaultValue, t, `
        CREATE TABLE t1(
            id int primary key,
            id2 integer
            );
        `, newTestResults())
}
[/TEST]
``` 

其中包含一些不在代码训练数据中的概念 (部分包含在AST训练数据中), 将AST训练数据去掉, 并将目标训练数据增殖成两倍, 再次尝试

\---

发现rule的定义错了. 应是: 

"In PostgreSQL(PG), for DDL statement: integer columns must have a default value"

但写成了

"In PostgreSQL(PG), for DDL statement: integer columns must have a default value 0"

修正rule的定义后, 效果: 

效果较好, 但测试代码中 多个int的判断有误, 存在某个int (primary key)不满足条件

```
[CODE]
func rulePgDDLTableIntegerFieldMustHaveDefaultValue(ctx context.Context, rule *driverV2.Rule, astSQL interface{}, nextSQL []string) (string, error) {
    node, ok := astSQL.(*parser.RawStmt)
    if !ok {
        return "", fmt.Errorf("invalid type of ast of rule[%v]", rule.Name)
    }

    switch stmt := node.GetStmt().GetNode().(type) {
    case *parser.Node_CreateStmt:
        // check "create table"
        
        for _, elt := range stmt.CreateStmt.TableElts {
            //if the column whose type is "integer" and has no DEFAULT
            if isColumnTypeIs(elt, "integer") && 
                !isColumnHasConstraint(elt, parser.ConstrType_CONSTR_DEFAULT) {
                return RuleHandlerMap[rule.Name].Message, nil
            }
        }
    case *parser.Node_AlterTableStmt:
        cmds := stmt.AlterTableStmt.GetCmds()
        for _, cmd := range cmds {
            alterTableCmd := cmd.GetAlterTableCmd()
            
            if alterTableCmd.GetSubtype() != parser.AlterTableType_AT_AddColumn {
                // check "alter table add column"
                columnDef := alterTableCmd.Def.GetColumnDef()
                if columnDef == nil {
                    continue
                }

                //if the column whose type is "integer" and has no DEFAULT
                if isColumnTypeIs(alterTableCmd.Def, "integer") && 
                    !isColumnHasConstraint(alterTableCmd.Def, parser.ConstrType_CONSTR_DEFAULT) {
                    return RuleHandlerMap[rule.Name].Message, nil
                }
            }
        }
    }
    return "", nil
}

//check if a PostgreSQL column is of type _targetType_ (integer/varchar/...) or not
func isColumnTypeIs(columnDefNode *parser.Node, targetType string) bool {
    columnDef := columnDefNode.GetColumnDef()
    if nil == columnDef { 
        return false
    }
    
    types := columnDef.TypeName.Names
    return types[len(types)-1].GetString_().Str == targetType
}

//check if a PostgreSQL column has constraint _targetConstraint_ (DEFAULT/PRIMARY/FOREIGN/...) or not
func isColumnHasConstraint(columnDefNode *parser.Node, targetConstraint parser.ConstrType) bool {
    columnDef := columnDefNode.GetColumnDef()
    if nil == columnDef { 
        return false
    }
    
    for _, constraint := range columnDef.Constraints {
        c := constraint.GetConstraint()
        if c != nil && c.Contype == targetConstraint {
            return true
        }
    }
    return false
}
[/CODE]
[TEST]
func TestRulePgDDLTableIntegerFieldMustHaveDefaultValue(t *testing.T) {
    //check "create table" with integer column which has default value
    testSingleSqlAudit(PgDDLTableIntegerFieldMustHaveDefaultValue, t, `
        CREATE TABLE t1(
            id int primary key,
            id2 int default 1,
            id3 int
        );
    `, newTestResults())

    //check "create table" with integer column which has no default value
    testSingleSqlAudit(PgDDLTableIntegerFieldMustHaveDefaultValue, t, `
        CREATE TABLE t1(
            id int primary key,
            id2 int,
            id3 int
        );
    `, newTestResults().add(PgDDLTableIntegerFieldMustHaveDefaultValue))

    //check "alter table add column" with integer column which has default value
    testSingleSqlAudit(PgDDLTableIntegerFieldMustHaveDefaultValue, t, `
        ALTER TABLE exist_tb_1
        ADD COLUMN v3 int default 1;
    `, newTestResults())

    //check "alter table add column" with integer column which has no default value
    testSingleSqlAudit(PgDDLTableIntegerFieldMustHaveDefaultValue, t, `
        ALTER TABLE exist_tb_1
        ADD COLUMN v3 int;
    `, newTestResults().add(PgDDLTableIntegerFieldMustHaveDefaultValue))
}
[/TEST]
``` 

当rule修改为 "In PostgreSQL(PG), for DDL statement: integer columns must have a default value, the default value must be 123", 不改变中文描述, 效果: 

```
[CODE]
func rulePgDDLTableIntegerFieldMustHaveDefaultValue(ctx context.Context, rule *driverV2.Rule, astSQL interface{}, nextSQL []string) (string, error) {
    node, ok := astSQL.(*parser.RawStmt)
    if !ok {
        return "", fmt.Errorf("invalid type of ast of rule[%v]", rule.Name)
    }

    switch stmt := node.GetStmt().GetNode().(type) {
    case *parser.Node_CreateStmt:
        // check "create table"
        
        for _, elt := range stmt.CreateStmt.TableElts {
            //if !isColumnDef(elt) {
            //    continue
            //}

            columnDef := elt.(*parser.Node_ColumnDef)
            
            if !isIntegerColumn(columnDef) {
                // integer column has default value
                if len(columnDef.ColumnDef.Constraints) == 0 {
                    return RuleHandlerMap[rule.Name].Message, nil
                }

                for _, constraint := range columnDef.ColumnDef.Constraints {
                    c := constraint.GetConstraint()
                    if c != nil && c.Contype == parser.ConstrType_CONSTR_DEFAULT {
                        // integer column has default value
                        return RuleHandlerMap[rule.Name].Message, nil
                    }
                }
            }
        }
    case *parser.Node_AlterTableStmt:
        cmds := stmt.AlterTableStmt.GetCmds()
        for _, cmd := range cmds {
            alterTableCmd := cmd.GetAlterTableCmd()
            
            if alterTableCmd.GetSubtype() != parser.AlterTableType_AT_AddColumn {
                // check "alter table add column"
                columnDef := alterTableCmd.Def.GetColumnDef()
                if columnDef == nil {
                    continue
                }

                if !isIntegerColumn(alterTableCmd.Def) {
                    // integer column has default value
                    if len(alterTableCmd.Def.Constraints) == 0 {
                        return RuleHandlerMap[rule.Name].Message, nil
                    }

                    for _, constraint := range alterTableCmd.Def.Constraints {
                        c := constraint.GetConstraint()
                        if c != nil && c.Contype == parser.ConstrType_CONSTR_DEFAULT {
                            // integer column has default value
                            return RuleHandlerMap[rule.Name].Message, nil
                        }
                    }
                }
            }
        }
    }
    return "", nil
}

func isIntegerColumn(columnDefNode *parser.Node) bool {
    columnDef := columnDefNode.GetColumnDef()
    if columnDef == nil {
        return false
    }
    
    types := columnDef.TypeName.Names
    return types[len(types)-1].GetString_().Str == "int4"
}
[/CODE]
[TEST]
func TestRulePgDDLTableIntegerFieldMustHaveDefaultValue(t *testing.T) {
    testSingleSqlAudit(PgDDLTableIntegerFieldMustHaveDefaultValue, t, `
        CREATE TABLE t1(
            id int primary key,
            id2 int
        );
    `, newTestResults())

    testSingleSqlAudit(PgDDLTableIntegerFieldMustHaveDefaultValue, t, `
        CREATE TABLE t1(
            id int primary key,
            id2 int default 123
        );
    `, newTestResults().add(PgDDLTableIntegerFieldMustHaveDefaultValue))

    testSingleSqlAudit(PgDDLTableIntegerFieldMustHaveDefaultValue, t, `
        CREATE TABLE t1(
            id int primary key,
            id2 int default 1.23
        );
    `, newTestResults().add(PgDDLTableIntegerFieldMustHaveDefaultValue))

    testSingleSqlAudit(PgDDLTableIntegerFieldMustHaveDefaultValue, t, `
        CREATE TABLE t1(
            id int primary key,
            id2 int NOT NULL
        );
    `, newTestResults().add(PgDDLTableIntegerFieldMustHaveDefaultValue))

    testSingleSqlAudit(PgDDLTableIntegerFieldMustHaveDefaultValue, t, `
        ALTER TABLE exist_tb_1
        ADD COLUMN v3 int
    `,  newTestResults())

    testSingleSqlAudit(PgDDLTableIntegerFieldMustHaveDefaultValue, t, `
        ALTER TABLE exist_tb_1
        ADD COLUMN v3 int default 123
    `,  newTestResults().add(PgDDLTableIntegerFieldMustHaveDefaultValue))

    testSingleSqlAudit(PgDDLTableIntegerFieldMustHaveDefaultValue, t, `
        ALTER TABLE exist_tb_1
        ADD COLUMN v3 int default 1.23
    `,  newTestResults().add(PgDDLTableIntegerFieldMustHaveDefaultValue))

    testSingleSqlAudit(PgDDLTableIntegerFieldMustHaveDefaultValue, t, `
        ALTER TABLE exist_tb_1
        ADD COLUMN v3 int NOT NULL
    `,  newTestResults().add(PgDDLTableIntegerFieldMustHaveDefaultValue))
}
[/TEST]
``` 

讲规则调整为: "In PostgreSQL(PG), for DDL statement: integer columns must have a default value, the default value must be '123'", 中文对应修改. 效果: 

产生了与 'CURRENT_TIMESTAMP' 规则的混乱

```
[CODE]
func rulePgDDLTableIntegerFieldMustHaveDefaultValue(ctx context.Context, rule *driverV2.Rule, astSQL interface{}, nextSQL []string) (string, error) {
    node, ok := astSQL.(*parser.RawStmt)
    if !ok {
        return "", fmt.Errorf("invalid type of ast of rule[%v]", rule.Name)
    }

    switch stmt := node.GetStmt().GetNode().(type) {
    case *parser.Node_CreateStmt:
        // check "create table"
        
        for _, elt := range stmt.CreateStmt.TableElts {
            //if !isColumnDef(elt) {
            //    continue
            //}

            columnDef := elt.(*parser.Node_ColumnDef)

            if !isIntegerColumn(columnDef) {
                // integer column has default value
                if len(columnDef.ColumnDef.Constraints) == 0 {
                    return RuleHandlerMap[rule.Name].Message, nil
                }

                for _, constraint := range columnDef.ColumnDef.Constraints {
                    c := constraint.GetConstraint()
                    if c != nil && c.Contype == parser.ConstrType_CONSTR_DEFAULT {
                        if fn := c.RawExpr.GetSqlvalueFunction(); nil != fn {
                            if parser.SQLValueFunctionOp_SVFOP_CURRENT_TIMESTAMP == fn.Op {
                                return RuleHandlerMap[rule.Name].Message, nil
                            }
                        }
                    }
                }

                // integer column has no default value
                return RuleHandlerMap[rule.Name].Message, nil
            }

            // check "alter table add column"
            if !isColumnDef(elt) {
                continue
            }

            columnDef = elt.(*parser.Node_ColumnDef)

            if !isIntegerColumn(columnDef) {
                // integer column has default value
                if len(columnDef.ColumnDef.Constraints) == 0 {
                    return RuleHandlerMap[rule.Name].Message, nil
                }

                for _, constraint := range columnDef.ColumnDef.Constraints {
                    c := constraint.GetConstraint()
                    if c != nil && c.Contype == parser.ConstrType_CONSTR_DEFAULT {
                        if fn := c.RawExpr.GetSqlvalueFunction(); nil != fn {
                            if parser.SQLValueFunctionOp_SVFOP_CURRENT_TIMESTAMP == fn.Op {
                                return RuleHandlerMap[rule.Name].Message, nil
                            }
                        }
                    }
                }

                // integer column has no default value
                return RuleHandlerMap[rule.Name].Message, nil
            }
        }
    case *parser.Node_AlterTableStmt:
        cmds := stmt.AlterTableStmt.GetCmds()
        for _, cmd := range cmds {
            alterTableCmd := cmd.GetAlterTableCmd()
            
            if alterTableCmd.GetSubtype() != parser.AlterTableType_AT_AddColumn {
                // check "alter table add column"
                columnDef := alterTableCmd.Def.GetColumnDef()
                if !isIntegerColumn(columnDef) {
                    // integer column has default value
                    if len(columnDef.Constraints) == 0 {
                        return RuleHandlerMap[rule.Name].Message, nil
                    }

                    for _, constraint := range columnDef.Constraints {
                        c := constraint.GetConstraint()
                        if c != nil && c.Contype == parser.ConstrType_CONSTR_DEFAULT {
                            if fn := c.RawExpr.GetSqlvalueFunction(); nil != fn {
                                if parser.SQLValueFunctionOp_SVFOP_CURRENT_TIMESTAMP == fn.Op {
                                    return RuleHandlerMap[rule.Name].Message, nil
                                }
                            }
                        }
                    }

                    // integer column has no default value
                    return RuleHandlerMap[rule.Name].Message, nil
                }
            }
        }
    }
    return "", nil
}

func isColumnDef(node *parser.Node) bool {
    if nil == node { 
        return false
    }
    
    switch node.GetNode().(type) {
    case *parser.Node_ColumnDef:
        return true
    default:
        return false
    }
}

func isIntegerColumn(columnDef *parser.ColumnDef) bool {
    if nil == columnDef { 
        return false
    }
    
    types := columnDef.TypeName.Names
    if (types[len(types)-1].GetString_().Str != "int") {
        return false
    }
    
    for _, constraint := range columnDef.Constraints {
        c := constraint.GetConstraint()
        if c != nil && c.Contype == parser.ConstrType_CONSTR_DEFAULT {
            if fn := c.RawExpr.GetSqlvalueFunction(); nil != fn {
                if parser.SQLValueFunctionOp_SVFOP_CURRENT_TIMESTAMP == fn.Op {
                    return false
                }
            }
        }
    }
    
    return true
}
[/CODE]
[TEST]
func TestRulePgDDLTableIntegerFieldMustHaveDefaultValue(t *testing.T) {
    rule := RuleHandlerMap[PgDDLTableIntegerFieldMustHaveDefaultValue].Rule
    rule.AllowOffline = true
    defer func() {
        rule.AllowOffline = false
    }()

    testSingleSqlAudit(PgDDLTableIntegerFieldMustHaveDefaultValue, t, `
        CREATE TABLE t1(
            id int primary key default 1,
            create_time datetime default current_timestamp
        );
    `)
    testSingleSqlAudit(PgDDLTableIntegerFieldMustHaveDefaultValue, t, `
        CREATE TABLE t1(
            id int
        );
    `, newTestResults().add(PgDDLTableIntegerFieldMustHaveDefaultValue))
    testSingleSqlAudit(PgDDLTableIntegerFieldMustHaveDefaultValue, t, `
        CREATE TABLE t1(
            id int default 1
        );
    `)
    testSingleSqlAudit(PgDDLTableIntegerFieldMustHaveDefaultValue, t, `
        ALTER TABLE exist_tb_1 ADD COLUMN id_next int
    `)
    testSingleSqlAudit(PgDDLTableIntegerFieldMustHaveDefaultValue, t, `
        ALTER TABLE exist_tb_1 ADD COLUMN id_next int default 1
    `)
    testSingleSqlAudit(PgDDLTableIntegerFieldMustHaveDefaultValue, t, `
        ALTER TABLE exist_tb_1 ADD COLUMN id_next int default '123'
    `)
    testSingleSqlAudit(PgDDLTableIntegerFieldMustHaveDefaultValue, t, `
        ALTER TABLE exist_tb_1 ADD COLUMN id_next int default current_timestamp
    `, newTestResults().add(PgDDLTableIntegerFieldMustHaveDefaultValue))
}
[/TEST]
``` 

尝试修订更多的PG规则, 并将MySQL规则去除

# 逐步增加PG规则, 目标: 测试"对于PostgreSQL的DDL, integer类型的字段必须定义默认值"

已去除MySQL规则

增加了MySQL规则"TIMESTAMP 类型的列必须添加默认值", 测试生成相关的PG规则: 

```
[CODE]
func rulePgDDLCheckColumnTimestampWithoutDefault(ctx context.Context, rule *driverV2.Rule, astSQL interface{}, nextSQL []string) (string, error) {
    node, ok := astSQL.(*parser.RawStmt)
    if !ok {
        return "", fmt.Errorf("invalid type of ast of rule[%v]", rule.Name)
    }

    switch stmt := node.GetStmt().GetNode().(type) {
    case *parser.Node_CreateStmt:
        // check "create table"
        
        for _, elt := range stmt.CreateStmt.TableElts {
            //if the column type is "timestamp" and it has no DEFAULT
            if isColumnTypeIs(elt, "timestamp") && 
                !isColumnHasConstraint(elt, parser.ConstrType_CONSTR_DEFAULT) {
                return RuleHandlerMap[rule.Name].Message, nil
            }
        }
    case *parser.Node_AlterTableStmt:
        for _, cmd := range getAlterTableCommandsByTypes(stmt.AlterTableStmt, parser.AlterTableType_AT_AddColumn) {
            //it is "alter table add column"
            
            columnDef := cmd.Def.GetColumnDef()
            if columnDef == nil {
                continue
            }

            //if the column type is "timestamp" and it has no DEFAULT
            if isColumnTypeIs(cmd.Def, "timestamp") && 
                !isColumnHasConstraint(cmd.Def, parser.ConstrType_CONSTR_DEFAULT) {
                return RuleHandlerMap[rule.Name].Message, nil
            }
        }
    }
    return "", nil
}

//check if a PostgreSQL column is of type _targetType_ (varchar/integer/...) or not
func isColumnTypeIs(columnDefNode *parser.Node, targetType string) bool {
    columnDef := columnDefNode.GetColumnDef()
    if nil == columnDef { 
        return false
    }
    
    typeName := columnDef.TypeName.Names[len(columnDef.TypeName.Names)-1].GetString_().Str
    return typeName == targetType
}

//check if a PostgreSQL column has constraint _targetConstraint_ (DEFAULT/PRIMARY/FOREIGN/...) or not
func isColumnHasConstraint(columnDefNode *parser.Node, targetConstraint parser.ConstrType) bool {
    columnDef := columnDefNode.GetColumnDef()
    if nil == columnDef { 
        return false
    }
    
    for _, constraint := range columnDef.Constraints {
        c := constraint.GetConstraint()
        if c != nil && c.Contype == targetConstraint {
            return true
        }
    }
    return false
}

//get PostgreSQL alter-table commands by command types
func getAlterTableCommandsByTypes(alterTableStmt *parser.AlterTableStmt, ts ...parser.AlterTableType) []*parser.AlterTableCmd {
    ret := []*parser.AlterTableCmd{}
    
    if alterTableStmt == nil || alterTableStmt.GetCmds() == nil {
        return ret
    }
    
	for _, cmd := range alterTableStmt.GetCmds() {
        alterTableCmd := cmd.GetAlterTableCmd()
		for _, tp := range ts {
			if alterTableCmd.GetSubtype() == tp {
				ret = append(ret, alterTableCmd)
			}
		}
	}
	return ret
}
[/CODE]
[TEST]
func TestRulePgDDLCheckColumnTimestampWithoutDefault(t *testing.T) {
    //check "create table", the column has "TIMESTAMP" type and has no DEFAULT
    testSingleSqlAudit(PgDDLCheckColumnTimestampWithoutDefault, t, `
        CREATE TABLE t1(
            id int primary key,
            id2 integer
        );
        `, newTestResults().add(PgDDLCheckColumnTimestampWithoutDefault))

    //check "create table", the column has "DATETIME" type and has DEFAULT
    testSingleSqlAudit(PgDDLCheckColumnTimestampWithoutDefault, t, `
        CREATE TABLE t1(
            id int primary key,
            id2 datetime default current_timestamp
        );
        `, newTestResults())

    //check "alter table add column", the column has "TIMESTAMP" type and has no DEFAULT
    testSingleSqlAudit(PgDDLCheckColumnTimestampWithoutDefault, t, `
        ALTER TABLE exist_tb_1
        ADD COLUMN v3 integer
        `, newTestResults().add(PgDDLCheckColumnTimestampWithoutDefault))

    //check "alter table add column", the column has "DATETIME" type and has DEFAULT
    testSingleSqlAudit(PgDDLCheckColumnTimestampWithoutDefault, t, `
        ALTER TABLE exist_tb_1
        ADD COLUMN v3 datetime default current_timestamp
        `, newTestResults())
}
[/TEST]
``` 

仅代码中对于timestamp和datetime的引用部分有误, 其他质量很高 (MySQL规则可以推广到PG规则)

再次测试规则推广 (MySQL -> PG)

增加MySQL规则"CHAR长度大于20时，必须使用VARCHAR类型"作为训练数据, 新增辅助函数getColumnLength ,尝试推广到PG, 效果欠佳: 

```
[CODE]
func rulePgDDLCheckColumnCharLength(ctx context.Context, rule *driverV2.Rule, astSQL interface{}, nextSQL []string) (string, error) {
    node, ok := astSQL.(*parser.RawStmt)
    if !ok {
        return "", fmt.Errorf("invalid type of ast of rule[%v]", rule.Name)
    }

    switch stmt := node.GetStmt().GetNode().(type) {
    case *parser.Node_CreateStmt:
        // check "create table"
        
        for _, elt := range stmt.CreateStmt.TableElts {
            //if the column type is char or varchar, and the column length > 20
            if (parser.SqlfmtDDLTableColType[parser.ConstrType_CONSTR_TYPE_CHAR] || 
                parser.SqlfmtDDLTableColType[parser.ConstrType_CONSTR_TYPE_VARCHAR]) && 
                elt.Collation != 30 //COLLATE "UTF8"
            {
                return RuleHandlerMap[rule.Name].Message, nil
            }
        }
    case *parser.Node_AlterTableStmt:
        for _, cmd := range getAlterTableCommandsByTypes(stmt.AlterTableStmt, parser.AlterTableType_AT_AddColumn) {
            //it is "alter table add column"
            
            columnDef := cmd.Def.GetColumnDef()
            if columnDef == nil {
                continue
            }

            //if the column type is char or varchar, and the column length > 20
            if (parser.SqlfmtDDLTableColType[parser.ConstrType_CONSTR_TYPE_CHAR] || 
                parser.SqlfmtDDLTableColType[parser.ConstrType_CONSTR_TYPE_VARCHAR]) && 
                columnDef.Collation != 30 //COLLATE "UTF8"
            {
                return RuleHandlerMap[rule.Name].Message, nil
            }
        }
    }
    return "", nil
}
[/CODE]
[/INST] 
[TEST]
func TestRulePgDDLCheckColumnCharLength(t *testing.T) {
    //create table, char length <= 20
    testSingleSqlAudit(PgDDLCheckColumnCharLength, t, `
        CREATE TABLE t1(
            id int primary key,
            id2 char(10)
        );
        `, newTestResults())

    //create table, char length > 20, without COLLATE
    testSingleSqlAudit(PgDDLCheckColumnCharLength, t, `
        CREATE TABLE t1(
            id int primary key,
            id2 char(22)
        );
        `, newTestResults().add(PgDDLCheckColumnCharLength))

    //create table, char length > 20, with COLLATE
    testSingleSqlAudit(PgDDLCheckColumnCharLength, t, `
        CREATE TABLE t1(
            id int primary key,
            id2 char(22) collate "UTF8"
        );
        `, newTestResults())

    //alter table add column, char length <= 20
    testSingleSqlAudit(PgDDLCheckColumnCharLength, t, `
        ALTER TABLE exist_tb_1
        ADD COLUMN v3 char(10);
        `, newTestResults())

    //alter table add column, char length > 20, without COLLATE
    testSingleSqlAudit(PgDDLCheckColumnCharLength, t, `
        ALTER TABLE exist_tb_1
        ADD COLUMN v3 char(22);
        `, newTestResults().add(PgDDLCheckColumnCharLength))

    //alter table add column, char length > 20, with COLLATE
    testSingleSqlAudit(PgDDLCheckColumnCharLength, t, `
        ALTER TABLE exist_tb_1
        ADD COLUMN v3 char(22) collate "UTF8";
        `, newTestResults())
}
[/TEST]
``` 

再次生成, 效果很好: 

需要修正varchar的判断, 完善PG的getColumnLength函数, 修正alter types

```
[CODE]
func rulePgDDLCheckColumnCharLength(ctx context.Context, rule *driverV2.Rule, astSQL interface{}, nextSQL []string) (string, error) {
    node, ok := astSQL.(*parser.RawStmt)
    if !ok {
        return "", fmt.Errorf("invalid type of ast of rule[%v]", rule.Name)
    }

    switch stmt := node.GetStmt().GetNode().(type) {
    case *parser.Node_CreateStmt:
        // check "create table"
        
        for _, elt := range stmt.CreateStmt.TableElts {
            //if the column type is char or varchar, and the length > 20, it violates the rule
            if (isColumnTypeIs(elt, "varchar") || isColumnTypeIs(elt, "char")) && 
                getColumnLength(elt) > 20 {
                return RuleHandlerMap[rule.Name].Message, nil
            }
        }
    case *parser.Node_AlterTableStmt:
        for _, cmd := range getAlterTableCommandsByTypes(stmt.AlterTableStmt, parser.AlterTableType_AT_AddColumn) {
            //it is "alter table add column"
            
            //the column type is char or varchar, and the length > 20, it violates the rule
            if (isColumnTypeIs(cmd.Def, "varchar") || isColumnTypeIs(cmd.Def, "char")) && 
                getColumnLength(cmd.Def) > 20 {
                return RuleHandlerMap[rule.Name].Message, nil
            }
        }
    }
    return "", nil
}

//check if a PostgreSQL column is of type _targetType_ (varchar or char) or not
func isColumnTypeIs(columnDefNode *parser.Node, targetType string) bool {
    columnDef := columnDefNode.GetColumnDef()
    if nil == columnDef { 
        return false
    }

    types := columnDef.TypeName.Names
    return types[len(types)-1].GetString_().Str == targetType
}

//get the length of a PostgreSQL column in definition
func getColumnLength(columnDefNode *parser.Node) int {
    columnDef := columnDefNode.GetColumnDef()
    if nil == columnDef { 
        return 0
    }

    return columnDef.TypeName.Names[0].GetString_().Str
}

//get PostgreSQL alter-table commands by command types
func getAlterTableCommandsByTypes(alterTableStmt *parser.AlterTableStmt, ts ...parser.AlterTableType) []*parser.AlterTableCmd {
    ret := []*parser.AlterTableCmd{}
    
    if alterTableStmt == nil || alterTableStmt.GetCmds() == nil {
        return ret
    }
    
	for _, cmd := range alterTableStmt.GetCmds() {
        alterTableCmd := cmd.GetAlterTableCmd()
		for _, tp := range ts {
			if alterTableCmd.GetSubtype() == tp {
				ret = append(ret, alterTableCmd)
			}
		}
	}
	return ret
}
[/CODE]
[TEST]
func TestRulePgDDLCheckColumnCharLength(t *testing.T) {
    //create table, char length <= 20
    testSingleSqlAudit(PgDDLCheckColumnCharLength, t, `
        CREATE TABLE t1(
            id int primary key,
            id2 char(20)
        );
    `, newTestResults())

    //create table, char length > 20
    testSingleSqlAudit(PgDDLCheckColumnCharLength, t, `
        CREATE TABLE t1(
            id int primary key,
            id2 char(22)
        );
    `, newTestResults().add(PgDDLCheckColumnCharLength))

    //alter table add column, char length <= 20
    testSingleSqlAudit(PgDDLCheckColumnCharLength, t, `
        ALTER TABLE exist_tb_1
        ADD COLUMN v3 char(20);
    `, newTestResults())

    //alter table add column, char length > 20
    testSingleSqlAudit(PgDDLCheckColumnCharLength, t, `
        ALTER TABLE exist_tb_1
        ADD COLUMN v3 char(22);
    `, newTestResults().add(PgDDLCheckColumnCharLength))
}
[/TEST]
``` 

待测试: 

```
user_prompt = """
<s>[INST] 
<<SYS>> 
Given a SQL audit rule(wrapped by [RULE][/RULE]), and rule metadata (wrapped by [META][/META]), you should: 
1. write a rule function to check if SQL ast matches the SQL audit rule or not, and helper functions if needed, wrapped by [CODE][/CODE]
2. write unit test to test the rule function, wrapped by [TEST][/TEST]
<</SYS>> 
[RULE]"In PostgreSQL(PG), for DDL statement: column whose type is integer must have a default value"[/RULE]
[META]
{
    Rule: driverV2.Rule{
        Name:       PgDDLTableIntegerFieldMustHaveDefaultValue,
        Desc:       "对于PostgreSQL的DDL, integer类型的字段必须定义默认值",
        Annotation: "",
        Level:      driverV2.RuleLevelWarn,
        Category:   RuleTypeDDLConvention,
    },
    Message:              "对于PostgreSQL的DDL, integer类型的字段必须定义默认值",
    AstSQLHandler:        rulePgDDLTableIntegerFieldMustHaveDefaultValue,
    AllowOffline:         true,
    NotAllowOfflineStmts: nil,
}
[/META]
[/INST] 
"""
output = infill(user_prompt, model, tokenizer, max_new_tokens = 2048, temperature = 0.6)
print(output)
```
