---
title: 20240819 - 阅读论文: Principled Instructions Are All You Need for Questioning LLaMA-1/2, GPT-3.5/4
confluence_page_id: 3146194
created_at: 2024-08-19T08:11:48+00:00
updated_at: 2024-08-20T12:43:43+00:00
---

# 主要内容

论文中描述了 提示词的一些设计原则, 包括: 

```
## 论文中给出的设计原则分类列表:

这篇论文将 26 条设计原则归纳为五大类，以下逐条列出：

**一、提示词结构和清晰度 (Prompt Structure and Clarity)**

1. **简洁和清晰 (Conciseness and Clarity):**  提示词应简洁易懂，避免冗余信息，同时也要足够具体，能够引导模型理解任务。
2. **肯定指令 (Affirmative Directives):**  使用“做...”、“写...”等肯定指令，避免使用“不要...”、“避免...”等否定语言。
3. **引导词 (Leading Words):**  使用引导词，例如在需要模型逐步思考时，可以在提示词中加入“逐步思考”。
4. **输出引导 (Output Primers):**  在提示词末尾加入预期输出的开头部分，引导模型生成符合预期的内容。
5. **分隔符 (Delimiters):**  使用分隔符清晰地划分提示词的不同部分，例如指令、示例、问题、上下文和输入数据等。
6. **格式规范 (Formatted Prompting):**  使用统一的格式规范编写提示词，例如使用 "###Instruction###"、"###Example###"、"###Question###" 等标识符来区分不同部分。

**二、具体性和信息量 (Specificity and Information)**

7. **上下文相关性 (Contextual Relevance):**  提供与任务相关的背景信息，例如关键词、领域术语或情景描述，帮助模型理解任务。
8. **任务对齐 (Task Alignment):**  提示词应与任务目标紧密 aligned，使用清晰的语言和结构指示任务类型，例如以问题、命令或填空的形式呈现。
9. **示例演示 (Example Demonstrations):**  对于复杂任务，在提示词中加入示例可以帮助模型理解预期输出的格式或类型，例如输入-输出对。
10. **避免偏见 (Avoiding Bias):**  使用中立的语言，避免使用可能引发模型偏见的词汇或表达，特别是在处理敏感话题时。
11. **相似语言 (Similar Language):**  如果希望模型生成与特定文本风格相似的输出，可以在提示词中加入“使用与[提供的段落/标题/文本/答案]相同的语言”等指令。
12. **关键词和规范 (Keywords and Regulations):**  明确说明模型生成内容时需要遵循的要求，例如关键词、规则、提示或指令。

**三、用户交互和参与 (User Interaction and Engagement)**

13. **增量提示 (Incremental Prompting):**  将复杂任务分解成一系列简单的步骤，逐步引导模型完成任务。
14. **迭代反馈 (Iterative Feedback):**  根据模型的初始输出和行为，不断调整和优化提示词。
15. **主动提问 (Eliciting Questions):**  允许模型通过提问来获取更多信息，例如在提示词中加入“从现在开始，我希望你问我问题，以便……”等指令。

**四、内容和语言风格 (Content and Language Style)**

16. **目标受众 (Target Audience):**  在提示词中明确目标受众，例如“目标受众是该领域的专家”或“目标受众是5岁的孩子”。
17. **角色扮演 (Role Assignment):**  为模型设定一个角色，例如“你是一位历史学家”或“你是一位诗人”。
18. **自然语言 (Natural Language):**  使用自然语言编写提示词，避免使用过于正式或过于口语化的表达。
19. **简洁直接 (Concise and Direct):**  避免使用“请”、“如果你不介意”、“谢谢”等客套话，直接表达需求。
20. **重复强调 (Repetition):**  在提示词中多次重复特定的词语或短语，以强调其重要性。

**五、复杂任务和代码提示词 (Complex Tasks and Coding Prompts)**

21. **步骤分解 (Task Decomposition):**  将复杂任务分解成多个简单的子任务，并使用多个提示词引导模型逐步完成。
22. **代码生成 (Code Generation):**  对于涉及代码生成的提示词，需要明确说明编程语言、代码格式和功能要求等。
23. **多文件代码 (Multi-file Code):**  如果需要模型生成跨越多个文件的代码，需要提供清晰的文件结构和代码组织方式。
24. **思维链 (Chain-of-Thought):**  将思维链方法与少样本提示相结合，引导模型进行逻辑推理和问题解决。
25. **详细内容 (Detailed Content):**  如果需要模型生成详细的内容，例如文章或段落，可以在提示词中加入“详细说明”或“添加所有必要信息”等指令。
26. **文本修改 (Text Revision):**  如果需要模型修改文本但保留其风格，可以在提示词中明确说明修改要求，例如“只修改语法和词汇，保持原文风格”。

希望以上信息对您有所帮助。
``` 

这些原则都比较常见. 

另需分析其 related works 部分

# related works

1 complete 20: Autoprompt: Eliciting knowledge from language models with automatically generated prompts 2 complete 2: Language models are few-shot learners (few-shot是有效的) 3 complete 1: Ask me anything: A simple strategy for prompting language models 4 complete 27: Least-to-most prompting enables complex reasoning in large language models 5 complete 8: Can language models learn from explanations in context (答案后解释 也可以增强答案的正确性) 6 complete 25: A prompt pattern catalog to enhance prompt engineering with chatgpt 7 incomplete 12: Guiding large language models via directional stimulus prompting
