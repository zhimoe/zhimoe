+++
title = "Neo4j 入门"
date = 2018-12-01
categories = [ "编程",]
tags = [ "neo4j", "code", "database",]
toc = "true"
+++


### neo4j 图数据库介绍
neo4j 是目前排名最高的图数据库，分为商业和社区版本，社区版只支持单机，而且查询的运行时 (runtime) 不同 (cypher runtime:interpreted(社区版),slotted(企业版)). 数据库排名可以在 https://db-engines.com/en/ranking/graph+dbms 查看，下一代最有前景的开源图数据库是 dgraph，目前还积极开发中，生产未就绪，等他的 Java 客户端再成熟一点可以试用。

neo4j 数据库中只有 3 个概念：Node, Relationship, Properties. Node 表示实体类别，使用 Label 区分，例如一个节点可以有 Person/Father 等多个标签，Relationship 即关系，雇佣关系，父子关系，投资关系，交易关系等。Node 和 Relationship 都可以有 Proerties，属性自身不分是属于节点还是属于关系，例如 Person 可以有属性 name，关系也可以用属性 name.你可以在 neo4j browser 左侧看到当前数据库的所有 Node Label,Relationship Type,Properties. 

<!--more-->

#### 本地安装和在线沙箱
neo4j 背后的公司为了吸引用户，提供了一些好玩的数据库沙箱，这些沙箱数据库已经提前放了一些主题数据，例如购物数据，国会关系数据。你可以通过注册登录 https://neo4j.com/sandbox-v2/, 选择一个数据沙箱实例进行学习试玩。当然你也可以下载社区版，命令行 `neo4j.bat console`启动，打开 127.0.0.1:7474 开始学习。

一个 neo4j 支持多个数据库但是一次只能激活一个数据库，一个数据库所有文件都在$neo4j_home\data\databases 目录的独立文件夹，在 conf/neo4j.conf 的`dbms.active_database=graph.db`指定激活那个数据库。

#### cypher 查询语言
neo4j 使用 cypher 语言作为查询语言。这是一种模式匹配的声明式语言。基本语法和 SQL 相似。
cypher 中常用的子句 (clause) 有：MATCH,RETURN,WITH,WHERE,UNWIND,LIMIT,UNION,SKIP,SET. RETURN,LIMIT WHERE 和 SQL 中是一样的，UNWIND 这些需要用到再查看文档，这里介绍 MATCH 和 WITH.
`MTACH`用于指定搜索的模式。例如希望找到'Tom Hanks'在 2018 演过的所有电影：`MATCH (p:ACTOR {name:'Tom Hanks' }) -[r: ACT_IN]->(m:MOVIE {time: '2018'})`,这是一个模式，可以直接 REUREN 返回 p,r,m 等变量。可以看到模式中的节点 (Node Label) 使用`()`;关系类型 (Relationship Type) 使用`[]`指定，如果不关心 type，那么[]可以省略。使用--; 属性 (Properties) 使用{pname: pvalue}指定。
`WITH`的作用和 python 的 with 非常相似 (实际上 cypher 语言借鉴了 python 的 list 处理语法),用于修改一些变量，变量一般都是上一个子句的查询结果，修改之后传给下一个子句。例如下面的语句找到和 Anders 有关系的人的年龄最大的那个人，返回那个人的所有认识的人的名字。

```s
MATCH (n { name: 'Anders' })--(m)
WITH m
ORDER BY m.age DESC LIMIT 1
MATCH (m)--(o)
RETURN o.name
```
cypher 手册：https://neo4j.com/docs/cypher-manual/3.5/clauses/


#### cypher 的操作符
如果需要进行 cypher 调优，有必要了解一下 cypher 的操作符。一般编程语言的代码在被执行前都会被编译得到抽象语法树 (AST). 例如 Java 代码，一个 Java 文件会被抽象为一个 package,class, method,variable declare 等不同部分得到一个 Class 对象。cypher 语句一样会被编译得到一棵语法树 (AST),每个树节点是一个操作符。从叶节点的操作符开始执行，得到的结果依次返回给父节点进一步处理。常见的操作符有:AllNodesScan(全局扫描，只能作为叶节点),NodeByLabelScan,Apply 等。例如`MATCH (n) return n`会得到一个`AllNodesScan`和`ProduceResults`操作符构成的 AST, 你可以通过`PROFILE`查看你语句编译后得到的操作符构成的执行计划。

