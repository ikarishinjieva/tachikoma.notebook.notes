---
title: 20230417 - 测试chatglm-6b + llama_index
confluence_page_id: 2130658
created_at: 2023-04-18T02:23:40+00:00
updated_at: 2023-04-19T07:00:36+00:00
---

# 归档

[chatglm-6b-llama_index.ipynb](/assets/01KJBYXVTC6KDC68A0P56CC6ZA/chatglm-6b-llama_index.ipynb)

# 效果

![image2023-4-19 14:27:45.png](/assets/01KJBYXVTC6KDC68A0P56CC6ZA/image2023-4-19%2014%3A27%3A45.png)

出现中英文混杂

![image2023-4-19 14:29:1.png](/assets/01KJBYXVTC6KDC68A0P56CC6ZA/image2023-4-19%2014%3A29%3A1.png)

对dynamically的理解不对

# TODO

  - 测试: <https://github.com/imClumsyPanda/langchain-ChatGLM>

# 诊断 warning

```
Token indices sequence length is longer than the specified maximum sequence length for this model (5446 > 2048). Running this sequence through the model will result in indexing errors
``` 

代码: 

```
    def _eventual_warn_about_too_long_sequence(self, ids: List[int], max_length: Optional[int], verbose: bool):
        """
        Depending on the input and internal state we might trigger a warning about a sequence that is too long for its
        corresponding model

        Args:
            ids (`List[str]`): The ids produced by the tokenization
            max_length (`int`, *optional*): The max_length desired (does not trigger a warning if it is set)
            verbose (`bool`): Whether or not to print more information and warnings.

        """
        if max_length is None and len(ids) > self.model_max_length and verbose:
            if not self.deprecation_warnings.get("sequence-length-is-longer-than-the-specified-maximum", False):
                logger.warning(
                    "Token indices sequence length is longer than the specified maximum sequence length "
                    f"for this model ({len(ids)} > {self.model_max_length}). Running this sequence through the model "
                    "will result in indexing errors"
                )
            self.deprecation_warnings["sequence-length-is-longer-than-the-specified-maximum"] = True
``` 

条件: len(ids) > self.model_max_length

在函数tokenizer_fn中, 增加打印堆栈的代码, 获取调用tokenizer的堆栈: 

```
import traceback
first_print_stacktrace = False

def tokenizer_fn(text: str) -> List:
    global first_print_stacktrace
    if not first_print_stacktrace:
        print("stackback: ------\n")
        traceback.print_stack()
        print("------\n")
        first_print_stacktrace = True
    
    ret = tokenizer(text, return_tensors="pt")["input_ids"]
    ret = ret.to(device)
    return ret
``` 

获得的堆栈: 

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
  File "/tmp/ipykernel_2708416/141392982.py", line 1, in <module>
    response = new_index.query("I want to increase mysql buffer pool, what's the steps?")
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
  File "/root/miniconda3/lib/python3.8/site-packages/llama_index/indices/query/base.py", line 382, in _query
    nodes = self.retrieve(query_bundle)
  File "/root/miniconda3/lib/python3.8/site-packages/llama_index/indices/query/base.py", line 249, in retrieve
    nodes = self._retrieve(query_bundle, similarity_tracker=similarity_tracker)
  File "/root/miniconda3/lib/python3.8/site-packages/llama_index/indices/vector_store/base_query.py", line 53, in _retrieve
    self._service_context.embed_model.get_agg_embedding_from_queries(
  File "/root/miniconda3/lib/python3.8/site-packages/llama_index/embeddings/base.py", line 79, in get_agg_embedding_from_queries
    query_embeddings = [self.get_query_embedding(query) for query in queries]
  File "/root/miniconda3/lib/python3.8/site-packages/llama_index/embeddings/base.py", line 79, in <listcomp>
    query_embeddings = [self.get_query_embedding(query) for query in queries]
  File "/root/miniconda3/lib/python3.8/site-packages/llama_index/embeddings/base.py", line 69, in get_query_embedding
    query_tokens_count = len(self._tokenizer(query))
  File "/tmp/ipykernel_2708416/675257185.py", line 11, in tokenizer_fn
    traceback.print_stack()
```
