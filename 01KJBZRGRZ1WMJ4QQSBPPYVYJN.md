---
title: 20250212 - 使用ms-swift对Qwen2-VL进行微调 [24] - 研究训练数据的收敛
confluence_page_id: 3344332
created_at: 2025-02-13T02:17:19+00:00
updated_at: 2025-02-15T15:44:01+00:00
---

# 前置

在[20250207 - 使用ms-swift对Qwen2-VL进行微调 [23] - 在基模型上增加原子数] 中建立了一个基线:

  - 这个基线的问题是随着epoch增大, 评估效果会升高, 但需要的数据太多
  - 期望能在步骤1的基础上, 挑选一些数据, 进行快速的效果增强
  - 所以需要研究如何让训练数据收敛 (在已知epoch增大时, 可以达成更好的效果, 这个研究才成立)

基线:

commit: b499dc1cec38c06f88a80aff50b7078dadf3942b

|  | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=6) (LR=5e-05) | 步骤3 (epoch=9) (LR=3e-05) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 84.21% | 78.95% | 100.00% |
| 基础原子, 长度8 | 94.74% | 100.00% | 100.00% |
| ring分子, 长度7 | 97.50% | 100.00% | 97.50% |
| ring分子, 长度8 | 87.50% | 95.00% | 95.00% |
| branch分子, 长度7 | 83.33% | 90.00% | 93.33% |
| branch分子, 长度8 | 76.67% | 90.00% | 93.33% |
  
  

三个步骤的checkpoint:

  - /opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v817-20250212-002052/checkpoint-801
  - /opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v818-20250212-010602/checkpoint-801
  - /opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v819-20250212-015105/checkpoint-801

但发现问题是evaluate中的temperature会影响效果, 所以需要重新跑一次基线的评估:

|  | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=6) (LR=5e-05) | 步骤3 (epoch=9) (LR=3e-05) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 100.00% | 84.21% | 100.00% |
| 基础原子, 长度8 | 100.00% | 100.00% | 100.00% |
| ring分子, 长度7 | 97.50% | 100.00% | 97.50% |
| ring分子, 长度8 | 85.00% | 95.00% | 95.00% |
| branch分子, 长度7 | 86.67% | 86.67% | 90.00% |
| branch分子, 长度8 | 83.33% | 86.67% | 90.00% |
  
跟之前的基线差异不大, 只在步骤1的(基础院子, 长度7) 的记忆保持上有偏差

# 基线的步骤1的 (branch分子) 错误分析

(branch分子, 长度7)

```
19    /opt/huangyan/mol-ai/images/branch_selfies/length_7/1.png  {
"selfies": "[O][Branch1][Ring1][O][N][C][N]"}          O(ON)CN  {
"selfies": "[O][Branch1][Ring2][N][O][C][N]"}   O(NOC)N    0.150000   
mismatch
[N][O][O][C][N]和[N][O][N][O][C]的差别

11  /opt/huangyan/mol-ai/images/branch_selfies/length_7/137.png  {
"selfies": "[C][Branch1][Ring2][C][O][N][O]"}          C(CON)O  {
"selfies": "[C][Branch1][Ring2][C][N][O][O]"}   C(CNO)O    0.210526   
mismatch
[O][C][C][O][N]和[O][C][C][N][O]的差别

12  /opt/huangyan/mol-ai/images/branch_selfies/length_7/103.png  {
"selfies": "[N][Branch1][Ring1][N][C][O][C]"}          N(NC)OC  {
"selfies": "[N][Branch1][Ring1][N][C][C][O]"}   N(NC)CO    0.210526   
mismatch
[C][N][N][O][C]和[C][N][N][C][O]的差别

18  /opt/huangyan/mol-ai/images/branch_selfies/length_7/110.png  {
"selfies": "[O][Branch1][Ring1][C][C][C][C]"}          O(CC)CC  {
"selfies": "[C][Branch1][Ring1][O][C][C][C]"}   C(OC)CC    0.266667   
mismatch
[C][C][O][C][C]和[C][C][C][O][C]的差别
``` 

  

(branch分子, 长度8)

