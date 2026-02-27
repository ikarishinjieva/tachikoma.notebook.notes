---
title: 20241219 - 使用ms-swift对Qwen2-VL进行微调 [13] - 递增式训练[2] - 找出 "学习逻辑" 的变量
confluence_page_id: 3343523
created_at: 2024-12-19T02:22:49+00:00
updated_at: 2024-12-24T15:40:45+00:00
---

# 前继

[20241216 - 使用ms-swift对Qwen2-VL进行微调 [13] - 递增式训练]

需要找到提高"学习逻辑"的变量, 待测试: 

  - 推理的步骤长短: 
    - 两步推理训练: 1-4; 5
    - 多步推理训练: 1-3; 4; 5
    - 往后推理: 1-4; 5; 6
  - 混合数据量的多少
    - 混合少数据量: 1-4 (500); 5(500) + 1-4(100)
    - 混合多数据量: 1-4 (500); 5(500) + 1-4(200)
  - 反复训练: 
    - 重复 1-4, 5 这种训练模式

# 基准数据

两阶段递增训练, 数据量: 1-4 (500); 5(500) + 1-4(100) 

  - 解释: 第一次训练, 长度1-4的数据500个; 第二次训练, 长度1-4的数据100个, 加上长度5的数据500个
  - 基准: 
    - 长度4的校验数据: 正确率: 66 / 75 = 88.0 %.
    - 长度5的校验数据: 正确率: 270 / 300 = 90.0 %.
    - 长度6的校验数据: 正确率: 28%

# 混合多数据量: 1-4 (500); 5(500) + 1-4(200)

增加阶段2的训练数据 长度1-4.

  - 长度4的校验数据: 正确率: 74 / 75 = 98.66666666666667 %
  - 长度5的校验数据: 正确率: 263 / 300 = 87.66666666666667 %
  - 长度6的校验数据: 正确率: 48 / 300 = 16.0 %

过分强调了长度4, 导致长度6的推理能力下降

# 增多推理步骤

脚本: [12.sh](/assets/01KJBZP6X3BNGMNB5KJYX0X60Q/12.sh)

步骤:

  - 长度2的数据, epoch=2
  - 长度2的数据, 长度3的数据, epoch=4
  - 长度2的数据, 长度3的数据, 长度4的数据(200个), epoch=6
  - 评估: 长度4
    - 正确率: 71 / 75 = 94.66666666666667 %
  - 评估: 长度5
    - 正确率: 119 / 300 = 39.666666666666664 %
    - 大部分错误case是 原子数错误
  - 长度2的数据, 长度3的数据, 长度4的数据(200个), 长度5的数据(250个), epoch=6
  - 评估: 长度4
    - 正确率: 72 / 75 = 96.0 %
  - 评估: 长度5
    - 正确率: 269 / 300 = 89.66666666666666 %
  - 评估: 长度6
    - 正确率: 114 / 300 = 38.0 %
    - 正确的case中, 原子数预测大部分是正确的.
    - 模型具有推理能力
  - 评估: 长度7
    - 正确率: 29 / 300 = 9.666666666666666 %
    - 即使是正确的case, 原子数预测错误概率也很高, 只是分子结构恰好同型. 
    - 模型丢失了推理能力.

# 增强数据间的"推理性"

[loop_selfies_and_image.generate_chain.ipynb](/assets/01KJBZP6X3BNGMNB5KJYX0X60Q/loop_selfies_and_image.generate_chain.ipynb)

用以上脚本, 按"生成链"生成数据, (生成链: 原子数1的分子 + 原子 = 原子数2的分子)

数据组织上, 将同一个生成链的分子都包括在训练数据中

使用100个生成链的数据训练 (原子数1包括100个分子, 原子数2包括100个分子, ...)

训练脚本: [2.sh](/assets/01KJBZP6X3BNGMNB5KJYX0X60Q/2.sh)

评估数据 (没有从生成链中选取, 而是从之前的穷举selfies的数据中选取):

  - 评估长度4的数据: 正确率: 39 / 75 = 52.0 %
  - 评估长度5的数据: 正确率: 133 / 300 = 44.333333333333336 %
  - 评估长度6的数据: 正确率: 46 / 300 = 15.333333333333332 %

增加生成链的数量, 使用200个生成链: 

  - 评估长度4的数据: 正确率: 41 / 75 = 54.666666666666664 %
  - 评估长度5的数据: 正确率: 156 / 300 = 52.0 %
  - 评估长度6的数据: 正确率: 53 / 300 = 17.666666666666668 %

提升不大

