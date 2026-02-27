---
title: 20240323 - 对embedding进行压缩
confluence_page_id: 2949225
created_at: 2024-03-22T16:24:20+00:00
updated_at: 2024-03-24T14:33:55+00:00
---

# 目的

在对embedding进行微调时, 能微调的数据的维度数有限制.

希望将embedding的结果进行压缩, 减小维度. 这样微调embedding模型的训练数据 维度会减小, 而信息量增加, 可能有助于微调的效果优化.

# 问答

```
是的,在使用LLM制作知识库应用时,对embedding向量进行压缩是一种常见的优化方法。压缩的目的是在保留关键信息的同时,降低存储和计算开销。以下是几种常用的embedding压缩方法:

1. 主成分分析(PCA):
   - PCA是一种线性降维方法,通过正交变换将原始高维向量投影到低维空间
   - 它找到了数据中的主要成分(方差最大的方向),并按照重要性排序
   - 通过选取前k个主成分,可以将原始d维向量压缩到k维(k<d)
   - PCA可以有效地保留数据的主要特征,同时去除噪声和冗余信息

2. t-SNE(t-Distributed Stochastic Neighbor Embedding):
   - t-SNE是一种非线性降维方法,特别适用于高维数据的可视化
   - 它通过优化原始空间和低维空间中点对之间的相似度,试图保留数据的局部结构
   - t-SNE可以将高维向量压缩到2维或3维空间,便于可视化和理解数据的分布
   - 但t-SNE主要用于可视化,不太适合用于下游任务的特征表示

3. 自编码器(Autoencoder):
   - 自编码器是一种无监督学习的神经网络模型,由编码器和解码器两部分组成
   - 编码器将高维输入映射到低维的潜在表示,解码器则尝试从潜在表示重构出原始输入
   - 通过最小化重构误差,自编码器学习到了数据的压缩表示
   - 压缩后的潜在表示可以用于下游任务,如聚类、检索等

4. 量化(Quantization):
   - 量化是一种将连续值离散化的方法,可以用于压缩向量表示
   - 常见的量化方法包括二值量化(BinaryConnect)、4比特量化(Ternary)等
   - 量化后的向量由少量离散值组成,大大减少了存储空间
   - 虽然量化会损失一定的精度,但在许多任务上仍能保持较好的性能

5. 局部敏感哈希(Locality-Sensitive Hashing, LSH):
   - LSH是一种概率性的降维方法,将相似的向量映射到同一个哈希桶中
   - 通过多个哈希函数将高维向量映射为低维的二进制编码
   - 编码后的向量可以高效地进行相似性搜索和比较
   - LSH在大规模数据的近似最近邻搜索中特别有效

6. 乘积量化(Product Quantization, PQ):
   - PQ是一种将高维向量分解为多个低维子向量,并对每个子向量进行量化的方法
   - 它将原始向量空间划分为笛卡尔积形式的子空间,每个子空间独立地进行量化
   - PQ可以在保留较高精度的同时,大大减少存储和计算开销
   - PQ在大规模向量检索和相似性搜索任务中表现出色

以上方法各有优缺点,选择时需要根据具体任务的需求权衡。例如,PCA和自编码器适合用于数据压缩和可视化,LSH和PQ适合用于大规模向量检索和最近邻搜索。在实践中,也可以将多种方法结合使用,以达到更好的压缩效果。

除了选择合适的压缩算法,压缩的实现也需要注意一些细节。例如,在训练压缩模型时,要使用与下游任务相似的数据分布,以避免分布漂移导致的性能下降。此外,压缩后的向量表示应该尽可能与原始任务解耦,以提高通用性和可复用性。

总之,embedding压缩是一种优化LLM知识库应用的有效方法。通过选择合适的压缩算法和实现细节,可以在降低存储和计算开销的同时,保留关键信息,提高下游任务的性能。这对于构建高效、可扩展的知识库应用非常重要。
``` 

# 但是

BGE-M3本身就是embedding的模型, 将 文本 -> 向量. 

那么在训练BGE-M3时, 为了让效果更明显, 需要将文本进行压缩, 而不是向量进行压缩, 之前的方向错了. 

预计将文本使用LLM进行重写, 让文本变小

