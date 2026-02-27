---
title: 20240220 - 对reranker进行微调
confluence_page_id: 2590190
created_at: 2024-02-20T06:20:44+00:00
updated_at: 2024-02-27T11:51:38+00:00
---

在[gpu888.cn](<http://gpu888.cn>)机器上进行微调

参考: <https://github.com/FlagOpen/FlagEmbedding/tree/master/examples/reranker>

训练命令: 

```
ALL_PROXY=http://127.0.0.1:7890 torchrun --nproc_per_node 1 \
-m FlagEmbedding.reranker.run \
--output_dir ./train_output/ \
--model_name_or_path BAAI/bge-reranker-base \
--train_data ./train_data.jsonl \
--learning_rate 6e-5 \
--num_train_epochs 5 \
--per_device_train_batch_size 1 \
--gradient_accumulation_steps 4 \
--dataloader_drop_last True \
--train_group_size 16 \
--max_len 512 \
--weight_decay 0.01 \
--logging_steps 10 
``` 

问题: 训练数据中, 对于问题Q的召回文档C1/C2/.../Cn中, 如何表达 C1与Q的相关度 > C2与Q的相关度 > ... > Cn与Q的相关度

尝试使用三元法: 

三元组(Triplet Ranking)  
与成对方法类似,您可以使用三元组,其中包括一个锚点(问题g),一个正样本(更相关的文档c1),和一个负样本(较少相关的文档c2或c3)  
(q, c1, c2)  
(q, c1, c3)  
(q, c2, c3)

代码效果: [rerank.html](/assets/01KJBZ67AKY1THP7X6CANMSBQP/rerank.html)

对于训练数据本身, rerank微调效果很好. 但对于陌生数据, 会导致效果变差. 

目前仅使用13组训练数据, 需要将训练数据集扩大, 再看对陌生数据的效果是否变好. (选取陌生问题时, 需要选择与训练数据相似的问题??)

另外: 对于如何产生训练数据, 使用问题+召回+gpt评估的方法, 可以使用stackoverflow和CSDN上的一些问题作为样例: [https://stackoverflow.com/questions/tagged/mysql?tab=votes&pagesize=50](<https://stackoverflow.com/questions/tagged/mysql?tab=votes&pagesize=50>)

使用LLM, 为99个问题生成评判, 作为训练数据: [result.tar.gz](/assets/01KJBZ67AKY1THP7X6CANMSBQP/result.tar.gz)

扩大数据集后, 在未知问题上的召回结果, 通过简单训练 (epoch = 2, train_loss = 0.34), 获得的效果: [rerank (1).html](/assets/01KJBZ67AKY1THP7X6CANMSBQP/rerank%20%281%29.html)

```
delta_before_FT=228, delta_after_FT=148
delta_before_FT=202, delta_after_FT=122
delta_before_FT=262, delta_after_FT=226
delta_before_FT=396, delta_after_FT=178
delta_before_FT=318, delta_after_FT=160
delta_before_FT=384, delta_after_FT=298
delta_before_FT=488, delta_after_FT=354
delta_before_FT=370, delta_after_FT=292
``` 

再扩大epoch=10, 查看效果: 微调进行了7.5小时, 效果: 

```
delta_before_FT=228, delta_after_FT=136
delta_before_FT=202, delta_after_FT=114
delta_before_FT=262, delta_after_FT=244
delta_before_FT=396, delta_after_FT=130
delta_before_FT=318, delta_after_FT=156
delta_before_FT=384, delta_after_FT=282
delta_before_FT=488, delta_after_FT=328
delta_before_FT=370, delta_after_FT=270
``` 

扩大epoch的收效不大, 预计需要更多组训练数据

测试Hard Negatives, epoch = 1, loss = 0.2, 效果: 

```
delta_before_FT=228, delta_after_FT=194
delta_before_FT=202, delta_after_FT=182
delta_before_FT=262, delta_after_FT=320
delta_before_FT=396, delta_after_FT=276
delta_before_FT=318, delta_after_FT=306
delta_before_FT=384, delta_after_FT=314
delta_before_FT=488, delta_after_FT=394
delta_before_FT=370, delta_after_FT=320
``` 

Hard Negatives的效果提升不大. 推测是因为原本的三元组里就有足够的负向案例, 并且负向案例都是最近近邻生成的

# TODO

  1. 尝试微调embedding, 这样在召回时就可以进行排序
  2. ~~微调时, 使用 Hard Negatives 技术~~
  3. 使用large模型
