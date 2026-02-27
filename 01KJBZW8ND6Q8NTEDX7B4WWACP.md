---
title: 20250801 - 进行SQL优化的GRPO - 缩小训练效果和校验效果的差异
confluence_page_id: 4161808
created_at: 2025-08-01T07:17:39+00:00
updated_at: 2025-08-13T14:17:55+00:00
---

# 之前

[20250727 - 进行SQL优化的GRPO - 对completion进行分析]

TODO:

  - warmup阶段会导致reward大幅下降, 考虑将warmup阶段拉长, 或去除
  - 在训练数据中: 
    - 计算列分析/表连接分析/列引用分析 的出现次数为: 65/64/210
    - 但在评估中, 绝大部分都是 列引用分析, 其他两种步骤出现的很少
  - 目前解决了训练数据拟合的问题, 需要考虑如何解决训练数据效果好, 但评估数据很差的gap

# 尝试1 - 使用Qwen3-8B-base

  - commit: 2c831028ebcca3129b3f924affeea02c73ff2ecd
  - 修复了训练数据的错误
  - 尝试使用Qwen3-8B-base进行训练尝试

训练效果: 

  - 从completion中, 刚开始是乱码, 然后会有正常输出, LR继续升高后, 大部分是乱码超长导致clipped. 认为可能能调小LR来增强效果

![image2025-8-1 21:15:32.png](/assets/01KJBZW8ND6Q8NTEDX7B4WWACP/image2025-8-1%2021%3A15%3A32.png)

# 尝试2 - 加强对生成步骤的约束

重新设计脚本: /data/huangyan/sql-ai-2/make_train_dataset/analyze_sql.v6.py

将所有步骤都进行字样固定:

````
动作1:
- 动作字样: "从外往内, 逐个语句进行检查: 检查顺序: {{...}}"

动作2:
- 动作字样: "检查语句({{语句名}})的各查询字段: 检查项: {{...}}"
- 动作说明: 必须作为动作1的子动作

动作3:
- 动作字样: 
  - "检查语句({{语句名}})的查询字段({{字段内容}}): 被其他语句({{语句名列表}})作为({{选择字段/表连接列/计算列/...}})引用: {{具体说明}}"
  - "检查语句({{语句名}})的查询字段({{字段内容}}): 没有被其他语句({{语句名列表}})引用, 可以删除"
  - "检查语句({{语句名}})的查询字段({{字段内容}}): {{检查结果}}, 可以删除"
- 动作说明: 必须作为动作2的子动作

动作4:
- 动作字样: "优化后的SQL: \\n```sql\\n{{SQL}}\\n```"

动作5:
- 动作字样: "由于SQL发生了优化, 需要检查优化是否超出了优化规则, 进行了额外的优化"
- 动作说明: 在SQL优化后, 都需要检查: 不应进行额外的优化

动作6:
- 动作字样: "由于SQL发生了优化, 需要重新进行检查"
- 动作说明: 在SQL优化后, 都需要进行重新检查, 是否能继续根据优化规则进行优化

动作7:
- 动作字样: "检查结论: {{具体结论}}"
```` 

调整奖励函数, 使用三个奖励函数, 综合三方面: 

```
重要级从高到低：
    1. 必须存在action1和action4，在最后一个action4后，最好存在action5和action6
    2. 提取action3中的语句+查询字段，去重，将predict和GT中的匹配进行对比
    3. 统计各个动作的数量(根据关键信息去重)，所有动作的总和，在predict中越多越好

``` 

训练数据中, 去除了WITH语句 (缩减训练集, 进行尝试).

  - 训练数据量: 21条. 
  - 训练commit: 13890044a76b5751ec644203d3bdcd4afd26cc9d
  - 训练效果: (因为OOM, epoch没跑完)
    - ![image2025-8-2 22:46:1.png](/assets/01KJBZW8ND6Q8NTEDX7B4WWACP/image2025-8-2%2022%3A46%3A1.png)
    - ![image2025-8-2 22:47:17.png](/assets/01KJBZW8ND6Q8NTEDX7B4WWACP/image2025-8-2%2022%3A47%3A17.png)
    - 看起来: 
      - 训练的reward仍有上升趋势, 主要的上升空间只剩Action3F1, 对于其他两个奖励已经都接近0.2最高值
      - eval的clip比较高, 认为是长度限制导致, Action3F1和ActionCount缺的比较多, 而Completeness已经接近0.2最高值

下一步: 调整参数, 降低OOM. 另外尝试一下其他基模型

# 尝试3 - 调整参数, 降低OOM, 完成一次epoch=12的训练

调整参数: 

```
    --per_device_train_batch_size 2 \
    --per_device_eval_batch_size 1 \
    --gradient_accumulation_steps 2 \
    --vllm_gpu_memory_utilization 0.85 \
--move_model_batches 10 \
``` 

训练效果: 

![image2025-8-3 13:31:38.png](/assets/01KJBZW8ND6Q8NTEDX7B4WWACP/image2025-8-3%2013%3A31%3A38.png)

![image2025-8-3 13:34:19.png](/assets/01KJBZW8ND6Q8NTEDX7B4WWACP/image2025-8-3%2013%3A34%3A19.png)

分析: 

  - 训练数据, ActionCount和Completeness基本到顶, Action3F1只到一半
    - 需要分析completion
  - 评估数据, clip比例越来越高, ActionCount只能到-0.01, 也就是说生成长度不够, 没法拉升ActionCount
    - 需要放大生成长度
    - 分析completion: 评估数据出现了之前的问题: 
      - 语句引用不一致
        - ![image2025-8-3 14:27:25.png](/assets/01KJBZW8ND6Q8NTEDX7B4WWACP/image2025-8-3%2014%3A27%3A25.png)
      - 列引用不一致
        - \- 动作3: 检查语句(子查询v1-1.1.1.1)的查询字段(bingoggr): 没有被其他语句(主FROM子语句-1.1.1)引用, 可以删除
        - \- 动作3: 检查语句("子查询v1-1.1.1.1")的查询字段("sum(bingoggr) as bingoggr"): 没有被其他语句("主FROM子语句-1.1")引用, 可以删除
      - 需要修复数据问题

# 尝试4 - 增大生成长度, 更换基模型

