---
title: 20241112 - 使用ms-swift对Qwen2-VL进行微调 [1]
confluence_page_id: 3343065
created_at: 2024-11-12T11:16:56+00:00
updated_at: 2024-12-27T10:49:40+00:00
---

环境配置

```
conda create -n ms-swift
conda activate ms-swift
ALL_PROXY=http://127.0.0.1:7890 git clone https://github.com/modelscope/ms-swift.git
 
cd ms-swift
ALL_PROXY=http://127.0.0.1:7890 pip install packaging
ALL_PROXY=http://127.0.0.1:7890 pip install -e .[llm]
 
ALL_PROXY=http://127.0.0.1:7890 pip install git+https://github.com/huggingface/transformers.git
ALL_PROXY=http://127.0.0.1:7890 pip install pyav qwen_vl_utils torchvision
 
CUDA_VISIBLE_DEVICES=0 swift infer --model_type qwen2-vl-7b-instruct
 
# swift infer --model_type qwen2_vl --model /home/user01/model_from_huangyan
``` 

使用: 

```
conda activate ms-swift
 
pip install deepspeed torchvision
 
CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
  --model_type qwen2-vl-7b-instruct \
  --model_id_or_path qwen/Qwen2-VL-7B-Instruct \
  --sft_type lora \
  --dataset /opt/huangyan/LLaMA-Factory/data/decimer-img2smiles.cot.ms-swift.json \
  --dataset_test_ratio 0.01 \
  --eval_steps 20 \
  --logging_steps 20
 
``` 

# 配置deepspeed? 

```
CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
  --model_type qwen2-vl-7b-instruct \
  --model_id_or_path qwen/Qwen2-VL-7B-Instruct \
  --sft_type lora \
  --dataset /opt/huangyan/LLaMA-Factory/data/decimer-img2smiles.cot.ms-swift.json \
  --deepspeed default-zero2
``` 

# 微调Qwen2-VL

```
screen
 
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
  --model_type qwen2-vl-7b-instruct \
  --model_id_or_path qwen/Qwen2-VL-7B-Instruct \
  --sft_type lora \
  --dataset /opt/huangyan/LLaMA-Factory/data/decimer-img2smiles.cot.ms-swift.json \
  --dataset_test_ratio 0.01 \
  --eval_steps 20 \
  --learning_rate 4e-4 \
  --num_train_epochs 5 \
  --logging_steps 20" > train.log 2>&1
``` 

效果: 

"(eval/loss)|(train/loss)|learning|grad"

qwen2-vl-7b-instruct/v0-20241112-203829/runs

![image2024-11-13 0:40:13.png](/assets/01KJBZJT9YFKP1XPAKC0PHG4GA/image2024-11-13%200%3A40%3A13.png)

先解决过拟合的问题, 调整weight-decay和dropout (上一例的配置: "weight_decay": 0.1, lora_dropout:0.05):

```
 bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
>   --model_type qwen2-vl-7b-instruct \
>   --model_id_or_path qwen/Qwen2-VL-7B-Instruct \
>   --sft_type lora \
>   --dataset /opt/huangyan/LLaMA-Factory/data/decimer-img2smiles.cot.ms-swift.json \
>   --dataset_test_ratio 0.01 \
>   --eval_steps 20 \
>   --learning_rate 4e-4 \
>   --num_train_epochs 5 \
>   --logging_steps 20 \
>   --weight_decay 0.2 \
>   --lora_dropout 0.1 \
>   " > train.log 2>&1
``` 

效果: 

qwen2-vl-7b-instruct/v1-20241113-005532/runs

![image2024-11-13 11:32:26.png](/assets/01KJBZJT9YFKP1XPAKC0PHG4GA/image2024-11-13%2011%3A32%3A26.png)

曲线与之前没有变化

再增加一倍: 

```
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
>   --model_type qwen2-vl-7b-instruct \
>   --model_id_or_path qwen/Qwen2-VL-7B-Instruct \
>   --sft_type lora \
>   --dataset /opt/huangyan/LLaMA-Factory/data/decimer-img2smiles.cot.ms-swift.json \
>   --dataset_test_ratio 0.01 \
>   --eval_steps 20 \
>   --learning_rate 4e-4 \
>   --num_train_epochs 5 \
>   --logging_steps 20 \
>   --weight_decay 0.4 \
>   --lora_dropout 0.2 \
>   " > train.log 2>&1
``` 

