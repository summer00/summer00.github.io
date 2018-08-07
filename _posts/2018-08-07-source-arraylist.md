---
title:  "List 源码分析"
date:   2018-08-07 17:17:00 +0800
categories: [source]
---

`List`是常用的数据类型，用来存储一组有序的数据，常用的实现有`ArrayList`和`LinkedList`。

# ArrayList实现

`ArrayList`内部通过数组实现，


```java
    transient Object[] elementData; // non-private to simplify nested class access
    private int size;
```
<!--more-->