增大生成长度: 12000, 更换基模型 Shanghai_AI_Laboratory/internlm3-8b-instruct, 跑了 8/12个 epoch, 进行训练: 

![image2025-8-3 18:30:16.png](/assets/01KJBZW8ND6Q8NTEDX7B4WWACP/image2025-8-3%2018%3A30%3A16.png)

![image2025-8-3 18:30:58.png](/assets/01KJBZW8ND6Q8NTEDX7B4WWACP/image2025-8-3%2018%3A30%3A58.png)

效果: 

  - train学习的曲线比较慢, ActionCount和Completeness学习都没有到顶, Action3F1的学习刚刚开始
  - eval的曲线都增长较慢

可以尝试调大LR=1.e-4, epoch=20, 效果: 

![image2025-8-4 10:25:2.png](/assets/01KJBZW8ND6Q8NTEDX7B4WWACP/image2025-8-4%2010%3A25%3A2.png)

![image2025-8-4 10:25:39.png](/assets/01KJBZW8ND6Q8NTEDX7B4WWACP/image2025-8-4%2010%3A25%3A39.png)

  - 训练中, ActionCount/Completeness 都没有到顶, Action3F1比较低, 不如之前LR=5e-5时的epoch=8

尝试模型: ZhipuAI/GLM-4-9B-0414/

遇到报错: 

```
size mismatch for base_model.model.lm_head.base_layer.weight: copying a param with shape torch.Size([151552, 4096]) from checkpoint, the shape in current model is torch.Size([0])
``` 

GLM不是标准的Seq2SeqLM、CausalLM, peft支持可能没有完善, 得用专门的微调工具? 暂不考虑.

# 尝试5 - 使用Qwen3-14B

LR=1e-4, 数据量: 21

![image2025-8-4 16:22:2.png](/assets/01KJBZW8ND6Q8NTEDX7B4WWACP/image2025-8-4%2016%3A22%3A2.png)

![image2025-8-4 16:23:8.png](/assets/01KJBZW8ND6Q8NTEDX7B4WWACP/image2025-8-4%2016%3A23%3A8.png)

跑到epoch=9时, 有一个GPU OOM

从训练和评估的趋势上, reward的上升 都比 8B 模型更快

# 尝试6 - 使用孙健构建的SQL + GSPO

GSPO: <https://swift.readthedocs.io/zh-cn/latest/Instruction/GRPO/AdvancedResearch/GSPO.html>

生成144个训练数据 (并未跑完所有SQL). 使用其中22条数据, 加上GSPO参数, Qwen-3-14B

加上GSPO后, 很容易OOM, 换回Qwen-3-8B, + GSPO, 增大最大长度为16000, 如果有效果, 再进行消融实验

![image2025-8-5 11:2:2.png](/assets/01KJBZW8ND6Q8NTEDX7B4WWACP/image2025-8-5%2011%3A2%3A2.png)

![image2025-8-5 11:2:31.png](/assets/01KJBZW8ND6Q8NTEDX7B4WWACP/image2025-8-5%2011%3A2%3A31.png)

效果: 

  - train过程, ActionCount和Completeness可以达到最高值, 但Action3F1只能到0.7附近
  - eval过程, Action3F1只能到0.3
  - 随着最大长度调整为16000, clipped_ratio得到控制
  - 查看completions分析
    - GT中, reward的匹配条件, 没匹配到 "可以删除。"的句号
    - GT中的语句名"账单聚合子查询-1.7", 在模型推理中语句名始终为"子语句-1.7"
      - 这个现象在特定的几个case上出现, 其他case上不是恒定出现
      - 检查了训练数据, 出现"子语句-1.2"类似字样的case有两个, 认为该现象不是因为训练数据偏斜引起
      - 考虑增加训练数据

去除GSPO参数, 进行消融实验, 短线为 去除GSPO参数后的曲线

![image2025-8-5 13:37:4.png](/assets/01KJBZW8ND6Q8NTEDX7B4WWACP/image2025-8-5%2013%3A37%3A4.png)

![image2025-8-5 13:37:47.png](/assets/01KJBZW8ND6Q8NTEDX7B4WWACP/image2025-8-5%2013%3A37%3A47.png)

可以看到: 

  - GSPO让reward发展更好
  - GSPO的显存占用, 多4-8GB

# 尝试6 - 使用孙健构建的SQL + GSPO + 增加训练数据

增加训练数据到40条, Qwen-3-8B, + GSPO, 增大最大长度为16000

修复上一轮发现的reward的字符串匹配问题

![image2025-8-5 22:23:58.png](/assets/01KJBZW8ND6Q8NTEDX7B4WWACP/image2025-8-5%2022%3A23%3A58.png)

![image2025-8-5 22:24:41.png](/assets/01KJBZW8ND6Q8NTEDX7B4WWACP/image2025-8-5%2022%3A24%3A41.png)

  - train/reward有明显提升, eval/reward提升不大

发现问题如下: 

  - 观察eval/reward的completion过程
    - 在step=100左右, 正确结果和通配符一起出现:

```
      - 动作3: 检查语句("主FROM子语句-1.1.1")的查询字段("v1.months"): 被其他语句("主FROM子语句-1.1")作为(表连接列)引用: {具体说明}
      - 动作3: 检查语句("主FROM子语句-1.1.1")的查询字段("v1.bingoggr"): 没有被其他语句引用, 可以删除
```

      - 导致step > 100以后, 模型只输出通配符, 而不做其他判断: 

```
      - 动作3: 检查语句("主语句-1")的查询字段("v1.months"): 被其他语句("主FROM子语句-1.1")作为("表连接列")引用: {具体说明}
      - 动作3: 检查语句("主语句-1")的查询字段("v1.bingoggr"): 被其他语句("主FROM子语句-1.1")作为("选择字段")引用: {具体说明}
      - 动作3: 检查语句("主语句-1")的查询字段("v1.valid_account"): 被其他语句("主FROM子语句-1.1")作为("选择字段")引用: {具体说明}
      - 动作3: 检查语句("主语句-1")的查询字段("t1.fd_ggr"): 被其他语句("主FROM子语句-1.1")作为("选择字段")引用: {具体说明}
```

      - 考虑修改reward, 对通配符进行惩罚

校验数据中的不一致, 引起了校验数据reward不高: 

