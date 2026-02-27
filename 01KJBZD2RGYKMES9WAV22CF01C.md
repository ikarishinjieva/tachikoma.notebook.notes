---
title: 20240804 - 阅读论文*: ToolCoder: Teach Code Generation Models to use API search tools
confluence_page_id: 3146000
created_at: 2024-08-04T04:13:15+00:00
updated_at: 2024-08-04T04:13:15+00:00
---

# 要解决的问题

解决生成代码中, 使用调用其他库的API的不准确性

# 主要思路

将"何时应该查询什么API" 这个知识, 通过微调训练到LLM中.

![image2024-8-4 12:10:55.png](/assets/01KJBZD2RGYKMES9WAV22CF01C/image2024-8-4%2012%3A10%3A55.png)

推理时, LLM生成代码, 代码中带有 "搜索API" 的调用要求. 之后通过RAG的方式完善代码

```
ToolCoder 的步骤可以概括为以下三个主要阶段：

**第一阶段：数据标注 (Data Annotation)**

1. **选择基础数据集 (Base Dataset Selection):**  选择一个已有的代码数据集作为基础，例如论文中使用的 CodeSearchNet-Python 数据集。
2. **设计 ChatGPT 提示词 (Prompt Selection):**  创建一个详细的提示词，指导 ChatGPT 如何为代码添加 API 搜索工具调用过程的标注信息。提示词应包含：
    * ChatGPT 的角色：数据标注器
    * 标注格式：`<API>APISearch(query)->answer</API>`，其中 query 是描述所需 API 功能的搜索查询，answer 是搜索返回的 API。
    * 一些示例：提供一些人工标注的代码片段，帮助 ChatGPT 理解任务。
3. **使用 ChatGPT 生成标注数据:** 将基础数据集和提示词输入 ChatGPT，让其自动生成带有 API 搜索工具调用过程标注的代码数据。
4. **过滤和清理数据 (Filter and Clean):**  对 ChatGPT 生成的标注数据进行过滤和清理，去除格式错误、语义不符或质量较低的样本，最终得到高质量的标注数据集。

**第二阶段：参数高效微调 (Parameter-efficient Fine-tuning)**

1. **选择预训练模型:** 选择一个预训练的代码生成模型作为基础，例如论文中使用的 CodeGen-350M 和 CodeGen-2B。
2. **应用 LoRA 技术:** 使用 LoRA (Low-Rank Adaptation) 技术对预训练模型进行参数高效微调，只训练模型中一小部分参数，降低训练成本并提高训练效率。
3. **使用标注数据集进行微调:**  使用第一阶段得到的标注数据集对预训练模型进行微调，使模型学会生成包含 API 搜索工具调用过程的代码。

**第三阶段：增强工具的推理 (Inference enhanced with Tools)**

1. **接收自然语言描述:** 接收用户输入的自然语言描述，例如“将数组中所有元素的值加 1”。
2. **代码生成与 API 搜索:**  使用微调后的模型进行代码生成，当模型生成到需要调用 API 的部分时：
    * 生成 API 搜索查询 (query)：模型根据上下文生成描述所需 API 功能的搜索查询。
    * 调用 API 搜索工具：使用预先定义好的 API 搜索工具 (例如在线搜索引擎或文档搜索工具) 进行搜索。
    * 获取搜索结果 (answer)：获取搜索工具返回的 API 结果。
    * 将搜索结果插入代码：将搜索结果插入到生成的代码中，替换掉原本的 API 搜索调用过程。
3. **输出最终代码:**  输出最终生成的代码，其中包含了根据搜索结果选择的 API。

总而言之，ToolCoder 通过数据标注、模型微调和推理过程中的工具增强，实现了将 API 搜索工具集成到代码生成模型中，从而提高了模型在代码生成过程中选择合适 API 的能力。
```
