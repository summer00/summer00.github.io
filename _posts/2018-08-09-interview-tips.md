---
title:  "面试小抄"
date:   2018-08-09 17:00:00 +0800
categories: [summary]
---

- [JVM](#jvm)
  - [内存模型](#%E5%86%85%E5%AD%98%E6%A8%A1%E5%9E%8B)
    - [线程共享隔离](#%E7%BA%BF%E7%A8%8B%E5%85%B1%E4%BA%AB%E9%9A%94%E7%A6%BB)
    - [解释](#%E8%A7%A3%E9%87%8A)
  - [对象创建时内存分配方式](#%E5%AF%B9%E8%B1%A1%E5%88%9B%E5%BB%BA%E6%97%B6%E5%86%85%E5%AD%98%E5%88%86%E9%85%8D%E6%96%B9%E5%BC%8F)
  - [GC](#gc)
    - [回收哪里的对象：](#%E5%9B%9E%E6%94%B6%E5%93%AA%E9%87%8C%E7%9A%84%E5%AF%B9%E8%B1%A1)
    - [哪些对象需要被回收：](#%E5%93%AA%E4%BA%9B%E5%AF%B9%E8%B1%A1%E9%9C%80%E8%A6%81%E8%A2%AB%E5%9B%9E%E6%94%B6)
    - [何时回收](#%E4%BD%95%E6%97%B6%E5%9B%9E%E6%94%B6)
    - [方法](#%E6%96%B9%E6%B3%95)
  - [类的生命周期](#%E7%B1%BB%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F)
  - [synchronized原理](#synchronized%E5%8E%9F%E7%90%86)
  - [常用参数](#%E5%B8%B8%E7%94%A8%E5%8F%82%E6%95%B0)
- [JDK源码](#jdk%E6%BA%90%E7%A0%81)
  - [JDK线程池实现](#jdk%E7%BA%BF%E7%A8%8B%E6%B1%A0%E5%AE%9E%E7%8E%B0)
  - [CurrentHashMap实现](#currenthashmap%E5%AE%9E%E7%8E%B0)
  - [concurrent包中的类](#concurrent%E5%8C%85%E4%B8%AD%E7%9A%84%E7%B1%BB)
    - [CyclicBarrier](#cyclicbarrier)
    - [CountDownLatch](#countdownlatch)
    - [Semaphore](#semaphore)
    - [总结](#%E6%80%BB%E7%BB%93)
- [Spring](#spring)
  - [Spring IOC](#spring-ioc)
  - [流程](#%E6%B5%81%E7%A8%8B)
    - [初始化](#%E5%88%9D%E5%A7%8B%E5%8C%96)
    - [依赖注入](#%E4%BE%9D%E8%B5%96%E6%B3%A8%E5%85%A5)
  - [bean生命周期](#bean%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F)
  - [FactoryBean](#factorybean)
  - [Spring AOP](#spring-aop)
  - [Spring Bean创建过程](#spring-bean%E5%88%9B%E5%BB%BA%E8%BF%87%E7%A8%8B)
- [分布式缓存设计](#%E5%88%86%E5%B8%83%E5%BC%8F%E7%BC%93%E5%AD%98%E8%AE%BE%E8%AE%A1)
  - [问题与解决](#%E9%97%AE%E9%A2%98%E4%B8%8E%E8%A7%A3%E5%86%B3)
- [MySQL](#mysql)
  - [锁](#%E9%94%81)
  - [隔离级别](#%E9%9A%94%E7%A6%BB%E7%BA%A7%E5%88%AB)
  - [索引](#%E7%B4%A2%E5%BC%95)
- [Tips](#tips)
  - [`Error`和`Exception`](#error%E5%92%8Cexception)
  - [覆盖`equals()`](#%E8%A6%86%E7%9B%96equals)
  - [重写（overloading）与重载（overwrite）](#%E9%87%8D%E5%86%99overloading%E4%B8%8E%E9%87%8D%E8%BD%BDoverwrite)
  - [final, finally, finalize](#final-finally-finalize)
  - [Object类方法](#object%E7%B1%BB%E6%96%B9%E6%B3%95)
- [微服务](#%E5%BE%AE%E6%9C%8D%E5%8A%A1)
  - [什么是](#%E4%BB%80%E4%B9%88%E6%98%AF)
  - [优点](#%E4%BC%98%E7%82%B9)
  - [挑战](#%E6%8C%91%E6%88%98)
  - [设计原则](#%E8%AE%BE%E8%AE%A1%E5%8E%9F%E5%88%99)
- [消息中间件](#%E6%B6%88%E6%81%AF%E4%B8%AD%E9%97%B4%E4%BB%B6)
  - [为什么使用](#%E4%B8%BA%E4%BB%80%E4%B9%88%E4%BD%BF%E7%94%A8)
  - [rabbitMQ](#rabbitmq)

# JVM

## 内存模型

### 线程共享隔离 

* 线程共享：堆，方法区（包括运行区常量池）
* 线程隔离：方法栈，本地方法栈，程序计数器

### 解释

* 堆：在虚拟机启动的时候创建，被所有线程共享。对象存在的地方，GC的主要场所。
* 方法区：线程共享的区域。存放被加载的类信息，常量，静态变量，即时编译后的代码等数据。
* 运行时常量池：存储在类加载后生成的各种字面量和符号引用。
* 方法栈：线程私有，与线程的生命周期相同。描述java执行的内存模型：每个方法执行是都会创建一个栈帧用来存储局部变量表，方法出口，操作数栈，动态链接等，方法开始时入栈，结束时出栈
* 本地方法栈：与虚拟机栈类似，只不过执行的是Native方法
* 程序计数器：线程私有的，是当前执行的字节码行号指示器
* 直接内存：并不是运行时数据区的一部分，也不是jvm规范中定义的内存区域，但被频繁使用。NIO类引入了一种基于channel和buffer的IO方式，可以直接使用Native函数库分配堆外内存，然后通过`DirectByteBuffer`对象作为这块区域的直接引用进行操作。

<!--more-->

## 对象创建时内存分配方式

* 指针碰撞，规整
* 空闲列表，不规整，已使用和未使用交错

## GC

### 回收哪里的对象：

堆，方法区，本地方法区，虚拟机栈

### 哪些对象需要被回收：

当对象访问不到时会被回收，不再被任何存活的对象继续引用。
* 计数法
* 根搜索法：以根对象集合作为起点，从上到下的方式搜索被根对象集合所连接的对象是否可达。根对象集合包括：栈中对象引用，本地方法栈对象引用，常量池对象引用，方法区类静态属性对象引用，与类唯一对应的Class对象

### 何时回收

* 对于 Minor GC，其触发条件非常简单，当 Eden 空间满时，就将触发一次 Minor GC
* Full GC触发条件
  * 调用 System.gc()。只是建议虚拟机执行 Full GC，但是虚拟机不一定真正去执行。不建议使用这种方式，而是让虚拟机管理内存。
  * 老年代空间不足的常见场景为前文所讲的大对象直接进入老年代、长期存活的对象进入老年代等。为了避免以上原因引起的 Full GC，应当尽量不要创建过大的对象以及数组。除此之外，可以通过 -Xmn 虚拟机参数调大新生代的大小，让对象尽量在新生代被回收掉，不进入老年代。还可以通过 -XX:MaxTenuringThreshold 调大对象进入老年代的年龄，让对象在新生代多存活一段时间。
  * 空间分配担保失败。使用复制算法的 Minor GC 需要老年代的内存空间作担保，如果担保失败会执行一次 Full GC
  * JDK 1.7 及以前的永久代空间不足。在 JDK 1.7 及以前，HotSpot 虚拟机中的方法区是用永久代实现的，永久代中存放的为一些 Class 的信息、常量、静态变量等数据。当系统中要加载的类、反射的类和调用的方法较多时，永久代可能会被占满，在未配置为采用 CMS GC 的情况下也会执行 Full GC。如果经过 Full GC 仍然回收不了，那么虚拟机会抛出 java.lang.OutOfMemoryError。为避免以上原因引起的 Full GC，可采用的方法为增大永久代空间或转为使用 CMS GC。
  * Concurrent Mode Failure。执行 CMS GC 的过程中同时有对象要放入老年代，而此时老年代空间不足（可能是 GC 过程中浮动垃圾过多导致暂时性的空间不足），便会报 Concurrent Mode Failure 错误，并触发 Full GC。

### 方法

* 标记清除：标记后直接回收
* 复制算法：用一半内存，空一半，直接把还有用的复制到空的一半
* 标记-整理：标记后，把存活对象复制到内存一端
* 分代回收算法：把堆分为新生代和老年代，新生代用复制算法，老年代用标记清除/整理

## 类的生命周期

加载--》验证--》准备--》解析--》初始化--》使用--》卸载

## synchronized原理

* 可重入、互斥锁
* 三种使用方式：1）修饰方法，锁定当前对象 2）修饰静态方法，锁定的当前类的Class实例 3）修饰代码块，锁定指定的对象
* synchronized用的锁是存在Java对象头里的。JVM基于进入和退出Monitor对象来实现方法同步和代码块同步。代码块同步是使用monitorenter和monitorexit指令实现的，monitorenter指令是在编译后插入到同步代码块的开始位置，而monitorexit是插入到方法结束处和异常处。任何对象都有一个monitor与之关联，当且一个monitor被持有后，它将处于锁定状态。根据虚拟机规范的要求，在执行monitorenter指令时，首先要去尝试获取对象的锁，如果这个对象没被锁定，或者当前线程已经拥有了那个对象的锁，把锁的计数器加1；相应地，在执行monitorexit指令时会将锁计数器减1，当计数器被减到0时，锁就释放了。如果获取对象锁失败了，那当前线程就要阻塞等待，直到对象锁被另一个线程释放为止。

## 常用参数

| 参数                     | 描述                                                                                         |
|--------------------------|----------------------------------------------------------------------------------------------|
| -Xms                     | 设置Java堆大小的初始值/最小值。例如：-Xms512m (请注意这里没有”=”).                           |
| -Xmx                     | 设置Java堆大小的最大值                                                                       |
| -Xmn                     | 设置年轻代对空间的初始值，最小值和最大值。请注意，年老代堆空间大小是依赖于年轻代堆空间大小的 |
| -XX:PermSize=n [g/m/k]   | 设置持久代堆空间的初始值和最小值                                                             |
| -XX:MaxPermSize=n[g/m/k] | 设置持久代堆空间的最大值                                                                     |

# JDK源码

## JDK线程池实现

## CurrentHashMap实现

## concurrent包中的类

### CyclicBarrier

字面意思回环栅栏，通过它可以实现让一组线程等待至某个状态之后再全部同时执行。叫做回环是因为当所有等待线程都被释放以后，CyclicBarrier可以被重用。我们暂且把这个状态就叫做barrier，当调用await()方法之后，线程就处于barrier了。

### CountDownLatch

利用它可以实现类似计数器的功能。比如有一个任务A，它要等待其他4个任务执行完毕之后才能执行，此时就可以利用CountDownLatch来实现这种功能了。

### Semaphore

Semaphore翻译成字面意思为 信号量，Semaphore可以控同时访问的线程个数，通过 acquire() 获取一个许可，如果没有就等待，而 release() 释放一个许可。

### 总结

* CountDownLatch和CyclicBarrier都能够实现线程之间的等待，只不过它们侧重点不同：
  * CountDownLatch一般用于某个线程A等待若干个其他线程执行完任务之后，它才执行；
  * 而CyclicBarrier一般用于一组线程互相等待至某个状态，然后这一组线程再同时执行；
  * CountDownLatch是不能够重用的，而CyclicBarrier是可以重用的。
* Semaphore其实和锁有点类似，它一般用于控制对某组资源的访问权限。



# Spring

## Spring IOC

控制反转，是依赖倒置的一种实现形式。

## 流程

### 初始化

容器初始化的过程就是将我们定义在 xml 或者使用注解的 bean 信息注册到ioc容器的过程。首先会做的是解析元信息，然后将解析到的信息封装成BeanDefinition对象，最后将对象保存在BeanDefinition容器（一个hashmap）中。

### 依赖注入

容器注册完成BeanDefinition后，如果bean没有设置lazyInit会对bean进行实例化，如果设置了会在第一次调用时对bean实例化。首先取得 BeanDefinition，然后根据里面的信息循环调用得到依赖的Bean，这里会触发一个第归调用getBean方法，直到当前初始化bean的所有依赖都得到，将他们注册到当前bean的依赖关系中。然后进行当前bean的创建，并根据BeanDefinition设置它的依赖和属性。

## bean生命周期

1. Bean实例的创建
2. 设置实例的属性
3. 调用Bean初始化方法
4. 应用可以通过IOC容器使用Bean
5. 当容器关闭时，调用Bean销毁的方法

## FactoryBean

FactoryBean是一个类似于AbstractFactory，在获取Bean的时候如果发现是FactoryBean将调用getObject返回其生成的对象。

## Spring AOP

AOP 面向切面编程，生成代理类

## Spring Bean创建过程

# 分布式缓存设计

## 问题与解决

# MySQL

## 锁

粒度：服务器，表，行
* MyISAM：支持服务器和表级锁
* InnoDB：都支持。行级锁锁定的是索引，所以没有索引会直接锁表

表级锁与后续操作关系
| 关系    | A读  | A写  | B读  | B写  | B加读 | B加写 |
|---------|------|------|------|------|-------|-------|
| A加读表 | 可以 | 报错 | 可以 | 阻塞 | 可以  | 阻塞  |
| A加写表 | 可以 | 可以 | 阻塞 | 阻塞 | 阻塞  | 阻塞  |

行级锁与后续操作关系
|  关系     |       |       |       |       |       |
|  ---  |  ---  |  ---  |  ---  |  ---  |  ---  |
|  A加行共享锁     |       |       |       |       |       |
|  A加行派他锁     |       |       |       |       |       |

意象锁：简化加行级别锁后，再加表锁时的check

## 隔离级别

ACID:原子，一致性，隔离性，持久性

|                         | 脏读 | 不可重复读 | 幻读 |
|-------------------------|------|------------|------|
| read uncommit           | Y    | Y          | Y    |
| read commited           | N    | Y          | Y    |
| reaptable read(default) | N    | N          | Y    |
| serializable            | N    | N          | N    |


## 索引

# Tips

## `Error`和`Exception`

Error和Exception都是继承于Throwable，Error和RuntimeException及其子类称为未检查异常，其它异常成为受检查异常（Checked Exception）。

Error类一般是指与虚拟机相关的问题，如系统崩溃，虚拟机错误，内存空间不足，方法调用栈溢出等。对于这类错误的导致的应用程序中断，仅靠程序本身无法恢复和预防，遇到这样的错误，建议让程序终止。如`StackOverFlowError`,`outOfMemoryError`

Exception类表示程序可以处理的异常，可以捕获且可能恢复。遇到这类异常，应该尽可能处理异常，使程序恢复运行，而不应该随意终止异常

RuntimeException其特点是Java编译器不去检查它，处理RuntimeException的原则是：如果出现RuntimeException，那么一定是程序员的错误。如`IndexOutOfBoundsException`,`RuntimeException`

Checked Exception，继承自`Exception`，必须被显式地捕获或者传递。如`IOException`,`NoSuchFieldException`
<!--more-->

## 覆盖`equals()`

必须满足的约定：自反性，对称性，传递性，一致性，`x.equals(null)`必须为`false`

注意：

* 覆盖equals时总要覆盖hashCode：如果不这样做的话，就会违反Object.hashcode的通用约定，如果两个对象根据equals(Object)方法比较是相等的，那么调用这两个对象中任意一个对象的hashCode方法都必须产生同样的整数结果
* 不要将equals声明中的Object对象替换为其他的类型

## 重写（overloading）与重载（overwrite）

重写是在同一个类中定义的方法名相同但参数不同的一组函数；重载是子类中重写父类中方法，必须与父类中方法同名且参数一致，子类的访问性不能小于父类的定义

## final, finally, finalize

* final可修饰类（不能被继承），方法（不能被重载），变量（只能赋值一次）
* finally用于异常处理，表示这段语句最终一定会被执行（不管有没有抛出异常）
* finalize()是在java.lang.Object里定义的，也就是说每一个对象都有这么个方法。这个方法在gc启动，该对象被回收的时候被调用。

## Object类方法

* `getClass()`:final方法，获得运行时类型。
* `hashCode()`:返回该对象的哈希码值，若`a.equals(b)==true`则`a.hashCode()=b.hashCode()`，反之不一定
* `equals()`:比较对象是否相等
* `notify()`:用于唤醒正在等待当前对象监视器的线程，唤醒的线程是随机的
* `notifyAll()`:用于唤醒所有等待对象监视器锁的线程，notify只唤醒所有等待线程中的一个
* `wait()`,`wait(long timeout)`,`wait(long timeout, int nanos)`:一般和上面说的notify方法搭配使用。一个线程调用一个对象的wait方法后，线程将进入WAITING状态或者TIMED_WAITING状态。直到其他线程唤醒这个线程。
* `finalize`:垃圾回收器在回收一个无用的对象的时候，会调用对象的finalize方法，我们可以覆写对象的finalize方法来做一些清除工作
* `toString()`:默认class的名称+对象的哈希码。一般在子类中我们可以对这个方法进行覆写
* `clone()`:用于克隆一个对象，被克隆的对象需要implements Cloneable接口，否则调用这个对象的clone方法，将会抛出CloneNotSupportedException异常


# 微服务

## 什么是

微服务是一系列实现不同功能的服务。多个实现特定功能的服务组成一个系统，互相之间通过轻量级的传输协议协作完成功能任务。不同的微服务运行在自己的进程中，互相不干扰。

## 优点

* 高内聚，易于开发和维护
* 体积小，启动速度快
* 可以部分更新
* 一般采用DevOps方式实现自动化部署，简化部署的流程
* 每个微服务对运行的需求不同，可以按需分配资源
* 相对独立，采用接口的形式暴露数据，所以可以使用不同的技术开发

## 挑战

* 基础设施建设，增加开发成本
* 运维要求高
* 接口调整需要较高的开发成本
* 若使用不同技术开发，有重复工作
* 分布式系统的复杂性

## 设计原则

* 单一职责
* 轻量级，通用性通信协议
* 服务自治
* 接口明确

# 消息中间件

## 为什么使用

* 解耦：多应用间通过消息队列对同一消息进行处理，避免调用接口失败导致整个过程失败；
* 限流削峰:广泛应用于秒杀或抢购活动中，避免流量过大导致应用系统挂掉的情况；
* 异步：多应用对消息队列中同一消息进行处理，应用间并发处理消息，相比串行处理，减少处理时间；
* 消息驱动的系统：系统分为消息队列、消息生产者、消息消费者，生产者负责产生消息，消费者(可能有多个)负责对消息进行处理；

## rabbitMQ