效果:

qwen2-vl-7b-instruct/v2-20241113-113424/runs

![image2024-11-13 13:24:9.png](/assets/01KJBZJT9YFKP1XPAKC0PHG4GA/image2024-11-13%2013%3A24%3A9.png)

曲线没有变化

降低lora_rank为4, 观察过拟合是否缓解: 

```
 bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset /opt/huangyan/LLaMA-Factory/data/decimer-img2smiles.cot.ms-swift.json \
--dataset_test_ratio 0.01 \
--eval_steps 20 \
--learning_rate 4e-4 \
--num_train_epochs 5 \
--logging_steps 20 \
--weight_decay 0.4 \
--lora_dropout 0.2 \
--lora_rank 4 \
" > train.log 2>&1
``` 

效果: 

qwen2-vl-7b-instruct/v3-20241113-134101/runs

![image2024-11-13 14:52:47.png](/assets/01KJBZJT9YFKP1XPAKC0PHG4GA/image2024-11-13%2014%3A52%3A47.png)

曲线仍没有变化

尝试限制微调的lora层个数: 

如果曲线改变了：

1\. 过拟合减轻（train/eval loss差距变小）：  
\- 证明之前确实是参数过多导致的过拟合  
\- 最后几层足以完成任务适配  
\- 这是最理想的情况

2\. 性能下降（两个loss都变高）：  
\- 说明任务可能需要更多层的调整  
\- 或者我们限制的层选择不够恰当  
\- 需要考虑增加层数或调整层的选择

如果曲线不变：  
1\. 说明问题可能不在模型结构：  
\- 即使限制了可训练参数，模型仍然可以"记住"数据  
\- 数据的多样性或质量可能是主要问题  
\- 或者验证集（0.01）太小，不足以反映真实分布

下一步建议：  
1\. 如果曲线改变：根据改变方向调整策略  
2\. 如果曲线不变：  
\- 优先检查数据问题  
\- 增加验证集比例  
\- 检查数据质量和分布

你觉得先试试哪个方向？

模型结构: 视觉层32个vision block, 语言层一共28层(每层包括self_attn注意力模块和mlp前馈网络):

