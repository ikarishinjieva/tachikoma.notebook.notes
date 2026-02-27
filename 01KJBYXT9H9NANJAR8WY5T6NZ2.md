---
title: 20230406 - 测试 cohere + llama-index
confluence_page_id: 2130588
created_at: 2023-04-07T03:28:01+00:00
updated_at: 2023-04-08T15:15:34+00:00
---

# llama-index的前处理的消耗研究

```
#!/usr/bin/env python
# coding: utf-8

# In[1]:

from llama_index import (
    GPTSimpleVectorIndex,
    SimpleDirectoryReader,
    LLMPredictor,
    ServiceContext
)
from langchain.llms import Cohere

# In[2]:

import os
os.environ["COHERE_API_KEY"] = 'pHIjyjM2fCYUbDEea8CZqQQqBL1ipZkjyMDzfltY'

# In[3]:

documents = SimpleDirectoryReader('data').load_data()

# In[4]:

# define LLM
llm_predictor = LLMPredictor(llm=Cohere())
service_context = ServiceContext.from_defaults(llm_predictor=llm_predictor)

# In[5]:

# build index
index = GPTSimpleVectorIndex.from_documents(documents, service_context=service_context)
index.save_to_disk('index.cohere.json')

# In[ ]:

# In[9]:

# get response from query
response = index.query("What is mysql")

# In[ ]:

``` 

这段代码, 在 前处理 完成后, 创建 embedding 时 由于使用了默认的OpenAI embed, 会报错. 

所以跑这段代码, 可以观察到 前处理 中的cpu消耗

```
python -m cProfile -o 1.prof 1.py
 
flameprof.py 1.prof > 1.svg
``` 

火焰图如附件: [1.svg](/assets/01KJBYXT9H9NANJAR8WY5T6NZ2/1.svg)

![image2023-4-6 23:11:2.png](/assets/01KJBYXT9H9NANJAR8WY5T6NZ2/image2023-4-6%2023%3A11%3A2.png)

都出现在 tokenize 阶段

# 使用Cohere + llama-index, mysql manual作为输入

```
#!/usr/bin/env python
# coding: utf-8

# In[1]:

proxy_addr = "http://192.168.22.8:7890"

import os
os.environ["http_proxy"] = proxy_addr
os.environ["https_proxy"] = proxy_addr

# In[2]:

from llama_index import (
    GPTSimpleVectorIndex,
    SimpleDirectoryReader,
    LLMPredictor,
    LangchainEmbedding,
    ServiceContext
)
from langchain.llms import Cohere

# In[3]:

import os
os.environ["COHERE_API_KEY"] = 'pHIjyjM2fCYUbDEea8CZqQQqBL1ipZkjyMDzfltY'

# In[4]:

documents = SimpleDirectoryReader('data').load_data()

# In[5]:

# TODO: add max_tokens and truncate to avoid Cohere error: too long tokens
llm_predictor = LLMPredictor(llm=Cohere(max_tokens=1024, truncate="END"))

from langchain.embeddings import CohereEmbeddings
embed_model = LangchainEmbedding(CohereEmbeddings(model="large", cohere_api_key=os.environ["COHERE_API_KEY"]))
service_context = ServiceContext.from_defaults(llm_predictor=llm_predictor, embed_model=embed_model)

# In[6]:

# build index
index = GPTSimpleVectorIndex.from_documents(documents, service_context=service_context)
index.save_to_disk('index.cohere.large.json')

# In[ ]:

# get response from query
response = index.query("how to rename a tablespace", service_context=service_context)

# In[ ]:

print(response)

``` 

在from_documents阶段, 用时1h以上, 产生的index.cohere.large.json为223M. Cohere花费 $2.17

![image2023-4-7 13:12:4.png](/assets/01KJBYXT9H9NANJAR8WY5T6NZ2/image2023-4-7%2013%3A12%3A4.png)

# 结果

[附件: llama_index-cohere.ipynb] 

答案并不准确 

注意: 此处使用max_tokens对token长度进行了限制, 可能造成答案不准确

# llama-index的逻辑梳理

  - BaseGPTIndex.from_documents: 从document生成index
    - node_parser.get_nodes_from_documents: 
      - get_nodes_from_documents
        - get_nodes_from_document: 从document生成node  

          - get_text_splits_from_document: 将document切割成文字片段 (text)

# 使用Cohere + llama-index, mysql manual作为输入

```
#!/usr/bin/env python
# coding: utf-8

# In[1]:

proxy_addr = "http://192.168.22.8:7890"

import os
os.environ["http_proxy"] = proxy_addr
os.environ["https_proxy"] = proxy_addr

# In[2]:

from llama_index import (
    GPTSimpleVectorIndex,
    SimpleDirectoryReader,
    LLMPredictor,
    LangchainEmbedding,
    ServiceContext
)
from langchain.llms import Cohere

# In[3]:

import os
os.environ["COHERE_API_KEY"] = 'pHIjyjM2fCYUbDEea8CZqQQqBL1ipZkjyMDzfltY'

# In[4]:

documents = SimpleDirectoryReader('data').load_data()

# In[5]:

# TODO: add max_tokens and truncate to avoid Cohere error: too long tokens
llm_predictor = LLMPredictor(llm=Cohere(max_tokens=1024, truncate="END"))

from langchain.embeddings import CohereEmbeddings
embed_model = LangchainEmbedding(CohereEmbeddings(model="large", cohere_api_key=os.environ["COHERE_API_KEY"]))
service_context = ServiceContext.from_defaults(llm_predictor=llm_predictor, embed_model=embed_model)

# In[6]:

# build index
index = GPTSimpleVectorIndex.from_documents(documents, service_context=service_context)
index.save_to_disk('index.cohere.large.json')

# In[ ]:

# get response from query
response = index.query("how to rename a tablespace", service_context=service_context)

# In[ ]:

print(response)

``` 

在from_documents阶段, 用时1h以上, 产生的index.cohere.large.json为223M. Cohere花费 $2.17

![image2023-4-7 13:12:4.png](/assets/01KJBYXT9H9NANJAR8WY5T6NZ2/image2023-4-7%2013%3A12%3A4.png)
