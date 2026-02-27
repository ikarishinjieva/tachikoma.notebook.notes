---
title: 20240605 - git: cherry-pick 保持时间戳的脚本
confluence_page_id: 2949595
created_at: 2024-06-04T22:02:00+00:00
updated_at: 2024-06-04T22:02:00+00:00
---

```
#!/bin/bash

# 检查是否提供了提交哈希作为参数
if [ -z "$1" ]; then
  echo "Usage: $0 <commit-hash>"
  exit 1
fi

commit_hash=$1

# 获取原始提交的详细信息，使用 ISO 8601 格式的日期
commit_info=$(git show --no-patch --pretty=format:"%an <%ae>|%aI" "$commit_hash")
if [ $? -ne 0 ]; then
  echo "Error: Unable to find commit $commit_hash"
  exit 1
fi

author_info=$(echo "$commit_info" | cut -d'|' -f1)
author_date=$(echo "$commit_info" | cut -d'|' -f2)

author_date=$(echo "$author_date" | sed 's/+.*//')

# 设置环境变量
export GIT_COMMITTER_DATE="$author_date"
export GIT_AUTHOR_DATE="$author_date"

# Cherry-pick 并保留原始提交者和时间
git cherry-pick "$commit_hash" --no-commit
if [ $? -ne 0 ]; then
  echo "Error: Cherry-pick failed"
  exit 1
fi

# 使用 --reuse-message 保留提交信息
GIT_COMMITTER_DATE="$author_date" GIT_AUTHOR_DATE="$author_date" git commit --author="$author_info" --reuse-message="$commit_hash"
if [ $? -ne 0 ]; then
  echo "Error: Commit failed"
  exit 1
fi

echo "Cherry-pick and commit completed successfully"
```