```
[INFO:swift] PeftModelForCausalLM(
  (base_model): LoraModel(
    (model): Qwen2VLForConditionalGeneration(
      (visual): Qwen2VisionTransformerPretrainedModel(
        (patch_embed): PatchEmbed(
          (proj): Conv3d(3, 1280, kernel_size=(2, 14, 14), stride=(2, 14, 14), bias=False)
        )
        (rotary_pos_emb): VisionRotaryEmbedding()
        (blocks): ModuleList(
          (0-31): 32 x Qwen2VLVisionBlock(
            (norm1): LayerNorm((1280,), eps=1e-06, elementwise_affine=True)
            (norm2): LayerNorm((1280,), eps=1e-06, elementwise_affine=True)
            (attn): VisionSdpaAttention(
              (qkv): Linear(in_features=1280, out_features=3840, bias=True)
              (proj): Linear(in_features=1280, out_features=1280, bias=True)
            )
            (mlp): VisionMlp(
              (fc1): Linear(in_features=1280, out_features=5120, bias=True)
              (act): QuickGELUActivation()
              (fc2): Linear(in_features=5120, out_features=1280, bias=True)
            )
          )
        )
        (merger): PatchMerger(
          (ln_q): LayerNorm((1280,), eps=1e-06, elementwise_affine=True)
          (mlp): Sequential(
            (0): Linear(in_features=5120, out_features=5120, bias=True)
            (1): GELU(approximate='none')
            (2): Linear(in_features=5120, out_features=3584, bias=True)
          )
        )
      )
      (model): Qwen2VLModel(
        (embed_tokens): Embedding(152064, 3584)
        (layers): ModuleList(
          (0-27): 28 x Qwen2VLDecoderLayer(
            (self_attn): Qwen2VLSdpaAttention(
              (q_proj): lora.Linear(
                (base_layer): Linear(in_features=3584, out_features=3584, bias=True)
                (lora_dropout): ModuleDict(
                  (default): Dropout(p=0.2, inplace=False)
                )
                (lora_A): ModuleDict(
                  (default): Linear(in_features=3584, out_features=4, bias=False)
                )
                (lora_B): ModuleDict(
                  (default): Linear(in_features=4, out_features=3584, bias=False)
                )
                (lora_embedding_A): ParameterDict()
                (lora_embedding_B): ParameterDict()
                (lora_magnitude_vector): ModuleDict()
              )
              (k_proj): lora.Linear(
                (base_layer): Linear(in_features=3584, out_features=512, bias=True)
                (lora_dropout): ModuleDict(
                  (default): Dropout(p=0.2, inplace=False)
                )
                (lora_A): ModuleDict(
                  (default): Linear(in_features=3584, out_features=4, bias=False)
                )
                (lora_B): ModuleDict(
                  (default): Linear(in_features=4, out_features=512, bias=False)
                )
                (lora_embedding_A): ParameterDict()
                (lora_embedding_B): ParameterDict()
                (lora_magnitude_vector): ModuleDict()
              )
              (v_proj): lora.Linear(
                (base_layer): Linear(in_features=3584, out_features=512, bias=True)
                (lora_dropout): ModuleDict(
                  (default): Dropout(p=0.2, inplace=False)
                )
                (lora_A): ModuleDict(
                  (default): Linear(in_features=3584, out_features=4, bias=False)
                )
                (lora_B): ModuleDict(
                  (default): Linear(in_features=4, out_features=512, bias=False)
                )
                (lora_embedding_A): ParameterDict()
                (lora_embedding_B): ParameterDict()
                (lora_magnitude_vector): ModuleDict()
              )
              (o_proj): lora.Linear(
                (base_layer): Linear(in_features=3584, out_features=3584, bias=False)
                (lora_dropout): ModuleDict(
                  (default): Dropout(p=0.2, inplace=False)
                )
                (lora_A): ModuleDict(
                  (default): Linear(in_features=3584, out_features=4, bias=False)
                )
                (lora_B): ModuleDict(
                  (default): Linear(in_features=4, out_features=3584, bias=False)
                )
                (lora_embedding_A): ParameterDict()
                (lora_embedding_B): ParameterDict()
                (lora_magnitude_vector): ModuleDict()
              )
              (rotary_emb): Qwen2VLRotaryEmbedding()
            )
            (mlp): Qwen2MLP(
              (gate_proj): lora.Linear(
                (base_layer): Linear(in_features=3584, out_features=18944, bias=False)
                (lora_dropout): ModuleDict(
                  (default): Dropout(p=0.2, inplace=False)
                )
                (lora_A): ModuleDict(
                  (default): Linear(in_features=3584, out_features=4, bias=False)
                )
                (lora_B): ModuleDict(
                  (default): Linear(in_features=4, out_features=18944, bias=False)
                )
                (lora_embedding_A): ParameterDict()
                (lora_embedding_B): ParameterDict()
                (lora_magnitude_vector): ModuleDict()
              )
              (up_proj): lora.Linear(
                (base_layer): Linear(in_features=3584, out_features=18944, bias=False)
                (lora_dropout): ModuleDict(
                  (default): Dropout(p=0.2, inplace=False)
                )
                (lora_A): ModuleDict(
                  (default): Linear(in_features=3584, out_features=4, bias=False)
                )
                (lora_B): ModuleDict(
                  (default): Linear(in_features=4, out_features=18944, bias=False)
                )
                (lora_embedding_A): ParameterDict()
                (lora_embedding_B): ParameterDict()
                (lora_magnitude_vector): ModuleDict()
              )
              (down_proj): lora.Linear(
                (base_layer): Linear(in_features=18944, out_features=3584, bias=False)
                (lora_dropout): ModuleDict(
                  (default): Dropout(p=0.2, inplace=False)
                )
                (lora_A): ModuleDict(
                  (default): Linear(in_features=18944, out_features=4, bias=False)
                )
                (lora_B): ModuleDict(
                  (default): Linear(in_features=4, out_features=3584, bias=False)
                )
                (lora_embedding_A): ParameterDict()
                (lora_embedding_B): ParameterDict()
                (lora_magnitude_vector): ModuleDict()
              )
              (act_fn): SiLU()
            )
            (input_layernorm): Qwen2RMSNorm((3584,), eps=1e-06)
            (post_attention_layernorm): Qwen2RMSNorm((3584,), eps=1e-06)
          )
        )
        (norm): Qwen2RMSNorm((3584,), eps=1e-06)
        (rotary_emb): Qwen2VLRotaryEmbedding()
      )
      (lm_head): Linear(in_features=3584, out_features=152064, bias=False)
    )
  )
)

``` 

