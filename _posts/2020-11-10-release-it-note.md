---
title: "《发布！》读书笔记"
date: 2020-11-10 12:00:00 +0800
categories: [book-note]
---

# Living in production

Software design as taught today is terribly incomplete. It only talks about what systems should do. It doesn't address the converse - what systems should not do.

Most software is designed for the development lab or the testers in the QA
department. Testing - even agile, pragmatic, automated testing - is not enough to prove that software is ready for the real world.

## The scope of the challenge

1. large active user counts
2. Uptime demands have increased
3. to build software fast that's cheap to build, good for users, and cheap to operate

# Stabilize your system

A robust system keeps processing transactions, even when transient impulses, persistent stresses, or component failures disrupt normal processing.

## Extending your life span

Testing makes problems visible so you can fix them. Following Murphy's Law, whatever you do not test against will happen.

The only way you catch bugs before they bite you in production is to run your own longevity tests.

If all else fails, production becomes your longevity testing environment by default. You'll definitely find the bugs there, but it's not a recipe for a happy life style.

## Failure modes

The original trigger and the way the crack spreads to the rest of the system, together with the result of the damage, are collectively called a failure mode. No matter what, your system will have a variety of failure modes. Denying
the inevitability of failures robs you of your power to control and contain
them.

## Stopping crack propagation

1. not blocking forever when all connections are checked out
2. using timeout in remote calls
3. multiple server groups
4. the more tightly coupled the architecture, the greater the chance this coding error can propagate

# Stability anti-patterns

## Integration Points

Every single one of those feeds presents a stability risk. Every socket, process, pipe, or remote procedure call can and will hang.

**socket-based protocols**: Network failures can hit you in two ways: fast(connection refused) or slow(dropped ACK).The ways that such an integration point can harm the caller:

1. The provider may accept the connection but never response to the HTTP request.
2. The provider may accept the connection but not read the request.
3. The provider may send back a response with the status the caller doesn't know how to handle.
4. The provider may send back a response with a content type the caller doesn’t expect or know how to handle.
5. The provider may claim to be sending JSON but actually sending plain text.

**Patterns - Make Integrate Point Safer**:

1. Circuit Breaker
2. Decoupling Middleware
3. Timeouts
4. Handshaking
5. Test Harnesses

### Remember This

1. **Beware this necessary evil**
2. **Prepare for the many forms of failure**
3. **Know when to open up abstractions**: Debugging integration point failures usually requires peeling back a layer of abstraction. Failures are often difficult to debug at the application layer because most of them violate the high-level protocols. Packet sniffers and other network diagnostics can help
4. **Failures propagate quickly**
5. **Apply patterns to avert integrate point problems**

## Chain Reactions

The dominant architectural style today is the horizontally scaled farm of commodity hardware. A chain reaction occurs when an application has some defect -- usually a resource leak or a load-related crash.

### Remember This

- **Recognize that one server down jeopardizes the rest.**
- **Hunt for resource leaks.** Most of time, a chain reaction happens when your application has a memory leak.
- **Hunt for obscure time bugs.**
- **Use Autoscaling.**
- **Defend with Bulkheads.**

## Cascading Failures

Cascading failures often result from resource pools that get drained because of a failure in a lower layer. Integration points without timeouts are a surefire way to create cascading failures.

### Remember This

- **Stop cracks from jumping the gap.** Your system surely calls out to other enterprise systems; make sure you can stay up when they go down.
- Scrutinize resource pools. Safe resource pool always limit the time a thread can wait to check out a resource.
- **Defend with Timeouts and Circuit Breaker.** Circuit Breaker protects your system by avoiding calls out to the troubled integration point. Using Timeouts ensures that you can come back from a call out to the troubled point.

## Users

Users are a terrible thing. System would be much better off with no users. Human users have a gift for doing exactly the worst possible thing at the worst possible time.

### Traffic

"Capacity" is the maximum throughput your system can sustain under a given workload while maintaining acceptable performance. When a transaction takes too long to execute, it means that the demand on your system exceeds its capacity.

### Heap Memory

One such hard limit is memory available, particularly in interpreted or man-aged code languages. When memory gets short, a large number of surprising things can happen.

We always put session in memory. Every additional user
means more memory. Weak references are a useful way to respond to changing memory conditions, but they do add complexity. When you can, it's best to just keep things out of the memory.