```s
# 执行语句得到下面的表格
PROFILE MATCH (p:Person { name: 'Tom Hanks' }) RETURN p
# 省略了部分列
+-----------------+----------------+------+---------+-----------------+
| Operator        | Estimated Rows | Rows | DB Hits | Page Cache Hits |
+-----------------+----------------+------+---------+-----------------+
| +ProduceResults |              1 |    1 |       0 |               0 |
| |               +----------------+------+---------+-----------------+
| +NodeIndexSeek  |              1 |    1 |       2 |               0 |
+-----------------+----------------+------+---------+-----------------+

```

#### cypher runtime
pass


#### neo4j browser 介绍
和大多数数据库一样，neo4j 是 server-client 的数据库，支持 http 和 bolt2 中协议.neo4j 自带一个基于浏览器的客户端，只需在浏览器输入 serverIp:7474 即可使用。
neo4j browser 自带一个教程和电影关系的数据库初始化脚本。方便你可以学习。下面介绍几个常用的命令。
- :help <topic> help 命令显示各种帮助提示。常见的 topic 有 :help cypher :help commands :help keys :help param
- :play 交互式学习命令。例如，:play movie graph 进入基于电影数据库的教程。
- :param 命令，设置变量。 :param usrname => "zhimoe",注意，变量名和=>之间有空格。设置变量之后可以使用变量`MATCH (n:Person) WHERE n.name = $usrname`
- :params 显示当前已经设置的所有变量。也可以使用:params {name: 'Stella', age: 24} 覆盖目前的变量。但是这个命令没用类型安全。

### spring-neo4j 配置
pass 

