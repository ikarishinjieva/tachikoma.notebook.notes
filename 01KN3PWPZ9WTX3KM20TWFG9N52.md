---
title: 20260401 - 电池CT.ai
created_at: 2026-04-01T05:05:08.074176+00:00
updated_at: 2026-04-01T08:02:19.108894+00:00
---

# 数据集准备

<https://pmc.ncbi.nlm.nih.gov/articles/PMC11251086/>

其有一个大数据集, 其下载需要AI来整理出脚本:

```
declare -A files=(
  ["Demo_18650_NaIon.zip"]="44809771"
  ["Demo_Prismatic.zip"]="44809954"
  ["Demo_Pouch.zip"]="44810215"
  ["Demo_18650.zip"]="44813026"
  ["Demo_4680.zip"]="44815120"
  ["Demo_2170_HED.zip"]="44815270"
  ["Scan_inventory.csv"]="44816284"
  ["Demo_2170.zip"]="44816857"
)
for name in "${!files[@]}"; do
  curl -L -C - -o "$name" \
    "https://ndownloader.figshare.com/files/${files[$name]}"
done
```

- 待下载: python3 figshare\_[download.py](http://download.py) 44809954 44810215 44813026 44815120 44815270 44816284 44816857
- 数据集内容: 很像目标场景
  - 

    ```
    数据集主要被分为 7 类。
    
    以下是这7类电池的具体内容和参数（基于论文中的 Table I）：
    
    EVE INR18650/33V
    
    化学成分：锂离子 (Lithium-ion)
    封装形式：圆柱形 (18650)
    电池数量：400 个
    HAKADI SIB18650/3V
    
    化学成分：锂离子 (Lithium-ion) 注：论文提到该批次中发现了4个伪造的钠离子电池，实为锂离子电池。
    封装形式：圆柱形 (18650)
    电池数量：49 个
    Samsung 50E
    
    化学成分：锂离子 (Lithium-ion)
    封装形式：圆柱形 (2170)
    电池数量：500 个
    Vapcell F56
    
    化学成分：钠离子 (Sodium-ion)
    封装形式：圆柱形 (2170)
    电池数量：25 个
    BYD FC4680
    
    化学成分：锂离子 (Lithium-ion)
    封装形式：圆柱形 (4680)
    电池数量：25 个
    Tenergy 6050100
    
    化学成分：锂离子 (Lithium-ion)
    封装形式：软包 (Pouch)
    电池数量：10 个
    PowerSonic PSL-FP-IFP2770180EC
    
    化学成分：锂离子 (Lithium-ion)
    封装形式：方形/棱柱形 (Prismatic)
    电池数量：5 个
    ```

另一个数据集: 伦敦大学学院 (UCL) 电池 3D X-ray CT 数据集

[https://rdr.ucl.ac.uk/articles/dataset/Lithium-ion_Battery_INR18650_MJ1_3D_X-ray_CT_Data_NMC811_Cathode_EIL-014\\\_/12159366](https://rdr.ucl.ac.uk/articles/dataset/Lithium-ion_Battery_INR18650_MJ1_3D_X-ray_CT_Data_NMC811_Cathode_EIL-014%5C_/12159366)

```
https://rdr.ucl.ac.uk/ndownloader/files/22360440
```

数据集举例: 与目标场景不同

![](/assets/01KN3PWPZ9WTX3KM20TWFG9N52/23b94870769a4313ae0b7040b87db4c5.png)