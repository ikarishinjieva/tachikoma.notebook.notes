---
title: 20230423 - 测试 llama_index + openai/azure
confluence_page_id: 2130696
created_at: 2023-04-23T02:38:27+00:00
updated_at: 2023-05-02T16:15:09+00:00
---

# 测试文件

开通了azure的openai服务

使用text-embedding-ada-002作为embeding模型, text-davinci-003作为LLM模型

[附件: llama_index-openai-azure.ipynb] 

# 测试结果

![image2023-4-23 13:18:39.png](/assets/01KJBYYMHN0C5CA4ARZ21B6AAN/image2023-4-23%2013%3A18%3A39.png)

![image2023-4-23 13:19:56.png](/assets/01KJBYYMHN0C5CA4ARZ21B6AAN/image2023-4-23%2013%3A19%3A56.png)

![image2023-4-23 13:49:47.png](/assets/01KJBYYMHN0C5CA4ARZ21B6AAN/image2023-4-23%2013%3A49%3A47.png)

# 测试模型 gpt-35-turbo-0301

报错: 

```
INFO:openai:error_code=BadRequest error_message='logprobs, best_of and echo parameters are not available on gpt-35-turbo model. Please remove the parameter and try again. For more details, see https://go.microsoft.com/fwlink/?linkid=2227346.' error_param=None error_type=None message='OpenAI API error received' stream_error=False

``` 

需要升级langchain, 并使用AzureChatOpenAI

效果: 

![image2023-4-23 13:51:25.png](/assets/01KJBYYMHN0C5CA4ARZ21B6AAN/image2023-4-23%2013%3A51%3A25.png)

![image2023-4-23 13:51:43.png](/assets/01KJBYYMHN0C5CA4ARZ21B6AAN/image2023-4-23%2013%3A51%3A43.png)

![image2023-4-23 13:55:34.png](/assets/01KJBYYMHN0C5CA4ARZ21B6AAN/image2023-4-23%2013%3A55%3A34.png)

效果不如text-davinci-003好. 带有上下文连续性

# Total LLM token usage和embedding token usage的作用

根据llama_index的代码,

  - LLM token 是计算前后 llm_predictor.total_tokens_used 的差值
    - llm_predictor.total_tokens_used = prompt_tokens_count + prediction_tokens_count
    - 也就是prompt的长度, 加上预测后返回结果的长度
  - embedding token 是计算前后 embed_model.total_tokens_used 的差值
    - embed_model.total_tokens_used 是 embed 之前的输入内容的长度
  - 上图日志中的统计是 query 方法. 找到标记 "@llm_token_counter("query")" 的方法: 
    - BaseGPTIndexQuery.query (其中包括 retrieve 和 synthesize)

# 诊断warning

warning信息: 

```
Token indices sequence length is longer than the specified maximum sequence length for this model (3415 > 1024). Running this sequence through the model will result in indexing errors
``` 

在 _eventual_warn_about_too_long_sequence, 增加traceback打印stack的代码

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
                import traceback
                traceback.print_stack()
                logger.warning(
                    "Token indices sequence length is longer than the specified maximum sequence length "
                    f"for this model ({len(ids)} > {self.model_max_length}). Running this sequence through the model "
                    "will result in indexing errors"
                )
            self.deprecation_warnings["sequence-length-is-longer-than-the-specified-maximum"] = True
``` 

获取堆栈: 

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
  File "/tmp/ipykernel_2972995/1005497613.py", line 2, in <module>
    response = index.query("how to enlarge buffer pool")
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
``` 

变更tokenizer参数, 可消除warning: 

```
# tokenizer
from llama_index.utils import globals_helper
import transformers
from typing import List
tokenizer = transformers.GPT2TokenizerFast.from_pretrained("gpt2")
def tokenizer_fn(text: str) -> List:
    return tokenizer(text, truncation=True, max_length=1024)["input_ids"]
globals_helper._tokenizer = tokenizer_fn
``` 

但这一步骤只是截断输出, 需要调查为什么输入会导致输出超长:

从模型得到的prompt_helper参数为: 

```
llm = AzureOpenAI(deployment_name="text-davinci-003")
llm_predictor = LLMPredictor(llm=llm)
prompt_helper = PromptHelper.from_llm_predictor(llm_predictor)
print("prompt_helper:", prompt_helper.max_input_size, prompt_helper.num_output, prompt_helper.max_chunk_overlap)
 
输出: 3900 256 200
``` 

参数的使用堆栈: 

  - get_text_from_nodes
    - get_text_splitter_given_prompt
      - chunk_size = self.get_chunk_size_given_prompt
        - result = (max_input_size - prompt长度 - num_output) / num_chunks
        - result 受到 embedding_limit 约束
        - result 受到 chunk_size_limit 约束
      - TokenTextSplitter(..., chunk_size, ...)

控制 chunk_size_limit+use_chunk_size_limit 或者 embedding_limit 可以控制warning的行为: (好像不生效??)

```
# model
llm = AzureOpenAI(deployment_name="text-davinci-003")
#llm = AzureChatOpenAI(deployment_name="gpt-35-turbo-0301")
llm_predictor = LLMPredictor(llm=llm)
embed_model = LangchainEmbedding(OpenAIEmbeddings(model="text-embedding-ada-002"))
prompt_helper = PromptHelper.from_llm_predictor(llm_predictor)
prompt_helper.chunk_size_limit = 1024 * 0.85 # 15% margin for extra string
prompt_helper.use_chunk_size_limit = True
print("prompt_helper:", prompt_helper.max_input_size, prompt_helper.num_output, prompt_helper.max_chunk_overlap)
service_context = ServiceContext.from_defaults(llm_predictor=llm_predictor, embed_model=embed_model, prompt_helper=prompt_helper)

``` 

在service_context中设置 chunk_size_limit 有效, 其初始化过程会将chunk_size_limit带入各组件 (此处llama_index的配置比较乱, service_context是统一入口)

