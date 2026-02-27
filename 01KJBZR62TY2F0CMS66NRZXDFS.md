---
title: 20250203 - 阅读论文: Golden-Retriever: High-Fidelity Agentic Retrieval Augmented Generation for Industrial Knowledge Base
confluence_page_id: 3344127
created_at: 2025-02-03T04:54:09+00:00
updated_at: 2025-02-03T04:54:09+00:00
---

# 主要思路

Golden-Retriever 引入了一个术语查询解释装置, 主要解决问题: 领域特定的术语和缩写词使得在这些文档中进行有效检索变得更加复杂

Golden-Retriever 的"基于反思的问题增强步骤"，发生在在线处理阶段，包含以下四个关键步骤：

  1. **识别术语 (Identify Jargons):** 这一步的目标是识别用户问题中所有可能存在的领域特定术语和缩写。系统使用LLM并结合提示模板来完成这项任务。LLM 不仅能识别已知的术语，还能识别拼写错误或未在词典中录入的新术语。

     - **示例:** 用户提问："What is the PUC architecture of Samsung or Hynix NAND chip?"
     - **输出:** ["PUC", "NAND"]
  2. **识别上下文 (Identify Context):** 确定问题的上下文对于正确理解术语至关重要。相同的术语在不同的上下文中可能有不同的含义。系统使用包含预定义上下文列表的提示模板，引导 LLM 判断用户问题的上下文。

     - **示例:** 用户提问 (同上): "What is the PUC architecture of Samsung or Hynix NAND chip?"
     - **预定义上下文列表:** "Process Question", "Design Question", "Manufacturing Question", "Test Question", ...
     - **输出:** "Design Question"
  3. **查询术语 (Query Jargons):** 一旦识别了术语和上下文，系统就会查询预先构建的术语词典，获取术语的详细定义、描述和相关注释。系统使用 SQL 查询数据库，而不是依赖 LLM 生成 SQL 查询，以确保查询的安全性和可靠性。

     - **示例:** 输入术语列表: ["PUC", "NAND"] 和上下文: "Design Question"
     - **SQL 查询:** `SELECT * FROM Design WHERE Jargon IN ('PUC', 'NAND');`
     - **输出:**
       - PUC: Peripheral Under Cell，一种将外围电路放置在存储单元下方的架构...
       - NAND: 非易失性闪存的一种类型...
  4. **增强问题 (Augment Question):** 最后，系统将原始问题、上下文信息和术语定义整合在一起，形成一个增强问题。这个增强问题更加清晰、完整，消除了歧义，有助于 RAG 框架检索到最相关的文档。

     - **示例:**
       - 原始问题: "What is the PUC architecture of Samsung or Hynix NAND chip?"
       - 增强问题: "Answer the following *Design Question* : What is the Peripheral Under Cell (PUC) architecture of Samsung or Hynix NAND flash chip? PUC is an architecture defined as … NAND flash refers to …"

通过这四个步骤，Golden-Retriever 能够有效地理解用户的问题，即使问题中包含模糊或不完整的术语，也能检索到最相关的文档，最终提高问答的准确性。
