---
title: 20250221 - 使用ms-swift对Qwen2-VL进行微调 [26] - 让模型学习"变化"
confluence_page_id: 3344385
created_at: 2025-02-23T04:16:50+00:00
updated_at: 2025-03-05T07:20:25+00:00
---

# 思路

主要思路: 让模型学习"变化"

之前已经总结出了有效的"变化": 

  - 表达式规范化, 图形规范化 (让模型对于一张图, 只输出唯一的表达式)
  - 镜像 (选取最后一个原子作为起点)
  - 选取某一个原子作为起点
  - 对局部子串进行轮换 (将第一个或最后一个原子, 插入到其他位置)
  - 在表达式中添加branch/ring操作符
  - 调整branch/ring操作符的位置
  - 调整branch/ring操作符的长度
  - 将branch/ring操作符后的原子往前调整
  - 选取两个原子互换位置

算法思路: 

  - 从已知的分子式出发, 使用某一种"变化", 产生新的分子式数据
  - 对于新的分子式数据进行评估, 检查哪些会出错
  - 使用部分新数据作为训练数据, 另一部分作为校验数据  

    - 如果校验数据正确率低, 需要研究数据特征? 从新的分子式中, 找到有助于训练的子集
  - 验证已知的分子式, 正确率是否下降

# 第1次训练: 直链长度4+长度5

commit: 80bbc1590730f80fd4dc8dc1b59641b65684a4f5

使用生成脚本: 

  - v1: 用add_basic_atom_to_end, 从长度1扩展到长度4, 数据训练量: 97
  - v2: 用add_basic_atom_to_end, 从v1开始, 从长度4扩展到长度5, 数据训练量: 144
  - v3: 用add_basic_atom_to_begin, 从v1开始, 从长度4扩展到长度5, 数据训练量: 130

训练效果: 

|  | v1+v2, epoch=1 (LR=1e-4) | v3, epoch=1 (LR=1e-4) | v1+v2, epoch=2 (LR=5e-5) | v3, epoch=2 (LR=5e-5) |
| --- | --- | --- | --- | --- |
| 直链, 长度5 | 75.00% | 100.00% | 87.50% | 93.75% |
| 直链, 长度6 | 44.44% | 66.67% | 61.11% | 50.00% |
| 直链, 长度7 | 5.26% | 26.32% | 10.53% | 21.05% |
| ring分子, 长度7 | 0.00% | 0.00% | 0.00% | 0.00% |
| branch分子, 长度7 | 50.00% | 63.33% | 66.67% | 70.00% |
  
发现:

  - 对长度6产生了推理能力
  - branch分子的校验数据中, 有一些是直链, 所以识别率升高 (长度7的branch分子, 刚好是长度5的直链)

# 第2次训练

使用生成脚本: 

  - v1: 用add_basic_atom_to_end, 从长度1扩展到长度4, 数据训练量: 97
  - v2: 用add_basic_atom_to_end, 从v1开始, 从长度4扩展到长度5, 数据训练量: 144
  - v3: 用add_basic_atom_to_begin, 从v1开始, 从长度4扩展到长度5, 数据训练量: 130
  - v4: 用add_basic_atom_to_end, 从v2+v3开始, 从长度5扩展到长度6, 数据训练量: 150
  - v5: 用add_basic_atom_to_begin, 从v2+v3开始, 从长度5扩展到长度6, 数据训练量: 150

|  | v1+v2+v3, epoch=1 (LR=1e-4) | v4, epoch=1 (LR=1e-4) | v5, epoch=1 (LR=1e-4) | v1+v2+v3, epoch=2 (LR=5e-5) | v4, epoch=2 (LR=5e-5) | v5, epoch=2 (LR=5e-5) |
| --- | --- | --- | --- | --- | --- | --- |
| 直链, 长度5 | 100.00% | 100.00% | 100.00% | 100.00% | 100.00% | 100.00% |
| 直链, 长度6 | 55.56% | 88.89% | 94.44% | 77.78% | 88.89% | 94.44% |
| 直链, 长度7 | 15.79% | 78.95% | 78.95% | 63.16% | 68.42% | 100.00% |
| ring分子, 长度7 | 0.00% | 0.00% | 0.00% | 0.00% | 0.00% | 0.00% |
| branch分子, 长度7 | 66.67% | 70.00% | 70.00% | 70.00% | 70.00% | 70.00% |
  
发现: 

  - (直链, 长度6) 准确率始终无法到100%
  - 第2个epoch, 会让第一个epoch的结果变差, 然后再变好, 并且带来了对 (直链, 长度7) 的推理能力大幅提升. 
    - 猜测: 重现推理过程, 会让模型有更好的理解? 

# 第3次训练

使用生成脚本: 

  - v1: 用add_basic_atom_to_end, 从长度1扩展到长度4, 数据训练量: 97
  - v2: 用add_basic_atom_to_end, 从v1开始, 从长度4扩展到长度5, 数据训练量: 144
  - v3: 用add_basic_atom_to_begin, 从v1开始, 从长度4扩展到长度5, 数据训练量: 130
  - v4: 用add_basic_atom_to_end, 从v2+v3开始, 从长度5扩展到长度6, 数据训练量: 150
  - v5: 用add_basic_atom_to_begin, 从v2+v3开始, 从长度5扩展到长度6, 数据训练量: 150
  - v6: 用add_basic_atom_to_end, 从v4+v5开始, 从长度6扩展到长度7, 数据训练量: 150
  - v7: 用add_basic_atom_to_begin, 从v4+v5开始, 从长度6扩展到长度7, 数据训练量: 150

|| v1+v2+v3, epoch=1   
(LR=1e-4)| v4+v5, epoch=1   
(LR=1e-4)| v6+v7, epoch=1   
(LR=1e-4)| v1+v2+v3, epoch=2   
(LR=5e-5)| v4+v5, epoch=2   
(LR=5e-5)| v6+v7, epoch=2   
(LR=5e-5)  
---|---|---|---|---|---|---  
直链, 长度5| 81.25%| 100.00%| 100.00%| 100.00%| 100.00%| 100.00%  
直链, 长度6| 50.00%| 88.89%| 100.00%| 94.44%| 94.44%| 88.89%  
直链, 长度7| 10.53%| 89.47%| 78.95%| 84.21%| 63.16%| 94.74%  
直链, 长度8| 5.00%| 35.00%| 25.00%| 40.00%| 25.00%| 55.00%  
直链, 长度9| 0.00%| 15.00%| 10.00%| 5.00%| 0.00%| 25.00%  
ring分子, 长度7| 0.00%| 0.00%| 0.00%| 0.00%| 0.00%| 0.00%  
branch分子, 长度7| 70.00%| 70.00%| 60.00%| 70.00%| 70.00%| 70.00%  
  
观察: 

  - epoch 2, (直链, 长度6) 准确率下降
  - epoch 2, (直链, 长度8) 的推理能力上升
  - 需要分析标红处的偏差问题:
    - epoch=1, 直链, 长度7:

```
13  /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_7/20.png  {
"selfies": "[O][C][C][N][N][O][O]"}          OCCNNOO  {
"selfies": "[O][C][N][C][N][O][O]"}   OCNCNOO    0.280000   
mismatch
3/4对调

8   /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_7/25.png  {
"selfies": "[C][C][O][C][O][N][O]"}          CCOCONO  {
"selfies": "[O][N][C][O][C][O][C]"}   ONCOCOC    0.307692   
mismatch
镜像, 2/3对调, 3/4对调
[C][O][C][O][C][N][O]

15  /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_7/78.png     {
"selfies": "[N][O][C][O][O][N]"}           NOCOON  {
"selfies": "[N][O][C][C][O][O][N]"}   NOCCOON    0.470588   
mismatch
少原子

0   /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_7/16.png     {
"selfies": "[O][O][C][C][O][O]"}           OOCCOO  {
"selfies": "[C][O][O][C][C][O][O]"}   COOCCOO    0.533333   
mismatch
少原子

```

    - epoch=2, 直链, 长度7: 少原子
    - epoch=2, 直链, 长度6: 多原子

结论: 到epoch=2, 直链分子只剩下原子数量错误

# 第4次训练

使用生成脚本: 

  - v1: 用add_basic_atom_to_end, 从长度1扩展到长度4, 数据训练量: 97
  - v2: 用add_basic_atom_to_end, 从v1开始, 从长度4扩展到长度5, 数据训练量: 144
  - v3: 用add_basic_atom_to_begin, 从v1开始, 从长度4扩展到长度5, 数据训练量: 130
  - v4: 用add_basic_atom_to_end, 从v2+v3开始, 从长度5扩展到长度6, 数据训练量: 150
  - v5: 用add_basic_atom_to_begin, 从v2+v3开始, 从长度5扩展到长度6, 数据训练量: 150
  - v6: 用add_basic_atom_to_end, 从v4+v5开始, 从长度6扩展到长度7, 数据训练量: 150
  - v7: 用add_basic_atom_to_begin, 从v4+v5开始, 从长度6扩展到长度7, 数据训练量: 150
  - v8: 用add_basic_atom_to_end, 从v6+v7开始, 从长度7扩展到长度8, 数据训练量: 150
  - v9: 用add_basic_atom_to_begin, 从v6+v7开始, 从长度7扩展到长度8, 数据训练量: 150

|| v1-v5, epoch=1  
(LR=1e-4)| v6-v7, epoch=1   
(LR=1e-4)| v8-v9, epoch=1   
(LR=1e-4)| v1-v5, epoch=2   
(LR=5e-5)| v6-v7, epoch=2   
(LR=5e-5)| v8-v9, epoch=2   
(LR=5e-5)  
---|---|---|---|---|---|---  
直链, 长度5| 93.75%| 100.00%| 93.75%| 100.00%| 100.00%| 100.00%  
直链, 长度6| 100.00%| 88.89%| 83.33%| 94.44%| 88.89%| 94.44%  
直链, 长度7| 89.47%| 89.47%| 89.47%| 84.21%| 100.00%| 100.00%  
直链, 长度8| 60.00%| 45.00%| 95.00%| 30.00%| 65.00%| 85.00%  
直链, 长度9| 10.00%| 10.00%| 50.00%| 5.00%| 25.00%| 25.00%  
ring分子, 长度7| 0.00%| 0.00%| 0.00%| 0.00%| 0.00%| 0.00%  
branch分子, 长度7| 63.33%| 70.00%| 60.00%| 70.00%| 70.00%| 70.00%  
  
长度8效果并不好, 查看错误分类: 

  - (直链, 长度6), 多一个原子
  - (直链, 长度8), 都是少一个原子

两者都是往长度7靠拢. 

# 第5次训练

使用生成脚本: 

  - v1: 用add_basic_atom_to_end, 从长度1扩展到长度4, 数据训练量: 97
  - v2: 用add_basic_atom_to_end, 从v1开始, 从长度4扩展到长度5, 数据训练量: 144
  - v3: 用add_basic_atom_to_begin, 从v1开始, 从长度4扩展到长度5, 数据训练量: 130
  - v4: 用add_basic_atom_to_end, 从v2+v3开始, 从长度5扩展到长度6, 数据训练量: 150
  - v5: 用add_basic_atom_to_begin, 从v2+v3开始, 从长度5扩展到长度6, 数据训练量: 150
  - v6: 用add_basic_atom_to_end, 从v4+v5开始, 从长度6扩展到长度7, 数据训练量: 150
  - v7: 用add_basic_atom_to_begin, 从v4+v5开始, 从长度6扩展到长度7, 数据训练量: 150
  - v8: 用add_basic_atom_to_end, 从v6+v7开始, 从长度7扩展到长度8, 数据训练量: 150
  - v9: 用add_basic_atom_to_begin, 从v6+v7开始, 从长度7扩展到长度8, 数据训练量: 150
  - v10: 用add_basic_atom_to_end, 从v8+v9开始, 从长度8扩展到长度9, 数据训练量: 150
  - v11: 用add_basic_atom_to_begin, 从v8+v9开始, 从长度8扩展到长度9, 数据训练量: 150

|| v1-v7, epoch=1  
(LR=1e-4)| v8-v9, epoch=1   
(LR=1e-4)| v10-v11, epoch=1   
(LR=1e-4)| v1-v7, epoch=2   
(LR=5e-5)| v8-v9, epoch=2   
(LR=5e-5)| v10-v11, epoch=2   
(LR=5e-5)  
---|---|---|---|---|---|---  
直链, 长度5| 87.50%| 75.00%| 81.25%| 93.75%| 100.00%| 100.00%  
直链, 长度6| 72.22%| 72.22%| 77.78%| 88.89%| 94.44%| 94.44%  
直链, 长度7| 73.68%| 84.21%| 68.42%| 73.68%| 100.00%| 100.00%  
直链, 长度8| 55.00%| 60.00%| 80.00%| 70.00%| 95.00%| 95.00%  
直链, 长度9| 5.00%| 35.00%| 35.00%| 45.00%| 50.00%| 55.00%  
ring分子, 长度7| 0.00%| 0.00%| 0.00%| 0.00%| 0.00%| 0.00%  
branch分子, 长度7| 50.00%| 43.33%| 43.33%| 70.00%| 70.00%| 70.00%  
  
长度9的识别率很低:

```
2    /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_9/7.png     {
"selfies": "[C][O][C][N][C][N][C][N]"}         COCNCNCN  {
"selfies": "[C][O][O][C][N][C][C][N][N]"}  COOCNCCNN    0.266667   
mismatch
位置3少原子, 7/8对调

5   /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_9/45.png  {
"selfies": "[N][N][N][O][N][C][C][C][O]"}        NNNONCCCO  {
"selfies": "[N][N][N][O][C][N][C][C][O]"}  NNNOCNCCO    0.413793   
mismatch
5/6对调

0   /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_9/10.png  {
"selfies": "[N][O][C][N][N][N][C][N][N]"}        NOCNNNCNN  {
"selfies": "[N][O][N][C][N][N][C][N][N]"}  NONCNNCNN    0.440000   
mismatch
3/4对调

18  /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_9/47.png  {
"selfies": "[O][C][C][N][N][C][N][C][O]"}        OCCNNCNCO  {
"selfies": "[O][C][N][C][N][N][C][O][C]"}  OCNCNNCOC    0.480000   
mismatch
3/4对调, 6/7对调, 8/9对调

17  /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_9/39.png     {
"selfies": "[N][O][C][N][O][N][O][N]"}         NOCNONON  {
"selfies": "[N][O][N][O][N][N][C][O][N]"}  NONONNCON    0.565217   
mismatch
[N][O][C][N][N][O][N][O][N]
位置4, 少原子

14   /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_9/6.png     {
"selfies": "[N][C][C][C][N][N][O][N]"}         NCCCNNON  {
"selfies": "[C][N][C][C][C][N][N][O][N]"}  CNCCCNNON    0.583333   
mismatch
位置1, 少原子

16  /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_9/93.png     {
"selfies": "[C][N][C][O][O][C][O][N]"}         CNCOOCON  {
"selfies": "[C][N][C][O][O][C][C][O][N]"}  CNCOOCCON    0.608696   
mismatch
位置7, 少原子

9   /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_9/27.png     {
"selfies": "[C][N][N][N][C][N][C][O]"}         CNNNCNCO  {
"selfies": "[C][N][N][N][C][N][C][C][O]"}  CNNNCNCCO    0.652174   
mismatch
位置8, 少原子

8   /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_9/63.png     {
"selfies": "[C][O][N][C][O][C][C][O]"}         CONCOCCO  {
"selfies": "[O][C][C][C][O][C][N][O][C]"}  OCCCOCNOC    0.739130   
mismatch
位置8, 少原子
[C][O][N][C][O][C][C][C][O] 
``` 

补充对调数据

# 第6次训练

使用生成脚本: 

  - v1: 用add_basic_atom_to_end, 从长度1扩展到长度4, 数据训练量: 97
  - v2: 用add_basic_atom_to_end, 从v1开始, 从长度4扩展到长度5, 数据训练量: 144
  - v3: 用add_basic_atom_to_begin, 从v1开始, 从长度4扩展到长度5, 数据训练量: 130
  - v4: 用add_basic_atom_to_end, 从v2+v3开始, 从长度5扩展到长度6, 数据训练量: 150
  - v5: 用add_basic_atom_to_begin, 从v2+v3开始, 从长度5扩展到长度6, 数据训练量: 150
  - v6: 用add_basic_atom_to_end, 从v4+v5开始, 从长度6扩展到长度7, 数据训练量: 150
  - v7: 用add_basic_atom_to_begin, 从v4+v5开始, 从长度6扩展到长度7, 数据训练量: 150
  - v8: 用add_basic_atom_to_end, 从v6+v7开始, 从长度7扩展到长度8, 数据训练量: 150
  - v9: 用add_basic_atom_to_begin, 从v6+v7开始, 从长度7扩展到长度8, 数据训练量: 150
  - v10: 用add_basic_atom_to_end, 从v8+v9开始, 从长度8扩展到长度9, 数据训练量: 150
  - v11: 用add_basic_atom_to_begin, 从v8+v9开始, 从长度8扩展到长度9, 数据训练量: 150
  - v12: 用swap_adjacent_atoms, 从v6+v7开始, 从长度7扩展 交换相邻原子的数据, 数据训练量: 150
  - v13: 用swap_adjacent_atoms, 从v8+v9开始, 从长度8扩展 交换相邻原子的数据, 数据训练量: 150
  - v14: 用swap_adjacent_atoms, 从v10+v11开始, 从长度9扩展 交换相邻原子的数据, 数据训练量: 150
  - v15 = 14 (脚本错误)

每阶段将数据混合起来 (原来是分步骤训练):

(因为脚本原因, 只采集到最后一组数值)

|| v1-v9, epoch=1  
(LR=1e-4)| v10-v11, epoch=1   
(LR=1e-4)| v12-v13, epoch=1   
(LR=1e-4)| v14-v15, epoch=1   
(LR=1e-4)  
---|---|---|---|---  
直链, 长度5|  |  |  | 62.50%  
直链, 长度6|  |  |  | 66.67%  
直链, 长度7|  |  |  | 84.21%  
直链, 长度8|  |  |  | 90.00%  
直链, 长度9|  |  |  | 95.00%  
ring分子, 长度7|  |  |  | 0.00%  
branch分子, 长度7|  |  |  | 56.67%  
  
可以看到: (直链, 长度9) 准确率上升没问题: (交换相邻原子的数据) 是有效的. 但随着训练步骤变多, 长度5/6/7在遗忘 (查看错误类型, 都是原子数多一个)

下一阶段使用两阶段训练, 试试遗忘是否能减轻

# 第7次训练

使用生成脚本: 

  - v1: 用add_basic_atom_to_end, 从长度1扩展到长度4, 数据训练量: 97
  - v2: 用add_basic_atom_to_end, 从v1开始, 从长度4扩展到长度5, 数据训练量: 144
  - v3: 用add_basic_atom_to_begin, 从v1开始, 从长度4扩展到长度5, 数据训练量: 130
  - v4: 用add_basic_atom_to_end, 从v2+v3开始, 从长度5扩展到长度6, 数据训练量: 150
  - v5: 用add_basic_atom_to_begin, 从v2+v3开始, 从长度5扩展到长度6, 数据训练量: 150
  - v6: 用add_basic_atom_to_end, 从v4+v5开始, 从长度6扩展到长度7, 数据训练量: 150
  - v7: 用add_basic_atom_to_begin, 从v4+v5开始, 从长度6扩展到长度7, 数据训练量: 150
  - v8: 用add_basic_atom_to_end, 从v6+v7开始, 从长度7扩展到长度8, 数据训练量: 150
  - v9: 用add_basic_atom_to_begin, 从v6+v7开始, 从长度7扩展到长度8, 数据训练量: 150
  - v10: 用add_basic_atom_to_end, 从v8+v9开始, 从长度8扩展到长度9, 数据训练量: 150
  - v11: 用add_basic_atom_to_begin, 从v8+v9开始, 从长度8扩展到长度9, 数据训练量: 150
  - v12: 用swap_adjacent_atoms, 从v6+v7开始, 从长度7扩展 交换相邻原子的数据, 数据训练量: 150
  - v13: 用swap_adjacent_atoms, 从v8+v9开始, 从长度8扩展 交换相邻原子的数据, 数据训练量: 150
  - v14: 用swap_adjacent_atoms, 从v10+v11开始, 从长度9扩展 交换相邻原子的数据, 数据训练量: 150
  - v15: 用add_basic_atom_to_end, 从v10+v11开始, 从长度9扩展到长度10, 数据训练量: 150
  - v16: 用add_basic_atom_to_begin, 从v10+v11开始, 从长度9扩展到长度10, 数据训练量: 150
  - v17: 用swap_adjacent_atoms, 从v15+v16开始, 从长度10扩展 交换相邻原子的数据, 数据训练量: 150

直接在上一步checkpoint上, 追加v15-v17:

|  | 追加 v15-v17, epoch=1 |
| --- | --- |
| 直链, 长度5 | 75.00% |
| 直链, 长度6 | 72.22% |
| 直链, 长度7 | 100.00% |
| 直链, 长度8 | 100.00% |
| 直链, 长度9 | 100.00% |
| 直链, 长度10 | 75.00% |
| ring分子, 长度7 | 0.00% |
| branch分子, 长度7 | 60.00% |
  
查看 (直链, 长度10)的错误类型: 

