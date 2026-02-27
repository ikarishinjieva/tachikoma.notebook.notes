---
title: 20241021 - 研究化学论文中的截图分类
confluence_page_id: 3342759
created_at: 2024-10-21T10:01:45+00:00
updated_at: 2024-10-22T15:19:37+00:00
---

通过化学结构名生成smiles表达式: <https://cactus.nci.nih.gov/chemical/structure>

# 样例1

(kuznetsova2019.pdf, page 174)

  1. 截图: ![from-paper-1.png](/assets/01KJBZHDXH5GGF3B25C8H81E93/from-paper-1.png)
  2. 路径1: 
     1. 用genimi, 生成名称: 六氟磷酸二苯基碘 (diphenyliodonium hexafluorophosphate)
     2. <https://cactus.nci.nih.gov/chemical/structure> , 按名字搜索, 生成smiles表达式: 
        1. F[P-](F)(F)(F)(F)F.[I+](c1ccccc1)c2ccccc2

     3. 生成图片: 
        1. ![image](https://cactus.nci.nih.gov/chemical/structure/F%5BP-%5D(F)(F)(F)(F)F.%5BI+%5D(c1ccccc1)c2ccccc2/image?width=278&height=278)
     4. 转换正确
  3. 路径2:
     1. 通过Img2Mol, 生成表达式: CC(c1ccccc1)c1ccccc1C(F)F, 生成图片: 
        1. ![image2024-10-21 16:20:26.png](/assets/01KJBZHDXH5GGF3B25C8H81E93/image2024-10-21%2016%3A20%3A26.png)
     2. 生成错误
     3. 研究cddd模型, 使用自回归的方法: 

```
from cddd.inference import InferenceModel as CDDDInferenceModel
cddd_inference_model = CDDDInferenceModel()
smiles = "F[P-](F)(F)(F)(F)F.[I+](c1ccccc1)c2ccccc2"
cddd = cddd_inference_model.seq_to_emb(smiles)
infer_smiles = cddd_inference_model.emb_to_seq(cddd)
print(infer_smiles)
```

        2. 结果: F[P+](F)(F)(F)(F)[I+](c1ccccc1)c1ccccc1
           1. ![image](https://cactus.nci.nih.gov/chemical/structure/F%5BP+%5D(F)(F)(F)(F)%5BI+%5D(c1ccccc1)c1ccccc1/image?width=278&height=278)
        3. 集团联结, 但比Img2Mol的结果略好. 认为CDDD中包含的知识很接近正确答案, 但Img -> CDDD这一步劣化严重

# 样例2

(kuznetsova2019.pdf, page 174)

  1. 截图: ![from-paper-2.png](/assets/01KJBZHDXH5GGF3B25C8H81E93/from-paper-2.png)
  2. 路径1:
     1. 用genimi, 生成名称: [4-(Phenoxy)phenyl]diphenylsulfonium trifluoromethanesulfonate
        1. 此处需要一些引导, 默认生成的名称会丢掉右上角的取代基(S+一个苯环)
     2. 生成smiles表达式:
        1. [O-][S](=O)(=O)C(F)(F)F.O(c1ccccc1)c2ccc(cc2)[S+](c3ccccc3)c4ccccc4
     3. 生成图片:
        1. ![image2024-10-21 18:4:16.png](/assets/01KJBZHDXH5GGF3B25C8H81E93/image2024-10-21%2018%3A4%3A16.png)
        2. 其中红色的氧原子都是错的
     4. 手工修正为正确的smiles: F[Sb-](F)(F)(F)(F)F.S(c1ccccc1)c2ccc(cc2)[S+](c3ccccc3)c4ccccc4
        1. 探索其命名: <https://cactus.nci.nih.gov/chemical/structure/F%5BSb-%5D(F)(F)(F)(F)F.S(c1ccccc1)c2ccc(cc2)%5BS+%5D(c3ccccc3)c4ccccc4/names>
        2. diphenyl-(4-phenylsulfanylphenyl)sulfanium; hexafluoroantimony

        3. smiles: [F]|[Sb-](|[F])(|[F])(|[F])(|[F])|[F].S(c1ccccc1)c2ccc(cc2)[S+](c3ccccc3)c4ccccc4
        4. 画图: 
           1. ![image2024-10-21 18:0:58.png](/assets/01KJBZHDXH5GGF3B25C8H81E93/image2024-10-21%2018%3A0%3A58.png)

# 样例3

截图: 

![from-paper-3.png](/assets/01KJBZHDXH5GGF3B25C8H81E93/from-paper-3.png)

路径1:

  - 用gemini生成名称: 
    - 需要人工引导

```
附图是一个分子结构式, 用 取代基命名法/加和命名法 (举例: "diphenyl-(4-phenylsulfanylphenyl)sulfanium; hexafluoroantimony"), 输出该分子结构式的命名
 
---
 
这个答案中, 好像丢失了右上角的O和苯环
``` 
    - 得到的名称: [(4-苯氧基苯基)二苯基硫]三氟甲磺酸盐 (Diphenyl(4-phenoxyphenyl)sulfonium triflate)
    - cactus生成 Smiles: [O-][S](=O)(=O)C(F)(F)F.O(c1ccccc1)c2ccc(cc2)[S+](c3ccccc3)c4ccccc4
    - Smiles生成图像: 
      - ![image](https://cactus.nci.nih.gov/chemical/structure/%5BO-%5D%5BS%5D(=O)(=O)C(F)(F)F.O(c1ccccc1)c2ccc(cc2)%5BS+%5D(c3ccccc3)c4ccccc4/image?width=278&height=278)

# 样例4

截图: ![from-paper-4.png](/assets/01KJBZHDXH5GGF3B25C8H81E93/from-paper-4.png)

路径1:

  - 用gemini生成名称: [methyltris(pentafluorophenyl)borate] diphenyl(4-propylphenyl)iodonium
  - 生成smiles表达式: CCCc1ccc(cc1)[IH+](c2ccccc2)c3ccccc3.C[B-](c4c(F)c(F)c(F)c(F)c4F)(c5c(F)c(F)c(F)c(F)c5F)c6c(F)c(F)c(F)c(F)c6F
  - 内容不正确
  - 图片: 

![image](https://cactus.nci.nih.gov/chemical/structure/%5Bmethyltris(pentafluorophenyl)borate%5D%20diphenyl(4-propylphenyl)iodonium/image?width=278&height=278)

重试路径1:

  - 用引导方式gemini生成名称: 

```
按照图片的方式, 用中文描述这张图片的各部分 (不要按照分子结构式的方法描述)
---
用 取代基命名法/加和命名法 (举例: "diphenyl-(4-phenylsulfanylphenyl)sulfanium; hexafluoroantimony"), 输出该分子结构式的命名
---
这个描述不对, 硼原子连接了四个苯环
---
根据以上描述, 用 取代基命名法/加和命名法 (举例: "diphenyl-(4-phenylsulfanylphenyl)sulfanium; hexafluoroantimony"), 输出该分子结构式的命名
---
在这个命名中, 应该有几个碘离子
```

    - 名称: Tetrakis(perfluorophenyl)borate phenyl(4-propylphenyl)iodonium **iodide**
    - 去除碘化物后缀: Tetrakis(perfluorophenyl)borate phenyl(4-propylphenyl)iodonium

    - Smiles: 

CCCc1ccc([I+]c2ccccc2)cc1.Fc3c(F)c(F)c(c(F)c3F)[B-](c4c(F)c(F)c(F)c(F)c4F)(c5c(F)c(F)c(F)c(F)c5F)c6c(F)c(F)c(F)c(F)c6F

    - 图像: 

      - ![image](https://cactus.nci.nih.gov/chemical/structure/Tetrakis(perfluorophenyl)borate%20phenyl(4-propylphenyl)iodonium/image?width=278&height=278)
      - 除了Me和Pri集团, 其他结构正确

    - 增加引导过程: 

```
图片中的Me和Pri是什么意思? 在命名中有表达么?
4-甲基-2,3,5,6-四氟苯基, 为什么认为这部分的四个位置被氟原子取代?
```

      - 得到名称: Tetrakis(perfluorophenyl)borate (4-propylphenyl)(4-methylphenyl)iodonium iodide
      - 去除碘化物后缀: Tetrakis(perfluorophenyl)borate (4-propylphenyl)(4-methylphenyl)iodonium
      - Smiles: 

CCCc1ccc([I+]c2ccc(C)cc2)cc1.Fc3c(F)c(F)c(c(F)c3F)[B-](c4c(F)c(F)c(F)c(F)c4F)(c5c(F)c(F)c(F)c(F)c5F)c6c(F)c(F)c(F)c(F)c6F

      - 图像: 

        - ![image](https://cactus.nci.nih.gov/chemical/structure/Tetrakis(perfluorophenyl)borate%20(4-propylphenyl)(4-methylphenyl)iodonium/image?width=278&height=278)
        - 完全正确

# 样例5

![e25c2804bffc45323ebb08716f22467c1466991baf6f2a768ffc1ed586173ac8.png](/assets/01KJBZHDXH5GGF3B25C8H81E93/e25c2804bffc45323ebb08716f22467c1466991baf6f2a768ffc1ed586173ac8.png)

大模型无法正确处理这个模型 (其中有空间部分的内容, 所以不容易转换)