```
llm = AzureOpenAI(deployment_name="text-davinci-003")
llm_predictor = LLMPredictor(llm=llm)
embed_model = LangchainEmbedding(OpenAIEmbeddings(model="text-embedding-ada-002"))
service_context = ServiceContext.from_defaults(llm_predictor=llm_predictor, embed_model=embed_model, chunk_size_limit=1024)
``` 

将chunk_size_limit调小后, 效果变好 (三段对应了之前代码中的三个问答)

![image2023-4-24 10:53:27.png](/assets/01KJBYYMHN0C5CA4ARZ21B6AAN/image2023-4-24%2010%3A53%3A27.png)

总共有将近10000个embedding向量: 

```
less index.openai.azure.chunk_size_1000.json | jq '.vector_store.simple_vector_store_data_dict.embedding_dict | keys' | wc -l
 
9755
``` 

# 确认知识的来源

从confluence上导出mysqlbackup的使用文档 (中文).

在将文档加入index前, 询问问题 "如何用mysqlbackup进行单文件全备恢复, 给我具体的步骤", 获得答案: 

```
1. Start the MySQL server with the '--log-bin' option enabled.
2. Use the command `mysqldump --all-databases --master-data --single-transaction > backup_filename.sql` to generate the full backup file.
3. Move the backup file to the desired destination.
4. To restore the full backup, use the command `mysql < backup_filename.sql`.
``` 

在加入文档后, 获得答案: 

```
1. 下载并安装MySQL Enterprise Backup（MEB）
2. 运行MEB备份，将备份文件存放到单一文件中
3. 使用MEB的恢复命令进行恢复，指定备份文件的路径
4. 根据恢复过程中的提示，完成恢复
``` 

代码: 

```
from llama_index import download_loader
ReadabilityWebPageReader = download_loader("ReadabilityWebPageReader")

loader = ReadabilityWebPageReader()
documents = loader.load_data(url='file:///data/huangyan/confluence1.html')

for doc in documents:
    index.insert(doc)
 
# 注意, 在jupyter notebook中运行, 会报错关于async io的使用
``` 

# 代码文件

[附件: llama_index-openai-azure (1).ipynb] 

测试效果: 

![image2023-4-25 13:44:9.png](/assets/01KJBYYMHN0C5CA4ARZ21B6AAN/image2023-4-25%2013%3A44%3A9.png)

![image2023-4-25 13:44:22.png](/assets/01KJBYYMHN0C5CA4ARZ21B6AAN/image2023-4-25%2013%3A44%3A22.png)

在材料原文([归档 12.zip](/assets/01KJBYYMHN0C5CA4ARZ21B6AAN/%E5%BD%92%E6%A1%A3%2012.zip))中, 

![image2023-4-25 13:46:10.png](/assets/01KJBYYMHN0C5CA4ARZ21B6AAN/image2023-4-25%2013%3A46%3A10.png)

并未对service mysql stop做出标注, 而是在之前章节中有步骤描述, LLM将两者对应在一起: 

![image2023-4-25 13:46:51.png](/assets/01KJBYYMHN0C5CA4ARZ21B6AAN/image2023-4-25%2013%3A46%3A51.png)

# chatbot

增加聊天bot代码: 

```
# chatbot
# https://github.com/jerryjliu/llama_index/blob/c261a1751f85c6308adc31ef5ca1d8b45f168be7/docs/how_to/integrations/using_with_langchain.md

from llama_index.langchain_helpers.agents import IndexToolConfig, LlamaToolkit

index_config = IndexToolConfig(
    index=index, 
    name=f"Vector Index",
    description=f"useful for when you want to answer queries about X",
    index_query_kwargs={"similarity_top_k": 3},
    tool_kwargs={"return_direct": True}
)

toolkit = LlamaToolkit(
    index_configs=[index_config],
    graph_configs=[]
)

from langchain.chains.conversation.memory import ConversationBufferMemory
memory = ConversationBufferMemory(memory_key="chat_history")

from llama_index.langchain_helpers.agents import create_llama_chat_agent

agent_chain = create_llama_chat_agent(
    toolkit,
    llm,
    memory=memory,
    verbose=True
)

while True:
    text_input = input("User: ")
    response = agent_chain.run(input=text_input)
    print(f'Agent: {response}')
``` 

结果: 

```
(base) root@r750-136:/data/huangyan# python 1.py
======
User: 如何用mysqlbackup进行备份
Agent: MySQL Backup是一种用于备份MySQL数据库的简单工具。要使用MySQL Backup进行备份，您需要先下载它，然后使用它的命令行工具运行备份任务。您还可以使用MySQL Backup的图形界面版本进行备份，它提供了一个可视化的工具，可以让您轻松地设置备份任务。
User: 如何用mysqlbackup进行单文件全备恢复 给出详细步骤和命令, 对命令的每个参数做出解释
Agent: 使用MySQL Backup进行单文件全备恢复非常简单。首先，您需要使用以下命令下载MySQL Backup：

<code>wget http://www.mysqlbackup.com/mysqlbackup.tar.gz</code>

然后，解压缩文件：

<code>tar -xvf mysqlbackup.tar.gz</code>

接下来，使用以下命令运行MySQL Backup：

<code>./mysqlbackup restore --single-file --restore-file [backup file]</code>

其中，--single-file参数表示您将使用单
``` 

问题: 

  - 好像没有使用到 相应文档, 而是从模型内直接出了知识并翻译成中文, 当问题足够具体时, 能从文档中获取答案. 跟 非chat模式 不同
  - 回答的长度仍然被截断

# chatbot解决中文问答的问题

~~对langchain的流程进行分析:~~

````
- full_inputs = get_full_inputs(intermediate_steps)
	- thoughts = _construct_scratchpad(intermediate_steps)
		```
        for action, observation in intermediate_steps:
            thoughts += action.log
            thoughts += f"\n{self.observation_prefix}{observation}\n{self.llm_prefix}"
        return thoughts
        ```
	- new_inputs = {"agent_scratchpad": thoughts, "stop": self._stop}
	- full_inputs = {**kwargs, **new_inputs}