![image2025-8-5 18:40:6.png](/assets/01KJBZW8ND6Q8NTEDX7B4WWACP/image2025-8-5%2018%3A40%3A6.png)

结构分析中, 语句示例并没有引用子语句的名称, 会引起字段归属的混乱: 

![image2025-8-5 18:56:31.png](/assets/01KJBZW8ND6Q8NTEDX7B4WWACP/image2025-8-5%2018%3A56%3A31.png)

  - 训练过程中, reward下降的数据 跟此相关. 随着训练, 对字段检查越来越严格, 会导致误判 (子语句的内容被当做父语句的内容)
  - 检查训练数据
    - 有部分数据是没有引用子语句的名称, 需要增加检查
    - 有部分数据使用了"..."
    - 大部分数据正确引用了子语句的名称

# 尝试7

修复尝试6中发现的问题: 

  - 修复reward的字符串匹配问题. 
  - 增加reward函数, 对通配符 ("{具体说明}"等) 进行惩罚
  - 在生成SQL结构时, 要求在SQL示意中, 替换子语句的内容为子语句的名称. 将生成结构的大模型换成gemini

重新生成训练数据, 使用40条数据, 效果: 

![image2025-8-6 10:5:9.png](/assets/01KJBZW8ND6Q8NTEDX7B4WWACP/image2025-8-6%2010%3A5%3A9.png)

![image2025-8-6 10:5:40.png](/assets/01KJBZW8ND6Q8NTEDX7B4WWACP/image2025-8-6%2010%3A5%3A40.png)

从clip/kl/grad看, 数据比较正常. 对reward检查: 

  - train阶段: 
    - Completeness的惩罚过高
    - Action3F1的上升到0.8左右, 就没有继续上升
  - eval阶段: 
    - 跟之前的下降不同, 这次在0.15附近反复震荡

分析completion:

  - 问题1: 字段匹配的大小写问题, 举例: 
    - 主要问题是, 在SQL结构分析时, 模型修改了原SQL的关键词大小写

```
GT: 动作3: 检查语句(FROM子查询-h1-1.2)的查询字段(count(*) as total_haoyou): 没有被其他语句({'主语句-1'})引用, 可以删除
推理: - 动作3: 检查语句("FROM子查询-h1-1.2")的查询字段("COUNT(*) AS total_haoyou"): 没有被其他语句("主语句-1`)引用, 可以删除
``` 
    - 需要修改reward函数的匹配规则, 忽略大小写和空格
  - 问题2: "字段内容"的不准确, 出现了'count(distinct fenlei_id) as total_fenlei' 写成了 'COUNT(DISTINCT fenlei_id)'. 需要一个更明确的标准.
  - 问题3: 有GT的没有action3, 这个不对, 增加了抛出异常
  - CompletenessReward惩罚过高, 调低

# 尝试8

修复尝试7碰到的问题, 训练效果: 

![image2025-8-6 16:26:42.png](/assets/01KJBZW8ND6Q8NTEDX7B4WWACP/image2025-8-6%2016%3A26%3A42.png)

![image2025-8-6 16:28:8.png](/assets/01KJBZW8ND6Q8NTEDX7B4WWACP/image2025-8-6%2016%3A28%3A8.png)

  - train/reward仍有上升趋势, 等待训练结束
  - eval/reward在step=80后坍塌, 并且clipped_ratio保障, 但eval/kl变化不大
    - step=80时, train/ActionCount奖励有一个提升, 生成了更多的动作数量
      - 检查ActionCount reward的逻辑: 现在的奖励措施是 最多奖励GT动作数量的两倍 (对应奖励值是0.2), 而不是测算偏离量.
        - 如果生成GT动作数量的2倍, 追逐这个奖励的代价太高, 而且没有意义. 
        - 需要修改成: 动作应在GT动作数量附近, 不偏离太大
    - 查看eval的completion
      - 对于"可以删除"的匹配, 需要匹配"可以删除)"这种情况
      - 对于"其他语句"有误解, 应是"同级或父级语句", 举例: 

```
- 动作3: 检查语句(主语句-1)的查询字段(v1.months): 被其他语句(子查询v1-1.1.1, 子查询t1-1.1.2, 子查询v2-1.1.3, 子查询v3-1.1.4)作为表连接列引用: (v1.months用于连接v1与t1、v2、v3)
```

# 尝试9

修复尝试8的问题:

  - 修改ActionCount, 改成: 动作应在GT动作数量附近, 偏离允许20%, 超过则扣分, 扣分呈指数性
  - 手工替换训练数据中的"其他语句"为"同级或父级语句"
  - 修复reward中, 使其可以匹配: "可以删除)"

训练效果: 

![image2025-8-6 23:6:24.png](/assets/01KJBZW8ND6Q8NTEDX7B4WWACP/image2025-8-6%2023%3A6%3A24.png)

![image2025-8-6 23:7:17.png](/assets/01KJBZW8ND6Q8NTEDX7B4WWACP/image2025-8-6%2023%3A7%3A17.png)

  - train/reward和eval/reward都达到新高
  - eval/Action3F1的低谷, 随着ActionCount上升而上升, 达到新高 (train并没有在这个阶段有低谷, 应该是train过程中学到了某个特征, 让eval数据的ActionCount上升)
  - clipped_ratio健康, kl健康
  - 检查completion:
    - 对校验数据, 比较好的结果: 
      - 其中"(引用)"部分是有点怪的

```
- 动作1: 从外往内, 逐个语句进行检查: (主语句-1, 主FROM子语句-1.1, 子查询v1-1.1.1, 子查询t1-1.1.2, WHERE子查询-1.1.2.1, 子查询v2-1.1.3, UNION子查询a-1.1.3.1, 子查询v3-1.1.4)
  - 动作2: 检查语句(主语句-1)的各查询字段: (A44_T_1_.fd_roi)
    - 动作3: 检查语句(主语句-1)的查询字段(A44_T_1_.fd_roi): 被同级或父级语句(主FROM子语句-1.1)作为(选择字段)引用: (引用)
  - 动作2: 检查语句(主FROM子语句-1.1)的各查询字段: (v1.months, v1.bingoggr, v1.valid_account, t1.fd_ggr, v3.fd, fd_roi, v2.cost)
    - 动作3: 检查语句(主FROM子语句-1.1)的查询字段(v1.months): 被同级或父级语句(主语句-1)作为(选择字段)引用: (引用)
    - 动作3: 检查语句(主FROM子语句-1.1)的查询字段(v1.bingoggr): 没有被同级或父级语句(主语句-1)引用, 可以删除
    - 动作3: 检查语句(主FROM子语句-1.1)的查询字段(v1.valid_account): 没有被同级或父级语句(主语句-1)引用, 可以删除
    - 动作3: 检查语句(主FROM子语句-1.1)的查询字段(t1.fd_ggr): 被同级或父级语句(主FROM子语句-1.1)作为(计算列)引用: (引用)
    - 动作3: 检查语句(主FROM子语句-1.1)的查询字段(v3.fd): 被同级或父级语句(主FROM子语句-1.1)作为(计算列)引用: (引用)
    - 动作3: 检查语句(主FROM子语句-1.1)的查询字段(fd_roi): 被同级或父级语句(主语句-1)作为(选择字段)引用: (引用)
    - 动作3: 检查语句(主FROM子语句-1.1)的查询字段(v2.cost): 没有被同级或父级语句(主语句-1)引用, 可以删除
  - 动作2: 检查语句(子查询v1-1.1.1)的各查询字段: (bingoggr, valid_account, months)
    - 动作3: 检查语句(子查询v1-1.1.1)的查询字段(bingoggr): 没有被同级或父级语句(主FROM子语句-1.1)引用, 可以删除
    - 动作3: 检查语句(子查询v1-1.1.1)的查询字段(valid_account): 没有被同级或父级语句(主FROM子语句-1.1)引用, 可以删除
    - 动作3: 检查语句(子查询v1-1.1.1)的查询字段(months): 被同级或父级语句(主FROM子语句-1.1)作为(连接列)引用: (引用)
  - 动作2: 检查语句(子查询t1-1.1.2)的各查询字段: (fd_ggr, months)
    - 动作3: 检查语句(子查询t1-1.1.2)的查询字段(fd_ggr): 被同级或父级语句(主FROM子语句-1.1)作为(计算列)引用: (引用)
    - 动作3: 检查语句(子查询t1-1.1.2)的查询字段(months): 被同级或父级语句(主FROM子语句-1.1)作为(连接列)引用: (引用)
  - 动作2: 检查语句(WHERE子查询-1.1.2.1)的各查询字段: (login_name)
    - 动作3: 检查语句(WHERE子查询-1.1.2.1)的查询字段(login_name): 被同级或父级语句(子查询t1-1.1.2)作为(选择字段)引用: (引用)
  - 动作2: 检查语句(子查询v2-1.1.3)的各查询字段: (cost, state_date)
    - 动作3: 检查语句(子查询v2-1.1.3)的查询字段(cost): 没有被同级或父级语句(主FROM子语句-1.1)引用, 可以删除
    - 动作3: 检查语句(子查询v2-1.1.3)的查询字段(state_date): 被同级或父级语句(主FROM子语句-1.1)作为(连接列)引用: (引用)
  - 动作2: 检查语句(UNION子查询a-1.1.3.1)的各查询字段: (cost, state_date)
    - 动作3: 检查语句(UNION子查询a-1.1.3.1)的查询字段(cost): 被同级或父级语句(子查询v2-1.1.3)作为(选择字段)引用: (引用)
    - 动作3: 检查语句(UNION子查询a-1.1.3.1)的查询字段(state_date): 被同级或父级语句(子查询v2-1.1.3)作为(连接列)引用: (引用)
  - 动作2: 检查语句(子查询v3-1.1.4)的各查询字段: (months, fd)
    - 动作3: 检查语句(子查询v3-1.1.4)的查询字段(months): 被同级或父级语句(主FROM子语句-1.1)作为(连接列)引用: (引用)
    - 动作3: 检查语句(子查询v3-1.1.4)的查询字段(fd): 被同级或父级语句(主FROM子语句-1.1)作为(计算列)引用: (引用)