```
18  /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_10/95.png  {
"selfies": "[N][N][N][C][C][C][N][N][N][N]"}       NNNCCCNNNN  {
"selfies": "[N][N][N][C][N][C][C][N][N][N]"}  NNNCNCCNNN    0.391304   mismatch
镜像, 6/7对调
[N][N][N][C][C][N][C][N][N][N]

16  /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_10/84.png  {
"selfies": "[N][O][N][O][N][O][C][N][O][O]"}       NONONOCNOO  {
"selfies": "[N][O][N][N][O][N][C][N][O][O]"}  NONNONCNOO    0.400000   mismatch
[N][O][N][N][O][N][C][N][O][O]
4-6的N/O交换

14  /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_10/76.png     {
"selfies": "[C][O][N][O][N][C][O][O][N]"}        CONONCOON  {
"selfies": "[N][O][O][C][N][O][N][C][O][C]"}  NOOCNONCOC    0.640000   mismatch
[C][O][C][N][O][N][C][O][O][N]
镜像, 位置3少原子

17  /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_10/93.png     {
"selfies": "[C][O][N][O][N][N][C][O][N]"}        CONONNCON  {
"selfies": "[N][O][C][N][N][O][O][N][O][C]"}  NOCNNOONOC    0.653846   mismatch
镜像, 位置4少原子
[C][O][N][O][O][N][N][C][O][N]

11  /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_10/66.png     {
"selfies": "[C][O][C][O][N][O][C][O][N]"}        COCONOCON  {
"selfies": "[C][O][C][O][O][N][O][C][O][N]"}  COCOONOCON    0.666667   mismatch
位置4少原子

``` 

  

再扩展长度11, 找一下规律

# 第8次训练

使用生成脚本: 

  - v1: 用add_basic_atom_to_end, 从长度1扩展到长度4, 数据训练量: 97
  - v2: 用add_basic_atom_to_end, 从v1开始, 从长度4扩展到长度5, 数据训练量: 144
  - v3: 用add_basic_atom_to_begin, 从v1开始, 从长度4扩展到长度5, 数据训练量: 130
  - v4: 用add_basic_atom_to_end, 从v2+v3开始, 从长度5扩展到长度6, 数据训练量: 150
  - v5: 用add_basic_atom_to_begin, 从v2+v3开始, 从长度5扩展到长度6, 数据训练量: 150
  - v6: 用add_basic_atom_to_end, 从v4+v5开始, 从长度6扩展到长度7, 数据训练量: 150
  - v7: 用add_basic_atom_to_begin, 从v4+v5开始, 从长度6扩展到长度7, 数据训练量: 150
  - v8: 用add_basic_atom_to_end, 从v6+v7开始, 从长度7扩展到长度8, 数据训练量: 150
  - v9: 用add_basic_atom_to_begin, 从v6+v7开始, 从长度7扩展到长度8, 数据训练量: 150
  - v10: 用add_basic_atom_to_end, 从v8+v9开始, 从长度8扩展到长度9, 数据训练量: 150
  - v11: 用add_basic_atom_to_begin, 从v8+v9开始, 从长度8扩展到长度9, 数据训练量: 150
  - v12: 用swap_adjacent_atoms, 从v6+v7开始, 从长度7扩展 交换相邻原子的数据, 数据训练量: 150
  - v13: 用swap_adjacent_atoms, 从v8+v9开始, 从长度8扩展 交换相邻原子的数据, 数据训练量: 150
  - v14: 用swap_adjacent_atoms, 从v10+v11开始, 从长度9扩展 交换相邻原子的数据, 数据训练量: 150
  - v15: 用add_basic_atom_to_end, 从v10+v11开始, 从长度9扩展到长度10, 数据训练量: 150
  - v16: 用add_basic_atom_to_begin, 从v10+v11开始, 从长度9扩展到长度10, 数据训练量: 150
  - v17: 用swap_adjacent_atoms, 从v15+v16开始, 从长度10扩展 交换相邻原子的数据, 数据训练量: 150
  - v18: 用add_basic_atom_to_end, 从v15+v16开始, 从长度10扩展到长度11, 数据训练量: 150
  - v19: 用add_basic_atom_to_begin, 从v15+v16开始, 从长度10扩展到长度11, 数据训练量: 150
  - v20: 用swap_adjacent_atoms, 从v18+v19开始, 从长度11扩展 交换相邻原子的数据, 数据训练量: 150

直接在上一步checkpoint上, 追加v18-v20:

|  | 追加 v15-v17, epoch=1 |
| --- | --- |
| 直链, 长度5 | 31.25% |
| 直链, 长度6 | 33.33% |
| 直链, 长度7 | 36.84% |
| 直链, 长度8 | 60.00% |
| 直链, 长度9 | 85.00% |
| 直链, 长度10 | 90.00% |
| 直链, 长度11 | 85.00% |
| ring分子, 长度7 | 0.00% |
| branch分子, 长度7 | 20.00% |
  
(直链, 长度11)的错误是 输出了12原子的分子. (直链, 长度10)的错误是 输出了11原子的分子, 和 一个相邻原子交换

输出多余的原子, 认为是推理能力过高, 尝试继续增加原子数到15, 观察情况.

另外, 遗忘问题, 尝试通过整合数据到一次训练观察情况

  

# 第9次训练

使用生成脚本: 

  - v1: 用add_basic_atom_to_end, 从长度1扩展到长度4, 数据训练量: 97
  - v2: 用add_basic_atom_to_end, 从v1开始, 从长度4扩展到长度5, 数据训练量: 144
  - v3: 用add_basic_atom_to_begin, 从v1开始, 从长度4扩展到长度5, 数据训练量: 130
  - v4: 用add_basic_atom_to_end, 从v2+v3开始, 从长度5扩展到长度6, 数据训练量: 150
  - v5: 用add_basic_atom_to_begin, 从v2+v3开始, 从长度5扩展到长度6, 数据训练量: 150
  - v6: 用add_basic_atom_to_end, 从v4+v5开始, 从长度6扩展到长度7, 数据训练量: 150
  - v7: 用add_basic_atom_to_begin, 从v4+v5开始, 从长度6扩展到长度7, 数据训练量: 150
  - v8: 用add_basic_atom_to_end, 从v6+v7开始, 从长度7扩展到长度8, 数据训练量: 150
  - v9: 用add_basic_atom_to_begin, 从v6+v7开始, 从长度7扩展到长度8, 数据训练量: 150
  - v10: 用add_basic_atom_to_end, 从v8+v9开始, 从长度8扩展到长度9, 数据训练量: 150
  - v11: 用add_basic_atom_to_begin, 从v8+v9开始, 从长度8扩展到长度9, 数据训练量: 150
  - v12: 用swap_adjacent_atoms, 从v6+v7开始, 从长度7扩展 交换相邻原子的数据, 数据训练量: 150
  - v13: 用swap_adjacent_atoms, 从v8+v9开始, 从长度8扩展 交换相邻原子的数据, 数据训练量: 150
  - v14: 用swap_adjacent_atoms, 从v10+v11开始, 从长度9扩展 交换相邻原子的数据, 数据训练量: 150
  - v15: 用add_basic_atom_to_end, 从v10+v11开始, 从长度9扩展到长度10, 数据训练量: 150
  - v16: 用add_basic_atom_to_begin, 从v10+v11开始, 从长度9扩展到长度10, 数据训练量: 150
  - v17: 用swap_adjacent_atoms, 从v15+v16开始, 从长度10扩展 交换相邻原子的数据, 数据训练量: 150
  - v18: 用add_basic_atom_to_end, 从v15+v16开始, 从长度10扩展到长度11, 数据训练量: 150
  - v19: 用add_basic_atom_to_begin, 从v15+v16开始, 从长度10扩展到长度11, 数据训练量: 150
  - v20: 用swap_adjacent_atoms, 从v18+v19开始, 从长度11扩展 交换相邻原子的数据, 数据训练量: 150
  - v21: 用add_basic_atom_to_end, 从v18+v19开始, 从长度11扩展到长度12, 数据训练量: 150
  - v22: 用add_basic_atom_to_begin, 从v18+v19开始, 从长度11扩展到长度12, 数据训练量: 150
  - v23: 用swap_adjacent_atoms, 从v21+v22开始, 从长度12扩展 交换相邻原子的数据, 数据训练量: 150
  - v24: 用add_basic_atom_to_end, 从v21+v22开始, 从长度12扩展到长度13, 数据训练量: 150
  - v25: 用add_basic_atom_to_begin, 从v21+v22开始, 从长度12扩展到长度13, 数据训练量: 150
  - v26: 用swap_adjacent_atoms, 从v24+v25开始, 从长度13扩展 交换相邻原子的数据, 数据训练量: 150
  - v27: 用add_basic_atom_to_end, 从v24+v25开始, 从长度13扩展到长度14, 数据训练量: 150
  - v28: 用add_basic_atom_to_begin, 从v24+v25开始, 从长度13扩展到长度14, 数据训练量: 150
  - v29: 用swap_adjacent_atoms, 从v27+v28开始, 从长度14扩展 交换相邻原子的数据, 数据训练量: 150
  - v30: 用add_basic_atom_to_end, 从v27+v28开始, 从长度14扩展到长度15, 数据训练量: 150
  - v31: 用add_basic_atom_to_begin, 从v27+v28开始, 从长度14扩展到长度15, 数据训练量: 150
  - v32: 用swap_adjacent_atoms, 从v30+v31开始, 从长度15扩展 交换相邻原子的数据, 数据训练量: 150

  

|  | v1-v29, epoch=1 | v30-v32, epoch=1 | v1-v29, epoch=2 | v30-v32, epoch=2 |
| --- | --- | --- | --- | --- |
| 直链, 长度5 | 100.00% | 93.75% | 100.00% | 100.00% |
| 直链, 长度6 | 100.00% | 55.56% | 100.00% | 94.44% |
| 直链, 长度7 | 100.00% | 78.95% | 100.00% | 100.00% |
| 直链, 长度8 | 100.00% | 75.00% | 100.00% | 95.00% |
| 直链, 长度9 | 100.00% | 85.00% | 100.00% | 100.00% |
| 直链, 长度10 | 95.00% | 75.00% | 95.00% | 100.00% |
| 直链, 长度11 | 100.00% | 75.00% | 100.00% | 100.00% |
| 直链, 长度12 | 85.00% | 75.00% | 85.00% | 85.00% |
| 直链, 长度13 | 90.00% | 85.00% | 100.00% | 100.00% |
| 直链, 长度14 | 80.00% | 80.00% | 75.00% | 85.00% |
| 直链, 长度15 | 25.00% | 60.00% | 45.00% | 55.00% |
| ring分子, 长度7 | 0.00% | 0.00% | 0.00% | 0.00% |
| branch分子, 长度7 | 70.00% | 66.67% | 70.00% | 70.00% |
  
  

需要分析 (直链, 长度14) 和 (直链, 长度15)的错误分类: 

(直链, 长度14)

```
9   /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_14/46.png        {
"selfies": "[C][N][O][O][O][O][C][C][O][O][C][O][O]"}    CNOOOOCCOOCOO  {
"selfies": "[C][N][O][O][C][O][C][C][O][O][O][C][O][O]"}  CNOOCOCCOOOCOO     0.59375   
mismatch
位置10少原子, 位置5/10对调

17   /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_14/9.png        {
"selfies": "[N][O][O][O][C][O][O][C][C][C][O][C][N]"}    NOOOCOOCCCOCN  {
"selfies": "[N][C][C][O][C][C][C][O][O][C][O][O][O][N]"}  NCCOCCCOOCOOON     0.75000   
mismatch
镜像, 位置13少一个原子
[N][O][O][O][C][O][O][C][C][C][O][C][C][N]

5   /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_14/20.png  {
"selfies": "[C][O][N][O][C][C][C][O][O][C][O][C][N][O][O]"}  CONOCCCOOCOCNOO  {
"selfies": "[O][O][N][C][O][C][O][O][C][C][O][N][O][C]"}  OONCOCOOCCONOC     0.81250   
mismatch
镜像, 位置7多一个原子
[C][O][N][O][C][C][O][O][C][O][C][N][O][O]

``` 

(直链, 长度15)

```
 11  /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_15/61.png     {
"selfies": "[C][N][C][C][O][C][N][O][N][O][O][C][N][N][O]"}   CNCCOCNONOOCNNO  {
"selfies": "[O][N][C][N][O][O][N][O][N][C][O][C][C][N][C]"}  ONCNOONONCOCCNC    0.631579   
mismatch
镜像, 12/13对调
[C][N][C][C][O][C][N][O][N][O][O][N][C][N][O]

9   /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_15/51.png        {
"selfies": "[C][O][O][C][N][O][O][N][N][N][C][C][C][N]"}    COOCNOONNNCCCN  {
"selfies": "[N][C][C][N][C][N][N][N][O][O][N][C][O][O][C]"}  NCCNCNNNOONCOOC    0.722222   
mismatch
位置12少原子
[C][O][O][C][N][O][O][N][N][N][C][N][C][C][N]

3   /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_15/20.png  {
"selfies": "[O][O][O][N][N][C][N][C][O][O][N][C][O][N][O][N]"}  OOONNCNCOONCONON  {
"selfies": "[O][O][O][N][C][N][C][O][O][N][C][O][N][O][N]"}  OOONCNCOONCONON    0.750000   
mismatch
位置5多一个原子

16  /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_15/72.png        {
"selfies": "[O][O][N][C][N][C][N][C][N][N][C][N][O][O]"}    OONCNCNCNNCNOO  {
"selfies": "[O][O][O][N][C][N][C][N][C][N][N][C][N][O][O]"}  OOONCNCNCNNCNOO    0.782609   
mismatch
位置3少一个原子

19  /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_15/94.png        {
"selfies": "[C][O][N][O][C][O][O][N][O][N][O][C][O][C]"}    CONOCOONONOCOC  {
"selfies": "[C][O][N][O][C][O][O][O][N][O][N][O][C][O][C]"}  CONOCOOONONOCOC    0.785714   
mismatch
位置8少一个原子

1   /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_15/11.png        {
"selfies": "[O][C][N][O][O][C][N][C][O][N][C][O][C][N]"}    OCNOOCNCONCOCN  {
"selfies": "[N][C][O][C][N][O][O][C][N][C][O][O][N][C][O]"}  NCOCNOOCNCOONCO    0.827586   
mismatch
镜像, 位置10少一个原子
[O][C][N][O][O][C][N][C][O][O][N][C][O][C][N]

6   /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_15/30.png           {
"selfies": "[C][O][N][N][N][N][O][O][N][C][C][C][C]"}     CONNNNOONCCCC  {
"selfies": "[C][O][N][N][N][N][N][O][O][N][C][C][C][C][C]"}  CONNNNNOONCCCCC    0.851852   
mismatch
位置7和位置15各少一个原子

15  /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_15/71.png        {
"selfies": "[N][O][O][C][C][O][O][O][N][C][N][O][O][N]"}    NOOCCOOONCNOON  {
"selfies": "[N][O][O][C][C][O][O][O][O][N][C][N][O][O][N]"}  NOOCCOOOONCNOON    0.875000   
mismatch
位置9少一个原子

0    /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_15/1.png           {
"selfies": "[O][N][N][O][N][N][C][N][C][C][N][C][O]"}     ONNONNCNCCNCO  {
"selfies": "[O][N][N][O][N][N][C][N][C][N][C][C][N][C][O]"}  ONNONNCNCNCCNCO    0.923077   
mismatch
位置10和位置11各少一个原子
``` 

大部分错误都是原子数差异, 猜测可以增加epoch或者训练数据数量来改善 (从之前的经验中猜测)

TODO: 

  1. 将v1-v32进行一步训练
  2. 也可以尝试渐进式的课程训练
  3. 下一步增加ring的支持

# 第10次训练

将v1-v32进行一步训练, checkpiont: /opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v1147-20250225-022404/checkpoint-885

epoch=1的训练时间: 3059 s

训练效果: 

|  | v1-v32, epoch=1 |
| --- | --- |
| 直链, 长度5 | 100.00% |
| 直链, 长度6 | 100.00% |
| 直链, 长度7 | 100.00% |
| 直链, 长度8 | 95.00% |
| 直链, 长度9 | 100.00% |
| 直链, 长度10 | 95.00% |
| 直链, 长度11 | 95.00% |
| 直链, 长度12 | 85.00% |
| 直链, 长度13 | 85.00% |
| 直链, 长度14 | 70.00% |
| 直链, 长度15 | 65.00% |
| ring分子, 长度7 | 0.00% |
| branch分子, 长度7 | 70.00% |
  
标红处的错误分布, 大部分是原子数不一致

可尝试渐进式课程训练

# 第11次训练

增加ring的支持

在第10次训练的checkpoint上, 进行追加: 

  - v33: 用add_ring_op, 从v1开始, 为长度4增加ring op, 数据训练量: 58
  - v34: 用add_ring_op, 从v2+v3开始, 为长度5增加ring op, 数据训练量: 150
  - v35: 用add_ring_op, 从v4+v5开始, 为长度6增加ring op, 数据训练量: 150
  - v36: 用add_ring_op, 从v6+v7开始, 为长度7增加ring op, 数据训练量: 150

|  | 追加v33-v36, epoch=1 | 追加v33-v36, epoch=2 |
| --- | --- | --- |
| 直链, 长度5 | 100.00% | 100.00% |
| 直链, 长度6 | 94.44% | 94.44% |
| 直链, 长度7 | 100.00% | 100.00% |
| 直链, 长度8 | 100.00% | 95.00% |
| 直链, 长度9 | 85.00% | 85.00% |
| 直链, 长度10 | 80.00% | 75.00% |
| 直链, 长度11 | 70.00% | 55.00% |
| 直链, 长度12 | 40.00% | 30.00% |
| 直链, 长度13 | 40.00% | 35.00% |
| 直链, 长度14 | 35.00% | 20.00% |
| 直链, 长度15 | 25.00% | 10.00% |
| ring分子, 长度7 | 32.50% | 57.50% |
| branch分子, 长度7 | 70.00% | 70.00% |
  
  - 对追加数据训练的epoch增加, 会导致其他遗忘 (这是已知问题)
  - ring分子的错误分布比较杂, 也就是模型并未习得比较好的规律
  - 考虑增加基础数据量

将v33-v36的数据量限制, 从150放宽到300, 数据量: 

```
/opt/huangyan/mol-ai/v2-data/dataset/33.train.json:53
/opt/huangyan/mol-ai/v2-data/dataset/34.train.json:300
/opt/huangyan/mol-ai/v2-data/dataset/35.train.json:300
/opt/huangyan/mol-ai/v2-data/dataset/36.train.json:300
``` 

训练效果: 

|  | 追加v33-v36, epoch=1 | 追加v33-v36, epoch=2 |
| --- | --- | --- |
| ring分子, 长度7 | 62.50% | 67.50% |
| ring分子, 长度8 | 45.00% | 65.00% |
| ring分子, 长度9 | 40.00% | 35.00% |
| 直链, 长度7 | 100.00% | 100.00% |
| branch分子, 长度7 | 70.00% | 70.00% |
  
查看(ring分子, 长度9)的错误分布: 

