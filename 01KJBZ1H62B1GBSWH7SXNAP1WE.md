---
title: 20230708 - 大模型微调 - 分批次微调
confluence_page_id: 2588817
created_at: 2023-07-08T08:04:46+00:00
updated_at: 2023-07-11T13:04:05+00:00
---

# 问题

分批次微调: 

第一批次: 使用的训练数据是 问题类型A 的问题

第一批次训练结束后, 问题A的回答效果正常

第二批次: 使用的训练数据是 问题类型B 的问题

第二批次训练结束后, 问题B的回答效果正常, 问题A的回答效果异常

怀疑是第二批次的训练导致数据偏斜, 向数据类型B过拟合

# 方案

在第二批次, 训练数据仍然使用 问题类型B 的问题, 验证数据 使用 问题类型A+问题类型B 的数据

这样训练的结果, 会同时保有 问题类型A的特征 和 问题类型B的特征

# 具体操作

修改 src/train_sft.py -> src/train_sft.test.py

修改代码: 增加手工指定的eval数据集: 

```
...
# 增加以下部分
    import copy
    eval_data_args = copy.deepcopy(data_args)
    eval_data_args.dataset='test.eval'
    eval_dataset = prepare_data(model_args, eval_data_args)
    eval_dataset = preprocess_data(eval_dataset, tokenizer, eval_data_args, training_args, stage="sft")
 

#修改以下部分
    if training_args.do_train:
        if data_args.dev_ratio > 1e-6:
            dataset = dataset.train_test_split(test_size=data_args.dev_ratio)
            trainer_kwargs = {"train_dataset": dataset["train"], "eval_dataset": dataset["test"]}
        else:
			# 在此处增加eval_dataset
            trainer_kwargs = {"train_dataset": dataset, "eval_dataset": eval_dataset}
    else: # do_eval or do_predict
        trainer_kwargs = {"eval_dataset": dataset}
``` 

训练过程: 

```
# 第一批次用正常脚本训练
 
CUDA_VISIBLE_DEVICES=0 ALL_PROXY=http://127.0.0.1:7890 python src/train_sft.py     \
--do_train    \
--finetuning_type  lora \
--fp16 \
--per_device_train_batch_size 1     \
--gradient_accumulation_steps 4     \
--lr_scheduler_type cosine     \
--logging_steps 10     \
--save_steps 1000     \
--learning_rate 5e-5 \
--max_source_length 1024 \
--max_target_length 4096 \
--use_v2 \
--output_dir path_to_sft_checkpoint.test.v1     \
--dataset test.v1     \
--num_train_epochs 50
 
# 验证
python src/cli_demo.py --checkpoint_dir path_to_sft_checkpoint.test.v1 --use_v2

##  验证问题: 
AST<>是否直接(不包括子查询)使用了WhereClause: Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='tttt'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='xxx')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='5000'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)

 
# 第二批次训练
CUDA_VISIBLE_DEVICES=0 ALL_PROXY=http://127.0.0.1:7890 python src/train_sft.test.py     \
--do_train    \
--per_device_train_batch_size 1     \
--gradient_accumulation_steps 4     \
--lr_scheduler_type cosine     \
--logging_steps 10     \
--save_steps 1000     \
--max_source_length 1024 \
--max_target_length 4096 \
--fp16 \
--use_v2 \
--checkpoint_dir path_to_sft_checkpoint.test.v1 \
--output_dir path_to_sft_checkpoint.test.v2     \
--dataset test.v2     \
--learning_rate 5e-5 \
--finetuning_type lora     \
--num_train_epochs 30
 
# 验证
python src/cli_demo.py --checkpoint_dir path_to_sft_checkpoint.test.v2 --use_v2

 
## 验证问题
 
判断SQL是否符合 我们定义的SQL规则. SQL: <select * from vvvv where ccccc in (select tttt from xxxx limit 5000)>, 规则: <select语句必须使用where条件>
 
AST<>是否直接(不包括子查询)使用了WhereClause: Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='tttt'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='xxx')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='5000'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)
``` 

# 实操

在SQL审核规则匹配的场景下, 分批训练: 

  1. 训练SQL->AST: asts

