---
title:  "MySQL使用注意点"
date:   2019-01-09 15:00:00 +0800
categories: [database]
---
# 测试数据库
https://github.com/datacharmer/test_db

# select for update 锁表情况

对于表products,有两列id和name,id是主键列,name是索引
   
* 例1:where条件明确指定主键,并且该行存在,则为row lock
```sql
    SELECT * FROM products WHERE id='3' FOR UPDATE;
```
* 例2:where条件不明确指定主键,则为通过索引列扫描到的数据结果都会lock
```sql
    SELECT * FROM products WHERE name='Mouse' FOR UPDATE;
```
* 例3:where条件不是索引列,id不是索引，不是主键,则为所有的记录lock
```sql
    SELECT * FROM products WHERE id<>'3' FOR UPDATE;
```
<!--more-->

# 数据库容量
数据库在200G和400G时，会有大幅的性能下降

# 是否使用外键

优势：
  * 保证数据完整性，帮助开发
  * ER图可读性变强
  * 使用外键关联，提高连表查询的效率

缺点：
  * 更多性能的消耗
  * 复杂的外键强关联使得某些操作难以进行

# 分页优化

## 延迟关联
```sql
SELECT count(*) FROM salaries; -- 返回2,844,047

SELECT * FROM salaries WHERE salary <= 94000 LIMIT 2677500,10; -- 3.6s

-- 将id先查询出来，此时会使用索引，查询更快，之后在关联查出其他字段
SELECT * FROM salaries
INNER JOIN (SELECT id FROM salaries WHERE salary <= 94000 LIMIT 2677500,10) AS lim USING(id) --2.1s
```
# 执行过程

1. 链接器（登录用户认证）
2. 查询缓存（若缓存中存在，则直接返回）
3. 查询分析器（是否有语法错误）
4. 优化器（优化查询语句，制定执行计划）
5. 执行器（操作引擎，返回结果）
6. 存储器（存储数据，提供读写接口）