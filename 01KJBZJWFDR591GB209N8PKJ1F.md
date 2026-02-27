---
title: 20241114 - 使用ms-swift对Qwen2-VL进行微调 [3] - 使用更多数据
confluence_page_id: 3343115
created_at: 2024-11-14T07:07:29+00:00
updated_at: 2024-11-15T07:33:55+00:00
---

从 [20241112 - 使用ms-swift对Qwen2-VL进行微调 [1]] 的v0版本出发, 进行如下调整: 

增加校验集比例到0.05: 

```
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
  --model_type qwen2-vl-7b-instruct \
  --model_id_or_path qwen/Qwen2-VL-7B-Instruct \
  --sft_type lora \
  --dataset /opt/huangyan/LLaMA-Factory/data/decimer-img2smiles.cot.ms-swift.json \
  --dataset_test_ratio 0.05 \
  --eval_steps 20 \
  --learning_rate 4e-4 \
  --num_train_epochs 5 \
  --logging_steps 20" > train.log 2>&1
``` 

数据量调整为6000: qwen2-vl-7b-instruct/v8-20241114-151036/runs

![image2024-11-14 17:41:20.png](/assets/01KJBZJWFDR591GB209N8PKJ1F/image2024-11-14%2017%3A41%3A20.png)

training_loss最终差不多, 过拟合从step 900开始, 过拟合程度下降 (可能是因为dataset_test_ratio的调高)

数据量调整为8000: qwen2-vl-7b-instruct/v10-20241115-015532/runs

过拟合仍从step 900开始, train/eval loss的值都没有什么变化, loss下降周期拉长 (随着LR的变化周期拉长)

![image2024-11-15 11:14:22.png](/assets/01KJBZJWFDR591GB209N8PKJ1F/image2024-11-15%2011%3A14%3A22.png)

数据量调整为10000: qwen2-vl-7b-instruct/v11-20241115-112555/runs

![image2024-11-15 15:31:24.png](/assets/01KJBZJWFDR591GB209N8PKJ1F/image2024-11-15%2015%3A31%3A24.png)

过拟合从step 600开始, train/eval loss的值都没有什么变化, loss下降周期拉长 (随着LR的变化周期拉长)

增加数据量对loss改善并没有什么作用??
