---
title: 20250215 - 使用ms-swift对Qwen2-VL进行微调 [25] - 扩展长度9
confluence_page_id: 3344345
created_at: 2025-02-15T08:47:22+00:00
updated_at: 2025-02-21T15:18:34+00:00
---

# 前置基线

[20250212 - 使用ms-swift对Qwen2-VL进行微调 [24] - 研究训练数据的收敛]

checkpoint: /opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v922-20250215-170059/checkpoint-129

commit: 523d0736195b34affbe3c4f260517feca28cdcc0

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
  
与长度7 -> 长度8不同, 尝试用新的理解 (包括smiles正规化等), 来完成 长度8->长度9.

# 分析(基础原子,长度9)的错误

```
16  /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_9/14.png     {
"selfies": "[N][C][O][C][N][N][N][C][N]"}        NCOCNNNCN  {
"selfies": "[N][C][O][N][C][N][N][C][N]"}  NCONCNNCN    0.333333   
mismatch
4,5对调

3   /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_9/28.png  {
"selfies": "[C][C][C][O][O][C][C][C][C][C]"}       CCCOOCCCCC  {
"selfies": "[C][C][C][O][C][O][C][C][C]"}  CCCOCOCCC    0.350000   
mismatch
多一个原子, 6,7对调

11  /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_9/89.png     {
"selfies": "[C][O][N][O][N][N][C][O][C]"}        CONONNCOC  {
"selfies": "[C][O][N][O][O][N][N][C][C]"}  CONOONNCC    0.357143   
mismatch
5-8轮换

0    /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_9/4.png     {
"selfies": "[N][O][O][N][C][N][C][O][N]"}        NOONCNCON  {
"selfies": "[N][C][N][C][N][O][O][O][N]"}  NCNCNOOON    0.444444   
mismatch
镜像, 4-8轮换
``` 

# 分析(branch分子,长度9)的错误

```
21   /opt/huangyan/mol-ai/images/branch_selfies/length_9/36.png  {
"selfies": "[N][C][C][N][Branch1][C][N][C][O]"}        NCCN(N)CO         {
"selfies": "[C][N][Branch1][C][O][N][C][C][N]"}  CN(O)NCCN    0.230769   
mismatch
镜像, branch op平移
[N][C][C][N][N][Branch1][C][C][O]

1    /opt/huangyan/mol-ai/images/branch_selfies/length_9/47.png  {
"selfies": "[O][O][N][Branch1][C][O][O][N][N]"}        OON(O)ONN     {
"selfies": "[N][O][N][Branch1][Ring1][O][O][N][O]"}  NON(OO)NO    0.280000   
mismatch
branch长度错误, 1,9对调

29   /opt/huangyan/mol-ai/images/branch_selfies/length_9/11.png              {
"selfies": "[C][N][N][N][N][O][O]"}          CNNNNOO         {
"selfies": "[C][N][Branch1][C][N][N][N][O][O]"}  CN(N)NNOO    0.333333   
mismatch
缺少branch op

22   /opt/huangyan/mol-ai/images/branch_selfies/length_9/44.png  {
"selfies": "[N][N][O][N][Branch1][C][N][O][O]"}        NNON(N)OO     {
"selfies": "[N][N][Branch1][Ring2][O][N][O][O][N]"}  NN(ONO)ON    0.333333   
mismatch
branch长度错误, branch op平移

18  /opt/huangyan/mol-ai/images/branch_selfies/length_9/127.png  {
"selfies": "[N][N][C][Branch1][C][N][O][O][O]"}        NNC(N)OOO         {
"selfies": "[N][N][C][Branch1][C][O][N][O][O]"}  NNC(O)NOO    0.333333   
mismatch
6,7对调

5    /opt/huangyan/mol-ai/images/branch_selfies/length_9/51.png  {
"selfies": "[N][O][N][Branch1][C][O][O][N][N]"}        NON(O)ONN     {
"selfies": "[N][O][N][Branch1][Ring2][N][O][N][O]"}  NON(NON)O    0.347826   
mismatch
branch长度错误, 6,9对调

9    /opt/huangyan/mol-ai/images/branch_selfies/length_9/68.png  {
"selfies": "[N][O][O][C][Branch1][C][N][O][O]"}        NOOC(N)OO     {
"selfies": "[N][O][O][C][Branch1][Ring1][N][O][O]"}  NOOC(NO)O    0.347826   
mismatch
branch长度错误

10   /opt/huangyan/mol-ai/images/branch_selfies/length_9/14.png  {
"selfies": "[O][N][N][Branch1][C][O][N][N][C]"}        ONN(O)NNC     {
"selfies": "[C][N][N][Branch1][Ring1][N][O][N][O]"}  CNN(NO)NO    0.350000   
mismatch
镜像, branch长度错误, 复杂变换
[O][N][N][Branch1][Ring1][N][C][N][O]

7    /opt/huangyan/mol-ai/images/branch_selfies/length_9/63.png  {
"selfies": "[C][C][Branch1][C][N][N][N][N][C]"}        CC(N)NNNC         {
"selfies": "[N][N][C][Branch1][C][C][N][N][C]"}  NNC(C)NNC    0.363636   
mismatch
起点4, branch长度错误
[C][C][Branch1][Ring1][N][N][N][N][C]

28   /opt/huangyan/mol-ai/images/branch_selfies/length_9/32.png  {
"selfies": "[O][O][O][C][Branch1][C][N][N][O]"}        OOOC(N)NO         {
"selfies": "[N][N][C][Branch1][C][O][O][O][O]"}  NNC(O)OOO    0.391304   
mismatch
镜像, 7,9对调
[O][O][O][C][Branch1][C][O][N][N]

20   /opt/huangyan/mol-ai/images/branch_selfies/length_9/79.png  {
"selfies": "[N][O][N][N][Branch1][C][N][O][C]"}        NONN(N)OC     {
"selfies": "[N][O][N][Branch1][Ring2][N][O][N][C]"}  NON(NON)C    0.409091   
mismatch
起点6, 7,9对调
[N][O][N][N][Branch1][C][C][O][N]
``` 

# 分析(ring分子,长度9)的错误

```
8   /opt/huangyan/mol-ai/images/ring_selfies/length_9/120.png   {
"selfies": "[O][N][O][N][C][C][O][Ring1][Branch1]"}          ONONCCO  {
"selfies": "[O][C][N][O][N][O][C][Ring1][=Branch1]"}  OC1NONOC1    0.030303   
mismatch
2-5轮换, 6-7轮换, ring长度错误

21   /opt/huangyan/mol-ai/images/ring_selfies/length_9/62.png     {
"selfies": "[O][N][N][O][N][C][C][Ring1][Ring2]"}          ONNONCC  {
"selfies": "[O][N][C][N][O][N][C][Ring1][=Branch1]"}  ON1CNONC1    0.033333   
mismatch
3-7轮换, ring长度错误

35   /opt/huangyan/mol-ai/images/ring_selfies/length_9/82.png   {
"selfies": "[O][O][N][N][C][C][Ring1][Branch1][O]"}          OONNCCO   {
"selfies": "[O][C][N][O][N][C][Ring1][Branch1][O]"}  OC1NONC1O    0.038462   
mismatch
2,4,5 对调

24  /opt/huangyan/mol-ai/images/ring_selfies/length_9/136.png   {
"selfies": "[N][N][N][C][O][C][Ring1][Branch1][N]"}        NN1NCOC1N   {
"selfies": "[N][N][N][C][N][O][C][Ring1][Branch1]"}  NNN1CNOC1    0.156250   
mismatch
5-9轮换

13  /opt/huangyan/mol-ai/images/ring_selfies/length_9/108.png     {
"selfies": "[O][N][C][C][Ring1][Ring2][C][C][C]"}        O1NCC1CCC  {
"selfies": "[C][C][O][C][N][C][C][Ring1][=Branch1]"}  CC1OCNCC1    0.161290   
mismatch
复杂错误

39  /opt/huangyan/mol-ai/images/ring_selfies/length_9/122.png     {
"selfies": "[C][C][C][C][C][C][N][Ring1][Ring1]"}        CCCCC1CN1     {
"selfies": "[C][N][C][C][C][Ring1][Ring1][C][C]"}  CNC1CC1CC    0.172414   
mismatch
镜像, ring op平移, 8,9对调
[C][C][C][C][C][Ring1][Ring1][N][C]

17   /opt/huangyan/mol-ai/images/ring_selfies/length_9/51.png     {
"selfies": "[C][N][C][Ring1][Ring1][N][N][C][C]"}        C1NC1NNCC     {
"selfies": "[C][N][N][C][N][C][Ring1][Ring1][C]"}  CNNC1NC1C    0.178571   
mismatch
镜像, 1-9轮换

25  /opt/huangyan/mol-ai/images/ring_selfies/length_9/100.png     {
"selfies": "[C][N][O][C][Ring1][Ring1][C][N][C]"}        CN1OC1CNC     {
"selfies": "[C][N][C][O][C][Ring1][Ring1][C][N]"}  CNC1OC1CN    0.206897   
mismatch
3-9轮换

15   /opt/huangyan/mol-ai/images/ring_selfies/length_9/12.png     {
"selfies": "[O][C][C][N][O][O][C][Ring1][Ring2]"}        OCCN1OOC1     {
"selfies": "[O][C][N][O][O][C][Ring1][Ring2][C]"}  OCN1OOC1C    0.206897   
mismatch
3-9轮换

0   /opt/huangyan/mol-ai/images/ring_selfies/length_9/113.png   {
"selfies": "[O][O][C][C][C][Ring1][Branch1][C][N]"}        O1OCCC1CN   {
"selfies": "[N][C][O][O][C][C][Ring1][Branch1][C]"}  NC1OOCC1C    0.206897   
mismatch
1-9轮换, 环内再轮换

7    /opt/huangyan/mol-ai/images/ring_selfies/length_9/33.png     {
"selfies": "[N][C][N][N][O][O][O][Ring1][Ring2]"}        NCNN1OOO1   {
"selfies": "[O][O][O][N][N][Ring1][Branch1][C][N]"}  O1OONN1CN    0.214286   
mismatch
镜像, ring长度错误

19   /opt/huangyan/mol-ai/images/ring_selfies/length_9/63.png     {
"selfies": "[O][C][C][N][O][N][Ring1][Ring2][O]"}        OCC1NON1O   {
"selfies": "[O][C][N][O][N][Ring1][Branch1][C][O]"}  O1CNON1CO    0.233333   
mismatch
3-8轮换, ring长度错误

33   /opt/huangyan/mol-ai/images/ring_selfies/length_9/39.png     {
"selfies": "[O][N][O][C][N][N][O][Ring1][Ring2]"}        ONOC1NNO1     {
"selfies": "[O][N][O][C][N][N][O][Ring1][Ring1]"}  ONOCN1NO1    0.233333   
mismatch
ring长度错误

16    /opt/huangyan/mol-ai/images/ring_selfies/length_9/8.png   {
"selfies": "[N][C][C][N][N][Ring1][Branch1][N][O]"}        N1CCNN1NO   {
"selfies": "[O][N][N][C][C][N][Ring1][Branch1][N]"}  ON1NCCN1N    0.240000   
mismatch
1-9轮换

29  /opt/huangyan/mol-ai/images/ring_selfies/length_9/131.png  {
"selfies": "[C][N][N][O][O][N][Ring1][=Branch1][C]"}        C1NNOON1C  {
"selfies": "[C][N][N][C][O][O][N][Ring1][=Branch1]"}  CN1NCOON1    0.250000   
mismatch
4-9轮换

6   /opt/huangyan/mol-ai/images/ring_selfies/length_9/109.png     {
"selfies": "[C][C][C][N][N][Ring1][Ring1][C][N]"}        CCC1NN1CN     {
"selfies": "[C][C][N][C][N][Ring1][Ring1][C][N]"}  CCN1CN1CN    0.259259   
mismatch
3,4对调

28    /opt/huangyan/mol-ai/images/ring_selfies/length_9/4.png     {
"selfies": "[C][C][C][C][N][C][Ring1][Ring2][C]"}        CCC1CNC1C     {
"selfies": "[C][C][N][C][Ring1][Ring2][C][C][C]"}  C1CNC1CCC    0.259259   
mismatch
3-9轮换 * 2

12   /opt/huangyan/mol-ai/images/ring_selfies/length_9/52.png     {
"selfies": "[C][N][N][N][C][C][C][Ring1][Ring2]"}        CNNN1CCC1   {
"selfies": "[C][C][C][N][N][Ring1][Branch1][N][C]"}  C1CCNN1NC    0.269231   
mismatch
镜像, ring长度错误, 3-7轮换
[C][N][N][C][C][C][N][Ring1][Branch1]

30   /opt/huangyan/mol-ai/images/ring_selfies/length_9/94.png     {
"selfies": "[C][C][N][C][Ring1][Ring1][N][N][C]"}        CC1NC1NNC     {
"selfies": "[C][N][N][C][C][N][Ring1][Ring1][C]"}  CNNC1CN1C    0.291667   
mismatch
镜像, 2,3对调
[C][N][C][C][Ring1][Ring1][N][N][C]

14   /opt/huangyan/mol-ai/images/ring_selfies/length_9/21.png     {
"selfies": "[N][N][O][C][O][C][N][Ring1][Ring1]"}          NNOCOCN     {
"selfies": "[N][N][O][C][C][O][N][Ring1][Ring1]"}  NNOCC1ON1    0.296296   
mismatch
5,6对调

37   /opt/huangyan/mol-ai/images/ring_selfies/length_9/75.png  {
"selfies": "[O][C][O][O][N][Ring1][=Branch1][C][C]"}        O1COON1CC  {
"selfies": "[C][O][C][O][O][N][Ring1][=Branch1][C]"}  C1OCOON1C    0.307692   
mismatch
1-9轮换

18  /opt/huangyan/mol-ai/images/ring_selfies/length_9/189.png  {
"selfies": "[O][N][C][C][N][C][N][Ring1][=Branch1]"}        ON1CCNCN1  {
"selfies": "[O][N][C][N][C][C][N][Ring1][=Branch1]"}  ON1CNCCN1    0.375000   
mismatch
4,5对调

31  /opt/huangyan/mol-ai/images/ring_selfies/length_9/119.png     {
"selfies": "[C][N][C][N][N][Ring1][Ring1][C][C]"}        CNC1NN1CC     {
"selfies": "[C][N][C][N][N][C][Ring1][Ring1][C]"}  CNCN1NC1C    0.400000   
mismatch
6-9轮换

9   /opt/huangyan/mol-ai/images/ring_selfies/length_9/167.png     {
"selfies": "[C][O][C][C][C][N][C][Ring1][Ring2]"}        COCC1CNC1     {
"selfies": "[C][O][C][C][C][C][N][Ring1][Ring2]"}  COCC1CCN1    0.400000   
mismatch
6,7对调

10  /opt/huangyan/mol-ai/images/ring_selfies/length_9/169.png  {
"selfies": "[O][O][C][O][O][N][Ring1][=Branch1][C]"}        O1OCOON1C  {
"selfies": "[O][C][O][O][O][N][Ring1][=Branch1][C]"}  O1COOON1C    0.428571   
mismatch
2,3对调

11  /opt/huangyan/mol-ai/images/ring_selfies/length_9/134.png        {
"selfies": "[C][C][C][C][N][Ring1][Ring2][C]"}         CC1CCN1C     {
"selfies": "[C][N][C][C][C][Ring1][Ring2][O][C]"}  CN1CCC1OC    0.428571   
mismatch
少原子, 2,5对调

``` 

