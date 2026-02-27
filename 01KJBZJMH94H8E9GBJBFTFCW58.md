---
title: 20241108 - 尝试输出transformers的evaluate结果
confluence_page_id: 3343002
created_at: 2024-11-08T03:36:00+00:00
updated_at: 2024-11-12T02:14:08+00:00
---

# 背景

[20241107 - 训练模型(化学分子式识别) - 尝试使用CoT]

在以上实验中, 发现了过早的过拟合现象

![image2024-11-8 10:38:22.png](/assets/01KJBZJMH94H8E9GBJBFTFCW58/image2024-11-8%2010%3A38%3A22.png)

想查看在训练过程中的evaluate的输出值是什么

# 查找train中的evaluate调用链

代码: /root/miniconda3/envs/huangyan-llama-factory/lib/python3.12/site-packages/transformers/trainer.py

函数: _maybe_log_save_evaluate, 在train过程中会调用该函数, 进行训练中的evaluate

调查evaluate中, loss的相关配置, 关注predict_with_generate. 其解释: 

```
token 的预测概率最终会用于生成序列。但是两者评估的区别在于关注点不同，一个是局部（token级别），一个是全局（序列级别）。

**1. 基于每个 token 的预测概率计算 (不使用 `predict_with_generate`)**

这种方法评估的是模型在**每个位置**预测的准确性。它将参考序列和模型预测的概率分布逐个 token 进行比较，计算每个位置的交叉熵损失，然后将所有位置的损失平均起来。

**例子:**

假设我们要翻译 "Hello world" 成中文 "你好世界"。

* **参考序列 (Target):** 你 好 世 界
* **模型预测概率 (每个位置预测最可能的token，简化表示):** 你 好 世 界 (每个字的预测概率都比较高)
* **模型预测概率 (每个位置预测最可能的token，简化表示):** 你 好 的 界 ( "的" 预测错误)

即使第二个例子只错了一个字，在基于 token 预测概率的评估中，模型在 "的" 这个位置的损失会很高，从而拉高整体损失。这种评估方式**关注每个位置的预测准确性**，但**不考虑生成的完整序列是否流畅或符合语法**。

**2. 基于生成序列的评估指标 (使用 `predict_with_generate`)**

这种方法评估的是模型生成**完整序列**的质量。它将模型生成的完整序列和参考序列进行比较，使用例如 BLEU, ROUGE 等指标来衡量相似度。

**例子:**

假设我们要翻译 "The cat sat on the mat." 成中文 "猫坐在垫子上。"

* **参考序列 (Target):** 猫 坐在 垫子 上 。
* **模型生成序列 1:** 猫 坐在 垫子 上 。 (完美匹配)
* **模型生成序列 2:** 猫 在 垫子 上 坐着。 (语义基本相同，但顺序不同)
* **模型生成序列 3:** 猫 垫子 。 (信息丢失严重)

使用 BLEU 或 ROUGE 等指标评估时，序列 1 的得分最高，序列 2 次之，序列 3 最低。 即使序列 2 中某些 token 的预测概率可能不如序列 3 中的某些 token 高，但由于序列 2 整体上更接近参考序列，因此它的得分更高。这种评估方式**关注生成序列的整体质量**，而**不单独考虑每个 token 的预测概率**。

**总结:**

* **基于 token 预测概率的评估:**  关注模型在每个位置预测的准确性，适合分析模型在哪些位置容易出错。
* **基于生成序列的评估指标:** 关注模型生成完整序列的质量，更贴近实际应用场景，例如机器翻译、文本摘要等。

`predict_with_generate` 参数决定了使用哪种评估方式。在序列生成任务中，通常建议使用 `predict_with_generate=True`，以便更全面地评估模型的性能。
``` 

# 尝试开启predict_with_generate

额外: 报错ValueError: Cannot use `predict_with_generate` if `eval_dataset` is None.

是Llama-factory的参数检查逻辑错误, 需要修改检查逻辑:

![image2024-11-8 14:7:19.png](/assets/01KJBZJMH94H8E9GBJBFTFCW58/image2024-11-8%2014%3A7%3A19.png)

报错: This method only accepts left-padded tensor