``` 
    - 比较差的结果
      - 其中没有使用列的别名, 而是使用了列的表达式: 

```
- 动作1: 从外往内, 逐个语句进行检查: (主语句-1, 主FROM子语句-1.1, 子查询v1-1.1.1, 子查询t1-1.1.2, WHERE子查询-1.1.2.1, 子查询v2-1.1.3, UNION子查询a-1.1.3.1, 子查询v3-1.1.4)
  - 动作2: 检查语句(主语句-1)的各查询字段: (A44_T_1_."fd_roi")
    - 动作3: 检查语句(主语句-1)的查询字段(A44_T_1_."fd_roi"): 被同级或父级语句(主FROM子语句-1.1)作为(选择字段)引用: (引用)
  - 动作2: 检查语句(主FROM子语句-1.1)的各查询字段: (v1.months, v1.bingoggr, v1.valid_account, t1.fd_ggr, v3.fd, t1.fd_ggr/v3.fd, v2.cost)
    - 动作3: 检查语句(主FROM子语句-1.1)的查询字段(v1.months): 被同级或父级语句(主语句-1)作为(选择字段)引用: (引用)
    - 动作3: 检查语句(主FROM子语句-1.1)的查询字段(v1.bingoggr): 没有被同级或父级语句(主语句-1)引用, 可以删除
    - 动作3: 检查语句(主FROM子语句-1.1)的查询字段(v1.valid_account): 没有被同级或父级语句(主语句-1)引用, 可以删除
    - 动作3: 检查语句(主FROM子语句-1.1)的查询字段(t1.fd_ggr): 被同级或父级语句(主语句-1)作为(选择字段)引用: (引用)
    - 动作3: 检查语句(主FROM子语句-1.1)的查询字段(v3.fd): 被同级或父级语句(主语句-1)作为(选择字段)引用: (引用)
    - 动作3: 检查语句(主FROM子语句-1.1)的查询字段(t1.fd_ggr/v3.fd): 被同级或父级语句(主语句-1)作为(选择字段)引用: (引用)
    - 动作3: 检查语句(主FROM子语句-1.1)的查询字段(v2.cost): 被同级或父级语句(主语句-1)作为(选择字段)引用: (引用)
  - 动作2: 检查语句(子查询v1-1.1.1)的各查询字段: (SUM(bingoggr), SUM(valid_account), SUBSTRING(pt,1,6))
    - 动作3: 检查语句(子查询v1-1.1.1)的查询字段(SUM(bingoggr)): 没有被同级或父级语句(主FROM子语句-1.1)引用, 可以删除
    - 动作3: 检查语句(子查询v1-1.1.1)的查询字段(SUM(valid_account)): 没有被同级或父级语句(主FROM子语句-1.1)引用, 可以删除
    - 动作3: 检查语句(子查询v1-1.1.1)的查询字段(SUBSTRING(pt,1,6)): 被同级或父级语句(主FROM子语句-1.1)作为(选择字段/连接列)引用: (引用)
  - 动作2: 检查语句(子查询t1-1.1.2)的各查询字段: (SUM(bingoggr), SUBSTRING(pt,1,6))
    - 动作3: 检查语句(子查询t1-1.1.2)的查询字段(SUM(bingoggr)): 被同级或父级语句(主FROM子语句-1.1)作为(选择字段)引用: (引用)
    - 动作3: 检查语句(子查询t1-1.1.2)的查询字段(SUBSTRING(pt,1,6)): 被同级或父级语句(主FROM子语句-1.1)作为(连接列)引用: (引用)
  - 动作2: 检查语句(WHERE子查询-1.1.2.1)的各查询字段: (login_name)
    - 动作3: 检查语句(WHERE子查询-1.1.2.1)的查询字段(login_name): 被同级或父级语句(子查询t1-1.1.2)作为(选择字段/连接列)引用: (引用)
  - 动作2: 检查语句(子查询v2-1.1.3)的各查询字段: (SUM(a.cost), a.state_date)
    - 动作3: 检查语句(子查询v2-1.1.3)的查询字段(SUM(a.cost)): 被同级或父级语句(主FROM子语句-1.1)作为(选择字段)引用: (引用)
    - 动作3: 检查语句(子查询v2-1.1.3)的查询字段(a.state_date): 被同级或父级语句(主FROM子语句-1.1)作为(连接列)引用: (引用)
  - 动作2: 检查语句(UNION子查询a-1.1.3.1)的各查询字段: (SUM(cost), state_date)
    - 动作3: 检查语句(UNION子查询a-1.1.3.1)的查询字段(SUM(cost)): 被同级或父级语句(子查询v2-1.1.3)作为(选择字段)引用: (引用)
    - 动作3: 检查语句(UNION子查询a-1.1.3.1)的查询字段(state_date): 被同级或父级语句(子查询v2-1.1.3)作为(连接列)引用: (引用)
  - 动作2: 检查语句(子查询v3-1.1.4)的各查询字段: (SUBSTRING(TO_CHAR(first_deposit_date::timestamp, 'YYYYMMDD'),1,6 ), COUNT(1))
    - 动作3: 检查语句(子查询v3-1.1.4)的查询字段(SUBSTRING(TO_CHAR(first_deposit_date::timestamp, 'YYYYMMDD'),1,6 )): 被同级或父级语句(主FROM子语句-1.1)作为(连接列)引用: (引用)
    - 动作3: 检查语句(子查询v3-1.1.4)的查询字段(COUNT(1)): 没有被同级或父级语句(主FROM子语句-1.1)引用, 可以删除
