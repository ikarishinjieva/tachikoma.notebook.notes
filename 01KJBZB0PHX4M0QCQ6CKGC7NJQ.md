---
title: 20240723 - 知识图谱的关系抽取
confluence_page_id: 3145836
created_at: 2024-07-23T11:41:27+00:00
updated_at: 2024-07-25T09:00:55+00:00
---

# 目标: 

想将MySQL manual以及相关文档, 转换成知识图谱. 辅助RAG的召回, 来达成多重逻辑的推导. 

根据 [20240718 - ChatDBA: 如何使用知识类的文档] 的测试, 使用LLM进行朴素的三元组抽取并不实用

# ConceptNet

<https://conceptnet.io/s/contributor/petgame>

样例:

![image2024-7-23 18:34:16.png](/assets/01KJBZB0PHX4M0QCQ6CKGC7NJQ/image2024-7-23%2018%3A34%3A16.png)

ConceptNet带有固定的关系词表

得理解其中的关系词是如何设计的: <https://github.com/commonsense/conceptnet5/wiki/Relations> 一共34个关系词

测试使用关系词进行查询转化: 

````
对于我的问题: "MySQL临时表的使用场景有哪些?"
如果用ConceptNet的关系词来进行查询, 查询语句应该怎么写
 
---
 
这是一个很好的问题！将自然语言问题转换成 ConceptNet 查询语句，需要将问题中的实体和关系映射到 ConceptNet 的概念和关系词上。

以下是针对您的问题 "MySQL 临时表的使用场景有哪些？"，使用 ConceptNet 进行查询的步骤和示例语句：

**1. 实体识别和映射:**

* 问题中的实体: "MySQL 临时表", "使用场景"
* ConceptNet 概念: 
    * "MySQL 临时表" 可以映射到 "temporary table" 或 "MySQL database" (更广义)
    * "使用场景" 可以映射到 "use case" 或 "purpose"

**2. 关系识别和映射:**

* 问题中的隐含关系:  "MySQL 临时表" 与 "使用场景" 之间存在某种语义关联，表示 "MySQL 临时表"  "被用于" 哪些 "使用场景"。
* ConceptNet 关系词: 可以考虑以下关系词：
    * `/r/UsedFor`:  表示 "用于" 的关系，例如 "锤子 UsedFor 钉钉子"。
    * `/r/HasProperty`:  表示 "具有属性" 的关系，可以理解为某种场景是 "MySQL 临时表" 的一种属性。

**3.  构建查询语句:**

* **方法一**: 使用 `/r/UsedFor` 关系词

```
[ConceptNet 查询语句]
find * from where $c isa "Temporary table" and $c <UsedFor> $usecase
```

* **方法二**: 使用 `/r/HasProperty` 关系词

```
[ConceptNet 查询语句]
find * from where $c isa "MySQL database" and $c <HasProperty> $property and $property isa "Use case"
```

**4. 结果解释:**

* 查询结果会返回与 "temporary table" 或 "MySQL database" 存在  `/r/UsedFor` 或  `/r/HasProperty` 关系的概念，这些概念就可能代表 "MySQL 临时表" 的使用场景。
* 需要注意的是，ConceptNet 的知识库是基于常识构建的，不一定包含所有特定领域的专业知识。因此，查询结果可能不完整，需要结合其他信息进行分析和判断。

**总结:**

使用 ConceptNet 查询 "MySQL 临时表的使用场景" 需要进行实体和关系的映射，并选择合适的 ConceptNet 关系词。查询结果可以提供一些参考信息，但需要结合其他知识进行综合分析。
````
````
对于我的问题: ""如何解决 MySQL 数据库死锁问题？""
如果用ConceptNet的关系词来进行查询, 但不限于ConceptNet的实体词, 查询语句应该怎么写
 
---
 
这是一个很有意思的挑战！让我们尝试用 ConceptNet 的关系词，结合自定义的实体词，来构建一个表达 "如何解决 MySQL 数据库死锁问题？" 的查询语句。

**查询思路：**

1. **识别核心实体:**  "MySQL 数据库", "死锁", "解决方法"
2. **使用相关词语替换专业术语:** 
    * "MySQL 数据库" 可以用 "数据库系统" 替换
    * "死锁" 可以用 "资源竞争" 或 "程序卡住" 替换
    * "解决方法" 可以用 "解决方案" 或 "修复方法" 替换
