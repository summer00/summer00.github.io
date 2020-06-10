---
title: "Elasticsearch基础知识"
date: 2019-09-14 12:00:00 +0800
categories: [database, mid, iv]
---

- [基本概念](#%e5%9f%ba%e6%9c%ac%e6%a6%82%e5%bf%b5)
- [基本接口](#%e5%9f%ba%e6%9c%ac%e6%8e%a5%e5%8f%a3)
  - [集群管理](#%e9%9b%86%e7%be%a4%e7%ae%a1%e7%90%86)
  - [索引操作](#%e7%b4%a2%e5%bc%95%e6%93%8d%e4%bd%9c)
  - [文档操作](#%e6%96%87%e6%a1%a3%e6%93%8d%e4%bd%9c)
  - [批量操作](#%e6%89%b9%e9%87%8f%e6%93%8d%e4%bd%9c)
  - [搜索](#%e6%90%9c%e7%b4%a2)
- [分布式架构](#%e5%88%86%e5%b8%83%e5%bc%8f%e6%9e%b6%e6%9e%84)
  - [路由](#%e8%b7%af%e7%94%b1)
  - [容错的机制](#%e5%ae%b9%e9%94%99%e7%9a%84%e6%9c%ba%e5%88%b6)
  - [并发冲突](#%e5%b9%b6%e5%8f%91%e5%86%b2%e7%aa%81)
- [数据存储](#%e6%95%b0%e6%8d%ae%e5%ad%98%e5%82%a8)
  - [document id](#document-id)
  - [\_source 元数据](#source-%e5%85%83%e6%95%b0%e6%8d%ae)
  - [替换，删除](#%e6%9b%bf%e6%8d%a2%e5%88%a0%e9%99%a4)
  - [增删改操作过程](#%e5%a2%9e%e5%88%a0%e6%94%b9%e6%93%8d%e4%bd%9c%e8%bf%87%e7%a8%8b)
  - [读请求操作过程](#%e8%af%bb%e8%af%b7%e6%b1%82%e6%93%8d%e4%bd%9c%e8%bf%87%e7%a8%8b)
  - [写入更新索引内部逻辑（todo）](#%e5%86%99%e5%85%a5%e6%9b%b4%e6%96%b0%e7%b4%a2%e5%bc%95%e5%86%85%e9%83%a8%e9%80%bb%e8%be%91todo)
- [搜索](#%e6%90%9c%e7%b4%a2-1)
  - [倒排索引](#%e5%80%92%e6%8e%92%e7%b4%a2%e5%bc%95)
  - [分词器](#%e5%88%86%e8%af%8d%e5%99%a8)
  - [mapping](#mapping)
  - [搜索模式](#%e6%90%9c%e7%b4%a2%e6%a8%a1%e5%bc%8f)
  - [filter 与 query 的区别](#filter-%e4%b8%8e-query-%e7%9a%84%e5%8c%ba%e5%88%ab)
  - [搜索类型](#%e6%90%9c%e7%b4%a2%e7%b1%bb%e5%9e%8b)
  - [搜索连接词](#%e6%90%9c%e7%b4%a2%e8%bf%9e%e6%8e%a5%e8%af%8d)
  - [排序](#%e6%8e%92%e5%ba%8f)
  - [分数算法](#%e5%88%86%e6%95%b0%e7%ae%97%e6%b3%95)
  - [搜索过程](#%e6%90%9c%e7%b4%a2%e8%bf%87%e7%a8%8b)
- [实际问题解决方案](#%e5%ae%9e%e9%99%85%e9%97%ae%e9%a2%98%e8%a7%a3%e5%86%b3%e6%96%b9%e6%a1%88)
  - [不停机重建索引](#%e4%b8%8d%e5%81%9c%e6%9c%ba%e9%87%8d%e5%bb%ba%e7%b4%a2%e5%bc%95)
- [选型比较](#%e9%80%89%e5%9e%8b%e6%af%94%e8%be%83)
- [调优](#%e8%b0%83%e4%bc%98)
  - [索引速度](#%e7%b4%a2%e5%bc%95%e9%80%9f%e5%ba%a6)
  - [搜索速度](#%e6%90%9c%e7%b4%a2%e9%80%9f%e5%ba%a6)

# 基本概念

- 文档（document）: 最小数据单元，可以是一条用户数据，通常用 json 数据结构表示
- 索引（index）: 包含多个 document，表示一类类型的 document
- 类型（type）: type 是 index 中的逻辑数据分类，一个 type 下的 document 拥有相同的 fields。已经不推荐使用
- 碎片（shard）: 一个 document 会被拆分成多个 shard，分布在不同的机器中。好处：1）横向扩展；2）并行计算
- 复制（replica）: shard 的备份

# 基本接口

## 集群管理

```
GET /_cat/health?v  // 了解集群健康状态，green:每个索引的shard和replica都是可用的yellow：所以的shard可用，部分replica不可用；red：部分shard不可用

GET /_cat/indices?v // 查看有哪些索引
```

<!--more-->

## 索引操作

```
PUT /{index-name}/pretty // 创建
DELETE /{index-name}/pretty // 删除
GET /{index-name}/_mapping // 获取index的mapping类型
```

## 文档操作

```
PUT /{index-name}/_doc/1 //创建
{
    "user" : "kim",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}

GET /{index-name}/{id} // 查询

PUT /{index-name}/_doc/{id} // 更新
{
    "user" : "kim"
}

DELETE /{index-name}/{id} // 删除
```

## 批量操作

```
GET /_mget // 批量查询
GET /{index-name}/_mget // 同一个index下批量查询
GET /{index-name}/_doc/_mget // 同一个index和type下批量查询
{
    "docs" : [
        {
            "_index" : "test",
            "_type" : "_doc",
            "_id" : "1",
            "stored_fields" : ["field1", "field2"]
        },
        {
            "_index" : "test",
            "_type" : "_doc",
            "_id" : "2",
            "stored_fields" : ["field1", "field2"]
        }
    ]
}

GET /{index-name}/_doc/_mget?stored_fields=field1,field2 // 第三种，可简化为ids写法
{
    "ids" : ["1", "2"]
}

POST /{index-name}/_bulk
// 一般的形式为，一个json对象不能换行，不同的json对象直接换行
// 这种格式会省略一步json转换，直接将数据路由到处理node，减少内存占用
{"{{action}}": "{{metadata}}"}
{ {{data}} }
```

## 搜索

```
// 使用size和from可以分页查询
// es是分布式系统，分页每个node需要返回的是截至前的所有数据，交给coordinate node进行排序，所以会产生deep paging问题
// index和type都可以使用通配符
GET /[{index1},{index2}]/[{type1},{type2}]/_search?size=10&from=0 //搜索

// +表示必须包含value；-表示必须不包含
// 不添加field表示对每一个field进行搜索，使用es内置的_all字段（所有值拼在一起）
GET /_search?q=[+/-][{field}:]{value} // query string

// 获取数据类型
GET /{index-name}/_mapping

// 判断查询是否合法，request body为要验证的查询对象
GET /{index-name}/{type}/_validate/query?explain

// 滚动搜索，1m为失效时间，request body为查询对象，该操作会返回scroll_id
GET /{index-name}/{type}/_search?scroll=1m
// 滚动搜索之后的内容
GET /_search/scroll
{
    "scroll": 1m,
    "scroll_id": 123132
}

// 分析当前搜索
PUT /{index-name}/_analyze
```

# 分布式架构

- 分片
  - 负载均衡
  - 备份
- 集群发现
- 节点平等的分布式架构
  - master 节点只维护，索引的创建和删除，节点的创建和删除
  - 每个节点都能接收所有的请求，自动路由和响应收集

## 路由

算法：shard = hash(id or custom routing value) % number_of_primary_shards

> 自定义 routing value 可以将某一类的数据路由到同一个 shard，提升批量操作的性能，也能更好的控制负载均衡

## 容错的机制

1. 选举新 master
2. 将丢失的 shard 的 replica 提升变为 primary shard
3. 重启故障及，将挂机时产生的数据 copy 回原 shard 进行同步

## 并发冲突

当并发请求越多，用户操作读和写之间时间较长会发生并发冲突。es 解决方式是使用乐观锁，通过版本号（内置的`if_seq_no`和`if_primary_term`参数；也可以自定义，通过参数`version`和`version_type=external`）控制并发操作。es 同步时是多线程进行的，可能出现后修改的先同步到 replica，所以也有乐观锁控制。

# 数据存储

## document id

- 手动指定：之前的数据已经有 id 时可以手动指定
- 自动生成：es 是存储数据的主要仓库

## \_source 元数据

创建时传给 es 的 json 串，默认都返回，可以添加参数`_source`指定返回的字段

## 替换，删除

- 替换时会把之前的数据设置为逻辑删除
- 部分更新时，与替换一样，会将先前的记录设置为逻辑删除，创建一个新的 document。内部会自动使用乐观锁
- 删除命令也是逻辑删除，当数据越来越多时会自动清理逻辑删除过的数据

## 增删改操作过程

1. 客户端选择一个 node 发送请求，这个 node 就是 coordinating node（协调节点）
2. coordinating node 将 document 进行路由，转发给对应的 shard
3. shard 所在的 node 处理请求，并将信息同步给 replica。当所有 replica 同步成功后，shard 将消息报告给 coordinating node，coordinating node 将消息报告给客户端
   - consistency，一致性。值可以设为 one （只要主分片状态 ok 就允许执行*写*操作）,`all`（必须要主分片和所有副本分片的状态没问题才允许执行*写*操作）, 或 `quorum` 。默认值为 quorum , 即大多数的分片副本状态没问题就允许执行*写*操作。算法`int( (primary + number_of_replicas) / 2 ) + 1`
   - timeout，当没有足够副本时会等待，默认为 1 分钟，可以自行设置

## 读请求操作过程

1. 客户端选择一个 node 发送请求，这个 node 就是 coordinating node（协调节点）
2. coordinating node 将 document 进行路由，使用负载均衡转发给对应的 shard 或 replica
3. 实际执行 node 返回结果给 coordinating node
4. coordinating node 将结果返回给客户端

## 写入更新索引内部逻辑（todo）

# 搜索

## 倒排索引

- 正排索引：文档 id 到单词的关联关系
- 倒排索引：单词到文档 id 的关联关系；一般由单词词典和倒排表组成
- es 倒排索引是不可变的
  - 好处是：不需加锁，可以一直放在缓存中，也可以整块压缩节约 io 和 cpu；数据不变可以一直缓存在内存中；filter cache 一直驻留内存中
  - 坏处是：修改需要重新构建索引

## 分词器

1. character filter：预处理，去除干扰项目
2. tokenizer：分词
3. token filter：大小写，去停，同义词；

## mapping

- 设置 dynamic 策略
  - true：遇到陌生字段，自动进行 dynamic mapping
  - false：遇到陌生字段，忽略
  - strict：遇到陌生字段，报错

## 搜索模式

- exact value：搜索时必须是和关键字相同
- full text：全文检索；缩写，大小写转换，词性装换，同义词等情况也能搜索出来

## filter 与 query 的区别

- filter 只过滤，对相关度没有影响，性能较好
- query 会计算搜索的相关度，并进行排序

## 搜索类型

- match_all：查询所有
- match：某个 field 是否包含查询的词
- multi_match：对应多个 field 是否包含查询
- range：范围
- term：不会对查询语句分词，整个查询
- terms_query: 某个字段指定多个不分词的关键字

## 搜索连接词

- bool
  - must：必须
  - must_not：必须不
  - should：可以
  - filter：必须过滤，不影响评分

## 排序

- 默认：按照\_source 排序
- 自定义：在查询对象加入`sort`对象

## 分数算法

es 使用：term frequency/inverse document frequency 算法，简写为 TF/IDF

- TF: 搜索文本在个词条的 field 出现的次数，越多越相关
- IDF：搜索文本在整个索引中出现的次数，越多越不相关
- field length norm：field 越长，越不相关

## 搜索过程

查询阶段（query phrase）：

1. 协调节点构建一个优先队列（大小为 from + size）
2. 将请求转发到有这个索引分区的节点
3. 每个查询节点构建优先队列，并将查询结果放入，返回给协调节点
4. 协调节点合并队列，构建全局队列，此时获取的是 doc id

获取阶段（fetch phrase):

1. 协调节点根据查询阶段得到的 doc id 发送 mget 请求至索引分区节点获取对象 document
2. 各个节点返回结果给协调节点
3. 协调节点合并返回，将其发送至客户端

# 实际问题解决方案

## 不停机重建索引

当创建索引字段类型错误时，因为不能直接修改索引字段类型，此时唯一办法是重建索引（将旧数据批量查询出来，然后插入新索引）。步骤如下：

1. 新建正确的索引 mapping
2. 使用`scroll api`将数据批量查询出来
3. 使用`bulk api`批量插入数据
4. 重复 2 ～ 3 步骤，直到所有数据写入新索引
5. 将新索引使用 alias 重命名

# 选型比较

| 项目   | 优势                                                                                                                                                                                                                                                                                                                                             | 劣势                                                                                                                                                                                         | 特点                                                                                                                                                                                                                                                                                                                                     |
| ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ES     | Elasticsearch 是分布式的。不需要其他组件，分发是实时的，被叫做”Push replication”;Elasticsearch 完全支持 Apache Lucene 的接近实时的搜索;处理多租户（multitenancy）不需要特殊配置，而 Solr 则需要更多的高级设置;Elasticsearch 采用 Gateway 的概念，使得完备份更加简单;各节点组成对等的网络结构，某些节点出现故障时会自动分配其他节点代替其进行工作 | 只支持 json                                                                                                                                                                                  | Elasticsearch 是一个实时的分布式搜索和分析引擎                                                                                                                                                                                                                                                                                           |
| Solr   | Solr 有一个更大、更成熟的用户、开发和贡献者社区;支持添加多种格式的索引;不考虑建索引的同时进行搜索，速度更快                                                                                                                                                                                                                                      | 建立索引时，搜索效率下降，实时索引搜索效率不高                                                                                                                                               | Solr 是用 Java 编写、运行在 Servlet 容器（如 Apache Tomcat 或 Jetty）的一个独立的全文搜索服务器。 Solr 采用了 Lucene Java 搜索库为核心的全文索引和搜索，并具有类似 REST 的 HTTP/XML 和 JSON 的 API。Solr 强大的外部配置功能使得无需进行 Java 编码，便可对其进行调整以适应多种类型的应用程序。Solr 有一个插件架构，以支持更多的高级定制。 |
| Lucene | 成熟的解决方案，有很多的成功案例。apache 顶级项目，正在持续快速的进步。庞大而活跃的开发社区，大量的开发人员。它只是一个类库，有足够的定制和优化空间：经过简单定制，就可以满足绝大部分常见的需求；经过优化，可以支持 10 亿+ 量级的搜索。                                                                                                          | 需要额外的开发工作。所有的扩展，分布式，可靠性等都需要自己实现；非实时，从建索引到可以搜索中间有一个时间延迟，而当前的“近实时”(Lucene Near Real Time search)搜索方案的可扩展性有待进一步完善 | Lucene 是一个 JAVA 搜索类库，它本身并不是一个完整的解决方案，需要额外的开发工作                                                                                                                                                                                                                                                          |

# 调优

## 索引速度

1. 使用批量请求批量请求将产生比单文档索引请求好得多的性能
2. 充分利用 es 的分布式特性，从多个线程或进程发送数据
3. 调大 refresh interval
4. 加载大量数据时禁用 refresh 和 replicas（可能造成数据丢失的危险，完成后恢复原始值）
5. 为 filesystem cache 分配一半的物理内存
6. 使用自动生成的 id（auto-generated ids）
7. 加大 indexing buffer size

## 搜索速度

1. filesystem cache 越大越好
2. 用更好的硬件
3. 文档模型（document modeling），避免 join 查询
4. 预索引 数据，将经常进行的查询做更多的优化处理
5. Mappings（能用 keyword 最好了）
6. 避免运行脚本
7. 搜索 rounded 日期
8. 强制 merge 只读的 index，只读的 index 可以从“merge 成 一个单独的 大 segment”中收益
9. 预热 filesystem cache
10. 使用 preference 来优化高速缓存利用率。例如，使用标识当前用户或会话的优选值可以帮助优化高速缓存的使用
