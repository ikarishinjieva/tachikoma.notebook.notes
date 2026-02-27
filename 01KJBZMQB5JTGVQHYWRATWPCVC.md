---
title: 20241202 - 使用ms-swift对Qwen2-VL进行微调 [10] - 用梯度贡献来进行数据集的选择 [2]
confluence_page_id: 3343361
created_at: 2024-12-02T05:50:04+00:00
updated_at: 2024-12-04T05:32:25+00:00
---

根据[20241128 - 使用ms-swift对Qwen2-VL进行微调 [9] - 用梯度贡献来进行数据集的选择]的结果, 对长度5的数据进行筛选, 并训练测试.

已有数据: selected_data.length_1_4.500.json, 从长度1-4的数据中, 在裸模型上选取梯度贡献最大的500个

新数据: selected_data.length_5.500.json, 从长度5的数据中, 在裸模型上选取梯度贡献最大的500个:

![image2024-12-2 13:34:22.png](/assets/01KJBZMQB5JTGVQHYWRATWPCVC/image2024-12-2%2013%3A34%3A22.png)

```
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5.ms_swift.train.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5.enhanced.ms_swift.train.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5.ms_swift.val.json \
--eval_steps 20 \
--learning_rate 4e-4 \
--num_train_epochs 10 \
--logging_steps 20 \
" > train.log 2>&1
``` 

(总数据量: 2840, 从中选取500条数据)

训练:

```
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/selected_data.length_1_4.500.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/selected_data.length_5.500.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5.ms_swift.val.json \
--eval_steps 20 \
--learning_rate 4e-4 \
--num_train_epochs 10 \
--logging_steps 20 \
" > train.log 2>&1
``` 

训练效果: v139-20241202-135041

对比:

  - v90: 长度1-4数据带增强, 长度5数据不带增强
  - v92: 长度1-5数据带增强

![image2024-12-2 23:18:58.png](/assets/01KJBZMQB5JTGVQHYWRATWPCVC/image2024-12-2%2023%3A18%3A58.png)

过拟合现象更严重, train/loss收敛更快, 但eval/loss跟不上, eval/acc更低

评估: 使用best step: /opt/huangyan/ms-swift/output/qwen2-vl-7b-instruct/v139-20241202-135041/checkpoint-440

正确率: 81%:

