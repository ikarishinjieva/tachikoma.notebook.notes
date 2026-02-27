---
title: 20240912 - 对提示词自动调优(ai prompter)的开发评估 - 对挑战用例的分析
confluence_page_id: 3342384
created_at: 2024-09-12T06:00:55+00:00
updated_at: 2024-09-12T09:30:10+00:00
---

提示词: [challenge_prompt.txt](/assets/01KJBZFB847QGFABN1VAMV05DK/challenge_prompt.txt)

生成的用例结果: 

````
#### Sample 1: 
参考资料的内容是:
<参考资料>

</参考资料>

用户和DBA的对话历史:
<对话历史>

DBA团队: ```你正在使用一个较老版本的PostgreSQL，在执行复杂查询时遇到性能问题，这是可以理解的。这里有几个可能的原因和解决方法，或许能帮助你提高性能。

### 1. 更新统计信息
PostgreSQL依赖统计信息进行查询规划。如果统计信息过时，查询性能可能会下降。运行下面的命令更新统计信息：
```sql
ANALYZE;
```

### 2. 检查索引
确保相关列上有适当的索引。你可以使用如下命令创建索引：
```sql
CREATE INDEX idx_your_column ON your_table(your_column);
```

### 3. 查询重写和优化
复杂查询有时可以通过拆分、简化或使用临时表来优化。例如，避免嵌套子查询，可通过JOIN或CTE(Common Table Expressions)来实现更好的性能。

### 4. 调整内存设置
调整PostgreSQL的内存设置，比如`shared_buffers`，`work_mem`等参数来优化数据库性能。比如，可以在`postgresql.conf`文件中修改这些参数，并重启数据库服务。
```ini
shared_buffers = 4GB
work_mem = 64MB
```

希望这些建议能帮你提升查询性能。如果仍有问题，请提供具体的查询示例和表结构，以便进一步分析。```

用户: ```已经尝试了更新统计信息、创建索引和重写查询，但性能依旧没有显著改善，且没有明显硬件瓶颈。```

答案框架如下:
<答案框架>
1. 问题中的关键现象和关键信息
   - PostgreSQL版本较旧
   - 复杂查询性能问题
   - 已尝试更新统计信息、创建索引和重写查询，但效果不显著
   - 没有明显硬件瓶颈

2. 对已有信息进一步分析和补充
   - 确认统计信息已更新
   - 确认创建的索引合适
   - 查询逻辑和复杂程度确认
   - 硬件资源使用情况确认
   - PostgreSQL配置参数当前设置确认

3. 进一步排查可能原因
   - PostgreSQL查询规划器问题
   - 系统I/O瓶颈
   - 复杂查询涉及的数据量大
   - 配置参数不够优化

4. 行动建议
   - 确认查询是否可以进一步优化
   - 使用`EXPLAIN ANALYZE`查看查询执行计划
   - 检查系统I/O性能，使用`iostat`，`vmstat`等工具
   - 调整PostgreSQL配置参数并监控效果

5. 阶段性结论
   - 如果所有以上方法均无效，考虑升级PostgreSQL到最新版本以获得最新的性能优化和Bug修复
</答案框架>

用户本轮输入如下:
```
查询执行计划结果显示，主要耗时在一个特定的Join操作上。索引和统计信息已经确认更新，查询代码如下：
SELECT a.*, b.* FROM table_a a JOIN table_b b ON a.id = b.a_id WHERE a.date > '2022-01-01' AND b.status = 'active';
```
</用户本轮输入>
#### Sample 1: 当前你的查询性能问题可能和特定的JOIN操作相关。查询计划结果显示主要耗时在JOIN上，索引和统计信息已经更新，但效果不明显。这表明可能存在以下几个原因：

首先，检查是否为JOIN操作创建了合适的索引，确保在`table_a`的`id`列和`table_b`的`a_id`列上都有索引。如果没有，建议添加以提高JOIN性能：
```sql
CREATE INDEX idx_table_a_id ON table_a(id);
CREATE INDEX idx_table_b_a_id ON table_b(a_id);
```

其次，考虑是否可以简化查询或重写为更高效的查询方式。对于复杂的JOIN，可以尝试使用临时表或分步骤执行来减少单次操作的负担。例如：
```sql
CREATE TEMP TABLE temp_b AS
SELECT * FROM table_b WHERE status = 'active';
SELECT a.*, b.* FROM table_a a JOIN temp_b b ON a.id = b.a_id WHERE a.date > '2022-01-01';
```

再者，检查内存和I/O配置，确保PostgreSQL配置参数已优化，比如`work_mem`。该参数决定了单次操作的内存使用量，可以尝试增加：
```ini
work_mem = 128MB
```

最后，如果这些方法仍未解决，考虑检查硬件I/O性能，确认磁盘性能是否是瓶颈。使用`iostat -x`查看详细的磁盘I/O统计信息，确认磁盘负载是否过高。

如果以上措施均无效，建议分析是否在升级到新版本PostgreSQL，可以通过更新获得更好的性能优化和Bug修复。希望这些建议能助你提高查询性能，有任何问题随时联系！<eos>
############
```` 

# 分析提示词发现的问题

提示词的结构是: 

```
给出任务描述
 
给出错误的样例+预测+GT
 
给出模型产生的仿样例
 
给出新的提示词
 
要求生成新的挑战用例
``` 

问题: 

  1. "给出模型产生的仿样例" 这一步的作用不明
  2. "给出错误的样例+预测+GT" 这一步, 转换到当前场景上 (没有GT, 只有ranking), 模型好像不能理解这一步

通过对比实验来对以上两个问题进行评估, 使用的评分prompt如附件: [ranking_prompt.txt](/assets/01KJBZFB847QGFABN1VAMV05DK/ranking_prompt.txt)

模型: gemini-pro-1.5-128k

举例 (对张乾生成的挑战用例进行评测): 

  1. 将 任务要求 和 挑战用例 拼接, 进行挑战用例的预测: [predict.prompt](/assets/01KJBZFB847QGFABN1VAMV05DK/predict.prompt)
  2. 将 预测输入 和 预测结果 拼接, 进行评分: [ranking_prompt_2.txt](/assets/01KJBZFB847QGFABN1VAMV05DK/ranking_prompt_2.txt)
  3. 得到评分

张乾生成的挑战用例评分 (作为基准): 

```
评估的记录: 
1. 操作1: 未减分. 
2. 操作2: 未减分. 
3. 操作3: 未减分. 
4. 操作4: 未减分. 
5. 操作5: 未减分. 
6. 操作6: 未减分. 
7. 操作7: 未减分. 

一共评估了7个操作, 其中0个操作进行了减分.
```
