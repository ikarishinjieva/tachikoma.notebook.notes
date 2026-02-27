---
title: 20230502 - llama-index + openai + mcontriever, 使用prompt解决信息示踪的问题
confluence_page_id: 2130826
created_at: 2023-05-02T04:35:51+00:00
updated_at: 2023-05-02T08:31:30+00:00
---

# 提示词-1

```
from llama_index.prompts.prompts import QuestionAnswerPrompt, RefinePrompt

qa_prompt_template = (
    "上下文信息如下: \n"
    "---------------------\n"
    "{context_str}[document=external]"
    "\n---------------------\n"
    "使用以上信息, 回答如下问题: {query_str}\n"
    "并新增一个段落, 详细列举答案的每个部分来自于上下文的哪一章或者不来自于上下文\n"
)
text_qa_template = QuestionAnswerPrompt(qa_prompt_template)

refine_prompt_template = (
    "我们要回答这个问题: {query_str}\n"
    "目前的答案是: {existing_answer}\n"
    "新的上下文信息如下\n"
    "------------\n"
    "{context_msg}[document=external]\n"
    "------------\n"
    "使用以上上下文信息优化答案, 目前的答案中详细列举答案的每个部分来自于上下文的哪一章或者不来自于上下文, 将本次的信息合并到最后一段\n"
    "如果没有更优的答案, 则返回目前的答案"
)
refine_template = RefinePrompt(refine_prompt_template)
 
 
response = index.query("如何用mysqlbackup进行\"单文件全备恢复\"给出详细步骤和详细命令", similarity_top_k=2, text_qa_template=text_qa_template, refine_template=refine_template)
print(response)
``` 

提示词中增加了一个段落: 
    
    
    第一次迭代的结果: 

```
用mysqlbackup进行"单文件全备恢复"的步骤和详细命令如下:
1> 进行数据库全备: 使用mysqlbackup命令进行全备, 并将备份保存成一个镜像文件
mysqlbackup --user=root --password=password --host=localhost --socket=/data/mysqldata/mysql.sock --backup-image=/root/backup/my.mbi --backup-dir=/root/backup backup-to-image 
2> 校验备份镜像文件: 使用mysqlbackup命令校验备份镜像文件
mysqlbackup --backup-image=/root/backup/my.mbi validate
3> 恢复前,对备份文件或备份目录进行校验,确认备份可用
4> 开始恢复,首先停止数据库服务
5> 删除mysql的数据目录(datadir)、日志目录(binlog、redo log、undo log)中的文件,包括这些参数对应的目录: --innodb_data_home_dir, --innodb_log_group_home_dir, --innodb_undo_directory,以便恢复备份
6> 在全备的数据文件上应用redo log,并将备份的数据文件恢复到第4步涉及的数据和日志目录中
7> 改变mysql的数据目录、日志目录的所属者为mysql服务的运行用户
8> 启动数据库,能够正常登陆,查看到最新数据,说明恢复成功

以上步骤和详细命令来自于上下文的第四部分: "具体用法与示例" 其中包括 "备份恢复标准流程" 和 "单文件全备恢复" 两部分。
``` 

第二次迭代的结果: 

```
用mysqlbackup进行"单文件全备恢复"的步骤和详细命令如下:
1> 进行数据库全备: 使用mysqlbackup命令进行全备, 并将备份保存成一个镜像文件
mysqlbackup --user=root --password=password --host=localhost --socket=/data/mysqldata/mysql.sock --backup-image=/root/backup/my.mbi --backup-dir=/root/backup backup-to-image 
2> 校验备份镜像文件: 使用mysqlbackup命令校验备份镜像文件
mysqlbackup --backup-image=/root/backup/my.mbi validate
3> 恢复前,对备份文件或备份目录进行校验,确认备份可用
4> 开始恢复,首先停止数据库服务
5> 删除mysql的数据目录(datadir)、日志目录(binlog、redo log、undo log)中的文件,包括这些参数对应的目录: --innodb_data_home_dir, --innodb_log_group_home_dir, --innodb_undo_directory,以便恢复备份
6> 在全备的数据文件上应用redo log,并将备份的数据文件恢复到第4步涉及的数据和日志目录中
7> 改变mysql的数据目录、日志目录的所属者为mysql服务的运行用户
8> 启动数据库,能够正常登陆,查看到最新数据,说明恢复成功
9> 对于改变了 LCP 线程或者 LQH 线程数量的情况, 必须使用 mysqldump 创建一个备份文件, 然后用 mysql 导入这个备份文件
10> 使用 ndb_mgm 客户端发出 START BACKUP 命令进行全备
11> 把备份的目录复制到新的集群中
12> 使用 ndb_restore 来恢复备份数据

以上步骤和详细命令来自于上下文的第四部分: "具体用法与示例" 、第六部分: "Restoring to More Nodes Than the Original" 和第十二部分: "Restoring from Backup"。
``` 

# 提示词-2

代码: 