```
2   /opt/huangyan/mol-ai/images/branch_selfies/length_8/102.png      {
"selfies": "[N][C][N][Branch1][C][O][C][N]"}         NCN(O)CN    {
"selfies": "[O][C][Branch1][Ring2][N][C][N][N]"}  OC(NCN)N    0.200000   
mismatch
branch位置错误, 分支内外的原子调序
[N][C][N][C][Branch1][C][N][O]

0   /opt/huangyan/mol-ai/images/branch_selfies/length_8/109.png  {
"selfies": "[N][Branch1][Ring2][C][C][C][N][O]"}         N(CCC)NO    {
"selfies": "[N][Branch1][Ring1][C][C][C][N][O]"}  N(CC)CNO    0.272727   
mismatch
branch长度错误

18  /opt/huangyan/mol-ai/images/branch_selfies/length_8/137.png  {
"selfies": "[N][Branch1][Ring2][C][N][C][O][C]"}         N(CNC)OC    {
"selfies": "[N][Branch1][Ring2][C][N][C][C][O]"}  N(CNC)CO    0.300000   
mismatch
branch后的原子调序

23  /opt/huangyan/mol-ai/images/branch_selfies/length_8/126.png  {
"selfies": "[C][O][N][Branch1][Ring1][N][C][O]"}         CON(NC)O    {
"selfies": "[O][N][N][Branch1][Ring1][O][C][C]"}  ONN(OC)C    0.333333   
mismatch
分支内外的原子调序
[C][O][N][Branch1][C][C][N][O]

14  /opt/huangyan/mol-ai/images/branch_selfies/length_8/116.png      {
"selfies": "[C][N][N][Branch1][C][O][N][C]"}         CNN(O)NC        {
"selfies": "[O][N][N][Branch1][C][C][N][C]"}  ONN(C)NC    0.352941   
mismatch
分支内外的原子调序
[C][N][N][Branch1][C][C][N][O]

``` 

  

# 处理(branch分子, 长度7)的错误-1

其大量错误都是直链分子产生变化后, 其branch形式也发生了变化  

在基线上, 使用以下数据集训练: 

  - branch_selfies*1 \+ branch_selfies_by_diff_start_point_from_rotate_selfies_substring_from_random_selfies_only_basic_atom*2  

注意: 每个步骤只进行一个epoch

  

|  | 步骤1 (epoch=1) (LR=1e-4) | 步骤2 (epoch=2) (LR=5e-05) | 步骤3 (epoch=3) (LR=3e-05) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 26.32% | 31.58% | 36.84% |
| 基础原子, 长度8 | 21.05% | 31.58% | 42.11% |
| ring分子, 长度7 | 100.00% | 97.50% | 97.50% |
| ring分子, 长度8 | 82.50% | 82.50% | 90.00% |
| branch分子, 长度7 | 73.33% | 90.00% | 90.00% / 96.67%(偶尔) |
| branch分子, 长度8 | 70.00% | 63.33% | 70.00% |
  
  

可以看到, 这种数据让基础原子的输出, 从直链形式变成了分支形式, 挑战过大以至于准确率大幅下降.

(branch分子, 长度7) 的准确率略有上升, 重复评估, 准确率偶尔到96%

# 处理(branch分子, 长度7)的错误-2

尝试用更简单的数据增强, 对branch, 随机找两个不同的原子进行对调

commit: ade934782d7ff0377b20843d9d71395c5e55913a

在基线上, 使用以下数据集训练: 

  - branch_selfies*1 + exchange_two_atoms_from_branch_selfies*2

|  | 步骤1 (epoch=1) (LR=1e-4) | 步骤2 (epoch=2) (LR=5e-05) | 步骤3 (epoch=3) (LR=3e-05) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 52.63% | 47.37% | 42.11% |
| 基础原子, 长度8 | 42.11% | 52.63% | 31.58% |
| ring分子, 长度7 | 97.50% | 97.50% | 97.50% |
| ring分子, 长度8 | 87.50% | 90.00% | 87.50% |
| branch分子, 长度7 | 80.00% | 86.67% | 86.67% |
| branch分子, 长度8 | 80.00% | 80.00% | 83.33% |
|  | 步骤1 (epoch=1) (LR=1e-5) | 步骤2 (epoch=2) (LR=5e-06) | 步骤3 (epoch=3) (LR=3e-06) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 84.21% | 68.42% | 68.42% |
| 基础原子, 长度8 | 94.74% | 89.47% | 94.74% |
| ring分子, 长度7 | 97.50% | 97.50% | 97.50% |
| ring分子, 长度8 | 90.00% | 90.00% | 90.00% |
| branch分子, 长度7 | 66.67% | 66.67% | 66.67% |
| branch分子, 长度8 | 80.00% | 66.67% | 80.00% |
  
  

