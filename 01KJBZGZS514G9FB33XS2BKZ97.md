---
title: 20241016 - 测试Img2Mol项目
confluence_page_id: 3342687
created_at: 2024-10-16T15:58:28+00:00
updated_at: 2024-10-18T05:28:56+00:00
---

# 安装使用

```
GIT_TRACE=1 GIT_CURL_VERBOSE=1 HTTP_PROXY=http://10.186.16.136:7890 HTTPS_PROXY=http://10.186.16.136:7890 git clone https://github.com/bayer-science-for-a-better-life/Img2Mol.git
 
source ~/huangyan_init_env.sh #初始化conda
 
cd /data/huangyan/Img2Mol/
HTTP_PROXY=http://10.186.16.136:7890 HTTPS_PROXY=http://10.186.16.136:7890 conda env create -f environment.yml
 
conda activate img2mol
pip install . -i https://pypi.tuna.tsinghua.edu.cn/simple
``` 

按说明, 手工下载: <https://drive.google.com/file/d/1pk21r4Zzb9ZJkszJwP9SObTlfTaRMMtF/view> (model文件), google会有一个防病毒确认, 导致无法使用脚本

运行 example_inference.ipynb 会报错: 

```
Traceback (most recent call last):
  File "1.py", line 11, in <module>
    from img2mol.inference import *
  File "/data/huangyan/Img2Mol/img2mol/inference.py", line 26, in <module>
    from rdkit import Chem
  File "/data/huangyan/anaconda3/envs/img2mol/lib/python3.8/site-packages/rdkit/Chem/__init__.py", line 23, in <module>
    from rdkit.Chem.rdmolfiles import *
ImportError: /data/huangyan/anaconda3/envs/img2mol/lib/python3.8/site-packages/rdkit/Chem/../../../../libRDKitmaeparser.so.1: undefined symbol: _ZN5boost9iostreams4zlib8deflatedE

``` 

更新rdkit版本可修整: 

```
conda remove rdkit
conda install -c conda-forge rdkit
``` 

使用远端CDDD模型失败, 报错: 

```
Traceback (most recent call last):
  File "1.py", line 35, in <module>
    res = img2mol(filepath="examples/digital_example1.png", cddd_server=cddd_server)
  File "/data/huangyan/Img2Mol/img2mol/inference.py", line 146, in __call__
    smiles = cddd_server.cddd_to_smiles(cddd.tolist())
  File "/data/huangyan/Img2Mol/img2mol/cddd_server.py", line 51, in cddd_to_smiles
    return json.loads(response.content.decode("utf-8"))
  File "/data/huangyan/anaconda3/envs/img2mol/lib/python3.8/json/__init__.py", line 357, in loads
    return _default_decoder.decode(s)
  File "/data/huangyan/anaconda3/envs/img2mol/lib/python3.8/json/decoder.py", line 337, in decode
    obj, end = self.raw_decode(s, idx=_w(s, 0).end())
  File "/data/huangyan/anaconda3/envs/img2mol/lib/python3.8/json/decoder.py", line 355, in raw_decode
    raise JSONDecodeError("Expecting value", s, err.value) from None
json.decoder.JSONDecodeError: Expecting value: line 1 column 1 (char 0)
``` 

使用本地的CDDD模型, 得重新搭建conda环境:

将environment.local-cddd.yml 中改个环境名: img2mol-cddd

```
HTTP_PROXY=http://10.186.16.136:7890 HTTPS_PROXY=http://10.186.16.136:7890  conda env create -f environment.local-cddd.yml
 
conda activate img2mol-cddd
``` 

