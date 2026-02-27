---
title: 20230920 - 测试用codellama, 进行SQLe规则代码的微调
confluence_page_id: 2589145
created_at: 2023-09-20T05:11:34+00:00
updated_at: 2023-09-22T08:04:06+00:00
---

# 使用SQLe的DDL规则代码, 进行微调

微调epoch = 10, train_epoch_loss=0.1294, temperature=0.1

```
规则: For DDL statement: Using FIRST or AFTER is prohibited for altering table fields
代码: 
func disableAlterFieldType(input *RuleHandlerInput) error {
        switch stmt := input.Node.(type) {
    case *ast.AlterTableStmt:
        for _, spec := range stmt.Specs {
            for _, pos := range []int{ast.ColumnPositionFirst, ast.ColumnPositionAfter} {
                if spec.Pos == pos {
                    addResult(input.Res, input.Rule, DDLCheckColumnFieldType)
                    return nil
                }
            }
        }
    }
    return nil
}
 
---
 
规则: For DDL statement: Add new columns is prohibited
代码: 
   func disableAddColumn(input *RuleHandlerInput) error {
        switch stmt := input.Node.(type) {
        case *ast.AlterTableStmt:
            for _, spec := range stmt.Specs {
                for _, new := range spec.NewColumns {
                    if new == nil {
                        continue
                    }
                    for _, op := range spec.Options {
                        if op.Tp == ast.TableOptionAddColumns {
                            addResult(input.Res, input.Rule, input.Rule.Name)
                            return nil
                        }
                    }
                }
            }
        }
        return nil
    }
 
---
 
规则: For DDL statement: Adding columns is prohibited
代码: 
   func disableAlter(input *RuleHandlerInput) error {
                switch stmt := input.Node.(type) {
                case *ast.AlterTableStmt:
                        if stmt.Specs == nil {
                                return nil
                        }
                        for _, spec := range stmt.Specs {
                                if spec.Tp == ast.AlterTableAddColumns {
                                        addResult(input.Res, input.Rule, input.Rule.Name)
                                        return nil
                                }
                        }
                }
                return nil
    }
 
--
 
规则: For DDL statement: Removing columns is prohibited
对比: 与训练数据相比, 仅DDLCheckColumnCantBeDeleted部分不同
代码: 
   func checkColumnCantBeDeleted(input *RuleHandlerInput) error {
                switch stmt := input.Node.(type) {
                case *ast.AlterTableStmt:
                        if len(stmt.Specs) > 0 {
                                for _, spec := range stmt.Specs {
                                        if spec.Tp == ast.AlterTableDropColumn {
                                                addResult(input.Res, input.Rule, DDLCheckColumnCantBeDeleted)
                                                return nil
                                        }
                                }
                        }
                }

                return nil
    }
``` 

# 尝试增加上下文, 增加类型定义, 修正测试数据 For DDL statement: Using FIRST or AFTER is prohibited for altering table fields

原始结果: 

```
规则: For DDL statement: Using FIRST or AFTER is prohibited for altering table fields
代码: 
func disableAlterFieldType(input *RuleHandlerInput) error {
        switch stmt := input.Node.(type) {
    case *ast.AlterTableStmt:
        for _, spec := range stmt.Specs {
            for _, pos := range []int{ast.ColumnPositionFirst, ast.ColumnPositionAfter} {
                if spec.Pos == pos {
                    addResult(input.Res, input.Rule, DDLCheckColumnFieldType)
                    return nil
                }
            }
        }
    }
    return nil
}
``` 

增加上下文, 调整提示词: 

