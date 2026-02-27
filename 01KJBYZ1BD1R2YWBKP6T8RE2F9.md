---
title: 20230502 - 尝试openai的fine tuning
confluence_page_id: 2130830
created_at: 2023-05-02T10:00:36+00:00
updated_at: 2023-05-02T16:14:13+00:00
---

- <https://platform.openai.com/docs/guides/fine-tuning/preparing-your-dataset>
    - 场景1: 分类问题
      - 场景1.1: 章节: <https://platform.openai.com/docs/guides/fine-tuning/case-study-is-the-model-making-untrue-statements>
        - 当答案不正确时, 将其转换为 分类问题, 将不正确的说法归于 "Supported: no" 类, 使得该说法被排斥
      - 可以使用 logprobs=2 参数, 来获取答案中词的概率, 来测试fine tunning的效果, 举例: 

```
{
    "id": "cmpl-xxx",
    "object": "text_completion",
    "created": 1623194615,
    "model": "text-davinci-002",
    "choices": [
        {
            "text": "apple",
            "index": 0,
            "logprobs": {
                "token_logprobs": [
                    -0.013354,
                    -9.123482,
                    -9.430067,
                    -10.274847,
                    -10.301934
                ],
                "top_logprobs": [
                    {
                        "token": "apple",
                        "logprob": -0.013354
                    },
                    {
                        "token": "fruit",
                        "logprob": -9.123482
                    },
                    {
                        "token": "tree",
                        "logprob": -9.430067
                    },
                    {
                        "token": "orange",
                        "logprob": -10.274847
                    },
                    {
                        "token": "banana",
                        "logprob": -10.301934
                    }
                ],
                "text_offset": [
                    0,
                    0
                ],
                "finished": true
            },
            "finish_reason": "length"
        }
    ]
}
```

  

    - 场景2: 条件生成  

  

\---

fine tuning的过程文档显示: 过程比较简单直接, 问题是如何采集足够多的测试数据 (正确的+错误的)

一个想法: 用chatgpt生成很多问题, 由人对答案进行标记
