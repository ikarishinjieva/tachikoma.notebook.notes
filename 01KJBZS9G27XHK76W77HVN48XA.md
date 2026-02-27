---
title: 20250326 - 阅读论文*: The Ultra-Scale Playbook: Training LLMs on GPU Clusters Ultra-Scale Playbook：在 GPU 集群上训练LLMs
confluence_page_id: 3801279
created_at: 2025-03-26T07:05:11+00:00
updated_at: 2025-04-09T08:44:44+00:00
---

# 评论: 非常有用

# 主要内容

分析了大模型使用的内存的分布, 以及减少内存使用的各种类方法

逐章节进行分析

## 章节1: First Steps: Training on one GPU (包括内存使用的概念介绍, 和简单的内存优化手段)

缩写: 

  - bs: batch size: 批大小
  - bst = Batch Size Token: 批Token数
    - bst = bs * seq

在训练神经网络模型时，需要将以下几项存储在内存中：

  - 模型权重
  - 模型梯度
  - 优化器状态
  - 计算梯度所需的激活值

训练过程中的内存使用图:

![image2025-3-26 14:31:16.png](/assets/01KJBZS9G27XHK76W77HVN48XA/image2025-3-26%2014%3A31%3A16.png)

  - 第一步比较特别: torch的内存分配器会做准备工作, 不详述
  - 一般步骤: 
    - 向前传递: 激活值会迅速增加
    - 向后传递: 梯度会累积，随着向后传递的传播，用于计算梯度的存储激活值会逐渐清除 

内存的组成: 模型参数 + 梯度 + 优化器 + 激活值

### 状态使用的内存 - (模型参数 + 梯度 + 优化器)使用的内存

![image2025-3-26 14:47:25.png](/assets/01KJBZS9G27XHK76W77HVN48XA/image2025-3-26%2014%3A47%3A25.png)

对于FP32精度, 每个参数需要4字节:

  - 参数内存 = 4*N
  - 梯度内存 = 4*N
  - 优化器需要存储动量和方差 = (4+4) * N

(对于混合精度训练, 内存公式参考原文, 不详述)

### 激活值(Activations) 使用的内存

激活值（Activation Values） 是神经网络中每一层的输出结果。它们是前向传播过程中由输入数据经过模型的各层计算后生成的中间结果，并在反向传播中用于计算梯度. 激活值使用的内存 取决于 模型的输入

(公式不详述)

对于短序列（或类似的小批量），激活几乎可以忽略不计，但从大约 2-4k 个令牌开始，它们会占用大量内存，而参数、梯度和优化器状态的使用（我们将在后面讨论）大致独立于序列长度和批量大小。

### 对于激活值占用内存的优化 - 激活值重新计算

思路: 在前向传播中, 仅记录关键层(checkpoint)的输出结果作为激活值; 在后向传播中, 从检查点再计算得到需要的激活值

FlashAttention已经使用了重新计算技术

### 对梯度内存的优化 - 梯度累计

思路: 将我们的 batch 分成多个微 batch。我们将在每个微批处理上依次执行向前和向后传递，计算梯度，顾名思义，在执行优化器步骤之前，将所有微批处理的梯度相加

缩写: 

  - mbs: 微批处理大小
  - gbs: 全局批处理大小

梯度累计公式: 

![image2025-3-26 15:3:47.png](/assets/01KJBZS9G27XHK76W77HVN48XA/image2025-3-26%2015%3A3%3A47.png)

# 内存优化手段 - 数据并行

思路: 想法是在多个 GPU 上复制模型（我们称之为副本的 “模型实例”），并为每个 GPU 并行运行不同的微批量数据向前和向后传递，因此得名数据并行.

在不同的GPU上, 不同微批次会产生不同的梯度, 梯度需要平均, 就需要在GPU间进行梯度交换, 这个过程称为"all-reduce", 发生在向后传递期间

![image2025-3-26 16:18:11.png](/assets/01KJBZS9G27XHK76W77HVN48XA/image2025-3-26%2016%3A18%3A11.png)

对以上过程中, GPU的空闲浪费进行优化

### 第一次优化：将梯度同步与向后传递重叠

