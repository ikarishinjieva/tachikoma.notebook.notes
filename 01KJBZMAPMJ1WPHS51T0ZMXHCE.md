---
title: 20241128 - 使用ms-swift对Qwen2-VL进行微调 [9] - 用梯度贡献来进行数据集的选择
confluence_page_id: 3343295
created_at: 2024-11-28T16:11:23+00:00
updated_at: 2024-12-02T05:23:59+00:00
---

修改 /opt/huangyan/ms-swift/swift/llm/sft.py

程序: [sft.py](/assets/01KJBZMAPMJ1WPHS51T0ZMXHCE/sft.py)

主要修改: 增加train_with_gradient_selection方法, 在一个epoch 遍历所有数据, 选择100个 梯度贡献最大的数据, 加入下一个epoch的训练数据

# 选取梯度贡献最大的300个

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

选取了300个数据, 然后对length 1-4数据进行训练和评估

训练 (需要将sft.py换回来): 

```
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/selected_data.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4.ms_swift.val.json \
--eval_steps 20 \
--learning_rate 4e-4 \
--num_train_epochs 10 \
--logging_steps 20 \
--loss_name test_loss \
" > train.log 2>&1
``` 

qwen2-vl-7b-instruct/v111-20241128-232743/runs

与v87对比, v87包括长度1-4的全部穷举数据和增强数据 (500+500条左右). v111选取了其中梯度贡献最大的300个

![image2024-11-29 10:24:9.png](/assets/01KJBZMAPMJ1WPHS51T0ZMXHCE/image2024-11-29%2010%3A24%3A9.png)

结论: 看上去是 过拟合了, train/loss收敛更快, 但eval/loss中途爆炸 (好像是warmup部分)

尝试扩大epoch=40

```
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/selected_data.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4.ms_swift.val.json \
--eval_steps 20 \
--learning_rate 4e-4 \
--num_train_epochs 10 \
--logging_steps 20 \
--loss_name test_loss \
" > train.log 2>&1
``` 

qwen2-vl-7b-instruct/v112-20241129-102559/runs

![image2024-11-29 13:31:20.png](/assets/01KJBZMAPMJ1WPHS51T0ZMXHCE/image2024-11-29%2013%3A31%3A20.png)

过拟合出现得很早 (在高LR中找到了一个方向, 不像v111一样产生梯度爆炸, 但没法继续优化), v112的train/loss收敛也更快

# 选取梯度贡献最大的500个

修改代码, 选取500个记录

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

用选取的数据进行训练

```
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/selected_data.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4.ms_swift.val.json \
--eval_steps 20 \
--learning_rate 4e-4 \
--num_train_epochs 10 \
--logging_steps 20 \
--loss_name test_loss \
" > train.log 2>&1
``` 

qwen2-vl-7b-instruct/v114-20241129-140003/runs

![image2024-11-29 14:19:16.png](/assets/01KJBZMAPMJ1WPHS51T0ZMXHCE/image2024-11-29%2014%3A19%3A16.png)

仍没有到过拟合. 

\------

扩大epoch = 30

qwen2-vl-7b-instruct/v115-20241129-142005/runs

![image2024-11-29 16:59:32.png](/assets/01KJBZMAPMJ1WPHS51T0ZMXHCE/image2024-11-29%2016%3A59%3A32.png)

loss 比 v114 更高, v114在LR最高点找到了方向, 怀疑v115需要在LR最高点停留更久才能找到方向, 尝试调高LR

![image2024-11-29 21:35:48.png](/assets/01KJBZMAPMJ1WPHS51T0ZMXHCE/image2024-11-29%2021%3A35%3A48.png)

虽然loss不下降, 但acc会上升

\---

调高LR=8e-4, epoch=10

```
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/selected_data.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4.ms_swift.val.json \
--eval_steps 20 \
--learning_rate 8e-4 \
--num_train_epochs 10 \
--logging_steps 20 \
--loss_name test_loss \
" > train.log 2>&1
``` 

![image2024-11-29 23:13:40.png](/assets/01KJBZMAPMJ1WPHS51T0ZMXHCE/image2024-11-29%2023%3A13%3A40.png)

延长epoch=30, v118-20241129-231953

![image2024-11-30 11:0:22.png](/assets/01KJBZMAPMJ1WPHS51T0ZMXHCE/image2024-11-30%2011%3A0%3A22.png)

从step 640往后开始过拟合. 用step 640的结果进行评估

