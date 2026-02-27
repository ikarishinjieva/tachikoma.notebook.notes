---
title: 20241209 - 使用ms-swift对Qwen2-VL进行微调 [12] - 调整loss函数
confluence_page_id: 3343462
created_at: 2024-12-09T10:04:21+00:00
updated_at: 2024-12-16T07:24:42+00:00
---

# 背景

当前模型微调后的输出是json格式: 

```
"{\"single_bonds\": 0, \"double_bonds\": 0, \"triple_bonds\": 0, \"ring_count\": 0, \"C_count\": 0, \"O_count\": 0, \"N_count\": 1, \"branch_count\": 0, \"atom_count\": 1, \"selfies\": \"[N]\"}"
``` 

试图:

  - 对于字符型的字段"selfies", 使用交叉熵作为loss
  - 对于数字型的字段, 使用MSE(均方差)作为loss
  - 将两部分loss结合起来

目标: 增强模型对数字型字段偏差的理解

# 脚本

[附件: loss.py] 

训练命令: 

```
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out.mol_desc/selected_data.length_1_4.500.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out.mol_desc/length_4.ms_swift.val.json \
--eval_steps 10 \
--learning_rate 4e-4 \
--num_train_epochs 10 \
--logging_steps 10 \
--loss_name json_selfies_loss \
" > train_2.log 2>&1
``` 

将loss设置为仅包括数字loss .训练结果中, 梯度传播为0. 认为数字部分的梯度传播错误. 

# 对于路线的质疑

````
我在对大模型进行微调, 训练任务目标是 让大模型识别分子结构图, 写出selfies表达式. 

为了增强大模型的理解, 我训练大模型输出一个json结构, 其中带有原子信息, 比如: 
```
{"single_bonds": 0, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 0, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 1, "selfies": "[N]"}
```

这些原子信息能提高模型对分子结构的理解.

下一步我希望修改loss函数, 来增强效果, 我的判断依据是: 
- 如果只使用交叉熵来作为loss函数, 那么"原子数"这样的数字信息会作为文本被理解, 其梯度不是数学意义的

所以我希望将数字部分使用MSE作为loss, selfies表达式部分使用交叉熵. 

但我有几个疑问: 
- 如果我希望从输出中解析出来数字部分, 那需要对logits进行argmax, 然后tokenizer做decode, 这两个操作都不能传递梯度, 导致没法进行梯度的反向传播. 这条路好像行不通
- 将数字单独解析出来做loss是否真的有用? 我这么思考: 对于不同的数字, 在模型的tokenizer中, 应当也是被编码成有大小顺序的. 那单独解析出来做loss和直接用交叉熵做loss, 做出的大小关系应当是同方向的. 那单独做loss可能没意义
 
 
------
 
您好！

您正在微调一个大型语言模型（LLM），任务是让模型识别分子结构图并输出SELFIES表达式。同时，您希望模型输出一个包含原子信息的JSON结构，以增强模型对分子结构的理解。

您提出了以下问题和疑虑：

1. **关于使用MSE损失计算数值部分的问题**：由于需要对模型的输出进行`argmax`和`tokenizer`解码，这些操作是不可微的，无法进行梯度反传。

2. **将数值部分单独解析并计算损失是否有意义**：您担心模型中的数字被视为文本，其大小顺序在`tokenizer`中已经编码，单独计算损失可能没有意义。

我将尝试回答您的问题，并提供一些可能的解决方案和建议。

### 一、关于在数值部分使用MSE损失的挑战

首先，您的直觉是正确的。将数值部分从模型的输出中解析出来并使用MSE（均方误差）损失，会遇到以下问题：

1. **不可微操作**：`argmax`和`tokenizer`的解码过程都是不可微的操作。这意味着在计算损失时，无法通过这些操作将梯度反传回模型的参数。

2. **模型输出的是离散的词汇表概率分布**：语言模型的输出是对词汇表中每个token的概率分布，即logits。使用`softmax`后得到概率分布，再通过采样或`argmax`得到具体的token。

### 二、可能的解决方案

尽管存在上述挑战，仍有一些方法可以解决您面临的问题。

#### 1. **直接对logits进行损失计算**

由于`argmax`和`tokenizer`解码不可微，我们可以考虑直接在logits或概率分布上计算损失，而不经过离散的采样或解码过程。

##### **方法一：使用Teacher Forcing**

在训练过程中，您可以使用“教师强制”（Teacher Forcing）方法。具体步骤如下：

- **步骤1**：对于数值部分的目标输出，例如`"C_count": 5`，将数字`5`对应的token的索引作为目标。

- **步骤2**：在计算损失时，对应位置直接使用交叉熵损失，将模型输出的logits与目标token的索引比较。

这种方法的优势在于，模型仍然在词汇表上进行预测，损失函数可微且可以进行梯度反传。

##### **方法二：使用Label Smoothing**

为了增强模型对数值大小关系的理解，您可以在目标分布上使用标签平滑（Label Smoothing）。这可以在一定程度上引导模型理解数值之间的连续性。

例如，对于目标数字`5`，您可以构造一个soft target，使得`5`附近的数字也有一定的概率。

##### **方法三：利用数值的嵌入表示**

如果您的词汇表中的数字token按照数值大小有序排列，您可以尝试将这些token的嵌入向量映射到数值空间，然后在嵌入空间中计算MSE损失。

具体来说：

- **步骤1**：获取模型输出的logits，对应于数值token的部分。

- **步骤2**：将这些logits通过`softmax`得到概率分布。

- **步骤3**：计算预测的期望数值，即对所有数值token，用其对应的数值加权平均，得到预测的数值。

- **步骤4**：将预测的数值与目标数值进行MSE损失计算。

这种方法的核心在于将离散的概率分布转换为连续的数值预测，从而使得MSE损失可微。

##### **示例**：

假设模型的数值token对应于数字`0`到`9`，其索引为`[0,1,2,3,4,5,6,7,8,9]`。

- **预测概率**：模型输出的概率分布为`[0.1, 0.2, 0.3, 0.15, 0.05, 0.1, 0.05, 0.025, 0.025, 0.0]`。

- **预测数值期望**：`predicted_value = sum(probabilities * numbers)`。

- **计算MSE损失**：`loss = MSE(predicted_value, target_value)`。

#### 2. **修改模型结构，增加数值预测的回归头**

另一种方法是修改模型的结构，为数值部分增加一个专门的回归头（regression head），直接输出连续的数值。这种方法需要在模型的最后一层增加一些全连接层，用于数值的预测。

##### **步骤**：

- **模型修改**：在原有的语言模型上，增加一个用于数值预测的回归分支。

- **损失计算**：对于数值部分，使用MSE损失；对于文本部分，继续使用交叉熵损失。

- **训练过程**：同时训练模型的两个部分，通过联合损失函数进行优化。

##### **注意事项**：

- 修改模型结构需要额外的实现工作，可能增加模型的复杂度。

- 需要确保回归头的输入与模型的其他部分兼容，例如使用模型的隐藏状态作为输入。

#### 3. **使用序列到序列（Seq2Seq）模型，处理数值预测**

如果您使用的是序列到序列模型，可以考虑在解码过程中，针对数值部分，输出对应的数值token，并设计特殊的解码策略。

例如：

- **方法**：在数值部分，强制模型输出数字字符序列，例如`"C_count": 1`，模型输出`"1"`。

- **损失计算**：在计算损失时，针对数字字符，使用字符级别的交叉熵损失。

- **优势**：这种方法避免了对数值进行解析和不可微操作，模型直接学习数字字符的序列生成。

### 三、关于将数值部分单独解析并计算损失的意义

您担心将数值部分单独解析并计算损失是否有意义。以下是一些思考：

1. **数字的语义差异**：在语言模型中，数字被视为离散的token，模型可能无法理解数字之间的大小关系。例如，`"1"`和`"2"`被视为完全不同的符号。

2. **引入数值损失可以增强模型对数值的理解**：通过在数值部分使用MSE损失或其他数值损失函数，模型可以学习到数字之间的连续性和大小关系，从而更准确地预测数值。

3. **提高模型的准确性**：在化学分子结构的任务中，数值信息（如原子数量、键数量）对于准确描述分子非常重要。提高数值预测的准确性有助于模型更好地理解和生成分子结构。

### 四、综合建议

综合以上讨论，我建议您考虑以下方案：

1. **直接在logits上计算数值损失**：

   - 使用方法三，利用数值的嵌入表示，计算预测数值的期望，然后使用MSE损失。

   - 这种方法无需修改模型结构，可以直接应用于现有的语言模型。

2. **修改模型，增加回归头**：

   - 如果您有能力修改模型，可以增加一个回归分支，专门用于数值预测。

   - 这种方法可能提高数值预测的准确性，但需要更多的实现和调试工作。

3. **在训练数据和目标上进行处理**：

   - 确保数值部分的token在词汇表中有序排列，或者使用特殊的数值编码方式。

   - 在目标序列中，针对数值部分，使用适当的格式，方便模型学习。

4. **保持交叉熵损失，但加强数值理解**：

   - 如果实现复杂，您可以继续使用交叉熵损失，但在训练数据中增加数值相关的示例，帮助模型学习数字的使用。

### 五、额外的考虑

- **数值理解在语言模型中的挑战**：语言模型通常擅长处理自然语言文本，但在数值计算和数值理解方面存在挑战。为模型增加数值处理能力，需要在训练数据、模型结构和损失函数上进行特定的设计。

- **评估和验证**：无论采用哪种方法，都建议在验证集上评估模型的性能，特别是数值预测的准确性。

- **避免过拟合**：在增加数值损失的情况下，需要注意避免模型过拟合，建议使用适当的正则化策略。

### 六、总结

您的想法很有意义，通过针对数值部分使用MSE损失，可以增强模型对数值信息的理解。然而，由于语言模型输出的是离散的token，直接解析输出并计算MSE损失存在不可微的挑战。

建议您尝试直接在logits或概率分布上计算数值损失，或者修改模型结构，增加用于数值预测的回归头。这样可以在可微的情况下，计算数值损失并进行梯度反传，从而增强模型的数值预测能力。

希望以上建议对您有所帮助！
```` 