```
获取SQL<>对应的AST: select * from a where a in (select b from c limit 5) limit 10
```

  2. 训练对AST的判断: explains

```
AST<>是否直接(不包括子查询)使用了SelectLimitClause: Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Keyword(text='IN'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))), group_by=None, having=None, order_by=None, limit=None, into_before_lock_mode=None, lock_mode=None, into=None)
```

  3. 训练规则和判断流程: rules和audits

```
判断SQL是否符合 我们定义的SQL规则: SQL: <select * from a where a in (select b from c limit 5) limit 10>, 规则: <select语句必须使用where条件>
```

在第二批次训练后, "获取SQL<>对应的AST"开始少量失真. 第三批次训练后, 开始大量失真

尝试调大第一批次的epoch

记录各阶段的loss

```
第一批次: 
{'loss': 0.0338, 'learning_rate': 4.522542485937369e-05, 'epoch': 29.09}
{'loss': 0.0179, 'learning_rate': 4.401014914000078e-05, 'epoch': 32.73}
{'loss': 0.0099, 'learning_rate': 4.267766952966369e-05, 'epoch': 36.36}
{'loss': 0.0059, 'learning_rate': 4.123620120825459e-05, 'epoch': 40.0}
{'loss': 0.0049, 'learning_rate': 3.969463130731183e-05, 'epoch': 43.64}
{'loss': 0.0034, 'learning_rate': 3.8062464117898724e-05, 'epoch': 47.27}
{'loss': 0.003, 'learning_rate': 3.634976249348867e-05, 'epoch': 50.91}
{'loss': 0.0025, 'learning_rate': 3.456708580912725e-05, 'epoch': 54.55}
{'loss': 0.002, 'learning_rate': 3.272542485937369e-05, 'epoch': 58.18}
{'loss': 0.002, 'learning_rate': 3.083613409639764e-05, 'epoch': 61.82}
{'loss': 0.0018, 'learning_rate': 2.8910861626005776e-05, 'epoch': 65.45}
{'loss': 0.0015, 'learning_rate': 2.6961477393196126e-05, 'epoch': 69.09}
{'loss': 0.0015, 'learning_rate': 2.5e-05, 'epoch': 72.73}
{'loss': 0.0015, 'learning_rate': 2.303852260680388e-05, 'epoch': 76.36}
{'loss': 0.0013, 'learning_rate': 2.1089138373994223e-05, 'epoch': 80.0}
{'loss': 0.0013, 'learning_rate': 1.9163865903602374e-05, 'epoch': 83.64}
{'loss': 0.0012, 'learning_rate': 1.7274575140626318e-05, 'epoch': 87.27}
{'loss': 0.0011, 'learning_rate': 1.5432914190872757e-05, 'epoch': 90.91}
{'loss': 0.0011, 'learning_rate': 1.3650237506511331e-05, 'epoch': 94.55}
{'loss': 0.0011, 'learning_rate': 1.1937535882101281e-05, 'epoch': 98.18}
{'loss': 0.001, 'learning_rate': 1.0305368692688174e-05, 'epoch': 101.82}
{'loss': 0.001, 'learning_rate': 8.763798791745411e-06, 'epoch': 105.45}
{'loss': 0.0009, 'learning_rate': 7.3223304703363135e-06, 'epoch': 109.09}
{'loss': 0.001, 'learning_rate': 5.989850859999227e-06, 'epoch': 112.73}
{'loss': 0.0009, 'learning_rate': 4.7745751406263165e-06, 'epoch': 116.36}
{'loss': 0.0009, 'learning_rate': 3.6839958911476957e-06, 'epoch': 120.0}
{'loss': 0.0009, 'learning_rate': 2.7248368952908053e-06, 'epoch': 123.64}
{'loss': 0.0008, 'learning_rate': 1.9030116872178316e-06, 'epoch': 127.27}
{'loss': 0.0009, 'learning_rate': 1.2235870926211619e-06, 'epoch': 130.91}
{'loss': 0.0009, 'learning_rate': 6.907519900580861e-07, 'epoch': 134.55}
{'loss': 0.0009, 'learning_rate': 3.077914851215585e-07, 'epoch': 138.18}
{'loss': 0.0009, 'learning_rate': 7.706665667180091e-08, 'epoch': 141.82}
{'loss': 0.0008, 'learning_rate': 0.0, 'epoch': 145.45}
{'train_runtime': 406.8577, 'train_samples_per_second': 5.407, 'train_steps_per_second': 0.983, 'train_loss': 0.16249329805374146, 'epoch': 145
.45}
 
 
第二批次: 
{'loss': 5.5873, 'learning_rate': 4.8339510662430046e-05, 'epoch': 2.67}
{'loss': 0.3737, 'learning_rate': 4.073300977624594e-05, 'epoch': 5.33}
{'loss': 0.1934, 'learning_rate': 2.8910861626005776e-05, 'epoch': 8.0}
{'loss': 0.1127, 'learning_rate': 1.6040801261367493e-05, 'epoch': 10.67}
{'loss': 0.0571, 'learning_rate': 5.571350963575728e-06, 'epoch': 13.33}
{'loss': 0.042, 'learning_rate': 3.077914851215585e-07, 'epoch': 16.0}
{'train_runtime': 56.25, 'train_samples_per_second': 5.333, 'train_steps_per_second': 1.067, 'train_loss': 1.0610544602076213, 'epoch': 16.0}
 
第三批次: 
{'loss': 1.7708, 'learning_rate': 4.9786121534345265e-05, 'epoch': 2.86}
{'loss': 1.0738, 'learning_rate': 4.914814565722671e-05, 'epoch': 5.71}
{'loss': 0.7062, 'learning_rate': 4.8096988312782174e-05, 'epoch': 8.57}
{'loss': 0.3906, 'learning_rate': 4.665063509461097e-05, 'epoch': 11.43}
{'loss': 0.21, 'learning_rate': 4.4833833507280884e-05, 'epoch': 14.29}
{'loss': 0.11, 'learning_rate': 4.267766952966369e-05, 'epoch': 17.14}
{'loss': 0.0421, 'learning_rate': 4.021903572521802e-05, 'epoch': 20.0}
{'loss': 0.0158, 'learning_rate': 3.7500000000000003e-05, 'epoch': 22.86}
{'loss': 0.0057, 'learning_rate': 3.456708580912725e-05, 'epoch': 25.71}
{'loss': 0.0043, 'learning_rate': 3.147047612756302e-05, 'epoch': 28.57}
{'loss': 0.0029, 'learning_rate': 2.8263154805501297e-05, 'epoch': 31.43}
{'loss': 0.0023, 'learning_rate': 2.5e-05, 'epoch': 34.29}
{'loss': 0.0021, 'learning_rate': 2.173684519449872e-05, 'epoch': 37.14}
{'loss': 0.0018, 'learning_rate': 1.852952387243698e-05, 'epoch': 40.0}
{'loss': 0.0017, 'learning_rate': 1.5432914190872757e-05, 'epoch': 42.86}
{'loss': 0.0015, 'learning_rate': 1.2500000000000006e-05, 'epoch': 45.71}
{'loss': 0.0015, 'learning_rate': 9.780964274781984e-06, 'epoch': 48.57}
{'loss': 0.0015, 'learning_rate': 7.3223304703363135e-06, 'epoch': 51.43}
{'loss': 0.0014, 'learning_rate': 5.166166492719124e-06, 'epoch': 54.29}
{'loss': 0.0015, 'learning_rate': 3.3493649053890326e-06, 'epoch': 57.14}
{'loss': 0.0013, 'learning_rate': 1.9030116872178316e-06, 'epoch': 60.0}
{'loss': 0.0013, 'learning_rate': 8.51854342773295e-07, 'epoch': 62.86}
{'loss': 0.0014, 'learning_rate': 2.1387846565474045e-07, 'epoch': 65.71}
{'loss': 0.0014, 'learning_rate': 0.0, 'epoch': 68.57}
{'train_runtime': 500.7528, 'train_samples_per_second': 2.237, 'train_steps_per_second': 0.479, 'train_loss': 0.1813753217458725, 'epoch': 68.57}
``` 

