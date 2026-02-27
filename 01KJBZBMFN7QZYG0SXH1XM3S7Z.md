---
title: 20240725 - LlamaIndex Property Graph Index 分析
confluence_page_id: 3145880
created_at: 2024-07-25T16:21:28+00:00
updated_at: 2024-07-26T11:29:38+00:00
---

LlamaIndex提供了使用 知识图谱 增强 RAG的流程

文档: <https://docs.llamaindex.ai/en/stable/module_guides/indexing/lpg_index_guide/>

文档逻辑: 

```
这篇文档主要介绍了 LlamaIndex 中 Property Graph Index 的使用方法，按照以下叙事结构展开：

**1. 简介**

* 简要介绍了什么是 Property Graph 以及 LlamaIndex 中 PropertyGraphIndex 的作用，包括构建图谱、查询图谱等。

**2.  基本使用方法**

* 展示了 PropertyGraphIndex 的基本使用方法，包括：
    * 从文档中创建索引
    * 使用索引进行检索和查询
    * 保存和加载索引
    * 从现有的图存储和向量存储中加载索引

**3.  图谱构建**

*  详细介绍了如何使用 `kg_extractors` 从文本中提取信息构建图谱，并介绍了以下几种默认的 `kg_extractors`：
    *  `SimpleLLMPathExtractor`
    *  `ImplicitPathExtractor`
    *  `DynamicLLMPathExtractor`
    *  `SchemaLLMPathExtractor`

**4.  检索和查询**

*  介绍了如何使用不同的检索器从图谱中检索节点和路径，并介绍了以下几种检索器：
    *  `LLMSynonymRetriever`
    *  `VectorContextRetriever`
    *  `TextToCypherRetriever`
    *  `CypherTemplateRetriever`
    *  `CustomPGRetriever`

**5.  存储**

*  介绍了 Property Graph Index 支持的图数据库，包括：
    *  `SimplePropertyGraphStore`
    *  `Neo4jPropertyGraphStore`
    *  `NebulaPropertyGraphStore`
    *  `TiDBPropertyGraphStore`
*  并以 `SimplePropertyGraphStore` 和 `Neo4jPropertyGraphStore` 为例，分别介绍了如何将索引保存到磁盘和从磁盘加载，以及如何与外部向量数据库（如 Qdrant）结合使用。

**6.  直接使用 Property Graph Store**

*  介绍了如何直接使用 `PropertyGraphStore` 类来创建、插入和查询图谱数据。

**7.  高级定制**

*  介绍了如何通过子类化 `TransformComponent` 和 `BasePGRetriever` 来分别定制图谱提取器和检索器。

**8.  示例**

*  列举了一些使用 PropertyGraphIndex 的示例 notebooks，包括：
    *  基本使用方法
    *  使用 Neo4j
    *  使用 Nebula
    *  使用 Neo4j 和本地模型的高级用法
    *  使用 Property Graph Store
    *  创建自定义图谱检索器
    *  比较 KG 提取器

总而言之，这篇文档以循序渐进的方式，从基本概念到高级定制，全面介绍了 LlamaIndex 中 Property Graph Index 的使用方法，并辅以丰富的代码示例，方便用户快速上手和进行个性化定制。 
``` 

重点组件的工作机制: 

```
## 图谱构建组件功能差异

以下是 LlamaIndex 中几种常见图谱构建组件（`kg_extractors`）的功能差异：

* **SimpleLLMPathExtractor:** 
    - 使用 LLM 提示并解析文本，提取简单关系（主谓宾三元组）。
    - 可以自定义提示和解析函数。
    - 适用场景：提取文本中简单的、显式表达的关系。

* **ImplicitPathExtractor:** 
    - 解析 LlamaIndex 节点的 `node.relationships` 属性，提取关系。
    - 不需要 LLM 或嵌入模型。
    - 适用场景：节点关系已经预先定义好的情况，例如从结构化数据构建图谱。

* **DynamicLLMPathExtractor:** 
    - 根据预定义的实体类型和关系类型列表，提取关系。
    - 可以灵活地定义允许的实体和关系类型，但不会强制执行。
    - 适用场景：需要对实体和关系类型进行一定限制，但又希望保持一定灵活性。

* **SchemaLLMPathExtractor:** 
    - 根据严格定义的图谱模式（Schema）提取关系。
    - 使用 pydantic 进行验证，确保提取的关系符合预定义的模式。
    - 可以自定义提示、模式和验证规则。
    - 适用场景：需要严格控制图谱结构和关系类型的场景，例如构建领域知识图谱。

## 检查查询组件功能差异

以下是 LlamaIndex 中几种常见图谱查询组件（`Retrievers`）的功能差异：

* **LLMSynonymRetriever:** 
    - 使用 LLM 生成与查询相关的关键词和同义词。
    - 根据生成的关键词检索图谱节点。
    - 可以自定义提示和解析函数。
    - 适用场景：查询语义较为模糊，需要借助 LLM 扩展查询词的情况。

* **VectorContextRetriever:** 
    - 根据嵌入向量计算查询与图谱节点的相似度。
    - 返回相似度最高的节点及其相关路径。
    - 需要预先对图谱节点进行嵌入。
    - 适用场景：需要根据语义相似度进行检索，并且图谱节点已经进行嵌入的情况。

* **TextToCypherRetriever:** 
    - 使用 LLM 将自然语言查询转换为 Cypher 查询语句。
    - 在支持 Cypher 查询的图数据库上执行查询。
    - 需要预先定义图数据库的 Schema。
    - 适用场景：需要使用自然语言查询图数据库，并且图数据库支持 Cypher 查询的情况。

* **CypherTemplateRetriever:** 
    - 使用预定义的 Cypher 查询模板，并使用 LLM 填充模板参数。
    - 在支持 Cypher 查询的图数据库上执行查询。
    - 需要预先定义 Cypher 查询模板和参数类型。
    - 适用场景：需要使用结构化查询语句查询图数据库，并且希望使用 LLM 辅助生成查询参数的情况。

* **CustomPGRetriever:** 
    - 允许用户自定义检索逻辑，实现更灵活的查询方式。
    - 适用场景：需要根据特定需求定制检索逻辑的情况。
``` 

