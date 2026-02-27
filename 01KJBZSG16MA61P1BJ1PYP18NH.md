---
title: 20250504 - 研究SQL优化模型微调 - 扩展Rule-3
confluence_page_id: 3801870
created_at: 2025-05-04T17:05:33+00:00
updated_at: 2025-05-08T10:09:39+00:00
---

# 前继

[20250415 - 研究SQL优化模型微调 - 重新梳理SQL优化的数据构建过程]

# 训练1

commit: 1f6d41f7296d4300bd405fc644cd35ca31223a47

生成数据的脚本: /opt/huangyan/sql-ai/make_sql_analyze_data.json_format.ipynb

生成规则: 

```
{
        "id": 2,
        "violate_desc": """在SQL某处新增一个LEFT JOIN连接, 连接一个新表, 连接条件需要是 等值条件 或包含等值条件的AND条件, 连接条件中使用的新表的列 必须是主键或者带有唯一约束的列(你可以通过列名暗示这一点)""",
        "violate_key": "新表名",
        "violate_analyze": "LEFT JOIN \"{violate_value}\", 满足 外连接删除 的条件, 可以删除这个LEFT JOIN",
        "rewrite_rule": """对同时满足以下条件的LEFT JOIN连接进行消除(RIGHT JOIN同理):
- 不能被语句引用: LEFT JOIN右表 在SELECT、WHERE、GROUP BY、HAVING等子句中 *均未被引用*
- 连接谓词: 连接谓词是纯等值连接
- 消除后逻辑一致: 消除后的查询逻辑与原 LEFT JOIN 的行一一对应，即每个左表行在结果中出现的次数与消除前严格一致
""",
        "allowed_sql_regex": r"^\s*(SELECT|WITH)\b",
        "rule_checker": None,
        "max_data": 50
    }

``` 

训练数据量50, 其中存在的问题: 

  1. 对于"纯等值连接" 没有给出详细的标准
  2. 对于"消除后逻辑一致" 没有给出详细的标准
  3. 没有给表结构, 也就是说对"消除后逻辑一致"的准确判断并没有足够信息 (但是否需要准确判断, 尚没有想清楚)

训练效果: 

  - case 1
    - 使用2轮, 消除了两个LEFT JOIN, 判断和执行正确
    - 但: 如果对于左表的一个记录, 右表有对应的多个记录, 那么消除LEFT JOIN后, 返回的行数会变
      - 但: 根据业务含义, 因为外查询没有用右表的列, 那么认为从业务含义上, 不存在 "左表的一个记录, 右表有对应的多个记录", 其在业务上没意义
  - case 2
    - 使用1轮, 消除了两个LEFT JOIN, 判断和执行正确
    - 比起case 1, 其增加了DISTINCT, 其意图是确保消除LEFT JOIN时, 即使"左表的一个记录, 右表有对应的多个记录", 行数也不变
      - 也就是说case 1不应被消除LEFT JOIN
  - case 3
    - 使用2轮, 消除了两个LEFT JOIN, 判断和执行正确
    - 比起case 1, 其增加了GROUP BY, 其意图是确保消除LEFT JOIN时, 即使"左表的一个记录, 右表有对应的多个记录", 行数也不变
      - 也就是说case 1不应被消除LEFT JOIN
  - case 11
    - 使用1轮, 消除了一个RIGHT JOIN, 判断和执行正确
    - 但: 如有对于右表的一个记录, 左表有对应的多个记录, 那么消除RIGHT JOIN后, 返回的行数会变
      - 但: 根据业务含义, 其是否存在这种反例, 待判断
  - case 12
    - 不需要优化, LEFT JOIN的条件是一个非等值条件

