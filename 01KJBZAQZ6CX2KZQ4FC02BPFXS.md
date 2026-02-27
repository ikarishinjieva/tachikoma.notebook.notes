---
title: 20240718 - ChatDBA: 如何使用知识类的文档
confluence_page_id: 3145787
created_at: 2024-07-18T08:59:31+00:00
updated_at: 2024-07-23T12:01:12+00:00
---

# 先梳理已有的知识类的文档

地址: <http://10.186.18.21/llm/dataprocess>

基本说明:

old_data 表示现在在项目中使用的文章  
\- doc开头的文件夹表示原文   
\- 其中 doc_189 doc_619为工单没有保存文,保存的是切分后的chunk ，其他doc文件夹保存的是原文，  
\- TextSplicer_save文件夹表示重写后的文章

error_doc 表示故障类型文章   
\- 分为MySQL 与PG 的故障文章   
\- origin_doc 表示原文所在目录   
\- rewrite_doc 表示重写结果（评分均通过）所在目录

other_doc 表示其他类型文章   
\- 分为MySQL ,PG ,其他类数据库的，所有非故障张类型文章   
\- 每个目录下分为多个子目录 是开启多个进程爬取的结果 由于名字可能存在重复没有放在一个文件夹下

dataprocess.py为重写脚本（所有重写的部分均是相同的）

# 朴素方法

代码: [graph.html](/assets/01KJBZAQZ6CX2KZQ4FC02BPFXS/graph.html)

将以上文档 通过bge-m3做向量化, 在haystack的In memory存储中进行检索. 

检索问题为: "MySQL 中 signal 8 错误通常代表什么问题？"

检索的前十份匹配文档为 "signal 11/6/7"相关的文档

# 图方法

代码: <http://10.186.18.21/llm/chatdba/-/tree/knowledge_graph/knowledge_graph?ref_type=heads>

启动neo4j: 

```
docker run --name neo4j -p 127.0.0.1:7474:7474 -p 127.0.0.1:7687:7687 --volume=/home/user01/sql_audit/chatdba-knowledge_graph/knowledge_graph/neo4j/data:/data -d neo4j:latest
``` 

使用王昊单独传输的MySQL 8.4手册生成的图谱 作为数据文件

查看度数最多的节点: 

```
MATCH (n)
WITH n, size([(n)--() | 1]) AS degree
ORDER BY degree DESC
LIMIT 20
RETURN n, degree
``` 

结果: 

![graph (1).png](/assets/01KJBZAQZ6CX2KZQ4FC02BPFXS/graph%20%281%29.png)

# 测试case-1

查找问题: "MySQL临时表的使用场景有哪些?"

```
MATCH path=(t)<-[*1..3]->(u) 
WHERE
    toLower(t.name) CONTAINS 'temporary table' AND
    (toLower(u.name) CONTAINS 'usage' OR toLower(u.name) CONTAINS 'scenario')
RETURN path
``` 

结果: 

![graph (2).png](/assets/01KJBZAQZ6CX2KZQ4FC02BPFXS/graph%20%282%29.png)

并不能解决问题

# 分析

需要解决的问题: 构建知识图谱的规范 和 问题查询的规范 要一致, 否则问题查询 是 无法命中知识图谱的节点的

思路: 

  1. 参考一些NLP的辅助库: WordNet, ConceptNet
     1. ConceptNet: <https://conceptnet.io/s/contributor/petgame>
        1. ![image2024-7-23 18:34:16.png](/assets/01KJBZAQZ6CX2KZQ4FC02BPFXS/image2024-7-23%2018%3A34%3A16.png)
        2. 得理解其中的关系词是如何设计的: <https://github.com/commonsense/conceptnet5/wiki/Relations> 一共34个关系词
        3. 使用关系词进行查询: 

````
对于我的问题: "MySQL临时表的使用场景有哪些?"
如果用ConceptNet的关系词来进行查询, 查询语句应该怎么写
 
---
 
这是一个很好的问题！将自然语言问题转换成 ConceptNet 查询语句，需要将问题中的实体和关系映射到 ConceptNet 的概念和关系词上。

以下是针对您的问题 "MySQL 临时表的使用场景有哪些？"，使用 ConceptNet 进行查询的步骤和示例语句：

**1. 实体识别和映射:**

* 问题中的实体: "MySQL 临时表", "使用场景"
* ConceptNet 概念: 
    * "MySQL 临时表" 可以映射到 "temporary table" 或 "MySQL database" (更广义)
    * "使用场景" 可以映射到 "use case" 或 "purpose"

**2. 关系识别和映射:**

* 问题中的隐含关系:  "MySQL 临时表" 与 "使用场景" 之间存在某种语义关联，表示 "MySQL 临时表"  "被用于" 哪些 "使用场景"。
* ConceptNet 关系词: 可以考虑以下关系词：
    * `/r/UsedFor`:  表示 "用于" 的关系，例如 "锤子 UsedFor 钉钉子"。
    * `/r/HasProperty`:  表示 "具有属性" 的关系，可以理解为某种场景是 "MySQL 临时表" 的一种属性。

**3.  构建查询语句:**

* **方法一**: 使用 `/r/UsedFor` 关系词

```
[ConceptNet 查询语句]
find * from where $c isa "Temporary table" and $c <UsedFor> $usecase
```

* **方法二**: 使用 `/r/HasProperty` 关系词

```
[ConceptNet 查询语句]
find * from where $c isa "MySQL database" and $c <HasProperty> $property and $property isa "Use case"
```

**4. 结果解释:**

* 查询结果会返回与 "temporary table" 或 "MySQL database" 存在  `/r/UsedFor` 或  `/r/HasProperty` 关系的概念，这些概念就可能代表 "MySQL 临时表" 的使用场景。
* 需要注意的是，ConceptNet 的知识库是基于常识构建的，不一定包含所有特定领域的专业知识。因此，查询结果可能不完整，需要结合其他信息进行分析和判断。

**总结:**

使用 ConceptNet 查询 "MySQL 临时表的使用场景" 需要进行实体和关系的映射，并选择合适的 ConceptNet 关系词。查询结果可以提供一些参考信息，但需要结合其他知识进行综合分析。
```` 
  2. 使用特定领域的图谱, 提取关系词. 比如: DBpedia 等
  3. 从源文档中提取关系, 给出标准进行分析
