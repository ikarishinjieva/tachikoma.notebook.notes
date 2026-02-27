---
title: 20250808 - 分析ms-swift的GRPO compute_loss函数
confluence_page_id: 4161926
created_at: 2025-08-08T06:28:33+00:00
updated_at: 2025-08-08T10:07:52+00:00
---

# 获取每个token的对数概率和熵

一个token经过模型后, 对应一个跟词表一样大的概率表

对数概率: 在一个token的概率表中, 已经选择了最有可能的token特定值, 对于这个特定值的概率 取对数

熵: 对于一个token的概率表 进行熵计算, 概率越不均匀, 熵越小. 概率越平均, 熵越大.

  - 熵, 主要表示了在当前上下文, 模型对当前token的自信程度 (概率越平均, 熵越大, 越不自信)

# 处理 top_entropy_quantile 熵分位数

通过 top_entropy_quantile 熵分位数 (比如0.2), 取得 token列表中 熵最高的20%的token 的mask: entropy_mask

# 处理overlong_filter

如果结果被阶段, 则将completion_mask置空

# 计算KL

计算当前生成和参考模型生成之间的KL散度 (对应token列表): per_token_kl

# 处理importance_sampling_level

参考: [20250804 - 对GSPO的理解]

如果importance_sampling_level == token:

  - log_importance_weights = per_token_logps - old_per_token_logps = 当前生成的token列表的对数概率列表 - 之前生成的
  - log_importance_weights 是一组概率, 对每个token进行单独的修正

如果importance_sampling_level == sequence:

  - log_importance_weights = (per_token_logps - old_per_token_logps) / token个数, 以此进行归一化
  - log_importance_weights 是一个概率修正数值, 对当前case的所有completion进行统一修正

importance_sampling_level的主要作用: 将 按token为单位的 对数概率差异, 平均为 按completion为单位的 整体对数概率差异

![image2025-8-8 14:58:36.png](/assets/01KJBZWQZ9BA47ET5VQBKGXEVE/image2025-8-8%2014%3A58%3A36.png)

# 计算loss

GRPO loss 计算代码: 

  - 其中对 logps 进行裁剪
  - 如果 importance_sampling_level = sequence, 那么概率差异 (log_importance_weights) 只剩一个平均数值, 相当于对每个token都是这个数值, 来进行后续计算 (与advantage相乘来计算loss)

```
        # From here, log_importance_weights (and all subsequent tensors, coef_1, coef_2, etc.) shape depends on
        # importance_sampling_level: "token" level: (B, T); "sequence" level: (B, 1)

        coef_1 = torch.exp(log_importance_weights)
        coef_2 = torch.clamp(coef_1, 1 - self.epsilon_low, 1 + self.epsilon_high)
        if self.args.delta is not None:
            coef_1 = torch.clamp(coef_1, max=self.args.delta)

        per_token_loss1 = coef_1 * advantages.unsqueeze(1)
        per_token_loss2 = coef_2 * advantages.unsqueeze(1)
        per_token_loss = -torch.min(per_token_loss1, per_token_loss2)

``` 

代码分步解析

#### 第一步: 计算概率比 (`coef_1`)
    
    
    coef_1 = torch.exp(log_importance_weights)
    

  - **`log_importance_weights`** : 这是我们上一步计算出的 `log(P_new / P_old)`。
  - **`torch.exp(...)`** : 通过取指数 `exp(log(x)) = x`，我们将对数比率转换回了原始的概率比率 `P_new / P_old`。
  - **`coef_1`** : 这个变量现在精确地对应了PPO公式中的 `r_t`。

#### 第二步: 计算裁剪后的概率比 (`coef_2`)
    
    
    coef_2 = torch.clamp(coef_1, 1 - self.epsilon_low, 1 + self.epsilon_high)
    

  - **`torch.clamp(input, min, max)`** : 这个函数会将`input`中的每个元素限制在`[min, max]`的范围内。小于`min`的被提升到`min`，大于`max`的被降低到`max`。
  - **`1 - self.epsilon_low` 和 `1 + self.epsilon_high`**: 这定义了裁剪的区间。通常`epsilon_low`和`epsilon_high`是同一个值`epsilon`（例如0.2），所以区间就是`[0.8, 1.2]`。
  - **`coef_2`** : 这个变量现在精确地对应了PPO公式中的 `clip(r_t, 1-ε, 1+ε)`。

#### 第三步 (可选): 额外的上界裁剪
    
    
    if self.args.delta is not None:
        coef_1 = torch.clamp(coef_1, max=self.args.delta)
    

  - 这是一个**非标准** 但可能存在的额外稳定措施。
  - 它对**未裁剪** 的概率比 `coef_1` 施加了一个绝对的上限 `delta`。
  - **目的** : 即使在后续的计算中我们主要关心裁剪后的版本，这一步也能防止原始的概率比变得过大，从而在某些情况下（比如优势为负时）可能导致数值问题。这可以看作是额外的安全带。