从以上简单的测试, 得到如下结论: 

  - 因为训练数据的原因, 还不能处理"表上没有强制的唯一性约束"这种情况. 将所有情况都假设成了"表上存在必要的唯一性约束"
    - 模型应具有这样的能力: 
      - 根据已知的表结构, 确定是否有唯一性约束
      - 根据"业务含义猜测", 猜测是否有唯一性约束 ??
  - 重新梳理消除外连接的条件: <https://poe.com/s/wNLNqOpbZTWxZxLKRkHp>

# 注意

规则3是"外连接消除", 其中存在歧义: "删除外连接", "将外连接转成内连接", 需要修改规则的描述, 将其强制为"删除外连接".

新的规则描述: 

```
外连接删除: 对于同时满足以下两个条件的 外连接(形如 `T1 LEFT JOIN T2 ON ...` 或 `T1 RIGHT JOIN T2 ON ...` 的外连接), 将其从查询中完全删除:

#### 1. **不可被引用**  
外连接的 **右表（被保留表）** 在 `SELECT`、`WHERE`、`GROUP BY`、`HAVING` 等子句中均未被引用.  
- **LEFT JOIN** 的右表是 `T2`, **RIGHT JOIN** 的右表是 `T2`（而非 `T1`）.  

#### 2. **保证行数不变（需满足以下任一条件）**  

##### **条件 A: 等值连接 + 唯一性约束**  
- `ON` 子句中必须至少包含一个形如 `T1.c1 = T2.c2` 的等值条件.  
- **右表的连接列必须有 `UNIQUE` 或 `PRIMARY KEY` 约束**:   
  - 对 `LEFT JOIN`, 右表为 `T2`, 需 `T2.c2` 唯一.  
  - 对 `RIGHT JOIN`, 右表为 `T2`, 需 `T2.c2` 唯一且定义为 `NOT NULL`.  
  
  ##### **条件 B: 显式去重**  
- 查询结果已通过 `GROUP BY` 或 `DISTINCT` 显式去重, 且去重后的行数与原外连接中 **被保留表** 的行数一致.  

##### **条件 C: 特殊连接语义**  
- **永假条件**: `ON` 子句为永假（如 `1=0`）, 且右表未被引用.  
- **永真条件 + 右表为单行**: `ON` 子句为永真（如 `1=1`）, 且右表是单行表（如静态配置表或含 `LIMIT 1`）.  
- **右表为空表**: 右表无数据, 且未被引用.  

注意: 由于无法提供表结构信息, 你可能无法确认你需要的信息(比如唯一性约束), 你可以根据表名/列名/等信息进行假设推测(比如: 列名表示其很像一个唯一列, 你可以提出这个假设. 如果列名无法判断, 则不能作出假设): 你需要明确写出\"假设推测\"一节(级别等同\"分析结果\"), 并给出你假设的依据, 然后根据\"假设推测\"进行规则推理
``` 

其中指定了规则为"删除外连接", 并且允许进行"假设推测" (而不强制 提供表结构, 或者表结构必须显式表达唯一性)

# 训练2

commit: 0b7b4c51fd801a29a67e1b24060fba144d0dbd67

数据量: 50

训练效果: 

  - case 1
    - 第一轮: 没有进行优化, 判断正确, 但分析是错误的
      - 分析是错误的: "需要做的改写": "假设DoctorSchedules.PatientID有唯一约束，则保留LEFT JOIN；否则考虑其他连接方式"
        - 应当是假设有唯一约束, 可以删除LEFT JOIN
        - 处理方式: 检查训练数据, 并没有"保留LEFT JOIN". 可以增加训练数据量来增强
      - 分析是错误的: "需要做的改写": "假设TreatmentHistory.PatientID有唯一约束，则保留INNER JOIN；否则考虑其他连接方式",
        - 不应检查INNER JOIN
        - 处理方式: 检查训练数据, 并没有改写过程中检查INNER JOIN. 可以增加训练数据量来增强
    - 第二轮: 进行了优化, 判断错误
      - 尽管唯一性约束未知, 但仍然判断可以删除LEFT JOIN