```
10  /opt/huangyan/mol-ai/images/ring_selfies/length_9/123.png         {
"selfies": "[C][O][O][C][C][N][Ring1][Ring1][C]"}        COOC1CN1C     {
"selfies": "[C][C][N][C][Ring1][Ring1][C][O][O]"}  CC1NC1COO    0.100000   
mismatch
镜像, ring前的原子轮换
[O][O][C][C][N][C][Ring1][Ring1][C]

7    /opt/huangyan/mol-ai/images/ring_selfies/length_9/26.png      {
"selfies": "[C][O][O][C][N][N][Ring1][Ring1][N][C]"}       COOC1NN1NC     {
"selfies": "[C][N][N][N][Ring1][Ring1][C][O][O]"}  CN1NN1COO    0.121212   
mismatch
镜像, ring位置不动, 其他原子轮换
[O][O][C][N][NH1][N][Ring1][Ring1][C]

13   /opt/huangyan/mol-ai/images/ring_selfies/length_9/84.png      {
"selfies": "[C][C][N][N][O][O][O][Ring1][=Branch1]"}        CC1NNOOO1  {
"selfies": "[N][C][O][O][O][N][Ring1][=Branch1][C]"}  N1COOON1C    0.129032   
mismatch
镜像, 1-4轮换
[C][N][N][C][O][O][O][Ring1][=Branch1]

0   /opt/huangyan/mol-ai/images/ring_selfies/length_9/180.png         {
"selfies": "[O][O][C][C][N][Ring1][=Branch1][O]"}         O1OCCN1O  {
"selfies": "[N][O][O][O][C][C][Ring1][=Branch1][O]"}  N1OOOCC1O    0.137931   
mismatch
少一个原子, ring前的原子轮换
[O][C][C][O][O][O][N][Ring1][=Branch1]

25   /opt/huangyan/mol-ai/images/ring_selfies/length_9/96.png                    {
"selfies": "[C][C][C][O][C][N][O][N]"}         CCCOCNON     {
"selfies": "[C][O][C][C][Ring1][Ring2][N][O][N]"}  C1OCC1NON    0.161290   
mismatch
ring op识别成了[C], ring前的原子轮换

21  /opt/huangyan/mol-ai/images/ring_selfies/length_9/176.png                       {
"selfies": "[N][N][O][C][C][O][N]"}          NNOCCON     {
"selfies": "[O][C][N][N][Ring1][Ring2][C][O][N]"}  O1CNN1CON    0.178571   
mismatch
缺少ring op, ring前原子轮换

8    /opt/huangyan/mol-ai/images/ring_selfies/length_9/23.png                       {
"selfies": "[N][N][N][C][C][O][N]"}          NNNCCON     {
"selfies": "[N][C][N][N][Ring1][Ring2][C][O][N]"}  N1CNN1CON    0.178571   
mismatch
缺少ring op, ring前原子轮换

39  /opt/huangyan/mol-ai/images/ring_selfies/length_9/158.png         {
"selfies": "[O][C][N][C][C][C][N][Ring1][Ring2]"}        OCNC1CCN1     {
"selfies": "[O][C][N][C][C][C][Ring1][Ring2][N]"}  OCN1CCC1N    0.200000   
mismatch
ring op和相邻原子 对调

6   /opt/huangyan/mol-ai/images/ring_selfies/length_9/138.png       {
"selfies": "[N][C][N][C][N][N][O][Ring1][Branch1]"}        NCN1CNNO1     {
"selfies": "[O][C][N][N][Ring1][Ring2][C][N][N]"}  O1CNN1CNN    0.200000   
mismatch
镜像, 复杂差异
[N][N][C][N][N][C][O][Ring1][Ring2]

9    /opt/huangyan/mol-ai/images/ring_selfies/length_9/44.png    {
"selfies": "[N][N][N][C][C][N][O][N][Ring1][Branch1]"}       NNNC1CNON1   {
"selfies": "[C][C][N][O][N][Ring1][Branch1][N][N]"}  C1CNON1NN    0.212121   
mismatch
镜像, 多一个原子
[N][N][N][C][C][N][O][Ring1][Branch1]

4    /opt/huangyan/mol-ai/images/ring_selfies/length_9/13.png         {
"selfies": "[O][C][N][N][O][O][N][Ring1][Ring2]"}        OCNN1OON1     {
"selfies": "[O][C][N][O][O][N][Ring1][Ring2][N]"}  OCN1OON1N    0.214286   
mismatch
ring后的原子前插

11  /opt/huangyan/mol-ai/images/ring_selfies/length_9/124.png         {
"selfies": "[O][N][C][C][N][O][N][Ring1][Ring1]"}        ONCCN1ON1     {
"selfies": "[O][N][C][C][O][N][Ring1][Ring1][N]"}  ONCC1ON1N    0.250000   
mismatch
ring后的原子前插

14  /opt/huangyan/mol-ai/images/ring_selfies/length_9/108.png      {
"selfies": "[C][C][N][C][C][O][C][Ring1][=Branch1]"}        CC1NCCOC1  {
"selfies": "[C][C][O][C][N][C][C][Ring1][=Branch1]"}  CC1OCNCC1    0.250000   
mismatch
镜像, ring后的原子前插
[C][C][N][C][O][C][Ring1][=Branch1][C]

23  /opt/huangyan/mol-ai/images/ring_selfies/length_9/170.png      {
"selfies": "[O][C][N][C][N][N][N][Ring1][Ring2][O]"}       OCNC1NNN1O     {
"selfies": "[O][C][N][N][C][N][N][Ring1][Ring2]"}  OCNN1CNN1    0.275862   
mismatch
多一个原子, 4/5对调

3   /opt/huangyan/mol-ai/images/ring_selfies/length_9/196.png       {
"selfies": "[C][C][C][N][C][C][N][Ring1][Branch1]"}        CCC1NCCN1     {
"selfies": "[N][C][C][C][Ring1][Ring2][C][N][C]"}  N1CCC1CNC    0.280000   
mismatch
镜像, 2/4对调, ring长度错误
[C][N][C][C][C][C][N][Ring1][Ring2]

30   /opt/huangyan/mol-ai/images/ring_selfies/length_9/91.png      {
"selfies": "[C][C][N][C][O][C][N][Ring1][=Branch1]"}        CC1NCOCN1  {
"selfies": "[C][C][O][C][N][N][C][Ring1][=Branch1]"}  CC1OCNNC1    0.291667   
mismatch
3/5对调, 6/7对调

19  /opt/huangyan/mol-ai/images/ring_selfies/length_9/189.png      {
"selfies": "[C][C][N][C][N][Ring1][=Branch1][N][O]"}        C1CNCN1NO  {
"selfies": "[O][N][C][N][C][C][N][Ring1][=Branch1]"}  ON1CNCCN1    0.307692   
mismatch
镜像, 原子1插入ring op后
[N][C][C][N][C][N][Ring1][=Branch1][O]

28    /opt/huangyan/mol-ai/images/ring_selfies/length_9/8.png       {
"selfies": "[O][N][C][N][C][N][Ring1][Branch1][N]"}        ON1CNCN1N   {
"selfies": "[O][N][N][C][C][N][Ring1][Branch1][N]"}  ON1NCCN1N    0.320000   
mismatch
3/4对调

1   /opt/huangyan/mol-ai/images/ring_selfies/length_9/119.png         {
"selfies": "[C][N][C][N][C][N][Ring1][Ring1][C]"}        CNCN1CN1C     {
"selfies": "[C][N][C][N][N][C][Ring1][Ring1][C]"}  CNCN1NC1C    0.320000   
mismatch
5/6对调

20   /opt/huangyan/mol-ai/images/ring_selfies/length_9/24.png      {
"selfies": "[N][C][O][N][C][N][N][Ring1][Ring1][O]"}       NCONC1NN1O     {
"selfies": "[N][C][N][Ring1][Ring1][N][O][C][N]"}  N1CN1NOCN    0.333333   
mismatch
镜像, 多一个原子, 5/6对调
[N][C][O][N][N][C][N][Ring1][Ring1]

22   /opt/huangyan/mol-ai/images/ring_selfies/length_9/11.png         {
"selfies": "[C][N][C][N][C][N][Ring1][Ring1][N]"}        CNCN1CN1N     {
"selfies": "[C][N][C][N][N][C][Ring1][Ring1][N]"}  CNCN1NC1N    0.346154   
mismatch
5/6对调

18   /opt/huangyan/mol-ai/images/ring_selfies/length_9/14.png       {
"selfies": "[N][C][N][C][N][C][O][Ring1][Branch1]"}        NCN1CNCO1   {
"selfies": "[N][C][N][O][N][C][C][Ring1][Branch1]"}  NCN1ONCC1    0.357143   
mismatch
4/7对调

26   /opt/huangyan/mol-ai/images/ring_selfies/length_9/87.png      {
"selfies": "[N][C][O][C][C][C][O][Ring1][=Branch1]"}        NC1OCCCO1  {
"selfies": "[N][C][O][C][O][C][C][Ring1][=Branch1]"}  NC1OCOCC1    0.363636   
mismatch
5/7对调

38   /opt/huangyan/mol-ai/images/ring_selfies/length_9/61.png    {
"selfies": "[N][N][O][C][N][Ring1][Branch1][C][O][O]"}       N1NOCN1COO     {
"selfies": "[O][N][C][N][Ring1][Ring2][C][O][O]"}  O1NCN1COO    0.366667   
mismatch
多一个原子, 1/2对调, ring长度错误

35  /opt/huangyan/mol-ai/images/ring_selfies/length_9/157.png  {
"selfies": "[O][C][C][N][N][O][N][Ring1][Ring2][Ring3]"}        OCCN1NON1     {
"selfies": "[O][C][C][N][N][N][O][Ring1][Ring2]"}  OCCN1NNO1    0.375000   
mismatch
6/7对调, ring op错误(多一个ring)

31   /opt/huangyan/mol-ai/images/ring_selfies/length_9/76.png         {
"selfies": "[O][O][O][C][C][N][Ring1][Ring1][N]"}        OOOC1CN1N     {
"selfies": "[N][C][N][C][Ring1][Ring1][O][O][O]"}  NC1NC1OOO    0.375000   
mismatch
镜像, 5/6对调
[O][O][O][C][N][C][Ring1][Ring1][N]

``` 

  

其中 14个原子对调, 7个轮换, 还有其他错误. 

缩减数据量回到150, 然后进行ring基础上的相邻原子对调 (虽然在错误中, 相邻原子对调不多, 都是远端原子对调, 但先进行尝试): 

  - v33: 用add_ring_op, 从v1开始, 为长度4增加ring op, 数据训练量: 53
  - v34: 用add_ring_op, 从v2+v3开始, 为长度5增加ring op, 数据训练量: 150
  - v35: 用add_ring_op, 从v4+v5开始, 为长度6增加ring op, 数据训练量: 150
  - v36: 用add_ring_op, 从v6+v7开始, 为长度7增加ring op, 数据训练量: 150
  - v37: 用swap_adjacent_atoms, 从v33开始, 对长度6的ring分子进行相邻原子交换, 数据训练量: 29
  - v38: 用swap_adjacent_atoms, 从v34开始, 对长度7的ring分子进行相邻原子交换, 数据训练量: 150
  - v39: 用swap_adjacent_atoms, 从v35开始, 对长度8的ring分子进行相邻原子交换, 数据训练量: 150
  - v40: 用swap_adjacent_atoms, 从v36开始, 对长度9的ring分子进行相邻原子交换, 数据训练量: 150

|  | 追加v33-v36, epoch=1 | 追加v37-v40, epoch=1 | 追加v33-v36, epoch=2 | 追加v37-v40, epoch=2 |
| --- | --- | --- | --- | --- |
| ring分子, 长度7 | 50.00% | 70.00% | 65.00% | 67.50% |
| ring分子, 长度8 | 40.00% | 42.50% | 52.50% | 55.00% |
| ring分子, 长度9 | 22.50% | 42.50% | 50.00% | 42.50% |
| 直链, 长度7 | 94.74% | 84.21% | 100.00% | 100.00% |
| branch分子, 长度7 | 70.00% | 70.00% | 70.00% | 70.00% |
  
效果轻微提升, 并不理想. 转而分析长度7的错误分类 (长度9的错误太多, 分析时间长): 

```
[
    {
        "Predicted": "[N][C][N][O][Ring1][Ring2][O]",
        "GT": "[C][N][O][N][Ring1][Ring2][O]",
        "Annotation": "ring op前的原子轮换"
    },
    {
        "Predicted": "[O][O][N][O][N][Ring1][Ring2]",
        "GT": "[N][N][O][O][O][Ring1][Ring2]",
        "Annotation": "镜像([O][O][O][N][Ring1][Ring2][N]), ring op前移, 3/4对调"
    },
    {
        "Predicted": "[C][O][N][C][N][Ring1][Ring2]",
        "GT": "[N][O][C][N][Ring1][Ring2][C]",
        "Annotation": "镜像([C][N][C][O][N][Ring1][Ring2]), ring op前的原子轮换"
    },
    {
        "Predicted": "[N][C][C][C][O][Ring1][Branch1]",
        "GT": "[O][C][N][C][C][Ring1][Branch1]",
        "Annotation": "起点3([N][C][C][O][C][Ring1][Branch1]), 4/5对调"
    },
    {
        "Predicted": "[C][O][C][C][C][Ring1][Ring2]",
        "GT": "[C][O][C][C][Ring1][Ring2][C]",
        "Annotation": "ring op后移"
    },
    {
        "Predicted": "[N][O][N][O][N][Ring1][Ring2]",
        "GT": "[O][N][O][N][Ring1][Ring2][N]",
        "Annotation": "镜像([N][N][O][N][O][Ring1][Ring2]), ring op前的原子轮换"
    },
    {
        "Predicted": "[C][O][C][N][Ring1][Ring2][O]",
        "GT": "[O][C][N][C][Ring1][Ring2][O]",
        "Annotation": "ring op前的原子轮换"
    },
    {
        "Predicted": "[C][N][C][C][C][Ring1][Ring2]",
        "GT": "[C][N][C][C][Ring1][Ring2][C]",
        "Annotation": "ring op前移"
    },
    {
        "Predicted": "[O][N][C][N][Ring1][Ring1][N]",
        "GT": "[N][C][N][N][Ring1][Ring1][O]",
        "Annotation": "镜像([O][N][N][C][Ring1][Ring1][N]), 3/4对调"
    },
    {
        "Predicted": "[N][N][C][N][C][Ring1][Ring2]",
        "GT": "[C][N][N][N][C][Ring1][Ring2]",
        "Annotation": "1/3对调"
    },
    {
        "Predicted": "[C][N][C][O][O][Ring1][Ring2]",
        "GT": "[O][C][N][C][O][Ring1][Ring2]",
        "Annotation": "ring op前的原子轮换"
    },
    {
        "Predicted": "[C][C][O][C][N][Ring1][Ring1]",
        "GT": "[C][N][O][C][Ring1][Ring1][C]",
        "Annotation": "镜像([C][C][O][N][Ring1][Ring1][C]), 4/5对调, ring op前移"
    },
    {
        "Predicted": "[O][O][N][O][N][Ring1][Ring2]",
        "GT": "[O][O][O][N][Ring1][Ring2][N]",
        "Annotation": "3/4对调, ring op前移"
    }
]
``` 

其中: 

  - 6个对调, 5个相邻对调, 1个远端对调
  - 5个"ring op前的原子轮换"
  - 4个"ring op前移"

# 增加了可视化的结果程序

脚本: eval_web.py

![image2025-2-25 15:14:58.png](/assets/01KJBZRQQVCZ5ZSZZ4W1F9974D/image2025-2-25%2015%3A14%3A58.png)

<http://127.0.0.1:7860/>

# 第12次训练 - 增加"ring op 移动一位"的数据增强

  - v33: 用add_ring_op, 从v1开始, 为长度4增加ring op, 数据训练量: 53
  - v34: 用add_ring_op, 从v2+v3开始, 为长度5增加ring op, 数据训练量: 150
  - v35: 用add_ring_op, 从v4+v5开始, 为长度6增加ring op, 数据训练量: 150
  - v36: 用add_ring_op, 从v6+v7开始, 为长度7增加ring op, 数据训练量: 150
  - v37: 用move_ring_op_one_step, 从v33开始, 对长度6的ring分子进行 ring op 移动一位, 数据训练量: 0
  - v38: 用move_ring_op_one_step, 从v34开始, 对长度7的ring分子进行 ring op 移动一位, 数据训练量: 65
  - v39: 用move_ring_op_one_step, 从v35开始, 对长度8的ring分子进行 ring op 移动一位, 数据训练量: 110
  - v40: 用move_ring_op_one_step, 从v36开始, 对长度9的ring分子进行 ring op 移动一位, 数据训练量: 150

|  | 追加v33-v36, epoch=1 | 追加v37-v40, epoch=1 | 追加v33-v36, epoch=2 | 追加v37-v40, epoch=2 |
| --- | --- | --- | --- | --- |
| ring分子, 长度7 | 47.50% | 47.50% | 62.50% | 65.00% |
| ring分子, 长度8 | 37.50% | 37.50% | 45.00% | 42.50% |
| ring分子, 长度9 | 22.50% | 25.00% | 37.50% | 37.50% |
| 直链, 长度7 | 94.74% | 94.74% | 94.74% | 94.74% |
| branch分子, 长度7 | 70.00% | 70.00% | 70.00% | 70.00% |
  
正确率比起之前训练有降低, 分析标红的错误分布: 

```
[
    {
        "Predicted": "[N][O][O][N][O][Ring1][Ring2]",
        "GT": "[N][N][O][O][O][Ring1][Ring2]",
        "Annotation": "2/4对调"
    },
    {
        "Predicted": "[C][C][O][C][N][Ring1][Branch1]",
        "GT": "[C][O][N][C][C][Ring1][Branch1]",
        "Annotation": "镜像([C][C][N][O][C][Ring1][Branch1]), 3-5轮换"
    },
    {
        "Predicted": "[C][O][N][N][C][Ring1][Ring2]",
        "GT": "[N][O][C][N][Ring1][Ring2][C]",
        "Annotation": "起点3([C][O][N][N][Ring1][Ring2][C]), ring op右移"
    },
    {
        "Predicted": "[C][C][O][C][C][Ring1][Ring2]",
        "GT": "[C][O][C][C][Ring1][Ring2][C]",
        "Annotation": "ring op左移"
    },
    {
        "Predicted": "[C][O][C][N][N][Ring1][Ring1]",
        "GT": "[C][O][N][N][C][Ring1][Ring1]",
        "Annotation": "3/5对调"
    },
    {
        "Predicted": "[O][C][O][C][O][Ring1][Ring1]",
        "GT": "[O][C][C][Ring1][Ring1][O][O]",
        "Annotation": "镜像([O][O][C][C][O][Ring1][Ring1]), 2/3对调"
    },
    {
        "Predicted": "[N][O][N][O][Ring1][Ring2][N]",
        "GT": "[O][N][O][N][Ring1][Ring2][N]",
        "Annotation": "镜像([N][N][O][N][O][Ring1][Ring2]), ring op后的原子前插"
    },
    {
        "Predicted": "[C][O][C][N][O][Ring1][Ring2]",
        "GT": "[O][C][N][C][Ring1][Ring2][O]",
        "Annotation": "复杂变换"
    },
    {
        "Predicted": "[O][N][C][N][Ring1][Ring1][N]",
        "GT": "[N][C][N][N][Ring1][Ring1][O]",
        "Annotation": "镜像([O][N][N][C][Ring1][Ring1][N]), 3/4对调"
    },
    {
        "Predicted": "[C][N][N][C][N][Ring1][Ring2]",
        "GT": "[C][N][N][N][C][Ring1][Ring2]",
        "Annotation": "4/5对调"
    },
    {
        "Predicted": "[C][N][C][O][O][Ring1][Ring2]",
        "GT": "[O][C][N][C][O][Ring1][Ring2]",
        "Annotation": "1-5轮换"
    },
    {
        "Predicted": "[C][O][C][O][O][Ring1][Ring2]",
        "GT": "[O][C][O][C][Ring1][Ring2][O]",
        "Annotation": "镜像([O][C][O][C][O][Ring1][Ring2]), 1-5轮换"
    },
    {
        "Predicted": "[C][N][C][N][Ring1][Ring1][N]",
        "GT": "[N][C][N][N][Ring1][Ring1][C]",
        "Annotation": "镜像([C][N][N][C][Ring1][Ring1][N]), 3/4对调"
    },
    {
        "Predicted": "[N][O][O][N][O][Ring1][Ring2]",
        "GT": "[O][O][O][N][Ring1][Ring2][N]",
        "Annotation": "镜像([N][N][O][O][O][Ring1][Ring2]), 2-4轮换"
    }
]
``` 

发现脚本错误: 本次实验需要重新跑

重新审视错误分析, 除了一个"复杂变换", 可以认为都是将 "某一个原子插入另一个位置"

# 第13次训练 - 增加"relocate_atom"的数据增强

  - v33: 用add_ring_op, 从v1开始, 为长度4增加ring op, 数据训练量: 53
  - v34: 用add_ring_op, 从v2+v3开始, 为长度5增加ring op, 数据训练量: 150
  - v35: 用add_ring_op, 从v4+v5开始, 为长度6增加ring op, 数据训练量: 150
  - v36: 用add_ring_op, 从v6+v7开始, 为长度7增加ring op, 数据训练量: 150
  - v37: 用relocate_atom, 从v33开始, 对长度6的ring分子进行某一个原子重定位, 数据训练量: 83
  - v38: 用relocate_atom, 从v34开始, 对长度7的ring分子进行某一个原子重定位, 数据训练量: 150
  - v39: 用relocate_atom, 从v35开始, 对长度8的ring分子进行某一个原子重定位, 数据训练量: 150
  - v40: 用relocate_atom, 从v36开始, 对长度9的ring分子进行某一个原子重定位, 数据训练量: 150

训练效果: 

训练效果跟使用swap_adjacent_atoms类似

|  | 追加v33-v36, epoch=1 | 追加v37-v40, epoch=1 | 追加v33-v36, epoch=2 | 追加v37-v40, epoch=2 |
| --- | --- | --- | --- | --- |
| ring分子, 长度7 | 55.00% | 70.00% | 75.00% | 65.00% |
| ring分子, 长度8 | 37.50% | 62.50% | 60.00% | 57.50% |
| ring分子, 长度9 | 25.00% | 45.00% | 42.50% | 52.50% |
| 直链, 长度7 | 94.74% | 89.47% | 100.00% | 100.00% |
| branch分子, 长度7 | 70.00% | 70.00% | 70.00% | 70.00% |
  
发现另一个问题: 在生成数据时, 150组数据是300条数据, 在训练脚本中被限制成了150条, 应当放开限制

放开限制: 

  - v33: 用add_ring_op, 从v1开始, 为长度4增加ring op, 数据训练量: 53
  - v34: 用add_ring_op, 从v2+v3开始, 为长度5增加ring op, 数据训练量: 270
  - v35: 用add_ring_op, 从v4+v5开始, 为长度6增加ring op, 数据训练量: 270
  - v36: 用add_ring_op, 从v6+v7开始, 为长度7增加ring op, 数据训练量: 270
  - v37: 用relocate_atom, 从v33开始, 对长度6的ring分子进行某一个原子重定位, 数据训练量: 83
  - v38: 用relocate_atom, 从v34开始, 对长度7的ring分子进行某一个原子重定位, 数据训练量: 270
  - v39: 用relocate_atom, 从v35开始, 对长度8的ring分子进行某一个原子重定位, 数据训练量: 270
  - v40: 用relocate_atom, 从v36开始, 对长度9的ring分子进行某一个原子重定位, 数据训练量: 270

  

|  | 追加v33-v36, epoch=1 | 追加v37-v40, epoch=1 | 追加v33-v36, epoch=2 | 追加v37-v40, epoch=2 |
| --- | --- | --- | --- | --- |
| ring分子, 长度7 | 67.50% | 75.00% | 75.00% | 75.00% |
| ring分子, 长度8 | 47.50% | 65.00% | 60.00% | 70.00% |
| ring分子, 长度9 | 32.50% | 37.50% | 45.00% | 52.50% |
| 直链, 长度7 | 94.74% | 94.74% | 94.74% | 94.74% |
| branch分子, 长度7 | 70.00% | 70.00% | 70.00% | 70.00% |
  
  

放开数据限制后, 未见明显提升, 还是设置回原来的数据限制

  

查看错误分布: 

(ring分子, 长度7): 