认为是Qwen2-VL模型的问题, 直接修改llama-factory的代码, 在predicate过程中变更一下tokenizer配置, 用于generation:

![image2024-11-8 16:58:58.png](/assets/01KJBZJMH94H8E9GBJBFTFCW58/image2024-11-8%2016%3A58%3A58.png)

并且需要将per_device_eval_batch_size设置为 1, 否则会在model.generate中报错

配置文件:

```
### model
model_name_or_path: Qwen/Qwen2-VL-7B-Instruct

### method
stage: sft
do_train: true
finetuning_type: lora
lora_target: all

### dataset
dataset: decimer-img2smiles_cot
dataset_dir: data
template: qwen2_vl
cutoff_len: 1024
preprocessing_num_workers: 16

### output
output_dir: saves/Qwen2VL-7B-Chat/lora/train_qwen2vl_decimer-img2smiles_cot_5000.predict_with_generate
save_steps: 100
plot_loss: true

### train
per_device_train_batch_size: 2
gradient_accumulation_steps: 8
bf16: true
ddp_timeout: 180000000
include_num_input_tokens_seen: true
lora_rank: 8
lora_alpha: 16
optim: adamw_torch
max_grad_norm: 1.0
flash_attn: auto

### eval
val_size: 0.002
eval_strategy: steps

### special
max_samples: 5000
warmup_steps: 300
lora_dropout: 0.2
weight_decay: 0.01
report_to: tensorboard
logging_dir: tensorboard_logging/img2smiles_cot_5000.predict_with_generate
logging_steps: 20
eval_steps: 20
num_train_epochs: 50.0
learning_rate: 1.0e-4
lr_scheduler_type: cosine
seed: 97
predict_with_generate: true
per_device_eval_batch_size: 1 
``` 

训练效果: 

![image2024-11-8 22:54:34.png](/assets/01KJBZJMH94H8E9GBJBFTFCW58/image2024-11-8%2022%3A54%3A34.png)

![image2024-11-8 22:54:55.png](/assets/01KJBZJMH94H8E9GBJBFTFCW58/image2024-11-8%2022%3A54%3A55.png)

```
### model
model_name_or_path: Qwen/Qwen2-VL-7B-Instruct

### method
stage: sft
do_train: true
finetuning_type: lora
lora_target: all

### dataset
dataset: decimer-img2smiles_cot
dataset_dir: data
template: qwen2_vl
cutoff_len: 1024
preprocessing_num_workers: 16

### output
output_dir: saves/Qwen2VL-7B-Chat/lora/train_qwen2vl_decimer-img2smiles_cot_5000.predict_with_generate
save_steps: 100
plot_loss: true

### train
per_device_train_batch_size: 2
gradient_accumulation_steps: 8
bf16: true
ddp_timeout: 180000000
include_num_input_tokens_seen: true
lora_rank: 8
lora_alpha: 16
optim: adamw_torch
max_grad_norm: 1.0
flash_attn: auto

### eval
val_size: 0.002
eval_strategy: steps

### special
max_samples: 5000
warmup_steps: 300
lora_dropout: 0.2
weight_decay: 0.01
report_to: tensorboard
logging_dir: tensorboard_logging/img2smiles_cot_5000.predict_with_generate
logging_steps: 20
eval_steps: 20
num_train_epochs: 50.0
learning_rate: 1.0e-4
lr_scheduler_type: cosine
seed: 97
predict_with_generate: true
per_device_eval_batch_size: 1 
``` 

训练效果: 

![image2024-11-8 22:54:34.png](/assets/01KJBZJMH94H8E9GBJBFTFCW58/image2024-11-8%2022%3A54%3A34.png)

![image2024-11-8 22:54:55.png](/assets/01KJBZJMH94H8E9GBJBFTFCW58/image2024-11-8%2022%3A54%3A55.png)

获得的信息: 

  1. transformers框架中, predict_with_generate只影响eval阶段, 不影响train阶段
     1. 训练阶段的目标函数（基于 logits 的概率分布）和评估阶段的目标（生成序列的质量）之间存在差异。这确实可能导致问题，模型在训练阶段学习到的概率分布可能并不能直接转化为高质量的生成序列
  2. 过拟合的现象, 仍出现在300-500 step之间
  3. 使用 predict_with_generate 时, eval的指标包括 BLEU和ROUGE, 两者评分趋势相同

