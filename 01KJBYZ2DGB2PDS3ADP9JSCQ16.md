---
title: 20230503 - 测试方法"Hypothetical Document Embeddings" (HyDE)
confluence_page_id: 2130840
created_at: 2023-05-02T16:55:55+00:00
updated_at: 2023-05-03T07:32:03+00:00
---

问题: 如何动态地调整MySQL的buffer pool大小, 列出命令和步骤

目前llama_index的输出: 

```
可以使用以下步骤动态地调整MySQL的buffer pool大小: 
1. 使用SET GLOBAL语句，设定innodb_buffer_pool_size参数，将buffer pool大小设定为50-75%的系统内存，并可以在启动时设置或者动态调整；
2. 使用innodb_buffer_pool_resize参数，重新调整buffer pool大小，可以以1MB为单位增加或减少；
3. 改变innodb_buffer_pool_chunk_size参数时，需要计算其对innodb_buffer_pool_size的影响以确保得到的buffer pool size是可接受的，并且chunk数量（innodb_buffer_pool_size/innodb_buffer_pool_chunk_size）不要超过1000；
4. innodb_buffer_pool_size可以动态调整，但是需要满足innodb_buffer_pool_chunk_size * innodb_buffer_pool_instances的条件，并且修改这些参数需要重启MySQL服务器；
5. 启用innodb_buffer_pool_debug参数可以允许缓冲池小于1 GB时使用多个buffer pool实例；
6. 启用innodb_buffer_pool_dump_at_shutdown参数可以记录MySQL服务器关闭时缓存在InnoDB buffer pool中的页面，以缩短下次启动时的热身过程；
7. 启用innodb_buffer_pool_dump_now参数允许立即将缓存在InnoDB buffer pool中的页面记录下来；
8. 有关更多信息，请参阅*note innodb-buffer-pool-resize::和*note innodb-preload-buffer-pool::。
``` 

使用chatgpt, 生成形式答案: 

````
我想问模型一个问题: 如何动态地调整MySQL的buffer pool大小, 列出命令和步骤

请模型给出一个答案, 这个答案的形式应具体且详细, 但答案不用正确, 这个答案的形式将用于在私有文档库中搜索类似的信息, 以构成正确的答案
 
--- 答案如下: 
 
要动态地调整MySQL的缓冲池（buffer pool）大小，您可以使用以下命令和步骤：

1. 打开MySQL的命令行界面。

2. 运行以下命令来查看当前缓冲池的大小和使用情况：

```
SHOW VARIABLES LIKE 'innodb_buffer_pool_size';
```

3. 根据需要计算新的缓冲池大小。通常，缓冲池的大小应设置为可用内存的75%到80%左右。

4. 运行以下命令来设置新的缓冲池大小（例如，将缓冲池大小设置为4GB）：

```
SET GLOBAL innodb_buffer_pool_size = 4G;
```

5. 运行以下命令来再次验证缓冲池大小是否已更改：

```
SHOW VARIABLES LIKE 'innodb_buffer_pool_size';
```

6. 如果您希望MySQL在启动时自动使用新的缓冲池大小，请将以下行添加到MySQL的配置文件（通常为/etc/my.cnf或/etc/mysql/my.cnf）：

```
innodb_buffer_pool_size=4G
```

7. 重新启动MySQL以使更改生效。

需要注意的是，更改缓冲池大小可能会对MySQL的性能产生影响。如果您不确定如何正确地调整缓冲池大小，请参考MySQL的官方文档或咨询专业人士的建议。
```` 

# 测试langchain中的HyDE

<https://python.langchain.com/en/latest/modules/chains/index_examples/hyde.html>

langchain中使用HyDE的提示词: 

```
Please write a passage to answer the question 
Question: {QUESTION}
Passage:
``` 

变更代码: 

```
if use_hyde:
    from langchain.chains import LLMChain, HypotheticalDocumentEmbedder
    query_embeddings = HypotheticalDocumentEmbedder.from_llm(llm, mcontriever_embeddings, "web_search")
    query_service_context = ServiceContext.from_defaults(llm_predictor=llm_predictor, embed_model=LangchainEmbedding(query_embeddings), chunk_size_limit=chunk_size_limit)
 
if run_query_test:
    response = index.query("如何动态地调整MySQL的buffer pool大小, 列出命令和步骤", 
                           similarity_top_k=2, text_qa_template=text_qa_template, refine_template=refine_template,
                          service_context = query_service_context)
    print(response)
``` 

日志: 

