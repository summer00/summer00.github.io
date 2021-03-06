---
title: "Java多线程 概览"
date: 2018-08-23 13:00:00 +0800
categories: [summary, thread, iv]
---

# 可用性问题

## 死锁

- 一组互相竞争资源的线程因互相等待，导致“永久”阻塞的现象。
- 避免：
  - 破坏占用且等待条件，一次性申请所有的资源
  - 破坏不可抢占条件，申请不到下一资源，主动释放已申请到的资源
  - 破坏循环等待条件，按一定顺序申请资源
- 解决：导出现场，分析，重启服务

## 饥饿

- 高优先级线程侵占底优先级线程执行的 CPU 时间片
- 线程被同步块阻塞
- 等待的线程永远不会唤醒

## 活锁

- 任务或者执行者没有被阻塞，由于某些条件没有满足，导致一直重复尝试，失败，尝试，失败

<!--more-->

# 线程安全问题

出现的条件：1)多线程 2)共享资源 3)非原子性操作

## synchronized 原理

- 可重入、互斥锁
- 三种使用方式：1）修饰方法，锁定当前对象 2）修饰静态方法，锁定的当前类的 Class 实例 3）修饰代码块，锁定指定的对象
- synchronized 用的锁是存在 Java 对象头里的（mark word）
- 实现：JVM 基于进入和退出 Monitor 对象来实现方法同步和代码块同步。代码块同步是使用 monitorenter 和 monitorexit 指令实现的，monitorenter 指令是在编译后插入到同步代码块的开始位置，而 monitorexit 是插入到方法结束处和异常处。任何对象都有一个 monitor 与之关联，当且一个 monitor 被持有后，它将处于锁定状态。根据虚拟机规范的要求，在执行 monitorenter 指令时，首先要去尝试获取对象的锁，如果这个对象没被锁定，或者当前线程已经拥有了那个对象的锁，把锁的计数器加 1；相应地，在执行 monitorexit 指令时会将锁计数器减 1，当计数器被减到 0 时，锁就释放了。如果获取对象锁失败了，那当前线程就要阻塞等待，直到对象锁被另一个线程释放为止

### 偏向锁

- 解决问题：减少无实际竞争情况下，使用重量级锁产生的性能消耗
- 实现：一个线程第一次来访问互斥资源，则在对象头和栈帧的锁记录中存储偏向锁的线程 ID(可以理解为获取“锁”的动作)。偏向锁在获取锁之后，直到有竞争出现才会释放锁。第二个线程尝试获得偏向锁，若成功则第二个线程获得该偏向锁，否则膨胀成轻量级锁

### 轻量级锁

- 解决问题：自旋锁的目标是降低线程切换的成本
- 实现：使用轻量级锁时，不需要申请互斥量，仅仅*将 Mark Word 中的部分字节 CAS 更新指向线程栈中的 Lock Record，如果更新成功，则轻量级锁获取成功*，记录锁状态为轻量级锁；否则，说明已经有线程获得了轻量级锁，目前发生了锁竞争（不适合继续使用轻量级锁），接下来膨胀为重量级锁

## volatile

1. 语意：
   1. 保证可见性。在变量改变需要依赖当前值，或者需要与其他变量共同参与不变性约束时需要额外同步来保证原子性
   2. 禁止指令重排优化。通过在编译时加入一个 lock 前缀指令，相当于内存屏障
2. 当且仅当完全满足以下条件时，才可以使用 volatile:
   1. 写入不依赖当前值，或者保证只有单线程修改这个值
   2. 该变量的值不会与其他状态变量一起纳入不变性条件
   3. 在访问变量时不需要加锁

# J.U.C

## 并发容器

1. `ConcurrentHashMap`
   1. 1.8 之前通过分段锁保护减小锁粒度提高并发
   2. 1.8 改为`synchronized`内部`node`方式保护，进一步提高并发
   3. 不能放`null`的`key`和`value`原因是避免多线程环境下模糊的语义。如`get`的结果为`null`无法判断是真的没有值，还是值是空。在单线程环境可以通过`containKey`判断，但多线程情况下该操作并不是原子性的
2. `CopyOnWriteArrayList`
   1. 读写分离的思想，写的时候复制一份内部数组，不修改之前的对象，此时发布出去的数组对象可以看为是不可变的
   2. 1.8 之前用`ReentrantLock`加锁保护，复制一份内部数组
   3. 1.8 改为写入时加`synchronized`内部`object`锁保护，复制一份内部数组
3. `Queue`
   1. 非阻塞队列
   2. 阻塞队列

## Atomic

提供原子性的操作，常用` AtomicBoolean``AtomicInteger``AtomicReference `

## 同步工具

1. `CountDownLatch`
2. `CyclicBarrier`
3. `Semaphore`

## 锁

1. `ReentrantLock`
   1. `Condition`条件判断
   2. 内部类`NonfairSync`和`FairSync`提供是否公平的锁支持
2. `ReentrantReadWriteLock`
3.

## AQS

AQS 维护了一个`volatile int state`（代表共享资源）和一个 FIFO 现场等待队列（多线程争用资源被阻塞时会进入此队列）。AQS 定义了两种资源共享方式 Exclusive（独占）、Share（共享）。

# 实用

## 停止线程

1. 线程停止：根据同步状态判断是否应该停止现场
2. 线程池停止：可以使用`ThreadPool.shutdown()`拒绝接受新的任务，执行完已有任务后停止；或使用`TreadPool.shutdownNow()`拒绝接受新任务，终止正在执行的任务，丢弃队列中的任务。

## 并发设计模式

1. **Thread Pre Message**：为每个任务创建一个线程，由于 java 的线程创建和操作系统一一对应，是很重量级的对象，所以在 java 中使用这种模式并不常见。可以使用线程池的方式优化此模式，或者使用轻量级的协程
2. **Worker Tread**：线程池加阻塞队列
3. **Immutability**:状态不变对象天然线程安全

## 线程池创建的注意点：

1. 有界队列接收任务
2. 明确的拒绝策略
3. 设置名字
4. 避免线程死锁 -- 线程池中的任务最好不要有依赖

# 参考

1. https://juejin.im/post/5a5c09d051882573282164ae
2. https://kaimingwan.com/post/java/javanei-zhi-suo-kai-xiao-you-hua-pian-xiang-suo-qing-liang-ji-suo#toc_17