# 尝试LLM进行重写

提示词: 

```
我有一段文档如下: 
========================

本节逻辑位置: 工行错误日志分析open file list len in shard

本节主要内容: 本章节为工行错误日志分析的一部分，主要讨论了open file list len in shard的问题。
>>>>>>>>>>
文章标题: 工行错误日志分析open file list len in shard
本节逻辑位置: 工行错误日志分析open file list len in shard > 工单复盘整理

本节主要内容: 讨论了MySQL error log中出现的“Open file list len in shard 0 is 0”信息，解释了其产生原因和解决方案。建议根据服务器资源状况增加系统变量innodb_open_files的值。 

同级章节内容摘要: 无同级章节。

父级章节内容摘要: 本章节为工行错误日志分析的一部分，主要讨论了open file list len in shard的问题。
>>>>>>>>>>
# 工单复盘整理

# 一、问题总结

在MySQL的error log中“Open file list len in shard 0 is 0 ” 的信息，这只是info
级别的信息，不是报错，产生于 InnoDB 关闭表空间的过程中。如果错误日志中出现了这样的信息，说明系统现在比较忙，已经打开的文件都在工作中，不能关闭。

# 二、问题现象

  * 环境信息

数据库版本：MySQL8.0.25

应用从oracle迁移到MySQL环境，业务使用正常，但经常会有复制延时问题

  * 报错信息

    
    
    [Note]  [MY-012102]  [InnoDB]  Open file list len in shard 0 is 0
    [Note]  [MY-012102]  [InnoDB]  Open file list len in shard 1 is 0
    [Note]  [MY-012102]  [InnoDB]  Open file list len in shard 2 is 0
    [Note]  [MY-012102]  [InnoDB]  Open file list len in shard 3 is 0
    [Note]  [MY-012102]  [InnoDB]  Open file list len in shard 4 is 0
    

# 三、问题产生原因

当InnoDB已打开表空间数量大于等于系统变量 innodb_open_files
的值，就不能再打开新的表空间了。如果InnoDB不做任何处理，再打开新的表空间就会失败。

为了解决这个问题，InnoDB 会找到第一个可以关闭的表空间，然后关闭这个表空间，为后面打开新的表空间腾出位置。

每次触发 InnoDB
关闭表空间，都有可能会进行最多轮的关闭操作，每进行1轮只关闭一个表空间，关闭表空间后会判断打开表空间数量是否小于nnodb_open_files的值。

如果小于，本次关闭表空间的过程就结束了；否则进行下一轮，最多只进行3轮。

在第 2、3 轮关闭表空间时，就会产生 Open file list len in shard 0 is 0 这样的 info
信息，如果出现了这样的信息，说明系统现在比较忙，已经打开的文件都在工作中，不能关闭。  

# 四、解决方案

根据服务器资源状况，增加系统变量 innodb_open_files 的值，让 InnoDB 可以同时打开更多的表空间
本节逻辑位置: 工行错误日志分析open file list len in shard > 五、诊断步骤

本节主要内容: 介绍了报错信息 Open file list len in shard 0 is 0 的产生位置代码和代码分析，解释了其中的两个 0 分别表示的含义，以及为什么 InnoDB 把已打开的用户表空间对象分布到 64 个 Fil_shard 对象中。同时指出，MySQL 8.0.28 已经废弃了 ER_IB_MSG_277 错误描述。
>>>>>>>>>>
# 五、诊断步骤

## 1、问题产生位置代码

    
    
    //Open file list len in shard 0 is 0 产生于 Fil_system::close_file_in_all_LRU() 方法中，代码如下：
    /** Tries to close a file in all the LRU lists.
    The caller must hold the mutex.
    @param[in]     print_info     if true, prints information why it close a file
    @return true if success, false if should retry later */
    bool Fil_system::close_file_in_all_LRU(bool print_info) {
      for (auto shard : m_shards) {
        shard->mutex_acquire();
     
        if (print_info) {
          // 这就是产生报错信息 Open file list len in shard 0 is 0 的地方
          ib::info(ER_IB_MSG_277, shard->id(),
                   ulonglong{UT_LIST_GET_LEN(shard->m_LRU)});
        }
     
        bool success = shard->close_files_in_LRU(print_info);
     
        shard->mutex_release();
     
        if (success) {
          return true;
        }
      }
     
      return false;
    }
    
    

## 2、代码分析

报错信息对应的错误描述常量为 ER_IB_MSG_277，根据错误描述常量可以在 share/messages_to_error_log.txt
文件中找到错误描述:

    
    
    ER_IB_MSG_277  
    eng "Open file list len in shard %zu is %llu"
    

从错误描述可以看到，报错信息中的两个 0 都是程序运行时动态生成的

  * 第一个 0 就表示 Fil_shard 对象在数组中的序号（从 0 开始）
  * 第二个 0 表示序号为 0 的 Fil_shard 对象 m_LRU 链表中记录的表空间数量

InnoDB 有一个保存 Fil_shard 对象的数组，有 69 个元素，每个元素都是一个 Fil_shard 对象，其中前 64
个用于保存已打开的用户表空间对象。每个 Fil_shard 对象里都有一个 m_LRU 链表，记录了该 Fil_shard 对象中已经打开的表空间。  
InnoDB 把已打开的用户表空间对象分布到 64 个 Fil_shard 对象中，是因为打开或关闭表空间时，表空间对象加入 Fil_shard 对象
m_LRU 链表、从 m_LRU 链表删除，都需要加互斥锁。

如果只有一个 Fil_shard 对象，也就只有一个 m_LRU 链表，数据库繁忙时，争抢锁的情况就会比较频繁。拆分之后，有 64 个保存用户表空间对象的
Fil_shard 对象，就有 64 个 m_LRU 链表，这样锁粒度就小很多了。

MySQL 8.0.28 已经废弃了 ER_IB_MSG_277 错误描述，这个版及之后的版本都不会再看到 Open file list len in
shard 0 is 0 这样的信息了。
本节逻辑位置: 工行错误日志分析open file list len in shard > 六、参考文档

本节主要内容: 本章节提供了一个参考文档链接，可以用于进一步了解open file list len in shard的问题。
>>>>>>>>>>
# 六、参考文档

https://forums.mysql.com/read.php?11,698120,698145#msg-698145

========================

我想将这段文档进行embedding, 但这样获得的embedding向量会过大. 
我希望你对这段文档的长度进行压缩, 长度减少一半, 但表义不变. 

**尤其**是你需要保持原文的以下内容的主要信息, 方便RAG使用这些内容来进行搜索: 
1. 报错信息
2. 重点日志信息
3. 操作的输入和输出
4. 参数及相关信息
5. 诊断的主要步骤
6. 解决方案的主要步骤
``` 