```
Correct: 243 / 300 = 81.0 %
                                                          Image Path                 Predicted Predicted SMILES                         GT  GT SMILES  Similarity
163  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1186.png      [C][Ring1][N][Ring1]                C    [N][C][C][Ring1][Ring1]      N1CC1    0.000000
123  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1231.png                [N][C][=N]             NC=N    [N][C][N][Ring1][Ring1]      N1CN1    0.000000
190   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/315.png                   [C][=N]              C=N   [C][N][=C][Ring1][Ring1]     C1N=C1    0.000000
261   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/603.png  [C][#N][C][Ring1][Ring1]              C#N   [C][#C][N][Ring1][Ring1]     C1#CN1    0.000000
93    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/576.png                [C][#C][O]             C#CO   [C][#C][O][Ring1][Ring1]     C1#CO1    0.000000
204   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/747.png                [O][C][=N]             OC=N   [O][C][=N][Ring1][Ring1]     O1C=N1    0.000000
121  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1609.png  [N][=C][N][Ring1][Ring1]           N1=CN1  [N][=C][=N][Ring1][Ring1]    N1=C=N1    0.090909
207   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/138.png  [C][=C][C][Ring1][Ring1]           C1=CC1   [C][C][#C][Ring1][Ring1]     C1C#C1    0.111111
237   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/813.png   [O][O][C][Ring1][Ring1]            O1OC1    [O][O][O][Ring1][Ring1]      O1OO1    0.142857
279   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/742.png          [O][=C][N][N][O]           O=CNNO           [O][C][=N][N][O]     OC=NNO    0.200000
240   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/360.png          [C][C][#C][C][C]           CC#CCC          [C][=C][C][#C][C]    C=CC#CC    0.210526
224   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/361.png         [O][C][=C][C][=C]          OC=CC=C          [C][=C][C][#C][O]    C=CC#CO    0.210526
239  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1707.png           [N][=C][=C][#N]          N=C=C=N         [N][#C][C][=C][=N]   N#CC=C=N    0.214286
293    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/67.png             [C][C][=N][O]            CC=NO           [C][C][N][=C][O]     CCN=CO    0.222222
178  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1619.png             [O][O][C][=N]            OOC=N           [N][=N][C][O][O]     N=NCOO    0.222222
179  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1676.png            [N][=N][C][=O]           N=NC=O          [N][=N][N][=C][O]    N=NN=CO    0.235294
141  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1147.png             [O][=N][N][N]            O=NNN          [O][=N][N][=N][N]    O=NN=NN    0.250000
37   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1285.png             [O][C][#C][N]            OC#CN           [N][C][#C][O][C]     NC#COC    0.250000
275  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1215.png           [N][N][C][O][N]            NNCON            [N][C][N][O][N]      NCNON    0.263158
10    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/906.png             [C][=N][N][O]            C=NNO           [O][N][N][=C][C]     ONN=CC    0.294118
200   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/115.png             [C][=N][N][O]            C=NNO           [C][C][=N][N][O]     CC=NNO    0.294118
185   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/792.png             [C][#C][O][O]            C#COO           [O][O][C][#C][C]     OOC#CC    0.312500
118  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1387.png             [N][N][C][=C]            NNC=C           [N][N][C][=C][C]     NNC=CC    0.312500
148    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/77.png             [C][=C][C][O]            C=CCO           [C][C][=C][C][O]     CC=CCO    0.312500
221   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/103.png             [C][=N][C][C]            C=NCC           [C][C][=N][C][C]     CC=NCC    0.312500
104   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/130.png             [C][#C][O][O]            C#COO           [C][C][#C][O][O]     CC#COO    0.312500
253   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/867.png             [C][#C][N][O]            C#CNO           [O][N][C][#C][C]     ONC#CC    0.312500
17    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/107.png            [O][=C][N][=C]           O=CN=C          [C][C][=N][C][=O]    CC=NC=O    0.312500
89   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1080.png            [O][=N][C][=O]           O=NC=O          [O][=N][C][C][=O]    O=NCC=O    0.333333
250   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/734.png             [O][C][=N][O]            OC=NO          [O][C][=N][C][=O]    OC=NC=O    0.333333
259  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1200.png              [N][O][C][N]             NOCN            [N][C][O][N][N]      NCONN    0.333333
102  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1447.png             [N][C][=N][N]            NC=NN           [N][N][=C][C][N]     NN=CCN    0.333333
211  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1450.png             [N][C][=N][N]            NC=NN          [N][N][=C][C][=N]    NN=CC=N    0.333333
38    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/750.png             [N][C][#C][O]            NC#CO           [O][C][#C][C][N]     OC#CCN    0.333333
248   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/703.png             [C][C][=C][O]            CC=CO           [O][C][=C][C][C]     OC=CCC    0.333333
65    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/121.png             [C][C][#C][C]            CC#CC           [C][C][#C][C][C]     CC#CCC    0.333333
288   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/144.png             [C][O][C][=N]            COC=N           [C][O][C][C][=N]     COCC=N    0.375000
213  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1343.png              [N][O][N][C]             NONC            [N][O][N][C][C]      NONCC    0.375000
244   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/787.png             [O][=C][O][O]            O=COO          [O][O][C][=C][=O]    OOC=C=O    0.375000
60    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/851.png              [O][N][O][N]             ONON            [O][N][C][O][N]      ONCON    0.375000
262   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/219.png             [C][N][C][=N]            CNC=N           [C][N][C][C][=N]     CNCC=N    0.375000
21    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/814.png              [O][O][N][C]             OONC            [O][O][N][C][C]      OONCC    0.375000
184    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/58.png              [O][O][N][C]             OONC            [C][C][N][O][O]      CCNOO    0.375000
232   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/968.png            [O][=C][C][#N]           O=CC#N          [O][=C][C][C][#N]    O=CCC#N    0.375000
63   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1420.png              [N][C][N][N]             NCNN            [N][N][N][C][N]      NNNCN    0.461538
169  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1218.png              [N][C][N][N]             NCNN            [N][C][N][N][N]      NCNNN    0.461538
234  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1161.png             [N][C][C][=O]            NCC=O           [N][C][C][C][=O]     NCCC=O    0.500000
76      /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/7.png             [C][#C][C][C]            C#CCC           [C][C][C][C][#C]     CCCC#C    0.500000
103   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/493.png             [C][=N][O][O]            C=NOO           [C][=N][O][O][O]     C=NOOO    0.500000
75     /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/14.png              [C][C][N][N]             CCNN            [C][C][C][N][N]      CCCNN    0.500000
50   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1687.png             [N][C][C][#N]            NCC#N           [N][#C][C][C][N]     N#CCCN    0.500000
296   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/515.png             [C][=N][N][N]            C=NNN           [C][=N][N][N][N]     C=NNNN    0.500000
155   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/395.png             [C][=C][N][N]            C=CNN           [C][=C][N][N][N]     C=CNNN    0.500000
285   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/999.png             [O][=C][O][O]            O=COO           [O][=C][O][O][O]     O=COOO    0.538462
33    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/902.png              [O][N][N][N]             ONNN            [O][N][N][N][N]      ONNNN    0.583333
160   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/628.png              [O][C][C][C]             OCCC            [O][C][C][C][C]      OCCCC    0.583333
15   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1158.png              [N][C][C][O]             NCCO            [N][C][C][C][O]      NCCCO    0.583333
202   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/817.png          [C][=C][N][O][O]           C=CNOO           [O][O][N][C][=C]     OONC=C    1.000000
``` 

