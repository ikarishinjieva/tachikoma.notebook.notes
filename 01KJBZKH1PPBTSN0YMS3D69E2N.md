---
title: 20241115 - ms-swift是如何计算acc的
confluence_page_id: 3343140
created_at: 2024-11-15T07:35:19+00:00
updated_at: 2024-11-15T07:35:58+00:00
---

函数: 

```
def compute_acc_metrics(eval_prediction: EvalPrediction,
                        acc_strategy: Literal['token', 'sentence'] = 'token',
                        is_encoder_decoder: bool = False) -> Dict[str, torch.Tensor]:
    if is_encoder_decoder:
        labels = eval_prediction.label_ids[..., :]
        predictions = eval_prediction.predictions[..., :]
    else:
        labels = eval_prediction.label_ids[..., 1:]
        predictions = eval_prediction.predictions[..., :-1]
    if predictions.shape != labels.shape:
        return {}
    masks = labels != -100
    if acc_strategy == 'sentence':
        acc_list = []
        for i, m in enumerate(masks):
            acc_list.append(np.all(predictions[i, m] == labels[i, m]))
        acc = np.mean(np.array(acc_list))
    else:
        acc = np.mean((predictions[masks] == labels[masks]).astype(np.float64))
    return {'acc': acc}
```
````
这个函数`compute_acc_metrics` 计算生成式模型的准确率，并提供了按token或句子级别的准确率计算策略。让我们逐步分解：

1. **函数参数:**

   * `eval_prediction: EvalPrediction`:  这是一个包含模型预测结果和真实标签的数据结构。通常包含 `label_ids` (真实标签) 和 `predictions` (模型预测)。
   * `acc_strategy: Literal['token', 'sentence'] = 'token'`:  指定准确率的计算策略，可以是 `'token'` 或 `'sentence'`。默认为 `'token'`。
   * `is_encoder_decoder: bool = False`:  指示模型是否是编码器-解码器结构。默认为 `False`。

2. **处理编码器-解码器模型:**

   ```python
   if is_encoder_decoder:
       labels = eval_prediction.label_ids[..., :]
       predictions = eval_prediction.predictions[..., :]
   else:
       labels = eval_prediction.label_ids[..., 1:]
       predictions = eval_prediction.predictions[..., :-1]
   ```
   这段代码处理编码器-解码器模型和非编码器-解码器模型的标签和预测值。
   * 对于编码器-解码器模型，直接使用全部标签和预测。
   * 对于非编码器-解码器模型 (例如，自回归模型)，通常第一个标签是特殊起始标记，最后一个预测是特殊结束标记。这些标记不参与准确率计算，因此代码去除了它们。

3. **形状检查:**

   ```python
   if predictions.shape != labels.shape:
       return {}
   ```
   如果预测和标签的形状不匹配，则返回空字典，表示无法计算准确率。

4. **处理掩码:**

   ```python
   masks = labels != -100
   ```
   `-100` 通常用于表示 padding tokens，这些 tokens 不应该参与准确率计算。`masks` 是一个布尔数组，指示哪些位置的标签是有效的。

5. **计算准确率:**

   * **句子级别准确率 (`acc_strategy == 'sentence'`):**

     ```python
     acc_list = []
     for i, m in enumerate(masks):
         acc_list.append(np.all(predictions[i, m] == labels[i, m]))
     acc = np.mean(np.array(acc_list))
     ```
     对于每个句子，只有当该句子中所有有效 token 的预测都与标签完全匹配时，才算该句子预测正确。最终的准确率是所有句子预测正确率的平均值。

   * **Token级别准确率 (`acc_strategy == 'token'`):**

     ```python
     acc = np.mean((predictions[masks] == labels[masks]).astype(np.float64))
     ```
     计算所有有效 token 的预测准确率的平均值。

6. **返回值:**

   ```python
   return {'acc': acc}
   ```
   返回一个字典，其中键 `'acc'` 对应计算出的准确率。

**总结:**

该函数根据指定的策略计算生成式模型的准确率。它考虑了编码器-解码器结构、padding tokens，并提供了句子级别和 token 级别的准确率计算选项。  句子级别的准确率要求句子中所有有效token都预测正确，而 token 级别的准确率则分别计算每个有效token的准确率。
```` 

结论: sentence模式, 评估句子和预期完全一样, token模式, 评估token和预期完全一样

loss评估的是当前答案与预期的差距, acc评估的是当前答案是否是预期