### Off-Head Memory, Off-Host Memory

Another effective way to deal with per-user memory is to farm it out to a different process. Instead of keeping it inside the heap—that is, inside the address space of your server’s process—move it out to some other process, like Memcached, Redis.

### Sockets & Closed Sockets

Your application will probably need some
changes to listen on multiple IP addresses and handle connections across them all without starving any of the listen queues. A million connections also need a lot of kernel buffers. Plan to spend some time learning about your operating system’s TCP tuning parameters.

Not only can open sockets be a problem, but the ones you’ve already closed can bite you too. After your application code closes a socket, the TCP stack moves it through a couple of terminal states. One of them is the TIME_WAIT state. TIME_WAIT is a delay period before the socket can be reused for a new connection. It’s there as part of TCP’s defense against bogons.

### Expensive to Serve

There is no effective defense against expensive users. They are not a direct stability risk, but the increased stress they produce increases the likelihood of triggering cracks elsewhere in the system. The best thing you can do about expensive users is test aggressively. Identify whatever your most expensive transactions are and double or triple the proportion of those transactions.

### Remember This

- **Users consume memory.** Each user’s session requires some memory. Minimize that memory to improve your capacity. Use a session only for caching so you can purge the session’s contents if memory gets tight.
- **Users do weird, radom things.**
- **Malicious users are out there.** Become intimate with your network design; it should help avert attacks.
- **Users will gang up on you.** Run special stress tests to hammer deep links or hot URLs.

## Blocked Threads

### Remember This

- **Recall that the Blocked Threads anti-pattern is the proximate cause of failures.** The Blocked Threads anti-pattern leads to Chain Reactions and Cascading Failures anti-patterns.
- **Scrutinized resource pools.**
- **Use power primitives.** Any library of concurrency utilities has more testing than your newborn queue.
- **Defend with Timeout.**
- **Beware the code you cannot see.**

## Self-Denial Attacks

The classic example of a self-denial attack is the email from marketing to a “select group of users” that contains some privileged information or offer. These things replicate faster than the Anna Kournikova Trojan (or the Morris worm, if you’re really old school). Any special offer meant for a group of 10,000 users is guaranteed to attract millions. The community of networked bargain hunters can detect and share a reusable coupon code in milliseconds.

Not every self-inflicted wound can be blamed on the marketing department (although we sure can try). In a horizontal layer that has some shared resources, it’s possible for a single rogue server to damage all the others.

You can avoid machine-induced self-denial by building a “shared-nothing” architecture.

### Remember This

- **Keep the lines of communication open.** Self-denial attacks originate inside your own organization, when people cause self-inflicted wounds by creating their own flash mobs and traffic spikes. You can aid and abet these marketing efforts and protect your system at the same time, but only if you know what’s coming.
- **Protect shared resources.** Programming errors, unexpected scaling effects, and shared resources all create risks when traffic surges.
- **Expect rapid redistribution of any cool or valuable offer.**

## Scaling Effects

We run into such scaling effects all the time. Anytime you have a “many-to-one” or “many-to-few” relationship, you can be hit by scaling effects when one side increases.

### Point-to-Point Communications

Point-to-point communication between machines probably works just fine when only one or two instances are communicating. With point-to-point connections, each instance has to talk directly to every other instance. Scale that up to a hundred instances, and the O(n^2) scaling becomes quite painful.

Depending on your infrastructure, you can replace point-to-point communication with the following:

- UDP broadcasts
- TCP or UDP multicast
- Publish/subscribe messaging
- Message queues

### Shared Resources

Commonly seen in the guise of a service-oriented architecture or “common services” project, the shared resource is some facility that all members of a horizontally scalable layer need to use. When the shared resource gets overloaded, it’ll become a bottleneck limiting capacity.

The most scalable architecture is the shared-nothing architecture. Each server operates independently, without need for coordination or calls to any centralized services. In a shared nothing architecture, capacity scales more or less linearly with the number of servers.

### Remember This

