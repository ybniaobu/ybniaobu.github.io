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
1. for循环
    - `for循环`是一个通用的序列迭代器，用来遍历任何有序序列或者其他可迭代对象内的元素：
    ``` python
    for target in object:
        statements
    else:
        statements
    ```
    - python在运行for循环时，会逐个将可迭代对象object中的元素赋值给名称target：
    ``` python
    for x in ["spam", "eggs", "ham"]:
        print(x, end=' ')

    sum = 0
    for x in [1, 2, 3, 4]:
        sum = sum + x
    print(sum)
    ```
    - for循环用于字符串和元组：
    ``` python
    S = "lumberjack"
    T = ("and", "I'm", "okay")
    for x in S: print(x, end=' ')
    for x in T: print(x)
    ```

2. for循环中的元组赋值
    - 元组赋值：
    ``` python
    T = [(1, 2), (3, 4), (5, 6)]
    for (a, b) in T:
        print(a, b)

    D = {'a': 1, 'b': 2, 'c': 3}
    for key in D:
        print(key, '=>', D[key])
    for (key, value) in D.items():
        print(key, '=>', value)
    ```
    - 可以在循环中手动赋值解包：
    ``` python
    T = [(1, 2), (3, 4), (5, 6)]
    for both in T:
        a, b = both
        print(a, b)
    ```
    - 嵌套的结构自动解包：
    ``` python
    for ((a, b), c) in [((1, 2), 3), ((4, 5), 6)]: print(a, b, c)
    for ((a, b), c) in [([1, 2], 3), ['XY', 6]]: print(a, b, c)
    ```

3. for循环中的拓展序列赋值：
``` python
for (a, *b, c) in [(1, 2, 3, 4), (5, 6, 7, 8)]:
    print(a, b, c)
```

4. 嵌套for循环
``` python
items = ["aaa", 111, (4, 5), 2.01]
tests = [(4, 5), 3.14]
for key in tests:
    for item in items:
        if item == key:
            print(key, "was found")
            break
    else:
        print(key, "not found!")

seq1 = "spam"
seq2 = "scam"
res = []
for x in seq1:
    if x in seq2:
        res.append(x)
print(res)
# 上面例子可以直接用列表推导式
res = [x for x in seq1 if x in seq2]
print(res)
```

5. 文件扫描器File Scanners
``` python
for line in open('13-while and for Loops/test.txt').readlines(): # readlines方法生成每一行的字符串的列表
    print(line.rstrip())
for line in open('13-while and for Loops/test.txt'): # 文件迭代器
    print(line.rstrip())
```

### 三、Loop Coding Techniques
1. 计数器循环Counter Loops：range
    - `range`是一个可迭代对象，当range传入2个参数，第一个参数为下边界。第三个可选参数为步长（默认值+1）：
    ``` python
    print(list(range(5)))
    print(list(range(-5, 5)))
    print(list(range(5, -5, -1)))
    ```
    - 序列乱序器Sequence Shufflers：
    ``` python
    S = 'spam'
    for i in range(len(S)):
        S = S[1:] + S[:1]
        print(S)

    for i in range(len(S)):
        X = S[i:] + S[:i]
        print(X)
    ```
    - 非穷尽遍历Nonexhaustive Traversals：
    ``` python
    S = 'abcdefghijk'
    for i in range(0, len(S), 2):
        print(S[i], end=' ')
    # 上面例子不如直接用分片表达式的第三个参数
    S = 'abcdefghijk'
    for c in S[::2]:  # 见7-3
        print(c, end=' ')
    ```

2. 并行遍历Parallel Traversals：zip和map
    - `zip函数`将序列并排元素配对得到元组的列表，返回的是一个可迭代对象，需要用list调用来显示结果：
    ``` python
    L1 = [1,2,3,4]
    L2 = [5,6,7,8]
    print(list(zip(L1, L2)))

    for (x, y) in zip(L1, L2):
        print(x, y, '--', x+y)
    ```
    - 当各个参数长度不一时，zip按最短序列长度为准：
    ``` python
    S1 = 'abc'
    S2 = 'xyz123'
    print(list(zip(S1, S2)))
    ```
    - `map函数`：输入函数和序列，从序列抽取元素调用函数，并收集结果，详见19、20章：
    ``` python
    print(list(map(ord, 'spam')))
    ```
    - 使用`zip`来构造字典：
    ``` python
    keys = ['spam', 'eggs', 'toast']
    vals = [1, 3, 5]
    D3 = dict(zip(keys, vals))
    print(D3)
    ```
    - 字典推导：
    ``` python
    print({k: v for (k, v) in zip(keys, vals)})
    ```