3. **使用 ConceptNet 关系词连接:**  选择能够表达实体之间逻辑关系的词语。

**示例查询语句:**

```
find * from where
  $c1 <AtLocation> "数据库系统" and
  $c1 <RelatedTo> "资源竞争" and
  $c2 <CapableOf> "修复" and
  $c2 <ReceivesAction> $c1
```

**解释:**

*  `$c1 <AtLocation> "数据库系统"`:  寻找存在于 "数据库系统" 中的概念。
*  `$c1 <RelatedTo> "资源竞争"`:  该概念与 "资源竞争" 相关。
*  `$c2 <CapableOf> "修复"`:  寻找能够执行 "修复" 的概念。
*  `$c2 <ReceivesAction> $c1`:  "修复" 的对象是前面找到的与 "资源竞争" 相关的概念。

**预期结果和局限性:**

* **预期结果**:  这个查询语句尝试通过 "数据库系统"、"资源竞争"、"修复" 等词语的关联，找到解决数据库系统中资源竞争问题的方法。
* **局限性**:  由于 ConceptNet 主要基于常识性知识，对于 "资源竞争" 这样的专业术语，它不一定能准确理解其在数据库系统中的含义，返回的结果可能存在偏差。

**总结:**

使用 ConceptNet 关系词结合自定义实体词，可以在一定程度上表达特定领域的问题，但由于 ConceptNet 知识库的局限性，查询结果的准确性和完整性难以保证。 

**建议:**

为了更好地解决专业技术问题，建议您考虑以下方案:

* **构建特定领域的知识图谱:**  将 MySQL 数据库的专业知识构建成知识图谱，可以使用图数据库 (Neo4j) 或 RDF 存储 (Jena) 来实现。
* **使用问答系统**:  使用深度学习技术训练一个专门针对 MySQL 技术问题的问答系统，例如使用 BERT 模型结合 MySQL 官方文档进行训练。
```` 

# DBpedia

DBpedia使用RDF来进行定义, 通过sparql来查询. 

查询谓词的总数: <https://dbpedia.org/sparql>

```
SELECT (COUNT(DISTINCT ?property) AS ?propertyCount)
WHERE {
  ?property a rdf:Property .
}
``` 

一共有 56354 个谓词 (也就是关系词), 跟ConceptNet不同, 其谓词是分析得来

# Schema.org

## 样例

````
在Schema.org中,"数据库软件"可以被视为"软件应用程序"(SoftwareApplication)的一种特定类型。SoftwareApplication是CreativeWork的一个子类型,用于描述任何软件应用程序,包括桌面应用程序、移动应用程序、Web应用程序等。

以下是使用Schema.org描述"数据库软件"的示例:

```json
{
  "@context": "https://schema.org",
  "@type": "SoftwareApplication",
  "name": "MySQL",
  "operatingSystem": "Windows, Linux, macOS",
  "applicationCategory": "DatabaseApplication",
  "offers": {
    "@type": "Offer",
    "price": "0",
    "priceCurrency": "USD"
  }
}
```

在这个示例中,我们使用了以下Schema.org的类型和属性:

- @type: 指定实体的类型,这里是SoftwareApplication。
- name: 软件应用程序的名称,这里是"MySQL"。
- operatingSystem: 软件应用程序支持的操作系统。
- applicationCategory: 软件应用程序的类别,这里我们使用了"DatabaseApplication"来表示数据库应用程序。请注意,applicationCategory并不是Schema.org官方定义的属性,而是一个自定义的属性,用于进一步说明软件的具体类型。
- offers: 描述软件的报价信息,如价格和货币。

虽然Schema.org没有专门为"数据库软件"定义一个独立的类型,但通过使用SoftwareApplication类型并结合其他相关属性,我们仍然可以有效地描述和表示这个概念。

在实际应用中,你可以根据需要添加更多的属性,如软件版本(version)、发布日期(releaseDate)、软件官网(url)等,以提供更详细和准确的描述。如果有必要,你也可以创建自定义的子类型和属性,以更好地满足特定领域的需求。
````

