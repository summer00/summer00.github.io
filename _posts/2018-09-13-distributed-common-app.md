---
title:  "分布式系统基础设施"
date:   2018-09-13 15:00:00 +0800
categories: [distributed]
---

# 分布式缓存

# 持久化存储

# 消息系统 ActiveMQ

ActiveMQ完整支持JMS规范。JMS支持两种消息收发模型：
* Point-to-Point(P2P)模型，点对点的发送模型。基于队列的形式，发送者和接受者都注册到队列，发送者将消息发送给队列，只有一个接受者可以从队列中取出消息
* Pub/Sub模型，发布订阅模型。发送者将消息发送给内容节点（topic，主题），消息订阅者从主题订阅消息，支持一对多的广播形式

高可用：
* ActiveMQ目前的主要方案是基于Master-Slave模式的冷备份，可以基于共享文件或是共享数据库。
* 水平拓展，采用拆分broker的方式，将不相关的queue和topic拆分到多个broker

<!--more-->

# 垂直搜索引擎 Lucene

垂直化搜索引擎在分布式系统中十分重要，它能够解决全文搜索、模糊匹配、解决数据库like查询效率的问题；又能解决分表、分库、NoSQL导致的多表关联或者复杂查询问题

重要概念：
* 倒排索引：将文档中的词作为关键词，建立词与文档的映射
* 分词：将句子切割，从中提取出包含固定语意的词
* 停止词：词频很高且无意义的词，需要被忽略掉
* 排序：命中多个文档后的排序

## 索引构建过程

![Lucene索引构建过程 ]({{ "/assets/2018-09-13-1.png" | absolute_url }})

## 索引搜索过程

![Lucene索引搜索过程 ]({{ "/assets/2018-09-13-2.png" | absolute_url }})

## 分布式拓展

搜索应用可以忍受一定的数据延迟，大部分情况只需保证最终一致性。基于这个特性，可以使用读写分离的做法。每个query server实例保存一份完整索引，然后由生成索引的dump server周期性的推送替换，这样可以避免集群索引dump对后端数据造成压力

当搜索压力更大时，单机的存储、搜索能力会不足，需要对索引进行切分，将索引分布到集群的各个机器上，然后通过merge server将查询请求分发，并且对结果合并

更多

https://www.chedong.com/tech/lucene.html

https://doc.yonyoucloud.com/doc/mastering-elasticsearch/chapter-1/11_README.html