结果: 

```
以下是文档的压缩版本，保留了关键信息：

---

在工行的MySQL错误日志分析中，关注点在于"Open file list len in shard 0 is 0"这一信息，它并非错误，而是InnoDB关闭表空间过程中的提示。此问题可能表明系统繁忙，所有打开的文件正被使用。解决方案是调整`innodb_open_files`变量以允许更多并发表空间。

报错环境：MySQL 8.0.25，Oracle迁移后遇到复制延时，日志反复显示"Open file list len in shard x is 0"（x从0到4）。

问题源于InnoDB打开表空间数量达到`innodb_open_files`限制，导致无法再开新表空间。InnoDB尝试关闭最不活跃的表空间，关闭过程中可能产生上述日志。如果3轮内无法关闭，说明系统繁忙。

诊断步骤包括分析`Fil_system::close_file_in_all_LRU()`方法，此方法在关闭文件时生成报错信息。报错中的两个数字分别代表Fil_shard索引和对应链表中的表空间数量。InnoDB用64个Fil_shard对象分散用户表空间以减小锁竞争。

解决方案是根据服务器资源提升`innodb_open_files`值，以增加并发表空间。

参考文档：[论坛链接](https://forums.mysql.com/read.php?11,698120,698145#msg-698145)

---

这个压缩版本保留了原始文档的关键信息，包括报错信息、日志细节、诊断步骤和解决方案，适合用于后续的搜索和理解。

``` 

提示词: 