```

  - 总共720个步骤, 当前运行到280步, 已运行5h 10m 26s, 还剩8h 7m 50s

# 尝试10

尝试vllm_mode=server, commit: 8a1410d7971214480f8947ff5f3c732ff9d89921

使用2个GPU作为vllm server, 6个GPU进行训练.

注意: num_generation从8调整到了6

![image2025-8-7 10:1:57.png](/assets/01KJBZW8ND6Q8NTEDX7B4WWACP/image2025-8-7%2010%3A1%3A57.png)

![image2025-8-7 10:2:56.png](/assets/01KJBZW8ND6Q8NTEDX7B4WWACP/image2025-8-7%2010%3A2%3A56.png)

效果不如尝试9, 认为是num_generation变少, 导致对Action3F1的探索方向有限, train和eval的Action3F1奖励都比之前少

总体运行时间: "global_step/max_steps": "440/720", "percentage": "61.11%", "elapsed_time": "7h 42m 21s", "remaining_time": "4h 54m 13s"

总体运行时间与尝试9差不多

训练用的GPU mem下降很多: 

![image2025-8-7 10:9:15.png](/assets/01KJBZW8ND6Q8NTEDX7B4WWACP/image2025-8-7%2010%3A9%3A15.png)

再尝试用4个GPU作为vllm server:

```
#rollout参数
    --vllm_tensor_parallel_size 1 \
    --vllm_data_parallel_size 4 \
 
#vllm参数
--num_generations 8 \
--per_device_train_batch_size 1 \
--per_device_eval_batch_size 2 \
--gradient_accumulation_steps 4 \
``` 

在GPU-0上OOM, 增加 --deepspeed zero2, 内存正常

整体时间: 'elapsed_time': '16m 59s', 'remaining_time': '21h 14m 6s'

重复试验, 整体时间: "global_step/max_steps": "35/1520", "percentage": "2.30%", "elapsed_time": "33m 20s", "remaining_time": "23h 34m 33s"

日志: [kv_cache_utils.py:837] Maximum concurrency for 16,000 tokens per request: 12.16x

注意: 因为用于训练的GPU数量变化, step数量也有变化, 工作量增加了一倍

调整参数: 

```
#rollout参数
    --vllm_tensor_parallel_size 1 \
    --vllm_data_parallel_size 4 \

#vllm参数
--num_generations 8 \
--per_device_train_batch_size 4 \
--per_device_eval_batch_size 2 \
--gradient_accumulation_steps 1 \
--deepspeed zero2
``` 

OOM

调整参数: 

```
#rollout参数
    --vllm_tensor_parallel_size 1 \
    --vllm_data_parallel_size 4 \

#vllm参数
--num_generations 8 \
--per_device_train_batch_size 2 \
--per_device_eval_batch_size 2 \
--gradient_accumulation_steps 2 \
--deepspeed zero2
``` 

时间预估: "elapsed_time": "55m 42s", "remaining_time": "1d 6h 26m 6s". 时间变得更长

调整rollout参数: 从1/4调整到2/2

```
#rollout参数
    --vllm_tensor_parallel_size 2 \
    --vllm_data_parallel_size 2 \

 