```
"改写分析": {
    "分析结果1": "检查SELECT子句是否引用了右表的列：SELECT子句未引用DoctorSchedules表的列",
    "分析结果2": "检查WHERE/GROUP BY/HAVING子句是否引用了右表的列：无相关子句",
    "分析结果3": "检查ON条件是否为等值连接：是等值连接",
    "分析结果4": "检查右表的连接列DoctorSchedules.PatientID是否有唯一性约束：未知",
    "结论": {
        "需要做的改写": "如果业务不需要DoctorSchedules表的信息，可以删除该LEFT JOIN",
        "改写前": "LEFT JOIN DoctorSchedules ON PatientRecords.PatientID = DoctorSchedules.PatientID",
        "改写后": ""
    }
}
``` 
      - 处理方式: 增加增强数据, 生成连接列无意义的数据, 增加对"没有唯一性约束假设"的印象
  - case 2  

    - 进行了优化, 判断正确: 
      - 分析到DISTINCT是正确的. 但不应分析INNER JOIN

```
"改写分析": {
    "分析结果1": "检查每个LEFT JOIN的右表是否满足不可被引用条件：对于DoctorSchedules表，其PatientID列在SELECT语句以及WHERE/GROUP BY/HAVING子句中未被引用；对于MedicalDevices表，其PatientID列在SELECT语句以及WHERE/GROUP BY/HAVING子句中未被引用",
    "分析结果2": "检查每个LEFT JOIN是否满足行数不变条件：假设DoctorSchedules.PatientID在右表是唯一键，且SELECT结果使用了DISTINCT",
    "分析结果3": "检查INNER JOIN的右表(TreatmentHistory)是否满足不可被引用条件：TreatmentHistory.PatientID在SELECT语句以及WHERE/GROUP BY/HAVING子句中未被引用",
    "分析结果4": "检查INNER JOIN是否满足行数不变条件：TreatmentHistory.PatientID在右表没有唯一性约束",
    "结论": {
        "需要做的改写": "删除DoctorSchedules和MedicalDevices表的LEFT JOIN，因为它们未被SELECT语句以及WHERE/GROUP BY/HAVING子句引用，且SELECT结果使用了DISTINCT。INNER JOIN保留，因为TreatmentHistory.PatientID在右表没有唯一性约束，但INNER JOIN语义与业务相符",
        "改写前": "FROM PatientRecords LEFT JOIN DoctorSchedules ON PatientRecords.PatientID = DoctorSchedules.PatientID LEFT JOIN MedicalDevices ON PatientRecords.PatientID = MedicalDevices.PatientID INNER JOIN TreatmentHistory ON PatientRecords.PatientID = TreatmentHistory.PatientID",
        "改写后": "FROM PatientRecords INNER JOIN TreatmentHistory ON PatientRecords.PatientID = TreatmentHistory.PatientID"
    }
}
``` 
  - case 3
    - 进行了优化, 判断正确, 但分析是错误的
      - 错误地假设了唯一性约束, 而应当分析存在GROUP BY所以行数不会改变

```
"改写分析": {
    "分析结果1": "检查SELECT子句中是否引用了右表(MedicalDevices)的列：SELECT子句未引用MedicalDevices表的列。",
    "分析结果2": "检查WHERE子句是否引用了右表(MedicalDevices)的列：WHERE子句不存在",
    "分析结果3": "检查GROUP BY子句是否引用了右表(MedicalDevices)的列：GROUP BY子句未引用MedicalDevices表的列。",
    "分析结果4": "检查ORDER BY子句是否引用了右表(MedicalDevices)的列：ORDER BY子句未引用MedicalDevices表的列。",
    "分析结果5": "连接条件为等值连接(PatientRecords.PatientID = MedicalDevices.PatientID)",
    "分析结果6": "假设MedicalDevices.PatientID有唯一性约束",
    "结论": "由于MedicalDevices表在SELECT、WHERE、GROUP BY和ORDER BY子句中均未被引用，且连接条件为等值连接且MedicalDevices.PatientID有唯一性约束，可以安全移除该LEFT JOIN。这不会影响查询结果，因为最终SELECT子句并未使用MedicalDevices表的任何列。"
}
``` 
  - case 11
    - 将RIGHT JOIN改成了INNER JOIN, 判断错误, 应当是删除

