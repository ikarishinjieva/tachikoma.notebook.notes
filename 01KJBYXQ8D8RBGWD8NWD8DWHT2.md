---
title: 20221101 - 全密态数据库材料
confluence_page_id: 2129927
created_at: 2022-10-31T16:43:31+00:00
updated_at: 2022-11-03T05:34:06+00:00
---

# 材料1

[附件: 56176409-5afa-4e86-85f7-1060116c01af.pdf] 

## 材料2

[20220814战新答辩材料-上海爱可生-云树安全可信数据库.pptx](/assets/01KJBYXQ8D8RBGWD8NWD8DWHT2/20220814%E6%88%98%E6%96%B0%E7%AD%94%E8%BE%A9%E6%9D%90%E6%96%99-%E4%B8%8A%E6%B5%B7%E7%88%B1%E5%8F%AF%E7%94%9F-%E4%BA%91%E6%A0%91%E5%AE%89%E5%85%A8%E5%8F%AF%E4%BF%A1%E6%95%B0%E6%8D%AE%E5%BA%93.pptx)

\---

![image2022-11-1 11:44:19.png](/assets/01KJBYXQ8D8RBGWD8NWD8DWHT2/image2022-11-1%2011%3A44%3A19.png)

ECDH协议 用于交换 公共的秘钥

\---

![image2022-11-1 13:38:40.png](/assets/01KJBYXQ8D8RBGWD8NWD8DWHT2/image2022-11-1%2013%3A38%3A40.png)

这两个概念不好区分

\---

![image2022-11-1 13:41:2.png](/assets/01KJBYXQ8D8RBGWD8NWD8DWHT2/image2022-11-1%2013%3A41%3A2.png)

\---

建议: 按照数据的生命周期, 图示 加密/脱敏 的体系, 以及在其中的工作

\---

提醒: 目前的全密态数据库设计, 并不是以 授权分离 为目的

![image2022-11-1 13:50:55.png](/assets/01KJBYXQ8D8RBGWD8NWD8DWHT2/image2022-11-1%2013%3A50%3A55.png)

\---

材料问题: 

没有 非专业人士 能看懂的立项价值: 参考opengauss: <https://opengauss.org/zh/docs/3.0.0/docs/CharacteristicDescription/%E5%85%A8%E5%AF%86%E6%80%81%E6%95%B0%E6%8D%AE%E5%BA%93%E7%AD%89%E5%80%BC%E6%9F%A5%E8%AF%A2.html>

![image2022-11-1 14:39:24.png](/assets/01KJBYXQ8D8RBGWD8NWD8DWHT2/image2022-11-1%2014%3A39%3A24.png)

\---

材料问题: 

投资价值 = 业界难点: 

  - 支持足够多的SQL类型和算子类型
  - 更强的密级 (软硬结合)
  - 更好的性能 (全同态算法慢, 无法实用)

\---

![image2022-11-1 15:24:53.png](/assets/01KJBYXQ8D8RBGWD8NWD8DWHT2/image2022-11-1%2015%3A24%3A53.png)

算法意义? 

\---

材料问题: 

需求层次: 

  - 开发安全辅助: SQL审核
  - 数据权/运维权 分离: 密态计算 (库级别)
  - 防止硬件入侵: TEE
  - 存储安全 (非必要): TDE
  - 传输安全: SSL
  - 数据使用合规: 脱敏

\---

