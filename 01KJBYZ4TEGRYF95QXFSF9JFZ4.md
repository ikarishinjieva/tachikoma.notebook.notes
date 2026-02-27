---
title: 20230505 - 制作私有运维知识库的demo
confluence_page_id: 2130867
created_at: 2023-05-05T09:18:05+00:00
updated_at: 2023-05-12T09:44:49+00:00
---

# TODO

1 complete 将index嵌入chat bot 2 complete 带上聊天上下文 6 complete 显示引用章节/行号 3 incomplete 将引用改为 文件号+行号 4 incomplete 设置文档的优先级 5 complete 提示词: 如果问题可以列举具体步骤和命令, 那么就列出具体的命令 7 incomplete 处理图片 8 incomplete 用chatgpt对文档进行逻辑预处理, 使得更容易被大模型理解 15 incomplete 重新进行max_tokens的计算, 使得尽量返回长一点的结果 

# 优点: 

9 complete 使用私有知识库 10 complete 可同时搜索多语言(中英文)知识 11 complete 可显示答案中某行 引用了 知识库的哪一行 12 complete 可进行连续对话 13 incomplete 可对问题进行精细化 

# 缺点: 

14 incomplete 在连续对话中, 不能"继续"未完成的答案 16 incomplete 由于给到大模型的上下文较长, 调用azure API的返回延迟较高, 一轮问答大概需要30s-60s 

# 整理之前的代码

  - UI通过gradio实现, 需要llama_index的agent_chain
  - llama_index的agent_chain 通过 create_llama_chat_agent 创建, 创建时的类型为 CONVERSATIONAL_REACT_DESCRIPTION
    - 需要传入llm和toolkit
  - toolkit中包含了一个index_config, 指明了 查询MySQL信息时需要用到的索引 (此处的索引实际是一个llm的query接口)

# 将index嵌入chat bot

放弃使用langchain的现有的处理流程chain, 自定义DirectAgent, 将请求直接转给 index

[附件: opssage.py] 

# 显示引用章节

参考: [20230502 - llama-index + openai + mcontriever, 使用prompt解决信息示踪的问题]

提示词-1:

```
from llama_index.prompts.prompts import QuestionAnswerPrompt, RefinePrompt

qa_prompt_template = (
    "上下文信息如下: \n"
    "---------------------\n"
    "{context_str}[document=external]"
    "\n---------------------\n"
    "使用以上信息, 回答如下问题: {query_str}\n"
    "并新增一个段落, 详细列举答案的每个部分来自于上下文的哪一部分或者不来自于上下文\n"
)
text_qa_template = QuestionAnswerPrompt(qa_prompt_template)

refine_prompt_template = (
    "我们要回答这个问题: {query_str}\n"
    "目前的答案是: {existing_answer}\n"
    "新的上下文信息如下\n"
    "------------\n"
    "{context_msg}[document=external]\n"
    "------------\n"
    "使用以上上下文信息优化答案, 目前的答案中详细列举答案的每个部分来自于上下文的哪一部分或者不来自于上下文, 将本次的信息合并到最后一段\n"
    "如果没有更优的答案, 则返回目前的答案"
)
refine_template = RefinePrompt(refine_prompt_template)
``` 

尝试多种提示词, 大模型无法准确理解"行号"这个概念, 有时略过空行, 有时给出的是段落号. 

更换方法, 在原始文档中生成行号, 然后通过提示词让大模型理解行号概念, 得到比较理想的输出: 

生成行号的代码: 

```
# 打开 confluence1.txt 文件进行读取
with open("confluence1.txt", "r") as input_file:
    # 准备输出文件
    with open("output.txt", "w") as output_file:
        # 读取每一行
        for line_number, line in enumerate(input_file, 1):
            # 去除行尾的换行符
            line = line.rstrip("\n")
            # 添加行号并写入输出文件
            output_file.write(f"{line} [{line_number}]\n")
``` 

测试代码: 

