---
title: 20250509 - 处理华南光明区的数据
confluence_page_id: 3801914
created_at: 2025-05-09T06:38:42+00:00
updated_at: 2025-05-09T07:53:02+00:00
---

# 问数目标: 

[光明区问数反馈.doc](/assets/01KJBZSQ7AEHM7FAT1YTMV1K4N/%E5%85%89%E6%98%8E%E5%8C%BA%E9%97%AE%E6%95%B0%E5%8F%8D%E9%A6%88.doc)

# 测试1:

2024年12月光明区总人口数，深圳户籍人数（光明户籍人数，深圳非光明区户籍人数），非深圳户籍人数

![image2025-5-9 13:32:53.png](/assets/01KJBZSQ7AEHM7FAT1YTMV1K4N/image2025-5-9%2013%3A32%3A53.png)

# 测试2:

在我区连续居住六个月以上非户籍常住人数（深户非光明人数，非深户人数）

![image2025-5-9 13:38:8.png](/assets/01KJBZSQ7AEHM7FAT1YTMV1K4N/image2025-5-9%2013%3A38%3A8.png)

问题: 居住时长的粒度过大, 没有"六个月"的选项

# 测试3:

居住在我区或外出不满半年的户籍常住人数

![image2025-5-9 13:43:18.png](/assets/01KJBZSQ7AEHM7FAT1YTMV1K4N/image2025-5-9%2013%3A43%3A18.png)

问题:

  - 居住时长的粒度过大, 没有"六个月"的选项
  - "外出" 如何判定

# 测试4:

对于 "人口变化分析", 需要多张统计表进行联合查询

  - 其中有一些旧表使用spare_1, 新表使用spare_5, 需要确认逻辑

# 测试5:

总人口平均年龄，户籍人口平均年龄。非户籍常住人口平均年龄

![image2025-5-9 14:12:29.png](/assets/01KJBZSQ7AEHM7FAT1YTMV1K4N/image2025-5-9%2014%3A12%3A29.png)

# 测试6:

![image2025-5-9 14:17:4.png](/assets/01KJBZSQ7AEHM7FAT1YTMV1K4N/image2025-5-9%2014%3A17%3A4.png)

![image2025-5-9 14:17:18.png](/assets/01KJBZSQ7AEHM7FAT1YTMV1K4N/image2025-5-9%2014%3A17%3A18.png)

# 测试7:

户籍人口0-4岁、5-9岁、10-14岁、15-19岁...以此相隔4岁至94岁以上男性、女性人数，并分析整体男女比例趋势。

![image2025-5-9 14:18:58.png](/assets/01KJBZSQ7AEHM7FAT1YTMV1K4N/image2025-5-9%2014%3A18%3A58.png)

![image2025-5-9 14:19:33.png](/assets/01KJBZSQ7AEHM7FAT1YTMV1K4N/image2025-5-9%2014%3A19%3A33.png)

# 测试8

非户籍常住人口0-4岁、5-9岁、10-14岁、15-19岁...以此相隔4岁至94岁以上男性、女性人数，并分析整体男女比例趋势

![image2025-5-9 14:21:24.png](/assets/01KJBZSQ7AEHM7FAT1YTMV1K4N/image2025-5-9%2014%3A21%3A24.png)

![image2025-5-9 14:21:41.png](/assets/01KJBZSQ7AEHM7FAT1YTMV1K4N/image2025-5-9%2014%3A21%3A41.png)

# 测试9

户籍人口10周岁以下（0-1周岁，1-2周岁...9-10周岁）儿童出生数量

![image2025-5-9 14:24:47.png](/assets/01KJBZSQ7AEHM7FAT1YTMV1K4N/image2025-5-9%2014%3A24%3A47.png)

非户籍同理, 不测试

# 测试10

统计15岁及以上户籍人口的文化程度人数及占比

![image2025-5-9 14:26:21.png](/assets/01KJBZSQ7AEHM7FAT1YTMV1K4N/image2025-5-9%2014%3A26%3A21.png)

非户籍同理, 不测试

# 测试11

近年来，光明区人口学历结构变化趋势

需要多个日期的表镜像

# 测试12

统计分析各街道户籍人口、非户籍常住人口人数（占比）、增量、增长率等情况

![image2025-5-9 14:30:14.png](/assets/01KJBZSQ7AEHM7FAT1YTMV1K4N/image2025-5-9%2014%3A30%3A14.png)

![image2025-5-9 14:30:31.png](/assets/01KJBZSQ7AEHM7FAT1YTMV1K4N/image2025-5-9%2014%3A30%3A31.png)

# 测试13

统计分析各社区户籍人口2025年4月、2024年12月、6月数量，增量，增长率等情况

![image2025-5-9 14:33:14.png](/assets/01KJBZSQ7AEHM7FAT1YTMV1K4N/image2025-5-9%2014%3A33%3A14.png)

![image2025-5-9 14:34:7.png](/assets/01KJBZSQ7AEHM7FAT1YTMV1K4N/image2025-5-9%2014%3A34%3A7.png)

# 测试14

统计分析近一年对应学年秋学期适龄儿童（小一、初一）人口数，占人口总数比，户籍人口数和非户籍人口数

![image2025-5-9 14:35:24.png](/assets/01KJBZSQ7AEHM7FAT1YTMV1K4N/image2025-5-9%2014%3A35%3A24.png)

![image2025-5-9 14:35:43.png](/assets/01KJBZSQ7AEHM7FAT1YTMV1K4N/image2025-5-9%2014%3A35%3A43.png)

# 测试15

统计分析户籍人口和非户籍常住民族结构

![image2025-5-9 14:37:14.png](/assets/01KJBZSQ7AEHM7FAT1YTMV1K4N/image2025-5-9%2014%3A37%3A14.png)

![image2025-5-9 14:37:17.png](/assets/01KJBZSQ7AEHM7FAT1YTMV1K4N/image2025-5-9%2014%3A37%3A17.png)

# 测试16

前五位非户籍人口来源统计人口数和占比

![image2025-5-9 14:38:10.png](/assets/01KJBZSQ7AEHM7FAT1YTMV1K4N/image2025-5-9%2014%3A38%3A10.png)

![image2025-5-9 14:38:21.png](/assets/01KJBZSQ7AEHM7FAT1YTMV1K4N/image2025-5-9%2014%3A38%3A21.png)

# 测试17

统计区内居住的港澳台居民人口数，较6月对比情况，居住社区分布情况

缺少港澳台居民和外国人的数据判断条件

# TODO

  - 日志
  - 如果SQL报错, 需要给予错误信息, 重新生成并重试
  - area_rn的意图不明, SQL样例中使用了area_rn=1
  - "变化趋势", 需要在多张表间查询
  - 需要支持: 对于一个问题, 拆解出多个查询, 将查询结果一并返回 (后期)
  - 使用源数据表, 而不是拼接后的表?
  - 藏代码
  - API接口换成 xinference
  - 参数配置文件
  - 会话管理, 清除上下文
  - 连续对话 (后期)

streamlit