exchange_two_atoms_from_branch_selfies 会让(基础原子, 长度7) 变成 [branch]形式, 导致(基础原子, 长度7) 准确度下降.

每步骤仅1个epoch效果不好. 

  

  

|  | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=6) (LR=5e-05) | 步骤3 (epoch=9) (LR=3e-05) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 36.84% | 26.32% | 31.58% |
| 基础原子, 长度8 | 31.58% | 10.53% | 15.79% |
| ring分子, 长度7 | 97.50% | 92.50% | 92.50% |
| ring分子, 长度8 | 87.50% | 85.00% | 87.50% |
| branch分子, 长度7 | 100.00% | 96.67% | 93.33% |
| branch分子, 长度8 | 80.00% | 96.67% | 86.67% |
  
  

  - (基础原子, 长度7) 变成了branch形式, 导致准确率下降
  - (branch分子, 长度7) 准确率大幅提升

  

# 处理(branch分子, 长度7)的错误-3

  

**只"尝试用更少的数据", 不增加基础原子的数据:**

(branch_selfies*1 + exchange_two_atoms_from_branch_selfies*1)

|  | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=6) (LR=5e-05) | 步骤3 (epoch=9) (LR=3e-05) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 47.37% | 21.05% | 36.84% |
| 基础原子, 长度8 | 47.37% | 26.32% | 15.79% |
| ring分子, 长度7 | 97.50% | 95.00% | 95.00% |
| ring分子, 长度8 | 85.00% | 85.00% | 77.50% |
| branch分子, 长度7 | 83.33% | 90.00% | 93.33% |
| branch分子, 长度8 | 76.67% | 70.00% | 83.33% |
  
(branch分子, 长度7) 的准确率提升没有之前快

  

**尝试用更少的数据, 增加基础原子的数据, 尝试保持基础原子的效果:**

(branch_selfies*1 + exchange_two_atoms_from_branch_selfies*1 + random_selfies_only_basic_atom * 0.5)

|  | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=6) (LR=5e-05) | 步骤3 (epoch=9) (LR=3e-05) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 78.95% | 84.21% | 84.21% |
| 基础原子, 长度8 | 94.74% | 84.21% | 84.21% |
| ring分子, 长度7 | 97.50% | 97.50% | 95.00% |
| ring分子, 长度8 | 82.50% | 82.50% | 82.50% |
| branch分子, 长度7 | 86.67% | 93.33% | 90.00% |
| branch分子, 长度8 | 83.33% | 76.67% | 73.33% |
  
效果:

  - 基础原子的正确率上升, 但没有达标. 其上升的原因是: 更多的原子没有被识别成[branch]形式, 而不是 [branch]形式的正确率在提升
  - (branch分子, 长度7)的正确率略下降

  

**增加branch的增强, 尝试保持基础原子的效果:**

(branch_selfies*1 + exchange_two_atoms_from_branch_selfies*2 + random_selfies_only_basic_atom * 0.5):

  

|  | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=6) (LR=5e-05) | 步骤3 (epoch=9) (LR=3e-05) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 94.74% | 68.42% | 68.42% |
| 基础原子, 长度8 | 94.74% | 84.21% | 94.74% |
| ring分子, 长度7 | 97.50% | 95.00% | 90.00% |
| ring分子, 长度8 | 87.50% | 87.50% | 87.50% |
| branch分子, 长度7 | 90.00% | 93.33% | 93.33% |
| branch分子, 长度8 | 83.33% | 86.67% | 83.33% |
  
  

基础原子的正确率在步骤2/3下降, 还是因为将基础原子链识别为[branch], 但分支正确率又不高

  

**继续增加branch的增强, 尝试保持基础原子的效果:**

(branch_selfies*1 + exchange_two_atoms_from_branch_selfies*3 + random_selfies_only_basic_atom * 0.5):

  