```
[
    {
        "Predicted": "[C][C][C][N][Ring1][Ring2][O]",
        "GT": "[O][C][N][C][C][Ring1][Ring2]",
        "Annotation": "镜像, 3/4对调"
    },
    {
        "Predicted": "[N][O][O][N][O][Ring1][Ring2]",
        "GT": "[N][N][O][O][O][Ring1][Ring2]",
        "Annotation": "2/4对调"
    },
    {
        "Predicted": "[C][O][C][N][N][Ring1][Ring2]",
        "GT": "[N][O][C][N][Ring1][Ring2][C]",
        "Annotation": "起点3([C][O][N][N][Ring1][Ring2][C]), ring op前原子后插"
    },
    {
        "Predicted": "[C][C][O][C][Ring1][Ring2][C]",
        "GT": "[C][O][C][C][Ring1][Ring2][C]",
        "Annotation": "2/3对调"
    },
    {
        "Predicted": "[N][O][N][O][Ring1][Ring2][N]",
        "GT": "[O][N][O][N][Ring1][Ring2][N]",
        "Annotation": "镜像([N][N][O][N][O][Ring1][Ring2]\n), ring op后原子前插. 或认为整体轮换"
    },
    {
        "Predicted": "[C][O][C][N][Ring1][Ring2][O]",
        "GT": "[O][C][N][C][Ring1][Ring2][O]",
        "Annotation": "位置1调整到位置4"
    },
    {
        "Predicted": "[C][N][N][C][N][Ring1][Ring2]",
        "GT": "[C][N][N][N][C][Ring1][Ring2]",
        "Annotation": "4/5对调"
    },
    {
        "Predicted": "[C][N][C][O][Ring1][Ring2][O]",
        "GT": "[O][C][N][C][O][Ring1][Ring2]",
        "Annotation": "ring op后原子前插. 或认为整体轮换"
    },
    {
        "Predicted": "[C][O][C][O][Ring1][Ring2][O]",
        "GT": "[O][C][O][C][Ring1][Ring2][O]",
        "Annotation": "镜像([O][C][O][C][O][Ring1][Ring2]\n), ring op前原子后插. 或认为整体轮换"
    },
    {
        "Predicted": "[N][O][O][N][O][Ring1][Ring2]",
        "GT": "[O][O][O][N][Ring1][Ring2][N]",
        "Annotation": "镜像, 位置1调整到位置4"
    }
]
``` 

(ring分子, 长度8): 

```
[
    {
        "Predicted": "[C][N][C][O][O][N][Ring1][=Branch1]",
        "GT": "[O][N][C][N][O][C][Ring1][=Branch1]",
        "Annotation": "位置3([C][N][O][C][O][N][Ring1][=Branch1]), 3/4对调"
    },
    {
        "Predicted": "[C][N][C][O][N][Ring1][Ring2][O]",
        "GT": "[C][N][O][N][Ring1][Ring2][C][O]",
        "Annotation": "ring op 前原子后插"
    },
    {
        "Predicted": "[C][O][C][N][Ring1][Ring1][C]",
        "GT": "[C][C][N][O][C][Ring1][Ring1][O]",
        "Annotation": "少原子"
    },
    {
        "Predicted": "[N][C][O][O][N][Ring1][Ring1][O]",
        "GT": "[N][C][O][O][N][Ring1][Ring2][O]",
        "Annotation": "ring长度错误"
    },
    {
        "Predicted": "[C][N][O][C][O][O][Ring1][=Branch1]",
        "GT": "[O][O][N][C][O][C][Ring1][=Branch1]",
        "Annotation": "起点4([C][N][O][O][C][O][Ring1][=Branch1]), 3/4对调"
    },
    {
        "Predicted": "[O][O][O][N][C][O][N][Ring1][Ring1]",
        "GT": "[O][N][N][Ring1][Ring1][O][O][O]",
        "Annotation": "多一个原子"
    },
    {
        "Predicted": "[C][O][N][O][Ring1][Ring2][C][O]",
        "GT": "[O][C][O][N][Ring1][Ring2][C][O]",
        "Annotation": "1/2对调, 3/4对调"
    },
    {
        "Predicted": "[C][O][C][O][C][O][Ring1][=Branch1]",
        "GT": "[C][O][C][O][O][C][Ring1][=Branch1]",
        "Annotation": "5/6对调"
    },
    {
        "Predicted": "[N][C][N][N][Ring1][Ring2][N][N]",
        "GT": "[C][N][N][N][Ring1][Ring2][N][N]",
        "Annotation": "1/2对调"
    },
    {
        "Predicted": "[O][C][O][N][O][Ring1][Ring2][O]",
        "GT": "[O][C][O][O][N][Ring1][Ring2][O]",
        "Annotation": "4/5对调"
    },
    {
        "Predicted": "[C][C][C][C][N][Ring1][Branch1][O]",
        "GT": "[O][C][C][N][C][Ring1][Branch1][C]",
        "Annotation": "1/8对调, 4/5对调"
    },
    {
        "Predicted": "[C][N][C][N][N][Ring1][Branch1][N]",
        "GT": "[C][N][N][N][Ring1][Ring2][N][C]",
        "Annotation": "ring长度错误"
    }
]
``` 

  

相邻对调还是比较多, 叠加这部分的训练数据试试: 测试结果, 叠加了 "swap_adjacent_atoms" 后, 效果下降了10%

  

# 第14次训练 - relocate_atom + move_ring_op_one_step

[附件: 归档 7.zip] 

# 第15次训练 - relocate_atom叠加

  - v33: 用add_ring_op, 从v1开始, 为长度4增加ring op, 数据训练量: 53
  - v34: 用add_ring_op, 从v2+v3开始, 为长度5增加ring op, 数据训练量: 150
  - v35: 用add_ring_op, 从v4+v5开始, 为长度6增加ring op, 数据训练量: 150
  - v36: 用add_ring_op, 从v6+v7开始, 为长度7增加ring op, 数据训练量: 150
  - v37: 用relocate_atom, 从v33开始, 对长度6的ring分子进行某一个原子重定位, 数据训练量: 83
  - v38: 用relocate_atom, 从v34开始, 对长度7的ring分子进行某一个原子重定位, 数据训练量: 150
  - v39: 用relocate_atom, 从v35开始, 对长度8的ring分子进行某一个原子重定位, 数据训练量: 150
  - v40: 用relocate_atom, 从v36开始, 对长度9的ring分子进行某一个原子重定位, 数据训练量: 150
  - v41: 用relocate_atom, 从v37开始, 对长度6的ring分子进行某一个原子重定位*2, 数据训练量: 83
  - v42: 用relocate_atom, 从v38开始, 对长度7的ring分子进行某一个原子重定位*2, 数据训练量: 150
  - v43: 用relocate_atom, 从v39开始, 对长度8的ring分子进行某一个原子重定位*2, 数据训练量: 150
  - v44: 用relocate_atom, 从v40开始, 对长度9的ring分子进行某一个原子重定位*2, 数据训练量: 150

|  | 追加v33-v36, epoch=1 | 追加v37-v40, epoch=1 | 追加v41-v44, epoch=1 | 追加v33-v36, epoch=2 | 追加v37-v40, epoch=2 | 追加v41-v44, epoch=2 |
| --- | --- | --- | --- | --- | --- | --- |
| ring分子, 长度7 | 50.00% | 67.50% | 77.50% | 72.50% | 67.50% | 75.00% |
| ring分子, 长度8 | 40.00% | 47.50% | 52.50% | 62.50% | 52.50% | 50.00% |
| ring分子, 长度9 | 27.50% | 35.00% | 37.50% | 45.00% | 32.50% | 30.00% |
| 直链, 长度7 | 94.74% | 94.74% | 68.42% | 94.74% | 100.00% | 94.74% |
| branch分子, 长度7 | 70.00% | 70.00% | 70.00% | 70.00% | 70.00% | 70.00% |
  
  

未见明显提升. 分析错误(标红): 

```
 [
    {
        "Predicted": "[N][O][O][N][O][Ring1][Ring2]",
        "GT": "[N][N][O][O][O][Ring1][Ring2]",
        "Annotation": "原子4 -> 位置2"
    },
    {
        "Predicted": "[C][O][N][C][N][Ring1][Ring2]",
        "GT": "[N][O][C][N][Ring1][Ring2][C]",
        "Annotation": "镜像, 原子2 -> 位置4"
    },
    {
        "Predicted": "[N][C][N][Ring1][Ring1][N]",
        "GT": "[N][N][N][C][Ring1][Ring1][O]",
        "Annotation": "镜像([O][C][N][N][Ring1][Ring1][N]), 少一个原子"
    },
    {
        "Predicted": "[C][C][O][C][Ring1][Ring2][C]",
        "GT": "[C][O][C][C][Ring1][Ring2][C]",
        "Annotation": "原子3 -> 位置2"
    },
    {
        "Predicted": "[N][O][N][O][N][Ring1][Ring2]",
        "GT": "[O][N][O][N][Ring1][Ring2][N]",
        "Annotation": "原子1 -> 位置7"
    },
    {
        "Predicted": "[C][O][C][N][Ring1][Ring2][O]",
        "GT": "[O][C][N][C][Ring1][Ring2][O]",
        "Annotation": "原子1 -> 位置4"
    },
    {
        "Predicted": "[N][O][N][C][Ring1][Ring1][N]",
        "GT": "[N][C][N][N][Ring1][Ring1][O]",
        "Annotation": "镜像([O][N][N][C][Ring1][Ring1][N]), 原子1 -> 位置2"
    },
    {
        "Predicted": "[C][N][N][C][N][Ring1][Ring2]",
        "GT": "[C][N][N][N][C][Ring1][Ring2]",
        "Annotation": "原子4 -> 位置5"
    },
    {
        "Predicted": "[C][N][C][O][O][Ring1][Ring2]",
        "GT": "[O][C][N][C][O][Ring1][Ring2]",
        "Annotation": "原子4 -> 位置1"
    },
    {
        "Predicted": "[N][O][O][N][O][Ring1][Ring2]",
        "GT": "[O][O][O][N][Ring1][Ring2][N]",
        "Annotation": "镜像([N][N][O][O][O][Ring1][Ring2]), 原子4 -> 位置2"
    }
]
``` 

  

尝试: 进行多次 v37-v44 的迭代, 未见提升

# 第16次训练 - 增加增强图像

为每一个版本, 都生成对应的增强图像

  - v33: 用add_ring_op, 从v1开始, 为长度4增加ring op, 数据训练量: 53
  - v34: 用add_ring_op, 从v2+v3开始, 为长度5增加ring op, 数据训练量: 150
  - v35: 用add_ring_op, 从v4+v5开始, 为长度6增加ring op, 数据训练量: 150
  - v36: 用add_ring_op, 从v6+v7开始, 为长度7增加ring op, 数据训练量: 150
  - v37: 用relocate_atom, 从v33开始, 对长度6的ring分子进行某一个原子重定位, 数据训练量: 83
  - v38: 用relocate_atom, 从v34开始, 对长度7的ring分子进行某一个原子重定位, 数据训练量: 150
  - v39: 用relocate_atom, 从v35开始, 对长度8的ring分子进行某一个原子重定位, 数据训练量: 150
  - v40: 用relocate_atom, 从v36开始, 对长度9的ring分子进行某一个原子重定位, 数据训练量: 150
  - v41: 用swap_adjacent_atoms, 从v37开始, 对长度6的ring分子交换相邻原子, 数据训练量: 83
  - v42: 用swap_adjacent_atoms, 从v38开始, 对长度7的ring分子交换相邻原子, 数据训练量: 150
  - v43: 用swap_adjacent_atoms, 从v39开始, 对长度8的ring分子交换相邻原子, 数据训练量: 150
  - v44: 用swap_adjacent_atoms, 从v40开始, 对长度9的ring分子交换相邻原子, 数据训练量: 150

|  | 追加v33-v36, epoch=1 | 追加v37-v44, epoch=1 | 追加v33-v36, epoch=2 | 追加v37-v44, epoch=2 |
| --- | --- | --- | --- | --- |
| ring分子, 长度7 | 65.00% | 75.00% | 65.00% | 70.00% |
| ring分子, 长度8 | 47.50% | 65.00% | 60.00% | 67.50% |
| ring分子, 长度9 | 27.50% | 50.00% | 42.50% | 45.00% |
| 直链, 长度7 | 100.00% | 94.74% | 100.00% | 100.00% |
| branch分子, 长度7 | 70.00% | 70.00% | 70.00% | 70.00% |
  
  

未见明显增强

去掉图片增强

  

# 第17次训练 - 引入单原子变化

(带了图像增强, 之后需去除)

  - v33: 用add_ring_op, 从v1开始, 为长度4增加ring op, 数据训练量: 53
  - v34: 用add_ring_op, 从v2+v3开始, 为长度5增加ring op, 数据训练量: 150
  - v35: 用add_ring_op, 从v4+v5开始, 为长度6增加ring op, 数据训练量: 150
  - v36: 用add_ring_op, 从v6+v7开始, 为长度7增加ring op, 数据训练量: 150
  - v37: 用relocate_atom, 从v33开始, 对长度6的ring分子进行某一个原子重定位, 数据训练量: 83
  - v38: 用relocate_atom, 从v34开始, 对长度7的ring分子进行某一个原子重定位, 数据训练量: 150
  - v39: 用relocate_atom, 从v35开始, 对长度8的ring分子进行某一个原子重定位, 数据训练量: 150
  - v40: 用relocate_atom, 从v36开始, 对长度9的ring分子进行某一个原子重定位, 数据训练量: 150
  - v41: 用change_basic_atom, 从v33开始, 对长度6的ring分子引入单原子变化, 数据训练量: 83
  - v42: 用change_basic_atom, 从v34开始, 对长度7的ring分子引入单原子变化, 数据训练量: 150
  - v43: 用change_basic_atom, 从v35开始, 对长度8的ring分子引入单原子变化, 数据训练量: 150
  - v44: 用change_basic_atom, 从v36开始, 对长度9的ring分子引入单原子变化, 数据训练量: 150  
  

|  | 追加v33-v40, epoch=1 | 追加v41-v44, epoch=1 | 追加v33-v40, epoch=2 | 追加v41-v44, epoch=2 |
| --- | --- | --- | --- | --- |
| ring分子, 长度7 | 67.50% | 77.50% | 72.50% | 70.00% |
| ring分子, 长度8 | 65.00% | 62.50% | 70.00% | 67.50% |
| ring分子, 长度9 | 40.00% | 52.50% | 45.00% | 45.00% |
| 直链, 长度7 | 100.00% | 100.00% | 94.74% | 89.47% |
| branch分子, 长度7 | 70.00% | 70.00% | 70.00% | 70.00% |
  
  

未见明显提升, 分析错误类型 (标红), 从图像上进行分析: 

```
[
    {
        "Predicted": "[N][C][O][N][Ring1][Ring2][O]",
        "GT": "[C][N][O][N][Ring1][Ring2][O]",
        "Annotation": "环内相邻原子对调"
    },
    {
        "Predicted": "[O][O][N][O][Ring1][Ring2][N]",
        "GT": "[N][N][O][O][O][Ring1][Ring2]",
        "Annotation": "更换环后原子的接入点, 或环内序列轮换"
    },
    {
        "Predicted": "[C][C][C][O][Ring1][Branch1][N]",
        "GT": "[C][O][N][C][C][Ring1][Branch1]",
        "Annotation": "ring op后移"
    },
    {
        "Predicted": "[C][N][O][C][N][Ring1][Ring2]",
        "GT": "[N][O][C][N][Ring1][Ring2][C]",
        "Annotation": "更换环后原子的接入点, 或环内序列轮换"
    },
    {
        "Predicted": "[C][C][O][C][Ring1][Ring2][C]",
        "GT": "[C][O][C][C][Ring1][Ring2][C]",
        "Annotation": "更换环后原子的接入点, 或环内序列轮换"
    },
    {
        "Predicted": "[N][O][N][O][N][Ring1][Ring2]",
        "GT": "[O][N][O][N][Ring1][Ring2][N]",
        "Annotation": "整体轮换, 或者对接入点的理解不足"
    },
    {
        "Predicted": "[C][C][C][C][Ring1][Ring2][N]",
        "GT": "[C][N][C][C][Ring1][Ring2][C]",
        "Annotation": "环后原子和环内原子对调"
    },
    {
        "Predicted": "[C][N][N][C][N][Ring1][Ring2]",
        "GT": "[C][N][N][N][C][Ring1][Ring2]",
        "Annotation": "更换环后原子的接入点, 或环内序列轮换"
    },
    {
        "Predicted": "[O][O][N][O][Ring1][Ring2][N]",
        "GT": "[O][O][O][N][Ring1][Ring2][N]",
        "Annotation": "3/4对调, 或者对接入点的理解不足"
    }
]
``` 

  

# 第18次训练 - 对ring的接入点进行变化

  - v33: 用add_ring_op, 从v1开始, 为长度4增加ring op, 数据训练量: 53
  - v34: 用add_ring_op, 从v2+v3开始, 为长度5增加ring op, 数据训练量: 150
  - v35: 用add_ring_op, 从v4+v5开始, 为长度6增加ring op, 数据训练量: 150
  - v36: 用add_ring_op, 从v6+v7开始, 为长度7增加ring op, 数据训练量: 150
  - v37: 用relocate_atom, 从v33开始, 对长度6的ring分子进行某一个原子重定位, 数据训练量: 83
  - v38: 用relocate_atom, 从v34开始, 对长度7的ring分子进行某一个原子重定位, 数据训练量: 150
  - v39: 用relocate_atom, 从v35开始, 对长度8的ring分子进行某一个原子重定位, 数据训练量: 150
  - v40: 用relocate_atom, 从v36开始, 对长度9的ring分子进行某一个原子重定位, 数据训练量: 150
  - v41: 用rotate_atoms_in_ring, 从v37开始, 对长度6的ring分子的接入点进行变化, 数据训练量: 83
  - v42: 用rotate_atoms_in_ring, 从v38开始, 对长度7的ring分子的接入点进行变化, 数据训练量: 150
  - v43: 用rotate_atoms_in_ring, 从v39开始, 对长度8的ring分子的接入点进行变化, 数据训练量: 150
  - v44: 用rotate_atoms_in_ring, 从v40开始, 对长度9的ring分子的接入点进行变化, 数据训练量: 150

|  | 追加v33-v40, epoch=1 | 追加v41-v44, epoch=1 | 再追加v41-v44, epoch=1 |
| --- | --- | --- | --- |
| ring分子, 长度7 |  | 65.00% | 75.00% |
| ring分子, 长度8 |  | 65.00% | 55.00% |
| ring分子, 长度9 |  | 45.00% | 47.50% |
| 直链, 长度7 |  | 100.00% | 94.74% |
| branch分子, 长度7 |  | 70.00% | 70.00% |
  
  

增加 v45-v48, 对v41-v44再次进行 rotate_atoms_in_ring (ring分子的接入点进行变化):

|  | 追加v33-v40, epoch=1 | 追加v41-v48, epoch=1 | 再追加v41-v48, epoch=2 | 再追加v41-v48, epoch=3 |
| --- | --- | --- | --- | --- |
| ring分子, 长度7 |  | 77.50% | 65.00% | 77.50% |
| ring分子, 长度8 |  | 65.00% | 67.50% | 60.00% |
| ring分子, 长度9 |  | 35.00% | 45.00% | 57.50% |
| 直链, 长度7 |  | 100.00% | 94.74% | 94.74% |
| branch分子, 长度7 |  | 70.00% | 70.00% | 60.00% |
  
  

整体测试: 未见明显提升

|  | 追加v33-v40, epoch=1 | 追加v41-v48, epoch=1 | 追加v33-v40, epoch=2 | 追加v41-v48, epoch=2 | 追加v33-v40, epoch=3 | 追加v41-v48, epoch=3 |
| --- | --- | --- | --- | --- | --- | --- |
| ring分子, 长度7 | 72.50% | 75.00% | 70.00% | 80.00% | 72.50% | 70.00% |
| ring分子, 长度8 | 57.50% | 52.50% | 70.00% | 55.00% | 67.50% | 67.50% |
| ring分子, 长度9 | 40.00% | 42.50% | 50.00% | 47.50% | 45.00% | 47.50% |
| 直链, 长度7 | 89.47% | 94.74% | 94.74% | 89.47% | 89.47% | 89.47% |
| branch分子, 长度7 | 70.00% | 70.00% | 70.00% | 70.00% | 70.00% | 70.00% |
  
  

(ring分子, 长度9) 的效果在提升? 研究v41-48的训练数据分布, 试试找到其与验证数据分布的差异:

发现v42中, 分子中很少出现2个[O]及以上, 但其源头 v34中, 分子中经常有多个[O], 生成算法可能会导致数据偏斜

  

对生成数据的算法进行调整: 

  1. 原来是要求selfies表达式合法, 并且规范. 否则变化无效. 现在调整成 selfies表达式合法就可以变换, 变换后的表达式再进行规范化, 以及去重.
  2. 对于rotate_atoms_in_ring, 原来只进行一步rotate, 但这样可能无法获取合法表达式, 也就无法叠加下一步rotate. 将算法调整成遍历所有rotate的可能性.

  

使用调整后的数据进行训练: 