下一步: 从评估函数中, 打印eval的输出试试

![image2024-11-9 22:2:13.png](/assets/01KJBZJMH94H8E9GBJBFTFCW58/image2024-11-9%2022%3A2%3A13.png)

研究输出: 

一种日志是推理输出了答案但没有过程: 

```
TEST: metric: pred=答: `Cc1ccc(C)c(NC(=O)CS(=O)(=O)c2cccc3nsnc23)c1`, 
 
label=分析:
1. **分析步骤一**
   - **方位**: 图片左侧的苯环
   - **部分特征**: 此部分有两个甲基（-CH₃）取代基，分别位于1位和4位。
   - **对应的smiles**: `Cc1ccc(C)c`
   - **拼接到smiles**: 将其作为起始部分。

2. **分析步骤二**
   - **方位**: 图片中央部分，连接苯环和下一个结构部分
   - **部分特征**: 含有一个氨基（-NH₂）和一个酰氨基部分。
   - **对应的smiles**: `NC(=O)`
   - **拼接到smiles**: 加入到当前部分的后面，形成 `Cc1ccc(C)c(NC(=O)`。

3. **分析步骤三**
   - **方位**: 右侧部分，含有一个亚磺酰（-SO₂）基团
   - **部分特征**: 表现为两个氧原子和硫原子。
   - **对应的smiles**: `CS(=O)(=O)`
   - **拼接到smiles**: 加入，形成 `Cc1ccc(C)c(NC(=O)CS(=O)(=O)`。

4. **分析步骤四**
   - **方位**: 整个分子右侧，包含一个连接的杂环部分
   - **部分特征**: 结构包含一个包含硫和氮的杂环带有两个氮原子。
   - **对应的smiles**: `c2cccc3nsnc23`
   - **拼接到smiles**: 加入，形成最终完整的smiles表达式。
``` 

第二种输出简短的无关单词: 

```
TEST: metric: pred=回答问题, label=分析:
1. 第一部分分析了分子的左下角，包含一个芳香环碳（C）和一个甲基（Me）取代基。对应的smiles是 "c1cc(...)", 这里的"c"表示该部分为芳香烃。
2. 第二部分分析了分子的中间部分，包括一个连接着氨基（NH）和氨基甲酸(N=C)的核心结构。对应的smiles是 "NC(C#N)=C(...)", 为氨基连接结构。
3. 第三部分分析了分子的右上方，包括多个取代基，例如甲氧基（OC）和另一个芳香环。对应的smiles是 "c1cc(...)", 这里的"c"同样表示芳香烃。
4. 第四部分分析了分子右端的取代基，有一个较大的硫酸盐（S(=O)(=O)）结构。对应的smiles是 "NS(=O)(=O)", 反映了该部分的官能团。

这些步骤通过将每个部分的smiles串联起来构成完整的分子表达式。
``` 

第三种输出空: 

```
TEST: metric: pred=, label=分析:
1. 当前步骤分析了图片的上方部分（包含一个氮原子和一氧基）。这个部分是一个环，具有一个氮原子（N）和一个羟基（=O），对应的smiles是 `O=C1N`. 这个部分作为一个简单的五元环结构的一部分
被添加到最终的smiles表达式中。

2. 当前步骤分析了图片的中间部分（含有多个苯环）。这个部分包含两个苯环和一个连接部分（C），对应的smiles是 `CC(c2cccc(c2))`. 由于这些苯环是连接的，所以需要保持此结构完整性并拼接到s
miles中。

3. 当前步骤分析了图片的下方部分（展示了一个带有阳离子的氮原子和一个负离子的氧原子）。这个部分是硝基基团（[N+](=O)[O-]），对应的smiles是 `c2c(ccc3ccccc23)N+`. 这是一个重要的功能团
，需要拼接在整个分子中。

4. 最后，结合所有部分，构建出完整的smiles表达式。确认各部分连接的正确性以确保结构的完整性。
``` 

第四种能输出步骤: 

```
TEST: metric: pred=解析：
1. 分析了化学分子的左侧，得出 `O=C(N ... )`，表示一个酰胺基团。
2. 接下来针对中心的苯环，得出 `c1ccc(... )cc1`，包含连接的苯基和其衍生基团。
3. 继续向右侧，识别出另一个苯环及其氟取代基，得出 `F` 的存在。
4. 最终，通过各部分构建，得出完整的smiles表达式。, 
 
label=分析:
1. 第一部分分析了化学分子的左侧，包含一个羧基 (C=O) 和一个氨基 (N)。这部分对应的是分子的中心结构，所以对应的smiles是 `O=C(N ... )`。

2. 接下来分析中心的苯环部分，显示有两个苯环连接在一起，其中一个苯环带有一个磺酰基 (S(=O)(=O)) 和一个氨基 (N)。对于这一部分，smiles为 `c1ccc(S(=O)(=O)Nc2ncccn2)cc1`，表示这个苯环
连接了其他的功能团。

3. 继续向右侧分析一个另一个苯环，并且有一个磺酰基和氨基的装置，smiles是 `NS(=O)(=O)c1ccc(F)cc1`，包括了氟原子（F），表明该环含有氟取代基。

4. 最后汇总起来，逐步将各个部分拼接起来形成完整的smiles表达式，依次添加每一部分，并保持合适的顺序。

完整的smiles表达式是: O=C(Nc1ccc(S(=O)(=O)Nc2ncccn2)cc1)c1ccccc1NS(=O)(=O)c1ccc(F)cc1
``` 

向模型询问: 

````
我在微调一个模型, 其训练数据都是问答, 答案类似于: 
```
分析:
1. 第一部分分析了化学分子的左侧，包含一个羧基 (C=O) 和一个氨基 (N)。这部分对应的是分子的中心结构，所以对应的smiles是 `O=C(N ... )`。

2. 接下来分析中心的苯环部分，显示有两个苯环连接在一起，其中一个苯环带有一个磺酰基 (S(=O)(=O)) 和一个氨基 (N)。对于这一部分，smiles为 `c1ccc(S(=O)(=O)Nc2ncccn2)cc1`，表示这个苯环
连接了其他的功能团。

3. 继续向右侧分析一个另一个苯环，并且有一个磺酰基和氨基的装置，smiles是 `NS(=O)(=O)c1ccc(F)cc1`，包括了氟原子（F），表明该环含有氟取代基。

4. 最后汇总起来，逐步将各个部分拼接起来形成完整的smiles表达式，依次添加每一部分，并保持合适的顺序。

完整的smiles表达式是: O=C(Nc1ccc(S(=O)(=O)Nc2ncccn2)cc1)c1ccccc1NS(=O)(=O)c1ccc(F)cc1
```

经过模型微调, 发现已经产生了过拟合, 并且其eval的输出分为以下几类: 
- 第一类: 输出了答案但没有过程, 比如: "答: `Cc1ccc(C)c(NC(=O)CS(=O)(=O)c2cccc3nsnc23)c1`"
- 第二类: 输出简短的无关单词, 比如: "回答问题"
- 第三类: 输出为空, 比如: ""
- 第四类: 输出步骤, 但没有答案, 比如: "解析：
1. 分析了化学分子的左侧，得出 `O=C(N ... )`，表示一个酰胺基团。
2. 接下来针对中心的苯环，得出 `c1ccc(... )cc1`，包含连接的苯基和其衍生基团。
3. 继续向右侧，识别出另一个苯环及其氟取代基，得出 `F` 的存在。
4. 最终，通过各部分构建，得出完整的smiles表达式。"

我观察了在训练过程中 还没有过拟合/开始过拟合/轻微过拟合/严重过拟合 几个阶段, 输出都差不多.

我想知道这个微调的问题在哪, 在我看来, 已经产生了过拟合, 但eval的输出就连格式都不像训练数据.
```` 

模型给的思路: 

```
看起来您的模型在微调后输出的结果与训练数据的格式和内容不一致，出现了多种类型的输出，包括仅有答案没有过程、无关的简短词语、空输出以及仅有步骤没有最终答案。这种现象可能与以下几个方面的问题有关：

**1. 训练数据和标签的对齐问题**

最常见的原因是**训练数据的输入和输出（标签）没有正确对齐**。这可能导致模型在学习过程中无法正确地将输入映射到预期的输出。

- **检查数据预处理**：确保在数据预处理阶段，输入文本和对应的输出（答案，包括过程和最终答案）是正确配对的。
- **输入和输出分隔**：在训练数据中，明确地分隔输入和输出。例如，使用特殊的标记（如 `<|prompt|>` 和 `<|answer|>`）来区分用户提问和模型回答。

**2. 模型微调的参数设置**

如果模型在微调过程中参数设置不当，可能导致过拟合或欠拟合，进而影响模型的生成质量。

- **学习率**：过高的学习率可能导致模型无法收敛，过低的学习率可能导致训练速度过慢或陷入局部最优。
- **训练轮数（epochs）**：过多的训练轮数可能导致过拟合，过少的训练轮数可能导致欠拟合。
- **优化器选择**：确保选择适合的优化器，并正确设置其参数。

**3. 特殊字符和标记处理**

您的回答中包含了化学式和特殊符号，如 `O=C(N ... )`、`c1ccc(S(=O)(=O)Nc2ncccn2)cc1` 等。

- **分词器（Tokenizer）**：确保分词器能够正确处理这些特殊符号。必要时，可以自定义分词器或添加特殊的词汇表。
- **编码方式**：检查数据的编码方式，确保在读取和处理数据时不会出现字符编码问题。

**4. 训练目标和损失函数**

如果训练过程中使用了不适当的损失函数，或者训练目标没有正确设置，可能导致模型无法正确学习。

- **损失函数**：对于生成式任务，通常使用交叉熵损失函数。确保损失函数适合您的任务。
- **训练目标**：确保模型的训练目标是让它在给定输入后生成期望的输出（包括详细的步骤和最终答案）。

**5. 评估方法和指标**

在评估模型时，使用的输入可能与训练时的输入不一致，或者评估指标不能有效反映模型的性能。

- **评估输入**：确保在评估时，提供给模型的输入与训练时的格式和内容一致。
- **温度和采样策略**：在生成文本时，设置合适的温度参数和采样策略，以平衡创造性和准确性。

**6. 数据规模和多样性**

如果训练数据规模过小，或者数据缺乏多样性，模型可能无法学到有效的生成模式。

- **增加数据量**：收集更多的问答对，尤其是包含步骤分析和最终答案的高质量数据。
- **数据增强**：通过数据增强技术，增加数据的多样性和丰富性。

**7. 模型容量和限制**

选择的模型规模可能限制了它的学习能力。

- **模型大小**：考虑使用更大的模型，或者调整模型的参数，使其有能力学习复杂的模式。
- **模型结构**：确保模型结构适合生成任务，尤其是需要生成长文本和复杂内容的任务。

**建议的解决方案：**

- **检查并确保训练数据的输入和输出正确对齐**，并且数据预处理过程没有问题。
- **验证分词器对特殊符号的处理**，必要时自定义分词器或调整词汇表。
- **调整训练参数**，如学习率、训练轮数和优化器设置，防止过拟合或欠拟合。
- **在评估时，确保输入格式与训练时一致**，并设置合适的生成策略。
- **如果可能，增大训练数据量并提高数据质量**，以帮助模型学习更好的生成模式。

通过上述步骤，您应该能够找出问题所在，并改进模型的微调效果，使其生成与训练数据格式和内容一致的输出。
``` 

# 尝试LR=1e-2

![image2024-11-10 17:22:26.png](/assets/01KJBZJMH94H8E9GBJBFTFCW58/image2024-11-10%2017%3A22%3A26.png)

eval的日志输出: 

```
TEST: metric: pred=2 - `cc)cOO=O=OO1.，有一个，表示连接，`部分，cO=(O==OO==1-])ccO1-.连接，，“c=====(OO)((1），图片为拼iles部分，特征。这个表示结构（ccc2ccOOO1-2COO1cc=12`苯SMsmiles
（11.部分的smsm式，苯的smsmiles环的的的smsmiles一个是对应的ILES的原子。对应的式部分到键结构是 `cccO(C)(1cccO1)3NNcccO23.步骤:
.cO((==1., 
 
label=分析:
1. **分析部分**: 从分子结构的左上方开始分析，看到有一个氧原子和一个氮原子连接的部分。
   - **特征**: 这是硝基（-NO2）基团，其中氮原子带正电，氧原子带负电。
   - **对应的smiles**: `[N+](=O)[O-]`
   - **拼接**: 将此部分拼接到最终的SMILES中。

2. **分析部分**: 然后分析中间部分，看到一个六元环的苯环。
   - **特征**: 苯环（c1ccccc1），有代替基团。
   - **对应的smiles**: `c1ccccc1`
   - **拼接**: 连接前面的硝基与此苯环，形成`Cc1cccccc1`。

3. **分析部分**: 继续分析，发现苯环上有一个氨基（-NH2）取代基。
   - **特征**: 这个氨基连接在苯环的右侧。
   - **对应的smiles**: `N`
   - **拼接**: 在苯环的右侧添加氨基，形成`Cc1cc([N+](=O)[O-])ccc1N`。
```
````
TEST: metric: pred=smILES基为拼ILES氧一个，包含表达smiles基是特征为键碳的氧碳，一个环结构，连接连接为拼ILES，**:部分为对应的iles的smiles部分是部分的基，`团。

3. ** "--c=O))N.是步c(OO==OOO)c=2`的SMILES。

32ccc1Nccc12-`部分。�的一个（-.团表达iles。 "`表达smiles为。 "cO1cc=(=1cc=O(C===(OO1N`，苯环环基的环原子，部分的的sm式，`（c1）部分（(OO=O=OcccO).团。

N -:分析， **苯接是:分析了是这个分析：，“cO1C=13cccO)）（) -，, 
 
 
label=```
分析:
1. 分析步骤1: 图片左侧部分显示了一个氨基(-NH2)基团，位于分子的起始位置，与氨基相连的碳原子有一个羧基(C(=O))。这对应的SMILES是"NC(=O)"，因为氨基和羧基确定了这个片段。
2. 分析步骤2: 接下来是一个五元环的结构，位于图片中心位置，由五个碳原子和一个氮原子组成，形成环系统。这个环与之前的片段相连。它在SMILES中用"CC1"表示，其中"1"表示环的起始。
3. 分析步骤3: 在环的另一侧，有一个旁支，连接到一个含有两个硝基(-NO2)的苯环部分，苯环显示有两个氮正离子状态的硝基，位于图片的右侧。这个片段在SMILES中是"c2c([N+](=O)[O-])cc(C(F)(F)F)c
c2[N+](=O)[O-]"，表示苯环和取代基。
4. 分析步骤4: 将所有部分拼接在一起。首先是氨基的羧基部分，再加上环部分，最后是苯环部分。最后得到的完整SMILES表达式是"NC(=O)C1CCN(c2c([N+](=O)[O-])cc(C(F)(F)F)cc2[N+](=O)[O-])CC1"。

完整的smiles表达式是: NC(=O)C1CCN(c2c([N+](=O)[O-])cc(C(F)(F)F)cc2[N+](=O)[O-])CC1
```
```` 