### cypher 调优
cypher 是一种声明式的，模式匹配的查询语言。模式在 cypher 语言中非常重要。如何合理地设计查询中的模式是 cypher 性能可调优空间最大的地方。下面给出常见的优化建议。
需要说明的是，后面的这些建议其实大都可以在[cypher 手册](https://neo4j.com/docs/cypher-manual/3.5/query-tuning/)找到，如果感兴趣，建议通读这份长文档...
#### 避免全局 scan
cypher 是一种模式匹配的语言，默认会进行全局扫描，除非你告诉它不要。所以起始节点的 label 非常重要。起始的模式匹配基数大小也非常重要。
#### 缓存和硬盘 IO
neo4j 数据库将数据文件和 Page Cache 作了映射，如果在缓存中没有查询到，neo4j 会从硬盘加载数据文件。第二次查询就可以走缓存。所以需要充分利用 Page Cache.记住第一次查询总是会比较慢，因为没用缓存.neo4j 有 2 级缓存:string cache 和 AST cache
+ string cache
默认 neo4j 在 cache 中保留 1000 个查询计划，可在`conf/neo4j.conf`中参数` dbms.query_cache_size`修改这个设置。

需要注意的是 cache 是根据语句的 string hash 值判断的，所以一样的语句仅仅是大小写不一样或者空白符不一样对缓存来说也是 2 个语句。

`PROFILE/EXPLAIN`语句只会 cache 其去掉`PROFILE/EXPLAIN`之后的部分。例如：`MATCH (n) return COUNT(n);`和`PROFILE MATCH (n) return COUNT(n);`的 cache 是一致的。

+ AST cache
编程语言都有语法树。如果在 string cache 中没有找到缓存。那么会将查询正规化，得到语法树并将其缓存。正规化的同时也会做一些优化，例如
`match (n:Person {id:101}) return n;`在正规化之后得到`match (n:Person) where n.id={param1} return n;   {param1: 101}`,AST cache 不区分大小写，空格等，所以以下查询是一致的：

```s
match (n:Person) where n.id=101 return n;
match (n:Person {id:101}) return n;

MATCH ( n:Person { id : 101 } )
RETURN n;
```

#### execution plan
当 cypher 引擎收到查询语句后如果没用找到对应的缓存，那么 Cypher query planner 会将语句规范化，优化后编译得到一个执行计划 (execution plan).这个执行计划会缓存一切且可以复用。当查询缓存过多，或者数据库的数据变化大时 (设置参数是) 这个执行计划则失效被移除。在查询中使用参数而不是字面量值，可以提高一个执行计划的复用率。
更多信息参考文档:https://neo4j.com/docs/cypher-manual/3.5/execution-plans/#execution-plan-introduction 

#### 查看查询计划
如果想要查看查询语句的执行计划，可以在查询语句前加上 `EXPLANIN` OR `PROFILE` 关键字，你可以在 neo4j browser 查看 query plan 找到性能瓶颈。结果左侧边里面第 3 个 tab 会给出详细的性能警告 (warn).
`EXPLAIN`只会给出语句的分析结果;而`PROFILE`则会执行你的查询语句把给出耗时最多的报告，以及每个操作符返回了多少行记录。注意，profiling 会消耗很多资源，所以不要在生产环境中频繁使用。调优的基础是基于 cypher 的操作符，所以需要你对操作符有基本的了解。
#### 索引
数据库离不开索引。这里有个小陷阱，最早谱系的节点是企业客户 (label: COR_CUSTOMER)+和几十个零售客户节点 (label:RTL_CUSTOMER),我在查询语句起始节点没有指定 label，没用遇到性能问题，后来加入了 3 百万的个人节点数据后，原来 1s 的查询变成了 1 分半钟。所以在干扰的 label 比较少时，你不会察觉到性能问题。务必在起始节点指定 label，即使目前只有一个 label，最好也提前加上。

然而，cypher 语句目前不允许在一个节点指定多个 label，例如你希望起点 label 是` COR_CUSTOMER|RTL_CUSTOMER `,这个是不允许的。只能在 where 语句指定。

```s
MATCH n
WHERE n:COR_CUSTOMER OR n:RTL_CUSTOMER
RETURN n
```
在 3.0 之前的 neo4j 中使用上面的语句，会导致一个 AllNodesScan，在 3.0 之后，该语句则是将 2 个 NodeByLabelScan 匹配结果`UNION`然后`DISTINCT`的结果。所以是搜索 2 次再合并结果。你可以在上面的 cypher 语句前面添加`EXPLAIN`查看执行计划，已确定你的语句是否会导致全局扫描。
[SO 上关于多个 label 匹配的讨论](https://stackoverflow.com/questions/20003769/neo4j-match-multiple-labels-2-or-more)

#### 大结果集
如果你的查询返回结果集太大，例如几 M 大小，那么你可能需要考虑你的设计了。过大的结果集会导致查询返回变慢，要注意，这些结果会占用你的缓存空间，而如果在网络情况不好时，情况更加糟糕了。

目前谱系对这一块并没有优化，最大的谱系的返回接口可能达到 1M 多，加上 ES 的数据，前端接收数据会有 4M 多。

#### 锁
当你修改节点的信息时，节点会被锁定;如果修改关系，关系会被锁定;如果增加/删除关系，那么 2 个节点和这个关系都会被锁定。而如果此时有节点/关系的相关查询请求，这些请求会等待。所以，如果你需要将 50 个节点加入一个组 (group)--即添加 50 个关系，如果你调用 50 次方法，那么这个 group 节点被 lock 的时间较长，此时可以通过`UNWIND`和列表 (list) 参数处理这个问题。

```S
MATCH (g:Group { uuid: $groupUuid })
UNWIND $personUuidList as personUuid
MATCH (p:Person { uuid : personUuid })
MERGE (p)-[:IS_MEMBER]->(g)
 
```

#### 常见查询错误
+ 变量名
+ label 忘记添加冒号，例如 MATCH (Person) 和 MATCH (:Person) 是完全不一样的，前者 Person 是变量，不走索引。
+ 有过大的中间结果集，优化你的语句时思考：尽早 distinct，尽早 limit，使用 collect 减少结果的行数，在正确地方使用 order by;


#### 多个 UNWIND 语句导致笛卡尔积
多个 UNWIND 会导致一个笛卡尔积的结果，这个结果可能会很大。例如下面的结果会得到 3*3=9 行，所以尽量避免笛卡尔积。

```S
with ['a','b','c'] as lts, [1,2,3] as nrs unwind lts as char unwind nrs as nr return char,nr
```
#### 在 MATCH 中使用多个模式笛卡尔积
在 MATCH 中使用多个模式也会导致笛卡尔积，比较下面的 2 个结果相同的语句，第一个耗时 80s，第二个只需 8ms.

```s
#  1. 笛卡尔积 80000 ms w/ ~900 players, ~40 teams, ~1200 games
MATCH (pl:Player),(t:Team),(g:Game)
RETURN COUNT(DISTINCT pl),
COUNT(DISTINCT t),
COUNT(DISTINCT g)

# 2. 8ms w/~900 players, ~40 teams, ~1200 games
MATCH (pl:Player)
WITH COUNT(pl) as players
MATCH (t:Team)
WITH COUNT(t) as teams, players
MATCH (g:Game)
RETURN COUNT(g) as games, teams, players
```

#### 模式中的方向
下面的查询中，如果给关系 ACTED_IN 添加上方向，可以提高查询速度。

```S
MATCH (p:Person)-[:ACTED_IN]-(m)
WHERE p.name = "Tom Hanks"
RETURN m
```
