# 增加基础分子和branch分子训练

训练数据: branch_selfies.regular_smiles*1 + random_selfies_only_basic_atom.regular_smiles*1

|  | 基线 | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=3) (LR=5e-5) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 100.00% | 89.47% | 89.47% |
| 基础原子, 长度8 | 100.00% | 100.00% | 100.00% |
| 基础原子, 长度9 | 80.00% | 100.00% | 95.00% |
| ring分子, 长度7 | 90.00% | 97.50% | 95.00% |
| ring分子, 长度8 | 90.00% | 67.50% | 70.00% |
| ring分子, 长度9 | 35.00% | 22.50% | 30.00% |
| branch分子, 长度7 | 96.67% | 83.33% | 83.33% |
| branch分子, 长度8 | 90.00% | 76.67% | 83.33% |
| branch分子, 长度9 | 63.33% | 80.00% | 73.33% |
  
先处理标红的 对于"记忆"的衰退:

```
 9   /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_7/56.png  {
"selfies": "[C][N][N][N][Branch1][C][O][O][C]"}        CNNN(O)OC  {
"selfies": "[C][N][N][N][O][C][O]"}   CNNNOCO    0.222222   
mismatch
基础分子被识别成branch

10  /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_7/49.png  {
"selfies": "[O][C][C][N][Branch1][C][O][N][O]"}        OCCN(O)NO  {
"selfies": "[O][C][C][N][O][N][O]"}   OCCNONO    0.240000   
mismatch
基础分子被识别成branch

23  /opt/huangyan/mol-ai/images/branch_selfies/length_7/133.png  {
"selfies": "[O][C][N][Branch1][C][O][O]"}          OCN(O)O      {
"selfies": "[O][Branch1][C][O][N][C][O]"}   O(O)NCO    0.176471   
mismatch
基础分子被识别成branch
[O][C][N][O][O]

11  /opt/huangyan/mol-ai/images/branch_selfies/length_7/108.png  {
"selfies": "[O][C][N][Branch1][C][O][O]"}          OCN(O)O  {
"selfies": "[N][Branch1][Ring1][O][O][C][O]"}   N(OO)CO    0.176471   
mismatch
基础分子被识别成branch
[O][C][N][O][O]

20  /opt/huangyan/mol-ai/images/branch_selfies/length_7/120.png  {
"selfies": "[N][O][N][Branch1][C][N][N]"}          NON(N)N      {
"selfies": "[O][Branch1][C][N][N][N][N]"}   O(N)NNN    0.187500   
mismatch
基础分子被识别成branch
[N][N][N][O][N]

14   /opt/huangyan/mol-ai/images/branch_selfies/length_7/67.png  {
"selfies": "[N][N][N][Branch1][C][N][N]"}          NNN(N)N      {
"selfies": "[N][Branch1][C][N][N][N][N]"}   N(N)NNN    0.250000   
mismatch
基础分子被识别成branch
[N][N][N][N][N]

18   /opt/huangyan/mol-ai/images/branch_selfies/length_7/66.png  {
"selfies": "[N][N][N][Branch1][C][N][N]"}          NNN(N)N  {
"selfies": "[N][Branch1][Ring2][N][N][N][N]"}   N(NNN)N    0.250000   
mismatch
基础分子被识别成branch
[N][N][N][N][N]

25   /opt/huangyan/mol-ai/images/branch_selfies/length_8/84.png      {
"selfies": "[N][C][C][N][Branch1][C][O][O]"}         NCCN(O)O    {
"selfies": "[C][Branch1][Ring1][C][N][N][O][O]"}  C(CN)NOO    0.227273   
mismatch
基础分子被识别成branch
[N][C][C][N][O][O]

14  /opt/huangyan/mol-ai/images/branch_selfies/length_8/105.png      {
"selfies": "[N][O][N][N][Branch1][C][O][C]"}         NONN(O)C    {
"selfies": "[N][Branch1][Ring2][N][O][N][C][O]"}  N(NON)CO    0.250000   
mismatch
基础分子被识别成branch
[N][O][N][N][C][O]

21   /opt/huangyan/mol-ai/images/branch_selfies/length_8/29.png      {
"selfies": "[N][O][C][N][Branch1][C][N][N]"}         NOCN(N)N    {
"selfies": "[C][Branch1][Ring2][N][N][N][O][N]"}  C(NNN)ON    0.250000   
mismatch
基础分子被识别成branch
[N][O][C][N][N][N]

22  /opt/huangyan/mol-ai/images/branch_selfies/length_8/121.png      {
"selfies": "[N][C][N][N][Branch1][C][O][C]"}         NCNN(O)C  {
"selfies": "[C][Branch1][Branch1][N][N][C][N][O]"}  C(NNCN)O    0.272727   
mismatch
基础分子被识别成branch
[N][C][N][N][C][O]

18   /opt/huangyan/mol-ai/images/branch_selfies/length_8/90.png      {
"selfies": "[N][O][C][N][Branch1][C][O][O]"}         NOCN(O)O  {
"selfies": "[O][Branch1][Branch1][N][C][O][N][O]"}  O(NCON)O    0.285714   
mismatch
基础分子被识别成branch
[N][O][C][N][O][O]

12  /opt/huangyan/mol-ai/images/branch_selfies/length_8/112.png      {
"selfies": "[C][C][Branch1][C][C][C][N][N]"}         CC(C)CNN        {
"selfies": "[N][N][C][Branch1][C][C][C][C]"}  NNC(C)CC    0.350000   
mismatch
镜像, branch op平移
[C][C][C][Branch1][C][C][N][N]

17   /opt/huangyan/mol-ai/images/branch_selfies/length_8/50.png      {
"selfies": "[N][N][C][Branch1][C][C][N][N]"}         NNC(C)NN    {
"selfies": "[N][C][Branch1][Ring1][N][N][N][C]"}  NC(NN)NC    0.375000   
mismatch
起点4, 6,8对调 (分支内外对调)
[N][N][C][Branch1][C][N][N][C]

``` 

大部分错误都是在直链分子中间直接添加了一个branch op

增加branch op的训练数据: branch_selfies.regular_smiles*1 + random_selfies_only_basic_atom.regular_smiles*1 + add_branch_op_from_random_selfies_only_basic_atom.regular_smiles*1

训练效果: 

|  | 基线 | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=3) (LR=5e-5) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 100.00% | 100.00% | 94.74% |
| 基础原子, 长度8 | 100.00% | 100.00% | 100.00% |
| 基础原子, 长度9 | 80.00% | 80.00% | 80.00% |
| ring分子, 长度7 | 90.00% | 90.00% | 92.50% |
| ring分子, 长度8 | 90.00% | 75.00% | 75.00% |
| ring分子, 长度9 | 35.00% | 30.00% | 35.00% |
| branch分子, 长度7 | 96.67% | 100.00% | 100.00% |
| branch分子, 长度8 | 90.00% | 96.67% | 96.67% |
| branch分子, 长度9 | 63.33% | 80.00% | 80.00% |
  
(branch分子, 长度9) 有提升但不达标, 继续分析错误: 

```
 7    /opt/huangyan/mol-ai/images/branch_selfies/length_9/47.png      {
"selfies": "[N][O][O][N][Branch1][C][N][O][O]"}        NOON(N)OO     {
"selfies": "[N][O][N][Branch1][Ring1][O][O][N][O]"}  NON(OO)NO    0.291667   
mismatch
起点5, branch长度错误(Ring1识别成C), 1-9轮换
[O][O][N][Branch1][Ring1][N][O][O][N]

9    /opt/huangyan/mol-ai/images/branch_selfies/length_9/54.png  {
"selfies": "[N][C][Branch1][Ring1][C][O][N][O][O]"}        NC(CO)NOO         {
"selfies": "[O][C][Branch1][C][N][O][N][C][O]"}  OC(N)ONCO    0.307692   
mismatch
起点3, branch长度错误(Ring1识别成C), 5/8对调
[N][C][Branch1][C][O][O][N][C][O]

2    /opt/huangyan/mol-ai/images/branch_selfies/length_9/86.png      {
"selfies": "[O][O][N][N][Branch1][C][O][N][C]"}        OONN(O)NC         {
"selfies": "[C][N][Branch1][C][O][N][N][O][O]"}  CN(O)NNOO    0.333333   
mismatch
镜像, branch op右移, 4/8对调
[O][O][N][N][N][Branch1][C][C][O]

25   /opt/huangyan/mol-ai/images/branch_selfies/length_9/22.png      {
"selfies": "[N][O][N][Branch1][C][O][O][N][O]"}        NON(O)ONO     {
"selfies": "[O][O][N][Branch1][Ring1][O][N][N][O]"}  OON(ON)NO    0.333333   
mismatch
起点5, branch长度错误(Ring1识别成C), 6/8对调
[N][O][N][Branch1][Ring1][N][O][O][O]

24   /opt/huangyan/mol-ai/images/branch_selfies/length_9/20.png  {
"selfies": "[N][N][Branch1][Ring1][N][O][N][C][O]"}        NN(NO)NCO         {
"selfies": "[O][C][N][N][N][Branch1][C][O][N]"}  OCNNN(O)N    0.333333   
mismatch
镜像, branch长度错误(C识别成Ring1), 5/6对调
[N][N][Branch1][C][O][N][N][C][O]

14   /opt/huangyan/mol-ai/images/branch_selfies/length_9/12.png      {
"selfies": "[N][O][O][C][Branch1][C][N][O][N]"}        NOOC(N)ON     {
"selfies": "[N][O][O][C][Branch1][Ring1][N][O][N]"}  NOOC(NO)N / NOOC(N)NO    0.380952   
mismatch
branch长度错误(Ring1识别成C)

11  /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_9/98.png  {
"selfies": "[C][O][C][C][C][O][N][N][C]"}        COCCCONNC  {
"selfies": "[C][O][C][C][C][N][O][N][C]"}  COCCCNONC    0.392857   
mismatch
6/7对调

4   /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_9/71.png  {
"selfies": "[N][N][O][O][C][N][N][N][O]"}        NNOOCNNNO  {
"selfies": "[N][N][N][C][O][O][N][N][O]"}  NNNCOONNO    0.500000   
mismatch
镜像, 1-9轮换
[O][N][N][O][O][C][N][N][N]

6   /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_9/11.png  {
"selfies": "[N][O][O][N][O][C][N][O][N]"}        NOONOCNON  {
"selfies": "[O][N][O][N][O][C][N][O][N]"}  ONONOCNON    0.538462   
mismatch
1/2对调

13  /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_9/99.png     {
"selfies": "[N][C][N][N][N][N][C][N]"}         NCNNNNCN  {
"selfies": "[N][N][C][N][N][N][N][C][N]"}  NNCNNNNCN    0.625000   
mismatch
分子头少原子

``` 

分析:

  - 对于直链分子, 关注于原子加载头部和尾部的区别.
  - 对于branch分子, 关注于其branch的长度差 (尤其是 Ring1识别成C)
    - 发现在 add_branch_op_from_random_selfies_only_basic_atom.regular_smiles 的数据中, 没有 "[Branch1][Ring1]"的数据
      - 其主要原因是在规范化时, 长度为2的分支失败率比较高, 比如: NON(OO)NO 和 NON(NO)OO, 规范化只认后面一个, 前面的不规范. 这样就导致 长度为2的分支 数量少

修正add_branch_op_from_random_selfies_only_basic_atom.regular_smiles , 算法调整: 

  - 将所有生成的selfies, 让不同长度的branch尽量均衡
  - 对于均衡后仍确实的部分, 选择在原始数组中最相近的selfies (位置相近的selfies 可能是 同源的, 其有关联)

训练数据: branch_selfies.regular_smiles*1 + random_selfies_only_basic_atom.regular_smiles*1 + add_branch_op_from_random_selfies_only_basic_atom.regular_smiles (改)*1

commit: abf37a0ae47a02e0a6d24df9ca1c7a7dee9b9a04

训练效果: 

|  | 基线 | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=3) (LR=5e-5) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 100.00% | 100.00% | 100.00% |
| 基础原子, 长度8 | 100.00% | 100.00% | 90.00% |
| 基础原子, 长度9 | 80.00% | 100.00% | 100.00% |
| ring分子, 长度7 | 90.00% | 90.00% | 92.50% |
| ring分子, 长度8 | 90.00% | 72.50% | 82.50% |
| ring分子, 长度9 | 35.00% | 42.50% | 42.50% |
| branch分子, 长度7 | 96.67% | 100.00% | 100.00% |
| branch分子, 长度8 | 90.00% | 90.00% | 96.67% |
| branch分子, 长度9 | 63.33% | 90.00% | 83.33% |
  
  

基本达标:

  - 基础原子的提升原因未知 (branch的增强数据改进, 为什么会导致 基础原子的准确率上升)
    - 准备的增强数据: add_head_tail_atom_from_random_selfies_only_basic_atom.regular_smiles, 并没有使用
  - (branch分子, 长度9) 随着epoch增加, 准确率还会下降, 放到后面再处理

  

开始改进ring分子的问题

  

# 改进ring分子的训练

对于上述结果, 进行ring分子的错误分析 (只进行了部分错误分析): 