原子数错误 占大多数

\---

增加数据筛选, 增加到1000数据, 训练参数同上.

效果: v142-20241202-235044

![image2024-12-3 8:17:45.png](/assets/01KJBZMQB5JTGVQHYWRATWPCVC/image2024-12-3%208%3A17%3A45.png)

评估: 使用best step: /opt/huangyan/ms-swift/output/qwen2-vl-7b-instruct/v142-20241202-235044/checkpoint-600

正确率: 94%

```
Correct: 282 / 300 = 94.0 %
                                                          Image Path                 Predicted Predicted SMILES                         GT  GT SMILES  Similarity
204   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/747.png  [O][=N][C][Ring1][Ring1]             O=NC   [O][C][=N][Ring1][Ring1]     O1C=N1    0.000000
27   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1103.png            [O][#C][C][=O]           O=CC=O          [O][=N][C][#C][O]    O=NC#CO    0.062500
163  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1186.png  [N][=C][N][Ring1][Ring1]           N1=CN1    [N][C][C][Ring1][Ring1]      N1CC1    0.090909
121  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1609.png  [C][N][=C][Ring1][Ring1]           C1N=C1  [N][=C][=N][Ring1][Ring1]    N1=C=N1    0.090909
240   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/360.png             [C][C][=C][C]            CC=CC          [C][=C][C][#C][C]    C=CC#CC    0.133333
79    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/528.png  [C][N][=C][Ring1][Ring1]           C1N=C1   [C][=N][N][Ring1][Ring1]     C1=NN1    0.166667
35    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/164.png          [C][O][C][N][=N]           COCN=N           [C][O][C][=N][N]     COC=NN    0.181818
236   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/158.png          [C][O][C][C][=N]           COCC=N           [C][O][C][=C][N]     COC=CN    0.190476
28    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/333.png  [C][N][=C][Ring1][Ring1]           C1N=C1   [C][N][=N][Ring1][Ring1]     C1N=N1    0.200000
17    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/107.png         [O][=N][C][=C][C]          O=NC=CC          [C][C][=N][C][=O]    CC=NC=O    0.263158
36    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/914.png          [O][N][=N][N][N]           ON=NNN           [O][N][N][=N][N]     ONN=NN    0.263158
199  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1638.png            [N][#C][N][=N]           N#CN=N          [N][=N][C][#C][N]    N=NC#CN    0.312500
65    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/121.png             [C][C][#C][C]            CC#CC           [C][C][#C][C][C]     CC#CCC    0.333333
0     /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/536.png            [C][#C][C][#N]           C#CC#N          [C][#C][C][C][#N]    C#CCC#N    0.333333
89   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1080.png            [O][=N][C][=O]           O=NC=O          [O][=N][C][C][=O]    O=NCC=O    0.333333
102  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1447.png             [N][C][=N][N]            NC=NN           [N][N][=C][C][N]     NN=CCN    0.333333
218   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/626.png             [C][#C][C][N]            C#CCN     [C][Branch1][C][N][#C]     C(N)#C    0.363636
56    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/970.png             [O][=C][C][O]            O=CCO           [O][=C][C][O][O]     O=CCOO    0.375000
201  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1698.png          [N][#C][C][N][N]           N#CCNN           [N][#C][C][N][N]     N#CCNN    1.000000
``` 

