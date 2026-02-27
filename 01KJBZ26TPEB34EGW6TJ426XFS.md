---
title: 20230802 - 修复gitlab损坏
confluence_page_id: 2588971
created_at: 2023-08-01T16:30:55+00:00
updated_at: 2023-08-01T16:30:55+00:00
---

gitlab日志报错: get default branch: EOF

![image2023-8-2 0:28:18.png](/assets/01KJBZ26TPEB34EGW6TJ426XFS/image2023-8-2%200%3A28%3A18.png)

git clone报错: upload-pack: not our ref 0000000000000000000000000000000000000000

![image2023-8-2 0:28:27.png](/assets/01KJBZ26TPEB34EGW6TJ426XFS/image2023-8-2%200%3A28%3A27.png)

先在gitlab服务器上, 找到出问题的项目的repo文件夹: 

```
cd /var/opt/gitlab/git-data/repositories/@hashed
grep 'fullpath =' */*/*/config
``` 

确定repo文件夹为: /var/opt/gitlab/git-data/repositories/@hashed/59/e1/59e19706d51d39f66711c2653cd7eb1291c94d9b55eb14bda74ce4dc636d015a.git

检查文件夹下的refs, 发现refs/heads/master损坏. 将其替换成refs/heads/下其他分支的commit号. 可跳过错误 (但master分支可能丢失)