- action = _get_next_action(full_inputs)
```` 

对调用过程进行分析, 开启debug log: 

```
import logging
import sys
logging.basicConfig(stream=sys.stdout, level=logging.DEBUG)
logging.getLogger().addHandler(logging.StreamHandler(stream=sys.stdout))
 
import openai
openai.log='debug'
``` 

获得有问题的情况下输出: 

````
User: 如何用mysqlbackup进行备份

> Entering new AgentExecutor chain...
message='Request to OpenAI API' method=post path=https://tachikoma.openai.azure.com//openai/deployments/text-davinci-003/completions?api-version=2023-03-15-preview
DEBUG:openai:message='Request to OpenAI API' method=post path=https://tachikoma.openai.azure.com//openai/deployments/text-davinci-003/completions?api-version=2023-03-15-preview
message='Request to OpenAI API' method=post path=https://tachikoma.openai.azure.com//openai/deployments/text-davinci-003/completions?api-version=2023-03-15-preview
api_version=2023-03-15-preview data='{"prompt": ["Assistant is a large language model trained by OpenAI.\\n\\nAssistant is designed to be able to assist with a wide range of tasks, from answering simple questions to providing in-depth explanations and discussions on a wide range of topics. As a language model, Assistant is able to generate human-like text based on the input it receives, allowing it to engage in natural-sounding conversations and provide responses that are coherent and relevant to the topic at hand.\\n\\nAssistant is constantly learning and improving, and its capabilities are constantly evolving. It is able to process and understand large amounts of text, and can use this knowledge to provide accurate and informative responses to a wide range of questions. Additionally, Assistant is able to generate its own text based on the input it receives, allowing it to engage in discussions and provide explanations and descriptions on a wide range of topics.\\n\\nOverall, Assistant is a powerful tool that can help with a wide range of tasks and provide valuable insights and information on a wide range of topics. Whether you need help with a specific question or just want to have a conversation about a particular topic, Assistant is here to assist.\\n\\nTOOLS:\\n------\\n\\nAssistant has access to the following tools:\\n\\n> Vector Index: useful for when you want to answer queries about X\\n\\nTo use a tool, please use the following format:\\n\\n```\\nThought: Do I need to use a tool? Yes\\nAction: the action to take, should be one of [Vector Index]\\nAction Input: the input to the action\\nObservation: the result of the action\\n```\\n\\nWhen you have a response to say to the Human, or if you do not need to use a tool, you MUST use the format:\\n\\n```\\nThought: Do I need to use a tool? No\\nAI: [your response here]\\n```\\n\\nBegin!\\n\\nPrevious conversation history:\\n\\n\\nNew input: \\u5982\\u4f55\\u7528mysqlbackup\\u8fdb\\u884c\\u5907\\u4efd\\n"], "temperature": 0.7, "max_tokens": 256, "top_p": 1, "frequency_penalty": 0, "presence_penalty": 0, "n": 1, "best_of": 1, "logit_bias": {}, "stop": ["\\nObservation:", "\\n\\tObservation:"]}' message='Post details'
DEBUG:openai:api_version=2023-03-15-preview data='{"prompt": ["Assistant is a large language model trained by OpenAI.\\n\\nAssistant is designed to be able to assist with a wide range of tasks, from answering simple questions to providing in-depth explanations and discussions on a wide range of topics. As a language model, Assistant is able to generate human-like text based on the input it receives, allowing it to engage in natural-sounding conversations and provide responses that are coherent and relevant to the topic at hand.\\n\\nAssistant is constantly learning and improving, and its capabilities are constantly evolving. It is able to process and understand large amounts of text, and can use this knowledge to provide accurate and informative responses to a wide range of questions. Additionally, Assistant is able to generate its own text based on the input it receives, allowing it to engage in discussions and provide explanations and descriptions on a wide range of topics.\\n\\nOverall, Assistant is a powerful tool that can help with a wide range of tasks and provide valuable insights and information on a wide range of topics. Whether you need help with a specific question or just want to have a conversation about a particular topic, Assistant is here to assist.\\n\\nTOOLS:\\n------\\n\\nAssistant has access to the following tools:\\n\\n> Vector Index: useful for when you want to answer queries about X\\n\\nTo use a tool, please use the following format:\\n\\n```\\nThought: Do I need to use a tool? Yes\\nAction: the action to take, should be one of [Vector Index]\\nAction Input: the input to the action\\nObservation: the result of the action\\n```\\n\\nWhen you have a response to say to the Human, or if you do not need to use a tool, you MUST use the format:\\n\\n```\\nThought: Do I need to use a tool? No\\nAI: [your response here]\\n```\\n\\nBegin!\\n\\nPrevious conversation history:\\n\\n\\nNew input: \\u5982\\u4f55\\u7528mysqlbackup\\u8fdb\\u884c\\u5907\\u4efd\\n"], "temperature": 0.7, "max_tokens": 256, "top_p": 1, "frequency_penalty": 0, "presence_penalty": 0, "n": 1, "best_of": 1, "logit_bias": {}, "stop": ["\\nObservation:", "\\n\\tObservation:"]}' message='Post details'
api_version=2023-03-15-preview data='{"prompt": ["Assistant is a large language model trained by OpenAI.\\n\\nAssistant is designed to be able to assist with a wide range of tasks, from answering simple questions to providing in-depth explanations and discussions on a wide range of topics. As a language model, Assistant is able to generate human-like text based on the input it receives, allowing it to engage in natural-sounding conversations and provide responses that are coherent and relevant to the topic at hand.\\n\\nAssistant is constantly learning and improving, and its capabilities are constantly evolving. It is able to process and understand large amounts of text, and can use this knowledge to provide accurate and informative responses to a wide range of questions. Additionally, Assistant is able to generate its own text based on the input it receives, allowing it to engage in discussions and provide explanations and descriptions on a wide range of topics.\\n\\nOverall, Assistant is a powerful tool that can help with a wide range of tasks and provide valuable insights and information on a wide range of topics. Whether you need help with a specific question or just want to have a conversation about a particular topic, Assistant is here to assist.\\n\\nTOOLS:\\n------\\n\\nAssistant has access to the following tools:\\n\\n> Vector Index: useful for when you want to answer queries about X\\n\\nTo use a tool, please use the following format:\\n\\n```\\nThought: Do I need to use a tool? Yes\\nAction: the action to take, should be one of [Vector Index]\\nAction Input: the input to the action\\nObservation: the result of the action\\n```\\n\\nWhen you have a response to say to the Human, or if you do not need to use a tool, you MUST use the format:\\n\\n```\\nThought: Do I need to use a tool? No\\nAI: [your response here]\\n```\\n\\nBegin!\\n\\nPrevious conversation history:\\n\\n\\nNew input: \\u5982\\u4f55\\u7528mysqlbackup\\u8fdb\\u884c\\u5907\\u4efd\\n"], "temperature": 0.7, "max_tokens": 256, "top_p": 1, "frequency_penalty": 0, "presence_penalty": 0, "n": 1, "best_of": 1, "logit_bias": {}, "stop": ["\\nObservation:", "\\n\\tObservation:"]}' message='Post details'
DEBUG:urllib3.util.retry:Converted retries value: 2 -> Retry(total=2, connect=None, read=None, redirect=None, status=None)
Converted retries value: 2 -> Retry(total=2, connect=None, read=None, redirect=None, status=None)
DEBUG:urllib3.connectionpool:Starting new HTTPS connection (1): tachikoma.openai.azure.com:443
Starting new HTTPS connection (1): tachikoma.openai.azure.com:443
DEBUG:urllib3.connectionpool:https://tachikoma.openai.azure.com:443 "POST //openai/deployments/text-davinci-003/completions?api-version=2023-03-15-preview HTTP/1.1" 200 560
https://tachikoma.openai.azure.com:443 "POST //openai/deployments/text-davinci-003/completions?api-version=2023-03-15-preview HTTP/1.1" 200 560
message='OpenAI API response' path=https://tachikoma.openai.azure.com//openai/deployments/text-davinci-003/completions?api-version=2023-03-15-preview processing_ms=7888.0053 request_id=28b6cc20-c84e-49de-bfb3-dc7dd0de304b response_code=200
DEBUG:openai:message='OpenAI API response' path=https://tachikoma.openai.azure.com//openai/deployments/text-davinci-003/completions?api-version=2023-03-15-preview processing_ms=7888.0053 request_id=28b6cc20-c84e-49de-bfb3-dc7dd0de304b response_code=200
message='OpenAI API response' path=https://tachikoma.openai.azure.com//openai/deployments/text-davinci-003/completions?api-version=2023-03-15-preview processing_ms=7888.0053 request_id=28b6cc20-c84e-49de-bfb3-dc7dd0de304b response_code=200
body='{"id":"cmpl-799cu3hWWfK9lUXjOiShAXY7Ly9aG","object":"text_completion","created":1682416312,"model":"text-davinci-003","choices":[{"text":"\\nThought: Do I need to use a tool? No\\nAI: 你可以使用MySQL Backup工具来进行备份。你可以使用它来备份整个数据库或单个表，并可以将备份文件保存到本地或远程位置。你可以使用MySQL命令行客户端或图形界面客户端来操作MySQL Backup。","index":0,"finish_reason":"stop","logprobs":null}],"usage":{"completion_tokens":171,"prompt_tokens":423,"total_tokens":594}}\n' headers="{'Cache-Control': 'no-cache, must-revalidate', 'Content-Length': '560', 'Content-Type': 'application/json', 'Access-Control-Allow-Origin': '*', 'Aicon-Correlation-Id': 'eb98f6db-f776-4605-9403-506d1f0bd9f0', 'Strict-Transport-Security': 'max-age=31536000; includeSubDomains; preload', 'Openai-Model': 'text-davinci-003', 'apim-request-id': '63c0e02d-d425-47d3-9d73-a91cec30b490', 'Openai-Processing-Ms': '7888.0053', 'x-content-type-options': 'nosniff', 'X-Accel-Buffering': 'no', 'X-Request-Id': '28b6cc20-c84e-49de-bfb3-dc7dd0de304b', 'x-ms-region': 'East US', 'Date': 'Tue, 25 Apr 2023 09:52:00 GMT'}" message='API response body'
DEBUG:openai:body='{"id":"cmpl-799cu3hWWfK9lUXjOiShAXY7Ly9aG","object":"text_completion","created":1682416312,"model":"text-davinci-003","choices":[{"text":"\\nThought: Do I need to use a tool? No\\nAI: 你可以使用MySQL Backup工具来进行备份。你可以使用它来备份整个数据库或单个表，并可以将备份文件保存到本地或远程位置。你可以使用MySQL命令行客户端或图形界面客户端来操作MySQL Backup。","index":0,"finish_reason":"stop","logprobs":null}],"usage":{"completion_tokens":171,"prompt_tokens":423,"total_tokens":594}}\n' headers="{'Cache-Control': 'no-cache, must-revalidate', 'Content-Length': '560', 'Content-Type': 'application/json', 'Access-Control-Allow-Origin': '*', 'Aicon-Correlation-Id': 'eb98f6db-f776-4605-9403-506d1f0bd9f0', 'Strict-Transport-Security': 'max-age=31536000; includeSubDomains; preload', 'Openai-Model': 'text-davinci-003', 'apim-request-id': '63c0e02d-d425-47d3-9d73-a91cec30b490', 'Openai-Processing-Ms': '7888.0053', 'x-content-type-options': 'nosniff', 'X-Accel-Buffering': 'no', 'X-Request-Id': '28b6cc20-c84e-49de-bfb3-dc7dd0de304b', 'x-ms-region': 'East US', 'Date': 'Tue, 25 Apr 2023 09:52:00 GMT'}" message='API response body'
body='{"id":"cmpl-799cu3hWWfK9lUXjOiShAXY7Ly9aG","object":"text_completion","created":1682416312,"model":"text-davinci-003","choices":[{"text":"\\nThought: Do I need to use a tool? No\\nAI: 你可以使用MySQL Backup工具来进行备份。你可以使用它来备份整个数据库或单个表，并可以将备份文件保存到本地或远程位置。你可以使用MySQL命令行客户端或图形界面客户端来操作MySQL Backup。","index":0,"finish_reason":"stop","logprobs":null}],"usage":{"completion_tokens":171,"prompt_tokens":423,"total_tokens":594}}\n' headers="{'Cache-Control': 'no-cache, must-revalidate', 'Content-Length': '560', 'Content-Type': 'application/json', 'Access-Control-Allow-Origin': '*', 'Aicon-Correlation-Id': 'eb98f6db-f776-4605-9403-506d1f0bd9f0', 'Strict-Transport-Security': 'max-age=31536000; includeSubDomains; preload', 'Openai-Model': 'text-davinci-003', 'apim-request-id': '63c0e02d-d425-47d3-9d73-a91cec30b490', 'Openai-Processing-Ms': '7888.0053', 'x-content-type-options': 'nosniff', 'X-Accel-Buffering': 'no', 'X-Request-Id': '28b6cc20-c84e-49de-bfb3-dc7dd0de304b', 'x-ms-region': 'East US', 'Date': 'Tue, 25 Apr 2023 09:52:00 GMT'}" message='API response body'

