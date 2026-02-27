---
title: 20230523 - llama_index为什么不使用refine
confluence_page_id: 2392075
created_at: 2023-05-22T16:54:40+00:00
updated_at: 2023-05-23T07:57:48+00:00
---

# 问题和解决

发现llama_index没使用refine

经排查, 是因为 index.as_query_engine中, response_mode的默认值是compact, 其作用是 CompactAndRefine, 将多个文章片段合并

由于使用的文章片段较少, 所以被压缩到一个query中, 不会使用refine

当调大 similarity_top_k , 会返回更多的文章片段, 就会用到refine

# 新的问题

当调大 similarity_top_k, 调用llm会超过max_token: 

```
Exception in thread Thread-6:
Traceback (most recent call last):
  File "/root/miniconda3/envs/py39/lib/python3.9/threading.py", line 980, in _bootstrap_inner
    self.run()
  File "/root/miniconda3/envs/py39/lib/python3.9/threading.py", line 917, in run
    self._target(*self._args, **self._kwargs)
  File "/root/miniconda3/envs/py39/lib/python3.9/site-packages/llama_index/llm_predictor/base.py", line 210, in _predict
    llm_prediction = retry_on_exceptions_with_backoff(
  File "/root/miniconda3/envs/py39/lib/python3.9/site-packages/llama_index/utils.py", line 177, in retry_on_exceptions_with_backoff
    return lambda_fn()
  File "/root/miniconda3/envs/py39/lib/python3.9/site-packages/llama_index/llm_predictor/base.py", line 211, in <lambda>
    lambda: llm_chain.predict(**full_prompt_args),
  File "/root/miniconda3/envs/py39/lib/python3.9/site-packages/langchain/chains/llm.py", line 213, in predict
    return self(kwargs, callbacks=callbacks)[self.output_key]
  File "/root/miniconda3/envs/py39/lib/python3.9/site-packages/langchain/chains/base.py", line 140, in __call__
    raise e
  File "/root/miniconda3/envs/py39/lib/python3.9/site-packages/langchain/chains/base.py", line 134, in __call__
    self._call(inputs, run_manager=run_manager)
  File "/root/miniconda3/envs/py39/lib/python3.9/site-packages/langchain/chains/llm.py", line 69, in _call
    response = self.generate([inputs], run_manager=run_manager)
  File "/root/miniconda3/envs/py39/lib/python3.9/site-packages/langchain/chains/llm.py", line 79, in generate
    return self.llm.generate_prompt(
  File "/root/miniconda3/envs/py39/lib/python3.9/site-packages/langchain/llms/base.py", line 134, in generate_prompt
    return self.generate(prompt_strings, stop=stop, callbacks=callbacks)
  File "/tmp/ipykernel_13592/3489256537.py", line 20, in generate
  File "/root/miniconda3/envs/py39/lib/python3.9/site-packages/langchain/llms/base.py", line 191, in generate
    raise e
  File "/root/miniconda3/envs/py39/lib/python3.9/site-packages/langchain/llms/base.py", line 185, in generate
    self._generate(prompts, stop=stop, run_manager=run_manager)
  File "/root/miniconda3/envs/py39/lib/python3.9/site-packages/langchain/llms/openai.py", line 303, in _generate
    for stream_resp in completion_with_retry(
  File "/root/miniconda3/envs/py39/lib/python3.9/site-packages/langchain/llms/openai.py", line 106, in completion_with_retry
    return _completion_with_retry(**kwargs)
  File "/root/miniconda3/envs/py39/lib/python3.9/site-packages/tenacity/__init__.py", line 289, in wrapped_f
    return self(f, *args, **kw)
  File "/root/miniconda3/envs/py39/lib/python3.9/site-packages/tenacity/__init__.py", line 379, in __call__
    do = self.iter(retry_state=retry_state)
  File "/root/miniconda3/envs/py39/lib/python3.9/site-packages/tenacity/__init__.py", line 314, in iter
    return fut.result()
  File "/root/miniconda3/envs/py39/lib/python3.9/concurrent/futures/_base.py", line 439, in result
    return self.__get_result()
  File "/root/miniconda3/envs/py39/lib/python3.9/concurrent/futures/_base.py", line 391, in __get_result
    raise self._exception
  File "/root/miniconda3/envs/py39/lib/python3.9/site-packages/tenacity/__init__.py", line 382, in __call__
    result = fn(*args, **kwargs)
  File "/root/miniconda3/envs/py39/lib/python3.9/site-packages/langchain/llms/openai.py", line 104, in _completion_with_retry
    return llm.client.create(**kwargs)
  File "/root/miniconda3/envs/py39/lib/python3.9/site-packages/openai/api_resources/completion.py", line 25, in create
    return super().create(*args, **kwargs)
  File "/root/miniconda3/envs/py39/lib/python3.9/site-packages/openai/api_resources/abstract/engine_api_resource.py", line 153, in create
    response, _, api_key = requestor.request(
  File "/root/miniconda3/envs/py39/lib/python3.9/site-packages/openai/api_requestor.py", line 230, in request
    resp, got_stream = self._interpret_response(result, stream)
  File "/root/miniconda3/envs/py39/lib/python3.9/site-packages/openai/api_requestor.py", line 624, in _interpret_response
    self._interpret_response_line(
  File "/root/miniconda3/envs/py39/lib/python3.9/site-packages/openai/api_requestor.py", line 687, in _interpret_response_line
    raise self.handle_error_response(
openai.error.InvalidRequestError: This model's maximum context length is 4097 tokens, however you requested 4215 tokens (3547 in your prompt; 668 for the completion). Please reduce your prompt; or completion length.
``` 

对于文档片段用量的逻辑整理: 

CompactAndRefine.get_response

  - new_texts=prompt_helper.compact_text_chunks(...)
  - Refine.get_response(new_texts)
    - for chunk in new_texts: 
      - _give_response_single
        - 初始化切分器: qa_text_splitter = prompt_helper.get_text_splitter_given_prompt
          - get_text_splitter_given_prompt

```
    def get_text_splitter_given_prompt(
        self, prompt: Prompt, num_chunks: int, padding: Optional[int] = 1
    ) -> TokenTextSplitter:
        """Get text splitter given initial prompt.

        Allows us to get the text splitter which will split up text according
        to the desired chunk size.

        """
        # generate empty_prompt_txt to compute initial tokens
 
# prompt的净长度
        empty_prompt_txt = self._get_empty_prompt_txt(prompt)
 
# 计算chunk_size
        chunk_size = self.get_chunk_size_given_prompt(
            empty_prompt_txt, num_chunks, padding=padding
        )
 
# 生成text_splitter
        text_splitter = TokenTextSplitter(
            separator=self._separator,
            chunk_size=chunk_size,
            chunk_overlap=self.max_chunk_overlap // num_chunks,
            tokenizer=self._tokenizer,
        )
        return text_splitter
```

            - get_chunk_size_given_prompt的逻辑
              - result = (self.max_input_size - num_prompt_tokens - self.num_output) // num_chunks  

                - = (max_input_size配置 - prompt净长度的tokens - num_output配置) // 1
                - result受到 embedding_limit 和 chunk_size_limit 的约束
              - max_input_size = modelname_to_contextsize(...) (上下文总大小)
              - num_output = llm.max_tokens (输出大小)
        - 切分文档: for text_chunks = qa_text_splitter.split_text(text_chunk):
          - 第一个片段直接predict: llm_predictor.predict(text_qa_template)
          - 后续片段进行refine: _refine_response_single
            - llm_predictor.predict(refine_template)