看上去推理已经可以出结构 (不是所有数据都有, 部分数据还是无意义的输出)

考虑更换train阶段的loss??

# 更换train的loss计算函数

[trainer_rouge.py](/assets/01KJBZJMH94H8E9GBJBFTFCW58/trainer_rouge.py)

更换loss函数, 使用rouge+bleu作为loss. 但更换后, 发现训练的loss曲线为直线, 梯度完全为0:

![image2024-11-11 23:44:57.png](/assets/01KJBZJMH94H8E9GBJBFTFCW58/image2024-11-11%2023%3A44%3A57.png)![image2024-11-11 23:45:16.png](/assets/01KJBZJMH94H8E9GBJBFTFCW58/image2024-11-11%2023%3A45%3A16.png)

询问大模型: 

````
我在微调一个模型, 他原来的loss计算函数如下: 
```
def compute_loss(self, model, inputs, return_outputs=False):
        """
        How the loss is computed by Trainer. By default, all models return the loss in the first element.

        Subclass and override for custom behavior.
        """
        if self.label_smoother is not None and "labels" in inputs:
            labels = inputs.pop("labels")
        else:
            labels = None

        logger.warning(f"TEST: compute_loss: labels={labels}")
        
        outputs = model(**inputs)
        # Save past state if it exists
        # TODO: this needs to be fixed and made cleaner later.
        if self.args.past_index >= 0:
            self._past = outputs[self.args.past_index]

        if labels is not None:
            unwrapped_model = self.accelerator.unwrap_model(model)
            if _is_peft_model(unwrapped_model):
                model_name = unwrapped_model.base_model.model._get_name()
            else:
                model_name = unwrapped_model._get_name()
            if model_name in MODEL_FOR_CAUSAL_LM_MAPPING_NAMES.values():
                loss = self.label_smoother(outputs, labels, shift_labels=True)
            else:
                loss = self.label_smoother(outputs, labels)
        else:
            if isinstance(outputs, dict) and "loss" not in outputs:
                raise ValueError(
                    "The model did not return a loss from the inputs, only the following keys: "
                    f"{','.join(outputs.keys())}. For reference, the inputs it received are {','.join(inputs.keys())}."
                )
            # We don't use .loss here since the model may return tuples instead of ModelOutput.
            loss = outputs["loss"] if isinstance(outputs, dict) else outputs[0]

        return (loss, outputs) if return_outputs else loss
```

