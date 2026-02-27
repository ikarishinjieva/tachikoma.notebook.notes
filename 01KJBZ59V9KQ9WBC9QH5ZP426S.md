---
title: 20240204 - 阅读论文: REPLUG: Retrieval-Augmented Black-Box Language Models
confluence_page_id: 2589999
created_at: 2024-02-04T07:02:43+00:00
updated_at: 2024-02-04T09:07:00+00:00
---

<https://app.txyz.ai/chat/7b008cd8-988a-4a57-9c44-cdad93192722>

# 目录

# 问答

```
---

本论文主要解决的问题是什么, 用中文回答
本论文主要解决的问题是如何通过引入检索模型来改进大型语言模型的性能。通过将检索到的文档添加到冻结的黑盒语言模型的输入中，REPLUG可以提供有助于改进预测的文档。研究问题包括：如何通过检索增强语言模型来提高性能？语言模型是否可以监督检索模型以找到相关文档？REPLUG与其他检索增强语言模型相比的性能如何？该论文的关键发现是：REPLUG将GPT-3在语言建模方面的性能提高了6.3%，将Codex在五轮MMLU上的性能提高了5.1%。这篇论文的重要性在于它提出了一种简单而有效的方法，通过引入检索模型来增强大型语言模型的性能。

---

REPlUG和RAG一样, 是将检索文档放在LLM的上下文中么
是的，REPLUG和RAG（Retrieval-Augmented Generation）都是将检索到的文档添加到语言模型（LLM）的上下文中的方法。它们的目标都是通过引入外部检索信息来增强语言模型的性能。不同之处在于，REPLUG将语言模型视为黑盒，并通过可调节的检索模块对其进行增强，而RAG则是通过更新语言模型的参数来实现检索增强。REPLUG的黑盒假设使其适用于大型语言模型，而RAG则需要对语言模型进行微调或训练。

在REPLUG中，首先使用一个现成的检索模型从外部语料库中检索相关文档。然后，将这些检索到的文档与输入上下文进行拼接，并将其作为增强后的输入传递给语言模型，以进行最终的预测。为了克服语言模型上下文长度的限制，REPLUG还引入了一种新的集成策略，可以并行地对检索到的文档进行编码，并与相同的语言模型进行集成。

---

列出REPlUG的主要步骤
REPLUG的主要步骤如下：

1. 文档检索：给定输入上下文，使用一个现成的检索模型从外部语料库中检索相关文档。这个检索模型通常是基于双编码器架构的密集检索器，它将输入上下文和文档编码为嵌入向量，并计算它们之间的相似度得分。通过比较相似度得分，选择与输入上下文最相关的前k个文档。

2. 输入重构：将检索到的前k个文档与输入上下文进行拼接，形成增强后的输入。为了克服语言模型上下文长度的限制，采用了一种集成策略。首先，将每个检索到的文档与输入上下文分别拼接，然后将这些拼接后的输入分别传递给语言模型，并从所有k次传递的输出概率中进行集成。集成方法采用加权平均的方式，其中权重是基于文档与输入上下文之间的相似度得分计算的。

3. 预测生成：将增强后的输入传递给语言模型，进行最终的预测生成。语言模型根据输入上下文和检索到的文档的信息，生成下一个标记的概率分布。

通过将检索到的文档与语言模型的上下文进行集成，REPLUG能够提供更丰富的信息，帮助语言模型做出更准确的预测。实验证明，REPLUG可以显著提高不同大小的语言模型在语言建模和下游任务上的性能。

---

列出输入重构的详细步骤
输入重构的详细步骤如下：

1. 检索到的文档：根据输入上下文使用检索器从外部语料库中检索到一小组相关文档。这些文档被认为与输入上下文相关，并且将用于增强语言模型的预测。

2. 文档与上下文的拼接：将每个检索到的文档与输入上下文进行拼接。这样可以将文档的信息融入到输入上下文中，以帮助语言模型做出更准确的预测。

3. 并行传递到语言模型：将拼接后的输入分别传递给语言模型。对于每个拼接后的输入，语言模型会生成下一个标记的概率分布。

4. 输出概率的集成：对于每个拼接后的输入，语言模型都会生成一个输出概率分布。通过对所有拼接后输入的输出概率进行加权平均，得到最终的输出概率。权重是基于文档与输入上下文之间的相似度得分计算
 
---
 
为什么要生成最终的输出概率

生成最终的输出概率是为了综合考虑从检索到的文档中获得的信息和输入上下文，以帮助语言模型做出更准确的预测。通过将每个检索到的文档与输入上下文进行拼接，并将拼接后的输入传递给语言模型，可以融合文档的信息并扩展语言模型的上下文。每个拼接后的输入都会生成一个输出概率分布，通过对所有拼接后输入的输出概率进行加权平均，可以得到最终的输出概率。这样做的目的是利用检索到的文档提供的丰富信息，帮助语言模型更好地预测下一个标记。

``` 