重点: DynamicLLMPathExtractor使用的提示词: 

<https://github.com/run-llama/llama_index/blob/15112b76d1e205c9686ddd4eba73311081916241/llama-index-core/llama_index/core/prompts/default_prompts.py#L336>

```
DEFAULT_DYNAMIC_EXTRACT_TMPL = (
    "Extract up to {max_knowledge_triplets} knowledge triplets from the given text. "
    "Each triplet should be in the form of (head, relation, tail) with their respective types.\n"
    "---------------------\n"
    "INITIAL ONTOLOGY:\n"
    "Entity Types: {allowed_entity_types}\n"
    "Relation Types: {allowed_relation_types}\n"
    "\n"
    "Use these types as a starting point, but introduce new types if necessary based on the context.\n"
    "\n"
    "GUIDELINES:\n"
    "- Output in JSON format: [{{'head': '', 'head_type': '', 'relation': '', 'tail': '', 'tail_type': ''}}]\n"
    "- Use the most complete form for entities (e.g., 'United States of America' instead of 'USA')\n"
    "- Keep entities concise (3-5 words max)\n"
    "- Break down complex phrases into multiple triplets\n"
    "- Ensure the knowledge graph is coherent and easily understandable\n"
    "---------------------\n"
    "EXAMPLE:\n"
    "Text: Tim Cook, CEO of Apple Inc., announced the new Apple Watch that monitors heart health. "
    "UC Berkeley researchers studied the benefits of apples.\n"
    "Output:\n"
    "[{{'head': 'Tim Cook', 'head_type': 'PERSON', 'relation': 'CEO_OF', 'tail': 'Apple Inc.', 'tail_type': 'COMPANY'}},\n"
    " {{'head': 'Apple Inc.', 'head_type': 'COMPANY', 'relation': 'PRODUCES', 'tail': 'Apple Watch', 'tail_type': 'PRODUCT'}},\n"
    " {{'head': 'Apple Watch', 'head_type': 'PRODUCT', 'relation': 'MONITORS', 'tail': 'heart health', 'tail_type': 'HEALTH_METRIC'}},\n"
    " {{'head': 'UC Berkeley', 'head_type': 'UNIVERSITY', 'relation': 'STUDIES', 'tail': 'benefits of apples', 'tail_type': 'RESEARCH_TOPIC'}}]\n"
    "---------------------\n"
    "Text: {text}\n"
    "Output:\n"
)
``` 

其中抽取了实体的类型

VectorContextRetriever 会使用 查询和str(Node)来做向量匹配. 

  - <https://github.com/run-llama/llama_index/blob/d15f45bcec205a4cd0c8f2a4b7c829385e2910e1/llama-index-integrations/graph_stores/llama-index-graph-stores-neo4j/llama_index/graph_stores/neo4j/neo4j_property_graph.py#L254> , store中存储方法, 都需要应对EntityNode和ChunkNode
  - ChunkNode的str是 返回text属性, EntityNode的str是返回name
  - llama_index中默认的extractor, 都不涉及如何构建ChunkNode

# Langchain中抽取图的提示词

<https://github.com/langchain-ai/langchain/blob/b65ac8d39c8b42b08a8a7218d9f915d469793346/libs/experimental/langchain_experimental/graph_transformers/llm.py#L203>
