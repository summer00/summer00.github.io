---
title:  "分布式事务"
date:   2019-08-16 12:00:00 +0800
categories: [distributed]
---

# XA

```
Transaction Manager --> Resource Manager --> XA Resource
                    --> Resource Manager --> XA Resource
```
XA协议作为资源管理器（数据库）与事务管理器的接口标准。

XA协议定义了`Transaction Manager`接口，它管理多个`Resource Manager`，每个`Resource Manager`管理一个`XA Resource`，从而实现多个`XA Resource`的事务一致性

# JTA

XA的Java标准。采用两阶段提交方式，保证多数据源的事务同步机制。

## 缺陷

* 两阶段提交，性能是最差性能决定
* 事务时间长，数据锁保持长
* 低性能，低吞吐