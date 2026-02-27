---
title: 20240131 - 对git分支中的时间和committer进行调整
confluence_page_id: 2949560
created_at: 2024-05-31T11:56:22+00:00
updated_at: 2024-06-01T07:29:24+00:00
---

```
# 备份整个repo
cp -R actiondb-pm-dev-ts actiondb-pm-dev-ts.bak
cd actiondb-pm-dev-ts
#  
git fast-export main > commits.export

LC_ALL=C sed '/author .*/ {
    s/^/ /
    x; s/^/xxxxxxxx/; x;
}

x; /^x/ {
    s/^x//; x; s/^/ /
    b
}
x' commits.export > commits.export.tmp

diff commits.export commits.export.tmp | sed 's/^>  */> /' > commits.patch

# 编辑diff文件
## 编辑时间/committer
## 将分支都从main改到refine

patch commits.export commits.patch
LC_ALL=C sed -i '' 's|\(refs/.*/\)main|\1refine|g' commits.export

git checkout --orphan refine 
git rm -rf .

git fast-import < commits.export
 
# 检查refine git log
 
git checkout main
git reset --hard refine
git push --force origin main
``` 

按照指定时间和committer进行提交: 

```
GIT_AUTHOR_NAME="liwei" GIT_AUTHOR_EMAIL="liwei@actionsky.com" GIT_COMMITTER_NAME="liwei" GIT_COMMITTER_EMAIL="liwei@actionsky.com" GIT_COMMITTER_DATE="2024-04-16T15:45:08" GIT_AUTHOR_DATE="2024-04-16T15:45:08" git commit -m 'update ActionDB 4.1.0.1 baselines: OB base update'
```