```
# HyDE的请求: 要求write a passage, API给出回复
 
 
message='Request to OpenAI API' method=post path=https://tachikoma.openai.azure.com//openai/deployments/text-davinci-003/completions?api-version=2023-03-15-preview
api_version=2023-03-15-preview data='{"prompt": ["Please write a passage to answer the question \\nQuestion: \\u5982\\u4f55\\u52a8\\u6001\\u5730\\u8c03\\u6574MySQL\\u7684buffer pool\\u5927\\u5c0f, \\u5217\\u51fa\\u547d\\u4ee4\\u548c\\u6b65\\u9aa4\\nPassage:"], "temperature": 0.7, "max_tokens": 1000, "top_p": 1, "frequency_penalty": 0, "presence_penalty": 0, "n": 1, "best_of": 1, "logit_bias": {}}' message='Post details'
message='OpenAI API response' path=https://tachikoma.openai.azure.com//openai/deployments/text-davinci-003/completions?api-version=2023-03-15-preview processing_ms=8817.3543 request_id=ef92ae2d-47b0-4a16-b5a5-b74eb8e8722b response_code=200
body='{"id":"cmpl-7ByWXhLBU1CQsfAkunellOGCQ8rGU","object":"text_completion","created":1683088617,"model":"text-davinci-003","choices":[{"text":"\\n\\nMySQL的buffer pool大小可以通过执行以下步骤动态调整：\\n\\n1. 使用SET GLOBAL命令设置innodb_buffer_pool_size参数：SET GLOBAL innodb_buffer_pool_size=新的大小;\\n\\n2. 如果需要，使用SET GLOBAL命令调整innodb_buffer_pool_instances参数：SET GLOBAL innodb_buffer_pool_instances=新的instances数量;\\n\\n3. 使用FLUSH TABLES WITH READ LOCK命令将所有表锁定：FLUSH TABLES WITH READ LOCK;\\n\\n4. 使用SHOW VARIABLES LIKE命令检查innodb_buffer_pool_size参数是否已更新：SHOW VARIABLES LIKE \'innodb_buffer_pool_size\';\\n\\n5. 如果innodb_buffer_pool_size参数更新了，就使用UNLOCK TABLES命令解锁所有表：UNLOCK TABLES;\\n\\n6. 重新启动MySQL服务器，以使新的innodb_buffer_pool_size参数生效：sudo service mysql restart。","index":0,"finish_reason":"stop","logprobs":null}],"usage":{"completion_tokens":418,"prompt_tokens":58,"total_tokens":476}}\n' headers="{'Cache-Control': 'no-cache, must-revalidate', 'Content-Length': '1063', 'Content-Type': 'application/json', 'Access-Control-Allow-Origin': '*', 'Aicon-Correlation-Id': '72c6b456-90a9-4a7b-abca-59f9a6ed46a2', 'Strict-Transport-Security': 'max-age=31536000; includeSubDomains; preload', 'Openai-Model': 'text-davinci-003', 'apim-request-id': '9d450681-9fa4-4dce-84ca-2fdd4441d19a', 'Openai-Processing-Ms': '8817.3543', 'x-content-type-options': 'nosniff', 'X-Accel-Buffering': 'no', 'X-Request-Id': 'ef92ae2d-47b0-4a16-b5a5-b74eb8e8722b', 'x-ms-region': 'East US', 'Date': 'Wed, 03 May 2023 04:37:05 GMT'}" message='API response body'

# 整理出抽取的文档node

message='Request to OpenAI API' method=post path=https://tachikoma.openai.azure.com//openai/deployments/text-davinci-003/completions?api-version=2023-03-15-preview
api_version=2023-03-15-preview data='{"prompt": ["\\u4e0a\\u4e0b\\u6587\\u4fe1\\u606f\\u5982\\u4e0b: \\n---------------------\\n* innodb_buffer_pool_instances\\n\\n          mysql> SELECT @@innodb_buffer_pool_size;\\n          +---------------------------+\\n          | @@innodb_buffer_pool_size |\\n          +---------------------------+\\n          |                4286578688 |\\n          +---------------------------+\\n\\n     Care should be taken when changing \'innodb_buffer_pool_chunk_size\',\\n     as changing this value can increase the size of the buffer pool, as\\n     shown in the examples above.  Before you change\\n     \'innodb_buffer_pool_chunk_size\', calculate the effect on\\n     \'innodb_buffer_pool_size\' to ensure that the resulting buffer pool\\n     size is acceptable.\\n\\n*Note*:\\n\\nTo avoid potential performance issues, the number of chunks\\n(\'innodb_buffer_pool_size\' / \'innodb_buffer_pool_chunk_size\') should not\\nexceed 1000.\\n\\n*Configuring InnoDB Buffer Pool Size Online*\\n\\nThe \'innodb_buffer_pool_size\' configuration option can be set\\ndynamically using a *note \'SET\': set. statement, allowing you to resize\\nthe buffer pool without restarting the server.  For example:\\n\\n     mysql> SET GLOBAL innodb_buffer_pool_size=402653184;\\n\\n*Note*:\\n\\nThe buffer pool size must be equal to or a multiple of\\n\'innodb_buffer_pool_chunk_size\' * \'innodb_buffer_pool_instances\'.\\nChanging those variable settings requires restarting the server.\\n\\nActive transactions and operations performed through \'InnoDB\' APIs\\nshould be completed before resizing the buffer pool.  When initiating a\\nresizing operation, the operation does not start until all active\\ntransactions are completed.  Once the resizing operation is in progress,\\nnew transactions and operations that require access to the buffer pool\\nmust wait until the resizing operation finishes.  The exception to the\\nrule is that concurrent access to the buffer pool is permitted while the\\nbuffer pool is defragmented and pages are withdrawn when buffer pool\\nsize is decreased.  A drawback of allowing concurrent access is that it\\ncould result in a temporary shortage of available pages while pages are\\nbeing withdrawn.\\n\\n*Note*:\\n\\nNested transactions could fail if initiated after the buffer pool\\nresizing operation begins.\\n\\n*Monitoring Online Buffer Pool Resizing Progress*\\n\\nThe \'Innodb_buffer_pool_resize_status\' variable reports a string value\\nindicating buffer pool resizing progress; for example:\\n\\n     mysql> SHOW STATUS WHERE Variable_name=\'InnoDB_buffer_pool_resize_status\';\\n     +----------------------------------+----------------------------------+\\n     | Variable_name                    | Value                            |\\n     +----------------------------------+----------------------------------+\\n     | Innodb_buffer_pool_resize_status | Resizing also other hash tables. |\\n     +----------------------------------+----------------------------------+\\n\\nFrom MyQL 8.0.31, you can also monitor an online buffer pool resizing\\noperation using the \'Innodb_buffer_pool_resize_status_code\' and\\n\'Innodb_buffer_pool_resize_status_progress\' status variables, which\\nreport numeric values, preferable for programmatic monitoring.\\n\\nThe \'Innodb_buffer_pool_resize_status_code\' status variable reports a\\nstatus code indicating the stage of an online buffer pool resizing\\noperation.  Status codes include:\\n\\n   * 0: No Resize operation in progress\\n\\n   * 1: Starting Resize\\n\\n   * 2: Disabling AHI (Adaptive Hash Index)\\n\\n   * 3: Withdrawing Blocks\\n\\n   * 4: Acquiring Global Lock\\n\\n   * 5: Resizing Pool\\n\\n   * 6: Resizing Hash\\n\\n   * 7: Resizing Failed\\n\\nThe \'Innodb_buffer_pool_resize_status_progress\' status variable reports\\na percentage value indicating the progress of each stage.  The\\npercentage value is updated after each buffer pool instance is\\nprocessed.  As the status (reported by\\n\'Innodb_buffer_pool_resize_status_code\') changes from one status to\\nanother, the percentage value is reset to 0.\\n\\nThe following query returns a string value indicating the buffer pool\\nresizing progress, a code indicating the current stage of the operation,\\nand the current progress of that stage, expressed as a percentage value:\\n\\n     SELECT variable_name, variable_value\\n      FROM performance_schema.global_status\\n      WHERE LOWER(variable_name) LIKE \\"innodb_buffer_pool_resize%\\";\\n\\nBuffer pool resizing progress is also visible in the server error log.\\nThis example shows notes that are logged when increasing the size of the\\nbuffer pool:\\n\\n     [Note] InnoDB: Resizing buffer pool from 134217728 to 4294967296. (unit=134217728)\\n     [Note] InnoDB: disabled adaptive hash index.\\n     [Note] InnoDB: buffer pool 0 : 31 chunks (253952 blocks) was added.\\n     [Note] InnoDB: buffer pool 0 : hash tables were resized.\\n     [Note] InnoDB: Resized hash tables at lock_sys, adaptive hash index, dictionary.\\n     [Note] InnoDB: completed to resize buffer pool from 134217728 to[document=external]\\n---------------------\\n\\u4f7f\\u7528\\u4ee5\\u4e0a\\u4fe1\\u606f, \\u56de\\u7b54\\u5982\\u4e0b\\u95ee\\u9898: \\u5982\\u4f55\\u52a8\\u6001\\u5730\\u8c03\\u6574MySQL\\u7684buffer pool\\u5927\\u5c0f, \\u5217\\u51fa\\u547d\\u4ee4\\u548c\\u6b65\\u9aa4\\n\\u5e76\\u65b0\\u589e\\u4e00\\u4e2a\\u6bb5\\u843d, \\u8be6\\u7ec6\\u5217\\u4e3e\\u7b54\\u6848\\u7684\\u6bcf\\u4e2a\\u90e8\\u5206\\u6765\\u81ea\\u4e8e\\u4e0a\\u4e0b\\u6587\\u7684\\u54ea\\u4e00\\u90e8\\u5206\\u6216\\u8005\\u4e0d\\u6765\\u81ea\\u4e8e\\u4e0a\\u4e0b\\u6587\\n"], "temperature": 0.7, "max_tokens": 1000, "top_p": 1, "frequency_penalty": 0, "presence_penalty": 0, "n": 1, "best_of": 1, "logit_bias": {}}' message='Post details'
message='OpenAI API response' path=https://tachikoma.openai.azure.com//openai/deployments/text-davinci-003/completions?api-version=2023-03-15-preview processing_ms=12125.6953 request_id=bcd1af88-1da6-4945-bf3b-f413ae8dceb0 response_code=200
body='{"id":"cmpl-7ByWhI9ZKJ4LKPHIYFz59ELtj7YPQ","object":"text_completion","created":1683088627,"model":"text-davinci-003","choices":[{"text":"\\n答: 调整MySQL的buffer pool大小的命令和步骤如下:\\n\\n1. 计算改变innodb_buffer_pool_chunk_size后buffer pool的大小, 确保改变后的buffer pool大小合理: \\n   使用`SELECT @@innodb_buffer_pool_size;`命令查询当前的buffer pool大小。\\n\\n2. 使用`SET GLOBAL innodb_buffer_pool_size=\\u003cnew_size\\u003e;`命令动态调整buffer pool大小, 其中\\u003cnew_size\\u003e为新的buffer pool大小。\\n\\n3. 确保buffer pool大小等于或者是innodb_buffer_pool_chunk_size * innodb_buffer_pool_instances的倍数。\\n\\n4. 执行完步骤1-3后, 使用`SHOW STATUS WHERE Variable_name=\'InnoDB_buffer_pool_resize_status\';`查询buffer pool调整状态, 使用`SELECT variable_name, variable_value FROM performance_schema.global_status WHERE LOWER(variable_name) LIKE \\"innodb_buffer_pool_resize%\\";`查询buffer pool调整进度。\\n\\n*答案的每个部分来自于上下文的哪一部分*:\\n\\n步骤1来自于上下文中的“*Configuring InnoDB Buffer Pool Size Online*”部分, 步骤2来自于上下文中的“*Configuring InnoDB Buffer Pool Size Online*”部分, 步骤3来自于上下文中的“*Note*”部分, 步骤4来自于上下文中的“*Monitoring Online Buffer Pool Resizing Progress*”部分。","index":0,"finish_reason":"stop","logprobs":null}],"usage":{"completion_tokens":583,"prompt_tokens":1383,"total_tokens":1966}}\n' headers="{'Cache-Control': 'no-cache, must-revalidate', 'Content-Length': '1516', 'Content-Type': 'application/json', 'Access-Control-Allow-Origin': '*', 'Aicon-Correlation-Id': '68120327-12a7-4057-a32b-f76d3b21f952', 'Strict-Transport-Security': 'max-age=31536000; includeSubDomains; preload', 'Openai-Model': 'text-davinci-003', 'apim-request-id': '1b4bc405-8ef0-4404-b99c-10af93255ece', 'Openai-Processing-Ms': '12125.6953', 'x-content-type-options': 'nosniff', 'X-Accel-Buffering': 'no', 'X-Request-Id': 'bcd1af88-1da6-4945-bf3b-f413ae8dceb0', 'x-ms-region': 'East US', 'Date': 'Wed, 03 May 2023 04:37:18 GMT'}" message='API response body'
message='Request to OpenAI API' method=post path=https://tachikoma.openai.azure.com//openai/deployments/text-davinci-003/completions?api-version=2023-03-15-preview
api_version=2023-03-15-preview data='{"prompt": ["\\u6211\\u4eec\\u8981\\u56de\\u7b54\\u8fd9\\u4e2a\\u95ee\\u9898: \\u5982\\u4f55\\u52a8\\u6001\\u5730\\u8c03\\u6574MySQL\\u7684buffer pool\\u5927\\u5c0f, \\u5217\\u51fa\\u547d\\u4ee4\\u548c\\u6b65\\u9aa4\\n\\u76ee\\u524d\\u7684\\u7b54\\u6848\\u662f: \\n\\u7b54: \\u8c03\\u6574MySQL\\u7684buffer pool\\u5927\\u5c0f\\u7684\\u547d\\u4ee4\\u548c\\u6b65\\u9aa4\\u5982\\u4e0b:\\n\\n1. \\u8ba1\\u7b97\\u6539\\u53d8innodb_buffer_pool_chunk_size\\u540ebuffer pool\\u7684\\u5927\\u5c0f, \\u786e\\u4fdd\\u6539\\u53d8\\u540e\\u7684buffer pool\\u5927\\u5c0f\\u5408\\u7406: \\n   \\u4f7f\\u7528`SELECT @@innodb_buffer_pool_size;`\\u547d\\u4ee4\\u67e5\\u8be2\\u5f53\\u524d\\u7684buffer pool\\u5927\\u5c0f\\u3002\\n\\n2. \\u4f7f\\u7528`SET GLOBAL innodb_buffer_pool_size=<new_size>;`\\u547d\\u4ee4\\u52a8\\u6001\\u8c03\\u6574buffer pool\\u5927\\u5c0f, \\u5176\\u4e2d<new_size>\\u4e3a\\u65b0\\u7684buffer pool\\u5927\\u5c0f\\u3002\\n\\n3. \\u786e\\u4fddbuffer pool\\u5927\\u5c0f\\u7b49\\u4e8e\\u6216\\u8005\\u662finnodb_buffer_pool_chunk_size * innodb_buffer_pool_instances\\u7684\\u500d\\u6570\\u3002\\n\\n4. \\u6267\\u884c\\u5b8c\\u6b65\\u9aa41-3\\u540e, \\u4f7f\\u7528`SHOW STATUS WHERE Variable_name=\'InnoDB_buffer_pool_resize_status\';`\\u67e5\\u8be2buffer pool\\u8c03\\u6574\\u72b6\\u6001, \\u4f7f\\u7528`SELECT variable_name, variable_value FROM performance_schema.global_status WHERE LOWER(variable_name) LIKE \\"innodb_buffer_pool_resize%\\";`\\u67e5\\u8be2buffer pool\\u8c03\\u6574\\u8fdb\\u5ea6\\u3002\\n\\n*\\u7b54\\u6848\\u7684\\u6bcf\\u4e2a\\u90e8\\u5206\\u6765\\u81ea\\u4e8e\\u4e0a\\u4e0b\\u6587\\u7684\\u54ea\\u4e00\\u90e8\\u5206*:\\n\\n\\u6b65\\u9aa41\\u6765\\u81ea\\u4e8e\\u4e0a\\u4e0b\\u6587\\u4e2d\\u7684\\u201c*Configuring InnoDB Buffer Pool Size Online*\\u201d\\u90e8\\u5206, \\u6b65\\u9aa42\\u6765\\u81ea\\u4e8e\\u4e0a\\u4e0b\\u6587\\u4e2d\\u7684\\u201c*Configuring InnoDB Buffer Pool Size Online*\\u201d\\u90e8\\u5206, \\u6b65\\u9aa43\\u6765\\u81ea\\u4e8e\\u4e0a\\u4e0b\\u6587\\u4e2d\\u7684\\u201c*Note*\\u201d\\u90e8\\u5206, \\u6b65\\u9aa44\\u6765\\u81ea\\u4e8e\\u4e0a\\u4e0b\\u6587\\u4e2d\\u7684\\u201c*Monitoring Online Buffer Pool Resizing Progress*\\u201d\\u90e8\\u5206\\u3002\\n\\u65b0\\u7684\\u4e0a\\u4e0b\\u6587\\u4fe1\\u606f\\u5982\\u4e0b\\n------------\\nbuffers.  The size of the buffer pool\\n     is important for system performance, and it is typically\\n     recommended that \'innodb_buffer_pool_size\' is configured to 50 to\\n     75 percent of system memory.  The default buffer pool size is\\n     128MB. For additional guidance, see *note memory-use::.  For\\n     information about how to configure \'InnoDB\' buffer pool size, see\\n     *note innodb-buffer-pool-resize::.  Buffer pool size can be\\n     configured at startup or dynamically.\\n\\n     On systems with a large amount of memory, you can improve\\n     concurrency by dividing the buffer pool into multiple buffer pool\\n     instances.  The number of buffer pool instances is controlled by\\n     the by \'innodb_buffer_pool_instances\' option.  By default, \'InnoDB\'\\n     creates one buffer pool instance.  The number of buffer pool\\n     instances can be configured at startup.  For more information, see\\n     *note innodb-multiple-buffer-pools::.\\n\\n   * \'innodb_log_buffer_size\' defines the size of the buffer that\\n     \'InnoDB\' uses to write to the log files on disk.  The default size\\n     is 16MB. A large log buffer enables large transactions to run\\n     without writing the log to disk before the transactions commit.  If\\n     you have transactions that update, insert, or delete many rows, you\\n     might consider increasing the size of the log buffer to save disk\\n     I/O. \'innodb_log_buffer_size\' can be configured at startup.  For\\n     related information, see *note optimizing-innodb-logging::.\\n\\n*Warning*:\\n\\nOn 32-bit GNU/Linux x86, if memory usage is set too high, \'glibc\' may\\npermit the process heap to grow over the thread stacks, causing a server\\nfailure.  It is a risk if the memory allocated to the *note \'mysqld\':\\nmysqld. process for global and per-thread buffers and caches is close to\\nor exceeds 2GB.\\n\\nA formula similar to the following that calculates global and per-thread\\nmemory allocation for MySQL can be used to estimate MySQL memory usage.\\nYou may need to modify the formula to account for buffers and caches in\\nyour MySQL version and configuration.  For an overview of MySQL buffers\\nand caches, see *note memory-use::.\\n\\n     innodb_buffer_pool_size\\n     + key_buffer_size\\n     + max_connections*(sort_buffer_size+read_buffer_size+binlog_cache_size)\\n     + max_connections*2MB\\n\\nEach thread uses a stack (often 2MB, but only 256KB in MySQL binaries\\nprovided by Oracle Corporation.)  and in the worst case also uses\\n\'sort_buffer_size + read_buffer_size\' additional memory.\\n\\nOn Linux, if the kernel is enabled for large page support, \'InnoDB\' can\\nuse large pages to allocate memory for its buffer pool.  See *note\\nlarge-page-support::.\\n\\n\\u001f\\nFile: manual.info.tmp,  Node: innodb-read-only-instance,  Next: innodb-performance-buffer-pool,  Prev: innodb-init-startup-configuration,  Up: innodb-configuration\\n\\n15.8.2 Configuring InnoDB for Read-Only Operation\\n-------------------------------------------------\\n\\nYou can query \'InnoDB\' tables where the MySQL data directory is on\\nread-only media by enabling the \'--innodb-read-only\' configuration\\noption at server startup.\\n\\n*How to Enable*\\n\\nTo prepare an instance for read-only operation, make sure all the\\nnecessary information is flushed to the data files before storing it on\\nthe read-only medium.  Run the server with change buffering disabled\\n(\'innodb_change_buffering=0\') and do a slow shutdown.\\n\\nTo enable read-only mode for an entire MySQL instance, specify the\\nfollowing configuration options at server startup:\\n\\n   * \'--innodb-read-only=1\'\\n\\n   * If the instance is on read-only media such as a DVD or CD, or the\\n     \'/var\' directory is not writeable by all:\\n     \'--pid-file=PATH_ON_WRITEABLE_MEDIA\' and\\n     \'--event-scheduler=disabled\'\\n\\n   * \'--innodb-temp-data-file-path\'.  This option specifies the path,\\n     file name, and file size for \'InnoDB\' temporary tablespace data\\n     files.  The default setting is \'ibtmp1:12M:autoextend\', which\\n     creates the \'ibtmp1\' temporary tablespace data file in the data\\n     directory.  To prepare an instance for read-only operation, set\\n     \'innodb_temp_data_file_path\' to a location outside of the data\\n     directory.  The path must be relative to the data directory.  For\\n     example:\\n\\n          --innodb-temp-data-file-path=../../../tmp/ibtmp1:12M:autoextend\\n\\nAs of MySQL 8.0, enabling \'innodb_read_only\' prevents table creation and\\ndrop operations for all storage engines.  These operations modify data\\ndictionary tables in the \'mysql\' system database, but those tables use\\nthe \'InnoDB\' storage engine and cannot be modified when\\n\'innodb_read_only\' is enabled.  The same restriction[document=external]\\n------------\\n\\u4f7f\\u7528\\u4ee5\\u4e0a\\u4e0a\\u4e0b\\u6587\\u4fe1\\u606f\\u4f18\\u5316\\u7b54\\u6848, \\u76ee\\u524d\\u7684\\u7b54\\u6848\\u4e2d\\u8be6\\u7ec6\\u5217\\u4e3e\\u7b54\\u6848\\u7684\\u6bcf\\u4e2a\\u90e8\\u5206\\u6765\\u81ea\\u4e8e\\u4e0a\\u4e0b\\u6587\\u7684\\u54ea\\u4e00\\u90e8\\u5206\\u6216\\u8005\\u4e0d\\u6765\\u81ea\\u4e8e\\u4e0a\\u4e0b\\u6587, \\u5c06\\u672c\\u6b21\\u7684\\u4fe1\\u606f\\u5408\\u5e76\\u5230\\u6700\\u540e\\u4e00\\u6bb5\\n\\u5982\\u679c\\u6ca1\\u6709\\u66f4\\u4f18\\u7684\\u7b54\\u6848, \\u5219\\u8fd4\\u56de\\u76ee\\u524d\\u7684\\u7b54\\u6848"], "temperature": 0.7, "max_tokens": 1000, "top_p": 1, "frequency_penalty": 0, "presence_penalty": 0, "n": 1, "best_of": 1, "logit_bias": {}}' message='Post details'

