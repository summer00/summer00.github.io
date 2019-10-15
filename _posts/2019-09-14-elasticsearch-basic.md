---
title:  "Elasticsearch基础知识"
date:   2019-09-14 12:00:00 +0800
categories: [database]
---

# 基本概念

* 文档（document）: 最小数据单元，可以是一条用户数据，通常用json数据结构表示
* 索引（index）: 包含多个document，表示一类类型的document
* 类型（type）: type是index中的逻辑数据分类，一个type下的document拥有相同的fields。已经不推荐使用
* 碎片（shard）: 一个document会被拆分成多个shard，分布在不同的机器中。好处：1）横向扩展；2）并行计算
* 复制（replica）: shard的备份

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
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}

GET /{index-name}/{id} // 查询

PUT /{index-name}/_doc/{id} // 更新
{
    "user" : "kimchy"
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
```

# 分布式架构

* 分片
  * 负载均衡
  * 备份
* 集群发现
* 节点平等的分布式架构
  * master节点只维护，索引的创建和删除，节点的创建和删除
  * 每个节点都能接收所有的请求，自动路由和响应收集

## 路由

算法：shard = hash(id or custom routing value) % number_of_primary_shards
> 自定义routing value可以将某一类的数据路由到同一个shard，提升批量操作的性能，也能更好的控制负载均衡

## 容错的机制

1. 选举新master
2. 将丢失的shard的replica提升变为primary shard
3. 重启故障及，将挂机时产生的数据copy回原shard进行同步

## 并发冲突

当并发请求越多，用户操作读和写之间时间较长会发生并发冲突。es解决方式是使用乐观锁，通过版本号（内置的`if_seq_no`和`if_primary_term`参数；也可以自定义，通过参数`version`和`version_type=external`）控制并发操作。es同步时是多线程进行的，可能出现后修改的先同步到replica，所以也有乐观锁控制。

# 数据存储

## document id

* 手动指定：之前的数据已经有id时可以手动指定
* 自动生成：es是存储数据的主要仓库

## _source元数据

创建时传给es的json串，默认都返回，可以添加参数`_source`指定返回的字段

## 替换，删除

* 替换时会把之前的数据设置为逻辑删除
* 部分更新时，与替换一样，会将先前的记录设置为逻辑删除，创建一个新的document。内部会自动使用乐观锁
* 删除命令也是逻辑删除，当数据越来越多时会自动清理逻辑删除过的数据

## 增删改操作过程

1. 客户端选择一个node发送请求，这个node就是coordinating node（协调节点）
2. coordinating node将document进行路由，转发给对应的shard
3. shard所在的node处理请求，并将信息同步给replica。当所有replica同步成功后，shard将消息报告给coordinating node，coordinating node将消息报告给客户端
    * consistency，一致性。值可以设为 one （只要主分片状态 ok 就允许执行_写_操作）,`all`（必须要主分片和所有副本分片的状态没问题才允许执行_写_操作）, 或 `quorum` 。默认值为 quorum , 即大多数的分片副本状态没问题就允许执行_写_操作。算法`int( (primary + number_of_replicas) / 2 ) + 1`
    * timeout，当没有足够副本时会等待，默认为1分钟，可以自行设置

## 读请求操作过程

1. 客户端选择一个node发送请求，这个node就是coordinating node（协调节点）
2. coordinating node将document进行路由，使用负载均衡转发给对应的shard或replica
3. 实际执行node返回结果给coordinating node
4. coordinating node将结果返回给客户端

# 搜索

## 倒排索引

* 正排索引：文档id到单词的关联关系
* 倒排索引：单词到文档id的关联关系；一般由单词词典和倒排表组成

## 分词器

1. character filter：预处理，去除干扰项目
2. tokenizer：分词
3. token filter：大小写，去停，同义词；

## mapping

## 搜索模式

* exact value：搜索时必须是和关键字相同
* full text：全文检索；缩写，大小写转换，词性装换，同义词等情况也能搜索出来

## filter与query的区别

* filter只过滤，对相关度没有影响，性能较好
* query会计算搜索的相关度，并进行排序

## 搜索类型

* match_all：查询所有
* match：某个field是否包含查询的词
* multi_match：对应多个field是否包含查询
* range：范围
* term：不会对查询语句分词，整个查询
* terms_query: 某个字段指定多个不分词的关键字

## 搜索连接词

* bool
  * must：必须
  * must_not：必须不
  * should：可以
  * filter：必须过滤，不影响评分

## 排序

* 默认：按照_source排序
* 自定义：在查询对象加入`sort`对象

## 分数算法

es使用：term frequency/inverse document frequency算法，简写为TF/IDF
* TF: 搜索文本在个词条的field出现的次数，越多越相关
* IDF：搜索文本在整个索引中出现的次数，越多越不相关
* field length norm：field越长，越不相关
  
