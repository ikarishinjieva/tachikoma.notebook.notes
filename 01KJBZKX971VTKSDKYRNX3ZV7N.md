---
title: 20241121 - 使用ms-swift对Qwen2-VL进行微调 [7] - 用规范数据减少过拟合
confluence_page_id: 3343232
created_at: 2024-11-21T11:44:17+00:00
updated_at: 2024-11-28T10:49:22+00:00
---

# 想法

用代码生成selfies表达式, 然后生成图, 从简单表达式开始训练模型, 查看递进的效果

# 生成数据脚本

[loop_selfies_and_image.ipynb](/assets/01KJBZKX971VTKSDKYRNX3ZV7N/loop_selfies_and_image.ipynb)

# 训练1

用长度1/2/3/4 的selfies的图来 训练, 用长度4的数据来 校验

```
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_1.ms_swift.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_2.ms_swift.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_3.ms_swift.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4.ms_swift.train.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4.ms_swift.val.json \
--eval_steps 20 \
--learning_rate 8e-5 \
--num_train_epochs 10 \
--logging_steps 20 \
--loss_name test_loss \
" > train.log 2>&1
``` 

效果: qwen2-vl-7b-instruct/v78-20241126-102223/runs

![image2024-11-26 15:21:26.png](/assets/01KJBZKX971VTKSDKYRNX3ZV7N/image2024-11-26%2015%3A21%3A26.png)

训练集出现了过拟合 (交叉熵)

使用评估脚本: [evaluate_loop_selfies.ipynb](/assets/01KJBZKX971VTKSDKYRNX3ZV7N/evaluate_loop_selfies.ipynb)

评估结果: 

```
Overall Similarity: 97.96%
                                                        Image Path        Predicted Predicted SMILES               GT GT SMILES  Similarity
24  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/119.png    [C][C][C][#C]            CCC#C   [C][#C][C][=C]    C#CC=C    0.200000
2   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/327.png   [N][#C][C][#N]           N#CC#N   [N][=C][C][#N]    N=CC#N    0.272727
0   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/348.png    [N][C][N][=N]            NCN=N    [N][=N][C][N]     N=NCN    1.000000
53  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/354.png    [C][O][N][=N]            CON=N    [N][=N][O][C]     N=NOC    1.000000
52   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/69.png   [C][N][=C][=N]           CN=C=N   [C][N][=C][=N]    CN=C=N    1.000000
51  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/129.png    [C][#C][N][N]            C#CNN    [C][#C][N][N]     C#CNN    1.000000
50  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/186.png    [O][N][C][#N]            ONC#N    [O][N][C][#N]     ONC#N    1.000000
49  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/171.png     [O][O][O][O]             OOOO     [O][O][O][O]      OOOO    1.000000
48  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/290.png     [N][O][N][N]             NONN     [N][O][N][N]      NONN    1.000000
47   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/43.png     [C][O][N][N]             CONN     [C][O][N][N]      CONN    1.000000
46  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/357.png    [C][N][N][=N]            CNN=N    [N][=N][N][C]     N=NNC    1.000000
``` 

从selfies表达式的角度, 有表达式不一致, 但结构一致, 比如: [C][O][N][=N] 和 [N][=N][O][C]

出错的两个样例 (多次实验都类似), 都是对 "三键+双键" 的识别有问题

想办法改进这部分

观察原始数据, 筛选出"三键+双键"的训练数据: 

```
['/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/352.png'] , [N][=N][C][#C]
['/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/368.png'] , [N][#C][C][=N]
['/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/377.png'] , [N][#C][N][=C]
['/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/130.png'] , [C][#C][N][=C]
['/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/131.png'] , [C][#C][N][=O]
['/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/80.png'] , [C][=C][C][#N]
['/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/326.png'] , [N][=C][C][#C]
['/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/237.png'] , [O][=N][C][#C]
['/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/353.png'] , [N][=N][C][#N]
['/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/238.png'] , [O][=N][C][#N]
['/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/120.png'] , [C][#C][C][=O]
['/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/212.png'] , [O][=C][C][#N]
['/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/121.png'] , [C][#C][C][=N]
['/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/379.png'] , [N][#C][N][=N]
['/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/378.png'] , [N][#C][N][=O]
['/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/105.png'] , [C][=N][C][#C]
['/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/132.png'] , [C][#C][N][=N]
['/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/79.png'] , [C][=C][C][#C]
``` 

