---
title: 20241016 - 阅读论文*: Img2Mol – accurate SMILES recognition from molecular graphical depictions†
confluence_page_id: 3342683
created_at: 2024-10-16T07:47:24+00:00
updated_at: 2024-10-16T07:48:49+00:00
---

# 主要思路

将 图片 到 SMILES任务, 拆解成 图片到CDDD向量 + CDDD向量到SMILES 两个任务

# 其他有用的信息

  - 论文中用到的数据集可参考

# 其他类似于CDDD向量的技术

```
是的，除了 CDDD 向量之外，还有其他一些类似的技术，用于将分子结构表示成连续的向量，也称为分子嵌入（molecular embeddings）或分子表征（molecular representations）。这些技术的目标都是将离散的分子结构信息转换成连续的向量表示，以便于进行机器学习任务。以下是一些常见的技术：

1. **变分自动编码器 (Variational Autoencoders, VAEs):**  与 CDDD 使用的普通自编码器类似，VAEs 也用于学习分子嵌入。不同之处在于，VAEs 尝试学习一个潜在空间的概率分布，而不是简单的编码和解码。这使得 VAEs 可以生成新的分子结构，并且可以用于分子设计和优化。一些著名的基于 VAE 的分子嵌入方法包括：
    * **Junction Tree VAEs:**  这种方法将分子分解成连接树，然后在连接树上进行编码和解码。
    * **Graph VAEs:**  这种方法直接在分子图上进行操作，可以更好地捕捉分子图的拓扑结构信息。

2. **生成对抗网络 (Generative Adversarial Networks, GANs):** GANs 也可以用于学习分子嵌入和生成新的分子结构。GANs 包含两个网络：生成器和判别器。生成器尝试生成新的分子结构，而判别器尝试区分生成的分子和真实的分子。通过对抗训练，生成器可以学习到生成逼真的分子结构的能力。一些基于 GANs 的分子嵌入方法包括：
    * **MolGAN:**  一种用于生成小分子的 GAN 模型。
    * **GraphGAN:**  一种用于生成图结构数据的 GAN 模型，可以应用于分子生成。

3. **图神经网络 (Graph Neural Networks, GNNs):** GNNs 是一种专门用于处理图结构数据的深度学习模型。GNNs 可以直接在分子图上进行操作，学习分子嵌入，并用于分子性质预测、虚拟筛选等任务。一些常用的 GNN 模型包括：
    * **图卷积网络 (Graph Convolutional Networks, GCNs):**  一种用于学习图节点表示的卷积神经网络。
    * **图注意力网络 (Graph Attention Networks, GATs):**  一种结合注意力机制的 GNN 模型，可以更好地捕捉图节点之间的关系。
    * **消息传递神经网络 (Message Passing Neural Networks, MPNNs):**  一种通用的 GNN 框架，可以涵盖多种 GNN 模型。

4. **其他方法:**
    * **Extended-connectivity fingerprints (ECFPs):**  一种基于分子图的指纹，可以转换成固定长度的位向量，也可以视为一种分子嵌入。
    * **Morgan fingerprints (circular fingerprints):** 与 ECFPs 类似，也是一种常用的分子指纹。

这些技术各有优缺点，选择哪种技术取决于具体的应用场景和需求。例如，如果需要生成新的分子结构，则可以选择 VAEs 或 GANs；如果需要进行分子性质预测，则可以选择 GNNs 或传统的分子描述符。
```
