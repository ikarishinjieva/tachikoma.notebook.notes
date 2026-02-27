---
title: 20241204 - 使用ms-swift对Qwen2-VL进行微调 [11] - 尝试多任务训练
confluence_page_id: 3343399
created_at: 2024-12-04T08:42:38+00:00
updated_at: 2024-12-06T07:08:33+00:00
---

# 遇到的问题

在[20241202 - 使用ms-swift对Qwen2-VL进行微调 [10] - 用梯度贡献来进行数据集的选择 [2]] 中, 企图通过梯度筛选来减少训练所需的数据.

当进行到长度为6时, 使用梯度筛选选择了2000数据, 训练准确率仍不到90%, 大部分错误集中于原子数不对. 

想尝试使用多任务训练, 让模型重视原子数.

# 准备数据

[loop_selfies_and_image copy.ipynb](/assets/01KJBZN5YYTD5QQ8Z1D621Z88K/loop_selfies_and_image%20copy.ipynb)

数据样例: 

```
{"query": "Print atom count and selfies of the image <image>, in json format", "response": "{\"atom_count\": 1, \"selfies\": \"[N]\"}", "images": ["/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_1/3.png"]}
``` 

# 从裸模型中, 选取长度1-4的500数据

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
" > train.log 2>&1
``` 

生成数据: /opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/selected_data.length_1_4.500.json

训练: 

```
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/selected_data.length_1_4.500.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/length_4.ms_swift.val.json \
--eval_steps 20 \
--learning_rate 4e-4 \
--num_train_epochs 10 \
--logging_steps 20 \
" > train.log 2>&1
``` 

与v134 对比训练效果 ([20241128 - 使用ms-swift对Qwen2-VL进行微调 [9] - 用梯度贡献来进行数据集的选择]):

![image2024-12-4 15:57:22.png](/assets/01KJBZN5YYTD5QQ8Z1D621Z88K/image2024-12-4%2015%3A57%3A22.png)

注意: 由于 标准输出 的长度改变, loss和acc可能不可比

评估:

```
Correct: 71 / 75 = 94.66666666666667 %
                                                        Image Path                                       Predicted Predicted SMILES                                              GT GT SMILES  Similarity
38   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/96.png  {"atom_count": 4, "selfies": "[C][C][=C][=N]"}           CC=C=N  {"atom_count": 4, "selfies": "[C][=C][=N][C]"}    C=C=NC    0.125000
34  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/243.png   {"atom_count": 4, "selfies": "[O][N][C][=O]"}            ONC=O   {"atom_count": 4, "selfies": "[O][=N][N][O]"}     O=NNO    0.285714
33   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/87.png  {"atom_count": 4, "selfies": "[C][=C][C][=C]"}           C=CC=C  {"atom_count": 4, "selfies": "[C][=C][N][=C]"}    C=CN=C    0.300000
67  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/191.png       {"atom_count": 3, "selfies": "[O][N][O]"}              ONO    {"atom_count": 4, "selfies": "[O][N][N][O]"}      ONNO    0.500000
``` 

有一个长度判断错误

调大LR:

```
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/selected_data.length_1_4.500.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/length_4.ms_swift.val.json \
--eval_steps 20 \
--learning_rate 8e-4 \
--num_train_epochs 10 \
--logging_steps 20 \
" > train.log 2>&1
``` 

![image2024-12-4 16:38:41.png](/assets/01KJBZN5YYTD5QQ8Z1D621Z88K/image2024-12-4%2016%3A38%3A41.png)

train/loss差不多, 但梯度更大, 导致eval/loss更好 (学得更激进, 导致泛化性升高)

评估: 用last step: /opt/huangyan/ms-swift/output/qwen2-vl-7b-instruct/v156-20241204-162054/checkpoint-310

```
Correct: 72 / 75 = 96.0 %
                                                        Image Path                                       Predicted Predicted SMILES                                              GT GT SMILES  Similarity
16   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/28.png   {"atom_count": 4, "selfies": "[C][#C][C][O]"}            C#CCO   {"atom_count": 4, "selfies": "[C][C][#C][O]"}     CC#CO    0.133333
22  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/273.png   {"atom_count": 4, "selfies": "[N][=C][N][N]"}            N=CNN   {"atom_count": 4, "selfies": "[N][C][=N][N]"}     NC=NN    0.133333
72  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/369.png      {"atom_count": 3, "selfies": "[N][#C][C]"}             N#CC  {"atom_count": 4, "selfies": "[N][#C][C][#C]"}    N#CC#C    0.272727
``` 

仍然存在原子数不同的情况

# 从裸模型中, 选取长度5的500数据, 与v139进行比较

选取数据: 

```
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/length_5.ms_swift.train.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/length_5.enhanced.ms_swift.train.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/length_5.ms_swift.val.json \
--eval_steps 100 \
--learning_rate 4e-4 \
--num_train_epochs 10 \
--logging_steps 20 \
" > train.log 2>&1
``` 

训练: 

```
#!/bin/bash

bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/selected_data.length_1_4.500.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/selected_data.length_5.500.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/length_5.ms_swift.val.json \
--eval_steps 20 \
--learning_rate 4e-4 \
--num_train_epochs 10 \
--logging_steps 20 \
" > train.log 2>&1

sleep 300

bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/selected_data.length_1_4.500.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/selected_data.length_5.500.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/length_5.ms_swift.val.json \
--eval_steps 20 \
--learning_rate 6e-4 \
--num_train_epochs 10 \
--logging_steps 20 \
" > train.log 2>&1

sleep 300

bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/selected_data.length_1_4.500.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/selected_data.length_5.500.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/length_5.ms_swift.val.json \
--eval_steps 20 \
--learning_rate 8e-4 \
--num_train_epochs 10 \
--logging_steps 20 \
" > train.log 2>&1

sleep 300

bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/selected_data.length_1_4.500.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/selected_data.length_5.500.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/length_5.ms_swift.val.json \
--eval_steps 20 \
--learning_rate 8e-4 \
--num_train_epochs 40 \
--logging_steps 20 \
" > train.log 2>&1

sleep 300

bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/selected_data.length_1_4.500.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/selected_data.length_5.500.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/length_5.ms_swift.val.json \
--eval_steps 20 \
--learning_rate 1e-3 \
--num_train_epochs 10 \
--logging_steps 20 \
" > train.log 2>&1

sleep 300

bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/selected_data.length_1_4.500.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/selected_data.length_5.500.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/length_5.ms_swift.val.json \
--eval_steps 20 \
--learning_rate 1e-3 \
--num_train_epochs 40 \
--logging_steps 20 \
" > train.log 2>&1