调整每个批次的epoch, 让每个批次大概训练到loss的拐点处, 失真厉害. 

查看最后一批次日志: 

```
{'loss': 1.7753, 'learning_rate': 4.9453690018345144e-05, 'epoch': 2.86}
{'loss': 1.0908, 'learning_rate': 4.783863644106502e-05, 'epoch': 5.71}
{'loss': 0.7348, 'learning_rate': 4.522542485937369e-05, 'epoch': 8.57}
{'loss': 0.4164, 'learning_rate': 4.172826515897146e-05, 'epoch': 11.43}
{'loss': 0.238, 'learning_rate': 3.7500000000000003e-05, 'epoch': 14.29}
{'loss': 0.1389, 'learning_rate': 3.272542485937369e-05, 'epoch': 17.14}
{'loss': 0.0689, 'learning_rate': 2.761321158169134e-05, 'epoch': 20.0}
{'loss': 0.0372, 'learning_rate': 2.238678841830867e-05, 'epoch': 22.86}
{'loss': 0.0197, 'learning_rate': 1.7274575140626318e-05, 'epoch': 25.71}
{'loss': 0.0125, 'learning_rate': 1.2500000000000006e-05, 'epoch': 28.57}
{'loss': 0.0088, 'learning_rate': 8.271734841028553e-06, 'epoch': 31.43}
{'loss': 0.0074, 'learning_rate': 4.7745751406263165e-06, 'epoch': 34.29}
{'loss': 0.0071, 'learning_rate': 2.1613635589349756e-06, 'epoch': 37.14}
{'loss': 0.0071, 'learning_rate': 5.463099816548579e-07, 'epoch': 40.0}
{'loss': 0.007, 'learning_rate': 0.0, 'epoch': 42.86}
{'train_runtime': 311.7591, 'train_samples_per_second': 2.245, 'train_steps_per_second': 0.481, 'train_loss': 0.3046612803141276, 'epoch': 42.86}
``` 