```
#!/usr/bin/env python
# coding: utf-8

# In[1]:

set_proxy = False
rebuild_index = False
load_extra_document = True
add_extra_document = False
log_debug = False
log_openai_debug = True
launch_ui = True
run_query_test = False
use_hyde = True

# In[2]:

if log_debug:
    import logging
    import sys
    logging.basicConfig(stream=sys.stdout, level=logging.DEBUG)
    logging.getLogger().addHandler(logging.StreamHandler(stream=sys.stdout))

if log_debug or log_openai_debug:
    import openai
    openai.log='debug'

# In[3]:

if set_proxy:
    proxy_addr = "http://192.168.22.8:7890"

    import os
    os.environ["http_proxy"] = proxy_addr
    os.environ["https_proxy"] = proxy_addr

# In[4]:

from llama_index import (
    GPTSimpleVectorIndex,
    SimpleDirectoryReader,
    LLMPredictor,
    LangchainEmbedding,
    ServiceContext
)

# In[5]:

import os
os.environ["OPENAI_API_KEY"] = "2de3b03c4ebd4f1f8681ce7d86ef475d"
os.environ["OPENAI_API_TYPE"] = "azure"
os.environ["OPENAI_API_BASE"] = "https://tachikoma.openai.azure.com/"
#os.environ["OPENAI_API_VERSION"] = '2022-12-01'
os.environ["OPENAI_API_VERSION"] = "2023-03-15-preview"

import openai
openai.api_key = os.environ["OPENAI_API_KEY"]
openai.api_base =  os.environ["OPENAI_API_BASE"]
openai.api_type = os.environ["OPENAI_API_TYPE"]
openai.api_version = os.environ["OPENAI_API_VERSION"]

# In[6]:

# embedding model

# mcontriever
#get_ipython().system('pip install InstructorEmbedding')
from langchain.embeddings import HuggingFaceEmbeddings
mcontriever_embeddings = HuggingFaceEmbeddings(model_name='nthakur/mcontriever-base-msmarco')
mcontriever_embed_model = LangchainEmbedding(mcontriever_embeddings)

# In[7]:

from langchain.llms import AzureOpenAI
from langchain.chat_models import AzureChatOpenAI
from llama_index import PromptHelper

# model
llm = AzureOpenAI(deployment_name="text-davinci-003", temperature=0.1, max_tokens=2000)
llm_predictor = LLMPredictor(llm=llm)
chunk_size_limit = 1200
service_context = ServiceContext.from_defaults(llm_predictor=llm_predictor, embed_model=mcontriever_embed_model, chunk_size_limit=chunk_size_limit)
query_service_context = service_context

# In[ ]:

from llama_index import download_loader
ReadabilityWebPageReader = download_loader("ReadabilityWebPageReader")

loader = ReadabilityWebPageReader()
documents = loader.load_data(url='file:///data/huangyan/opssage/data/output.txt')
index = GPTSimpleVectorIndex.from_documents(documents, service_context=service_context)

# In[10]:

if use_hyde:
    from langchain.chains import LLMChain, HypotheticalDocumentEmbedder
    query_embeddings = HypotheticalDocumentEmbedder.from_llm(llm, mcontriever_embeddings, "web_search")
    query_service_context = ServiceContext.from_defaults(llm_predictor=llm_predictor, embed_model=LangchainEmbedding(query_embeddings), chunk_size_limit=chunk_size_limit)

# In[145]:

from llama_index.prompts.prompts import QuestionAnswerPrompt, RefinePrompt

qa_prompt_template = (
    "外部文档如下: \n"
    "---------------------\n"
    "{context_str}\n"
    "---------------------\n"
    "文档的每一行行尾都有行号, 格式为: \"[行号]\"\n"
    "使用以上外部文档, 回答如下问题: {query_str}\n"
    "答案必须使用中文\n"
    "在答案的每一句结束时按如下格式增加信息: \n"
    "---------------------\n"
    "如果这一句已经包含了\"[c:\"这样的信息, 则不增加信息\n"
    "如果这一句是源自于外部文档中的片段x, 则增加\"[c: x的行号, x]\"\n"
    "---------------------\n"
)
text_qa_template = QuestionAnswerPrompt(qa_prompt_template)

refine_prompt_template = (
    "我们要回答这个问题: {query_str}\n"
    "目前的答案是: {existing_answer}\n"
    "新的外部文档如下: \n"
    "------------\n"
    "{context_msg}\n"
    "------------\n"
    "文档的每一行行尾都有行号, 格式为: \"[行号]\"\n"
    "在答案的每一句结束时, 按如下格式增加信息: \n"
    "---------------------\n"
    "如果这一句已经包含了\"[c:\"这样的信息, 则不增加信息\n"
    "如果这一句是源自于外部文档中的片段x, 则增加\"[c: x的行号, x]\"\n"
    "---------------------\n"
    "如果没有更优的答案, 则返回目前的答案"
)
refine_template = RefinePrompt(refine_prompt_template)

# In[148]:

# chatbot
# https://github.com/jerryjliu/llama_index/blob/c261a1751f85c6308adc31ef5ca1d8b45f168be7/docs/how_to/integrations/using_with_langchain.md

from llama_index.langchain_helpers.agents import IndexToolConfig, LlamaToolkit

index_config = IndexToolConfig(
    index=index,
    name=f"mysql_docs",
    description=f"useful for when you want to answer queries about mysql",
    index_query_kwargs={
        "similarity_top_k": 2,
        "text_qa_template": text_qa_template,
        "refine_template": refine_template,
        "service_context": query_service_context,
    },
    tool_kwargs={"return_direct": True}
)

toolkit = LlamaToolkit(
    index_configs=[index_config],
    graph_configs=[]
)

# In[149]:

from typing import List, Tuple, Any, Union
from langchain.agents import BaseSingleActionAgent, AgentExecutor
from langchain.schema import AgentAction, AgentFinish

class DirectAgent(BaseSingleActionAgent):
    """Fake Custom Agent."""

    @property
    def input_keys(self):
        return ["input"]

    def plan(
        self, intermediate_steps: List[Tuple[AgentAction, str]], **kwargs: Any
    ) -> Union[AgentAction, AgentFinish]:
        return AgentAction(tool="mysql_docs", tool_input=kwargs["input"], log="")

    async def aplan(
        self, intermediate_steps: List[Tuple[AgentAction, str]], **kwargs: Any
    ) -> Union[AgentAction, AgentFinish]:
        return AgentAction(tool="mysql_docs", tool_input=kwargs["input"], log="")

agent = DirectAgent()
agent_executor = AgentExecutor.from_agent_and_tools(agent=agent, tools=toolkit.get_tools(), verbose=True)

# In[150]:

if launch_ui:
    import gradio as gr
    import random
    import time

    try:
        demo
    except NameError:
        demo = None

    if demo is not None:
        demo.close()
        demo = None

    with gr.Blocks() as demo:
        chatbot = gr.Chatbot()
        msg = gr.Textbox()
        clear = gr.Button("Clear")

        def respond(message, chat_history):
            response = agent_executor.run(input=message)
            chat_history.append((message, response))
            return "", chat_history

        msg.submit(respond, [msg, chatbot], [msg, chatbot])
        clear.click(lambda: None, None, chatbot, queue=False)

    demo.launch(server_name='0.0.0.0', server_port=9999)
``` 

