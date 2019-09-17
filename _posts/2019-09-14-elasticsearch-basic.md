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

```json
GET /_cat/health?v  // 了解集群健康状态，green:每个索引的shard和replica都是可用的yellow：所以的shard可用，部分replica不可用；red：部分shard不可用

GET /_cat/indices?v // 查看有哪些索引
```

<!--more-->

## 索引操作

```json
PUT /{index-name}/pretty // 创建
DELETE /{index-name}/pretty // 删除
GET /{index-name}/_mapping // 获取index的mapping类型
```

## 文档操作

```json
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

```json
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

```json
// 使用size和from可以分页查询
// es是分布式系统，分页每个node需要返回的是截至前的所有数据，交给coordinate node进行排序，所以会产生deep paging问题
GET /[{index1},{index2}]/[{type1},{type2}]/_search?size=10&from=0 //搜索

// +表示必须包含value；-表示必须不包含
// 不添加field表示对每一个field进行搜索，使用es内置的_all字段（所有值拼在一起）
GET /_search?q=[+/-][{field}:]{value} // query string
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
3. shard所在的node处理请求，并将信息同步给replica
4. coordinating node发现shard所在node完成操作后，返回结果给客户端

## 读请求操作过程

1. 客户端选择一个node发送请求，这个node就是coordinating node（协调节点）
2. coordinating node将document进行路由，使用负载均衡转发给对应的shard或replica
3. 实际执行node返回结果给coordinating node
4. coordinating node将结果返回给客户端

# 搜索

## 倒排索引

## 分词器