只微调最后三层: 

```
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset /opt/huangyan/LLaMA-Factory/data/decimer-img2smiles.cot.ms-swift.json \
--dataset_test_ratio 0.01 \
--eval_steps 20 \
--learning_rate 4e-4 \
--num_train_epochs 5 \
--logging_steps 20 \
--weight_decay 0.4 \
--lora_dropout 0.2 \
--lora_rank 4 \
--target_regex 'model\.layers\.(25|26|27)\..*' \
" > train.log 2>&1
``` 

效果: qwen2-vl-7b-instruct/v4-20241113-154311/runs

![image2024-11-13 17:53:50.png](/assets/01KJBZJT9YFKP1XPAKC0PHG4GA/image2024-11-13%2017%3A53%3A50.png)

loss整体曲线升高, 过拟合现象变轻, 符合预期

尝试多一层, 并且增加dataset_test_ratio, 查看loss曲线趋势和过拟合趋势: 

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
--logging_steps 20 \
--weight_decay 0.4 \
--lora_dropout 0.2 \
--lora_rank 4 \
--target_regex 'model\.layers\.(24|25|26|27)\..*' \
" > train.log 2>&1 
``` 

效果: 

qwen2-vl-7b-instruct/v5-20241113-231527/runs

![image2024-11-14 8:25:11.png](/assets/01KJBZJT9YFKP1XPAKC0PHG4GA/image2024-11-14%208%3A25%3A11.png)

loss曲线整体下移 (符合预期), 过拟合的情况也减轻 (需要增加一个对比测试)

增加对比测试, 同样条件, 将dataset_test_ratio设置回0.01:

效果:

qwen2-vl-7b-instruct/v6-20241114-082358/runs

![image2024-11-14 9:40:34.png](/assets/01KJBZJT9YFKP1XPAKC0PHG4GA/image2024-11-14%209%3A40%3A34.png)

确实dataset_test_ratio会干扰到过拟合的程度

WebUI测试效果: 

```
CUDA_VISIBLE_DEVICES=0 swift web-ui
``` 

# 测试结果

测试训练数据3:

GT: ![3.png](/assets/01KJBZJT9YFKP1XPAKC0PHG4GA/3.png) TEST: ![image2024-11-14 10:34:6.png](/assets/01KJBZJT9YFKP1XPAKC0PHG4GA/image2024-11-14%2010%3A34%3A6.png) (中间的连接键和右二苯环上的O 都有一些问题)

```
GT:
分析:
1. 步骤一：分析了图片的左下角部分，看到一个苯环（C6H5）结构，带有一个-OCH3（甲氧基）取代基。对此部分的特征是一个带有甲氧基的苯环，对应的smiles是“COc1cccc”，因为“CO”表示甲氧基，后面的“c1cccc”表示这个苯环。
 
2. 步骤二：分析了图片中间部分，发现一个含有硫酸（-SO2）结构的部分，与一个氮原子相连。这里的特征是一个-NS(=O)(=O)的结构，对应的smiles是“NS(=O)(=O)”，因为N连接到两个双键的O。
 
3. 步骤三：分析了图片的右中部，发现另一个苯环，与前面的-(NS(=O)(=O))相连。这个苯环有一个取代基，如氮上的取代基。对此部分，对应的smiles是“c2ccc”，对应下一个苯环的结构。
 
4. 步骤四：分析了图片的右侧，上部有一个氧与氮相连的结构，形成一个含O和N的基团。其结构对应的smiles是“(N)”，标记为-NH。
 
5. 步骤五：分析了图片的右上角，有一个含有-OCH和-N的环状结构。这个部分的smiles为“-c3coc(C)n3”，表示一个含氮的环和醇基。
 
