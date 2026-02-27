---
title: 20250103 - 使用ms-swift对Qwen2-VL进行微调 [16] - 加强对双键/三键的理解
confluence_page_id: 3343656
created_at: 2024-12-31T12:12:53+00:00
updated_at: 2025-01-08T06:45:47+00:00
---

前继: [20241228 - 使用ms-swift对Qwen2-VL进行微调 [14] - 转换为分类任务, 增强对原子数的理解[2]]

  

有效的结论: 

  1. 前继中的结论章节
  2. 在提示词中明确指定原子数, 会大幅提高正确率

经过前继的分析 ("长度6的评估结果: evaluate_3.log, 对结果进行分析"): 双键/三键的理解需要加强

# 修改评估脚本, 增加对双键/三键的争取率的分析指标

[evaluate_loop_selfies.py](/assets/01KJBZPD5CR2NC9F6T3QCWX175/evaluate_loop_selfies.py)

# 根据前继的经验, 在提示词中明确指定双键/三键数, 测试在"完美"状态下, 整体正确率是否会提升

### 之前的基线: 

[20241228 - 使用ms-swift对Qwen2-VL进行微调 [14] - 转换为分类任务, 增强对原子数的理解[2]]

提示词中明确指定原子数

  1. 开始训练, 提示词中明确指定原子数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+1
  2. 继续训练, 提示词中明确指定原子数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+1
  3. 评估长度4: 56 / 75 = 74.66666666666667 %
  4. 评估长度5: 55 / 100 = 55.00000000000001 %
  5. 评估长度6: 29 / 100 = 28.999999999999996 %

### 重新做一根基线

提示词中明确指定原子数

  1. 开始训练, 提示词中明确指定原子数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+1
  2. 继续训练, 提示词中明确指定原子数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+1
  3. 评估长度4: 59 / 75 = 78.66666666666666 %, 双键准确率: 86.67%, 三键准确率: 90.67%
  4. 评估长度5: 54 / 100 = 54.0 %, 双键准确率: 83.00%, 三键准确率: 84.00%
  5. 评估长度6: 36 / 100 = 36.0 %, 双键准确率: 76.00%, 三键准确率: 89.00%

提示词中明确指定原子数

  1. 开始训练, 提示词中明确指定原子数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+2
  2. 继续训练, 提示词中明确指定原子数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+2
  3. 评估长度4: 66 / 75 = 88.0 %, 双键准确率: 92.00%, 三键准确率: 94.67%
  4. 评估长度5: 68 / 100 = 68.0 %, 双键准确率: 83.00%, 三键准确率: 87.00%
  5. 评估长度6: 49 / 100 = 49.0 %, 双键准确率: 79.00%, 三键准确率: 91.00%

提示词中明确指定原子数

  1. 开始训练, 提示词中明确指定原子数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+4
  2. 继续训练, 提示词中明确指定原子数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+4
  3. 评估长度4: 70 / 75 = 93.33333333333333 %, 双键准确率: 100.00%, 三键准确率: 96.00%
  4. 评估长度5: 81 / 100 = 81.0 %, 双键准确率: 91.00%, 三键准确率: 93.00%
  5. 评估长度6: 50 / 100 = 50.0 %, 双键准确率: 87.00%, 三键准确率: 97.00%

### **额外插入 - 缩短提示词**

将提示词调整简单, 调整前: 

```
         query = """Analyze the molecular structure in the provided image and infer its SELFIES (Simplified Molecular Input Line Entry System) representation by constructing it incrementally from smaller, simpler molecular substructures. Generate a JSON object containing the following information:

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
<image>"""
``` 

调整后: 

```
query = """Analyze the molecular structure in the provided image and infer its SELFIES (Simplified Molecular Input Line Entry System) representation by constructing it incrementally from smaller, simpler molecular substructures. 
### Output Format:

Provide the output in a single, well-formatted JSON object.

**Output Example:**

For a molecule with a single nitrogen atom:
`{"single_bonds": 0, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 0, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 1, "selfies": "[N]"}`

### Image
<image>""" 

``` 

提示词中明确指定原子数

  1. 开始训练, 提示词中明确指定原子数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+1
  2. 继续训练, 提示词中明确指定原子数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+1
  3. 评估长度4: 59 / 75 = 78.66666666666666 %, 双键准确率: 89.33%, 三键准确率: 93.33%
  4. 评估长度5: 55 / 100 = 55.00000000000001 %, 双键准确率: 84.00%, 三键准确率: 90.00%
  5. 评估长度6: 35 / 100 = 35.0 %, 双键准确率: 66.00%, 三键准确率: 91.00%

提示词中明确指定原子数

  1. 开始训练, 提示词中明确指定原子数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+2
  2. 继续训练, 提示词中明确指定原子数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+2
  3. 评估长度4: 63 / 75 = 84.0 %, 双键准确率: 90.67%, 三键准确率: 97.33%
  4. 评估长度5: 70 / 100 = 70.0 %, 双键准确率: 87.00%, 三键准确率: 91.00%
  5. 评估长度6: 39 / 100 = 39.0 %, 双键准确率: 76.00%, 三键准确率: 91.00%

提示词中明确指定原子数

  1. 开始训练, 提示词中明确指定原子数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+4
  2. 继续训练, 提示词中明确指定原子数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+4
  3. 评估长度4: 67 / 75 = 89.33333333333333 %, 双键准确率: 94.67%, 三键准确率: 97.33%
  4. 评估长度5: 77 / 100 = 77.0 %, 双键准确率: 88.00%, 三键准确率: 94.00%
  5. 评估长度6: 59 / 100 = 59.0 %, 双键准确率: 85.00%, 三键准确率: 95.00%

提示词缩短以后, 效果仍然能维持. 以后的实验会使用缩短后的提示词, 缩短训练时间

### 在提示词中明确指定双键/三键数

提示词中明确指定原子数, 和双键/三键数

  1. 开始训练, 提示词中明确指定原子数和双键/三键数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+1
  2. 继续训练, 提示词中明确指定原子数和双键/三键数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+1
  3. 评估长度4: 53 / 75 = 70.66666666666667 %, 双键准确率: 89.33%, 三键准确率: 90.67%
  4. 评估长度5: 55 / 100 = 55.00000000000001 %, 双键准确率: 89.00%, 三键准确率: 90.00%
  5. 评估长度6: 30 / 100 = 30.0 %, 双键准确率: 78.00%, 三键准确率: 89.00%

评估结果: [evaluate_7.log](/assets/01KJBZPD5CR2NC9F6T3QCWX175/evaluate_7.log)

提示词中明确指定原子数, 和双键/三键数

  1. 开始训练, 提示词中明确指定原子数和双键/三键数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+2
  2. 继续训练, 提示词中明确指定原子数和双键/三键数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+2
  3. 评估长度4: 60 / 75 = 80.0 %, 双键准确率: 88.00%, 三键准确率: 89.33%
  4. 评估长度5: 70 / 100 = 70.0 %, 双键准确率: 87.00%, 三键准确率: 90.00%
  5. 评估长度6: 48 / 100 = 48.0 %, 双键准确率: 86.00%, 三键准确率: 93.00%

提示词中明确指定原子数, 和双键/三键数

  1. 开始训练, 提示词中明确指定原子数和双键/三键数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+4
  2. 继续训练, 提示词中明确指定原子数和双键/三键数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+4
  3. 评估长度4: 64 / 75 = 85.33333333333334 %, 双键准确率: 92.00%, 三键准确率: 93.33%
  4. 评估长度5: 79 / 100 = 79.0 %, 双键准确率: 93.00%, 三键准确率: 92.00%
  5. 评估长度6: 57 / 100 = 56.99999999999999 %, 双键准确率: 84.00%, 三键准确率: 93.00%

