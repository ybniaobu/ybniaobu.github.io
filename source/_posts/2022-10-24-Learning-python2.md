---
title: ':2022-:10-:24-:Learning python2'
date: 2022-10-24 13:30:33
categories: 
  - [python, python入门.读书笔记]
tags:
  - python
  - 读书笔记
cover: https://s2.loli.net/2022/09/17/JxdLBRD5Cjfvg37.jpg
---

> 该笔记为 **《Learning Python》** 的读书笔记，未经过系统的整理；  
> 该书涉及的内容可能过于啰嗦，但包含一些python背后的逻辑和机制，值得初学者观看；  
> 该笔记内容过多，所以不展示部分代码的结果，需复制到编辑器中查看；
> 学习完成日期为2022年10月20日。

![learning_python.jpg](https://s2.loli.net/2022/09/17/ri9Ue6nguJdq1Ca.jpg)

# PART II Types and Operations
## chapter 6 The Dynamic Typing Interlude
### 一、The Case of the Missing Declaration Statements
1. 在python中，不用申明脚本中使用的对象的确切类型。python为动态类型模式，c、c++、java为静态类型语言。

2. 变量、对象和引用
    - 在运行a = 3后的变量名和对象。变量a变成对象3的一个引用reference。在内部，变量事实上是到对象内存空间的一个指针pointer。These links from variables to objects are called references in Python。引用通过内存中的指针的形式来实现。
    - （1）变量是一个系统表的入口，包含了指向对象的连接。（2）对象是被分配到的一块内存，有足够的空间去表示它们所代表的值。（3）引用是自动形成的从变量到对象的指针。
    - 每一个对象都有两个标准的头部信息：类型标志符type designator标识了这个对象的类型；引用的计数器reference counter决定何时回收这个对象。  

3. 对象的垃圾回收garbage collection
    - 在python中，每当一个变量名被赋予一个新的对象，如果原来的对象没有被其他的变量名或对象所引用，那么之前占用的空间会被回收，即垃圾回收；
    - 每个对象保留一个计数器，记录当前指向对象的引用的数目。一旦计数器被设置为0，这个对象的内存空间会自动回收。假设每次x都被赋值给了一个新的对象，则前对象的引用计数器会变为零。