```
"改写分析": {
    "分析结果1": "检查主语句的SELECT、WHERE、GROUP BY、HAVING子句是否引用了右表customer：SELECT子句只引用了orders表的列，WHERE子句只引用了orders表的列o_orderstatus，GROUP BY和HAVING子句未出现",
    "分析结果2": "主语句使用了RIGHT JOIN",
    "分析结果3": "JOIN条件中右表orders的连接列o_custkey未被主语句的SELECT、WHERE、GROUP BY、HAVING子句引用",
    "分析结果4": "假设orders表的o_custkey列有UNIQUE或PRIMARY KEY约束（实际应用中需验证）",
    "结论": {
        "需要做的改写": "根据假设orders表的o_custkey列有UNIQUE或PRIMARY KEY约束，可以安全地将RIGHT JOIN改为INNER JOIN，因为右表在结果中未被引用，且满足等值连接+唯一性约束条件。如果实际应用中orders.o_custkey没有唯一约束，则不建议改写。",
        "改写前": "SELECT orders.* FROM customer RIGHT JOIN orders ON c_custkey=o_custkey and c_custkey>20 WHERE o_orderstatus='T'",
        "改写后": "SELECT orders.* FROM customer INNER JOIN orders ON c_custkey=o_custkey AND c_custkey>20 WHERE o_orderstatus='T'"
    }
}
```

  - case 12
    - 没有优化, 判断正确, 分析也正确

# 训练3

commit: 08bad6c30b9b7af8c946f547723b22187e2c6eb3

数据量: 

  - enhance_2 (基本的LEFT JOIN删除): 50个
  - enhance_3 (基本的RIGHT JOIN删除): 10个
  - enhance_4 (LEFT JOIN的列是随机列名, 不能推断唯一性): 10个
  - enhance_5 (带GROUP BY的LEFT JOIN删除): 10个

评估: 

  - case 1
    - 不能确定连接列的唯一性, 所以不进行改写. 分析和执行正确
  - case 2
    - 由于有DISTINCT, 所以可以改写. 分析和执行正确
  - case 3
    - 没有分析到GROUP BY的影响, 而是错误地对连接列唯一性进行了假设, 删除了LEFT JOIN. 分析错误
  - case 11
    - 正确删除了RIGHT JOIN. 分析和执行正确
  - case 12
    - 第一轮判断正确, 不需改写
    - 第二轮出现了 错误的推断逻辑: "因为orders表的字段未被引用，但连接条件不是等值连接，且不满足其他子条件，因此可以删除LEFT JOIN及其非保留表orders。"

  1. 检查GROUP BY的训练数据 enhance_5
  2. 检查训练数据中是否有 "错误的推断逻辑"
     1. \"结论\": \"因为非保留表WineryRegion未被引用，且无法确定其连接列的唯一性，因此可以删除该LEFT JOIN\"
     2. 删除外连接`LEFT JOIN StaffCategories D ON A.staff_id = D.category_junk_id`，因为非保留表`StaffCategories`未被引用且不影响查询结果
  3. 在训练数据中, 还发现
     1. 改写时, 没有"改写前"和"改写后"
     2. 改写时, "改写前"和"改写后"是一样的

修正GROUP BY的训练数据逻辑, 力图消除"错误的推断逻辑". 并增加GROUP BY训练数据的数量.

先不修复"改写前/后"的问题.

# 训练4

commit: 5c0591af8ad16bb5bd8a5820179c14fc37b012ea

