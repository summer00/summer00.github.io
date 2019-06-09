---
title:  "Java并发编程学习笔记"
date:   2019-05-11 15:00:00 +0800
categories: [summary]
---

# 二、 线程安全性

## 2.1 什么是线程安全

* 当多个线程同时访问某个类时，这个类始终能表现正确的行为，那么就称这个类时线程安全的。
* 在线程安全类中封装了必要的同步机制，因此客户端无须进一步采取同步措施。
* 无状态的对象一定是线程安全的

## 2.2 原子性

* 竞态条件：
  * 在并发编程中，由于不恰当的执行时序而出现不正确的结果的情况；
  * 当某个计算的正确性取决于多个线程的交替执行时序时，就会发生竞态条件；
  * 大多数竞态条件的本质是，基于一种可能失效的观察结果来做出判断或者执行某个计算;
  * 要避免这个问题，就需使产生竞态条件的复合操作以原子性的形式完成

## 2.3 加锁机制

* 要保持状态的一致性，就需要在单个的原子操作中更新所有相关的状态变量