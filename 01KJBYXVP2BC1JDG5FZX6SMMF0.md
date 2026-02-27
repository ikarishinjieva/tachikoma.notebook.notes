---
title: 20230412 - 测试 alpaca 与 llama_index结合
confluence_page_id: 2130619
created_at: 2023-04-12T02:33:25+00:00
updated_at: 2023-04-17T06:58:48+00:00
---

参考代码

<https://github.com/thohag/alpaca_llama_index>

# 额外知识

  - 其中用到了 huggingface/peft 库, 主要作用是对于 预训练好的模型, 仅对少数参数进行 fine-tuning, 达成效果
    - 支持的方法包括 LORA, Prefix Tuning, 等

# 效果提升的可能

  - 使用参考代码, 效果并不好, 不如chatgpt. 效果提升的可能性:   

    - 模型使用的是 decapoda-research/llama-7b-hf + tloen/alpaca-lora-7b. 尝试使用更大的模型
    - llama-index切片的大小 (获取原始片段的质量, 是否能跟chatgpt对比?)
    - 改善tokenizer
      - 切片阶段, tokenizer使用了llama-index默认的tokenizer, 考虑更换LlamaTokenizer(llama-7b), 或者lora后的tokenizer
      - 目前使用LlamaTokenizer(llama-7b), 是否要更换成lora后的tokenizer 
    - 改善prompt (与chatgpt对比)
    - generate的参数

# 效果提升1

使用llama 13b相关模型: 

  - hf_model_path = "decapoda-research/llama-13b-hf"
  - alpaca_model_path = "chansung/alpaca-lora-13b"

使用llama-13b对应的tokenizer, 同时用到切片阶段和预测阶段

文件: [test.py.ipynb](/assets/01KJBYXVP2BC1JDG5FZX6SMMF0/test.py.ipynb)

效果: 当问题比较精确时, 效果中等, 比7b效果好 (但仍然不能将dynamically的需求进行响应): 

![image2023-4-14 14:47:29.png](/assets/01KJBYXVP2BC1JDG5FZX6SMMF0/image2023-4-14%2014%3A47%3A29.png)

但当问题比较模糊, 或者词不命中时, 效果不理想: 

![image2023-4-14 14:48:13.png](/assets/01KJBYXVP2BC1JDG5FZX6SMMF0/image2023-4-14%2014%3A48%3A13.png)

![image2023-4-14 14:48:40.png](/assets/01KJBYXVP2BC1JDG5FZX6SMMF0/image2023-4-14%2014%3A48%3A40.png)

# 效果提升2

尝试使用30b的模型, 显存不足

```
ValueError:
                        Some modules are dispatched on the CPU or the disk. Make sure you have enough GPU RAM to fit
                        the quantized model. If you want to dispatch the model on the CPU or the disk while keeping
                        these modules in 32-bit, you need to set `load_in_8bit_fp32_cpu_offload=True` and pass a custom
                        `device_map` to `from_pretrained`. Check
                        https://huggingface.co/docs/transformers/main/en/main_classes/quantization#offload-between-cpu-and-gpu
                        for more details.
``` 

将所有计算切换成只是用cpu

  - 安装cpu版本的torch: pip3 install torch==2.0.0+cpu torchvision==0.15.1+cpu -f <https://download.pytorch.org/whl/torch_stable.html>
  - 失败, bitsandbytes会仍然调用 torch.cuda, 然后报错

将参数load_in_8bit设置为False, 好像是可以完成加载 (参考: <https://aiqianji.com/openoker/alpaca-lora/src/dockerfile/generate.py>, 其中调用了model.half())

关于load_in_8bit的说明: 

```
            load_in_8bit (`bool`, *optional*, defaults to `False`):
                If `True`, will convert the loaded model into mixed-8bit quantized model. To use this feature please
                install `bitsandbytes` compiled with your CUDA version by running `pip install -i
                https://test.pypi.org/simple/ bitsandbytes-cudaXXX` where XXX is your CUDA version (e.g. 11.6 = 116).
                Make also sure that you have enough GPU RAM to store half of the model size since the 8bit modules are
                not compiled and adapted for CPUs.
``` 

调整代码, 使之可以在cpu上运行: 

[alpaca_lora-llama_inex.ipynb](/assets/01KJBYXVP2BC1JDG5FZX6SMMF0/alpaca_lora-llama_inex.ipynb)

运行速度很慢, 输出比使用7b/13b时稳定, 但效果一般 (无法识别enlarge): 

![image2023-4-17 14:58:19.png](/assets/01KJBYXVP2BC1JDG5FZX6SMMF0/image2023-4-17%2014%3A58%3A19.png)

待测试: 

  - <https://github.com/huggingface/transformers/issues/20361>
  - <https://huggingface.co/docs/transformers/main/en/main_classes/quantization#offload-between-cpu-and-gpu>
    - 测试失败
  - 使用10.186.16.131, 已传输 huggingface的.cache