Thought: Do I need to use a tool? No
AI: 你可以使用MySQL Backup工具来进行备份。你可以使用它来备份整个数据库或单个表，并可以将备份文件保存到本地或远程位置。你可以使用MySQL命令行客户端或图形界面客户端来操作MySQL Backup。

> Finished chain.
Agent: 你可以使用MySQL Backup工具来进行备份。你可以使用它来备份整个数据库或单个表，并可以将备份文件保存到本地或远程位置。你可以使用MySQL命令行客户端或图形界面客户端来操作MySQL Backup。
User:
```` 

分析错误情况的输出:

````
"Assistant is a large language model trained by OpenAI.

Assistant is designed to be able to assist with a wide range of tasks, from answering simple questions to providing in-depth explanations and discussions on a wide range of topics. As a language model, Assistant is able to generate human-like text based on the input it receives, allowing it to engage in natural-sounding conversations and provide responses that are coherent and relevant to the topic at hand.

Assistant is constantly learning and improving, and its capabilities are constantly evolving. It is able to process and understand large amounts of text, and can use this knowledge to provide accurate and informative responses to a wide range of questions. Additionally, Assistant is able to generate its own text based on the input it receives, allowing it to engage in discussions and provide explanations and descriptions on a wide range of topics.

Overall, Assistant is a powerful tool that can help with a wide range of tasks and provide valuable insights and information on a wide range of topics. Whether you need help with a specific question or just want to have a conversation about a particular topic, Assistant is here to assist.

