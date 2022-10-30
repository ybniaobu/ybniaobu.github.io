---
title: 《Learning Python》读书笔记（三）
date: 2022-10-30 13:36:03
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
> 本篇主要内容为：XXXXXXXXXXXXXXXXXXXXXXX

![learning_python.jpg](https://s2.loli.net/2022/09/17/ri9Ue6nguJdq1Ca.jpg)

# PART II Types and Operations
## chapter 9 Tuples, Files, and Everything Else
### 一、Tuples
1. 元组的特性
任意对象的有序集合；通过偏移量存取；属于**不可变序列**；固定长度、多样性、任意嵌套；对象引用的数组。

2. 元组字面量和运算

| Operation | Interpretation |
| :---- | :---- |
| () | 空元组 |
| T = (0,) | 单个元素的元组 |
| T = (0, 'Ni', 1.2, 3) | 多元素元组 |
| T = 0, 'Ni', 1.2, 3 | 同上(无括号) |
| T = ('Bob', ('dev', 'mgr')) | 嵌套元组 |
| T = tuple('spam') | 可迭代对象的元素组成的元组 |
| T[i] | 索引 |
| T[i][j] | 嵌套索引 |
| T[i:j] | 切片 |
| len(T) | 长度 |
| T1 + T2 | 拼接 |
| T * 3 | 重复 |
| for x in T: print(x) | 迭代 |
| 'spam' in T | 成员关系 |
| [x ** 2 for x in T] | 列表推导 |
| T.index('Ni') | 查找方法 |
| T.count('Ni') | 计数 |
| namedtuple('Emp', ['name', 'jobs']) | 有名元组拓展类型 |

### 二、Tuples in Action