``` 

v139: [20241202 - 使用ms-swift对Qwen2-VL进行微调 [10] - 用梯度贡献来进行数据集的选择 [2]], 从长度1-4中选取500数据, 从长度5中选取500数据, LR,`4e-``4, epoch=10, `正确率81%

v159: 从长度1-4中选取500数据, 从长度5中选取500数据, 输出中增加了对原子数的描述, LR=`4e-``4, epoch=10`

v160: 同上, LR=6e-4, epoch=10

v161: 同上, LR=8e-4, epoch=10

v162: 同上, LR=8e-4, epoch=40

v163: 同上, LR=1e-3, epoch=10

v164: 同上, LR=1e-3, epoch=40

![image2024-12-5 11:29:46.png](/assets/01KJBZN5YYTD5QQ8Z1D621Z88K/image2024-12-5%2011%3A29%3A46.png)

逐一对比: 

v159 与 v139 对比: 

![image2024-12-5 11:30:45.png](/assets/01KJBZN5YYTD5QQ8Z1D621Z88K/image2024-12-5%2011%3A30%3A45.png)

带有原子数的描述, 进行训练, 效果明显增强 (但需注意: 训练数据的输出格式改变, 导致输出长度增加, 所以loss和acc可能不能进行对比)

v159 与 v160 与 v161 与 v163 对比: 

![image2024-12-5 11:33:28.png](/assets/01KJBZN5YYTD5QQ8Z1D621Z88K/image2024-12-5%2011%3A33%3A28.png)

随着LR升高, 未见效果提升, 反而会变得糟糕

v161与v162对比: 

![image2024-12-5 11:34:42.png](/assets/01KJBZN5YYTD5QQ8Z1D621Z88K/image2024-12-5%2011%3A34%3A42.png)

随着epoch升高, 效果未见明显提升

v163与v164对比:

![image2024-12-5 11:35:27.png](/assets/01KJBZN5YYTD5QQ8Z1D621Z88K/image2024-12-5%2011%3A35%3A27.png)

LR过大, epoch越长反而导致 不收敛

评估: v159 last model: /opt/huangyan/ms-swift/output/qwen2-vl-7b-instruct/v159-20241204-180322/checkpoint-620

准确率92,3%

```
Correct: 327 / 354 = 92.37288135593221 %
                                                          Image Path                                                  Predicted Predicted SMILES                                                        GT  GT SMILES  Similarity
81     /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/75.png  {"atom_count": 5, "selfies": "[N][Branch1][C][C][Ring1]"}               NC   {"atom_count": 5, "selfies": "[C][C][N][Ring1][Ring1]"}      C1CN1    0.000000
98                                                              None                                                       None             None                                                      None       None    0.000000
301    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/30.png      {"atom_count": 5, "selfies": "[P][Branch1][C][C][C]"}            P(C)C   {"atom_count": 5, "selfies": "[C][C][C][Ring1][Ring1]"}      C1CC1    0.000000
9     /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/243.png  {"atom_count": 5, "selfies": "[N][Branch1][C][C][Ring1]"}               NC   {"atom_count": 5, "selfies": "[C][N][C][Ring1][Ring1]"}      C1NC1    0.000000
184   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/702.png      {"atom_count": 5, "selfies": "[N][Branch1][C][C][O]"}            N(C)O   {"atom_count": 5, "selfies": "[O][C][N][Ring1][Ring1]"}      O1CN1    0.000000
17    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/603.png                 {"atom_count": 3, "selfies": "[N][C][=C]"}             NC=C  {"atom_count": 5, "selfies": "[C][#C][N][Ring1][Ring1]"}     C1#CN1    0.000000
325   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/657.png   {"atom_count": 5, "selfies": "[O][=C][C][Ring1][Ring1]"}             O=CC   {"atom_count": 5, "selfies": "[O][C][C][Ring1][Ring1]"}      O1CC1    0.000000
130  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1258.png   {"atom_count": 5, "selfies": "[N][=C][C][Ring1][Ring1]"}           N1=CC1  {"atom_count": 5, "selfies": "[N][C][=C][Ring1][Ring1]"}     N1C=C1    0.090909
189   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/745.png          {"atom_count": 5, "selfies": "[O][N][=N][=C][O]"}           ON=NCO         {"atom_count": 5, "selfies": "[O][C][=N][N][=O]"}    OC=NN=O    0.095238
272  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1690.png          {"atom_count": 5, "selfies": "[N][=C][C][#C][N]"}          N=CC#CN         {"atom_count": 5, "selfies": "[N][#C][C][C][=N]"}    N#CCC=N    0.190476
32    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/480.png          {"atom_count": 5, "selfies": "[C][#C][C][N][=C]"}          C#CCN=C         {"atom_count": 5, "selfies": "[C][=N][C][#C][C]"}    C=NC#CC    0.190476
188  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1103.png           {"atom_count": 5, "selfies": "[O][N][C][#C][O]"}           ONC#CO         {"atom_count": 5, "selfies": "[O][=N][C][#C][O]"}    O=NC#CO    0.210526
104  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1040.png         {"atom_count": 5, "selfies": "[O][=C][=C][N][#N]"}         O=C=CN=N        {"atom_count": 5, "selfies": "[O][=C][=C][C][#N]"}   O=C=CC#N    0.250000
85    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/597.png          {"atom_count": 5, "selfies": "[C][=N][N][C][#C]"}          C=NNC#C        {"atom_count": 5, "selfies": "[C][#C][N][=C][=C]"}   C#CN=C=C    0.250000
19     /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/55.png           {"atom_count": 5, "selfies": "[F][#C][N][C][C]"}            FCNCC          {"atom_count": 5, "selfies": "[C][C][N][C][#C]"}     CCNC#C    0.250000
137   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/654.png           {"atom_count": 5, "selfies": "[O][C][#C][C][C]"}           OC#CCC          {"atom_count": 5, "selfies": "[O][C][C][#C][C]"}     OCC#CC    0.263158
161  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1304.png            {"atom_count": 5, "selfies": "[N][O][O][C][O]"}            NOOCO           {"atom_count": 5, "selfies": "[N][O][C][O][O]"}      NOCOO    0.263158
79    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/750.png           {"atom_count": 5, "selfies": "[N][C][#C][C][O]"}           NC#CCO          {"atom_count": 5, "selfies": "[O][C][#C][C][N]"}     OC#CCN    0.263158
128  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1042.png           {"atom_count": 5, "selfies": "[O][C][=C][O][O]"}           OC=COO         {"atom_count": 5, "selfies": "[O][=C][=C][O][O]"}    O=C=COO    0.263158
37    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/594.png          {"atom_count": 5, "selfies": "[C][N][=C][C][#C]"}          CN=CC#C         {"atom_count": 5, "selfies": "[C][#C][N][=C][C]"}    C#CN=CC    0.300000
121   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/360.png              {"atom_count": 4, "selfies": "[C][C][#C][C]"}            CC#CC         {"atom_count": 5, "selfies": "[C][=C][C][#C][C]"}    C=CC#CC    0.307692
282   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/111.png              {"atom_count": 4, "selfies": "[C][=N][O][C]"}            C=NOC          {"atom_count": 5, "selfies": "[C][C][=N][O][C]"}     CC=NOC    0.312500
299   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/129.png              {"atom_count": 4, "selfies": "[C][#C][O][C]"}            C#COC          {"atom_count": 5, "selfies": "[C][C][#C][O][C]"}     CC#COC    0.333333
56    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/165.png              {"atom_count": 4, "selfies": "[C][#C][O][C]"}            C#COC          {"atom_count": 5, "selfies": "[C][O][C][#C][C]"}     COC#CC    0.333333
148   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/250.png           {"atom_count": 5, "selfies": "[K][N][O][C][#C]"}         [K]NOC#C          {"atom_count": 5, "selfies": "[C][N][O][C][#C]"}     CNOC#C    0.444444
245   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/629.png               {"atom_count": 4, "selfies": "[O][C][C][O]"}             OCCO           {"atom_count": 5, "selfies": "[O][C][C][C][O]"}      OCCCO    0.500000
12    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/632.png              {"atom_count": 4, "selfies": "[O][C][C][=O]"}            OCC=O          {"atom_count": 5, "selfies": "[O][C][C][C][=O]"}     OCCC=O    0.500000
250  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1632.png         {"atom_count": 5, "selfies": "[N][=C][=C][N][=N]"}         N=C=CN=N        {"atom_count": 5, "selfies": "[N][=N][C][=C][=N]"}   N=NC=C=N    1.000000
``` 

其中有6/27个case 原子种类识别错误, 7/27个case 原子数量识别错误

# 从裸模型中, 选取长度6的500数据, 与v146 (长度6的1000数据) 进行比较

选取数据: 

```
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/length_6.ms_swift.train.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/length_6.enhanced.ms_swift.train.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/length_6.ms_swift.val.json \
--eval_steps 100 \
--learning_rate 4e-4 \
--num_train_epochs 10 \
--logging_steps 20 \
" > train.log 2>&1
``` 

训练: 

v166-20241205-125425

```
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/selected_data.length_1_4.500.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/selected_data.length_5.500.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/selected_data.length_6.500.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/length_6.ms_swift.val.json \
--eval_steps 20 \
--learning_rate 4e-4 \
--num_train_epochs 10 \
--logging_steps 20 \
" > train.log 2>&1
``` 

![image2024-12-5 15:11:0.png](/assets/01KJBZN5YYTD5QQ8Z1D621Z88K/image2024-12-5%2015%3A11%3A0.png)

loss大幅下降 (但需注意: 训练数据的输出格式改变, 导致输出长度增加, 所以loss和acc可能不能进行对比)

评估: last model: /opt/huangyan/ms-swift/output/qwen2-vl-7b-instruct/v166-20241205-125425/checkpoint-930

准确率74%:

```
Correct: 224 / 300 = 74.66666666666667 %
                                                          Image Path                                                    Predicted Predicted SMILES                                                              GT   GT SMILES  Similarity