```
36   /opt/huangyan/mol-ai/images/ring_selfies/length_8/80.png    {
"selfies": "[C][N][N][N][Branch1][C][C][N][N]"}        CNNN(C)NN     {
"selfies": "[C][N][N][N][Ring1][Ring2][N][C]"}  C1NNN1NC    0.111111   
mismatch
Ring识别成branch

15  /opt/huangyan/mol-ai/images/ring_selfies/length_8/122.png     {
"selfies": "[C][C][N][N][Ring1][Ring1][O][O]"}         CC1NN1OO     {
"selfies": "[O][N][O][N][Ring1][Ring1][C][C]"}  ON1ON1CC    0.115385   
mismatch
镜像, ring op后移一位, 4/5对调
[C][C][N][O][N][Ring1][Ring1][O]

13    /opt/huangyan/mol-ai/images/ring_selfies/length_8/1.png     {
"selfies": "[C][N][C][C][Ring1][Ring1][C][O]"}         CN1CC1CO     {
"selfies": "[O][C][C][C][Ring1][Ring1][N][C]"}  OC1CC1NC    0.160000   
mismatch
镜像, ring op后移一位
[C][N][C][C][C][Ring1][Ring1][O]

7     /opt/huangyan/mol-ai/images/ring_selfies/length_8/2.png     {
"selfies": "[C][C][N][C][N][Ring1][Ring2][C]"}         CC1NCN1C     {
"selfies": "[C][C][N][C][C][N][Ring1][Ring2]"}  CCN1CCN1    0.160000   
mismatch
ring op后移一位, 5/6对调

25  /opt/huangyan/mol-ai/images/ring_selfies/length_8/126.png     {
"selfies": "[O][C][O][N][Ring1][Ring2][O][O]"}         O1CON1OO     {
"selfies": "[O][C][O][O][N][Ring1][Ring2][O]"}  OC1OON1O    0.173913   
mismatch
ring op后移一位, 4/5对调

14  /opt/huangyan/mol-ai/images/ring_selfies/length_8/144.png     {
"selfies": "[O][O][N][O][N][Ring1][Ring1][O]"}         OON1ON1O     {
"selfies": "[O][N][N][Ring1][Ring1][O][O][O]"}  O1NN1OOO    0.217391   
mismatch
ring op后移两位, 2/5对调

30   /opt/huangyan/mol-ai/images/ring_selfies/length_8/20.png  {
"selfies": "[C][C][C][N][N][N][Ring1][=Branch1]"}         C1CCNNN1  {
"selfies": "[N][C][C][N][C][N][Ring1][=Branch1]"}  N1CCNCN1    0.222222   
mismatch
1/5对调

3   /opt/huangyan/mol-ai/images/ring_selfies/length_8/149.png   {
"selfies": "[N][N][O][O][N][C][Ring1][Branch1]"}         NN1OONC1   {
"selfies": "[O][C][N][O][N][Ring1][Branch1][N]"}  O1CNON1N    0.280000   
mismatch
镜像, 4/6对调
[N][N][O][C][N][O][Ring1][Branch1]

19  /opt/huangyan/mol-ai/images/ring_selfies/length_8/198.png     {
"selfies": "[O][O][C][C][O][Ring1][Ring1][O]"}          OOC1CO1     {
"selfies": "[O][C][O][C][Ring1][Ring1][O][O]"}  OC1OC1OO    0.300000   
mismatch
镜像, 4/5对调
[O][O][C][O][C][Ring1][Ring1][O]

6    /opt/huangyan/mol-ai/images/ring_selfies/length_8/60.png   {
"selfies": "[N][C][C][N][C][Ring1][Branch1][N]"}         N1CCNC1N   {
"selfies": "[C][N][C][N][C][Ring1][Branch1][N]"}  C1NCNC1N    0.300000   
mismatch
1/2对调

38  /opt/huangyan/mol-ai/images/ring_selfies/length_8/102.png     {
"selfies": "[O][O][C][C][Ring1][Ring2][C][N]"}         O1OCC1CN     {
"selfies": "[O][C][O][C][Ring1][Ring2][C][N]"}  O1COC1CN    0.318182   
mismatch
2/3对调

0   /opt/huangyan/mol-ai/images/ring_selfies/length_9/180.png     {
"selfies": "[O][O][O][N][C][Ring1][Ring2][C][O]"}          OOONCCO  {
"selfies": "[N][O][O][O][C][C][Ring1][=Branch1][O]"}  N1OOOCC1O    0.030303   
mismatch
ring长度错误, ring op后移一位, 1/4对调

9    /opt/huangyan/mol-ai/images/ring_selfies/length_9/44.png     {
"selfies": "[N][N][N][O][N][C][C][Ring1][Ring2]"}          NNNONCC   {
"selfies": "[C][C][N][O][N][Ring1][Branch1][N][N]"}  C1CNON1NN    0.090909   
mismatch
镜像, ring长度错误, 4-7镜像
[N][N][N][C][C][N][O][Ring1][Branch1]

4    /opt/huangyan/mol-ai/images/ring_selfies/length_9/13.png     {
"selfies": "[O][C][N][O][N][Ring1][Ring2][N][O]"}        OC1NON1NO     {
"selfies": "[O][C][N][O][O][N][Ring1][Ring2][N]"}  OCN1OON1N    0.096774   
mismatch
ring op后移一位, 5/9对调

30   /opt/huangyan/mol-ai/images/ring_selfies/length_9/91.png  {
"selfies": "[C][N][N][C][C][O][C][Ring1][=Branch1]"}        CN1NCCOC1  {
"selfies": "[C][C][O][C][N][N][C][Ring1][=Branch1]"}  CC1OCNNC1    0.129032   
mismatch
镜像, ring op前移一位, 5/6对调
[C][N][N][C][O][C][Ring1][=Branch1][C]

15  /opt/huangyan/mol-ai/images/ring_selfies/length_9/198.png     {
"selfies": "[C][C][C][C][N][O][C][Ring1][Ring1]"}        CCCCN1OC1     {
"selfies": "[C][N][C][O][C][Ring1][Ring1][C][C]"}  CNC1OC1CC    0.133333   
mismatch
镜像, ring op前移两位, 4-6轮换
[C][C][C][O][C][Ring1][Ring1][N][C]

3   /opt/huangyan/mol-ai/images/ring_selfies/length_9/196.png     {
"selfies": "[C][C][C][C][N][N][Ring1][Ring2][C]"}        CCC1CNN1C     {
"selfies": "[N][C][C][C][Ring1][Ring2][C][N][C]"}  N1CCC1CNC    0.161290   
mismatch
ring op前移两位, 1/7对调

28    /opt/huangyan/mol-ai/images/ring_selfies/length_9/8.png   {
"selfies": "[N][C][N][C][N][Ring1][Branch1][N][O]"}        N1CNCN1NO   {
"selfies": "[O][N][N][C][C][N][Ring1][Branch1][N]"}  ON1NCCN1N    0.166667   
mismatch
镜像, 2/3对调, ring op后移一位
[N][N][C][C][N][N][Ring1][Branch1][O]

29  /opt/huangyan/mol-ai/images/ring_selfies/length_9/129.png     {
"selfies": "[C][O][C][N][C][O][O][Ring1][Ring2]"}        COCN1COO1   {
"selfies": "[O][O][C][N][C][Ring1][Branch1][O][C]"}  O1OCNC1OC    0.193548   
mismatch
镜像, ring长度错误
[C][O][C][N][C][O][O][Ring1][Branch1]
``` 

先解决ring op位置和ring长度错误问题, 模仿branch, 使用add_ring_op_from_random_selfies_only_basic_atom.regular_smiles*1来训练: 

在上一步的步骤1后, 进行继续训练: 

commit: 799b6f94e158aec008b77cb3e138800bb50fac7c

|  | 基线 | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=3) (LR=5e-5) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 100.00% | 100.00% | 100.00% |
| 基础原子, 长度8 | 100.00% | 85.00% | 95.00% |
| 基础原子, 长度9 | 100.00% | 85.00% | 90.00% |
| ring分子, 长度7 | 90.00% | 85.00% | 77.50% |
| ring分子, 长度8 | 72.50% | 67.50% | 60.00% |
| ring分子, 长度9 | 42.50% | 52.50% | 60.00% |
| branch分子, 长度7 | 100.00% | 90.00% | 93.33% |
| branch分子, 长度8 | 90.00% | 86.67% | 86.67% |
| branch分子, 长度9 | 90.00% | 76.67% | 83.33% |
  
  

(ring分子, 长度9) 提升不大, branch分子和基础分子的正确率下降, 怀疑是因为只用了ring训练, 导致一些误解. 分析错误: 

```
36   /opt/huangyan/mol-ai/images/ring_selfies/length_9/56.png     {
"selfies": "[C][C][O][C][C][Ring1][Ring2][N][O]"}        CC1OCC1NO     {
"selfies": "[O][C][C][N][O][C][C][Ring1][Ring2]"}  OCCN1OCC1    0.093750   
mismatch
镜像, ring op左移一位, 4/8对调 (环内外对调)
[C][C][O][N][Ring1][Ring2][C][C][O]

8    /opt/huangyan/mol-ai/images/ring_selfies/length_9/23.png     {
"selfies": "[N][C][N][N][C][Ring1][Ring2][O][N]"}        NC1NNC1ON     {
"selfies": "[N][C][N][N][Ring1][Ring2][C][O][N]"}  N1CNN1CON    0.142857   
mismatch
ring op左移一位

24   /opt/huangyan/mol-ai/images/ring_selfies/length_9/93.png     {
"selfies": "[N][N][N][C][C][C][Ring1][Ring1][N]"}        NNNC1CC1N     {
"selfies": "[N][C][C][C][N][Ring1][Ring1][N][N]"}  NCC1CN1NN    0.178571   
mismatch
镜像, ring op左移一位
[N][N][N][C][C][Ring1][Ring1][C][N]

27  /opt/huangyan/mol-ai/images/ring_selfies/length_9/193.png     {
"selfies": "[C][C][N][Ring1][Ring1][C][C][O][O]"}        C1CN1CCOO     {
"selfies": "[O][C][C][N][C][C][Ring1][Ring1][O]"}  OCCN1CC1O    0.185185   
mismatch
镜像, 1-9轮换
[O][C][C][N][Ring1][Ring1][C][C][O]

38   /opt/huangyan/mol-ai/images/ring_selfies/length_9/61.png     {
"selfies": "[C][N][N][O][C][Ring1][Ring2][O][O]"}        CN1NOC1OO     {
"selfies": "[O][N][C][N][Ring1][Ring2][C][O][O]"}  O1NCN1COO    0.187500   
mismatch
起点3, ring op左移一位, 3/4对调 (环内对调)
[C][N][O][N][Ring1][Ring2][C][O][O]

0   /opt/huangyan/mol-ai/images/ring_selfies/length_9/180.png  {
"selfies": "[O][N][O][C][C][Ring1][=Branch1][O][O]"}        O1NOCC1OO  {
"selfies": "[N][O][O][O][C][C][Ring1][=Branch1][O]"}  N1OOOCC1O    0.193548   
mismatch
1-9轮换, 1/3对调 (环内对调)

21  /opt/huangyan/mol-ai/images/ring_selfies/length_9/176.png     {
"selfies": "[C][N][O][C][Ring1][Ring2][N][O][N]"}        C1NOC1NON     {
"selfies": "[O][C][N][N][Ring1][Ring2][C][O][N]"}  O1CNN1CON    0.193548   
mismatch
复杂差异

6   /opt/huangyan/mol-ai/images/ring_selfies/length_9/138.png     {
"selfies": "[N][N][O][C][N][Ring1][Ring2][C][N]"}        NN1OCN1CN     {
"selfies": "[O][C][N][N][Ring1][Ring2][C][N][N]"}  O1CNN1CNN    0.206897   
mismatch
起点3, 1-9轮换, 2/3对调 (环内对调)
[N][C][O][N][Ring1][Ring2][C][N][N]

3   /opt/huangyan/mol-ai/images/ring_selfies/length_9/196.png     {
"selfies": "[C][C][C][C][N][C][N][Ring1][Ring2]"}        CCCC1NCN1     {
"selfies": "[N][C][C][C][Ring1][Ring2][C][N][C]"}  N1CCC1CNC    0.214286   
mismatch
镜像, 2/5对调(环内外对调)
[C][N][C][C][C][C][N][Ring1][Ring2]

22   /opt/huangyan/mol-ai/images/ring_selfies/length_9/11.png     {
"selfies": "[C][N][C][N][C][Ring1][Ring1][N][N]"}        CNC1NC1NN     {
"selfies": "[C][N][C][N][N][C][Ring1][Ring1][N]"}  CNCN1NC1N    0.214286   
mismatch
ring op右移一位, 5/6对调(环内对调)

16   /opt/huangyan/mol-ai/images/ring_selfies/length_9/41.png     {
"selfies": "[C][C][O][O][C][Ring1][Ring2][C][O]"}        CC1OOC1CO     {
"selfies": "[O][C][C][C][O][O][C][Ring1][Ring2]"}  OCCC1OOC1    0.222222   
mismatch
1-9轮换2次

12  /opt/huangyan/mol-ai/images/ring_selfies/length_9/114.png   {
"selfies": "[N][N][N][C][C][O][N][Ring1][Branch1]"}        NNN1CCON1   {
"selfies": "[N][O][C][N][C][Ring1][Branch1][N][N]"}  N1OCNC1NN    0.225806   
mismatch
镜像, 3/4对调(环内对调)
[N][N][C][N][C][O][N][Ring1][Branch1]

14  /opt/huangyan/mol-ai/images/ring_selfies/length_9/108.png  {
"selfies": "[C][N][C][O][C][C][Ring1][=Branch1][C]"}        C1NCOCC1C  {
"selfies": "[C][C][O][C][N][C][C][Ring1][=Branch1]"}  CC1OCNCC1    0.285714   
mismatch
镜像, 1-6轮换(环内轮换)
[C][C][N][C][O][C][Ring1][=Branch1][C]

26   /opt/huangyan/mol-ai/images/ring_selfies/length_9/87.png  {
"selfies": "[C][O][C][O][C][C][Ring1][=Branch1][N]"}        C1OCOCC1N  {
"selfies": "[N][C][O][C][O][C][C][Ring1][=Branch1]"}  NC1OCOCC1    0.304348   
mismatch
1-9轮换

9    /opt/huangyan/mol-ai/images/ring_selfies/length_9/44.png   {
"selfies": "[N][N][N][N][C][C][O][Ring1][Branch1]"}        NNN1NCCO1   {
"selfies": "[C][C][N][O][N][Ring1][Branch1][N][N]"}  C1CNON1NN    0.310345   
mismatch
镜像, 4/6对调(环内对调)
[N][N][N][C][C][N][O][Ring1][Branch1]

19  /opt/huangyan/mol-ai/images/ring_selfies/length_9/189.png  {
"selfies": "[C][N][C][N][C][N][Ring1][=Branch1][O]"}        C1NCNCN1O  {
"selfies": "[O][N][C][N][C][C][N][Ring1][=Branch1]"}  ON1CNCCN1    0.318182   
mismatch
镜像, 1/2对调(环内对调)
[N][C][C][N][C][N][Ring1][=Branch1][O]

1   /opt/huangyan/mol-ai/images/ring_selfies/length_9/119.png     {
"selfies": "[C][N][C][N][C][N][Ring1][Ring1][C]"}        CNCN1CN1C     {
"selfies": "[C][N][C][N][N][C][Ring1][Ring1][C]"}  CNCN1NC1C    0.320000   
mismatch
5/6对调(环内对调)

11  /opt/huangyan/mol-ai/images/ring_selfies/length_9/124.png     {
"selfies": "[O][N][C][C][N][O][Ring1][Ring1][N]"}         ONCC1NO1     {
"selfies": "[O][N][C][C][O][N][Ring1][Ring1][N]"}  ONCC1ON1N    0.360000   
mismatch
5/6对调(环内对调)

28    /opt/huangyan/mol-ai/images/ring_selfies/length_9/8.png   {
"selfies": "[N][N][N][C][C][N][Ring1][Branch1][O]"}        NN1NCCN1O   {
"selfies": "[O][N][N][C][C][N][Ring1][Branch1][N]"}  ON1NCCN1N    0.478261   
mismatch
1/9对调(环内外对调)

``` 

  