- **Examine production versus QA environments to spot Scaling Effects.** Patterns that work fine in small environments or one-to-one environments might slow down or fail completely when you move to production sizes.
- **Watch out for point-to-point communication.** Once you’re dealing with tens of servers, you will probably need to replace it with some kind of one-to-many communication.
- **Watch out for shared resources.** If your system must use some sort of shared resource, stress-test it heavily. Also, be sure its clients will keep working if the shared resource gets slow or locks up.

## Unbalanced Capacities

If you can’t build every service large enough to meet the potentially over- whelming demand from the front end, then you must build both callers and providers to be resilient in the face of a tsunami of requests. For the caller, Circuit Breaker will help by relieving the pressure on downstream services when responses get slow or connections get refused. For service providers, use Handshaking and Backpressure to inform callers to throttle back on the requests. Also consider Bulkheads to reserve capacity for high-priority callers of critical services.

### Drive Out Through Testing

- Use capacity modeling to make sure you're at least in the ballpark.
- Don't just test your system with your usual workloads.
- If you can, use autoscaling to react to surging demand.

### Remember This

- Examine server and thread counts. Check the ratio of front-end to back-end servers, along with the number of threads each side can handle in production compared to QA.
- Observe near scaling effects and users.
- Virtualize QA and scale it up. Even if your production environment is a fixed size, don’t let your QA languish at a measly pair of servers. Scale it up. Try test cases where you scale the caller and provider to different ratios. You should be able to automate this all through your data center automation tools.
- Stress both sides of the interface. If you provide the back-end system, see what happens if it suddenly gets ten times the highest-ever demand, hitting the most expensive transaction.If you provide the front-end system, see what happens if calls to the back end stop responding or get very slow.

## Dogpile

“Dogpile” is a term from American football in which the ball-carrier gets compressed at the base of a giant pyramid of steroid-infused flesh.

A dogpile can occur in several different situations:

- When booting up several servers, such as after a code upgrade and restart
- When a cron job triggers at midnight (or on the hour for any hour, really)
- When the configuration management system pushes out a change
- Dogpiles can also occur when some external phenomenon causes a synchronized “pulse” of traffic.

### Remember This

- Dog-piles force you to spend too much to handle peak demand.
- Use random clock slew to diffuse the demand.
- Use increasing back-off times to avoid pulsing. Use a back-off algorithm so different callers will be at different points in their back-off periods.

## Force Multiplier

# Words

1. dominant
2. failover
3. presence
4. deduce
5. portion
6. wary
7. peculiar
8. ounce
9. leverage
10. impulse
11. stubbing
12. jettison
13. terminology
14. propagating
15. continuously
16. boundaries
17. chaotic
18. scenario
19. hammered
20. ramp
21. paranoid
22. probe
23. torn
24. lingua france
25. plugged
26. propagate
27. accelerating
28. bulkhead
29. resilience
30. capacity
31. midstream
32. machinery
33. malicious
34. descendant
35. invalidation
36. beware
37. bottleneck
38. miserably
39. distinguish
40. fan-in
41. relieving
42. drive out
43. outage

# Sentence

1. A postmortem is like a murder mystery. You have a set of clues. Some are reliable, such as server log copied from the time of the outage. Some are unreliable, such as statements from people about what they saw. The postmortem can actually be harder to solve than a murder, because the body goes away.
2. In other words, once you know where to look, it's simple to make a test that finds it.
3. A stand-alone system that doesn't integrate with anything else is rare, not to mention being almost useless.
4. Just like we draw trees upside-down with their roots pointing to the sky, our problems cascade upward through the layers.
5. In other words, the garbage collector will take advantage of all the help you give it before it gives up.
6. Users in the real world do things that you won't predict or sometimes understand.
7. Every cache should have an invalidation strategy to remove items from cache when its source data changes.
8. As the site scales horizontally, the lock manager becomes a bottleneck then finally a risk.