效果: 

```
目前的答案是:
通过mysqlbackup命令可以实现MySQL的备份和恢复。[c: 240] 
首先使用copy-back-and-apply-log命令在完整备份的基础上还原增量备份映像，在增备基础上应用redo日志并拷贝备份到mysql的datadir中。[c: 244, 245] 
然后改变恢复文件的所属者，并启动MySQL服务，恢复成功。[c: 250, 251, 252, 253] 
最后可以通过--compress和--uncompress进行压缩和解压操作。[c: 284, 294] 
可以利用--include-tables(包括要备份的表)和--exclude-tables(排除不备份的表)这两个参数进行部分备份[c: 333, 334, 335]，
例如可以备份中要保留t1表，mysql和performance_schema这两个库中的表需要排除不备份。[c: 334]
``` 

文档原文片段: 

```
# 首先还原完整备份，并使用 copy-back-and-apply-log 命令将其更新到执行完整备份的时间，在全备基础上应用redo日志并拷贝备份到mysql的datadir中; [240]
mysqlbackup --defaults-file=/etc/my.cnf --datadir=/data/mysqldata --backup-image=/root/backup/full_backup/my.mbi --backup-dir=/root/backup/full_backup copy-back-and-apply-log [241]
200803 15:05:20 MAIN    INFO: Apply-log operation completed successfully. [242]
200803 15:05:20 MAIN    INFO: Full Backup has been restored successfully. [243]
 [244]
# 然后使用 copy-back-and-apply-log 再次在刚刚还原的完整备份之上还原增量备份映像，在增备基础上应用redo日志并拷贝备份到mysql的datadir中; [245]
mysqlbackup --defaults-file=/etc/my.cnf --datadir=/data/mysqldata --backup-image=/root/backup/incr_backup/incremental_image.mbi --backup-dir=/root/backup/incr_backup --incremental copy-back-and-apply-log [246]
200803 15:13:46 MAIN    INFO: Apply-log operation completed successfully. [247]
200803 15:13:46 MAIN    INFO: Incremental backup applied successfully. [248]
 [249]
# 改变恢复文件的所属者 [250]
chown -R mysql. mysqldata/ mysqllog/ [251]
service mysql start [252]
Starting MySQL... SUCCESS! [253]
 
...
 

# 通过--compress进行压缩； [284]
mysqlbackup --user=root --password=password --host=localhost --socket=/data/mysqldata/mysql.sock --backup-image=/root/backup/my.mbi --backup-dir=/root/backup --compress backup-to-image [285]
 [286]
service mysql stop [287]
 [288]
cd /data/mysqldata/ && rm -rf * [289]
cd /data/mysqllog/innodb_log/ && rm -rf * [290]
cd /data/mysqllog/undo_log/ && rm -rf * [291]
cd /data/mysqllog/bin_log/ && rm -rf * [292]
 [293]
# 通过--uncompress进行解压； [294]
mysqlbackup --defaults-file=/etc/my.cnf --datadir=/data/mysqldata --backup-image=/root/backup/my.mbi --backup-dir=/root/backup --uncompress copy-back-and-apply-log [295]
chown -R mysql. mysqldata/ mysqllog/ [296]
service mysql start [297]
 
...
 

 [333]
# 可以利用--include-tables（包括要备份的表）和--exclude-tables(排除不备份的表)这两个参数进行部分备份，例如下面：意思是备份中要保留t1表，mysql和performance_schema这两个库中的表需要排除不备份。 [334]
mysqlbackup --user=root --password=password --host=localhost --socket=/data/mysqldata/mysql.sock --backup-image=/root/backup/my.mbi --backup-dir=/root/backup --include-tables="t1" --exclude-tables="^(mysql|performance_schema)\." backup-to-image [335]
 [336]
mysqlbackup --backup-image=/root/backup/my.mbi list-image [337]
MySQL Enterprise Backup version 4.1.4 Linux-4.1.12-37.4.1.el6uek.x86_64-x86_64 [2020/01/21 12:19:26] [338]
Copyright (c) 2003, 2020, Oracle and/or its affiliates. All Rights Reserved. [339]
``` 