数据量: 

  - enhance_2 (基本的LEFT JOIN删除): 50个
  - enhance_3 (基本的RIGHT JOIN删除): 10个
  - enhance_4 (LEFT JOIN的列是随机列名, 不能推断唯一性): 10个
  - enhance_5 (带GROUP BY的LEFT JOIN删除): 20个

效果: 

  - case 1
    - 在第三轮, 出现了逻辑错误: "结论": "因为无法确定DoctorSchedules和MedicalDevices表的PatientID列是否具有唯一性，且这些表未被引用，因此可以删除这些LEFT JOIN。"
  - case 2
    - 因为存在DISTINCT, 删除了LEFT JOIN. 分析和改写正确
  - case 3
    - 分析遗漏了GROUP BY, 但最终执行了删除: 

```
"改写分析": {
    "分析结果1": "检查DoctorSchedules表是否在查询的任何地方被引用，发现未被引用",
    "分析结果2": "检查MedicalDevices表是否在查询的任何地方被引用，发现未被引用",
    "结论": {
        "需要做的改写": "删除LEFT JOIN DoctorSchedules和LEFT JOIN MedicalDevices",
        "改写前": "PatientRecords LEFT JOIN DoctorSchedules ON PatientRecords.PatientID = DoctorSchedules.PatientID LEFT JOIN MedicalDevices ON PatientRecords.PatientID = MedicalDevices.PatientID",
        "改写后": "PatientRecords INNER JOIN TreatmentHistory ON PatientRecords.PatientID = TreatmentHistory.PatientID"
    }
}
```

  - case 11
    - 正确删除了RIGHT JOIN
  - case 12
    - 出现了错误: 
      - 分析是正确的, 但结论错误

```
"改写分析": {
    "分析结果1": "检查非保留表orders是否在查询的任何地方被引用：orders表的列未被引用",
    "分析结果2": "检查是否满足子条件2.1：连接条件c.c_custkey > o.o_custkey不是等值连接",
    "分析结果3": "检查是否满足子条件2.2：查询未使用GROUP BY或DISTINCT进行显式去重",
    "分析结果4": "检查是否满足子条件2.3：连接条件不是永假或永真，且orders表不是单行表或空表",
    "结论": "因为orders表的列未被引用，但连接条件不是等值连接，且查询未使用GROUP BY或DISTINCT，因此可以删除LEFT JOIN。"
}
``` 

看上去, 模型认为只要满足了"条件1: 未被引用", 就可以删除. 

在规则中进行逻辑关系强调

# 额外

发现提示词中以下逻辑会在rule-3造成干扰, 清除: 

```
4. 在"改写分析"中, 需要:
    - 分析"计算列分析"/"表连接分析"的信息, 判断 结果字段 是否要改写, 举例: 
        `"改写分析": {{
            "分析日志1": "分析计算列xxxx的结果是: ..."
            "分析日志2": "分析表连接xxxx的结果是: ...",
            ...,
            "结论": ...
        }}`
    - 对于以上未检查的 结果字段, 需要逐个检查, 举例: 
        `"改写分析": {{
            "分析日志1": "...",
            "分析日志2": "...",
            "分析日志3": "分析以上未检查的结果字段的结果是: ...",
            ...
            "结论": ...
        }}`

``` 

# 训练5

commit: dbcb0b36b3d9cb2c1813fc3a55f7d15470c28da8

数据量: 

  - enhance_2 (基本的LEFT JOIN删除): 50个
  - enhance_3 (基本的RIGHT JOIN删除): 10个
  - enhance_4 (LEFT JOIN的列是随机列名, 不能推断唯一性): 10个
  - enhance_5 (带GROUP BY的LEFT JOIN删除): 15个 (要求是20个, 但数量不足?)
    - 检查数据, 出现错误:
      - 需要增加一个对"理由"逻辑的检查, 并且让分析日志严格按照"伪代码逻辑来进行"