164    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/31.png                 {"atom_count": 4, "selfies": "[C][C][C][C]"}             CCCC      {"atom_count": 6, "selfies": "[C][C][C][C][Ring1][Ring2]"}      C1CCC1    0.000000
229   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/144.png               {"atom_count": 4, "selfies": "[C][=C][C][=C]"}           C=CC=C     {"atom_count": 6, "selfies": "[C][C][C][#C][Ring1][Ring2]"}     C1CC#C1    0.000000
210  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2759.png  {"atom_count": 6, "selfies": "[N][N][C][=C][Ring1][Ring1]"}          NN1C=C1     {"atom_count": 6, "selfies": "[C][#C][N][N][Ring1][Ring2]"}     C1#CNN1    0.000000
122  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/5720.png            {"atom_count": 5, "selfies": "[N][C][=C][C][=O]"}          NC=CC=O      {"atom_count": 6, "selfies": "[N][C][O][C][Ring1][Ring1]"}      NC1OC1    0.052632
104    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/94.png         {"atom_count": 6, "selfies": "[N][=O][C][=C][C][C]"}              N=O            {"atom_count": 6, "selfies": "[C][C][C][=C][N][=O]"}    CCC=CN=O    0.058824
227   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/531.png         {"atom_count": 6, "selfies": "[N][=O][N][O][=C][C]"}              N=O            {"atom_count": 6, "selfies": "[C][C][=N][O][N][=O]"}    CC=NON=O    0.058824
163  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7250.png            {"atom_count": 5, "selfies": "[N][=C][N][C][=N]"}          N=CNC=N    {"atom_count": 6, "selfies": "[N][=C][C][Ring1][Ring1][=N]"}    N1=CC1=N    0.062500
208  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2605.png                {"atom_count": 4, "selfies": "[O][C][C][=C]"}            OCC=C     {"atom_count": 6, "selfies": "[C][#C][C][Ring1][Ring1][O]"}     C1#CC1O    0.066667
105  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3335.png   {"atom_count": 6, "selfies": "[N][N][C][O][Ring1][Ring2]"}           N1NCO1      {"atom_count": 6, "selfies": "[O][C][N][N][Ring1][Ring1]"}      OC1NN1    0.071429
34   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3179.png     {"atom_count": 5, "selfies": "[O][=C][O][Ring1][Ring1]"}             O=CO     {"atom_count": 6, "selfies": "[O][C][C][Ring1][Ring1][=O]"}     O1CC1=O    0.071429
78    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/149.png    {"atom_count": 5, "selfies": "[O][=C][=C][Ring1][Ring1]"}            O=C=C     {"atom_count": 6, "selfies": "[C][C][C][Ring1][Ring1][=O]"}     C1CC1=O    0.083333
3    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1028.png     {"atom_count": 5, "selfies": "[N][=C][C][Ring1][Ring1]"}           N1=CC1      {"atom_count": 6, "selfies": "[C][N][C][C][Ring1][Ring2]"}      C1NCC1    0.083333
89   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1446.png            {"atom_count": 5, "selfies": "[C][=C][=C][N][C]"}          C=C=CNC           {"atom_count": 6, "selfies": "[C][N][=C][=C][=C][C]"}   CN=C=C=CC    0.125000
66   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4588.png     {"atom_count": 6, "selfies": "[N][Branch1][C][C][O][N]"}           N(C)ON        {"atom_count": 6, "selfies": "[O][N][Branch1][C][N][C]"}      ON(N)C    0.133333
290   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/480.png    {"atom_count": 5, "selfies": "[N][=C][=C][Ring1][Ring1]"}          N1=C=C1    {"atom_count": 6, "selfies": "[C][C][=C][=N][Ring1][Ring1]"}    CC1=C=N1    0.142857
110  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3674.png     {"atom_count": 6, "selfies": "[O][Branch1][C][O][C][C]"}           O(O)CC        {"atom_count": 6, "selfies": "[O][C][Branch1][C][O][C]"}      OC(O)C    0.153846
144  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2737.png          {"atom_count": 6, "selfies": "[N][N][N][C][C][=O]"}          NNNCC=O            {"atom_count": 6, "selfies": "[C][#C][N][N][C][=O]"}    C#CNNC=O    0.153846
226  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7206.png    {"atom_count": 5, "selfies": "[N][=C][=C][Ring1][Ring1]"}          N1=C=C1    {"atom_count": 6, "selfies": "[N][=C][C][=C][Ring1][Ring2]"}    N1=CC=C1    0.166667
277  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4584.png        {"atom_count": 5, "selfies": "[N][Branch1][C][C][O]"}            N(C)O        {"atom_count": 6, "selfies": "[O][N][Branch1][C][C][N]"}      ON(C)N    0.166667
203  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3210.png      {"atom_count": 5, "selfies": "[O][C][C][Ring1][Ring1]"}            O1CC1      {"atom_count": 6, "selfies": "[O][C][O][C][Ring1][Ring1]"}      OC1OC1    0.166667
61   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4848.png          {"atom_count": 6, "selfies": "[N][C][N][O][N][=O]"}          NCNON=O            {"atom_count": 6, "selfies": "[O][=C][N][O][N][=C]"}    O=CNON=C    0.185185
127  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/5923.png            {"atom_count": 5, "selfies": "[N][C][=C][C][=C]"}          NC=CC=C            {"atom_count": 6, "selfies": "[N][C][=C][C][#C][C]"}    NC=CC#CC    0.190476
225   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/313.png         {"atom_count": 6, "selfies": "[N][C][=C][C][C][#C]"}         NC=CCC#C            {"atom_count": 6, "selfies": "[C][C][N][=C][C][#C]"}    CCN=CC#C    0.192308
205  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6043.png         {"atom_count": 6, "selfies": "[N][C][N][=C][=C][O]"}         NCN=C=CO            {"atom_count": 6, "selfies": "[N][C][=N][C][=C][O]"}    NC=NC=CO    0.200000
240  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6641.png         {"atom_count": 6, "selfies": "[N][N][=C][=N][C][N]"}         NN=C=NCN            {"atom_count": 6, "selfies": "[N][N][C][=C][=N][N]"}    NNC=C=NN    0.208333
114  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3653.png              {"atom_count": 5, "selfies": "[O][N][N][C][O]"}            ONNCO             {"atom_count": 6, "selfies": "[O][C][#C][N][N][O]"}     OC#CNNO    0.210526
255  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2357.png         {"atom_count": 6, "selfies": "[N][=C][N][N][N][=C]"}         N=CNNN=C            {"atom_count": 6, "selfies": "[C][=N][N][C][N][=C]"}    C=NNCN=C    0.217391
49   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2195.png         {"atom_count": 6, "selfies": "[N][=N][C][C][N][=C]"}         N=NCCN=C            {"atom_count": 6, "selfies": "[C][=N][C][N][=N][C]"}    C=NCN=NC    0.227273
195  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4650.png          {"atom_count": 6, "selfies": "[N][N][C][C][C][=O]"}          NNCCC=O             {"atom_count": 6, "selfies": "[O][=C][C][N][N][C]"}     O=CCNNC    0.250000
56   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1531.png         {"atom_count": 6, "selfies": "[N][=N][N][N][C][#C]"}         N=NNNC#C            {"atom_count": 6, "selfies": "[C][N][=N][N][C][#C]"}    CN=NNC#C    0.250000
63    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/744.png          {"atom_count": 6, "selfies": "[N][N][N][C][=C][C]"}          NNNC=CC             {"atom_count": 6, "selfies": "[C][O][C][=C][N][N]"}     COC=CNN    0.260870
120  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/5242.png           {"atom_count": 5, "selfies": "[N][#C][C][#N][=O]"}           N#CC#N           {"atom_count": 6, "selfies": "[O][=N][C][#C][C][#N]"}   O=NC#CC#N    0.266667
257   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/844.png          {"atom_count": 6, "selfies": "[N][=C][O][O][O][C]"}          N=COOOC             {"atom_count": 6, "selfies": "[C][O][O][O][N][=C]"}     COOON=C    0.272727
241   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/503.png         {"atom_count": 6, "selfies": "[O][C][=N][C][=N][C]"}         OC=NC=NC            {"atom_count": 6, "selfies": "[C][C][=N][C][=C][O]"}    CC=NC=CO    0.272727
0    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/5995.png        {"atom_count": 6, "selfies": "[N][C][=C][=C][C][=C]"}        NC=C=CC=C          {"atom_count": 6, "selfies": "[N][C][=C][=C][=C][=C]"}  NC=C=C=C=C    0.272727
141  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4493.png            {"atom_count": 5, "selfies": "[N][#C][N][=C][O]"}          N#CN=CO           {"atom_count": 6, "selfies": "[O][N][=C][=N][C][#N]"}   ON=C=NC#N    0.285714
91   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3630.png             {"atom_count": 5, "selfies": "[N][#C][O][C][O]"}           N#COCO            {"atom_count": 6, "selfies": "[O][C][#C][O][C][#N]"}    OC#COC#N    0.285714
223  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3771.png          {"atom_count": 6, "selfies": "[N][O][O][C][=C][O]"}          NOOC=CO             {"atom_count": 6, "selfies": "[O][O][C][=C][O][N]"}     OOC=CON    0.285714
230  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1428.png  {"atom_count": 6, "selfies": "[N][N][=C][C][Ring1][Ring2]"}          N1N=CC1     {"atom_count": 6, "selfies": "[C][N][=C][N][Ring1][Ring2]"}     C1N=CN1    0.285714
7    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1584.png        {"atom_count": 6, "selfies": "[N][=C][=C][C][C][=C]"}        N=C=CCC=C            {"atom_count": 6, "selfies": "[C][=C][C][C][=N][C]"}    C=CCC=NC    0.304348
296  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1753.png          {"atom_count": 6, "selfies": "[N][C][O][O][C][=C]"}          NCOOC=C             {"atom_count": 6, "selfies": "[C][=C][O][O][N][C]"}     C=COONC    0.304348
69   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/8033.png            {"atom_count": 5, "selfies": "[N][#C][C][#C][O]"}          N#CC#CO            {"atom_count": 6, "selfies": "[N][#C][C][C][#C][O]"}    N#CCC#CO    0.315789
88    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/570.png            {"atom_count": 5, "selfies": "[N][#C][C][#C][C]"}          N#CC#CC            {"atom_count": 6, "selfies": "[C][C][#C][C][C][#N]"}    CC#CCC#N    0.315789
129  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2207.png         {"atom_count": 6, "selfies": "[N][=C][C][=C][O][C]"}         N=CC=COC            {"atom_count": 6, "selfies": "[C][=N][C][=C][O][C]"}    C=NC=COC    0.318182
209  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1024.png             {"atom_count": 5, "selfies": "[C][N][C][#C][C]"}           CNC#CC             {"atom_count": 6, "selfies": "[C][N][C][C][#C][C]"}     CNCC#CC    0.333333
250   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/856.png     {"atom_count": 6, "selfies": "[N][Branch1][C][C][O][C]"}           N(C)OC              {"atom_count": 6, "selfies": "[C][O][O][N][O][C]"}      COONOC    0.333333
186  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3795.png         {"atom_count": 6, "selfies": "[O][O][C][=N][N][#C]"}         OOC=NN=C            {"atom_count": 6, "selfies": "[O][O][C][=N][C][#N]"}    OOC=NC#N    0.347826
207   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/718.png           {"atom_count": 6, "selfies": "[N][N][N][C][O][C]"}           NNNCOC             {"atom_count": 6, "selfies": "[C][O][C][N][N][=C]"}     COCNN=C    0.347826
169  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3574.png             {"atom_count": 5, "selfies": "[O][O][N][=C][O]"}           OON=CO             {"atom_count": 6, "selfies": "[O][C][=N][N][O][O]"}     OC=NNOO    0.350000
218  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4168.png            {"atom_count": 5, "selfies": "[O][N][C][#C][=O]"}           ONC#CO            {"atom_count": 6, "selfies": "[O][N][C][#C][N][=O]"}    ONC#CN=O    0.368421
11   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7228.png            {"atom_count": 5, "selfies": "[N][C][C][#C][=N]"}           NCC#CN            {"atom_count": 6, "selfies": "[N][=C][C][#C][C][N]"}    N=CC#CCN    0.368421
269   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/600.png            {"atom_count": 5, "selfies": "[N][#C][O][C][#C]"}          N#COC#C            {"atom_count": 6, "selfies": "[C][C][#C][O][C][#N]"}    CC#COC#N    0.368421
26   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6322.png            {"atom_count": 5, "selfies": "[N][O][C][#C][#C]"}           NOC#CC            {"atom_count": 6, "selfies": "[N][O][C][#C][C][#C]"}    NOC#CC#C    0.368421
64   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4723.png            {"atom_count": 5, "selfies": "[O][N][C][#C][=O]"}           ONC#CO            {"atom_count": 6, "selfies": "[O][=C][C][#C][N][O]"}    O=CC#CNO    0.368421
289  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6017.png        {"atom_count": 6, "selfies": "[N][C][=C][=C][N][=C]"}        NC=C=CN=C           {"atom_count": 6, "selfies": "[N][C][=C][=N][N][=C]"}   NC=C=NN=C    0.380952
99   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4484.png        {"atom_count": 6, "selfies": "[N][N][=C][=C][=C][O]"}        NN=C=C=CO           {"atom_count": 6, "selfies": "[O][N][=C][=C][=N][N]"}   ON=C=C=NN    0.400000
27   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/8054.png             {"atom_count": 5, "selfies": "[N][C][N][C][#N]"}           NCNC#N             {"atom_count": 6, "selfies": "[N][#C][C][N][C][N]"}     N#CCNCN    0.400000
151  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/5350.png            {"atom_count": 5, "selfies": "[O][=N][N][C][#C]"}          O=NNC#C            {"atom_count": 6, "selfies": "[O][=N][N][C][#C][C]"}    O=NNC#CC    0.400000
217  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1425.png            {"atom_count": 5, "selfies": "[C][N][=C][N][=C]"}          CN=CN=C            {"atom_count": 6, "selfies": "[C][N][=C][N][=N][C]"}    CN=CN=NC    0.411765
20    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/135.png             {"atom_count": 5, "selfies": "[O][O][C][#C][C]"}           OOC#CC             {"atom_count": 6, "selfies": "[C][C][C][#C][O][O]"}     CCC#COO    0.421053
252  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4206.png             {"atom_count": 5, "selfies": "[N][C][#C][O][O]"}           NC#COO             {"atom_count": 6, "selfies": "[O][N][O][C][#C][N]"}     ONOC#CN    0.421053
71   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/5426.png        {"atom_count": 6, "selfies": "[O][=N][N][=N][N][=O]"}        O=NN=NN=O           {"atom_count": 6, "selfies": "[O][=N][N][=N][C][=O]"}   O=NN=NC=O    0.428571
100  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2213.png        {"atom_count": 6, "selfies": "[N][=C][C][=C][N][=C]"}        N=CC=CN=C           {"atom_count": 6, "selfies": "[C][=N][C][=C][N][=C]"}   C=NC=CN=C    0.466667
60   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2402.png          {"atom_count": 6, "selfies": "[N][N][N][N][N][=C]"}          NNNNN=C             {"atom_count": 6, "selfies": "[C][=N][N][N][N][C]"}     C=NNNNC    0.473684
70   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1059.png           {"atom_count": 6, "selfies": "[N][N][N][C][N][C]"}           NNNCNC              {"atom_count": 6, "selfies": "[C][N][C][N][N][C]"}      CNCNNC    0.500000
273  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/8250.png          {"atom_count": 6, "selfies": "[N][N][O][N][C][#N]"}          NNONC#N             {"atom_count": 6, "selfies": "[N][#C][N][O][N][C]"}     N#CNONC    0.500000
275  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1772.png          {"atom_count": 6, "selfies": "[N][N][N][O][C][=C]"}          NNNOC=C             {"atom_count": 6, "selfies": "[C][=C][O][N][N][C]"}     C=CONNC    0.500000
93   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/8017.png          {"atom_count": 6, "selfies": "[N][N][C][C][C][#N]"}          NNCCC#N             {"atom_count": 6, "selfies": "[N][#C][C][C][N][C]"}     N#CCCNC    0.500000
162  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2753.png        {"atom_count": 6, "selfies": "[N][#C][N][N][=C][=C]"}        N#CNN=C=C           {"atom_count": 6, "selfies": "[C][#C][N][N][=C][=C]"}   C#CNN=C=C    0.500000
165  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/8089.png         {"atom_count": 6, "selfies": "[N][N][C][=C][C][#N]"}         NNC=CC#N            {"atom_count": 6, "selfies": "[N][#C][C][=C][N][C]"}    N#CC=CNC    0.500000
295  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1388.png         {"atom_count": 6, "selfies": "[N][N][=C][O][C][=O]"}         NN=COC=O            {"atom_count": 6, "selfies": "[C][N][=C][O][C][=O]"}    CN=COC=O    0.500000
270  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2036.png    {"atom_count": 5, "selfies": "[N][=C][=C][Ring1][Ring1]"}          N1=C=C1   {"atom_count": 6, "selfies": "[C][=C][=C][=N][Ring1][Ring2]"}   C1=C=C=N1    0.500000
55   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1958.png         {"atom_count": 6, "selfies": "[N][N][O][C][=C][=C]"}         NNOC=C=C            {"atom_count": 6, "selfies": "[C][=C][=C][O][N][C]"}    C=C=CONC    0.523810
131  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1006.png           {"atom_count": 6, "selfies": "[N][N][C][C][O][C]"}           NNCCOC              {"atom_count": 6, "selfies": "[C][N][C][C][O][C]"}      CNCCOC    0.526316
111  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7141.png             {"atom_count": 5, "selfies": "[N][=C][C][O][O]"}           N=CCOO             {"atom_count": 6, "selfies": "[N][=C][C][O][O][O]"}     N=CCOOO    0.555556
222  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/8174.png            {"atom_count": 5, "selfies": "[N][#C][O][O][#C]"}           N#COOC            {"atom_count": 6, "selfies": "[N][#C][O][O][C][#N]"}    N#COOC#N    0.666667
 