每计算完一部分梯度, 就将其传播出去, 让计算和传播尽量并行

(原文中, 对于如何划分梯度的"部分", 没看懂. 不妨碍理解思路)

![image2025-3-26 16:30:54.png](/assets/01KJBZS9G27XHK76W77HVN48XA/image2025-3-26%2016%3A30%3A54.png)

### 第二次优化：分桶梯度

思路: 将梯度进行"批量"传输, 以减少传输的次数, 降低延迟:

![image2025-3-26 16:35:55.png](/assets/01KJBZS9G27XHK76W77HVN48XA/image2025-3-26%2016%3A35%3A55.png)

### 第三次优化：与梯度累积相互作用

梯度累积的工作原理是在使用 optimizer.step（） 更新参数之前执行多次向前和向后传递。

每次向后传递后都会自动触发 all-reduce . 优化方向: 将多次all-reduce批处理成一次 (在梯度累积的最后一步之后的单个 reduce 会产生相同的效果)

### 数据并行对批处理大小的影响

![image2025-3-26 16:41:0.png](/assets/01KJBZS9G27XHK76W77HVN48XA/image2025-3-26%2016%3A41%3A0.png)

对于固定的gbs, 可以增加数据并行实例的数量, 降低梯度累计步骤, 来加快训练速度

### 额外

数据并行的实例数越多, 实例间的通讯成本越高, 会不断吃掉数据并行的优势

### 下一步: 如何让一个微批次(mbs)放入一个GPU的显存中: ZeRO (Zero Redundancy Optimizer)

以上结论, 假设mbs可以放入一个GPU的显存中, 但实际上不一定:

![image2025-3-26 16:52:20.png](/assets/01KJBZS9G27XHK76W77HVN48XA/image2025-3-26%2016%3A52%3A20.png)

优化器状态、梯度和参数在每个 DP(数据并行) 实例上的简单复制会引入大量的内存冗余. 

ZeRO 通过在数据并行维度上对优化器状态、梯度和参数进行分区来消除内存冗余，同时仍然允许使用完整的参数集进行计算。ZeRO有三个优化阶段: 

  - ZeRO-1: optimizer state partitioning  
ZeRO-1：优化器状态分区
  - ZeRO-2: optimizer state + gradient partitioning  
ZeRO-2：优化器状态 + 梯度分配
  - ZeRO-3 (also called FSDP for “Fully-Sharded Data Parallelism”): optimizer state + gradient + parameter partitioning  
ZeRO-3（也称为 FSDP，即“全分片数据并行”）：优化器状态 + 梯度 + 参数分区

### 分析内存占用

对于Adam优化器, 使用混合精度训练, 内存的使用情况: 

![image2025-3-27 13:27:25.png](/assets/01KJBZS9G27XHK76W77HVN48XA/image2025-3-27%2013%3A27%3A25.png)

ZeRO三种级别的对应的内存占用示意图: 

![image2025-3-27 13:29:26.png](/assets/01KJBZS9G27XHK76W77HVN48XA/image2025-3-27%2013%3A29%3A26.png)

### ZeRO-1：优化器分片

思路: 让优化器的参数分片到各个GPU中. 

运算步骤: 

  1. 前向传播, 没有变化
  2. 后向传播, 没有变化
  3. Reduce-scatter: 将DP(数据分片) 后向传播的各梯度传播到各个GPU上. (在ZeRO-1中, 其就是all-reduce)
  4. 每个GPU上的优化器分片, 计算自己负责的那部分参数
  5. All-gather: 将各个GPU上的参数传播出去, 合并成完整的模型参数
  6. 进行下一轮前向传播

![image2025-3-27 15:17:53.png](/assets/01KJBZS9G27XHK76W77HVN48XA/image2025-3-27%2015%3A17%3A53.png)

### ZeRO-2：梯度分片

思路: 在优化器分片中, 其计算的只是梯度的一部分, 所以可以将梯度也分片 (在上面reduce-scatter步骤, 只分发有用的梯度到某GPU)

![image2025-3-27 15:35:21.png](/assets/01KJBZS9G27XHK76W77HVN48XA/image2025-3-27%2015%3A35%3A21.png)