最终，将所有部分拼接在一起，形成完整的smiles表达式。
 
完整的smiles表达式是: COc1cccc(NS(=O)(=O)c2ccc(-c3coc(C)n3)cc2)c1
  
------
TEST:
分析:
步骤一: 分析图片左侧的部分，看到一个甲氧基（-OCH₃）结构。特征是有一个氧原子连接到一个碳原子上，形成一个甲基的连接。对应的smiles是CO，因为甲氧基在分子中通常表示为C与O的结合。

步骤二: 分析中间部分，识别出苯环（c1ccc...cc1）。特征是有六个碳原子形成的环状结构，并且有取代基。对应的smiles是c1ccc(...)cc1，其中省略号代表后续的取代基。

步骤三: 继续分析中间部分，发现有一个磺酰胺基团（-S(=O)(=O)N-），特征是一个硫原子连接两个氧原子和一个氮原子。对应的smiles是S(=O)(=O)N，表示这个基团的结构。

步骤四: 分析右侧部分，识别出另一个苯环及其取代基（-c2ccc(...)o2）。特征同样是六个碳原子形成的环，带有取代基。对应的smiles是c2ccc(...)o2，同样省略号代表后续的取代基。

步骤五: 最后，分析整个分子的连接方式，将各个部分拼接在一起。根据化学键的连接关系，最终组合成完整的smiles表达式。

完整的smiles表达式是: COc1ccc(S(=O)(=O)Nc2ccc(-c3coc(C)n3)o2)cc1
``` 

测试训练数据113:

GT: ![113.png](/assets/01KJBZJT9YFKP1XPAKC0PHG4GA/113.png) TEST: ![image2024-11-14 10:43:39.png](/assets/01KJBZJT9YFKP1XPAKC0PHG4GA/image2024-11-14%2010%3A43%3A39.png)

````
GT:
```
分析:
1. 当前步骤分析了图片的左侧部分，该部分是一个带有氟和硝基取代基的苯环结构。特征包括氟原子的存在和相应的硝基基团。对应的smiles是 `c(ccc(c1)N(=O)(=O)c1)F`，这里的`N(=O)(=O)`代表硝基，`c`表示芳香环。
 
2. 当前步骤分析了图片的中心部分，这是一座连接两个苯环的部分。特征是连接两个芳香环的单键。对应的smiles是 `c2c`，表示这是从一个苯环到另一个苯环的过渡。
 
3. 当前步骤分析了右侧部分，这是一部分含有醚基的苯环并带有羟基。特征是存在醚和羟基。该部分对应的smiles是 `Oc3ccc(...)`，这里的`O`表示醚基。
 
4. 当前步骤分析了连接的苯环上有一个酮基。特征是酮基的双键氧。对应的smiles是 `c(=O)`，表示酮基的氢被替换。
 
5. 当前步骤分析了整个分子的左下角部分和氟的排列。整体结构里有三个氟原子，齐齐出现在苯环上。对应的smiles是 `C(F)(F)`，表示该取代基。
 
最终组合得到的smiles表达式是：
 