```
code_context = """
//definitions

// AlterTableStmt is a statement to change the structure of a table.
// See https://dev.mysql.com/doc/refman/5.7/en/alter-table.html
type AlterTableStmt struct {
	ddlNode

	Table *TableName
	Specs []*AlterTableSpec
}

// AlterTableSpec represents alter table specification.
type AlterTableSpec struct {
	node

	// only supported by MariaDB 10.0.2+ (DROP COLUMN, CHANGE COLUMN, MODIFY COLUMN, DROP INDEX, DROP FOREIGN KEY, DROP PARTITION)
	// see https://mariadb.com/kb/en/library/alter-table/
	IfExists bool

	// only supported by MariaDB 10.0.2+ (ADD COLUMN, ADD PARTITION)
	// see https://mariadb.com/kb/en/library/alter-table/
	IfNotExists bool

	NoWriteToBinlog bool
	OnAllPartitions bool

	Tp              AlterTableType
	Name            string
	Constraint      *Constraint
	Options         []*TableOption
	OrderByList     []*AlterOrderItem
	NewTable        *TableName
	NewColumns      []*ColumnDef
	NewConstraints  []*Constraint
	OldColumnName   *ColumnName
	NewColumnName   *ColumnName
	Position        *ColumnPosition
	LockType        LockType
	Algorithm       AlgorithmType
	Comment         string
	FromKey         model.CIStr
	ToKey           model.CIStr
	Partition       *PartitionOptions
	PartitionNames  []model.CIStr
	PartDefinitions []*PartitionDefinition
	WithValidation  bool
	Num             uint64
	Visibility      IndexVisibility
	TiFlashReplica  *TiFlashReplicaSpec
}

// ColumnPosition Types
const (
	ColumnPositionNone ColumnPositionType = iota
	ColumnPositionFirst
	ColumnPositionAfter
)

// ColumnPosition represent the position of the newly added column
type ColumnPosition struct {
	node
	// Tp is either ColumnPositionNone, ColumnPositionFirst or ColumnPositionAfter.
	Tp ColumnPositionType
	// RelativeColumn is the column the newly added column after if type is ColumnPositionAfter
	RelativeColumn *ColumnName
}
"""
for case in cases:
    prompt = f"""<s>[INST] 
<<SYS>> 
Given a SQL audit rule, and some code context, write a function to check if SQL ast matches the SQL audit rule or not. Call `addResult` when there is a rule violation. 
You MUST follow the struct definitions in context when using the objects of the struct.
<</SYS>> 
[SQL audit rule]"{case["Desc_en"]}"[/SQL audit rule]
[Code context]
{code_context}
[/Code context]
[/INST] 
[CODE]{case["Code"]}[/CODE]</s>"""
``` 

temperature=0.4, 效果: 

```
func AlterTableFieldNotFirstAndAfter(input *RuleHandlerInput) error {
	    switch stmt := input.Node.(type) {
        case *ast.AlterTableStmt:
                if len(stmt.Specs) > 0 {
                        for _, spec := range stmt.Specs {
                                if spec.Tp == ast.AlterTableAddColumns || spec.Tp == ast.AlterTableChangeColumn {
                                        if spec.Position.Tp == ast.ColumnPositionFirst || spec.Position.Tp == ast.ColumnPositionAfter {
                                                addResult(input.Res, input.Rule, DDLCheckTableFieldNotFirstAndAfter)
                                        }
                                }
                        }
                }
                return nil
        default:
                return nil
        }
``` 

AB测试, 去掉上下文: 

```
func disableAlterTableWithPosition(input *RuleHandlerInput) error {
        var columnPositions []*ast.ColumnPosition
        switch stmt := input.Node.(type) {
        case *ast.AlterTableStmt:
                for _, spec := range stmt.Specs {
                        for _, newColumn := range spec.NewColumns {
                                columnPositions = append(columnPositions, newColumn.Position)
                        }
                }
        case *ast.AlterTableChangeColumn:
                columnPositions = append(columnPositions, spec.NewColumn.Position)
        default:
                return nil
        }

        for _, position := range columnPositions {
                if position.Tp == ast.ColumnPositionFirst || position.Tp == ast.ColumnPositionAfter {
                        addResult(input.Res, input.Rule, input.Rule.Name)
                        break
                }
        }

        return nil
}
``` 

AB测试, 加回上下文: 

