---
title: 20241024 - 阅读decimer.ai的论文
confluence_page_id: 3342797
created_at: 2024-10-24T05:15:16+00:00
updated_at: 2024-10-27T16:52:46+00:00
---

# DECIMER-Image-Segmentation

  - <https://github.com/Kohulan/DECIMER-Image-Segmentation>
    - 用于从PDF或图片中, 分离出化学式
    - 样例: <https://github.com/Kohulan/DECIMER-Image-Segmentation/blob/master/DECIMER_Segmentation_notebook.ipynb>
      - ![image2024-10-24 11:15:49.png](/assets/01KJBZHWF83XQC7AE6RQD6E3YZ/image2024-10-24%2011%3A15%3A49.png)

# DECIMER-Image-Classifier

  - <https://github.com/Iagea/DECIMER-Image-Classifier>
    - 用于分别图片是否是化学式
    - 效果: <https://github.com/Iagea/DECIMER-Image-Classifier/blob/main/examples/DECIMER-Image-classifier_example.ipynb>

# DECIMER-Image_Transformer

  - <https://github.com/Kohulan/DECIMER-Image_Transformer>
  - 大体结构: encoder (将图片编码), transformer(输入图片的encoding, 输出结果)

## encoder

用EfficientNet-V2模型, 对象Efficient_Net_encoder.Encoder, 定义如下: 

```
class Encoder(tf.keras.Model):
    def __init__(
        self,
        image_embedding_dim,
        preprocessing_fn,
        backbone_fn,
        image_shape,
        do_permute=False,
        include_top=False,
        pretrained_weights=None,
        scale_factor=0,
    ):
        super(Encoder, self).__init__()

        self.image_embedding_dim = image_embedding_dim
        self.preprocessing_fn = preprocessing_fn
        self.encoder_backbone = backbone_fn(
            model_name=MODEL,
            include_top=include_top,
            weights=pretrained_weights,
            input_shape=image_shape,
        )
        self.reshape = tf.keras.layers.Reshape(
            self.image_embedding_dim, name="image_embedding"
        )
        self.include_top = include_top
        self.scale_factor = scale_factor

    def call(self, x, training):
        x = self.preprocessing_fn(x)
        x = self.encoder_backbone(
            x, training=training, features_only=not self.include_top
        )[self.scale_factor]
        x = self.reshape(x, training=training)
        return x
``` 

初始化参数如下:

```
IMG_EMB_DIM = (16, 16, 512)
IMG_EMB_DIM = (IMG_EMB_DIM[0] * IMG_EMB_DIM[1], IMG_EMB_DIM[2])
PREPROCESSING_FN = tf.keras.applications.efficientnet.preprocess_input
BB_FN = Efficient_Net_encoder.get_efficientnetv2_backbone
IMG_SHAPE = (512, 512, 3)
 
training_config.initialize_encoder_config(
    image_embedding_dim=IMG_EMB_DIM,
    preprocessing_fn=PREPROCESSING_FN,
    backbone_fn=BB_FN,
    image_shape=IMG_SHAPE,
    do_permute=IMG_EMB_DIM[1] < IMG_EMB_DIM[0],
)
``` 

分析Encoder的流程: 

  1. 调用karas提供的标准efficientnet的预处理函数
  2. 调用定制的efficientnetv2模型
  3. 选取特征层 (scale_factor指定选取哪一层)
  4. 进行reshape整容

## transformer

Transformer_decoder.Decoder

参数: 

```
VOCAB_LEN = len(tokenizer.word_index)
max_length = pickle.load(open("max_length.pkl", "rb"))
MAX_LEN = max_length
N_LAYERS = 6
D_MODEL = 512
D_FF = 2048
N_HEADS = 8
DROPOUT_RATE = 0.1

 
training_config.initialize_transformer_config(
    vocab_len=VOCAB_LEN,
    max_len=MAX_LEN,
    n_transformer_layers=N_LAYERS,
    transformer_d_dff=D_FF,
    transformer_n_heads=N_HEADS,
    image_embedding_dim=IMG_EMB_DIM,
)

def initialize_transformer_config(
    self,
    vocab_len,
    max_len,
    n_transformer_layers,
    transformer_d_dff,
    transformer_n_heads,
    image_embedding_dim,
    rate=0.1,
):
    """This functions initializes the Transformer model as decoder with
    user defined configurations.

    Args:
        vocab_len (int): Total number of words in the input vocabulary
        max_len (int): Maximum length of the string found on the training dataset
        n_transformer_layers (int): Number of layers present in the transformer model
        transformer_d_dff (int): Transformer feed forward upwards projection size
        transformer_n_heads (int): Number of heads present in the transformer model
        image_embedding_dim (int): Total number of dimension the image gets embedded
        dropout_rate (float, optional): Fraction of the input units to drop. Defaults to 0.1.
    """
    self.transformer_config = dict(
        num_layers=n_transformer_layers,
        d_model=image_embedding_dim,
        num_heads=transformer_n_heads,
        dff=transformer_d_dff,
        target_vocab_size=vocab_len,
        max_len=max_len,
        rate=0.1,
    )
``` 