效果未见提升, 分析评估结果: [evaluate_7.log](/assets/01KJBZPD5CR2NC9F6T3QCWX175/evaluate_7.log)

错误的9个case中, 有2个的selfies表达式的双键/三键数量确实错误, 其他7个case的selfies表达式的双键/三键的数量正确, 但转换成smiles表达式时键被改变了, 比如: 

  - [C][O][N][#C] 对应 CON=C, 键的性质变了
  - [C][O][O][#C] 对应 COOC , 键的性质变了
  - ...

目前的结论: 

  - 生成的selfies表达式, 双键三键的准确率很高.
  - 在提示词中指定双键/三键数, 也不能提高对双键/三键的准确率
  - 大部分的问题出现在 selfies式 不是个合法式, 转成smiles后, 键性质变了.

# 需要增强模型对selfies合法性的理解

增加一个修正selfies的任务: 输入图片+错误的selfies表达式 (对其中一个原子进行键的替换), 要求进行修正

生成数据的脚本: [loop_selfies_and_image.generate_correct_bond.ipynb](/assets/01KJBZPD5CR2NC9F6T3QCWX175/loop_selfies_and_image.generate_correct_bond.ipynb)

训练结果: 

增加selfies修正任务

  1. 开始训练, selfies修正任务, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量), epoch+1
  2. 继续训练, selfies修正任务, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量), epoch+1
  3. 继续训练, 提示词中明确指定原子数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+1
  4. 继续训练, 提示词中明确指定原子数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+1
  5. 评估长度4: 54 / 75 = 72.0 %, 双键准确率: 88.00%, 三键准确率: 92.00%
  6. 评估长度5: 52 / 100 = 52.0 %, 双键准确率: 75.00%, 三键准确率: 88.00%
  7. 评估长度6: 31 / 100 = 31.0 %, 双键准确率: 74.00%, 三键准确率: 96.00%

对比基线, 没有明显变化

增加selfies修正任务

  1. 开始训练, selfies修正任务, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量), epoch+2
  2. 继续训练, selfies修正任务, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量), epoch+2
  3. 继续训练, 提示词中明确指定原子数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+1
  4. 继续训练, 提示词中明确指定原子数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+1
  5. 评估长度4: 50 / 75 = 66.66666666666666 %, 双键准确率: 80.00%, 三键准确率: 88.00%
  6. 评估长度5: 60 / 100 = 60.0 %, 双键准确率: 75.00%, 三键准确率: 90.00%
  7. 评估长度6: 37 / 100 = 37.0 %, 双键准确率: 77.00%, 三键准确率: 91.00%

比起上一任务, 没有明显提升 (看上去修正任务和主任务无关)

增加selfies修正任务

  1. 开始训练, selfies修正任务, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量), epoch+2
  2. 继续训练, selfies修正任务, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量), epoch+2
  3. 继续训练, 提示词中明确指定原子数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+2
  4. 继续训练, 提示词中明确指定原子数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+2
  5. 评估长度4: 64 / 75 = 85.33333333333334 %, 双键准确率: 92.00%, 三键准确率: 94.67%
  6. 评估长度5: 75 / 100 = 75.0 %, 双键准确率: 89.00%, 三键准确率: 93.00%
  7. 评估长度6: 50 / 100 = 50.0 %, 双键准确率: 86.00%, 三键准确率: 94.00%

对比基线, 没有明显变化, 仅长度5有小幅提升(认为是实验偏差)

增加selfies修正任务

  1. 开始训练, selfies修正任务, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量), epoch+4
  2. 继续训练, selfies修正任务, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量), epoch+4
  3. 继续训练, 提示词中明确指定原子数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+2
  4. 继续训练, 提示词中明确指定原子数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+2
  5. 评估长度4: 61 / 75 = 81.33333333333333 %, 双键准确率: 88.00%, 三键准确率: 89.33%
  6. 评估长度5: 79 / 100 = 79.0 %, 双键准确率: 90.00%, 三键准确率: 95.00%
  7. 评估长度6: 51 / 100 = 51.0 %, 双键准确率: 81.00%, 三键准确率: 96.00%

比起上一任务, 没有明显变化, 仅长度5有小幅提升(认为是实验偏差)

增加selfies修正任务

  1. 开始训练, selfies修正任务, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量), epoch+4
  2. 继续训练, selfies修正任务, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量), epoch+4
  3. 继续训练, 提示词中明确指定原子数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+4
  4. 继续训练, 提示词中明确指定原子数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+4
  5. 评估长度4: 66 / 75 = 88.0 %, 双键准确率: 90.67%, 三键准确率: 100.00%
  6. 评估长度5: 79 / 100 = 79.0 %, 双键准确率: 88.00%, 三键准确率: 95.00%
  7. 评估长度6: 61 / 100 = 61.0 %, 双键准确率: 84.00%, 三键准确率: 92.00%

对比基线, 没有明显变化

增加selfies修正任务, 进行混合数据训练

  1. 开始训练, 混合如下数据: epoch+1
     1. selfies修正任务, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量)
     2. 提示词中明确指定原子数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量)
  2. 继续训练, 混合如下数据: epoch+1
     1. selfies修正任务, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量)
     2. 提示词中明确指定原子数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量)
  3. 评估长度4: 47 / 75 = 62.66666666666667 %, 双键准确率: 80.00%, 三键准确率: 93.33%
  4. 评估长度5: 54 / 100 = 54.0 %, 双键准确率: 82.00%, 三键准确率: 89.00%
  5. 评估长度6: 38 / 100 = 38.0 %, 双键准确率: 65.00%, 三键准确率: 91.00%

对比基线, 没有明显提升, 长度4的准确度下降16%

增加selfies修正任务, 进行混合数据训练

  1. 开始训练, 混合如下数据: epoch+2
     1. selfies修正任务, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量)
     2. 提示词中明确指定原子数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量)
  2. 继续训练, 混合如下数据: epoch+2
     1. selfies修正任务, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量)
     2. 提示词中明确指定原子数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量)
  3. 评估长度4: 71 / 75 = 94.66666666666667 %, 双键准确率: 97.33%, 三键准确率: 98.67%
  4. 评估长度5: 84 / 100 = 84.0 %, 双键准确率: 93.00%, 三键准确率: 95.00%
  5. 评估长度6: 60 / 100 = 60.0 %, 双键准确率: 84.00%, 三键准确率: 96.00%

对比基线, 没有明显提升

  

增加selfies修正任务, 进行混合数据训练

  1. 开始训练, 混合如下数据: epoch+4
     1. selfies修正任务, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量)
     2. 提示词中明确指定原子数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量)
  2. 继续训练, 混合如下数据: epoch+4
     1. selfies修正任务, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量)
     2. 提示词中明确指定原子数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量)
  3. 评估长度4: 66 / 75 = 88.0 %, 双键准确率: 97.33%, 三键准确率: 98.67%
  4. 评估长度5: 79 / 100 = 79.0 %, 双键准确率: 93.00%, 三键准确率: 95.00%
  5. 评估长度6: 57 / 100 = 56.99999999999999 %, 双键准确率: 85.00%, 三键准确率: 95.00%

对比基线, 没有明显提升

增加selfies修正任务

  1. 开始训练, selfies修正任务, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量), epoch+1
  2. 继续训练, selfies修正任务, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量), epoch+1
  3. 继续训练, 混合如下数据: epoch+1
     1. selfies修正任务, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量)
     2. 提示词中明确指定原子数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量)
  4. 继续训练, 混合如下数据: epoch+1
     1. selfies修正任务, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量)
     2. 提示词中明确指定原子数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量)
  5. 评估长度4: 53 / 75 = 70.66666666666667 %, 双键准确率: 82.67%, 三键准确率: 88.00%
  6. 评估长度5: 63 / 100 = 63.0 %, 双键准确率: 83.00%, 三键准确率: 90.00%
  7. 评估长度6: 34 / 100 = 34.0 %, 双键准确率: 78.00%, 三键准确率: 92.00%