原子数错误一共9例, 占错误的一半. 

在这个基础上, 训练长度6的数据, 选取数据1000.

# 选取长度6的数据, 选取1000个

![image2024-12-3 8:51:16.png](/assets/01KJBZMQB5JTGVQHYWRATWPCVC/image2024-12-3%208%3A51%3A16.png)

提取数据: 

```
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6.ms_swift.train.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6.enhanced.ms_swift.train.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6.ms_swift.val.json \
--eval_steps 20 \
--learning_rate 4e-4 \
--num_train_epochs 10 \
--logging_steps 20 \
" > train.log 2>&1
``` 

数据总量: 13608

训练: 

```
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/selected_data.length_1_4.500.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/selected_data.length_5.1000.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/selected_data.length_6.1000.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6.ms_swift.val.json \
--eval_steps 20 \
--learning_rate 4e-4 \
--num_train_epochs 10 \
--logging_steps 20 \
" > train.log 2>&1
``` 

训练效果: qwen2-vl-7b-instruct/v146-20241203-095647/runs

![image2024-12-3 13:44:57.png](/assets/01KJBZMQB5JTGVQHYWRATWPCVC/image2024-12-3%2013%3A44%3A57.png)

过拟合严重, eval/loss整体围绕0.3变化不大, 怀疑是没有找对方向. 准备调大LR尝试: 

```
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/selected_data.length_1_4.500.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/selected_data.length_5.1000.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/selected_data.length_6.1000.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6.ms_swift.val.json \
--eval_steps 20 \
--learning_rate 8e-4 \
--num_train_epochs 10 \
--logging_steps 20 \
" > train.log 2>&1
``` 

![image2024-12-3 17:56:42.png](/assets/01KJBZMQB5JTGVQHYWRATWPCVC/image2024-12-3%2017%3A56%3A42.png)

LR调大后, 过拟合变得更严重, v147

调小LR

```
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/selected_data.length_1_4.500.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/selected_data.length_5.1000.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/selected_data.length_6.1000.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6.ms_swift.val.json \
--eval_steps 20 \
--learning_rate 1e-4 \
--num_train_epochs 10 \
--logging_steps 20 \
" > train.log 2>&1
``` 

效果: 

qwen2-vl-7b-instruct/v148-20241203-175906/runs

![image2024-12-3 23:20:55.png](/assets/01KJBZMQB5JTGVQHYWRATWPCVC/image2024-12-3%2023%3A20%3A55.png)

调小LR, 导致了过拟合在后期更严重. 

# 选取长度6的数据, 选取2000个

步骤同上, 训练: 

```
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/selected_data.length_1_4.500.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/selected_data.length_5.1000.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/selected_data.length_6.2000.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6.ms_swift.val.json \
--eval_steps 20 \
--learning_rate 4e-4 \
--num_train_epochs 10 \
--logging_steps 20 \
" > train.log 2>&1
``` 

效果: v151-20241204-004403