transformer大概的结构: 

  - Embedding层: 将输入的整数编码转换为稠密向量。参数为 `target_vocab_size` 和 `d_model`。
  - Positional Encoding: 用于给嵌入向量添加位置信息。这里有两种类型的位置编码：
    - pos_encoding_1d: 一维的位置编码，用于序列中的位置信息。
    - pos_encoding_2d: 二维的位置编码，可能用于图像或其他需要二维位置信息的情况。
  - Decoder Layer * 6: 解码器包含多个 `DecoderLayer`，每个层负责一部分解码任务。这里创建了一个包含 `num_layers` 个 `DecoderLayer` 的列表。
  - Dropout层: 用于防止过拟合。
  - Dense层: 最终的全连接层，用于将解码器的输出转换为目标词汇的概率分布，并使用softmax激活函数。

对于DecoderLayer的定义: 

````
这段代码定义了一个名为 `DecoderLayer` 的类，它是Transformer架构中的解码器层的一部分。这个类继承自 `tf.keras.layers.Layer`，并且实现了多头注意力机制（Multi-Head Attention）和前馈神经网络（Feed-Forward Network）等关键组件。下面是对这个类的详细分析：

### 1. 初始化 (`__init__` 方法)

#### 参数定义

- **d_model**: 模型的隐藏层维度。
- **num_heads**: 多头注意力机制中的头数。
- **dff**: 前馈神经网络中的中间层维度。
- **max_len**: 序列的最大长度。
- **rate**: Dropout的比率。

#### 层定义

- **mha1**: 第一个多头注意力层，用于处理解码器层内部的自注意力机制。
- **mha2**: 第二个多头注意力层，用于处理解码器层与编码器之间的交互。
- **ffn**: 点对点前馈神经网络（point-wise feed-forward network），用于对解码器层的输出进行变换。
- **layernorm1**, **layernorm2**, **layernorm3**: 三个归一化层，用于在每个子层之后进行归一化处理。
- **dropout1**, **dropout2**, **dropout3**: 三个Dropout层，用于防止过拟合。

### 2. 前向传播 (`call` 方法)

#### 参数

- **x**: 当前解码器层的输入。
- **enc_output**: 编码器的输出，用于与解码器交互。
- **enc_pos**: 编码器的位置编码。
- **dec_pos**: 解码器的位置编码。
- **training**: 是否在训练模式。
- **look_ahead_mask**: 用于掩蔽未来词的mask。
- **padding_mask**: 用于掩蔽填充词的mask。

#### 执行流程

1. **自注意力机制**：
   ```python
   attn1, attn_weights_block1 = self.mha1(
       x, x, x, q_pos=dec_pos, k_pos=dec_pos, mask=look_ahead_mask
   )
   ```
   这里 `mha1` 计算了解码器层内部的自注意力机制，输入 `x` 与自身进行自注意力计算，并使用解码器的位置编码 `dec_pos`。

2. **Dropout层**：
   ```python
   attn1 = self.dropout1(attn1, training=training)
   ```
   添加Dropout层以防止过拟合。

3. **残差连接与归一化**：
   ```python
   out1 = self.layernorm1(attn1 + x)
   ```
   将自注意力机制的输出与输入 `x` 进行残差连接，并通过归一化层进行归一化处理。

4. **编码器-解码器注意力机制**：
   ```python
   attn2, attn_weights_block2 = self.mha2(
       out1, enc_output, enc_output, q_pos=dec_pos, k_pos=enc_pos
   )
   ```
   这里 `mha2` 计算了解码器与编码器之间的注意力机制，`out1` 作为查询向量，`enc_output` 作为键和值向量，并使用解码器和编码器的位置编码。

5. **Dropout层**：
   ```python
   attn2 = self.dropout2(attn2, training=training)
   ```
   添加Dropout层以防止过拟合。

6. **残差连接与归一化**：
   ```python
   out2 = self.layernorm2(attn2 + out1)
   ```
   将编码器-解码器注意力机制的输出与上一步的结果 `out1` 进行残差连接，并通过归一化层进行归一化处理。

7. **前馈神经网络**：
   ```python
   ffn_output = self.ffn(out2)
   ```
   使用前馈神经网络对 `out2` 进行变换。