TOOLS:
------

Assistant has access to the following tools:

> Vector Index: useful for when you want to answer queries about X

To use a tool, please use the following format:

```
Thought: Do I need to use a tool? Yes
Action: the action to take, should be one of [Vector Index]
Action Input: the input to the action
Observation: the result of the action
```

When you have a response to say to the Human, or if you do not need to use a tool, you MUST use the format:

```
Thought: Do I need to use a tool? No
AI: [your response here]
```

Begin!

Previous conversation history:

New input: \\u5982\\u4f55\\u7528mysqlbackup\\u8fdb\\u884c\\u5907\\u4efd
"
```` 

  - 其中给出了
    - 应当返回的输出格式
    - 可以使用的工具 (但工具的描述词是错误的, "Vector Index: useful for when you want to answer queries about X")
    - 对话历史
    - 当前你的问题

GPT返回信息: (直接从模型内提取了信息)

```
\\nThought: Do I need to use a tool? No
\\nAI: 你可以使用MySQL Backup工具来进行备份。你可以使用它来备份整个数据库或单个表，并可以将备份文件保存到本地或远程位置。你可以使用MySQL命令行客户端或图形界面客户端来操作MySQL Backup。
``` 

修正工具的描述后的输出:

[附件: log] 

分析: 

请求1: text-davinci-003/completions

````
Assistant is a large language model trained by OpenAI.

Assistant is designed to be able to assist with a wide range of tasks, from answering simple questions to providing in-depth explanations and discussions on a wide range of topics. As a language model, Assistant is able to generate human-like text based on the input it receives, allowing it to engage in natural-sounding conversations and provide responses that are coherent and relevant to the topic at hand.

Assistant is constantly learning and improving, and its capabilities are constantly evolving. It is able to process and understand large amounts of text, and can use this knowledge to provide accurate and informative responses to a wide range of questions. Additionally, Assistant is able to generate its own text based on the input it receives, allowing it to engage in discussions and provide explanations and descriptions on a wide range of topics.

Overall, Assistant is a powerful tool that can help with a wide range of tasks and provide valuable insights and information on a wide range of topics. Whether you need help with a specific question or just want to have a conversation about a particular topic, Assistant is here to assist.

TOOLS:
------

Assistant has access to the following tools:

> Vector Index: useful for when you want to answer queries about MySQL

To use a tool, please use the following format:

```
Thought: Do I need to use a tool? Yes
Action: the action to take, should be one of [Vector Index]
Action Input: the input to the action
Observation: the result of the action
```

When you have a response to say to the Human, or if you do not need to use a tool, you MUST use the format:

```
Thought: Do I need to use a tool? No
AI: [your response here]
```

Begin!

Previous conversation history:

New input:  \\u5728MySQL\\u4e2d, \\u5982\\u4f55\\u7528mysqlbackup\\u8fdb\\u884c\\u5355\\u6587\\u4ef6\\u5168\\u5907\\u6062\\u590d \\u7ed9\\u51fa\\u8be6\\u7ec6\\u6b65\\u9aa4\\u548c\\u547d\\u4ee4, \\u5bf9\\u547d\\u4ee4\\u7684\\u6bcf\\u4e2a\\u53c2\\u6570\\u505a\\u51fa\\u89e3\\u91ca

```` 

返回1: (此处应使用中文, 否则向量匹配不到中文文档)

```
Thought: Do I need to use a tool? Yes
Action: Vector Index
Action Input: How to do a single-file full recovery with mysqlbackup in MySQL
``` 

请求2: text-embedding-ada-002/embeddings

```
data='{"input": [[4438, 311, 656, 264, 3254, 14537, 2539, 13654, 449, 10787, 32471, 304, 27436]], "encoding_format": "base64"}'
``` 

返回2:

```
"embedding": "cg2yvFWTjzy77M888GHMvPCj0rwrRt88jKx8vBbUJL2xki+7gpRivG3YCD2xDiM9R+KjPFtMxTwDOK86z8pLPAjZmTyZE8k866gWPSqAzLouUwu8rbfXPP4Kn7zO/B
......
......
``` 

llama_index反查到chunk:

```
DEBUG:llama_index.indices.utils:> Top 3 nodes:
> [Node 47b99d1a-5370-4318-9c07-3a7e61aad061] [Similarity score:             0.847256] data files; *note 'mysqldump': mysqldump.
provides online logical backup.  This discussion uses *...
> [Node 57faf034-0375-47ec-8930-ec7888aa5249] [Similarity score:             0.824803] Backups*

If you can shut down the MySQL server, you can make a physical backup
that consists of ...
> [Node baee1f28-c859-4614-99bc-22e87ca57694] [Similarity score:             0.821314] when you need
to restore an old backup and replay the changes that happened since that
backup), i...
``` 

请求3: text-davinci-003/completions

```
Context information is below. 
---------------------
data files; *note \'mysqldump\': mysqldump.
provides online logical backup.  This discussion uses *note \'mysqldump\':
mysqldump.

Assume that we make a full backup of all our \'InnoDB\' tables in all
databases using the following command on Sunday at 1 p.m., when load is
low:

     $> mysqldump --all-databases --master-data --single-transaction > backup_sunday_1_PM.sql

     The resulting \'.sql\' file produced by *note \'mysqldump\': mysqldump.
     contains a set of SQL *note \'INSERT\': insert. statements that can be
     used to reload the dumped tables at a later time.

     This backup operation acquires a global read lock on all tables at the
     beginning of the dump (using \'FLUSH TABLES WITH READ LOCK\').  As soon as
     this lock has been acquired, the binary log coordinates are read and the
     lock is released.  If long updating statements are running when the
     *note \'FLUSH\': flush. statement is issued, the backup operation may
     stall until those statements finish.  After that, the dump becomes
     lock-free and does not disturb reads and writes on the tables.

It was assumed earlier that the tables to back up are \'InnoDB\' tables,
so \'--single-transaction\' uses a consistent read and guarantees that
data seen by *note \'mysqldump\': mysqldump. does not change.  (Changes
made by other clients to \'InnoDB\' tables are not seen by the *note
\'mysqldump\': mysqldump. process.)  If the backup operation includes
nontransactional tables, consistency requires that they do not change
during the backup.  For example, for the \'MyISAM\' tables in the \'mysql\'
database, there must be no administrative changes to MySQL accounts
during the backup.

Full backups are necessary, but it is not always convenient to create
them.  They produce large backup files and take time to generate.  They
are not optimal in the sense that each successive full backup includes
all data, even that part that has not changed since the previous full
backup.  It is more efficient to make an initial full backup, and then
to make incremental backups.  The incremental backups are smaller and
take less time to produce.  The tradeoff is that, at recovery time, you
cannot restore your data just by reloading the full backup.  You must
also process the incremental backups to recover the incremental changes.

To make incremental backups, we need to save the incremental changes.
In MySQL, these changes are represented in the binary log, so the MySQL
server should always be started with the \'--log-bin\' option to enable
that log.  With binary logging enabled, the server writes each data
change into a file while it updates data.  Looking at the data directory
of a MySQL server that has been running for some days, we find these
MySQL binary log files:

     -rw-rw---- 1 guilhem  guilhem   1277324 Nov 10 23:59 gbichot2-bin.000001
          -rw-rw---- 1 guilhem  guilhem         4 Nov 10 23:59 gbichot2-bin.000002
          -rw-rw---- 1 guilhem  guilhem        79 Nov 11 11:06 gbichot2-bin.000003
          -rw-rw---- 1 guilhem  guilhem       508 Nov 11 11:08 gbichot2-bin.000004
          -rw-rw---- 1 guilhem  guilhem 220047446 Nov 12 16:47 gbichot2-bin.000005
          -rw-rw---- 1 guilhem  guilhem    998412 Nov 14 10:08 gbichot2-bin.000006
       
---------------------
Given the context information and not prior knowledge, answer the question: How to do a single-file full recovery with mysqlbackup in MySQL
``` 

返回3: 

```
A single-file full recovery with mysqlbackup in MySQL can be accomplished by using the following command:

$> mysqlbackup --all-databases --master-data --single-transaction > backup_sunday_1_PM.sql

This command will create a single '.sql' file that contains a set of 'INSERT' statements that can be used to reload the dumped tables at a later time. This backup operation will acquire a global read lock on all tables at the beginning of the dump (using 'FLUSH TABLES WITH READ LOCK'). This lock will then be released once the binary log coordinates have been read.  The backup operation will also guarantee that data seen by *note 'mysqldump': mysqldump. does not change.
> Initial response:
A single-file full recovery with mysqlbackup in MySQL can be accomplished by using the following command:

$> mysqlbackup --all-databases --master-data --single-transaction > backup_sunday_1_PM.sql

This command will create a single '.sql' file that contains a set of 'INSERT' statements that can be used to reload the dumped tables at a later time. This backup operation will acquire a global read lock on all tables at the beginning of the dump (using 'FLUSH TABLES WITH READ LOCK'). This lock will then be released once the binary log coordinates have been read.  The backup operation will also guarantee that data seen by *note 'mysqldump': mysqldump. does not change.
``` 