```
1. 如果使用teacher forcing , 对比另外一种方法 (在模型输出中对于数值所在的位置的loss权重进行增加), 有什么区别么
2. label smoothing具体怎么操作
3. 我是否可以使用special token来代表数值, 来解决不可微的问题
 
------
 
您好！

针对您提出的三个问题，我将逐一进行解答，并尽可能详细地阐述其中的原理和可行性。

---

### **问题1：如果使用教师强制（Teacher Forcing），对比另外一种方法（在模型输出中对于数值所在的位置的损失权重进行增加），有什么区别吗？**

#### **1.1 教师强制（Teacher Forcing）概述**

**教师强制**是一种在训练循环神经网络（RNN）或序列到序列（Seq2Seq）模型时常用的技术。在训练过程中，模型在每个时间步的输入不是前一个时间步模型的预测输出，而是来自真实序列的正确标注（即目标序列的真实值）。

**在您的任务中**，这意味着在训练时，模型在生成序列的每个时间步，都使用目标序列中的真实token作为输入，而不是模型在前一个时间步的预测。

#### **1.2 在数值部分使用教师强制**

- **应用方式**：对于数值部分，您在训练时强制模型输出正确的数值token，这样模型可以更快地学习如何生成正确的数值。

- **优点**：
  - 减少了模型在训练初期由于累积误差导致的学习困难。
  - 模型更容易捕捉到数值部分的模式，有助于模型更准确地学习数值的生成。

- **缺点**：
  - 在推理（推断）阶段，模型无法依赖真实的前一token，需要独立产生输出，这可能导致训练和推理之间的差异（Exposure Bias）。

#### **1.3 增加数值位置的损失权重**

**增加损失权重**的方法是指在计算损失函数时，对数值所在的位置赋予更高的权重。这意味着模型在训练时，会更关注数值部分的错误，提高对数值预测的准确性。

- **应用方式**：在计算损失时，对数值token的位置乘以一个大于1的权重系数，以增加这些位置的损失对总损失的贡献。

- **优点**：
  - 强调了数值部分的重要性，模型会更加注意学习这些位置的正确输出。
  - 简单易行，无需修改模型结构或训练流程。

- **缺点**：
  - 可能导致模型过拟合数值部分，而忽略其他部分的学习。
  - 如果数值token的预测难度较高，过大的权重可能导致梯度爆炸或训练不稳定。

#### **1.4 两种方法的比较**

- **相同点**：
  - 都旨在提高模型对数值部分的准确性。
  - 不需要对模型结构进行大的修改。

- **不同点**：

| 特点 | 教师强制 | 增加损失权重 |
| --- | --- | --- |
| **训练方式** | 使用真实的目标token作为输入 | 对数值位置的损失赋予更高权重 |
| **对误差的处理** | 减少了由前一时间步错误累积的误差 | 增加数值部分错误的惩罚力度 |
| **潜在问题** | 训练和推理阶段不一致（Exposure Bias） | 可能导致过拟合或训练不稳定 |

#### **1.5 建议**

- **结合使用**：实际上，可以将两种方法结合起来使用。在训练时使用教师强制，同时对数值部分的损失赋予更高的权重，以同时利用两种方法的优势。

- **实验验证**：建议您在实际应用中分别或组合使用这两种方法，并通过实验验证哪种方法对您的模型效果更好。

---

### **问题2：标签平滑（Label Smoothing）具体怎么操作？**

#### **2.1 标签平滑概述**

**标签平滑（Label Smoothing）**是一种正则化技术，旨在防止模型过度自信地输出某个类别，从而提高模型的泛化能力。它通过在训练时，将目标分布从one-hot分布转换为一个平滑的分布，即为非正确类别分配一个小的概率。

#### **2.2 在您的任务中的应用**

针对数值预测，您希望模型不仅能预测正确的数值token，还能理解数值之间的大小关系。通过标签平滑，您可以在目标概率分布中，为正确的数值token分配最大的概率，同时为相邻的数值token分配较小的概率，以反映数值之间的连续性。

#### **2.3 具体操作步骤**

**假设**：

- 数值token的词汇表为`[0, 1, 2, ..., 9]`（或更大的范围）。
- 您的目标数值是`t`，是一个整数。

**标签平滑的操作**：

1. **定义平滑因子**：设定一个平滑因子`ε`，通常是一个小的值，例如`ε=0.1`。

2. **计算平滑后的目标分布**：

   - 对于正确的数值token`t`，目标概率为：`p_t = 1 - ε`。
   - 对于与`t`相邻的数值token，例如`t-1`和`t+1`，目标概率为：`p_{t-1} = p_{t+1} = ε/2`。
   - 对于其他数值token，目标概率为`0`。

3. **计算损失**：

   - 使用交叉熵损失函数，将模型输出的概率分布与平滑后的目标分布进行比较。

**示例**：

- **目标数值**：`t = 5`
- **平滑因子**：`ε = 0.1`
- **目标概率分布**：

| Token | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 概率 | 0 | 0 | 0 | 0 | 0.05 | 0.9 | 0.05 | 0 | 0 | 0 |

- **解释**：正确的数值`5`的目标概率为`0.9`，相邻的数值`4`和`6`的概率为`0.05`，其他数值的概率为`0`。

#### **2.4 优点**

- **反映数值之间的连续性**：通过为相邻的数值token分配概率，模型可以学习到数值之间的大小关系。

- **减少过拟合**：标签平滑有助于防止模型过度自信，提高泛化能力。

#### **2.5 注意事项**

- **平滑因子的选择**：`ε`的值需要适当调节，过大可能导致目标分布过于平滑，模型难以收敛；过小则效果不明显。

- **数值范围的选择**：如果数值范围较大，可以考虑为更远的相邻数值分配更小的概率，以反映更广泛的数值连续性。

---

### **问题3：我是否可以使用特殊的token来代表数值，来解决不可微的问题？**

#### **3.1 使用特殊token表示数值的想法**

您的想法是引入一些特殊的token，用于表示特定的数值，希望通过这种方式绕过`argmax`和`tokenizer`解码的不可微问题。

#### **3.2 分析**

- **引入全新的数值token**：您可以为每个可能的数值创建一个单独的token，将其加入词汇表。例如，`<NUM_0>`, `<NUM_1>`, ..., `<NUM_N>`，其中`N`是最大可能的数值。

- **模型的输出**：模型在输出时，直接预测这些特殊的数值token，不需要进行额外的解析或转换。

- **优势**：

  - **避免了解码过程**：模型直接在词汇表中预测数值token，避免了需要解析数字字符串的问题。

  - **计算损失时可微**：由于模型的输出仍然是一个softmax概率分布，对应于词汇表中的每个token，计算损失函数时依然是可微的。

#### **3.3 能否解决不可微的问题**

- **部分解决**：使用特殊的数值token，可以避免对模型输出进行`argmax`和`tokenizer`解码，从而使得损失计算过程保持可微。

- **但仍然存在的问题**：

  - **数值的大小关系**：即使使用特殊的数值token，模型仍然可能将不同的数值视为离散的符号，无法理解数值之间的连续性和大小关系。

  - **词向量的学习**：模型需要从数据中学习这些数值token的嵌入表示，可能需要大量的数据才能学到数值之间的关系。

#### **3.4 可能的改进**

- **嵌入层的设计**：在设计数值token的嵌入时，可以为它们赋予反映数值大小的特征。例如，可以使用数值大小作为嵌入向量的某个维度，或者在嵌入空间中保持数值token的排序。

- **结合数值嵌入和文本嵌入**：您可以为数值部分单独设计一个数值嵌入模块，与文本嵌入分开处理，这样模型可以更好地捕捉数值的信息。

- **模型结构的调整**：如果可能，可以在模型中增加处理数值信息的子模块，例如专门的数值预测头。

#### **3.5 实践建议**

- **扩展词汇表**：在词汇表中添加数值token，确保模型能够直接预测这些数值。

- **数据预处理**：在数据集中，将数值部分替换为对应的数值token。例如，将`"C_count": 5`替换为`"C_count": <NUM_5>`。

- **损失函数的设计**：继续使用交叉熵损失函数，但可以结合前面提到的标签平滑或增加数值位置的损失权重等方法，增强模型对数值token的学习。

- **实验验证**：需要通过实验来验证这种方法是否有效，观察模型对数值预测的准确性和稳定性。

---

### **综合总结**

**针对您的三个问题**：

1. **教师强制 vs. 增加损失权重**：教师强制通过使用真实的目标token作为输入，帮助模型更快地学习正确的输出；增加损失权重则是通过强调数值部分的错误，促使模型更加关注数值的准确性。这两种方法各有优缺点，可以结合使用。

2. **标签平滑的具体操作**：标签平滑可以通过为正确的数值token分配高概率，为相邻的数值token分配较小的概率来实现。这有助于模型理解数值之间的大小关系，提高数值预测的准确性。

3. **使用特殊的数值token**：引入特殊的数值token，可以部分解决不可微的问题，使损失计算保持可微。然而，模型仍需要学习数值之间的关系，可以通过设计嵌入层和调整模型结构来增强模型对数值的理解。

---

### **进一步的建议**

- **综合运用多种方法**：您可以尝试将以上方法结合起来，包括使用特殊的数值token、标签平滑、教师强制和增加损失权重，以促进模型更好地学习数值信息。

- **模型结构的改进**：如果条件允许，考虑对模型进行适当的修改，例如增加专门处理数值的模块或回归头。

- **实验与验证**：在实际应用中，通过实验来验证不同方法的效果非常重要。您可以设置对照实验，观察各方法对模型性能的影响。

- **数据的丰富性**：确保训练数据中包含足够丰富和多样的数值信息，帮助模型更好地学习数值部分的模式。

---

希望以上解答能够帮助您更好地理解这些方法，并在您的项目中取得成功！
```
```
如果我给特殊token的值也是大小有序的, 比如<NUM_0>对应15600, <NUM_1>对应15601, 以此类推, 是否可以解决 "即使使用特殊的数值token，模型仍然可能将不同的数值视为离散的符号，无法理解数值之间的连续性和大小关系" 这个问题
 
 
------
 
您好！

非常感谢您提出的进一步问题。让我来仔细分析您的想法，并解释是否这种方法可以解决模型将不同数值视为离散符号、无法理解数值之间连续性和大小关系的问题。

---

### **您的问题**

**如果我给特殊token的值也是大小有序的，比如`<NUM_0>`对应15600，`<NUM_1>`对应15601，以此类推，是否可以解决模型将不同数值视为离散符号，无法理解数值之间的连续性和大小关系的问题？**

---

### **分析与解答**

#### **1. 理解Token ID和嵌入层的作用**

在Transformer模型（如BERT、GPT等）中，**Token ID**和**嵌入（Embedding）**的作用如下：

- **Token ID**：每个词汇表中的token（词或符号）都有一个唯一的ID，这个ID用于索引嵌入矩阵。ID本身在模型计算过程中不携带任何语义信息，仅作为查找的索引。
  
- **嵌入矩阵**：将Token ID映射到高维实数向量空间的矩阵。模型在训练过程中，会学习嵌入向量，使得具有相似语义的token的嵌入向量在向量空间中接近。

**关键点**：模型在处理时实际上是使用嵌入向量，Token ID只是用于查找对应的嵌入向量。**Token ID的数值大小本身不会对模型的理解产生影响**。

#### **2. Token ID的数值大小不会影响模型的数值理解**

即使您将特殊的数值token的ID按照数值大小顺序分配，例如：

- `<NUM_0>`的ID为15600
- `<NUM_1>`的ID为15601
- 依此类推

然而，在模型的计算过程中，**Token ID仅作为索引，不参与数值计算**。模型并不知道ID的数值大小，也不会利用ID的数值来理解token之间的关系。

**因此，仅仅将Token ID按照数值大小顺序分配，并不能帮助模型理解数值之间的连续性和大小关系**。

#### **3. 嵌入向量的学习才是关键**

模型对token的理解主要来自于嵌入向量的表示，这些嵌入向量是在训练过程中学习的。

- **如果嵌入向量能够反映数值的大小关系**，那么模型就可以理解数值之间的连续性。
- **如果嵌入向量没有反映数值的大小关系**，那么模型仍然会将不同的数值token视为离散的符号。

#### **4. 如何让嵌入向量反映数值大小关系**

要解决模型无法理解数值之间连续性的问题，需要让数值token的嵌入向量能够反映数值的大小关系。以下是一些可能的解决方案：

##### **4.1 手动初始化嵌入向量**

- **方法**：在模型初始化时，为数值token的嵌入向量赋予特定的数值信息。例如，让嵌入向量中的某一维度或某几维度直接对应数值的大小，或者按数值大小顺序初始化嵌入向量，使得数值大小与嵌入向量的距离或方向相关。
  
- **效果**：这种方法可以使得数值token的嵌入向量在向量空间中按照数值大小排列，模型可以更容易地学习到数值之间的大小关系。

- **挑战**：在训练过程中，模型可能会调整嵌入向量，破坏初始的排列。需要在训练中加入约束，保持数值嵌入的顺序性。

##### **4.2 使用数值编码方法**

- **方法**：使用特定的数值编码方法，将数值转换为向量表示。例如，使用位置编码（Positional Encoding）的方式，将数值映射到连续的向量空间。

- **示例**：对于数值`n`，可以计算`sin(n)`和`cos(n)`等函数，将结果作为嵌入向量的一部分。

- **效果**：数值之间的大小关系可以反映在嵌入向量的差异中，模型可以学习到数值的连续性。

##### **4.3 修改模型结构**

- **方法**：在模型中引入专门处理数值的模块，例如：

  - **数值嵌入层**：为数值token设计专门的嵌入层，直接将数值映射为连续的数值表示。

  - **回归头（Regression Head）**：在模型的输出层，为数值部分增加回归头，直接预测连续的数值。

- **效果**：模型可以直接处理数值信息，理解数值之间的大小关系。

##### **4.4 在损失函数中引入数值约束**

- **方法**：在计算损失时，对数值部分引入额外的约束或正则化项。例如：

  - **排序损失（Ranking Loss）**：确保数值嵌入向量的顺序与数值大小一致。

  - **MSE损失**：对于数值部分，使用均方误差损失，使得模型输出的嵌入向量可以预测实际的数值。

---

### **总结**

**结论**：仅仅通过将特殊token的ID按照数值大小顺序分配，无法解决模型将不同数值视为离散符号、无法理解数值之间连续性和大小关系的问题。

原因在于：

- **Token ID的数值不参与模型计算**：ID只是用于查找嵌入向量，模型不会利用ID的数值大小来理解token的含义。

- **嵌入向量决定了模型对token的理解**：需要在嵌入向量中引入数值大小的信息，模型才能理解数值之间的关系。

---

### **建议**

为了让模型理解数值的连续性和大小关系，您可以考虑以下方法：

#### **1. 调整嵌入层**

- **为数值token设计专门的嵌入方式**，例如：

  - **数值映射**：将数值直接映射为嵌入向量的某一维度。

  - **位置编码**：使用正弦和余弦函数编码数值，类似于Transformer中的位置编码。

- **确保数值嵌入向量的顺序性**：在训练过程中，保持数值嵌入向量按照数值大小有序。

#### **2. 修改模型结构**

- **增加回归模块**：在模型中增加用于数值预测的回归头，直接输出连续的数值。

- **混合输出**：对于数值部分，模型输出连续的数值；对于文本部分，模型输出离散的token。

#### **3. 调整训练目标**

- **使用适当的损失函数**：

  - **MSE损失**：针对数值部分，使用MSE损失，使得模型能够准确预测数值。

  - **排序损失**：引入损失项，确保数值嵌入的顺序性。

- **数据增强**：在训练数据中，提供数值之间关系的示例，帮助模型学习数值的连续性。

#### **4. 结合数值和文本信息**

- **混合嵌入**：将数值信息和文本信息结合起来，形成综合的嵌入向量。

- **多任务学习**：同时训练模型的文本生成和数值预测任务，促进模型在两个方面的理解。

---

### **举例说明**

**示例1：数值嵌入映射**

- **嵌入方式**：将数值`n`映射为一个向量`[f(n)]`，其中`f(n)`是一个数值编码函数，如归一化的数值、对数等。

- **嵌入向量**：对于数值token`<NUM_n>`，其嵌入向量为`[Embedding_text, Embedding_number]`，其中`Embedding_text`是普通的词嵌入，`Embedding_number`是数值嵌入。

- **模型处理**：模型在处理时，可以同时利用文本和数值信息。

**示例2：模型输出连续数值**

- **模型结构**：在模型的输出层，对于数值部分，增加一个回归头，直接输出数值。

- **损失计算**：对于数值部分，使用MSE损失；对于文本部分，使用交叉熵损失。

- **优势**：模型可以直接预测数值，避免了数值token的离散化问题。

---

### **回答您的问题**

**仅仅通过将特殊token的ID按照数值大小顺序分配，无法解决模型无法理解数值连续性的问题。**

**原因**：

- Token ID的数值大小不影响模型的理解。

- 模型需要在嵌入层和结构上做出调整，才能理解数值之间的关系。

---

### **建议的改进方向**

- **调整嵌入层**：确保数值嵌入向量能够反映数值的大小关系。

- **修改模型结构**：增加处理数值信息的模块，直接预测数值。

- **调整损失函数**：使用MSE损失、排序损失等，增强模型对数值的理解。

- **数据处理**：在训练数据中，提供数值关系的示例，帮助模型学习。

---

### **最后的思考**

处理数值信息是自然语言处理中的一个挑战，特别是在使用预训练的大型语言模型时。模型往往倾向于将数值视为独立的符号，缺乏对数值大小和连续性的理解。

因此，**需要在模型的嵌入层、结构、损失函数和训练数据上做出针对性的设计和调整**，才能使模型更好地理解和处理数值信息。

希望以上解答对您有所帮助！ 
``` 

# 尝试采用softmax, 解决不可微的问题

找到0-1000对应的token:

```
                # 测试 0 到 1000 范围内的数字
                test_numbers = [str(i) for i in range(1001)]  # 生成 0 到 1000 的数字字符串

                # 存储所有生成的 token IDs
                all_token_ids = set()

                # 编码数字字符串并收集 token IDs
                for number in test_numbers:
                    token_ids = tokenizer.encode(number, add_special_tokens=False)
                    all_token_ids.update(token_ids)  # 将 token_ids 添加到集合中（避免重复）

                # 找到最小值和最大值
                min_token_id = min(all_token_ids)
                max_token_id = max(all_token_ids)

                print(f"all_token_ids: {all_token_ids}")
                print(f"Token ID Range for numbers 0-1000: {min_token_id} to {max_token_id}")

```

```
all_token_ids: {15, 16, 17, 18, 19, 20, 21, 22, 23, 24}
Token ID Range for numbers 0-1000: 15 to 24
``` 

脚本: 

[附件: loss.numbric_loss.py] 

重点: 需要保障梯度传递, 使用了softmax, 使用了原始的logits进行计算, 才能保障梯度传递.

不能使用argmax+decode来获取原文, 然后再decode的结果上计算, 这样不能传递梯度.

训练曲线: 

![image2024-12-10 17:58:20.png](/assets/01KJBZNKZMMHGYY4D6PE7BBQMW/image2024-12-10%2017%3A58%3A20.png)

过拟合

评估: 使用两组数据进行demo: (selfies部分没有计算loss, 所以会无限输出, 速度很慢)

![image2024-12-10 17:57:5.png](/assets/01KJBZNKZMMHGYY4D6PE7BBQMW/image2024-12-10%2017%3A57%3A5.png)

可以看到有部分错误, 但整体在收敛. 

# 将 selfies_loss 和 数值loss 加和, 进行训练

两个修改: 

  - 将 selfies_loss 和 数值loss 加和
  - 对于json无法解析的情况, 使用标准交叉熵, 乘以惩罚倍数 (5)

训练结果: 

![image2024-12-10 19:19:38.png](/assets/01KJBZNKZMMHGYY4D6PE7BBQMW/image2024-12-10%2019%3A19%3A38.png)

出现的问题: 

  - eval/loss 比 train/loss 高40倍
  - 在评估过程中, 能正确生成的selfies表达式很少

num_items_in_batch的意义: 参考函数, 其是用于得出num_items_in_batch: 

```
def get_batch_samples(self, epoch_iterator, num_batches):
        batch_samples = []
        num_items_in_batch = None
        for _ in range(num_batches):
            try:
                batch_samples += [next(epoch_iterator)]
            except StopIteration:
                break

        # Keep default behavior the same
        if not self.model_accepts_loss_kwargs:
            return batch_samples, None

        if len(batch_samples) > 0 and "labels" in batch_samples[0]:
            # For now we don't support object detection
            try:
                num_items_in_batch = sum([(batch["labels"].ne(-100)).sum() for batch in batch_samples])
            except (TypeError, AttributeError):
                pass

        if self.args.average_tokens_across_devices:
            num_items_in_batch = self.accelerator.gather(num_items_in_batch).sum().item()
        return batch_samples, num_items_in_batch
``` 

得到num_items_in_batch的逻辑是: 一个训练batch中, 所有数据的labels的有效token数总和

在loss函数的计算中, 得到的一个数据的loss需要除以num_items_in_batch, 然后返回. 这样对于一个batch的所有数据的 loss函数返回值 相加, 相当于 (所有数据的loss 除以 所有数据的labels的有效token数), 意义是 平均每个有效token的损失. 进行基于batch的归一化.