```
Correct: 73 / 75 = 97.33333333333334 %
                                                        Image Path        Predicted Predicted SMILES               GT GT SMILES  Similarity
40  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/366.png    [C][C][C][#N]            CCC#N   [N][#C][C][=C]    N#CC=C         0.2
62   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/53.png   [C][=N][C][#C]           C=NC#C    [C][N][C][#C]     CNC#C         0.2
0   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/348.png    [N][=N][C][N]            N=NCN    [N][=N][C][N]     N=NCN         1.0
52   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/69.png   [N][=C][=N][C]           N=C=NC   [C][N][=C][=N]    CN=C=N         1.0
51  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/129.png    [C][#C][N][N]            C#CNN    [C][#C][N][N]     C#CNN         1.0
``` 

# 将选取500数据时, 每轮间跑的epoch调大

![image2024-11-30 15:25:3.png](/assets/01KJBZMAPMJ1WPHS51T0ZMXHCE/image2024-11-30%2015%3A25%3A3.png)

![image2024-11-30 15:22:14.png](/assets/01KJBZMAPMJ1WPHS51T0ZMXHCE/image2024-11-30%2015%3A22%3A14.png)

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

数据文件名: /opt/huangyan/LLaMA-Factory/data/loop_selfies/selected_data.epoch_2.500.json

进行训练: 

```
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/selected_data.epoch_2.500.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4.ms_swift.val.json \
--eval_steps 20 \
--learning_rate 4e-4 \
--num_train_epochs 10 \
--logging_steps 20 \
--loss_name test_loss \
" > train_1.log 2>&1
``` 

训练效果: v122-20241130-164254

![image2024-11-30 22:12:26.png](/assets/01KJBZMAPMJ1WPHS51T0ZMXHCE/image2024-11-30%2022%3A12%3A26.png)

eval/loss 会比 间隔epoch=1时的效果 (v114)更好

扩大 训练epoch=30, LR=8e-4, 效果: v123-20241130-170527

![image2024-11-30 22:16:11.png](/assets/01KJBZMAPMJ1WPHS51T0ZMXHCE/image2024-11-30%2022%3A16%3A11.png)

并没有比 间隔epoch=1是的效果 (v118) 更好

# 每轮筛选50条记录, 多筛选几轮

![image2024-11-30 15:59:26.png](/assets/01KJBZMAPMJ1WPHS51T0ZMXHCE/image2024-11-30%2015%3A59%3A26.png)

数据文件名: /opt/huangyan/LLaMA-Factory/data/loop_selfies/selected_data.iterate_10.json

训练:

```
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/selected_data.iterate_10.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4.ms_swift.val.json \
--eval_steps 20 \
--learning_rate 4e-4 \
--num_train_epochs 10 \
--logging_steps 20 \
--loss_name test_loss \
" > train_3.log 2>&1

``` 

效果: v124-20241130-180140

![image2024-11-30 22:17:59.png](/assets/01KJBZMAPMJ1WPHS51T0ZMXHCE/image2024-11-30%2022%3A17%3A59.png)

效果没有提升

  
扩大 训练epoch=30, LR=8e-4, 效果: v125-20241130-182404

![image2024-11-30 22:20:31.png](/assets/01KJBZMAPMJ1WPHS51T0ZMXHCE/image2024-11-30%2022%3A20%3A31.png)

效果也没有提升

# 在长度1-3的微调结果上, 进行数据选择

首先训练长度1-3的模型: 

```
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_1.ms_swift.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_1.enhanced.ms_swift.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_2.ms_swift.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_2.enhanced.ms_swift.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_3.ms_swift.train.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/length_3.enhanced.ms_swift.train.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_3.ms_swift.val.json \
--eval_steps 20 \
--learning_rate 4e-4 \
--num_train_epochs 10 \
--logging_steps 20 \
--loss_name test_loss \
" > train.log 2>&1
``` 

获取最好的checkpoint: /opt/huangyan/ms-swift/output/qwen2-vl-7b-instruct/v127-20241130-225432/checkpoint-40

merge lora:

```
CUDA_VISIBLE_DEVICES=0 swift export \
    --ckpt_dir '/opt/huangyan/ms-swift/output/qwen2-vl-7b-instruct/v127-20241130-225432/checkpoint-40' --merge_lora true
``` 

获得模型: /opt/huangyan/ms-swift/output/qwen2-vl-7b-instruct/v127-20241130-225432/checkpoint-40-merged

在其上进行数据选择, 修改sft.py, 选取300个数据 (总共有300+300个数据): 