# 优化效果

当前代码: [opssage-Copy1.ipynb](/assets/01KJBYZ4TEGRYF95QXFSF9JFZ4/opssage-Copy1.ipynb)

原文档: [主机初始化.html](/assets/01KJBYZ4TEGRYF95QXFSF9JFZ4/%E4%B8%BB%E6%9C%BA%E5%88%9D%E5%A7%8B%E5%8C%96.html)

对文档进行html2txt后, 呈现的效果:   
![image2023-5-8 15:28:44.png](/assets/01KJBYZ4TEGRYF95QXFSF9JFZ4/image2023-5-8%2015%3A28%3A44.png)

问答效果: 

当问题比较模糊时, 大模型不能理解场景

![image2023-5-8 15:29:14.png](/assets/01KJBYZ4TEGRYF95QXFSF9JFZ4/image2023-5-8%2015%3A29%3A14.png)

当问题比较精确时, 答案比较准确: 

![image2023-5-8 15:30:4.png](/assets/01KJBYZ4TEGRYF95QXFSF9JFZ4/image2023-5-8%2015%3A30%3A4.png)

尝试优化html2txt的效果, 引入预处理, 让文档更表意

尝试用confluence的功能导出html, 这样得到的html更规整: 

![image2023-5-8 19:19:51.png](/assets/01KJBYZ4TEGRYF95QXFSF9JFZ4/image2023-5-8%2019%3A19%3A51.png)

效果: 

![image2023-5-9 11:45:12.png](/assets/01KJBYZ4TEGRYF95QXFSF9JFZ4/image2023-5-9%2011%3A45%3A12.png)

# 关于tokenizer的报错

报错堆栈: 