![image2024-12-4 9:55:46.png](/assets/01KJBZMQB5JTGVQHYWRATWPCVC/image2024-12-4%209%3A55%3A46.png)

用best step: /opt/huangyan/ms-swift/output/qwen2-vl-7b-instruct/v151-20241204-004403/checkpoint-1340 进行评估: 

正确率89%

```
Correct: 269 / 300 = 89.66666666666666 %
                                                          Image Path                     Predicted Predicted SMILES                              GT    GT SMILES  Similarity
105  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3544.png              [C][=N][C][C][O]           C=NCCO     [O][C][=N][C][Ring1][Ring1]      OC1=NC1    0.050000
159   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/736.png          [N][=C][C][C][=C][O]         N=CCC=CO            [C][O][C][=C][C][=N]     COC=CC=N    0.120000
287  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6022.png   [N][C][=C][C][Ring1][Ring1]          NC1=CC1     [N][C][=C][Ring1][Ring1][C]      N1C=C1C    0.125000
292  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2087.png   [C][=C][N][O][Ring1][Ring2]          C1=CNO1    [C][=C][=N][O][Ring1][Ring2]     C1=C=NO1    0.133333
143  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7016.png   [N][C][=N][Ring1][Ring1][N]           N1C=N1     [N][N][=C][Ring1][Ring1][N]      N1N=C1N    0.142857
58   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1915.png   [C][=N][C][Ring1][Ring1][O]          C1=NC1O     [C][=C][N][Ring1][Ring1][O]      C1=CN1O    0.142857
249   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/483.png   [C][=C][C][Ring1][Ring1][O]          C1=CC1O     [C][C][=C][Ring1][Ring1][O]      C1C=C1O    0.142857
86   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7398.png          [N][C][C][=N][N][=N]         NCC=NN=N            [N][=C][N][N][=C][N]     N=CNN=CN    0.153846
203  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1894.png   [C][=N][C][C][Ring1][Ring2]          C1=NCC1    [C][=C][N][=C][Ring1][Ring2]     C1=CN=C1    0.153846
147  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3669.png                  [O][C][C][C]             OCCC        [O][C][Branch1][C][C][O]       OC(C)O    0.166667
139  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2018.png  [C][=C][=C][C][Ring1][Ring2]         C1=C=CC1   [C][=C][=C][=C][Ring1][Ring2]    C1=C=C=C1    0.200000
211  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3258.png    [C][N][O][O][Ring1][Ring2]           C1NOO1      [O][C][O][N][Ring1][Ring2]       O1CON1    0.250000
116  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1639.png          [C][=C][C][C][=C][C]         C=CCC=CC            [C][=C][C][=C][C][C]     C=CC=CCC    0.272727
97   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1661.png        [N][=C][C][=C][=C][=C]       N=CC=C=C=C          [C][=C][C][=C][=C][=N]   C=CC=C=C=N    0.272727
240  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7150.png   [C][=N][C][O][Ring1][Ring2]          C1=NCO1     [N][=C][C][O][Ring1][Ring2]      N1=CCO1    0.285714
189  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7550.png       [N][=C][=C][=C][=C][=C]      N=C=C=C=C=C          [N][=C][=C][=C][=C][O]   N=C=C=C=CO    0.300000
259  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6007.png         [N][C][=C][=C][N][=O]        NC=C=CN=O           [N][C][=C][=N][C][=O]    NC=C=NC=O    0.304348
25   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2925.png             [C][=C][C][#C][C]          C=CC#CC  [C][Branch1][Ring1][C][=C][#C]     C(C=C)#C    0.312500
4    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4871.png          [O][=C][N][C][=C][N]         O=CNC=CN            [O][=C][N][N][=C][N]     O=CNN=CN    0.318182
207  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/5109.png         [O][=C][=C][N][=N][O]        O=C=CN=NO           [O][=C][=N][N][=N][O]    O=C=NN=NO    0.318182
96   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2538.png          [C][#C][C][C][=N][O]         C#CCC=NO            [C][#C][C][N][=N][O]     C#CCN=NO    0.347826
77   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/5051.png         [O][=C][=N][C][N][=O]        O=C=NCN=O            [O][=C][=N][C][N][O]     O=C=NCNO    0.363636
218   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/394.png             [C][=C][O][C][#C]          C=COC#C            [C][C][=C][O][C][#C]     CC=COC#C    0.400000
34   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/5206.png      [C][=N][C][C][=C][N][=O]       C=NCC=CN=O           [O][=N][C][=C][N][=C]    O=NC=CN=C    0.428571
296  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7287.png          [N][=C][O][O][C][#N]         N=COOC#N            [N][=C][O][O][C][=N]     N=COOC=N    0.437500
277  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2356.png              [N][C][N][N][=C]           NCNN=C             [C][=N][N][C][N][N]      C=NNCNN    0.473684
222  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6016.png             [N][C][=C][=N][N]          NC=C=NN            [N][C][=C][=N][N][N]     NC=C=NNN    0.500000
88   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7078.png              [N][N][=N][N][N]           NN=NNN             [N][N][=N][N][N][N]      NN=NNNN    0.500000
253  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3653.png              [O][N][C][#C][O]           ONC#CO             [O][C][#C][N][N][O]      OC#CNNO    0.533333
197  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3965.png               [O][N][N][O][O]            ONNOO              [O][O][N][N][N][O]       OONNNO    0.600000
153  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7102.png              [O][C][C][C][=N]           OCCC=N             [N][=C][C][C][C][O]      N=CCCCO    0.625000
201  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3600.png          [N][#C][C][C][#C][O]         N#CCC#CO            [O][C][#C][C][C][#N]     OC#CCC#N    1.000000
``` 