增加selfies修正任务

  1. 开始训练, selfies修正任务, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量), epoch+2
  2. 继续训练, selfies修正任务, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量), epoch+2
  3. 继续训练, 混合如下数据: epoch+2
     1. selfies修正任务, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量)
     2. 提示词中明确指定原子数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量)
  4. 继续训练, 混合如下数据: epoch+2
     1. selfies修正任务, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量)
     2. 提示词中明确指定原子数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量)
  5. 评估长度4: 60 / 75 = 80.0 %, 双键准确率: 92.00%, 三键准确率: 90.67%
  6. 评估长度5: 63 / 100 = 63.0 %, 双键准确率: 82.00%, 三键准确率: 90.00%
  7. 评估长度6: 49 / 100 = 49.0 %, 双键准确率: 88.00%, 三键准确率: 93.00%

  

增加selfies修正任务

  1. 开始训练, selfies修正任务, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量), epoch+4
  2. 继续训练, selfies修正任务, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量), epoch+4
  3. 继续训练, 混合如下数据: epoch+4
     1. selfies修正任务, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量)
     2. 提示词中明确指定原子数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量)
  4. 继续训练, 混合如下数据: epoch+4
     1. selfies修正任务, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量)
     2. 提示词中明确指定原子数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量)
  5. 评估长度4: 69 / 75 = 92.0 %, 双键准确率: 96.00%, 三键准确率: 98.67%
  6. 评估长度5: 78 / 100 = 78.0 %, 双键准确率: 90.00%, 三键准确率: 92.00%
  7. 评估长度6: 62 / 100 = 62.0 %, 双键准确率: 81.00%, 三键准确率: 95.00%

对比基线, 没有明显提升

目前多种训练方法显示: 两种训练任务仿佛没有明显关系, 让提示词接近, 尝试让两种训练任务之间"有关联"

# 尝试让两种训练任务之间"有关联"

常用方法 From o1p: 

```
对于大模型微调的多任务训练, 多种任务之间的"关系", 或者模型是否能理解这几种任务之间的关联性, 进行迁移学习. 常见的解决方案是什么? 总不能我让多种任务都呈现相同的提示词吧?
 
---
 
在大型模型的微调过程中，实现多任务训练并让模型理解任务之间的关联性是一个重要的研究方向。以下是一些常见的解决方案：

1. **统一的输入格式与任务描述**：
   - **任务提示**：在输入中明确加入任务的描述，使模型知道需要执行哪种任务。例如，在翻译任务中，可以在输入前添加`"翻译成英文："`，在摘要任务中添加`"生成摘要："`。
   - **自然语言提示**：利用自然语言作为提示，使模型更好地理解任务。例如，`"请将以下文本翻译成英文：..."`。

2. **任务标识符（Task Identifier）**：
   - **特殊标记**：为每个任务分配一个独特的标记或编码，在输入中加入该标记，帮助模型区分不同的任务。
   - **嵌入向量**：为每个任务学习一个任务嵌入向量，在模型的输入层或特定层加入该嵌入，显式地告诉模型当前的任务。

3. **多任务学习架构**：
   - **硬参数共享（Hard Parameter Sharing）**：在模型的底层共享参数，高层为每个任务保留特定的参数。这种方式可以共享低级特征，捕获任务之间的共性。
   - **软参数共享（Soft Parameter Sharing）**：为每个任务创建独立的模型，但在训练过程中对其参数施加正则化约束，使其尽可能相似。

4. **任务关系建模**：
   - **联合损失函数**：设计一个联合损失函数，结合所有任务的损失，使模型在优化时同时考虑多个任务的目标。
   - **跨任务注意力机制**：在模型中引入注意力机制，以便在不同任务之间动态地共享信息，捕捉任务之间的相关性。

5. **迁移学习与持续学习（Continual Learning）**：
   - **顺序训练**：先在一个或几个相关任务上训练模型，然后将学到的参数作为初始化，继续在新的任务上微调。
   - **记忆机制**：在训练新任务时，保留之前任务的知识，防止模型遗忘（如弹性权重保护、经验回放等方法）。

6. **使用元学习（Meta-learning）**：
   - **学习如何学习**：让模型学习跨任务的学习规则，提高其在新任务上的适应能力。
   - **任务嵌入与条件生成**：模型根据任务嵌入来调整自身的参数或生成策略，以适应不同的任务需求。

7. **数据集的合理构建**：
   - **任务相关的数据增强**：通过数据增强方法，生成跨任务的样本，丰富模型的训练数据。
   - **平衡数据分布**：确保各任务的数据分布平衡，避免模型过度偏向某一特定任务。

**总结**：

在多任务训练中，关键是让模型能够区分并理解不同的任务，同时利用任务之间的关联性来提升整体性能。简单地使用相同的提示词不足以达到这个目的。相反，我们需要在模型输入、架构设计和训练策略上进行精心设计，使模型既能捕获共享的特征，又能保留各任务的特定信息。这些方法包括在输入中加入明确的任务描述、使用任务标识符、设计共享与特定的模型架构，以及引入机制来捕捉任务之间的关系。

通过以上策略，模型不仅能够在多个任务上取得良好的性能，还能实现知识的迁移和共享，提高在新任务上的学习效率。
``` 

可尝试的方法:

  - 统一的输入格式与任务描述
  - 任务标识符 / 特殊标识  

    - 可尝试使用few-shot, 将特殊标识的任务连接起来

### 先尝试: 统一的输入格式与任务描述

调整数据生成脚本: [loop_selfies_and_image.generate_correct_bond.3.ipynb](/assets/01KJBZPD5CR2NC9F6T3QCWX175/loop_selfies_and_image.generate_correct_bond.3.ipynb)

将提示词/输入/输出调整成跟主任务相似, 多提供一个selfies参考表达式

增加selfies修正任务, 逐步学习

  1. 开始训练, selfies修正任务, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量), epoch+1
  2. 继续训练, selfies修正任务, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量), epoch+1
  3. 继续训练, 提示词中明确指定原子数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+1
  4. 继续训练, 提示词中明确指定原子数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+1
  5. 评估长度4: Correct: 73.33%, Double Bonds Match Accuracy: 85.33%, Triple Bonds Match Accuracy: 92.00%
  6. 评估长度5: Correct: 50.00%, Double Bonds Match Accuracy: 74.00%, Triple Bonds Match Accuracy: 91.00%
  7. 评估长度6: Correct: 33.00%, Double Bonds Match Accuracy: 70.00%, Triple Bonds Match Accuracy: 92.00%

对比基线(epoch=1), 没有明显提升

增加selfies修正任务, 逐步学习

  1. 开始训练, selfies修正任务, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量), epoch+2
  2. 继续训练, selfies修正任务, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量), epoch+2
  3. 继续训练, 提示词中明确指定原子数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+1
  4. 继续训练, 提示词中明确指定原子数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+1
  5. 评估长度4: Correct: 69.33%, Double Bonds Match Accuracy: 85.33%, Triple Bonds Match Accuracy: 85.33%
  6. 评估长度5: Correct: 46.00%, Double Bonds Match Accuracy: 79.00%, Triple Bonds Match Accuracy: 87.00%
  7. 评估长度6: Correct: 26.00%, Double Bonds Match Accuracy: 71.00%, Triple Bonds Match Accuracy: 92.00%

对比基线(epoch=1), 出现轻微下降

