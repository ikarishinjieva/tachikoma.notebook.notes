---
title: 20220213 - 向量数据库性能测试用例集
confluence_page_id: 1736855
created_at: 2022-02-13T14:40:04+00:00
updated_at: 2022-02-22T08:56:14+00:00
---

# 数据集

使用Milvus提供的SIFT数据集

<https://github.com/milvus-io/bootcamp/blob/master/benchmark_test/lab1_sift1b_1m.md>

<https://github.com/milvus-io/bootcamp/tree/master/benchmark_test/scripts>

# 工具

jupyter notebook

<http://10.186.62.73:8888/notebooks/Untitled.ipynb?kernel_name=python3>

# 特征二值化

二值化的方法举例: 

![image2022-2-20 0:12:6.png](/assets/01KJBYH6Z36KSJE0C1YDGDDQKD/image2022-2-20%200%3A12%3A6.png)

![image2022-2-14 0:0:28.png](/assets/01KJBYH6Z36KSJE0C1YDGDDQKD/image2022-2-14%200%3A0%3A28.png)

尝试: <https://github.com/RSIA-LIESMARS-WHU/LSHBOX>

  - 编译python模块的方法
    1. 编辑CMakeLists.txt, 将'ADD_SUBDIRECTORY(python)'的注释解开
    2. 下载 <https://github.com/RSIA-LIESMARS-WHU/LSHBOX-3rdparty>, 解压到源码目录
    3. mv python/CMakeLists2.txt python/CMakeLists.txt
    4. make
  - 使用方法

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# pylshbox_example.py
import libpylshbox
import numpy as np
print('prepare test data')
float_mat = np.random.randn(100000, 192)
float_query = float_mat[0]
unsigned_mat = np.uint32(float_mat * 5)
unsigned_query = unsigned_mat[0]
print('')
print('Test rbsLsh')
rbs_mat = libpylshbox.rbslsh()
rbs_mat.init_mat(unsigned_mat.tolist(), '', 521, 5, 20, 5)
result = rbs_mat.query(unsigned_query.tolist(), 1)
indices, dists = result[0], result[1]
for i in range(len(indices)):
    print(indices[i], '\t', dists[i])
print('')
```

    - 使用python2运行正常

    - 使用python3运行会报错: ImportError: dynamic module does not define module export function (PyInit_libpylshbox)
  - 对ITQ的理解
    - 原文: <https://slazebni.cs.illinois.edu/publications/ITQ.pdf>
      - 论文中的符号: 
      - ![image2022-2-20 18:14:51.png](/assets/01KJBYH6Z36KSJE0C1YDGDDQKD/image2022-2-20%2018%3A14%3A51.png)
        - 源数据: X是n个数据, 每个数据有d个维度
        - 目标数据: B是n个数据, 每个数据有c个维度, 每个维度都是 -1或+1  
  

      - ![image2022-2-20 18:16:58.png](/assets/01KJBYH6Z36KSJE0C1YDGDDQKD/image2022-2-20%2018%3A16%3A58.png)
        - 转换函数如上, B = sgn(XW), W为转换矩阵, 维度为d*c  
  

      - ![image2022-2-20 18:49:59.png](/assets/01KJBYH6Z36KSJE0C1YDGDDQKD/image2022-2-20%2018%3A49%3A59.png)
        - V=XW: v是x在W转换后空间的投影
        - R是c*c的正交矩阵, VR为对V进行旋转
        - 量化损失为 Q(B,R), 是B (二值化以后的结果) 与 R 旋转结果的差值
    - 参考解析: <https://blog.csdn.net/liuheng0111/article/details/52242491>
  - 对代码的理解: 
    - ![image2022-2-21 0:9:4.png](/assets/01KJBYH6Z36KSJE0C1YDGDDQKD/image2022-2-21%200%3A9%3A4.png)

# 尝试用ITQ运行milvus的测试用例集?