而在评估时, 不存在batch概念, 而是一个一个数据进行评估, 那么loss就应当除以当前数据的有效token数, 评估N组数据后取平均值. 

所以loss函数应修改: 

![image2024-12-10 20:6:23.png](/assets/01KJBZNKZMMHGYY4D6PE7BBQMW/image2024-12-10%2020%3A6%3A23.png)

修复脚本: [loss.fix_normalize.py](/assets/01KJBZNKZMMHGYY4D6PE7BBQMW/loss.fix_normalize.py)

重新训练, 会发现现象: 

  - 初期的时候, 输出不能被解析成json, 所以loss会很高: 
    - `TEST: ce_loss.sum()=11.168405532836914, num_items_in_batch=1328.0`
    - 因为ce_loss是整个输出字符串的loss
  - 初期的时候, train_loss会高于eval_loss, 是因为评估过程中会生成更多的合法的json字符串, 训练到后期这个现象会消失

训练脚本

```
## train

cp /opt/huangyan/ms-swift/swift/llm/sft.py.raw /opt/huangyan/ms-swift/swift/llm/sft.py

bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0 NPROC_PER_NODE=1 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out.mol_desc/selected_data.length_1_4.500.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out.mol_desc/length_4.ms_swift.val.json \
--eval_steps 10 \
--learning_rate 4e-4 \
--num_train_epochs 10 \
--logging_steps 10 \
--loss_name json_selfies_loss \
" > train_2.log 2>&1

last_model_checkpoint=$(awk -F ': ' '/last_model_checkpoint/ {print $2}' train_2.log)
best_model_checkpoint=$(awk -F ': ' '/best_model_checkpoint/ {print $2}' train_2.log)

echo "last_model_checkpoint: ${last_model_checkpoint}"
echo "best_model_checkpoint: ${best_model_checkpoint}"

sleep 60

## evaluate 
cp /opt/huangyan/ms-swift/swift/llm/sft.py.raw /opt/huangyan/ms-swift/swift/llm/sft.py
 
python3 /opt/huangyan/ms-swift/huangyan_test/evaluate_loop_selfies/evaluate_loop_selfies.py \
    ${last_model_checkpoint} \
    /opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out.mol_desc/length_4.ms_swift.val.json \
    --limit 300 1>evaluate_2.last_model.log 2>&1

sleep 60
 
python3 /opt/huangyan/ms-swift/huangyan_test/evaluate_loop_selfies/evaluate_loop_selfies.py \
    ${best_model_checkpoint} \
    /opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out.mol_desc/length_4.ms_swift.val.json \
    --limit 300 1>evaluate_2.best_model.log 2>&1

sleep 60

######

``` 

效果:

  - v312: LR=4e-4, epoch=10
  - v313: LR=8e-4, epoch=20
  - v314: LR=8e-4, epoch=40

![image2024-12-11 10:6:58.png](/assets/01KJBZNKZMMHGYY4D6PE7BBQMW/image2024-12-11%2010%3A6%3A58.png)

v312 和 v313 的loss和acc的结果差不多, v314的loss和acc稍差. 但评估效果有差异: 

  - v312:
    - 会产生很多超长的非法selfies, 类似于"[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ ""...
    - 比较数字字段: last_model 比 best_model 好,, 只有四个case的数字字段错误.
  - v313:
    - 不会产生超长的selfies, 但selfies大部分非法, 类似于: "[N][=C][][][][=C]". 
    - 比较数字字段: last_model 比 best_model 好, 只有四个case的数字字段错误.
    - 会存在数字字段和selfies的元素个数不一样, 比如: 
      - `{"single_bonds": 2, "double_bonds": 0, "triple_bonds": 1, "ring_count": 0, "C_count": 1, "O_count": 2, "N_count": 0, "branch_count": 0, "atom_count": 3, "selfies": "[O][C][###OOO]"}`
  - v314: 
    - 特点同v313

分析: 对于selfies生成, 为什么v313比v312效果更好 (loss差不多), 分析训练日志中的loss日志: 

去最后一组batch的loss日志:

```
v312: 
TEST: selfies_loss=0.8889110088348389, numeric_loss=0.07483845204114914, total_loss=0.9637494683265686, num_items_in_batch=1322.0
TEST: selfies_loss=0.8746189475059509, numeric_loss=0.31682151556015015, total_loss=1.191440463066101, num_items_in_batch=1322.0
TEST: selfies_loss=0.8664361834526062, numeric_loss=0.22252027690410614, total_loss=1.0889564752578735, num_items_in_batch=1322.0
TEST: selfies_loss=0.7579860687255859, numeric_loss=0.2233933061361313, total_loss=0.9813793897628784, num_items_in_batch=1322.0
TEST: ce_loss.sum()=7.2562761306762695, num_items_in_batch=1322.0
TEST: selfies_loss=0.8491185903549194, numeric_loss=0.5272004008293152, total_loss=1.3763189315795898, num_items_in_batch=1322.0
TEST: selfies_loss=0.9117059111595154, numeric_loss=0.22378642857074738, total_loss=1.1354923248291016, num_items_in_batch=1322.0
TEST: selfies_loss=0.8779617547988892, numeric_loss=0.2969735860824585, total_loss=1.1749353408813477, num_items_in_batch=1322.0
TEST: ce_loss.sum()=6.639133453369141, num_items_in_batch=1322.0
TEST: selfies_loss=0.9308701753616333, numeric_loss=0.22222746908664703, total_loss=1.1530976295471191, num_items_in_batch=1322.0
TEST: selfies_loss=0.8312338590621948, numeric_loss=1.1879029273986816, total_loss=2.019136905670166, num_items_in_batch=1322.0
TEST: selfies_loss=0.8293998837471008, numeric_loss=0.3824135959148407, total_loss=1.2118134498596191, num_items_in_batch=1322.0
TEST: selfies_loss=0.8104195594787598, numeric_loss=0.30451884865760803, total_loss=1.1149383783340454, num_items_in_batch=1322.0
TEST: selfies_loss=0.7927178740501404, numeric_loss=0.23258812725543976, total_loss=1.025305986404419, num_items_in_batch=1322.0
TEST: selfies_loss=0.942640483379364, numeric_loss=0.2996349036693573, total_loss=1.242275357246399, num_items_in_batch=1322.0
TEST: selfies_loss=0.8737549185752869, numeric_loss=0.23024334013462067, total_loss=1.1039983034133911, num_items_in_batch=1322.0

 
---
 
v313: 
TEST: selfies_loss=0.7153105139732361, numeric_loss=0.297091007232666, total_loss=1.0124015808105469, num_items_in_batch=1309.0
TEST: selfies_loss=0.9187151789665222, numeric_loss=0.22259320318698883, total_loss=1.1413084268569946, num_items_in_batch=1309.0
TEST: selfies_loss=0.7612434029579163, numeric_loss=0.2969760596752167, total_loss=1.0582194328308105, num_items_in_batch=1309.0
TEST: selfies_loss=0.7899219989776611, numeric_loss=0.2981487810611725, total_loss=1.0880707502365112, num_items_in_batch=1309.0
TEST: selfies_loss=0.6898037791252136, numeric_loss=0.014412100426852703, total_loss=0.7042158842086792, num_items_in_batch=1309.0
TEST: selfies_loss=0.8445447683334351, numeric_loss=0.07529500126838684, total_loss=0.9198397397994995, num_items_in_batch=1309.0
TEST: ce_loss.sum()=4.129866600036621, num_items_in_batch=1309.0
TEST: selfies_loss=0.8242271542549133, numeric_loss=0.3033217787742615, total_loss=1.1275489330291748, num_items_in_batch=1309.0
TEST: selfies_loss=0.768479585647583, numeric_loss=0.5217519998550415, total_loss=1.2902315855026245, num_items_in_batch=1309.0
TEST: selfies_loss=0.7241959571838379, numeric_loss=0.22556383907794952, total_loss=0.9497597813606262, num_items_in_batch=1309.0
TEST: selfies_loss=0.8567250967025757, numeric_loss=0.6669949293136597, total_loss=1.5237200260162354, num_items_in_batch=1309.0
TEST: selfies_loss=0.7331029772758484, numeric_loss=0.29838424921035767, total_loss=1.031487226486206, num_items_in_batch=1309.0
TEST: selfies_loss=0.7300705909729004, numeric_loss=0.5226706266403198, total_loss=1.2527412176132202, num_items_in_batch=1309.0
TEST: selfies_loss=0.9396170377731323, numeric_loss=0.2966736853122711, total_loss=1.236290693283081, num_items_in_batch=1309.0
TEST: selfies_loss=0.8976741433143616, numeric_loss=0.6704689264297485, total_loss=1.5681431293487549, num_items_in_batch=1309.0
TEST: selfies_loss=0.9907329082489014, numeric_loss=0.30029430985450745, total_loss=1.2910271883010864, num_items_in_batch=1309.0
 
---
 
v314:
TEST: ce_loss.sum()=8.835972785949707, num_items_in_batch=1320.0
TEST: ce_loss.sum()=7.913494110107422, num_items_in_batch=1320.0
TEST: selfies_loss=1.1131606101989746, numeric_loss=0.7138765454292297, total_loss=1.8270370960235596, num_items_in_batch=1320.0
TEST: selfies_loss=1.0724847316741943, numeric_loss=0.3031390309333801, total_loss=1.3756237030029297, num_items_in_batch=1320.0
TEST: selfies_loss=0.8644635081291199, numeric_loss=0.5210970640182495, total_loss=1.3855605125427246, num_items_in_batch=1320.0
TEST: selfies_loss=1.0892177820205688, numeric_loss=0.3427719473838806, total_loss=1.4319896697998047, num_items_in_batch=1320.0
TEST: selfies_loss=0.7037408947944641, numeric_loss=0.675991952419281, total_loss=1.3797328472137451, num_items_in_batch=1320.0
TEST: selfies_loss=0.8499117493629456, numeric_loss=0.29886144399642944, total_loss=1.148773193359375, num_items_in_batch=1320.0
TEST: selfies_loss=0.9609934687614441, numeric_loss=0.30122390389442444, total_loss=1.262217402458191, num_items_in_batch=1320.0
TEST: selfies_loss=1.1176273822784424, numeric_loss=0.11838001757860184, total_loss=1.2360074520111084, num_items_in_batch=1320.0
TEST: selfies_loss=0.7694374322891235, numeric_loss=0.6748235821723938, total_loss=1.444261074066162, num_items_in_batch=1320.0
TEST: selfies_loss=1.1191611289978027, numeric_loss=0.2983955144882202, total_loss=1.417556643486023, num_items_in_batch=1320.0
TEST: selfies_loss=0.9951404929161072, numeric_loss=0.5203248262405396, total_loss=1.515465259552002, num_items_in_batch=1320.0
TEST: ce_loss.sum()=8.008184432983398, num_items_in_batch=1320.0
TEST: selfies_loss=0.6095992922782898, numeric_loss=0.6683732867240906, total_loss=1.2779725790023804, num_items_in_batch=1320.0
TEST: selfies_loss=1.017974853515625, numeric_loss=0.30132991075515747, total_loss=1.3193047046661377, num_items_in_batch=1320.0

``` 

看统计值:

  - v313中, 非法的json更少, "TEST: ce_loss.sum()=" 对整体的影响较小,
  - v312中, "TEST: ce_loss.sum()=" 对整体loss影响更大, 导致模型无法学习selfies的细节
  - v314中, "TEST: ce_loss.sum()=" 对整体loss影响更大, 导致模型无法学习selfies的细节

结论: 现在的loss方式, 受到"非法json格式"的影响过大, 需要解决"让模型稳定输出json格式"的问题

# 需要解决微调中输出json格式的问题 - 在样例数据中增加few-shot

prompt修改: 

```
Print mol descriptions (atom count, single_bonds count, double_bonds count, triple_bonds count, ring count, C/O/N count, branch count) and selfies of the image <image>, in json format. 
Output json format example:
<output_json_format>{"single_bonds": 0, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 0, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 1, "selfies": "[N]"}</output_json_format>
<output_json_format>{"single_bonds": 0, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 0, "N_count": 0, "branch_count": 0, "atom_count": 1, "selfies": "[C]"}</output_json_format>
``` 

并将非法的json字符串打印出来. 检验结果, 发现"非法json"并不是因为json结构遭到破坏, 而是因为selfies表达式中混入了双引号, 形成了类似下面这种表达式: 

```
"selfies": "[ "[][][][=C][O]"
``` 

另外的现象: 生成的selfies表达式中, 会有大量的空括号 和 一个括号中多元素或多键, 类似于: 

```
"[C][][][][][][][][][][][][][NN][][][][][][][][CC]"
"[OO][N][===C][][NNN]"
``` 

以上两种现象, 在没有增加few-shot时并未验证是否存在, 所以不能说few-shot是否起到了作用. 

提示词中: 增加对selfies结构的说明: 

````
Print mol descriptions (atom count, single_bonds count, double_bonds count, triple_bonds count, ring count, C/O/N count, branch count) and selfies of the image <image>, in json format. 
<SELFIES Expression Construction Rules>

1. Basic Symbol Rules:
- SELFIES must start with a single atom symbol, each symbol enclosed in square brackets []
- Atom symbols must be valid chemical elements, such as [C], [N], [O]
- Empty brackets [] or consecutive empty brackets [][][] are not allowed