增加selfies修正任务, 逐步学习

  1. 开始训练, selfies修正任务, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量), epoch+4
  2. 继续训练, selfies修正任务, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量), epoch+4
  3. 继续训练, 提示词中明确指定原子数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+1
  4. 继续训练, 提示词中明确指定原子数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+1
  5. 评估长度4: Correct: 62.67%, Double Bonds Match Accuracy: 86.67%, Triple Bonds Match Accuracy: 88.00%
  6. 评估长度5: Correct: 43.00%, Double Bonds Match Accuracy: 80.00%, Triple Bonds Match Accuracy: 90.00%
  7. 评估长度6: Correct: 27.00%, Double Bonds Match Accuracy: 77.00%, Triple Bonds Match Accuracy: 95.00%

对比基线(epoch=1), 出现轻微下降

增加selfies修正任务, 逐步学习

  1. 开始训练, selfies修正任务, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量), epoch+2
  2. 继续训练, selfies修正任务, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量), epoch+2
  3. 继续训练, 提示词中明确指定原子数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+2
  4. 继续训练, 提示词中明确指定原子数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+2
  5. 评估长度4: Correct: 88.00%, Double Bonds Match Accuracy: 96.00%, Triple Bonds Match Accuracy: 96.00%
  6. 评估长度5: Correct: 65.00%, Double Bonds Match Accuracy: 86.00%, Triple Bonds Match Accuracy: 91.00%
  7. 评估长度6: Correct: 44.00%, Double Bonds Match Accuracy: 74.00%, Triple Bonds Match Accuracy: 96.00%

对比基线(epoch=2), 没有明显提升

增加selfies修正任务, 逐步学习

  1. 开始训练, selfies修正任务, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量), epoch+4
  2. 继续训练, selfies修正任务, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量), epoch+4
  3. 继续训练, 提示词中明确指定原子数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+2
  4. 继续训练, 提示词中明确指定原子数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+2
  5. 评估长度4: Correct: 81.33%, Double Bonds Match Accuracy: 89.33%, Triple Bonds Match Accuracy: 89.33%
  6. 评估长度5: Correct: 67.00%, Double Bonds Match Accuracy: 81.00%, Triple Bonds Match Accuracy: 90.00%
  7. 评估长度6: Correct: 44.00%, Double Bonds Match Accuracy: 73.00%, Triple Bonds Match Accuracy: 92.00%

对比基线(epoch=2), 没有明显提升

增加selfies修正任务, 逐步学习

  1. 开始训练, selfies修正任务, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量), epoch+4
  2. 继续训练, selfies修正任务, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量), epoch+4
  3. 继续训练, 提示词中明确指定原子数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+4
  4. 继续训练, 提示词中明确指定原子数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+4
  5. 评估长度4: Correct: 92.00%, Double Bonds Match Accuracy: 94.67%, Triple Bonds Match Accuracy: 98.67%
  6. 评估长度5: Correct: 77.00%, Double Bonds Match Accuracy: 87.00%, Triple Bonds Match Accuracy: 94.00%
  7. 评估长度6: Correct: 54.00%, Double Bonds Match Accuracy: 86.00%, Triple Bonds Match Accuracy: 96.00%

对比基线(epoch=4), 没有明显提升

增加selfies修正任务, 逐步学习, 使用混合数据增强理解

  1. 开始训练, selfies修正任务, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量), epoch+1
  2. 继续训练, selfies修正任务, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量), epoch+1
  3. 继续训练, 使用混合数据: epoch+1
     1. 提示词中明确指定原子数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量)
     2. selfies修正任务, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量)
  4. 继续训练, 使用混合数据: epoch+1
     1. 提示词中明确指定原子数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量)
     2. selfies修正任务, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量)
  5. 评估长度4: Correct: 78.67%, Double Bonds Match Accuracy: 89.33%, Triple Bonds Match Accuracy: 90.67%
  6. 评估长度5: Correct: 59.00%, Double Bonds Match Accuracy: 83.00%, Triple Bonds Match Accuracy: 88.00%
  7. 评估长度6: Correct: 28.00%, Double Bonds Match Accuracy: 72.00%, Triple Bonds Match Accuracy: 87.00%

对比基线(epoch=1), 对长度6的预测下降8%

增加selfies修正任务, 逐步学习, 使用混合数据增强理解

  1. 开始训练, selfies修正任务, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量), epoch+2
  2. 继续训练, selfies修正任务, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量), epoch+2
  3. 继续训练, 使用混合数据: epoch+1
     1. 提示词中明确指定原子数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量)
     2. selfies修正任务, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量)
  4. 继续训练, 使用混合数据: epoch+1
     1. 提示词中明确指定原子数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量)
     2. selfies修正任务, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量)
  5. 评估长度4: Correct: 84.00%, Double Bonds Match Accuracy: 92.00%, Triple Bonds Match Accuracy: 92.00%
  6. 评估长度5: Correct: 64.00%, Double Bonds Match Accuracy: 83.00%, Triple Bonds Match Accuracy: 87.00%
  7. 评估长度6: Correct: 43.00%, Double Bonds Match Accuracy: 76.00%, Triple Bonds Match Accuracy: 88.00%

对比基线(epoch=1), 对长度4/5/6的准确率都有6-10%提升, 与基线(epoch=2)性能类似

需要重复试验, AB验证

增加selfies修正任务, 逐步学习, 使用混合数据增强理解

  1. 开始训练, selfies修正任务, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量), epoch+2
  2. 继续训练, selfies修正任务, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量), epoch+2
  3. 继续训练, 使用混合数据: epoch+2
     1. 提示词中明确指定原子数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量)
     2. selfies修正任务, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量)
  4. 继续训练, 使用混合数据: epoch+2
     1. 提示词中明确指定原子数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量)
     2. selfies修正任务, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量)
  5. 评估长度4: Correct: 80.00%, Double Bonds Match Accuracy: 90.67%, Triple Bonds Match Accuracy: 93.33%
  6. 评估长度5: Correct: 69.00%, Double Bonds Match Accuracy: 88.00%, Triple Bonds Match Accuracy: 91.00%
  7. 评估长度6: Correct: 52.00%, Double Bonds Match Accuracy: 82.00%, Triple Bonds Match Accuracy: 92.00%

对比基线(epoch=2), 没有明显提升

增加selfies修正任务, 逐步学习, 使用混合数据增强理解

  1. 开始训练, selfies修正任务, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量), epoch+4
  2. 继续训练, selfies修正任务, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量), epoch+4
  3. 继续训练, 使用混合数据: epoch+2
     1. 提示词中明确指定原子数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量)
     2. selfies修正任务, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量)
  4. 继续训练, 使用混合数据: epoch+2
     1. 提示词中明确指定原子数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量)
     2. selfies修正任务, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量)
  5. 评估长度4: Correct: 86.67%, Double Bonds Match Accuracy: 92.00%, Triple Bonds Match Accuracy: 97.33%
  6. 评估长度5: Correct: 70.00%, Double Bonds Match Accuracy: 84.00%, Triple Bonds Match Accuracy: 94.00%
  7. 评估长度6: Correct: 49.00%, Double Bonds Match Accuracy: 78.00%, Triple Bonds Match Accuracy: 96.00%

对比基线(epoch=2), 没有明显提升

增加selfies修正任务, 逐步学习, 使用混合数据增强理解

  1. 开始训练, selfies修正任务, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量), epoch+4
  2. 继续训练, selfies修正任务, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量), epoch+4
  3. 继续训练, 使用混合数据: epoch+4
     1. 提示词中明确指定原子数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量)
     2. selfies修正任务, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量)
  4. 继续训练, 使用混合数据: epoch+4
     1. 提示词中明确指定原子数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量)
     2. selfies修正任务, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量)
  5. 评估长度4: Correct: 89.33%, Double Bonds Match Accuracy: 93.33%, Triple Bonds Match Accuracy: 96.00%
  6. 评估长度5: Correct: 76.00%, Double Bonds Match Accuracy: 89.00%, Triple Bonds Match Accuracy: 91.00%
  7. 评估长度6: Correct: 52.00%, Double Bonds Match Accuracy: 83.00%, Triple Bonds Match Accuracy: 90.00%

