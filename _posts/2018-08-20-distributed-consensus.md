---
title:  "分布式系统"
date:   2018-08-20 15:00:00 +0800
categories: [distributed]
---

# CAP Theorem
对于一个分布式系统，不能同时满足：一致性，可用性，分区容错性

# 弱一致性
* 最终一致性： DNS系统

# 强一致性
* 同步
* Paxos
* Raft (multi-paxos)
* ZAB (multi-paxos)
<!--more-->

# 强一致性算法：
强一致性算法两种类型：
1. 主从同步，在主中写，同步到所有从中（可用性低）
2. 多数派，写入1/2个节点，从1/2节点读（多线程环境下无法保证系统正确性，顺序很重要）

# Basic Paxos
1. 角色：
    * Client 系统外部角色，请求发起者
    * Propser 接受Client请求，向集群提出提议，冲突时，起到条件作用
    * Accpetor 投票者
    * Learner 提议接受者，不参与一致性解决，会接受记录结果
2. 流程：
    
    ![ Process1 ]({{ "/assets/2018-08-20-1.png" | absolute_url }})
3. 缺点：活锁，效率低（2轮RPC），难实现

# Mutil Paxos
1. 改进：1)Propser先竞争Leader，Leader作为唯一可以提出请求的节点 2)简化角色
2. 好处：减少了Propser请求的竞争，除了竞选只需要一次RPC
3. 流程：

    ![ Process2 ]({{ "/assets/2018-08-20-2.png" | absolute_url }})
    ![ Process3 ]({{ "/assets/2018-08-20-3.png" | absolute_url }})

# Raft
1. 将一致性问题划分成三个子问题：竞选(leader election)，log复制(log replication), 安全(safety)
2. [流程](http://thesecretlivesofdata.com/raft/)
3. [模拟](https://raft.github.io/)