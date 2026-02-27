---
title: 20230403 - 测试 google Flan T5 XL模型
confluence_page_id: 2130563
created_at: 2023-04-03T10:27:48+00:00
updated_at: 2023-04-05T02:59:25+00:00
---

由于OpenAI的API超过用量, 且不容易充值, 换用 google Flan T5 XL模型

由于需要大量内存, 迁移到物理机上进行测试

```
docker run --privileged -it --name openai -p 8888:8888 -d python:3

pip install pysocks -i https://pypi.tuna.tsinghua.edu.cn/simple
pip install llama-index openai -i https://pypi.tuna.tsinghua.edu.cn/simple
 
//配置jupyter
pip install notebook ipywidgets -i https://pypi.tuna.tsinghua.edu.cn/simple
 
//设置密码
jupyter notebook password
  
//启动notebook
nohup jupyter notebook --allow-root --ip=0.0.0.0 &
``` 

huggingface的缓存文件, 留了一份在 10.186.17.104:/data/huangyan/openai/.cache 中, 复制到容器中

```
pip install transformers -i https://pypi.tuna.tsinghua.edu.cn/simple
pip install torch -i https://pypi.tuna.tsinghua.edu.cn/simple
pip install sentence_transformers -i https://pypi.tuna.tsinghua.edu.cn/simple
 
export ALL_PROXY=socks5://192.168.22.8:7890

```
