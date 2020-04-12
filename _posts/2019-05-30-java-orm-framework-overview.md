---
title:  "Java ORM对比"
date:   2019-05-30 15:00:00 +0800
categories: [database]
---

# Jpa

## 在使用 Jpa 或者 Hibernate 前需要先考虑几个问题：
* 使用何种数据库（关系型 or 非关系型）？ --> 关系型
* 静态域对象 or 可配置的域对象？ --> 静态
* 项目的目标功能（标准CRUD or 复杂的报表） --> 标准CRUD
* 项目中有多少复杂的查询，存储过程，自定义数据库方法 --> 少

## 7个提升 Hibernate 的方法
* 通过Hibernate Statistics发现问题
  * 设置`hibernate.generate_statistics=true`
  * 设置`org.hibernate.stat`的日志等级到DEBUG`
* 优化慢查询
* 选择正确的`FetchType`（大多数情况是`FetchType.LAZY`）
* 使用查询指定特定的`fetching`
  * `join fetch`
  * `@NamedEntityGroup`
  * java api
* 让数据库处理数据密集型操作
* 使用缓存避免重复读取相同数据
* 批处理`update`和`delete`操作

![ hibernate cache ]({{ "/assets/2019-06-03-1.png" | absolute_url }})

<!--more-->

# Mybatis

## 缓存
* 默认开启一级缓存
* 一级缓存是基于`PerpetualCache`的`HashMap`本地缓存，其存储作用域为`Session`，当`Session` `flush`或`close`之后，该Session中的所有缓存就将清空
* 二级缓存默认也是基于`PerpetualCache`的`HashMap`本地缓存，但其作用与为`namespace`
* 二级缓存可以自定义存储源
* 当作用域中（`session` or `namespace`）进行了C/U/D操作后，将清空当前作用域下所有缓存
* 缓存基于查询结果，而不是查询得到的数据
* 缓存默认没有失效时间，需要自定配置`flushInterval`
* 缓存使用最少使用算法（LRU）清除不需要的缓存

