---
title:  "openjdk8与maven surefire的一个bug"
date:   2018-11-12 15:00:00 +0800
categories: [bug-snapshot]
---

今天重装了下系统，因为之前Ubutu16升级到18采用的是自动升级，结果当然是坑爹的，动不动黑屏，一睡眠就再也醒不来。实在忍不了，再加上最近工作不是特别多，就决定重装18系统。没想到呀，重装也是坑。遇到了openJdk和maven的bug。泪目。

# 重现

* 安装 openjdk8

```sh
sudo apt install openjdk-8-jdk
```

* 安装 maven

```sh
mv ~/Downloads/apache-maven-3.6.0-bin.tar.gz /opt/
tar xzvf apache-maven-3.6.0-bin.tar.gz
export PATH=/opt/apache-maven-3.6.0/bin:$PATH
```

* 创建一个maven的默认项目，试一试刚刚安装的东西对不对

```sh
mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=my-app -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

* build以下项目，然后就挂了

```sh
mvn package
```

<!--more-->

# 结果

```sh
-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Error: Could not find or load main class org.apache.maven.surefire.booter.ForkedBooter
```

# 解决

面向搜索引擎的编程大法：

1. [stackoverflow相同问题](https://stackoverflow.com/questions/53010200/maven-surefire-could-not-find-forkedbooter-class?noredirect=1&lq=1)
2. [一个日本小哥的解决办法](https://qiita.com/watanabk/items/16e19e30659d0acca519)

就是说这是一个openjdk的bug，也是一个maven的bug，要解决就是升级以下。

方案一：指定具体的maven-surefire-plugin版本

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>2.22.1</version>
    <configuration>
        <useSystemClassLoader>false</useSystemClassLoader>
    </configuration>
</plugin>
```

方案二：使用oracle-jdk