```
func AlterTableFieldNotFirstAndAfter(input *RuleHandlerInput) error {
        switch stmt := input.Node.(type) {
        case *ast.AlterTableStmt:
                if len(stmt.Specs) > 0 {
                        for _, spec := range stmt.Specs {
                                if spec.Tp == ast.AlterTableAddColumns || spec.Tp == ast.AlterTableChangeColumn {
                                        if spec.Position.Tp == ast.ColumnPositionFirst || spec.Position.Tp == ast.ColumnPositionAfter {
                                                addResult(input.Res, input.Rule, DDLCheckTableFieldNotFirstAndAfter)
                                        }
                                }
                        }
                }
                return nil
        default:
                return nil
        }
``` 

在训练中保持上下文, 但在推断中去掉上下文: 

其中AlterTableAlterColumn是错误的 (?, 人类也无法通过这些信息区分Change和Alter的区别)

```
func alterTable(input *RuleHandlerInput) error {
        switch stmt := input.Node.(type) {
        case *ast.AlterTableStmt:
                if len(stmt.Specs) > 0 {
                        for _, spec := range stmt.Specs {
                                if spec.Tp == ast.AlterTableChangeColumn ||
                                        spec.Tp == ast.AlterTableAlterColumn {
                                        if spec.Position.Tp == ast.ColumnPositionFirst ||
                                                spec.Position.Tp == ast.ColumnPositionAfter {
                                                        addResult(input.Res, input.Rule, input.Rule.Name)
                                        }
                                }
                        }
                }
                return nil
        default:
                return nil
        }
``` 

带代码上下文, 测试几个规则, 均正确: 

```
[SQL audit rule]"For DDL statement: Add new columns is prohibited"[/SQL audit rule]
[CODE]
func hintAddColumn(input *RuleHandlerInput) error {
        switch stmt := input.Node.(type) {
        case *ast.AlterTableStmt:
                if len(stmt.Specs) > 0 {
                        for _, spec := range stmt.Specs {
                                if spec.Tp == ast.AlterTableAddColumns {
                                        addResult(input.Res, input.Rule, input.Rule.Name)
                                        break
                                }
                        }
                }
                return nil
        default:
                return nil
}
[/CODE]
 
---
 
[SQL audit rule]For DDL statement: Adding columns is prohibited[/SQL audit rule]
[CODE]
func checkAddColumn(input *RuleHandlerInput) error {
        switch stmt := input.Node.(type) {
        case *ast.AlterTableStmt:
                if len(stmt.Specs) > 0 {
                        for _, spec := range stmt.Specs {
                                if spec.Tp == ast.AlterTableAddColumns {
                                        addResult(input.Res, input.Rule, input.Rule.Name)
                                        break
                                }
                        }
                }
                return nil
        default:
                return nil
        }
}
[/CODE]
 
---
 
[SQL audit rule]For DDL statement: Removing columns is prohibited[/SQL audit rule]
[CODE]
func checkAddColumn(input *RuleHandlerInput) error {
        switch stmt := input.Node.(type) {
        case *ast.AlterTableStmt:
                if len(stmt.Specs) > 0 {
                        for _, spec := range stmt.Specs {
                                if spec.Tp == ast.AlterTableAddColumns {
                                        addResult(input.Res, input.Rule, input.Rule.Name)
                                        break
                                }
                        }
                }
                return nil
        default:
                return nil
        }
}
[/CODE]
``` 

# 尝试用微调取代上下文

增加训练数据, 写入引用代码: 

```
(训练数据样例)
code = {
    "name": "ast.AlterTableStmt",
    "code": """
// AlterTableStmt is a statement to change the structure of a table.
// See https://dev.mysql.com/doc/refman/5.7/en/alter-table.html
type AlterTableStmt struct {
	ddlNode

	Table *TableName
	Specs []*AlterTableSpec
}
"""
}
import_codes.append(code)
 
 
(多组code用作训练数据)
...
 
 
(prompt)
 
for code in import_codes:
    prompt = f"""<s><<SYS>> 
Given a SQL audit rule, and some code context, write a function to check if SQL ast matches the SQL audit rule or not. 
Here is the code context.
<</SYS>>
[INST]{code["name"]} definitions in code context[/INST]
[CODE]{code["code"]}[/CODE]</s>"""
    prompts.append({"prompt": prompt})
``` 