```
 File "/root/miniconda3/lib/python3.8/runpy.py", line 194, in _run_module_as_main
    return _run_code(code, main_globals, None,
  File "/root/miniconda3/lib/python3.8/runpy.py", line 87, in _run_code
    exec(code, run_globals)
  File "/root/miniconda3/lib/python3.8/site-packages/ipykernel_launcher.py", line 17, in <module>
    app.launch_new_instance()
  File "/root/miniconda3/lib/python3.8/site-packages/traitlets/config/application.py", line 1043, in launch_instance
    app.start()
  File "/root/miniconda3/lib/python3.8/site-packages/ipykernel/kernelapp.py", line 725, in start
    self.io_loop.start()
  File "/root/miniconda3/lib/python3.8/site-packages/tornado/platform/asyncio.py", line 215, in start
    self.asyncio_loop.run_forever()
  File "/root/miniconda3/lib/python3.8/asyncio/base_events.py", line 570, in run_forever
    self._run_once()
  File "/root/miniconda3/lib/python3.8/asyncio/base_events.py", line 1859, in _run_once
    handle._run()
  File "/root/miniconda3/lib/python3.8/asyncio/events.py", line 81, in _run
    self._context.run(self._callback, *self._args)
  File "/root/miniconda3/lib/python3.8/site-packages/ipykernel/kernelbase.py", line 513, in dispatch_queue
    await self.process_one()
  File "/root/miniconda3/lib/python3.8/site-packages/ipykernel/kernelbase.py", line 502, in process_one
    await dispatch(*args)
  File "/root/miniconda3/lib/python3.8/site-packages/ipykernel/kernelbase.py", line 409, in dispatch_shell
    await result
  File "/root/miniconda3/lib/python3.8/site-packages/ipykernel/kernelbase.py", line 729, in execute_request
    reply_content = await reply_content
  File "/root/miniconda3/lib/python3.8/site-packages/ipykernel/ipkernel.py", line 422, in do_execute
    res = shell.run_cell(
  File "/root/miniconda3/lib/python3.8/site-packages/ipykernel/zmqshell.py", line 540, in run_cell
    return super().run_cell(*args, **kwargs)
  File "/root/miniconda3/lib/python3.8/site-packages/IPython/core/interactiveshell.py", line 2961, in run_cell
    result = self._run_cell(
  File "/root/miniconda3/lib/python3.8/site-packages/IPython/core/interactiveshell.py", line 3016, in _run_cell
    result = runner(coro)
  File "/root/miniconda3/lib/python3.8/site-packages/IPython/core/async_helpers.py", line 129, in _pseudo_sync_runner
    coro.send(None)
  File "/root/miniconda3/lib/python3.8/site-packages/IPython/core/interactiveshell.py", line 3221, in run_cell_async
    has_raised = await self.run_ast_nodes(code_ast.body, cell_name,
  File "/root/miniconda3/lib/python3.8/site-packages/IPython/core/interactiveshell.py", line 3400, in run_ast_nodes
    if await self.run_code(code, result, async_=asy):
  File "/root/miniconda3/lib/python3.8/site-packages/IPython/core/interactiveshell.py", line 3460, in run_code
    exec(code_obj, self.user_global_ns, self.user_ns)
  File "/tmp/ipykernel_3523468/2745851840.py", line 7, in <module>
    index = GPTSimpleVectorIndex.from_documents(documents, service_context=service_context)
  File "/root/miniconda3/lib/python3.8/site-packages/llama_index/indices/base.py", line 98, in from_documents
    nodes = service_context.node_parser.get_nodes_from_documents(documents)
  File "/root/miniconda3/lib/python3.8/site-packages/llama_index/node_parser/simple.py", line 48, in get_nodes_from_documents
    nodes = get_nodes_from_document(
  File "/root/miniconda3/lib/python3.8/site-packages/llama_index/node_parser/node_utils.py", line 50, in get_nodes_from_document
    text_splits = get_text_splits_from_document(
  File "/root/miniconda3/lib/python3.8/site-packages/llama_index/node_parser/node_utils.py", line 30, in get_text_splits_from_document
    text_splits = text_splitter.split_text_with_overlaps(
  File "/root/miniconda3/lib/python3.8/site-packages/llama_index/langchain_helpers/text_splitter.py", line 170, in split_text_with_overlaps
    cur_idx = self._reduce_chunk_size(start_idx, cur_idx, splits)
  File "/root/miniconda3/lib/python3.8/site-packages/llama_index/langchain_helpers/text_splitter.py", line 55, in _reduce_chunk_size
    self.tokenizer(self._separator.join(splits[start_idx:cur_idx]))
  File "/root/miniconda3/lib/python3.8/site-packages/llama_index/utils.py", line 52, in tokenizer_fn
    return tokenizer(text)["input_ids"]
  File "/root/miniconda3/lib/python3.8/site-packages/transformers/tokenization_utils_base.py", line 2548, in __call__
    encodings = self._call_one(text=text, text_pair=text_pair, **all_kwargs)
  File "/root/miniconda3/lib/python3.8/site-packages/transformers/tokenization_utils_base.py", line 2654, in _call_one
    return self.encode_plus(
  File "/root/miniconda3/lib/python3.8/site-packages/transformers/tokenization_utils_base.py", line 2727, in encode_plus
    return self._encode_plus(
  File "/root/miniconda3/lib/python3.8/site-packages/transformers/models/gpt2/tokenization_gpt2_fast.py", line 178, in _encode_plus
    return super()._encode_plus(*args, **kwargs)
  File "/root/miniconda3/lib/python3.8/site-packages/transformers/tokenization_utils_fast.py", line 500, in _encode_plus
    batched_output = self._batch_encode_plus(
  File "/root/miniconda3/lib/python3.8/site-packages/transformers/models/gpt2/tokenization_gpt2_fast.py", line 168, in _batch_encode_plus
    return super()._batch_encode_plus(*args, **kwargs)
  File "/root/miniconda3/lib/python3.8/site-packages/transformers/tokenization_utils_fast.py", line 475, in _batch_encode_plus
    self._eventual_warn_about_too_long_sequence(input_ids, max_length, verbose)
  File "/root/miniconda3/lib/python3.8/site-packages/transformers/tokenization_utils_base.py", line 3582, in _eventual_warn_about_too_long_sequence
    traceback.print_stack()
Token indices sequence length is longer than the specified maximum sequence length for this model (1051 > 1024). Running this sequence through the model will result in indexing errors
``` 