````
对于MySQL的存储引擎InnoDB,在Schema.org中没有一个直接对应的类型。这是因为Schema.org主要关注于描述常见的网络资源和实体,而不是深入到特定软件系统的技术细节。
然而,我们可以使用Schema.org中的其他类型和属性,来近似地描述InnoDB这样的存储引擎。一种可能的方法是将其视为SoftwareApplication的一个组件或特性。下面是一个示例:
```json
{
 "@context": "https://schema.org",
 "@type": "SoftwareApplication",
 "name": "MySQL",
 "operatingSystem": "Windows, Linux, macOS",
 "applicationCategory": "DatabaseApplication",
 "offers": {
 "@type": "Offer",
 "price": "0",
 "priceCurrency": "USD"
 },
 "featureList": [
 {
 "@type": "Text",
 "name": "Storage Engine",
 "description": "InnoDB is the default storage engine for MySQL. It provides ACID-compliant transactions, row-level locking, and foreign key support."
 }
 ]
}
```
在这个示例中,我们在SoftwareApplication中添加了一个featureList属性,它是一个列表,包含软件的各种特性或组件。每个特性都用一个包含name和description属性的Text类型来描述。
我们使用了Text类型的name属性来指定特性的名称为"Storage Engine",并在description属性中提供了有关InnoDB的简要说明。
请注意,这种描述方式是一种近似和变通的方法,因为Schema.org并没有专门为数据库存储引擎设计的类型。在实际应用中,你可能需要根据具体情况和需求,调整和扩展Schema.org的类型和属性,以更准确地描述像InnoDB这样的技术概念。
另一种选择是使用其他更专业和细粒度的本体或词汇表,如软件工程本体(Software Engineering Ontology)或数据库领域的特定本体,它们可能提供了更适合描述数据库组件和特性的类型和属性。你可以考虑将这些专业本体与Schema.org结合使用,以实现更全面和精确的知识表示。 
```` 

## 想法

schema.org中, property是从wikipedia中的各个"属性"提取出来, 可以用来做"关系词"

# YAGO

\- YAGO differs from DBpedia, because YAGO has a predefined schema, predefined and non-redundant relations, and logical constraints. The manually curated part of DBpedia has all of these, too, but contains only 4 million instances. YAGO contains 50 million.

YAGO是如何整理"关系"的: 

<https://github.com/yago-naga/yago-4.5/blob/main/01-make-schema.py>

其中读取了 00-shapes.ttl 定义文件, 其样例: 

```
schema:Thing a sh:NodeShape ;
    # ys:fromClass wd:Q35120 ; No mapping!
	sh:property [
        sh:path schema:image ;
        sh:datatype xsd:anyURI ;
        ys:fromProperty wdt:P18 ;
    ] ;	
    sh:property [
        sh:path schema:alternateName ;
        sh:datatype rdf:langString ;
        ys:fromProperty skos:altLabel, wdt:P225, wdt:1843 ;
    ] ;
    sh:property [
        sh:path rdfs:comment ;
        sh:datatype rdf:langString ;
        sh:uniqueLang true ;
        ys:fromProperty schema:description ;
    ] ;
...
``` 

其中: sh:path 是property的唯一标识, ys:fromProperty 是其他数据源中同义的属性名 (应该被映射到当前property的属性名)

  - wdt是wikidata中的数据名空间
  - schema是schema.org中的数据名空间

YAGO的整体思路是: 用schema.org的实体定义, 并加以修订; 将其他数据集的 属性 映射到该套定义下

(比如上例中, schema:description 就被映射成了 rdfs:comment)

目前在GPU的机器上, 下载yago-4.5.0.2.zip, 目的是分析其中有多少个关系 (rdf:Property)

YAGO的数据一共有: 

```
-rw-r--r--  1 root   root   120G 4月   5 08:15 yago-beyond-wikipedia.ttl
-rw-r--r--  1 root   root    22G 4月   5 08:15 yago-facts.ttl
-rw-r--r--  1 root   root   974M 4月   5 08:15 yago-meta-facts.ntx
-rw-r--r--  1 root   root    36K 4月   4 17:25 yago-schema.ttl
-rw-r--r--  1 root   root    13M 4月   5 08:15 yago-taxonomy.ttl
``` 

将这些三元组的中间谓词拆离出来: 

```
cat *.ttl | awk '{print $2}' | uniq > rel.txt
cat rel.txt | sort | uniq > rel.uniq.txt
``` 

获得的谓词有限, 448个: 

[附件: rel.uniq.txt] 

将脏数据和带序号的无用谓词去除, 获得 148个谓词:

[附件: rel.uniq.fixed.txt] 

