---
title: 20241119 - 使用ms-swift对Qwen2-VL进行微调 [6] - 增强Selfies的效果
confluence_page_id: 3343208
created_at: 2024-11-19T12:42:04+00:00
updated_at: 2024-11-21T11:43:09+00:00
---

[20241115 - 使用ms-swift对Qwen2-VL进行微调 [5] - 解决过拟合状态]

目前的状态: 没有找到解决过拟合的做法

在上一个实验中, 进行了小数据集下 smiles和selfies的效果对比, 发现selfies生成的表达式中的非法表达式比较多. 

想通过 修改loss函数, 同时进行selfies表达式生成 和 合法性 检查: 

```
# ... other code ...

# 定义特殊标记 (注意：这里不需要 "是" 和 "否" 了)
tokenizer.add_special_tokens({'additional_special_tokens': ['[EXP]', '[VALID]', '[INVALID]']})
model.resize_token_embeddings(len(tokenizer))

# 数据处理
def process_data(image, smiles, is_valid):
    prefix = "[VALID]" if is_valid else "[INVALID]"
    input_text = f"你的任务是识别以下图片, 逐步生成对应的smiles表达式: {image}"
    output_text = f"{prefix} smiles表达式: [EXP]{smiles}[EXP]" #  这里去掉了 "是" 和 "否"
    encoding = tokenizer(input_text, output_text, return_tensors="pt", padding=True, truncation=True)
    return encoding

# ... training loop ...

for batch in dataloader:
    # ...

    outputs = model(input_ids=input_ids, attention_mask=attention_mask, labels=labels)
    smiles_loss = outputs.loss

    # 获取 validity prediction
    validity_logits = outputs.logits[:, 1, [tokenizer.convert_tokens_to_ids('[VALID]'), tokenizer.convert_tokens_to_ids('[INVALID]') ]] # 取第二个token (第一个是'[VALID]'或'[INVALID]') 的logits

    # 计算 validity loss
    validity_labels = torch.tensor([1 if x['is_valid'] else 0 for x in batch]).to(device) # 1 for [VALID], 0 for [INVALID]
    validity_loss = torch.nn.CrossEntropyLoss()(validity_logits, validity_labels)

    total_loss = alpha * smiles_loss + (1 - alpha) * validity_loss
    # ...
``` 

从register_loss_func入手, 但其中不能进行decode (trainer.tokenizer.decode), 需要传入tokenizer:

![image2024-11-19 20:50:22.png](/assets/01KJBZKTQRNA64HW30T7QG691S/image2024-11-19%2020%3A50%3A22.png)

```
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset /opt/huangyan/LLaMA-Factory/data/decimer-img2smiles.cot.ms-swift.data-1000.SELFIES.train.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/decimer-img2smiles.cot.ms-swift.data-1000.SELFIES.val.json \
--eval_steps 20 \
--learning_rate 8e-5 \
--num_train_epochs 10 \
--logging_steps 20 \
--loss_name test_loss \
" > train.log 2>&1
``` 

loss函数: [附件: loss.1.py] 

对Selfies表达式后段非法的部分进行惩罚

基线: v31-20241119-120543

效果: qwen2-vl-7b-instruct/v75-20241120-174030/runs (惩罚系数 5.0)

![image2024-11-21 0:47:26.png](/assets/01KJBZKTQRNA64HW30T7QG691S/image2024-11-21%200%3A47%3A26.png)

对比v31, 改造loss后的版本 train/loss 走势类似, eval/loss会更高

评估结果: 

[evaluate_smiles_selfies.ipynb](/assets/01KJBZKTQRNA64HW30T7QG691S/evaluate_smiles_selfies.ipynb)

v31基线: 