第 1 章 生产环境的生存法则 1
1.1 瞄准正确的目标 1
1.2 应对不断扩大的挑战范围 2
1.3 多花 5 万美元来节省 100 万美元 3
1.4 让“原力”与决策同在 4
1.5 设计务实的架构 4
1.6 小结 5
第 一部分 创造稳定性 7
第 2 章 案例研究：让航空公司停飞的代码异常 8
2.1 进行变 9
2.2 遭遇停机 10
2.3 严重后果 12
2.4 事后分析 12
2.5 寻找线索 13
2.6 证据确凿 16
2.7 预防管用吗 18
第 3 章 让系统稳定运行 19
3.1 定义稳定性 20
3.2 延长系统寿命 20
3.3 系统失效方式 21
3.4 阻止裂纹蔓延 22
3.5 系统失效链 23
3.6 小结 24
第 4 章 稳定性的反模式 25
4.1 集成点 26
4.1.1 套接字协议 29
4.1.2 凌晨 5 点的紧急电话 31
4.1.3 HTTP 协议 35
4.1.4 供应商的 API 程序库 36
4.1.5 应对集成点的问题 37
4.1.6 要点回顾 37
4.2 同层连累反应 38
4.3 层叠失效 41
4.4 用户 42
4.4.1 网络流量 42
4.4.2 难伺候的用户 46
4.4.3 不受欢迎的用户 47
4.4.4 恶意用户 50
4.4.5 要点回顾 51
4.5 线程阻塞 51
4.5.1 发现阻塞 53
4.5.2 程序库 55
4.5.3 要点回顾 56
4.6 自黑式攻击 57
4.6.1 避免自黑式攻击 57
4.6.2 要点回顾 58
4.7 放大效应 58
4.7.1 点对点通信 59
4.7.2 共享资源 60
4.7.3 要点回顾 61
4.8 失衡的系统容量 62
4.8.1 通过测试发现系统容量失衡 63
4.8.2 要点回顾 63
4.9 一窝蜂 64
4.10 做出误判的机器 66
4.10.1 被放大的停机事故 66
4.10.2 控制和防护措施 69
4.10.3 要点回顾 69
4.11 缓慢的响应 70
4.12 无限长的结果集 71
4.12.1 黑色星期一 71
4.12.2 要点回顾 73
4.13 小结 74
第 5 章 稳定性的模式 75
5.1 超时 75
5.2 断路器 78
5.3 舱壁 80
5.4 稳态 83
5.4.1 数据清除 84
5.4.2 日志文件 85
5.4.3 内存中的缓存 86
5.4.4 要点回顾 86
5.5 快速失败 87
5.6 任其崩溃并替换 89
5.6.1 有限的粒度 89
5.6.2 快速替换 90
5.6.3 监管 90
5.6.4 重新归队 91
5.6.5 要点回顾 91
5.7 握手 91
5.8 考验机 93
5.9 中间件解耦 96
5.10 卸下负载 98
5.11 背压机制 99
5.12 调速器 101
5.13 小结 102
第二部分 为生产环境而设计 103
第 6 章 案例研究：屋漏偏逢连夜雨 104
6.1 宝宝的第 一个感恩节 105
6.2 把脉 106
6.3 感恩节 106
6.4 黑色星期五 107
6.5 生命体征 108
6.6 进行诊断 109
6.7 求助专家 110
6.8 如何应对 111
6.9 应对奏效吗 111
6.10 尾声 112
第 7 章 基础层 114
7.1 数据中心和云端的联网 115
7.1.1 网卡和名字 116
7.1.2 多网络编程 118
7.2 物理主机、虚拟机和容器 119
7.2.1 物理主机 119
7.2.2 数据中心的虚拟机 119
7.2.3 数据中心的容器 120
7.2.4 云上的虚拟机 123
7.2.5 云上的容器 125
7.3 小结 125
第 8 章 实例层 126
8.1 代码 128
8.1.1 构建代码 128
8.1.2 不可变、易处理的基础设施 129
8.2 配置 130
8.2.1 配置文件 131
8.2.2 易处理基础设施的配置 131
8.3 明晰性 132
8.3.1 明晰性设计 133
8.3.2 提升明晰性的实现技术 134
8.3.3 记录日志 134
8.3.4 实例的健康度量指标 137
8.3.5 健康状况检查 138
8.4 小结 138
第 9 章 互连层 139
9.1 不同规模的解决方案 139
9.2 使用 DNS 140
9.2.1 基于 DNS 的服务发现 140
9.2.2 基于 DNS 的负载均衡 141
9.2.3 基于 DNS 的 GSLB 142
9.2.4 DNS 的可用性 144
9.2.5 要点回顾 144
9.3 负载均衡 144
9.3.1 软件负载均衡 145
9.3.2 硬件负载均衡 146
9.3.3 健康状况检查 147
9.3.4 会话黏性 147
9.3.5 按请求类型分隔流量 148
9.3.6 要点回顾 148
9.4 控制请求数量 148
9.4.1 系统为何会失效 149
9.4.2 防止灾难 150
9.4.3 要点回顾 151
9.5 网络路由 151
9.6 发现服务 153
9.7 迁移虚拟 IP 地址 154
9.8 小结 155
第 10 章 控制层 156
10.1 适合的控制层工具 156
10.2 机械效益 157
10.2.1 属于系统失效，而非人为错误 158
10.2.2 运行得太快也有问题 158
10.3 平台和生态系统 159
10.4 开发环境就是生产环境 160
10.5 整个系统的明晰性 161
10.5.1 真实用户监控 162
10.5.2 经济价值高于技术价值 162
10.5.3 碎片化的风险 164
10.5.4 日志和统计信息 164
10.5.5 要监控什么 165
10.6 配置服务 166
10.7 环境整备和部署服务 167
10.8 命令与控制 169
10.8.1 要控制什么 169
10.8.2 发送命令 170
10.8.3 可编写脚本的界面 170
10.8.4 要点回顾 171
10.9 平台厂商 171
10.10 工具清单 172
10.11 小结 172
第 11 章 安全性 173
11.1 OWASP 十大安全漏洞 173
11.1.1 注入 174
11.1.2 失效的身份验证和会话管理 175
11.1.3 跨站脚本攻击 178
11.1.4 失效的访问控制 179
11.1.5 安全配置出现失误 181
11.1.6 敏感数据泄露 182
11.1.7 防范攻击不足 183
11.1.8 CSRF 183
11.1.9 使用含有已知漏洞的组件 184
11.1.10 API 保护不足 185
11.2 小特权原则 186
11.3 密码的配置 187
11.4 安全即持续的过程 187
11.5 小结 188
第三部分 将系统交付 189
第 12 章 案例研究：等待戈多 190
第 13 章 为部署而设计 193
13.1 机器与服务 193
13.2 计划停机时间的谬误 193
13.3 自动化部署 194
13.4 持续部署 197
13.5 部署中的各个阶段 198
13.5.1 关系数据库模式 200
13.5.2 无模式数据库 202
13.5.3 Web 资源 205
13.5.4 推出新代码 206
13.5.5 清理 208
13.6 像行家一样部署 209
13.7 小结 210
第 14 章 处理版本问题 211
14.1 帮助他人处理版本问题 211
14.1.1 不会破坏 API 的变 211
14.1.2 破坏 API 的变 215
14.2 处理其他系统的版本问题 217
14.3 小结 219
第四部分 解决系统性问题 221
第 15 章 案例研究：不能承受的巨大顾客流量 222
15.1 后推出新系统 222
15.2 以 QA 测试为目标 223
15.3 负载测试 225
15.4 被众多因素所害 227
15.5 测试仍然有差距 229
15.6 善后 229
第 16 章 适应性 232
16.1 努力与回报的关系 232
16.2 过程和组织 233
16.2.1 平台团队 235
16.2.2 愉快地发布 236
16.2.3 演化 重要的部分是灭 237
16.2.4 在团队级别实现自治 239
16.2.5 谨防高效率 240
16.2.6 过程和组织小结 242
16.3 系统架构 242
16.3.1 演进式架构 242
16.3.2 松散的集群 245
16.3.3 显式上下文 246
16.3.4 创造 多选项 247
16.3.5 系统架构小结 252
16.4 信息架构 252
16.4.1 消息、事件和命令 253
16.4.2 让服务自己控制其资源的标识符 254
16.4.3 URL 的两重性 256
16.4.4 拥抱多义性 259
16.4.5 避免概念泄露 260
16.4.6 信息架构小结 261
16.5 小结 261
第 17 章 混沌工程 262
17.1 不可能构建第二个 Facebook 去做测试 262
17.2 混沌工程的先驱 263
17.3 猴子军团 264
17.4 使用自己的混沌猴 265
17.4.1 先决条件 266
17.4.2 设计实验 266
17.4.3 3 种混沌注入 267
17.4.4 有针对性地注入混沌 268
17.4.5 自动和重复 269
17.5 从人的方面模拟灾难 270
17.6 小结 271
