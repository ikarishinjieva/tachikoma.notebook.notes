---
title: 20220531 - 用户权限模型调研
confluence_page_id: 1737189
created_at: 2022-05-31T04:50:47+00:00
updated_at: 2022-06-07T10:57:26+00:00
---

# 材料

  - <https://docs.authing.cn/v2/guides/access-control/choose-the-right-access-control-model.html>
    - 描述 RBAC和ABAC
    - RBAC: 通过用户的角色（Role）授权其相关权限
      - <https://docs.authing.cn/v2/guides/access-control/rbac.html>
      - 缺点: 通过用户是否具备某个角色来控制权限，这种权限控制还是比较粗粒度的
        - Authing 还同时支持给用户、角色授权  

      - 资源概念: 把系统的一些对象抽象为资源, 比如gitlab的 "Repository"  

    - ABAC: 一个操作是否被允许是基于对象、资源、操作和环境信息共同动态计算决定的  

      - <https://docs.authing.cn/v2/guides/access-control/abac.html>
      - 环境信息 可以是各种附加信息计算, 比如说: "要求当前请求的用户经过了 MFA 认证"
  - <https://study.sf.163.com/documents/read/manual/3role_manage.md>  

  - <http://www.woshipm.com/pd/5253059.html>

# 模型重点

  - 保持模型灵活, 兼容几种不同的权限模型
  - 模型需要能 对接外部用户系统 
    - 可参考jira映射LDAP, 将LDAP organization unit映射到静态只读用户组, 删除的用户标记为 失效用户
  - 可溯源: 可查询某个授权, 是沿着哪个路径进行的授权
  - 按概念矩阵, 同一维度内的概念, 在授权中地位一致

# 概念矩阵

| 维度1 | 维度2 | 维度3 |
| --- | --- | --- |
| 用户 | 表 | 数据操作 |
| 用户组 | 模式 | 数据操作集 |
| 用户组嵌套用户组 | 库 |  |
| 组织架构 | 业务 |  |
| 用户组嵌套组织架构 | 业务嵌套业务 |  |
|  |  |  |
| 维度4 |  |  |
| 角色 |  |  |
| 角色嵌套角色 |  |  |
  
# 设计

  - 授权 = *谁* 对 *数据目标* 有 *怎样的数据操作*
    - 带变量的授权: (也可以转换成EL) [低优先级, 不确定使用场景]
      - 举例: 当一个文档的所属部门跟用户的部门相同时，用户可以访问这个文档 (数据目标的某属性 = {who的某属性})
      - 可以考虑: 将条件放在cypher公式的where中
    - 逻辑与或 和 分支: 
      - 可层级嵌套, 产生逻辑?? 或者使用EL产生逻辑??
      - 授权的两种状态: 允许/禁止
      - 对于同一对象的不同授权路径的优先级??
  - 谁 (who)
    - 用户
      - 带有属性, 属性可用于配置动态组
    - 用户组
      - 普通用户组
      - 动态用户组
      - 用户组可嵌套, 可重叠
    - 组织结构
      - 逐层组织的用户组
      - 不可重叠
  - 数据目标 (object)
    - 数据目标 = 库/模式/表/列
      - 带有属性, 属性可用于配置动态组
    - 业务: 数据目标的集合
      - 业务可嵌套, 可重叠
      - 可配置动态的数据目标组
    - 业务/库/模式/表/列, 均可以设置为 "脱敏" 模式
      - 不同用户, 具有访问脱敏数据/访问原始数据的权限
      - 与数据发现的互动??
  - 数据 *操作 (operation)*
    - 数据操作集
      - 可嵌套, 可重叠
      - 举例: 谁 对 数据目标 有管理权限
    - 待解决: 不同数据库的权限, 会包括其他权限, 比如MySQL的super, 包含了select等权限. 但每种数据库对这类权限的定义不同.
      - 考虑如何拉平??
  - 数据权限 (permission) = 数据目标 + 数据操作  

    - 角色 (role): 数据权限的集合
      - 可嵌套, 可重叠
      - 举例: 谁 是 订单业务的管理员

