---
title: 20230409 - llama-index 逻辑整理
confluence_page_id: 2130604
created_at: 2023-04-08T16:00:41+00:00
updated_at: 2023-04-13T10:33:46+00:00
---

# 逻辑

  - BaseGPTIndex.from_documents: 从document生成index
    - node_parser.get_nodes_from_documents: 
      - get_nodes_from_documents
        - get_nodes_from_document: 从document生成node  

          - get_text_splits_from_document: 将document切割成文字片段 (text), 再拼接成chunk (chunk 由 token/词 构成)
          - 每个chunk生成一个node
    - 构造 BaseGPTIndex, 传入 nodes
      - build_index_from_nodes: 
        - 将nodes传入docstore, 作为原始文档
        - _build_index_from_nodes: 构造index
          - 以GPTVectorStoreIndex._build_index_from_nodes为例
            - _get_node_embedding_results(nodes)
              - 对每一个node:  

                - _service_context.embed_model.queue_text_for_embeddding(node.get_text(), ...)
                - _service_context.embed_model.get_queued_text_embeddings()
                  - 调用到具体的embed_model, 例如OpenAIEmbedding
            - 将embedding的结果加入index
  - BaseGPTIndex.query
    - _preprocess_query
      - 以GPTListIndex._preprocess_query为例: 设置 text_qa_template 的默认值: 

```
DEFAULT_TEXT_QA_PROMPT_TMPL = (
    "Context information is below. \n"
    "---------------------\n"
    "{context_str}"
    "\n---------------------\n"
    "Given the context information and not prior knowledge, "
    "answer the question: {query_str}\n"
)
DEFAULT_TEXT_QA_PROMPT = QuestionAnswerPrompt(DEFAULT_TEXT_QA_PROMPT_TMPL)
```

    - QueryRunner.query (docstore在QueryRunner中)
      - _prepare_query_objects 选择合适的 query_combiner (处理单个query或者多个query)
      - query_combiner.run: 以SingleQueryCombiner为例
        - _prepare_update: 生成query_bundle (代表一个query或多个query)
          - updated_query_bundle = _query_transform(...), 以 DecomposeQueryTransform 为例
            - 调用 _llm_predictor.predict(_decompose_query_prompt, ...), 将query拆解成 新的query, 使得结果更好
        - _query_runner.query_transformed (updated_query_bundle)
          - _get_query_obj: 获取对应的query_obj (index上执行query的执行器), 对应列表如下: 

```
INDEX_STRUCT_TYPE_TO_INDEX_CLASS: Dict[IndexStructType, Type[BaseGPTIndex]] = {
    IndexStructType.TREE: GPTTreeIndex,
    IndexStructType.LIST: GPTListIndex,
    IndexStructType.KEYWORD_TABLE: GPTKeywordTableIndex,
    IndexStructType.SIMPLE_DICT: GPTSimpleVectorIndex,
    IndexStructType.DICT: GPTFaissIndex,
    IndexStructType.WEAVIATE: GPTWeaviateIndex,
    IndexStructType.PINECONE: GPTPineconeIndex,
    IndexStructType.QDRANT: GPTQdrantIndex,
    IndexStructType.MILVUS: GPTMilvusIndex,
    IndexStructType.CHROMA: GPTChromaIndex,
    IndexStructType.VECTOR_STORE: GPTVectorStoreIndex,
    IndexStructType.SQL: GPTSQLStructStoreIndex,
    IndexStructType.PANDAS: GPTPandasIndex,
    IndexStructType.KG: GPTKnowledgeGraphIndex,
    IndexStructType.EMPTY: GPTEmptyIndex,
    IndexStructType.CHATGPT_RETRIEVAL_PLUGIN: ChatGPTRetrievalPluginIndex,
    IndexStructType.OPENSEARCH: GPTOpensearchIndex,
}

INDEX_STRUT_TYPE_TO_QUERY_MAP: Dict[IndexStructType, QueryMap] = {
    index_type: index_cls.get_query_map()
    for index_type, index_cls in INDEX_STRUCT_TYPE_TO_INDEX_CLASS.items()
}
```

          - query_obj.query(query_bundle): 以GPTVectorStoreIndex.get_query_map = GPTVectorStoreIndexQuery为例: 
            - nodes = self.retrieve(query_bundle): 根据query,   

              - 如有需要, 生成query的embedding

```
self._service_context.embed_model.get_agg_embedding_from_queries(
        query_bundle.embedding_strs
)
```

              - query_result = _vector_store.query(query_str, ...)
                - 以SimpleVectorStore.query为例
                  - query通过 query_embedding 进行 top k 查询
              - query_result.nodes = _docstore.get_nodes (根据查询的text结果, 还原到原始doc/图像等)
              - retrieve最终返回的就是nodes (原始文档的切片)
            - self.synthesize(query_bundle, nodes)
              - _prepare_response_builder
                - 将上一步的nodes中的文字抽出并整理 (_get_text_from_node), 形成text_chunk数组 (文档切片)
              - _give_response_for_nodes
                - response_builder.get_response, 以get_response_over_chunks为例
                  - 逐个文档切片/text_chunk处理. (第一个chunk, 调用give_response_single, 输出response, 后面的chunk, 根据上一个response进行refine)
                    - response = give_response_single(query_str, text_chunk)  

                      - text_qa_template.partial_format(query_str): 对请求进行格式化
                      - split_text(text_chunk): 切割 文档切片 -> 文档碎片
                      - 逐一对文档碎片进行: (第一个chunk, 调用生成, 后面的chunk, 根据上一个结果 进行 refine)
                        - llm_predicator.predict(text_qa_template, context_str = text_chunk), 调用模型进行predict
              - _prepare_response_output