对比基线(epoch=4), 没有明显提升

增加selfies修正任务, 逐步学习, 使用混合数据增强理解

  1. 开始训练, 使用混合数据: epoch+1
     1. 提示词中明确指定原子数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量)
     2. selfies修正任务, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量)
  2. 继续训练, 使用混合数据: epoch+1
     1. 提示词中明确指定原子数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量)
     2. selfies修正任务, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量)
  3. 评估长度4: Correct: 73.33%, Double Bonds Match Accuracy: 93.33%, Triple Bonds Match Accuracy: 93.33%
  4. 评估长度5: Correct: 60.00%, Double Bonds Match Accuracy: 85.00%, Triple Bonds Match Accuracy: 89.00%
  5. 评估长度6: Correct: 39.00%, Double Bonds Match Accuracy: 80.00%, Triple Bonds Match Accuracy: 91.00%

对比基线(epoch=1), 没有明显提升

增加selfies修正任务, 逐步学习, 使用混合数据增强理解

  1. 开始训练, 使用混合数据: epoch+2
     1. 提示词中明确指定原子数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量)
     2. selfies修正任务, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量)
  2. 继续训练, 使用混合数据: epoch+2
     1. 提示词中明确指定原子数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量)
     2. selfies修正任务, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量)
  3. 评估长度4: Correct: 86.67%, Double Bonds Match Accuracy: 89.33%, Triple Bonds Match Accuracy: 98.67%
  4. 评估长度5: Correct: 72.00%, Double Bonds Match Accuracy: 85.00%, Triple Bonds Match Accuracy: 94.00%
  5. 评估长度6: Correct: 41.00%, Double Bonds Match Accuracy: 79.00%, Triple Bonds Match Accuracy: 89.00%

对比基线(epoch=2), 长度6的准确率下降8%

增加selfies修正任务, 逐步学习, 使用混合数据增强理解

  1. 开始训练, 使用混合数据: epoch+4
     1. 提示词中明确指定原子数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量)
     2. selfies修正任务, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量)
  2. 继续训练, 使用混合数据: epoch+4
     1. 提示词中明确指定原子数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量)
     2. selfies修正任务, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量)
  3. 评估长度4: Correct: 90.67%, Double Bonds Match Accuracy: 96.00%, Triple Bonds Match Accuracy: 98.67%
  4. 评估长度5: Correct: 81.00%, Double Bonds Match Accuracy: 92.00%, Triple Bonds Match Accuracy: 94.00%
  5. 评估长度6: Correct: 45.00%, Double Bonds Match Accuracy: 84.00%, Triple Bonds Match Accuracy: 96.00%

对比基线(epoch=4), 长度6的准确率下降5%

目前的结论: 即使提示词非常相似, 这两个任务也无法起到相互辅助的作用

# 尝试让两种训练任务之间"有关联" (2)

调整任务: 

  - 任务1: 键合检验, 提示词: 

```
You are a chemical informatics expert tasked with validating the bonding rules of a SELFIES expression. The input includes:  
1. **Molecular image**: Shows the molecular structure.  
2. **SELFIES expression**: Describes the molecular structure.  

**Task**:  
1. Parse the molecular structure from the image and the SELFIES expression.  
2. Validate whether the SELFIES expression follows bonding rules:
   - Check if each atom's bonding matches its valence.  
   - Ensure the SELFIES expression matches the molecular structure in the image.  

**Output**:  
- **Validation Result**: Success or failure.  

# Image
<image>

# SELFIES
{selfies}
```

  - 任务2: 主任务, 调整提示词: 

```
Analyze the molecular structure in the provided image and infer its SELFIES (Simplified Molecular Input Line Entry System) representation by constructing it incrementally from smaller, simpler molecular substructures. 
### Output Format:

Provide the output in a single, well-formatted JSON object.

**Output Example:**

For a molecule with a single nitrogen atom:
`{"single_bonds": 0, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 0, "O_count": 0, "N_count": 1, "branch_count": 0, "atom_count": 1, "selfies": "[N]"}`

### Special Rule:
You should provide the SELFIES expression only if its bonding rules are successfully validated

### Image
<image>

```

增加键合检验任务, 逐步学习

  1. 开始训练, 键合检验任务, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量), epoch+1
  2. 继续训练, 键合检验任务, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量), epoch+1
  3. 继续训练, 提示词中明确指定原子数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+1
  4. 继续训练, 提示词中明确指定原子数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+1
  5. 评估长度4: Correct: 58.67%, Double Bonds Match Accuracy: 84.00%, Triple Bonds Match Accuracy: 84.00%
  6. 评估长度5: Correct: 47.00%, Double Bonds Match Accuracy: 73.00%, Triple Bonds Match Accuracy: 82.00%
  7. 评估长度6: Correct: 25.00%, Double Bonds Match Accuracy: 65.00%, Triple Bonds Match Accuracy: 89.00%

与基线(epoch=1)对比, 三组评估都有10%以上的下降 (说明两种任务之间有干扰?)

增加键合检验任务, 使用混合数据增强理解

  1. 开始训练, 使用混合数据: epoch+1
     1. 提示词中明确指定原子数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量)
     2. 键合检验任务, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量)
  2. 继续训练, 使用混合数据: epoch+1
     1. 提示词中明确指定原子数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量)
     2. 键合检验任务, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量)
  3. 评估长度4: Correct: 62.67%, Double Bonds Match Accuracy: 77.33%, Triple Bonds Match Accuracy: 82.67%
  4. 评估长度5: Correct: 45.00%, Double Bonds Match Accuracy: 76.00%, Triple Bonds Match Accuracy: 80.00%
  5. 评估长度6: Correct: 25.00%, Double Bonds Match Accuracy: 67.00%, Triple Bonds Match Accuracy: 87.00%

与基线(epoch=1)对比, 三组评估都有10%以上的下降 (说明两种任务之间有干扰?)

增加键合检验任务, 逐步学习, 使用混合数据增强理解

  1. 开始训练, 键合检验, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量), epoch+1
  2. 继续训练, 键合检验, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量), epoch+1
  3. 继续训练, 使用混合数据: epoch+1
     1. 提示词中明确指定原子数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量)
     2. 键合检验, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量)
  4. 继续训练, 使用混合数据: epoch+1
     1. 提示词中明确指定原子数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量)
     2. 键合检验, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量)
  5. 评估长度4: Correct: 66.67%, Double Bonds Match Accuracy: 80.00%, Triple Bonds Match Accuracy: 90.67%
  6. 评估长度5: Correct: 54.00%, Double Bonds Match Accuracy: 76.00%, Triple Bonds Match Accuracy: 87.00%
  7. 评估长度6: Correct: 37.00%, Double Bonds Match Accuracy: 76.00%, Triple Bonds Match Accuracy: 94.00%

与基线(epoch=1)对比, 长度4的准确率下降12%, 其他长度没变

但从准确率看上去, 两种任务间是负关联, 且程度不深.

尝试在evaluate中, 增加 predict_selfies_valid_accuracy, 对selfies合法率进行判断: [evaluate_loop_selfies.specific_atom_count.2.py](/assets/01KJBZPD5CR2NC9F6T3QCWX175/evaluate_loop_selfies.specific_atom_count.2.py)

### 重新建立基线 (用修改后的主任务提示词跑基线, 让主任务进行键合检验), 新基线:

提示词中明确指定原子数

  1. 开始训练, 提示词中明确指定原子数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+1
  2. 继续训练, 提示词中明确指定原子数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+1
  3. 评估长度4: Correct: 66.67%, Predict selfies valid Accuracy: 93.33%, Double Bonds Match Accuracy: 77.33%, Triple Bonds Match Accuracy: 86.67%
  4. 评估长度5: Correct: 52.00%, Predict selfies valid Accuracy: 92.00%, Double Bonds Match Accuracy: 73.00%, Triple Bonds Match Accuracy: 88.00%
  5. 评估长度6: Correct: 38.00%, Predict selfies valid Accuracy: 85.00%, Double Bonds Match Accuracy: 70.00%, Triple Bonds Match Accuracy: 91.00%

提示词中明确指定原子数

  1. 开始训练, 提示词中明确指定原子数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+2
  2. 继续训练, 提示词中明确指定原子数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+2
  3. 评估长度4: Correct: 78.67%, Predict selfies valid Accuracy: 94.67%, Double Bonds Match Accuracy: 89.33%, Triple Bonds Match Accuracy: 94.67%
  4. 评估长度5: Correct: 69.00%, Predict selfies valid Accuracy: 93.00%, Double Bonds Match Accuracy: 81.00%, Triple Bonds Match Accuracy: 91.00%
  5. 评估长度6: Correct: 43.00%, Predict selfies valid Accuracy: 90.00%, Double Bonds Match Accuracy: 78.00%, Triple Bonds Match Accuracy: 93.00%

提示词中明确指定原子数

  1. 开始训练, 提示词中明确指定原子数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+4
  2. 继续训练, 提示词中明确指定原子数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+4
  3. 评估长度4: 
     1. Correct: 80.00%, Predict selfies valid Accuracy: 97.33%, Double Bonds Match Accuracy: 92.00%, Triple Bonds Match Accuracy: 93.33%
     2. Correct: 85.33%, Predict selfies valid Accuracy: 98.67%, Double Bonds Match Accuracy: 92.00%, Triple Bonds Match Accuracy: 97.33%

  4. 评估长度5: 
     1. Correct: 75.00%, Predict selfies valid Accuracy: 96.00%, Double Bonds Match Accuracy: 86.00%, Triple Bonds Match Accuracy: 92.00%
     2. Correct: 76.00%, Predict selfies valid Accuracy: 97.00%, Double Bonds Match Accuracy: 86.00%, Triple Bonds Match Accuracy: 95.00%

  5. 评估长度6: 
     1. Correct: 47.00%, Predict selfies valid Accuracy: 92.00%, Double Bonds Match Accuracy: 77.00%, Triple Bonds Match Accuracy: 96.00%
     2. Correct: 52.00%, Predict selfies valid Accuracy: 93.00%, Double Bonds Match Accuracy: 82.00%, Triple Bonds Match Accuracy: 94.00%

提示词中明确指定原子数

  1. 开始训练, 提示词中明确指定原子数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+6
  2. 继续训练, 提示词中明确指定原子数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+6
  3. 评估长度4: 
     1. Correct: 92.00%, Predict selfies valid Accuracy: 100.00%, Double Bonds Match Accuracy: 97.33%, Triple Bonds Match Accuracy: 100.00%

  4. 评估长度5: 
     1. Correct: 81.00%, Predict selfies valid Accuracy: 97.00%, Double Bonds Match Accuracy: 91.00%, Triple Bonds Match Accuracy: 92.00%

  5. 评估长度6: 
     1. Correct: 59.00%, Predict selfies valid Accuracy: 86.00%, Double Bonds Match Accuracy: 80.00%, Triple Bonds Match Accuracy: 94.00%

提示词中明确指定原子数

  1. 开始训练, 提示词中明确指定原子数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+8
  2. 继续训练, 提示词中明确指定原子数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+8
  3. 评估长度4: 
     1. Correct: 86.67%, Predict selfies valid Accuracy: 98.67%, Double Bonds Match Accuracy: 93.33%, Triple Bonds Match Accuracy: 98.67%

  4. 评估长度5: 
     1. Correct: 73.00%, Predict selfies valid Accuracy: 96.00%, Double Bonds Match Accuracy: 89.00%, Triple Bonds Match Accuracy: 92.00%

  5. 评估长度6: 
     1. Correct: 53.00%, Predict selfies valid Accuracy: 88.00%, Double Bonds Match Accuracy: 87.00%, Triple Bonds Match Accuracy: 90.00%

与增加了键合检验任务的情况进行对比:

增加键合检验任务, 使用混合数据增强理解

  1. 开始训练, 使用混合数据: epoch+1
     1. 提示词中明确指定原子数, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量)
     2. 键合检验任务, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量)
  2. 继续训练, 使用混合数据: epoch+1
     1. 提示词中明确指定原子数, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量)
     2. 键合检验任务, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量)
  3. 评估长度4: Correct: 61.33%, Predict selfies valid Accuracy: 96.00%, Double Bonds Match Accuracy: 80.00%, Triple Bonds Match Accuracy: 82.67%
  4. 评估长度5: Correct: 51.00%, Predict selfies valid Accuracy: 87.00%, Double Bonds Match Accuracy: 77.00%, Triple Bonds Match Accuracy: 85.00%
  5. 评估长度6: Correct: 25.00%, Predict selfies valid Accuracy: 85.00%, Double Bonds Match Accuracy: 66.00%, Triple Bonds Match Accuracy: 90.00%

两类任务仍然没有关联, Predict selfies valid Accuracy 未见明显提高

**使用COT结构提示词** , 将最终任务和键合验证任务, 使用同一套提示词 (相当于对最终任务做数据增强, 要求最终任务给出并修正selfies). 提示词变成: 

````
Analyze the molecular structure in the provided image and generate its SELFIES (Simplified Molecular Input Line Entry System) representation. Use a step-by-step reasoning process to ensure the generated SELFIES is valid. Follow this workflow:

### Workflow:
1. **Generate SELFIES:** First, identify the molecular structure in the image and construct an initial SELFIES representation by incrementally combining smaller, simpler substructures.
2. **Validate Bonding Rules:** Verify the bonding rules of the generated SELFIES expression to ensure all atoms and bonds conform to chemical valency constraints.
3. **Refine SELFIES:** If any validation errors are identified, refine the SELFIES expression to ensure correctness.

### Output Format:
Provide the output in a single, well-formatted JSON object containing the following fields:
- `"initial_selfies"`: The initial SELFIES expression generated in step 1.
- `"validation"`: A detailed analysis of whether the bonding rules are satisfied. If the rules are not satisfied, explain the specific issues (e.g., incorrect valency, missing bonds, extra bonds, etc.).
- `"final_selfies"`: The refined and validated SELFIES expression (same as the initial expression if valid).

### Output Example:
For a molecule with a single nitrogen atom:
```json
{
  "initial_selfies": "[N]",
  "validation": "The bonding rules are satisfied.",
  "final_selfies": "[N]"
}
```
For a molecule with an incorrect initial SELFIES:
```json
{
  "initial_selfies": "[C][C][O]",
  "validation": "The bonding rules are not satisfied.",
  "final_selfies": "[C][C][=O]"
}
```

### Image
<image>

###Notice 
we already know: the atom count is {length}
```` 

