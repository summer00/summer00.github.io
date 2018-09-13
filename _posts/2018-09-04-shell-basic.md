---
title:  "Shell学习笔记"
date:   2018-09-04 15:00:00 +0800
categories: [note]
---

# Day 1

## jobs 任务

```shell
sleep 100   # 前台应用，直接输入命令
sleep 100 & # 后台任务，在命令后加`&`
jobs        # 列出当前后台任务
fg 1        # 将任务1放回前台执行
bg 1        # 将任务1放到后台继续执行，交互式的不会继续运行
kill %1     # 杀掉任务1
```

## pipeline 管道

```shell
touch a{0..10}  # 创建文件
ls a? | wc -l   # 无名管道，输出以a开头文件的行数

mkfifo /temp/f  # 创建有名管道
ls a? > f       # 输入到管道
wc -l /temp/f   # 使用管道
```

<!--more-->

## 内部命令

```shell
#!/bin/bash # 设置执行器

type who        # 查看命令的类型
help            # 查看帮助
alias g='git'   # 别名
echo '123'      # 打印
pwd             # 打印当前路径
cd              # 改变路径
：              # 空操作，永真
```

# 常用外部命令

```shell
head -n 1 test.sh           # 查看test.sh第一行
tail -n 1 test.sh           # 查看test.sh最后一行
echo a:b:c:d | cut -d: f1   # 剪切
basename /test/test.sh      # strip directory and suffix from filenames
mkdir -p a/b/c              # 创建目录
rm -rf a                    # 强制删除
sort -rn nums.test          # 排序 -n 按数字 -r 逆序
ls a && echo find || echo no find #连接符号
```

## 变量

```shell
name=xx         # 初始化
echo $name      # 输出xx
unset name      # 删除name里的值
echo $name      # 输出空行
name=xx
name='$name xx' # 单引号，不使用name中存储的值
echo $name      # $name xx
name=xx
name="$name xx" # 双引号，使用name中的值
echo $name      # xx xx
```