---
title: 20230527 - 吴恩达: 提示词工程课
confluence_page_id: 2392095
created_at: 2023-05-26T17:25:27+00:00
updated_at: 2023-05-28T02:24:43+00:00
---

# URL

<https://juejin.cn/post/7229891190900441125>

# 知识: 使用输出样例

Step 1 ... Step N 的输出样例

```
text_1 = f"""
Making a cup of tea is easy! First, you need to get some \ 
water boiling. While that's happening, \ 
grab a cup and put a tea bag in it. Once the water is \ 
hot enough, just pour it over the tea bag. \ 
Let it sit for a bit so the tea can steep. After a \ 
few minutes, take out the tea bag. If you \ 
like, you can add some sugar or milk to taste. \ 
And that's it! You've got yourself a delicious \ 
cup of tea to enjoy.
"""
prompt = f"""
You will be provided with text delimited by triple quotes. 
If it contains a sequence of instructions, \ 
re-write those instructions in the following format:

Step 1 - ...
Step 2 - …
…
Step N - …

If the text does not contain a sequence of instructions, \ 
then simply write "No steps provided."

"""{text_1}"""
"""
response = get_completion(prompt)
print("Completion for Text 1:")
print(response)
Completion for Text 1:
Step 1 - Get some water boiling.
Step 2 - Grab a cup and put a tea bag in it.
Step 3 - Once the water is hot enough, pour it over the tea bag.
Step 4 - Let it sit for a bit so the tea can steep.
Step 5 - After a few minutes, take out the tea bag.
Step 6 - Add some sugar or milk to taste.
Step 7 - Enjoy your delicious cup of tea!
``` 

# 知识: 少样本提示

使用样例, 并要求LLM输出相同的风格

```
prompt = f"""
Your task is to answer in a consistent style.

<child>: Teach me about patience.

<grandparent>: The river that carves the deepest \ 
valley flows from a modest spring; the \ 
grandest symphony originates from a single note; \ 
the most intricate tapestry begins with a solitary thread.

<child>: Teach me about resilience.
"""
response = get_completion(prompt)
print(response)
"""
<grandparent>: Resilience is like a tree that bends with the wind but never breaks. It's the ability to bounce back from adversity and keep moving forward, even when things get tough. Just like a tree needs strong roots to withstand the storm, we need to cultivate inner strength and perseverance to overcome life's challenges.
"""

``` 

# 知识 - 让模型先梳理再给结论

通过提示词, 让模型思考, 然后给结论

参考URL中的相关章节

# 知识 - 用Redlines可以标记文本差异

![image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ca93f2dbcd67486c9438da0b652f5690~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

# OpenAI文档节选

<https://platform.openai.com/docs/guides/chat/instructing-chat-models>

If the model isn’t generating the output you want, feel free to iterate and experiment with potential improvements. You can try approaches like:

  - Make your instruction more explicit
  - Specify the format you want the answer in
  - Ask the model to think step by step or debate pros and cons before settling on an answer
