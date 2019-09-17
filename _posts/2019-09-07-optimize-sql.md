---
title:  "SQL优化"
date:   2019-09-07 12:00:00 +0800
categories: [database]
---
- [索引](#%e7%b4%a2%e5%bc%95)
  - [类型](#%e7%b1%bb%e5%9e%8b)
  - [聚集索引](#%e8%81%9a%e9%9b%86%e7%b4%a2%e5%bc%95)
- [物理结构](#%e7%89%a9%e7%90%86%e7%bb%93%e6%9e%84)
  - [共享池优化](#%e5%85%b1%e4%ba%ab%e6%b1%a0%e4%bc%98%e5%8c%96)
    - [绑定变量，节省解析时间](#%e7%bb%91%e5%ae%9a%e5%8f%98%e9%87%8f%e8%8a%82%e7%9c%81%e8%a7%a3%e6%9e%90%e6%97%b6%e9%97%b4)
  - [日志优化](#%e6%97%a5%e5%bf%97%e4%bc%98%e5%8c%96)
- [Oracle逻辑体系结构](#oracle%e9%80%bb%e8%be%91%e4%bd%93%e7%b3%bb%e7%bb%93%e6%9e%84)
  - [最小单位block](#%e6%9c%80%e5%b0%8f%e5%8d%95%e4%bd%8dblock)
    - [一个block能装多少行？](#%e4%b8%80%e4%b8%aablock%e8%83%bd%e8%a3%85%e5%a4%9a%e5%b0%91%e8%a1%8c)
    - [行迁移](#%e8%a1%8c%e8%bf%81%e7%a7%bb)
    - [行链接](#%e8%a1%8c%e9%93%be%e6%8e%a5)
    - [优化点](#%e4%bc%98%e5%8c%96%e7%82%b9)
  - [段（segment）](#%e6%ae%b5segment)
    - [segment高水平位](#segment%e9%ab%98%e6%b0%b4%e5%b9%b3%e4%bd%8d)
    - [优化点](#%e4%bc%98%e5%8c%96%e7%82%b9-1)
- [表设计](#%e8%a1%a8%e8%ae%be%e8%ae%a1)
  - [分区表](#%e5%88%86%e5%8c%ba%e8%a1%a8)
  - [全局临时表](#%e5%85%a8%e5%b1%80%e4%b8%b4%e6%97%b6%e8%a1%a8)
- [优化点总结](#%e4%bc%98%e5%8c%96%e7%82%b9%e6%80%bb%e7%bb%93)
  - [减少全表扫描](#%e5%87%8f%e5%b0%91%e5%85%a8%e8%a1%a8%e6%89%ab%e6%8f%8f)
  - [其他](#%e5%85%b6%e4%bb%96)

# 索引

## 类型

* 主键：一种特殊的唯一索引，不允许有空值。
* 唯一键:索引列的值必须唯一，但允许有空值。
* 普通
* 组合（最左前缀）:为了更多的提高mysql效率可建立组合索引，遵循”最左前缀“原则。创建复合索引时应该将最常用（频率）作限制条件的列放在最左边，依次递减。组合索引最左字段用in是可以用到索引的。

<!--more-->

## 聚集索引

* 聚集索引：索引中键值的逻辑顺序决定了表中相应行的物理顺序（索引中的数据物理存放地址和索引的顺序是一致的）。
* 非聚集索引：索引的逻辑顺序与磁盘上的物理存储顺序不同。
  
Inno DB的聚集索引规则：

* 如果一个主键被定义了，那么这个主键就是作为聚集索引
* 如果没有主键被定义，那么该表的第一个唯一非空索引被作为聚集索引
* 如果没有主键也没有合适的唯一索引，那么innodb内部会生成一个隐藏的主键作为聚集索引，这个隐藏的主键是一个6个字节的列，改列的值会随着数据的插入自增。

# 物理结构

## 共享池优化

### 绑定变量，节省解析时间

排查方式：
* awr报表
* trace
  ```sql
  overall totals for all recursive statements
  ```

反例：

* 影响SQL索引选择
  ``` sql
  select count(*) from t where id < 990;  -- 全表
  select count(*) from t where id < 10;   -- 索引
  select count(*) from t where id < :id;  -- 不做优化，一直使用索引
  ```

## 日志优化

* 事务提交需要写日志，批处理减小日志的性能损耗

# [Oracle逻辑体系结构](https://www.oraclejsq.com/oraclegl/010300758.html)

```
logical : database --> table space --> segment --> extent --> data block
physical:              data file               -->            OS block
```
![ oracle logic data structure ]({{ "/assets/2019-09-07-2.png" | absolute_url }})
![ oracle logic data structure ]({{ "/assets/2019-09-07-1.png" | absolute_url }})

## 最小单位block

* 数据块头（类型，地址，归属segment）
* 表目录（某行数据插入到块中，该行数据所在表的信息）
* 行目录（行地址）
* 可用空间（剩余空间；若是表或索引块，会存放事物条目）
* 行数据区（行或索引的信息）
  
### 一个block能装多少行？

各种开销导致，每行最小长度大致是11字节，例如，一个8k块理论上最多存储不超过8096/11行

### 行迁移

* 成因：当行update时，若update更新的行大于数据库的pctfree（可用空间）就需要申请新的块，从而形成迁移
* 后果：导致应用需要更多的快，性能下降
* 预防：pctfree调大；块调大
* 检查：
  ```sql
  analyze table <table name> validate structure cascade into chained_rows
  ```

### 行链接

* 成因：如果我们往数据库中插入（INSERT）一行数据，这行数据很大，以至于一个数据块存不下一整行，Oracle就会把一行数据分作几段存在几个数据块中，这个过程叫行链接（Row Chaining）

### 优化点

* block空间越大，逻辑读越少
* block空间越大，并发争抢更激烈
 
## 段（segment）

建表产生表段，建索引产生索引段，空间申请以区（extend）为单位，最小单位是块（block）。extend以若干个连续的block组成。随着记录变多，segment包含的extents和blocks也增多。

### [segment高水平位](https://www.iteye.com/blog/czmmiao-2185543)

* 成因：delete无法降低高水平位，表扫描依然需要大量的逻辑读，并且表的大小依然不变。move能减低高水平位，逻辑读和表大小也会减小。
* 检查：blocks和row nums不成比例；全表扫描时间异常

### 优化点

* 利用分区表优化查询，因为分区可以只扫描特定的段，性能更好

# 表设计

## 分区表

分区表可以包括多个分区， 每个分区都是一个独立的段（ SEGMENT），可以存放到不同的表空间中 。查询时可以通过查询表来访问各个分区中的数据，也可以通过在查询时直接指定分区的方法来进行查询

分区表有四种方式：范围R（月，天），列表L（地区，区号），哈希H（负载均衡），组合：11g前（RL，RH）；11g后（RR，LL，LH，LR）。

优点：
* 由于将数据分散到各个分区中，减少了数据损坏的可能性；
* 可以对单独的分区进行备份和恢复；
* 可以将分区映射到不同的物理磁盘上，来分散IO；
* 提高可管理性、可用性和性能。

## 全局临时表

如果数据是临时的，也就是说用完即抛，需要频繁执行删除操作（删除操作会造成大量日志写入，占用服务） 

# 优化点总结

## 减少全表扫描

* 减少不必要的方法调用，缩小调用方法的次数
* 只取需要的列（只用索引无需回表；只用索引连表速度变快）
* 索引优化

## 其他

* 批操作能提升性能
* 使用绑定变量，减少硬解析次数
* 功能性表选取，临时表或分段表
* 合理设置块，区，表空间大小