由于前批次的epoch调整, 导致当前批次的loss拐点变化

再次调整, 增大第三批次的epoch: 

```
第一批次: 
{'loss': 2.1402, 'learning_rate': 4.962019382530521e-05, 'epoch': 3.64}
{'loss': 1.6823, 'learning_rate': 4.849231551964771e-05, 'epoch': 7.27}
{'loss': 1.196, 'learning_rate': 4.665063509461097e-05, 'epoch': 10.91}
{'loss': 0.7936, 'learning_rate': 4.415111107797445e-05, 'epoch': 14.55}
{'loss': 0.4486, 'learning_rate': 4.1069690242163484e-05, 'epoch': 18.18}
{'loss': 0.2081, 'learning_rate': 3.7500000000000003e-05, 'epoch': 21.82}
{'loss': 0.0872, 'learning_rate': 3.355050358314172e-05, 'epoch': 25.45}
{'loss': 0.0519, 'learning_rate': 2.9341204441673266e-05, 'epoch': 29.09}
{'loss': 0.0336, 'learning_rate': 2.5e-05, 'epoch': 32.73}
{'loss': 0.0225, 'learning_rate': 2.0658795558326743e-05, 'epoch': 36.36}
{'loss': 0.0142, 'learning_rate': 1.6449496416858284e-05, 'epoch': 40.0}
{'loss': 0.0133, 'learning_rate': 1.2500000000000006e-05, 'epoch': 43.64}
{'loss': 0.0102, 'learning_rate': 8.930309757836517e-06, 'epoch': 47.27}
{'loss': 0.0106, 'learning_rate': 5.848888922025553e-06, 'epoch': 50.91}
{'loss': 0.0099, 'learning_rate': 3.3493649053890326e-06, 'epoch': 54.55}
{'loss': 0.0086, 'learning_rate': 1.5076844803522922e-06, 'epoch': 58.18}
{'loss': 0.0091, 'learning_rate': 3.7980617469479953e-07, 'epoch': 61.82}
{'loss': 0.0095, 'learning_rate': 0.0, 'epoch': 65.45}
{'train_runtime': 182.4393, 'train_samples_per_second': 5.426, 'train_steps_per_second': 0.987, 'train_loss': 0.3749786270989312, 'epoch': 65.4
5}
***** train metrics *****
  epoch                    =      65.45
  train_loss               =      0.375
  train_runtime            = 0:03:02.43
  train_samples_per_second =      5.426
  train_steps_per_second   =      0.987
 
 
第二批次: 
{'loss': 5.457, 'learning_rate': 4.925739315689991e-05, 'epoch': 2.67}
{'loss': 0.3881, 'learning_rate': 4.572593931387604e-05, 'epoch': 5.33}
{'loss': 0.2008, 'learning_rate': 3.969463130731183e-05, 'epoch': 8.0}
{'loss': 0.1256, 'learning_rate': 3.1890933895424976e-05, 'epoch': 10.67}
{'loss': 0.0522, 'learning_rate': 2.3256088156396868e-05, 'epoch': 13.33}
{'loss': 0.024, 'learning_rate': 1.4831583923104999e-05, 'epoch': 16.0}
{'loss': 0.0065, 'learning_rate': 7.633540738525066e-06, 'epoch': 18.67}
{'loss': 0.0029, 'learning_rate': 2.5301488425208296e-06, 'epoch': 21.33}
{'loss': 0.002, 'learning_rate': 1.3695261579316777e-07, 'epoch': 24.0}
{'train_runtime': 84.4019, 'train_samples_per_second': 5.332, 'train_steps_per_second': 1.066, 'train_loss': 0.6954566404223442, 'epoch': 24.0}
***** train metrics *****
  epoch                    =       24.0
  train_loss               =     0.6955
  train_runtime            = 0:01:24.40
  train_samples_per_second =      5.332
  train_steps_per_second   =      1.066
 
第三批次: 
{'loss': 1.7752, 'learning_rate': 4.972077065562821e-05, 'epoch': 2.86}
{'loss': 1.0887, 'learning_rate': 4.888932014465352e-05, 'epoch': 5.71}
{'loss': 0.7257, 'learning_rate': 4.752422169756048e-05, 'epoch': 8.57}
{'loss': 0.4032, 'learning_rate': 4.5655969357899874e-05, 'epoch': 11.43}
{'loss': 0.2194, 'learning_rate': 4.332629679574566e-05, 'epoch': 14.29}
{'loss': 0.118, 'learning_rate': 4.058724504646834e-05, 'epoch': 17.14}
{'loss': 0.0488, 'learning_rate': 3.7500000000000003e-05, 'epoch': 20.0}
{'loss': 0.021, 'learning_rate': 3.413352560915988e-05, 'epoch': 22.86}
{'loss': 0.008, 'learning_rate': 3.056302334890786e-05, 'epoch': 25.71}
{'loss': 0.0064, 'learning_rate': 2.686825233966061e-05, 'epoch': 28.57}
{'loss': 0.0043, 'learning_rate': 2.3131747660339394e-05, 'epoch': 31.43}
{'loss': 0.0033, 'learning_rate': 1.9436976651092144e-05, 'epoch': 34.29}
{'loss': 0.003, 'learning_rate': 1.5866474390840125e-05, 'epoch': 37.14}
{'loss': 0.0028, 'learning_rate': 1.2500000000000006e-05, 'epoch': 40.0}
{'loss': 0.0026, 'learning_rate': 9.412754953531663e-06, 'epoch': 42.86}
{'loss': 0.0023, 'learning_rate': 6.673703204254347e-06, 'epoch': 45.71}
{'loss': 0.0024, 'learning_rate': 4.344030642100133e-06, 'epoch': 48.57}
{'loss': 0.0024, 'learning_rate': 2.475778302439524e-06, 'epoch': 51.43}
{'loss': 0.0023, 'learning_rate': 1.1106798553464804e-06, 'epoch': 54.29}
{'loss': 0.0024, 'learning_rate': 2.7922934437178695e-07, 'epoch': 57.14}
{'loss': 0.0022, 'learning_rate': 0.0, 'epoch': 60.0}
{'train_runtime': 438.1136, 'train_samples_per_second': 2.237, 'train_steps_per_second': 0.479, 'train_loss': 0.21164307026636034, 'epoch': 60.
0}
***** train metrics *****
  epoch                    =       60.0
  train_loss               =     0.2116
  train_runtime            = 0:07:18.11
  train_samples_per_second =      2.237
  train_steps_per_second   =      0.479
``` 