我将其更换成了以下函数: 
```
def compute_loss(self, model, inputs, return_outputs=False):
        # Get the labels
        labels = inputs.pop("labels")
        
        # Forward pass
        outputs = model(**inputs)

        # Get logits and predictions
        logits = outputs.logits
        predictions = torch.argmax(logits, dim=-1)

        # Move tensors to CPU and convert to numpy arrays
        predictions = predictions.detach().cpu().numpy()
        labels = labels.detach().cpu().numpy()
        
        # Replace ignore_index and pad_token_id in labels
        labels = np.where(labels != -100, labels, self.tokenizer.pad_token_id)

        # Decode predictions and labels
        # decoded_preds = self.tokenizer.batch_decode(predictions, skip_special_tokens=True)
        # decoded_labels = self.tokenizer.batch_decode(labels, skip_special_tokens=True)

        # Compute metrics using ComputeSimilarity
        compute_similarity = ComputeSimilarity(self.tokenizer)
        eval_prediction = EvalPrediction(predictions=predictions, label_ids=labels)
        scores = compute_similarity(eval_prediction, compute_result=True)

        # Extract the scores
        rouge1 = scores['rouge-1']
        rouge2 = scores['rouge-2']
        rougel = scores['rouge-l']
        bleu4 = scores['bleu-4']

        # Compute a pseudo-loss (note: this is not differentiable)
        loss = (rouge1 + rouge2 + rougel + bleu4) / 100

        # Convert loss to tensor
        loss = torch.tensor(loss, requires_grad=True, device=self.args.device)
        
        return (loss, outputs) if return_outputs else loss
```