#vllm参数
--num_generations 8 \
--per_device_train_batch_size 1 \
--per_device_eval_batch_size 2 \
--gradient_accumulation_steps 4 \
--deepspeed zero2
 
``` 

时间预估: "global_step/max_steps": "20/1520", "percentage": "1.32%", "elapsed_time": "11m 28s", "remaining_time": "14h 20m 15s"

日志: [kv_cache_utils.py:837] Maximum concurrency for 16,000 tokens per request: 31.25x

增加flash-infer, 需要安装0.2.9版本 (不能安装0.2.10, 因为vllm的版本判断逻辑有错: <https://github.com/vllm-project/vllm/issues/22297>)

时间预估: 'global_step/max_steps': '20/1520', 'percentage': '1.32%', 'elapsed_time': '9m 55s', 'remaining_time': '12h 23m 52s'

没有很大区别

调整rollout参数: 调整到4/1

```
#rollout参数
    --vllm_tensor_parallel_size 4 \
    --vllm_data_parallel_size 1 \

 
#vllm参数
--num_generations 8 \
--per_device_train_batch_size 1 \
--per_device_eval_batch_size 2 \
--gradient_accumulation_steps 4 \
--deepspeed zero2
 
``` 

日志: 

INFO 08-07 15:18:45 [kv_cache_utils.py:837] Maximum concurrency for 16,000 tokens per request: 69.45x

时间预估: 'global_step/max_steps': '15/1520', 'percentage': '0.99%', 'elapsed_time': '5m 45s', 'remaining_time': '9h 37m 25s' . 时间在9-13h波动

与rollout参数:2/2相比, 没有很大区别

调整rollout参数: 开启async模式

```
#rollout参数    
    --vllm_tensor_parallel_size 4 \
    --vllm_data_parallel_size 1 \
    --vllm_use_async_engine true \

 
#vllm参数
--num_generations 8 \
--per_device_train_batch_size 1 \
--per_device_eval_batch_size 2 \
--gradient_accumulation_steps 4 \
--deepspeed zero2 \
--async_generate true 
 
