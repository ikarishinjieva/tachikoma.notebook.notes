---
title: 20230405 - 测试 llama-index + GPT4All
confluence_page_id: 2130567
created_at: 2023-04-05T11:21:45+00:00
updated_at: 2023-04-06T15:04:32+00:00
---

从new bing中获取信息: 

  - Huggingface的Transformers库, 提供了预训练模型
    - 提供了用于自然语言理解（NLU）和自然语言生成（NLG）的BERT家族通用结构，包含超过32种、涵盖100多种语言的预训练模型，如BERT、GPT-2、RoBERTa、XLM、DistilBert、XLNet等。
  - LangChain
    - 是一个开源的代码库，它是一种LLMs接口框架，允许用户围绕大型语言模型快速构建应用程序和管道。它直接与OpenAI的GPT-3和GPT-3.5模型以及Hugging Face的开源替代品（如Google的flan-t5模型）集成。LangChain提供了一系列模块，这些模块可以组合在一起，用于创建复杂的应用程序，也可以单独用于简单的应用程序
  - llama-index
    - 是一个项目，它提供了一个中心接口，可以将您的语言模型（LLM）与外部数据连接起来。它提供了一组数据结构，可以为各种LLM任务索引大量数据，并消除了关于提示大小限制和数据摄入的问题。LlamaIndex 是您的外部数据和 LLM 之间的一个简单、灵活的接口。它提供了一组数据结构，可以为各种LLM任务索引大量数据，并消除了关于提示大小限制和数据摄入的问题。LlamaIndex 可以帮助您训练pdf等文档格式的模型

GPT4All 自身带的cli, 效果不错

项目调用关系:

  - langchain.llms.GPT4All
    - gpt4all
      - pyllamacpp

# Llama-index

  - LLMPredictor 是 LangChain中 LLMChain 的包装
    - <https://github.com/jerryjliu/llama_index/blob/66db86d2e875fb47384a77a0469bc6c6f45c866e/docs/reference/service_context/llm_predictor.rst#L6>

# 测试LangChain支持GPT4All

<https://python.langchain.com/en/latest/ecosystem/gpt4all.html>

TODO: 测试GPT4All, 并通过llama-index -> LangChain -> GPT4All模型

  - 模型文件, 下载到了 root@10.186.16.136:/data/huangyan/gpt4all-lora-quantized.bin
  - 模型文件, 已复制到 root@10.186.17.104的容器中, 测试langchain与gpt4all的结合使用: <https://python.langchain.com/en/latest/ecosystem/gpt4all.html>
  - 使用到了 pyllamacpp 库
    - <https://github.com/nomic-ai/pyllamacpp>: Official supported Python bindings for llama.cpp + gpt4all

      - llama.cpp is a port of Facebook's LLaMA model in pure C/C++
    - pip需要使用编译安装:

```
git clone --recursive https://github.com/nomic-ai/pyllamacpp && cd pyllamacpp
pip install .
```

    - 在10.186.17.104中的容器内运行会报 Illegal instruction. 原因不详
    - 迁移到 10.186.16.136物理机上运行
  - 需要转换模型
    - 根据 <https://github.com/nomic-ai/pyllamacpp/issues/5> 找到 tokenizer
    - 进行模型转换 (旧模型 -> 新模型)  
  

```
pyllamacpp-convert-gpt4all gpt4all/chat/gpt4all-lora-quantized.bin gpt4all/chat/tokenizer.model gpt4all/chat/gpt4all-lora-quantized.converted.bin
``` 
    - 可以跑通demo: <https://github.com/nomic-ai/pyllamacpp>
  - 与LangChain结合比较简单, 可跑通demo: <https://python.langchain.com/en/latest/ecosystem/gpt4all.html>

结合GPT4All与llama_index: 

```
#!/usr/bin/env python
# coding: utf-8

# In[2]:

proxy_addr = "http://192.168.22.8:7890"
rebuild = False

import os
os.environ["http_proxy"] = proxy_addr
os.environ["https_proxy"] = proxy_addr

print("loaded")

# In[3]:

from langchain.llms import GPT4All
from langchain.embeddings.huggingface import HuggingFaceEmbeddings
from llama_index import PromptHelper, GPTSimpleVectorIndex, ServiceContext, LLMPredictor, SimpleDirectoryReader, LangchainEmbedding

llm_predictor = LLMPredictor(llm=GPT4All(model="./gpt4all/chat/gpt4all-lora-quantized.converted.bin", n_ctx=512, n_threads=8))
hfemb = HuggingFaceEmbeddings()
embed_model = LangchainEmbedding(hfemb)

max_input_size = 4000
num_output = 256
max_chunk_overlap = 0

prompt_helper = PromptHelper(max_input_size, num_output, max_chunk_overlap)

service_context = ServiceContext.from_defaults(
    llm_predictor=llm_predictor, 
    prompt_helper=prompt_helper,
    embed_model=embed_model
)

if rebuild:
    # Load documents from the 'data' directory
    documents = SimpleDirectoryReader('data').load_data()
    index = GPTSimpleVectorIndex.from_documents(
        documents, service_context=service_context
    )
    index.save_to_disk('index.json')
else:
    index = GPTSimpleVectorIndex.load_from_disk('index.json', service_context=service_context)

# In[4]:

print(index)

# In[7]:

resp = index.query("what should I do if buffer pool hit rate is too low", mode="default", service_context=service_context)
print(resp)

# In[6]:

print(resp)

# In[ ]:

``` 

使用MySQL手册作为输入, 效果并不好, 并会有报错. 先解决报错, 再测试效果

### 报错1: 

```
llama_generate: error: prompt is too long (1344 tokens, max 508)
``` 