|  | 追加v33-v40, epoch=1 | 追加v41-v48, epoch=1 | 追加v33-v40, epoch=2 | 追加v41-v48, epoch=2 | 追加v33-v40, epoch=3 | 追加v41-v48, epoch=3 |
| --- | --- | --- | --- | --- | --- | --- |
| ring分子, 长度7 | 70.00% | 72.50% | 75.00% | 75.00% | 75.00% | 72.50% |
| ring分子, 长度8 | 60.00% | 57.50% | 67.50% | 72.50% | 65.00% | 67.50% |
| ring分子, 长度9 | 40.00% | 55.00% | 52.50% | 50.00% | 57.50% | 52.50% |
| 直链, 长度7 | 94.74% | 100.00% | 100.00% | 89.47% | 100.00% | 89.47% |
| branch分子, 长度7 | 70.00% | 70.00% | 70.00% | 70.00% | 70.00% | 70.00% |
|  | 追加v33-v40, epoch=1 | 追加v41-v44, epoch=1 | 追加v33-v40, epoch=2 | 追加v41-v44, epoch=2 | 追加v33-v40, epoch=3 | 追加v41-v44, epoch=3 |
| --- | --- | --- | --- | --- | --- | --- |
| ring分子, 长度7 | 67.50% | 70.00% | 77.50% | 75.00% | 75.00% | 67.50% |
| ring分子, 长度8 | 62.50% | 52.50% | 70.00% | 57.50% | 70.00% | 52.50% |
| ring分子, 长度9 | 37.50% | 35.00% | 47.50% | 50.00% | 40.00% | 60.00% |
| 直链, 长度7 | 94.74% | 89.47% | 94.74% | 89.47% | 94.74% | 89.47% |
| branch分子, 长度7 | 70.00% | 66.67% | 70.00% | 70.00% | 70.00% | 70.00% |
  
  

在调整后的数据中, v45-v48 跟 v41-v44基本是重复的, 也就是说, 增加rotate_atoms_in_ring的数据, 提高了 (ring分子, 长度9)的准确率 (两张表中标蓝部分的对比), 但不能提高(ring分子, 长度7)的准确率  
分析(ring分子, 长度7)的错误分布 (标红): (基于图像区别的分析)

```
 [
    {
        "Predicted": "[O][C][C][O][O][Ring1][Branch1]",
        "GT": "[O][O][C][O][C][Ring1][Branch1]",
        "Annotation": "环内原子对调"
    },
    {
        "Predicted": "[N][C][N][O][Ring1][Ring2][O]",
        "GT": "[C][N][O][N][Ring1][Ring2][O]",
        "Annotation": "以结合点为起点开始写, 写成了错误的表达式. 主链原子写在了ring op后"
    },
    {
        "Predicted": "[N][N][O][O][N][O][Ring1][Ring2]",
        "GT": "[N][N][O][O][O][Ring1][Ring2]",
        "Annotation": "多一个原子? 可能是(以结合点为起点开始写, 写成了错误的表达式, 主链原子写在了整个环前面)"
    },
    {
        "Predicted": "[C][C][C][O][N][Ring1][Branch1]",
        "GT": "[O][C][N][C][C][Ring1][Branch1]",
        "Annotation": "环内原子对调"
    },
    {
        "Predicted": "[C][C][O][C][Ring1][Ring2][C]",
        "GT": "[C][O][C][C][Ring1][Ring2][C]",
        "Annotation": "以结合点为起点开始写, 写成了错误的表达式, 主链原子写在了ring op后"
    },
    {
        "Predicted": "[N][O][N][O][N][Ring1][Ring2]",
        "GT": "[O][N][O][N][Ring1][Ring2][N]",
        "Annotation": "以结合点为起点开始写, 写成了错误的表达式, 主链原子写在了ring op前"
    },
    {
        "Predicted": "[C][O][C][N][Ring1][Ring2][O]",
        "GT": "[O][C][N][C][Ring1][Ring2][O]",
        "Annotation": "以结合点为起点开始写, 写成了错误的表达式, 主链原子写在了ring op后"
    },
    {
        "Predicted": "[C][N][N][C][N][Ring1][Ring2][C]",
        "GT": "[C][N][N][N][C][Ring1][Ring2]",
        "Annotation": "多一个原子?"
    },
    {
        "Predicted": "[C][N][C][O][Ring1][Ring2][O]",
        "GT": "[O][C][N][C][O][Ring1][Ring2]",
        "Annotation": "以结合点为起点开始写, 写成了错误的表达式, 主链原子写在了ring op后"
    },
    {
        "Predicted": "[C][O][C][O][O][Ring1][Ring2]",
        "GT": "[O][C][O][C][Ring1][Ring2][O]",
        "Annotation": "以结合点为起点开始写, 写成了错误的表达式, 主链原子写在了ring op前"
    },
    {
        "Predicted": "[N][N][O][O][N][O][Ring1][Ring2]",
        "GT": "[O][O][O][N][Ring1][Ring2][N]",
        "Annotation": "多一个原子? 可能是(以结合点为起点开始写, 写成了错误的表达式, 主链原子写在了主链原子写在了整个环前面)"
    }
]
``` 

  

额外的测试, 扩大41-44数据量为300: 

|  | 追加v33-v40, epoch=1 | 追加v41-v44, epoch=1 | 追加v33-v40, epoch=2 | 追加v41-v44, epoch=2 |
| --- | --- | --- | --- | --- |
| ring分子, 长度7 | 72.50% | 60.00% | 72.50% | 72.50% |
| ring分子, 长度8 | 57.50% | 70.00% | 75.00% | 72.50% |
| ring分子, 长度9 | 37.50% | 50.00% | 55.00% | 57.50% |
| 直链, 长度7 | 94.74% | 94.74% | 100.00% | 100.00% |
| branch分子, 长度7 | 70.00% | 70.00% | 70.00% | 70.00% |
  
  

未见明显提升.

# 第19次训练 - 对ring分子增加原子

对于 "以结合点为起点开始写, 写成了错误的表达式" 的问题: 

  1. 错误的表达式 有可能是不合法的, 所以有部分是不能进入训练数据进行对比训练
  2. 对于"结合点"的不同, 通过rotate_atoms_in_ring能覆盖, 对于"主链原子"写在了不同位置的情况, 想通过在ring分子上进行add_basic_atom_to_begin/add_basic_atom_to_end来覆盖

v45-v48: 对长度6/7/8/9的ring分子, 进行add_basic_atom_to_end

v49-v52: 对长度6/7/8/9的ring分子, 进行add_basic_atom_to_begin

训练效果: 

|  | 追加v33-v44, epoch=1 | 追加v45-v52, epoch=1 | 追加v33-v44, epoch=2 | 追加v45-v52, epoch=2 |
| --- | --- | --- | --- | --- |
| ring分子, 长度7 | 70.00% | 65.00% | 70.00% | 70.00% |
| ring分子, 长度8 | 62.50% | 62.50% | 77.50% | 67.50% |
| ring分子, 长度9 | 42.50% | 52.50% | 62.50% | 55.00% |
| 直链, 长度7 | 100.00% | 94.74% | 100.00% | 94.74% |
| branch分子, 长度7 | 70.00% | 70.00% | 70.00% | 70.00% |
  
# 第20次训练 - 对整个ring分子进行rotate

尝试对整个selfies表达式进行rotate: (在错误中发现了整体rotate的错误case):

|  | 追加v33-v44, epoch=1 | 追加v45-v48, epoch=1 | 追加v45-v48, epoch=2 | 追加v45-v48, epoch=3 |
| --- | --- | --- | --- | --- |
| ring分子, 长度7 |  | 72.50% | 65.00% | 77.50% |
| ring分子, 长度8 |  | 57.50% | 45.00% | 52.50% |
| ring分子, 长度9 |  | 55.00% | 42.50% | 50.00% |
| 直链, 长度7 |  | 94.74% | 94.74% | 94.74% |
| branch分子, 长度7 |  | 70.00% | 70.00% | 70.00% |
  
分析错误: 

```
 [
    {
        "Predicted": "[C][O][N][O][O][Ring1][Branch1]",
        "GT": "[O][O][O][C][N][Ring1][Branch1]",
        "Annotation": "环内原子交换"
    },
    {
        "Predicted": "[N][C][O][N][Ring1][Ring2][O]",
        "GT": "[C][N][O][N][Ring1][Ring2][O]",
        "Annotation": "环内原子交换"
    },
    {
        "Predicted": "[N][O][O][N][O][Ring1][Ring2]",
        "GT": "[N][N][O][O][O][Ring1][Ring2]",
        "Annotation": "调整接入点(环内轮换)"
    },
    {
        "Predicted": "[C][N][O][C][N][Ring1][Ring2][Ring1][Ring2][C]",
        "GT": "[N][O][C][N][Ring1][Ring2][C]",
        "Annotation": "环内原子交换"
    },
    {
        "Predicted": "[C][C][C][O][N][Ring1][Branch1]",
        "GT": "[O][C][N][C][C][Ring1][Branch1]",
        "Annotation": "环内原子交换"
    },
    {
        "Predicted": "[C][C][O][C][Ring1][Ring2][C]",
        "GT": "[C][O][C][C][Ring1][Ring2][C]",
        "Annotation": "环内原子交换"
    },
    {
        "Predicted": "[N][O][N][O][N][Ring1][Ring2]",
        "GT": "[O][N][O][N][Ring1][Ring2][N]",
        "Annotation": "调整接入点(ring op前的原子后置)"
    },
    {
        "Predicted": "[C][O][C][N][Ring1][Ring2][O]",
        "GT": "[O][C][N][C][Ring1][Ring2][O]",
        "Annotation": "调整接入点(环内轮换)"
    },
    {
        "Predicted": "[C][N][O][C][Ring1][Ring2][O]",
        "GT": "[O][C][N][C][O][Ring1][Ring2]",
        "Annotation": "环内原子交换"
    },
    {
        "Predicted": "[N][C][O][N][O][Ring1][Branch1]",
        "GT": "[C][O][N][N][O][Ring1][Branch1]",
        "Annotation": "环内原子交换"
    },
    {
        "Predicted": "[N][O][O][N][O][Ring1][Ring2]",
        "GT": "[O][O][O][N][Ring1][Ring2][N]",
        "Annotation": "调整接入点(环内轮换, ring op前的原子后置)"
    }
]
``` 

# 第21次训练 - 对环内相邻原子进行交换

根据上一版的错误, 对环内相邻原子进行交换

v45-v48, 对于v33-v36 (add_ring_op on length 4/5/6/7), 增加swap_adjacent_atoms_in_ring

|  | 追加v33-v44, epoch=1 | 追加v45-v48, epoch=1 | 追加v45-v48, epoch=2 | 追加v45-v48, epoch=3 |
| --- | --- | --- | --- | --- |
| ring分子, 长度7 |  | 75.00% | 75.00% | 75.00% |
| ring分子, 长度8 |  | 65.00% | 67.50% | 62.50% |
| ring分子, 长度9 |  | 40.00% | 32.50% | 40.00% |
| 直链, 长度7 |  | 100.00% | 100.00% | 100.00% |
| branch分子, 长度7 |  | 70.00% | 70.00% | 70.00% |
```
[
    {
        "Predicted": "[N][C][N][O][Ring1][Ring2][O]",
        "GT": "[C][N][O][N][Ring1][Ring2][O]",
        "Annotation": "接口点错误, ring op后的原子移至原子1"
    },
    {
        "Predicted": "[N][O][O][N][O][Ring1][Ring2]",
        "GT": "[N][N][O][O][O][Ring1][Ring2]",
        "Annotation": "接口点错误, 环内轮换"
    },
    {
        "Predicted": "[C][C][O][C][Ring1][Branch1][N]",
        "GT": "[C][O][N][C][C][Ring1][Branch1]",
        "Annotation": "ring op后的原子前插, 但在图形上没有规律"
    },
    {
        "Predicted": "[C][N][O][C][N][Ring1][Ring2]",
        "GT": "[N][O][C][N][Ring1][Ring2][C]",
        "Annotation": "环内原子交换"
    },
    {
        "Predicted": "[C][C][O][C][Ring1][Ring2][C]",
        "GT": "[C][O][C][C][Ring1][Ring2][C]",
        "Annotation": "环内原子交换"
    },
    {
        "Predicted": "[N][O][N][O][N][Ring1][Ring2]",
        "GT": "[O][N][O][N][Ring1][Ring2][N]",
        "Annotation": "接口点错误, 原子1应移至ring op后"
    },
    {
        "Predicted": "[O][C][O][C][N][Ring1][Ring2][Ring1][O]",
        "GT": "[O][C][N][C][Ring1][Ring2][O]",
        "Annotation": "长度错误"
    },
    {
        "Predicted": "[C][C][C][N][Ring1][Ring2][C]",
        "GT": "[C][N][C][C][Ring1][Ring2][C]",
        "Annotation": "接口点错误, ring op后的原子移至原子1"
    },
    {
        "Predicted": "[O][N][C][N][C][Ring1][Ring1]",
        "GT": "[C][N][C][N][Ring1][Ring1][O]",
        "Annotation": "镜像([O][N][C][N][Ring1][Ring1][C]), ring op左移(成环的序列不对)"
    },
    {
        "Predicted": "[N][O][O][N][O][Ring1][Ring2]",
        "GT": "[O][O][O][N][Ring1][Ring2][N]",
        "Annotation": "镜像([N][N][O][O][O][Ring1][Ring2]), 2/4对调"
    }
]
``` 