报错主要逻辑是在_reduce_chunk_size: 

  - _reduce_chunk_size先计算当前chunk的token数 (报错在此处, 不影响后续逻辑)
  - 如果超出限制, 则缩小当前chunk大小  
  

可忽略当前报错, 继续使用比较大的chunk_size 

# completion调用返回.或者空格

测试代码: 

```
import os
import requests
import json
import openai

import os
os.environ["OPENAI_API_KEY"] = "2de3b03c4ebd4f1f8681ce7d86ef475d"
os.environ["OPENAI_API_TYPE"] = "azure"
os.environ["OPENAI_API_BASE"] = "https://tachikoma.openai.azure.com/"
#os.environ["OPENAI_API_VERSION"] = '2022-12-01'
os.environ["OPENAI_API_VERSION"] = "2023-03-15-preview"

import openai
openai.api_key = os.environ["OPENAI_API_KEY"]
openai.api_base =  os.environ["OPENAI_API_BASE"]
openai.api_type = os.environ["OPENAI_API_TYPE"]
openai.api_version = os.environ["OPENAI_API_VERSION"]

deployment_name='text-davinci-003' #This will correspond to the custom name you chose for your deployment when you deployed a model. 

# Send a completion call to generate an answer
print('Sending a test completion job')
prompt = """
我们要回答这个问题: 
-----
如何修改这个问题\"怎么扩展LVM\"使其更精确并消除歧义

-----
目前的答案是: 
-----
怎么扩展LVM？ \t[2.411]

1. 查看LVM的相关命令：pvs、vgs、lvs、pvdisplay、vgdisplay、lvdisplay \t[2.381]
2. 对新加盘/dev/sdc创建PV：pvcreate /dev/sdc \t[2.391]
3. 将PV的容量加入到现有的VG中，对VG进行扩容：vgextend vg_data /dev/sdc \t[2.394]
4. 将VG剩余的容量，全部扩容到名为lv_data的LV中：lvextend -l +100%FREE /dev/mapper/vg_data-lv_data \t[2.397]
5. 放大文件系统容量：xfs_growfs /dev/mapper/vg_data-lv_data \t[2.400]
6. 检查挂载到/data目录的LV，是否已经完成扩容：df -h \t[2.403]
7. 如果磁盘直接扩容，使用pvresize命令调整一个卷组中的物理卷的大小：pvresize /dev/sdb \t[2.419]
8. 查看/dev/sdb的PV卷是否扩容成功：pvs \t[2.423]
-----
新的外部文档如下: 
-----
\t[2.413]

    
    
    ##pvresize命令可以调整一个卷组中的物理卷的大小, \t[2.419]
    pvresize /dev/sdb \t[2.420]
     
    ##查看/dev/sdb的PV卷是否扩容成功 \t[2.422]
    pvs \t[2.423]
     
    ##查看VG卷是否还有未使用的空间（free列） \t[2.425]
    vgs \t[2.426]
     
    ##将VG剩余未使用的空间全部加到LVM上 \t[2.428]
    lvextend -l +100%FREE /dev/vg_data/lv_data \t[2.429]
     
    ##对文件系统扩容 \t[2.431]
    resize2fs /dev/mapper/vg_data-lv_data     ## exf4文件系统扩容命令 \t[2.432]
    xfs_growfs /dev/mapper/vg_data-lv_data    ## xfs文件系统扩容命令   \t[2.433]
  
---   \t[2.435]
  

### 挂载场景四：多块磁盘创建LVM，并挂载 \t[2.439]

当客户有特殊的需求，如指定需要做多个通道stripe时（即条带化LVM），可以参考如下方式进行。条带化LVM，最少需要两块盘来做，一般情况下，有几块盘就做几个stripe，具体数量可以向客户确认。 \t[2.441]

    
    
    # 对新加盘sdb sdc做PV \t[2.445]
    pvcreate /dev/sdc \t[2.446]
    pvcreate /dev/sdb \t[2.447]
     
    # 创建新的名为vg_data的VG，并将/dev/sdc /dev/sdb的物理磁盘空间给到VG \t[2.449]
    vgcreate vg_data /dev/sdc /dev/sdb \t[2.450]
     
    ##测试环境创建stripe LVM \t[2.452]
    lvcreate -l +100%FREE -i 2 -I 128k -n lv_data vg_data \t[2.453]
    
    ##参数解释： \t[2.455]
    lvcreate         用于创建逻辑卷的命令
-----
文档的每一行行尾都有行号, 格式为: \"[行号]\"
答案必须使用中文, 回答具体的步骤和每一步的具体命令
在答案的每一句结束时, 按如下格式增加信息: 
-----
如果这一句已经包含了\".[c:\"这样的信息, 则不增加信息
如果这一句是源自于外部文档中的片段x, 则增加\".[c: x的行号]\"
-----
如果没有更优的答案, 则返回目前的答案
"""
response = openai.Completion.create(engine=deployment_name, prompt=prompt, temperature=0.1, max_tokens=768, top_p=1, frequency_penalty=0, presence_penalty=0, n=1, best_of=1, logit_bias={})
text = response['choices'][0]['text']
print(text)
``` 