其中compute_similarity函数如下: 
```
class ComputeSimilarity:
    r"""
    Computes text similarity scores and supports `batch_eval_metrics`.

    Wraps the tokenizer into metric functions, used in CustomSeq2SeqTrainer.
    """

    tokenizer: "PreTrainedTokenizer"

    def _dump(self) -> Optional[Dict[str, float]]:
        result = None
        if hasattr(self, "score_dict"):
            result = {k: float(np.mean(v)) for k, v in self.score_dict.items()}

        self.score_dict = {"rouge-1": [], "rouge-2": [], "rouge-l": [], "bleu-4": []}
        return result

    def __post_init__(self):
        self._dump()

    def __call__(self, eval_preds: "EvalPrediction", compute_result: bool = True) -> Optional[Dict[str, float]]:
        preds, labels = numpify(eval_preds.predictions), numpify(eval_preds.label_ids)

        preds = np.where(preds != IGNORE_INDEX, preds, self.tokenizer.pad_token_id)
        labels = np.where(labels != IGNORE_INDEX, labels, self.tokenizer.pad_token_id)

        decoded_preds = self.tokenizer.batch_decode(preds, skip_special_tokens=True)
        decoded_labels = self.tokenizer.batch_decode(labels, skip_special_tokens=True)

        for pred, label in zip(decoded_preds, decoded_labels):
            print(f"TEST: metric: pred={pred}, label={label}")

            hypothesis = list(jieba.cut(pred))
            reference = list(jieba.cut(label))

            if len(" ".join(hypothesis).split()) == 0 or len(" ".join(reference).split()) == 0:
                result = {"rouge-1": {"f": 0.0}, "rouge-2": {"f": 0.0}, "rouge-l": {"f": 0.0}}
            else:
                rouge = Rouge()
                scores = rouge.get_scores(" ".join(hypothesis), " ".join(reference))
                result = scores[0]

            for k, v in result.items():
                self.score_dict[k].append(round(v["f"] * 100, 4))

            bleu_score = sentence_bleu([list(label)], list(pred), smoothing_function=SmoothingFunction().method3)

            self.score_dict["bleu-4"].append(round(bleu_score * 100, 4))

        if compute_result:
            return self._dump()
```

