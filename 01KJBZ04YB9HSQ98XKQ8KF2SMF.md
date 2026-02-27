---
title: 20230523 - 修复llama_index的refine接口
confluence_page_id: 2392085
created_at: 2023-05-23T08:00:20+00:00
updated_at: 2023-05-23T08:01:32+00:00
---

# 问题

refine接口, 在连续chunk中, 后续的chunk使用的response并不是字符串, 会形成错误的prompt: 

![image2023-5-23 15:59:51.png](/assets/01KJBZ04YB9HSQ98XKQ8KF2SMF/image2023-5-23%2015%3A59%3A51.png)

# 解决

增加等待response的代码

![image2023-5-23 15:58:29.png](/assets/01KJBZ04YB9HSQ98XKQ8KF2SMF/image2023-5-23%2015%3A58%3A29.png)

# TODO

提交给社区
