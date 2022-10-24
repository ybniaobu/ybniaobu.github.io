---
title: 《Learning Python》读书笔记（二）
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
1. 在python中，不用申明脚本中使用的对象的确切类型。python为***动态类型模式***，c、c++、java为***静态类型语言***。

2. 变量、对象和引用
    - 在运行a = 3后的变量名和对象。变量a变成对象3的一个***引用reference***。在内部，变量事实上是到对象内存空间的一个***指针pointer***。These links from variables to objects are called references in Python。*引用通过内存中的指针的形式来实现*。
    - （1）变量是一个系统表的入口，包含了指向对象的连接。（2）对象是被分配到的一块内存，有足够的空间去表示它们所代表的值。（3）引用是自动形成的从变量到对象的指针。
    - 每一个对象都有两个标准的头部信息：类型标志符type designator标识了这个对象的类型；引用的计数器reference counter决定何时回收这个对象。  

3. 对象的***垃圾回收garbage collection***
    - 在python中，每当一个变量名被赋予一个新的对象，如果原来的对象没有被其他的变量名或对象所引用，那么之前占用的空间会被回收，即垃圾回收；
    - 每个对象保留一个计数器，记录当前指向对象的引用的数目。一旦计数器被设置为0，这个对象的内存空间会自动回收。假设每次x都被赋值给了一个新的对象，则前对象的引用计数器会变为零。

### 二、Shared References
1. ***共享引用Shared References***
    - 下面的示例的实际效果是变量a、b都引用了相同对象3（即相同的内存空间，该空间由字面量表达式3创建），a和b并没有彼此的关联：
    ``` python
    a = 3  
    b = a
    ```
    - 在python中，变量总是一个指向对象的指针：给变量赋一个新值，并不是替换原始的对象，而是让变量去引用另一个对象：
    ``` python 
    a = a + 2  
    print(a, b)
    ```

2. ***原位置修改In-Place Changes***
    - 有一些对象和操作确实会在原位置改变对象（包括列表、字典和集合在内的`Python可变类型`）。
    - 在一个列表中对一个偏移进行赋值确实会改变这个列表对象，而不是生成一个全新的列表对象：
    ``` python
    L1 = [2, 3, 4]
    L2 = L1
    L1[0] = 24
    print(L1, L2)
    ```
    - 如果你不想这样的现象发生，可以请求python复制对象，而不是创建引用，常用的是从头到尾的分片，也可以用list函数：
    ``` python
    L1 = [2, 3, 4]
    L2 = L1[:]
    L1[0] = 24
    print(L1, L2) # 2个变量指向不同的内存空间
    ```
    - 这种分片不会应用在字典和集合上（因为不是序列），要用标准库的`copy模块`进行复制，或者用dict和set函数：
    ``` python
    D = {'a':'haha', 'b':'bob'}
    import copy
    X = copy.copy(D) # 浅拷贝：拷贝父对象，不会拷贝对象的内部的子对象。
    Y = copy.deepcopy(D) # 深度拷贝：完全拷贝了父对象及其子对象。
    D['c'] = 'niaobu'
    print(X, D, Y) # 关于浅拷贝和深度拷贝，详见第八、九章
    ```

3. 共享引用和相等
    - 基于python的引用模型，有2个不同的方法去检查是否相等:
    ``` python
    L = [1, 2, 3]
    M = L
    print(L == M) # 测试对象是否具有相同的值
    print(L is M) # 检查对象的同一性，如果2个变量名指向同一对象，它会返回True。即is比较了实现引用的指针
    ```
    - ***对象缓存cache和复用reuse机制***:
    ``` python
    L = [1, 2, 3]
    M = [1, 2, 3] # M和L引用了不同的对象
    print(L == M, L is M)
    X = 42
    Y = 42
    print(X == Y,X is Y) # 因为小的整数和字符串被缓存并复用了，所以is告诉我们X和Y引用了一个相同的对象。
    ```
    - 查询一个对象的引用次数，用标准的`sys模块`的`getrefcount`函数
    ``` python
    import sys
    print(sys.getrefcount(1))
    ```


## chapter 7 String Fundamentals