```
  Filename                                                                                                                                                                                                                                          Predicted                               Predicted SMILES                                                True SMILES  Similarity
0    6.png                                                       [C][O][C][=C][C][=C][C][=C][Ring1][=Branch1][C][C][C][N][S][Ring1][=Branch2][C][=C][C][=C][C][=C][Ring1][=Branch1][C][S][C][=C][C][=C][Ring1][=Branch1][C][=O][O][N+1][=Branch1][C][=O][O-1]   COC1=CC2=CC=C1CCCNS2C3=CC=CC=C3C4SC=CC=C4C=O  COc1ccc(C2Sc3cc(Cc4cccs4)ccc3N(CC[NH+](C)C)CC2OC(C)=O)cc1    0.128440
1    3.png  [O][=C][Branch2][Ring2][Ring1][C][=C][C][=C][Branch1][S][C][=Branch1][C][=O][C][C][C][Ring1][Ring1][Ring1][C][Branch1][C][C][N+1][=Branch1][C][=O][O-1][=C][Ring1][C][C][C][C][=C][C][=C][C][=C][C][=C][Ring1][=Branch1][=C][Ring1][Ring1][Ring2]  O=C(C=CC=C(C(=O)C1C=C1CC[N+1])C2=O)C=CC3=C2C3                   CCCCCC[NH+]1CCC(C(=O)CC)(c2cccc(O)c2)CC1    0.088608
2    1.png                                                                                                                                                                                                                                               None                                           None                                                       None    0.000000
3    2.png                                                                                                                                                                                    [N][N][C][=N][Branch1][=C][N][C][=C][Branch1][O][C][C][C][C][C]                                NNC=NCNC=CCCCCC                               CCCOc1nc(NN)nc(N(CC)C(C)C)n1    0.120690
4    7.png                                   [C][C][O][C][=C][C][=C][C][=C][Ring1][Branch2][Ring1][#Branch2][N][C][=Branch1][C][=O][C][C][C][=Branch1][C][=O][N][C][=C][C][=C][Branch1][C][O][C][=C][Ring1][#Branch2][O][C][C][O][C][=C][C][=C][C][=C][Ring1]                               C1C2OC=CC=CC=C21            CCCCOc1ccc(-c2[nH]ncc2C(=O)NCc2ccc(OC)cc2OC)cc1    0.027778
5    5.png                                                                                                                                                                                         [C][C][Branch1][C][F][S][C][C][C][C][N+1][=Branch1][C][=O]                              CC(F)SCCCC[N+1]=O                                             [NH3+]CCCSCCCF    0.142857
6    4.png                                                                                                            [O][=C][Branch1][O][C][C][N][C][C][C][C][C][Branch2][Ring1][=Branch1][C][=O][C][C][=Branch1][C][=O][N][C][=N][N][=N][Ring1][Branch1][C]                                 O=CCCNCCCCCC=O                                   Cn1cc(C(=O)N2CCCC2CO)nn1    0.037736
7    9.png                                                        [C][C][Branch1][C][F][Branch1][C][F][F][N][C][C][C][=C][C][=C][C][=C][Ring1][Branch1][=Branch1][C][=O][O][C][C][=C][C][=C][C][=C][Ring1][Branch1][N][Branch1][C][N+1][C][=Branch1][O][O][O]                                      CC(F)(F)F          [NH3+]C1CCCC(C(=O)NCC2(c3cccc(C(F)(F)F)c3)CCC2)C1    0.056604
8    8.png                                                                                                   [C][C][=C][Branch2][Ring1][Ring1][N][=N][Branch1][S][C][C][N+1][=Branch1][C][C][=Branch1][C][=O][O-1][C][=C][Ring2][Ring1][Ring1][S][Branch1][C]                        C1C=C1N=NSCC[N+1](C)C=O                                CSCC(C)[NH2+]Cc1nc(C)c(C)s1    0.071429
9   10.png                                                                                                                                                                  [C][O][C][=C][C][=C][Branch2][Ring1][C][N][C][=O][C][N][C][N][C][Ring2][Ring1][N]                                   COC=CC=CNC=O                         O=C(CN1CCC[NH2+]CC1)NCC(O)c1ccccc1    0.051724

``` 

v75 (惩罚系数 5.0)

