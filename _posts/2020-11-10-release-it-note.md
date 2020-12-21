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

**Remember this**

1. _Beware this necessary evil_
2. _Prepare for the many forms of failure_
3. _Know when to open up abstractions_: Debugging integration point failures usually requires peeling back a layer of abstraction. Failures are often difficult to debug at the application layer because most of them violate the high-level protocols. Packet sniffers and other network diagnostics can help
4. _Failures propagate quickly_
5. _Apply patterns to avert integrate point problems_

## Chain Reactions

The dominant architectural style today is the horizontally scaled farm of commodity hardware. A chain reaction occurs when an application has some defect -- usually a resource leak or a load-related crash.

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

# Sentence

1. A postmortem is like a murder mystery. You have a set of clues. Some are reliable, such as server log copied from the time of the outage. Some are unreliable, such as statements from people about what they saw. The postmortem can actually be harder to solve than a murder, because the body goes away.
2. In other words, once you know where to look, it's simple to make a test that finds it.
3. A stand-alone system that doesn't integrate with anything else is rare, not to mention being almost useless.

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
