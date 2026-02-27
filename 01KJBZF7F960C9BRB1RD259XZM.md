---
title: 20240902 - 关于Graph与RAG结合的探索结论 (by 王昊)
confluence_page_id: 3342341
created_at: 2024-09-02T05:19:48+00:00
updated_at: 2024-09-02T05:19:48+00:00
---

# 重新划定需要使用知识图谱解决的问题范围

# 背景

由于上述使用的例子在最终看下来是否适合使用知识图谱来解决是未知的。所以现在需要重新划定一个范围，

  - 什么样子的场景更适合使用知识图谱来解决。

    - 或者说什么样类型的问题是现在RAG表现的不太好的，看看graph能不能有更好的表现的。

## 场景1: “全局”问题

所谓“全局”并不一定特指范围宽泛的问题，而是有可能和整个数据集相关的问题。举例：

  - [what are the top themes in this dataset? ](<https://github.com/microsoft/graphrag/blob/main/RAI_TRANSPARENCY.md#what-can-graphrag-do>)

    - 微软graphRAG 中举的例子，这个问题是跟整个数据集相关的。所以这个问题本质上RAG回答不了，因为不可能召回所有chunk来解析。
  - 西游记中有多少只妖怪是被孙悟空打死的？

    - 这个问题也一样，是跟整个数据集相关。RAG不太能召回整个西游记来做解析。

但是上述场景跟我们的场景并没有那么契合。因为即使用户问的问题是“介绍一下MySQL数据库的所有数据库引擎”这种问题，理论也是有可能存在一篇“MySQL数据库引擎详解”这样的文章被RAG召回。一样可以回答的很好。

能考虑到的可能的实用场景是：

  - 介绍一下参数“xxx_xxx_xx”在mysql版本迭代过程当中的变化。

## 场景2: 数据集本身存在链接

举例的话：比如工单系统和jira的任务本身存在链接。

理论上可以构建工单和jira任务的，jira任务又关联了版本信息等相关内容。基于这些显示存在的关系构建关系图谱。

这样可以达成以下功能：（用户同时是DMP和chatdba的用户）

用户问了问题A, => chatdba 使用 工单A 生成了答案A => 可以在图上召回 工单A关联的jira任务 在答案A上补充 DMP是如何解决的这个问题 or 如果使用DMP即可避免这个问题 等。

## 场景3：问题中实体关系非常明确的

这个场景理论上RAG也可以完成的很好，graph 同样也可以简单实现的场景。

比如问题：'xxx'参数是在mysql哪个版本新增的

相对来说可能RAG不一定能比较好处理的场景是 =>“问题本身实体关系很明确，但是关系链路比较长的”

举例：

  - [Which former husband of Elizabeth Taylor died in Chichester?](<https://arxiv.org/pdf/2405.12035>)

![image](https://actiontech.feishu.cn/space/api/box/stream/download/asynccode/?code=YTcxZjBiYmRlYWZhNmU4ZjkxMGE1OTNiYTlhZDUwNWZfT2tsMjlHSmw0Qk1uVTFyOUNNUTJoeENseVBjdkNaclFfVG9rZW46TERmWmJWWDhmb1VKOTJ4SVlCa2NQbUlZbm1kXzE3MjUyNTQzOTE6MTcyNTI1Nzk5MV9WNA)

  - Which country with the smallest ISO number borders Argentina?

![image](https://actiontech.feishu.cn/space/api/box/stream/download/asynccode/?code=Y2JlNDY3OWRmMWZiYWJiOTBkZmFkMzdhOTg1ZmI3NDFfd2pUS1FJV3puSUdNeGRwZGhQTk0wUWdzZEdGeDRDQUFfVG9rZW46RzRJQWIxY1U4b1pRVjZ4c3dydmMxV3dmbmFoXzE3MjUyNTQzOTE6MTcyNTI1Nzk5MV9WNA)

但是有一个问题是：在例子中是主观信任用户输入的问题中存在的关系是正确的。但是实际上的场景并不是这样的，如果用户输入的问题中本身存在的关系就有问题。

那么“如果图中不存在的关系即可以认定关系错误”的前提的，我们本身拥有非常靠谱且关系完成的图。但是这图在“非人工干预”的前提下，本身就不太好做。。

## 场景4: 对 “对话历史”进行graph结构构建，以试图解决对话过长时的遗忘问题

场景启发，从https://arxiv.org/abs/2408.12333来。

可以试图对提问和回答进行图结构的构建，将AI已经（未） 回答过/解决过的点都作为节点，然后构建关系。之后就可以个在图上找到还未解决的问题等内容。

## 总结

  - “全局”问题

    - 这个解决的意义我觉得不大，假设数据集还是mysql官方文档的话，暂时想不出来什么跟整个数据集相关的有价值的问题。类似“mysql一共有多少个参数”“在mysql官方文档中，都有哪些章节提到了查询优化”。主观上这些问题自身没有什么被解决的价值。
    - 所谓的比较“宽泛”的问题，目前的RAG在这个问题的方向上并不突出。主观上，在这个问题不突出的时候也没有必要进行解决。
  - 数据集本身存在链接的。

    - 按照上述定义的 “工单=>jira系统” 的链接的话，这个还是要先定义我们是否需要考虑做 chatdba 和 dmp 的关联功能。这个因为关系明确 所以graph应该很好构建，只是需要设计一下具体功能，来定义一下怎么用这个graph就可以了。
  - 问题中实体关系非常明确的。

    - 对于问题中实体关系不复杂的RAG表现也挺好的，我觉得不需要是graph处理这个场景。
    - 而对于问题中实体关系链路较长的，这个是我觉得graph表现可以更好的。只不过这个单一场景的影响面比较小，需要确认是否需要试图解决一下？
  - 试图解决长对话的 “遗忘 ”问题

    - 这个就只需要看这个问题是否突出了，毕竟这个问题是存在于对话过程，而上面的都是数据召回过程的。感觉不在同一个分类里。