增加键合检验COT任务, 使用混合数据增强理解

  1. 开始训练, 使用混合数据: epoch+1
     1. 提示词中明确指定原子数, 使用COT流程, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量)
     2. 键合检验任务, 使用COT流程, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量)
  2. 继续训练, 使用混合数据: epoch+1
     1. 提示词中明确指定原子数, 使用COT流程, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量)
     2. 键合检验任务, 使用COT流程, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量)
  3. 评估长度4: 
     1. Correct: 78.67%, Predict selfies valid Accuracy: 98.67%, Double Bonds Match Accuracy: 86.67%, Triple Bonds Match Accuracy: 92.00%
     2. Correct: 76.00%, Predict selfies valid Accuracy: 97.33%, Double Bonds Match Accuracy: 80.00%, Triple Bonds Match Accuracy: 86.67%

  4. 评估长度5: 
     1. Correct: 68.00%, Predict selfies valid Accuracy: 93.00%, Double Bonds Match Accuracy: 81.00%, Triple Bonds Match Accuracy: 89.00%
     2. Correct: 66.00%, Predict selfies valid Accuracy: 93.00%, Double Bonds Match Accuracy: 81.00%, Triple Bonds Match Accuracy: 87.00%

  5. 评估长度6: 
     1. Correct: 54.00%, Predict selfies valid Accuracy: 89.00%, Double Bonds Match Accuracy: 77.00%, Triple Bonds Match Accuracy: 96.00%
     2. Correct: 49.00%, Predict selfies valid Accuracy: 87.00%, Double Bonds Match Accuracy: 74.00%, Triple Bonds Match Accuracy: 90.00%

对比基线(epoch=2), Predict selfies valid Accuracy未见明显提升, 但长度6的正确率率提高5-10%

增加键合检验COT任务, 使用混合数据增强理解

  1. 开始训练, 使用混合数据: epoch+2
     1. 提示词中明确指定原子数, 使用COT流程, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量)
     2. 键合检验任务, 使用COT流程, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量)
  2. 继续训练, 使用混合数据: epoch+2
     1. 提示词中明确指定原子数, 使用COT流程, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量)
     2. 键合检验任务, 使用COT流程, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量)
  3. 评估长度4: 
     1. Correct: 84.00%, Predict selfies valid Accuracy: 97.33%, Double Bonds Match Accuracy: 89.33%, Triple Bonds Match Accuracy: 93.33%

  4. 评估长度5: 
     1. Correct: 72.00%, Predict selfies valid Accuracy: 96.00%, Double Bonds Match Accuracy: 86.00%, Triple Bonds Match Accuracy: 90.00%

  5. 评估长度6: 
     1. Correct: 61.00%, Predict selfies valid Accuracy: 89.00%, Double Bonds Match Accuracy: 87.00%, Triple Bonds Match Accuracy: 95.00%

对比基线(epoch=4), Predict selfies valid Accuracy未见明显提升, 但长度6的正确率率提高14%

增加键合检验COT任务, 使用混合数据增强理解

  1. 开始训练, 使用混合数据: epoch+4
     1. 提示词中明确指定原子数, 使用COT流程, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量)
     2. 键合检验任务, 使用COT流程, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量)
  2. 继续训练, 使用混合数据: epoch+4
     1. 提示词中明确指定原子数, 使用COT流程, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量)
     2. 键合检验任务, 使用COT流程, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量)
  3. 评估长度4: 
     1. Correct: 92.00%, Predict selfies valid Accuracy: 100.00%, Double Bonds Match Accuracy: 93.33%, Triple Bonds Match Accuracy: 97.33%

  4. 评估长度5: 
     1. Correct: 87.00%, Predict selfies valid Accuracy: 95.00%, Double Bonds Match Accuracy: 94.00%, Triple Bonds Match Accuracy: 95.00%

  5. 评估长度6: 
     1. Correct: 71.00%, Predict selfies valid Accuracy: 94.00%, Double Bonds Match Accuracy: 90.00%, Triple Bonds Match Accuracy: 95.00%

对比基线(epoch=6), Predict selfies valid Accuracy在长度6提升了8%, 长度6的正确率率提高12%

基线(epoch=8)性能 比 基线(epoch=6) 要差

日志: [evaluate_9.log](/assets/01KJBZPD5CR2NC9F6T3QCWX175/evaluate_9.log)

增加键合检验COT任务, 使用混合数据增强理解

  1. 开始训练, 使用混合数据: epoch+6
     1. 提示词中明确指定原子数, 使用COT流程, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量)
     2. 键合检验任务, 使用COT流程, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量)
  2. 继续训练, 使用混合数据: epoch+6
     1. 提示词中明确指定原子数, 使用COT流程, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量)
     2. 键合检验任务, 使用COT流程, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量)
  3. 评估长度4: 
     1. Correct: 85.33%, Predict selfies valid Accuracy: 100.00%, Double Bonds Match Accuracy: 86.67%, Triple Bonds Match Accuracy: 98.67%

  4. 评估长度5: 
     1. Correct: 81.00%, Predict selfies valid Accuracy: 95.00%, Double Bonds Match Accuracy: 88.00%, Triple Bonds Match Accuracy: 94.00%

  5. 评估长度6: 
     1. Correct: 62.00%, Predict selfies valid Accuracy: 92.00%, Double Bonds Match Accuracy: 84.00%, Triple Bonds Match Accuracy: 94.00%

比起epoch=4, 性能开始下降

  

  

增加键合检验COT任务, 使用混合数据增强理解

  1. 开始训练, 使用混合数据: epoch+8
     1. 提示词中明确指定原子数, 使用COT流程, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量)
     2. 键合检验任务, 使用COT流程, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量)
  2. 继续训练, 使用混合数据: epoch+8
     1. 提示词中明确指定原子数, 使用COT流程, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量)
     2. 键合检验任务, 使用COT流程, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量)
  3. 评估长度4: 
     1. Correct: 90.67%, Predict selfies valid Accuracy: 100.00%, Double Bonds Match Accuracy: 90.67%, Triple Bonds Match Accuracy: 97.33%

  4. 评估长度5: 
     1. Correct: 80.00%, Predict selfies valid Accuracy: 97.00%, Double Bonds Match Accuracy: 90.00%, Triple Bonds Match Accuracy: 93.00%

  5. 评估长度6: 
     1. Correct: 66.00%, Predict selfies valid Accuracy: 96.00%, Double Bonds Match Accuracy: 90.00%, Triple Bonds Match Accuracy: 97.00%

比起epoch=4, 性能开始下降  

  

增加键合检验COT任务, 进行课程训练

  1. 开始训练, 键合检验任务, 使用COT流程, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量): epoch+1
  2. 继续训练, 键合检验任务, 使用COT流程, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量): epoch+1
  3. 继续训练, 最终任务, 提示词中明确指定原子数, 使用COT流程, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量): epoch+1
  4. 继续训练, 最终任务, 提示词中明确指定原子数, 使用COT流程, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量): epoch+1
  5. 评估长度4: 
     1. Correct: 77.33%, Predict selfies valid Accuracy: 96.00%, Double Bonds Match Accuracy: 85.33%, Triple Bonds Match Accuracy: 90.67%
  6. 评估长度5: 
     1. Correct: 72.00%, Predict selfies valid Accuracy: 93.00%, Double Bonds Match Accuracy: 82.00%, Triple Bonds Match Accuracy: 88.00%
  7. 评估长度6: 
     1. Correct: 53.00%, Predict selfies valid Accuracy: 91.00%, Double Bonds Match Accuracy: 81.00%, Triple Bonds Match Accuracy: 95.00%

对比基线(epoch=2), Predict selfies valid Accuracy未见明显提升, 但长度6的正确率率提高10%

增加键合检验COT任务, 进行课程训练

  1. 开始训练, 键合检验任务, 使用COT流程, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量): epoch+2
  2. 继续训练, 键合检验任务, 使用COT流程, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量): epoch+2
  3. 继续训练, 最终任务, 提示词中明确指定原子数, 使用COT流程, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量): epoch+2
  4. 继续训练, 最终任务, 提示词中明确指定原子数, 使用COT流程, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量): epoch+2
  5. 评估长度4: 
     1. Correct: 90.67%, Predict selfies valid Accuracy: 100.00%, Double Bonds Match Accuracy: 90.67%, Triple Bonds Match Accuracy: 98.67%
  6. 评估长度5: 
     1. Correct: 77.00%, Predict selfies valid Accuracy: 94.00%, Double Bonds Match Accuracy: 85.00%, Triple Bonds Match Accuracy: 93.00%
  7. 评估长度6: 
     1. Correct: 68.00%, Predict selfies valid Accuracy: 94.00%, Double Bonds Match Accuracy: 87.00%, Triple Bonds Match Accuracy: 96.00%

