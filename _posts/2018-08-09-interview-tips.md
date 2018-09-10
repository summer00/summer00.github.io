---
title:  "面试小抄"
date:   2018-08-09 17:00:00 +0800
categories: [summary]
---

# 谈谈`Error`和`Exception`

Error和Exception都是继承于Throwable，Error和RuntimeException及其子类称为未检查异常，其它异常成为受检查异常（Checked Exception）。

Error类一般是指与虚拟机相关的问题，如系统崩溃，虚拟机错误，内存空间不足，方法调用栈溢出等。对于这类错误的导致的应用程序中断，仅靠程序本身无法恢复和预防，遇到这样的错误，建议让程序终止。如`StackOverFlowError`,`outOfMemoryError`

Exception类表示程序可以处理的异常，可以捕获且可能恢复。遇到这类异常，应该尽可能处理异常，使程序恢复运行，而不应该随意终止异常

RuntimeException其特点是Java编译器不去检查它，处理RuntimeException的原则是：如果出现RuntimeException，那么一定是程序员的错误。如`IndexOutOfBoundsException`,`RuntimeException`

Checked Exception，继承自`Exception`，必须被显式地捕获或者传递。如`IOException`,`NoSuchFieldException`
<!--more-->

# 覆盖`equals()`

必须满足的约定：自反性，对称性，传递性，一致性，`x.equals(null)`必须为`false`

注意：
* 覆盖equals时总要覆盖hashCode：如果不这样做的话，就会违反Object.hashcode的通用约定，如果两个对象根据equals(Object)方法比较是相等的，那么调用这两个对象中任意一个对象的hashCode方法都必须产生同样的整数结果
* 不要将equals声明中的Object对象替换为其他的类型

# synchronized原理

* 可重入、互斥锁
* 三种使用方式：1）修饰方法，锁定当前对象 2）修饰静态方法，锁定的当前类的Class实例 3）修饰代码块，锁定指定的对象
* synchronized用的锁是存在Java对象头里的。JVM基于进入和退出Monitor对象来实现方法同步和代码块同步。代码块同步是使用monitorenter和monitorexit指令实现的，monitorenter指令是在编译后插入到同步代码块的开始位置，而monitorexit是插入到方法结束处和异常处。任何对象都有一个monitor与之关联，当且一个monitor被持有后，它将处于锁定状态。根据虚拟机规范的要求，在执行monitorenter指令时，首先要去尝试获取对象的锁，如果这个对象没被锁定，或者当前线程已经拥有了那个对象的锁，把锁的计数器加1；相应地，在执行monitorexit指令时会将锁计数器减1，当计数器被减到0时，锁就释放了。如果获取对象锁失败了，那当前线程就要阻塞等待，直到对象锁被另一个线程释放为止。

# Spring IOC

控制反转，是依赖倒置的一种实现形式

# Spring AOP

AOP 面向切面编程，生成代理类

# 分布式缓存设计

## 问题与解决