```
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path /opt/huangyan/ms-swift/output/qwen2-vl-7b-instruct/v127-20241130-225432/checkpoint-40-merged \
--sft_type lora \
--dataset \
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

数据文件: /opt/huangyan/LLaMA-Factory/data/loop_selfies/selected_data.length_4.300.json

用这个数据文件进行训练: 

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
/opt/huangyan/LLaMA-Factory/data/loop_selfies/selected_data.length_4.300.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4.ms_swift.val.json \
--eval_steps 20 \
--learning_rate 4e-4 \
--num_train_epochs 10 \
--logging_steps 20 \
--loss_name test_loss \
" > train.log 2>&1
``` 

训练效果: v131-20241130-232234

![image2024-11-30 23:38:17.png](/assets/01KJBZMAPMJ1WPHS51T0ZMXHCE/image2024-11-30%2023%3A38%3A17.png)

放大LR和epoch: 

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
/opt/huangyan/LLaMA-Factory/data/loop_selfies/selected_data.length_4.300.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4.ms_swift.val.json \
--eval_steps 20 \
--learning_rate 8e-4 \
--num_train_epochs 30 \
--logging_steps 20 \
--loss_name test_loss \
" > train.log 2>&1
``` 

qwen2-vl-7b-instruct/v132-20241130-234425/runs

![image2024-12-1 21:22:30.png](/assets/01KJBZMAPMJ1WPHS51T0ZMXHCE/image2024-12-1%2021%3A22%3A30.png)

比v131 (没放大LR和epoch) 的效果差不多

# 总结

v87: 包括长度1-4的全部穷举数据和增强数据 (500+500条左右)  
v118: 选取梯度最大的 500数据, LR=8e-4, epoch=30  
v123: 选取梯度最大的 500数据, 每轮间跑的epoch调大为2, LR=8e-4, epoch=30  
v125: 选取梯度最大的 500数据, 每轮筛选50条记录, 多筛选几轮, LR=8e-4, epoch=30  
v132: 在长度1-3的微调结果上, 进行数据选择了300数据, 总训练数据是长度1/2/3加选择出的300数据

![image2024-12-1 21:34:30.png](/assets/01KJBZMAPMJ1WPHS51T0ZMXHCE/image2024-12-1%2021%3A34%3A30.png)

效果最好的是v118, 但也敌不过v87的数据量的优势. 

有一个猜测, v118比起其他几组数据, 都是选取了对于裸模型来说梯度最高的数据. 

再做一个尝试, 增加从裸模型选取的数据

# 增加从裸模型选取的数据

![image2024-12-1 21:44:29.png](/assets/01KJBZMAPMJ1WPHS51T0ZMXHCE/image2024-12-1%2021%3A44%3A29.png)

获取数据:

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

(去除--loss_name test_loss, 为失误, 但不影响之前结果)

训练: 

```
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/selected_data.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4.ms_swift.val.json \
--eval_steps 20 \
--learning_rate 4e-4 \
--num_train_epochs 10 \
--logging_steps 20 \
--loss_name test_loss \
" > train.log 2>&1
``` 

![image2024-12-1 22:42:50.png](/assets/01KJBZMAPMJ1WPHS51T0ZMXHCE/image2024-12-1%2022%3A42%3A50.png)

效果比较明显. 增大LR和epoch再训练:

```
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/selected_data.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4.ms_swift.val.json \
--eval_steps 20 \
--learning_rate 8e-4 \
--num_train_epochs 30 \
--logging_steps 20 \
--loss_name test_loss \
" > train.log 2>&1
``` 

![image2024-12-2 10:57:46.png](/assets/01KJBZMAPMJ1WPHS51T0ZMXHCE/image2024-12-2%2010%3A57%3A46.png)

效果没有提升

不调整LR, 延长epoch, 再尝试: 

```
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/selected_data.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4.ms_swift.val.json \
--eval_steps 20 \
--learning_rate 4e-4 \
--num_train_epochs 30 \
--logging_steps 20 \
--loss_name test_loss \
" > train.log 2>&1
``` 

eval/loss没有改善, 但eval/acc上升 ([20241115 - ms-swift是如何计算acc的] 按照这个分析, 是sentences/token级别的预测完全正确的概率在变高, "置信度"在变高)

![image2024-12-2 12:55:9.png](/assets/01KJBZMAPMJ1WPHS51T0ZMXHCE/image2024-12-2%2012%3A55%3A9.png)

# 结论

  - 从裸模型选择梯度贡献最大的数据, (相比从部分训练的模型中选择数据), 目前效果最好
  - 数据量仍然是关键指标, 数据选取 仍然无法比肩 大数据量的效果
