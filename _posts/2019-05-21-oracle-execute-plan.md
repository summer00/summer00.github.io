---
title:  "Oracle执行计划"
date:   2019-05-21 15:00:00 +0800
categories: [database]
---

# 执行顺序

* 缩进最多的最先执行
* 同样缩进先执行上面的
* 同级先执行没id的动作
* 同级遵循最上最右原则

```sql
1 SELECT STATEMENT, GOAL = ALL ROWS
2    SORT GROUP BY
3        NESTED LOOPS OUTER
4            TABLE ACCESS BY GLOBAL INDEX ROWID
5                INDEX RANG SCAN
6            TABLE ACCESS BY INDEX ROWID
7                INDEX UNIQUE SCAN
```
顺序： 5 -> 4 -> 7 -> 6 -> 3 -> 2 -> 1

# 常见动作说明