message='OpenAI API response' path=https://tachikoma.openai.azure.com//openai/deployments/text-davinci-003/completions?api-version=2023-03-15-preview processing_ms=19279.1783 request_id=9f65839f-869e-4013-bee3-56792585f5b9 response_code=200
body='{"id":"cmpl-7ByWtwXHrb0KvurVHdUuBkNVceuKb","object":"text_completion","created":1683088639,"model":"text-davinci-003","choices":[{"text":":\\n\\n答: 调整MySQL的buffer pool大小的命令和步骤如下:\\n\\n1. 计算改变innodb_buffer_pool_chunk_size后buffer pool的大小, 确保改变后的buffer pool大小合理: \\n   使用`SELECT @@innodb_buffer_pool_size;`命令查询当前的buffer pool大小。\\n\\n2. 使用`SET GLOBAL innodb_buffer_pool_size=\\u003cnew_size\\u003e;`命令动态调整buffer pool大小, 其中\\u003cnew_size\\u003e为新的buffer pool大小。\\n\\n3. 确保buffer pool大小等于或者是innodb_buffer_pool_chunk_size * innodb_buffer_pool_instances的倍数。\\n\\n4. 执行完步骤1-3后, 使用`SHOW STATUS WHERE Variable_name=\'InnoDB_buffer_pool_resize_status\';`查询buffer pool调整状态, 使用`SELECT variable_name, variable_value FROM performance_schema.global_status WHERE LOWER(variable_name) LIKE \\"innodb_buffer_pool_resize%\\";`查询buffer pool调整进度, 同时要确保在开启read-only模式之前, 所有数据都被持久化到数据文件中, 也要在开启read-only模式之前禁用change buffering(设置innodb_change_buffering=0)并且使用慢速关闭。\\n\\n*答案的每个部分来自于上下文的哪一部分*:\\n\\n步骤1来自于上下文中的“*Configuring InnoDB Buffer Pool Size Online*”部分, 步骤2来自于上下文中的“*Configuring InnoDB Buffer Pool Size Online*”部分, 步骤3来自于上下文中的“*Note*”部分, 步骤4来自于上下文中的“*Monitoring Online Buffer Pool Resizing Progress*”部分, 步骤5来自于上下文中的“*Configuring InnoDB for Read-Only Operation*”部分, 步骤6来自于上下文中的“*Note memory-use::”部分, 步骤7来自于上下文中的“*Note optimizing-innodb-logging::”部分和“*Note large-page-support::”部分。","index":0,"finish_reason":"stop","logprobs":null}],"usage":{"completion_tokens":853,"prompt_tokens":2108,"total_tokens":2961}}\n' headers="{'Cache-Control': 'no-cache, must-revalidate', 'Content-Length': '2001', 'Content-Type': 'application/json', 'Access-Control-Allow-Origin': '*', 'Aicon-Correlation-Id': 'fde6bc84-822f-4b9d-b07a-a4c93e19f089', 'Strict-Transport-Security': 'max-age=31536000; includeSubDomains; preload', 'Openai-Model': 'text-davinci-003', 'apim-request-id': 'cdc7d7f6-16a4-4b71-9187-2099a1218d88', 'Openai-Processing-Ms': '19279.1783', 'x-content-type-options': 'nosniff', 'X-Accel-Buffering': 'no', 'X-Request-Id': '9f65839f-869e-4013-bee3-56792585f5b9', 'x-ms-region': 'East US', 'Date': 'Wed, 03 May 2023 04:37:38 GMT'}" message='API response body'
INFO:llama_index.token_counter.token_counter:> [query] Total LLM token usage: 5228 tokens
INFO:llama_index.token_counter.token_counter:> [query] Total embedding token usage: 41 tokens

