---
title: 20230514 - 制作私有运维知识库的demo (2)
confluence_page_id: 2130922
created_at: 2023-05-13T17:18:35+00:00
updated_at: 2023-05-18T14:30:45+00:00
---

继续之前的工作

# 人工修订文档

增加了总的内容介绍和每部分的概要补全: 

![image2023-5-16 0:3:13.png](/assets/01KJBYZYZDAJKTP18T9VCBWZ1M/image2023-5-16%200%3A3%3A13.png)

效果: 

问题1: 

![image2023-5-16 0:4:49.png](/assets/01KJBYZYZDAJKTP18T9VCBWZ1M/image2023-5-16%200%3A4%3A49.png)

问题2: 

![image2023-5-16 0:5:24.png](/assets/01KJBYZYZDAJKTP18T9VCBWZ1M/image2023-5-16%200%3A5%3A24.png)

问题3: 

![image2023-5-16 0:6:12.png](/assets/01KJBYZYZDAJKTP18T9VCBWZ1M/image2023-5-16%200%3A6%3A12.png)

答案错误, 构造一些相关问题

![image2023-5-16 0:27:6.png](/assets/01KJBYZYZDAJKTP18T9VCBWZ1M/image2023-5-16%200%3A27%3A6.png)

问题4: 

![image2023-5-16 0:6:28.png](/assets/01KJBYZYZDAJKTP18T9VCBWZ1M/image2023-5-16%200%3A6%3A28.png)

问题4-2: 

![image2023-5-16 0:6:52.png](/assets/01KJBYZYZDAJKTP18T9VCBWZ1M/image2023-5-16%200%3A6%3A52.png)

# TODO

1 incomplete 根据问题4的状况: 需要将问题诱导到 Linux/MySQL 等 外部工具的关键词方向, 否则不会调用外部工具 2 incomplete token计算有问题, "主机环境初始化的主要步骤" 的解答会报错, token超长, 跟聊天历史的长度有关 3 incomplete 问题3中, 大模型对 "主机环境初始化前" 的边界界定不同, 此处需要找到方法 分析出 其中的歧义 

# 对文档进行预处理

计划: 先对文档进行summary, 列出大纲, 然后对文档进行重写

# 获取摘要

代码: [data_preprocess.rewrite.ipynb](/assets/01KJBYZYZDAJKTP18T9VCBWZ1M/data_preprocess.rewrite.ipynb)

效果并不好, 原因分析:

  - 对代码块没有良好切分 (尝试: 对confluence的数据进行重新获取, 从"存储格式" 中提取信息, 但这需要修改 html2text 项目, 使其兼容 confluence storage format)
  - 对文档切片后进行map-reduce处理, 切片的切口处, 无法保留文档之前的章节信息, 导致reduce后出来的章节层次不清晰 (尝试: 调整提示词? )

# 对文档进行重写

在没有summary和大纲的情况下, 是否能对文档进行重写? 

或者尝试使用超大上下文模型, 比如claude, 来暂时避开这个问题

# 增加streaming功能, 模拟聊天效果

目前的状况: 通过llama_index, 调用tool, tool调用llm, 此时streaming不能触发监听handler, 无法实时突出结果. streaming触发堆栈: 

```
(Pdb) bt
  /usr/lib/python3.9/bdb.py(580)run()
-> exec(cmd, globals, locals)
  <string>(1)<module>()
  /data/huangyan/opssage/4.py(503)<module>()
-> asyncio.run(main())
  /root/miniconda3/envs/py39/lib/python3.9/asyncio/runners.py(44)run()
-> return loop.run_until_complete(main)
  /root/miniconda3/envs/py39/lib/python3.9/asyncio/base_events.py(634)run_until_complete()
-> self.run_forever()
  /root/miniconda3/envs/py39/lib/python3.9/asyncio/base_events.py(601)run_forever()
-> self._run_once()
  /root/miniconda3/envs/py39/lib/python3.9/asyncio/base_events.py(1905)_run_once()
-> handle._run()
  /root/miniconda3/envs/py39/lib/python3.9/asyncio/events.py(80)_run()
-> self._context.run(self._callback, *self._args)
  /root/miniconda3/envs/py39/lib/python3.9/site-packages/langchain/agents/agent.py(878)_aperform_agent_action()
-> observation = await tool.arun(
  /root/miniconda3/envs/py39/lib/python3.9/site-packages/langchain/tools/base.py(288)arun()
-> await self._arun(*tool_args, run_manager=run_manager, **tool_kwargs)
  /data/huangyan/opssage/4.py(332)_arun()
-> response = await self.query_engine.aquery(tool_input)
  /root/miniconda3/envs/py39/lib/python3.9/site-packages/llama_index/indices/query/base.py(25)aquery()
-> return await self._aquery(str_or_query_bundle)
  /root/miniconda3/envs/py39/lib/python3.9/site-packages/llama_index/query_engine/retriever_query_engine.py(167)_aquery()
-> response = await self._response_synthesizer.asynthesize(
  /root/miniconda3/envs/py39/lib/python3.9/site-packages/llama_index/indices/query/response_synthesis.py(194)asynthesize()
-> response_str = await self._response_builder.aget_response(
  /root/miniconda3/envs/py39/lib/python3.9/site-packages/llama_index/indices/response/response_builder.py(270)aget_response()
-> return self.get_response(query_str, text_chunks, prev_response)
  /root/miniconda3/envs/py39/lib/python3.9/site-packages/llama_index/indices/response/response_builder.py(295)get_response()
-> response = super().get_response(
  /root/miniconda3/envs/py39/lib/python3.9/site-packages/llama_index/token_counter/token_counter.py(78)wrapped_llm_predict()
-> f_return_val = f(_self, *args, **kwargs)
  /root/miniconda3/envs/py39/lib/python3.9/site-packages/llama_index/indices/response/response_builder.py(134)get_response()
-> response = self._give_response_single(
  /root/miniconda3/envs/py39/lib/python3.9/site-packages/llama_index/indices/response/response_builder.py(179)_give_response_single()
-> response, formatted_prompt = self._service_context.llm_predictor.stream(
> /root/miniconda3/envs/py39/lib/python3.9/site-packages/llama_index/llm_predictor/base.py(270)stream()
-> formatted_prompt = prompt.format(llm=self._llm, **prompt_args)

``` 

判断: llama_index不能将callback传入tool中的llm调用, 只能使用绕开方法 (代码质量很糟糕)

代码: [opssage (2).py](/assets/01KJBYZYZDAJKTP18T9VCBWZ1M/opssage%20%282%29.py)

# TODO

4 incomplete 只能显示知识库内的知识 5 incomplete 使用langchain, 消除对llama_index的调用