进行关系抽取测试: 

````
我希望你使用以上谓词, 将以下文档变成知识图谱用的三元组: 
```
MySQL permits a subquery to refer to a stored function that has data-modifying side effects such as inserting rows into a table. For example, if f() inserts rows, the following query can modify data:
 
SELECT ... WHERE x IN (SELECT f() ...);
 
This behavior is an extension to the SQL standard. In MySQL, it can produce nondeterministic results because f() might be executed a different number of times for different executions of a given query depending on how the optimizer chooses to handle it.
 
For statement-based or mixed-format replication, one implication of this indeterminism is that such a query can produce different results on the source and its replicas.
```
````

```
好的，我会尽力将您提供的文本文档转换为知识图谱三元组。以下是我分析文档后得到的三元组，以及我对选择这些谓词和构建三元组的解释：

**实体识别和类型定义:**

* MySQL: rdf:type schema:Software,  rdfs:label "MySQL" 
* subquery: rdf:type  yago:Worker, rdfs:label "subquery" 
* stored function: rdf:type schema:Software, rdfs:label "stored function"
* data-modifying side effects: rdf:type schema:Thing, rdfs:label "data-modifying side effects"
* inserting rows into a table: rdf:type schema:Thing, rdfs:label "inserting rows into a table"
* f(): rdf:type schema:Software, rdfs:label "f()"
* query: rdf:type schema:Software, rdfs:label "query"
* SQL standard: rdf:type schema:CreativeWork, rdfs:label "SQL standard"
* nondeterministic results: rdf:type schema:Thing, rdfs:label "nondeterministic results"
* optimizer: rdf:type schema:Software, rdfs:label "optimizer"
* statement-based replication: rdf:type schema:Thing, rdfs:label "statement-based replication"
* mixed-format replication: rdf:type schema:Thing, rdfs:label "mixed-format replication"
* indeterminism: rdf:type schema:Thing, rdfs:label "indeterminism"
* source: rdf:type schema:Thing, rdfs:label "source"
* replicas: rdf:type schema:Thing, rdfs:label "replicas"

**关系抽取和三元组构建:**

* (MySQL, schema:permits, subquery): MySQL 允许子查询
* (subquery, schema:memberOf, query): 子查询是查询的一部分
* (subquery, schema:worksFor, stored function): 子查询可以调用存储函数
* (stored function, schema:hasPart, data-modifying side effects): 存储函数可以有数据修改副作用
* (data-modifying side effects, owl:sameAs, inserting rows into a table): 数据修改副作用等同于插入行到表中
* (f(), rdf:type, stored function): f() 是一个存储函数
* (f(), schema:hasPart, data-modifying side effects): f() 有数据修改副作用
* (query, schema:hasPart, f()): 查询包含 f() 函数调用
* (query, schema:potentialAction, modify data): 查询可以修改数据
* (MySQL, schema:iswcCode, SQL standard): MySQL 是 SQL 标准的扩展
* (MySQL, schema:hasPart, nondeterministic results): MySQL 中可能产生不确定结果
* (nondeterministic results, schema:reason, f()): 不确定结果是由 f() 函数的执行次数不同导致的
* (optimizer, schema:influence, f()): 优化器影响 f() 函数的执行次数
* (statement-based replication, schema:hasPart, indeterminism): 基于语句的复制具有不确定性
* (mixed-format replication, schema:hasPart, indeterminism): 混合格式复制具有不确定性
* (indeterminism, schema:cause, different results): 不确定性导致不同结果
* (different results, schema:location, source): 不同结果出现在源数据库
* (different results, schema:location, replicas): 不同结果出现在复制数据库

**解释:**

* 以上三元组试图捕捉文档中描述的关键实体和关系。
* 我尽量使用您提供的 YAGO 和 Schema.org 谓词来表示这些关系。
* 部分谓词可能需要根据实际情况进行调整或替换。
* 由于文档内容有限，部分实体的类型和关系可能需要更多上下文信息才能确定。

**请注意:** 

* 以上三元组只是根据我的理解对文档内容的初步转换，可能存在不准确或不完整的地方。
* 建议您根据实际需求对三元组进行进一步的完善和修正。
``` 

改写问题"MySQL临时表的使用场景有哪些?"为查询: 