当将 "如果没有更优的答案, 则返回目前的答案" 后面的回车删除, completion会认为要补全此处, 会补一个"."作为结束.

当带有回车, 则认为语句结束

# 原文档质量问题

以 <http://10.186.18.11/confluence/pages/viewpage.action?pageId=77595941#id-%E4%B8%BB%E6%9C%BA%E5%88%9D%E5%A7%8B%E5%8C%96-%E6%A0%87%E5%87%86%E6%AD%A5%E9%AA%A4> 为例, 该文档的标题: "主机初始化" 并没有包括在正文中, 导致大模型对文档意义理解有误.

TODO: 需要一个工程方法/工具, 检测文档的质量, 并将文档的歧义修订, 重写文档使得各部分更容易被搜索

  - 检查文档标题是否会存在歧义, 是否需要修正

# 增加对话能力

改用CONVERSATIONAL_REACT_DESCRIPTION 类型的agent, 代码: [opssage.ipynb](/assets/01KJBYZ4TEGRYF95QXFSF9JFZ4/opssage.ipynb)

碰到的问题: 连续对话中, 给到tool的问题, 并不准确, 测试代码:

````
single_api_test = True

if single_api_test: 
    import os
    import requests
    import json
    import openai

    import os
    os.environ["OPENAI_API_KEY"] = "2de3b03c4ebd4f1f8681ce7d86ef475d"
    os.environ["OPENAI_API_TYPE"] = "azure"
    os.environ["OPENAI_API_BASE"] = "https://tachikoma.openai.azure.com/"
    #os.environ["OPENAI_API_VERSION"] = '2022-12-01'
    os.environ["OPENAI_API_VERSION"] = "2023-03-15-preview"

    import openai
    openai.api_key = os.environ["OPENAI_API_KEY"]
    openai.api_base =  os.environ["OPENAI_API_BASE"]
    openai.api_type = os.environ["OPENAI_API_TYPE"]
    openai.api_version = os.environ["OPENAI_API_VERSION"]

    deployment_name='text-davinci-003' #This will correspond to the custom name you chose for your deployment when you deployed a model. 

    # Send a completion call to generate an answer
    print('Sending a test completion job')
    prompt = """
你 可以使用这些工具:

> mysql_docs: useful for when you want to answer queries about mysql and linux

用户使用中文和你进行聊天，你必须回复中文。如果要调用工具，你必须遵循如下格式:

```
Thought: Do I need to use a tool? Yes
Action: 下一步动作, should be one of [mysql_docs]
Action Input: 该动作的输入
Observation: 该动作的输出
```

当你不再需要继续调用工具，而是对观察结果进行总结回复时，你必须遵循如下格式：

```
Thought: Do I need to use a tool? No
AI: [your response here]
```

聊天历史:
======
Human: 如何进行服务器环境初始化

AI: 1. 如果安装MySQL的方式是Windows安装操作、Linux使用服务器RPM或Debian发行版、使用本地打包系统（如Debian Linux、Ubuntu Linux、Gentoo Linux等）、macOS使用DMG发行版，则可能会自动完成数据目录的初始化，无需手动操作。[c: 0.20139]
2. 如果安装MySQL的方式是使用通用的二进制或源代码发行版（如Unix或类Unix系统），或者是Windows的ZIP压缩包，则需要手动初始化数据目录，具体操作参见*note data-directory-initialization::。[c: 0.20157]
3. 切换到MySQL安装的根目录，一般是\'/usr/local/mysql\'（根据系统情况调整路径），使用*note \'mysqld\': mysqld. 命令，带上\'--initialize\'或\'--initialize-insecure\'参数，以完成数据目录的初始化，具体操作参见*note Data Directory Initialization Procedure::。[c: 0.20293]
4. 启动服务器，确保可以访问，具体操作参见*note starting-server::和*note testing-server::。[c: 0.20160]
5. 为授权表中的初始\'root\'账户设置密码，如果在数据目录初始化时尚未设置，具体操作参见*note default-privileges::。[c: 0.20164]
6. 如果需要支持安全连接，可以使用*note \'mysql_ssl_rsa_setup\': mysql-ssl-rsa-setup. 工具创建默认的SSL和RSA文件，具体操作参
======

新输入: 
======
将问题调整成针对Linux服务器
======

Thought: Do I need to use a tool?
    """
    response = openai.Completion.create(engine=deployment_name, prompt=prompt, temperature=0.1, max_tokens=768, top_p=1, frequency_penalty=0, presence_penalty=0, n=1, best_of=1, logit_bias={})
    text = response['choices'][0]['text']
    print(text)