主任务提示词变更: 

```
[INST] 
<<SYS>> 
Given a SQL audit rule, and some code context, write a function to check if SQL ast matches the SQL audit rule or not. Call `addResult` when there is a rule violation. 
You MUST follow the struct definitions in code context when using the objects of the struct.
<</SYS>> 
[SQL audit rule]For DDL statement: Using FIRST or AFTER is prohibited for altering table fields[/SQL audit rule]
[/INST] 
[CODE]<FILL_ME>[/CODE]
``` 

效果 (temperature=0.4): 

  - Using FIRST or AFTER is prohibited for altering table fields 结果完全正确
  - Add/Remove的结果都不正确, 其中Add的结果完全为空, 无逻辑

```
[SQL audit rule]For DDL statement: Using FIRST or AFTER is prohibited for altering table fields[/SQL audit rule]
 
func disableAlterUseFirstAndAfter(input *RuleHandlerInput) error {
        switch stmt := input.Node.(type) {
        case *ast.AlterTableStmt:
                if len(stmt.Specs) > 0 {
                        for _, spec := range stmt.Specs {
                                if spec.Position == nil {
                                        continue
                                }
                                if spec.Position.Tp == ast.PositionFirst ||
                                        spec.Position.Tp == ast.PositionAfter {
                                        addResult(input.Res, input.Rule, DDLDisableAlterUseFirstAndAfter)
                                        return nil
                                }
                        }
                }
        }
        return nil
}
 
---
 
[SQL audit rule]For DDL statement: Add new columns is prohibited[/SQL audit rule]
func disableAddColumn(input *RuleHandlerInput) error {
        return input.AddResult(input.Rule, input.Rule.Name)
}

---

[SQL audit rule]For DDL statement: Adding columns is prohibited[/SQL audit rule]
func disableAddColumns(input *RuleHandlerInput) error {
        return input.DisableAddColumns()
}

---

[SQL audit rule]For DDL statement: Removing columns is prohibited[/SQL audit rule]
func checkDropColumn(input *RuleHandlerInput) error {
        min := input.Rule.Params.GetParam(DefaultSingleParamKeyName).Int()
        tables := []*ast.TableName{}
        switch stmt := input.Node.(type) {
        case *ast.AlterTableStmt:
                tables = append(tables, stmt.Table)
                if stmt.Specs == nil {
                        return nil
                }
                for _, spec := range stmt.Specs {
                        if spec.NewColumns == nil {
                                continue
                        }
                        if min >= len(spec.NewColumns) {
                                min = len(spec.NewColumns)
                        }
                }
        case *ast.DropColumnStmt:
                tables = append(tables, stmt.Table)
                if min >= len(stmt.Columns) {
                        min = len(stmt.Columns)
                }
        default:
                return nil
        }
        if min >= 1 {
                addResult(input.Res, input.Rule, input.Rule.Name)
        }
        return nil
}
 
---
 
[SQL audit rule]For DDL statement: Deleting columns is prohibited[/SQL audit rule]
func checkDropColumn(input *RuleHandlerInput) error {
        return input.CheckColumn(func(col *ast.ColumnDef) bool {
                if col.Tp == nil {
                        return false
                }
                if col.Tp.Tp == mysql.TypeDeleteRow {
                        addResult(input.Res, input.Rule, DDLCheckDropColumn)
                        return true
                }
                return false
        })
}
``` 

增加对AlterTableType的定义, 效果: 