```
SELECT ?description
WHERE {
  ?tempTable rdf:type ?type .
  FILTER regex(str(?type), "MySQL临时表", "i") .

  ?tempTable rdfs:comment ?description .
  FILTER regex(str(?description), "使用场景|应用场景", "i") .
}
``` 

看上去这些关系谓词无法直接满足问题查询的要求

另一种改写 (Claude): 用description谓词

```
根据提供的关系谓词,我尝试构建一个SPARQL查询来查找 "MySQL临时表的使用场景有哪些" 的答案。以下是我的尝试:

PREFIX schema: <http://schema.org/>
PREFIX yago: <http://yago-knowledge.org/resource/> 

SELECT ?usage ?description
WHERE {
  ?tmpTable schema:about "MySQL Temporary Table"@en .
  ?tmpTable schema:name ?name .
  ?usage schema:about ?name .
  ?usage yago:appearsIn ?mysqlManual .
  ?mysqlManual schema:name "MySQL Manual"@en .
  ?usage schema:description ?description .
}

这个查询的逻辑是:

1. 找到一个主题为 "MySQL Temporary Table" 的实体 ?tmpTable
2. 获取该实体的名字 ?name  
3. 找到一个包含 ?name 主题的实体 ?usage,认为它是一种临时表的使用场景
4. 确保 ?usage 出现在名为 "MySQL Manual" 的实体 ?mysqlManual 中
5. 获取 ?usage 的描述 ?description 作为使用场景的解释

最后返回每个使用场景 ?usage 及其描述 ?description。

但是这个查询有一些局限性和不确定性:

1. MySQL manual中可能没有明确列举临时表的使用场景,或者使用了不同的术语
2. 关系谓词可能无法准确表达 "使用场景" 这样的概念,在知识图谱里可能没有直接的关联
3. 临时表使用场景的描述可能分布在手册的不同地方,单一的描述字段可能无法完整涵盖

所以这个查询能否获得令人满意的结果,还需要看知识图谱的具体建模方式。但我尽量利用手头的谓词做出了一个尝试,希望对你有参考价值。如果结果不理想的话,可能需要换一种方式去建模和查询这类问题。
``` 

测试以上改写是否有效, 对原始文档进行图谱构建: 

````
 我从YAGO中抽取了关系词, 一共148个如下: 
```
geo:wktLiteral
owl:disjointWith
owl:sameAs
rdf:first
rdf:langString
rdf:nil
rdfs:Class
rdfs:comment
rdfs:label
rdfs:subClassOf
rdf:type
schema:about
schema:actor
schema:address
schema:AdministrativeArea
schema:affiliation
schema:alternateName
schema:alumniOf
schema:author
schema:award
schema:birthDate
schema:birthPlace
schema:BodyOfWater
schema:children
schema:City
schema:contentLocation
schema:Corporation
schema:Country
schema:CreativeWork
schema:dateCreated
schema:deathDate
schema:deathPlace
schema:description
schema:director
schema:dissolutionDate
schema:duns
schema:duration
schema:editor
schema:endDate
schema:Event
schema:founder
schema:gender
schema:geo
schema:gtin
schema:homeLocation
schema:iataCode
schema:icaoCode
schema:illustrator
schema:image
schema:inLanguage
schema:Intangible
schema:isbn
schema:knowsLanguage
schema:Landform
schema:Language
schema:leiCode
schema:location
schema:locationCreated
schema:logo
schema:lyricist
schema:mainEntityOfPage
schema:manufacturer
schema:material
schema:memberOf
schema:musicBy
schema:name
schema:nationality
schema:numberOfEmployees
schema:numberOfEpisodes
schema:numberOfPages
schema:numberOfSeasons
schema:Organization
schema:organizer
schema:owns
schema:parentTaxon
schema:performer
schema:PerformingGroup
schema:Person
schema:Place
schema:postalCode
schema:Product
schema:productionCompany
schema:publisher
schema:recordLabel
schema:sameAs
schema:slogan
schema:sponsor
schema:spouse
schema:startDate
schema:superEvent
schema:Taxon
schema:Thing
schema:url
schema:worksFor
sh:class
sh:datatype
sh:path
sh:PropertyShape
skos:altLabel
xsd:anyURI
xsd:dateTime
xsd:decimal
xsd:string
yago:academicDegree
yago:appearsIn
yago:area
yago:AstronomicalObject
yago:Award
yago:beliefSystem
yago:BeliefSystem
yago:candidateIn
yago:capital
yago:conferredBy
yago:consumes
yago:demonym
yago:director
yago:distanceFromEarth
yago:doctoralAdvisor
yago:elevation
yago:flowsInto
yago:follows
yago:Gender
yago:highestPoint
yago:humanDevelopmentIndex
yago:HumanMadeGeographicalEntity
yago:influencedBy
yago:iswcCode
yago:leader
yago:length
yago:lowestPoint
yago:luminosity
yago:mass
yago:neighbors
yago:notableWork
yago:officialLanguage
yago:ownedBy
yago:parallax
yago:parentBody
yago:participant
yago:playsIn
yago:populationNumber
yago:radialVelocity
yago:replaces
yago:sportNumber
yago:studentOf
yago:studentsCount
yago:terminus
yago:unemploymentRate
yago:Worker
```