在更换loss计算函数后, 发现训练出现了问题: train loss一直不下降, 并且梯度近似为0, 你能帮我分析一下原因么? 
```` 

大模型的答案: 

````
问题在于你新的 `compute_loss` 函数计算的 loss 不可微分。你使用 `rouge`、`bleu` 和 `jieba` 等工具计算了相似度得分，然后将这些得分组合成一个标量 loss 值。这些操作（分词、n-gram 匹配、字符串操作等）都是离散的，因此无法通过它们反向传播梯度。PyTorch 无法计算这些操作的梯度，导致模型参数无法更新。

更具体地说，以下步骤导致了不可微分的问题：

1. **`jieba.cut`**:  结巴分词是一个基于词典和规则的分词工具，它涉及到字符串匹配和查找等离散操作，这些操作无法计算梯度。

2. **`Rouge()` 和 `sentence_bleu`**:  Rouge 和 BLEU 分数的计算涉及到计数和比较 n-gram 的出现次数，这也是离散的操作，无法进行梯度计算。

3. **`np.where` 和类型转换**:  虽然 `np.where` 本身不影响可微性，但它与 `tokenizer.batch_decode` 和 `jieba.cut` 等操作结合使用，最终导致结果的离散化。同样，将张量转换为 NumPy 数组也切断了计算图，使得梯度无法回传。