``` 

结果分析: 

错误用例76个, 原子数错误数35/76

# 从裸模型中, 选取长度6的1000数据, 与v146 (长度6的1000数据) 进行比较

选取:

```
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/length_6.ms_swift.train.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/length_6.enhanced.ms_swift.train.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/length_6.ms_swift.val.json \
--eval_steps 100 \
--learning_rate 4e-4 \
--num_train_epochs 10 \
--logging_steps 20 \
" > train.log 2>&1
``` 

训练: 

```
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/selected_data.length_1_4.500.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/selected_data.length_5.500.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/selected_data.length_6.1000.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/length_6.ms_swift.val.json \
--eval_steps 20 \
--learning_rate 4e-4 \
--num_train_epochs 10 \
--logging_steps 20 \
" > train.log 2>&1
``` 

![image2024-12-5 22:23:41.png](/assets/01KJBZN5YYTD5QQ8Z1D621Z88K/image2024-12-5%2022%3A23%3A41.png)

比起v166, 多500个长度6的数据, 性能更好

比起v146, 未添加atom_count的版本, 性能可能不可比 (训练数据的输出格式变长)

评估: 使用best_model: /opt/huangyan/ms-swift/output/qwen2-vl-7b-instruct/v170-20241205-170005/checkpoint-840

```
Correct: 248 / 300 = 82.66666666666667 %
                                                          Image Path                                                         Predicted Predicted SMILES                                                              GT   GT SMILES  Similarity