``` 

报错: 

```
INFO:     127.0.0.1:38552 - "POST /infer/ HTTP/1.1" 500 Internal Server Error
ERROR:    Exception in ASGI application
Traceback (most recent call last):
  File "/home/ubuntu/anaconda3/envs/huangyan/lib/python3.10/site-packages/uvicorn/protocols/http/httptools_impl.py", line 409, in run_asgi
    result = await app(  # type: ignore[func-returns-value]
  File "/home/ubuntu/anaconda3/envs/huangyan/lib/python3.10/site-packages/uvicorn/middleware/proxy_headers.py", line 60, in __call__
    return await self.app(scope, receive, send)
  File "/home/ubuntu/anaconda3/envs/huangyan/lib/python3.10/site-packages/fastapi/applications.py", line 1054, in __call__
    await super().__call__(scope, receive, send)
  File "/home/ubuntu/anaconda3/envs/huangyan/lib/python3.10/site-packages/starlette/applications.py", line 113, in __call__
    await self.middleware_stack(scope, receive, send)
  File "/home/ubuntu/anaconda3/envs/huangyan/lib/python3.10/site-packages/starlette/middleware/errors.py", line 186, in __call__
    raise exc
  File "/home/ubuntu/anaconda3/envs/huangyan/lib/python3.10/site-packages/starlette/middleware/errors.py", line 164, in __call__
    await self.app(scope, receive, _send)
  File "/home/ubuntu/anaconda3/envs/huangyan/lib/python3.10/site-packages/starlette/middleware/exceptions.py", line 63, in __call__
    await wrap_app_handling_exceptions(self.app, conn)(scope, receive, send)
  File "/home/ubuntu/anaconda3/envs/huangyan/lib/python3.10/site-packages/starlette/_exception_handler.py", line 53, in wrapped_app
    raise exc
  File "/home/ubuntu/anaconda3/envs/huangyan/lib/python3.10/site-packages/starlette/_exception_handler.py", line 42, in wrapped_app
    await app(scope, receive, sender)
  File "/home/ubuntu/anaconda3/envs/huangyan/lib/python3.10/site-packages/starlette/routing.py", line 716, in __call__
    await self.middleware_stack(scope, receive, send)
  File "/home/ubuntu/anaconda3/envs/huangyan/lib/python3.10/site-packages/starlette/routing.py", line 736, in app
    await route.handle(scope, receive, send)
  File "/home/ubuntu/anaconda3/envs/huangyan/lib/python3.10/site-packages/starlette/routing.py", line 290, in handle
    await self.app(scope, receive, send)
  File "/home/ubuntu/anaconda3/envs/huangyan/lib/python3.10/site-packages/starlette/routing.py", line 78, in app
    await wrap_app_handling_exceptions(app, request)(scope, receive, send)
  File "/home/ubuntu/anaconda3/envs/huangyan/lib/python3.10/site-packages/starlette/_exception_handler.py", line 53, in wrapped_app
    raise exc
  File "/home/ubuntu/anaconda3/envs/huangyan/lib/python3.10/site-packages/starlette/_exception_handler.py", line 42, in wrapped_app
    await app(scope, receive, sender)
  File "/home/ubuntu/anaconda3/envs/huangyan/lib/python3.10/site-packages/starlette/routing.py", line 75, in app
    response = await f(request)
  File "/home/ubuntu/anaconda3/envs/huangyan/lib/python3.10/site-packages/fastapi/routing.py", line 302, in app
    raw_response = await run_endpoint_function(
  File "/home/ubuntu/anaconda3/envs/huangyan/lib/python3.10/site-packages/fastapi/routing.py", line 213, in run_endpoint_function
    return await dependant.call(**values)
  File "/data/ms-swift/swift/llm/infer/rollout.py", line 344, in infer
    all_outputs = list(chain.from_iterable(all_outputs))  # from list of list to single list
TypeError: 'NoneType' object is not iterable

``` 

调整参数, 让总步骤与尝试9一致 (720步):

```
#rollout参数
    --vllm_tensor_parallel_size 4 \
    --vllm_data_parallel_size 1 \

 
#vllm参数
--num_generations 8 \
--per_device_train_batch_size 2 \
--per_device_eval_batch_size 2 \
--gradient_accumulation_steps 4 \
--deepspeed zero2
 
``` 

时间: 'epoch': 0.28, 'global_step/max_steps': '10/720', 'percentage': '1.39%', 'elapsed_time': '9m 25s', 'remaining_time': '11h 9m 10s'

```
#rollout参数
    --vllm_tensor_parallel_size 4 \
    --vllm_data_parallel_size 1 \

 
#vllm参数
--num_generations 8 \
--per_device_train_batch_size 1 \
--per_device_eval_batch_size 2 \
--gradient_accumulation_steps 8 \
--deepspeed zero2
 
``` 

时间: 'global_step/max_steps': '20/720', 'percentage': '2.78%', 'elapsed_time': '21m 32s', 'remaining_time': '12h 34m 14s'

rollout只使用2个GPU, 让另外2个GPU空下来. TP=2

```
#rollout参数
    --vllm_tensor_parallel_size 2 \
    --vllm_data_parallel_size 1 \

 
#vllm参数
--num_generations 8 \
--per_device_train_batch_size 1 \
--per_device_eval_batch_size 2 \
--gradient_accumulation_steps 8 \
--deepspeed zero2
 
``` 

时间: 'epoch': 0.28, 'global_step/max_steps': '10/720', 'percentage': '1.39%', 'elapsed_time': '11m 29s', 'remaining_time': '13h 35m 55s'

rollout只使用2个GPU, 让另外2个GPU空下来. DP=2

```
#rollout参数
    --vllm_tensor_parallel_size 1 \
    --vllm_data_parallel_size 2 \

 
#vllm参数
--num_generations 8 \
--per_device_train_batch_size 1 \
--per_device_eval_batch_size 2 \
--gradient_accumulation_steps 8 \
--deepspeed zero2
 
``` 

日志: [kv_cache_utils.py:837] Maximum concurrency for 16,000 tokens per request: 12.16x

时间: "global_step/max_steps": "10/720", "percentage": "1.39%", "elapsed_time": "16m 15s", "remaining_time": "19h 14m 33s"

rollout只使用2个GPU, 训练使用6个GPU. TP=2

```
#rollout参数
    --vllm_tensor_parallel_size 2 \
    --vllm_data_parallel_size 1 \

 
#vllm参数
--num_generations 12 \
--per_device_train_batch_size 1 \
--per_device_eval_batch_size 2 \
--gradient_accumulation_steps 8 \
--deepspeed zero2
``` 

时间: 'epoch': 0.28, 'global_step/max_steps': '10/720', 'percentage': '1.39%', 'elapsed_time': '13m 44s', 'remaining_time': '16h 15m 7s'

rollout只使用2个GPU, 训练使用6个GPU, 去掉zero2, OOM (增加了--log_entropy true 参数, 导致了内存会有增加)

rollout只使用2个GPU, 训练使用6个GPU, 使用zero1 (增加了--log_entropy true 参数)

  - 'epoch': 0.28, 'global_step/max_steps': '10/720', 'percentage': '1.39%', 'elapsed_time': '15m 43s', 'remaining_time': '18h 36m 25s'

rollout只使用2个GPU, 训练使用6个GPU, 使用zero2 (增加了--log_entropy true 参数)

  - "epoch": 0.27777778, "global_step/max_steps": "10/720", "percentage": "1.39%", "elapsed_time": "13m 29s", "remaining_time": "15h 58m 10s"

对比实验: rollout只使用2个GPU, 训练使用6个GPU, 使用zero2 (去除了--log_entropy true 参数)

  - "epoch": 0.27777778, "global_step/max_steps": "10/720", "percentage": "1.39%", "elapsed_time": "12m 26s", "remaining_time": "14h 43m 48s"

对比实验: rollout只使用2个GPU, 训练使用6个GPU, 使用zero2, 修改rollout参数为DP=2, TP=1

  - "epoch": 0.27777778, "global_step/max_steps": "10/720", "percentage": "1.39%", "elapsed_time": "15m 35s", "remaining_time": "18h 26m 47s"

观察到使用6个GPU时, GPU-0的占用率周期性为0, GPU 1-5的占用率饱和但功率不高, 认为其在通讯上卡主. 

通过nvidia-smi topo -m查看通讯链路: 

```
(base) ubuntu@10-60-141-108:/data/huangyan/sql-ai-2$ nvidia-smi topo -m
	GPU0	GPU1	GPU2	GPU3	GPU4	GPU5	GPU6	GPU7	CPU Affinity	NUMA Affinity	GPU NUMA ID
GPU0	 X 	NODE	NODE	NODE	SYS	SYS	SYS	SYS	0-31,64-95	0		N/A
GPU1	NODE	 X 	NODE	NODE	SYS	SYS	SYS	SYS	0-31,64-95	0		N/A
GPU2	NODE	NODE	 X 	NODE	SYS	SYS	SYS	SYS	0-31,64-95	0		N/A
GPU3	NODE	NODE	NODE	 X 	SYS	SYS	SYS	SYS	0-31,64-95	0		N/A
GPU4	SYS	SYS	SYS	SYS	 X 	NODE	NODE	NODE	32-63,96-123	1		N/A
GPU5	SYS	SYS	SYS	SYS	NODE	 X 	NODE	NODE	32-63,96-123	1		N/A
GPU6	SYS	SYS	SYS	SYS	NODE	NODE	 X 	NODE	32-63,96-123	1		N/A
GPU7	SYS	SYS	SYS	SYS	NODE	NODE	NODE	 X 	32-63,96-123	1		N/A

Legend:

  X    = Self
  SYS  = Connection traversing PCIe as well as the SMP interconnect between NUMA nodes (e.g., QPI/UPI)
  NODE = Connection traversing PCIe as well as the interconnect between PCIe Host Bridges within a NUMA node
  PHB  = Connection traversing PCIe as well as a PCIe Host Bridge (typically the CPU)
  PXB  = Connection traversing multiple PCIe bridges (without traversing the PCIe Host Bridge)
  PIX  = Connection traversing at most a single PCIe bridge
  NV#  = Connection traversing a bonded set of # NVLinks
``` 

在通过nvidia-smi dmon -s t, 确认GPU上的通讯量

还是将训练放在一个NUMA (4个GPU)上, rollout使用另外4个GPU, 使用DP=4的结构, num_generations调整为8, 使得总step=720

  - "epoch": 0.27777778, "global_step/max_steps": "10/720", "percentage": "1.39%", "elapsed_time": "15m 58s", "remaining_time": "18h 54m 24s"

将训练放在一个NUMA (4个GPU)上, rollout使用另外4个GPU, 使用TP=4的结构, num_generations调整为8, 使得总step=720

  - "epoch": 0.27777778, "global_step/max_steps": "10/720", "percentage": "1.39%", "elapsed_time": "10m 12s", "remaining_time": "12h 4m 16s"

将训练放在一个NUMA (4个GPU)上, rollout使用另外4个GPU, 使用TP=2, DP=2的结构, num_generations调整为8, 使得总step=720

  - 'epoch': 0.28, 'global_step/max_steps': '10/720', 'percentage': '1.39%', 'elapsed_time': '9m 53s', 'remaining_time': '11h 41m 53s'

观察到: 每8个step, 会有长时间处于低功耗的状态, GPU间会有700MB/s左右的通讯 (组间通讯, 不是组内通讯)

将训练放在一个NUMA (4个GPU)上, rollout使用另外4个GPU, 使用TP=2, DP=2的结构, num_generations调整为8, per_device_train_batch_size=2,

gradient_accumulation_steps=4, 使得总step=720

  - OOM (log_entropy计算时OOM)

# 尝试11

增加训练数据量到60, 训练方法: (训练放在一个NUMA (4个GPU)上, rollout使用另外4个GPU, 使用TP=2, DP=2的结构)

增加参数: --log_entropy true

commit: 3e7868532d3036e0d03bb2b7ee3d0e5e8d00da7f

训练量: "epoch": 11.07142857, "global_step/max_steps": "620/1120", "percentage": "55.36%", "elapsed_time": "11h 31m 13s", "remaining_time": "9h 17m 26s"

![image2025-8-8 10:18:32.png](/assets/01KJBZW8ND6Q8NTEDX7B4WWACP/image2025-8-8%2010%3A18%3A32.png)

![image2025-8-8 10:20:35.png](/assets/01KJBZW8ND6Q8NTEDX7B4WWACP/image2025-8-8%2010%3A20%3A35.png)

  - train阶段: 
    - kl/grad/clipped_ratio 健康
    - ActionCount的震荡比较大
  - eval阶段:
    - step=450后, Action3F1崩溃
  - 分析completion:
    - 不能达到最高分的case, 一共5个
      - GT中出现了符合格式的多列判断, 无法匹配:

```
- 动作3: 检查语句(主FROM子语句-1.1)的查询字段(order_weight, create_time, delivery_status, transport_mode, distance_km, estimated_hours, customer_id, order_volume, payment_method): 没有被同级或父级语句(主语句-1)引用, 可以删除。
```

      - GT中出现了"可以删除"后多余的内容, reward无法匹配: 

```
- 动作3: 检查语句(子查询inner_activity-1.1.1)的查询字段(activity_type): (检查结果), 可以删除。因为引用此字段的父查询(主FROM子语句-1.1)中的activity_type字段将被删除, 所以此字段也变得无用。
```

      - 对于UNION ALL, GT中值对UNION涉及的子句进行了判断, 而模型输出中, 先对UNION句子本身进行了判断, 然后才对两个子句进行了判断, 所以判断数量多于GT
      - GT中, 使用了对于"可以删除"的另一种说法, 判断错误: 

```
- 动作3: 检查语句(子查询-1.1.1.1)的查询字段(age): (检查结果), 该字段在当前语句的`WHERE age > 18`子句中使用，但并未被外部查询`子查询-1.1.1`所选择(因为`子查询-1.1.1`中的`age`字段已被确定为可删除)，因此可以从SELECT列表中删除。
```

    - 训练过程中大幅下降的case分析: 5个
      - SQL比较长, 导致检查字段出现多值模式: 

```
- 动作3: 检查语句(北京百人以上社群筛选-1.3.2.1)的其他字段(如 s.mingcheng, s.chengshi, chengyuan_count)未被父级语句(北京大社群成员筛选-1.3.2)引用, 可以删除
------
- 检查结论: 这些字段 (last_access_date, total_time_spent, short_notes, device_type, browser_info, ip_address, session_id, user_agent, country_code) 没有被其他语句使用，可以删除。
------
- 动作3: 检查语句(FROM子查询-内容-1.1)的查询字段(neirong, zuozhe_id, chuangjian_shijian, xiugai_shijian, zhuangtai): 没有被同级或父级语句(主语句-1)引用, 可以删除
```

      - GT中, 对UNION ALL的判断, 只判断了父语句, 而推理中判断了父语句+子语句
      - 对UNION的判断, 会遗漏一些字段的分析, 需要在reward中进行检查?
      - 推理中, 出现别名全表达式, 跟GT不匹配. (是否调整reward, 兼容这种问题):

```
- 动作3: 检查语句(左连接子查询-pvs-1.1)的查询字段(SUM(view_count) AS raw_view_count): 没有被同级或父级语句(主语句-1)引用, 可以删除
```

    - 查看评估的崩溃阶段
      - 对AS的处理: 

```
- 动作3: 检查语句(子查询v1-1.1.1)的查询字段(SUM(bingoggr) AS bingoggr): 没有被同级或父级语句(主FROM子语句-1.1)引用, 可以删除
```

      - 大部分的崩溃, 是因为对所有字段都判断了被引用
      - reward需要支持"。可以删除", "。因此可以删除" , "，可删除", ", 所以可以删除","(主语句-1未使用该字段，可删除)", "**未被使用, 可以删除**"

entropy曲线: 

![image2025-8-8 10:27:48.png](/assets/01KJBZW8ND6Q8NTEDX7B4WWACP/image2025-8-8%2010%3A27%3A48.png)

对比测试: 增加参数 --top_entropy_quantile 0.2 

  - 白色曲线为top_entropy_quantile=1.0, epoch没有完全跑完
  - 蓝色曲线为top_entropy_quantile=0.2, epoch完全跑完 (总训练时间 21h 35m 49s)
  - 图中并没有看出 top_entropy_quantile的不同带来的决定性优势

![image2025-8-13 22:12:34.png](/assets/01KJBZW8ND6Q8NTEDX7B4WWACP/image2025-8-13%2022%3A12%3A34.png)