```
[SQL audit rule]For DDL statement: Deleting columns is prohibited[/SQL audit rule]
func checkDropColumn(input *RuleHandlerInput) error {
        switch stmt := input.Node.(type) {
        case *ast.DropColumnTable:
                if len(stmt.Columns) > 0 {
                        addResult(input.Res, input.Rule, input.Rule.Name)
                }
                return nil
        default:
                return nil
        }
}
 
---
 
[SQL audit rule]For DDL statement: Removing columns is prohibited[/SQL audit rule]
func checkRemoveColumn(input *RuleHandlerInput) error {
        switch stmt := input.Node.(type) {
        case *ast.AlterTableStmt:
                if len(stmt.Specs) > 0 {
                        for _, spec := range stmt.Specs {
                                if spec.Tp == ast.AlterTableDropColumn {
                                        addResult(input.Res, input.Rule, DDLCheckDropColumn)
                                        return nil
                                }
                        }
                }
        default:
                return nil
        }
        return nil
}
 
---
 
[SQL audit rule]For DDL statement: Using FIRST or AFTER is prohibited for altering table fields[/SQL audit rule]
func checkAlterUseFirstAndAfter(input *RuleHandlerInput) error {
        var fieldNames []string
        var hasError bool
        switch stmt := input.Node.(type) {
        case *ast.AlterTableStmt:
                for _, spec := range stmt.Specs {
                        if spec.Position == nil {
                                continue
                        }
                        hasError = true
                        fieldNames = append(fieldNames, spec.NewFieldsName...)
                }
        default:
                return nil
        }
        if hasError {
                addResult(input.Res, input.Rule, input.Rule.Name, strings.Join(fieldNames, " ,"))
        }
        return nil
}
 
---
 
[SQL audit rule]For DDL statement: Adding columns is prohibited[/SQL audit rule]
func disableAlterAddColumn(input *RuleHandlerInput) error {
        switch stmt := input.Node.(type) {
        case *ast.AlterTableStmt:
                if stmt.Specs == nil {
                        return nil
                }
                for _, spec := range stmt.Specs {
                        if spec.Tp == ast.AlterTableAddColumns {
                                addResult(input.Res, input.Rule, DDLDisableAlterAddColumn)
                                return nil
                        }
                }
        }
        return nil
}
 
 
---
 
[SQL audit rule]For DDL statement: When altering table field, Using FIRST or AFTER as field position is prohibited[/SQL audit rule]

func checkAlterUseFirstAndAfter(input *RuleHandlerInput) error {
        var hasError bool
        var fieldOrderByAlterAndPosition []string
        switch stmt := input.Node.(type) {
        case *ast.AlterTableStmt:
                for _, spec := range stmt.Specs {
                        for _, newColumn := range spec.NewColumns {
                                hasColumnPosition := false
                                // if has update position, it means it has update field position, so ignore it
                                // https://github.com/actiontech/sqle/issues/279
                                if HasAlterFieldUpdatePosition(newColumn.Constraint) {
                                        continue
                                }
                                // if new column not define position, it means update field position, so ignore it
                                if newColumn.Position == nil {
                                        continue
                                }
                                fieldOrderByAlterAndPosition = append(fieldOrderByAlterAndPosition, newColumn.Name.Name.L)
                                hasColumnPosition = true
                        }
                        // https://github.com/actiontech/sqle/issues/279
                        if spec.Constraint == nil {
                                continue
                        }
                        for _, constraint := range spec.Constraint {
                                if constraint.Tp == ast.ConstraintFieldUpdate {
                                        continue
                                }
                                fieldOrderByAlterAndPosition = append(fieldOrderByAlterAndPosition, constraint.Name)
                                hasColumnPosition = true
                        }
                        if hasColumnPosition {
                                sort.Strings(fieldOrderByAlterAndPosition)
                                // if strings not in order, it means update field position
                                if !IsInOrder(fieldOrderByAlterAndUpdate, fieldOrderByAlterAndPosition) {
                                        hasError = true
                                }
                        }
                }
                if hasError {
                        addResult(input.Res, input.Rule, DDLCheckAlterUseFirstAndAfter)
                }
                return nil
        }
        return nil
}
 
---
 
[SQL audit rule]For DDL statement: For DDL statement: Using FIRST or AFTER as position is prohibited for altering table fields[/SQL audit rule]

func checkAlterUseFirstAndAfter(input *RuleHandlerInput) error {
        var hasError bool
        tables := []string{}
        switch stmt := input.Node.(type) {
        case *ast.AlterTableStmt:
                for _, spec := range stmt.Specs {
                        if spec.Position == nil {
                                continue
                        }
                        if spec.Position.Tp == ast.ColumnPositionFirst ||
                                spec.Position.Tp == ast.ColumnPositionAfter {
                                hasError = true
                                tables = append(tables, spec.Table.Name.O)
                        }
                }
        default:
                return nil
        }
        if hasError {
                for _, table := range tables {
                        addResult(input.Res, input.Rule, DDLCheckAlterUseFirstAndAfter, table)
                }
        }
        return nil
}
 
---
[SQL audit rule]For DDL statement: Using FIRST as position is prohibited for altering table fields[/SQL audit rule]

func checkAlterUseFirst(input *RuleHandlerInput) error {
        var hasError bool
        var fields []string
        switch stmt := input.Node.(type) {
        case *ast.AlterTableStmt:
                for _, spec := range stmt.Specs {
                        for _, newColumn := range spec.NewColumns {
                                if newColumn.Tp != nil && newColumn.Tp.Pos == ast.ColumnPositionFirst {
                                        hasError = true
                                        fields = append(fields, newColumn.Name.OrigStr())
                                }
                        }
                }
        default:
                return nil
        }
        if hasError {
                addResult(input.Res, input.Rule, input.Rule.Name, strings.Join(fields, " , "))
        }
        return nil
}
 
---
 
[SQL audit rule]For DDL statement: Table's first-field must have a NOT NULL constraint[/SQL audit rule]

func checkColumnNotNullFirst(input *RuleHandlerInput) error {
        var columnShouldBeNotNull []string
        switch stmt := input.Node.(type) {
        case *ast.CreateTableStmt:
                for _, col := range stmt.Cols {
                        for i, option := range col.Options {
                                if i == 0 && option.Tp == ast.ColumnOptionNotNull {
                                        continue
                                }
                                columnShouldBeNotNull = append(columnShouldBeNotNull, col.Name.Name.O)
                        }
                }
        case *ast.AlterTableStmt:
                for _, spec := range stmt.Specs {
                        for i, option := range spec.Options {
                                if i == 0 && option.Tp == ast.ColumnOptionNotNull {
                                        continue
                                }
                                columnShouldBeNotNull = append(columnShouldBeNotNull, spec.NewColumns[i].Name.Name.O)
                        }
                }
        default:
                return nil
        }
        if len(columnShouldBeNotNull) > 0 {
                addResult(input.Res, input.Rule, DDLCheckColumnNotNullFirst, strings.Join(columnShouldBeNotNull, ","))
        }
        return nil
}
``` 