|  | 步骤1 (epoch=3) (LR=1e-4) |
| --- | --- |
| 基础原子, 长度7 | 57.89% |
| 基础原子, 长度8 | 89.47% |
| ring分子, 长度7 | 95.00% |
| ring分子, 长度8 | 82.50% |
| branch分子, 长度7 | 96.67% |
| branch分子, 长度8 | 83.33% |
  
  

在步骤1, 就能让更多的基础原子使用[branch]形式, 错误率上升

# 额外的发现

发现 基础直链分子 和 直链分子的branch形式 是冲突的, 模型会在两者间反复横跳. 

需要对训练数据进行规范:

  - 验证随机selfies的合理性, 还需要验证其是smiles的规范式:

```
mol = Chem.MolFromSmiles(smiles)
canon_smi = Chem.MolToSmiles(mol)
```

  - 生成图片时, 需要符合smiles规范式的左右顺序

```
from rdkit import Chem
from rdkit.Chem import AllChem
from rdkit.Chem import Draw
from rdkit.Chem import rdGeometry

def mirror_coords(mol):
    """镜像反转所有原子的x坐标"""
    conf = mol.GetConformer()
    for i in range(mol.GetNumAtoms()):
        pos = conf.GetAtomPosition(i)
        new_pos = rdGeometry.Point3D(-pos.x, pos.y, pos.z)
        conf.SetAtomPosition(i, new_pos)

def draw_mol_with_coord_flip(smiles, img_size=(400, 300)):
    # 生成分子并计算初始坐标
    mol = Chem.MolFromSmiles(smiles)
    AllChem.Compute2DCoords(mol)
    
    # 检查是否需要翻转
    conf = mol.GetConformer()
    atom0_x = conf.GetAtomPosition(0).x
    min_x = min(conf.GetAtomPosition(i).x for i in range(mol.GetNumAtoms()))
    
    if atom0_x != min_x:
        # 执行原子级坐标翻转
        mirror_coords(mol)
    
    # 绘制分子
    return Draw.MolToImage(mol, size=img_size)
```

    - 但看起来符合smiles的左右顺序, 图片会看起来有点怪怪的, 但先这样继续进行.

其他想法: 

  - 规范smiles表达式和图片左右顺序, 让训练时对某一张图片, 不会产生歧义. 这样降低对训练的影响. (这也是模拟人的一种思路)
    - 构建出标准空间后, 再扩展图片的类型.
  - 另一种想法是 对于 直链分子的branch变种, 按照selfies定义的顺序来画图, 但测试了几种化学库, 其都不支持这种形式

# 处理(branch分子, 长度7)的错误-4, 使用规范的smiles表达式和图片

commit: bea9f79a02f8b740e065a0046a54a494a99e7f42

训练步骤: 

  - 步骤1.1: branch_selfies.regular_smiles*1
  - 步骤1.2: random_selfies_only_basic_atom*1):

|  | 步骤1.1 (epoch=3) (LR=1e-4) | 步骤1.2 (epoch=3) (LR=1e-4) | 步骤2.1 (epoch=3) (LR=1e-4) | 步骤2.2 (epoch=3) (LR=1e-4) |
| --- | --- | --- | --- | --- |
| 基础原子, 长度7 | 94.74% | 100.00% | 100.00% | 100.00% |
| 基础原子, 长度8 | 100.00% | 89.47% | 100.00% | 100.00% |
| ring分子, 长度7 | 97.50% | 100.00% | 80.00% | 90.00% |
| ring分子, 长度8 | 92.50% | 85.00% | 85.00% | 87.50% |
| branch分子, 长度7 | 100.00% | 93.33% | 100.00% | 100.00% |
| branch分子, 长度8 | 76.67% | 70.00% | 80.00% | 93.33% |
  
标红的两处, 都是将 branch和直链 识别成对方, 导致了错误. 在迭代后得以修正. 

异常现象: 

  1. 在步骤2中, ring分子长度7的准确率下降, 需要分析. (分析绿色部分):
     1. 模型将ring和branch进行了类比, 让环也处于ring之后, 比如: 
        1. 将 [O][N][C][N][O][Ring1][Ring1] 识别成了 [O][N][C][Ring1][Ring1][O][N]
  2. 在步骤2中, (branch分子, 长度8), 的准确率上升. 分析步骤1中的错误分布 (绿色部分):
     1. 其中3/7是直链相关错误
     2. 在步骤2中的错误都不包括直链, 也就是直链已经被正确处理