实测, 第一批次的问题会大幅偏移

再次增大前两批次的权重: 效果没有改善. 只有减少后面批次的权重, 才能保留第一批次的部分结果

尝试: 将第二批次 checkpoint进行冻结, 生成model. 再对model进行第三批次训练. 第一批次的问题完全失真: 

````
User: 获取SQL<>对应的AST: select * from a where a in (select b from c limit 5) limit 10

ChatGLM-6B: 为了获取给定的 SQL 查询语句对应的 AST,我们需要将其分解为逻辑查询和投影操作。下面是一种可能的实现方式:

```
WITH c AS (
  SELECT * FROM a
  WHERE a IN (SELECT b FROM c LIMIT 5)
)
SELECT * FROM c LIMIT 10;
```

在这个实现中,我们首先在子查询 `c` 中选择所有与给定的 SQL 查询语句中的条件匹配的行。然后,我们在主查询 `c` 中选择前 10 行,以便只返回与给定 SQL 查询语句匹配的前 10 行数据。

请注意,这种实现方式仅仅是一种可能的方式,具体的 AST 结构可能因查询语句的不同而有所不同。
```` 

感觉eval数据集没有生效

# 调整方案

李剑楠提供了新的想法. 参考论文 , 论文链接： https://arxiv.org/pdf/2212.09278.pdf