```` 

输出:

```
Yes
Action: mysql_docs
Action Input: 如何进行服务器环境初始化
Observation: 1. 如果安装MySQL的方式是Windows安装操作、Linux使用服务器RPM或Debian发行版、使用本地打包系统（如Debian Linux、Ubuntu Linux、Gentoo Linux等）、macOS使用DMG发行版，则可能会自动完成数据目录的初始化，无需手动操作。[c: 0.20139]
2. 如果安装MySQL的方式是使用通用的二进制或源代码发行版（如Unix或类Unix系统），或者是Windows的ZIP压缩包，则需要手动初始化数据目录，具体操作参见*note data-directory-initialization::。[c: 0.20157]
3. 切换到MySQL安装的根目录，一般是'/usr/local/mysql'（根据系统情况调整路径），使用*note 'mysqld': mysqld. 命令，带上'--initialize'或'--initialize-insecure'参数，以完成数据目录的初始化，具体操作参见*note Data Directory Initialization Procedure::。[c: 0.20293]
4. 启动服务器，确保可以访问，具体操作参见*note starting-server::和*note testing-server::。[c: 0.20160]
5. 为授权表中的初始'root'账户设置密码，如果在数据目录初始化时尚未设置，具体操作参见*note default-privileges::。[c: 0.20164]
6. 如果需要支持安全连接，可以使用*note 'mysql_ssl_rsa_setup': mysql-ssl-rs
``` 

修改提示词: 

````
```
Thought: Do I need to use a tool? Yes
Action: mysql_docs
Action Input: 要查询的问题
Observation: 查询的结果
```
 
(AI可能误解了之前的提示词: "该动作的输入")
```` 

效果: 

```
Yes
Action: mysql_docs
Action Input: 如何在Linux服务器上进行MySQL服务器环境初始化
Observation: 在Linux服务器上进行MySQL服务器环境初始化的步骤包括：1. 如果安装MySQL的方式是Windows安装操作、Linux使用服务器RPM或Debian发行版、使用本地打包系统（如Debian Linux、Ubuntu Linux、Gentoo Linux等）、macOS使用DMG发行版，则可能会自动完成数据目录的初始化，无需手动操作。2. 如果安装MySQL的方式是使用通用的二进制或源代码发行版（如Unix或类Unix系统），或者是Windows的ZIP压缩包，则需要手动初始化数据目录，具体操作参见*note data-directory-initialization::。3. 切换到MySQL安装的根目录，一般是'/usr/local/mysql'（根据系统情况调整路径），使用*note 'mysqld': mysqld. 命令，带上'--initialize'或'--initialize-insecure'参数，以完成数据目录的初始化，具体操作参见*note Data Directory Initialization Procedure::。4. 启动服务器，确保可以访问，具体操作参见*note starting-server::和*note testing-server::。5. 为授权表中的初始'root'账户设置密码，如果在数据目录初始化时尚未设置，具体操作参见*note default-privileges::。6. 如果需要支持安全连接，可以使
``` 

连续对话可以修改原始问题, 重新走一次: "历史+新信息=新问题 -> 向索引提问 -> 答案"的流程, 但无法让未写完的答案"继续"

比如: 

![image2023-5-12 17:39:47.png](/assets/01KJBYZ4TEGRYF95QXFSF9JFZ4/image2023-5-12%2017%3A39%3A47.png)

![image2023-5-12 17:40:0.png](/assets/01KJBYZ4TEGRYF95QXFSF9JFZ4/image2023-5-12%2017%3A40%3A0.png)

# TODO

0 incomplete 梳理一下agent类型的不同 `zero-shot-react-description` `react-docstore` `self-ask-with-search` `conversational-react-description` `chat-zero-shot-react-description`, `chat-conversational-react-description`,
