---
title: redis笔记(1)
date: 2019-10-15 21:24:18
tags:
- Redis
categories:
- Redis
---

### 第2章 简单动态字符串(Simple Dynamic String)

s没有使用C语言的传统字符串，自己构建了名为简单动态字符串的抽象类型，并将SDS用作Redis默认字符串。

在redis中，c字符串只用作字符串字面量，就是无须修改字符串的值的地方。

SDS还被用作缓冲区，AOF(Append of file)模块的AOF缓冲区，客户端状态的输入缓冲区(buffer)都是SDS实现的

SDS的数据结构如图
![](https://hedgehog-img.pek3b.qingstor.com/SDS_201910160001.png)

* SDS和C字符串的区别
    1. SDS读取字符串长度的时间复杂度只有O(1)，而C字符串要O(n)
    2. 杜绝缓冲区溢出，SDS的API需要对SDS修改时先检查free的大小是否够用，不够就扩展后再执行操作

* SDS通过free空间实现空间欲分配和惰性空间释放两种优化策略