164    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/31.png  {"atom_count": 6, "selfies": "[N][Branch1][C][C][Ring1][Ring2]"}               NC      {"atom_count": 6, "selfies": "[C][C][C][C][Ring1][Ring2]"}      C1CCC1    0.000000
3    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1028.png         {"atom_count": 6, "selfies": "[N][Branch1][C][C][C][=C]"}          N(C)C=C      {"atom_count": 6, "selfies": "[C][N][C][C][Ring1][Ring2]"}      C1NCC1    0.000000
230  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1428.png         {"atom_count": 6, "selfies": "[N][Branch1][C][C][C][=N]"}          N(C)C=N     {"atom_count": 6, "selfies": "[C][N][=C][N][Ring1][Ring2]"}     C1N=CN1    0.000000
270  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2036.png        {"atom_count": 6, "selfies": "[N][Branch1][C][C][=C][=C]"}         N(C)=C=C   {"atom_count": 6, "selfies": "[C][=C][=C][=N][Ring1][Ring2]"}   C1=C=C=N1    0.000000
229   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/144.png      {"atom_count": 6, "selfies": "[C][Branch1][C][C][Ring1][C]"}               CC     {"atom_count": 6, "selfies": "[C][C][C][#C][Ring1][Ring2]"}     C1CC#C1    0.000000
226  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7206.png         {"atom_count": 6, "selfies": "[N][Branch1][C][C][C][=C]"}          N(C)C=C    {"atom_count": 6, "selfies": "[N][=C][C][=C][Ring1][Ring2]"}    N1=CC=C1    0.000000
105  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3335.png               {"atom_count": 6, "selfies": "[N][N][=C][C][O][N]"}          NN=CCON      {"atom_count": 6, "selfies": "[O][C][N][N][Ring1][Ring1]"}      OC1NN1    0.000000
208  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2605.png                 {"atom_count": 4, "selfies": "[O][C][Ring1][=C]"}              O=C     {"atom_count": 6, "selfies": "[C][#C][C][Ring1][Ring1][O]"}     C1#CC1O    0.000000
203  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3210.png              {"atom_count": 5, "selfies": "[O][=C][C][Ring1][O]"}             O=CC      {"atom_count": 6, "selfies": "[O][C][O][C][Ring1][Ring1]"}      OC1OC1    0.000000
122  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/5720.png                 {"atom_count": 5, "selfies": "[O][=C][C][=C][N]"}          O=CC=CN      {"atom_count": 6, "selfies": "[N][C][O][C][Ring1][Ring1]"}      NC1OC1    0.052632
295  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1388.png              {"atom_count": 6, "selfies": "[C][=O][O][C][=N][C]"}              C=O            {"atom_count": 6, "selfies": "[C][N][=C][O][C][=O]"}    CN=COC=O    0.058824
72    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/834.png               {"atom_count": 6, "selfies": "[C][=O][O][O][O][C]"}              C=O             {"atom_count": 6, "selfies": "[C][O][O][O][C][=O]"}     COOOC=O    0.062500
290   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/480.png          {"atom_count": 5, "selfies": "[C][N][=C][Ring1][Ring1]"}           C1N=C1    {"atom_count": 6, "selfies": "[C][C][=C][=N][Ring1][Ring1]"}    CC1=C=N1    0.066667
34   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3179.png          {"atom_count": 5, "selfies": "[O][=C][C][Ring1][Ring1]"}             O=CC     {"atom_count": 6, "selfies": "[O][C][C][Ring1][Ring1][=O]"}     O1CC1=O    0.071429
78    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/149.png          {"atom_count": 5, "selfies": "[O][=C][C][Ring1][Ring1]"}             O=CC     {"atom_count": 6, "selfies": "[C][C][C][Ring1][Ring1][=O]"}     C1CC1=O    0.083333
217  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1425.png              {"atom_count": 6, "selfies": "[N][=N][N][=C][N][C]"}         N=NN=CNC            {"atom_count": 6, "selfies": "[C][N][=C][N][=N][C]"}    CN=CN=NC    0.125000
66   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4588.png          {"atom_count": 6, "selfies": "[N][Branch1][C][C][O][N]"}           N(C)ON        {"atom_count": 6, "selfies": "[O][N][Branch1][C][N][C]"}      ON(N)C    0.133333
277  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4584.png          {"atom_count": 6, "selfies": "[N][Branch1][C][C][O][N]"}           N(C)ON        {"atom_count": 6, "selfies": "[O][N][Branch1][C][C][N]"}      ON(C)N    0.133333
163  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7250.png      {"atom_count": 6, "selfies": "[N][=C][C][=C][Ring1][Ring2]"}         N1=CC=C1    {"atom_count": 6, "selfies": "[N][=C][C][Ring1][Ring1][=N]"}    N1=CC1=N    0.142857
210  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2759.png      {"atom_count": 6, "selfies": "[N][N][=C][=C][Ring1][Ring2]"}         N1N=C=C1     {"atom_count": 6, "selfies": "[C][#C][N][N][Ring1][Ring2]"}     C1#CNN1    0.166667
180  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/5442.png                     {"atom_count": 4, "selfies": "[C][O][C][#C]"}            COC#C       {"atom_count": 6, "selfies": "[O][Branch1][C][C][C][=C]"}     O(C)C=C    0.200000
114  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3653.png               {"atom_count": 6, "selfies": "[O][N][C][#C][C][O]"}          ONC#CCO             {"atom_count": 6, "selfies": "[O][C][#C][N][N][O]"}     OC#CNNO    0.238095
225   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/313.png              {"atom_count": 6, "selfies": "[C][N][C][=C][C][#C]"}         CNC=CC#C            {"atom_count": 6, "selfies": "[C][C][N][=C][C][#C]"}    CCN=CC#C    0.240000
269   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/600.png                  {"atom_count": 5, "selfies": "[N][#C][O][C][C]"}           N#COCC            {"atom_count": 6, "selfies": "[C][C][#C][O][C][#N]"}    CC#COC#N    0.285714
89   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1446.png                 {"atom_count": 5, "selfies": "[N][N][=C][=C][C]"}          NN=C=CC           {"atom_count": 6, "selfies": "[C][N][=C][=C][=C][C]"}   CN=C=C=CC    0.285714
90   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/5990.png                {"atom_count": 5, "selfies": "[N][C][=C][=C][=O]"}         NC=C=C=O           {"atom_count": 6, "selfies": "[N][C][=C][=C][N][=O]"}   NC=C=CN=O    0.285714
285  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6007.png             {"atom_count": 6, "selfies": "[N][C][=C][=C][N][=O]"}        NC=C=CN=O           {"atom_count": 6, "selfies": "[N][C][=C][=N][C][=O]"}   NC=C=NC=O    0.304348
21   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2494.png                 {"atom_count": 5, "selfies": "[N][C][#C][C][#C]"}          NC#CC#C            {"atom_count": 6, "selfies": "[C][#C][C][C][#C][N]"}    C#CCC#CN    0.315789
296  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1753.png               {"atom_count": 6, "selfies": "[N][O][O][O][C][=C]"}          NOOOC=C             {"atom_count": 6, "selfies": "[C][=C][O][O][N][C]"}     C=COONC    0.318182
292   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/310.png              {"atom_count": 6, "selfies": "[C][C][N][=C][C][#C]"}         CCN=CC#C            {"atom_count": 6, "selfies": "[C][C][N][=C][C][=C]"}    CCN=CC=C    0.347826
7    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1584.png                 {"atom_count": 5, "selfies": "[C][=C][C][C][=N]"}          C=CCC=N            {"atom_count": 6, "selfies": "[C][=C][C][C][=N][C]"}    C=CCC=NC    0.350000
70   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1059.png                {"atom_count": 6, "selfies": "[C][N][C][C][N][C]"}           CNCCNC              {"atom_count": 6, "selfies": "[C][N][C][N][N][C]"}      CNCNNC    0.357143
240  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6641.png             {"atom_count": 6, "selfies": "[N][N][C][=C][=C][=N]"}        NNC=C=C=N            {"atom_count": 6, "selfies": "[N][N][C][=C][=N][N]"}    NNC=C=NN    0.363636
169  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3574.png               {"atom_count": 6, "selfies": "[O][C][N][N][=C][O]"}          OCNN=CO             {"atom_count": 6, "selfies": "[O][C][=N][N][O][O]"}     OC=NNOO    0.363636
11   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7228.png                 {"atom_count": 5, "selfies": "[N][C][C][#C][=N]"}           NCC#CN            {"atom_count": 6, "selfies": "[N][=C][C][#C][C][N]"}    N=CC#CCN    0.368421
88    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/570.png                 {"atom_count": 5, "selfies": "[N][#C][C][C][#C]"}          N#CCC#C            {"atom_count": 6, "selfies": "[C][C][#C][C][C][#N]"}    CC#CCC#N    0.368421
127  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/5923.png                 {"atom_count": 5, "selfies": "[N][C][=C][C][#C]"}          NC=CC#C            {"atom_count": 6, "selfies": "[N][C][=C][C][#C][C]"}    NC=CC#CC    0.368421
26   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6322.png                  {"atom_count": 5, "selfies": "[N][O][C][#C][C]"}           NOC#CC            {"atom_count": 6, "selfies": "[N][O][C][#C][C][#C]"}    NOC#CC#C    0.368421
151  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/5350.png                 {"atom_count": 5, "selfies": "[O][=N][N][C][#C]"}          O=NNC#C            {"atom_count": 6, "selfies": "[O][=N][N][C][#C][C]"}    O=NNC#CC    0.400000
52   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3360.png              {"atom_count": 6, "selfies": "[O][C][N][=C][=C][C]"}         OCN=C=CC            {"atom_count": 6, "selfies": "[O][C][N][=C][=N][C]"}    OCN=C=NC    0.409091
172  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4606.png                 {"atom_count": 5, "selfies": "[O][=C][C][N][=O]"}          O=CCN=O            {"atom_count": 6, "selfies": "[O][=C][C][C][N][=O]"}    O=CCCN=O    0.444444
48   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2448.png             {"atom_count": 6, "selfies": "[C][=N][N][=N][N][=C]"}        C=NN=NN=C           {"atom_count": 6, "selfies": "[C][=N][N][=N][C][=C]"}   C=NN=NC=C    0.461538
77   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6665.png               {"atom_count": 6, "selfies": "[N][C][C][#C][C][N]"}          NCC#CCN             {"atom_count": 6, "selfies": "[N][N][C][#C][C][N]"}     NNC#CCN    0.466667
80    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/845.png               {"atom_count": 6, "selfies": "[O][O][O][O][N][=O]"}          OOOON=O             {"atom_count": 6, "selfies": "[C][O][O][O][N][=O]"}     COOON=O    0.473684
266  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/8393.png                  {"atom_count": 5, "selfies": "[N][N][N][N][=N]"}           NNNN=N       {"atom_count": 6, "selfies": "[N][Branch1][C][N][N][=N]"}     N(N)N=N    0.500000
194  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4687.png                {"atom_count": 5, "selfies": "[O][=C][=C][C][=O]"}         O=C=CC=O          {"atom_count": 6, "selfies": "[O][=C][C][=C][=C][=O]"}  O=CC=C=C=O    0.500000
258  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1995.png                {"atom_count": 5, "selfies": "[C][=C][=C][C][=C]"}         C=C=CC=C          {"atom_count": 6, "selfies": "[C][=C][=C][=C][C][=C]"}  C=C=C=CC=C    0.500000
167  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4934.png              {"atom_count": 6, "selfies": "[N][O][C][C][=C][=C]"}         NOCC=C=C            {"atom_count": 6, "selfies": "[O][=C][=C][C][O][N]"}    O=C=CCON    0.523810
25   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2562.png                {"atom_count": 5, "selfies": "[O][=C][=C][C][#C]"}         O=C=CC#C          {"atom_count": 6, "selfies": "[C][#C][C][=C][=C][=O]"}  C#CC=C=C=O    0.529412
239  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/8008.png                  {"atom_count": 5, "selfies": "[N][C][C][C][#N]"}           NCCC#N             {"atom_count": 6, "selfies": "[N][#C][C][C][C][N]"}     N#CCCCN    0.625000
0    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/5995.png                {"atom_count": 5, "selfies": "[N][C][=C][=C][=C]"}         NC=C=C=C          {"atom_count": 6, "selfies": "[N][C][=C][=C][=C][=C]"}  NC=C=C=C=C    0.625000
40    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/455.png                {"atom_count": 5, "selfies": "[C][C][=C][=C][=C]"}         CC=C=C=C          {"atom_count": 6, "selfies": "[C][C][=C][=C][=C][=C]"}  CC=C=C=C=C    0.625000
190  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3791.png              {"atom_count": 6, "selfies": "[O][O][C][=N][C][=C]"}         OOC=NC=C            {"atom_count": 6, "selfies": "[O][O][C][=N][C][=C]"}    OOC=NC=C    1.000000
``` 

对比v151 (长度6的数据为2000个), 正确率89%. 看不出增加atom_count的效果是否正常.

需要将 长度6的数据增加到2000个

# 需要将 长度6的数据增加到2000个

选取数据+训练:

```
cp /opt/huangyan/ms-swift/swift/llm/sft.py.modified2 /opt/huangyan/ms-swift/swift/llm/sft.py

bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/length_6.ms_swift.train.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/length_6.enhanced.ms_swift.train.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/length_6.ms_swift.val.json \
--eval_steps 100 \
--learning_rate 4e-4 \
--num_train_epochs 10 \
--logging_steps 20 \
" > train_2.log 2>&1

sleep 60

cp /opt/huangyan/ms-swift/swift/llm/sft.py.raw /opt/huangyan/ms-swift/swift/llm/sft.py

bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/selected_data.length_1_4.500.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/selected_data.length_5.500.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/selected_data.length_6.2000.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out/length_6.ms_swift.val.json \
--eval_steps 100 \
--learning_rate 4e-4 \
--num_train_epochs 10 \
--logging_steps 20 \
" > train_3.log 2>&1

``` 

模型版本: best model: /opt/huangyan/ms-swift/output/qwen2-vl-7b-instruct/v187-20241206-114917/checkpoint-1100

对比 v170 (长度6的数据1000个), v166(长度6的数据500个), v146 (长度6的数据1000, 输出不包括atom_count) 

![image2024-12-6 15:1:9.png](/assets/01KJBZN5YYTD5QQ8Z1D621Z88K/image2024-12-6%2015%3A1%3A9.png)

评估数据:

比起v170, 提高5%

与v151 (长度6的数据2000, 长度5的数据多500个, 输出不包括atom_count, 其他条件相等) 相比, 减弱2%

但当前版本的长度错误是10个, v151是28个. 

```
Correct: 261 / 300 = 87.0 %
                                                          Image Path                                                                                 Predicted Predicted SMILES                                                              GT   GT SMILES  Similarity