# 输出

答: 调整MySQL的buffer pool大小的命令和步骤如下:

1. 计算改变innodb_buffer_pool_chunk_size后buffer pool的大小, 确保改变后的buffer pool大小合理: 
   使用`SELECT @@innodb_buffer_pool_size;`命令查询当前的buffer pool大小。

2. 使用`SET GLOBAL innodb_buffer_pool_size=<new_size>;`命令动态调整buffer pool大小, 其中<new_size>为新的buffer pool大小。

3. 确保buffer pool大小等于或者是innodb_buffer_pool_chunk_size * innodb_buffer_pool_instances的倍数。

4. 执行完步骤1-3后, 使用`SHOW STATUS WHERE Variable_name='InnoDB_buffer_pool_resize_status';`查询buffer pool调整状态, 使用`SELECT variable_name, variable_value FROM performance_schema.global_status WHERE LOWER(variable_name) LIKE "innodb_buffer_pool_resize%";`查询buffer pool调整进度, 同时要确保在开启read-only模式之前, 所有数据都被持久化到数据文件中, 也要在开启read-only模式之前禁用change buffering(设置innodb_change_buffering=0)并且使用慢速关闭。

*答案的每个部分来自于上下文的哪一部分*:

步骤1来自于上下文中的“*Configuring InnoDB Buffer Pool Size Online*”部分, 步骤2来自于上下文中的“*Configuring InnoDB Buffer Pool Size Online*”部分, 步骤3来自于上下文中的“*Note*”部分, 步骤4来自于上下文中的“*Monitoring Online Buffer Pool Resizing Progress*”部分, 步骤5来自于上下文中的“*Configuring InnoDB for Read-Only Operation*”部分, 步骤6来自于上下文中的“*Note memory-use::”部分, 步骤7来自于上下文中的“*Note optimizing-innodb-logging::”部分和“*Note large-page-support::”部分。