与其他模型的映射:

  - RBAC0  

    - ![image](https://image.yunyingpai.com/wp/2021/12/xaPnOm7hocgFFfaSNW41.jpeg)
  - RBAC1
    - ![image](https://image.yunyingpai.com/wp/2021/12/F3zi6CaGyzBsi4GUPvJi.jpeg)
    - 角色可嵌套
  - RBAC2
    - RBAC0 + 激活一个角色时所应遵循的强制性规则
    - 比如: 互斥关系角色/基数约束/先决条件角色
    - 暂不映射 RBAC2
  - ACL
    - ![image](https://image.yunyingpai.com/wp/2021/12/j7AlU9gO7TiveZukRvCF.jpeg)
  - ABAC
    - 映射方案: who带属性, 数据目标带属性, 授权带逻辑EL

# 使用Neo4j Aura

  - 删除所有记录: MATCH (n) DETACH DELETE n

  - 返回所有记录: MATCH (n) RETURN n

  

脚本: 重建数据: 

```
//clean up
MATCH (n) DETACH DELETE n

//User
CREATE (Huangyan:User {name:'Huangyan'})
CREATE (Zhouwenya:User {name:'Zhouwenya'})
CREATE (Wangyue:User {name:'Wangyue'})
CREATE (Sunjian:User {name:'Sunjian'})
CREATE (Yanhuqing:User {name:'Yanhuqing'})
CREATE (Baofengqi:User {name:'Baofengqi'})
CREATE (Leixia:User {name:'Leixia'})
CREATE (Jinchanglong:User {name:'Jinchanglong'})

//OrganizationUnit
CREATE (Dev:OrganizationUnit {name:'Dev'})
CREATE (DmpDev:OrganizationUnit {name:'Dmp Dev'})
CREATE (DbleDev:OrganizationUnit {name:'Dble Dev'})
CREATE (QA:OrganizationUnit {name:'QA'})

//OrganizationUnit relations
CREATE (DmpDev)-[:MEMBER_OF]->(Dev)
CREATE (DbleDev)-[:MEMBER_OF]->(Dev)
CREATE (Zhouwenya)-[:MEMBER_OF]->(DmpDev)
CREATE (Sunjian)-[:MEMBER_OF]->(DmpDev)
CREATE (Wangyue)-[:MEMBER_OF]->(DmpDev)
CREATE (Yanhuqing)-[:MEMBER_OF]->(DbleDev)
CREATE (Baofengqi)-[:MEMBER_OF]->(DbleDev)
CREATE (Leixia)-[:MEMBER_OF]->(QA)
CREATE (Jinchanglong)-[:MEMBER_OF]->(QA)

//UserGroup
CREATE (DmpGroup:UserGroup {name:'Dmp Group'})
CREATE (DmpDev)-[:MEMBER_OF]->(DmpGroup)
CREATE (Jinchanglong)-[:MEMBER_OF]->(DmpGroup)

//DataObject

CREATE (DmpConfluenceTable1:DataObject {name:'DmpConfluenceTable1', type:'Table'})
CREATE (DmpConfluenceSchema2:DataObject {name:'DmpConfluenceSchema2', type:'Schema'})
CREATE (DmpConfluenceTable1)-[:MEMBER_OF]->(DmpConfluenceSchema2)
CREATE (DmpConfluenceSchema2)-[:MEMBER_OF]->(DmpConfluenceSchema2)

CREATE (DmpJiraTable3:DataObject {name:'DmpJiraTable3', type:'Table'})
CREATE (DmpJiraSchema4:DataObject {name:'DmpJiraSchema4', type:'Schema'})
CREATE (DmpJiraTable3)-[:MEMBER_OF]->(DmpJiraSchema4)

CREATE (DbleConfluenceTable5:DataObject {name:'DbleConfluenceTable5', type:'Table'})
CREATE (DbleConfluenceSchema6:DataObject {name:'DbleConfluenceSchema6', type:'Schema'})
CREATE (DbleConfluenceTable5)-[:MEMBER_OF]->(DbleConfluenceSchema6)

CREATE (DmpDatabase7:DataObject {name:'DmpDatabase7', type:'Database', DSN: '172.17.1.101:3306'})
CREATE (DmpConfluenceSchema2)-[:MEMBER_OF]->(DmpDatabase7)
CREATE (DmpJiraSchema4)-[:MEMBER_OF]->(DmpDatabase7)

CREATE (DbleDatabase8:DataObject {name:'DbleDatabase8', type:'Database', DSN: '172.17.1.102:3306'})
CREATE (DbleConfluenceSchema6)-[:MEMBER_OF]->(DbleDatabase8)

CREATE (DmpBiz:DataObject {name:'Dmp Biz', type:'Biz'})
CREATE (DmpDatabase7)-[:MEMBER_OF]->(DmpBiz)

CREATE (DbleBiz:DataObject {name:'Dble Biz', type:'Biz'})
CREATE (DbleDatabase8)-[:MEMBER_OF]->(DbleBiz)
 
// operation and operation set
CREATE (Insert:DataOperation {name:'Insert'})
CREATE (Select:DataOperation {name:'Select'})
CREATE (Delete:DataOperation {name:'Delete'})
CREATE (AlterTable:DataOperation {name:'AlterTable'})
CREATE (CreateSchema:DataOperation {name:'CreateSchema'})

CREATE (TableReadonly:DataOperationSet {name:'TableReadonly'})
CREATE (TableAdmin:DataOperationSet {name:'TableAdmin'})
CREATE (DatabaseAdmin:DataOperationSet {name:'DatabaseAdmin'})

CREATE (TableReadonly)-[:OPERATION]->(Select)
CREATE (TableAdmin)-[:OPERATION]->(Insert)
CREATE (TableAdmin)-[:OPERATION]->(Delete)
CREATE (TableAdmin)-[:OPERATION]->(TableReadonly)
CREATE (DatabaseAdmin)-[:OPERATION]->(AlterTable)
CREATE (DatabaseAdmin)-[:OPERATION]->(CreateSchema)
CREATE (DatabaseAdmin)-[:OPERATION]->(TableAdmin)

// assign data permission
CREATE (SelectDmpConfluenceSchema2:DataPermission {name:'Select DmpConfluenceSchema2'})
CREATE (DmpConfluenceSchema2)-[:OBJECT_OF]->(SelectDmpConfluenceSchema2)
CREATE (SelectDmpConfluenceSchema2)-[:OPERATION]->(Select)
CREATE (Sunjian)-[:ASSIGN]->(SelectDmpConfluenceSchema2)

// assign data role
CREATE (AdminDmpDatabase:DataPermission {name:'AdminDmpDatabase'})
CREATE (DmpBiz)-[:OBJECT_OF]->(AdminDmpDatabase)
CREATE (AdminDmpDatabase)-[:OPERATION]->(DatabaseAdmin)

CREATE (DmpAdminRole:DataRole {name:'DmpAdminRole'})
CREATE (DmpAdminRole)-[:PERMISSION]->(AdminDmpDatabase)
CREATE (Zhouwenya)-[:ASSIGN]->(DmpAdminRole)
 
//assign to user group
CREATE (SelectDmpBiz:DataPermission {name:'SelectDmpBiz'})
CREATE (DmpBiz)-[:OBJECT_OF]->(SelectDmpBiz)
CREATE (SelectDmpBiz)-[:OPERATION]->(Select)
CREATE (DmpGroup)-[:ASSIGN]->(SelectDmpBiz)
``` 

请求: 

```
// 查询DmpGroup 嵌套结构下所有的用户
MATCH (user:User)-[:MEMBER_OF*..10]->(DmpGroup:UserGroup)
    RETURN DISTINCT user
 
// 查询属于DmpBiz的所有表
MATCH (tbl:DataObject {type:'Table'})-[:MEMBER_OF*..10]->(DmpBiz:DataObject {name:'Dmp Biz', type:'Biz'})
RETURN DISTINCT tbl

//查询 Sunjian对DmpConfluenceSchema2是否有Select权限, 并列出权限路径 (赋权给Schema, 搜索Schema, 直接授权)
MATCH p1=(a:User {name:'Sunjian'})-[:ASSIGN]->(perm:DataPermission)<-[:OBJECT_OF|MEMBER_OF*]-(tbl:DataObject {name:'DmpConfluenceSchema2', type:'Schema'})
MATCH p2=(perm:DataPermission)-[:OPERATION]->(o:DataOperation{name: 'Select'})
RETURN p1,p2
 
//查询 Sunjian对DmpConfluenceTable1是否有权限, 并列出权限路径 (赋权给Schema, 搜索Table, 直接授权)
MATCH p1=(a:User {name:'Sunjian'})-[:ASSIGN]->(perm:DataPermission)<-[:OBJECT_OF|MEMBER_OF*]-(tbl:DataObject {name:'DmpConfluenceTable1', type:'Table'})
MATCH p2=(perm:DataPermission)-[:OPERATION]->(o:DataOperation{name: 'Select'})
RETURN p1,p2
 
//查询Zhouwenya对DmpConfluenceTable1是否有权限, 并列出权限路径  (赋权给Schema, 搜索Table, 授权给role)
 
MATCH p1=(a:User {name:'Zhouwenya'})-[:ASSIGN|PERMISSION*]->(perm:DataPermission)<-[:OBJECT_OF|MEMBER_OF*]-(tbl:DataObject {name:'DmpConfluenceTable1', type:'Table'})
MATCH p2=(perm:DataPermission)-[:OPERATION*]->(o:DataOperation{name: 'Select'})
RETURN p1,p2
 
//查询DmpGroup成员对DmpBiz是否有权限, 并列出权限路径  (赋权给Biz, 搜索Table, 给组授权)
 
MATCH p1=(a:User {name:'Jinchanglong'})-[:ASSIGN|PERMISSION|MEMBER_OF*]->(perm:DataPermission)<-[:OBJECT_OF|MEMBER_OF*]-(tbl:DataObject {name:'DmpConfluenceTable1', type:'Table'})
MATCH p2=(perm:DataPermission)-[:OPERATION*]->(o:DataOperation{name: 'Select'})
RETURN p1,p2
 
//如何查询单个数据表具有什么权限
MATCH p1=(u:User)-[:ASSIGN|PERMISSION|MEMBER_OF*]->(perm:DataPermission)<-[:OBJECT_OF|MEMBER_OF*]-(tbl:DataObject {name:'DmpConfluenceTable1', type:'Table'})
MATCH p2=(perm:DataPermission)-[:OPERATION*]->(o:DataOperation)
RETURN DISTINCT u.name,o.name

``` 

# 问题

建议 不同类型的关系命名不同

如上模型, MEMBER_OF用于多种模型, 会导致 获取授权列表时, 无法阻断 schema -> table的关系

如: 

```
MATCH p1=(u:User)-[:ASSIGN|PERMISSION|MEMBER_OF*]->(perm:DataPermission)<-[:OBJECT_OF|MEMBER_OF*]-(obj:DataObject)
WHERE obj.type = 'Schema' or obj.type='Table'
MATCH p2=(perm:DataPermission)-[:OPERATION*]->(oper:DataOperation)
RETURN DISTINCT obj.name,u.name,oper.name
``` 

会获取所有Table的授权, 而不是明确被授权的Table, 导致有多余授权

如果 Table 和 Schema的关系定义为 (Table) -- [:TABLE_OF_SCHEMA] -> (Schema), 那上述请求的 MEMBER_OF*就可以分离出这部分关系, 进行处理

# TODO

  - constraints: <https://neo4j.com/docs/cypher-manual/current/constraints/>
