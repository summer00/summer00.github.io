---
title:  "MySQL使用注意点"
date:   2019-01-09 15:00:00 +0800
categories: [database]
---

1. select for update 锁表情况
    对于表products,有两列id和name,id是主键列,name是索引
   
    例1:where条件明确指定主键,并且该行存在,则为row lock
    ```
    SELECT * FROM products WHERE id='3' FOR UPDATE;
    ```
    
    例2:where条件不明确指定主键,则为通过索引列扫描到的数据结果都会lock
    ```
    SELECT * FROM products WHERE name='Mouse' FOR UPDATE;
    ```

    例3:where条件不是索引列,id不是索引，不是主键,则为所有的记录lock
    ```
    SELECT * FROM products WHERE id<>'3' FOR UPDATE;
    ```
<!--more-->

2. 数据库容量
    
    数据库在200G和400G时，会有大幅的性能下降

3. 是否使用外键

    优势：
        
        * 保证数据完整性，帮助开发
        * ER图可读性变强
        * 使用外键关联，提高连表查询的效率
    
    缺点：
        
        * 更多性能的消耗
        * 复杂的外键强关联使得某些操作难以进行