# 整理

  - 可信计算引擎 (TEE)
    - Intel SGX
    - ARM TrustZone  
  

  - 密文计算引擎 (REE)
    - Crypt kernel
    - Range Identify: 范围查询
    - Cipher index  
  

  - 架构考虑方向
    - 秘钥安全体系
    - ECDH + TEE
    - 安全计算
      - 等值查询
      - 范围查询
      - 排序比较
      - 索引
      - 模糊查询
    - 可信算子
      - 数学算子、类型转换、聚集函数、条件表达式、Hash函数、数组操作、范围函数、排序函数、模式匹配
  - TrustedDB
    - <https://zxr.io/research/sion2011sigmod-trusteddb.pdf>
    - 算子分布
      - ![image2022-11-1 16:7:1.png](/assets/01KJBYXQ8D8RBGWD8NWD8DWHT2/image2022-11-1%2016%3A7%3A1.png)
  - Intel SGX和Arm TrustZone的区别: <https://www.sohu.com/a/373973519_655101>
  - secGear可以抹平 Intel SGX 和 Arm TrustZone 的使用差异, 并提供了Hello World
  - 其他知识
    - SM2 & SM4: [参考算法](<https://www.jianshu.com/p/e80d33b3178f?u_atoken=cf447d07-10a7-44ee-bdcb-538247e9211f&u_asession=01fy3DFe09Ji7_A94I03fhaFYNN84XYAlqozGpaBMxxYYKzvVA-EgQEW8Hvk1knp01X0KNBwm7Lovlpxjd_P_q4JsKWYrT3W_NKPr8w6oU7K86OyAmAeHon_so4L7JDFqD7TzbK64jWsJYZqMKGduDQWBkFo3NEHBv0PZUm6pbxQU&u_asig=05Y_GO1vKbHGrEmwMy-8Bd_ju9MQRITuQeyri92CYjJBKD9Wx0O-EtqyMN51k004v65YHGeyvwrvrgjvDlB6UUxvwCV2i73RNFq2MzRJL7QlegQqZpTanJYVBO7wibZcCgAzwhMO6oKLB9RkKKzKn5xfo6KMX8slgOWsW67kCz6-H9JS7q8ZD7Xtz2Ly-b0kmuyAKRFSVJkkdwVUnyHAIJzX3DQI4MlNUi72Qf7leVXq1BBhCJv2UsOB2bWjIBEZ8dRZfeyxQBxz7loGfMAdy0Tu3h9VXwMyh6PgyDIVSG1W9G7v-kEAyaSBNFnM3sop36M1CcDvpYE0FByRD-zGgt7Ux5gA9s4fTSaJaIOHhdLMlgjn6OskZuLlbgYk79luaQmWspDxyAEEo4kbsryBKb9Q&u_aref=ioZ3pA8RBQpqEjCRth%2Fvk2vXnuY%3D>)
      - 类比RSA和3DES: 使用3DES对长内容加密，使用RSA对3DES使用的密钥加密，这样加密速度快，效率高。 <https://blog.csdn.net/zhan10001/article/details/79084600>
    - 参考: Cryptdb
      - <https://yiwenshao.github.io/2017/05/01/Cryptdb%E5%8E%9F%E7%90%86%E6%A6%82%E8%BF%B0/#more>
        - OPE算法可以支持大小比较, DET算法可以支持两个数是否想等的比较, HOM可以支持加法. 但是没有一种算法可以同时支持多种操作: 在论文中提出的方法是多洋葱模型,
        - SEARCH: <https://www.cossacklabs.com/blog/secure-search-over-encrypted-data-acra-se/#:~:text=The%20most%20basic%20approach%20to,the%20possible%20significant%20data%20size>.
      - <http://blog.leanote.com/post/heming/2017-07-11>
    - 鲲鹏: <https://forum.huawei.com/enterprise/zh/thread/580939194191200257>
      - ![image2022-11-1 16:41:51.png](/assets/01KJBYXQ8D8RBGWD8NWD8DWHT2/image2022-11-1%2016%3A41%3A51.png)
      - TODO: <https://www.hikunpeng.com/document/detail/zh/kunpengcctrustzone/devpg-tzrsademo/kunpengtrustzonersa_06_0001.html>

# PPT材料修改

![image2022-11-2 13:35:30.png](/assets/01KJBYXQ8D8RBGWD8NWD8DWHT2/image2022-11-2%2013%3A35%3A30.png)

去除软件自主化率, 容易引起误解

![image2022-11-3 13:19:30.png](/assets/01KJBYXQ8D8RBGWD8NWD8DWHT2/image2022-11-3%2013%3A19%3A30.png)

去除 "18进4"
