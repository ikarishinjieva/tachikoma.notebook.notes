---
title: 20241005 - 阅读论文*: REAP: Enhancing LLM Problem Solving with REAP: Reflection, Explicit Problem Deconstruction, and Advanc
confluence_page_id: 3342625
created_at: 2024-10-05T16:46:15+00:00
updated_at: 2024-10-05T16:46:15+00:00
---

# 论文目标

```
REAP 方法旨在解决现有 LLM 提示策略在处理复杂推理问题时的一些关键局限性。这些局限性可以概括如下：

1. **缺乏整合性**: 现有的提示策略通常将问题理解、推理和迭代改进视为孤立的步骤，缺乏将它们有效整合到统一框架内的机制。这种割裂式的处理方式导致模型在处理复杂任务时，难以保持一致性和连贯性。例如，模型可能在理解问题的某个方面时表现良好，但在将其与其他方面整合以进行推理时却出现错误。

2. **零样本提示的不足**: 零样本提示（Zero-shot prompting）虽然允许模型在没有特定训练数据的情况下处理新任务，但其推理深度和准确性往往有限。模型难以捕捉到复杂问题中的细微之处，容易生成不完整或不相关的答案。

3. **思维链提示的连贯性问题**: 思维链提示（Chain-of-thought prompting）通过引导模型生成一系列中间推理步骤来提高其推理能力。然而，在复杂任务中，维持这些推理步骤的连贯性和逻辑一致性仍然是一个挑战。模型容易偏离正确的推理路径，或者在不同步骤之间产生矛盾。

4. **高级提示方法的效率和一致性问题**:  像思维树（Tree-of-thought）这样的高级提示方法虽然能够探索多种推理路径，但同时也增加了计算复杂度，并可能导致模型难以在众多可能性中选择最佳方案。此外，这些方法也难以保证不同推理分支之间的一致性，可能导致模型生成逻辑冲突的答案。

5. **反思机制的局限性**: 一些提示策略引入了反思机制，允许模型对自身的输出进行评估和改进。然而，这些反思机制通常受限于模型初始的知识和能力，如果初始阶段出现错误，可能会导致错误在后续步骤中传播和放大。

6. **缺乏可解释性**: 许多现有的提示策略难以解释模型是如何得出最终答案的。这使得用户难以理解模型的推理过程，也难以评估模型输出的可靠性。尤其在一些对可解释性要求较高的领域（如医疗、金融），这种缺乏透明度会限制 LLM 的应用。

总而言之，REAP 方法通过将反思、显式问题解构和高级提示整合到一个统一的动态上下文生成框架中，旨在克服上述局限性。它鼓励模型对问题进行更深入、更系统的分析，并通过迭代改进和反思来提高输出的质量、连贯性和可解释性。
``` 

# REAP方法

```
REAP 方法包含三个核心组件：反思、显式问题解构和高级提示。这些组件被整合到一个统一的提示框架中，并按照以下步骤执行：

**阶段一：反思 (Reflection)**

1. **字面解释规则 (Literal Interpretation Rule):**  模型首先要严格按照字面意思理解问题陈述中的每个语句，避免任何隐含意义或推断。这一步旨在防止模型在早期阶段就因为误解而偏离正确的方向。

2. **严格解释规则 (Strict Interpretation Rule):**  模型必须严格遵守问题陈述中提供的信息，不得做出任何假设或推论。如果信息不足以得出结论，模型应明确指出。

3. **关键洞察检查 (Key Insight Check):** 模型需要回顾所有已识别出的特征和过程，寻找可能简化问题或直接指向解决方案的关键洞察。

4. **伦理检查和不确定性下的决策 (Ethical Check and Decision-Making Under Uncertainty):** 模型需要评估潜在解决方案的伦理含义和风险，尤其是在结果不确定的情况下。

5. **贝叶斯思维 (Bayesian Thinking):**  如果问题中提供了新的显式信息，模型需要使用贝叶斯推理来更新其对问题的理解和解决方案的评估。

**阶段二：显式问题解构 (Explicit Problem Deconstruction)**

1. **全面特征分析 (Comprehensive Feature Analysis):** 模型需要从问题陈述中提取所有相关的特征、参与者、动作和关系，并列出清单。

2. **顺序和机械过程检查 (Sequential and Mechanical Process Check):**  模型需要分析问题中描述的任何顺序、循环或机械过程，并考虑这些过程如何影响结果。

3. **已知和推断信息 (Known and Deduced Information):**  模型需要列出所有已知的事实，并根据已知信息进行逻辑推断。

4. **问题分解 (Problem Decomposition):** 模型需要将复杂问题分解成更小、更易于管理的子问题。

5. **空间和对象分析 (Spatial and Object Analysis):**  如果问题涉及空间关系或对象属性，模型需要进行空间和对象分析。

**阶段三：高级提示 (Advanced Prompting)**

1. **思维图 (Graph of Thought):** 模型构建一个图来表示问题的结构，节点代表不同的概念或状态，边代表它们之间的关系。

2. **多种解决方案生成 (Multiple Solution Generation):** 基于收集到的信息和思维图，模型尝试生成多种可能的解决方案。

3. **最快和最简单解决方案识别 (Quickest and Easiest Solution Identification):**  从生成的多个解决方案中，模型尝试识别最快捷、最简单的解决方案。

4. **最终输出和建议 (Final Output & Recommendation):** 模型整合所有信息和分析结果，给出最终的答案和建议。

**输出格式:**

最终输出需要包含以上所有步骤的分析结果，并按照以下格式呈现：

* REAP 结果：
    * a. 全面特征列表
    * b. 顺序和机械过程检查
    * c. 关键洞察检查结果
    * d. 重新表述的问题
    * e. 已知和推断信息
    * f. 问题分解
    * g. 思维图
    * h. 空间和对象分析
    * i. 贝叶斯更新
    * j. 伦理检查和不确定性下的决策
    * k. 多种解决方案
    * l. 最快和最简单解决方案
    * m. 反思

* 建议：

通过以上步骤，REAP 方法引导 LLM 对问题进行全面、系统的分析，并通过反思和迭代改进，最终生成更准确、更连贯、更具解释性的答案。
``` 

