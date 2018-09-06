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

## 结构

```shell

```
