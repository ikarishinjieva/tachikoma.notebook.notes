---
title: 20221025 - 百胜, 唯一键表会有重复记录
confluence_page_id: 1934440
created_at: 2022-10-25T08:20:53+00:00
updated_at: 2022-10-25T08:20:53+00:00
---

# 现象

![image2022-10-25 13:21:40.png](/assets/01KJBYQVBBQDQM7HRJ1Y8WK6PM/image2022-10-25%2013%3A21%3A40.png)

![image2022-10-25 13:21:46.png](/assets/01KJBYQVBBQDQM7HRJ1Y8WK6PM/image2022-10-25%2013%3A21%3A46.png)

将生产的表空间文件复制下来, 在测试环境可复现. 但通过mysqldump导入会出现 唯一键冲突 报错

# 分析

通过innodb_space分析: 

![image2022-10-25 13:30:54.png](/assets/01KJBYQVBBQDQM7HRJ1Y8WK6PM/image2022-10-25%2013%3A30%3A54.png)

![image2022-10-25 13:30:30.png](/assets/01KJBYQVBBQDQM7HRJ1Y8WK6PM/image2022-10-25%2013%3A30%3A30.png)

对于 无主键表, innodb_space会报错

![image2022-10-25 16:20:48.png](/assets/01KJBYQVBBQDQM7HRJ1Y8WK6PM/image2022-10-25%2016%3A20%3A48.png)