# PromptHelper参数整理

  - 参数名: max_input_size, num_output (代码位置: LLMMetadata._get_llm_metadata)
    - 对于OpenAI模型: (<https://python.langchain.com/en/latest/_modules/langchain/llms/openai.html#OpenAI>)
      - max_input_size
        - text-davinci-003: 4,097 tokens
      - num_output = max_tokens = 256
    - 其他模型: ChatOpenAI/Cohere/AI21: ...
    - 其他模型使用默认值: 
      - max_input_size = 3900
      - num_output = 256
  - 参数名: max_chunk_overlap
    - 使用方: get_text_splitter_given_prompt, 生成TokenTextSplitter(chunk_overlap = max_chunk_overlap/num_chunks)
  - 参数名: embedding_limit
    - 使用方: get_chunk_size_given_prompt, 用于TokenTextSplitter(chunk_size: chunk的大小)
      - 返回每个chunk的大小 = (max_input_size - num_output - prompt的token数) / chunk的个数
      - 最大值限制为 embedding_limit 或 chunk_size_limit
  - 参数名: chunk_size_limit

# 逻辑整理

  - 从输入的文档 构建 nodes
    - 将文档切割成文字片段, 再拼接成chunk, 每个chunk生成一个node
    - 简单理解: 一个node对应一个文档片段
  - 构建索引 index
    - 将nodes 放入 index的docstore (原始文档库)
    - 使用embed_model, 对node进行嵌入化, 将结果加入索引中
  - 处理查询请求 query
    - 可以使用 预测模型(llm_predictor.predict) + 拆解query的提示语, 将query拆解成新的query, 使得结果更好
    - 在索引中, 根据query查询文档片段
      - 生成query的嵌入值
      - 在index中, 查询query的嵌入值, 获得top k
      - 获取top k对应的文档片段
    - 用 query + 文档片段, 进行答案合成
      - 注意对文档碎片进行合成
        - 第一个文档碎片 + query, 套用提示语, 生成第一个结果
        - 第一个结果 + 第二个文档碎片 + query, 套用refine提示语, 生成第二个结果
        - ...
        - 直到生成最终答案

# 遇到问题1

输出中带有乱码字符: 出现在头部 ("TEST:" 是手工打印的行标志)

```
TEST:  ⁇  Context information is below. 
---------------------
if the buffer pool is initialized with a size of '2GB'
     (2147483648 bytes), '4' buffer pool instances, and a chunk size of
     '1GB' (1073741824 bytes), chunk size is truncated to a value equal
     to 'innodb_buffer_pool_size' / 'innodb_buffer_pool_instances', as
     shown below:

          $> mysqld --innodb-buffer-pool-size=2147483648 --innodb-buffer-pool-instances=4
          --innodb-buffer-pool-chunk-size=1073741824;

          mysql> SELECT @@innodb_buffer_pool_size;
          +---------------------------+
          | @@innodb_buffer_pool_size |
          +---------------------------+
          |                2147483648 |
          +---------------------------+

          mysql> SELECT @@innodb_buffer_pool_instances;
          +--------------------------------+
          | @@innodb_buffer_pool_instances |
          +--------------------------------+
          |                              4 |
          +--------------------------------+

          # Chunk size was set to 1GB (1073741824 bytes) on startup but was
          # truncated to innodb_buffer_pool_size / innodb_buffer_pool_instances

          mysql> SELECT @@innodb_buffer_pool_chunk_size;
          +---------------------------------+
          | @@innodb_buffer_pool_chunk_size |
          +---------------------------------+
          |                       536870912 |
          +---------------------------------+
``` 

调整了一下LLaMALLM代码, 将generation_output.sequences输出的tensor第一位0去掉: 

```
class LLaMALLM(LLM):
    def _call(self, prompt, stop=None):
        prompt += "### Response:"

        inputs = tokenizer(prompt, return_tensors="pt")
        input_ids = inputs["input_ids"].cuda()
        
        generation_config = GenerationConfig(
            temperature=0.6,
            top_p=0.95,
            repetition_penalty=1.15,
        )
        with torch.no_grad():
            generation_output = model.generate(
                input_ids=input_ids,
                generation_config=generation_config,
                return_dict_in_generate=True,
                output_scores=True,
                max_new_tokens=128,
            )
        response = ""
        for s in generation_output.sequences:
            # s is a tensor with header "0". remove the "0"
            s = s[1:]
            response += tokenizer.decode(s)
        
        response = response[len(prompt):]
        # print("Model Response:", response)
        return response
```