8. **Dropout层**：
   ```python
   ffn_output = self.dropout3(ffn_output, training=training)
   ```
   添加Dropout层以防止过拟合。

9. **残差连接与归一化**：
   ```python
   out3 = self.layernorm3(ffn_output + out2)
   ```
   将前馈神经网络的输出与 `out2` 进行残差连接，并通过归一化层进行归一化处理。

10. **返回结果**：
    ```python
    return out3, attn_weights_block1, attn_weights_block2
    ```
    返回最终的输出 `out3` 以及两个注意力权重矩阵 `attn_weights_block1` 和 `attn_weights_block2`。

### 总结

这个 `DecoderLayer` 类的结构如下：

1. **自注意力机制**：处理解码器层内部的自注意力。
2. **编码器-解码器注意力机制**：处理解码器与编码器之间的交互。
3. **前馈神经网络**：对解码器层的输出进行变换。
4. **残差连接与归一化**：在每个子层之后进行残差连接，并通过归一化层进行归一化处理。

这种结构使得每个解码器层都能够处理序列中的局部和全局依赖关系，并且通过多层叠加可以更好地捕捉长距离依赖关系。此外，通过使用位置编码，模型能够理解输入序列中的位置信息，这对于序列到序列的任务（如机器翻译）非常重要。
```` 
    
    
    ffn层的定义: 

```
def point_wise_feed_forward_network(d_model, dff):
    return tf.keras.Sequential(
        [tf.keras.layers.Dense(dff, activation="relu"), tf.keras.layers.Dense(d_model)]
    )

``` 

整个模型的结构: 

```
代码中 Decoder 的结构如下：

1. **Embedding 层:** 将输入的词索引转换为词向量。`self.embedding = tf.keras.layers.Embedding(target_vocab_size, d_model)`

2. **位置编码:**  Decoder 使用两种位置编码：
    * 一维位置编码：`self.pos_encoding_1d = positional_encoding_1d(max_len, d_model)`，用于与词嵌入相加。
    * 二维位置编码：`self.pos_encoding_2d = positional_encoding_2d(16, 16, self.d_model)`，传递给 DecoderLayer 中的第二个 MultiHeadAttention 层，作为 key 和 value 的位置信息。 这暗示模型可能处理的是某种二维数据（例如图像特征）。

3. **N 个 DecoderLayer 层:**  `self.dec_layers = [DecoderLayer(d_model, num_heads, dff, max_len, rate) for _ in range(num_layers)]`  这是 Decoder 的核心部分，每个 DecoderLayer 包含：

    * **Masked Multi-Head Attention:** `self.mha1(x, x, x, q_pos=dec_pos, k_pos=dec_pos, mask=look_ahead_mask)`  对 Decoder 的输入进行自注意力计算，并使用 look_ahead_mask 防止模型关注到未来的信息。
    * **Multi-Head Attention:** `self.mha2(out1, enc_output, enc_output, q_pos=dec_pos, k_pos=enc_pos)`  将 Decoder 的输出与 Encoder 的输出进行注意力计算。这里 Decoder 的输出作为 query，Encoder 的输出作为 key 和 value。
    * **前馈神经网络:** `self.ffn(out2)`  一个两层的前馈神经网络，对每个位置的向量进行非线性变换。
    * **层归一化和 Dropout:**  每个子层后都应用了层归一化和 Dropout。

4. **最终输出层:** `self.final_layer = tf.keras.layers.Dense(target_vocab_size, activation="softmax")` 将 Decoder 的最终输出转换为目标词汇表上的概率分布。

总而言之，Decoder 接受目标序列作为输入，并利用自注意力机制和与 Encoder 输出的交叉注意力机制生成输出序列。  二维位置编码的应用表明，该 Decoder 可能用于处理图像等二维数据，并结合 Encoder 提供的特征进行解码。
``` 

模型的特点: 