看上去: add_ring_op_from_random_selfies_only_basic_atom.regular_smiles 解决了ring长度的错误, 但并不能解决ring op的位置的错误, 查看训练数据:

  - 在长度9的训练数据中, 确实没有出现[Ring1][C]
    - 主要原因是 [Ring1][C] 被解释成双键, 比如: [C][C][N][C][C][O][Ring1][C][O] 被解析为 [C][C][N][C][C][=O]

  - Ring的起点也都在selfies的后半段, 起点变化很少
    - ring会被规范化到末尾, 比如: [O][C][N][N][Ring1][Ring2][C][N][N] 被解析为 [N][N][C][N][N][C][O][Ring1][Ring2]

  

增加add_ring_op_from_random_selfies_only_basic_atom.regular_smiles 数据量一倍, 训练效果: 

|  | 基线 | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=3) (LR=5e-5) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 100.00% | 100.00% | 100.00% |
| 基础原子, 长度8 | 100.00% | 100.00% | 100.00% |
| 基础原子, 长度9 | 100.00% | 90.00% | 100.00% |
| ring分子, 长度7 | 90.00% | 90.00% | 87.50% |
| ring分子, 长度8 | 72.50% | 82.50% | 82.50% |
| ring分子, 长度9 | 42.50% | 57.50% | 60.00% |
| branch分子, 长度7 | 100.00% | 83.33% | 90.00% |
| branch分子, 长度8 | 90.00% | 90.00% | 90.00% |
| branch分子, 长度9 | 90.00% | 76.67% | 80.00% |
  
(ring分子, 长度8)准确率有提升, 但(ring分子, 长度9)未见提升.

  

分析(ring分子, 长度9)的错误: 

```
21  /opt/huangyan/mol-ai/images/ring_selfies/length_9/176.png     {
"selfies": "[N][C][O][N][N][Ring1][Ring2][C][O]"}        NC1ONN1CO     {
"selfies": "[O][C][N][N][Ring1][Ring2][C][O][N]"}  O1CNN1CON    0.156250   
mismatch
1-9轮换, 1/2对调

6   /opt/huangyan/mol-ai/images/ring_selfies/length_9/138.png     {
"selfies": "[N][N][C][N][O][N][Ring1][Ring2][C]"}        NNC1NON1C     {
"selfies": "[O][C][N][N][Ring1][Ring2][C][N][N]"}  O1CNN1CNN    0.193548   
mismatch
镜像, ring op右移一位, 5/7轮换
[N][N][C][N][N][C][O][Ring1][Ring2]

7    /opt/huangyan/mol-ai/images/ring_selfies/length_9/26.png     {
"selfies": "[C][C][N][N][Ring1][Ring1][N][O][O]"}        CC1NN1NOO     {
"selfies": "[C][N][N][N][Ring1][Ring1][C][O][O]"}  CN1NN1COO    0.206897   
mismatch
2/7轮换

3   /opt/huangyan/mol-ai/images/ring_selfies/length_9/196.png     {
"selfies": "[C][C][C][N][Ring1][Ring2][C][N][C]"}        C1CCN1CNC     {
"selfies": "[N][C][C][C][Ring1][Ring2][C][N][C]"}  N1CCC1CNC    0.214286   
mismatch
1/4轮换

38   /opt/huangyan/mol-ai/images/ring_selfies/length_9/61.png     {
"selfies": "[C][N][O][N][C][Ring1][Ring2][O][O]"}        CN1ONC1OO     {
"selfies": "[O][N][C][N][Ring1][Ring2][C][O][O]"}  O1NCN1COO    0.225806   
mismatch
起点3, ring op左移一位
[C][N][O][N][Ring1][Ring2][C][O][O]

28    /opt/huangyan/mol-ai/images/ring_selfies/length_9/8.png   {
"selfies": "[N][N][N][C][N][C][Ring1][Branch1][O]"}        NN1NCNC1O   {
"selfies": "[O][N][N][C][C][N][Ring1][Branch1][N]"}  ON1NCCN1N    0.250000   
mismatch
镜像, 3/7对调
[N][N][C][C][N][N][Ring1][Branch1][O]

0   /opt/huangyan/mol-ai/images/ring_selfies/length_9/180.png  {
"selfies": "[O][O][O][N][C][C][Ring1][=Branch1][O]"}        O1OONCC1O  {
"selfies": "[N][O][O][O][C][C][Ring1][=Branch1][O]"}  N1OOOCC1O    0.285714   
mismatch
1/4对调

14  /opt/huangyan/mol-ai/images/ring_selfies/length_9/108.png  {
"selfies": "[C][C][N][C][O][C][C][Ring1][=Branch1]"}        CC1NCOCC1  {
"selfies": "[C][C][O][C][N][C][C][Ring1][=Branch1]"}  CC1OCNCC1    0.285714   
mismatch
3/5对调

9    /opt/huangyan/mol-ai/images/ring_selfies/length_9/44.png   {
"selfies": "[N][N][N][C][N][O][C][Ring1][Branch1]"}        NNN1CNOC1   {
"selfies": "[C][C][N][O][N][Ring1][Branch1][N][N]"}  C1CNON1NN    0.310345   
mismatch
镜像, 5/7轮换
[N][N][N][C][C][N][O][Ring1][Branch1]

19  /opt/huangyan/mol-ai/images/ring_selfies/length_9/189.png  {
"selfies": "[C][N][C][N][C][N][Ring1][=Branch1][O]"}        C1NCNCN1O  {
"selfies": "[O][N][C][N][C][C][N][Ring1][=Branch1]"}  ON1CNCCN1    0.318182   
mismatch
镜像, 1/2对调
[N][C][C][N][C][N][Ring1][=Branch1][O]

1   /opt/huangyan/mol-ai/images/ring_selfies/length_9/119.png     {
"selfies": "[C][N][C][N][C][N][Ring1][Ring1][C]"}        CNCN1CN1C     {
"selfies": "[C][N][C][N][N][C][Ring1][Ring1][C]"}  CNCN1NC1C    0.320000   
mismatch
5/6对调

12  /opt/huangyan/mol-ai/images/ring_selfies/length_9/114.png   {
"selfies": "[N][N][C][N][N][C][O][Ring1][Branch1]"}        NNC1NNCO1   {
"selfies": "[N][O][C][N][C][Ring1][Branch1][N][N]"}  N1OCNC1NN    0.321429   
mismatch
镜像, 5-7轮换
[N][N][C][N][C][O][N][Ring1][Branch1]

33  /opt/huangyan/mol-ai/images/ring_selfies/length_9/106.png     {
"selfies": "[C][N][C][C][N][Ring1][Ring1][C][N]"}        CNC1CN1CN     {
"selfies": "[N][N][C][C][Ring1][Ring1][C][N][C]"}  NN1CC1CNC    0.321429   
mismatch
镜像, ring op右移, 5/9对调
[C][N][C][C][C][N][Ring1][Ring1][N]

18   /opt/huangyan/mol-ai/images/ring_selfies/length_9/14.png   {
"selfies": "[N][C][N][C][N][C][O][Ring1][Branch1]"}        NCN1CNCO1   {
"selfies": "[N][C][N][O][N][C][C][Ring1][Branch1]"}  NCN1ONCC1    0.357143   
mismatch
4/7对调

11  /opt/huangyan/mol-ai/images/ring_selfies/length_9/124.png     {
"selfies": "[O][N][C][C][N][O][Ring1][Ring1][N]"}         ONCC1NO1     {
"selfies": "[O][N][C][C][O][N][Ring1][Ring1][N]"}  ONCC1ON1N    0.360000   
mismatch
5/6对调

16   /opt/huangyan/mol-ai/images/ring_selfies/length_9/41.png     {
"selfies": "[O][C][C][C][O][C][O][Ring1][Ring2]"}        OCCC1OCO1     {
"selfies": "[O][C][C][C][O][O][C][Ring1][Ring2]"}  OCCC1OOC1    0.375000   
mismatch
6/7对调

37  /opt/huangyan/mol-ai/images/ring_selfies/length_9/167.png     {
"selfies": "[C][O][C][C][C][N][C][Ring1][Ring2]"}        COCC1CNC1     {
"selfies": "[C][O][C][C][C][C][N][Ring1][Ring2]"}  COCC1CCN1    0.400000   
mismatch
6/7对调
``` 

看起来: 增加add_ring_op_from_random_selfies_only_basic_atom.regular_smiles 数据量一倍, 能改善ring op的位置错误

将之前的数据混合在一起训练: 

训练数据: branch_selfies.regular_smiles*1 + random_selfies_only_basic_atom.regular_smiles*1 + add_branch_op_from_random_selfies_only_basic_atom.regular_smiles*1 + add_ring_op_from_random_selfies_only_basic_atom.regular_smiles*2

|  | 基线 | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=3) (LR=5e-5) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 100.00% | 100.00% | 0.00% |
| 基础原子, 长度8 | 100.00% | 80.00% | 85.00% |
| 基础原子, 长度9 | 100.00% | 90.00% | 100.00% |
| ring分子, 长度7 | 90.00% | 90.00% | 90.00% |
| ring分子, 长度8 | 72.50% | 77.50% | 72.50% |
| ring分子, 长度9 | 42.50% | 62.50% | 60.00% |
| branch分子, 长度7 | 100.00% | 100.00% | 100.00% |
| branch分子, 长度8 | 90.00% | 96.67% | 90.00% |
| branch分子, 长度9 | 90.00% | 80.00% | 83.33% |
  
与混合前没有差异.

先增加原子对调的case:

训练数据: add_ring_op_from_random_selfies_only_basic_atom.regular_smiles*2 + exchange_two_atoms_from.add_ring_op_from_random_selfies_only_basic_atom.regular_smiles*2

训练效果: 

|  | 基线 | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=3) (LR=5e-5) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 100.00% | 100.00% | 100.00% |
| 基础原子, 长度8 | 100.00% | 100.00% | 100.00% |
| 基础原子, 长度9 | 100.00% | 95.00% | 95.00% |
| ring分子, 长度7 | 90.00% | 100.00% | 97.50% |
| ring分子, 长度8 | 72.50% | 80.00% | 77.50% |
| ring分子, 长度9 | 42.50% | 77.50% | 80.00% |
| branch分子, 长度7 | 100.00% | 93.33% | 96.67% |
| branch分子, 长度8 | 90.00% | 96.67% | 93.33% |
| branch分子, 长度9 | 90.00% | 86.67% | 80.00% |
  
(ring分子, 长度9) 仍不达标, 分析错误:

```
3   /opt/huangyan/mol-ai/images/ring_selfies/length_9/196.png   {
"selfies": "[C][N][C][C][C][N][Ring1][Branch1][C]"}        CN1CCCN1C     {
"selfies": "[N][C][C][C][Ring1][Ring2][C][N][C]"}  N1CCC1CNC    0.076923   
mismatch
镜像, ring op右移, ring长度错误, 6/7对调
[C][N][C][C][C][C][N][Ring1][Ring2]

21  /opt/huangyan/mol-ai/images/ring_selfies/length_9/176.png     {
"selfies": "[N][C][O][C][N][N][Ring1][Ring2][O]"}          NCOCNNO     {
"selfies": "[O][C][N][N][Ring1][Ring2][C][O][N]"}  O1CNN1CON    0.093750   
mismatch
整体轮换两次, 后三原子对调

6   /opt/huangyan/mol-ai/images/ring_selfies/length_9/138.png     {
"selfies": "[N][N][C][N][C][Ring1][Ring2][O][N]"}        NN1CNC1ON     {
"selfies": "[O][C][N][N][Ring1][Ring2][C][N][N]"}  O1CNN1CNN    0.125000   
mismatch
镜像, ring op右移两位, 三原子对调
[N][N][C][N][N][C][O][Ring1][Ring2]

8    /opt/huangyan/mol-ai/images/ring_selfies/length_9/23.png   {
"selfies": "[N][C][O][N][C][N][Ring1][Branch1][N]"}        NC1ONCN1N     {
"selfies": "[N][C][N][N][Ring1][Ring2][C][O][N]"}  N1CNN1CON    0.129032   
mismatch
复杂变化

33  /opt/huangyan/mol-ai/images/ring_selfies/length_9/106.png     {
"selfies": "[C][N][C][C][C][Ring1][Ring1][N][N]"}        CNC1CC1NN     {
"selfies": "[N][N][C][C][Ring1][Ring1][C][N][C]"}  NN1CC1CNC    0.214286   
mismatch
镜像, ring op右移一位
[C][N][C][C][C][N][Ring1][Ring1][N]

38   /opt/huangyan/mol-ai/images/ring_selfies/length_9/61.png   {
"selfies": "[O][C][O][N][C][N][Ring1][Branch1][O]"}        OC1ONCN1O     {
"selfies": "[O][N][C][N][Ring1][Ring2][C][O][O]"}  O1NCN1COO    0.233333   
mismatch
ring op右移一位, ring长度错误, 2/3对调
[O][O][C][N][C][N][O][Ring1][Ring2]

30   /opt/huangyan/mol-ai/images/ring_selfies/length_9/91.png  {
"selfies": "[C][C][O][C][N][C][N][Ring1][=Branch1]"}        CC1OCNCN1  {
"selfies": "[C][C][O][C][N][N][C][Ring1][=Branch1]"}  CC1OCNNC1    0.296296   
mismatch
6/7对调

0   /opt/huangyan/mol-ai/images/ring_selfies/length_9/180.png  {
"selfies": "[O][C][N][O][O][C][O][Ring1][=Branch1]"}        OC1NOOCO1  {
"selfies": "[N][O][O][O][C][C][Ring1][=Branch1][O]"}  N1OOOCC1O    0.333333   
mismatch
镜像, 3/6/7对调
[O][C][C][O][O][O][N][Ring1][=Branch1]

18   /opt/huangyan/mol-ai/images/ring_selfies/length_9/14.png   {
"selfies": "[N][C][N][C][N][C][O][Ring1][Branch1]"}        NCN1CNCO1   {
"selfies": "[N][C][N][O][N][C][C][Ring1][Branch1]"}  NCN1ONCC1    0.357143   
mismatch
4/7对调
``` 