```
Overall Similarity: 8.09%
  Filename                                                                                                                                                                                                                                         Predicted                                          Predicted SMILES                                                           True SMILES  Similarity
0    6.png                                   [C][O][C][=C][C][=C][C][=C][C][Ring1][=Branch1][C][C][C][C][Branch1][S][C][C][C][C][Ring1][=Branch1][C][=C][C][=C][C][=C][C][Ring1][=Branch1][C][C][S][C][C][=C][C][=C][S][Ring1][#Branch1][=Branch1][C][=O][N]  COC=C1C=CC=CC1CCC2C(CCCC2C=C3C=CC=CC3)CCS4CC=CC=CS4(=O)N             COc1ccc(C2Sc3cc(Cc4cccs4)ccc3N(CC[NH+](C)C)CC2OC(C)=O)cc1    0.082645
1    3.png                                                                                 [O][=C][Ring1][=Branch1][C][=C][C][=C][C][=C][Ring1][=Branch1][C][=O][C][C][C][C][O][Branch1][C][C][C][Branch1][N][C][C][C][C][C][Ring1][Branch1][=Branch1][C][C]                                         O=CC1=CC=CC=C1C=O                              CCCCCC[NH+]1CCC(C(=O)CC)(c2cccc(O)c2)CC1    0.098039
2    1.png               [C][C][=C][C][Branch2][Ring1][=Branch1][N][O][N][C][C][Ring1][Branch4][Branch3][N][C][C][C][Branch3][N+1][=Branch1][C][=O][C][C][O][C][=C][C][=C][C][=C][Ring1][Branch3][S][Ring1][=C][Ring1][=Branch1][C][=C][Ring2][Ring1][Ring1]                       CC=CCN1ONC2=CCOCCOC=C3C=CC#CS23C=C1  CCOC(=O)CCCCN1CC[NH+](Cc2cc3nc(N4CCOC(c5cccc6[nH]ncc56)C4)ncc3s2)CC1    0.085938
3    2.png                                                                           [N][N][C][=N][N][=C][C][=C][C][Branch1][=Branch1][C][=N][C][=C][Ring1][=Branch2][=Branch2][C][Branch1][C][N][C][O][C][C][C][C][=C][Ring1][Branch1][N][Branch1][C][C][C]                   NNC=NN=CC=CC(C=NC=C)(CNCO)C1CCC=C1N(C)C                                          CCCOc1nc(NN)nc(N(CC)C(C)C)n1    0.083333
4    7.png  [C][C][O][C][=C][C][=C][C][=C][Ring1][=Branch1][C][=O][=Branch1][N][C][=Branch1][N][C][C][=C][C][=C][C][=C][Ring1][Branch1][O][C][=C][C][=C][C][=C][Ring1][=Branch1][C][=C][Ring2][Ring1][Ring1][=Branch1][C][=C][Ring1][=Branch1][=C][Ring1][N]                                         CCOC1=CC=CC=C1C=O                       CCCCOc1ccc(-c2[nH]ncc2C(=O)NCc2ccc(OC)cc2OC)cc1    0.149254
5    5.png                                                                                                                                             [C][C][Branch1][C][F][S][=Branch1][C][=O][C][=Branch1][C][=O][C][=N][C][C][N+1][=Branch1][C][=O][O-1]                        CC(F)S(=O)C(=O)C=NCC[N+1](=O)[O-1]                                                        [NH3+]CCCSCCCF    0.041667
6    4.png                                                                                                                             [O][=C][Branch1][Ring1][N][C][C][N][C][N][C][Branch1][=Branch1][C][=O][C][N][N][Branch1][C][N][=N][Ring1][Branch2][C]                                  O=C(NC)CN1CNC(C=O)(N)N1C                                              Cn1cc(C(=O)N2CCCC2CO)nn1    0.126984
7    9.png                                                                 [N][C][=C][C][=C][Branch2][Ring1][=Branch2][C][=O][Branch2][C][C][=Branch1][C][Branch1][C][F][Branch1][C][F][F][N][N][C][Branch1][C][F][C][F][C][F][F][F][C][=C][Ring1][#Branch2]                                             NC=CC=C(C=O)F                     [NH3+]C1CCCC(C(=O)NCC2(c3cccc(C(F)(F)F)c3)CCC2)C1    0.046154
8    8.png                                                                                [C][C][=C][Branch2][Ring2][S][N][C][=Branch1][C][=O][C][C][N][Branch1][C][N+1][=Branch1][C][=O][=Branch1][C][=O][C][=C][Ring1][Branch2][=C][Ring2][Ring1][C][O][S]                                   CC=CNC(=O)CCN([N+1])C=O                                           CSCC(C)[NH2+]Cc1nc(C)c(C)s1    0.036364
9   10.png                                                                                                                                            [C][C][Branch1][Ring1][O][C][C][N][Branch1][C][C][C][O][C][Ring1][=Branch1][N+1][=Branch1][C][=O][O-1]                            CC(OC)C1N(C)COC1[N+1](=O)[O-1]                                    O=C(CN1CCC[NH2+]CC1)NCC(O)c1ccccc1    0.058824

``` 