代码位置为 llama_index引用的pyllamacpp: <https://github.com/nomic-ai/pyllamacpp/blob/c55660790f5a84eb623b01deaebc76741d4758ae/src/main.cpp#L170>

1344 为 embd_inp.size(), embedding的输入?

508 为 n_ctx - 4, n_ctx 是建立 GPT4All 模型时指定的. 未能解决报错

### 报错2: 

```
Token indices sequence length is longer than the specified maximum sequence length for this model (3693 > 1024). Running this sequence through the model will result in indexing errors
``` 

在代码中插桩, 获取调用堆栈: 

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
  File "/tmp/ipykernel_3645101/3684562405.py", line 1, in <module>
    resp = index.query("what should I do if buffer pool hit rate is too low", mode="default", service_context=service_context)
  File "/root/miniconda3/lib/python3.8/site-packages/llama_index/indices/base.py", line 244, in query
    return query_runner.query(query_str)
  File "/root/miniconda3/lib/python3.8/site-packages/llama_index/indices/query/query_runner.py", line 342, in query
    return query_combiner.run(query_bundle, level)
  File "/root/miniconda3/lib/python3.8/site-packages/llama_index/indices/query/query_combiner/base.py", line 66, in run
    return self._query_runner.query_transformed(
  File "/root/miniconda3/lib/python3.8/site-packages/llama_index/indices/query/query_runner.py", line 202, in query_transformed
    return query_obj.query(query_bundle)
  File "/root/miniconda3/lib/python3.8/site-packages/llama_index/token_counter/token_counter.py", line 78, in wrapped_llm_predict
    f_return_val = f(_self, *args, **kwargs)
  File "/root/miniconda3/lib/python3.8/site-packages/llama_index/indices/query/base.py", line 396, in query
    return self._query(query_bundle)
  File "/root/miniconda3/lib/python3.8/site-packages/llama_index/indices/query/base.py", line 383, in _query
    return self.synthesize(query_bundle, nodes)
  File "/root/miniconda3/lib/python3.8/site-packages/llama_index/indices/query/base.py", line 340, in synthesize
    response_str = self._give_response_for_nodes(
  File "/root/miniconda3/lib/python3.8/site-packages/llama_index/indices/query/base.py", line 223, in _give_response_for_nodes
    response = response_builder.get_response(
  File "/root/miniconda3/lib/python3.8/site-packages/llama_index/indices/response/builder.py", line 387, in get_response
    return self._get_response_default(query_str, prev_response)
  File "/root/miniconda3/lib/python3.8/site-packages/llama_index/indices/response/builder.py", line 242, in _get_response_default
    return self.get_response_over_chunks(
  File "/root/miniconda3/lib/python3.8/site-packages/llama_index/indices/response/builder.py", line 224, in get_response_over_chunks
    response = self.give_response_single(
  File "/root/miniconda3/lib/python3.8/site-packages/llama_index/indices/response/builder.py", line 172, in give_response_single
    text_chunks = qa_text_splitter.split_text(text_chunk)
  File "/root/miniconda3/lib/python3.8/site-packages/llama_index/langchain_helpers/text_splitter.py", line 118, in split_text
    text_splits = self.split_text_with_overlaps(text, extra_info_str=extra_info_str)
  File "/root/miniconda3/lib/python3.8/site-packages/llama_index/langchain_helpers/text_splitter.py", line 170, in split_text_with_overlaps
    cur_idx = self._reduce_chunk_size(start_idx, cur_idx, splits)
  File "/root/miniconda3/lib/python3.8/site-packages/llama_index/langchain_helpers/text_splitter.py", line 55, in _reduce_chunk_size
    self.tokenizer(self._separator.join(splits[start_idx:cur_idx]))
  File "/root/miniconda3/lib/python3.8/site-packages/
llama_index/utils.py", line 52, in tokenizer_fn
    return tokenizer(text)["input_ids"]
  File "/root/miniconda3/lib/python3.8/site-packages/transformers/tokenization_utils_base.py", line 2530, in __call__
    encodings = self._call_one(text=text, text_pair=text_pair, **all_kwargs)
  File "/root/miniconda3/lib/python3.8/site-packages/transformers/tokenization_utils_base.py", line 2636, in _call_one
    return self.encode_plus(
  File "/root/miniconda3/lib/python3.8/site-packages/transformers/tokenization_utils_base.py", line 2709, in encode_plus
    return self._encode_plus(
  File "/root/miniconda3/lib/python3.8/site-packages/transformers/models/gpt2/tokenization_gpt2_fast.py", line 176, in _encode_plus
    return super()._encode_plus(*args, **kwargs)
  File "/root/miniconda3/lib/python3.8/site-packages/transformers/tokenization_utils_fast.py", line 500, in _encode_plus
    batched_output = self._batch_encode_plus(
  File "/root/miniconda3/lib/python3.8/site-packages/transformers/models/gpt2/tokenization_gpt2_fast.py", line 166, in _batch_encode_plus
    return super()._batch_encode_plus(*args, **kwargs)
  File "/root/miniconda3/lib/python3.8/site-packages/transformers/tokenization_utils_fast.py", line 475, in _batch_encode_plus
    self._eventual_warn_about_too_long_sequence(input_ids, max_length, verbose)
  File "/root/miniconda3/lib/python3.8/site-packages/transformers/tokenization_utils_base.py", line 3562, in _eventual_warn_about_too_long_sequence
    traceback.print_stack()
``` 

未能解决报错

# 结果

[llama_index_gpt4all (1).ipynb](/assets/01KJBYXRTFKTV6KQWHN0ZVT2KY/llama_index_gpt4all%20%281%29.ipynb)

测试效果并不理想