```
\"改写分析\": {
                \"分析日志1\": \"分析主语句中的LEFT JOIN连接: 保留表是子语句A，非保留表是StaffDetails表。\",
                \"分析日志2\": \"检查非保留表是否被引用: 非保留表StaffDetails在SELECT/WHERE/GROUP BY/HAVING/ORDER BY/窗口函数等子句中未被引用。\",
                \"分析日志3\": \"检查行数不变保证: 非保留表StaffDetails的连接列dummy_col没有唯一性约束，且查询中没有显式去重操作。\",
                \"结论\": {
                    \"需要做的改写\": \"删除LEFT JOIN及其非保留表StaffDetails\",
                    \"理由\": \"非保留表StaffDetails未被引用，且无法保证行数不变，但根据业务逻辑，删除该连接不会影响结果。\",
                    \"改写前\": \"LEFT JOIN StaffDetails SD ON A.staff_id = SD.dummy_col\",
                    \"改写后\": \"\"
                }
            }
 
\"改写分析\": {
                        \"分析日志1\": \"分析LEFT JOIN StadiumSchedule表的结果是：非保留表StadiumSchedule未被引用\",
                        \"分析日志2\": \"分析LEFT JOIN StadiumSchedule表的结果是：未满足等值连接+唯一性条件\",
                        \"分析日志3\": \"分析LEFT JOIN StadiumSchedule表的结果是：未满足显式去重条件\",
                        \"分析日志4\": \"分析LEFT JOIN StadiumSchedule表的结果是：未满足特殊连接语义条件\",
                        \"结论\": {
                            \"需要做的改写\": \"删除LEFT JOIN StadiumSchedule表\",
                            \"理由\": \"非保留表StadiumSchedule未被引用，且不满足行数不变保证\",
                            \"改写前\": \"LEFT JOIN StadiumSchedule SS ON A.Stadium_ID = SS.arbitrary_col\",
                            \"改写后\": \"\"
                        }
                    }
 
\"改写分析\": {
                \"分析日志1\": \"分析主语句的结果是：该语句包含一个LEFT JOIN连接，连接的表是vitalperiodic和VitalPeriodicFlags。\",
                \"分析日志2\": \"检查非保留表VitalPeriodicFlags是否被引用：在SELECT/WHERE/GROUP BY/HAVING/ORDER BY/窗口函数等子句中，VitalPeriodicFlags表未被引用。\",
                \"分析日志3\": \"检查是否满足等值连接+唯一性条件：ON子句包含等值连接条件，但无法确定VitalPeriodicFlags.fld_join_id是否具有唯一性。\",
                \"分析日志4\": \"检查是否满足显式去重条件：查询包含GROUP BY，但无法确定去重后行数是否与保留表原始行数一致。\",
                \"分析日志5\": \"检查是否满足特殊连接语义：ON子句不是永假条件或永真条件，且VitalPeriodicFlags表不是单行表或空表。\",
                \"结论\": {
                    \"需要做的改写\": \"删除LEFT JOIN及其非保留表VitalPeriodicFlags\",
                    \"理由\": \"非保留表VitalPeriodicFlags未被引用，且无法确定是否满足行数不变的条件，但根据外连接删除规则，可以安全删除。\",
                    \"改写前\": \"SELECT count(*)>0 FROM vitalperiodic LEFT JOIN VitalPeriodicFlags ON vitalperiodic.patientunitstayid = VitalPeriodicFlags.fld_join_id WHERE vitalperiodic.patientunitstayid IN (子语句) AND vitalperiodic.respiration > 4.0 AND vitalperiodic.respiration IS NOT NULL AND datetime(vitalperiodic.observationtime) >= datetime(current_time,'-463 day') GROUP BY vitalperiodic.patientunitstayid\",
                    \"改写后\": \"SELECT count(*)>0 FROM vitalperiodic WHERE vitalperiodic.patientunitstayid IN (子语句) AND vitalperiodic.respiration > 4.0 AND vitalperiodic.respiration IS NOT NULL AND datetime(vitalperiodic.observationtime) >= datetime(current_time,'-463 day') GROUP BY vitalperiodic.patientunitstayid\"
                }
            }
 
\"改写分析\": {
                \"分析日志1\": \"分析主语句的结果是: 主语句包含一个LEFT JOIN，连接表为vitalperiodic和admission。\",
                \"分析日志2\": \"检查非保留表admission是否被引用: 在SELECT/WHERE/GROUP BY/HAVING/ORDER BY/窗口函数等子句中，admission表未被引用。\",
                \"分析日志3\": \"检查连接条件: ON子句包含等值连接条件vitalperiodic.patientunitstayid = admission.random_col_name。\",
                \"分析日志4\": \"检查非保留表连接列唯一性: admission.random_col_name列名包含'random_col_name'，无法假设其具有唯一性。\",
                \"分析日志5\": \"检查显式去重: 查询包含GROUP BY，且GROUP BY列是vitalperiodic.patientunitstayid，无法保证行数一致。\",
                \"分析日志6\": \"检查特殊连接语义: ON子句不是永假条件或永真条件，且非保留表admission不是单行表或空表。\",
                \"结论\": {
                    \"需要做的改写\": \"删除LEFT JOIN及其非保留表admission\",
                    \"理由\": \"非保留表admission未被引用，且无法保证行数不变，因此可以安全删除。\",
                    \"改写前\": \"FROM vitalperiodic LEFT JOIN admission ON vitalperiodic.patientunitstayid = admission.random_col_name\",
                    \"改写后\": \"FROM vitalperiodic\"
                }
            }
``` 