v76 (惩罚系数 20.0)

```
Overall Similarity: 5.91%
  Filename                                                                                                                                                                                                                   Predicted                                       Predicted SMILES                                                True SMILES  Similarity
0    6.png  [C][O][C][=C][C][=C][Branch2][Ring1][#Branch1][S][C][C][C][C][C][C][Ring1][N][C][=C][C][=C][C][=C][Ring1][=Branch1][C][=C][C][=C][Ring2][Ring1][=Branch1][C][Branch1][C][C][C][N+1][=Branch1][C][=O][O-1][S][Ring2][Ring1]  COC=CC=C(SCCCCCCC1=CC=CC=C1C=CC=C)C(C)C[N+1](=O)[O-1]  COc1ccc(C2Sc3cc(Cc4cccs4)ccc3N(CC[NH+](C)C)CC2OC(C)=O)cc1    0.097345
1    3.png                                                               [O][=C][C][=C][C][=C][C][Branch2][Ring1][#C][C][=O][C][Branch1][=Branch1][N+1][=Branch1][C][C][C][C][C][C][C][C][C][C][C][Ring1][=Branch2][C][N][=O][C][C][C]                                          O=CC=CC=CCC=O                   CCCCCC[NH+]1CCC(C(=O)CC)(c2cccc(O)c2)CC1    0.092593
2    1.png                                                                                                                                                                                                                        None                                                   None                                                       None    0.000000
3    2.png                                                                                                                        [C][N][N][C][=C][N][Branch2][=C][Branch1][=C][Ring1][Branch1][C][C][C][Branch1][C][C][C][N][O][C][C]                                  CNN1C=CNC1CCC(C)CNOCC                               CCCOc1nc(NN)nc(N(CC)C(C)C)n1    0.104478
4    7.png                                            [C][N][=C][Branch1][O][C][=N][N][=C][=Branch1][C][=O][C][N][O][C][=Branch1][C][=O][O][C][C][=C][C][=C][C][=C][Ring1][=Branch1][Branch1][C][O][C][C][N][=C][Branch1][O][C][=N][N]        CN=C(C=NN=C(O)CNO)C(=O)OCC1=CC=CC=C1COCCN=CC=NN            CCCCOc1ccc(-c2[nH]ncc2C(=O)NCc2ccc(OC)cc2OC)cc1    0.156250
5    5.png                                                                                                                                                                                                                        None                                                   None                                                       None    0.000000
6    4.png                                                                                              [O][H][C][C][C][C][C][Branch1][C][C][C][C][C][C][C][C][Ring1][N][C][C][=O][S][C][C][C][=C][C][=C][Branch1][=Branch1][N][Ring1]                                                   O[H]                                   Cn1cc(C(=O)N2CCCC2CO)nn1    0.000000
7    9.png                                                  [C][C][C][Branch1][C][N][C][=C][=O][C][=C][C][=C][C][Branch1][C][C][C][C][Ring1][Branch1][N][C][Branch1][C][C][F][F][F][C][Branch1][N][Branch1][C][C][F][F][F][C][Branch1]                                            CCC(N)C=C=O          [NH3+]C1CCCC(C(=O)NCC2(c3cccc(C(F)(F)F)c3)CCC2)C1    0.030303
8    8.png                                                                                                                                             [C][C][=C][S][N][=C][C][Ring1][C][N][Branch1][C][C][O][C][C][C][S][C][C][Ring1]                                 CC=CSN=C=CN(C)OCCCSC=C                                CSCC(C)[NH2+]Cc1nc(C)c(C)s1    0.046875
9   10.png                                                                                              [O][C][Branch2][C][=Branch1][C][N][C][Branch1][C][=O][C][Branch1][N][N][Branch1][N+1][=Branch1][C][=O][C][=C][C][=N][C][Ring1]                                        OC(CNCO1)C1NC=O                         O=C(CN1CCC[NH2+]CC1)NCC(O)c1ccccc1    0.063492

``` 