#### 第四步: 计算两个版本的损失项
    
    
    per_token_loss1 = coef_1 * advantages.unsqueeze(1)
    per_token_loss2 = coef_2 * advantages.unsqueeze(1)
    

  - **`advantages.unsqueeze(1)`** : `advantages`通常是按token计算的，但为了和`log_importance_weights`在`sequence`级别下的形状 `(B, 1)` 匹配，可能需要调整维度以利用广播机制。
  - **`per_token_loss1`** : 这对应了PPO公式中的 `r_t * A_t`。这是“乐观”的、未裁剪的目标。
  - **`per_token_loss2`** : 这对应了PPO公式中的 `clip(r_t, 1-ε, 1+ε) * A_t`。这是“悲观”的、被裁剪的目标。

#### 第五步: 取最小值并取反，得到最终损失
    
    
    per_token_loss = -torch.min(per_token_loss1, per_token_loss2)
    

这是整个逻辑的核心，我们来分析一下 `torch.min` 的作用：

  - **Case 1: 优势`A_t` > 0 (这是一个“好”的token)**

    - 我们希望增加这个token的概率，即让 `r_t = P_new/P_old` 变大。
    - `min` 函数会选择 `per_token_loss1` 和 `per_token_loss2` 中较小的那一个。
    - 只要 `r_t` 在 `[1, 1+ε]` 之间，`r_t` < `1+ε`，所以 `r_t * A_t` < `(1+ε) * A_t`。此时 `min` 会选择 `per_token_loss1`，允许策略自由更新。
    - 但如果 `r_t` 变得太大，超过 `1+ε`，那么 `r_t * A_t` > `(1+ε) * A_t`。此时 `min` 就会选择被裁剪的 `per_token_loss2`。这就像一个“刹车”，它阻止我们因为一个好的动作而过于激动，从而防止了过大的策略更新。
  - **Case 2: 优势`A_t` < 0 (这是一个“坏”的token)**

    - 我们希望减小这个token的概率，即让 `r_t` 变小。
    - 因为 `A_t` 是负数，乘以一个更大的正数会得到一个更小（更负）的值。
    - `min` 函数会选择 `per_token_loss1` 和 `per_token_loss2` 中更负的那一个。
    - 这会鼓励模型减小 `r_t`，但同样会限制其不能减小得太过分（不能低于 `1-ε`），从而防止过度惩罚。
  - **最后的负号`-`**:

    - PPO的目标 `L_CLIP` 是一个我们想要**最大化** 的函数。
    - 然而，在PyTorch等深度学习框架中，优化器的工作是**最小化** 一个损失函数（loss）。
    - 因此，我们简单地在目标函数前加上一个负号，将最大化问题转变成了等价的最小化问题。`loss = -L_CLIP`。

### 总结

这段代码完美地实现了PPO-Clip的核心机制：

| 变量 | 对应PPO公式 | 作用 |
| --- | --- | --- |
| `coef_1` | `r_t` | 原始的重要性采样权重（概率比） |
| `coef_2` | `clip(r_t, 1-ε, 1+ε)` | 被限制在安全范围内的概率比 |
| `per_token_loss1` | `r_t * A_t` | 未裁剪的、乐观的策略目标 |
| `per_token_loss2` | `clip(...) * A_t` | 裁剪后的、悲观的策略目标 |
| `torch.min(...)` | `min(...)` | 在乐观和悲观目标中选择更保守的一个，形成“下界” |
| `-torch.min(...)` | `-L_CLIP` | 将最大化目标转换为最小化损失，供优化器使用 |
  
通过这种方式，PPO算法确保了模型在学习过程中既能取得进步，又不会因为步子迈得太大而“摔倒”，从而实现了非常稳定和高效的训练。

# 使用熵阈值来修正loss

使用之前得到的entropy_mask, 修正loss: (只选取熵超过阈值的token的loss)

```
per_token_loss = per_token_loss * entropy_mask
``` 

# 进行KL惩罚

使用之前得到的per_token_kl, 修正loss: 

```
per_token_loss = per_token_loss + self.beta * per_token_kl
``` 

# loss聚合

将per_token_loss聚合成最终值

```
loss = ((per_token_loss * completion_mask).sum(-1) / completion_mask.sum(-1).clamp(min=1.0)).mean()
``` 

# 讨论

对于 importance_sampling_level = sequence, 每个token的logps是被平均成一样的, advantage (优势) 是从 reward 函数中得到, 对于一个prompt的各个token, advantage值是亦一样的.

那么用entropy_mask (熵阈值) 过滤token时, 熵阈值过滤只是决定token的loss是否有值, 选中的token的loss值是一样的. 这些loss参与训练, 而训练过程不会考虑被过滤掉的token.