### ZeRO-3：参数分片

思路: 将模型参数按层进行分片, 每层的参数都是按需加载: 

  - 前向传播: 
    - ![image](https://nanotron-ultrascale-playbook.static.hf.space/assets/images/dp_zero3_fwd.svg)
  - 后向传播: 
    - ![image](https://nanotron-ultrascale-playbook.static.hf.space/assets/images/dp_zero3_bwd.svg)

其调度流举例如下: 

![image](https://nanotron-ultrascale-playbook.static.hf.space/assets/images/dp_zero3_overlap.svg)

调度流说明: 

  1. AllGather Params 0, 将第0层的参数进行汇聚
  2. 将第0层进行前向传播, 得到了下一层的输入, 并释放第0层的内存
     1. 同时AllGather Params 1, 将第1层的参数进行汇聚
  3. 将第1层进行前向传播, 得到了下一层的输入, 并释放第1层的内存
     1. 同时AllGather Params 2, 将第2层的参数进行汇聚
  4. 将第2层进行前向传播, 并进行后向传播, 得到了第2层的梯度, 并释放第2层的内存
     1. 同时AllGather Params 1, 将第1层的参数进行汇聚, (为第1层的后向传播做准备)
  5. ReduceScatter Grads 2, 将第2层的梯度进行分片分发
  6. 将第1层进行后向传播, 得到了第1层的梯度, 并释放第1层的内存
     1. 同时AllGather Params 0, 将第0层的参数进行汇聚, (为第0层的后向传播做准备)
  7. ReduceScatter Grads 1, 将第1层的梯度进行分片分发
  8. 将第0层进行后向传播, 得到了第0层的梯度 (不释放内存, 为下一轮前向传播做准备)
  9. ReduceScatter Grads 0, 将第0层的梯度进行分片分发

### 下一步

DP + ZeRO的效果

![image2025-3-27 15:59:8.png](/assets/01KJBZS9G27XHK76W77HVN48XA/image2025-3-27%2015%3A59%3A8.png)

我们尚不能对 激活值 进行分区, 需要使用 张量并行 （TP）来解决

# 张量并行 Tensor Parallelism

张量并行 （TP），这是一种对权重、梯度和优化器状态以及激活进行分片的方法，无需在计算之前收集它们

主要思路: 将X*W的矩阵乘法, 将X进行广播, 将W进行分片 (按行或按列分片), 举例: 

  - 矩阵乘法
    - ![image](https://nanotron-ultrascale-playbook.static.hf.space/assets/images/tp_diagram.svg)
  - 按列分片
    - ![image](https://nanotron-ultrascale-playbook.static.hf.space/assets/images/tp_diagram2.png)
  - 按行分片
    - ![image](https://nanotron-ultrascale-playbook.static.hf.space/assets/images/tp_diagram3.png)

### Transformer 块中的张量并行性

背景知识

Transformers的结构:

典型结构（以GPT为例）：

  - 输入层：词嵌入 + 位置编码。
  - N个堆叠的Transformer块，每块包含：
    - MHA子层：计算自注意力并输出加权后的特征。
    - Add & Norm：残差连接 + 层归一化。
    - FFN子层：对每个位置独立进行非线性变换。
    - Add & Norm：残差连接 + 层归一化。
  - 输出层：线性投影 + Softmax。

1\. 多头注意力（MHA）的作用

  - 建模长程依赖：通过自注意力机制捕捉输入序列中任意两个位置的关系，无论它们距离多远。
  - 动态权重分配：根据当前输入动态计算注意力权重，使模型能聚焦于关键信息（如上下文关键词）。
  - 多视角学习：通过多个“头”并行学习不同的注意力模式（如语法、语义等），增强模型表达能力。

2\. 前馈网络（FFN）的作用

  - 非线性变换：通过激活函数（如GeLU）引入非线性，提升模型复杂度。
  - 维度扩展与压缩：先将输入从d维扩展到更大的中间维度（如4d），再压缩回d维，增强特征表示能力。
  - 位置独立计算：对每个位置的输入独立处理，弥补注意力机制在局部特征提取上的不足。

### Transformer 块中的张量并行性 - 前馈网络(FFN)

前馈网络FFN的结构是 矩阵乘法 + 非线性变换 + 矩阵乘法, 

所以可以通过 按列分片 + 非线性变换 + 按行分片 来进行分片处理, 这样在整个FFN过程中都可以不交换数据, 示意图: 

(注意: 图中Y1_n, 并没有施加非线性变换, Y2_n 中的计算数值是不准确的, 仅供示意)

(Y1是FFN第一阶段的乘法, Y2是FFN第二阶段的乘法)

![image](https://nanotron-ultrascale-playbook.static.hf.space/assets/images/tp_diagram4.png)

### Transformer 块中的张量并行性 - 多头注意力(MHA)

思路: 

  - 多头注意力的每个头是可以单独计算的
  - 在一个头中, K/Q/V三个矩阵是按列拆分, B矩阵是按行拆分, 这样就像在FFN中类似, 可以在一个GPU内将一个分片的两个乘积操作完成
  - 最后一步Dropout, 需要将Zn合并后再进行

![image2025-3-31 11:19:13.png](/assets/01KJBZS9G27XHK76W77HVN48XA/image2025-3-31%2011%3A19%3A13.png)

### TP的性能效果

TP会让计算分片并行, 减少显存使用:

![image2025-3-31 11:28:6.png](/assets/01KJBZS9G27XHK76W77HVN48XA/image2025-3-31%2011%3A28%3A6.png)

但会引入额外的通讯成本, 其性能平衡效果: 

![image2025-3-31 11:24:23.png](/assets/01KJBZS9G27XHK76W77HVN48XA/image2025-3-31%2011%3A24%3A23.png)

### 下一步 - Sequence Parallelism (序列并行度)

layer normalization 和 dropout 都需要对全部激活 进行计算, 那么计算前就要求进行分片合并. 这两种操作也需要并行化.

SP是沿着输入序列维度拆分, 而不(像TP)是沿着跨隐藏维度拆分:

  - 输入序列维度, 输入N个单词, 将N拆分
  - 跨隐藏维度拆分, 输入N个单词, 每个单词的隐藏状态向量是768维, 将768维拆分
  - 不能沿着跨隐藏维度拆分, 是因为LayerNorm等操作需要完整的隐藏维度 来计算

### SP的流程

向量表示: (b,s,h) = (batch_size, sequence_lengh, hidden_dimensions)

流程图: 

![image](https://nanotron-ultrascale-playbook.static.hf.space/assets/images/tp_sp_diagram_zoomed.png)

  - 初始化LayerNorm (SP)
    - 输入的张量 X1 和 X2 已经对输入X进行了序列维度拆分(b,s/2,h), 分别进行LayerNorm.
      - LayerNorm归一化 是对于每个token是单独进行的, 所以可以按序列维度拆分后再拼接
    - 得到 Y1 和 Y2
  - 第一次过渡 (SP -> TP), 这一阶段进行张量并行
    - g操作(all-gather), 将Y1和Y2合并成Y, Y的维度是(b,s,h)
  - 第一次张量并行  

    - A1,A2是A矩阵沿列拆分, Y进行分片矩阵乘法
    - GeLU操作在分片上进行计算
    - Z1*, Z2*的维度是 (b,s,h/2)
  - 第二次张量并行
    - B1,B2是B矩阵沿行拆分, Z1,Z2分别进行B矩阵乘法
    - W1, W2的维度是 (b,s,h)
  - 最终过渡 (TP -> SP)
    - g*操作(reduce-scatter), 沿序列维度, 将W1+W2进行按序列维度的重分发
    - W1*, W2* 的维度是 (b, s/2, h)

### SP节省内存的效果

![image2025-3-31 13:22:29.png](/assets/01KJBZS9G27XHK76W77HVN48XA/image2025-3-31%2013%3A22%3A29.png)

### SP的计算和通讯的分布

根据其计算性质, 计算和通讯不容易重叠, 那么吞吐量很大程度上依赖于通信带宽

![image2025-4-1 13:23:9.png](/assets/01KJBZS9G27XHK76W77HVN48XA/image2025-4-1%2013%3A23%3A9.png)

### 下一步

TP 和 SP 有两个限制：

  - 1） 如果我们缩放序列长度，激活记忆仍然会在 TP 区域爆炸
  - 2） 如果模型太大而无法适应 TP=8，那么由于节点间连接，我们将看到巨大的减速。
    - 说明: 如果模型太大, 以至于MHA的一个头也无法放在一个GPU内运行, 那么将一个MHA的头拆分到不同的GPU中就增加了大量通讯
      - 如果矩阵乘法的两个矩阵都进行分片, 那么TP的通讯成本过高

通过Context parallelism (上下文并行)来解决限制, 问题定义:

  - 使用了TP+SP后, 当训练序列变长, 那么(参数激活) 仍然可能超过单个GPU
  - 即使我们使用激活的完全重新计算（这需要 ~30% 的沉重计算开销），我们仍然需要在内存中保存一些与序列长度线性缩放的层边界激活

# Context parallelism

CP的主要思想:

  - 之前讨论的TP+SP交替使用, 是将两部分交替使用于不同的模块
  - CP是将TP+SP同时使用在同一模块, 这涉及到复杂的通讯
    - 沿序列维度拆分, 在某些操作上不会有影响, 比如MLP和LayerNorm, 其中每个 token 都是独立处理的.
    - 不过，有一个重要的例外: 在 attention 模块中，每个 token 都需要访问来自所有其他序列 token 的键/值对，或者在因果 attention 的情况下，至少关注每个前一个 token

      - (所以如果 沿序列维度拆分输入, 注意力模块需要 GPU 之间的完全通信才能交换必要的键/值数据)

  

背景资料: 

transformers模型结构:

输入序列 --> Positional Encoding -->   
[Attention Block 1]  
\--> Multi-Head Attention (MHA)  
\--> Residual Connection + LayerNorm  
\--> Feed-Forward Network (FFN, 实现为 MLP)  
\--> Residual Connection + LayerNorm  
[Attention Block 2]  
...  
[Attention Block N]  
\--> 输出序列

引入了Ring Attention技术, 解决在TP+SP同时使用时的通讯调度

### Ring Attention

背景: 使用标准注意力算法进行注意力计算, 其计算一个token时, 需要知道前面和后面的所有token.

Ring attention算法如下图: 

  - 算法成立的前提: 对于标准注意力算法, 其输入可以分片, 而且对输入分片的计算是顺序无关的 (可以先算后面, 在算前面, 然后合并)
  - 将注意力矩阵, 使用TP分散到4个GPU中
  - 对于输入序列 沿序列维度拆分成4份, 每个GPU中计算后, 传给下一个GPU进行计算. 其传递构成了环. 
    - 与之前SP章节相比
      - 之前的SP应用于: layer normalization 和 dropout, 没有应用于整个 注意力块 (其中包括: FFA/MHA/LayerNorm/等)
      - 而: Ring attention应用于整个注意力块

![image](https://nanotron-ultrascale-playbook.static.hf.space/assets/images/ring-attention.gif)

算法存在的问题: 

  - 其只对 顺序无关 的算法有效. 
  - 而因果注意力矩阵(causal attention matrix): 每个位置的 token 只能关注它自己及其之前的 token，而不能看到后面的 token. 所以其计算是有顺序的.
  - 会导致分片计算的压力不均匀

对 因果注意力矩阵 导致的压力不均匀的说明:

  - GPU-1 不需要前置信息, 可以直接计算, 而GPU-2需要等待GPU-1的信息, 以此类推GPU-4需要等待最长时间
  - 与普通注意力矩阵对比: 
    - 普通注意力矩阵: 所有GPU都需要等待其他GPU的信息, 所以各GPU压力均衡

![image](https://nanotron-ultrascale-playbook.static.hf.space/assets/images/cp_attnmask.svg)

解决方案: Zig-Zag Ring Attention

### Zig-Zag Ring Attention

主要思路: 将本轮的部分数据和下一轮的部分数据 混合分布到 某个GPU, 这样综合来看, GPU的压力是均衡的: 

![image2025-4-7 15:16:34.png](/assets/01KJBZS9G27XHK76W77HVN48XA/image2025-4-7%2015%3A16%3A34.png)

说明: 

  - 普通Ring Attention: 将压力轮换限制在某一轮计算 (如果有N个GPU, 那么一轮指的是 N倍token构成的一个token序列, 或者说是输入的一部分)
  - Zig Zag Ring Attention: 将压力轮换限制 放开到多轮计算中, 在多轮计算中均衡, 举一个时序例子: 
    - ![image2025-4-7 15:27:48.png](/assets/01KJBZS9G27XHK76W77HVN48XA/image2025-4-7%2015%3A27%3A48.png)
    - 效率: 

传统 Ring Attention：需要 4×2T = 8T 时间（纯串行）。  
Zig-Zag Ring Attention：仅需 5T 时间（如上表），效率提升 37%。

两种通讯模式: 

  - AllGather: 所有 GPU 同时从所有其他 GPU 收集完整的键/值对
    - ![image](https://nanotron-ultrascale-playbook.static.hf.space/assets/images/cp_overlap_allgather.svg)
  - All-to-All: GPU 以环状模式交换 KV 对，一次一个块
    - ![image](https://nanotron-ultrascale-playbook.static.hf.space/assets/images/cp_overlap_all2all.svg)

# 管道并行度 pipeline parallelism

在TP中提到两个限制, 一个通过CP解决, 另一个是: 

  - 如果模型太大而无法适应 TP=8，那么由于节点间连接，我们将看到巨大的减速。
    - 说明: 如果模型太大, 以至于MHA的一个头也无法放在一个GPU内运行, 那么将一个MHA的头拆分到不同的GPU中就增加了大量通讯
      - 如果矩阵乘法的两个矩阵都进行分片, 那么TP的通讯成本过高

想通过PP来解决: 

  - 思路: 让模型按层拆分到不同的GPU上 (这样一个GPU的显存下降, 可以让 该GPU上的层中的FFN/MHA等组件 不进行复杂拆分)

PP的效果: 

![image2025-4-8 13:19:52.png](/assets/01KJBZS9G27XHK76W77HVN48XA/image2025-4-8%2013%3A19%3A52.png)

(注意: 激活内存并没有因为PP而减少, 因为每层都需要处理整批数据.)

PP的思路有点像ZeRO-3: 

  - ZeRO-3是按需加载参数分片, 在GPU间传输的是某一层的参数
  - PP传输的是 激活张量, 而不是参数分片 (参数分片固定在GPU显存里)

### AFAB(All forward, all backward)

示意图: 全部做前向, 然后全部做后向. 仅在边界(4-5, 8-9, 12-13, 等), 需要传输 激活张量

(图中数字表示模型层)

![image](https://nanotron-ultrascale-playbook.static.hf.space/assets/images/pp_afab.svg)

使用微批次进行优化, 缩减空闲时段: 

(图中数字表示微批次, 不再表示模型层, 模型层的分布如上一张图)

![image](https://nanotron-ultrascale-playbook.static.hf.space/assets/images/pp_afab2.svg)

下一步: 传递的激活, 都需要保存在内存中, 知道后向传递开始, 这会占用大量内存

  - 解决思路: 在执行其他前向计算时, 让后向计算尽早开始, 这样可以尽早释放内存
  - 引入1F1B

### 1F1B - one-forward-one-backward

![image](https://nanotron-ultrascale-playbook.static.hf.space/assets/images/pp_1f1b.svg)

1F1B 节省了内存, 但没有减少空闲时段

下一步: 引入交错阶段(interleaved stages)

### 交错阶段(interleaved stages)

将大模型的层 交错分布在GPU间, 这样: 

  - 增加了需要跨GPU传输的数据量, 原来GPU-1包括层1-4, 现在GPU-1包括层1,3,5,7. 那么层1计算完后, 就需要传输到下一个GPU
  - 当把GPU-1的任务转到下一个GPU后, 当前GPU承接另一个batch的任务, 来填满空闲时段

示意说明:

![image2025-4-8 18:26:28.png](/assets/01KJBZS9G27XHK76W77HVN48XA/image2025-4-8%2018%3A26%3A28.png)

原文的示意图 (没太看懂):

![image](https://nanotron-ultrascale-playbook.static.hf.space/assets/images/pp_1f1b_interleaved.svg)

工程平衡: 

  - 平衡一端: 让当前批次尽早的完成 (尽早开启后向传播, 并完成)
  - 平衡另一端: 让其他批次尽早完成前向传播 (晚开启后向传播)

下一步: 还有一种方法几乎达成了零空闲

### Zero Bubble and DualPipe

观察: 

  - 向后传播的过程中, 有两部分: 
    - 输入 (B) 的向后传播
    - 权重 (W) 的向后传播
  - 向后传播的过程中, 对于下一层的向后传播计算, B是必要的, 而W只需要在优化其步骤前传播就行
    - 原理: 梯度计算和权重优化是两个相对独立的过程, 或者说 权重优化过程依赖于梯度计算, 但梯度计算不依赖于权重优化
    - 附加信息: 

### **背景：反向传播的两种计算**

      1. **输入（B，Backward for Input）** ：

         - 这是指反向传播过程中，当前层需要计算梯度，并将误差（梯度）传递给上一层的输入。
         - 每一层的输入梯度是下一层进行反向传播计算的必要条件。
         - 因此，**输入梯度（B）必须在下一层的反向传播开始之前完成计算** 。
      2. **权重（W，Backward for Weight）** ：

         - 这是指反向传播过程中，当前层需要根据误差计算与其自身权重相关的梯度（用于优化权重）。
         - 权重梯度（W）只需要在优化步骤（例如梯度下降）之前计算即可，不直接影响反向传播到上一层的输入梯度。
         - 换句话说，**权重梯度（W）的计算可以延迟到反向传播的最后阶段** 。

* * *

### **关键点**

      - **输入梯度（B）是必须的** ：

        - 当前层的输入梯度是上一层计算其权重和输入梯度的必要信息。
        - 因此，**B 必须优先计算** ，因为它是向前依赖的（给上一层提供梯度信息）。
      - **权重梯度（W）可以延迟** ：

        - 当前层的权重梯度只与优化权重相关，而优化步骤通常发生在所有反向传播完成后。
        - 因此，权重梯度（W）的计算可以稍后进行，而不会影响反向传播的整体流程。

* * *

### **具体例子**

假设有三层的神经网络（层 1 → 层 2 → 层 3），在反向传播时：

      1. 在计算层 3 的梯度时：

         - 首先计算层 3 对输入的梯度（B3），并传递给层 2。
         - 如果需要更新层 3 的权重，则同时计算层 3 对权重的梯度（W3）。
      2. 在计算层 2 的梯度时：

         - 层 2 需要层 3 传递下来的梯度（B3），才能进行输入梯度（B2）的计算。
         - 层 2 的权重梯度（W2）可以稍后计算，因为它不影响层 1 的反向传播。
      3. 在计算层 1 的梯度时：

         - 层 1 需要层 2 的输入梯度（B2）才能计算自己的输入梯度（B1）。

最终，在所有层的反向传播完成后，权重梯度（W1、W2、W3）会参与权重更新（如梯度下降），而这些权重梯度的计算可以在反向传播的过程中延迟。

* * *

### **总结**

      - **输入梯度（B）** 是反向传播过程中每一层计算的核心，必须在下一层的反向传播计算之前完成。
      - **权重梯度（W）** 只与优化步骤相关，可以延迟到反向传播完成后再计算。
      - 这种区分允许反向传播的过程更加高效，因为可以优先处理计算依赖的部分（B），而将无依赖的部分（W）延后。

  
  

将B(输入传递)和W(权重传递)分离开, 时间规划: (与1F1B不分离时进行对比)

![image](https://nanotron-ultrascale-playbook.static.hf.space/assets/images/pp_zerobubble_ppschedule.png)

最后一种并行方法: 专家并行度

# 专家并行度 Expert parallelism (MoE)

MoE本身就可以切换FFN层, 所以不同的FFN可以天然地进行并行

![image](https://nanotron-ultrascale-playbook.static.hf.space/assets/images/ep_moe.png)

EP与其他并行技术混合使用: 

![image](https://nanotron-ultrascale-playbook.static.hf.space/assets/images/ep_schema.png)