先将两步整合在一起, 看一下异常是否仍然存在, 训练步骤: 

(branch_selfies.regular_smiles*1) + (random_selfies_only_basic_atom*1):

|  | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=3) (LR=1e-4) |
| --- | --- | --- |
| 基础原子, 长度7 | 100.00% | 100.00% |
| 基础原子, 长度8 | 100.00% | 100.00% |
| ring分子, 长度7 | 97.50% | 95.00% |
| ring分子, 长度8 | 90.00% | 92.50% |
| branch分子, 长度7 | 100.00% | 96.67% |
| branch分子, 长度8 | 86.67% | 93.33% |
  
通过少量数据训练, 是可以满足长度8的准确率要求

下一步可以往长度9扩展

# 往长度9扩展时, 重新生成了 分子数据, 发现ring分子准确率下降

长度7训练到长度8的脚本: commit (d49d9a75b5eb08e1536ce17bf908b8226905cbf3), 脚本(run_tasks.04.length_8.sh)

以其最后一次训练, 评估长度9:

|  | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=3) (LR=1e-4) | 步骤3 (epoch=3) (LR=1e-4) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 100.00% | 100.00% | 100.00% |
| 基础原子, 长度8 | 94.74% | 94.74% | 100.00% |
| 基础原子, 长度9 | 75.00% | 60.00% | 65.00% |
| ring分子, 长度7 | 80.00% | 85.00% | 85.00% |
| ring分子, 长度8 | 75.00% | 82.50% | 75.00% |
| ring分子, 长度9 | 42.50% | 35.00% | 42.50% |
| branch分子, 长度7 | 100.00% | 100.00% | 100.00% |
| branch分子, 长度8 | 100.00% | 90.00% | 96.67% |
| branch分子, 长度9 | 66.67% | 73.33% | 66.67% |
  
评估两处错误(标红): 

对(ring分子, 长度7)错误评估: 

```
 17   /opt/huangyan/mol-ai/images/ring_selfies/length_7/73.png    {
"selfies": "[O][C][N][Ring1][Ring1][O][O]"}          O1CN1OO    {
"selfies": "[O][O][N][Ring1][Ring1][C][O]"}   O1ON1CO    0.142857   
mismatch
2,6对调 (环内外对调)

29  /opt/huangyan/mol-ai/images/ring_selfies/length_7/199.png    {
"selfies": "[C][N][N][Ring1][Ring1][N][N]"}          C1NN1NN    {
"selfies": "[C][N][N][N][N][Ring1][Ring1]"}   CNN1NN1    0.142857   
mismatch
ring op平移

34  /opt/huangyan/mol-ai/images/ring_selfies/length_7/164.png    {
"selfies": "[C][O][C][Ring1][Ring1][O][O]"}          C1OC1OO    {
"selfies": "[O][O][C][Ring1][Ring1][O][C]"}   O1OC1OC    0.142857   
mismatch
1,7对调 (环内外对调)

1    /opt/huangyan/mol-ai/images/ring_selfies/length_7/64.png    {
"selfies": "[N][C][C][C][Ring1][Ring1][O]"}          NC1CC1O    {
"selfies": "[N][C][C][C][O][Ring1][Ring1]"}   NCC1CO1    0.150000   
mismatch
ring op平移

11   /opt/huangyan/mol-ai/images/ring_selfies/length_7/86.png    {
"selfies": "[O][C][C][C][Ring1][Ring2][C]"}          O1CCC1C    {
"selfies": "[O][C][C][C][C][Ring1][Ring2]"}   OC1CCC1    0.176471   
mismatch
ring op平移

13   /opt/huangyan/mol-ai/images/ring_selfies/length_7/53.png  {
"selfies": "[C][C][O][N][N][Ring1][Branch1]"}          C1CONN1  {
"selfies": "[C][N][O][N][C][Ring1][Branch1]"}   C1NONC1    0.222222   
mismatch
2,5对调 (环内对调)

0   /opt/huangyan/mol-ai/images/ring_selfies/length_7/160.png    {
"selfies": "[C][N][N][Ring1][Ring1][C][N]"}          C1NN1CN    {
"selfies": "[C][N][N][N][C][Ring1][Ring1]"}   CNN1NC1    0.238095   
mismatch
ring op平移, 4,5对调(环内对调)

10   /opt/huangyan/mol-ai/images/ring_selfies/length_7/22.png  {
"selfies": "[O][O][C][O][N][Ring1][Branch1]"}          O1OCON1  {
"selfies": "[O][N][C][O][O][Ring1][Branch1]"}   O1NCOO1    0.238095   
mismatch
2,5对调 (环内对调)

``` 