扩大epoch一倍, 每阶段从2个epoch调整为4个epoch

  - 评估长度4的数据: 正确率: 46 / 75 = 61.33333333333333 %
  - 评估长度5的数据: 正确率: 163 / 300 = 54.333333333333336 %
  - 评估长度6的数据: 正确率: 37 / 300 = 12.333333333333334 %

长度4的识别率升高, 长度5没变化, 对长度6的推理能力下降, 

为什么使用了长度4/5训练, 但对长度4/5的识别率不好, 几种猜测: 

  1. 长度4/5的训练数据不够
  2. 校验数据 并不是 生成链方式做出的, 而是使用原本穷举的方式做出的. 两种数据有偏差 (可以尝试用生成链生成的校验集进行 校验)
  3. 原来的训练中, 数据是经过梯度筛选的, 生成链的数据没有经过梯度筛选, 学习效率并不高
  4. 目前使用的数据, 在长度1-4的数据量比原来的大 (原来是1-4总共500, 其中还包括增强数据), 并且长度1-4的数据有大量重复, 导致学到了无用的东西过多. 

增加生成链的数量, 使用500个生成链, 每阶段4个epoch: 

  - 评估长度4的数据: 正确率: 57 / 75 = 76.0 %
  - 评估长度5的数据: 正确率: 221 / 300 = 73.66666666666667 %
  - 评估长度6的数据: 正确率: 42 / 300 = 14.000000000000002 %

  1. 长度4/5的准确率提升
  2. 长度6的推理能力未见明显提升

增加生成链的数量, 使用500个生成链, 每阶段2个epoch: 

  - 评估长度4的数据: 正确率: 55 / 75 = 73.33333333333333 %
  - 评估长度5的数据: 正确率: 209 / 300 = 69.66666666666667 %
  - 评估长度6的数据: 正确率: 89 / 300 = 29.666666666666668 %

增加生成链的数量, 使用500个生成链, 每阶段1个epoch: 

  - 评估长度4的数据: 正确率: 45 / 75 = 60.0 %
  - 评估长度5的数据: 正确率: 179 / 300 = 59.66666666666667 %
  - 评估长度6的数据: 正确率: 83 / 300 = 27.666666666666668 %

# 训练两轮:

(需要从epoch改成steps)

  - 训练长度1, steps=40
  - 训练长度1+长度2, steps=140
  - 训练 长度1(25) + 长度2 + 长度3, steps=240
  - 训练 长度1(25) + 长度2(25) + 长度3 + 长度4, steps=340
  - 训练 长度1(25) + 长度2(25) + 长度3(25) + 长度4 + 长度5, steps=440
  - 评估长度4
    - 37 / 75 = 49.333333333333336 %
  - 评估长度5
    - 167 / 300 = 55.666666666666664 %
  - 评估长度6
    - 58 / 300 = 19.333333333333332 %
  - 训练长度1, steps=540
  - 训练长度1+长度2, steps=640
  - 训练 长度1(25) + 长度2 + 长度3, steps=740
  - 训练 长度1(25) + 长度2(25) + 长度3 + 长度4, steps=840
  - 训练 长度1(25) + 长度2(25) + 长度3(25) + 长度4 + 长度5, steps=940
  - 评估长度4
    - 42 / 75 = 56.00000000000001 %
  - 评估长度5
    - 187 / 300 = 62.33333333333333 %
  - 评估长度6
    - 72 / 300 = 24.0 %

调大steps, 梯度累计是16, 那么一个step能跑16个数据, 每一个长度的数据将近1000, 就是接近60个steps. 所以调整step, 让每一套数据是1个epoch

  - 训练长度1, steps=60
  - 训练长度1+长度2, steps=180
  - 训练 长度1(25) + 长度2 + 长度3, steps=400
  - 训练 长度1(25) + 长度2(25) + 长度3 + 长度4, steps=520
  - 训练 长度1(25) + 长度2(25) + 长度3(25) + 长度4 + 长度5, steps=640
  - 评估长度4: 38 / 75 = 50.66666666666667 %
  - 评估长度5: 135 / 300 = 45.0 %
  - 评估长度6: 48 / 300 = 16.0 %
  - 训练长度1, steps=700
  - 训练长度1+长度2, steps=820
  - 训练 长度1(25) + 长度2 + 长度3, steps=940
  - 训练 长度1(25) + 长度2(25) + 长度3 + 长度4, steps=1060
  - 训练 长度1(25) + 长度2(25) + 长度3(25) + 长度4 + 长度5, steps=1180
  - 评估长度4: 50 / 75 = 66.66666666666666 %
  - 评估长度5: 178 / 300 = 59.333333333333336 %
  - 评估长度6: 57 / 300 = 19.0 %

# 更换提示词: 

原提示词: 

