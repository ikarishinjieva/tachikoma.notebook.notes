---
title: 20230501 - 测试 mcontriever + openai + llama_index
confluence_page_id: 2130808
created_at: 2023-05-01T11:40:40+00:00
updated_at: 2023-05-01T17:22:30+00:00
---

# 测试文件-1

测试 使用 HuggingFaceInstructEmbeddings

[附件: llama_index-openai-azure.ipynb] 

使用中文问题, 可以从英文语料中获取材料, 举例: 

![image2023-5-1 19:37:23.png](/assets/01KJBYYVC6H5SK2N47CBNHJD5X/image2023-5-1%2019%3A37%3A23.png)

来自于MySQL官方文档英文语料: 

![image2023-5-1 19:39:33.png](/assets/01KJBYYVC6H5SK2N47CBNHJD5X/image2023-5-1%2019%3A39%3A33.png)

# 测试文件-2

换用 HuggingFaceEmbeddings

[附件: llama_index-openai-azure (1).ipynb] 

使用index.query, 效果比较理想, 并带有了 NDB 相关的信息 (是MySQL的英文文档中的信息)

![image2023-5-1 20:31:26.png](/assets/01KJBYYVC6H5SK2N47CBNHJD5X/image2023-5-1%2020%3A31%3A26.png)

问题: restore-from-image 这个子命令, 是目前的两个外源文档 (MySQL官方英文手册 和 mysqlbackup公司内部规范文档) 都没有提及的, 应该是从模型中整理出来的信息

# 发现问题

使用query接口 (不是chat接口)时, 日志中, 仅包括了一个Node: 

```
DEBUG:llama_index.indices.utils:> Top 1 nodes:
> [Node 9a152fbc-6a09-4cb6-9c9d-847a48a87af1] [Similarity score:             0.528281] title: 查看源
length: 14617
excerpt: 
byline: None
dir: None
lang: None
siteName: None
``` 

而提供给LLM的上下文长度有限, 所以不应只有一个node, 怀疑与更换了embedding model有关

结论: 怀疑是错误的. 是query的top K参数未设置, 默认值是1, 导致了top 1的存在. 修订后: 

```
    response = index.query("如何用mysqlbackup进行\"单文件全备恢复\"给出详细步骤和详细命令, 对命令的每个参数做出解释", similarity_top_k=3)
``` 

结果: 

![image2023-5-2 0:23:36.png](/assets/01KJBYYVC6H5SK2N47CBNHJD5X/image2023-5-2%200%3A23%3A36.png)

其中第5步是错误的, 文档中并没有 "--compress-method=lz4" 参数值在命令样例中. 并且与第1步重复

# 下一步要解决

  - 如何修订结果错误
  - 如何将来源文档进行标记
  - 如何指导模型, 尽量尊重规范文档
  - 如何给予负向反馈