对(ring分子, 长度8)错误评估: 

```
1    /opt/huangyan/mol-ai/images/ring_selfies/length_8/44.png       {
"selfies": "[N][N][O][N][C][Branch1][C][O]"}           NNONCO     {
"selfies": "[N][N][O][N][C][Ring1][Ring2][O]"}  NN1ONC1O    0.071429   
mismatch
ring识别成了branch

27    /opt/huangyan/mol-ai/images/ring_selfies/length_8/3.png     {
"selfies": "[C][N][C][O][Ring1][Ring2][C][O]"}           C1NCO1     {
"selfies": "[O][C][O][C][N][Ring1][Ring2][C]"}  OC1OCN1C    0.100000   
mismatch
镜像, ring op平移
[C][N][C][O][C][Ring1][Ring2][O]

21  /opt/huangyan/mol-ai/images/ring_selfies/length_8/111.png   {
"selfies": "[O][N][C][N][N][Ring1][Branch1][N]"}         O1NCNN1N   {
"selfies": "[C][N][N][N][N][Ring1][Branch1][O]"}  C1NNNN1O    0.107143   
mismatch
1,3,8对调(环内外对调)

37  /opt/huangyan/mol-ai/images/ring_selfies/length_8/148.png     {
"selfies": "[C][O][C][O][N][Ring1][Ring2][C]"}           COCONC     {
"selfies": "[C][O][C][N][Ring1][Ring2][O][C]"}  C1OCN1OC    0.120000   
mismatch
ring op平移, 4,7对调(环内外对调)

20   /opt/huangyan/mol-ai/images/ring_selfies/length_8/48.png   {
"selfies": "[N][O][N][N][C][Ring1][Branch1][N]"}         N1ONNC1N   {
"selfies": "[C][O][N][N][N][Ring1][Branch1][N]"}  C1ONNN1N    0.148148   
mismatch
1,5对调(环内对调)

30  /opt/huangyan/mol-ai/images/ring_selfies/length_8/187.png        {
"selfies": "[C][O][O][C][Ring1][Ring2][C]"}          C1OOC1C     {
"selfies": "[N][O][O][C][Ring1][Ring2][C][C]"}  N1OOC1CC    0.190476   
mismatch
少原子

39  /opt/huangyan/mol-ai/images/ring_selfies/length_8/198.png     {
"selfies": "[O][C][C][O][Ring1][Ring1][O][O]"}           OC1CO1     {
"selfies": "[O][C][O][C][Ring1][Ring1][O][O]"}  OC1OC1OO    0.222222   
mismatch
3,4对调, 环内对调

34  /opt/huangyan/mol-ai/images/ring_selfies/length_8/149.png   {
"selfies": "[N][N][O][O][C][N][Ring1][Branch1]"}         NN1OOCN1   {
"selfies": "[O][C][N][O][N][Ring1][Branch1][N]"}  O1CNON1N    0.280000   
mismatch
镜像, 4-6轮换(环内轮换)
[N][N][O][C][N][O][Ring1][Branch1]

25   /opt/huangyan/mol-ai/images/ring_selfies/length_8/90.png   {
"selfies": "[C][C][C][N][C][O][Ring1][Branch1]"}         CC1CNCO1   {
"selfies": "[O][C][C][N][C][Ring1][Branch1][C]"}  O1CCNC1C    0.280000   
mismatch
ring op平移, 1,8对调(环内外对调)

11  /opt/huangyan/mol-ai/images/ring_selfies/length_8/107.png     {
"selfies": "[O][N][C][C][O][Ring1][Ring1][O]"}          ONC1CO1     {
"selfies": "[O][C][O][C][Ring1][Ring1][N][O]"}  OC1OC1NO    0.300000   
mismatch
镜像, 4,5对调(环内对调)
[O][N][C][O][C][Ring1][Ring1][O]

 
``` 

