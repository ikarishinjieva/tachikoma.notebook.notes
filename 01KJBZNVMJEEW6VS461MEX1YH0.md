---
title: 20241215 - 对于分子式识别的阶段性移交
confluence_page_id: 3343498
created_at: 2024-12-15T04:34:34+00:00
updated_at: 2024-12-15T04:44:11+00:00
---

# 目前的主要思路

用selfies表达式生成 规范的图片, 逐步生成 原子数从低到高 的分子图片

对Qwen2-VL进行微调, 用提示词+规范的图片作为输入, 在输出中输出: 识别出的关键信息 + selfies表达式

为什么要输出 识别出的关键信息: [20241204 - 使用ms-swift对Qwen2-VL进行微调 [11] - 尝试多任务训练]

在训练前需要进行数据选择, 选择对裸模型梯度贡献最大的数据, 参考后续章节

以上实验是 探索Qwen2-VL是否足够识别规范的图片图片

后续的方向: 

  - 如何在有限的成本内, 训练模型识别 原子数更大的规范的图片
  - 如何让模型识别 不规范的图片 (限于论文中的图片, 但不是从selfies表达式生成的图片)
  - 如何让模型学习到 一些selfies生成规律, 比如 "原子数5的分子 是从原子数4的分子如何增加了一个原子生成, 那么原子数22的分子 是从原子数21的分子如何增加一个原子生成"

# 代码所在服务器

```
ssh -p 22199 leixia@gpu888.cn
Dp6bKUih
``` 

# 主要代码

  - 从selfies表达式, 生成图片, 以及生成图片增强
    - /opt/huangyan/ms-swift/data_gen/loop_selfies_and_image.generate_pic.ipynb
  - 从图片, 生成训练用的数据集
    - /opt/huangyan/ms-swift/data_gen/loop_selfies_and_image.mol_desc.ipynb
  - 任务脚本 (选择数据 + 训练 + 评估)
    - /opt/huangyan/ms-swift/run_multi_tasks.sh
    - 选择数据在后面章节交待细节
    - 评估脚本在后面章节交待细节
  - 选择数据的 需要修改的脚本: 
    - /opt/huangyan/ms-swift/swift/llm/sft.py.modified2 细节在后面章节
  - 评估脚本
    - /opt/huangyan/ms-swift/huangyan_test/evaluate_loop_selfies/evaluate_loop_selfies.py 细节在后面章节

# 选择数据

实验: 

  - [20241128 - 使用ms-swift对Qwen2-VL进行微调 [9] - 用梯度贡献来进行数据集的选择]
  - [20241202 - 使用ms-swift对Qwen2-VL进行微调 [10] - 用梯度贡献来进行数据集的选择 [2]]

需要修改sft.py, 所以在做选择数据之前, 需要将 sft.py 更换成 /opt/huangyan/ms-swift/swift/llm/sft.py.modified2. 在训练时需要换回.

主要函数: train_with_gradient_selection

# 评估脚本

/opt/huangyan/ms-swift/huangyan_test/evaluate_loop_selfies/evaluate_loop_selfies.py

脚本用多线程进行并行评估, 结果会输出正确率, 样例输出: 

```
Overall Similarity: 97.69%
Correct: 73 / 75 = 97.33333333333334 %
                                                        Image Path                                                                                                                                                                               Predicted Predicted SMILES                                                                                                                                                                                      GT GT SMILES  Similarity
55  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/317.png    
	{"single_bonds": 2, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 0, "N_count": 3, "branch_count": 0, "atom_count": 4, "selfies": "[N][=N][N][C]"}            N=NNC    
	{"single_bonds": 2, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 0, "N_count": 3, "branch_count": 0, "atom_count": 4, "selfies": "[N][N][=N][C]"}     NN=NC    0.133333
1    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/72.png    
	{"single_bonds": 2, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 0, "N_count": 3, "branch_count": 0, "atom_count": 4, "selfies": "[N][=N][N][C]"}            N=NNC    
	{"single_bonds": 2, "double_bonds": 1, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 0, "N_count": 3, "branch_count": 0, "atom_count": 4, "selfies": "[C][N][=N][N]"}     CN=NN    0.133333
53  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/113.png   
	{"single_bonds": 1, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 4, "selfies": "[C][=N][N][=C]"}           C=NN=C   
	{"single_bonds": 1, "double_bonds": 2, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 0, "N_count": 2, "branch_count": 0, "atom_count": 4, "selfies": "[C][=N][N][=C]"}    C=NN=C    1.000000
52  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/128.png    
	{"single_bonds": 2, "double_bonds": 0, "triple_bonds": 1, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 4, "selfies": "[O][N][C][#C]"}            ONC#C    
	{"single_bonds": 2, "double_bonds": 0, "triple_bonds": 1, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 4, "selfies": "[C][#C][N][O]"}     C#CNO    1.000000
......
``` 

其他失败的实验, 见实验日志