2. Bond Representation:
- Single bonds require no special marking
- Double bonds use [=]
- Triple bonds use [#]

3. Ring and Branch Operations:
- Ring operators are denoted as [Ring1], [Ring2], etc., where numbers indicate ring size
- Branch operators are denoted as [Branch1], [Branch2], etc., where numbers indicate branch length
- Ring operators must appear in pairs to form complete ring structures
- Branches must have clear starting and ending points

4. Structural Integrity:
- Expression must form a complete and chemically valid molecular structure
- All atoms must not exceed their maximum valence
- Ring and branch structures must be properly closed
- No dangling bonds or incomplete ring structures allowed

Correct examples:
```
[C][=O][C]           # Simple chain structure
[C][C][Ring1][O][C]  # Structure with ring
[C][Branch1][C][O][C][Branch1][C]  # Structure with branch
```

Incorrect examples:
```
[C][Ring1][O]        # Unclosed ring
[C][Branch1][C]      # Incomplete branch
[Ring2][C][O]        # Ring operator at start
```
</SELFIES Expression Construction Rules>

Output json format example:
<output_json_format>{"single_bonds": 0, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 0, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 1, "selfies": "[N]"}</output_json_format>
<output_json_format>{"single_bonds": 0, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 0, "N_count": 0, "branch_count": 0, "atom_count": 1, "selfies": "[C]"}</output_json_format>

```` 

训练效果: v322

[图片: image2024-12-13 23:42:10.png]

评估结果: 

  - 现象没有改善, 出现大量非法selfies表达式, 类似于: 

```
"[ "[ "[C][==CCC][][][NNNN]"
```

  - 甚至变得更糟糕, 出现了超长非法表达式: 

```
[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[ "[
```

改进方向: 让提示词中带有selfies表达式的正例和反例: v328

```
Analyze the molecular structure in the provided image. Generate a JSON object containing the following information:

*   **atom_count**: The total number of atoms in the molecule.
*   **single_bonds_count**: The number of single bonds in the molecule.
*   **double_bonds_count**: The number of double bonds in the molecule.
*   **triple_bonds_count**: The number of triple bonds in the molecule.
*   **ring_count**: The number of rings in the molecule.
*   **C_count**: The number of carbon (C) atoms in the molecule.
*   **O_count**: The number of oxygen (O) atoms in the molecule.
*   **N_count**: The number of nitrogen (N) atoms in the molecule.
*   **branch_count**: The number of branches in the molecule (a branch is defined as a non-linear chain of atoms).
*   **selfies**: A valid SELFIES (Simplified Molecular-Input Line-Entry System) representation of the molecule. Ensure the generated SELFIES string is syntactically correct and accurately represents the molecular structure.

**Important:** Pay close attention to the rules of SELFIES notation.

**Examples:**

*   **Correct SELFIES:**
    *   Methane: `[C]`
    *   Ethane: `[C][C]`
    *   Ethene: `[C][=C]`
    *   Ethyne: `[C][#C]`
    *   Propane: `[C][C][C]`
    *   Butane: `[C][C][C][C]`
    *   Isobutane: `[C][C]([C])[C]`
    *   Cyclohexane: `[C]1[C][C][C][C][C]1`
    *   Ethanol: `[C][C][O]`
    *   Ammonia: `[N]`
    *   Water: `[O]`
    *   Carbon Dioxide: `[O][=C]=[O]`
*   **Incorrect SELFIES (Do not generate these):**
    *   `[CC][][C][][][#CC]` (Incorrect bond representation)
    *   `[NN][][][][O]` (Incorrect bond representation)
    *   `[ "[ "[ "[CC][==N]` (Incorrect syntax)
    *   `[CC][##C][CCCCC]` (Incorrect bond representation)
    *   `[OOOC][=CCC][][=NNNNN]` (Incorrect bond representation and syntax)
    *   `[C]1[C][C][C][C][C]` (Missing closing ring bond)
    *   `[C][C][C][C][O]1` (Incorrect ring bond placement)

**Rules:**
    *   All atoms are enclosed in square brackets, e.g., `[C]`, `[O]`, `[N]`.
    *   All bonds are represented by symbols: `[=]` for double bond, `[#]` for triple bond, and no symbol for single bond.
    *   Branches are represented by parentheses, e.g., `[C]([C])[C]`.
    *   Rings are represented by digits, e.g., `[1]`, `[2]`, and the corresponding opening and closing ring bonds.
    *   Ensure that the number of bonds for each atom is correct based on its valency.

Provide the output in a single, well-formatted JSON object.

Image:
<image>

Output example:
{"single_bonds": 0, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 0, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 1, "selfies": "[N]"}

``` 

训练效果: (跟v322相比, loss更好)

![image2024-12-14 22:4:34.png](/assets/01KJBZNKZMMHGYY4D6PE7BBQMW/image2024-12-14%2022%3A4%3A34.png)

评估效果: 

比v322的效果更差, 出现了多个超长的无效selfies串

# 思考

模型到底从哪里学习到了 "[]"这种知识, 训练数据中并没有

猜测: 

  1. 使用分段式的loss (selfies和数值的loss分别计算), 使其变成了两个独立的任务 (在评估中缺失发现原子数 和 selfies中的原子数 不同)
  2. 仅计算selfies的交叉熵loss, 使得模型认为 "越短" 的符号越不容易犯错, "[]" 就变成了最简单且最不容易犯错的符号

对比不修改loss, 使用传统交叉熵的情况. 训练结果: v330. 评估结果: 

```
last model: 
 
Overall Similarity: 97.69%
Correct: 73 / 75 = 97.33333333333334 %
                                                        Image Path                                                                                                                                                                               Predicted Predicted SMILES                                                                                                                                                                                      GT GT SMILES  Similarity
55  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/317.png    
	{"single_bonds": 2, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 0, "N_count": 3, "branch_count": 0, "atom_count": 4, "selfies": "[N][=N][N][C]"}            N=NNC    
	{"single_bonds": 2, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 0, "N_count": 3, "branch_count": 0, "atom_count": 4, "selfies": "[N][N][=N][C]"}     NN=NC    0.133333
1    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/72.png    
	{"single_bonds": 2, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 0, "N_count": 3, "branch_count": 0, "atom_count": 4, "selfies": "[N][=N][N][C]"}            N=NNC    
	{"single_bonds": 2, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 0, "N_count": 3, "branch_count": 0, "atom_count": 4, "selfies": "[C][N][=N][N]"}     CN=NN    0.133333

------
 
best_model:
Overall Similarity: 87.01%
Correct: 63 / 75 = 84.0 %

...
``` 

改 loss, 将数值loss附加在整个loss之上: 

```
total_loss = 0.8 * ce_loss + numeric_loss
``` 

评估效果轻微下降: 

```
Overall Similarity: 96.71%
Correct: 72 / 75 = 96.0 %
                                                        Image Path                                                                                                                                                                               Predicted Predicted SMILES                                                                                                                                                                                      GT GT SMILES  Similarity
60  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/314.png    
	{"single_bonds": 1, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 4, "selfies": "[N][N][C][=C]"}            NNC=C   
	{"single_bonds": 1, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 4, "selfies": "[N][N][=C][=C]"}    NN=C=C    0.125000
46  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/313.png    
	{"single_bonds": 2, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 0, "N_count": 3, "branch_count": 0, "atom_count": 4, "selfies": "[N][N][C][=N]"}            NNC=N    
	{"single_bonds": 2, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 0, "N_count": 3, "branch_count": 0, "atom_count": 4, "selfies": "[N][N][=C][N]"}     NN=CN    0.133333
32  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/119.png   
	{"single_bonds": 1, "double_bonds": 0, "triple_bonds": 2, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 0, "branch_count": 0, "atom_count": 4, "selfies": "[C][#C][C][#C]"}           C#CC#C   
	{"single_bonds": 1, "double_bonds": 1, "triple_bonds": 1, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 0, "branch_count": 0, "atom_count": 4, "selfies": "[C][#C][C][=C]"}    C#CC=C    0.272727

``` 

出现了一个对双键/三键的判断错误

在长度4的训练过程中, 无法判断数值loss是否有正向作用 (在不使用数值loss的情况下, 并没有出现数值判断错误), 只知道其让评估中多出一组数值判断错误

进行长度5, 长度6的训练+评估, 判断数值loss是否有正向作用

# 长度5的数据的训练

```
cp /opt/huangyan/ms-swift/swift/llm/sft.py.raw /opt/huangyan/ms-swift/swift/llm/sft.py

bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=1  PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True \
swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out.mol_desc/selected_data.length_1_4.500.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out.mol_desc/selected_data.length_5.500.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out.mol_desc/length_5.ms_swift.val.json \
--eval_steps 50 \
--learning_rate 4e-4 \
--num_train_epochs 10 \
--logging_steps 10 \
--loss_name json_selfies_loss \
" > train_2.log 2>&1

``` 

评估结果: 其中大部分失败case, 都是原子数信息不对

```
Overall Similarity: 93.55%
Correct: 274 / 300 = 91.33333333333333 %
                                                          Image Path                                                                                                                                                                                                Predicted Predicted SMILES                                                                                                                                                                                               GT GT SMILES  Similarity
225   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/315.png              
    {"single_bonds": 2, "double_bonds": 0, "triple_bonds": 1, "ring_count": 1, "C_count": 2, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[C][N][C][#C][Ring1]"}            CNC#C  
    {"single_bonds": 2, "double_bonds": 1, "triple_bonds": 0, "ring_count": 1, "C_count": 2, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[C][N][=C][Ring1][Ring1]"}    C1N=C1    0.000000
246   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/675.png          
    {"single_bonds": 2, "double_bonds": 1, "triple_bonds": 0, "ring_count": 1, "C_count": 1, "O_count": 2, "N_count": 0, "branch_count": 0, "atom_count": 5, "selfies": "[O][=C][O][Ring1][Ring1]"}             O=CO   
    {"single_bonds": 3, "double_bonds": 0, "triple_bonds": 0, "ring_count": 1, "C_count": 1, "O_count": 2, "N_count": 0, "branch_count": 0, "atom_count": 5, "selfies": "[O][C][O][Ring1][Ring1]"}     O1CO1    0.000000
53    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/138.png             
    {"single_bonds": 2, "double_bonds": 1, "triple_bonds": 0, "ring_count": 1, "C_count": 4, "O_count": 0, "N_count": 0, "branch_count": 0, "atom_count": 4, "selfies": "[C][=C][Ring1][Ring1]"}              C#C  
    {"single_bonds": 2, "double_bonds": 0, "triple_bonds": 1, "ring_count": 1, "C_count": 3, "O_count": 0, "N_count": 0, "branch_count": 0, "atom_count": 5, "selfies": "[C][C][#C][Ring1][Ring1]"}    C1C#C1    0.000000
96   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1186.png  
    {"single_bonds": 2, "double_bonds": 0, "triple_bonds": 0, "ring_count": 1, "C_count": 2, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[C][Triangular][N][Ring1][Ring1]"}                    
    {"single_bonds": 3, "double_bonds": 0, "triple_bonds": 0, "ring_count": 1, "C_count": 2, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[N][C][C][Ring1][Ring1]"}              0.000000
289   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/979.png                 
    {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 2, "N_count": 0, "branch_count": 0, "atom_count": 5, "selfies": "[O][=C][C][C][=O]"}          O=CCC=O         
    {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 2, "N_count": 0, "branch_count": 0, "atom_count": 5, "selfies": "[O][=C][C][=C][O]"}   O=CC=CO    0.187500
22    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/718.png                 
    {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 2, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[O][=C][C][N][=O]"}          O=CCN=O         
    {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 2, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[O][C][=C][N][=O]"}   OC=CN=O    0.200000
231   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/294.png                  
    {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[N][=C][C][C][N]"}           N=CCCN         
    {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[C][N][=C][C][=N]"}   CN=CC=N    0.200000
140    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/52.png                     
    {"single_bonds": 2, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 4, "selfies": "[C][C][N][=C]"}            CCN=C          
    {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[C][C][N][C][=C]"}    CCNC=C    0.222222
273   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/386.png                    
    {"single_bonds": 1, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 4, "selfies": "[C][=C][N][=O]"}           C=CN=O         
    {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[C][=C][N][C][=O]"}   C=CNC=O    0.235294
100   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/775.png                   
    {"single_bonds": 4, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 4, "N_count": 0, "branch_count": 0, "atom_count": 5, "selfies": "[O][C][O][O][O]"}            OCOOO           
    {"single_bonds": 4, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 4, "N_count": 0, "branch_count": 0, "atom_count": 5, "selfies": "[O][O][C][O][O]"}     OOCOO    0.266667
144   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/823.png                   
    {"single_bonds": 4, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 0, "O_count": 4, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[O][N][O][O][O]"}            ONOOO           
    {"single_bonds": 4, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 0, "O_count": 4, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[O][O][N][O][O]"}     OONOO    0.266667
30    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/478.png                 
    {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[O][N][=N][C][=C]"}          ON=NC=C         
    {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[C][=N][C][=N][O]"}   C=NC=NO    0.277778
68   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1463.png                    
    {"single_bonds": 1, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 4, "selfies": "[N][=C][=C][O]"}           N=C=CO         
    {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[N][N][=C][=C][O]"}   NN=C=CO    0.294118
59    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/535.png                    
    {"single_bonds": 1, "double_bonds": 0, "triple_bonds": 2, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 0, "branch_count": 0, "atom_count": 4, "selfies": "[C][#C][C][#C]"}           C#CC#C         
    {"single_bonds": 2, "double_bonds": 0, "triple_bonds": 2, "ring_count": 0, "C_count": 5, "O_count": 0, "N_count": 0, "branch_count": 0, "atom_count": 5, "selfies": "[C][#C][C][C][#C]"}   C#CCC#C    0.300000
260   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/384.png                  
    {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 0, "N_count": 3, "branch_count": 0, "atom_count": 5, "selfies": "[N][C][N][N][=C]"}           NCNN=C          
    {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[C][=C][N][C][N]"}    C=CNCN    0.300000
129   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/173.png                     
    {"single_bonds": 2, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 2, "N_count": 0, "branch_count": 0, "atom_count": 4, "selfies": "[C][O][O][=C]"}             COOC          
    {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 3, "N_count": 0, "branch_count": 0, "atom_count": 5, "selfies": "[C][O][O][C][=O]"}    COOC=O    0.307692
110   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/360.png                     
    {"single_bonds": 2, "double_bonds": 0, "triple_bonds": 1, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 0, "branch_count": 0, "atom_count": 4, "selfies": "[C][C][#C][C]"}            CC#CC         
    {"single_bonds": 2, "double_bonds": 1, "triple_bonds": 1, "ring_count": 0, "C_count": 5, "O_count": 0, "N_count": 0, "branch_count": 0, "atom_count": 5, "selfies": "[C][=C][C][#C][C]"}   C=CC#CC    0.307692
69    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/761.png                     
    {"single_bonds": 2, "double_bonds": 0, "triple_bonds": 1, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 4, "selfies": "[N][N][C][#C]"}            NNC#C          
    {"single_bonds": 3, "double_bonds": 0, "triple_bonds": 1, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[O][C][#C][N][N]"}    OC#CNN    0.312500
177   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/993.png                    
    {"single_bonds": 1, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 2, "N_count": 0, "branch_count": 0, "atom_count": 4, "selfies": "[C][=C][O][=O]"}            C=COO         
    {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 2, "N_count": 0, "branch_count": 0, "atom_count": 5, "selfies": "[O][=C][O][C][=C]"}   O=COC=C    0.312500
255   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/794.png                     
    {"single_bonds": 2, "double_bonds": 0, "triple_bonds": 1, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 4, "selfies": "[O][C][#C][N]"}            OC#CN          
    {"single_bonds": 3, "double_bonds": 0, "triple_bonds": 1, "ring_count": 0, "C_count": 2, "O_count": 2, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[O][O][C][#C][N]"}    OOC#CN    0.333333
285   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/536.png                    
    {"single_bonds": 1, "double_bonds": 0, "triple_bonds": 2, "ring_count": 0, "C_count": 3, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 4, "selfies": "[N][#C][C][#C]"}           N#CC#C         
    {"single_bonds": 2, "double_bonds": 0, "triple_bonds": 2, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[C][#C][C][C][#N]"}   C#CCC#N    0.333333
167  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1752.png             
    {"single_bonds": 1, "double_bonds": 2, "triple_bonds": 1, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[N][#C][N][=C][=C][O]"}        N#CN=C=CO        
    {"single_bonds": 1, "double_bonds": 2, "triple_bonds": 1, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[N][#C][N][=C][=O]"}  N#CN=C=O    0.350000
207    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/34.png                     
    {"single_bonds": 2, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 0, "branch_count": 0, "atom_count": 4, "selfies": "[C][C][O][=C]"}             CCOC          
    {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 1, "N_count": 0, "branch_count": 0, "atom_count": 5, "selfies": "[C][C][O][C][=C]"}    CCOC=C    0.400000
210   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/147.png                      
    {"single_bonds": 3, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 2, "N_count": 0, "branch_count": 0, "atom_count": 4, "selfies": "[C][O][C][O]"}             COCO           
    {"single_bonds": 4, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 2, "N_count": 0, "branch_count": 0, "atom_count": 5, "selfies": "[C][O][C][O][C]"}     COCOC    0.416667
151   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/565.png                    
    {"single_bonds": 1, "double_bonds": 0, "triple_bonds": 2, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 0, "branch_count": 0, "atom_count": 4, "selfies": "[C][#C][O][#C]"}            C#COC         
    {"single_bonds": 2, "double_bonds": 0, "triple_bonds": 2, "ring_count": 0, "C_count": 4, "O_count": 1, "N_count": 0, "branch_count": 0, "atom_count": 5, "selfies": "[C][#C][O][C][#C]"}   C#COC#C    0.416667
89    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/297.png               
    {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[C][N][=C][O][C][C]"}          CN=COCC          
    {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[C][N][=C][O][C]"}    CN=COC    0.421053
``` 

# 长度6的数据的训练

```
cp /opt/huangyan/ms-swift/swift/llm/sft.py.raw /opt/huangyan/ms-swift/swift/llm/sft.py

bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=1  PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True \
swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out.mol_desc/selected_data.length_1_4.500.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out.mol_desc/selected_data.length_5.500.json \
/opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out.mol_desc/selected_data.length_6.500.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/loop_selfies/image_in_json_out.mol_desc/length_6.ms_swift.val.json \
--eval_steps 50 \
--learning_rate 4e-4 \
--num_train_epochs 10 \
--logging_steps 10 \
--loss_name json_selfies_loss \
" > train_4.log 2>&1

``` 

评估结果: 大部分的原子数识别错误

```
Overall Similarity: 59.23%
Correct: 113 / 300 = 37.666666666666664 %
                                                          Image Path                                                                                                                                                                                        Predicted Predicted SMILES                                                                                                                                                                                                     GT   GT SMILES  Similarity
73   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2683.png       {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 1, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[N][N][=N][O][C]#C]"}                             {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 1, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 3, "branch_count": 0, "atom_count": 6, "selfies": "[C][#C][O][N][=N][N]"}                0.000000
124  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3592.png            {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 1, "C_count": 1, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 4, "selfies": "[C][=N][=O][N]"}            C=NON     {"single_bonds": 0, "double_bonds": 0, "triple_bonds": 0, "ring_count": 1, "C_count": 1, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[O][C][=N][N][Ring1][Ring2]"}     O1C=NN1    0.000000
212  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/5874.png     {"single_bonds": 0, "double_bonds": 2, "triple_bonds": 0, "ring_count": 1, "C_count": 1, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[C][=N][N][=C][Ring1]"}           C=NN=C     {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 1, "C_count": 2, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[N][C][N][=C][Ring1][Ring2]"}     N1CN=C1    0.000000
28   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6462.png              {"single_bonds": 3, "double_bonds": 0, "triple_bonds": 0, "ring_count": 1, "C_count": 0, "O_count": 2, "N_count": 2, "branch_count": 0, "atom_count": 4, "selfies": "[N][O][N][O]"}             NONO      {"single_bonds": 4, "double_bonds": 0, "triple_bonds": 0, "ring_count": 1, "C_count": 0, "O_count": 2, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[N][O][N][O][Ring1][Ring2]"}      N1ONO1    0.000000
159  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2603.png            {"single_bonds": 0, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 0, "branch_count": 0, "atom_count": 4, "selfies": "[C][=C][C][=C]"}           C=CC=C    {"single_bonds": 2, "double_bonds": 0, "triple_bonds": 2, "ring_count": 1, "C_count": 4, "O_count": 0, "N_count": 0, "branch_count": 0, "atom_count": 6, "selfies": "[C][#C][C][#C][Ring1][Ring2]"}    C1#CC#C1    0.000000
187  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6150.png  {"single_bonds": 2, "double_bonds": 1, "triple_bonds": 0, "ring_count": 1, "C_count": 1, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[C][=O][N][Ring1][Ring1]"}              C=O     {"single_bonds": 0, "double_bonds": 0, "triple_bonds": 1, "ring_count": 1, "C_count": 2, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[N][C][#C][O][Ring1][Ring2]"}     N1C#CO1    0.000000
290  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/5893.png             {"single_bonds": 2, "double_bonds": 1, "triple_bonds": 0, "ring_count": 1, "C_count": 1, "O_count": 0, "N_count": 3, "branch_count": 0, "atom_count": 4, "selfies": "[C][N][=N][N]"}            CN=NN     {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 1, "C_count": 1, "O_count": 0, "N_count": 3, "branch_count": 0, "atom_count": 6, "selfies": "[N][C][N][=N][Ring1][Ring2]"}     N1CN=N1    0.000000
49   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6586.png              {"single_bonds": 2, "double_bonds": 0, "triple_bonds": 0, "ring_count": 1, "C_count": 1, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 4, "selfies": "[N][N][C][O]"}             NNCO      {"single_bonds": 4, "double_bonds": 0, "triple_bonds": 0, "ring_count": 1, "C_count": 1, "O_count": 1, "N_count": 2, "branch_count": 1, "atom_count": 6, "selfies": "[N][N][C][O][Ring1][Ring1]"}      NN1CO1    0.058824
41   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4665.png             {"single_bonds": 2, "double_bonds": 1, "triple_bonds": 0, "ring_count": 1, "C_count": 2, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 4, "selfies": "[O][=C][N][C]"}            O=CNC     {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 1, "C_count": 2, "O_count": 1, "N_count": 1, "branch_count": 1, "atom_count": 6, "selfies": "[O][=C][C][N][Ring1][Ring1]"}     O=C1CN1    0.058824
36   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1818.png             {"single_bonds": 2, "double_bonds": 1, "triple_bonds": 0, "ring_count": 1, "C_count": 3, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 4, "selfies": "[N][C][C][=C]"}            NCC=C     {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 1, "C_count": 3, "O_count": 0, "N_count": 1, "branch_count": 1, "atom_count": 6, "selfies": "[C][=C][N][C][Ring1][Ring1]"}     C=C1NC1    0.058824
181  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4360.png              {"single_bonds": 2, "double_bonds": 0, "triple_bonds": 0, "ring_count": 1, "C_count": 1, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 4, "selfies": "[N][C][N][O]"}             NCNO     {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 1, "C_count": 1, "O_count": 1, "N_count": 2, "branch_count": 1, "atom_count": 6, "selfies": "[O][N][N][=C][Ring1][Ring1]"}     ON1N=C1    0.058824
145  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6335.png              {"single_bonds": 2, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 2, "N_count": 1, "branch_count": 0, "atom_count": 4, "selfies": "[O][C][O][N]"}             OCON      {"single_bonds": 4, "double_bonds": 0, "triple_bonds": 0, "ring_count": 1, "C_count": 1, "O_count": 2, "N_count": 1, "branch_count": 1, "atom_count": 6, "selfies": "[N][O][C][Ring1][Ring1][O]"}      N1OC1O    0.058824
235  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6687.png             {"single_bonds": 2, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 0, "N_count": 3, "branch_count": 0, "atom_count": 4, "selfies": "[N][=C][N][N]"}            N=CNN     {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 1, "C_count": 1, "O_count": 0, "N_count": 3, "branch_count": 1, "atom_count": 6, "selfies": "[N][N][C][Ring1][Ring1][=N]"}     N1NC1=N    0.066667
168  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4320.png       {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 1, "N_count": 4, "branch_count": 0, "atom_count": 6, "selfies": "[N][N][N][N][O][=C]"}           NNNNOC             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 1, "N_count": 4, "branch_count": 0, "atom_count": 6, "selfies": "[O][N][N][N][N][=C]"}     ONNNN=C    0.076923
161  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4493.png        {"single_bonds": 1, "double_bonds": 2, "triple_bonds": 1, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[N][=C][=N][C][#O]"}         N=C=NC=O           {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 1, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 3, "branch_count": 0, "atom_count": 6, "selfies": "[O][N][=C][=N][C][#N]"}   ON=C=NC#N    0.080000
14   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/5312.png         {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 1, "N_count": 3, "branch_count": 0, "atom_count": 5, "selfies": "[=C][N][N][O][=N]"}            CNNON            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 2, "N_count": 3, "branch_count": 0, "atom_count": 6, "selfies": "[O][=N][O][N][N][=C]"}    O=NONN=C    0.080000
275  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6193.png                {"single_bonds": 1, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 3, "selfies": "[N][C][=N]"}             NC=N       {"single_bonds": 2, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 0, "N_count": 2, "branch_count": 1, "atom_count": 6, "selfies": "[N][C][Branch1][C][N][=C]"}     NC(N)=C    0.090909
224   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/641.png             {"single_bonds": 2, "double_bonds": 0, "triple_bonds": 1, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 0, "branch_count": 0, "atom_count": 4, "selfies": "[C][C][#C][C]"}            CC#CC       {"single_bonds": 2, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 0, "branch_count": 1, "atom_count": 6, "selfies": "[C][C][Branch1][C][C][=C]"}     CC(C)=C    0.100000
198   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/332.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 0, "N_count": 3, "branch_count": 0, "atom_count": 5, "selfies": "[C][N][N][=C][=N]"}          CNN=C=N            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 0, "N_count": 3, "branch_count": 0, "atom_count": 6, "selfies": "[C][C][N][=C][=N][N]"}    CCN=C=NN    0.120000
100  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6494.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 2, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[O][=C][C][N][O]"}           O=CCNO            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 2, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[N][O][N][=C][C][=O]"}    NON=CC=O    0.120000
12   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4762.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 3, "N_count": 0, "branch_count": 0, "atom_count": 5, "selfies": "[C][=O][O][O][=C]"}              C=O            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 3, "N_count": 0, "branch_count": 0, "atom_count": 6, "selfies": "[O][=C][O][O][C][=C]"}    O=COOC=C    0.142857
156  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6222.png      {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 1, "ring_count": 0, "C_count": 4, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[N][O][C][=C][C][#C]"}         NOC=CC#C             {"single_bonds": 4, "double_bonds": 0, "triple_bonds": 1, "ring_count": 0, "C_count": 4, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[N][O][C][C][#C][C]"}     NOCC#CC    0.153846
53     /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/49.png  {"single_bonds": 2, "double_bonds": 1, "triple_bonds": 0, "ring_count": 1, "C_count": 2, "O_count": 1, "N_count": 0, "branch_count": 0, "atom_count": 5, "selfies": "[C][O][=C][Ring1][Ring1]"}            C1OC1      {"single_bonds": 4, "double_bonds": 0, "triple_bonds": 0, "ring_count": 1, "C_count": 3, "O_count": 1, "N_count": 0, "branch_count": 1, "atom_count": 6, "selfies": "[C][C][C][O][Ring1][Ring1]"}      CC1CO1    0.166667
177  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6184.png     {"single_bonds": 3, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[N][Branch1][C][O][C]"}            N(O)C        {"single_bonds": 3, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 1, "branch_count": 1, "atom_count": 6, "selfies": "[N][C][Branch1][C][O][C]"}      NC(O)C    0.166667
166  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2047.png        {"single_bonds": 1, "double_bonds": 2, "triple_bonds": 1, "ring_count": 0, "C_count": 3, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[N][#C][C][=N][=C]"}          N#CC=NC           {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 1, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[C][=C][=N][C][C][#N]"}   C=C=NCC#N    0.166667
113   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/122.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[C][C][N][=N][=O]"}           CCN=NO            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[C][C][C][=N][N][=O]"}    CCC=NN=O    0.173913
202  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2574.png         {"single_bonds": 2, "double_bonds": 1, "triple_bonds": 2, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[C][#C][N][C][=C]"}          C#CNC=C           {"single_bonds": 2, "double_bonds": 1, "triple_bonds": 2, "ring_count": 0, "C_count": 5, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[C][#C][C][=N][C][#C]"}   C#CC=NC#C    0.181818
5     /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/757.png   {"single_bonds": 2, "double_bonds": 0, "triple_bonds": 0, "ring_count": 1, "C_count": 2, "O_count": 1, "N_count": 0, "branch_count": 0, "atom_count": 5, "selfies": "[C][C][O][Ring1][Ring1]"}            C1CO1     {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 1, "C_count": 3, "O_count": 1, "N_count": 0, "branch_count": 0, "atom_count": 6, "selfies": "[C][O][C][=C][Ring1][Ring2]"}     C1OC=C1    0.181818
6    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/5991.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[N][C][=C][C][=N]"}          NC=CC=N           {"single_bonds": 2, "double_bonds": 3, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 0, "N_count": 3, "branch_count": 0, "atom_count": 6, "selfies": "[N][C][=C][=C][N][=N]"}   NC=C=CN=N    0.181818
17   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/5850.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[C][=C][N][=N][C]"}          C=CN=NC            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[N][C][N][=C][C][=C]"}    NCN=CC=C    0.181818
60   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/5195.png     {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[O][=N][C][C][=C][=C]"}        O=NCC=C=C           {"single_bonds": 2, "double_bonds": 3, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[O][=N][C][=C][C][=C]"}   O=NC=CC=C    0.200000
40   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1433.png      {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[O][=C][C][=C][N][C]"}         O=CC=CNC           {"single_bonds": 2, "double_bonds": 3, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[C][N][=C][=C][C][=O]"}   CN=C=CC=O    0.208333
228  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4847.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 1, "N_count": 3, "branch_count": 0, "atom_count": 5, "selfies": "[N][N][O][N][=C]"}           NNON=C             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 2, "N_count": 3, "branch_count": 0, "atom_count": 6, "selfies": "[O][=C][N][O][N][N]"}     O=CNONN    0.217391
291  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3765.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 2, "N_count": 0, "branch_count": 0, "atom_count": 5, "selfies": "[O][O][C][=C][=C]"}          OOC=C=C            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 3, "N_count": 0, "branch_count": 0, "atom_count": 6, "selfies": "[O][O][C][=C][C][=O]"}    OOC=CC=O    0.227273
131  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3153.png         {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[N][=N][C][=C][O]"}          N=NC=CO            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 3, "branch_count": 0, "atom_count": 6, "selfies": "[O][C][C][=N][N][=N]"}    OCC=NN=N    0.227273
109  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1803.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[C][=C][N][C][=N]"}          C=CNC=N            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[C][=C][N][C][N][=C]"}    C=CNCN=C    0.227273
48   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2282.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[C][=N][O][C][=N]"}          C=NOC=N            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[C][=N][O][C][N][=C]"}    C=NOCN=C    0.238095
92   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2434.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 0, "N_count": 3, "branch_count": 0, "atom_count": 5, "selfies": "[N][=N][C][N][=C]"}          N=NCN=C           {"single_bonds": 2, "double_bonds": 3, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 0, "N_count": 4, "branch_count": 0, "atom_count": 6, "selfies": "[C][=N][N][=C][N][=N]"}   C=NN=CN=N    0.238095
178  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4539.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 2, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[O][N][=N][C][O]"}           ON=NCO             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 2, "N_count": 3, "branch_count": 0, "atom_count": 6, "selfies": "[O][N][=N][O][C][N]"}     ON=NOCN    0.250000
78   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4416.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 2, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[N][C][O][=N][O]"}            NCONO             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 2, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[O][N][=C][O][C][N]"}     ON=COCN    0.260870
142  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4181.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 2, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[O][O][N][C][=C]"}           OONC=C             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 2, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[O][N][O][C][C][=C]"}     ONOCC=C    0.260870
52    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/215.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[C][C][O][N][=N]"}           CCON=N             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[C][C][O][N][N][=C]"}     CCONN=C    0.260870
182  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6602.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 0, "N_count": 4, "branch_count": 0, "atom_count": 5, "selfies": "[N][N][C][N][=N]"}           NNCN=N             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 0, "N_count": 4, "branch_count": 0, "atom_count": 6, "selfies": "[N][N][C][N][N][=C]"}     NNCNN=C    0.272727
117  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4817.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 1, "N_count": 3, "branch_count": 0, "atom_count": 5, "selfies": "[N][N][C][N][=O]"}           NNCN=O             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 3, "branch_count": 0, "atom_count": 6, "selfies": "[O][=C][N][C][N][N]"}     O=CNCNN    0.272727
65    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/348.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 0, "N_count": 3, "branch_count": 0, "atom_count": 5, "selfies": "[N][N][N][=C][C]"}           NNN=CC             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 0, "N_count": 4, "branch_count": 0, "atom_count": 6, "selfies": "[C][C][N][=N][N][N]"}     CCN=NNN    0.272727
176  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6212.png          {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[N][O][C][C][=N]"}           NOCC=N             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 3, "branch_count": 0, "atom_count": 6, "selfies": "[N][O][C][C][N][=N]"}     NOCCN=N    0.272727
243  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1983.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[C][=C][=C][N][O]"}          C=C=CNO           {"single_bonds": 2, "double_bonds": 3, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[C][=C][=C][N][=C][O]"}   C=C=CN=CO    0.272727
130  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3585.png        {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[C][=C][=N][C][=O]"}         C=C=NC=O           {"single_bonds": 2, "double_bonds": 3, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[O][C][=N][N][=C][=C]"}   OC=NN=C=C    0.272727
87   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7482.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 0, "N_count": 3, "branch_count": 0, "atom_count": 5, "selfies": "[N][=C][=C][N][N]"}          N=C=CNN           {"single_bonds": 2, "double_bonds": 3, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 0, "N_count": 3, "branch_count": 0, "atom_count": 6, "selfies": "[N][=C][=C][C][=N][N]"}   N=C=CC=NN    0.272727
214  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/5302.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 2, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[O][=N][O][C][=N]"}          O=NOC=N            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 3, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[O][=N][O][N][C][=O]"}    O=NONC=O    0.272727
70   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6783.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 0, "N_count": 3, "branch_count": 0, "atom_count": 5, "selfies": "[N][N][N][=C][C]"}           NNN=CC             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 0, "N_count": 3, "branch_count": 0, "atom_count": 6, "selfies": "[N][N][N][C][=C][C]"}     NNNC=CC    0.285714
169  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7934.png         {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 0, "N_count": 4, "branch_count": 0, "atom_count": 5, "selfies": "[N][=N][N][C][=N]"}          N=NNC=N            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 0, "N_count": 4, "branch_count": 0, "atom_count": 6, "selfies": "[N][=N][N][N][C][=C]"}    N=NNNC=C    0.285714
229   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/420.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 0, "N_count": 3, "branch_count": 0, "atom_count": 5, "selfies": "[N][N][N][=C][C]"}           NNN=CC             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 0, "N_count": 3, "branch_count": 0, "atom_count": 6, "selfies": "[C][C][=C][N][N][N]"}     CC=CNNN    0.285714
249  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1813.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[O][N][=C][N][=C]"}          ON=CN=C            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[C][=C][N][C][=N][O]"}    C=CNC=NO    0.285714
45   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7222.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[N][=C][C][N][=O]"}          N=CCN=O           {"single_bonds": 2, "double_bonds": 3, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 3, "branch_count": 0, "atom_count": 6, "selfies": "[N][=C][C][=N][N][=O]"}   N=CC=NN=O    0.285714
27   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2218.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[N][C][=C][=N][C]"}          NC=C=NC           {"single_bonds": 2, "double_bonds": 3, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[C][=N][C][=C][=C][N]"}   C=NC=C=CN    0.285714
129  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4476.png      {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[O][N][=C][=C][C][C]"}         ON=C=CCC           {"single_bonds": 2, "double_bonds": 3, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[O][N][=C][=C][=C][C]"}   ON=C=C=CC    0.291667
35   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2109.png        {"single_bonds": 1, "double_bonds": 3, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[O][=C][=N][C][=C]"}         O=C=NC=C          {"single_bonds": 1, "double_bonds": 4, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[C][=C][=N][N][=C][=O]"}  C=C=NN=C=O    0.300000
281  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7182.png      {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 5, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[N][=C][C][=C][C][C]"}         N=CC=CCC           {"single_bonds": 2, "double_bonds": 3, "triple_bonds": 0, "ring_count": 0, "C_count": 5, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[N][=C][C][=C][C][=C]"}   N=CC=CC=C    0.300000
232  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4091.png           {"single_bonds": 4, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 1, "N_count": 3, "branch_count": 0, "atom_count": 5, "selfies": "[N][C][N][N][O]"}            NCNNO              {"single_bonds": 5, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 1, "N_count": 4, "branch_count": 0, "atom_count": 6, "selfies": "[O][N][C][N][N][N]"}      ONCNNN    0.300000
234  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3098.png       {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[C][N][=C][C][C][O]"}          CN=CCCO             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[O][C][C][N][=C][C]"}     OCCN=CC    0.304348
62   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1792.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[C][=C][N][C][=C]"}          C=CNC=C            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 5, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[C][=C][N][C][C][=C]"}    C=CNCC=C    0.312500
247  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2494.png         {"single_bonds": 2, "double_bonds": 0, "triple_bonds": 2, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[N][C][#C][C][#C]"}          NC#CC#C            {"single_bonds": 3, "double_bonds": 0, "triple_bonds": 2, "ring_count": 0, "C_count": 5, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[C][#C][C][C][#C][N]"}    C#CCC#CN    0.315789
93   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7963.png      {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 0, "N_count": 4, "branch_count": 0, "atom_count": 6, "selfies": "[N][=C][C][=N][N][N]"}         N=CC=NNN           {"single_bonds": 2, "double_bonds": 3, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 0, "N_count": 4, "branch_count": 0, "atom_count": 6, "selfies": "[N][=N][N][=C][C][=N]"}   N=NN=CC=N    0.333333
206  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4524.png      {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 3, "branch_count": 0, "atom_count": 6, "selfies": "[C][C][=N][N][=N][O]"}         CC=NN=NO            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[O][N][=N][C][=C][C]"}    ON=NC=CC    0.333333
215  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4611.png        {"single_bonds": 1, "double_bonds": 3, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 1, "N_count": 0, "branch_count": 0, "atom_count": 5, "selfies": "[O][=C][C][=C][=C]"}         O=CC=C=C           {"single_bonds": 2, "double_bonds": 3, "triple_bonds": 0, "ring_count": 0, "C_count": 5, "O_count": 1, "N_count": 0, "branch_count": 0, "atom_count": 6, "selfies": "[O][=C][C][C][=C][=C]"}   O=CCC=C=C    0.350000
197  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7816.png          {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 2, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[C][O][C][O][=N]"}            COCON             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 2, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[N][=N][O][C][O][C]"}     N=NOCOC    0.350000
111   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/824.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 2, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[C][O][O][C][=N]"}           COOC=N             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 3, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[C][O][O][C][=N][O]"}     COOC=NO    0.350000
190  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3393.png         {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 1, "ring_count": 0, "C_count": 4, "O_count": 1, "N_count": 0, "branch_count": 0, "atom_count": 5, "selfies": "[C][#C][C][=C][O]"}          C#CC=CO            {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 1, "ring_count": 0, "C_count": 5, "O_count": 1, "N_count": 0, "branch_count": 0, "atom_count": 6, "selfies": "[O][C][=C][C][C][#C]"}    OC=CCC#C    0.350000
79   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2245.png         {"single_bonds": 2, "double_bonds": 1, "triple_bonds": 1, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[O][C][C][#C][=N]"}           OCC#CN            {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 1, "ring_count": 0, "C_count": 4, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[C][=N][C][#C][C][O]"}    C=NC#CCO    0.350000
172  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3400.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[N][N][C][=C][O]"}           NNC=CO             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[O][C][=C][C][N][N]"}     OC=CCNN    0.350000
219  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/5292.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 2, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[C][N][O][O][=N]"}            CNOON             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 3, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[O][=N][O][O][N][C]"}     O=NOONC    0.350000
217  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4047.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[N][C][=C][N][O]"}           NC=CNO             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[O][N][C][C][=C][N]"}     ONCC=CN    0.350000
195   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/691.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 2, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[C][O][C][O][=N]"}            COCON             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 2, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[C][O][C][O][C][=N]"}     COCOC=N    0.350000
57   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4422.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 2, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[C][O][O][C][=N]"}           COOC=N             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 3, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[O][N][=C][O][O][C]"}     ON=COOC    0.350000
171  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7582.png          {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 0, "N_count": 3, "branch_count": 0, "atom_count": 5, "selfies": "[N][C][C][N][=C]"}           NCCN=C            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 0, "N_count": 3, "branch_count": 0, "atom_count": 6, "selfies": "[N][=C][=N][C][C][N]"}    N=C=NCCN    0.350000
255  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7867.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 1, "N_count": 3, "branch_count": 0, "atom_count": 5, "selfies": "[C][N][N][O][=N]"}            CNNON             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 1, "N_count": 4, "branch_count": 0, "atom_count": 6, "selfies": "[N][=N][O][N][N][C]"}     N=NONNC    0.350000
269  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6267.png          {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[C][N][=N][C][O]"}           CN=NCO             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 3, "branch_count": 0, "atom_count": 6, "selfies": "[N][O][C][N][=N][C]"}     NOCN=NC    0.350000
74   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1124.png          {"single_bonds": 3, "double_bonds": 0, "triple_bonds": 1, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[O][C][C][#C][N]"}           OCC#CN             {"single_bonds": 4, "double_bonds": 0, "triple_bonds": 1, "ring_count": 0, "C_count": 4, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[C][N][C][#C][C][O]"}     CNC#CCO    0.350000
264  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4616.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[N][N][=C][C][=O]"}          NN=CC=O            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[O][=C][C][C][=N][N]"}    O=CCC=NN    0.350000
297  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2329.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 1, "N_count": 3, "branch_count": 0, "atom_count": 5, "selfies": "[N][N][N][O][=C]"}            NNNOC             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 1, "N_count": 4, "branch_count": 0, "atom_count": 6, "selfies": "[C][=N][O][N][N][N]"}     C=NONNN    0.350000
3    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1126.png         {"single_bonds": 2, "double_bonds": 1, "triple_bonds": 1, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[C][=C][C][#C][N]"}          C=CC#CN            {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 1, "ring_count": 0, "C_count": 5, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[C][N][C][#C][C][=C]"}    CNC#CC=C    0.350000
10   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7675.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[N][=N][C][C][N]"}           N=NCCN             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 0, "N_count": 3, "branch_count": 0, "atom_count": 6, "selfies": "[N][=N][C][C][N][C]"}     N=NCCNC    0.350000
230  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/5438.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 0, "O_count": 1, "N_count": 4, "branch_count": 0, "atom_count": 5, "selfies": "[N][=N][N][=N][O]"}          N=NN=NO           {"single_bonds": 2, "double_bonds": 3, "triple_bonds": 0, "ring_count": 0, "C_count": 0, "O_count": 1, "N_count": 5, "branch_count": 0, "atom_count": 6, "selfies": "[O][=N][N][=N][N][=N]"}   O=NN=NN=N    0.352941
144  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4821.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[C][=C][N][C][=O]"}          C=CNC=O            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[O][=C][N][C][=C][C]"}    O=CNC=CC    0.368421
283  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3762.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 2, "N_count": 0, "branch_count": 0, "atom_count": 5, "selfies": "[O][O][C][=C][C]"}           OOC=CC             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 3, "N_count": 0, "branch_count": 0, "atom_count": 6, "selfies": "[O][O][C][=C][C][O]"}     OOC=CCO    0.368421
108  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2599.png         {"single_bonds": 2, "double_bonds": 1, "triple_bonds": 1, "ring_count": 0, "C_count": 3, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[N][N][C][#C][=C]"}           NNC#CC            {"single_bonds": 3, "double_bonds": 0, "triple_bonds": 2, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[C][#C][C][#C][N][N]"}    C#CC#CNN    0.368421
19   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3389.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[N][C][C][C][=O]"}           NCCC=O             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[O][C][=C][C][C][N]"}     OC=CCCN    0.368421
194  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2559.png        {"single_bonds": 1, "double_bonds": 2, "triple_bonds": 1, "ring_count": 0, "C_count": 4, "O_count": 1, "N_count": 0, "branch_count": 0, "atom_count": 5, "selfies": "[C][#C][C][=C][=O]"}         C#CC=C=O           {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 1, "ring_count": 0, "C_count": 5, "O_count": 1, "N_count": 0, "branch_count": 0, "atom_count": 6, "selfies": "[C][#C][C][=C][=C][O]"}   C#CC=C=CO    0.368421
175  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2596.png         {"single_bonds": 2, "double_bonds": 0, "triple_bonds": 2, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[N][O][C][#C][#C]"}           NOC#CC            {"single_bonds": 3, "double_bonds": 0, "triple_bonds": 2, "ring_count": 0, "C_count": 4, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[C][#C][C][#C][O][N]"}    C#CC#CON    0.368421
121  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7346.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[N][=C][N][C][=C]"}          N=CNC=C            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[N][=C][N][C][=C][C]"}    N=CNC=CC    0.368421
18   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3392.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[N][=C][C][C][=O]"}          N=CCC=O            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[O][C][=C][C][C][=N]"}    OC=CCC=N    0.368421
267  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/5232.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[C][=N][N][=C][O]"}          C=NN=CO           {"single_bonds": 2, "double_bonds": 3, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 3, "branch_count": 0, "atom_count": 6, "selfies": "[O][=N][C][=N][N][=C]"}   O=NC=NN=C    0.368421
173  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6227.png           {"single_bonds": 4, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 2, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[N][O][C][O][C]"}            NOCOC              {"single_bonds": 5, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 3, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[N][O][C][O][C][O]"}      NOCOCO    0.368421
38    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/582.png         {"single_bonds": 2, "double_bonds": 1, "triple_bonds": 1, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[N][C][=C][C][#C]"}          NC=CC#C            {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 1, "ring_count": 0, "C_count": 5, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[C][C][#C][C][=C][N]"}    CC#CC=CN    0.368421
207  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6531.png         {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 1, "N_count": 3, "branch_count": 0, "atom_count": 5, "selfies": "[N][O][N][=N][=C]"}           NON=NC            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 1, "N_count": 4, "branch_count": 0, "atom_count": 6, "selfies": "[N][O][N][=N][N][=C]"}    NON=NN=C    0.368421
208   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/736.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[C][O][C][=C][N]"}           COC=CN            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[C][O][C][=C][C][=N]"}    COC=CC=N    0.368421
125  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1850.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 0, "N_count": 3, "branch_count": 0, "atom_count": 5, "selfies": "[C][=C][N][N][N]"}           C=CNNN             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 0, "N_count": 3, "branch_count": 0, "atom_count": 6, "selfies": "[C][=C][N][N][N][C]"}     C=CNNNC    0.368421
51   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3456.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[N][C][=N][C][=O]"}          NC=NC=O            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[O][C][=C][N][=C][N]"}    OC=CN=CN    0.368421
69   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1347.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 1, "N_count": 3, "branch_count": 0, "atom_count": 5, "selfies": "[C][N][N][=N][=O]"}           CNN=NO            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 1, "N_count": 4, "branch_count": 0, "atom_count": 6, "selfies": "[C][N][N][=N][N][=O]"}    CNN=NN=O    0.368421
67   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3302.png           {"single_bonds": 4, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 2, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[O][N][O][C][O]"}            ONOCO              {"single_bonds": 5, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 3, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[O][C][N][O][N][O]"}      OCNONO    0.368421
231  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/8304.png        {"single_bonds": 1, "double_bonds": 3, "triple_bonds": 1, "ring_count": 0, "C_count": 3, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[N][#C][N][=C][=C]"}         N#CN=C=C          {"single_bonds": 1, "double_bonds": 3, "triple_bonds": 1, "ring_count": 0, "C_count": 3, "O_count": 0, "N_count": 3, "branch_count": 0, "atom_count": 6, "selfies": "[N][#C][N][=C][=C][=N]"}  N#CN=C=C=N    0.368421
226    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/59.png           {"single_bonds": 4, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[C][C][C][N][O]"}            CCCNO              {"single_bonds": 5, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[C][C][C][N][O][C]"}      CCCNOC    0.368421
150  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3247.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 2, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[N][=N][O][C][O]"}           N=NOCO             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 2, "N_count": 3, "branch_count": 0, "atom_count": 6, "selfies": "[O][C][O][N][N][=N]"}     OCONN=N    0.380952
112  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7337.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[N][=C][N][C][O]"}           N=CNCO             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[N][=C][N][C][O][C]"}     N=CNCOC    0.380952
223  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4661.png        {"single_bonds": 1, "double_bonds": 3, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[N][=C][=N][C][=O]"}         N=C=NC=O           {"single_bonds": 2, "double_bonds": 3, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[O][=C][C][N][=C][=N]"}   O=CCN=C=N    0.380952
15   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4394.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 2, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[O][N][=C][O][N]"}           ON=CON             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 2, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[O][N][=C][C][O][N]"}     ON=CCON    0.380952
260  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1895.png         {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[C][=C][N][=N][C]"}          C=CN=NC            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[C][=C][N][=N][C][C]"}    C=CN=NCC    0.388889
16   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7063.png         {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 0, "O_count": 1, "N_count": 4, "branch_count": 0, "atom_count": 5, "selfies": "[N][N][=N][O][=N]"}           NN=NON            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 0, "O_count": 1, "N_count": 5, "branch_count": 0, "atom_count": 6, "selfies": "[N][N][=N][O][N][=N]"}    NN=NON=N    0.388889
196   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/508.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[C][C][=N][C][=N]"}          CC=NC=N            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[C][C][=N][C][=N][C]"}    CC=NC=NC    0.388889
32   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6888.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 0, "O_count": 0, "N_count": 5, "branch_count": 0, "atom_count": 5, "selfies": "[N][N][=N][N][N]"}           NN=NNN            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 0, "O_count": 0, "N_count": 6, "branch_count": 0, "atom_count": 6, "selfies": "[N][N][N][=N][N][=N]"}    NNN=NN=N    0.388889
189  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/5766.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 1, "N_count": 3, "branch_count": 0, "atom_count": 5, "selfies": "[N][N][=N][O][C]"}           NN=NOC             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 1, "N_count": 4, "branch_count": 0, "atom_count": 6, "selfies": "[N][C][O][N][=N][N]"}     NCON=NN    0.388889
167  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/5276.png         {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 2, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[O][N][=C][O][=N]"}           ON=CON            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 3, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[O][=N][O][C][=N][O]"}    O=NOC=NO    0.400000
147  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/5662.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[N][C][C][=N][O]"}           NCC=NO            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 3, "branch_count": 0, "atom_count": 6, "selfies": "[N][C][C][=N][N][=O]"}    NCC=NN=O    0.400000
165   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/513.png         {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 1, "ring_count": 0, "C_count": 3, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[C][C][=N][C][#N]"}          CC=NC#N            {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 1, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[C][C][=N][C][#C][N]"}    CC=NC#CN    0.400000
149  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3731.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 3, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[O][=N][C][O][O]"}           O=NCOO             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 4, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[O][O][C][O][N][=O]"}     OOCON=O    0.400000
77   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7819.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[N][=N][O][C][N]"}           N=NOCN             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 3, "branch_count": 0, "atom_count": 6, "selfies": "[N][=N][O][C][N][C]"}     N=NOCNC    0.400000
29    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/417.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[N][O][N][C][=C]"}           NONC=C             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[C][C][=C][N][O][N]"}     CC=CNON    0.400000
241  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6585.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 1, "N_count": 3, "branch_count": 0, "atom_count": 5, "selfies": "[N][N][C][O][=N]"}            NNCON             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 1, "N_count": 4, "branch_count": 0, "atom_count": 6, "selfies": "[N][N][C][O][N][=N]"}     NNCON=N    0.400000
236  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6938.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[C][=N][O][C][=N]"}          C=NOC=N            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 3, "branch_count": 0, "atom_count": 6, "selfies": "[N][N][=C][O][N][=C]"}    NN=CON=C    0.400000
151  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1394.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 2, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[C][N][=C][O][O]"}           CN=COO             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 2, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[C][N][=C][O][O][N]"}     CN=COON    0.400000
80   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7909.png         {"single_bonds": 2, "double_bonds": 1, "triple_bonds": 1, "ring_count": 0, "C_count": 3, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[N][=N][N][C][#C]"}          N=NNC#C            {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 1, "ring_count": 0, "C_count": 3, "O_count": 0, "N_count": 3, "branch_count": 0, "atom_count": 6, "selfies": "[N][=N][N][C][#C][C]"}    N=NNC#CC    0.400000
261  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4816.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 2, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[O][=C][N][C][N]"}           O=CNCN             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 2, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[O][=C][N][C][N][O]"}     O=CNCNO    0.400000
96   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/5929.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 2, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[O][C][O][C][=N]"}           OCOC=N             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 2, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[N][C][=C][O][C][O]"}     NC=COCO    0.400000
288   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/967.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[C][O][N][=C][=C]"}          CON=C=C            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[C][O][N][=C][=C][C]"}    CON=C=CC    0.400000
191  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7255.png         {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 2, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[O][=C][C][O][=N]"}           O=CCON            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 2, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[N][=C][O][C][C][=O]"}    N=COCC=O    0.400000
2    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6603.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 1, "N_count": 3, "branch_count": 0, "atom_count": 5, "selfies": "[N][N][C][N][=O]"}           NNCN=O             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 1, "N_count": 4, "branch_count": 0, "atom_count": 6, "selfies": "[N][N][C][N][N][=O]"}     NNCNN=O    0.400000
1     /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/630.png         {"single_bonds": 2, "double_bonds": 1, "triple_bonds": 1, "ring_count": 0, "C_count": 3, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[N][C][=N][C][#C]"}          NC=NC#C            {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 1, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[C][C][#C][N][=C][N]"}    CC#CN=CN    0.400000
205   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/754.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[C][O][C][=C][=N]"}          COC=C=N            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[C][O][C][=C][=N][C]"}    COC=C=NC    0.400000
162  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1500.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 0, "N_count": 3, "branch_count": 0, "atom_count": 5, "selfies": "[C][N][=N][C][=N]"}          CN=NC=N            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 0, "N_count": 3, "branch_count": 0, "atom_count": 6, "selfies": "[C][N][=N][C][=N][C]"}    CN=NC=NC    0.411765
277  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1494.png         {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[C][N][=N][C][=C]"}          CN=NC=C            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[C][N][=N][C][=C][C]"}    CN=NC=CC    0.411765
160  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4747.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 3, "N_count": 0, "branch_count": 0, "atom_count": 5, "selfies": "[O][C][=C][O][=O]"}           OC=COO            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 3, "N_count": 0, "branch_count": 0, "atom_count": 6, "selfies": "[O][=C][O][C][=C][O]"}    O=COC=CO    0.411765
245   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/502.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[C][C][=N][C][=C]"}          CC=NC=C            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 5, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[C][C][=N][C][=C][C]"}    CC=NC=CC    0.411765
280  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2391.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 0, "N_count": 3, "branch_count": 0, "atom_count": 5, "selfies": "[C][=N][N][N][C]"}           C=NNNC             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 0, "N_count": 3, "branch_count": 0, "atom_count": 6, "selfies": "[C][=N][N][N][C][C]"}     C=NNNCC    0.421053
139  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/5627.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[C][O][C][=C][N]"}           COC=CN             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[N][C][C][=C][O][C]"}     NCC=COC    0.421053
186  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7448.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 0, "N_count": 4, "branch_count": 0, "atom_count": 5, "selfies": "[N][N][=N][C][=N]"}          NN=NC=N            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 0, "N_count": 5, "branch_count": 0, "atom_count": 6, "selfies": "[N][=C][N][=N][N][N]"}    N=CN=NNN    0.421053
248  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6877.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 0, "N_count": 4, "branch_count": 0, "atom_count": 5, "selfies": "[N][N][=N][C][=N]"}          NN=NC=N            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 0, "N_count": 5, "branch_count": 0, "atom_count": 6, "selfies": "[N][N][N][=N][C][=N]"}    NNN=NC=N    0.421053
257  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/5213.png        {"single_bonds": 1, "double_bonds": 3, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 2, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[O][=N][C][=C][=C]"}         O=NC=C=C          {"single_bonds": 1, "double_bonds": 4, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 2, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[O][=N][C][=C][=C][=O]"}  O=NC=C=C=O    0.421053
43   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/5726.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 3, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[O][=C][O][O][N]"}           O=COON             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 3, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[N][C][O][O][C][=O]"}     NCOOC=O    0.421053
220   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/212.png           {"single_bonds": 4, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[C][C][O][N][N]"}            CCONN              {"single_bonds": 5, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[C][C][O][N][N][C]"}      CCONNC    0.421053
123  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2199.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[C][=N][C][=C][C]"}          C=NC=CC            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 5, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[C][=N][C][=C][C][C]"}    C=NC=CCC    0.421053
98     /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/97.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 1, "N_count": 0, "branch_count": 0, "atom_count": 5, "selfies": "[O][C][=C][=C][C]"}          OC=C=CC            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 5, "O_count": 1, "N_count": 0, "branch_count": 0, "atom_count": 6, "selfies": "[C][C][C][=C][=C][O]"}    CCC=C=CO    0.421053
104  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4973.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[C][=C][N][C][=O]"}          C=CNC=O           {"single_bonds": 2, "double_bonds": 3, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[O][=C][=C][N][C][=C]"}   O=C=CNC=C    0.421053
188   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/195.png           {"single_bonds": 4, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 2, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[N][N][O][O][C]"}            NNOOC              {"single_bonds": 5, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 2, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[C][C][O][O][N][N]"}      CCOONN    0.421053
163  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1368.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[C][N][=C][C][N]"}           CN=CCN            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[C][N][=C][C][N][=C]"}    CN=CCN=C    0.421053
75   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4362.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 1, "N_count": 3, "branch_count": 0, "atom_count": 5, "selfies": "[C][N][=N][N][O]"}           CN=NNO             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 3, "branch_count": 0, "atom_count": 6, "selfies": "[O][N][N][=N][C][C]"}     ONN=NCC    0.421053
63   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2553.png   {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 1, "ring_count": 0, "C_count": 4, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 7, "selfies": "[O][N][C][=C][C][C][#C]"}        ONC=CCC#C            {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 1, "ring_count": 0, "C_count": 4, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[C][#C][C][=C][N][O]"}    C#CC=CNO    0.434783
138  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7232.png            {"single_bonds": 1, "double_bonds": 1, "triple_bonds": 1, "ring_count": 0, "C_count": 3, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 4, "selfies": "[N][=C][C][#C]"}           N=CC#C           {"single_bonds": 2, "double_bonds": 1, "triple_bonds": 2, "ring_count": 0, "C_count": 5, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[N][=C][C][#C][C][#C]"}   N=CC#CC#C    0.437500
146  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6601.png           {"single_bonds": 4, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 0, "N_count": 4, "branch_count": 0, "atom_count": 5, "selfies": "[N][C][N][N][N]"}            NCNNN              {"single_bonds": 5, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 0, "N_count": 5, "branch_count": 0, "atom_count": 6, "selfies": "[N][N][C][N][N][N]"}      NNCNNN    0.437500
183  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4190.png           {"single_bonds": 4, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 2, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[O][N][C][O][N]"}            ONCON              {"single_bonds": 5, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 3, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[O][N][O][C][N][O]"}      ONOCNO    0.444444
286   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/631.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 1, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[C][C][#C][N][=C]"}          CC#CN=C           {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 1, "ring_count": 0, "C_count": 5, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[C][C][#C][N][=C][=C]"}   CC#CN=C=C    0.444444
0     /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/810.png           {"single_bonds": 4, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 2, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[C][O][O][C][N]"}            COOCN              {"single_bonds": 5, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 3, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[C][O][O][C][O][N]"}      COOCON    0.444444
58   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1717.png         {"single_bonds": 2, "double_bonds": 1, "triple_bonds": 1, "ring_count": 0, "C_count": 4, "O_count": 1, "N_count": 0, "branch_count": 0, "atom_count": 5, "selfies": "[C][#C][C][O][=C]"}           C#CCOC            {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 1, "ring_count": 0, "C_count": 5, "O_count": 1, "N_count": 0, "branch_count": 0, "atom_count": 6, "selfies": "[C][=C][O][C][C][#C]"}    C=COCC#C    0.450000
239  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6057.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 2, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[N][C][=N][O][O]"}           NC=NOO             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 2, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[N][C][=N][O][C][O]"}     NC=NOCO    0.450000
42   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3150.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[N][N][=C][C][O]"}           NN=CCO             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 3, "branch_count": 0, "atom_count": 6, "selfies": "[O][C][C][=N][N][N]"}     OCC=NNN    0.450000
218  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3972.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 1, "N_count": 3, "branch_count": 0, "atom_count": 5, "selfies": "[N][C][=N][N][O]"}           NC=NNO             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 2, "N_count": 3, "branch_count": 0, "atom_count": 6, "selfies": "[O][O][N][N][=C][N]"}     OONN=CN    0.450000
216   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/250.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[O][=C][=C][N][C]"}          O=C=CNC            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[C][C][N][C][=C][=O]"}    CCNC=C=O    0.450000
21   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7945.png         {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 0, "N_count": 4, "branch_count": 0, "atom_count": 5, "selfies": "[N][=N][N][N][=C]"}          N=NNN=C            {"single_bonds": 3, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 0, "N_count": 5, "branch_count": 0, "atom_count": 6, "selfies": "[N][=N][N][N][N][=C]"}    N=NNNN=C    0.470588
55    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/262.png           {"single_bonds": 4, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[C][C][N][O][N]"}            CCNON              {"single_bonds": 5, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[C][C][N][O][C][N]"}      CCNOCN    0.473684
157  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/5128.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 2, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[C][O][C][N][=O]"}           COCN=O             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 2, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[O][=N][C][C][O][C]"}     O=NCCOC    0.473684
143  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4990.png        {"single_bonds": 1, "double_bonds": 3, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[O][=C][=C][N][=C]"}         O=C=CN=C          {"single_bonds": 1, "double_bonds": 4, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[O][=C][=C][N][=C][=C]"}  O=C=CN=C=C    0.473684
30   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3144.png         {"single_bonds": 2, "double_bonds": 1, "triple_bonds": 1, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[N][#C][N][=C][O]"}          N#CN=CO            {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 1, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[O][C][C][=N][C][#N]"}    OCC=NC#N    0.473684
13    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/920.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 2, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[O][=N][O][N][C]"}           O=NONC             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 3, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[C][O][N][O][N][=O]"}     CONON=O    0.473684
11   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6725.png          {"single_bonds": 3, "double_bonds": 0, "triple_bonds": 1, "ring_count": 0, "C_count": 2, "O_count": 2, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[N][N][O][C][#C]"}           NNOC#C             {"single_bonds": 4, "double_bonds": 0, "triple_bonds": 1, "ring_count": 0, "C_count": 2, "O_count": 2, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[N][N][O][O][C][#C]"}     NNOOC#C    0.473684
119  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1477.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[C][N][=N][C][C]"}           CN=NCC             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[C][N][=N][C][C][C]"}     CN=NCCC    0.500000
284  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/8001.png        {"single_bonds": 1, "double_bonds": 3, "triple_bonds": 0, "ring_count": 0, "C_count": 0, "O_count": 0, "N_count": 5, "branch_count": 0, "atom_count": 5, "selfies": "[N][=N][N][=N][=N]"}          N=NN=NN           {"single_bonds": 2, "double_bonds": 3, "triple_bonds": 0, "ring_count": 0, "C_count": 0, "O_count": 0, "N_count": 6, "branch_count": 0, "atom_count": 6, "selfies": "[N][=N][N][=N][N][=N]"}   N=NN=NN=N    0.500000
83   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/5691.png           {"single_bonds": 4, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[C][C][C][O][N]"}            CCCON              {"single_bonds": 5, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[N][C][O][C][C][C]"}      NCOCCC    0.529412
180  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3832.png           {"single_bonds": 4, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 2, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[N][C][C][O][O]"}            NCCOO              {"single_bonds": 5, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 3, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[O][O][O][C][C][N]"}      OOOCCN    0.529412
193  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1538.png          {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 0, "N_count": 4, "branch_count": 0, "atom_count": 5, "selfies": "[C][N][=N][N][N]"}           CN=NNN             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 0, "N_count": 5, "branch_count": 0, "atom_count": 6, "selfies": "[C][N][=N][N][N][N]"}     CN=NNNN    0.529412
296    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/52.png           {"single_bonds": 4, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[O][C][N][C][C]"}            OCNCC              {"single_bonds": 5, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[C][C][C][N][C][O]"}      CCCNCO    0.529412
137  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/2219.png        {"single_bonds": 1, "double_bonds": 3, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[C][=N][C][=C][=C]"}         C=NC=C=C          {"single_bonds": 1, "double_bonds": 4, "triple_bonds": 0, "ring_count": 0, "C_count": 5, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[C][=N][C][=C][=C][=C]"}  C=NC=C=C=C    0.529412
86   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3578.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 1, "N_count": 3, "branch_count": 0, "atom_count": 5, "selfies": "[N][N][N][=C][O]"}           NNN=CO             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 1, "N_count": 4, "branch_count": 0, "atom_count": 6, "selfies": "[O][C][=N][N][N][N]"}     OC=NNNN    0.555556
71    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/327.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[C][C][N][=C][=C]"}          CCN=C=C           {"single_bonds": 2, "double_bonds": 3, "triple_bonds": 0, "ring_count": 0, "C_count": 5, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[C][C][N][=C][=C][=C]"}   CCN=C=C=C    0.555556
22   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7723.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 0, "N_count": 3, "branch_count": 0, "atom_count": 5, "selfies": "[N][=N][C][N][C]"}           N=NCNC             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 0, "N_count": 4, "branch_count": 0, "atom_count": 6, "selfies": "[N][=N][C][N][N][C]"}     N=NCNNC    0.555556
114  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3893.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 0, "O_count": 3, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[O][=N][N][O][O]"}           O=NNOO             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 0, "O_count": 4, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[O][O][O][N][N][=O]"}     OOONN=O    0.555556
120  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/1992.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 5, "O_count": 0, "N_count": 0, "branch_count": 0, "atom_count": 5, "selfies": "[C][=C][=C][C][C]"}          C=C=CCC           {"single_bonds": 2, "double_bonds": 3, "triple_bonds": 0, "ring_count": 0, "C_count": 6, "O_count": 0, "N_count": 0, "branch_count": 0, "atom_count": 6, "selfies": "[C][=C][=C][=C][C][C]"}   C=C=C=CCC    0.555556
279  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6256.png           {"single_bonds": 4, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 2, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[N][O][C][N][O]"}            NOCNO              {"single_bonds": 5, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 2, "N_count": 3, "branch_count": 0, "atom_count": 6, "selfies": "[N][O][C][N][N][O]"}      NOCNNO    0.555556
4    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/5898.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[N][C][=C][C][O]"}           NC=CCO             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[N][C][=C][C][C][O]"}     NC=CCCO    0.562500
81   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6629.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 0, "N_count": 3, "branch_count": 0, "atom_count": 5, "selfies": "[N][N][C][=C][N]"}           NNC=CN             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 0, "N_count": 4, "branch_count": 0, "atom_count": 6, "selfies": "[N][N][C][=C][N][N]"}     NNC=CNN    0.583333
50    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/182.png           {"single_bonds": 4, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 2, "N_count": 0, "branch_count": 0, "atom_count": 5, "selfies": "[C][C][O][O][C]"}            CCOOC              {"single_bonds": 5, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 2, "N_count": 0, "branch_count": 0, "atom_count": 6, "selfies": "[C][C][O][O][C][C]"}      CCOOCC    0.583333
240  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3035.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 2, "N_count": 0, "branch_count": 0, "atom_count": 5, "selfies": "[O][=C][C][C][O]"}           O=CCCO             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 2, "N_count": 0, "branch_count": 0, "atom_count": 6, "selfies": "[O][C][C][C][C][=O]"}     OCCCC=O    0.625000
127  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6831.png          {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 0, "O_count": 1, "N_count": 4, "branch_count": 0, "atom_count": 5, "selfies": "[N][N][N][N][=O]"}           NNNN=O             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 0, "O_count": 1, "N_count": 5, "branch_count": 0, "atom_count": 6, "selfies": "[N][N][N][N][N][=O]"}     NNNNN=O    0.625000
46   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/7942.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 0, "N_count": 4, "branch_count": 0, "atom_count": 5, "selfies": "[N][=N][N][N][C]"}           N=NNNC             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 0, "N_count": 5, "branch_count": 0, "atom_count": 6, "selfies": "[N][=N][N][N][N][C]"}     N=NNNNC    0.625000
170  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/3036.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[N][=C][C][C][O]"}           N=CCCO             {"single_bonds": 4, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[O][C][C][C][C][=N]"}     OCCCC=N    0.625000
155    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/10.png           {"single_bonds": 4, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 2, "N_count": 0, "branch_count": 0, "atom_count": 5, "selfies": "[C][C][C][O][O]"}            CCCOO              {"single_bonds": 5, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 2, "N_count": 0, "branch_count": 0, "atom_count": 6, "selfies": "[C][C][C][C][O][O]"}      CCCCOO    0.625000
265  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/4316.png           {"single_bonds": 4, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 0, "O_count": 2, "N_count": 3, "branch_count": 0, "atom_count": 5, "selfies": "[N][O][N][N][O]"}            NONNO              {"single_bonds": 5, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 0, "O_count": 2, "N_count": 4, "branch_count": 0, "atom_count": 6, "selfies": "[O][N][N][N][O][N]"}      ONNNON    0.625000
90   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/6380.png           {"single_bonds": 4, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 0, "O_count": 3, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[N][O][O][O][N]"}            NOOON              {"single_bonds": 5, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 0, "O_count": 4, "N_count": 2, "branch_count": 0, "atom_count": 6, "selfies": "[N][O][O][O][O][N]"}      NOOOON    0.750000
``` 

# 提高数值loss的占比

```
total_loss = 0.8 * ce_loss + 5 * numeric_loss
``` 

原子数5的评估结果: 正确率下降, 且错误的分子数case数跟提高数值前 差不多

```

Overall Similarity: 92.34%
Correct: 269 / 300 = 89.66666666666666 %
                                                          Image Path                                                                                                                                                                                        Predicted Predicted SMILES                                                                                                                                                                                               GT GT SMILES  Similarity
225   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/315.png  {"single_bonds": 2, "double_bonds": 0, "triple_bonds": 1, "ring_count": 1, "C_count": 2, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[C][#C][N][Ring1][Ring1]"}           C1#CN1  {"single_bonds": 2, "double_bonds": 1, "triple_bonds": 0, "ring_count": 1, "C_count": 2, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[C][N][=C][Ring1][Ring1]"}    C1N=C1    0.000000
53    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/138.png             {"single_bonds": 2, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 0, "branch_count": 0, "atom_count": 4, "selfies": "[C][C][=C][C]"}            CC=CC  {"single_bonds": 2, "double_bonds": 0, "triple_bonds": 1, "ring_count": 1, "C_count": 3, "O_count": 0, "N_count": 0, "branch_count": 0, "atom_count": 5, "selfies": "[C][C][#C][Ring1][Ring1]"}    C1C#C1    0.000000
81   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1639.png  {"single_bonds": 2, "double_bonds": 1, "triple_bonds": 0, "ring_count": 1, "C_count": 1, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[N][=C][N][Ring1][Ring1]"}           N1=CN1  {"single_bonds": 2, "double_bonds": 1, "triple_bonds": 0, "ring_count": 1, "C_count": 1, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[N][=N][C][Ring1][Ring1]"}    N1=NC1    0.090909
96   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1186.png  {"single_bonds": 2, "double_bonds": 1, "triple_bonds": 0, "ring_count": 1, "C_count": 2, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[C][=C][N][Ring1][Ring1]"}           C1=CN1   {"single_bonds": 3, "double_bonds": 0, "triple_bonds": 0, "ring_count": 1, "C_count": 2, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[N][C][C][Ring1][Ring1]"}     N1CC1    0.111111
250  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1469.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 1, "N_count": 3, "branch_count": 0, "atom_count": 5, "selfies": "[N][=C][=N][N][O]"}          N=C=NNO         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 1, "N_count": 3, "branch_count": 0, "atom_count": 5, "selfies": "[N][N][=C][=N][O]"}   NN=C=NO    0.136364
123  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1605.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 0, "N_count": 4, "branch_count": 0, "atom_count": 5, "selfies": "[N][N][=C][=N][N]"}          NN=C=NN         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 0, "N_count": 4, "branch_count": 0, "atom_count": 5, "selfies": "[N][=C][=N][N][N]"}   N=C=NNN    0.166667
17    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/728.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[N][C][=N][=C][O]"}           NC=NCO         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[O][C][=C][=N][N]"}   OC=C=NN    0.181818
168  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1348.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 1, "N_count": 3, "branch_count": 0, "atom_count": 5, "selfies": "[N][C][=N][O][N]"}           NC=NON          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 1, "N_count": 3, "branch_count": 0, "atom_count": 5, "selfies": "[N][O][N][C][=N]"}    NONC=N    0.190476
40     /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/15.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[C][C][N][C][=C]"}           CCNC=C          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[C][C][C][N][=C]"}    CCCN=C    0.190476
12    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/101.png         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[C][C][=N][N][=C]"}          CC=NN=C         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[C][C][=C][=N][N]"}   CC=C=NN    0.190476
10    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/809.png           {"single_bonds": 4, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 0, "O_count": 3, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[O][O][N][N][O]"}            OONNO           {"single_bonds": 4, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 0, "O_count": 3, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[O][O][O][N][N]"}     OOONN    0.210526
140    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/52.png             {"single_bonds": 2, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 4, "selfies": "[C][C][N][=C]"}            CCN=C          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[C][C][N][C][=C]"}    CCNC=C    0.222222
273   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/386.png            {"single_bonds": 1, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 4, "selfies": "[O][=C][N][=C]"}           O=CN=C         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[C][=C][N][C][=O]"}   C=CNC=O    0.235294
154   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/387.png            {"single_bonds": 1, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 4, "selfies": "[C][=C][N][=N]"}           C=CN=N         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[C][=C][N][C][=N]"}   C=CNC=N    0.235294
0     /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/790.png          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 3, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[O][C][=N][O][O]"}           OC=NOO          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 3, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[O][O][C][=N][O]"}    OOC=NO    0.263158
132    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/29.png          {"single_bonds": 3, "double_bonds": 0, "triple_bonds": 1, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[N][C][C][#C][C]"}           NCC#CC          {"single_bonds": 3, "double_bonds": 0, "triple_bonds": 1, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[C][C][C][#C][N]"}    CCC#CN    0.263158
176   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/850.png           {"single_bonds": 4, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 3, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[O][C][N][O][O]"}            OCNOO           {"single_bonds": 4, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 3, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[O][N][C][O][O]"}     ONCOO    0.263158
22    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/718.png          {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 2, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[O][=N][C][C][O]"}           O=NCCO         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 2, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[O][C][=C][N][=O]"}   OC=CN=O    0.263158
144   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/823.png           {"single_bonds": 4, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 0, "O_count": 4, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[O][N][O][O][O]"}            ONOOO           {"single_bonds": 4, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 0, "O_count": 4, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[O][O][N][O][O]"}     OONOO    0.266667
100   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/775.png           {"single_bonds": 3, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 4, "N_count": 0, "branch_count": 0, "atom_count": 5, "selfies": "[O][C][O][O][O]"}            OCOOO           {"single_bonds": 4, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 4, "N_count": 0, "branch_count": 0, "atom_count": 5, "selfies": "[O][O][C][O][O]"}     OOCOO    0.266667
59    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/535.png            {"single_bonds": 1, "double_bonds": 0, "triple_bonds": 2, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 0, "branch_count": 0, "atom_count": 4, "selfies": "[C][#C][C][#C]"}           C#CC#C         {"single_bonds": 2, "double_bonds": 0, "triple_bonds": 2, "ring_count": 0, "C_count": 5, "O_count": 0, "N_count": 0, "branch_count": 0, "atom_count": 5, "selfies": "[C][#C][C][C][#C]"}   C#CCC#C    0.300000
110   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/360.png             {"single_bonds": 2, "double_bonds": 0, "triple_bonds": 1, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 0, "branch_count": 0, "atom_count": 4, "selfies": "[C][C][#C][C]"}            CC#CC         {"single_bonds": 2, "double_bonds": 1, "triple_bonds": 1, "ring_count": 0, "C_count": 5, "O_count": 0, "N_count": 0, "branch_count": 0, "atom_count": 5, "selfies": "[C][=C][C][#C][C]"}   C=CC#CC    0.307692
299  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1250.png            {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 4, "selfies": "[N][C][=C][=O]"}           NC=C=O         {"single_bonds": 2, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[N][C][=C][=C][O]"}   NC=C=CO    0.312500
285   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/536.png            {"single_bonds": 1, "double_bonds": 0, "triple_bonds": 2, "ring_count": 9, "C_count": 3, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 4, "selfies": "[N][#C][C][#C]"}           N#CC#C         {"single_bonds": 2, "double_bonds": 0, "triple_bonds": 2, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[C][#C][C][C][#N]"}   C#CCC#N    0.333333
188   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/156.png             {"single_bonds": 2, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 0, "branch_count": 0, "atom_count": 4, "selfies": "[C][O][C][=C]"}            COC=C          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 1, "N_count": 0, "branch_count": 0, "atom_count": 5, "selfies": "[C][O][C][=C][C]"}    COC=CC    0.333333
207    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/34.png             {"single_bonds": 2, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 1, "N_count": 0, "branch_count": 0, "atom_count": 4, "selfies": "[C][C][O][=C]"}             CCOC          {"single_bonds": 3, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 1, "N_count": 0, "branch_count": 0, "atom_count": 5, "selfies": "[C][C][O][C][=C]"}    CCOC=C    0.400000
1    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1707.png            {"single_bonds": 1, "double_bonds": 2, "triple_bonds": 1, "ring_count": 0, "C_count": 2, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 4, "selfies": "[N][#C][C][=N]"}           N#CC=N        {"single_bonds": 1, "double_bonds": 2, "triple_bonds": 1, "ring_count": 0, "C_count": 3, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[N][#C][C][=C][=N]"}  N#CC=C=N    0.400000
210   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/147.png              {"single_bonds": 3, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 2, "N_count": 0, "branch_count": 0, "atom_count": 4, "selfies": "[C][O][C][O]"}             COCO           {"single_bonds": 4, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 2, "N_count": 0, "branch_count": 0, "atom_count": 5, "selfies": "[C][O][C][O][C]"}     COCOC    0.416667
65    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/214.png              {"single_bonds": 3, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 4, "selfies": "[C][N][C][C]"}             CNCC           {"single_bonds": 4, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 5, "selfies": "[C][N][C][C][C]"}     CNCCC    0.461538
113  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1159.png              {"single_bonds": 3, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 4, "selfies": "[N][C][C][N]"}             NCCN           {"single_bonds": 4, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 3, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 5, "selfies": "[N][C][C][C][N]"}     NCCCN    0.500000
230     /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_5/1.png              {"single_bonds": 3, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 4, "O_count": 0, "N_count": 0, "branch_count": 0, "atom_count": 4, "selfies": "[C][C][C][C]"}             CCCC           {"single_bonds": 4, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 5, "O_count": 0, "N_count": 0, "branch_count": 0, "atom_count": 5, "selfies": "[C][C][C][C][C]"}     CCCCC    0.500000

``` 

# 下一步: 

  - 对比不修改loss, 使用传统交叉熵的情况
  - 考虑不用数据筛选, 用所有1-4的数据, 增大数据量进行尝试
  - 另外需要研究数据, 看模型是如何学得这个知识的 (这个现象在没增加 原子数 等元信息时, 好像出现概率不高)
