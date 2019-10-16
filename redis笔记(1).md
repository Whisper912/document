---
title: redis设计与实现-笔记(1)
date: 2019-10-15 21:24:18
tags:
- Redis
categories:
- Redis
---

发现了一个redis在线测试网站，可以在网页上在线测试redis命令
!(https://try.redis.io/)

---

### 第2章 简单动态字符串(Simple Dynamic String)

s没有使用C语言的传统字符串，自己构建了名为简单动态字符串的抽象类型，并将SDS用作Redis默认字符串。

在redis中，c字符串只用作字符串字面量，就是无须修改字符串的值的地方。

SDS还被用作缓冲区，AOF(Append of file)模块的AOF缓冲区，客户端状态的输入缓冲区(buffer)都是SDS实现的

SDS的数据结构如图
![](https://hedgehog-img.pek3b.qingstor.com/SDS_201910160001.png)

#### SDS和C字符串的区别
1. SDS读取字符串长度的时间复杂度只有O(1)，而C字符串要O(n)
2. 杜绝缓冲区溢出，SDS的API需要对SDS修改时先检查free的大小是否够用，不够就扩展后再执行操作
3. 内存重分配
    SDS通过free空间`实现空间预分配`和`惰性空间释放`两种优化策略
    * 空间预分配：当使用SDS API对SDS操作时，并且需要扩展时，程序不仅对SDS分配需要的内存，还会额外分配free空间， 
        * 当SDS修改后，len将会小于1M，程序分配与len相等的free空间，buffer实际长度为len+free+1
        * 当修改后，len大于1M，程序分配1M的free，buffer实际长度为len+1M+1
    * 惰性空间释放：用于优化SDS字符串缩短操作，当SDS的API执行缩短SDS字符串时，不释放空间，而是作为free保存起来’
4. 二进制安全
    二进制安全就是输入任何字节都能正确处理，包括`\0`。
    因为C字符串使用`\0`标识字符串结尾，所以字符串中不能包含`\0`，但SDS使用buffer存字符串，使用len来判断字符串结束，所以可以存储任何二进制数据。
5. 兼容部分C字符串函数
    SDS在buffer最后一位仍然以`\0`结尾，是为了让保存文本数据的SDS可以重用一部分<string.h>库的函数。

下图是以上5点的表格总结：
![](https://hedgehog-img.pek3b.qingstor.com/difference_between_C_String_and_SDS_201910162242.png)

#### SDS API
如下图所示：
![](https://hedgehog-img.pek3b.qingstor.com/SDS_API_1_201910162333.png)
![](https://hedgehog-img.pek3b.qingstor.com/SDS_API_2_201910162336.png)

---

### 第三章 链表

