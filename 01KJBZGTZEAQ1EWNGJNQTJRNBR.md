---
title: 20241015 - 化学分子式识别
confluence_page_id: 3342674
created_at: 2024-10-15T09:58:05+00:00
updated_at: 2024-10-16T06:59:01+00:00
---

# 举例文件

[归档 16.zip](/assets/01KJBZGTZEAQ1EWNGJNQTJRNBR/%E5%BD%92%E6%A1%A3%2016.zip)

# 有用的工具

image to smile: <https://cactus.nci.nih.gov/cgi-bin/osra/index.cgi>

smile to image, 以及信息查询: <http://www.swissadme.ch/index.php>

smile to image 前端库: <https://www.cheminfo.org/Chemistry/Cheminformatics/SMILES_to_svg/index.html>

smile to image 模型: <https://github.com/bayer-science-for-a-better-life/Img2Mol>

# 探索步骤1

先使用分子式生成的SVG, 让LLM识别SVG, 转成smiles格式. 目前的步骤: 

步骤1: 对SVG中的节点进行标号 (目前是手工诱导LLM进行)

步骤2: 识别连接关系 (否则LLM会使用错误的连接关系)

```
<svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="100%" height="100%" viewBox="0 0 200 150" style="display: block;">
    <line x1="68" y1="105" x2="68" y2="81" style="stroke:rgb(0,0,0);stroke-width:1"></line>
    <line x1="89" y1="117" x2="68" y2="105" style="stroke:rgb(0,0,0);stroke-width:1"></line>
    <line x1="110" y1="105" x2="89" y2="117" style="stroke:rgb(0,0,0);stroke-width:1"></line>
    <line x1="110" y1="81" x2="110" y2="105" style="stroke:rgb(0,0,0);stroke-width:1"></line>
    <line x1="89" y1="69" x2="110" y2="81" style="stroke:rgb(0,0,0);stroke-width:1"></line>
    <line x1="89" y1="69" x2="68" y2="81" style="stroke:rgb(0,0,0);stroke-width:1"></line>
    <line x1="89" y1="45" x2="89" y2="69" style="stroke:rgb(0,0,0);stroke-width:1"></line>
    <line x1="110" y1="33" x2="89" y2="45" style="stroke:rgb(0,0,0);stroke-width:1"></line>
    <line x1="131" y1="44" x2="110" y2="33" style="stroke:rgb(0,0,0);stroke-width:1"></line>
    <line x1="131" y1="69" x2="131" y2="44" style="stroke:rgb(0,0,0);stroke-width:1"></line>
    <line x1="131" y1="69" x2="110" y2="81" style="stroke:rgb(0,0,0);stroke-width:1"></line>
    
    <!-- Atom 1 (Start) -->
    <circle cx="68" cy="105" r="3" fill="red"></circle>
    <text x="72" y="105" font-size="5" fill="red">1</text>
    
    <!-- Atom 2 -->
    <circle cx="68" cy="81" r="3" fill="blue"></circle>
    <text x="72" y="81" font-size="5" fill="blue">2</text>
    
    <!-- Atom 3 -->
    <circle cx="89" cy="69" r="3" fill="green"></circle>
    <text x="93" y="69" font-size="5" fill="green">3</text>
    
    <!-- Atom 4 -->
    <circle cx="89" cy="45" r="3" fill="orange"></circle>
    <text x="93" y="45" font-size="5" fill="orange">4</text>
    
    <!-- Atom 5 -->
    <circle cx="110" cy="33" r="3" fill="purple"></circle>
    <text x="114" y="33" font-size="5" fill="purple">5</text>
    
    <!-- Atom 6 -->
    <circle cx="131" cy="44" r="3" fill="yellow"></circle>
    <text x="135" y="44" font-size="5" fill="yellow">6</text>
    
    <!-- Atom 7 -->
    <circle cx="131" cy="69" r="3" fill="pink"></circle>
    <text x="135" y="69" font-size="5" fill="pink">7</text>
    
    <!-- Atom 8 -->
    <circle cx="110" cy="81" r="3" fill="brown"></circle>
    <text x="114" y="81" font-size="5" fill="brown">8</text>
    
    <!-- Atom 9 -->
    <circle cx="110" cy="105" r="3" fill="cyan"></circle>
    <text x="114" y="105" font-size="5" fill="cyan">9</text>
    
    <!-- Atom 10 -->
    <circle cx="89" cy="117" r="3" fill="magenta"></circle>
    <text x="93" y="117" font-size="5" fill="magenta">10</text>
</svg>

列出这张图中的所有的连接关系, 从那个节点到那个节点有连接
``` 