# 疑问

至此, 不能理解 REPlUG最终输出的概率, 与大模型应用的关系

查询样例代码: <https://github.com/IntelLabs/fastRAG/blob/main/examples/replug_parallel_reader.ipynb>

其中REPlUG是影响了invocation_layer: 

```
PrompterModel = PromptModel(
    model_name_or_path= "meta-llama/Llama-2-7b-chat-hf",
    use_gpu= True,
    invocation_layer_class=ReplugHFLocalInvocationLayer,
    model_kwargs= dict(
        max_new_tokens=10,
        model_kwargs= dict(
            device_map = "auto",
            torch_dtype = torch.bfloat16),
        generation_kwargs=dict(do_sample=True)))
``` 

对invocation layer的理解, 是大模型最外的一层, 组织tokenizer + generate + decode的整个流程

REPlUG定制了invocation layer, (<https://github.com/IntelLabs/fastRAG/blob/d7e23835a9556601d3688e4968a5ca57ac781ccb/fastrag/prompters/invocation_layers/replug.py#L486>), 其中做了两个注入: 

  1. 在tokenizer之后, model.generate时, 传入了logits processor, (REPLUGLogitsProcessor), REPLUGLogitsProcessor会根据document的权重, 产生修订权重, 给下一步使用
  2. 将model修订, 融合了REPLUG_Generation. (相当于将REPLUG_Generation功能附加到了各种模型上), 在model.sample方法中, 用REPLUG对输出的next_token的权重进行修订 (其中用到了上一步的修订权重):   
![image2024-2-4 14:14:16.png](/assets/01KJBZ59V9KQ9WBC9QH5ZP426S/image2024-2-4%2014%3A14%3A16.png)

如果我在sample中, 修订了output的权重, 会发生什么

如果在 `model.sample()` 方法的执行过程中修改了输出的权重（即概率分布），那么生成的样本将会受到这些修订的权重的影响。

通常情况下，在 `model.sample()` 方法中，输出的权重是根据模型预测的概率分布自动计算得出的。这些概率分布反映了模型对每个可能的词或标记的预测置信度。当选择下一个词或标记时，采样方法会基于这些权重进行选择。

如果你在采样过程中修改了输出的权重，可能会导致生成结果的变化。例如，你可以增加某个特定词或标记的权重，使其更有可能被选择，或者减少某个词或标记的权重，使其不太可能被选择。通过修改权重，你可以对生成的样本施加一定的偏好或约束。

请注意，修改输出的权重可能会对生成的样本产生不可预测的影响。这取决于修改的方式和具体的采样方法。如果修改权重不合理，可能会导致生成的样本失去语义连贯性、语法正确性或现实性。因此，在修改输出权重时，需要仔细考虑和测试，以确保生成的样本仍然符合预期的质量标准和应用需求。

# 对于REPLUGLogitsProcessor的逻辑整理: 

代码: 

```
class REPLUGLogitsProcessor(LogitsProcessor):
    """
    REPLUG processor uses documents similarity scores to weight the different examples in a batch.
    """

    def __init__(self, documents_scores: torch.FloatTensor):
        "Documents' scores assumed to be L1 normalized."
 
		//假设 document_scores有三个文档的评分: [0.4,0.5,0.1]
		//召回的文档个数
        self.num_docs = documents_scores.shape[0]
 
		//为document_scores增加一个维度, 变为: [[0.4],[0.5],[0.1]]
        self.doc_scores = torch.unsqueeze(documents_scores, 1)

    def __call__(self, input_ids: torch.LongTensor, scores: torch.FloatTensor) -> torch.FloatTensor:
        "Scores assumed to be logits. Returns logits averaged over the batch."
 
		//此处的scores是大模型输出的next_tokens的概率分布, 每组对应一个document, 比如 [[0.4,0.5],[0.1,0.9],[0.2,0.1]]
 
		//将每个文档的相似度, 作为系数, 铺到每组next_token中
        replug_scores = self.doc_scores * scores 
		/*
		tensor([[0.1600, 0.2000],
        [0.0500, 0.4500],
        [0.0200, 0.0100]])
		*/
 
		//对next_token的每一维进行加和, 相当于根据 每个文档的相似度, 进行加权平均
        replug_scores = replug_scores.sum(dim=0)
		//tensor([0.2300, 0.6600])

		//替换原有的next_tokens, 将每组文档对应的next_token读调整为加权平均后的值
        replug_scores = torch.tile(replug_scores, (self.num_docs, 1))
		/*
		tensor([[0.2300, 0.6600],
        [0.2300, 0.6600],
        [0.2300, 0.6600]])
		*/

        return replug_scores
``` 

# 整理REPlUG

REPlUG的主要思想: 

  - 对于召回的多个文档, 将每个文档与原始问题的相似程度 归一化, 作为修订权重
  - 在模型中融入REPIUG的过程: 
    - 对于多篇文档的 next_token 的概率, 进行权重修订
    - 使用修订后的概率, 进行大模型的(next token的)推理

# 关于REPlUG LSR的问答

问答: 

```
REPLUG LSR的具体步骤如下：

1. 检索文档并计算检索概率：给定输入上下文x，使用检索器从语料库D中检索与x相关的一小组文档D'。使用余弦相似度计算查询嵌入和文档嵌入之间的相似性得分，选择具有最高相似性得分的前k个文档作为检索结果。计算每个检索到的文档d的检索概率PR(d | x)。

2. 使用语言模型计算概率：使用语言模型作为评分函数，衡量每个文档d对于改善语言模型困惑度的程度。计算每个文档d的语言模型概率Q(d | x, y)，其中y是给定输入上下文x和文档d的下一个标记的真实输出。

3. 更新检索模型参数：通过最小化检索概率PR(d | x)和语言模型概率Q(d | x, y)之间的KL散度来更新检索模型参数。KL散度衡量两个概率分布之间的差异，目标是使检索概率和语言模型概率分布尽可能接近。

4. 异步更新数据存储索引：对数据存储索引进行异步更新，以反映检索模型参数的变化。

通过这些步骤，REPLUG LSR利用语言模型本身提供的监督信号来调整检索器，以找到能够降低语言模型困惑度的文档。这种方法可以进一步改进REPLUG中的检索组件，提高语言模型在各种任务上的性能。
``` 

# 关于REPlUG LSR的整理

对Retriever的训练思路 (文中只讲了思路, 没有细节, 也没有实现案例): 

  - 对比 Retriever检索文档 和 使用语言模型评估文档 的指标
    - 对于Retriever检索文档, 选取的指标是 每个文档对于当前输入的检索概率
    - 使用语言模型评估文档, 选取的指标是: 对于当前输入, 能输出 文档的下一标记的概率
  - 通过KL散度, 评估以上两个概率分布的差异
  - 调整Retriever的参数 (但文中没有提及Retriever的什么参数), 使得KL散度最小化
    - 也就是说让 Retriever 的选择, 接近于 语言模型在没有文档辅助时的选择
