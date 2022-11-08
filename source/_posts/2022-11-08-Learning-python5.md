---
title: 《Learning Python》读书笔记（五）
date: 2022-11-08 14:00:31
categories: 
  - [python, python读书笔记]
tags:
  - python
  - 读书笔记
cover: https://s2.loli.net/2022/09/17/JxdLBRD5Cjfvg37.jpg
---

> 该笔记为 **《Learning Python》** 的读书笔记，未经过系统的整理；  
> 该书涉及的内容可能过于啰嗦，但包含一些python背后的逻辑和机制，值得初学者观看；  
> 该笔记内容过多，所以不展示部分代码的结果，需复制到编辑器中查看；  
> 学习完成日期为2022年10月20日。  
> 本篇主要内容为：XXXXXXXXXXXXXXXXXXXXXXXXXXXX

<div  align="center">  
<img src="https://s2.loli.net/2022/09/17/ri9Ue6nguJdq1Ca.jpg" width = "80%" height = "80%" alt="Learning Python"/>
</div>

# PART IV Functions and Generators
## chapter 16 Function Basics
### 一、Coding Functions
1. 函数相关的语句和表达式
    - `def`创建了一个函数对象并将其赋值给了某一变量名；
    - `lambda`创建一个函数对象并将其作为结果返回，见第19章；
    - `return`将一个结果对象传回给调用者，默认返回None；
    - `yield`向调用者发回一个结果对象，并挂起它们的状态；
    - `global`声明了一个模块级的可被赋值的变量；
    - `nonlocal`声明了一个需要被赋值的外层函数变量；
    - 参数通过赋值(对象引用)传递给函数，除非你显式指明形式参数与实际参数的对应，否则实际参数按位置赋值给形式参数。参数、返回值与变量不需要被声明。

2. def语句
    - 一般格式一下：
    ``` python
    def name(arg1, arg2,... argN):
        statements
    ```
    - def的头部定义了被赋值函数对象的函数名，圆括号parentheses中包含了**形式参数arguments (sometimes called parameters)**，简称**形参**；
    - 在函数调用时，括号内的传入对象将赋值给头部的形式参数；
    - Python的return语句将结束函数调用并把结果返回至函数调用处，return语句是可选的，一个没有返回值的函数自动返回None对象；
    - 可以将函数赋值给一个不同的变量名，并通过新的变量名进行调用：
    ``` python
    othername = func
    othername()
    ```
    - 函数也是对象，在程序运行时被明确地记录在内存中。除了调用以外，函数允许将任意属性附加到其中以供之后使用：
    ``` python
    def func(): ... 
    func() 
    func.attr = value
    ```

### 二、A First Example: Definitions and Calls
1. 定义Definition
``` python
def times(x, y):
    return x * y
```

2. 调用Calls
    - 参数是通过赋值传入的，函数头部的形式参数x被赋值为2，y被赋值为4：
    ``` python
    print(times(2, 4))
    print(times('Ni', 4))
    ```
    - 