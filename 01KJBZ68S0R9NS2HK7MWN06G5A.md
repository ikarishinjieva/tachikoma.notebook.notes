---
title: 20240227 - 对embedding进行微调
confluence_page_id: 2590226
created_at: 2024-02-27T11:56:35+00:00
updated_at: 2024-02-28T15:58:07+00:00
---

使用制定的embedding将数据导入FAISS: [embedding_importing.html](/assets/01KJBZ68S0R9NS2HK7MWN06G5A/embedding_importing.html)

对训练数据进行评估: [embedding_testing (1).html](/assets/01KJBZ68S0R9NS2HK7MWN06G5A/embedding_testing%20%281%29.html)

# 对训练数据进行评估

## 当前使用mcontriever + openai

使用EnsembleRetriever将 mcontriever + openai两个embedding结合起来

效果: 

```
question[0] total delta = 121
question[1] total delta = 71
question[2] total delta = 92
question[3] total delta = 77
question[4] total delta = 126
question[5] total delta = 52
question[6] total delta = 126
question[7] total delta = 72
question[8] total delta = 76
question[9] total delta = 48
question[10] total delta = 82
question[11] total delta = 44
question[12] total delta = 149
question[13] total delta = 53
question[14] total delta = 164
question[15] total delta = 109
question[16] total delta = 92
question[17] total delta = 121
question[18] total delta = 122
question[19] total delta = 125
``` 

## BAAI/llm-embedder

效果: 

```
question[0] total delta = 330
question[1] total delta = 102
question[2] total delta = 339
question[3] total delta = 87
question[4] total delta = 221
question[5] total delta = 341
question[6] total delta = 119
question[7] total delta = 338
question[8] total delta = 211
question[9] total delta = 275
question[10] total delta = 333
question[11] total delta = 340
question[12] total delta = 306
question[13] total delta = 281
question[14] total delta = 364
question[15] total delta = 291
question[16] total delta = 350
question[17] total delta = 380
question[18] total delta = 158
question[19] total delta = 294
``` 

## BAAI/bge-large-zh-v1.5

效果: 

```
question[0] total delta = 243
question[1] total delta = 101
question[2] total delta = 144
question[3] total delta = 90
question[4] total delta = 184
question[5] total delta = 245
question[6] total delta = 127
question[7] total delta = 186
question[8] total delta = 183
question[9] total delta = 192
question[10] total delta = 229
question[11] total delta = 200
question[12] total delta = 181
question[13] total delta = 142
question[14] total delta = 360
question[15] total delta = 214
question[16] total delta = 309
question[17] total delta = 155
question[18] total delta = 96
question[19] total delta = 293
``` 

## 对BAAI/bge-large-zh-v1.5 进行微调

TODO: 目前用 前5000个 三元组 进行训练, 缩短训练时间, 查看效果:

epoch = 1, loss = 0.68

```
question[0] total delta = 203
question[1] total delta = 162
question[2] total delta = 17
question[3] total delta = 109
question[4] total delta = 139
question[5] total delta = 86
question[6] total delta = 146
question[7] total delta = 150
question[8] total delta = 163
question[9] total delta = 50
question[10] total delta = 319
question[11] total delta = 258
question[12] total delta = 194
question[13] total delta = 267
question[14] total delta = 284
question[15] total delta = 320
question[16] total delta = 322
question[17] total delta = 224
question[18] total delta = 96
question[19] total delta = 298
``` 

epoch = 5, loss = 0.38, 效果; 

```
question[0] total delta = 175
question[1] total delta = 166
question[2] total delta = 18
question[3] total delta = 145
question[4] total delta = 138
question[5] total delta = 88
question[6] total delta = 147
question[7] total delta = 148
question[8] total delta = 161
question[9] total delta = 46
question[10] total delta = 290
question[11] total delta = 303
question[12] total delta = 198
question[13] total delta = 283
question[14] total delta = 291
question[15] total delta = 320
question[16] total delta = 292
question[17] total delta = 236
question[18] total delta = 90
question[19] total delta = 271
``` 

效果有所改善

## BAAI/bge-m3 

效果:

```
question[0] total delta = 245
question[1] total delta = 132
question[2] total delta = 108
question[3] total delta = 90
question[4] total delta = 133
question[5] total delta = 131
question[6] total delta = 127
question[7] total delta = 229
question[8] total delta = 206
question[9] total delta = 215
question[10] total delta = 284
question[11] total delta = 173
question[12] total delta = 167
question[13] total delta = 80
question[14] total delta = 328
question[15] total delta = 183
question[16] total delta = 311
question[17] total delta = 164
question[18] total delta = 73
question[19] total delta = 329
``` 

## BAAI/bge-m3 微调

全数据, epoch = 10, loss = 0.976, 效果: 

```
question[0] total delta = 62
question[1] total delta = 34
question[2] total delta = 54
question[3] total delta = 17
question[4] total delta = 47
question[5] total delta = 17
question[6] total delta = 36
question[7] total delta = 34
question[8] total delta = 25
question[9] total delta = 32
question[10] total delta = 57
question[11] total delta = 27
question[12] total delta = 40
question[13] total delta = 36
question[14] total delta = 73
question[15] total delta = 42
question[16] total delta = 57
question[17] total delta = 46
question[18] total delta = 50
question[19] total delta = 97
``` 

# 对陌生数据的评估

## 当前使用mcontriever + openai:

```
question[91] total delta = 69
question[92] total delta = 58
question[93] total delta = 24
question[94] total delta = 29
question[95] total delta = 21
question[96] total delta = 68
question[97] total delta = 85
question[98] total delta = 78
all delta = 432
``` 

## BAAI/bge-m3 微调: 

[embedding_testing (2).html](/assets/01KJBZ68S0R9NS2HK7MWN06G5A/embedding_testing%20%282%29.html)

```
question[91] total delta = 79.0
question[92] total delta = 97.0
question[93] total delta = 78
question[94] total delta = 74.0
question[95] total delta = 71.0
question[96] total delta = 129.0
question[97] total delta = 261.0
question[98] total delta = 40
all delta = 829.0
``` 

经过微调, bge-m3的评估会逐渐逼近 当前使用的双embedding方案. 但尚未超过, 猜测原因:

  1. 训练数据不足
  2. 已经过拟合

拟先尝试增加训练数据
