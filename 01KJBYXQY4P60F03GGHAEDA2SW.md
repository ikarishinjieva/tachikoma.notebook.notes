---
title: 20230330 - llama-index 试用
confluence_page_id: 2130542
created_at: 2023-03-30T15:30:52+00:00
updated_at: 2023-04-02T07:59:50+00:00
---

参考: <https://zhuanlan.zhihu.com/p/615926148>

环境: 10.186.62.73:/opt/openai

docker run --privileged -it --name openai -d python:3

```
pip install pysocks
 
//使用本机的clash proxy 翻墙
export ALL_PROXY=socks5://192.168.22.210:7890
 
pip install llama-index
pip install openai
 
//配置jupyter
pip install notebook ipywidgets
 
//设置密码
jupyter notebook password
 
//启动notebook
nohup jupyter notebook --allow-root --ip=0.0.0.0 &
``` 

将MySQL manual的info版 放在 data目录下, 作为语料

输入和输出样例: [附件: llama_index_demo.html] 

# 其他信息

  - llama-index 可以使用 langchain中定义的LLM模型:
    - <https://python.langchain.com/en/latest/reference/modules/llms.html>