只使用 exchange_two_atoms_from.add_ring_op_from_random_selfies_only_basic_atom.regular_smiles*4 (提高了数据量):

训练效果: 

|  | 基线 | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=3) (LR=5e-5) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 100.00% | 100.00% | 100.00% |
| 基础原子, 长度8 | 100.00% | 100.00% | 100.00% |
| 基础原子, 长度9 | 100.00% | 100.00% | 100.00% |
| ring分子, 长度7 | 90.00% | 97.50% | 100.00% |
| ring分子, 长度8 | 72.50% | 85.00% | 87.50% |
| ring分子, 长度9 | 42.50% | 82.50% | 80.00% |
| branch分子, 长度7 | 100.00% | 100.00% | 96.67% |
| branch分子, 长度8 | 90.00% | 90.00% | 96.67% |
| branch分子, 长度9 | 90.00% | 80.00% | 76.67% |
  
比起混合数据训练, 单一训练的效果略微提升, 错误分析: 

```
8    /opt/huangyan/mol-ai/images/ring_selfies/length_9/23.png     {
"selfies": "[N][O][C][N][N][C][Ring1][Ring2][N]"}        NOC1NNC1N     {
"selfies": "[N][C][N][N][Ring1][Ring2][C][O][N]"}  N1CNN1CON    0.142857   
mismatch
镜像, ring op后移一位
[N][O][C][N][N][C][N][Ring1][Ring2]

36   /opt/huangyan/mol-ai/images/ring_selfies/length_9/56.png     {
"selfies": "[O][C][C][C][N][O][C][Ring1][Ring2]"}        OCCC1NOC1     {
"selfies": "[O][C][C][N][O][C][C][Ring1][Ring2]"}  OCCN1OCC1    0.206897   
mismatch
4-6轮换 (环内)

3   /opt/huangyan/mol-ai/images/ring_selfies/length_9/196.png     {
"selfies": "[C][N][C][C][C][N][Ring1][Ring2][C]"}        CNC1CCN1C     {
"selfies": "[N][C][C][C][Ring1][Ring2][C][N][C]"}  N1CCC1CNC    0.206897   
mismatch
镜像, ring op右移一位, 6/7对调 (环内)
[C][N][C][C][C][C][N][Ring1][Ring2]

19  /opt/huangyan/mol-ai/images/ring_selfies/length_9/189.png  {
"selfies": "[O][N][C][C][C][N][N][Ring1][=Branch1]"}        ON1CCCNN1  {
"selfies": "[O][N][C][N][C][C][N][Ring1][=Branch1]"}  ON1CNCCN1    0.269231   
mismatch
4-6轮换(环内)

38   /opt/huangyan/mol-ai/images/ring_selfies/length_9/61.png   {
"selfies": "[O][C][N][C][N][O][O][Ring1][Branch1]"}        OCN1CNOO1     {
"selfies": "[O][N][C][N][Ring1][Ring2][C][O][O]"}  O1NCN1COO    0.310345   
mismatch
镜像, ring长度错误, 1-6轮换(环内外)
[O][O][C][N][C][N][O][Ring1][Ring2]

6   /opt/huangyan/mol-ai/images/ring_selfies/length_9/138.png     {
"selfies": "[N][N][C][N][C][O][N][Ring1][Ring2]"}        NNCN1CON1     {
"selfies": "[O][C][N][N][Ring1][Ring2][C][N][N]"}  O1CNN1CNN    0.423077   
mismatch
镜像, 5-7轮换(环内)
[N][N][C][N][N][C][O][Ring1][Ring2]

21  /opt/huangyan/mol-ai/images/ring_selfies/length_9/176.png     {
"selfies": "[N][O][C][N][N][O][C][Ring1][Ring2]"}        NOCN1NOC1     {
"selfies": "[O][C][N][N][Ring1][Ring2][C][O][N]"}  O1CNN1CON    0.423077   
mismatch
镜像, 6-7对调(环内)
[N][O][C][N][N][C][O][Ring1][Ring2]

``` 

增强 "轮换(环内)":

训练数据: exchange_two_atoms_from.add_ring_op_from_random_selfies_only_basic_atom.regular_smiles*4 + rotate_selfies_substring_from.add_ring_op_from_random_selfies_only_basic_atom.regular_smiles*1

|  | 基线 | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=3) (LR=5e-5) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 100.00% | 100.00% | 100.00% |
| 基础原子, 长度8 | 100.00% | 100.00% | 100.00% |
| 基础原子, 长度9 | 100.00% | 95.00% | 95.00% |
| ring分子, 长度7 | 90.00% | 100.00% | 97.50% |
| ring分子, 长度8 | 72.50% | 80.00% | 85.00% |
| ring分子, 长度9 | 42.50% | 80.00% | 80.00% |
| branch分子, 长度7 | 100.00% | 100.00% | 96.67% |
| branch分子, 长度8 | 90.00% | 90.00% | 93.33% |
| branch分子, 长度9 | 90.00% | 83.33% | 76.67% |
  
未见提升, 查看错误分布: 

```
38   /opt/huangyan/mol-ai/images/ring_selfies/length_9/61.png     {
"selfies": "[O][O][C][N][N][Ring1][Ring2][C][O]"}          OOCNNCO     {
"selfies": "[O][N][C][N][Ring1][Ring2][C][O][O]"}  O1NCN1COO    0.172414   
mismatch
镜像, ring op后面的原子向前插入
[O][O][C][N][C][N][O][Ring1][Ring2]

21  /opt/huangyan/mol-ai/images/ring_selfies/length_9/176.png     {
"selfies": "[N][O][C][N][N][Ring1][Ring2][C][O]"}          NOCNNCO     {
"selfies": "[O][C][N][N][Ring1][Ring2][C][O][N]"}  O1CNN1CON    0.172414   
mismatch
镜像, ring op后面的原子向前插入
[N][O][C][N][N][C][O][Ring1][Ring2]

8    /opt/huangyan/mol-ai/images/ring_selfies/length_9/23.png   {
"selfies": "[N][O][C][N][N][C][Ring1][Branch1][N]"}          NOCNNCN     {
"selfies": "[N][C][N][N][Ring1][Ring2][C][O][N]"}  N1CNN1CON    0.185185   
mismatch
镜像, ring op后面的原子向前插入, ring长度错误
[N][O][C][N][N][C][N][Ring1][Ring2]

6   /opt/huangyan/mol-ai/images/ring_selfies/length_9/138.png   {
"selfies": "[N][N][C][N][N][O][C][Ring1][Branch1]"}        NNC1NNOC1     {
"selfies": "[O][C][N][N][Ring1][Ring2][C][N][N]"}  O1CNN1CNN    0.200000   
mismatch
镜像, ring长度错误, 6/7对调
[N][N][C][N][N][C][O][Ring1][Ring2]

2   /opt/huangyan/mol-ai/images/ring_selfies/length_9/152.png     {
"selfies": "[C][N][O][C][N][Ring1][Ring1][C][O]"}          CNOCNCO     {
"selfies": "[O][N][C][C][Ring1][Ring1][O][N][C]"}  ON1CC1ONC    0.206897   
mismatch
镜像, ring op后面的原子向前插入
[C][N][O][C][C][N][Ring1][Ring1][O]

3   /opt/huangyan/mol-ai/images/ring_selfies/length_9/196.png     {
"selfies": "[C][N][C][C][C][N][Ring1][Ring2][C]"}        CNC1CCN1C     {
"selfies": "[N][C][C][C][Ring1][Ring2][C][N][C]"}  N1CCC1CNC    0.206897   
mismatch
镜像, ring op后面的原子向前插入
[C][N][C][C][C][C][N][Ring1][Ring2]

5    /opt/huangyan/mol-ai/images/ring_selfies/length_9/34.png     {
"selfies": "[C][N][C][N][N][Ring1][Ring1][N][O]"}        CNC1NN1NO     {
"selfies": "[O][N][N][N][Ring1][Ring1][C][N][C]"}  ON1NN1CNC    0.206897   
mismatch
镜像, ring op后面的原子向前插入
[C][N][C][N][NH1][N][Ring1][Ring1][O]

17   /opt/huangyan/mol-ai/images/ring_selfies/length_9/35.png  {
"selfies": "[O][N][C][C][Ring1][Ring1][O][O][O][O]"}       ON1CC1OOOO     {
"selfies": "[O][N][C][C][Ring1][Ring1][O][O][O]"}  ON1CC1OOO    0.700000   
mismatch
多一个原子

 
``` 

增加"rotate_selfies_substring_from.add_ring_op_from_random_selfies_only_basic_atom.regular_smiles" 到4倍, 仍不见提升

增加"ring_selfies.regular_smiles*4", 不论单独训练还是混合训练, 都不见提升

commit: 74fd373f2654d1fe8ae468ea4dc340c0c2f94964

还是尝试将 "ring op后面的原子向前插入" 作为增强数据, 只使用 move_atoms_after_ring_backward_from.ring_selfies.regular_smiles*4 训练: 

|  | 基线 | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=3) (LR=5e-5) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 100.00% | 100.00% | 100.00% |
| 基础原子, 长度8 | 100.00% | 95.00% | 75.00% |
| 基础原子, 长度9 | 100.00% | 100.00% | 95.00% |
| ring分子, 长度7 | 90.00% | 80.00% | 77.50% |
| ring分子, 长度8 | 72.50% | 82.50% | 82.50% |
| ring分子, 长度9 | 42.50% | 70.00% | 62.50% |
| branch分子, 长度7 | 100.00% | 76.67% | 83.33% |
| branch分子, 长度8 | 90.00% | 80.00% | 86.67% |
| branch分子, 长度9 | 90.00% | 60.00% | 60.00% |
  
混合数据训练: exchange_two_atoms_from.add_ring_op_from_random_selfies_only_basic_atom.regular_smiles*4 + rotate_selfies_substring_from.add_ring_op_from_random_selfies_only_basic_atom.regular_smiles*4 + move_atoms_after_ring_backward_from.ring_selfies.regular_smiles*4

|  | 基线 | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=3) (LR=5e-5) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 100.00% | 100.00% | 100.00% |
| 基础原子, 长度8 | 100.00% | 100.00% | 100.00% |
| 基础原子, 长度9 | 100.00% | 90.00% | 90.00% |
| ring分子, 长度7 | 90.00% | 100.00% | 100.00% |
| ring分子, 长度8 | 72.50% | 82.50% | 90.00% |
| ring分子, 长度9 | 42.50% | 85.00% | 87.50% |
| branch分子, 长度7 | 100.00% | 93.33% | 80.00% |
| branch分子, 长度8 | 90.00% | 86.67% | 83.33% |
| branch分子, 长度9 | 90.00% | 76.67% | 60.00% |
  
查看错误: 

```
30   /opt/huangyan/mol-ai/images/ring_selfies/length_8/20.png  {
"selfies": "[C][C][C][N][N][N][Ring1][=Branch1]"}         C1CCNNN1  {
"selfies": "[N][C][C][N][C][N][Ring1][=Branch1]"}  N1CCNCN1    0.222222   
mismatch
1/5对调(环内)

0   /opt/huangyan/mol-ai/images/ring_selfies/length_8/188.png  {
"selfies": "[C][N][C][O][O][N][Ring1][=Branch1]"}         C1NCOON1  {
"selfies": "[O][N][C][N][O][C][Ring1][=Branch1]"}  O1NCNOC1    0.238095   
mismatch
起点3, 3/4对调(环内)
[C][N][O][C][O][N][Ring1][=Branch1]

22  /opt/huangyan/mol-ai/images/ring_selfies/length_8/117.png     {
"selfies": "[N][N][C][N][N][Ring1][Ring2][N]"}         NN1CNN1N     {
"selfies": "[C][N][N][N][Ring1][Ring2][N][N]"}  C1NNN1NN    0.285714   
mismatch
ring op 左移一位, 1/3对调(环内)

36   /opt/huangyan/mol-ai/images/ring_selfies/length_8/80.png   {
"selfies": "[C][N][N][N][N][C][Ring1][Branch1]"}         CN1NNNC1     {
"selfies": "[C][N][N][N][Ring1][Ring2][N][C]"}  C1NNN1NC    0.304348   
mismatch
ring长度错误, ring op 左移两位

3   /opt/huangyan/mol-ai/images/ring_selfies/length_9/196.png     {
"selfies": "[C][N][C][C][C][N][Ring1][Ring2][C]"}        CNC1CCN1C     {
"selfies": "[N][C][C][C][Ring1][Ring2][C][N][C]"}  N1CCC1CNC    0.206897   
mismatch
ring op 左移两位, 1/2对调(环内)

8    /opt/huangyan/mol-ai/images/ring_selfies/length_9/23.png     {
"selfies": "[N][O][C][N][C][N][Ring1][Ring2][N]"}        NOC1NCN1N     {
"selfies": "[N][C][N][N][Ring1][Ring2][C][O][N]"}  N1CNN1CON    0.206897   
mismatch
镜像, ring op右移一位, 5/6对调(环内)
[N][O][C][N][N][C][N][Ring1][Ring2]

38   /opt/huangyan/mol-ai/images/ring_selfies/length_9/61.png   {
"selfies": "[O][O][C][N][C][N][O][Ring1][Branch1]"}        OOC1NCNO1     {
"selfies": "[O][N][C][N][Ring1][Ring2][C][O][O]"}  O1NCN1COO    0.225806   
mismatch
镜像, ring长度错误
[O][O][C][N][C][N][O][Ring1][Ring2]

21  /opt/huangyan/mol-ai/images/ring_selfies/length_9/176.png   {
"selfies": "[N][O][C][N][N][C][O][Ring1][Branch1]"}        NOC1NNCO1     {
"selfies": "[O][C][N][N][Ring1][Ring2][C][O][N]"}  O1CNN1CON    0.233333   
mismatch
镜像, ring长度错误
[N][O][C][N][N][C][O][Ring1][Ring2]

6   /opt/huangyan/mol-ai/images/ring_selfies/length_9/138.png   {
"selfies": "[N][N][C][N][C][O][N][Ring1][Branch1]"}        NNC1NCON1     {
"selfies": "[O][C][N][N][Ring1][Ring2][C][N][N]"}  O1CNN1CNN    0.233333   
mismatch
镜像, ring长度错误, 5-7轮换
[N][N][C][N][N][C][O][Ring1][Ring2]

``` 

