---
title: 20241127 - 使用ms-swift对Qwen2-VL进行微调 [8] - 用规范数据减少过拟合[2]
confluence_page_id: 3343274
created_at: 2024-11-27T07:38:27+00:00
updated_at: 2024-11-28T10:39:49+00:00
---

续 [20241121 - 使用ms-swift对Qwen2-VL进行微调 [7] - 用规范数据减少过拟合]

# 用长度1-5的训练结果, 评估长度6的数据

由于评估数据过多, 将评估数据限制为300条. 

评估结果: 

```
Correct: 19 / 300 = 6.333333333333334 %
                                                          Image Path                 Predicted Predicted SMILES                              GT    GT SMILES  Similarity
292  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2087.png            [C][=N][O][=C]            C=NOC    [C][=C][=N][O][Ring1][Ring2]     C1=C=NO1    0.000000
203  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1894.png            [C][=N][C][=C]           C=NC=C    [C][=C][N][=C][Ring1][Ring2]     C1=CN=C1    0.000000
72   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7578.png  [C][=C][O][Ring1][Ring1]           C1=CO1    [N][=C][=C][Ring1][Ring1][O]     N1=C=C1O    0.000000
6    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6968.png             [N][N][=C][N]            NN=CN     [N][N][=C][N][Ring1][Ring2]      N1N=CN1    0.000000
...
``` 

大部分输出都是4-5个原子, 而不是6个原子.

对于正确的记录, 分为两类: 

  1. 4-5个原子 + 1-2个分支符号, 比如: 
     1. 这两个分子在结构上是一致的

```
274  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/8494.png             [O][N][N][=O]            ONN=O   [N][Branch1][Ring1][N][=O][O]      N(N=O)O    1.000000
``` 
     2. 偶尔能计算出6个原子的正确的分子式: 

```
39   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7288.png      [N][=C][O][O][C][#C]         N=COOC#C            [N][=C][O][O][C][#C]     N=COOC#C    1.000000
264  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7641.png      [N][N][N][N][=C][=N]         NNNN=C=N            [N][=C][=N][N][N][N]     N=C=NNNN    1.000000
``` 

# 训练6

增加长度为6的数据(由selfies生成的遍历数据), 总共8504个

数据量增大, eval step改为100: 

```
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_1.ms_swift.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_1.enhanced.ms_swift.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_2.ms_swift.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_2.enhanced.ms_swift.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_3.ms_swift.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_3.enhanced.ms_swift.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4.ms_swift.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4.enhanced.ms_swift.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5.ms_swift.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5.enhanced.ms_swift.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6.ms_swift.train.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6.ms_swift.val.json \
--eval_steps 100 \
--learning_rate 4e-4 \
--num_train_epochs 10 \
--logging_steps 100 \
--loss_name test_loss \
" > train.log 2>&1
``` 

效果: qwen2-vl-7b-instruct/v95-20241128-101050

![image2024-11-28 18:32:8.png](/assets/01KJBZM0EKNHYYJ7Q27SMJ2WV6/image2024-11-28%2018%3A32%3A8.png)

评估: 

```
Correct: 298 / 300 = 99.33333333333333 %
                                                          Image Path                      Predicted Predicted SMILES                              GT    GT SMILES  Similarity
249   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/483.png    [C][=C][C][Ring1][Ring1][O]          C1=CC1O     [C][C][=C][Ring1][Ring1][O]      C1C=C1O    0.142857
123  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2597.png              [C][#C][C][#C][N]          C#CC#CN            [C][#C][C][#C][N][C]     C#CC#CNC    0.315789
0    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4267.png             [C][N][C][N][N][O]           CNCNNO              [O][N][N][C][N][C]       ONNCNC    1.000000
202  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6085.png            [N][O][N][N][=C][N]          NONN=CN             [N][C][=N][N][O][N]      NC=NNON    1.000000
``` 

两个失败的case: 

  - 双键位置错误
  - 原子个数错误

# 训练7

使用少量长度为6的数据 (让其与长度5的数据一致), 看看预测是否正确

# 后续方向

  - 将原子数增加 进行逐步训练, 直到某一步不需要该长度的训练集, 就可以在该长度的校验集上达成较好效果
  - 将其他类型图片加入, 找到训练方法
  - 计算数据的疑惑度, 找到重点数据, 减小训练集
  - 将selfies的原子可用列表 扩大