3. enumerate函数：同时给出偏移量和元素
    ``` python
    S = 'spam'
    for (offset, item) in enumerate(S):
        print(item, 'appears at offset', offset)
    ```
    - `enumerate函数`会返回一个**生成器对象generator object**：支持**迭代协议iteration protocol**，可以被`内置函数next`调用，详见第14章：
    ``` python
    E = enumerate(S)
    print(E)
    print(next(E))
    print(next(E))
    print(next(E))
    ```
    - 推导和for循环：
    ``` python
    print([c * i for (i, c) in enumerate(S)])

    for (i, l) in enumerate(S):
        print('%s) %s' % (i, l.rstrip()))
    ```


## chapter 14 Iterations and Comprehensions
### 一、Iterations
1. 简单介绍
**迭代工具iteration tools**可用于任何**可迭代对象iterable object**；*迭代工具包括for循环、列表推导、in成员关系测试以及map函数等*，可迭代对象本质上就是序列观念的通用化。可迭代对象包括实际序列以及虚拟序列。

2. 可迭代对象iterable与迭代器iterator
    - ***可迭代对象***指代一个支持`iter`调用的对象；
    - ***迭代器***指代一个支持`next()`调用的对象：
    - ***生成器generator***指代能自动支持迭代协议的对象，生成器本身就是可迭代对象。

3. 迭代协议Iteration Protocol：文件迭代器
    - `readline方法`：
    ``` python
    print(open('14-Iterations and Comprehensions/script2.py').read())
    f = open('14-Iterations and Comprehensions/script2.py')
    print(f.readline())
    print(f.readline())
    print(f.readline())
    print(f.readline())
    print(f.readline())
    ```
    - 文件也有一个名为`_next_`的方法，跟上面有着相同的效果，即每次调用返回文件的下一行。唯一的区别是，到达文件尾部，`_next_`会引发内置`StopIteration异常`，而不是返回空字符串：
    ``` python
    f = open('14-Iterations and Comprehensions/script2.py')
    print(f.__next__())
    print(f.__next__())
    print(f.__next__())
    print(f.__next__())
    ```
    - 这个接口基本上就是Python中的迭代协议：所有带`_next_方法`的对象会前进到下一个结果，这个对象也被称为迭代器；
    - 任何这类对象也能在for循环或其他迭代工具遍历，因为所有迭代工具都是在每次迭代中调用`_next_`，并且捕捉`StopIteration异常`来确定何时离开；
    - 完整的迭代协议包括iter调用，见后面。
    ``` python
    for line in open('14-Iterations and Comprehensions/script2.py'):
        print(line.upper(), end='') # 这里end=''，因为行字符串已经自带一个\n，如果没有end=''，则会变成2行隔开
    ```
    - `readlines方法`会形成行字符串的列表，并一次性加载至内存，如果文件太大可能会无法工作。而`readline方法`基于迭代器，一次只读一行，所以运行更快：
    ``` python
    for line in open('14-Iterations and Comprehensions/script2.py').readlines():
        print(line.upper(), end='')
    ```
    - 或用`while循环`逐行读取文件，然而while循环会比基于迭代器的for循环运行得更慢，因为迭代器在python内部以C语言的速度运行，而while循环则是通过Python虚拟机运行Python字节码：
    ``` python
    f = open('14-Iterations and Comprehensions/script2.py')
    while True:
        line = f.readline()
        if not line: break
        print(line.upper(), end='')
    ```
    - 在第21章会介绍一种计时技术，可以衡量替代方案的相对速度。

4. 手动迭代：iter和next