与测试中出错的数据相比: 

```
Overall Similarity: 97.96%
                                                        Image Path        Predicted Predicted SMILES               GT GT SMILES  Similarity
24  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/119.png    [C][C][C][#C]            CCC#C   [C][#C][C][=C]    C#CC=C    0.200000
2   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/327.png   [N][#C][C][#N]           N#CC#N   [N][=C][C][#N]    N=CC#N    0.272727
 
---
 
Overall Similarity: 97.60%
                                                        Image Path        Predicted Predicted SMILES               GT GT SMILES  Similarity
68  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/116.png    [C][C][=C][C]            CC=CC    [C][#C][C][C]     C#CCC    0.076923
23  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/106.png   [C][#C][N][=N]           C#CN=N   [C][=N][C][#N]    C=NC#N    0.125000
``` 

  1. 与119 ([C][#C][C][=C]) 最像的就是79 ([C][=C][C][#C]), 两者本质上是同一个分子
  2. 与327 ([N][=C][C][#N]) 最像的是326 ([N][=C][C][#C]), 但预测结果 ([N][#C][C][#N]), 模型是更容易将原子摆正确, 而忽略"键"的重要性

# 训练2

将长度4的训练数据中, 二键和三键联合的分子式的数据, 放在一个单独的文件中 (length_4.ms_swift.key2andkey3.train.json), 参与训练: 

```
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_1.ms_swift.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_2.ms_swift.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_3.ms_swift.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4.ms_swift.train.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4.ms_swift.key2andkey3.train.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4.ms_swift.val.json \
--eval_steps 20 \
--learning_rate 8e-5 \
--num_train_epochs 10 \
--logging_steps 20 \
--loss_name test_loss \
" > train.log 2>&1
``` 

效果: 

qwen2-vl-7b-instruct/v80-20241126-184058/runs

![image2024-11-26 18:57:23.png](/assets/01KJBZKX971VTKSDKYRNX3ZV7N/image2024-11-26%2018%3A57%3A23.png)

评估: 

```
Overall Similarity: 96.88%
                                                        Image Path        Predicted Predicted SMILES               GT GT SMILES  Similarity
68  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/116.png    [C][C][=C][C]            CC=CC    [C][#C][C][C]     C#CCC    0.076923
62   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/53.png    [C][=C][N][C]            C=CNC    [C][N][C][#C]     CNC#C    0.200000
29  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/300.png    [N][#C][N][N]            N#CNN    [N][N][C][#C]     NNC#C    0.384615
 
---
 
Overall Similarity: 95.64%
                                                        Image Path        Predicted Predicted SMILES               GT GT SMILES  Similarity
51  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/129.png    [N][C][#C][N]            NC#CN    [C][#C][N][N]     C#CNN    0.166667
44  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/203.png    [O][=N][N][O]            O=NNO    [O][N][=N][O]     ON=NO    0.166667
24  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/119.png   [C][C][=C][=C]           CC=C=C   [C][#C][C][=C]    C#CC=C    0.200000
62   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/53.png    [C][=C][N][C]            C=CNC    [C][N][C][#C]     CNC#C    0.200000
``` 

错误无法单一归类, 发现有一些分子式是镜像后出的某种错误 (印证Qwen2的图像预处理), 发现的错误: 

  - 原子 顺序错误
  - 键 类型错误
  - 原子 类型错误

未见效果提升

# 训练3

在训练1的基础上, 调整LR=4e-4

```
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_1.ms_swift.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_2.ms_swift.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_3.ms_swift.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4.ms_swift.train.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4.ms_swift.val.json \
--eval_steps 20 \
--learning_rate 4e-4 \
--num_train_epochs 10 \
--logging_steps 20 \
--loss_name test_loss \
" > train.log 2>&1
``` 

效果: qwen2-vl-7b-instruct/v83-20241126-200235/runs

![image2024-11-26 20:19:38.png](/assets/01KJBZKX971VTKSDKYRNX3ZV7N/image2024-11-26%2020%3A19%3A38.png)

学习率升高, train/loss不变, eval/loss降低 (泛化性变好?)

评估: 

```
Overall Similarity: 99.00%
                                                        Image Path        Predicted Predicted SMILES               GT GT SMILES  Similarity
21  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/365.png        [N][C][C]              NCC    [N][#C][C][N]     N#CCN        0.25
 
---
 
Overall Similarity: 99.00%
                                                        Image Path        Predicted Predicted SMILES               GT GT SMILES  Similarity
21  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/365.png        [N][C][C]              NCC    [N][#C][C][N]     N#CCN        0.25

---
 
Overall Similarity: 99.11%
                                                        Image Path        Predicted Predicted SMILES               GT GT SMILES  Similarity
57   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/39.png        [O][O][O]              OOO     [C][O][O][O]      COOO    0.333333
``` 

评估的正确率也在上升

继续提高LR=8e-4, 效果: qwen2-vl-7b-instruct/v85-20241126-231908/runs

![image2024-11-26 23:38:58.png](/assets/01KJBZKX971VTKSDKYRNX3ZV7N/image2024-11-26%2023%3A38%3A58.png)

训练loss差不多, 校验loss继续下降. 中段有梯度爆炸.

评估: 

```
Overall Similarity: 97.87%
                                                        Image Path        Predicted Predicted SMILES               GT GT SMILES  Similarity
50  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/186.png    [O][N][C][=N]            ONC=N    [O][N][C][#N]     ONC#N         0.2
69  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/375.png    [O][N][C][=N]            ONC=N    [N][#C][N][O]     N#CNO         0.2
 
---
 
Overall Similarity: 95.55%
                                                        Image Path        Predicted Predicted SMILES               GT GT SMILES  Similarity
52   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/69.png   [C][=N][=C][N]            C=NCN   [C][N][=C][=N]    CN=C=N    0.058824
21  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/365.png    [N][C][C][=N]            NCC=N    [N][#C][C][N]     N#CCN    0.200000
69  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/375.png    [N][=C][N][O]            N=CNO    [N][#C][N][O]     N#CNO    0.200000
50  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/186.png    [O][N][C][=N]            ONC=N    [O][N][C][#N]     ONC#N    0.200000
``` 

# 训练4

使用增值脚本: [loop_selfies_and_image.20241127.ipynb](/assets/01KJBZKX971VTKSDKYRNX3ZV7N/loop_selfies_and_image.20241127.ipynb)

一个测试数据仅增值一个

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
/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4.ms_swift.train.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4.enhanced.ms_swift.train.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4.ms_swift.val.json \
--eval_steps 20 \
--learning_rate 4e-4 \
--num_train_epochs 10 \
--logging_steps 20 \
--loss_name test_loss \
" > train.log 2>&1
``` 

效果: qwen2-vl-7b-instruct/v87-20241127-000737/runs

![image2024-11-27 11:39:40.png](/assets/01KJBZKX971VTKSDKYRNX3ZV7N/image2024-11-27%2011%3A39%3A40.png)

评估: 多次测试, 准确率100%

```
Overall Similarity: 100.00%
                                                        Image Path        Predicted Predicted SMILES               GT GT SMILES  Similarity
0   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/348.png    [N][C][N][=N]            NCN=N    [N][=N][C][N]     N=NCN         1.0
53  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/354.png    [C][O][N][=N]            CON=N    [N][=N][O][C]     N=NOC         1.0
52   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/69.png   [N][=C][=N][C]           N=C=NC   [C][N][=C][=N]    CN=C=N         1.0
51  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/129.png    [N][N][C][#C]            NNC#C    [C][#C][N][N]     C#CNN         1.0
50  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/186.png    [O][N][C][#N]            ONC#N    [O][N][C][#N]     ONC#N         1.0
49  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/171.png     [O][O][O][O]             OOOO     [O][O][O][O]      OOOO         1.0
``` 

  
数据增强 有助于: 双键/三键 位置等的识别. 单靠 重复数据 做不到这么好的效果

# 训练5

增加长度为5的数据(由selfies生成的遍历数据), 总共1774个

与训练4的不同: 长度1-4的数据使用全量(增强数据), 增加长度5的数据 (但不进行增强)

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
/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5.ms_swift.train.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5.ms_swift.val.json \
--eval_steps 20 \
--learning_rate 4e-4 \
--num_train_epochs 10 \
--logging_steps 20 \
--loss_name test_loss \
" > train.log 2>&1
``` 

效果: 

```
Correct: 352 / 354 = 99.43502824858757 %
                                                          Image Path                  Predicted Predicted SMILES                         GT  GT SMILES  Similarity
154   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/471.png           [C][N][C][=C][C]           CNC=CC          [C][=N][C][=C][C]    C=NC=CC    0.210526
240   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/360.png           [C][C][C][#C][C]           CCC#CC          [C][=C][C][#C][C]    C=CC#CC    0.210526
1    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1468.png          [C][N][=C][=N][N]          CN=C=NN          [N][N][=C][=N][C]    NN=C=NC    1.000000
2     /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/237.png           [C][N][C][=N][C]           CNC=NC           [C][N][C][=N][C]     CNC=NC    1.000000
``` 

两条数据都是 "双键缺失"

增加长度5的增强数据: 

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
/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5.ms_swift.train.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5.enhanced.ms_swift.train.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5.ms_swift.val.json \
--eval_steps 20 \
--learning_rate 4e-4 \
--num_train_epochs 10 \
--logging_steps 20 \
--loss_name test_loss \
" > train.log 2>&1
``` 

结果: 

qwen2-vl-7b-instruct/v92-20241127-153212/runs

![image2024-11-27 18:6:30.png](/assets/01KJBZKX971VTKSDKYRNX3ZV7N/image2024-11-27%2018%3A6%3A30.png)

eval/loss略升高

评估: 

```
Correct: 352 / 354 = 99.43502824858757 %
                                                          Image Path                  Predicted Predicted SMILES                         GT  GT SMILES  Similarity
27   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1103.png           [O][N][C][#C][O]           ONC#CO          [O][=N][C][#C][O]    O=NC#CO    0.210526
240   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/360.png           [C][C][C][#C][C]           CCC#CC          [C][=C][C][#C][C]    C=CC#CC    0.210526
0     /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/536.png          [N][#C][C][C][#C]          N#CCC#C          [C][#C][C][C][#N]    C#CCC#N    1.000000
``` 

使用增强数据无改善, 问题都是 "丢失了双键"

尝试调高LR=8e-4, 拉长epoch=15, 是否能解决问题:

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
/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5.ms_swift.train.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5.enhanced.ms_swift.train.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5.ms_swift.val.json \
--eval_steps 20 \
--learning_rate 8e-4 \
--num_train_epochs 15 \
--logging_steps 20 \
--loss_name test_loss \
" > train.log 2>&1
``` 

效果: 

  
qwen2-vl-7b-instruct/v93-20241128-003746/runs

![image2024-11-28 18:48:45.png](/assets/01KJBZKX971VTKSDKYRNX3ZV7N/image2024-11-28%2018%3A48%3A45.png)

失败的case变多

```
79    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/528.png  [C][N][=N][Ring1][Ring1]           C1N=N1   [C][=N][N][Ring1][Ring1]     C1=NN1    0.090909
27   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1103.png          [O][N][#C][C][O]           ON=CCO          [O][=N][C][#C][O]    O=NC#CO    0.090909
121  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1609.png  [N][=C][N][Ring1][Ring1]           N1=CN1  [N][=C][=N][Ring1][Ring1]    N1=C=N1    0.090909
240   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/360.png          [C][C][#C][C][C]           CC#CCC          [C][=C][C][#C][C]    C=CC#CC    0.210526
209   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/147.png          [C][O][C][=O][C]            COC=O            [C][O][C][O][C]      COCOC    0.214286
```