# 测试AI修复编译错误

推断命令: 

```
(CUDA_VISIBLE_DEVICES=0 python ./code_infilling_example.py --model_name codellama/CodeLlama-7b-Instruct-hf --quantization --prompt_file test2.prompt.txt --use_fast_kernels False --max_new_tokens 1000 --temperature 0.4) | tee test.log
``` 

提示词: 

```
I wrote a function to check if SQL ast matches a rule "For DDL statement: It is recommended for columns and tables to use the same character set" or not  : 
"""
func recommendTableColumnCharsetSame(input *RuleHandlerInput) error {
	switch stmt := input.Node.(type) {
	case *ast.CreateTableStmt, *ast.CreateTableStmt2:
		for _, col := range stmt.Cols {
			if col.Tp.Charset != "" {
				addResult(input.Res, input.Rule, input.Rule.Name)
				break
			}
		}
		return nil
	default:
		return nil
	}
}
"""

It run in an error when compiling: "ERROR: there is no CreateTableStmt2 in ast package". 
You should fix the function code, only the function.
``` 

效果: 

````
```go
func recommendTableColumnCharsetSame(input *RuleHandlerInput) error {
        switch stmt := input.Node.(type) {
        case *ast.CreateTableStmt:
                for _, col := range stmt.Cols {
                        if col.Tp.Charset != "" {
                                addResult(input.Res, input.Rule, input.Rule.Name)
                                break
                        }
                }
                return nil
        default:
                return nil
        }
}
```
````