```
from llama_index.prompts.prompts import QuestionAnswerPrompt, RefinePrompt

qa_prompt_template = (
    "上下文信息如下: \n"
    "---------------------\n"
    "{context_str}[document=external]"
    "\n---------------------\n"
    "使用以上信息, 回答如下问题: {query_str}\n"
    "并新增一个段落, 详细列举答案的每个部分来自于上下文的哪一部分或者不来自于上下文\n"
)
text_qa_template = QuestionAnswerPrompt(qa_prompt_template)

refine_prompt_template = (
    "我们要回答这个问题: {query_str}\n"
    "目前的答案是: {existing_answer}\n"
    "新的上下文信息如下\n"
    "------------\n"
    "{context_msg}[document=external]\n"
    "------------\n"
    "使用以上上下文信息优化答案, 目前的答案中详细列举答案的每个部分来自于上下文的哪一部分或者不来自于上下文, 将本次的信息合并到最后一段\n"
    "如果没有更优的答案, 则返回目前的答案"
)
refine_template = RefinePrompt(refine_prompt_template)
 
 
response = index.query("如何用mysqlbackup进行\"单文件全备恢复\"给出详细步骤和详细命令", similarity_top_k=2, text_qa_template=text_qa_template, refine_template=refine_template)
print(response)
``` 

第一次迭代: 

```
用mysqlbackup进行"单文件全备恢复"的详细步骤和详细命令如下:
1）进行全备，并将备份保存成一个镜像文件：
mysqlbackup --user=root --password=password --host=localhost --socket=/data/mysqldata/mysql.sock --backup-image=/root/backup/my.mbi --backup-dir=/root/backup backup-to-image
2）校验备份镜像文件：
mysqlbackup --backup-image=/root/backup/my.mbi validate
3）停止数据库服务：
service mysql stop
4）删除mysql的数据目录、日志目录中的文件：
rm -rf /data/mysqldata/datadir/*
rm -rf /data/mysqldata/binlog/*
5）在全备的数据文件上应用redo log：
mysqlbackup --user=root --password=password --host=localhost --socket=/data/mysqldata/mysql.sock --backup-image=/root/backup/my.mbi apply-log
6）将备份的数据文件恢复到数据和日志目录中：
mysqlbackup --user=root --password=password --host=localhost --socket=/data/mysqldata/mysql.sock --backup-image=/root/backup/my.mbi --backup-dir=/root/backup copy-back
7）改变mysql的数据目录、日志目录的所属者为mysql服务的运行用户：
chown -R mysql:mysql /data/mysqldata/datadir
chown -R mysql:mysql /data/mysqldata/binlog
8）启动数据库：
service mysql start

这些部分的信息来自于上下文：
1）第1步：来自上下文的“mysqlbackup --user=root --password=password --host=localhost --socket=/data/mysqldata/mysql.sock --backup-image=/root/backup/my.mbi --backup-dir=/root/backup backup-to-image”；
2）第2步：来自上下文的“mysqlbackup --backup-image=/root/backup/my.mbi validate”；
3）第4步：不来自于上下文；
4）第5步：来自上下文的“mysqlbackup --user=root --password=password --host=localhost --socket=/data/mysqldata/mysql.sock --backup-image=/root/backup/my.mbi apply-log”；
5）第6步：来自上下文的“mysqlbackup --user=root --password=password --host=localhost --socket=/data/mysqldata/mysql.sock --backup-image=/root/backup/my.mbi --backup-dir=/root/backup copy-back”；
6）
...... (长度限制)
``` 

第二次迭代: 

```
用mysqlbackup进行"单文件全备恢复"的详细步骤和详细命令如下:
1）进行全备，并将备份保存成一个镜像文件：mysqlbackup --user=root --password=password --host=localhost --socket=/data/mysqldata/mysql.sock --backup-image=/root/backup/my.mbi --backup-dir=/root/backup backup-to-image
2）校验备份镜像文件：mysqlbackup --backup-image=/root/backup/my.mbi validate
3）停止数据库服务：service mysql stop
4）删除mysql的数据目录、日志目录中的文件：rm -rf /data/mysqldata/datadir/*、rm -rf /data/mysqldata/binlog/*（不来自于上下文）
5）在全备的数据文件上应用redo log：mysqlbackup --user=root --password=password --host=localhost --socket=/data/mysqldata/mysql.sock --backup-image=/root/backup/my.mbi apply-log
6）将备份的数据文件恢复到数据和日志目录中：mysqlbackup --user=root --password=password --host=localhost --socket=/data/mysqldata/mysql.sock --backup-image=/root/backup/my.mbi --backup-dir=/root/backup copy-back
7）改变mysql的数据目录、日志目录的所属者为mysql服务的运行用户：chown -R mysql:mysql /data/mysqldata/datadir、chown -R mysql:mysql /data/mysqldata/binlog
8）启动数据库：service mysql start
9）备份数据：使用ndb_mgm -e "START BACKUP 1"命令创建数据备份，并将备份数据复制到新集群（来自于上下文）
10）备份模式：如果改变了LQH线程或者LCP线程，那么需要使用mysqldump创建模式备份，并将备份模式复制到SQL节点，然后使用mysql导入（来自于上下文）
11）恢复备份：在新节点上使用ndb_restore命令恢复备份，不需要从特定的节点恢复（来自于上下文）
``` 

这里有一个想法, 可以修改prompt, 让大模型将每句的来源标记在每句的句尾, 这样比较省空间