在步骤2的checkpoint之上, 增加add_ring_op_from_random_selfies_only_basic_atom.regular_smiles, 来增强模型对ring长度的认知:

训练数据: exchange_two_atoms_from.add_ring_op_from_random_selfies_only_basic_atom.regular_smiles*4 + rotate_selfies_substring_from.add_ring_op_from_random_selfies_only_basic_atom.regular_smiles*4 + move_atoms_after_ring_backward_from.ring_selfies.regular_smiles*4 + add_ring_op_from_random_selfies_only_basic_atom.regular_smiles*4

训练效果:

|  | 基线 | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=3) (LR=5e-5) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 100.00% | 100.00% | 100.00% |
| 基础原子, 长度8 | 100.00% | 95.00% | 95.00% |
| 基础原子, 长度9 | 100.00% | 95.00% | 95.00% |
| ring分子, 长度7 | 90.00% | 100.00% | 100.00% |
| ring分子, 长度8 | 72.50% | 90.00% | 82.50% |
| ring分子, 长度9 | 42.50% | 87.50% | 82.50% |
| branch分子, 长度7 | 100.00% | 76.67% | 83.33% |
| branch分子, 长度8 | 90.00% | 76.67% | 80.00% |
| branch分子, 长度9 | 90.00% | 56.67% | 73.33% |
  
分析ring分子错误: (出错的ring长度都是 [Ring1][Ring2])

```
39   /opt/huangyan/mol-ai/images/ring_selfies/length_8/36.png   {
"selfies": "[O][N][C][N][N][O][Ring1][Branch1]"}         ON1CNNO1     {
"selfies": "[O][N][C][N][N][O][Ring1][Ring2]"}  ONC1NNO1    0.148148   
mismatch
ring长度错误

1   /opt/huangyan/mol-ai/images/ring_selfies/length_8/108.png     {
"selfies": "[O][C][N][O][N][Ring1][Ring2][C]"}         OC1NON1C     {
"selfies": "[C][N][O][N][Ring1][Ring2][C][O]"}  C1NON1CO    0.192308   
mismatch
1-9轮换. 或者, 镜像, ring op右边的原子向左插入

22  /opt/huangyan/mol-ai/images/ring_selfies/length_8/117.png     {
"selfies": "[N][N][C][N][N][Ring1][Ring2][N]"}         NN1CNN1N     {
"selfies": "[C][N][N][N][Ring1][Ring2][N][N]"}  C1NNN1NN    0.285714   
mismatch
镜像, 1-9轮换. 或者, 镜像, ring op右边的原子向左插入
[N][N][N][C][N][N][Ring1][Ring2]

36   /opt/huangyan/mol-ai/images/ring_selfies/length_8/80.png   {
"selfies": "[C][N][C][N][N][N][Ring1][Branch1]"}         CN1CNNN1     {
"selfies": "[C][N][N][N][Ring1][Ring2][N][C]"}  C1NNN1NC    0.304348   
mismatch
ring长度错误, 1-9轮换*2, 原子交换

8    /opt/huangyan/mol-ai/images/ring_selfies/length_9/23.png     {
"selfies": "[N][C][O][C][N][N][Ring1][Ring2][N]"}          NCOCNNN     {
"selfies": "[N][C][N][N][Ring1][Ring2][C][O][N]"}  N1CNN1CON    0.100000   
mismatch
复杂变化
[N][O][C][N][N][C][N][Ring1][Ring2]

21  /opt/huangyan/mol-ai/images/ring_selfies/length_9/176.png     {
"selfies": "[N][O][C][N][N][C][Ring1][Ring2][O]"}        NOC1NNC1O     {
"selfies": "[O][C][N][N][Ring1][Ring2][C][O][N]"}  O1CNN1CON    0.133333   
mismatch
镜像, 镜像, ring op右边的原子向左插入
[N][O][C][N][N][C][O][Ring1][Ring2]

38   /opt/huangyan/mol-ai/images/ring_selfies/length_9/61.png     {
"selfies": "[O][O][C][N][N][C][Ring1][Ring2][O]"}        OOC1NNC1O     {
"selfies": "[O][N][C][N][Ring1][Ring2][C][O][O]"}  O1NCN1COO    0.133333   
mismatch
镜像, ring op右边的原子向左插入
[O][O][C][N][C][N][O][Ring1][Ring2]

3   /opt/huangyan/mol-ai/images/ring_selfies/length_9/196.png     {
"selfies": "[C][N][C][C][C][N][Ring1][Ring2][C]"}        CNC1CCN1C     {
"selfies": "[N][C][C][C][Ring1][Ring2][C][N][C]"}  N1CCC1CNC    0.206897   
mismatch
镜像, ring op右边的原子向左插入
[C][N][C][C][C][C][N][Ring1][Ring2]

6   /opt/huangyan/mol-ai/images/ring_selfies/length_9/138.png     {
"selfies": "[N][N][C][N][C][O][N][Ring1][Ring2]"}        NNCN1CON1     {
"selfies": "[O][C][N][N][Ring1][Ring2][C][N][N]"}  O1CNN1CNN    0.423077   
mismatch
镜像, 5-7轮换
[N][N][C][N][N][C][O][Ring1][Ring2]

``` 

分析branch分子的错误率上升: 很多分子的branch被识别成了ring结构

检查训练数据中的 "[Ring1][Ring2]", 发现训练数据中是有不少"[Ring1][Ring2]"数据, 在(move_atoms_after_ring_backward_from.ring_selfies.regular_smiles) 中也有, 但其数据并不体现 "变化前后". 

对move_atoms_after_ring_backward_from.ring_selfies.regular_smiles做出调整:

  1. 让数据集有序
  2. 对于某一个selfies, 进行递归, 也就是将ring后的原子不断前移, 直到ring后没有原子
  3. 对于某一个selfies, 扩大其一次变化的数量为3个

只使用move_atoms_after_ring_backward_from.ring_selfies.regular_smiles*4进行训练: 

|  | 基线 | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=3) (LR=5e-5) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 100.00% | 100.00% | 100.00% |
| 基础原子, 长度8 | 100.00% | 100.00% | 100.00% |
| 基础原子, 长度9 | 100.00% | 100.00% | 100.00% |
| ring分子, 长度7 | 90.00% | 87.50% | 85.00% |
| ring分子, 长度8 | 72.50% | 85.00% | 87.50% |
| ring分子, 长度9 | 42.50% | 72.50% | 67.50% |
| branch分子, 长度7 | 100.00% | 76.67% | 76.67% |
| branch分子, 长度8 | 90.00% | 83.33% | 90.00% |
| branch分子, 长度9 | 90.00% | 66.67% | 63.33% |
  
(branch分子, 长度9)下降很厉害, 分析原因: 仍然是将很多branch结构解析成了ring结构

  

使用数据: move_atoms_after_ring_backward_from.ring_selfies.regular_smiles*4 + rotate_selfies_substring_from.add_ring_op_from_random_selfies_only_basic_atom.regular_smiles*4

|  | 基线 | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=3) (LR=5e-5) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 100.00% | 100.00% | 100.00% |
| 基础原子, 长度8 | 100.00% | 95.00% | 100.00% |
| 基础原子, 长度9 | 100.00% | 100.00% | 100.00% |
| ring分子, 长度7 | 90.00% | 85.00% | 95.00% |
| ring分子, 长度8 | 72.50% | 87.50% | 80.00% |
| ring分子, 长度9 | 42.50% | 65.00% | 62.50% |
| branch分子, 长度7 | 100.00% | 86.67% | 86.67% |
| branch分子, 长度8 | 90.00% | 86.67% | 86.67% |
| branch分子, 长度9 | 90.00% | 70.00% | 80.00% |
  
  

使用数据: move_atoms_after_ring_backward_from.ring_selfies.regular_smiles*4 + rotate_selfies_substring_from.add_ring_op_from_random_selfies_only_basic_atom.regular_smiles*4 + exchange_two_atoms_from.add_ring_op_from_random_selfies_only_basic_atom.regular_smiles*4

|  | 基线 | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=3) (LR=5e-5) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 100.00% | 100.00% | 100.00% |
| 基础原子, 长度8 | 100.00% | 100.00% | 100.00% |
| 基础原子, 长度9 | 100.00% | 100.00% | 95.00% |
| ring分子, 长度7 | 90.00% | 100.00% | 100.00% |
| ring分子, 长度8 | 72.50% | 90.00% | 90.00% |
| ring分子, 长度9 | 42.50% | 85.00% | 87.50% |
| branch分子, 长度7 | 100.00% | 90.00% | 93.33% |
| branch分子, 长度8 | 90.00% | 90.00% | 86.67% |
| branch分子, 长度9 | 90.00% | 63.33% | 66.67% |
  
使用数据: move_atoms_after_ring_backward_from.ring_selfies.regular_smiles*4 + rotate_selfies_substring_from.add_ring_op_from_random_selfies_only_basic_atom.regular_smiles*4 + exchange_two_atoms_from.add_ring_op_from_random_selfies_only_basic_atom.regular_smiles*4 + add_ring_op_from_random_selfies_only_basic_atom.regular_smiles*4

|  | 基线 | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=3) (LR=5e-5) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 100.00% | 100.00% | 100.00% |
| 基础原子, 长度8 | 100.00% | 100.00% | 95.00% |
| 基础原子, 长度9 | 100.00% | 90.00% | 85.00% |
| ring分子, 长度7 | 90.00% | 97.50% | 100.00% |
| ring分子, 长度8 | 72.50% | 80.00% | 85.00% |
| ring分子, 长度9 | 42.50% | 80.00% | 87.50% |
| branch分子, 长度7 | 100.00% | 86.67% | 80.00% |
| branch分子, 长度8 | 90.00% | 90.00% | 86.67% |
| branch分子, 长度9 | 90.00% | 76.67% | 73.33% |
  
  

使用数据: move_atoms_after_ring_backward_from.ring_selfies.regular_smiles*4 + exchange_two_atoms_from.add_ring_op_from_random_selfies_only_basic_atom.regular_smiles*4

|  | 基线 | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=3) (LR=5e-5) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 100.00% | 100.00% | 100.00% |
| 基础原子, 长度8 | 100.00% | 100.00% | 100.00% |
| 基础原子, 长度9 | 100.00% | 95.00% | 95.00% |
| ring分子, 长度7 | 90.00% | 97.50% | 100.00% |
| ring分子, 长度8 | 72.50% | 90.00% | 85.00% |
| ring分子, 长度9 | 42.50% | 85.00% | 77.50% |
| branch分子, 长度7 | 100.00% | 93.33% | 96.67% |
| branch分子, 长度8 | 90.00% | 90.00% | 80.00% |
| branch分子, 长度9 | 90.00% | 80.00% | 70.00% |
  
分析错误: 

```
18   /opt/huangyan/mol-ai/images/ring_selfies/length_8/70.png     {
"selfies": "[O][C][N][O][C][Ring1][Ring2][O]"}         OC1NOC1O     {
"selfies": "[O][C][O][N][Ring1][Ring2][C][O]"}  O1CON1CO    0.083333   
mismatch
镜像, ring op右移一位
[O][C][N][O][C][O][Ring1][Ring2]

39   /opt/huangyan/mol-ai/images/ring_selfies/length_8/36.png   {
"selfies": "[O][N][C][O][N][N][Ring1][Branch1]"}         ON1CONN1     {
"selfies": "[O][N][C][N][N][O][Ring1][Ring2]"}  ONC1NNO1    0.148148   
mismatch
ring长度错误, 4/6对调

13    /opt/huangyan/mol-ai/images/ring_selfies/length_8/1.png     {
"selfies": "[C][N][C][C][Ring1][Ring1][C][O]"}         CN1CC1CO     {
"selfies": "[O][C][C][C][Ring1][Ring1][N][C]"}  OC1CC1NC    0.160000   
mismatch
镜像, ring op右移一位
[C][N][C][C][C][Ring1][Ring1][O]

36   /opt/huangyan/mol-ai/images/ring_selfies/length_8/80.png   {
"selfies": "[C][N][N][N][N][C][Ring1][Branch1]"}         CN1NNNC1     {
"selfies": "[C][N][N][N][Ring1][Ring2][N][C]"}  C1NNN1NC    0.304348   
mismatch
镜像, ring长度错误, 4/6对调
[C][N][N][C][N][N][Ring1][Ring2]

8    /opt/huangyan/mol-ai/images/ring_selfies/length_9/23.png     {
"selfies": "[N][O][C][N][N][C][Ring1][Ring2][N]"}        NOC1NNC1N     {
"selfies": "[N][C][N][N][Ring1][Ring2][C][O][N]"}  N1CNN1CON    0.142857   
mismatch
镜像, ring op右移一位
[N][O][C][N][N][C][N][Ring1][Ring2]

7    /opt/huangyan/mol-ai/images/ring_selfies/length_9/26.png     {
"selfies": "[C][N][N][C][Ring1][Ring1][O][O][N]"}        CN1NC1OON     {
"selfies": "[C][N][N][N][Ring1][Ring1][C][O][O]"}  CN1NN1COO    0.172414   
mismatch
4/7/9对调
38   /opt/huangyan/mol-ai/images/ring_selfies/length_9/61.png   {
"selfies": "[O][O][C][N][N][O][C][Ring1][Branch1]"}        OOC1NNOC1     {
"selfies": "[O][N][C][N][Ring1][Ring2][C][O][O]"}  O1NCN1COO    0.187500   
mismatch
镜像, ring长度错误, 5/6/7对调
[O][O][C][N][C][N][O][Ring1][Ring2]

3   /opt/huangyan/mol-ai/images/ring_selfies/length_9/196.png     {
"selfies": "[C][N][C][C][C][N][Ring1][Ring2][C]"}        CNC1CCN1C     {
"selfies": "[N][C][C][C][Ring1][Ring2][C][N][C]"}  N1CCC1CNC    0.206897   
mismatch
ring op右移一位, 6/7对调
[C][N][C][C][C][C][N][Ring1][Ring2]

33  /opt/huangyan/mol-ai/images/ring_selfies/length_9/106.png     {
"selfies": "[C][N][C][C][N][Ring1][Ring1][C][N]"}        CNC1CN1CN     {
"selfies": "[N][N][C][C][Ring1][Ring1][C][N][C]"}  NN1CC1CNC    0.321429   
mismatch
镜像, ring op右移一位, 5/6对调
[C][N][C][C][C][N][Ring1][Ring1][N]

26   /opt/huangyan/mol-ai/images/ring_selfies/length_9/87.png  {
"selfies": "[N][C][O][C][C][O][C][Ring1][=Branch1]"}        NC1OCCOC1  {
"selfies": "[N][C][O][C][O][C][C][Ring1][=Branch1]"}  NC1OCOCC1    0.375000   
mismatch
5/6对调
``` 