# 逐步骤举例

```
问题: Question 18: Write a grammatically correct sentence without reusing any letter more than once.
 
a. 全面特征列表
a. **Comprehensive Feature List**:
- **Action**: "[Write]"
- **Object**: "[a grammatically correct sentence]"
- **Constraint**: "[without reusing any letter more than once]"

* b. 顺序和机械过程检查
b. **Sequential and Mechanical Process Check**:
- The action "Write" leads to creating "a grammatically correct sentence" under the constraint "[without reusing any letter more than once]".

* c. 关键洞察检查结果
c. **Key Insight Check Results**:
- **Key Insight**: Utilizing short words with unique letters allows for constructing a sentence that meets the constraint.

* d. 重新表述的问题
d. **Rephrased Question**:
- Can a grammatically correct sentence be written without reusing any letter more than once?

* e. 已知和推断信息
e. **Known and Deduced Information**:
- "[Write]" - Instruction to compose.
- "[a grammatically correct sentence]" - The sentence must follow grammatical rules.
- "[without reusing any letter more than once]" - Each letter can be used only once.
- **Deduction**: The sentence must consist of words that do not share any letters.

* f. 问题分解
f. **Problem Decomposition**:
- **Component 1**: Compose a sentence.
- **Supporting Quote**: "[Write]"
- **Component 2**: Ensure grammatical correctness.
- **Supporting Quote**: "[a grammatically correct sentence]"
- **Component 3**: Use unique letters throughout.
- **Supporting Quote**: "[without reusing any letter more than once]"
 
* g. 思维图
g. **Graph of Thought**:
- **Nodes**: Letters -> Words -> Sentence.
- **Connections**: Select words with non-overlapping letters to form a sentence adhering to grammatical rules.

* h. 空间和对象分析
h. **Spatial and Object Analysis**:
- **Objects**:
- Letters: Building blocks of words.
- Words: Components of the sentence.
- **Spatial Arrangement**: Letters form words without repeating; words are ordered to create a grammatically correct sentence.

* i. 贝叶斯更新
i. **Bayesian Updates**:
- Given the constraints, shorter sentences with fewer letters are more likely to meet the requirements.

* j. 伦理检查和不确定性下的决策
j. **Ethical Check and Decision-Making Under Uncertainty**:
- No ethical concerns identified; proceeding with solutions that fulfill the given constraints.

* k. 多种解决方案
k. **Multiple Solutions**:
1. **Solution 1**:
- Sentence: "Go!"
- **Supporting Information**: Uses letters G and O once each.
2. **Solution 2**:
- Sentence: "Fly!"
- **Supporting Information**: Uses letters F, L, and Y once each.
3. **Solution 3**:
- Sentence: "I go."
- **Supporting Information**: Uses letters I, G, and O once each.

* l. 最快和最简单解决方案
l. **Quickest and Easiest Solution**:
- "Go!" is a grammatically correct sentence that meets the constraint with the
fewest letters.

* m. 反思
m. **Reflection**:
- The solutions adhere strictly to the constraints.
- No assumptions were made beyond the explicit information.
- The answers directly address the question using only the provided information.
 
结论:
**Recommendation:**
The sentence **"Go!"** fulfills all the given requirements. It is grammatically
correct and does not reuse any letter more than once.
 
``` 

说明: 

  - "重新表述的问题"这个步骤, 在论文中的例子中, 并没有太大作用. 应有的作用是 去除冗余信息, 明确目标等.
  - "空间和对象分析" 在不涉及 空间位置 的问题中, 可以不使用
  - "贝叶斯思维" 是分析 新的显式信息, 来更新对问题的理解 (在获得新的证据后更新对某个假设的置信度). 例子中的作用不明显

# 作用

  - 可参考其中的思维过程, 对提示词进行优化