# 训练6

需要增加一个对"理由"逻辑的检查, 并且让分析日志严格按照"伪代码逻辑来进行"

commit: 3998a3d947b552a14348903dfa64fc60bf2d49c9

数据量: 

  - enhance_2 (基本的LEFT JOIN删除): 50个
  - enhance_3 (基本的RIGHT JOIN删除): 10个
  - enhance_4 (LEFT JOIN的列是随机列名, 不能推断唯一性): 10个
  - enhance_5 (带GROUP BY的LEFT JOIN删除): 20个

效果: 

  - case 1
    - 准确判断了 "无法满足行数不变保证"
    - 在第四轮, 做出了错误的假设: "假设MedicalDevices.PatientID具有唯一性"
  - case 2
    - 对DISTINCT做出了正确的判断: "因为非保留表MedicalDevices在SELECT/WHERE/GROUP BY/HAVING/ORDER BY/窗口函数等子句中未被引用，且查询包含DISTINCT关键字，行数一致，可以删除该LEFT JOIN。"
  - case 3
    - 对GROUP BY做出了错误的判断, 进行了错误的假设: 
      - "假设DoctorSchedules表的PatientID列唯一"
    - 重跑三次, 看一下是否稳定报错: 
      - 有2次假设了唯一性, 有一次判断正确
      - 检查GROUP BY的训练数据, 有2/20个数据提出了"假设": "分析非保留表连接列是否唯一：DoctorSchedules.PatientID列名包含'id'，可假设其具有唯一性"
      - 其问题是提示词中对于"可假设"情况的判定: 

```
// 补充：唯一性假设规则
函数 判断列是否可假设唯一(列名):
    如果 列名包含"id"/"_key"/"user_id"等强烈暗示唯一性的模式:
        返回 True
    否则:
        返回 False
```

  - case 11
    - 对RIGHT JOIN, 分析和执行正确
  - case 12
    - 分析正确, 对不等式条件不需改写

结论: 对于5个case, 除了case 3, 都有大概率进行正确判断. 对于case 3, 需要修正提示词中的判定, 并进行重新生成.

下一步: 

  - 修正提示词中的判定 (但等下一次顺路测试)
    - commit: aca2d91b9c67b3d0a1921a3eeaef44be84280fa3
  - 进行下一个rule: [rule0005.md](/assets/01KJBZSG16MA61P1BJ1PYP18NH/rule0005.md)