环内原子交换的问题变少, 接口点错误在变多, 将"对整个ring分子进行rotate" 混合训练 (在之前的训练中, 体现了环内原子交换的错误, 猜测"对整个ring分子进行rotate可能有助于减少 接口点错误)

|  | 追加v33-v44, epoch=1 | 追加v45-v52, epoch=1 | 追加v45-v52, epoch=2 |
| --- | --- | --- | --- |
| ring分子, 长度7 |  | 77.50% | 72.50% |
| ring分子, 长度8 |  | 70.00% | 70.00% |
| ring分子, 长度9 |  | 47.50% | 57.50% |
| 直链, 长度7 |  | 100.00% | 100.00% |
| branch分子, 长度7 |  | 70.00% | 70.00% |
  
未见明显提升, 错误类型分布没有明显变化, 环内原子交换的问题变多, 跟单独训练"对整个ring分子进行rotate" 类似

# 第22次训练 - 变更环接口点的原子类型

v53-v56, 对于v33-v36 (add_ring_op on length 4/5/6/7), 增加change_concat_atom_of_ring

|  | 追加v33-v44, epoch=1 | 追加v53-v56, epoch=1 | 追加v53-v56, epoch=2 |
| --- | --- | --- | --- |
| ring分子, 长度7 |  | 80.00% | 75.00% |
| ring分子, 长度8 |  | 70.00% | 62.50% |
| ring分子, 长度9 |  | 55.00% | 60.00% |
| 直链, 长度7 |  | 94.74% | 84.21% |
| branch分子, 长度7 |  | 70.00% | 70.00% |
  
未见明显提升

v53-v56与v45-v48一起训练, 效果未见提升

# 思考

不论使用如何的"变化", ring分子的效果都没有明显提升, 可能的情况: 

  1. 多种长度的数据一起给, 导致"特征"比较多
  2. 在ring的基础分子中, 对于验证集的数据"大类"就没有覆盖

# 第22次训练

只给长度7的训练数据:

v2+v3 (长度5的直链) + v34 (增加ring op) + v38 (relocate_atom) + v42(rotate_atoms_in_ring) + v46(swap_adjacent_atoms_in_ring) + v50(rotate_selfies) + v54(change_concat_atom_of_ring)

训练eval的loss很低: 

{'eval_loss': 0.00052689, 'eval_acc': 1.0, 'eval_runtime': 9.8578, 'eval_samples_per_second': 15.115, 'eval_steps_per_second': 7.608, 'epoch': 2.99, 'global_step/max_steps': '222/222', 'percentage': '100.00%', 'elapsed_time': '11m 18s', 'remaining_time': '0s'}

但评估集(ring分子, 长度7)的准确率下降到 Correct: 57.50%

当前数据量: 

/opt/huangyan/mol-ai/v2-data/dataset/2.train.json:144

/opt/huangyan/mol-ai/v2-data/dataset/3.train.json:150

/opt/huangyan/mol-ai/v2-data/dataset/34.train.json:150

/opt/huangyan/mol-ai/v2-data/dataset/38.train.json:150

/opt/huangyan/mol-ai/v2-data/dataset/42.train.json:150

/opt/huangyan/mol-ai/v2-data/dataset/46.train.json:150

/opt/huangyan/mol-ai/v2-data/dataset/50.train.json:150

/opt/huangyan/mol-ai/v2-data/dataset/54.train.json:150

考虑扩大整个ring分子涉及的数据量: 

/opt/huangyan/mol-ai/v2-data/dataset/2.train.json:144

/opt/huangyan/mol-ai/v2-data/dataset/3.train.json:162

/opt/huangyan/mol-ai/v2-data/dataset/34.train.json:300

/opt/huangyan/mol-ai/v2-data/dataset/38.train.json:300

/opt/huangyan/mol-ai/v2-data/dataset/42.train.json:283

/opt/huangyan/mol-ai/v2-data/dataset/46.train.json:300

/opt/huangyan/mol-ai/v2-data/dataset/50.train.json:276

/opt/huangyan/mol-ai/v2-data/dataset/54.train.json:300

  

评估集(ring分子, 长度7) 的准确率为 Correct: 62.50%, 错误类型与之前类似, 没有任何进展

  

# 第23次训练

使用扩大后的数据集, 进行完整的ring训练: 

从第10次训练的checkpoint出发, 当前数据为: 

```
# add_ring_op on length 4/5/6/7
v33 = expand_and_save(v1, iterations=1, expansion_func=add_ring_op, version=33, balanced_ops=ring_ops,limit=300)
v34 = expand_and_save(v2 + v3, iterations=1, expansion_func=add_ring_op, version=34, balanced_ops=ring_ops, limit=300)
v35 = expand_and_save(v4 + v5, iterations=1, expansion_func=add_ring_op, version=35, balanced_ops=ring_ops, limit=300)
v36 = expand_and_save(v6 + v7, iterations=1, expansion_func=add_ring_op, version=36, balanced_ops=ring_ops, limit=300)

# relocate_atom on ring length 6/7/8/9
v37 = expand_and_save(v33, iterations=1, expansion_func=relocate_atom, version=37, balanced_ops=ring_ops, limit=300)
v38 = expand_and_save(v34, iterations=1, expansion_func=relocate_atom, version=38, balanced_ops=ring_ops, limit=300)
v39 = expand_and_save(v35, iterations=1, expansion_func=relocate_atom, version=39, balanced_ops=ring_ops, limit=300)
v40 = expand_and_save(v36, iterations=1, expansion_func=relocate_atom, version=40, balanced_ops=ring_ops, limit=300)

# rotate_atoms_in_ring on ring length 6/7/8/9
v41 = expand_and_save(v33, iterations=1, expansion_func=rotate_atoms_in_ring, version=41, balanced_ops=ring_ops, limit=300)
v42 = expand_and_save(v34, iterations=1, expansion_func=rotate_atoms_in_ring, version=42, balanced_ops=ring_ops, limit=300)
v43 = expand_and_save(v35, iterations=1, expansion_func=rotate_atoms_in_ring, version=43, balanced_ops=ring_ops, limit=300)
v44 = expand_and_save(v36, iterations=1, expansion_func=rotate_atoms_in_ring, version=44, balanced_ops=ring_ops, limit=300)

v45 = expand_and_save(v33, iterations=1, expansion_func=swap_adjacent_atoms_in_ring, version=45, limit=300)
v46 = expand_and_save(v34, iterations=1, expansion_func=swap_adjacent_atoms_in_ring, version=46, limit=300)
v47 = expand_and_save(v35, iterations=1, expansion_func=swap_adjacent_atoms_in_ring, version=47, limit=300)
v48 = expand_and_save(v36, iterations=1, expansion_func=swap_adjacent_atoms_in_ring, version=48, limit=300)

v49 = expand_and_save(v33, iterations=1, expansion_func=rotate_selfies, version=49, limit=300)
v50 = expand_and_save(v34, iterations=1, expansion_func=rotate_selfies, version=50, limit=300)
v51 = expand_and_save(v35, iterations=1, expansion_func=rotate_selfies, version=51, limit=300)
v52 = expand_and_save(v36, iterations=1, expansion_func=rotate_selfies, version=52, limit=300)

v53 = expand_and_save(v33, iterations=1, expansion_func=change_concat_atom_of_ring, version=53, limit=300)
v54 = expand_and_save(v34, iterations=1, expansion_func=change_concat_atom_of_ring, version=54, limit=300)
v55 = expand_and_save(v35, iterations=1, expansion_func=change_concat_atom_of_ring, version=55, limit=300)
v56 = expand_and_save(v36, iterations=1, expansion_func=change_concat_atom_of_ring, version=56, limit=300)
```

|  | 追加v33-v56, epoch=1 | 追加v33-v56, epoch=2 | 追加v33-v56, epoch=3 |  |  |  |
| --- | --- | --- | --- | --- | --- | --- |
| ring分子, 长度7 | 75.00% | 67.50% | 72.50% |  |  |  |
| ring分子, 长度8 | 80.00% | 70.00% | 80.00% |  |  |  |
| ring分子, 长度9 | 60.00% | 60.00% | 60.00% |  |  |  |
| 直链, 长度7 | 94.74% | 94.74% | 94.74% |  |  |  |
| branch分子, 长度7 | 70.00% | 70.00% | 70.00% |  |  |  |
|  | 追加v33-v40, epoch=1 | 追加v45-v48,v53-v56, epoch=1 | 追加v33-v40, epoch=2 | 追加v45-v48,v53-v56, epoch=2 | 追加v33-v40, epoch=3 | 追加v45-v48,v53-v56, epoch=3 |
| --- | --- | --- | --- | --- | --- | --- |
| ring分子, 长度7 | 77.50% | 70.00% | 67.50% | 72.50% | 70.00% | 80.00% |
| ring分子, 长度8 | 70.00% | 65.00% | 80.00% | 77.50% | 75.00% | 80.00% |
| ring分子, 长度9 | 55.00% | 52.50% | 62.50% | 52.50% | 60.00% | 50.00% |
| 直链, 长度7 | 89.47% | 100.00% | 94.74% | 94.74% | 94.74% | 89.47% |
| branch分子, 长度7 | 70.00% | 70.00% | 70.00% | 70.00% | 70.00% | 70.00% |
  
两种训练方法, 都没看到明显的提升.

新的checkpoint: 

  - 追加v33-v56, epoch=1: /opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v1271-20250227-222946/checkpoint-1068
  - 追加v33-v40, epoch=1: /opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v1274-20250228-013219/checkpoint-366

分析错误类型: 

```
 [
    {
        "Predicted": "[N][O][O][N][O][Ring1][Ring2]",
        "GT": "[N][N][O][O][O][Ring1][Ring2]",
        "Annotation": "非法selfies, 环变成了直链"
    },
    {
        "Predicted": "[C][C][O][C][N][Ring1][Branch1]",
        "GT": "[C][O][N][C][C][Ring1][Branch1]",
        "Annotation": "环内原子对调"
    },
    {
        "Predicted": "[C][O][C][N][N][Ring1][Ring2]",
        "GT": "[N][O][C][N][Ring1][Ring2][C]",
        "Annotation": "非法selfies, 环变成了直链"
    },
    {
        "Predicted": "[N][O][N][O][N][Ring1][Ring2]",
        "GT": "[O][N][O][N][Ring1][Ring2][N]",
        "Annotation": "非法selfies, 环变成了直链"
    },
    {
        "Predicted": "[N][C][N][Ring1][Ring1][O][N]",
        "GT": "[N][C][N][N][Ring1][Ring1][O]",
        "Annotation": "环左右两个接入点, 识别成了一个"
    },
    {
        "Predicted": "[C][N][C][O][O][Ring1][Ring2][Ring3]",
        "GT": "[O][C][N][C][O][Ring1][Ring2]",
        "Annotation": "ring op识别错误, 结合点错误"
    },
    {
        "Predicted": "[C][O][C][O][O][Ring1][Ring2]",
        "GT": "[O][C][O][C][Ring1][Ring2][O]",
        "Annotation": "非法selfies, 环变成了直链"
    },
    {
        "Predicted": "[N][O][O][N][O][Ring1][Ring2]",
        "GT": "[O][O][O][N][Ring1][Ring2][N]",
        "Annotation": "非法selfies, 环变成了直链"
    }
]
``` 

  1. 非法selfies的问题, 大部分是因为环的结合点错误, 导致表达式最后的ring op被删除, 变成了直链. 考虑在规范化的时候, 容许这种情况.   
  

# 第24次训练

在规范化的时候, 容许环selfies解析成直链的情况, 进行训练: 

|  | 追加v33-v40, epoch=1 | 追加v45-v48, epoch=1 | 追加v53-v56, epoch=1 | 追加v45-v48, epoch=2 | 追加v53-v56, epoch=2 |
| --- | --- | --- | --- | --- | --- |
| ring分子, 长度7 |  | 80.00% | 67.50% | 70.00% | 65.00% |
| ring分子, 长度8 |  | 62.50% | 70.00% | 67.50% | 77.50% |
| ring分子, 长度9 |  | 47.50% | 57.50% | 50.00% | 57.50% |
| 直链, 长度7 |  | 94.74% | 100.00% | 94.74% | 89.47% |
| branch分子, 长度7 |  | 70.00% | 70.00% | 70.00% | 70.00% |
  
未见明显提升, 分析错误类型: 

```
[
    {
        "Predicted": "[O][C][C][O][O][Ring1][Branch1]",
        "GT": "[O][O][C][O][C][Ring1][Branch1]",
        "Annotation": "环内原子交换"
    },
    {
        "Predicted": "[N][C][N][O][Ring1][Ring2][O]",
        "GT": "[C][N][O][N][Ring1][Ring2][O]",
        "Annotation": "接入点错误, 接入点应从ring op后, 变更到第一原子"
    },
    {
        "Predicted": "[N][O][O][N][O][Ring1][Ring2]",
        "GT": "[N][N][O][O][O][Ring1][Ring2]",
        "Annotation": "接入点错误, 接入点应从第一原子变更到第三原子"
    },
    {
        "Predicted": "[C][N][O][C][N][Ring1][Ring2]",
        "GT": "[N][O][C][N][Ring1][Ring2][C]",
        "Annotation": "接入点错误, 接入点应从第一原子变更到ring op后"
    },
    {
        "Predicted": "[C][C][O][C][C][Ring1][Ring2]",
        "GT": "[C][O][C][C][Ring1][Ring2][C]",
        "Annotation": "接入点错误, 接入点应从第一原子变更到ring op后"
    },
    {
        "Predicted": "[N][O][N][O][Ring1][Ring2][N]",
        "GT": "[O][N][O][N][Ring1][Ring2][N]",
        "Annotation": "接入点错误, 接入点应从ring op后, 变更到第一原子"
    },
    {
        "Predicted": "[N][C][C][N][N][Ring1][Ring2]",
        "GT": "[N][N][N][C][C][Ring1][Ring2]",
        "Annotation": "接入点错误, 接入点应从第一原子变更到ring op后"
    },
    {
        "Predicted": "[N][O][O][N][O][Ring1][Ring2]",
        "GT": "[O][O][O][N][Ring1][Ring2][N]",
        "Annotation": "接入点错误, 接入点应从第一原子变更到第三原子"
    }
]
``` 

还是在面对接入点的错误, 尝试构建新的接入点数据: 

对于一个环, 将新的原子作为接入点, 插入位置:

  1. 第一原子 (接入点在环的位置1)
  2. ring op后 (接入点在环的位置n)

编写算法后, 发现: 

```
bfs_selfies_expansion([SelfiesRecord("", "", "[O][N][O][N][Ring1][Ring2][N]")], change_position_of_concat_atom_of_ring)
 
是能正确将最后一个原子[N], 变换到最前面, 然后因为ring接入点的键不合理, selfies会放弃掉ring op, 最终结果: 
 
[SelfiesRecord(old='[O][N][O][N][Ring1][Ring2][N]', extension='change_position_of_concat_atom_of_ring', new='[N][O][N][O][N]')]
 
这是我们想看到的结果 (因为接入点错误, ring op被放弃掉, 形成了直链分子)
``` 

但在生成的训练数据中, 所有的数据都带有ring op, 并没有直链分子

分析:

  - 在长度5的直链分子中, 因为生成后都进行了规范化, 所以导致其分布偏差 (尽量以C开头, 其次是N, 最后是O, 共价键从大到小, C=4, N=3, O=2). 在生成ring数据时, 添加ring op就导致了 ring op的左侧以C开头居多, 
  - 对于可能生成直链的变换: [A][B][C][D][Ring1][Ring2][X] 变换为 [X][A][B][C][D][Ring1][Ring2], 因为[A]都是以C开头 (在直链中, 以共价键高的原子开头), 所以变换后, 不会发生共价键不够用的情况

需要在添加ring op时, 对分子镜像也添加一套

# 第25次训练 - 在添加ring op时, 对分子镜像也添加一次

需要在添加ring op时, 对分子镜像也添加一套. 以这一套ring分子作为基底, 产生后续的变换.

先测试调整后的 v53-v56 (change_position_of_concat_atom_of_ring)

|  | 追加v33-v40, epoch=1 | 追加v53-v56, epoch=1 | 追加v53-v56, epoch=2 | 追加v53-v56, epoch=3 |
| --- | --- | --- | --- | --- |
| ring分子, 长度7 |  | 75.00% | 80.00% | 82.50% |
| ring分子, 长度8 |  | 75.00% | 75.00% | 70.00% |
| ring分子, 长度9 |  | 62.50% | 65.00% | 65.00% |
| 直链, 长度7 |  | 89.47% | 100.00% | 100.00% |
| branch分子, 长度7 |  | 66.67% | 70.00% | 66.67% |
  
分析错误 (标红):

```
[
    {
        "Predicted": "[O][C][C][O][O][Ring1][Branch1]",
        "GT": "[O][O][C][O][C][Ring1][Branch1]",
        "Annotation": "环内原子交换"
    },
    {
        "Predicted": "[O][N][C][O][N][Ring1][Ring2]",
        "GT": "[C][N][O][N][Ring1][Ring2][O]",
        "Annotation": "环内原子交换"
    },
    {
        "Predicted": "[C][C][C][N][Ring1][Ring2][O]",
        "GT": "[O][C][N][C][C][Ring1][Ring2]",
        "Annotation": "接入点错误(ring op后的接入点应当移到最前)"
    },
    {
        "Predicted": "[C][N][O][C][N][Ring1][Ring2]",
        "GT": "[N][O][C][N][Ring1][Ring2][C]",
        "Annotation": "环内原子交换"
    },
    {
        "Predicted": "[C][C][C][O][N][Ring1][Branch1]",
        "GT": "[O][C][N][C][C][Ring1][Branch1]",
        "Annotation": "环内原子交换"
    },
    {
        "Predicted": "[C][C][O][C][Ring1][Ring2][C]",
        "GT": "[C][O][C][C][Ring1][Ring2][C]",
        "Annotation": "环内原子交换"
    },
    {
        "Predicted": "[C][N][C][O][Ring1][Ring2][O]",
        "GT": "[O][C][N][C][O][Ring1][Ring2]",
        "Annotation": "接入点错误(ring op后的接入点应当移到最前)"
    }
]
``` 

将ring分子识别成直链 的问题消失. 但仍然有两个接入点错误, 其他都是环内原子交换

该版本的checkpoint: /opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v1296-20250228-142442/checkpoint-156

在其上修复环内相邻原子交换

# 重新思考分析

经过几次增强selfies"变化"种类的尝试, 发现没有作用. 判断是训练数据偏斜, 导致模型"不认识" 校验错误的数据, 再怎么增强也无法将逻辑推广到 这些数据

查找错误数据在训练数据中的分布, 举例: 

  - 错误数据: [C][N][O][N][Ring1][Ring2][O], 其在训练数据中的是等价形式"[O][N][C][N][O][Ring1][Ring2]" (经过规范化), 但两者图像不同: 
    - ![image]() 和 ![image2025-2-28 18:55:48.png](/assets/01KJBZRQQVCZ5ZSZZ4W1F9974D/image2025-2-28%2018%3A55%3A48.png)
    - 两者在x轴上成镜像
  - 错误数据: [N][O][C][N][Ring1][Ring2][C], 其在训练数据中的是等价形式"[C][N][C][O][N][Ring1][Ring2]" (经过规范化), 但两者图像不同:
    - ![image2025-2-28 18:57:37.png](/assets/01KJBZRQQVCZ5ZSZZ4W1F9974D/image2025-2-28%2018%3A57%3A37.png) 和 ![image2025-2-28 18:57:49.png](/assets/01KJBZRQQVCZ5ZSZZ4W1F9974D/image2025-2-28%2018%3A57%3A49.png)
    - 两者在x/y轴上都成镜像

# 第26次训练 - 增加训练数据(图形沿x/y轴进行翻转)

增强算法: 

  - 图形沿x轴翻转/图形沿y轴翻转/图形沿x+y轴翻转, 三种翻转算法随机选一种, 生成增强图片

commit: 457ad2cea6c5275773c7c09757cbecfc38102802

在/opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v1274-20250228-013219/checkpoint-366上, 训练v53-v56, 评估集(ring分子, 长度7)的准确率提升到90%

数据增强(图形沿x/y轴进行翻转), 这种训练方法有效

# 重新建立基线

所有数据带数据增强(图形沿x/y轴进行翻转). 

训练步骤: v1-v32: 直链分子

数据量:

```
/opt/huangyan/mol-ai/v2-data/dataset/1.train.json:97
/opt/huangyan/mol-ai/v2-data/dataset/1.coord_flip_enhanced.train.json:97
/opt/huangyan/mol-ai/v2-data/dataset/2.train.json:144
/opt/huangyan/mol-ai/v2-data/dataset/2.coord_flip_enhanced.train.json:144
/opt/huangyan/mol-ai/v2-data/dataset/3.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/3.coord_flip_enhanced.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/4.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/4.coord_flip_enhanced.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/5.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/5.coord_flip_enhanced.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/6.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/6.coord_flip_enhanced.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/7.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/7.coord_flip_enhanced.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/8.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/8.coord_flip_enhanced.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/9.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/9.coord_flip_enhanced.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/10.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/10.coord_flip_enhanced.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/11.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/11.coord_flip_enhanced.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/12.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/12.coord_flip_enhanced.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/13.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/13.coord_flip_enhanced.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/14.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/14.coord_flip_enhanced.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/15.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/15.coord_flip_enhanced.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/16.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/16.coord_flip_enhanced.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/17.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/17.coord_flip_enhanced.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/18.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/18.coord_flip_enhanced.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/19.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/19.coord_flip_enhanced.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/20.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/20.coord_flip_enhanced.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/21.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/21.coord_flip_enhanced.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/22.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/22.coord_flip_enhanced.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/23.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/23.coord_flip_enhanced.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/24.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/24.coord_flip_enhanced.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/25.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/25.coord_flip_enhanced.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/26.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/26.coord_flip_enhanced.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/27.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/27.coord_flip_enhanced.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/28.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/28.coord_flip_enhanced.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/29.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/29.coord_flip_enhanced.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/30.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/30.coord_flip_enhanced.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/31.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/31.coord_flip_enhanced.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/32.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/32.coord_flip_enhanced.train.json:150 
``` 

checkpoint: /opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v0-20250301-112246/checkpoint-1776

epoch=1, 训练效果: 

|  | 追加v1-v32, epoch=1 |
| --- | --- |
| ring分子, 长度7 | 0.00% |
| ring分子, 长度8 | 0.00% |
| ring分子, 长度9 | 0.00% |
| 直链, 长度7 | 100.00% |
| branch分子, 长度7 | 70.00% |
| 直链, 长度7 | 100.00% |
| 直链, 长度8 | 100.00% |
| 直链, 长度9 | 100.00% |
| 直链, 长度10 | 100.00% |
| 直链, 长度11 | 95.00% |
| 直链, 长度12 | 85.00% |
| 直链, 长度13 | 90.00% |
| 直链, 长度14 | 95.00% |
| 直链, 长度15 | 75.00% |
  
# 第27次训练 - 在新基线上训练ring分子识别

在以上基线上:

  - 追加v33-v36 (add_ring_op)

```
/opt/huangyan/mol-ai/v2-data/dataset/33.train.json:90
/opt/huangyan/mol-ai/v2-data/dataset/33.coord_flip_enhanced.train.json:90
/opt/huangyan/mol-ai/v2-data/dataset/34.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/34.coord_flip_enhanced.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/35.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/35.coord_flip_enhanced.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/36.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/36.coord_flip_enhanced.train.json:150
```

    - checkpoint: /opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v1-20250301-130127/checkpoint-201

  - 追加v53-v56 (change_position_of_concat_atom_of_ring)

```
 /opt/huangyan/mol-ai/v2-data/dataset/53.train.json:13
/opt/huangyan/mol-ai/v2-data/dataset/53.coord_flip_enhanced.train.json:13
/opt/huangyan/mol-ai/v2-data/dataset/54.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/54.coord_flip_enhanced.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/55.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/55.coord_flip_enhanced.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/56.train.json:150
/opt/huangyan/mol-ai/v2-data/dataset/56.coord_flip_enhanced.train.json:150
```

    - checkpoint: /opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v2-20250301-131429/checkpoint-171

训练效果: 

|  | 追加v1-v32, epoch=1 | 追加v33-v36, epoch=1 | 追加v53-v56, epoch=1 |
| --- | --- | --- | --- |
| ring分子, 长度7 | 0.00% | 82.50% | 92.50% |
| ring分子, 长度8 | 0.00% | 65.00% | 82.50% |
| ring分子, 长度9 | 0.00% | 52.50% | 70.00% |
| 直链, 长度7 | 100.00% | 100.00% | 94.74% |
| branch分子, 长度7 | 70.00% | 70.00% | 70.00% |
  
(ring分子, 长度7) 已达标, 研究(ring分子, 长度8)的错误分布: 

```
[
    {
        "Predicted": "[N][C][O][C][N][O][Ring1][=Branch1]",
        "GT": "[O][N][C][N][O][C][Ring1][=Branch1]",
        "Annotation": "环内原子交换*2"
    },
    {
        "Predicted": "[N][N][O][C][N][O][Ring1][Ring2]",
        "GT": "[O][C][N][O][N][Ring1][Branch1][N]",
        "Annotation": "ring长度错误 (变直链)"
    },
    {
        "Predicted": "[C][O][O][O][N][O][Ring1][=Branch1]",
        "GT": "[C][O][O][N][O][O][Ring1][=Branch1]",
        "Annotation": "环内原子交换"
    },
    {
        "Predicted": "[C][N][O][C][O][O][Ring1][=Branch1]",
        "GT": "[O][O][N][C][O][C][Ring1][=Branch1]",
        "Annotation": "环内原子交换"
    },
    {
        "Predicted": "[C][C][O][C][C][O][Ring1][=Branch1]",
        "GT": "[C][O][C][C][C][O][Ring1][=Branch1]",
        "Annotation": "环内原子交换"
    },
    {
        "Predicted": "[C][N][N][C][N][N][Ring1][Branch1]",
        "GT": "[C][N][N][N][Ring1][Ring2][N][C]",
        "Annotation": "ring op错误"
    },
    {
        "Predicted": "[N][C][C][O][O][C][Ring1][Ring2]",
        "GT": "[O][C][O][C][Ring1][Ring2][C][N]",
        "Annotation": "环内原子交换"
    }
]
``` 

增加v53-v56的数据量: 

|  | v1-v32, epoch=1 | 追加v33-v36, epoch=1 | 追加v53-v56 (300数据), epoch=1 |
| --- | --- | --- | --- |
| ring分子, 长度7 |  | 80.00% | 92.50% |
| ring分子, 长度8 |  | 65.00% | 80.00% |
| ring分子, 长度9 |  | 57.50% | 77.50% |
| 直链, 长度7 |  | 100.00% | 84.21% |
| branch分子, 长度7 |  | 70.00% | 70.00% |
  
未见明显提升. 分析错误: (全都是环内原子交换)

```
 [
    {
        "Predicted": "[N][C][O][C][N][O][Ring1][=Branch1]",
        "GT": "[O][N][C][N][O][C][Ring1][=Branch1]",
        "Annotation": "环内原子交换*2"
    },
    {
        "Predicted": "[C][O][O][O][N][O][Ring1][=Branch1]",
        "GT": "[C][O][O][N][O][O][Ring1][=Branch1]",
        "Annotation": "环内原子交换"
    },
    {
        "Predicted": "[C][O][C][O][N][O][Ring1][=Branch1]",
        "GT": "[O][O][N][C][O][C][Ring1][=Branch1]",
        "Annotation": "环内原子交换"
    },
    {
        "Predicted": "[C][C][C][N][C][Ring1][Ring2][O]",
        "GT": "[C][C][N][C][C][Ring1][Ring2][O]",
        "Annotation": "环内原子交换"
    },
    {
        "Predicted": "[C][O][C][O][C][O][Ring1][=Branch1]",
        "GT": "[C][O][C][O][O][C][Ring1][=Branch1]",
        "Annotation": "环内原子交换"
    },
    {
        "Predicted": "[C][C][C][O][C][N][Ring1][Branch1]",
        "GT": "[O][C][C][N][C][Ring1][Branch1][C]",
        "Annotation": "环内原子交换"
    },
    {
        "Predicted": "[C][C][O][C][C][O][Ring1][=Branch1]",
        "GT": "[C][O][C][C][C][O][Ring1][=Branch1]",
        "Annotation": "环内原子交换"
    },
    {
        "Predicted": "[N][C][C][O][C][O][Ring1][Branch1]",
        "GT": "[N][C][C][O][O][C][Ring1][Branch1]",
        "Annotation": "环内原子交换"
    }
]
``` 

checkpoint: /opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v10-20250301-215111/checkpoint-315

增加v45-v48 (swap_adjacent_atoms_in_ring): 未见明显提升, 但错误类型变化 (回到了接入点错误): 

|| v1-v32, 追加v33-v36, 追加v53-v56 (300数据)  
| epoch=1 | 追加v45-v48, epoch=1 |  |
| --- | --- | --- |
| ring分子, 长度7 |  | 92.50% |
| ring分子, 长度8 |  | 82.50% |
| ring分子, 长度9 |  | 82.50% |
| 直链, 长度7 |  | 94.74% |
| branch分子, 长度7 |  | 70.00% |
```
[
    {
        "Predicted": "[C][N][C][O][O][N][Ring1][=Branch1]",
        "GT": "[O][N][C][N][O][C][Ring1][=Branch1]",
        "Annotation": "环内原子交换"
    },
    {
        "Predicted": "[C][C][N][C][O][Ring1][Ring1][O]",
        "GT": "[C][C][N][O][C][Ring1][Ring1][O]",
        "Annotation": "接入点错误"
    },
    {
        "Predicted": "[O][C][N][C][O][O][Ring1][Ring2]",
        "GT": "[O][C][O][N][Ring1][Ring2][C][O]",
        "Annotation": "环内原子交换"
    },
    {
        "Predicted": "[O][C][O][C][O][Ring1][Ring1][O]",
        "GT": "[O][C][O][C][Ring1][Ring1][O][O]",
        "Annotation": "接入点错误"
    },
    {
        "Predicted": "[O][C][N][O][O][Ring1][Ring2][O]",
        "GT": "[O][C][O][O][N][Ring1][Ring2][O]",
        "Annotation": "接入点错误"
    },
    {
        "Predicted": "[C][N][C][O][O][Ring1][Ring2][O]",
        "GT": "[O][C][O][O][N][Ring1][Ring2][C]",
        "Annotation": "接入点错误"
    },
    {
        "Predicted": "[C][N][O][N][Ring1][Ring2][O]",
        "GT": "[C][N][O][O][N][Ring1][Ring2][O]",
        "Annotation": "接入点错误"
    }
]
``` 

将v45-v48和v53-v56合并在一起训练: 

|| v1-v32, 追加v33-v36, 追加v53-v56 (300数据)  
| epoch=1 | 追加v45-v48和v53-v56, epoch=1 |  |
| --- | --- | --- |
| ring分子, 长度7 |  | 97.50% |
| ring分子, 长度8 |  | 92.50% |
| ring分子, 长度9 |  | 85.00% |
| 直链, 长度7 |  | 94.74% |
| branch分子, 长度7 |  | 70.00% |
  
  
(ring分子, 长度8) 达标. 分析(ring分子, 长度9)的错误分布: 

```
 [
    {
        "Predicted": "[O][C][C][O][O][N][O][Ring1][=Branch1]",
        "GT": "[N][O][O][O][C][C][Ring1][=Branch1][O]",
        "Annotation": "环内原子交换"
    },
    {
        "Predicted": "[N][N][O][C][C][N][Ring1][Ring1][O]",
        "GT": "[O][N][C][C][O][N][Ring1][Ring1][N]",
        "Annotation": "主链正确, 环位置错误, ring op平移2位"
    },
    {
        "Predicted": "[C][C][C][C][O][Ring1][Ring1][N][C]",
        "GT": "[C][N][C][O][C][Ring1][Ring1][C][C]",
        "Annotation": "接入点更换 (环内接入点原子轮换)"
    },
    {
        "Predicted": "[N][C][C][O][C][O][C][Ring1][=Branch1]",
        "GT": "[N][C][O][C][O][C][C][Ring1][=Branch1]",
        "Annotation": "接入点更换 (环内接入点原子轮换)"
    },
    {
        "Predicted": "[C][O][C][C][C][N][C][Ring1][Ring2]",
        "GT": "[C][O][C][C][C][C][N][Ring1][Ring2]",
        "Annotation": "环内原子交换或轮换"
    },
    {
        "Predicted": "[N][N][C][C][C][C][Ring1][Ring2][O]",
        "GT": "[O][C][N][C][C][C][Ring1][Ring2][N]",
        "Annotation": "复杂错误"
    }
]
```

|| v1-v32, 追加v33-v36, 追加v53-v56 (300数据)  
| epoch=1 | 追加v45-v48和v53-v56, 300数据, epoch=1 |  |
| --- | --- | --- |
| ring分子, 长度7 |  | 100.00% |
| ring分子, 长度8 |  | 90.00% |
| ring分子, 长度9 |  | 87.50% |
| 直链, 长度7 |  | 100.00% |
| branch分子, 长度7 |  | 70.00% |
  
使用300数据, 未见明显提升

可尝试环内原子轮换

  

# 第28次训练 - 环内原子轮换

增加环内原子轮换

|| v1-v32, 追加v33-v36, 追加v53-v56 (300数据)  
| epoch=1 | 追加v45-v48和v53-v56, epoch=1 | 追加v41-v44, epoch=1 |  |
| --- | --- | --- | --- |
| ring分子, 长度7 |  | 100.00% | 97.50% |
| ring分子, 长度8 |  | 92.50% | 92.50% |
| ring分子, 长度9 |  | 85.00% | 85.00% |
| 直链, 长度7 |  | 94.74% | 100.00% |
| branch分子, 长度7 |  | 70.00% | 70.00% |
|| v1-v32, 追加v33-v36, 追加v53-v56 (300数据)  
| epoch=1 | 追加v45-v48和v53-v56和v41-v44, epoch=1 |  |
| --- | --- | --- |
| ring分子, 长度7 |  | 100.00% |
| ring分子, 长度8 |  | 87.50% |
| ring分子, 长度9 |  | 95.00% |
| 直链, 长度7 |  | 94.74% |
| branch分子, 长度7 |  | 70.00% |
  
  

混合训练比分步训练的效果好.

混合训练后, (ring分子, 长度9)达标, (ring分子, 长度8)下降了, 分析错误分布: 

```
[
    {
        "Predicted": "[C][N][C][O][O][N][Ring1][=Branch1]",
        "GT": "[O][N][C][N][O][C][Ring1][=Branch1]",
        "Annotation": "环内原子交换"
    },
    {
        "Predicted": "[C][C][N][C][O][Ring1][Ring1][O]",
        "GT": "[C][C][N][O][C][Ring1][Ring1][O]",
        "Annotation": "接入点错误(环内原子交换)"
    },
    {
        "Predicted": "[C][N][C][O][O][Ring1][Ring2][O]",
        "GT": "[O][C][O][O][N][Ring1][Ring2][C]",
        "Annotation": "接入点错误(环内原子交换)"
    },
    {
        "Predicted": "[N][C][C][C][O][O][Ring1][Ring2]",
        "GT": "[O][C][O][C][Ring1][Ring2][C][N]",
        "Annotation": "环内原子交换"
    },
    {
        "Predicted": "[O][N][C][N][N][O][Ring1][Branch1]",
        "GT": "[O][N][C][N][N][O][Ring1][Ring2]",
        "Annotation": "环长度错误"
    }
]
``` 

增大数据量后, 可能能修复. 

继续增长ring长度, 涉及以下训练集: 

  - v33-v36: add_ring_op
  - v41-v44: rotate_atoms_in_ring
  - v45-v48: swap_adjacent_atoms_in_ring
  - v53-v56: change_position_of_concat_atom_of_ring

  

# 第29次训练 - 增加长度10的ring分子

增加长度10的训练数据: 

```
# ring length 10
v57 = expand_and_save(v8 + v9, iterations=1, expansion_func=add_ring_op, version=57, balanced_ops=ring_ops, limit=300)
v58 = expand_and_save(v57, iterations=1, expansion_func=rotate_atoms_in_ring, version=58, balanced_ops=ring_ops, limit=300)
v59 = expand_and_save(v57, iterations=1, expansion_func=swap_adjacent_atoms_in_ring, version=59, limit=300)
v60 = expand_and_save(v57, iterations=1, expansion_func=change_position_of_concat_atom_of_ring, version=60, limit=300)
```

|| v1-v32, 追加v33-v36, 追加v53-v56 (300数据)  
追加v45-v48和v53-v56和v41-v44  
| epoch=1 | 追加v57-v60, epoch=1 |  |
| --- | --- | --- |
| ring分子, 长度7 |  | 92.50% |
| ring分子, 长度8 |  | 82.50% |
| ring分子, 长度9 |  | 100.00% |
| ring分子, 长度10 |  | 90.00% |
| ring分子, 长度11 |  | 62.50% |
| ring分子, 长度12 |  | 30.00% |
| 直链, 长度7 |  | 89.47% |
| branch分子, 长度7 |  | 70.00% |
  
  

# 第30次训练 - 增加ring分子的训练长度

数据版本: 

```
# ring length 10
v57 = expand_and_save(v8 + v9, iterations=1, expansion_func=add_ring_op, version=57, balanced_ops=ring_ops, limit=300)
v58 = expand_and_save(v57, iterations=1, expansion_func=rotate_atoms_in_ring, version=58, balanced_ops=ring_ops, limit=300)
v59 = expand_and_save(v57, iterations=1, expansion_func=swap_adjacent_atoms_in_ring, version=59, limit=300)
v60 = expand_and_save(v57, iterations=1, expansion_func=change_position_of_concat_atom_of_ring, version=60, limit=300)

# ring length 11
v61 = expand_and_save(v10 + v11, iterations=1, expansion_func=add_ring_op, version=61, balanced_ops=ring_ops, limit=300)
v62 = expand_and_save(v61, iterations=1, expansion_func=rotate_atoms_in_ring, version=62, balanced_ops=ring_ops, limit=300)
v63 = expand_and_save(v61, iterations=1, expansion_func=swap_adjacent_atoms_in_ring, version=63, limit=300)
v64 = expand_and_save(v61, iterations=1, expansion_func=change_position_of_concat_atom_of_ring, version=64, limit=300)

# ring length 12
v65 = expand_and_save(v15 + v16, iterations=1, expansion_func=add_ring_op, version=65, balanced_ops=ring_ops, limit=300)
v66 = expand_and_save(v65, iterations=1, expansion_func=rotate_atoms_in_ring, version=66, balanced_ops=ring_ops, limit=300)
v67 = expand_and_save(v65, iterations=1, expansion_func=swap_adjacent_atoms_in_ring, version=67, limit=300)
v68 = expand_and_save(v65, iterations=1, expansion_func=change_position_of_concat_atom_of_ring, version=68, limit=300)

# ring length 13
v69 = expand_and_save(v18 + v19, iterations=1, expansion_func=add_ring_op, version=69, balanced_ops=ring_ops, limit=300)
v70 = expand_and_save(v69, iterations=1, expansion_func=rotate_atoms_in_ring, version=70, balanced_ops=ring_ops, limit=300)
v71 = expand_and_save(v69, iterations=1, expansion_func=swap_adjacent_atoms_in_ring, version=71, limit=300)
v72 = expand_and_save(v69, iterations=1, expansion_func=change_position_of_concat_atom_of_ring, version=72, limit=300)

# ring length 14
v73 = expand_and_save(v21 + v22, iterations=1, expansion_func=add_ring_op, version=73, balanced_ops=ring_ops, limit=300)
v74 = expand_and_save(v73, iterations=1, expansion_func=rotate_atoms_in_ring, version=74, balanced_ops=ring_ops, limit=300)
v75 = expand_and_save(v73, iterations=1, expansion_func=swap_adjacent_atoms_in_ring, version=75, limit=300)
v76 = expand_and_save(v73, iterations=1, expansion_func=change_position_of_concat_atom_of_ring, version=76, limit=300)

# ring length 15
v77 = expand_and_save(v24 + v25, iterations=1, expansion_func=add_ring_op, version=77, balanced_ops=ring_ops, limit=300)
v78 = expand_and_save(v77, iterations=1, expansion_func=rotate_atoms_in_ring, version=78, balanced_ops=ring_ops, limit=300)
v79 = expand_and_save(v77, iterations=1, expansion_func=swap_adjacent_atoms_in_ring, version=79, limit=300)
v80 = expand_and_save(v77, iterations=1, expansion_func=change_position_of_concat_atom_of_ring, version=80, limit=300)

``` 

训练效果: 

|| v1-v32, 追加v33-v36, 追加v53-v56 (300数据)  
追加v45-v48和v53-v56和v41-v44  
| epoch=1 | 追加v57-v80, epoch=1 |  |
| --- | --- | --- |
| ring分子, 长度7 |  | 97.50% |
| ring分子, 长度8 |  | 87.50% |
| ring分子, 长度9 |  | 95.00% |
| ring分子, 长度10 |  | 95.00% |
| ring分子, 长度11 |  | 97.50% |
| ring分子, 长度12 |  | 90.00% |
| ring分子, 长度13 |  | 95.00% |
| ring分子, 长度14 |  | 77.50% |
| ring分子, 长度15 |  | 67.50% |
| 直链, 长度7 |  | 100.00% |
| 直链, 长度8 |  | 100.00% |
| 直链, 长度9 |  | 100.00% |
| 直链, 长度10 |  | 95.00% |
| 直链, 长度11 |  | 90.00% |
| 直链, 长度12 |  | 95.00% |
| 直链, 长度13 |  | 80.00% |
| 直链, 长度14 |  | 70.00% |
| 直链, 长度15 |  | 50.00% |
| branch分子, 长度5 |  | 100.00% |
| branch分子, 长度6 |  | 93.33% |
| branch分子, 长度7 |  | 70.00% |
| branch分子, 长度8 |  | 66.67% |
| branch分子, 长度9 |  | 50.00% |
  
  

checkpoint: /opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v19-20250302-131508/checkpoint-1350

需要处理: 

  1. ring, 长度14/15的准确率
  2. 直链, 长度13/14/15的准确率
  3. 然后增强branch分子的准确率

(ring分子, 长度14) 和 (ring分子, 长度15) 的错误分析:

```
(ring分子, 长度14)
 
[
    {
        "Predicted": "[O][N][N][N][O][C][O][C][O][N][Ring1][=Branch1][O]",
        "GT": "[O][N][N][O][C][O][C][Ring1][=Branch1][O][N][N][N][O]",
        "Annotation": "长度错误识别(13), 在ring op前少原子, 其他原子正确. 镜像([O][N][N][N][O][C][O][C][O][N][N][Ring1][=Branch1][O]\n),"
    },
    {
        "Predicted": "[C][C][O][N][C][C][C][O][O][N][N][C]",
        "GT": "[C][N][C][N][O][O][C][Ring1][Branch1][C][N][O][C][C]",
        "Annotation": "长度错误识别(12), 缺少ring结构."
    },
    {
        "Predicted": "[C][O][N][O][N][C][N][C][N][Ring1][=Branch1][C][O][N][O]",
        "GT": "[C][O][N][O][N][C][N][C][Ring1][=Branch1][C][N][O][O]",
        "Annotation": "长度错误识别(15), ring op前多一个原子, 支链原子交换"
    },
    {
        "Predicted": "[N][C][N][N][N][N][C][O][O][N][Ring1][Branch1][C][O]",
        "GT": "[N][C][N][N][N][C][O][O][N][Ring1][Branch1][C][N][O]",
        "Annotation": "两个支链, 原子从一个支链调整到另一个"
    },
    {
        "Predicted": "[N][O][O][N][N][C][C][O][O][N][Ring1][=Branch1][O]",
        "GT": "[O][N][N][O][O][C][C][C][Ring1][=Branch1][N][O][O][N]",
        "Annotation": "长度错误识别(13), 接入点原子错误, 支链少一个原子"
    },
    {
        "Predicted": "[C][C][C][N][O][N][Ring1][Ring2][O][C][C][C][O][N]",
        "GT": "[C][C][C][N][O][N][Ring1][Branch1][O][C][C][C][O][N]",
        "Annotation": "ring长度错误, 其他院子正确"
    },
    {
        "Predicted": "[N][O][O][N][N][C][O][N][O][N][Ring1][=Branch1][O]",
        "GT": "[O][N][N][O][N][O][C][C][Ring1][=Branch1][N][O][O][N]",
        "Annotation": "长度错误识别(13), 接入点原子错误, 支链少一个原子"
    },
    {
        "Predicted": "[C][O][O][N][N][O][N][C][N][C][C][N][Ring1][=Branch1]",
        "GT": "[N][C][N][C][C][N][Ring1][=Branch1][O][N][N][O][O][C]",
        "Annotation": "环内原子交换"
    },
    {
        "Predicted": "[C][C][O][N][N][C][Ring1][Branch1][O][C][C][C][O][N]",
        "GT": "[C][C][O][N][C][N][Ring1][Branch1][O][C][C][C][O][N]",
        "Annotation": "接入点错误(环内原子交换)"
    }
]
```

```
(ring分子, 长度15)
 
[
    {
        "Predicted": "[N][N][N][O][C][O][C][C][O][O][C][N][O][Ring1][Branch1]",
        "GT": "[O][O][C][N][O][C][Ring1][=Branch1][C][O][C][O][N][N][N]",
        "Annotation": "环内原子交换*2"
    },
    {
        "Predicted": "[O][C][N][C][C][N][N][N][O][N][C][C][Ring1][=Branch1][O]",
        "GT": "[N][O][N][C][C][N][Ring1][=Branch1][N][C][C][N][C][O][O]",
        "Annotation": "原子从一个支链末端调整到另一个支链末端"
    },
    {
        "Predicted": "[C][N][N][C][N][N][N][N][C][N][N][Ring1][Ring2][O][N][O]",
        "GT": "[O][N][O][N][C][N][N][Ring1][Ring2][N][N][C][N][N][C]",
        "Annotation": "长度识别错误(16), ring op前多一个原子"
    },
    {
        "Predicted": "[C][C][C][N][C][N][Ring1][Ring1][O][C][O][O][N][C]",
        "GT": "[C][N][O][O][C][O][N][N][C][N][Ring1][Ring1][C][C][C]",
        "Annotation": "长度识别错误(16), ring op后多一个原子"
    },
    {
        "Predicted": "[N][C][N][N][N][N][C][N][C][N][O][O]",
        "GT": "[N][C][N][N][N][Ring1][Ring2][N][N][N][C][N][C][O][O]",
        "Annotation": "缺少ring结构"
    },
    {
        "Predicted": "[C][O][C][N][C][O][N][N][N][O][O][O][Ring1][Branch1][C]",
        "GT": "[C][N][O][O][O][N][Ring1][Branch1][N][O][C][N][C][O][C]",
        "Annotation": "接入点错误(环内原子顺序相反, 导致接入点错误)"
    },
    {
        "Predicted": "[N][C][C][N][C][N][N][N][N][N][C][Ring1][Ring2][N]",
        "GT": "[N][N][C][N][N][N][Ring1][Ring2][N][N][C][N][C][C][N]",
        "Annotation": "长度识别错误(支链少一个原子)"
    },
    {
        "Predicted": "[C][O][C][O][N][O][N][N][N][O][O][O][Ring1][Branch1][C]",
        "GT": "[C][N][O][O][O][N][Ring1][Branch1][N][O][N][O][C][O][C]",
        "Annotation": "接入点错误(环内原子顺序相反, 导致接入点错误)"
    },
    {
        "Predicted": "[C][O][N][N][O][N][O][N][C][N][C][N][O][Ring1][=Branch1]",
        "GT": "[O][C][N][N][C][N][Ring1][=Branch1][O][N][O][N][N][O][C]",
        "Annotation": "环内原子交换, 其他原子正确"
    },
    {
        "Predicted": "[N][N][C][O][C][N][N][C][N][N][Ring1][Ring1][C][N]",
        "GT": "[N][C][N][N][C][Ring1][Ring1][N][N][C][C][O][C][N][N]",
        "Annotation": "支链少一个原子"
    },
    {
        "Predicted": "[C][C][N][O][C][C][N][C][N][C][Ring1][Ring2][O][N][O]",
        "GT": "[O][N][O][C][C][N][N][Ring1][Ring2][C][C][O][N][C][C]",
        "Annotation": "环内原子交换"
    },
    {
        "Predicted": "[C][N][N][O][N][O][O][C][N][Ring1][Branch1][N][O][N]",
        "GT": "[N][O][N][N][C][O][O][O][N][Ring1][=Branch1][O][N][N][C]",
        "Annotation": "ring长度错误, 少一个原子"
    },
    {
        "Predicted": "[C][N][C][C][N][Ring1][Ring1][C][O][O][C][N][O][C][N]",
        "GT": "[N][C][O][N][C][O][O][C][N][C][C][N][Ring1][Ring1][C]",
        "Annotation": "原子从一个支链末端调整到另一个支链末端"
    }
] 
``` 

提高v57-v80训练数据量到300: 

|| v1-v32, 追加v33-v36, 追加v53-v56 (300数据)  
追加v45-v48和v53-v56和v41-v44  
| epoch=1 | 追加v57-v80, epoch=1 (基线) | 追加v57-v80, 300数据, epoch=1 |  |
| --- | --- | --- | --- |
| ring分子, 长度7 |  | 97.50% | 97.50% |
| ring分子, 长度8 |  | 87.50% | 95.00% |
| ring分子, 长度9 |  | 95.00% | 100.00% |
| ring分子, 长度10 |  | 95.00% | 97.50% |
| ring分子, 长度11 |  | 97.50% | 100.00% |
| ring分子, 长度12 |  | 90.00% | 90.00% |
| ring分子, 长度13 |  | 95.00% | 100.00% |
| ring分子, 长度14 |  | 77.50% | 92.50% |
| ring分子, 长度15 |  | 67.50% | 75.00% |
| 直链, 长度7 |  | 100.00% | 100.00% |
| 直链, 长度8 |  | 100.00% | 100.00% |
| 直链, 长度9 |  | 100.00% | 100.00% |
| 直链, 长度10 |  | 95.00% | 100.00% |
| 直链, 长度11 |  | 90.00% | 95.00% |
| 直链, 长度12 |  | 95.00% | 90.00% |
| 直链, 长度13 |  | 80.00% | 95.00% |
| 直链, 长度14 |  | 70.00% | 70.00% |
| 直链, 长度15 |  | 50.00% | 65.00% |
| branch分子, 长度5 |  | 100.00% | 100.00% |
| branch分子, 长度6 |  | 93.33% | 93.33% |
| branch分子, 长度7 |  | 70.00% | 70.00% |
| branch分子, 长度8 |  | 66.67% | 66.67% |
| branch分子, 长度9 |  | 50.00% | 50.00% |
  
checkpoint: /opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v20-20250303-003349/checkpoint-2700

训练步骤: 

```
# last_model_checkpoint=/opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v0-20250301-112246/checkpoint-1776 #v1-v32
# last_model_checkpoint=/opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v10-20250301-215111/checkpoint-315 #v33-v36, v53-v56(300 data)
# last_model_checkpoint=/opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v16-20250302-012117/checkpoint-558 #45,46,47,48,53,54,55,56,41,42,43,44
last_model_checkpoint=/opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v20-20250303-003349/checkpoint-2700 #v57-v80 (300 data)

``` 

(ring分子, 长度14) 达标, (ring分子, 长度15) 正确率略微提升. 目前认为提高数据量, 可以继续提高(ring分子, 长度15) 的正确率. 目前先不处理.

分析(直链, 长度14)的错误分布: 

```
[
    {
        "Predicted": "[O][N][N][N][O][C][N][O][O][C][O][O][O][N][O]",
        "GT": "[O][O][N][N][O][C][N][O][O][C][O][O][N][O]",
        "Annotation": "位置2多一个原子, 2/11对调"
    },
    {
        "Predicted": "[N][N][O][O][O][O][N][C][O][N][C][N][O]",
        "GT": "[O][N][C][N][O][C][N][O][O][O][O][O][N][N]",
        "Annotation": "位置3少一个原子"
    },
    {
        "Predicted": "[C][N][O][O][O][C][O][C][C][O][O][O][C][O][O]",
        "GT": "[C][N][O][O][C][O][C][C][O][O][O][C][O][O]",
        "Annotation": "位置3多一个原子"
    },
    {
        "Predicted": "[N][O][O][C][N][C][O][O][N][O][C][O][O][O]",
        "GT": "[O][O][C][O][N][O][O][C][N][C][O][O][O][N]",
        "Annotation": "位置4插入最后"
    },
    {
        "Predicted": "[N][C][O][C][C][C][O][O][C][O][O][O][N]",
        "GT": "[N][C][C][O][C][C][C][O][O][C][O][O][O][N]",
        "Annotation": "位置2少一个原子"
    },
    {
        "Predicted": "[C][N][N][N][O][O][N][C][C][N][N][O][C]",
        "GT": "[C][O][N][N][N][C][C][N][O][O][N][N][N][C]",
        "Annotation": "位置10少一个原子"
    }
]
``` 

大部分是长度问题. 猜测继续增加数据量可以修复. 

对于branch的分析: 由于没有branch的训练数据, 导致branch的识别, 都是选择一个主链识别成直链, 或者直链附加支链的原子

先模仿add_ring_op, 构建add_branch_op

# 第31次训练 - 增加branch分子的训练数据

v81-v90: 为4-13的直链分子, add_branch_op, 形成长度6-15的branch:

|  | 追加v81-v90, epoch=1 |
| --- | --- |
| branch分子, 长度5 | 100.00% |
| branch分子, 长度6 | 100.00% |
| branch分子, 长度7 | 100.00% |
| branch分子, 长度8 | 100.00% |
| branch分子, 长度9 | 93.33% |
| branch分子, 长度10 | 93.33% |
| branch分子, 长度11 | 76.67% |
| branch分子, 长度12 | 76.67% |
| branch分子, 长度13 | 76.67% |
| branch分子, 长度14 | 63.33% |
| branch分子, 长度15 | 30.00% |
| 直链, 长度7 | 100.00% |
| 直链, 长度8 | 100.00% |
| 直链, 长度9 | 100.00% |
| 直链, 长度10 | 100.00% |
| 直链, 长度11 | 90.00% |
| 直链, 长度12 | 90.00% |
| 直链, 长度13 | 80.00% |
| 直链, 长度14 | 75.00% |
| 直链, 长度15 | 45.00% |
| ring分子, 长度7 | 65.00% |
| ring分子, 长度8 | 55.00% |
| ring分子, 长度9 | 47.50% |
| ring分子, 长度10 | 47.50% |
| ring分子, 长度11 | 30.00% |
| ring分子, 长度12 | 20.00% |
| ring分子, 长度13 | 15.00% |
| ring分子, 长度14 | 22.50% |
| ring分子, 长度15 | 7.50% |
  
出现的问题: 

  - branch分子, 长度11及以后就不达标, 需要分析错误
  - ring分子的准确率全线大幅下降

(branch分子, 长度11)的错误分析: (所有都包括branch长度识别错误)

```
[
    {
        "Predicted": "[C][C][Branch1][Ring2][N][N][N][O][O][O][C]",
        "GT": "[O][N][N][N][C][Branch1][C][C][O][O][C]",
        "Annotation": "起点5([C][C][Branch1][Branch1][N][N][N][O][O][O][C]), branch长度错误"
    },
    {
        "Predicted": "[C][N][Branch1][Ring1][C][N][O][O][N][C][O]",
        "GT": "[O][C][N][O][N][Branch1][Ring2][C][N][O][C]",
        "Annotation": "镜像([C][N][Branch1][Ring2][C][N][O][O][N][C][O]), branch长度错误"
    },
    {
        "Predicted": "[N][C][N][Branch1][Branch1][N][N][N][O][O][N]",
        "GT": "[N][O][O][N][Branch1][Ring1][C][N][N][N][N]",
        "Annotation": "起点6([N][C][N][Branch1][Ring2][N][N][N][O][O][N]), branch长度错误"
    },
    {
        "Predicted": "[C][O][C][C][Branch1][Ring1][C][C][O][O][O]",
        "GT": "[C][C][Branch1][Ring2][C][O][C][C][O][O][O]",
        "Annotation": "起点5([C][O][C][C][Branch1][C][C][C][O][O][O]\n), branch长度错误"
    },
    {
        "Predicted": "[C][C][Branch1][Ring2][C][O][C][O][N][O][O]",
        "GT": "[C][C][Branch1][=Branch1][C][O][C][O][N][O][O]",
        "Annotation": "branch长度错误"
    },
    {
        "Predicted": "[C][N][Branch1][Ring2][C][N][C][N][N][O][C][O]",
        "GT": "[C][N][Branch1][Branch1][C][N][N][N][O][C][O]",
        "Annotation": "多一个原子, branch长度错误"
    },
    {
        "Predicted": "[C][C][Branch1][Ring2][C][C][N][N][N][N][N]",
        "GT": "[C][C][Branch1][Ring1][N][N][C][C][N][N][N]",
        "Annotation": "branch长度错误, 分支内外原子交换"
    }
]
``` 

  

分析(ring分子, 长度7)的错误分布: 将ring结构识别成分支结构 (不闭环), 举例: 

![image2025-3-3 23:14:34.png](/assets/01KJBZRQQVCZ5ZSZZ4W1F9974D/image2025-3-3%2023%3A14%3A34.png)

  

目前猜测: 因为ring长度和branch长度, 在相同的操作符下, 长度差1. 模型在识别branch时, 用了ring的长度体系, 在识别ring时, 更倾向于认成branch.

将ring和branch的数据混合训练.

  - 基模型checkpoint: /opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v20-20250303-003349/checkpoint-2700
  - 数据集: "33,34,35,36,57,61,65,69,73,77,81,82,83,84,85,86,87,88,89,90"

|  | 追加v81-v90, 33,34,35,36,57,61,65,69,73,77, epoch=1 |
| --- | --- |
| branch分子, 长度5 | 100.00% |
| branch分子, 长度6 | 100.00% |
| branch分子, 长度7 | 96.67% |
| branch分子, 长度8 | 100.00% |
| branch分子, 长度9 | 90.00% |
| branch分子, 长度10 | 96.67% |
| branch分子, 长度11 | 76.67% |
| branch分子, 长度12 | 60.00% |
| branch分子, 长度13 | 66.67% |
| branch分子, 长度14 | 46.67% |
| branch分子, 长度15 | 26.67% |
| 直链, 长度7 | 100.00% |
| 直链, 长度8 | 100.00% |
| 直链, 长度9 | 100.00% |
| 直链, 长度10 | 95.00% |
| 直链, 长度11 | 95.00% |
| 直链, 长度12 | 90.00% |
| 直链, 长度13 | 80.00% |
| 直链, 长度14 | 75.00% |
| 直链, 长度15 | 60.00% |
| ring分子, 长度7 | 100.00% |
| ring分子, 长度8 | 95.00% |
| ring分子, 长度9 | 97.50% |
| ring分子, 长度10 | 97.50% |
| ring分子, 长度11 | 95.00% |
| ring分子, 长度12 | 82.50% |
| ring分子, 长度13 | 92.50% |
| ring分子, 长度14 | 77.50% |
| ring分子, 长度15 | 52.50% |
  
观察: 

  - (branch分子, 长度11) 及以后, 正确率仍然不达标, 需要分析错误
  - 直链分子和ring分子在长度13以后的衰减, 跟之前单独训练时类似(且正确率更低一点), 在下面的全数据训练中, 猜测能得到修复

错误分析(branch分子, 长度11):

```
 [
    {
        "Predicted": "[N][N][N][Branch1][Ring2][O][C][O][N][N][N]",
        "GT": "[N][N][Branch1][Ring1][N][N][O][C][O][N][N]",
        "Annotation": "从两个分支中, 都拿出最后一个原子, 补充到主链最后"
    },
    {
        "Predicted": "[N][C][N][Branch1][Branch1][N][N][N][O][O][N]",
        "GT": "[N][O][O][N][Branch1][Ring1][C][N][N][N][N]",
        "Annotation": "起点6([N][C][N][Branch1][Ring2][N][N][N][O][O][N]), branch长度错误. 或者将支链首原子(O)移到主链末端"
    },
    {
        "Predicted": "[C][O][C][C][Branch1][Ring1][C][O][C][O][O]",
        "GT": "[C][C][Branch1][Ring2][C][O][C][C][O][O][O]",
        "Annotation": "将支链的末端原子(OH)挂到主链最后"
    },
    {
        "Predicted": "[C][C][Branch1][Ring2][C][O][O][O][C][O][N]",
        "GT": "[C][C][Branch1][=Branch1][C][O][C][O][N][O][O]",
        "Annotation": "支链(C)挂错了位置 (平移branch op)"
    },
    {
        "Predicted": "[C][N][Branch1][Ring2][C][O][C][O][N][N][N]",
        "GT": "[C][N][Branch1][Branch1][C][N][N][N][O][C][O]",
        "Annotation": "复杂变化"
    },
    {
        "Predicted": "[C][C][O][N][Branch1][C][N][N][O][C][N]",
        "GT": "[C][C][O][N][N][Branch1][Ring2][O][C][N][N]",
        "Annotation": "branch op右移一位"
    },
    {
        "Predicted": "[C][C][Branch1][Ring2][C][C][N][N][N][N][N]",
        "GT": "[C][C][Branch1][Ring1][N][N][C][C][N][N][N]",
        "Annotation": "规范式([C][C][Branch1][=Branch1][C][C][N][N][N][N][N]), branch长度错误"
    }
]
``` 

额外进行一次全训练: 

  - 基模型checkpoint: /opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v0-20250301-112246/checkpoint-1776
  - 数据集: "33,34,35,36,41,42,43,44,45,46,47,48,53,54,55,56,57,58,59,60,61,62,63,64,65,66,67,68,69,70,71,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86,87,88,89,90,91,92,93,94,95,96"
  - 数据量: 300

|  | 追加v81-v90, 33,34,35,36,57,61,65,69,73,77, epoch=1 |
| --- | --- |
| branch分子, 长度5 | 100.00% |
| branch分子, 长度6 | 100.00% |
| branch分子, 长度7 | 100.00% |
| branch分子, 长度8 | 100.00% |
| branch分子, 长度9 | 96.67% |
| branch分子, 长度10 | 96.67% |
| branch分子, 长度11 | 83.33% |
| branch分子, 长度12 | 80.00% |
| branch分子, 长度13 | 80.00% |
| branch分子, 长度14 | 73.33% |
| branch分子, 长度15 | 40.00% |
| 直链, 长度7 | 100.00% |
| 直链, 长度8 | 100.00% |
| 直链, 长度9 | 100.00% |
| 直链, 长度10 | 100.00% |
| 直链, 长度11 | 100.00% |
| 直链, 长度12 | 95.00% |
| 直链, 长度13 | 100.00% |
| 直链, 长度14 | 100.00% |
| 直链, 长度15 | 100.00% |
| ring分子, 长度7 | 100.00% |
| ring分子, 长度8 | 100.00% |
| ring分子, 长度9 | 100.00% |
| ring分子, 长度10 | 97.50% |
| ring分子, 长度11 | 95.00% |
| ring分子, 长度12 | 97.50% |
| ring分子, 长度13 | 100.00% |
| ring分子, 长度14 | 95.00% |
| ring分子, 长度15 | 77.50% |
  
  - (branch分子, 长度11) 及以后, 正确率仍然不达标, 需要分析错误
  - 直链分子和ring分子在长度13以后的衰减, 已经得到修复

先尝试增加branch长度变化: 

  - 基模型: /opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v25-20250304-004423/checkpoint-1074
  - 数据: "99,100,101,102,103,104,105,106,107,108,109"
| - |  | 追加"99,100,101,102,103,104,105,106,107,108,109", epoch=1 |
| --- | --- | --- |
| branch分子, 长度5 | 100.00% |  |
| branch分子, 长度6 | 100.00% |  |
| branch分子, 长度7 | 100.00% |  |
| branch分子, 长度8 | 100.00% |  |
| branch分子, 长度9 | 100.00% |  |
| branch分子, 长度10 | 100.00% |  |
| branch分子, 长度11 | 80.00% |  |
| branch分子, 长度12 | 90.00% |  |
| branch分子, 长度13 | 76.67% |  |
| branch分子, 长度14 | 76.67% |  |
| branch分子, 长度15 | 43.33% |  | - (branch分子, 长度12)达标, 其他未见明显提升

使用branch op平移的数据:

  - 基模型: /opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v25-20250304-004423/checkpoint-1074
  - 数据: "110,111,112,113,114,115,116,117,118,119,120"
| - |  | 追加"110,111,112,113,114,115,116,117,118,119,120", epoch=1 |
| --- | --- | --- |
| branch分子, 长度5 | 100.00% |  |
| branch分子, 长度6 | 100.00% |  |
| branch分子, 长度7 | 100.00% |  |
| branch分子, 长度8 | 100.00% |  |
| branch分子, 长度9 | 100.00% |  |
| branch分子, 长度10 | 93.33% |  |
| branch分子, 长度11 | 83.33% |  |
| branch分子, 长度12 | 93.33% |  |
| branch分子, 长度13 | 96.67% |  |
| branch分子, 长度14 | 80.00% |  |
| branch分子, 长度15 | 63.33% |  |

观察:

  - (branch分子, 长度12/13)达标, 需要分析长度11/14的错误分布

(branch分子, 长度11)错误分布: 

```
[
    {
        "Predicted": "[N][N][O][C][O][C][Branch1][C][C][O][O]",
        "GT": "[O][C][C][Branch1][C][O][O][C][O][N][N]",
        "Annotation": "镜像([N][N][O][C][O][C][Branch1][C][O][C][O]), 分支内外原子对调"
    },
    {
        "Predicted": "[N][C][N][Branch1][Branch1][N][N][N][O][O][N]",
        "GT": "[N][O][O][N][Branch1][Ring1][C][N][N][N][N]",
        "Annotation": "起点5([N][C][N][Branch1][Ring2][N][N][N][O][O][N]), 分支长度错误(应是4, 识别成2)"
    },
    {
        "Predicted": "[C][O][C][Branch1][Ring1][C][C][C][O][O][O]",
        "GT": "[C][C][Branch1][Ring2][C][O][C][C][O][O][O]",
        "Annotation": "起点5([C][O][C][C][Branch1][C][C][C][O][O][O]), branch长度错误(应是2, 识别成1), branch op右移"
    },
    {
        "Predicted": "[C][N][Branch1][=Branch1][C][N][N][N][O][C][O]",
        "GT": "[C][N][Branch1][Branch1][C][N][N][N][O][C][O]",
        "Annotation": "branch长度错误(应是5, 识别成4)"
    },
    {
        "Predicted": "[C][C][Branch1][Ring2][C][C][N][N][N][N][N]",
        "GT": "[C][C][Branch1][Ring1][N][N][C][C][N][N][N]",
        "Annotation": "规范式([C][C][Branch1][=Branch1][C][C][N][N][N][N][N]), branch长度错误(应是5, 识别成3)"
    }
]
``` 

(branch分子, 长度14) 错误分布: 

```
[
    {
        "Predicted": "[C][N][O][N][C][Branch1][Ring2][C][C][N][N][N][N][O]",
        "GT": "[O][N][N][N][N][C][Branch1][Branch1][N][O][N][C][C][C]",
        "Annotation": "起点10([C][N][O][N][C][Branch1][Ring1][C][C][N][N][N][N][O]), branch长度错误(应是2, 识别成3)"
    },
    {
        "Predicted": "[C][O][C][Branch1][C][O][O][C][O][C][C][C][N][N]",
        "GT": "[N][N][C][C][O][C][O][C][Branch1][Ring1][O][C][O][C]",
        "Annotation": "起点10([C][O][C][Branch1][Ring1][O][C][O][C][O][C][C][N][N]), branch长度识别错误(应是2, 识别成1), branch内外原子交换"
    },
    {
        "Predicted": "[C][N][O][O][N][N][O][O][C][C][Branch1][C][O][O]",
        "GT": "[C][N][O][O][N][N][O][O][C][Branch1][C][C][O][O]",
        "Annotation": "branch op左移一位"
    },
    {
        "Predicted": "[O][N][O][N][Branch1][Ring1][C][C][O][O][O][O][N][O]",
        "GT": "[O][N][N][Branch1][Ring2][O][N][O][C][C][O][O][O][O]",
        "Annotation": "起点6([O][N][O][N][Branch1][#Branch1][C][C][O][O][O][O][N][O]), branch长度错误(应是6, 识别成2)"
    },
    {
        "Predicted": "[C][C][Branch1][Ring1][C][O][N][C][C][O][N][O][C][C][O]",
        "GT": "[N][O][C][Branch1][C][C][C][C][O][N][O][C][C][O]",
        "Annotation": "复杂变化"
    },
    {
        "Predicted": "[C][N][C][N][Branch1][Ring1][O][N][N][O][O][C][O][C]",
        "GT": "[C][N][C][N][Branch1][=Branch1][O][O][C][O][C][O][N][N]",
        "Annotation": "规范式([C][N][C][N][Branch1][Ring2][O][N][N][O][O][C][O][C]), branch长度错误(应是3, 识别成2)"
    }
] 
``` 

以上错误中, 大部分是branch长度错误, 除一例, 其他都是将branch长度识别低了.

使用branch op平移的数据 + (增加branch的变化lower_branch_length, 将已知的branch分子的branch长度降低1位-2位, 形成变化数据)

  - 基模型: /opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v25-20250304-004423/checkpoint-1074
  - 数据: "99,100,101,102,103,104,105,106,107,108,109,110,111,112,113,114,115,116,117,118,119,120"

|  | 追加"99,100,101,102,103,104,105,106,107,108,109,110,111,112,113,114,115,116,117,118,119,120", epoch=1 |
| --- | --- |
| branch分子, 长度5 | 100.00% |
| branch分子, 长度6 | 100.00% |
| branch分子, 长度7 | 100.00% |
| branch分子, 长度8 | 100.00% |
| branch分子, 长度9 | 100.00% |
| branch分子, 长度10 | 96.67% |
| branch分子, 长度11 | 96.67% |
| branch分子, 长度12 | 93.33% |
| branch分子, 长度13 | 93.33% |
| branch分子, 长度14 | 96.67% |
| branch分子, 长度15 | 63.33% |
  
除了(branch分子, 长度15), 其他branch长度均达标.

# 第32次训练 - 整体训练

目前的训练步骤: 

```
# last_model_checkpoint=/opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v0-20250301-112246/checkpoint-1776 #v1-v32
# last_model_checkpoint=/opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v10-20250301-215111/checkpoint-315 #v33-v36, v53-v56(300 data)
# last_model_checkpoint=/opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v16-20250302-012117/checkpoint-558 #45,46,47,48,53,54,55,56,41,42,43,44
# last_model_checkpoint=/opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v20-20250303-003349/checkpoint-2700 #v57-v80 (300 data)
last_model_checkpoint=/opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v25-20250304-004423/checkpoint-1074 #33,34,35,36,57,61,65,69,73,77,81,82,83,84,85,86,87,88,89,90
# last_model_checkpoint=/opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v29-20250304-164352/checkpoint-1104 #99,100,101,102,103,104,105,106,107,108,109,110,111,112,113,114,115,116,117,118,119,120
``` 

合并数据集: 1-36,41-48,53-120 (150数据)

commit: b289d1f6debe9c14307d90c81ff08c0097aa6e7b

checkpoint: /opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v30-20250305-001824/checkpoint-5976

训练时间: 27048 seconds = 7.5 hours

|  | 追加"1-36,41-48,53-120", epoch=1 |
| --- | --- |
| branch分子, 长度5 | 100.00% |
| branch分子, 长度6 | 100.00% |
| branch分子, 长度7 | 100.00% |
| branch分子, 长度8 | 100.00% |
| branch分子, 长度9 | 100.00% |
| branch分子, 长度10 | 100.00% |
| branch分子, 长度11 | 93.33% |
| branch分子, 长度12 | 93.33% |
| branch分子, 长度13 | 96.67% |
| branch分子, 长度14 | 96.67% |
| branch分子, 长度15 | 70.00% |
| 直链, 长度7 | 100.00% |
| 直链, 长度8 | 100.00% |
| 直链, 长度9 | 100.00% |
| 直链, 长度10 | 100.00% |
| 直链, 长度11 | 100.00% |
| 直链, 长度12 | 100.00% |
| 直链, 长度13 | 95.00% |
| 直链, 长度14 | 100.00% |
| 直链, 长度15 | 95.00% |
| ring分子, 长度7 | 97.50% |
| ring分子, 长度8 | 95.00% |
| ring分子, 长度9 | 100.00% |
| ring分子, 长度10 | 100.00% |
| ring分子, 长度11 | 100.00% |
| ring分子, 长度12 | 97.50% |
| ring分子, 长度13 | 100.00% |
| ring分子, 长度14 | 95.00% |
| ring分子, 长度15 | 90.00% |
  
除(branch分子, 长度15)外, 其他准确率都达标. 符合预期.

# TODO

  1. ~~branch支持到15原子~~
  2. 撤掉提示词中关于 原子种类-数量 的提示
  3. ~~增加长度16, 验证15原子的准确率是否上升~~
  4. 增加ring1/branch1 op的长度种类
  5. 多个环, 多个分支, 环和分支 混合
  6. 支持 键 的变化
  7. 注意: 有可能出现的问题: 训练数据和评估数据是两套生成方法, 但有可能过度相似
  8. 利用 RL 修正错误
  9. 建立好标准空间后, 往非标准空间扩展
  10. 支持ring2/branch2