122  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/5720.png                                         {"atom_count": 5, "selfies": "[C][=C][=C][N][O]"}          C=C=CNO      {"atom_count": 6, "selfies": "[N][C][O][C][Ring1][Ring1]"}      NC1OC1    0.000000
208  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2605.png                               {"atom_count": 6, "selfies": "[C][Ring1][Ring1][C][O][=C]"}             CCOC     {"atom_count": 6, "selfies": "[C][#C][C][Ring1][Ring1][O]"}     C1#CC1O    0.000000
164    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/31.png                             {"atom_count": 6, "selfies": "[N][=C][=C][=C][Ring1][Ring2]"}        N1=C=C=C1      {"atom_count": 6, "selfies": "[C][C][C][C][Ring1][Ring2]"}      C1CCC1    0.000000
290   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/480.png                                  {"atom_count": 5, "selfies": "[N][=C][C][Ring1][Ring1]"}           N1=CC1    {"atom_count": 6, "selfies": "[C][C][=C][=N][Ring1][Ring1]"}    CC1=C=N1    0.066667
78    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/149.png                                        {"atom_count": 5, "selfies": "[C][=C][C][=O][=C]"}           C=CC=O     {"atom_count": 6, "selfies": "[C][C][C][Ring1][Ring1][=O]"}     C1CC1=O    0.071429
3    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1028.png                              {"atom_count": 6, "selfies": "[N][N][=C][=C][Ring1][Ring2]"}         N1N=C=C1      {"atom_count": 6, "selfies": "[C][N][C][C][Ring1][Ring2]"}      C1NCC1    0.071429
34   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3179.png                                 {"atom_count": 5, "selfies": "[O][=C][Ring1][Ring1][=O]"}             O=CO     {"atom_count": 6, "selfies": "[O][C][C][Ring1][Ring1][=O]"}     O1CC1=O    0.071429
203  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3210.png                                  {"atom_count": 5, "selfies": "[O][C][=C][Ring1][Ring1]"}           O1C=C1      {"atom_count": 6, "selfies": "[O][C][O][C][Ring1][Ring1]"}      OC1OC1    0.076923
163  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7250.png                                 {"atom_count": 5, "selfies": "[N][=C][=C][Ring1][Ring1]"}          N1=C=C1    {"atom_count": 6, "selfies": "[N][=C][C][Ring1][Ring1][=N]"}    N1=CC1=N    0.142857
226  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7206.png                             {"atom_count": 6, "selfies": "[N][=C][=C][=C][Ring1][Ring2]"}        N1=C=C=C1    {"atom_count": 6, "selfies": "[N][=C][C][=C][Ring1][Ring2]"}    N1=CC=C1    0.153846
69   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/8033.png                                      {"atom_count": 6, "selfies": "[N][#C][C][#C][O][O]"}         N#CC#COO            {"atom_count": 6, "selfies": "[N][#C][C][C][#C][O]"}    N#CCC#CO    0.166667
210  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2759.png                              {"atom_count": 6, "selfies": "[N][N][=C][=C][Ring1][Ring2]"}         N1N=C=C1     {"atom_count": 6, "selfies": "[C][#C][N][N][Ring1][Ring2]"}     C1#CNN1    0.166667
217  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1425.png                                      {"atom_count": 6, "selfies": "[N][=N][C][N][=N][C]"}         N=NCN=NC            {"atom_count": 6, "selfies": "[C][N][=C][N][=N][C]"}    CN=CN=NC    0.181818
230  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1428.png                              {"atom_count": 6, "selfies": "[N][N][=C][=C][Ring1][Ring2]"}         N1N=C=C1     {"atom_count": 6, "selfies": "[C][N][=C][N][Ring1][Ring2]"}     C1N=CN1    0.200000
229   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/144.png                              {"atom_count": 6, "selfies": "[C][=C][=C][C][Ring1][Ring2]"}         C1=C=CC1     {"atom_count": 6, "selfies": "[C][C][C][#C][Ring1][Ring2]"}     C1CC#C1    0.200000
129  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2207.png                                     {"atom_count": 6, "selfies": "[C][=C][O][C][=N][=C]"}         C=COC=NC            {"atom_count": 6, "selfies": "[C][=N][C][=C][O][C]"}    C=NC=COC    0.200000
7    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1584.png                                      {"atom_count": 6, "selfies": "[N][=C][C][C][C][=C]"}         N=CCCC=C            {"atom_count": 6, "selfies": "[C][=C][C][C][=N][C]"}    C=CCC=NC    0.217391
260   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/755.png                                      {"atom_count": 6, "selfies": "[N][=C][=C][O][O][C]"}         N=C=COOC            {"atom_count": 6, "selfies": "[C][O][C][=C][=N][O]"}    COC=C=NO    0.240000
110  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3674.png  {"atom_count": 6, "selfies": "[O][C][Branch1][C][O][C][Branch1][C][O][C][Ring1][Ring1]"}      OC(O)C(O)=C        {"atom_count": 6, "selfies": "[O][C][Branch1][C][O][C]"}      OC(O)C    0.250000
87   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6582.png                                        {"atom_count": 6, "selfies": "[N][N][O][N][N][N]"}           NNONNN              {"atom_count": 6, "selfies": "[N][N][C][O][N][N]"}      NNCONN    0.250000
257   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/844.png                                       {"atom_count": 6, "selfies": "[N][=C][O][O][O][C]"}          N=COOOC             {"atom_count": 6, "selfies": "[C][O][O][O][N][=C]"}     COOON=C    0.272727
120  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/5242.png                                      {"atom_count": 6, "selfies": "[O][N][C][#C][C][#N]"}         ONC#CC#N           {"atom_count": 6, "selfies": "[O][=N][C][#C][C][#N]"}   O=NC#CC#N    0.272727
258  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1995.png                                    {"atom_count": 6, "selfies": "[C][C][=C][=C][=C][=C]"}       CC=C=C=C=C          {"atom_count": 6, "selfies": "[C][=C][=C][=C][C][=C]"}  C=C=C=CC=C    0.285714
239  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/8008.png                                       {"atom_count": 6, "selfies": "[N][C][C][C][#C][N]"}          NCCC#CN             {"atom_count": 6, "selfies": "[N][#C][C][C][C][N]"}     N#CCCCN    0.285714
88    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/570.png                                         {"atom_count": 5, "selfies": "[N][#C][C][#C][C]"}          N#CC#CC            {"atom_count": 6, "selfies": "[C][C][#C][C][C][#N]"}    CC#CCC#N    0.315789
89   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1446.png                                    {"atom_count": 6, "selfies": "[N][=C][=C][=C][=C][C]"}       N=C=C=C=CC           {"atom_count": 6, "selfies": "[C][N][=C][=C][=C][C]"}   CN=C=C=CC    0.333333
296  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1753.png                                       {"atom_count": 6, "selfies": "[C][N][O][O][O][=C]"}           CNOOOC             {"atom_count": 6, "selfies": "[C][=C][O][O][N][C]"}     C=COONC    0.333333
186  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3795.png                                      {"atom_count": 6, "selfies": "[O][O][C][=N][N][#N]"}         OOC=NN=N            {"atom_count": 6, "selfies": "[O][O][C][=N][C][#N]"}    OOC=NC#N    0.347826
141  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4493.png                                        {"atom_count": 5, "selfies": "[O][N][=C][=N][#N]"}          ON=C=NN           {"atom_count": 6, "selfies": "[O][N][=C][=N][C][#N]"}   ON=C=NC#N    0.368421
199  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4315.png                                        {"atom_count": 6, "selfies": "[O][N][N][O][O][O]"}           ONNOOO              {"atom_count": 6, "selfies": "[O][N][N][N][O][O]"}      ONNNOO    0.368421
26   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6322.png                                         {"atom_count": 5, "selfies": "[N][O][C][#C][#C]"}           NOC#CC            {"atom_count": 6, "selfies": "[N][O][C][#C][C][#C]"}    NOC#CC#C    0.368421
255  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2357.png                                      {"atom_count": 6, "selfies": "[N][=C][C][N][N][=C]"}         N=CCNN=C            {"atom_count": 6, "selfies": "[C][=N][N][C][N][=C]"}    C=NNCN=C    0.380952
191  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4429.png                                      {"atom_count": 6, "selfies": "[N][=N][O][C][=N][O]"}         N=NOC=NO            {"atom_count": 6, "selfies": "[O][N][=C][O][N][=O]"}    ON=CON=O    0.500000
93   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/8017.png                                       {"atom_count": 6, "selfies": "[N][N][C][C][C][#N]"}          NNCCC#N             {"atom_count": 6, "selfies": "[N][#C][C][C][N][C]"}     N#CCCNC    0.500000
162  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2753.png                                     {"atom_count": 6, "selfies": "[N][#C][N][N][=C][=C]"}        N#CNN=C=C           {"atom_count": 6, "selfies": "[C][#C][N][N][=C][=C]"}   C#CNN=C=C    0.500000
165  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/8089.png                                      {"atom_count": 6, "selfies": "[N][N][C][=C][C][#N]"}         NNC=CC#N            {"atom_count": 6, "selfies": "[N][#C][C][=C][N][C]"}    N#CC=CNC    0.500000
273  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/8250.png                                       {"atom_count": 6, "selfies": "[N][N][O][N][C][#N]"}          NNONC#N             {"atom_count": 6, "selfies": "[N][#C][N][O][N][C]"}     N#CNONC    0.500000
55   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1958.png                                      {"atom_count": 6, "selfies": "[N][N][O][C][=C][=C]"}         NNOC=C=C            {"atom_count": 6, "selfies": "[C][=C][=C][O][N][C]"}    C=C=CONC    0.523810
130  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7935.png                                      {"atom_count": 6, "selfies": "[N][=C][N][N][N][=N]"}         N=CNNN=N            {"atom_count": 6, "selfies": "[N][=N][N][N][C][=O]"}    N=NNNC=O    0.526316
205  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6043.png                                      {"atom_count": 6, "selfies": "[N][C][=N][C][=C][O]"}         NC=NC=CO            {"atom_count": 6, "selfies": "[N][C][=N][C][=C][O]"}    NC=NC=CO    1.000000

``` 

# 结论

从评估来说, 增加atom_count, 提高了"原子数正确"的概率, 但整体错误率没有上升 (转向了其他错误)

更加更多分子式描述, 继续修正