用last step: /opt/huangyan/ms-swift/output/qwen2-vl-7b-instruct/v151-20241204-004403/checkpoint-1340 进行评估: 

```
Correct: 263 / 300 = 87.66666666666667 %
                                                          Image Path                      Predicted Predicted SMILES                              GT    GT SMILES  Similarity
173  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3177.png        [O][C][Ring1][Ring1][N]             O=CN      [O][C][C][Ring1][Ring1][N]       O1CC1N    0.071429
58   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1915.png    [C][=N][O][Ring1][Ring1][O]           C1=NO1     [C][=C][N][Ring1][Ring1][O]      C1=CN1O    0.076923
72   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7578.png    [N][=C][C][Ring1][Ring1][O]          N1=CC1O    [N][=C][=C][Ring1][Ring1][O]     N1=C=C1O    0.125000
105  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3544.png    [O][C][N][=C][Ring1][Ring1]          OC1N=C1     [O][C][=N][C][Ring1][Ring1]      OC1=NC1    0.125000
292  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2087.png    [C][=C][N][O][Ring1][Ring2]          C1=CNO1    [C][=C][=N][O][Ring1][Ring2]     C1=C=NO1    0.133333
249   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/483.png    [C][=C][C][Ring1][Ring1][O]          C1=CC1O     [C][C][=C][Ring1][Ring1][O]      C1C=C1O    0.142857
147  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3669.png                   [O][C][O][C]             OCOC        [O][C][Branch1][C][C][O]       OC(C)O    0.153846
59   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6537.png     [N][N][O][Ring1][Ring1][N]            N1NO1      [N][O][N][Ring1][Ring1][N]       N1ON1N    0.166667
211  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3258.png     [O][N][C][O][Ring1][Ring2]           O1NCO1      [O][C][O][N][Ring1][Ring2]       O1CON1    0.250000
25   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2925.png                     [C][=C][C]             C=CC  [C][Branch1][Ring1][C][=C][#C]     C(C=C)#C    0.250000
54    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/487.png               [N][C][C][=N][C]           NCC=NC             [C][C][=N][C][C][N]      CC=NCCN    0.272727
36    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/428.png              [O][=C][=N][C][C]          O=C=NCC           [C][C][=C][N][=C][=O]    CC=CN=C=O    0.272727
2    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/8502.png                 [C][=N][N][=H]          C=NN[H]  [N][Branch1][Ring1][N][=N][=C]     N(N=N)=C    0.272727
63   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7751.png              [O][N][C][=N][=N]           ONC=NN            [N][=N][C][=C][N][O]     N=NC=CNO    0.272727
163   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/256.png                [C][C][N][C][O]            CCNCO             [C][C][N][C][#C][O]      CCNC#CO    0.300000
126  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7988.png          [C][=C][N][=N][N][=H]       C=CN=NN[H]           [N][=N][N][=N][C][=C]    N=NN=NC=C    0.333333
267  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4403.png           [N][C][=C][N][=N][O]         NC=CN=NO            [O][N][=C][C][=C][N]     ON=CC=CN    0.333333
276   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/804.png               [C][O][O][C][=O]           COOC=O             [C][O][O][C][C][=O]      COOCC=O    0.350000
254  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/5276.png              [O][=N][C][=N][O]          O=NC=NO            [O][=N][O][C][=N][O]     O=NOC=NO    0.350000
85    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/803.png               [C][O][O][C][=C]           COOC=C             [C][O][O][C][C][=C]      COOCC=C    0.350000
80   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1512.png           [C][N][=N][O][C][=H]        CN=NOC[H]            [C][N][=N][O][C][=N]     CN=NOC=N    0.368421
257  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4094.png               [N][=N][C][N][O]           N=NCNO             [O][N][C][N][N][=N]      ONCNN=N    0.400000
218   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/394.png              [C][=C][O][C][#C]          C=COC#C            [C][C][=C][O][C][#C]     CC=COC#C    0.400000
26   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/5166.png               [O][=N][N][C][C]           O=NNCC             [O][=N][C][N][C][C]      O=NCNCC    0.400000
52    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/388.png               [C][=C][O][C][C]           C=COCC             [C][C][=C][O][C][C]      CC=COCC    0.421053
230  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6271.png               [N][O][C][=C][C]           NOC=CC             [N][O][C][=C][C][C]      NOC=CCC    0.421053
106  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6436.png               [O][=C][N][O][N]           O=CNON            [N][O][N][C][=C][=O]     NONC=C=O    0.450000
222  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6016.png              [N][N][=C][=C][N]          NN=C=CN            [N][C][=C][=N][N][N]     NC=C=NNN    0.500000
88   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7078.png               [N][N][=N][N][N]           NN=NNN             [N][N][=N][N][N][N]      NN=NNNN    0.500000
149  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1477.png               [C][N][=N][C][C]           CN=NCC             [C][N][=N][C][C][C]      CN=NCCC    0.500000
253  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3653.png               [O][C][#C][N][O]           OC#CNO             [O][C][#C][N][N][O]      OC#CNNO    0.533333
177  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/5607.png               [N][C][N][N][=N]           NCNN=N             [N][C][C][N][N][=N]      NCCNN=N    0.555556
197  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3965.png                [O][O][N][N][O]            OONNO              [O][O][N][N][N][O]       OONNNO    0.600000
153  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7102.png               [N][=C][C][C][O]           N=CCCO             [N][=C][C][C][C][O]      N=CCCCO    0.625000
12   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2650.png               [C][#C][O][O][N]           C#COON             [C][#C][O][O][O][N]      C#COOON    0.625000
237  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/5556.png               [N][C][C][N][=O]           NCCN=O             [N][C][C][C][N][=O]      NCCCN=O    0.625000
221  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/5541.png                [N][C][C][C][C]            NCCCC              [N][C][C][C][C][C]       NCCCCC    0.769231
272   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/299.png           [C][=C][=N][N][C][C]         C=C=NNCC            [C][C][N][N][=C][=C]     CCNN=C=C    1.000000
``` 

原子数错误的问题占多数, 考虑用多任务学习来优化, 样例输出: 

```
{"atom_count": N, "selfies": "SELFIES表达式"}
``` 

同时考虑测试长度为7的数据, 看看是否2000数据足够用. 猜测需要数据翻倍.