请求4: text-davinci-003/completions

```
The original question is as follows: How to do a single-file full recovery with mysqlbackup in MySQL
We have provided an existing answer: 
A single-file full recovery with mysqlbackup in MySQL can be accomplished by using the following command: 

$> mysqlbackup --all-databases --master-data --single-transaction > backup_sunday_1_PM.sql

This command will create a single \'.sql\' file that contains a set of \'INSERT\' statements that can be used to reload the dumped tables at a later time. This backup operation will acquire a global read lock on all tables at the beginning of the dump (using \'FLUSH TABLES WITH READ LOCK\'). This lock will then be released once the binary log coordinates have been read.  The backup operation will also guarantee that data seen by *note \'mysqldump\': mysqldump. does not change.

We have the opportunity to refine the existing answer (only if needed) with some more context below.
------------
Backups*

If you can shut down the MySQL server, you can make a physical backup
that consists of all files used by \'InnoDB\' to manage its tables.  Use
the following procedure:

  1. Perform a slow shutdown of the MySQL server and make sure that it
       stops without errors.

  2. Copy all \'InnoDB\' data files (\'ibdata\' files and \'.ibd\' files) into
       a safe place.

  3. Copy all \'InnoDB\' redo log files (\'#ib_redoN\' files in MySQL 8.0.30
       and higher or \'ib_logfile\' files in earlier releases) to a safe
       place.

  4. Copy your \'my.cnf\' configuration file or files to a safe place.

  *Logical Backups Using mysqldump*

  In addition to physical backups, it is recommended that you regularly
  create logical backups by dumping your tables using *note \'mysqldump\':
  mysqldump.  A binary file might be corrupted without you noticing it.
  Dumped tables are stored into text files that are human-readable, so
  spotting table corruption becomes easier.  Also, because the format is
  simpler, the chance for serious data corruption is smaller.  *note
  \'mysqldump\': mysqldump. also has a \'--single-transaction\' option for
  making a consistent snapshot without locking out other clients.  See
  *note backup-policy::.

Replication works with *note \'InnoDB\': innodb-storage-engine. tables, so
you can use MySQL replication capabilities to keep a copy of your
database at database sites requiring high availability.  See *note
innodb-and-mysql-replication::.

\\u001f
File: manual.info.tmp,  Node: innodb-recovery,  Prev: innodb-backup,  Up: innodb-backup-recovery

15.18.2 InnoDB Recovery
-----------------------

This section describes \'InnoDB\' recovery.  Topics include:

   * *note innodb-recovery-point-in-time::

   * *note innodb-corruption-disk-failure-recovery::

   * *note innodb-crash-recovery::

   * *note innodb-recovery-tablespace-discovery::

   *Point-in-Time Recovery*

   To recover an \'InnoDB\' database to the present from the time at which
   the physical backup was made, you must run MySQL server with binary
   logging enabled, even before taking the backup.  To achieve
   point-in-time recovery after restoring a backup, you can apply changes
   from the binary log that occurred after the backup was made.  See *note
   point-in-time-recovery::.

*Recovery from Data Corruption or Disk Failure*

If your database becomes corrupted or disk failure occurs, you must
perform the recovery using a backup.  In the case of corruption, first
find a backup that is not corrupted.  After restoring the base backup,
do a point-in-time recovery from the binary log files using *note
\'mysqlbinlog\': mysqlbinlog. and *note \'mysql\': mysql. to restore the
changes that occurred after the backup was made.

In some cases of database corruption, it is enough to dump, drop, and
re-create one or a few corrupt tables.  You can use the *note \'CHECK
TABLE\': check-table. statement to check whether a table is corrupt,
although *note \'CHECK TABLE\': check-table. naturally cannot detect every
possible kind of corruption.

In some cases, apparent database page corruption is actually due to the
operating system corrupting its own file cache, and the data on disk may
be okay.  It is best to try restarting the computer first.  Doing so may
eliminate errors that appeared
------------
Given the new context, refine the original answer to better answer the question. If the context isn\'t useful, return the original answer.
``` 

基本结构: 

```
The original question is as follows:  ...
 
We have provided an existing answer: ...
 
We have the opportunity to refine the existing answer (only if needed) with some more context below.
...
...
 
Given the new context, refine the original answer to better answer the question. If the context isn\'t useful, return the original answer.
``` 

返回4: 

```
A single-file full recovery with mysqlbackup in MySQL can be accomplished by using the following command: 

$\\u003e mysqlbackup --all-databases --master-data --single-transaction \\u003e backup_sunday_1_PM.sql

This command will create a single \'.sql\' file that contains a set of \'INSERT\' statements that can be used to reload the dumped tables at a later time. This backup operation will acquire a global read lock on all tables at the beginning of the dump (using \'FLUSH TABLES WITH READ LOCK\'). This lock will then be released once the binary log coordinates have been read.  The backup operation will also guarantee that data seen by *note \'mysqldump\': mysqldump. does not change.

In addition to the single-file backup, it is recommended to also use a point-in-time recovery strategy. To do this, MySQL server must have binary logging enabled in order to restore changes that occurred after the backup was taken. After restoring the base backup, the changes can be restored from the binary log files using the mysqlbinlog utility and the mysql command. Additionally, it is important to keep regular logical backups by dumping
``` 

请求5+返回5: 再次进行refine

调整prompt, 使问答使用中文: 

**目前不能让其稳定地回答中文, 感觉是模型问题, 待更换gpt3.5模型测试 (另: 将prompt换成中文的比例越高, 回答越倾向于中文)**

