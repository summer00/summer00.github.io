---
title:  "Linux网络基础"
date:   2019-08-04 15:00:00 +0800
categories: [linux]
---

# 网络模型

| OSI    | TCP/IP | Linux                  | tcp协议   |
| ------ | ------ | ---------------------- | --------- |
| 应用层 | 应用层 | 应用程序               | HTTP,FTP  |
| 表示层 | -      | -                      | -         |
| 会话层 | -      | -                      | -         |
| 传输层 | 传输层 | 操作系统               | TCP, UDP  |
| 网络层 | 网络层 | 操作系统               | IP        |
| 链路层 | 链路层 | 设备驱动程序与网络接口 | ARP, RARP |
| 物理层 | -      | 设备驱动程序与网络接口 | -         |

# 网络层

## IP地址

## IP封包

每一层协议在下一层的基础上加入自己的首部信息，组成自己的数据报文
* 以太网首部 | IP首部 | TCP/UDP首部 | 应用数据 | 以太网尾部

![data-package ]({{ "/assets/data-package.jpg" | absolute_url }})

<!--more-->

## 常用网络设备

集线器 --> 交换机 --> 网桥 --> 路由器 --> 网关

# TCP协议

## 三次握手

## 四次挥手

# 应用层

## 协议访问的过程
* 输入网址
* DNS解析，查找服务器的IP
* 客户端发送TCP链接
  * 使用ARP查找本地交换机，是否存在此IP；若有则发送，没有则转发网关
  * 经过层层网关，到达IP所在交换机
  * 根据ARP查找IP对应的MAC地址
  * 数据转发到相应的端口

## URL

```
    https://www.baidu.com/index.js?wd=hello#ch
    协议   | 域名         |文件     |查询    |定位
```

# Linux网络命令

## Linux网络接口

* 命名规律：
  * eth0为第一个接口，eth1为第二个
  * lo为本地回接口，127.0.0.1
  * enp0s3：en代表以太网卡，p0s3代表PCI接口的物理位置（0,3）

## 常用命令

### ifconfig

ifconfig是linux中用于显示或配置网络设备（网络接口卡）的命令

### ping，四步走

* ping localhost --> 检测网卡安装或者配置
* ping 网关 --> 局域网网关或路由是否正常
* ping dns server --> dns是否可以解析
* ping 远程地址 --> 与外界连接是否正常

### mtr网络检测工具
```shell
mtr hostname
```

### traceroute/tracepath
traceroute (Windows 系统下是tracert) 命令利用ICMP 协议定位您的计算机和目标计算机之间的所有路由器。TTL 值可以反映数据包经过的路由器或网关的数量，通过操纵独立ICMP 呼叫报文的TTL 值和观察该报文被抛弃的返回信息，traceroute命令能够遍历到数据包传输路径上的所有路由器。

tracepath指令可以追踪数据到达目标主机的路由信息，同时还能够发现MTU值。它跟踪路径到目的地，沿着这条路径发现MTU。它使用UDP端口或一些随机端口。它类似于Traceroute，只是不需要超级用户特权，并且没有花哨的选项。

### curl/wget

```sh
curl hostname # http客户端
wget hostname # 下载
```

### netstat

```sh
netstat -a # 列出所有端口
netstat at| -au # 列出所有tcp/udp
netstat -p # 显示进程和端口
netstat -r # 列出路由信息
netstat -tnl # 列出所有监听端口
```