我希望你使用以上谓词, 将以下文档变成知识图谱用的三元组:
```
temporary table
A table whose data does not need to be truly permanent. For example, temporary tables might be used as storage areas for intermediate results in complicated calculations or transformations; this intermediate data would not need to be recovered after a crash. Database products can take various shortcuts to improve the performance of operations on temporary tables, by being less scrupulous about writing data to disk and other measures to protect the data across restarts.

Sometimes, the data itself is removed automatically at a set time, such as when the transaction ends or when the session ends. With some database products, the table itself is removed automatically too.

See Also table.
```
```` 

结果: 

```
Claude:

1. temporary table rdf:type schema:Intangible .
2. temporary table schema:description "A table whose data does not need to be truly permanent." .
3. temporary table schema:about "storage areas for intermediate results in complicated calculations or transformations" .
4. temporary table schema:about "intermediate data would not need to be recovered after a crash" .
5. database products schema:offers "shortcuts to improve the performance of operations on temporary tables" .
6. temporary table schema:sameAs table .
 
Gemini:
(temporary table, rdfs:subClassOf, schema:Thing)
(temporary table, rdfs:label, "temporary table")
(temporary table, schema:description, "A table whose data does not need to be truly permanent.")
(temporary table, schema:description, "For example, temporary tables might be used as storage areas for intermediate results in complicated calculations or transformations; this intermediate data would not need to be recovered after a crash.")
(temporary table, schema:description, "Database products can take various shortcuts to improve the performance of operations on temporary tables, by being less scrupulous about writing data to disk and other measures to protect the data across restarts.")
(temporary table, schema:description, "Sometimes, the data itself is removed automatically at a set time, such as when the transaction ends or when the session ends.")
(temporary table, schema:description, "With some database products, the table itself is removed automatically too.")
(temporary table, schema:sameAs, table)
``` 

信息都会几种在description和about中, 类似于Wiki组织的信息, 而不是强调"关系"

# OpenIE

通过OpenIE提取三元组: <https://corenlp.run/>

测试案例: 

```
MySQL permits a subquery to refer to a stored function that has data-modifying side effects such as inserting rows into a table. For example, if f() inserts rows, the following query can modify data:

SELECT ... WHERE x IN (SELECT f() ...);

This behavior is an extension to the SQL standard. In MySQL, it can produce nondeterministic results because f() might be executed a different number of times for different executions of a given query depending on how the optimizer chooses to handle it.

For statement-based or mixed-format replication, one implication of this indeterminism is that such a query can produce different results on the source and its replicas.
``` 

![image2024-7-24 14:41:0.png](/assets/01KJBZB0PHX4M0QCQ6CKGC7NJQ/image2024-7-24%2014%3A41%3A0.png)

其中有一些关系识别错误, 比如"number of"被识别成了一种Relation

# 结论

  - 尝试 通过固定"谓词"的方法构建图谱: 
    - 从已知的小型数据集中 (ConceptNet), 固定的34个谓词数量太少, 导致查询被改写时的不稳定性很高
      - 后续可以尝试一下实际效果, 通过多次改写和其他技巧来平衡不稳定性
    - 从已知的大型数据集中 (YAGO/...), 谓词数量很多, 需要尝试. 但很有可能无法放到提示词中
  - OpenIE这种NLP抽取关系的方法
    - 准确度比较低, 并且后续还得增加代词消歧等操作, 技术方案比较复杂
  - 后续: 需要尝试其他方案, 借助LLM的力量来规避NLP方案的复杂性