```
我有一段文档如下: 
========================

本节逻辑位置: [百胜中国_双主模式中主从不能同步]

本节主要内容: 本章节是文章《百胜中国_双主模式中主从不能同步》的一部分，主要讲述了双主模式下主从不能同步的问题。
>>>>>>>>>>
文章标题: 百胜中国_双主模式中主从不能同步
本节逻辑位置: [百胜中国_双主模式中主从不能同步 > 工单复盘整理]

本节主要内容: 讲述了双主模式下从库报错主键冲突、主从同步出现问题以及跳过记录恢复主从同步的解决方案，同时也提到了内核bug导致mysql被阻塞的情况。
>>>>>>>>>>
# 工单复盘整理：

# 1、问题总结

双主模式中从库报错主键冲突，主从同步出现问题。且通过跳过记录恢复主从同步，后续执行show slave status 等从库相关命令时会hang住。

# 2、现象描述

## 背景信息

    
    
    //数据库实例信息
    架构：双主模式，不在DMP平台的数据库。
    实例IP地址：172.31.51.108
                172.31.51.109
    用户名：yunsign
    密码：yum@yunsign1234
     
    //版本
    mysql：5.6.35
    系统版本：CentOS 6.6
    内核版本：2.6.32-504
    [root@SGH1VDBCONT03 ~]# cat /etc/redhat-release
    Centos release 6.6 (Final)
    [root@SGH1VDBCONT03 ~]# mysql --version
    mysql Ver 14.14 Distrib 5.6.35, for Linux(x86_64) using EditLine wrapper
    [root@SGH1VDBCONT03 ~]# uname -a
    Linux SGH1VDBCONT03 2.6.32-504.8.1.el6.x86_64 #1 SMP Wed Jan 28 21:11:36 UTC 2015 x86_64 x86_64 x86_64 GNU/LinJux
    

## 问题：

（1）主从无法同步。

（2） 执行 show slave status 等从库相关命令时会hang住。

# 3、原因

（1）主从同步报错主键冲突。客户在172.31.51.109主机上，删除ID为64630的记录，但主从无法同步，172.31.51.108主机上还是有64630记录。  
（2）因内核 2.6.32-504 bug，导致mysql出现被阻塞的情况。  
  
h1. 4、解决方案  
（1）在172.31.51.108主机上执行跳过该条记录，从库恢复正常。

    
    
    mysql>slave stop
    mysql>set GLOBAL SQL_SLAVE_SKIP_COUNTER=1;
    mysql>slave start;
    

（2） yum update kernal 小版本升级到 2.6.32-504.23.4 解决mysql被阻塞的情况。
本节逻辑位置: [百胜中国_双主模式中主从不能同步 > 诊断步骤]

========================

我想将这段文档进行embedding, 但这样获得的embedding向量会过大. 
我希望你对这段文档的长度进行压缩, 但表义不变. 尤其是以下内容需要保持不变, 方便RAG使用这些内容来进行搜索: 
1. 报错信息
2. 重点日志信息
3. 操作的输入和输出
4. 参数及相关信息
``` 

结果: 

````
为了压缩文档并保持关键信息不变，可以将文档简化如下：

---

**百胜中国_双主模式主从同步问题**

**问题概述：**
双主模式下，从库遇到主键冲突，导致主从同步中断。执行`show slave status`等命令时系统挂起。

**环境配置：**
- 双主数据库，非DMP平台
- IP地址：172.31.51.108, 172.31.51.109
- 用户：yunsign
- 密码：yum@yunsign1234
- MySQL 5.6.35, CentOS 6.6, 内核 2.6.32-504

**问题点：**
1. 主从同步错误，主键冲突（如ID为64630的记录未同步）
2. `show slave status`命令执行后系统冻结

**原因分析：**
1. 主从数据不一致，主库记录未删除
2. 内核bug导致MySQL阻塞

**解决步骤：**
1. **跳过冲突记录：**
   ```
   mysql> STOP SLAVE;
   mysql> SET GLOBAL SQL_SLAVE_SKIP_COUNTER = 1;
   mysql> START SLAVE;
   ```

2. **更新内核：**
   升级到2.6.32-504.23.4以修复MySQL阻塞问题

---

在这个简化版中，保留了原始文档的关键信息，包括报错、日志、操作步骤和相关参数，这将有助于RAG模型进行搜索和理解。
````