对比基线(epoch=4), Predict selfies valid Accuracy未见明显提升, 但长度4的正确率提高10%, 长度6的正确率率提高20%

增加键合检验COT任务, 进行课程训练

  1. 开始训练, 键合检验任务, 使用COT流程, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量): epoch+4
  2. 继续训练, 键合检验任务, 使用COT流程, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量): epoch+4
  3. 继续训练, 最终任务, 提示词中明确指定原子数, 使用COT流程, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量): epoch+4
  4. 继续训练, 最终任务, 提示词中明确指定原子数, 使用COT流程, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量): epoch+4
  5. 评估长度4: 
     1. Correct: 93.33%, Predict selfies valid Accuracy: 100.00%, Double Bonds Match Accuracy: 93.33%, Triple Bonds Match Accuracy: 98.67%

  6. 评估长度5: 
     1. Correct: 88.00%, Predict selfies valid Accuracy: 99.00%, Double Bonds Match Accuracy: 93.00%, Triple Bonds Match Accuracy: 95.00%

  7. 评估长度6: 
     1. Correct: 71.00%, Predict selfies valid Accuracy: 95.00%, Double Bonds Match Accuracy: 87.00%, Triple Bonds Match Accuracy: 93.00%

对比基线(epoch=6), 长度5准确率提高7%, 长度6准确率提高12% (Predict selfies valid Accuracy 提高9%)

与使用混合数据相比, 长度6的正确率差不多, epoch=2的正确率高增长停止

日志: [evaluate_18.log](/assets/01KJBZPD5CR2NC9F6T3QCWX175/evaluate_18.log)

增加键合检验COT任务, 进行课程训练

  1. 开始训练, 键合检验任务, 使用COT流程, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量): epoch+1
  2. 继续训练, 键合检验任务, 使用COT流程, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量): epoch+1
  3. 继续训练, 使用混合数据: epoch+1
     1. 提示词中明确指定原子数, 使用COT流程, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量)
     2. 键合检验任务, 使用COT流程, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量)
  4. 继续训练, 使用混合数据: epoch+1
     1. 提示词中明确指定原子数, 使用COT流程, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量)
     2. 键合检验任务, 使用COT流程, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量)
  5. 评估长度4: 
     1. Correct: 85.33%, Predict selfies valid Accuracy: 96.00%, Double Bonds Match Accuracy: 86.67%, Triple Bonds Match Accuracy: 90.67%

  6. 评估长度5: 
     1. Correct: 76.00%, Predict selfies valid Accuracy: 91.00%, Double Bonds Match Accuracy: 82.00%, Triple Bonds Match Accuracy: 89.00%

  7. 评估长度6: 
     1. Correct: 55.00%, Predict selfies valid Accuracy: 85.00%, Double Bonds Match Accuracy: 84.00%, Triple Bonds Match Accuracy: 95.00%

对比基线(介于 epoch=2 与 epoch=4 之间), 长度6的正确率率提高10%

阶段: 

  - 目前效果最好的, 是使用 键合检验任务(COT流程) + 最终任务(COT流程), 无论是混合式训练或者课程训练, (epoch=4, 数据量相当于基线中的epoch=8)对长度6的推理正确率提升到了71%
  - 需要补充epoch=8的基线
  - 需要建立一根新的基线, 只使用最终任务 (COT流程)
  - 需要研究71%正确率后, 其错误分布发生在哪里

### 只使用最终任务(CoT流程), 建立CoT的基线

提示词中明确指定原子数, 使用CoT流程

  1. 开始训练, 提示词中明确指定原子数, 使用CoT流程, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+1
  2. 继续训练, 提示词中明确指定原子数, 使用CoT流程, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+1
  3. 评估长度4: Correct: 74.67%, Predict selfies valid Accuracy: 92.00%, Double Bonds Match Accuracy: 77.33%, Triple Bonds Match Accuracy: 84.00%
  4. 评估长度5: Correct: 64.00%, Predict selfies valid Accuracy: 90.00%, Double Bonds Match Accuracy: 78.00%, Triple Bonds Match Accuracy: 83.00%
  5. 评估长度6: Correct: 49.00%, Predict selfies valid Accuracy: 86.00%, Double Bonds Match Accuracy: 78.00%, Triple Bonds Match Accuracy: 91.00%

比起之前 非CoT流程(仅要求做键合检测)的基线, 准确率都提高10%左右, Predict selfies valid Accuracy等没有提高

提示词中明确指定原子数, 使用CoT流程

  1. 开始训练, 提示词中明确指定原子数, 使用CoT流程, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+2
  2. 继续训练, 提示词中明确指定原子数, 使用CoT流程, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+2
  3. 评估长度4: Correct: 82.67%, Predict selfies valid Accuracy: 97.33%, Double Bonds Match Accuracy: 85.33%, Triple Bonds Match Accuracy: 92.00%
  4. 评估长度5: Correct: 73.00%, Predict selfies valid Accuracy: 96.00%, Double Bonds Match Accuracy: 82.00%, Triple Bonds Match Accuracy: 89.00%
  5. 评估长度6: Correct: 61.00%, Predict selfies valid Accuracy: 92.00%, Double Bonds Match Accuracy: 85.00%, Triple Bonds Match Accuracy: 95.00%

比起之前 非CoT流程(仅要求做键合检测)的基线, 长度4/5的准确率提高不到5%, 长度6的准确率提高18% (Double Bonds Match Accuracy提高8%, 其他准确率提升不明显)

提示词中明确指定原子数, 使用CoT流程

  1. 开始训练, 提示词中明确指定原子数, 使用CoT流程, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+4
  2. 继续训练, 提示词中明确指定原子数, 使用CoT流程, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+4
  3. 评估长度4: Correct: 85.33%, Predict selfies valid Accuracy: 97.33%, Double Bonds Match Accuracy: 89.33%, Triple Bonds Match Accuracy: 94.67%
  4. 评估长度5: Correct: 84.00%, Predict selfies valid Accuracy: 96.00%, Double Bonds Match Accuracy: 95.00%, Triple Bonds Match Accuracy: 91.00%
  5. 评估长度6: Correct: 67.00%, Predict selfies valid Accuracy: 94.00%, Double Bonds Match Accuracy: 90.00%, Triple Bonds Match Accuracy: 95.00%

比起之前 非CoT流程(仅要求做键合检测)的基线, 长度4/5/5的准确率分别提高5%/10%/20%, Double Bonds Match Accuracy分别提高 3%/9%/13%, 其他准确率提升不明显

对比现在的最优解 (长度4/5/6准确率92%/87%/71%), 准确率下降3%-6%

提示词中明确指定原子数, 使用CoT流程

  1. 开始训练, 提示词中明确指定原子数, 使用CoT流程, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+8
  2. 继续训练, 提示词中明确指定原子数, 使用CoT流程, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+8
  3. 评估长度4: Correct: 81.33%, Predict selfies valid Accuracy: 97.33%, Double Bonds Match Accuracy: 89.33%, Triple Bonds Match Accuracy: 96.00%
  4. 评估长度5: Correct: 80.00%, Predict selfies valid Accuracy: 96.00%, Double Bonds Match Accuracy: 90.00%, Triple Bonds Match Accuracy: 94.00%
  5. 评估长度6: Correct: 69.00%, Predict selfies valid Accuracy: 94.00%, Double Bonds Match Accuracy: 89.00%, Triple Bonds Match Accuracy: 97.00%

比起epoch=4, 没有提高

# 接新篇章进行情况整理
