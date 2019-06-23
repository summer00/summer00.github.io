---
title:  "Oracle错误码"
date:   2019-06-23 15:00:00 +0800
categories: [bug-snapshot]
---

最近才开始使用Oracle，踩了很多坑比较惨兮兮的，这里我来记录一下遇到的错误码，做一个总结吧。

# ORA-12505

```
ORA-12505, TNS:listener does not currently know of SID given in connect descriptor
```

项目部署到生产环境一启动就遇到了这个问题，查了百度上说的是`SIDs`和`SERVICE_NAME`不配对。然后就去找DBA解决，以为是人家提供的数据库配置不对。

结果问了DBA是JDBC链接串配的有问题。因为生产环境的Oracle做了高可用多节点的方式部署，所以JDBC连接串不能使用简单的IP形式，而是要使用TNS连接多节点的Oracle。下面是有4个ip和2个服务名的链接串模板。这个具体需要DBA提供，要把这一大串东西放到`jdbc:oracle:thin:@`后面。
```
(DESCRIPTION_LIST=(LOAD_BALANCE=ON)(FAILOVER=ON)
  (DESCRIPTION=(ENABLE=BROKEN)(CONNECT_TIMEOUT=3)(RETRY_COUNT=2)
      (ADDRESS_LIST=(LOAD_BALANCE=ON)(FAILOVER=ON)
      (ADDRESS=(PROTOCOL=TCP)(HOST=IP1)(PORT=1521))
      (ADDRESS=(PROTOCOL=TCP)(HOST=IP2)(PORT=1521))
      )
      (CONNECT_DATA=(SERVICE_NAME=SN1))
  )
  (DESCRIPTION=(ENABLE=BROKEN)(CONNECT_TIMEOUT=3)(RETRY_COUNT=2)
     (ADDRESS_LIST=(LOAD_BALANCE=ON)(FAILOVER=ON)
     (ADDRESS=(PROTOCOL=TCP)(HOST=IP3)(PORT=1521))
     (ADDRESS=(PROTOCOL=TCP)(HOST=IP4)(PORT=1521))
      )
      (CONNECT_DATA=(SERVICE_NAME=SN2))
  )
)
```
<!--more-->

## Oracle的SID和SERVICE_NAME

* SID：一个数据库可以有多个实例（如RAC集群环境），SID用来标识数据库内部每个实例的名字
* SERVICE_NAME：是对外的服务名，是服务器端使用的，一个库可以设置多个对外服务名，以实现不同的目的

# ORA-12514

```
ORA-12514: TNS:listener does not currently know of service requested in connect descriptor
```

换上TNS后重启应用，立马报了ORA-12514。这个就比较好解决了，就是连接串中的数据库服务找不到。这时就检查一下，数据库是不是可以使用其他工具访问，是不是挂了，如果没有就一定是连接串写错了。当时就是项目中配置少了一个`)`，把它加上后大功告成。

# ORA-00904

```
ORA-00904: "xx": invalid identifier
```

这个错很常见，就是报错中的`xx`写错了，可能是表名、列名、序列名。检查一下代码就可以了。