完整的smiles表达式是: Cc1cc(=O)oc2cc(Oc3ccc([N+](=O)[O-])cc3C(F)(F)F)ccc12
```
 
------
分析:
步骤一: 分析图片左侧的部分，识别出一个甲氧基（-OCH₃）结构。特征是有一个氧原子连接到一个碳原子上，形成一个甲氧基。对应的smiles是 CO，拼接到最终的smiles表达式中。

步骤二: 继续分析左侧的苯环部分，发现其上有两个取代基，一个是甲氧基，另一个是一个硝基（-NO₂）。特征是苯环的平面结构和取代基的位置。对应的smiles是 c1cc(...)ccc1，其中包含取代基的信息，拼接为 c1cc(OC)ccc1。

步骤三: 向右移动，看到一个连接的链段，识别出一个醚键（-O-）连接到下一个苯环。对应的smiles是 O，拼接为 c1cc(OC)c(O...)ccc1。

步骤四: 分析右侧的苯环，发现它是一个带有羟基（-OH）和一个酰胺基团（-C(=O)N-）的结构。特征是苯环上的这些官能团。对应的smiles是 c2ccc3c(c2)oc(=O)n3C，拼接为 c1cc(OC)c(Oc2ccc3c(c2)oc(=O)n3C)ccc1。

步骤五: 最后，分析分子中的氟原子及其连接方式，确认有两个全氟取代基。对应的smiles是 C(F)(F)F，拼接为 c1cc(OC)c(Oc2ccc3c(c2)oc(=O)n3C)ccc1C(F)(F)F。

完整的smiles表达式是: COc1cc(OC)c(Oc2ccc3c(c2)oc(=O)n3C)ccc1C(F)(F)F
```` 

测试陌生数据2: GT: ![from-paper-2.png](/assets/01KJBZJT9YFKP1XPAKC0PHG4GA/from-paper-2.png) TEST: ![image2024-11-14 10:46:11.png](/assets/01KJBZJT9YFKP1XPAKC0PHG4GA/image2024-11-14%2010%3A46%3A11.png)

测试陌生数据5: GT: ![from-paper-5.png](/assets/01KJBZJT9YFKP1XPAKC0PHG4GA/from-paper-5.png) TEST:![image2024-11-14 10:50:13.png](/assets/01KJBZJT9YFKP1XPAKC0PHG4GA/image2024-11-14%2010%3A50%3A13.png)

# 考虑使用更多数据

[20241114 - 使用ms-swift对Qwen2-VL进行微调 [3] - 使用更多数据]

# 考虑调整loss

[20241114 - 使用ms-swift对Qwen2-VL进行微调 [4] - 修改loss]

# 考虑更换SELFIES 代替smiles

[20241114 - 使用ms-swift对Qwen2-VL进行微调 [2] - 使用SELFIES替代SMILES]

# 解决过拟合

[20241115 - 使用ms-swift对Qwen2-VL进行微调 [5] - 解决过拟合状态]

  - 进行数据增值

# 考虑建立重点数据测试集

# 考虑进行数据正则化

增加训练数据的多样性确实是减轻过拟合的有效方法。以下是一些具体建议：

1\. 评估现有数据质量的方法：  
```python  
# 1. 计算样本与模型预测的困惑度(perplexity)  
def calculate_perplexity(model, text):  
with torch.no_grad():  
outputs = model(text)  
return torch.exp(outputs.loss).item()

# 2. 计算样本间的相似度，找出重复或近似的样本  
from sentence_transformers import SentenceTransformer  
def find_similar_samples(texts, threshold=0.9):  
model = SentenceTransformer('all-MiniLM-L6-v2')  
embeddings = model.encode(texts)  
similarities = cosine_similarity(embeddings)  
return np.where(similarities > threshold)  
```

2\. 评估新增数据价值的方法：

```python  
# 1. 使用小批量数据进行验证  
def validate_new_data(model, new_data):  
# 在现有验证集上的性能  
base_metrics = evaluate_model(model, val_data)  
  
# 加入新数据后的性能  
combined_val = combine_data(val_data, new_data)  
new_metrics = evaluate_model(model, combined_val)  
  
return {  
'performance_delta': new_metrics - base_metrics,  
'diversity_score': calculate_diversity(new_data)  
}

# 2. 计算数据多样性分数  
def calculate_diversity(data):  
# 可以考虑以下维度:  
# - 输入长度分布  
# - 词汇覆盖率  
# - 主题分布  
# - 分子结构的复杂度(针对你的化学数据)  
pass  
```

3\. 数据筛选策略：  
\- 移除困惑度特别高的样本(可能是噪声)  
\- 去除高度重复的样本  
\- 保留在验证集上能提升性能的样本  
\- 确保分子结构的多样性

4\. 持续评估：  
```python  
# 在训练过程中监控样本的影响  
def monitor_sample_impact(model, data_loader):  
sample_impacts = {}  
for batch in data_loader:  
# 记录每个样本对loss的贡献  
loss = model(batch).loss  
gradients = torch.autograd.grad(loss)  
sample_impacts[batch_id] = {  
'loss': loss.item(),  
'gradient_norm': torch.norm(gradients).item()  
}  
return sample_impacts  
```

要我详细展开任何一个方面吗？比如具体如何实现分子结构多样性的评估，或者如何设计更细致的数据筛选策略？