增加 various_ring_pos作为训练数据, 训练效果: 

|  | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=3) (LR=1e-4) | 步骤3 (epoch=3) (LR=1e-4) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 100.00% | 94.74% | 100.00% |
| 基础原子, 长度8 | 100.00% | 100.00% | 100.00% |
| 基础原子, 长度9 | 80.00% | 70.00% | 60.00% |
| ring分子, 长度7 | 90.00% | 97.50% | 97.50% |
| ring分子, 长度8 | 90.00% | 90.00% | 92.50% |
| ring分子, 长度9 | 35.00% | 35.00% | 25.00% |
| branch分子, 长度7 | 96.67% | 100.00% | 100.00% |
| branch分子, 长度8 | 90.00% | 93.33% | 93.33% |
| branch分子, 长度9 | 63.33% | 60.00% | 60.00% |
  
认为目前满足长度8的要求, 使用步骤1作为基线

checkpoint: /opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v922-20250215-170059/checkpoint-129

commit: 523d0736195b34affbe3c4f260517feca28cdcc0

分析 (ring分子, 长度7/8)的错误分布: 

```
19  /opt/huangyan/mol-ai/images/ring_selfies/length_7/178.png  {
"selfies": "[N][C][C][N][O][Ring1][Branch1]"}          N1CCNO1  {
"selfies": "[N][C][N][O][C][Ring1][Branch1]"}   N1CNOC1    0.222222   
mismatch
3,4对调(环内)

13   /opt/huangyan/mol-ai/images/ring_selfies/length_7/53.png  {
"selfies": "[C][C][O][N][N][Ring1][Branch1]"}          C1CONN1  {
"selfies": "[C][N][O][N][C][Ring1][Branch1]"}   C1NONC1    0.222222   
mismatch
2,5对调(环内)

25   /opt/huangyan/mol-ai/images/ring_selfies/length_7/47.png  {
"selfies": "[N][C][C][C][N][Ring1][Branch1]"}          N1CCCN1  {
"selfies": "[N][C][C][N][C][Ring1][Branch1]"}   N1CCNC1    0.230769   
mismatch
3,4对调(环内)

10   /opt/huangyan/mol-ai/images/ring_selfies/length_7/22.png  {
"selfies": "[O][O][C][O][N][Ring1][Branch1]"}          O1OCON1  {
"selfies": "[O][N][C][O][O][Ring1][Branch1]"}   O1NCOO1    0.238095   
mismatch
2,5对调(环内)

20   /opt/huangyan/mol-ai/images/ring_selfies/length_8/48.png   {
"selfies": "[N][O][N][N][N][C][Ring1][Branch1]"}           NONNNC   {
"selfies": "[C][O][N][N][N][Ring1][Branch1][N]"}  C1ONNN1N    0.034483   
mismatch
ring op平移, 1,8对调(环内外)

37  /opt/huangyan/mol-ai/images/ring_selfies/length_8/148.png     {
"selfies": "[C][O][C][O][N][Ring1][Ring2][C]"}           COCONC     {
"selfies": "[C][O][C][N][Ring1][Ring2][O][C]"}  C1OCN1OC    0.120000   
mismatch
ring op平移, 4,7对调(环内外)

39  /opt/huangyan/mol-ai/images/ring_selfies/length_8/198.png     {
"selfies": "[O][O][C][C][O][O][Ring1][Ring1]"}         OOCC1OO1     {
"selfies": "[O][C][O][C][Ring1][Ring1][O][O]"}  OC1OC1OO    0.227273   
mismatch
ring op平移, 2,3对调(环内)

11  /opt/huangyan/mol-ai/images/ring_selfies/length_8/107.png     {
"selfies": "[O][N][C][C][O][Ring1][Ring1][O]"}          ONC1CO1     {
"selfies": "[O][C][O][C][Ring1][Ring1][N][O]"}  OC1OC1NO    0.300000   
mismatch
镜像, 4,5对调(环内)
[O][N][C][O][C][Ring1][Ring1][O] 
``` 

都需要原子对调, 长度8都需要ring op平移