随着惩罚系数上升, 并没有发现Selfies表达式的合法率有提高

调大LR和epoch试试: 

```
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset /opt/huangyan/LLaMA-Factory/data/decimer-img2smiles.cot.ms-swift.data-1000.SELFIES.train.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/decimer-img2smiles.cot.ms-swift.data-1000.SELFIES.val.json \
--eval_steps 20 \
--learning_rate 4e-4 \
--num_train_epochs 20 \
--logging_steps 20 \
--loss_name test_loss \
" > train.log 2>&1
``` 

效果: qwen2-vl-7b-instruct/v77-20241121-131910/runs  
  
![image2024-11-21 15:55:16.png](/assets/01KJBZKTQRNA64HW30T7QG691S/image2024-11-21%2015%3A55%3A16.png)

```
Overall Similarity: 9.11%
  Filename                                                                                                                                                                                                                                                                                                                                                                                                                                                                          Predicted                                             Predicted SMILES                                                           True SMILES  Similarity
0    6.png                                                                                                                                                                                                                                                                                                                          [C][O][C][=C][C][=C][Branch2][Ring1][S][=Branch1][C][=O][C][N][C][C][C][=C][C][=C][C][=C][Branch1][=Branch1][C][C][=C][Ring1][=C][Ring2][Ring1][Ring1][C]                                                  COC=CC=CC=O             COc1ccc(C2Sc3cc(Cc4cccs4)ccc3N(CC[NH+](C)C)CC2OC(C)=O)cc1    0.064935
1    3.png                                                                                                                                                                                                                          [C][O][C][=C][C][=C][C][=C][C][=C][Ring1][Branch1][=Branch1][C][=Branch1][C][=O][C][=Branch1][C][=C][C][=C][C][=C][Ring1][Branch1][C][C][C][C][C][C][C][C][C][C][C][C][C][C][C][C][=Branch1][C][=O][O-1][=C][Ring1][=Branch1][C][=C][Ring2][Ring1][Ring1]  COC=CC=C1C=CC=C1C(=O)C(=C2)C=CC=C2CCCCCCCCCCCCCCCC(=O)[O-1]                              CCCCCC[NH+]1CCC(C(=O)CC)(c2cccc(O)c2)CC1    0.156627
2    1.png  [O][=C][Branch1][#C][C][O][N][C][N][C][=C][C][=C][N][Ring1][Branch2][C][=C][Branch1][=Branch1][C][C][=C][Branch1][=Branch1][C][=Branch1][C][=C][C][=C][C][=C][Ring1][Branch1][=Branch1][S][=C][Ring1][=Branch1][N][C][C][C][Ring1][Ring1][Ring1][Ring1][Ring1][Ring1][Ring1][Ring1][Ring1][O][Ring1][Branch1][N][N][C][C][C][C][C][C][Ring1][Ring1][Ring1][Ring1][Ring1][Ring1][Ring1][Ring1][Ring1][Ring1][Ring1][Ring1][Ring1][Ring1][Ring1][Ring1][Ring1][Ring1][Ring1][Ring1]        O=C(CON1CNC=CC=CN1C=C)CC=C(C(=C2)C3)CC=C2S=C3NC=4CC=4  CCOC(=O)CCCCN1CC[NH+](Cc2cc3nc(N4CCOC(c5cccc6[nH]ncc56)C4)ncc3s2)CC1    0.103704
3    2.png                                                                                                                                                                                                                                                                                                                                                                                                                     [N][C][N][Branch2][=N][C][Branch1][=N][C][Branch1][O][C][C][C]                                                     NCNNCCCC                                          CCCOc1nc(NN)nc(N(CC)C(C)C)n1    0.127660
4    7.png                                                                                                                                                                                                                                                                                                                             [C][C][C][C][N][=C][C][=C][C][=C][Ring1][=Branch1][C][=O][C][N][C][=C][C][=C][Branch2][Ring1][Ring1][C][O][C][=C][Ring1][=Branch1][C][Ring1][Ring1][O]                                             CCCCN=CC=CC=CC=O                       CCCCOc1ccc(-c2[nH]ncc2C(=O)NCc2ccc(OC)cc2OC)cc1    0.101449
5    5.png                                                                                                                                                                                                                                                                                                                                                                                                                                                                               None                                                         None                                                                  None    0.000000
6    4.png                                                                                                                                                                                                                                                                                                                                             [C][C][C][C][C][C][Branch1][=Branch1][C][C][=O][N][C][C][=C][C][=C][Branch1][=Branch1][N][=N][C][=C][Ring1][=Branch1][C][=C][Ring1][N]                            CCCCCC(CC1=O)C=CC2=C(N=NC=C2)C=C1                                              Cn1cc(C(=O)N2CCCC2CO)nn1    0.149254
7    9.png                                                                                                                                                  [C][C][C][N][C][Branch2][Ring1][Branch2][C][C][C][C][C][C][C][Ring1][=Branch1][C][C][C][C][C][C][Ring1][=Branch1][C][C][C][=Branch1][C][=O][O][C][C][=C][C][=C][C][=C][C][=C][Ring1][=Branch1][F][=C][Ring1][Branch1][C][C][=C][Ring1][Branch2][C][F][F][C][=C][Ring1][Branch2][C][Ring1][#Branch1][C][C][Ring2][Ring1][Ring2][C]               CCCNC(CC1CCCCC1C2CCCCC2CCC=O)OCC=CC3=CC=CC=C3F                     [NH3+]C1CCCC(C(=O)NCC2(c3cccc(C(F)(F)F)c3)CCC2)C1    0.125000
8    8.png                                                                                                                                                                                                                                                                                                           [C][C][=C][C][=C][C][Branch2][Ring1][Branch2][S][Ring1][C][=Branch1][C][N+1][=Branch1][C][C][C][C][C][C][C][C][C][C][Branch1][Ring1][S][=Branch1][C][=C][Ring1][O][S][C]                          CC=CC=CC(=S([N+1])(C)CCCCCCCCS=C)SC                                           CSCC(C)[NH2+]Cc1nc(C)c(C)s1    0.081967
9   10.png                                                                                                                                                                                                                                                                                  [C][C][=C][C][=C][Branch2][Ring2][Ring2][Ring2][C][C][=Branch1][C][C][=O][O][C][C][C][C][C][C][Ring1][=Branch1][=C][C][=Branch1][C][=O][O][=C][C][=C][Ring1][=Branch1][C][C][Ring2][Ring1][Ring1]                                                      CC=CC#C                                    O=C(CN1CCC[NH2+]CC1)NCC(O)c1ccccc1    0.000000

``` 

# 结论

通过对局部loss进行惩罚的方式, 并不能让Qwen2-VL意识到smiles表达式的合法性问题, 小量数据下完全没有提升

先回头解决过拟合的问题.