4. **`torch.tensor(loss, requires_grad=True)`**: 虽然你将最终的 loss 值转换为 PyTorch 张量并设置 `requires_grad=True`，但这并不能使之前的计算步骤变得可微分。梯度仍然无法通过离散的操作反向传播。

**解决方案:**

要解决这个问题，你需要使用可微分的指标来计算 loss。以下是一些建议：

1. **使用 Hugging Face 提供的 `seq2seq` loss**:  如果你使用的是 `transformers` 库中的 `Seq2SeqLMOutput`，可以直接使用其中的 `loss` 属性，它通常是交叉熵损失，是可微分的。这是最推荐的方案。

2. **使用 BERTScore 等可微分指标**:  BERTScore 等指标基于预训练的语言模型，可以提供更细粒度的文本相似度评估，并且是可微分的。你可以使用 `bertscore` 库来计算 BERTScore。

3. **使用 Sentence-BERT 计算余弦相似度**:  你可以使用 Sentence-BERT 将预测和标签编码成向量，然后计算它们之间的余弦相似度作为 loss。余弦相似度是可微分的。

4. **直接优化 token 级别的交叉熵损失**:  这是最标准的训练 seq2seq 模型的方法。

**示例：使用交叉熵损失 (推荐)**

```python
def compute_loss(self, model, inputs, return_outputs=False):
    """
    How the loss is computed by Trainer. By default, all models return the loss in the first element.

    Subclass and override for custom behavior.
    """
    outputs = model(**inputs)
    loss = outputs.loss  # Directly use the loss from the model output
    return (loss, outputs) if return_outputs else loss
```

选择哪种方案取决于你的具体需求和模型结构。  避免使用不可微分的操作来计算 loss，才能保证模型能够正常训练。
```` 

如果要进行loss的反向传播, 那么loss计算中的每个环节都必须能计算梯度. 也就是embedding模型要放在model.forward中, 参与到模型的传播中