错误分布: 

  - 3个长度错误 ([Ring1][Ring2]识别成[Ring1][Branch1])
  - 5个 "ring op右移一位" (移到最后两位)
  - 7个对调, 有6个跟位置6相关 (环内)

检查训练数据:

  - 两部分训练数据中, 没有体现ring op移动和不同长度的ring op, 这两部分在 add_ring_op_from_random_selfies_only_basic_atom.regular_smiles 都有. 
  - 对调的数据:
    - 需要增加原始数据, 否则体现不出"变化"
    - 需要让越短的对调提高概率, 越长的对调降低概率

使用调整后的 对调数据, 训练: move_atoms_after_ring_backward_from.ring_selfies.regular_smiles*4 + exchange_two_atoms_from.add_ring_op_from_random_selfies_only_basic_atom.regular_smiles*4

训练效果: 

|  | 基线 | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=3) (LR=5e-5) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 100.00% | 100.00% | 100.00% |
| 基础原子, 长度8 | 100.00% | 100.00% | 100.00% |
| 基础原子, 长度9 | 100.00% | 100.00% | 100.00% |
| ring分子, 长度7 | 90.00% | 100.00% | 97.50% |
| ring分子, 长度8 | 72.50% | 87.50% | 87.50% |
| ring分子, 长度9 | 42.50% | 85.00% | 87.50% |
| branch分子, 长度7 | 100.00% | 83.33% | 86.67% |
| branch分子, 长度8 | 90.00% | 93.33% | 90.00% |
| branch分子, 长度9 | 90.00% | 76.67% | 80.00% |
  
checkpoint: /opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v943-20250220-204354/checkpoint-360

增加: add_ring_op_from_random_selfies_only_basic_atom.regular_smiles

训练数据: move_atoms_after_ring_backward_from.ring_selfies.regular_smiles*4 + exchange_two_atoms_from.add_ring_op_from_random_selfies_only_basic_atom.regular_smiles*4 + add_ring_op_from_random_selfies_only_basic_atom.regular_smiles*4

训练效果: 

|  | 基线 | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=3) (LR=5e-5) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 100.00% | 100.00% | 100.00% |
| 基础原子, 长度8 | 100.00% | 95.00% | 100.00% |
| 基础原子, 长度9 | 100.00% | 85.00% | 85.00% |
| ring分子, 长度7 | 90.00% | 100.00% | 100.00% |
| ring分子, 长度8 | 72.50% | 85.00% | 82.50% |
| ring分子, 长度9 | 42.50% | 82.50% | 80.00% |
| branch分子, 长度7 | 100.00% | 80.00% | 73.33% |
| branch分子, 长度8 | 90.00% | 83.33% | 86.67% |
| branch分子, 长度9 | 90.00% | 63.33% | 63.33% |
  
以上两种训练, 都没有看到明显提升, 分析错误类型: 

```
34   /opt/huangyan/mol-ai/images/ring_selfies/length_8/90.png   {
"selfies": "[C][C][C][C][N][Ring1][Branch1][O]"}         C1CCCN1O   {
"selfies": "[O][C][C][N][C][Ring1][Branch1][C]"}  O1CCNC1C    0.040000   
mismatch
1/8对调, 4/5对调

27   /opt/huangyan/mol-ai/images/ring_selfies/length_8/69.png   {
"selfies": "[O][C][C][C][N][Ring1][Branch1][O]"}         O1CCCN1O   {
"selfies": "[O][C][C][N][C][Ring1][Branch1][O]"}  O1CCNC1O    0.148148   
mismatch
4/5对调

38  /opt/huangyan/mol-ai/images/ring_selfies/length_8/102.png     {
"selfies": "[N][C][C][O][C][Ring1][Ring2][O]"}         NC1COC1O     {
"selfies": "[O][C][O][C][Ring1][Ring2][C][N]"}  O1COC1CN    0.217391   
mismatch
镜像, ring op右移一位
[N][C][C][O][C][O][Ring1][Ring2]

22  /opt/huangyan/mol-ai/images/ring_selfies/length_8/117.png     {
"selfies": "[N][N][C][N][N][Ring1][Ring2][N]"}         NN1CNN1N     {
"selfies": "[C][N][N][N][Ring1][Ring2][N][N]"}  C1NNN1NN    0.285714   
mismatch
镜像, 1-9轮换
[N][N][N][C][N][N][Ring1][Ring2]

36   /opt/huangyan/mol-ai/images/ring_selfies/length_8/80.png   {
"selfies": "[C][N][C][N][N][N][Ring1][Branch1]"}         CN1CNNN1     {
"selfies": "[C][N][N][N][Ring1][Ring2][N][C]"}  C1NNN1NC    0.304348   
mismatch
镜像, ring长度错误, 3/4对调
[C][N][N][C][N][N][Ring1][Ring2]

38   /opt/huangyan/mol-ai/images/ring_selfies/length_9/61.png   {
"selfies": "[O][O][C][N][N][O][C][Ring1][Branch1]"}        OOC1NNOC1     {
"selfies": "[O][N][C][N][Ring1][Ring2][C][O][O]"}  O1NCN1COO    0.187500   
mismatch
镜像, ring长度错误, 5-7轮换
[O][O][C][N][C][N][O][Ring1][Ring2]

3   /opt/huangyan/mol-ai/images/ring_selfies/length_9/196.png     {
"selfies": "[C][N][C][C][C][N][Ring1][Ring2][C]"}        CNC1CCN1C     {
"selfies": "[N][C][C][C][Ring1][Ring2][C][N][C]"}  N1CCC1CNC    0.206897   
mismatch
ring op左移两位, 1/2对调

0   /opt/huangyan/mol-ai/images/ring_selfies/length_9/180.png  {
"selfies": "[O][C][N][O][O][C][O][Ring1][=Branch1]"}        OC1NOOCO1  {
"selfies": "[N][O][O][O][C][C][Ring1][=Branch1][O]"}  N1OOOCC1O    0.333333   
mismatch
镜像, 3/6/7对调
[O][C][C][O][O][O][N][Ring1][=Branch1]

18   /opt/huangyan/mol-ai/images/ring_selfies/length_9/14.png   {
"selfies": "[N][C][N][C][N][C][O][Ring1][Branch1]"}        NCN1CNCO1   {
"selfies": "[N][C][N][O][N][C][C][Ring1][Branch1]"}  NCN1ONCC1    0.357143   
mismatch
4/7对调

8    /opt/huangyan/mol-ai/images/ring_selfies/length_9/23.png     {
"selfies": "[N][O][C][N][C][N][N][Ring1][Ring2]"}        NOCN1CNN1     {
"selfies": "[N][C][N][N][Ring1][Ring2][C][O][N]"}  N1CNN1CON    0.400000   
mismatch
镜像, 5/6对调
[N][O][C][N][N][C][N][Ring1][Ring2]

6   /opt/huangyan/mol-ai/images/ring_selfies/length_9/138.png     {
"selfies": "[N][N][C][N][C][O][N][Ring1][Ring2]"}        NNCN1CON1     {
"selfies": "[O][C][N][N][Ring1][Ring2][C][N][N]"}  O1CNN1CNN    0.423077   
mismatch
镜像, 5-7轮换
[N][N][C][N][N][C][O][Ring1][Ring2] 
``` 

错误分布: 

  - 2个ring op平移
  - 2个ring长度错误
  - 6个只需要对调, 还有2个是其他错误+对调

在checkpoint(/opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v943-20250220-204354/checkpoint-360)上, 增加对调数据:

  - exchange_two_atoms_from.add_ring_op_from_random_selfies_only_basic_atom.regular_smiles*4

训练效果: 

|  | 基线 | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=3) (LR=5e-5) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 100.00% | 100.00% | 100.00% |
| 基础原子, 长度8 | 100.00% | 95.00% | 95.00% |
| 基础原子, 长度9 | 100.00% | 95.00% | 95.00% |
| ring分子, 长度7 | 90.00% | 97.50% | 97.50% |
| ring分子, 长度8 | 72.50% | 90.00% | 92.50% |
| ring分子, 长度9 | 42.50% | 90.00% | 87.50% |
| branch分子, 长度7 | 100.00% | 96.67% | 86.67% |
| branch分子, 长度8 | 90.00% | 93.33% | 86.67% |
| branch分子, 长度9 | 90.00% | 83.33% | 80.00% |
  
ring分子准确率达标, 分析错误分布: 

```
27   /opt/huangyan/mol-ai/images/ring_selfies/length_8/69.png   {
"selfies": "[O][C][C][C][N][Ring1][Branch1][O]"}         O1CCCN1O   {
"selfies": "[O][C][C][N][C][Ring1][Branch1][O]"}  O1CCNC1O    0.148148   
mismatch
4/5对调

34   /opt/huangyan/mol-ai/images/ring_selfies/length_8/90.png   {
"selfies": "[C][C][O][C][N][C][Ring1][Branch1]"}         CC1OCNC1   {
"selfies": "[O][C][C][N][C][Ring1][Branch1][C]"}  O1CCNC1C    0.280000   
mismatch
镜像, 3/5/6对调
[C][C][N][C][C][O][Ring1][Branch1]

22  /opt/huangyan/mol-ai/images/ring_selfies/length_8/117.png     {
"selfies": "[N][N][C][N][N][Ring1][Ring2][N]"}         NN1CNN1N     {
"selfies": "[C][N][N][N][Ring1][Ring2][N][N]"}  C1NNN1NN    0.285714   
mismatch
镜像, ring op右移, 3/4对调
[N][N][N][C][N][N][Ring1][Ring2]

36   /opt/huangyan/mol-ai/images/ring_selfies/length_8/80.png   {
"selfies": "[C][N][C][N][N][N][Ring1][Branch1]"}         CN1CNNN1     {
"selfies": "[C][N][N][N][Ring1][Ring2][N][C]"}  C1NNN1NC    0.304348   
mismatch
3/4对调, ring长度错误
[C][N][N][C][N][N][Ring1][Ring2]

8    /opt/huangyan/mol-ai/images/ring_selfies/length_9/23.png     {
"selfies": "[N][O][C][N][N][C][Ring1][Ring2][N]"}        NOC1NNC1N     {
"selfies": "[N][C][N][N][Ring1][Ring2][C][O][N]"}  N1CNN1CON    0.142857   
mismatch
镜像, ring op右移
[N][O][C][N][N][C][N][Ring1][Ring2]

3   /opt/huangyan/mol-ai/images/ring_selfies/length_9/196.png     {
"selfies": "[C][N][C][C][C][N][Ring1][Ring2][C]"}        CNC1CCN1C     {
"selfies": "[N][C][C][C][Ring1][Ring2][C][N][C]"}  N1CCC1CNC    0.206897   
mismatch
镜像, ring op右移, 6/7对调
[C][N][C][C][C][C][N][Ring1][Ring2]

6   /opt/huangyan/mol-ai/images/ring_selfies/length_9/138.png     {
"selfies": "[N][N][C][O][N][Ring1][Ring2][C][N]"}        NN1CON1CN     {
"selfies": "[O][C][N][N][Ring1][Ring2][C][N][N]"}  O1CNN1CNN    0.206897   
mismatch
复杂差异
[O][C][N][N][Ring1][Ring2][C][N][N]
[C][N][N][Branch1][Ring2][C][N][N][O][Ring1][#Branch1]
[N][C][O][N][Ring1][Ring2][C][N][N]
[N][Branch1][Ring2][C][N][N][N][C][O][Ring1][#Branch1]
[C][Branch1][Ring1][N][N][N][N][C][O][Ring1][Ring2]
[N][Branch1][C][N][C][N][N][C][O][Ring1][Ring2]
[N][N][C][N][N][C][O][Ring1][Ring2]

18   /opt/huangyan/mol-ai/images/ring_selfies/length_9/14.png   {
"selfies": "[N][C][N][O][C][C][N][Ring1][Branch1]"}        NCN1OCCN1   {
"selfies": "[N][C][N][O][N][C][C][Ring1][Branch1]"}  NCN1ONCC1    0.357143   
mismatch
5/7对调

``` 

对(branch分子, 长度9)错误进行分析: 

```
14    /opt/huangyan/mol-ai/images/branch_selfies/length_9/6.png  {
"selfies": "[N][C][O][N][O][N][Ring1][Ring1][C]"}        NCON1ON1C     {
"selfies": "[N][O][N][Branch1][Ring2][O][C][N][C]"}  NON(OCN)C    0.222222   
mismatch

18   /opt/huangyan/mol-ai/images/branch_selfies/length_9/48.png                {
"selfies": "[C][N][C][O][N][C][O]"}          CNCONCO         {
"selfies": "[C][N][O][C][N][Branch1][C][O][C]"}  CNOCN(O)C    0.259259   
mismatch

24   /opt/huangyan/mol-ai/images/branch_selfies/length_9/94.png  {
"selfies": "[N][O][N][O][N][N][Ring1][Ring1][O]"}          NONONNO     {
"selfies": "[N][O][N][Branch1][Ring1][O][N][N][O]"}  NON(ON)NO    0.260870   
mismatch

19   /opt/huangyan/mol-ai/images/branch_selfies/length_9/54.png                {
"selfies": "[N][C][O][O][N][C][O]"}          NCOONCO         {
"selfies": "[O][C][Branch1][C][N][O][N][C][O]"}  OC(N)ONCO    0.360000   
mismatch

5   /opt/huangyan/mol-ai/images/branch_selfies/length_9/104.png    {
"selfies": "[O][C][O][C][Branch1][C][O][C][O]"}        OCOC(O)CO     {
"selfies": "[O][C][O][C][Branch1][Ring1][O][O][C]"}  OCOC(OO)C    0.363636   
mismatch

``` 

  

