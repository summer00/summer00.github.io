---
title: "MySQL总结"
date: 2020-04-04 23:00:00 +0800
categories: [database, mid, iv]
---

- [SQL 优化](#sql-优化)
  - [性能下降的原因](#性能下降的原因)
  - [执行计划](#执行计划)
  - [`count(*)`很慢，怎么办](#count很慢怎么办)
  - [比较`count(id)`、`count(*)`、`count(1)`、`count(字段)`](#比较countidcountcount1count字段)
  - [只返回一条记录的语句（简单）会执行很慢吗？](#只返回一条记录的语句简单会执行很慢吗)
  - [连接优化](#连接优化)
  - [`group by`优化](#group-by优化)
  - [分页优化](#分页优化)
- [原理](#原理)
  - [SQL 执行过程](#sql-执行过程)
  - [索引](#索引)
    - [索引字段选择](#索引字段选择)
    - [注意：](#注意)
    - [自增主键](#自增主键)
  - [排序](#排序)
  - [锁](#锁)
    - [表锁](#表锁)
    - [行锁](#行锁)
    - [间隙锁](#间隙锁)
  - [日志](#日志)
    - [binglog：](#binglog)
- [维护](#维护)
  - [短链接风暴：短链接太多，系统压力大时的临时解决方案](#短链接风暴短链接太多系统压力大时的临时解决方案)
  - [紧急处理慢查询性能的问题](#紧急处理慢查询性能的问题)
  - [主备切换 -- 一主一从](#主备切换----一主一从)
  - [主备切换 -- 一主多从](#主备切换----一主多从)
  - [主备复制的策略：](#主备复制的策略)
  - [读写分离架构](#读写分离架构)
  - [读写分离的情况下解决延迟](#读写分离的情况下解决延迟)
  - [数据库健康状况监视](#数据库健康状况监视)
  - [误删数据处理](#误删数据处理)
  - [怎样快速地复制一张表](#怎样快速地复制一张表)

# SQL 优化

## 性能下降的原因

1. 没有索引
2. 索引创建不当
3. 数据变化，如数据量增大、特征值变化
4. join 过多
5. 配置不当

## 执行计划

1. `id`越大越先执行，相同 id 顺序执行
2. `select_type`查询类型：
   1. simple：简单查询
   2. primary： 使用主键
   3. derive：衍生表
   4. subquery：子查询
   5. union\union result：合并
3. `table`查询表，`partition`查询分区
4. `type`连接类型（访问类型），最好到最差：
   1. system：表中只有一行记录
   2. const：只查询一次
   3. eq_ref：唯一性索引扫描
   4. ref：查找条件列使用了索引而且不为主键和 unique。语句最好能达到这种情况
   5. range：索引范围查询
   6. index：全量索引扫描
   7. all：全表
5. `possible_keys`：可能用到的索引
6. `keys`：实际用到的索引
7. `key_len`：使用索引的长度
8. `ref`：索引使用的具体值，常量或者是其他列的值
9. `rows`：扫描行数
10. `filter`：用到的行数与扫描行数的百分例，越大越好，最大 100
11. `extra`：额外信息
    1. using filesort：使用文件排序
    2. using temporary：使用临时表
    3. using index：使用索引
    4. using where：使用查询条件
    5. using join buffer：使用链接缓存
    6. impossible value：条件永远不会达成

## `count(*)`很慢，怎么办

为什么慢：InnoDB 支持事务，采用多版本并发控制(MVCC)，每次执行`count(*)`返回的行数是不确定的，所以执行的时候会逐行读取后，累积计数。

MySQL 优化：InnoDB 是索引组织表，主键索引树的叶子节点是数据，而普通索引树的叶子节点是主键值。所以，普通索引树比主键索引树小很多。对于 `count(*)` 这样的操作，遍历哪个索引树得到的结果逻辑上都是一样的。因此，MySQL 优化器会找到最小的那棵树来遍历。会在保证逻辑正确的前提下，尽量减少扫描的数据量，是数据库系统设计的通用法则之一。

方案：

1. Redis 计数，由于调用 redis 时机不同，可能结果不准确
2. 单独存储，用事务的方式解决不准确的问题

## 比较`count(id)`、`count(*)`、`count(1)`、`count(字段)`

- 对于 count(主键 id) 来说，InnoDB 引擎会遍历整张表，把每一行的 id 值都取出来，返回给 server 层。server 层拿到 id 后，判断是不可能为空的，就按行累加。
- 对于 count(1) 来说，InnoDB 引擎遍历整张表，但不取值。server 层对于返回的每一行，放一个数字“1”进去，判断是不可能为空的，按行累加。
- 对于 count(字段) 来说：
  - 如果这个“字段”是定义为 not null 的话，一行行地从记录里面读出这个字段，判断不能为 null，按行累加；
  - 如果这个“字段”定义允许为 null，那么执行的时候，判断到有可能是 null，还要把值取出来再判断一下，不是 null 才累加。
- 但是`count(*)`是例外，并不会把全部字段取出来，而是专门做了优化，不取值。`count(*)` 肯定不是 null，按行累加。

所以结论是：按照效率排序的话，`count(字段)` < `count(id)` < `count(1)` =. `count(*)`

## 只返回一条记录的语句（简单）会执行很慢吗？

会，有两种情况：

1. 长时间无相应
   1. 等待锁（表级、行级）
   2. 等待 flush
2. 查询慢
   1. 扫描行多
   2. 事务长，其他链接执行了很多操作，导致回到事务开始状态做查询的时间变长

## 连接优化

**最佳实践**：小表驱动大表

- `join`被驱动表上有索引时会使用**Index Nested-Loop Join**，驱动表扫描，被驱动表走索引树。小表做驱动表。
- `join`被驱动表上没有索引时会使用**Block Nested-Loop Join**，将驱动表存入缓存中，被驱动表依次访问判断是否可以作为返回。当缓存足够大时是一样的，当缓存不够大时小表做驱动表更好。
- `in`先执行条件，`exist`先执行查询。选择遵循小表驱动大表原则

## `group by`优化

- 如果对`group by`没有排序要求，要在语句后加`order by null`
- 尽量让`group by`过程用上索引，用`explain`确认没有使用临时表（`Using temporary` or `Using filesort`）
- 如果`group by`需要统计的数据量不大，尽量只使用内存临时表；也可以通过适当调大`tmp_table_size`参数，来避免用到磁盘临时表；
- 如果数据量实在太大，使用`SQL_BIG_RESULT`这个提示，来告诉优化器直接使用排序算法得到`group by`的结果

## 分页优化

分页会找到 offset+size 条数据后，抛弃前 offset 条数据，故执行分页越深查询与慢。优化方式：

1. 使用索引，比如先查 id
2. 定位索引开始，然后查询大于该值的数据的 size 条数据

# 原理

## SQL 执行过程

## 索引

### 索引字段选择

1. 适合：
   1. 经常查询的字段
   2. 外键关系字段
   3. 排序、分组字段
2. 不适合
   1. 经常变更的字段
   2. 不作为查询条件的字段
   3. 重复度高的字段

### 注意：

1. 对索引字段做函数、计算、类型转换操作，不会使用索引
2. 查询条件使用`！=` `<>` `is not null` `or`，不会使用索引
3. 查询条件使用`%`开头，不会使用索引
4. 组合索引需遵守最左前缀原则
5. 尽量使用覆盖索引（查询字段刚刚与索引匹配），不用回表

### 自增主键

不同引擎存储自增主键的地方不相同：

- MyISAM 是保存在数据文件中；
- InnoDB 在 8.0 之前保存在内存中，重启后寻找最大值+1 作为当前的自增值
- InnoDB 在 8.0 之后保存在`redo log`中，重启时通过`redo log`恢复

如果插入的数据没有对自增主键列赋值，就是用当前值；若赋值则使用给定的值，当给定值大于当前自增值时会修改为给定值下一个值。

自增值可以是不连续的，造成原因有三种：

1. 唯一键冲突
2. 事务回滚
3. `insert...select`操作时会通过批量申请 id 多申请值，这些值会被浪费掉

## 排序

MySQL 执行带排序的查询时有两种情况：

- 当没有对应的排序索引时，使用`sort_buffer`在内存中缓存所有字段进行全排序
  - 如果只取排序结果的很小的数据集（不超过`sort_buffer_size`)，会使用优先队列排序
  - 如果排序的数据量太大（大于`sort_buffer_size`）内存放不下会使用临时文件归并排序方式辅助排序
  - 如果 MySQL 认为单行数据太大，会使用 rowid 排序，rowid 排序多访问了一次表的主键索引
- 当有对应顺序的索引时，直接使用索引的排序完成查询，若索引是覆盖索引会减少一次回表操作

## 锁

### 表锁

### 行锁

### 间隙锁

引用极客时间 MySQL 实战中的总结的可重复读情况下的加锁规则：

> 原则 1：加锁的基本单位是 next-key lock。希望你还记得，next-key lock 是前开后闭区间。<br>
> 原则 2：查找过程中访问到的对象才会加锁。<br>
> 优化 1：索引上的等值查询，给唯一索引加锁的时候，next-key lock 退化为行锁。此时，行锁和间隙锁是分两步完成的。<br>
> 优化 2：索引上的等值查询，向右遍历时且最后一个值不满足等值条件的时候，next-key lock 退化为间隙锁。<br>
> 一个 bug：唯一索引上的范围查询会访问到不满足条件的第一个值为止。<br>

![ table t ]({{ "/assets/2020-04-12-1.png" | absolute_url }})

案例 1-唯一索引等值查询的间隙锁：若等值查询中，没有命中的行，则锁单位是 next-key lock

```sql
-- session A
begin; update t set d = d + 1 where id = 7;

-- session B
insert into t values(8,8,8); -- 阻塞，锁(5,10]区间

-- session C
insert into t values(11,8,8); -- 正常执行
update t set d=d+1 where id = 10; -- 正常执行
```

案例 2-非唯一索引等值锁：只有访问到的对象才会加锁，需注意：锁存在与索引上，若使用覆盖索引，并不会锁主键。

```sql
-- session A
select id from t where c=5 lock in share mode;

-- session B
update t set d=d+1 where id =5; -- 不会被锁，正常执行，锁的是c索引(5,10)区间

-- session C
insert into t values (7,7,7); -- 阻塞
```

案例 3-主键索引范围锁：锁扫描到的行，若出现等值根据优化 1 会退化成行锁。

案例 4-非唯一索引范围锁：锁扫描到的行，因为非唯一索引不会进行等值退化成行锁

案例 5-limit 语句加锁：锁扫描到的行，若满足了 limit 会优化不用锁到 next key

## 日志

Innodb 日志有两种，binglog(MySQL 提供) 和 redolog(引擎实现)。

### binglog：

作用：归档，主备同步，数据恢复

格式：

1. statement 直接记录语句，有可能主备不一致
2. row 记录变化的行和数据，缺点是很占空间，如果一个语句改变多条记录，row 模式也会记录多条
3. mixed 一般情况下用 statement，遇到有可能主备不一致的情况用 row

# 维护

## 短链接风暴：短链接太多，系统压力大时的临时解决方案

1. 处理掉占着链接但不工作的线程，优先断开事物外空闲链接
2. 减少连接过程的消耗，如跳过验证（风险极高）

## 紧急处理慢查询性能的问题

慢查询通常有以下三种原因造成：

1. 索引没有设计好：紧急加索引解决（ONLINE DDL）
2. SQL 语句写的不好：用`query_rewrite`方法重写语句
3. MySQL 选错索引：加`force index`

预防：测试阶段在数据库中打开慢查询日志，并设置`long_query_time`为 0，记录每条语句，分析每个语句的扫描行和使用的索引是否和设想一致，提早发现问题。

## 主备切换 -- 一主一从

MySQL 通常使用主备模式实现高可用，主备同步之间会有一定的延迟，造成延迟的原因主要有：主备性能差异；备库执行其他耗时统计分析任务；大事务同步。由于延迟存在，所以切换主备时有两种策略：

1. 可靠性优先：这种方式会有不可用的时间，其中第 3 步比较耗时
   1. 判断备库延迟，若小于阈值，继续下一步，否则重试
   2. 将主库设置为只读状态
   3. 判断备库的延迟，直至延迟为 0
   4. 将备库设置为可写状态
   5. 切换流量
2. 可用性优先：将 4、5 步骤调整到最开始执行，之后通过 binglog 补数据

## 主备切换 -- 一主多从

此时从库找新主库的位点是一个痛点，5.6 版本加入了每个事务都有一个唯一的 GTID，可以根据它完成找位点的工作，从库同步时维护自己的 GTID 集合，切换主库后可以通过这个集合方便的跳过其他从库已经执行了的事务

## 主备复制的策略：

执行
MySQL5.5 之前官方支持单线程复制，如果主库并发度比较高，吞吐量会大于单线程的复制（生成日志的速度大于同步日志的速度），导致备库落后主库很多。

策略原则：

1. 对相同行的操作，需要按顺序执行
2. 对于一个事务里的操作，需要按顺序执行

策略：

1. MySQL5.6 按库并行复制
2. MariaBD redo log 组提交（group commit）优化，将不修改相同行的提交放在同一个 group 中，这些语句就是可以并发执行的
3. MySQL5.7 同时处于 redo log 的 prepare 状态和 commit 状态的语句可以并行执行
4. MySQL 5.7.22 除了 5.7 多提供了一种基于 hash 的方式，这个 hash 值通过库名+表名+索引名+值在写 binlog 时计算出来的，如果两个语句的 hash 没有交集，就可以并行执行

## 读写分离架构

1. 客户端路
   - 优点： 简单、性能好
   - 缺点： 需要知道后端细节
2. MySQL 和客户端之间加入中间代理层 Proxy
   - 优点： 不需要知道后端细节、主从切换客户端无感
   - 缺点： Proxy 层引入新的复杂性，且 Proxy 也需要做高可用

## 读写分离的情况下解决延迟

读写分离情况下，在主库新修改/新增了数据就立即查询，这时如果查的是从库，可能会由于延迟的原因看不到刚刚修改/新增的数据。解决该问题有以下方式：

1. 指定语句只读主库
2. 写入后等待一段时间后再查询，
3. 确定主备无延迟时执行查询，可以通过查询`seconds_behind_master`参数为 0，或者比较 GTID 是否一致确认
4. 配合 semi-sync，如果启用了 semi-sync，就表示所有给客户端发送过确认的事务，都确保了备库已经收到了这个日志

## 数据库健康状况监视

1. 差表判断：`select * from t`。无法检查磁盘空间等问题
2. 更新判断：`update t set update_time = now()`。有一定的延迟
3. 内部统计：`performance_schema`库中的表。有性能消耗

## 误删数据处理

- 如果是使用`delete`语句误删了几条数据可以用**Flashback**工具恢复，建议在从库或者临时库操作，确认无误后再执行。可以通过`sql_safe_updates`设置为`on`来避免误删，同时 SQL 一定要 review。
- 如果是误删了整个数据表或数据库，就需要使用全量备份加增量日志的方式恢复，这个过程可能比较耗时。可考虑使用**延迟复制的备库**方式减少全量恢复的时间。可以通过帐号权限隔离，只给开发或系统 DML 权限，预防误删。同时也要规范操作，先做备份再删除。

## 怎样快速地复制一张表

1. `mysqldump`

   ```sql
   mysqldump -h$host -P$port -u$user --add-locks=0 --no-create-info --single-transaction  --set-gtid-purged=OFF db1 t --where="a>900" --result-file=/client_tmp/t.sql

   mysql -h127.0.0.1 -P13000  -uroot db2 -e "source /client_tmp/t.sql"
   ```

2. 导出 csv 文件

   ```sql
   select * from db1.t where a>900 into outfile '/server_tmp/t.csv';

   load data infile '/server_tmp/t.csv' into table db2.t;
   ```