````
# chatbot
# https://github.com/jerryjliu/llama_index/blob/c261a1751f85c6308adc31ef5ca1d8b45f168be7/docs/how_to/integrations/using_with_langchain.md

from llama_index.langchain_helpers.agents import IndexToolConfig, LlamaToolkit

index_config = IndexToolConfig(
    index=index, 
    name=f"MySQL documents",
    description=f"useful for when you want to answer queries about mysql and its tool",
    index_query_kwargs={"similarity_top_k": 3},
    tool_kwargs={"return_direct": True}
)

toolkit = LlamaToolkit(
    index_configs=[index_config],
    graph_configs=[]
)

from langchain.chains.conversation.memory import ConversationBufferMemory
memory = ConversationBufferMemory(memory_key="chat_history")

from llama_index.langchain_helpers.agents import create_llama_chat_agent

agent_chain = create_llama_chat_agent(
    toolkit,
    llm,
    memory=memory,
    verbose=True,
   agent_kwargs={
#https://huggingface.co/spaces/microsoft/visual_chatgpt/blob/main/app.py
        "format_instructions": """用户使用中文和你进行聊天，你必须回复中文。如果要调用工具，你必须遵循如下格式:

```
Thought: Do I need to use a tool? Yes
Action: 下一步动作, should be one of [{tool_names}]
Action Input: 该动作的输入
Observation: 该动作的输出
```

当你不再需要继续调用工具，而是对观察结果进行总结回复时，你必须遵循如下格式：

```
Thought: Do I need to use a tool? No
{ai_prefix}: [your response here]
```""",
        "prefix": """Assistant是由OpenAI训练的大型语言模型。

Assistant旨在能够协助各种任务，从回答简单问题到提供深入的解释和讨论各种主题。作为一个语言模型，Assistant能够根据接收到的输入生成类似人类的文本，使其能够进行自然的对话，并提供与所讨论的主题相关的连贯和相关的响应。

Assistant不断学习和改进，其能力不断发展。它能够处理和理解大量的文本，并可以利用这些知识为各种问题提供准确和信息丰富的回答。此外，Assistant还能根据接收到的输入生成自己的文本，使其能够在各种主题上进行讨论，并提供解释和描述。

总的来说，Assistant是一个强大的工具，可以帮助各种任务，并提供有价值的见解和信息。无论您需要针对特定问题寻求帮助，还是只是想就特定主题进行对话，Assistant都会在这里提供帮助。

Assistant必须回复中文

工具列表:
------

Assistant 可以使用这些工具:""",
        "suffix": """开始!

聊天历史:
{chat_history}

新输入: {input}
{agent_scratchpad}"""
    }
)

while True:
    text_input = input("User: ")
    response = agent_chain.run(input=text_input)
    print(f'Agent: {response}')
```` 

正确结果: 

```
Agent: 

Answer:
1. 进行全备，将备份保存成单个镜像文件：
mysqlbackup --user=root --password=password --host=localhost --socket=/data/mysqldata/mysql.sock --backup-image=/root/backup/my.mbi --backup-dir=/root/backup --compress --compress-method=lzma --compress-level=1 --backup-include-tables=MyISAM backup-to-image

2. 校验备份镜像文件：
mysqlbackup --backup-image=/root/backup/my.mbi validate

3. 停止数据库服务：
service mysql stop

4. 删除mysql的数据目录(datadir)、日志目录(binlog、redo
``` 

运行日志(正确结果): [log.2](/assets/01KJBYYMHN0C5CA4ARZ21B6AAN/log.2)

错误结果: 

```
MySQL Enterprise Backup可以备份到单一文件，并可以选择压缩备份，这样可以实现单文件全备恢复。恢复过程首先需要创建一个MySQL用户，并授予这个用户相应的权限，然后将这个单一文件解压缩到数据目录，应用redo log，将备份的数据文件恢复到指定的数据和日志目录中，改变mysql的数据目录、日
``` 

运行日志(错误结果): [log.3](/assets/01KJBYYMHN0C5CA4ARZ21B6AAN/log.3) 

正确结果和错误结果间的差距在于: 第一个请求, gpt返回的第一个查询: 

  - 正确结果: Action Input: 如何用mysqlbackup进行单文件全备恢复,给出详细步骤和命令
  - 错误结果: Action Input: 如何用mysqlbackup进行单文件全备恢复

# 增加一个问答UI

使用gradio库: 

```
if launch_ui:
    import gradio as gr
    import random
    import time

    with gr.Blocks() as demo:
        chatbot = gr.Chatbot()
        msg = gr.Textbox()
        clear = gr.Button("Clear")

        def respond(message, chat_history):
            response = agent_chain.run(input=message)
            chat_history.append((message, response))
            return "", chat_history

        msg.submit(respond, [msg, chatbot], [msg, chatbot])
        clear.click(lambda: None, None, chatbot, queue=False)

    demo.launch(share=True)
``` 

# TODO

1 complete 搞清楚 Total LLM token usage 和 Total embedding token usage 的含义 2 complete 分析/处理warning 3 complete llama_index 如何增量加入内容 4 complete 如何确认知识是 从本地出去的, 而非模型内嵌的知识 10 incomplete 使用1024 chunk_size在中文会报错 11 incomplete 输出被截断, 需要能连续输出 5 incomplete 内嵌知识的准确度? 9 complete 需要一个问答UI 7 incomplete 处理chat的上下文 8 complete 中英文支持 12 incomplete 加载PDF: meb的文档 13 incomplete 使用LLM将提问拆分成更有意义的问题? 14 incomplete 连续对话时, 后续的问题, 不会调用本地的知识库来完成: [log.4](/assets/01KJBYYMHN0C5CA4ARZ21B6AAN/log.4) 15 incomplete 扩大文档库后, 进行fine tuning, 尝试效果是否更好 16 incomplete 如何处理图片 (文本类图片OCR, 表意类图片的引用) 17 complete 如何关联原始引用