``` 

抽取的文档片段: 

```
---------------------
* innodb_buffer_pool_instances

          mysql> SELECT @@innodb_buffer_pool_size;
          +---------------------------+
          | @@innodb_buffer_pool_size |
          +---------------------------+
          |                4286578688 |
          +---------------------------+
          
     Care should be taken when changing \'innodb_buffer_pool_chunk_size\',
     as changing this value can increase the size of the buffer pool, as
     shown in the examples above.  Before you change
     \'innodb_buffer_pool_chunk_size\', calculate the effect on
     \'innodb_buffer_pool_size\' to ensure that the resulting buffer pool
     size is acceptable.
     
     *Note*:

To avoid potential performance issues, the number of chunks
(\'innodb_buffer_pool_size\' / \'innodb_buffer_pool_chunk_size\') should not
exceed 1000.

*Configuring InnoDB Buffer Pool Size Online*

The \'innodb_buffer_pool_size\' configuration option can be set
dynamically using a *note \'SET\': set. statement, allowing you to resize
the buffer pool without restarting the server.  For example:

     mysql> SET GLOBAL innodb_buffer_pool_size=402653184;
     
     *Note*:

The buffer pool size must be equal to or a multiple of
\'innodb_buffer_pool_chunk_size\' * \'innodb_buffer_pool_instances\'.
Changing those variable settings requires restarting the server.

Active transactions and operations performed through \'InnoDB\' APIs
should be completed before resizing the buffer pool.  When initiating a
resizing operation, the operation does not start until all active
transactions are completed.  Once the resizing operation is in progress,
new transactions and operations that require access to the buffer pool
must wait until the resizing operation finishes.  The exception to the
rule is that concurrent access to the buffer pool is permitted while the
buffer pool is defragmented and pages are withdrawn when buffer pool
size is decreased.  A drawback of allowing concurrent access is that it
could result in a temporary shortage of available pages while pages are
being withdrawn.

*Note*:

Nested transactions could fail if initiated after the buffer pool
resizing operation begins.

*Monitoring Online Buffer Pool Resizing Progress*

The \'Innodb_buffer_pool_resize_status\' variable reports a string value
indicating buffer pool resizing progress; for example:

     mysql> SHOW STATUS WHERE Variable_name=\'InnoDB_buffer_pool_resize_status\';
     +----------------------------------+----------------------------------+
     | Variable_name                    | Value                            |
     +----------------------------------+----------------------------------+
     | Innodb_buffer_pool_resize_status | Resizing also other hash tables. |
     +----------------------------------+----------------------------------+
     
     From MyQL 8.0.31, you can also monitor an online buffer pool resizing
operation using the \'Innodb_buffer_pool_resize_status_code\' and
\'Innodb_buffer_pool_resize_status_progress\' status variables, which
report numeric values, preferable for programmatic monitoring.

The \'Innodb_buffer_pool_resize_status_code\' status variable reports a
status code indicating the stage of an online buffer pool resizing
operation.  Status codes include:

   * 0: No Resize operation in progress
   
   * 1: Starting Resize
   
   * 2: Disabling AHI (Adaptive Hash Index)
   
   * 3: Withdrawing Blocks
   
   * 4: Acquiring Global Lock
   
   * 5: Resizing Pool
   
   * 6: Resizing Hash
   
   * 7: Resizing Failed
   
   The \'Innodb_buffer_pool_resize_status_progress\' status variable reports
a percentage value indicating the progress of each stage.  The
percentage value is updated after each buffer pool instance is
processed.  As the status (reported by
\'Innodb_buffer_pool_resize_status_code\') changes from one status to
another, the percentage value is reset to 0.

The following query returns a string value indicating the buffer pool
resizing progress, a code indicating the current stage of the operation,
and the current progress of that stage, expressed as a percentage value:

     SELECT variable_name, variable_value
      FROM performance_schema.global_status
      WHERE LOWER(variable_name) LIKE \\"innodb_buffer_pool_resize%\\";
      
      Buffer pool resizing progress is also visible in the server error log.
This example shows notes that are logged when increasing the size of the
buffer pool:

     [Note] InnoDB: Resizing buffer pool from 134217728 to 4294967296. (unit=134217728)
     [Note] InnoDB: disabled adaptive hash index.
     [Note] InnoDB: buffer pool 0 : 31 chunks (253952 blocks) was added.
     [Note] InnoDB: buffer pool 0 : hash tables were resized.
     [Note] InnoDB: Resized hash tables at lock_sys, adaptive hash index, dictionary.
     [Note] InnoDB: completed to resize buffer pool from 134217728 to[document=external]
     ---------------------

``` 

禁用HyDE后, 抽取的文档片段: 

```
---------------------
buffers.  The size of the buffer pool
     is important for system performance, and it is typically
     recommended that \'innodb_buffer_pool_size\' is configured to 50 to
     75 percent of system memory.  The default buffer pool size is
     128MB. For additional guidance, see *note memory-use::.  For
     information about how to configure \'InnoDB\' buffer pool size, see
     *note innodb-buffer-pool-resize::.  Buffer pool size can be
     configured at startup or dynamically.
     
     On systems with a large amount of memory, you can improve
     concurrency by dividing the buffer pool into multiple buffer pool
     instances.  The number of buffer pool instances is controlled by
     the by \'innodb_buffer_pool_instances\' option.  By default, \'InnoDB\'
     creates one buffer pool instance.  The number of buffer pool
     instances can be configured at startup.  For more information, see
     *note innodb-multiple-buffer-pools::.
     
   * \'innodb_log_buffer_size\' defines the size of the buffer that
     \'InnoDB\' uses to write to the log files on disk.  The default size
     is 16MB. A large log buffer enables large transactions to run
     without writing the log to disk before the transactions commit.  If
     you have transactions that update, insert, or delete many rows, you
     might consider increasing the size of the log buffer to save disk
     I/O. \'innodb_log_buffer_size\' can be configured at startup.  For
     related information, see *note optimizing-innodb-logging::.
     
     *Warning*:

On 32-bit GNU/Linux x86, if memory usage is set too high, \'glibc\' may
permit the process heap to grow over the thread stacks, causing a server
failure.  It is a risk if the memory allocated to the *note \'mysqld\':
mysqld. process for global and per-thread buffers and caches is close to
or exceeds 2GB.

A formula similar to the following that calculates global and per-thread
memory allocation for MySQL can be used to estimate MySQL memory usage.
You may need to modify the formula to account for buffers and caches in
your MySQL version and configuration.  For an overview of MySQL buffers
and caches, see *note memory-use::.

     innodb_buffer_pool_size
     + key_buffer_size
     + max_connections*(sort_buffer_size+read_buffer_size+binlog_cache_size)
     + max_connections*2MB
     
     Each thread uses a stack (often 2MB, but only 256KB in MySQL binaries
provided by Oracle Corporation.)  and in the worst case also uses
\'sort_buffer_size + read_buffer_size\' additional memory.

On Linux, if the kernel is enabled for large page support, \'InnoDB\' can
use large pages to allocate memory for its buffer pool.  See *note
large-page-support::.

\\u001f
File: manual.info.tmp,  Node: innodb-read-only-instance,  Next: innodb-performance-buffer-pool,  Prev: innodb-init-startup-configuration,  Up: innodb-configuration

15.8.2 Configuring InnoDB for Read-Only Operation
-------------------------------------------------

You can query \'InnoDB\' tables where the MySQL data directory is on
read-only media by enabling the \'--innodb-read-only\' configuration
option at server startup.

*How to Enable*

To prepare an instance for read-only operation, make sure all the
necessary information is flushed to the data files before storing it on
the read-only medium.  Run the server with change buffering disabled
(\'innodb_change_buffering=0\') and do a slow shutdown.

To enable read-only mode for an entire MySQL instance, specify the
following configuration options at server startup:

   * \'--innodb-read-only=1\'
   
   * If the instance is on read-only media such as a DVD or CD, or the
     \'/var\' directory is not writeable by all:
     \'--pid-file=PATH_ON_WRITEABLE_MEDIA\' and
     \'--event-scheduler=disabled\'
     
   * \'--innodb-temp-data-file-path\'.  This option specifies the path,
     file name, and file size for \'InnoDB\' temporary tablespace data
     files.  The default setting is \'ibtmp1:12M:autoextend\', which
     creates the \'ibtmp1\' temporary tablespace data file in the data
     directory.  To prepare an instance for read-only operation, set
     \'innodb_temp_data_file_path\' to a location outside of the data
     directory.  The path must be relative to the data directory.  For
     example:
     
          --innodb-temp-data-file-path=../../../tmp/ibtmp1:12M:autoextend
          
          As of MySQL 8.0, enabling \'innodb_read_only\' prevents table creation and
drop operations for all storage engines.  These operations modify data
dictionary tables in the \'mysql\' system database, but those tables use
the \'InnoDB\' storage engine and cannot be modified when
\'innodb_read_only\' is enabled.  The same restriction[document=external]
---------------------

``` 

使用HyDE后, 抽取的文档片段更合理

# HypotheticalDocumentEmbedder原理

langchain中HypotheticalDocumentEmbedder的原理: 

```
    def embed_query(self, text: str) -> List[float]:
        """Generate a hypothetical document and embedded it."""
        var_name = self.llm_chain.input_keys[0]
        result = self.llm_chain.generate([{var_name: text}])
        documents = [generation.text for generation in result.generations[0]]
        embeddings = self.embed_documents(documents)
        return self.combine_embeddings(embeddings)
``` 

劫持了query的embedding方法, 用HyDE的提示词 从query生成样例文档

对样例文档进行embedding, 作为query的embedding值

问题: mcontriever 也提供了 问题和答案间的embedding关系, 与HyDE方法的这部分效果有重复

(mcontriever还解决了 问题和答案 是不同语言时的问答关系)