训练数据: 

  - 阶段1: add_ring_op_from_random_selfies_only_basic_atom.regular_smiles*4
  - 阶段2: move_atoms_after_ring_backward_from.ring_selfies.regular_smiles*4 + exchange_two_atoms_from.add_ring_op_from_random_selfies_only_basic_atom.regular_smiles*4

效果下降

  

  

训练数据: move_atoms_after_ring_backward_from.ring_selfies.regular_smiles*8 + exchange_two_atoms_from.add_ring_op_from_random_selfies_only_basic_atom.regular_smiles*8

|  | 基线 | 步骤1 (epoch=3) (LR=1e-4) |
| --- | --- | --- |
| 基础原子, 长度7 | 100.00% | 100.00% |
| 基础原子, 长度8 | 100.00% | 100.00% |
| 基础原子, 长度9 | 100.00% | 95.00% |
| ring分子, 长度7 | 90.00% | 100.00% |
| ring分子, 长度8 | 72.50% | 90.00% |
| ring分子, 长度9 | 42.50% | 92.50% |
| branch分子, 长度7 | 100.00% | 93.33% |
| branch分子, 长度8 | 90.00% | 90.00% |
| branch分子, 长度9 | 90.00% | 80.00% |
  
  

分析错误: (还是几种在ring平移)

```
 39   /opt/huangyan/mol-ai/images/ring_selfies/length_8/36.png   {
"selfies": "[O][N][C][N][N][O][Ring1][Branch1]"}         ON1CNNO1     {
"selfies": "[O][N][C][N][N][O][Ring1][Ring2]"}  ONC1NNO1    0.148148   
mismatch
ring长度错误

27   /opt/huangyan/mol-ai/images/ring_selfies/length_8/69.png   {
"selfies": "[O][C][C][C][N][Ring1][Branch1][O]"}         O1CCCN1O   {
"selfies": "[O][C][C][N][C][Ring1][Branch1][O]"}  O1CCNC1O    0.148148   
mismatch
4/5对调

22  /opt/huangyan/mol-ai/images/ring_selfies/length_8/117.png     {
"selfies": "[N][N][C][N][N][Ring1][Ring2][N]"}         NN1CNN1N     {
"selfies": "[C][N][N][N][Ring1][Ring2][N][N]"}  C1NNN1NN    0.285714   
mismatch
起点3, ring op右移一位
[N][N][C][N][Ring1][Ring2][N][N]

36   /opt/huangyan/mol-ai/images/ring_selfies/length_8/80.png   {
"selfies": "[C][N][N][N][N][C][Ring1][Branch1]"}         CN1NNNC1     {
"selfies": "[C][N][N][N][Ring1][Ring2][N][C]"}  C1NNN1NC    0.304348   
mismatch
ring op右移两位, ring长度错误

8    /opt/huangyan/mol-ai/images/ring_selfies/length_9/23.png     {
"selfies": "[N][O][C][N][N][C][Ring1][Ring2][N]"}        NOC1NNC1N     {
"selfies": "[N][C][N][N][Ring1][Ring2][C][O][N]"}  N1CNN1CON    0.142857   
mismatch
镜像, ring op右移
[N][O][C][N][N][C][N][Ring1][Ring2]

3   /opt/huangyan/mol-ai/images/ring_selfies/length_9/196.png     {
"selfies": "[C][N][C][C][C][N][Ring1][Ring2][C]"}        CNC1CCN1C     {
"selfies": "[N][C][C][C][Ring1][Ring2][C][N][C]"}  N1CCC1CNC    0.206897   
mismatch
镜像, ring op右移, 6/7对调
[C][N][C][C][C][C][N][Ring1][Ring2]

38   /opt/huangyan/mol-ai/images/ring_selfies/length_9/61.png   {
"selfies": "[O][C][N][C][N][O][O][Ring1][Branch1]"}        OCN1CNOO1     {
"selfies": "[O][N][C][N][Ring1][Ring2][C][O][O]"}  O1NCN1COO    0.310345   
mismatch
镜像, ring长度错误, 1-7轮换
[O][O][C][N][C][N][O][Ring1][Ring2]
``` 

  

checkpoint: /opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v968-20250221-114901/checkpoint-576

在这个checkpoint上, 加入对branch的修订: 

  - add_branch_op_from_random_selfies_only_basic_atom.regular_smiles*1 + add_ring_op_from_random_selfies_only_basic_atom.regular_smiles*1

|  | 基线 | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=3) (LR=5e-5) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 100.00% | 100.00% | 100.00% |
| 基础原子, 长度8 | 100.00% | 95.00% | 100.00% |
| 基础原子, 长度9 | 100.00% | 90.00% | 95.00% |
| ring分子, 长度7 | 90.00% | 97.50% | 100.00% |
| ring分子, 长度8 | 72.50% | 75.00% | 77.50% |
| ring分子, 长度9 | 42.50% | 77.50% | 82.50% |
| branch分子, 长度7 | 100.00% | 100.00% | 100.00% |
| branch分子, 长度8 | 90.00% | 96.67% | 93.33% |
| branch分子, 长度9 | 90.00% | 90.00% | 93.33% |
  
ring分子的准确率会急速下降, 根据错误分析, 看上去会逆转之前的训练

```
 18   /opt/huangyan/mol-ai/images/ring_selfies/length_8/70.png     {
"selfies": "[O][C][O][C][O][N][Ring1][Ring2]"}           OCOCON     {
"selfies": "[O][C][O][N][Ring1][Ring2][C][O]"}  O1CON1CO    0.120000   
mismatch
镜像, 3-6镜像
[O][C][N][O][C][O][Ring1][Ring2]

39   /opt/huangyan/mol-ai/images/ring_selfies/length_8/36.png   {
"selfies": "[O][N][C][O][N][N][Ring1][Branch1]"}         ON1CONN1     {
"selfies": "[O][N][C][N][N][O][Ring1][Ring2]"}  ONC1NNO1    0.148148   
mismatch
4/6对调, ring长度错误

13    /opt/huangyan/mol-ai/images/ring_selfies/length_8/1.png     {
"selfies": "[C][N][C][C][Ring1][Ring1][C][O]"}         CN1CC1CO     {
"selfies": "[O][C][C][C][Ring1][Ring1][N][C]"}  OC1CC1NC    0.160000   
mismatch
镜像, ring op右移
[C][N][C][C][C][Ring1][Ring1][O]

22  /opt/huangyan/mol-ai/images/ring_selfies/length_8/117.png     {
"selfies": "[N][C][N][N][N][Ring1][Ring2][N]"}         NC1NNN1N     {
"selfies": "[C][N][N][N][Ring1][Ring2][N][N]"}  C1NNN1NN    0.166667   
mismatch
1-8轮换

30   /opt/huangyan/mol-ai/images/ring_selfies/length_8/20.png  {
"selfies": "[C][C][C][N][N][N][Ring1][=Branch1]"}         C1CCNNN1  {
"selfies": "[N][C][C][N][C][N][Ring1][=Branch1]"}  N1CCNCN1    0.222222   
mismatch
1/5对调

0   /opt/huangyan/mol-ai/images/ring_selfies/length_8/188.png  {
"selfies": "[C][N][N][O][C][O][Ring1][=Branch1]"}         C1NNOCO1  {
"selfies": "[O][N][C][N][O][C][Ring1][=Branch1]"}  O1NCNOC1    0.238095   
mismatch
镜像, 2/4/5对调
[C][O][N][C][N][O][Ring1][=Branch1]

31  /opt/huangyan/mol-ai/images/ring_selfies/length_8/133.png     {
"selfies": "[C][N][C][O][O][Ring1][Ring2][O]"}          CN1COO1     {
"selfies": "[O][C][O][O][N][Ring1][Ring2][C]"}  OC1OON1C    0.263158   
mismatch
镜像, 3/5对调
[C][N][O][O][C][Ring1][Ring2][O]

27   /opt/huangyan/mol-ai/images/ring_selfies/length_8/69.png   {
"selfies": "[O][C][C][O][C][N][Ring1][Branch1]"}         OC1COCN1   {
"selfies": "[O][C][C][N][C][Ring1][Branch1][O]"}  O1CCNC1O    0.280000   
mismatch
镜像, 3/6对调
[O][C][N][C][C][O][Ring1][Branch1]

21  /opt/huangyan/mol-ai/images/ring_selfies/length_8/162.png  {
"selfies": "[C][O][C][O][C][O][Ring1][=Branch1]"}         C1OCOCO1  {
"selfies": "[C][O][C][O][O][C][Ring1][=Branch1]"}  C1OCOOC1    0.285714   
mismatch
4/5对调

36   /opt/huangyan/mol-ai/images/ring_selfies/length_8/80.png   {
"selfies": "[C][N][N][N][N][C][Ring1][Branch1]"}         CN1NNNC1     {
"selfies": "[C][N][N][N][Ring1][Ring2][N][C]"}  C1NNN1NC    0.304348   
mismatch
镜像4/6对调, ring长度错误
[C][N][N][C][N][N][Ring1][Ring2]

``` 

在第一轮的训练数据上, 加入对branch的修订: 

  - move_atoms_after_ring_backward_from.ring_selfies.regular_smiles*8 + exchange_two_atoms_from.add_ring_op_from_random_selfies_only_basic_atom.regular_smiles*8 + add_branch_op_from_random_selfies_only_basic_atom.regular_smiles*1 + add_ring_op_from_random_selfies_only_basic_atom.regular_smiles*1  
  

|  | 基线 | 步骤1 (epoch=3) (LR=1e-4) |
| --- | --- | --- |
| 基础原子, 长度7 | 100.00% | 100.00% |
| 基础原子, 长度8 | 100.00% | 95.00% |
| 基础原子, 长度9 | 100.00% | 95.00% |
| ring分子, 长度7 | 90.00% | 100.00% |
| ring分子, 长度8 | 72.50% | 87.50% |
| ring分子, 长度9 | 42.50% | 82.50% |
| branch分子, 长度7 | 100.00% | 96.67% |
| branch分子, 长度8 | 90.00% | 96.67% |
| branch分子, 长度9 | 90.00% | 100.00% |
  
训练数据: move_atoms_after_ring_backward_from.ring_selfies.regular_smiles*4 + exchange_two_atoms_from.add_ring_op_from_random_selfies_only_basic_atom.regular_smiles*4 + add_branch_op_from_random_selfies_only_basic_atom.regular_smiles*4

|  | 基线 | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=3) (LR=5e-5) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 100.00% | 100.00% | 100.00% |
| 基础原子, 长度8 | 100.00% | 95.00% | 100.00% |
| 基础原子, 长度9 | 100.00% | 90.00% | 100.00% |
| ring分子, 长度7 | 90.00% | 100.00% | 100.00% |
| ring分子, 长度8 | 72.50% | 87.50% | 87.50% |
| ring分子, 长度9 | 42.50% | 87.50% | 85.00% |
| branch分子, 长度7 | 100.00% | 100.00% | 100.00% |
| branch分子, 长度8 | 90.00% | 93.33% | 96.67% |
| branch分子, 长度9 | 90.00% | 83.33% | 90.00% |
  
除了扩大数据, 想不出其他办法

commit: f55e1b3a599c9f16889f76f3c1c18d14d726435f

# 总结这一批的训练

  - 第一个checkpoint: 课程训练, 直连
    - random_selfies_only_basic_atom + mirror_selfies_from_random_selfies_only_basic_atom + rotate_selfies_substring_from_random_selfies_only_basic_atom, 长度 3-4

    - random_selfies_only_basic_atom + mirror_selfies_from_random_selfies_only_basic_atom + rotate_selfies_substring_from_random_selfies_only_basic_atom, 长度 3-5

    - random_selfies_only_basic_atom + mirror_selfies_from_random_selfies_only_basic_atom + rotate_selfies_substring_from_random_selfies_only_basic_atom, 长度 3-6

    - random_selfies_only_basic_atom + mirror_selfies_from_random_selfies_only_basic_atom + rotate_selfies_substring_from_random_selfies_only_basic_atom, 长度 3-7

  - 第二个checkpoint: 课程训练, ring
    - ring_selfies + random_selfies_only_basic_atom + rotate_selfies_substring_from_ring_selfies, 长度3-6

    - ring_selfies + random_selfies_only_basic_atom + rotate_selfies_substring_from_ring_selfies, 长度3-7

  - 第三个checkpoint: 课程训练, branch
    - ring_selfies + random_selfies_only_basic_atom + branch_selfies + rotate_selfies_substring_from_branch_selfies + add_branch_op_from_random_selfies_only_basic_atom + randomize_start_point_from_branch_selfies, 长度3-6

    - ring_selfies + random_selfies_only_basic_atom + branch_selfies + rotate_selfies_substring_from_branch_selfies + add_branch_op_from_random_selfies_only_basic_atom + randomize_start_point_from_branch_selfies, 长度3-7

  - 第四个checkpoint: 扩展长度8
    - random_selfies_only_basic_atom + mirror_selfies_from_random_selfies_only_basic_atom + rotate_selfies_substring_from_random_selfies_only_basic_atom + ring_selfies + various_ring_pos + rotate_ring_atoms + rotate_selfies_substring_from_ring_selfies + branch_selfies + rotate_selfies_substring_from_branch_selfies + add_branch_op_from_random_selfies_only_basic_atom + branch_selfies_by_diff_start_point_from_random_selfies_only_basic_atom, 长度3-8
  - 第五个checkpoint: 扩展长度8
    - branch_selfies.regular_smiles + random_selfies_only_basic_atom + various_ring_pos, 长度3-8

  - 第六个checkpoint: 扩展长度9
    - branch_selfies.regular_smiles + random_selfies_only_basic_atom.regular_smiles + add_branch_op_from_random_selfies_only_basic_atom.regular_smiles, 长度3-9

  - 第七个checkpoint: 扩展长度9
    - move_atoms_after_ring_backward_from.ring_selfies.regular_smiles + exchange_two_atoms_from.add_ring_op_from_random_selfies_only_basic_atom.regular_smiles, 长度7-9

有效的变形措施: 

  - 表达式规范化, 图形规范化 (让模型对于一张图, 只输出唯一的表达式)
  - 镜像 (选取最后一个原子作为起点)
  - 选取某一个原子作为起点
  - 对局部子串进行轮换 (将第一个或最后一个原子, 插入到其他位置)
  - 在表达式中添加branch/ring操作符
  - 调整branch/ring操作符的位置
  - 调整branch/ring操作符的长度
  - 将branch/ring操作符后的原子往前调整
  - 选取两个原子互换位置