```
Analyze the molecular structure in the provided image. Generate a JSON object containing the following information:

*   **atom_count**: The total number of atoms in the molecule.
*   **single_bonds_count**: The number of single bonds in the molecule.
*   **double_bonds_count**: The number of double bonds in the molecule.
*   **triple_bonds_count**: The number of triple bonds in the molecule.
*   **ring_count**: The number of rings in the molecule.
*   **C_count**: The number of carbon (C) atoms in the molecule.
*   **O_count**: The number of oxygen (O) atoms in the molecule.
*   **N_count**: The number of nitrogen (N) atoms in the molecule.
*   **branch_count**: The number of branches in the molecule (a branch is defined as a non-linear chain of atoms).
*   **selfies**: A valid SELFIES (Simplified Molecular-Input Line-Entry System) representation of the molecule. Ensure the generated SELFIES string is syntactically correct and accurately represents the molecular structure.

**Important:** Pay close attention to the rules of SELFIES notation.

**Examples:**

*   **Correct SELFIES:**
    *   Methane: `[C]`
    *   Ethane: `[C][C]`
    *   Ethene: `[C][=C]`
    *   Ethyne: `[C][#C]`
    *   Propane: `[C][C][C]`
    *   Butane: `[C][C][C][C]`
    *   Isobutane: `[C][C]([C])[C]`
    *   Cyclohexane: `[C]1[C][C][C][C][C]1`
    *   Ethanol: `[C][C][O]`
    *   Ammonia: `[N]`
    *   Water: `[O]`
    *   Carbon Dioxide: `[O][=C]=[O]`
*   **Incorrect SELFIES (Do not generate these):**
    *   `[CC][][C][][][#CC]` (Incorrect bond representation)
    *   `[NN][][][][O]` (Incorrect bond representation)
    *   `[ "[ "[ "[CC][==N]` (Incorrect syntax)
    *   `[CC][##C][CCCCC]` (Incorrect bond representation)
    *   `[OOOC][=CCC][][=NNNNN]` (Incorrect bond representation and syntax)
    *   `[C]1[C][C][C][C][C]` (Missing closing ring bond)
    *   `[C][C][C][C][O]1` (Incorrect ring bond placement)

**Rules:**
    *   All atoms are enclosed in square brackets, e.g., `[C]`, `[O]`, `[N]`.
    *   All bonds are represented by symbols: `[=]` for double bond, `[#]` for triple bond, and no symbol for single bond.
    *   Branches are represented by parentheses, e.g., `[C]([C])[C]`.
    *   Rings are represented by digits, e.g., `[1]`, `[2]`, and the corresponding opening and closing ring bonds.
    *   Ensure that the number of bonds for each atom is correct based on its valency.

Provide the output in a single, well-formatted JSON object.

Image:
<image>

Output example:
{"single_bonds": 0, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 0, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 1, "selfies": "[N]"}
``` 

新提示词 (更强调从小分子进行推理): 

```
Analyze the molecular structure in the provided image and infer its SELFIES (Simplified Molecular Input Line Entry System) representation by constructing it incrementally from smaller, simpler molecular substructures. Generate a JSON object containing the following information:

*   **atom_count**: The total number of atoms in the molecule.
*   **single_bonds_count**: The number of single bonds in the molecule.
*   **double_bonds_count**: The number of double bonds in the molecule.
*   **triple_bonds_count**: The number of triple bonds in the molecule.
*   **ring_count**: The number of rings in the molecule.
*   **C_count**: The number of carbon (C) atoms in the molecule.
*   **O_count**: The number of oxygen (O) atoms in the molecule.
*   **N_count**: The number of nitrogen (N) atoms in the molecule.
*   **branch_count**: The number of branches in the molecule (a branch is defined as a non-linear chain of atoms).
*   **selfies**: A valid SELFIES (Simplified Molecular-Input Line-Entry System) representation of the molecule. Ensure the generated SELFIES string is syntactically correct and accurately represents the molecular structure.

### Incremental Construction Process:

1. **Start Simple**: Begin by identifying the simplest substructures (e.g., individual atoms, functional groups, or small linear chains). For example:
    - A single carbon atom (`C`) maps to `[C]`.
    - A hydroxyl group (`-OH`) maps to `[O]`.
    - A methyl group (`-CH3`) maps to `[C]`.
    - Ethene (`C=C`) maps to `[C][=C]`.

2. **Combine Substructures**: Incrementally combine these simpler components into larger structures. For instance:
    - Combine `[C]` with `[O]` to form ethanol: `[C][C][O]`.
    - Combine `[C]` with `[=C]` to form propene: `[C][C][=C]`.
    - Combine `[C]` groups into chains (e.g., `[C][C][C]` for propane) or branches (e.g., `[C][C]([C])[C]` for isobutane).

3. **Analyze Connectivity**: Consider the types of bonds (single, double, triple) and the connectivity between substructures:
    - Ensure the bond types match the molecular structure.
    - Account for branching points by using parentheses, e.g., `[C]([C])[C]`.
    - Represent rings accurately using numerical indices, e.g., `[C]1[C][C][C][C][C]1` for cyclohexane.

4. **Validate Incrementally**: At each step, validate the SELFIES representation for correctness (e.g., matching valency rules, bond types, and connectivity).

5. **Construct the Full Molecule**: Combine all identified substructures into the complete molecular SELFIES representation. Use the relationships between substructures to ensure accuracy.

### Emphasis on Inference:

When constructing the SELFIES representation for larger molecules, think in terms of building from smaller molecules:
- **Identify smaller molecules**: Break the target molecule down into smaller known molecules or fragments.
- **Infer relationships**: Use the known SELFIES representations of these smaller molecules to infer how they combine to form the larger molecule.
- **Validate step-by-step**: Ensure the SELFIES representation remains valid at each step of the construction.

### Output Format:

Provide the output in a single, well-formatted JSON object.

**Output Example:**

For a molecule with a single nitrogen atom:
`{"single_bonds": 0, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 0, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 1, "selfies": "[N]"}`

### Image
<image>
``` 

效果:

  - 训练长度1, steps=60
  - 训练长度1+长度2, steps=180
  - 训练 长度1(25) + 长度2 + 长度3, steps=400
  - 训练 长度1(25) + 长度2(25) + 长度3 + 长度4, steps=520
  - 训练 长度1(25) + 长度2(25) + 长度3(25) + 长度4 + 长度5, steps=640
  - 评估长度4: 40 / 75 = 53.333333333333336 %
  - 评估长度5: 148 / 300 = 49.333333333333336 %
  - 评估长度6: 45 / 300 = 15.0 %
  - 训练长度1, steps=700
  - 训练长度1+长度2, steps=820
  - 训练 长度1(25) + 长度2 + 长度3, steps=940
  - 训练 长度1(25) + 长度2(25) + 长度3 + 长度4, steps=1060
  - 训练 长度1(25) + 长度2(25) + 长度3(25) + 长度4 + 长度5, steps=1180
  - 评估长度4: 39 / 75 = 52.0 %
  - 评估长度5: 167 / 300 = 55.666666666666664 %
  - 评估长度6: 71 / 300 = 23.666666666666668 %

改变提示词, 对整体效果提升不大

# 重新建立基线

500个生成链的数据, 每一步微调的epoch=2

1\. 微调, 使用数据: 长度1(500个)*2  
2\. 继续微调, 使用数据: 长度1(100个)*2 + 长度2(500个)*2  
3\. 继续微调, 使用数据: 长度1(100个)*2 + 长度2(100个)*2 + 长度3(500个)*2  
4\. 继续微调, 使用数据: 长度1(100个)*2 + 长度2(100个)*2 + 长度3(100个)*2 + 长度4(500个)*2  
5\. 继续微调, 使用数据: 长度1(100个)*2 + 长度2(100个)*2 + 长度3(100个)*2 + 长度4(100个)*2 + 长度5(500个)*2

脚本: [run_multi_tasks.20241224.sh](/assets/01KJBZP6X3BNGMNB5KJYX0X60Q/run_multi_tasks.20241224.sh)

  - 评估长度4: 54 / 75 = 72.0 %
  - 评估长度5: 220 / 300 = 73.33333333333333 %
  - 评估长度6: 71 / 300 = 23.666666666666668 % (?)

# 调整数据分布, 增加小量"大分子"数据

500个生成链的数据, 每一步微调的epoch=2

1\. 微调, 使用数据: 长度1(500个)*2 + 长度2(100个)*2  
2\. 继续微调, 使用数据: 长度1(100个)*2 + 长度2(500个)*2 + 长度3(100个)*2  
3\. 继续微调, 使用数据: 长度1(100个)*2 + 长度2(100个)*2 + 长度3(500个)*2 + 长度4(100个)*2  
4\. 继续微调, 使用数据: 长度1(100个)*2 + 长度2(100个)*2 + 长度3(100个)*2 + 长度4(500个)*2 + 长度5(100个)*2  
5\. 继续微调, 使用数据: 长度1(100个)*2 + 长度2(100个)*2 + 长度3(100个)*2 + 长度4(100个)*2 + 长度5(500个)*2

  - 评估长度4: 56 / 75 = 74.66666666666667 %
  - 评估长度5: 218 / 300 = 72.66666666666667 %
  - 评估长度6: Correct: 68 / 300 = 22.666666666666664 %

增加小量"大分子"数据, 并不能改善推理性能