步骤3: 将SVG+连接关系都给到LLM, 进行生成: 

```
<svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="100%" height="100%" viewBox="0 0 200 150" style="display: block;">
    <line x1="68" y1="105" x2="68" y2="81" style="stroke:rgb(0,0,0);stroke-width:1"></line>
    <line x1="89" y1="117" x2="68" y2="105" style="stroke:rgb(0,0,0);stroke-width:1"></line>
    <line x1="110" y1="105" x2="89" y2="117" style="stroke:rgb(0,0,0);stroke-width:1"></line>
    <line x1="110" y1="81" x2="110" y2="105" style="stroke:rgb(0,0,0);stroke-width:1"></line>
    <line x1="89" y1="69" x2="110" y2="81" style="stroke:rgb(0,0,0);stroke-width:1"></line>
    <line x1="89" y1="69" x2="68" y2="81" style="stroke:rgb(0,0,0);stroke-width:1"></line>
    <line x1="89" y1="45" x2="89" y2="69" style="stroke:rgb(0,0,0);stroke-width:1"></line>
    <line x1="110" y1="33" x2="89" y2="45" style="stroke:rgb(0,0,0);stroke-width:1"></line>
    <line x1="131" y1="44" x2="110" y2="33" style="stroke:rgb(0,0,0);stroke-width:1"></line>
    <line x1="131" y1="69" x2="131" y2="44" style="stroke:rgb(0,0,0);stroke-width:1"></line>
    <line x1="131" y1="69" x2="110" y2="81" style="stroke:rgb(0,0,0);stroke-width:1"></line>
    
    <!-- Atom 1 (Start) -->
    <circle cx="68" cy="105" r="3" fill="red"></circle>
    <text x="72" y="105" font-size="5" fill="red">1</text>
    
    <!-- Atom 2 -->
    <circle cx="68" cy="81" r="3" fill="blue"></circle>
    <text x="72" y="81" font-size="5" fill="blue">2</text>
    
    <!-- Atom 3 -->
    <circle cx="89" cy="69" r="3" fill="green"></circle>
    <text x="93" y="69" font-size="5" fill="green">3</text>
    
    <!-- Atom 4 -->
    <circle cx="89" cy="45" r="3" fill="orange"></circle>
    <text x="93" y="45" font-size="5" fill="orange">4</text>
    
    <!-- Atom 5 -->
    <circle cx="110" cy="33" r="3" fill="purple"></circle>
    <text x="114" y="33" font-size="5" fill="purple">5</text>
    
    <!-- Atom 6 -->
    <circle cx="131" cy="44" r="3" fill="yellow"></circle>
    <text x="135" y="44" font-size="5" fill="yellow">6</text>
    
    <!-- Atom 7 -->
    <circle cx="131" cy="69" r="3" fill="pink"></circle>
    <text x="135" y="69" font-size="5" fill="pink">7</text>
    
    <!-- Atom 8 -->
    <circle cx="110" cy="81" r="3" fill="brown"></circle>
    <text x="114" y="81" font-size="5" fill="brown">8</text>
    
    <!-- Atom 9 -->
    <circle cx="110" cy="105" r="3" fill="cyan"></circle>
    <text x="114" y="105" font-size="5" fill="cyan">9</text>
    
    <!-- Atom 10 -->
    <circle cx="89" cy="117" r="3" fill="magenta"></circle>
    <text x="93" y="117" font-size="5" fill="magenta">10</text>
</svg>

这张图中的所有的连接关系如下: 
---
节点1 连接到 节点2
节点1 连接到 节点10
节点2 连接到 节点3
节点3 连接到 节点4
节点3 连接到 节点8
节点4 连接到 节点5
节点5 连接到 节点6
节点6 连接到 节点7
节点7 连接到 节点8
节点8 连接到 节点9
节点9 连接到 节点10
---

你需要按照原子顺序从小到大, 逐步扩展原子并建立smiles表达式, 每一步都按照如下流程进行思考: 

1. 扩展新的原子: 原子x. smiles-plus表达式: ...
2. 该原子是否和扩展过的原子封闭成环: ... 更新后的smiles表达式: ...
```
```
Let's break down the construction of the SMILES string step by step, following the provided connection list and the specified procedure.

**Step 1: Atom 1**

1. *Expanding Atom 1.*  SMILES-plus: `[1]`
2. *Ring Closure?* No ring closure is possible with only one atom. Updated SMILES: `[1]`

**Step 2: Atom 2**

1. *Expanding Atom 2.* Atom 2 is connected to Atom 1. SMILES-plus: `[1][2]`
2. *Ring Closure?* No ring closure. Updated SMILES: `[1][2]`

**Step 3: Atom 3**

1. *Expanding Atom 3.* Atom 3 is connected to Atom 2. SMILES-plus: `[1][2][3]`
2. *Ring Closure?* No ring closure. Updated SMILES: `[1][2][3]`

**Step 4: Atom 4**

1. *Expanding Atom 4.* Atom 4 is connected to Atom 3. SMILES-plus: `[1][2][3][4]`
2. *Ring Closure?* No ring closure. Updated SMILES: `[1][2][3][4]`

**Step 5: Atom 5**

1. *Expanding Atom 5.* Atom 5 is connected to Atom 4. SMILES-plus: `[1][2][3][4][5]`
2. *Ring Closure?* No ring closure. Updated SMILES: `[1][2][3][4][5]`

**Step 6: Atom 6**

1. *Expanding Atom 6.* Atom 6 is connected to Atom 5. SMILES-plus: `[1][2][3][4][5][6]`
2. *Ring Closure?* No ring closure. Updated SMILES: `[1][2][3][4][5][6]`

**Step 7: Atom 7**

1. *Expanding Atom 7.* Atom 7 is connected to Atom 6. SMILES-plus: `[1][2][3][4][5][6][7]`
2. *Ring Closure?* No ring closure. Updated SMILES: `[1][2][3][4][5][6][7]`

**Step 8: Atom 8**

1. *Expanding Atom 8.* Atom 8 is connected to Atom 7 and Atom 3. SMILES-plus: `[1][2][3][4][5][6][7][8]`
2. *Ring Closure?*  Atom 8 connects back to Atom 3, closing a ring. We number this ring closure "1". Updated SMILES: `[1][2][3][4][5][6][7]1[8]1`  (or equivalently, `[1][2][3]1[4][5][6][7][8]1`)

**Step 9: Atom 9**

1. *Expanding Atom 9.* Atom 9 is connected to Atom 8. SMILES-plus: `[1][2][3]1[4][5][6][7][8][9]1`
2. *Ring Closure?* No new ring closure. Updated SMILES: `[1][2][3]1[4][5][6][7][8][9]1`

**Step 10: Atom 10**

1. *Expanding Atom 10.* Atom 10 is connected to Atom 9 and Atom 1. SMILES-plus: `[1][2][3]1[4][5][6][7][8][9][10]1`
2. *Ring Closure?* Atom 10 connects back to Atom 1, closing a second ring. We number this ring closure "2". Updated SMILES: `[1]2[2][3]1[4][5][6][7][8][9][10]12`

Therefore, the final SMILES string representing the given structure is `[1]2[2][3]1[4][5][6][7][8][9][10]12`.  This SMILES string accurately reflects the connectivity and the two ring closures present in the molecule.
``` 

以上形式有一定概率能成功. 

# 思考

如果已经获取了连接信息, 通过程序进行图遍历生成效果会更稳定. 

下一步的方向, LLM能否变成一个图片标号和识别工具, 将图片转为文字描述.