剑楠写的分析: <https://actiontech.feishu.cn/docx/RGOXdwv5woXkXJx68jsc0PlpnUc>

有借鉴的思路: 

![image2023-7-11 17:26:1.png](/assets/01KJBZ1H62B1GBSWH7SXNAP1WE/image2023-7-11%2017%3A26%3A1.png)

名词解释: D(SG) 是 主任务, D(RSP)/D(TWP)/D(FUP) 是子任务

分两步训练:

  - 第一步, 将主任务+子任务放在一起训练 (按照数据的关联, 来选取相关的主任务和子任务)
  - 第二步, 单独训练主任务, 对数据效果进行增强

借鉴结果, 将SQL Audit分成三个批次: 

  - 第一批次: 训练SQL -> AST的数据 (数据量大, epoch=100)
  - 第二批次: 训练 其他子任务和主任务的数据 (数据量不大, epoch=50)
  - 第三批次: 将主任务和所有子任务 的数据进行训练, epoch=20较小, 增强效果

效果良好

# 另: 关于eval dataset的作用

在训练中, eval dataset的作用: 每隔eval_steps, 进行一次评估, 并保存成checkpoint. 

最终选择 eval score最高的checkpoint作为最终结果. 

也就是说eval dataset并不影响训练的过程 (仅由train dataset进行影响), 而eval dataset影响的是checkpoint的选择, 会选取 "最不 过拟合"的checkpoint作为最终结果

# 一次尝试

```
User: 判断SQL是否符合 我们定义的SQL规则. SQL: <select * from a where a in (select b from c limit 5) order by 1>, 规则: <select 语句不能使用order by>

ChatGLM-6B: 思考步骤:
1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句不能使用order by]:
    [检查AST中每一个Select和Subselect子句, 禁止使用OrderByClause;否则, SQL不满足规则]
2. [获取SQL<>对应的AST: select * from a where a in (select b from c limit 5) order by 1]:
    [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Keyword(text='IN'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='5'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))), group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='5'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]
3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
    - 步骤2中的Select的AST. AST<>是否直接(不包括子查询)使用了OrderByClause: Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Keyword(text='IN'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='5'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))), group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='5'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None): 否. 结论: 满足规则
    - 步骤2中的Subselect的AST. AST<>是否直接(不包括子查询)使用了OrderByClause: Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='5'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')): 否. 结论: 满足规则
4. 结论: 步骤3中所有检查均满足规则, 结论为该SQL满足规则
 
 
---
 
User: 获取SQL<>对应的AST: select * from a where a in (select b from c limit 5) order by 1

ChatGLM-6B: Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Keyword(text='IN'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='5'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))), group_by=None, having=None, order_by=OrderByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='1')), trailing_comma=None)]), limit=None, into_before_lock_mode=None, lock_mode=None, into=None)
 
``` 

