---
title: 《Learning Python》读书笔记（四）
date: 2022-11-04 14:17:54
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
> 本篇主要内容为：XXXXXXXXXXXXXXXXXXXXXXXXXX

<div  align="center">  
<img src="https://s2.loli.net/2022/09/17/ri9Ue6nguJdq1Ca.jpg" width = "80%" height = "80%" alt="Learning Python"/>
</div>

# PART III Statements and Syntax
## chapter 13 while and for Loops
### 一、while Loops
1. while循环Loops一般形式
``` python
while test:
    statements
else:
    statements
```

2. break、continue、pass和循环的else
    - `break`：跳过整个循环语句；
    - `continue`：来到循环头部；
    - `pass`：空占位语句；
    - `else`：当且仅当循环正常离开时才会执行。

3. pass  
    - `while True: pass`：pass语句是无运算的占位语句，因为主体只是空语句，所以python会陷入死循环（ctrl-c推出）；
    - pass主要用于定义类时，pass暂时填充函数主体，表示以后填上。

4. continue
``` python
x = 10
while x:
    x = x-1
    if x % 2 != 0: continue # 跳到循环顶端
    print(x, end=' ')
```

5. break
``` python
while True:
    name = input('Enter name:')
    if name == 'stop': break # 立即从循环体退出，包括else分句
    age = input('Enter age: ')
    print('Hello', name, '=>', int(age) ** 2)
```

6. 循环的else
``` python
y = input('Enter number:')
x = int(y) // 2
while x > 1:
    if int(y) % x == 0:
        print(y, 'has factor', x)
        break
    x -= 1
else:
    print(y, 'is prime')
```

### 二、for Loops