```
这个模型架构类似于 Transformer 的 Decoder 部分，并针对二维数据进行了调整。 因此，它擅长处理需要理解和生成序列数据的任务，尤其是在序列数据与某种二维空间信息相关联的情况下。

以下是更详细的理解模型特点的方法：

**1.  序列到序列的转换:**  模型的核心是 Decoder 结构，表明它主要用于序列到序列的转换任务。  输入和输出都是序列，例如机器翻译、文本摘要、图像描述生成等。

**2. 二维数据处理:**  `positional_encoding_2d` 函数以及在 DecoderLayer 中对 `enc_pos` (Encoder 位置编码) 的使用，强烈暗示该模型设计用于处理二维数据，例如图像或其他具有空间结构的数据。  这表明模型能够利用输入的二维空间信息来辅助序列生成。

**3.  自注意力机制:**  DecoderLayer 中的 `mha1` 使用了 Masked Multi-Head Attention，这是 Transformer 的关键机制。自注意力机制允许模型在生成序列的每个元素时，关注输入序列的其他相关部分，从而捕捉长距离依赖关系。  Mask 的作用是防止模型在生成当前位置的输出时“偷看”未来的信息。

**4.  交叉注意力机制:** DecoderLayer 中的 `mha2` 使用了 Multi-Head Attention 来处理 Encoder 的输出。这使得 Decoder 能够关注输入序列的不同部分，并将其信息融入到生成的输出序列中。  这对于理解输入序列的含义至关重要。

**5.  位置信息的整合:**  模型使用了两种位置编码方式（一维和二维），这表明位置信息对于模型理解和生成序列非常重要。  一维位置编码捕捉序列中元素的顺序关系，而二维位置编码则捕捉二维空间中的位置关系。

**可能的应用场景:**

* **图像描述生成:**  给定一张图像，生成描述图像内容的文本序列。  Encoder 可以提取图像特征，而 Decoder 则利用这些特征和二维位置信息生成描述文本。
* **从表格数据生成文本:**  将表格数据转换为文本描述。 二维位置编码可以捕捉表格单元格之间的空间关系。
* **视频字幕生成:**  将视频帧序列作为输入，生成描述视频内容的文本序列。

**总结:**

该模型结合了 Transformer Decoder 的强大序列建模能力和处理二维数据的能力。  它擅长于需要理解和生成序列数据的任务，并且特别适合于序列数据与二维空间信息相关联的场景。  理解其核心组件（自注意力、交叉注意力、位置编码）以及它们之间的相互作用，是理解模型特点的关键。
``` 

# 下一步尝试

用通用大模型替换decoder

```
将 Decoder 换成通用大模型的微调层是可行的，而且在很多情况下是更有效的做法。 这背后的逻辑是，通用大模型（例如 BERT、GPT 等）已经在大量的文本数据上进行了预训练，学习到了丰富的语言知识和世界知识。  利用这些预训练的知识，可以更有效地对图像编码进行描述。

以下是将 Decoder 替换为通用大模型微调层的几种具体方法：

1. **将图像编码作为输入:** 将 Encoder 提取的图像特征作为通用大模型的输入。  这可以通过在通用大模型的输入层之前添加一个线性层来实现，将图像特征映射到与通用大模型输入维度相同的空间。  然后，微调通用大模型，使其能够根据图像特征生成所需的描述。

2. **将图像编码与文本嵌入拼接:** 将图像编码和文本嵌入（例如起始标记的嵌入）拼接在一起作为通用大模型的输入。  这种方法可以更好地融合图像信息和文本信息。

3. **使用交叉注意力机制:**  在通用大模型中添加一个交叉注意力层，使其能够关注 Encoder 提取的图像特征。  这种方法类似于 Transformer Decoder 中的交叉注意力机制，可以更有效地利用图像信息。

**可行性分析:**

* **优点:**
    * **利用预训练知识:**  通用大模型已经学习到了丰富的语言知识，可以生成更流畅、更准确的描述。
    * **减少训练数据:**  由于通用大模型已经进行了预训练，微调阶段需要的训练数据量相对较少。
    * **简化模型结构:**  使用通用大模型可以简化模型结构，减少需要调整的超参数。

* **缺点:**
    * **计算资源:**  通用大模型通常参数量较大，需要更多的计算资源进行训练和推理。
    * **过拟合:**  如果训练数据量较少，通用大模型容易出现过拟合现象。

**最佳实践:**

* **选择合适的通用大模型:**  根据任务的需求选择合适的通用大模型。  例如，如果任务是生成图像描述，可以选择一些擅长文本生成的模型，例如 GPT。
* **调整微调策略:**  根据任务的需求调整微调策略。  例如，可以调整学习率、批量大小、训练轮数等。
* **使用合适的评估指标:**  使用合适的评估指标来评估模型的性能。  例如，可以使用 BLEU、CIDEr、ROUGE 等指标来评估图像描述生成的质量。

总而言之，将 Decoder 替换为通用大模型的微调层是一种可行且有效的策略，可以提高图像描述生成的质量。  但是，需要根据具体任务的需求选择合适的通用大模型和微调策略。
``` 

注意: DECIMER-Image_Transformer 是将encoder和decoder一起进行训练

第一步: 用 DECIMER 的encoder, 生成输入, 对LLM进行调优. 

之后: 用新的encoder? 将encoder和LLM联合训练? (数据量要求?)

  - 模仿PaliGemmaPlugin, 写一个图片处理插件
  - 将DECIMER的encoder迁移过来
  - 样例数据: 使用Img2Mol的数据