可以看到SQL->AST是正确的, 但用于主流程的AST是错误的. 错误的结果与我们的一组训练数据是一致的: 

```
audits.append({
    "instruction": "判断SQL是否符合 我们定义的SQL规则",
    "input": "SQL: <select * from a where a in (select b from c limit 5) limit 10>, 规则: <select 语句不能使用order by>",
    "output": """思考步骤:
1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句不能使用order by]:
    [检查AST中每一个Select和Subselect子句, 禁止使用OrderByClause;否则, SQL不满足规则]
2. [获取SQL<>对应的AST: select * from a where a in (select b from c limit 5) limit 10]:
    [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Keyword(text='IN'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='5'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))), group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='10'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None)]
3. 根据步骤1的规则, 对步骤2产生的AST进行分析: 
    - 步骤2中的Select的AST. AST<>是否直接(不包括子查询)使用了OrderByClause: Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Keyword(text='IN'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='5'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))), group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='10'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None): 否. 结论: 满足规则
    - 步骤2中的Subselect的AST. AST<>是否直接(不包括子查询)使用了OrderByClause: Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='5'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')): 否. 结论: 满足规则
4. 结论: 步骤3中所有检查均满足规则, 结论为该SQL满足规则
"""
})
``` 

增加第三批次的数据训练epoch: 20 -> 40, 查看效果

增加关联训练后, 效果正确: 

```
User: 判断SQL是否符合 我们定义的SQL规则. SQL: <select * from a where a in (select b from c limit 5) order by 1>, 规则: <select 语句不能使用order by>

ChatGLM-6B: 思考步骤:
1. [获取我们定义的SQL规则<> 对应的 '检查规则': select 语句不能使用order by]:
    [检查AST中每一个Select和Subselect子句, 禁止使用OrderByClause;否则, SQL不满足规则]
2. [获取SQL<>对应的AST: select * from a where a in (select b from c limit 5) order by 1]:
    [Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Keyword(text='IN'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='5'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))), group_by=None, having=None, order_by=OrderByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='1')), trailing_comma=None)]), limit=None, into_before_lock_mode=None, lock_mode=None, into=None)]
3. 根据步骤1的规则, 对步骤2产生的AST进行分析:
    - 步骤2中的Select的AST. AST<>是否直接(不包括子查询)使用了OrderByClause: Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Star(), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='a')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=WhereClause(conditions=BinOp(left=Identifier(text='a'), op=Keyword(text='IN'), right=Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='5'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')))), group_by=None, having=None, order_by=OrderByClause(expr=[WithTrailingComma(node=OrderByExpr(expr=Identifier(text='1')), trailing_comma=None)]), limit=None, into_before_lock_mode=None, lock_mode=None, into=None): 否. 结论: 不满足规则
    - 步骤2中的Subselect的AST. AST<>是否直接(不包括子查询)使用了OrderByClause: Subselect(left_paren=Punctuation(text='('), select=Select(with_clause=None, modifiers=[], select_exprs=[WithTrailingComma(node=SelectExpr(expr=Identifier(text='b'), alias=None), trailing_comma=None)], into_before_from=None, from_clause=FromClause(table=[WithTrailingComma(node=SimpleTableFactor(table_name=SimpleTableName(identifier=Identifier(text='c')), as_kw=None, alias=None, index_hint_list=[]), trailing_comma=None)]), where=None, group_by=None, having=None, order_by=None, limit=SelectLimitClause(row_count=NumericLiteral(value='5'), offset=None, offset_leaf=None), into_before_lock_mode=None, lock_mode=None, into=None), right_paren=Punctuation(text=')')): 是. 结论: 满足规则
4. 结论: 步骤3中有检查不满足规则, 所以该SQL不满足规则
```