下载本地的CDDD模型: [https://drive.usercontent.google.com/download?id=1oyknOulq_j0w9kzOKKIHdTLo5HphT99h&export=download&authuser=0](<https://drive.usercontent.google.com/download?id=1oyknOulq_j0w9kzOKKIHdTLo5HphT99h&export=download&authuser=0>)

解压下载的模型到: /data/huangyan/anaconda3/envs/img2mol-cddd/lib/python3.6/site-packages/cddd/data

运行: example_inference_local_cddd.ipynb

可执行成功

# 代码解读 - 推理Img2MolInference

  - __call__
    - read_image_to_tensor: 将image进行变换 (进行若干次), 将变换生成的若干图像合并, 转换成向量
      - transform_image: 图像变换: 进行了各种数据增强变换，包括旋转、亮度调整、剪切、对比度增强和大小调整，最终将其转换为 PyTorch 张量
        - 图像和张量的关系: 

```
PIL 图像通常以 (H x W x C) 的格式存储像素数据，其中 H 代表高度，W 代表宽度，C 代表颜色通道数（例如，RGB 图像的 C 为 3）。而 PyTorch 张量通常使用 (C x H x W) 的格式。因此，转换过程中会将颜色通道维度移动到最前面。这个过程只是改变了数据的排列方式，并没有改变图像本身的信息。
```

      - 合并 (增加一维, 在这一维上合并数据):
        - torch.unsqueeze(..., 0): 在变换后的图像张量的第 0 维（也就是批量维度）添加一个维度。这将把形状从 (C, H, W) 转换为 (1, C, H, W)。

        - [... for _ in range(repeats)]: 使用列表推导式将上述操作重复 repeats 次，创建一个包含 repeats 个相同图像张量的列表。

        - torch.cat(..., dim=0): 将列表中的所有张量沿着第 0 维（批量维度）连接起来，创建一个形状为 (repeats, C, H, W) 的张量。这意味着创建了一个包含 repeats 个相同图像的批量。

    - 将张量 (多个变换后的图片拼接的向量), 给到模型, 由模型生成 多个cddd向量
    - 对多个cddd向量取中位数 median
    - 查询另一个模型 cddd_inference_model, 将cddd转换成smiles表达式

    - 通过 Chem 库, 将生成的cddd转换成smiles, 在转换成cddd, 来验证其生成的正确性

# 代码解读 - 模型结构 - Img2MolPlModel

模型使用 "卷积器 + 分类器" 模式, 这种模式适用于: 

```
卷积器 + 分类器这种模式并非只适用于图像处理，它可以应用于其他领域，只要数据具有某种局部结构或平移不变性。

图像处理的适用性:

图像具有很强的局部相关性和平移不变性。这意味着图像中相邻像素之间通常存在关联，并且图像中的物体无论出现在哪个位置，都应该被识别为同一个物体。卷积操作正好可以利用这些特性，通过局部感受野提取特征，并且权重共享机制保证了平移不变性。
``` 

每个卷积层用四个参数 [输出通道数, 卷积核大小, 步幅, 填充] 表示: 

```
让我们详细解释一下卷积层中四个参数的作用，并用一些例子说明：

1. **输出通道数 (Output Channels):** 也称为卷积核的数量或滤波器的数量。这个参数决定了卷积操作后输出特征图的深度（通道数）。每个卷积核都会学习不同的特征，因此输出通道数越多，模型能够学习的特征就越丰富。

   * **例子:** 假设输入图像的通道数为 3（RGB 图像），输出通道数设置为 64，那么卷积层会包含 64 个卷积核。每个卷积核都会对输入图像的三个通道进行卷积操作，并生成一个输出特征图。最终，64 个卷积核会生成 64 个特征图，堆叠在一起形成深度为 64 的输出特征图。

2. **卷积核大小 (Kernel Size):**  定义了卷积核的尺寸，通常是一个奇数（例如 3x3, 5x5, 7x7）。卷积核会在输入图像上滑动，并与对应的局部区域进行卷积运算。卷积核越大，感受野就越大，能够捕捉更大范围的特征。

   * **例子:**  一个 3x3 的卷积核会考虑输入图像中 3x3 的像素区域，而一个 5x5 的卷积核会考虑 5x5 的像素区域。更大的卷积核可以捕捉更大范围的特征，例如纹理或形状，但也会增加计算量。

3. **步幅 (Stride):**  控制卷积核在输入图像上滑动的步长。步幅越大，卷积核滑动的步长就越大，输出特征图的尺寸就越小。

   * **例子:**  步幅为 1 表示卷积核每次滑动一个像素，步幅为 2 表示卷积核每次滑动两个像素。更大的步幅可以降低计算量和输出特征图的尺寸，但也可能会丢失一些细节信息。

4. **填充 (Padding):**  在输入图像的边缘添加额外的像素。填充通常用于保持输出特征图的尺寸与输入图像相同，或者增加卷积核对边缘像素的覆盖次数。

   * **例子:**  假设输入图像的大小为 5x5，卷积核大小为 3x3，步幅为 1。如果不使用填充，输出特征图的大小将为 3x3。如果使用填充 1，则会在输入图像的四周添加一层像素，使得输入图像的大小变为 7x7，输出特征图的大小将保持为 5x5。

**总结:**

这四个参数共同决定了卷积层的行为和输出特征图的特性。选择合适的参数值对于模型的性能至关重要。通常需要根据具体的任务和数据集进行调整。
``` 

卷积器模仿了VGG-11.

VGG-族的结构: (A为VGG-11)

![image2024-10-18 0:7:48.png](/assets/01KJBZGZS514G9FB33XS2BKZ97/image2024-10-18%200%3A7%3A48.png)

论文的补充信息ESI: <https://www.rsc.org/suppdata/d1/sc/d1sc01839f/d1sc01839f1.pdf>

其中给出了模型结构图和训练参数: 

![image2024-10-18 0:32:21.png](/assets/01KJBZGZS514G9FB33XS2BKZ97/image2024-10-18%200%3A32%3A21.png)

层数和每层的参数, 为什么这样选择, 并没有明确的方法论描述, 可能是出于实验的结果

# Img2Mol 与 VGG11的比较

卷积层: 

```
VGG11 和 Img2Mol 的卷积部分虽然都采用了卷积神经网络架构，但在层数、卷积核大小、步幅、通道数等方面存在一些差异。  这些差异主要源于它们的目标任务和设计理念的不同。

**VGG11 的卷积部分:**

VGG11 的卷积部分由 8 个卷积层组成，分为 5 个卷积块：

1. **块 1:** 两个 `Conv3-64` (卷积核大小 3x3，输出通道数 64)
2. **块 2:** 两个 `Conv3-128`
3. **块 3:** 三个 `Conv3-256`
4. **块 4:** 三个 `Conv3-512`
5. **块 5:** 三个 `Conv3-512`

每个卷积块后都跟着一个 MaxPooling 层，用于降维。  VGG11 的主要特点是使用小的卷积核 (3x3) 和较深的网络结构。

**Img2Mol 的卷积部分:**

Img2Mol 的卷积部分也由 8 个卷积层组成，分为 5 个卷积块，但结构和参数与 VGG11 不同：

1. **块 1:**  `Conv(128, 7, 3, 4)`, `Conv(256, 5, 1, 1)`, `Conv(384, 5, 1, 1)`
2. **块 2:**  `Conv(384, 3, 1, 1)`, `Conv(384, 3, 1, 1)`
3. **块 3:**  `Conv(512, 3, 1, 1)`, `Conv(512, 3, 1, 1)`, `Conv(512, 3, 1, 1)`

其中 `Conv(out_channels, kernel_size, stride, padding)` 表示一个卷积层。  Img2Mol 的卷积部分使用了不同大小的卷积核 (7x7, 5x5, 3x3) 和不同的步幅/填充。

**主要区别总结:**

| 特性 | VGG11 | Img2Mol |
| --- | --- | --- |
| 卷积核大小 | 统一 3x3 | 7x7, 5x5, 3x3 |
| 步幅 | 1 (卷积), 2 (池化) | 3, 1 (卷积), 2 (池化) |
| 填充 | 1 (卷积), 0 (池化) | 4, 1 (卷积), 0 (池化) |
| 通道数变化 | 64 -> 128 -> 256 -> 512 -> 512 | 128 -> 256 -> 384 -> 512 |
| 输入通道数 | 3 (RGB) | 1 (灰度) |

**解释差异的原因:**

* **目标任务:** VGG11 用于图像分类，需要学习更通用的图像特征。  而 Img2Mol 用于分子图像识别，可能需要学习更特定于分子结构的特征。
* **输入数据:** VGG11 的输入是彩色图像 (RGB)，而 Img2Mol 的输入是灰度图像。
* **设计理念:** VGG11 强调使用小的卷积核和更深的网络结构。  而 Img2Mol 的设计可能更多地基于实验和调优的结果。

总而言之，VGG11 和 Img2Mol 的卷积部分的设计差异反映了它们的目标任务和设计理念的不同。  VGG11 旨在学习通用的图像特征，而 Img2Mol 旨在学习特定于分子结构的特征。
``` 

分类器层:

```
Img2Mol 和 VGG11 的分类器部分存在显著差异，这主要是因为它们的目标任务不同：VGG11 主要用于图像分类，而 Img2Mol 用于从分子图像预测 SMILES 字符串（可以看作是一个回归任务）。

**VGG11 的分类器:**

VGG11 的分类器通常由三个全连接层组成：

1. `fc1`:  输入维度取决于前面的卷积层和池化层的输出，通常是一个很大的值。 输出维度通常是 4096。  使用 ReLU 激活函数。
2. `fc2`:  输入维度 4096，输出维度 4096。 使用 ReLU 激活函数。
3. `fc3`:  输入维度 4096，输出维度为类别数量（例如 ImageNet 数据集有 1000 个类别）。 使用 Softmax 激活函数（用于概率预测）。

**Img2Mol 的分类器:**

Img2Mol 的分类器也由三个全连接层组成，但结构和参数与 VGG11 不同：

1. `Linear(512 * 9 * 9, 4096)`: 输入维度是 41472 (512 * 9 * 9)，这是将最后一个卷积层的输出展平后得到的向量维度。 输出维度是 4096。 使用 ReLU 激活函数。
2. `Linear(4096, 4096)`: 输入维度 4096，输出维度 4096。 使用 ReLU 激活函数。
3. `Linear(4096, 512)`: 输入维度 4096，输出维度 512。 使用 Tanh 激活函数。  这个 512 维的输出向量就是 CDDD 向量，它会被送入 CDDD 解码器生成 SMILES 字符串。

**主要区别总结:**

| 特性 | VGG11 | Img2Mol |
| --- | --- | --- |
| 目标任务 | 图像分类 | SMILES 字符串预测 (回归) |
| 分类器输出维度 | 类别数量 (例如 1000) | 512 (CDDD 向量) |
| 最后一个激活函数 | Softmax | Tanh |
| 输出解释 | 每个类别的概率 | CDDD 向量，用于生成 SMILES |

**解释差异的原因:**

* **输出维度:** VGG11 的输出维度是类别数量，因为它需要预测输入图像属于哪个类别。  而 Img2Mol 的输出维度是 512，因为它需要预测一个 512 维的 CDDD 向量。
* **激活函数:** VGG11 使用 Softmax 激活函数，将输出转换为概率分布。  而 Img2Mol 使用 Tanh 激活函数，将 CDDD 向量的每个元素限制在 -1 到 1 之间。
* **输出解释:** VGG11 的输出是每个类别的概率，可以直接用于分类。  而 Img2Mol 的输出是一个 CDDD 向量，需要进一步解码才能得到最终的 SMILES 字符串。

总而言之，VGG11 和 Img2Mol 的分类器部分的设计差异反映了它们的目标任务的不同。  VGG11 的分类器用于图像分类，而 Img2Mol 的分类器用于预测 CDDD 向量，最终生成 SMILES 字符串。
``` 

# 如何构建数据集

Img2Mol的目标是对药物分子进行识别, 所以数据构建的方向也是偏重药物分子: 

```
让我们逐步分析 Img2Mol 论文中构建数据集的方法，并解释每一步背后的考虑：

1. **数据来源 (ChEMBL25 和 PubChem):**

* **考虑:** 选择 ChEMBL 和 PubChem 是因为它们是大型公开的化学数据库，包含丰富的分子信息，可以提供多样化的训练样本。  ChEMBL 侧重于药物相关的分子，而 PubChem 包含更广泛的化学物质，两者结合可以覆盖更广阔的化学空间。

2. **SMILES 获取和预处理 (使用 RDKit):**

* **考虑:**  SMILES 字符串是分子结构的文本表示形式，易于处理和存储。 RDKit 是一个功能强大的化学信息学工具包，可以用来处理 SMILES 字符串，并进行各种化学计算和转换。  使用 RDKit 获取 canonical SMILES 可以确保每个分子只有一个唯一的表示形式，避免重复数据。

3. **去除立体化学信息:**

* **考虑:**  立体化学信息（例如手性）在二维图像中难以准确表示，并且可能会增加模型的复杂度。  去除立体化学信息可以简化任务，并使模型更专注于学习分子的连接性和原子类型。  当然，这也意味着模型无法预测分子的立体化学信息。

4. **去除重复项:**

* **考虑:**  重复的样本会使模型偏向于某些特定的分子，降低模型的泛化能力。  去除重复项可以确保数据集中的每个分子都是独特的。

5. **分子过滤 (分子量、重原子数、logP):**

* **考虑:**  过滤分子是为了去除一些不符合要求的分子，例如过大或过小的分子、含有特殊元素的分子等。  这些分子可能会影响模型的训练效果，或者在实际应用中不常见。  文中提到的过滤标准（分子量 12-600 Da，重原子数 > 3，logP -7 到 5）是为了选择类药分子，因为 Img2Mol 的主要应用场景是药物发现。

6. **去除盐和保留最大片段:**

* **考虑:**  许多药物分子以盐的形式存在，但盐的结构对分子的核心性质影响较小。  去除盐可以简化分子结构，并使模型更专注于学习分子的核心骨架。  保留最大片段的目的是处理一些复杂的分子，例如含有溶剂或其他杂质的分子。

7. **数据集划分 (训练集、验证集、测试集):**

* **考虑:**  将数据集划分为训练集、验证集和测试集是为了评估模型的泛化能力。  训练集用于训练模型，验证集用于调整模型的超参数，测试集用于评估最终模型的性能。  使用聚类方法划分数据集可以确保训练集、验证集和测试集中的分子分布相似，避免数据偏差。

8. **分子图像生成 (RDKit, OEChem TK, Indigo):**

* **考虑:**  使用不同的化学信息学库和设置生成分子图像可以增加数据的多样性，并提高模型的鲁棒性。  不同的库和设置会生成不同风格的分子图像，例如不同的键的粗细、原子的大小、背景颜色等。  这可以模拟真实世界中分子图像的多样性，并使模型更适应不同的图像风格。

9. **分辨率变化和缩放:**

* **考虑:**  改变图像的分辨率可以模拟真实世界中图像质量的变化，例如扫描的图像、低分辨率的图像等。  将图像缩放至固定大小是为了适应模型的输入要求。

10. **数据增强 (随机设置变体):**

* **考虑:**  数据增强是一种常用的技术，可以增加训练数据量，并提高模型的泛化能力。  通过随机改变图像的各种设置，可以生成大量的不同版本的分子图像，从而增加训练数据的多样性。

总而言之，Img2Mol 数据集的构建方法经过精心设计，考虑了数据来源、数据质量、数据多样性、模型的应用场景以及计算效率等多个方面。  这些步骤旨在创建一个高质量、多样化且具有代表性的数据集，以训练一个鲁棒且泛化能力强的模型。
``` 

考虑从 ChemBERTa, MolT5 等模型的训练中, 获取数据集

# 考虑将图像训练+文本训练, 提升效果

可能的方案: (方案一 应当优先尝试)

```
你想要使用多模态模型和多任务学习来增强 Img2Mol 的性能，这是一个很有潜力的方向。以下是一些方案推荐，结合了多模态学习和多任务学习的思路：

**方案一：基于特征融合的多模态模型**

1. **图像分支:** 使用原有的 Img2Mol 架构（CNN）提取图像特征。
2. **文本分支:** 使用预训练的语言模型（如 BERT、SciBERT 或其他化学领域相关的语言模型）提取文本特征 (例如 IUPAC 名称)。
3. **特征融合:** 将图像特征和文本特征拼接或通过其他方式融合，例如：
    * **拼接:** 直接将两个特征向量连接在一起。
    * **注意力机制:**  使用注意力机制动态地融合特征，根据文本信息关注图像的不同区域。
4. **输出层:**  将融合后的特征输入到一个全连接层，输出 CDDD 向量。
5. **解码器:** 使用预训练的 CDDD 解码器将 CDDD 向量转换为 SMILES 字符串。
6. **多任务学习:**  添加一个辅助任务，例如预测分子量、logP 等，以帮助模型学习更丰富的特征。

**方案二：基于图神经网络 (GNN) 的多模态模型**

1. **分子图构建:** 将分子结构表示为图，原子作为节点，键作为边。
2. **图像特征提取:** 使用 CNN 提取图像特征，并将特征分配给对应的原子节点。
3. **文本特征提取:** 使用预训练的语言模型提取文本特征，并将特征分配给对应的原子节点。
4. **GNN 模型:** 使用 GNN 模型（如 GCN、GAT 等）学习节点和边的表示。
5. **输出层:** 将 GNN 的输出转换为 CDDD 向量或直接预测 SMILES 字符串。
6. **多任务学习:**  可以添加预测分子性质的辅助任务。

**方案三：端到端的多任务学习模型**

1. **共享编码器:** 使用一个 CNN 编码器同时处理图像和文本信息（文本信息可以转换为图像，例如使用 one-hot 编码）。
2. **多任务输出:**  设置多个输出层，分别用于预测 SMILES 字符串、分子量、logP 等。
3. **损失函数:**  使用多个损失函数的加权和作为总损失函数。

**选择方案的建议:**

* **方案一**相对简单易实现，适合作为初始尝试。
* **方案二**更能捕捉分子结构信息，但需要构建分子图，实现较为复杂。
* **方案三**更具挑战性，但有可能获得更好的性能。

**一些额外的建议:**

* **预训练:**  尽可能使用预训练的模型，例如图像领域的 ResNet、NLP 领域的 BERT 等。
* **数据增强:**  使用数据增强技术来扩充数据集，提高模型的泛化能力。
* **超参数调优:**  仔细调整模型的超参数，例如学习率、批大小、正则化强度等。
* **评估指标:**  使用合适的评估指标来评估模型的性能，例如准确率、Tanimoto 相似性等。

希望以上方案能帮助你改进模型。  记住，选择哪种方案最终取决于你的具体需求和资源。  建议你从简单的方案开始，逐步尝试更复杂的方案。
```
