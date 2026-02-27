---
title: 20240819 - 阅读论文: LEAST-TO-MOST PROMPTING ENABLES COMPLEX REASONING IN LARGE LANGUAGE MODELS
confluence_page_id: 3146199
created_at: 2024-08-19T09:25:12+00:00
updated_at: 2024-08-19T09:25:12+00:00
---

# 主要观点

CoT 在解决 "比示例更难的问题" 时, 性能下降很厉害

```
在论文的 3.1 节 “符号操作” 中，以 “last-letter-concatenation” 任务为例：
任务目标: 将单词列表中每个单词的最后一个字母拼接起来。例如，输入 "thinking, machine"，输出 "ge"。
问题: 当测试集中的单词列表长度比提示示例中的列表长度更长时，思维链提示方法的性能会显著下降。
原因: 思维链提示方法学习的是示例中的推理模式，而当测试集中的问题复杂度超过示例时，模型无法进行有效的泛化。
``` 

本论文提出的方法: 将 问题拆解成各个子问题, 对各个子问题进行求解 (没什么特别的技巧)
