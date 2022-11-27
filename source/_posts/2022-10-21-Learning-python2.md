---
title: 《Learning Python》读书笔记（二）
date: 2022-10-21 09:54:48
categories: 
  - [python, python读书笔记]
tags: 
  - python
  - 读书笔记
top_img: /images/black.jpg
cover: https://s2.loli.net/2022/11/27/R9htYK5LyFuEGTk.jpg
---

> 该笔记为 **《Learning Python》** 的读书笔记，由于是早期未搞熟博客系统时所写，笔记结构较为混乱；  
> 该书涉及的内容可能过于啰嗦，但包含一些python背后的逻辑和机制，可以粗略过一遍，但若仔细阅读就是在坑自己；  
> 该笔记内容过多，所以不展示部分代码的结果，需复制到编辑器中查看；  
> 学习完成日期为2022年10月20日。  
> 本篇主要内容为：Python解释器；Python对象类型的初步介绍；数值类型。

<div  align="center">  
<img src="https://s2.loli.net/2022/09/17/ri9Ue6nguJdq1Ca.jpg" width = "80%" height = "80%" alt="Learning Python"/>
</div>


# PART III Statements and Syntax
## chapter 10 Introducing Python Statements
### 一、Python’s Statements
1. Python程序结构  
*程序由模块构成；模块包含语句；语句包含表达式；表达式创建并处理对象*  
Programs are composed of modules；Modules contain statements；Statements contain expressions；Expressions create and process objects

2. Python’s Statements语句

| Statement | Role | Example |
| :---- | :---- | :---- |
| 赋值语句 | 创建引用reference | a, b = 'good', 'bad' |
| 调用和其他表达式 | 运行函数 | log.write("spam, ham") |
| print语句 | 打印对象 | print('The Killer', joke) |
| if/elif语句 | 选择行为 | if "python" in text: print(text) |
| for语句 | 迭代 | for x in mylist: print(x) |
| while语句 | 循环loop | while X > Y: print('hello') |
| pass | 空占位符Empty placeholder | while True: pass |
| break | 退出循环 | while True: if exittest(): break |
| continue | 循环继续 | while True: if skiptest(): continue |
| def | 函数与方法 | def f(a, b, c=1, *d): print(a+b+c+d[0]) |
| return | 函数结果 | def f(a, b, c=1, *d): return a+b+c+d[0] |
| yield | 生成器函数 | def gen(n): for i in n: yield i*2 |
| global | 命名空间Namespaces | def function(): global x |
| nonlocal | 非局部声明 | def outer(): x = 'old' def function(): nonlocal x; x = 'new' |
| import | 获取模块 | import sys |
| from | 获取模块属性 | from sys import stdin |
| class | 构建对象 | class Subclass(Superclass): |
| try/except/finally | 捕捉异常Catching exceptions | try: |
| raise | 触发异常 | raise EndSearch(location) |
| assert | 调试异常Debugging checks | assert X > Y, 'X too small' |
| with/as | 上下文管理器 | with open('data') as myfile: |
| del | 删除引用 | del data[k] |

3. **语句分隔符**：即分号，同一行出现多个语句时使用：
`a = 1; b = 2; print(a + b)`

## chapter 11 Assignments, Expressions, and Prints
### 一、Assignment Statements
1. 赋值语句形式Assignment Statement Forms

| Operation | Interpretation |
| :---- | :---- |
| spam = 'Spam' | 基础模式 |
| spam, ham = 'yum', 'YUM' | 元组赋值（基于位置） |
| [spam, ham] = ['yum', 'YUM'] | 列表赋值（基于位置） |
| a, b, c, d = 'spam' | 序列赋值 |
| a, *b = 'spam' | 扩展序列解包Extended sequence unpacking |
| spam = ham = 'lunch' | 多目标赋值 |
| spams += 42 | 增量赋值Augmented assignments |

2. 序列赋值
    - 序列赋值右侧可以接受任意类型的序列（可迭代对象），只要长度等于左侧序列即可：
    ``` python
    [a, b, c] = (1, 2, 3)
    print(a, c)
    (a, b, c) = "ABC"
    print(a, c)
    ```
    - 赋值内嵌序列：
    ``` python
    ((a, b), c) = ('SP', 'AM')
    print(a, b, c)
    ```
    - 序列解包赋值：
    ``` python
    red, green, blue = range(3)
    print(red, blue)
    ```
    - 循环中把序列分割为开头和剩余两部分：
    ``` python
    L = [1, 2, 3, 4]
    while L:
        front, L = L[0], L[1:]
        print(front, L)
    ```

3. 拓展序列解包Extended Sequence Unpacking
    - 带星号的名称（*X）会被赋值***一个列表***，收集序列剩下的没被赋值给其他名称的所有项：
    ``` python
    seq = [1, 2, 3, 4]
    a, *b = seq # a匹配第一项，b匹配剩下的内容
    print(a, b)
    *a, b = seq
    print(a, b)
    a, *b, c = seq
    print(a, b, c)
    ```
    运行结果如下：
    ``` python
    1 [2, 3, 4]
    [1, 2, 3] 4
    1 [2, 3] 4
    ```
    - 拓展的序列解包对于任意可迭代对象都有效：
    ``` python
    a, *b, c = 'spam'
    print(a, b, c)
    a, *b, c = range(4)
    print(a, b, c)
    ```
    运行结果如下：
    ``` python
    s ['p', 'a'] m
    0 [1, 2] 3
    ```
    - 和分片的区别：
    ``` python
    S = 'spam'
    print(S[0], S[1:3], S[3])
    ```
    运行结果如下：
    ``` python
    s pa m
    ```
    - 循环：
    ``` python
    L = [1, 2, 3, 4]
    while L:
        front, *L = L
        print(front, L)
    ```
    运行结果如下：
    ``` python
    1 [2, 3, 4]
    2 [3, 4]
    3 [4]
    4 []
    ```
    - *带星号的名称有可能只匹配到单个的项，但总会向其赋值一个列表*：
    ``` python
    seq = [1, 2, 3, 4]
    a, b, c, *d = seq
    print(a, b, c, d)
    ```
    运行结果如下：
    ``` python
    1 2 3 [4]
    ```
    - 若无剩下的内容，则被赋值一个空列表：
    ``` python
    a, b, c, d, *e = seq
    print(a, b, c, d, e)
    a, b, *e, c, d = seq
    print(a, b, c, d, e)
    *a, = seq
    print(a)
    a, *b, c = seq
    print(a, b, c)
    ```
    运行结果如下：
    ``` python
    1 2 3 4 []
    1 2 3 4 []
    [1, 2, 3, 4]
    1 [2, 3] 4
    ```

4. 增量赋值
在X += Y中，代码只需运行一次。但X = X + Y，X会出现2次，也必须执行2次；
增量赋值有自动选择的优化技术。对于支持原位置改变的对象，自动选择原位置修改，而不是更慢的复制运算：
``` python
L = [1, 2]
L = L + [3] # 拼接会创建一个新对象
L.append(4) # 原位置改变，会更快
L.extend([5, 6]) # 原位置改变，会更快
# 而增量赋值会自动调用较快的extend方法，而不是使用较慢的+拼接运算
L += [7, 8]
# 所以+=不在所有情况都等于+和=，对于列表+=更像extend，能接受任意序列
L = []
L += 'spam'
print(L)
# +=对于列表就是原位置修改
L = [1, 2]
M = L
L = L + [3, 4]
print(L, M)
L = [1, 2]
M = L
L += [3, 4]
print(L, M)
```

### 二、Expression Statements
1、常见的表达式语句  

| Operation | Interpretation |
| :---- | :---- |
| spam(eggs, ham) | 函数调用 |
| spam.ham(eggs) | 方法调用 |
| spam | 在交互式解释器打印 |
| print(a, b, c) | print语句 |
| yield x ** 2 | yield表达式语句 |

表达式作为语句（让表达式独占一行），这样不会存储表达式结果。  
print也会像其他函数调用一样返回一个值（返回值为None，为不返回任何有意义内容的函数的默认返回值）：
``` python
x = print('spam')
print(x)
```

2、 表达式语句用于原位置修改
``` python
L = [1, 2]
L.append(3) # 表达式语句
print(L)
# 但新手可能会写成赋值语句
L = L.append(4)
print(L) # 返回None对象
```

### 三、Print Operations
1. 打印操作  
python的打印操作与文件和流的概念紧密相连：  
（1）文件对象方法：print将对象写入**stdout流**，同时加入一些自动的格式化；  
（2）**标准输出流standard output stream（常称为stdout）**：是发送一个程序文本输出的默认位置。加上**标准输入流standard input**和**标准出错流error streams**，为脚本启动时所创建的3种数据连接；  
（3）标准输出流在python中可以作为内置的sys模块中的stdout文件对象来使用。  

2. print函数
    - print内置函数的调用通常独占一行，但它不会返回值（返回None）；
    - 调用形式：`print([object, ...][, sep=' '][, end='\n'][, file=sys.stdout][, flush=False])`
    - print内置函数打印一个或多个对象，在中间用字符串`sep`来分隔，`sep`默认一个单个的空格，在结尾加上字符串`end`，通过`file`来指定输出流，并按照`flush`来决定是否刷新输出缓冲区：
    ``` python
    x = 'spam'
    y = 99
    z = ['eggs']
    print(x, y, z)
    print(x, y, z, sep=', ')
    print(x, y, z, end=''); print(x, y, z)
    print(x, y, z, sep='...', file=open('11-Assignments, Expressions, and Prints\data.txt', 'w')) # print to a file
    print(x, y, z) # back to stdout
    print(open('11-Assignments, Expressions, and Prints\data.txt').read())
    ```

3. 打印流重定向
    - 打印都默认将文本发送到标准输出流，也可以发送到其他地方；
    - print提供了sys.stdout对象的简单接口，再加上一些默认的格式设置；
    - 也可以通过下面编写打印操作：
    ``` python
    import sys
    sys.stdout.write('hello world\n')
    ```
    - 上述程序显示调用了`sys.stdout`的`write方法`。而print操作隐藏了大部分细节：`print(X, Y)`等价于`import sys`和`sys.stdout.write(str(X) + ' ' + str(Y) + '\n')`
    - 可以把sys.stdout重新赋值给标准输出流以外的对象：
    ``` python
    import sys
    sys.stdout = open('11-Assignments, Expressions, and Prints\log.txt', 'a') # redirect prints to a file
    print(x, y, x) # shows up in log.txt
    ```
    - 在这里把`sys.stdout`重设成一个已打开的名为log.txt的文件对象，该文件以附加模式打开。重设后，print会将文本写至文件log.txt的末尾，而不是原本的输出流。print语句会持续调用`sys.stdout`的`write`方法。通过这种方式赋值`sys.stdout`会让程序中所有的print都被重新定向。

4. 恢复输出流定向
    - 先把sys.stdout存储到一个对象里：
    ``` python
    import sys
    temp = sys.stdout # 先把sys.stdout存储到一个对象里
    sys.stdout = open('11-Assignments, Expressions, and Prints\log.txt', 'a')
    print('spam')
    print(1, 2, 3)
    sys.stdout.close() # Flush output to disk
    sys.stdout = temp # Restore original stream
    print('back here')
    print(open('11-Assignments, Expressions, and Prints\log.txt').read())
    ```
    - 这样手动保存和恢复原始输出流需要额外工作，所以python引入了print拓展，`file`关键字允许一个单次的print调用将文本发送给一个文件的`write方法`，而不是费力地重设`sys.stdout`。print拓展的重定向是临时的，之后的print还是会继续打印到标准输出流。
    ``` python
    log2= open('11-Assignments, Expressions, and Prints\log2.txt', 'w') # 同上见test2.py
    print(1, 2, 3, file=log2)
    print(4, 5, 6, file=log2)
    log2.close()
    print(7, 8, 9)
    print(open('11-Assignments, Expressions, and Prints\log2.txt').read())
    ```

5. 标准错误流（standard error stream）sys.stderr
    - 拓展的print形式也常用于把错误消息打印到标准错误流`sys.stderr`：
    ``` python
    import sys
    sys.stderr.write(('Bad!' * 8) + '\n') # 标准错误流的write方法
    print('Bad!' * 8, file=sys.stderr)
    ```

6. 理解print语句和sys.stdout之间的等价性
    - print语句只是把文本传送给`sys.stdout.write方法`，所以可以把`sys.stdout`赋值给一个对象，来捕获程序中待打印的文本，并通过该对象的write方法处理文本：
    ``` python
    class FileFaker:
        def write(self, string):
            Do something with printed text in string
    import sys
    sys.stdout = FileFaker()
    print(someObjects) # Sends to class write method
    ```
    - `sys.stdout`是什么不重要，只要它有一个名为write的方法(接口)即可，详见第六部分类。


## chapter 12 if Tests and Syntax Rules
### 一、if Statements
1. 基础示例
``` python
if 1: # 1是布尔真值，即True
    print('true')

if not 1:
    print('true')
else:
    print('false')
```

2. 多路分支  
Python会执行第一次测试为真的语句，当所有的测试都为假时执行else部分：
``` python
x = 'killer rabbit'
if x == 'roger':
    print("shave and a haircut")
elif x == 'bugs':
    print("what's up doc?")
else:
    print('Run away! Run away!')
```

3. 语句分隔符：行与行间连接符
    - 如果使用语法括号对，语句可横跨数行，比如封闭的()、{}、[]；
    - 如果语句以反斜杠`\`结尾，可横跨数行；
    - 三重引号字符串可横跨数行：
    ``` python
    L = ["Good",
    "Bad",
    "Ugly"]
    # 反斜杠来继续多行（不常用）：
    if a == b and c == d and \
        d == e and f == g:
    print('olde')
    # 因为任何表达式都可以包含在括号内：
    if (a == b and c == d and
        d == e and e == f):
    print('new')
    ```

4. 真值和布尔测试
    - 所有对象都有一个固有的布尔真/假值，任何非零数字或非空对象都为真，数字零、空对象以及None都为假；
    - 比较相等测试会递归地应用到数据结构；
    - 布尔`and`和`or`运算符会在结果确定的时候立即停止计算（‘短路’）；
    - 对于`or`测试，python会从左到右计算操作对象，然后返回第一个为真的对象，一旦得出结果就使表达式其余部分短路（终止）：
    ``` python
    print(2 or 3, 3 or 2) # 真或真
    print([] or 3) # 假或真
    print([] or {}) # 假或假
    ```
    运行结果如下：
    ``` python
    2 3
    3
    {}
    ```
    - 对于`and`测试，python会从左到右计算操作对象，当遇到假的对象就停止运算：
    ``` python
    print(2 and 3, 3 and 2) # 真和真
    print([] and {}) # 假和假
    print(3 and []) # 真和假
    ```
    运行结果如下：
    ``` python
    3 2
    []
    []
    ```

5. if/else三元表达式Ternary Expression  
`A = Y if X else Z`：当X为真时，执行表达式Y，否则执行Z
``` python
A = 't' if 'spam' else 'f'
print(A)
A = 't' if '' else 'f'
print(A)
```

6. `filter函数`或列表推导来筛选真的对象
``` python
L = [1, 0, 2, 0, 'spam', '', 'ham', []]
print(list(filter(bool, L)))
print([x for x in L if x])
```

7. `any`和`all`内置函数用于检测是否存中或者所有元素都为真
``` python
print(any(L), all(L))
```


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
    - 内置函数`next`，自动调用对象的`_next_方法`：
    ``` python
    f = open('14-Iterations and Comprehensions/script2.py')
    print(next(f))
    print(next(f))
    ```
    - 迭代协议还有一点：*for循环在开始时，会首先把可迭代对象传入内置函数iter，并由此拿到一个迭代器，返回的迭代器有_next_方法*；
    - iter函数也是在内部调用_iter_方法。

5. 完整的迭代协议
    - (1) **可迭代对象The iterable object**：迭代的被调对象，其`_iter_方法`被`iter函数`调用
    - (2) **迭代器对象The iterator object**：可迭代对象的返回结果，在迭代过程中实际提供值的对象。其`_next_方法`被`next`运行，并在结束时触发`StopIteration异常`。
    ``` python
    L = [1, 2, 3]
    I = iter(L)
    print(I.__next__())
    print(I.__next__())
    print(I.__next__())
    ```
    - `iter`这一步对于文件不是必须的，因为文件对象自身就是迭代器：
    ``` python
    f = open('14-Iterations and Comprehensions/script2.py')
    print(iter(f) is f)
    print(iter(f) is f.__iter__())
    ```
    - 对于列表等可迭代对象但自身不是迭代器的，需要调用iter来启动迭代：
    ``` python
    L = [1, 2, 3]
    print(iter(L) is L)
    I = iter(L)
    print(I.__next__())
    print(next(I))
    ```

6. 其他内置类型可迭代对象Other Built-in Type Iterables
    - 字典在迭代上下文中会自动返回键：
    ``` python
    D = {'a':1, 'b':2, 'c':3}
    I = iter(D)
    print(next(I))
    print(next(I))
    print(next(I))

    for key in D:
        print(key, D[key])
    ```

### 二、List Comprehensions
1. 简单介绍
    - 列表推导会产生新的列表对象，并且比for循环语句运行更快，因为在解释器内部由C语言的速度执行：
    ``` python
    L = [1, 2, 3, 4, 5]
    L = [x + 10 for x in L]
    print(L)
    ```

2. 在文件上使用列表推导
    - 列表推导也有垃圾回收机制，在表达式运行结束后，将临时文件对象关闭，对于Cpython以外的python版本，需要手动关闭文件：
    ``` python
    lines = [line.rstrip() for line in open('14-Iterations and Comprehensions\script2.py')]
    print(lines)
    lines = [line.rstrip().upper() for line in open('14-Iterations and Comprehensions\script2.py')] # 方法链式调用是有效的
    print(lines)
    ```

3. 拓展的列表推导语法
    - 筛选分句：if：
    ``` python
    lines = [line.rstrip() for line in open('14-Iterations and Comprehensions\script2.py') if line[0] == 'p']
    print(lines)
    # 更复杂的例子：
    lines = [line.rstrip() for line in open('14-Iterations and Comprehensions\script2.py') if line.rstrip()[-1].isdigit()]
    print(lines)
    ```
    - 嵌套循环：for
    ``` python
    L = [x + y for x in 'abc' for y in 'lmn']
    print(L)
    # 其等价形式：
    res = []
    for x in 'abc':
        for y in 'lmn':
            res.append(x + y)
    print(res)
    ```

### 三、Other Iteration Contexts
1. 其他迭代上下文
    - 用户定义的类也可以实现迭代协议。
    - `map`把一个函数调用应用于传入的可迭代对象中的每一项，`内置函数map`当作用于一个文件时，也利用了文件对象的迭代器来逐行扫描，通过`_iter_`获取一个迭代器并每次调用`_next_方法`：
    ``` python
    M = map(str.upper, open('script.py'))
    print(M)
    print(list(M))
    ```

2. 其他可处理可迭代对象的内置函数、方法或工具
    - `sorted`排序可迭代对象中的每项，返回的是一个新的list：  
    `print(sorted(open('script.py')))`
    - `zip`能够组合可迭代对象中的每项：  
    `print(list(zip(open('script.py'), open('script.py'))))`
    - `enumerate`把可迭代对象中的项与它们的相对位置进行匹配：  
    `print(list(enumerate(open('script.py'))))`
    - `filter`按照一个函数是否为真来选择可迭代对象中的项：  
    `print(list(filter(bool, open('script.py'))))`
    - 字符串`join方法`：  
    `print('&&'.join(open('script.py')))`
    - 序列赋值：
    ``` python
    a, b, c, d = open('script.py')
    print(a, d)
    a, *b = open('script.py')
    print(a, b)
    ```
    - in成员测试：
    ``` python
    print('y = 2\n' in open('script.py'))
    print('x = 2\n' in open('script.py'))
    ```
    - 分片赋值：
    ``` python
    L = [11, 22, 33, 44]
    L[1:3] = open('script.py')
    print(L)
    ```
    - 列表的`extend方法`：
    ``` python
    L = [11]
    L.extend(open('script.py'))
    print(L)
    ```
    - `append`不能自动迭代：
    ``` python
    L = [11]
    L.append(open('script.py'))
    print(L)
    print(list(L[1]))
    ```
    - 字典推导表达式：
    ``` python
    print({ix: line for ix, line in enumerate(open('script.py'))})
    print({ix: line for (ix, line) in enumerate(open('script.py')) if line[0] == 'p'})
    ```
    - `*arg`形式，可以把对象的值解包成单个参数，也接受任何可迭代对象：
    ``` python
    def f(a, b, c, d): print(a, b, c, d, sep='&')
    f(*[1, 2, 3, 4])
    f(*open('script.py'))
    ```

### 四、New Iterables in Python 3.X
1. range可迭代对象
    - range对象支持迭代、索引以及len函数：
    ``` python
    M = map(lambda x: 2 ** x, range(3))
    for i in M: print(i)

    R = range(10)
    print(R)
    I = iter(R) # Make an iterator from the range iterable
    print(next(I))
    print(next(I))
    print(next(I))

    print(len(R))
    print(R[0])
    print(R[-1])
    print(next(I))
    print(I.__next__())
    ```

2. map、zip和filter可迭代对象
    - 与range不同，上述对象本身就是迭代器，无需用`iter()`转换；
    - `map函数`：
    ``` python
    M = map(abs, (-1, 0, 1))
    print(M)
    print(next(M))
    print(next(M))
    print(next(M))
    for x in M: print(x) # 结果为空，因为在遍历其结果一次后，就用尽了map iterator is now empty

    M = map(abs, (-1, 0, 1)) # Make a new iterable/iterator to scan again
    for x in M: print(x) # Iteration contexts auto call next()

    print(list(map(abs, (-1, 0, 1))))
    ```
    - `zip函数`：
    ``` python
    Z = zip((1, 2, 3), (10, 20, 30))
    print(Z)
    print(list(Z))
    for pair in Z: print(pair) # Exhausted after one pass

    Z = zip((1, 2, 3), (10, 20, 30))
    for pair in Z: print(pair)

    Z = zip((1, 2, 3), (10, 20, 30))
    print(next(Z))
    print(next(Z))
    ```
    - `filter函数`，传入一个函数返回得到True的各项，filter可以接受一个可迭代对象处理，并返回一个可迭代对象：
    ``` python
    print(filter(bool, ['spam', '', 'ni']))
    print(list(filter(bool, ['spam', '', 'ni'])))
    ```

3. **多遍迭代器vs单遍迭代器 Multiple Versus Single Pass Iterators**
    - range不是自己的迭代器，并且支持在其结果上的多个迭代器：
    ``` python
    R = range(3)
    I1 = iter(R)
    print(next(I1))
    print(next(I1))
    I2 = iter(R)
    print(next(I2))
    print(next(I1))
    ```
    运行结果如下：
    ``` python
    0
    1
    0
    2
    ```
    - 相反，zip、map和filter不支持同一结果上的多个活跃迭代器，因此iter调用是可选的，它们的iter结果就是它们自身：
    ``` python
    Z = zip((1, 2, 3), (10, 11, 12))
    I1 = iter(Z)
    I2 = iter(Z)
    print(next(I1))
    print(next(I1))
    print(next(I2))
    ```
    运行结果如下：
    ``` python
    (1, 10)
    (2, 11)
    (3, 12)
    ```

4. 字典视图可迭代对象Dictionary View Iterables
    - 字典的`keys`、`values`和`items`方法会返回可迭代的视图对象：
    ``` python
    D = dict(a=1, b=2, c=3)
    print(D)
    K = D.keys()
    print(K)
    I = iter(K)
    print(next(I))
    print(next(I))
    for k in D.keys(): print(k)
    ```
    - 字典本身就是可迭代对象，带有返回键的迭代器：
    ``` python
    I = iter(D)
    print(next(I))
    print(next(I))
    ```


## chapter 15 The Documentation Interlude
### 一、Python Documentation Sources
1. Python文档资源  
井号注释；dir函数；文档字符串：\_\_doc\_\_；PyDoc: help函数；PyDoc: HTML报告；Sphinx第三方工具；标准手册集；网络资源；已出版的书籍

2. 井号注释  
文档字符串docstrings是较大型功能性的文档的最优选择，#注释更适用于较小代码的文档。

3. dir函数
    - `dir函数`可以抓取对象内所有可用属性，可以向其传入对象、模块、内置类型或者数据类型的名字：
    ``` python
    import sys
    print(dir(sys))
    print(len(dir(sys)))
    print(len([x for x in dir(sys) if not x.startswith('__')])) # 双下划线开头意味着与解释器相关
    print(len([x for x in dir(sys) if not x[0] == '_'])) # 单下划线开头意味着非正式的私有属性实现
    ```
    - 查看列表和字符串的属性，可以向dir传入空列表和空字符串：
    ``` python
    print(dir([]))
    print(dir(''))
    print(len(dir([])), len([x for x in dir([]) if not x.startswith('__')]))
    print(len(dir('')), len([x for x in dir('') if not x.startswith('__')]))
    print([a for a in dir(list) if not a.startswith('__')])
    print([a for a in dir(dict) if not a.startswith('__')])
    ```
    - 可以传入一个类型名称而不是字面量：
    ``` python
    print(dir(str) == dir(''))
    print(dir(list) == dir([]))
    ```

4. 文档字符串：\_\_doc\_\_
    - Python会自动装载文档字符串的文本，使其成为相应对象的`__doc__属性`：
    ``` python
    import docstrings
    print(docstrings.__doc__)
    print(docstrings.square.__doc__)
    print(docstrings.Employee.__doc__)
    ```
    docstrings.py代码如下：
    ``` python
    """
    Module documentation
    Words Go Here
    """
    spam = 40
    def square(x):
        """
        function documentation
        can we have your liver then?
        """
        return x ** 2 # square

    class Employee:
        "class documentation"
        pass

    print(square(4))
    print(square.__doc__)
    ```
    - 内置文档字符串Built-in docstrings：
    ``` python
    import sys
    print(sys.__doc__)
    print(sys.getrefcount.__doc__)
    print(int.__doc__)
    print(map.__doc__)
    ```

5. PyDoc：help函数
    - 标准的PyDoc工具是一段Python程序，用于提取文档字符串及相关的结构化信息；
    - 最主要的PyDoc接口是内置的help函数同PyDoc基于GUI和基于Web的HTML报告接口：
    ``` python
    import sys
    help(sys.getrefcount)
    help(sys) # 按空格键移动到下一页，按回车键移动到下一行，按Q键退出
    ```
    - help也可以传入内置函数、方法以及类型名称：
    ``` python
    help(str.replace)
    help(''.replace)
    ```
    - help函数也可以用于自己的模块：
    ``` python
    import docstrings
    help(docstrings.square)
    help(docstrings.Employee)
    help(docstrings)
    ```

6. PyDoc：HTML报告
    - PyDoc的全浏览器模式，可以通过开始菜单中的模块文档启动（Python 3.10 Module Docs (64-bit)）；也可以通过命令行`pydoc -g`启动；可以任意选择下面三种命令：  
    `c:\code> python -m pydoc -b`  
    `c:\code> py -m pydoc -b`  
    `c:\code> C:\python33\python -m pydoc -b`  

7. Sphinx：更强大的方式为python系统编写文档  
详见http://sphinx-doc.org

8. 标准手册集The Standard Manual Set
    - 可以通过开始菜单中的Python 3.10 Manuals (64-bit)启动；
    - 也可以从IDLE的help选项菜单中打开；
    - 还可以从http://www.python.org 官方网站获取



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
    - *函数体（嵌套在函数定义语句中的代码）在函数被一个调用表达式调用时才会执行*；
    - 不调用是不执行函数体的，可以试试def一个有错误的函数，不调用它去运行。

3. Python中的**多态Polymorphism**
    - 如上所示，times函数中表达式x*y完全取决于x和y的对象类型，这种依赖类型的行为称为**多态Polymorphism**；
    - 函数可以自动地应用到所有类别的对象上。只要对象支持所预期的接口（也称为协议，即预期的方法和表达式运算符），函数就能处理它们；
    - 从宏观上来说，Python为对象接口object interfaces编程，不是为数据类型编程。


### 三、A Second Example: Intersecting Sequences
1. 定义
    - 将代码封装**package**（或者**wrap**）在函数中：
    ``` python
    def intersect(seq1, seq2):
        res = [] 
        for x in seq1: 
            if x in seq2: 
                res.append(x) 
        return res
    ```

2. 调用
    ``` python
    s1 = "SPAM"
    s2 = "SCAM"
    print(intersect(s1, s2))
    ```

3. 多态
    - 对于intersect函数，只要第一个参数支持for循环，第二个参数支持in成员测试，就能正常工作：
    ``` python
    x = intersect([1, 2, 3], (1, 4))
    print(x)
    ```

4. **局部变量Local Variables**
    - intersect函数中的res变量在Python中被称为局部变量，仅在函数运行时存在；
    - 在函数内部进行赋值的变量名都默认为**局部变量**；
    - res被赋值过，所以是局部变量；参数也是通过赋值被传入的，所以seq1和seq2也是局部变量；for循环中的变量x也是局部变量；
    - 所有的局部变量在函数调用时出现，在函数退出时消失。详见第17章。


## chapter 17 Scopes
### 一、Python Scope Basics
1. Python作用域基础
    - Python创建、改变或查找变量名都是在所谓的**命名空间namespace**中进行的，**作用域scope**就是命名空间；
    - 在代码中给一个变量赋值的地方决定了这个变量将存在于哪个命名空间；
    - 一个函数内赋值的所有变量名都与该函数的命名空间相关联；
    - 在def内赋值的变量名与在def外赋值的变量名不冲突，即使是相同的变量名。
    - 变量可以在3个不同的地方被赋值，分别对应3种不同的作用域：
        - If a variable is assigned inside a def, it is **local** to that function.
        - If a variable is assigned in an enclosing def, it is **nonlocal** to nested functions.
        - If a variable is assigned outside all defs, it is **global** to the entire file.
    - 我们将其称为语义作用域lexical scoping，因为变量的作用域由源代码的位置决定，而不是由函数调用决定：
    ``` python
    X = 99 # Global (module) scope X
    def func():
        X = 88 # Local (function) scope X: a different variable
    ```

2. 作用域细节  
函数提供了嵌套的命名空间（作用域），使其内部使用的变量名局部化。而模块定义了全局作用域。

3. 变量名解析Name Resolution：**LEGB机制**
    - 在默认情况下，变量名赋值会创建或改变局部变量；
    - 变量名引用至多在4种作用域内查找：*先局部local，再外层的函数enclosing functions，再全局global，最后内置built-in（这四个就是LEGB）*；
    - 使用`global`和`nonlocal`语句声明的名称将赋值的变量名分别映射到外围的模块和函数的作用域；
    - 所有在函数def语句内赋值的变量名默认均为局部变量。函数能够任意使用在外层函数内或全局作用域中的变量名，但必须声明为非局部变量和全局变量来改变其属性。

4. 作用域实例
    ``` python
    X = 99
    def func(Y):
        Z = X + Y
        return Z
    print(func(1))
    ```
    - 全局变量名：X，func；局部变量名：Y，Z。
    - 当函数调用结束时，局部变量会从内存中移除。

5. 内置作用域The Built-in Scope
    - 内置作用域仅仅是一个名为`builtins`的内置模块，要导入builtins才能使用内置作用域：
    ``` python
    import builtins
    print(dir(builtins))
    ```
    - 这个列表中的变量名组成了python中的内置作用域。前一半是内置的异常，后一半是内置函数；
    - Python会在LEGB查找中的最后自动查找这个模块；
    - 因此有2种方式引用一个内置函数：利用LEGB法则，或者手动导入builtins模块：
    ``` python
    print(zip)
    import builtins
    print(builtins.zip)
    print(zip is builtins.zip)
    ```

6. 不要重定义内置名称
    - 由于LEGB查找的流程，会使它在第一处找到变量名的地方生效。
    - 在局部作用域中的变量名可能会覆盖在全局作用域和内置作用域中有着相同变量名的变量，而全局变量名可能会覆盖内置的变量名：
    ``` python
    def hider():
        open = 'spam'
        open('data.txt')
    # 这样的话，open在函数内就不能打开文件了
    ```


### 二、The global Statement
1. global语句
    - 全局变量是在外层模块文件的顶层被赋值的变量名；
    - 全局变量如果是在函数内被赋值的话，必须经过声明；
    - 全局变量名在函数的内部不经过声明也可以被引用。
    - global允许我们修改一个def外的模块文件顶层的名称；nonlocal适用于外层def的局部作用域内的名称：
    ``` python
    X = 88
    def func():
        global X
        X = 99 # Global X: outside def
    func()
    print(X)

    y, z = 1, 2
    def all_global():
        global x
        x = y + z
    # 上面x、y、z都是all_global函数内的全局变量
    ```

2. 在Python中使用多线程Multithreading进行并行计算的程序也要依赖全局变量
    - 因为全局变量在并行线程中在不同的函数之间成为共享内存；
    - 多线程与程序的其余部分并行地执行函数调用，由Python的标准库模块_thread、threading和queue提供支持。多线程不在本书探讨内容，详见其他书籍。

3. 其他访问全局变量的方法
    - 一个模块文件的全局作用域一旦被导入就成了这个模块对象的一个属性命名空间：
    ``` python
    # thismod.py
    var = 99

    def local():
        var = 0 # 局部变量，不改变全局的var

    def glob1():
        global var
        var += 1

    def glob2():
        var = 0 # 局部变量，不改变全局的var
        import thismod # 导入自己
        thismod.var += 1 # 改变全局变量var

    def glob3():
        var = 0
        import sys
        glob = sys.modules['thismod'] # 得到模组对象
        glob.var += 1 # 改变全局变量var

    def test():
        print(var)
        local(); glob1(); glob2(); glob3()
        print(var)
    ```
    - 可以通过导入外层模块并对其属性进行赋值来模拟global语句，见thismod.py的glob2()、glob3()函数：
    ``` python
    import thismod
    thismod.test()
    print(thismod.var)
    ```


### 三、Scopes and Nested Functions
1. 嵌套作用域Nested Scope
    - 接下来深入学习LEGB查找规则中的E，即任意外层函数局部作用域的形式：
    ``` python
    X = 99
    def f1():
        X = 88
        def f2():
            print(X)
        f2() # f2是一个临时函数，仅在f1内部执行的过程中存在
    f1()

    def f1():
        X = 88
        def f2():
            print(X)
        return f2 # Return f2 but don't call it
    action = f1()
    action()
    ```
    - 对action名称的调用本质上运行了f1运行时我们命名为f2的函数；
    - 因为Python中的函数与其他一切一样是对象，因此可以作为其他函数的返回值传递回来；
    - f2也记住了f1的嵌套作用域中的X，尽管f1已经不处于激活状态，详见下面。

2. **工厂函数Factory Functions：闭包Closures**
    - 这种代码行为可以叫做**Closures**，也可以**Factory Functions**；
    - 即函数对象能够记忆外层作用域里的值，不管嵌套作用域是否还在内存中存在：
    ``` python
    def maker(N):
        def action(X): 
            return X ** N 
        return action
    ```
    - 上面定义了一个外层函数，用来生成并返回一个嵌套的函数，却不调用这个内嵌的函数；
    - maker创造出action，但只是简单地返回action而不执行它。
    ``` python
    f = maker(2) # 调用maker函数，执行maker内的代码，创建了action函数，并返回action函数
    print(f)
    ```
    - f是生成的内嵌函数即action的一个引用，因为是return action，maker函数创建并返回action函数。
    ``` python
    print(f(3)) # f(3)调用action函数，将3传递进X，返回3**2
    print(f(4))
    ```
    - 上面调用了maker函数创建的并传回的一个内嵌函数。这里不平常的地方在于，在调用action时，maker已经退出，但是内嵌的函数记住了整数2。实际上，在外层嵌套局部作用域内的N被作为执行的状态信息保留了下来，并附加到生成的action函数上。如果再调用外层的函数，可以得到一个新的有不同状态信息的嵌套函数
    ``` python
    g = maker(3)
    print(g(4))
    print(f(4))
    ```
    - 名称g的函数记住了3，名称f的函数记住了2；每个函数都有自己的状态信息state information。
    - 上述是一种相对高级的技术，在代码中不太常见；另外嵌套作用域常常被lambda函数创建表达式利用：
    ``` python
    def maker(N):
        return lambda X: X ** N
    h = maker(3)
    print(h(4))
    ```

3. 闭包vs类
    - 类是一个更好的实现这种状态记忆的选择，因为它们用属性赋值来更加显式地创建它们的内存；
    - 当记忆状态是唯一目标时，闭包函数经常提供一个轻量级的可行的替代方案；
    - 它为每一次调用提供局部化存储空间，来存储一个单独的内嵌函数所需的数据；
    - nonlocal语句允许嵌套作用域状态改变；
    - 当class嵌套在def中，闭包也可以被创建，详见第29章关于嵌套类的说明。

4. 使用默认值参数defaults,来保存外层作用域的状态
    ``` python
    def f1():
        x = 88
        def f2(x=x):
            print(x)
        f2()
    f1()
    ```
    - x=x意味着参数x会默认使用外层作用域中x的值，由于第二个x在python进入内嵌的def之前就已经完成其求值，仍引用f1中的x；实际上嵌套作用域查找规则之所以加入到python中就是为了让默认值参数不再扮演这种角色；所以无需x=x，因为python会自动记住所需要的外层作用域的任意值，从而在内嵌的def中使用。
    - 避免在def中嵌套def，会让程序更简单：
    ``` python
    def f1():
        x = 88
        f2(x)

    def f2(x):
        print(x)

    f1()
    ```

5. 嵌套作用域，lambda
    - `lambda`表达式也为其创建的函数引入新的局部作用域：
    ``` python
    def func():
        x = 4
        action = (lambda n: x ** n)
        return action
    x = func()
    print(x(2))
    ```

6. 循环变量可能需要默认值参数，而不是作用域
    - 如果在函数中定义的lambda或者def嵌套在一个循环之中，而这个内嵌函数又引用了一个外层作用域的变量，该变量被循环所改变，那么所有在这个循环中产生的函数会有相同的值，也就是最后一次循环中完成时被引用变量的值。在这种情形下，需要使用默认值参数来保存变量的当前值：
    ``` python
    def makeActions():
        acts = []
        for i in range(5): # Tries to remember each i
            acts.append(lambda x: i ** x)
        return acts

    T = makeActions()
    print(T[0])
    ```
    运行结果如下为`<function makeActions.<locals>.<lambda> at 0x0000020EEE2D9F30>`
    - 上面的代码会出问题：因为外层作用域中的变量在嵌套的函数被调用时才进行查找，而此时i=4。所以它们实际上记住的是同样的值，也就是最后一次循环迭代中循环变量的值。所以当下面的所有调用传入底数2时，列表中每个结果都是2的4次方：
    ``` python
    print(T[0](2)) # 因为acts列表里面都是lambda函数，所以要先索引再传递参数
    print(T[1](2))
    print(T[2](2))
    print(T[3](2))
    print(T[4](2))
    ```
    运行结果如下：
    ``` python
    0
    1
    4
    9
    16
    ```
    - 为了让这类代码能够工作，必须使用默认值参数来传入当前外层作用域中的值。因为默认值参数的求值是在嵌套函数创建时就发生的（而不是该函数之后被调用时），所以每一个函数都记住了属于自己的变量i：
    ``` python
    def makeActions():
        acts = []
        for i in range(5):
            acts.append(lambda x, i=i: i ** x)
        return acts

    T = makeActions()
    print(T[0](2))
    print(T[1](2))
    print(T[2](2))
    print(T[3](2))
    print(T[4](2))
    ```
    - 更详细见第18章对默认值参数和第19章对lambda的介绍。


### 四、The nonlocal Statement
1. nonlocal语句
    - `nonlocal`语句可以使内嵌的def对外层函数中的名称进行读取和写入访问；
    - `nonlocal`语句通过提供可改写的状态信息，让嵌套作用域闭包变得更加有用；
    - 声明nonlocal名称的时候，必须以及存在于该外层函数的作用域中，不能由内嵌def中的第一次赋值来创建。

2. nonlocal基础
    - `nonlocal`语句除了允许修改外层def中的名称外，还会强制引用的发起。nonlocal使得对该语句列出的名称的查找从外层的def的作用域开始，而不是从该函数的局部作用域开始。当执行到nonlocal语句时，nonlocal中列出的名称必须在一个外层的def中被提前定义过，否则将引发一个错误；
    - nonlocal将作用域查找限制为只在外层的def中，不会继续进入到全局或内置作用域。

3. nonlocal应用
    - 下面代码中，tester创建并返回函数nested以便随后调用，而nested中的state引用遵从常规的作用域查找规则：
    ``` python
    def tester(start):
        state = start
        def nested(label):
            print(label, state)
        return nested

    F = tester(0)
    F('spam')
    F('ham')
    ```
    运行结果如下：
    ``` python
    spam 0
    ham 0
    ```
    - 使用nonlocal进行修改，即使通过名称F调用返回的nested函数时，tester已经返回并退出了，这也是有效的：
    ``` python
    def tester(start):
        state = start
        def nested(label):
            nonlocal state # state，即nonlocal名称必须在外层def作用域中被赋值过，否则会得到一个错误。在全局作用域被赋值也会错误。
            print(label, state)
            state += 1
        return nested

    F = tester(0)
    F('spam')
    F('ham')
    F('eggs')
    ```
    运行结果如下：
    ``` python
    spam 0
    ham 1
    eggs 2
    ```
    - 可以多次调用tester工厂（闭包）函数，以便在内存中获得其状态的多个副本：
    ``` python
    G = tester(42)
    G('spam')
    G('eggs')
    F('bacon')
    ```


### 五、Why nonlocal? - State Retention Options
1. 为什么选nonlocal  
nonlocal增强了对外层作用域的引用：允许在内存中保持可更改状态的多个副本。

2. 全局变量的状态：只有一个副本
    ``` python
    def tester(start):
        global state 
        state = start 
        def nested(label):
            global state
            print(label, state)
            state += 1
        return nested
    
    F = tester(0)
    F('spam')
    F('eggs')
    ```
    运行结果如下：
    ``` python
    spam 0
    eggs 1
    ```
    - 上述代码可能会引起全局作用域的名称冲突，并且只允许模块作用域中保存状态信息的单个共享副本；
    - 如果再次调用tester，将会重置模块的state变量，而先前的调用的state会被覆盖：
    ``` python
    G = tester(42)
    G('toast')
    G('bacon')
    F('ham')
    ```
    运行结果如下：
    ``` python
    toast 42
    bacon 43
    ham 44
    ```

3. 类的状态：显式属性（预习）
    - 同嵌套函数和nonlocal一样，类支持所保存的数据存在多个副本：
    ``` python
    class tester:
        def __init__(self, start):
            self.state = start
        def nested(self, label):
            print(label, self.state)
            self.state += 1

    F = tester(0)
    F.nested('spam')
    F.nested('ham')

    G = tester(42)
    G.nested('toast')
    G.nested('bacon')
    F.nested('eggs')
    print(F.state)
    ```
    - 预习：运算符重载把类对象用作可调用函数。__call__拦截了一个实例上的直接调用，因此无需调用方法（详见第30章）：
    ``` python
    class tester:
        def __init__(self, start):
            self.state = start
        def __call__(self, label): 
            print(label, self.state) 
            self.state += 1
    
    H = tester(99)
    H('juice')
    H('pancakes')
    ```

4. **函数属性Function Attributes**的状态
    - 可以使用函数属性来达到与nonlocal相同的效果。函数属性允许状态变量从内嵌函数的外部被访问，就像类属性那样（内嵌函数的属性必须在内嵌的def之后初始化）：
    ``` python
    def tester(start):
        def nested(label):
            print(label, nested.state)
            nested.state += 1
        nested.state = start # 因为要先定义内嵌函数nested()，否则nested.state的nested函数就没有来源
        return nested

    F = tester(0)
    F('spam')
    F('ham')
    print(F.state)
    ```
    - 也支持调用多个副本：
    ``` python
    G = tester(42)
    G('eggs')
    F('ham')
    print(G.state)
    print(F is G)
    ```
    - 函数属性详见第19章。

5. 可变对象State with mutables的状态
    ``` python
    def tester(start):
        def nested(label):
            print(label, state[0])
            state[0] += 1
        state = [start]
        return nested
    ```
    - 这里利用了列表的可变性，而且与函数属性一样依赖于原位置对象修改不会将一个名称归类为局部。


## chapter 18 Arguments
### 一、Argument-Passing Basics
1. 参数传递基础
    - 参数的传递是通过自动将对象赋值给局部变量名来实现，参数都是通过指针传入的；
    - 在函数内部赋值参数名不会影响调用者作用域的变量；
    - 改变函数的可变对象也许会对调用者有影响。

2. 参数和共享引用
    ``` python
    def f(a):
        a = 99
        print(a)
    b = 88
    f(b)
    print(b)
    ```
    - 上述，在f(b)调用函数的时候，变量a被赋值了对象88；
    - 但是a只存在于被调用的函数之中，在被调函数中修改a（即a=99）对于主动调用函数的地方没有影响。

3. 可变对象的原位置修改
    ``` python
    def changer(a, b):
        a = 2
        b[0] = 'spam'
        print(a, b)
    X = 1
    L = [1, 2]
    changer(X, L)
    print(X, L)
    ```
    - L和b引用了相同对象。

4. 避免修改可变参数
    - 可以通过`list.copy`或者无参数切片，来复制列表，创建一个副本：
    ``` python
    L = [1, 2]
    changer(X, L[:])
    print(X, L)
    # 或者在函数内部复制
    def changer(a, b):
        b = b[:]
        a = 2
        b[0] = 'spam'
    ```

### 二、Special Argument-Matching Modes
1. 特殊的参数匹配模式  
默认情况下，参数按照从左到右的位置进行匹配。也可以通过*形式参数名*、*提供默认值的参数值*和*对额外参数使用容器collectors*三种方法来指定匹配。

2. 参数匹配基础
    - **位置参数Positionals**：从左到右；
    - **关键字参数Keywords**：通过参数名进行匹配；
    - **默认值参数Defaults**；
    - **可变长参数Varargs**收集：收集任意多的基于位置或关键字的参数：*或**开头的特殊参数；
    - **可变长参数Varargs解包unpacking**：传入任意多的基于位置或关键字的参数。

3. 参数匹配语法

| Syntax | Location | Interpretation |
| :---- | :---- | :---- |
| func(value) | 调用 | 常规参数：位置匹配 |
| func(name=value) | 调用 | 关键字参数：名称匹配 | 
| func(*iterable) | 调用 | 将iterable中所有对象作为独立的基于位置的参数传入 |
| func(**dict) | 调用 | 将dict中所有的键/值对作为独立的关键字参数传入 |
| def func(name) | 函数定义 | 常规参数：位置或名称匹配 |
| def func(name=value) | 函数定义 | 默认值参数 |
| def func(*name) | 函数定义 | 将剩下的基于位置的参数匹配并收集到一个元组中 |
| def func(**name) | 函数定义 | 将剩下的关键字参数匹配并收集到一个字典中 |
| def func(*other, name) | 函数定义 | 在调用中必须通过关键字传入的参数 |
| def func(*, name=value) | 函数定义 | 在调用中必须通过关键字传入的参数 |

4. 更深入的细节
    - 如果组合使用特殊参数匹配模式，需遵循下面顺序规则：
        - *函数调用的参数顺序：位置参数；关键字参数；*iterable形式的组合；**dict形式*；
        - *函数定义的参数顺序：一般参数；默认值参数；*name；keyword-only参数；**name*。
    - Python内部大致是使用以下的步骤来赋值前匹配参数的：
        - 通过位置分配物关键字参数；
        - 通过匹配名称分配关键字参数；
        - 将剩下的非关键字参数分配到*name元组中；
        - 将剩下的关键字参数分配到**name字典中；
        - 把默认值分配给在头部未得到匹配的参数。
    - 函数头部可以有个*注解值annotation values*，其指定形式为name:value，详见第19章函数注解。

5. 关键字参数和默认值参数的示例
    - 位置参数：
    ``` python
    def f(a, b, c): print(a, b, c)
    f(1, 2, 3)
    ```
    - 关键字参数Keywords：
    ``` python
    f(c=3, b=2, a=1)
    ```
    - 混用位置参数和关键字参数：
    ``` python
    f(1, c=3, b=2) # 只能先位置参数再关键字参数
    ```
    - 默认值参数Defaults：
    ``` python
    def f(a, b=2, c=3): print(a, b, c)
    f(1)
    f(a=1)
    ```
    - 当函数传递2个值时，只有c得到默认值，当且仅当3个值传递时，不会使用默认值：
    ``` python
    f(1, 4)
    f(1, 4, 5)
    ```
    - 关键字参数和默认值参数混用：
    ``` python
    f(1, c=6)
    ```

6. 混合使用关键字参数和默认值参数
    ``` python
    def func(spam, eggs, toast=0, ham=0):
        print((spam, eggs, toast, ham))

    func(1, 2)
    func(1, ham=1, eggs=0) 
    func(spam=1, eggs=0) 
    func(toast=1, eggs=2, spam=3) 
    func(1, 2, 3, 4)
    ```
    - 如果默认值参数是一个可变对象（比如def f(a=[])）。这个参数会保留上次调用的值，详见第21章陷阱。

7. 可变长参数Arbitrary Arguments的实例
    - **\*** 和 **\*\*** 旨在让函数支持接受任意多的参数：
    - (1) 函数定义：收集参数
    ``` python
    def f(*args): print(args)
    # "*"将所有基于位置的参数收集到新的元组
    f()
    f(1)
    f(1, 2, 3, 4)
    # "**"将关键字参数收集到一个新的字典中
    def f(**args): print(args)
    f()
    f(a=1, b=2)
    # 混用"*"和"**"
    def f(a, *pargs, **kargs): print(a, pargs, kargs)
    f(1, 2, 3, x=1, y=2)
    ```
    运行结果如下：
    ``` python
    ()
    (1,)
    (1, 2, 3, 4)
    {}
    {'a': 1, 'b': 2}
    1 (2, 3) {'x': 1, 'y': 2}
    ```
    - (2) 函数调用：解包参数
    ``` python
    def func(a, b, c, d): print(a, b, c, d)
    args = (1, 2)
    args += (3, 4)
    func(*args) # Same as func(1, 2, 3, 4) 解包参数
    # 上面不能直接func(args) 因为它会以为整个（1，2，3，4）都是a，就没有b、c、d了
    args = {'a': 1, 'b': 2, 'c': 3}
    args['d'] = 4
    func(**args) # Same as func(a=1, b=2, c=3, d=4) 解包键值对
    # 混用
    func(*(1, 2), **{'d': 4, 'c': 3})
    func(1, *(2, 3), **{'d': 4})
    func(1, c=3, *(2,), **{'d': 4})
    func(1, *(2,), c=3, **{'d':4})
    ```
    运行结果如下：
    ``` python
    1 2 3 4
    1 2 3 4
    1 2 3 4
    1 2 3 4
    1 2 3 4
    1 2 3 4
    ```
    - 注意1，在调用时的\*pargs形式是一个迭代上下文，接受迭代工具，解包参数；但在头部中的\*pargs，只是将额外参数绑定到一个元组；
    - 注意2，**扩展序列解包**（注意区分）赋值创建列表而非元组：
    ``` python
    x, *y = 'spam'
    print(x, y)
    ```
    运行结果如下：
    ``` python
    s ['p', 'a', 'm']
    ```

8. keyword-only参数
    - `keyword-only参数`为出现在*args之后的参数，必须使用关键字语法调用：
    ``` python
    def kwonly(a, *b, c): print(a, b, c) # 这里的c因为在*b后面，所以只能使用关键字参数
    kwonly(1, 2, c=3)
    ```
    - *可以在参数列表中使用一个"*"字符，使之后的参数只能作为关键字参数传入*：
    ``` python
    def kwonly(a, *, b, c): print(a, b, c)
    kwonly(1, c=3, b=2)
    kwonly(c=3, b=2, a=1)
    ```

9. 顺序规则
    - 有名参数不能出现在\*\*args的后面，\*\*也不能单独出现在参数列表中，这2种做法都会产生语法错误；
    - 也就是*顺序必须是参数（位置或关键字），\*args，keyword-only参数，\*\*args*：
    ``` python
    def f(a, *b, c=6, **d): print(a, b, c, d)
    f(1, 2, 3, x=4, y=5)
    f(1, 2, 3, x=4, y=5, c=7)
    f(1, 2, 3, c=7, x=4, y=5)
    def f(a, c=6, *b, **d): print(a, b, c, d)
    f(1, 2, 3, x=4)
    ```
    - 在函数调用中也是类似的情况：
    ``` python
    def f(a, *b, c=6, **d): print(a, b, c, d)
    f(1, *(2, 3), **dict(x=4, y=5))
    f(1, *(2, 3), c=7, **dict(x=4, y=5))
    f(1, c=7, *(2, 3), **dict(x=4, y=5))
    f(1, *(2, 3), **dict(x=4, y=5, c=7))
    ```


## chapter 19 Advanced Function Topics
### 一、Function Design Concepts
1. 函数设计概念
    - 函数的内聚性cohesion：将任务分解成有目的性的函数；
    - 函数的耦合性coupling：函数之间相互通信；
    - guidelines：
        - 耦合性：在输入时使用参数，输出时使用return语句；
        - 耦合性：只在真正必要的情况下使用全局变量；
        - 耦合性：不要改变可变类型的参数，除非调用者希望这么做；
        - 内聚性：每一个函数都应该有一个单一的、统一的目标；
        - 大小：每一个函数应该相对较小；
        - 耦合性：避免直接改变其他模块文件中的变量。

### 二、Recursive Functions
1. 用递归求和（**Recursive Functions递归函数**）
    - 当以这种方式使用递归的时候，对于函数调用的每一个打开的层级来说，在运行时的调用栈上都有自己的一个函数局部作用域的副本，每个层级的L都是不同的：
    ``` python
    def mysum(L):
        print(L)
        if not L:
            return 0
        else:
            return L[0] + mysum(L[1:])

    print(mysum([1, 2, 3, 4, 5]))
    ```
    - 将上面函数分为2个：
    ``` python
    def mysum(L):
        if not L: return 0
        return nonempty(L)
    
    def nonempty(L):
        return L[0] + mysum(L[1:])

    print(mysum([1.1, 2.2, 3.3, 4.4]))
    ```

2. 循环vs递归
    - Python强调循环这样的简单过程式语句，循环语句更为自然；
    - while语句：
    ``` python
    L = [1, 2, 3, 4, 5]
    sum = 0
    while L:
        sum += L[0]
        L = L[1:]
    print(sum)
    ```
    - for循环：
    ``` python
    L = [1, 2, 3, 4, 5]
    sum = 0
    for x in L: sum += x
    print(sum)
    ```
    - 有了循环语句，就不需要在调用栈上为每次迭代都保留一个局部作用域的副本，并避免相关的开销（详见第21章计时器）。

3. 处理任意结构
    - 递归能够遍历任意结构，比如嵌套子列表结构：[1, [2, [3, 4], 5], 6, [7, 8]]
    ``` python
    def sumtree(L):
        tot = 0
        for x in L:
            if not isinstance(x, list):
                tot += x
            else:
                tot += sumtree(x)
        return tot

    L = [1, [2, [3, 4], 5], 6, [7, 8]]
    print(sumtree(L))
    ```

4. 递归Recursion vs 队列queue和栈stacks
    - 在内部，Python通过每一次递归调用时把信息压入调用栈来实现递归；
    - 实际上，不使用递归调用而实现递归风格的过程式编程是可能的；
    - 例1：
    ``` python
    def sumtree(L):
        tot = 0
        items = list(L)
        while items:
            print(items)
            front = items.pop(0)
            if not isinstance(front, list):
                tot += front
            else:
                items.extend(front) # extend方法可处理可迭代器，详见8-2和14-3，注意extend和append方法的不同
        return tot
    L = [1, [2, [3, 4], 5], 6, [7, 8]]
    print(sumtree(L))
    ```
    - 上述代码采用**广度优先**的方式**breadth-first fashion**遍历了列表，形成了一个**先进先出的队列first-in-first-out queue**。
    - 例2：
    ``` python
    def sumtree(L):
        tot = 0
        items = list(L)
        while items:
            print(items)
            front = items.pop(0)
            if not isinstance(front, list):
                tot += front
            else:
                items[:0] = front
        return tot
    L = [1, [2, [3, 4], 5], 6, [7, 8]]
    print(sumtree(L))
    ```
    - 上述代码执行**深度优先**遍历**depth-first traversal**，形成**后进先出的栈last-in-first-out stack**。

5. 标准python限制了运行时调用栈的深度
    - 可以使用sys模块来扩大这一上限：
    ``` python
    import sys
    print(sys.getrecursionlimit())
    sys.setrecursionlimit(10000)
    print(sys.getrecursionlimit())
    ```

### 三、Function Objects: Attributes and Annotations
1. 函数本身是对象，存储在内存块里，也支持**属性attribute**存储和**注解annotation**

2. 间接函数调用：“一等”对象
    - 函数对象可以赋值给其他的名称、传递给其他函数、嵌入到数据结构中、从一个函数返回给另一个函数；
    - 函数和其他对象一样，属于一个通用类别，被称为first-class object model。
    - 函数赋值给其他的名称：
    ``` python
    def echo(message):
        print(message)
    x = echo
    x('Indirect call!')
    ```
    - 函数作为参数传入其他函数：
    ``` python
    def indirect(func, arg):
        func(arg)
    indirect(echo, 'Argument call!')
    ```
    - 函数对象存入数据结构：
    ``` python
    schedule = [ (echo, 'Spam!'), (echo, 'Ham!') ]
    for (func, arg) in schedule:
        func(arg)
    ```
    - 闭包closure：
    ``` python
    def make(label):
        def echo(message):
            print(label + ':' + message)
        return echo

    F = make('Spam')
    F('Ham!')
    F('Eggs!')
    ```

3. 函数自省Function Introspection
    - **自省工具Introspection tools**允许我们探索实现细节，函数已经附加了代码对象，代码对象提供了函数局部变量和参数等方面的细节：
    ``` python
    def func(a):
        b = 'spam'
        return b * a
    print(func.__code__)
    print(dir(func.__code__))
    print(func.__code__.co_varnames)
    print(func.__code__.co_argcount)
    ```
    - 编写者可以利用这些信息来管理函数。

4. 函数属性Function Attributes
    - 函数对象除了系统定义的属性，还可以*附加任意用户定义的属性*：
    ``` python
    print(func)
    func.count = 0
    func.count += 1
    print(func.count)
    func.handles = 'Button-Press'
    print(func.handles)
    print(dir(func)) # 属性多了count和handles
    ```
    - 这些属性可以直接把状态信息附加到函数对象，而不必使用全局、非局部和类等技术。

5. 函数注解Function Annotations
    - **函数注解**编写在def头部行，对于参数，注解在参数的冒号后；对于返回值，在->后：
    ``` python
    def func(a: float, b: float, c: float) -> int:
        return a + b + c
    print(func(1, 2, 3))
    print(func.__annotations__)
    ```
    - 编写了注解仍然可以对参数使用默认值：
    ``` python
    def func(a: float = 4, b: float = 5, c: float = 6) -> int:
        return a + b + c
    ```
    - 注解只在def语句有效，对lambda表达式无效。

### 四、Anonymous Functions: lambda
1. lambda表达式基础
    - `lambda argument1, argument2,... argumentN : expression using arguments`
    - lambda是一个表达式，而不是语句，可以选择性地被赋值给一个变量名；
    - lambda的主体是一个单独的表达式，而不是代码块；
    - 可以使用lambda表达式，通过显式地将结果赋值给一个变量名，用变量名调用函数：
    ``` python
    f = lambda x, y, z: x + y + z
    print(f(2, 3, 4))
    ```
    - lambda也可以使用默认参数：
    ``` python
    x = (lambda a="fee", b="fie", c="foe": a + b + c)
    print(x("wee"))
    ```
    - lambda的代码与def一样都遵循LEGB规则：
    ``` python
    def knights():
        title = 'Sir'
        action = (lambda x: title + ' ' + x)
        return action
    act = knights()
    msg = act('robin')
    print(msg)
    print(act)
    ```

2. 为什么使用lambda
    - lambda起到一个函数速写的作用：
    ``` python
    L = [lambda x: x ** 2,
        lambda x: x ** 3,
        lambda x: x ** 4]

    for f in L:
        print(f(2))
    print(L[0](3))
    ```

3. 多分支switch语句
    - 用字典或其他数据结构构建动作表：
    ``` python
    key = 'got'
    print({'already': (lambda: 2 + 2),
        'got': (lambda: 2 * 4),
        'one': (lambda: 2 ** 6)}[key]())
    ```
    - 不使用lambda，用def：
    ``` python
    def f1(): return 2 + 2
    def f2(): return 2 * 4
    def f3(): return 2 ** 6
    key = 'one'
    print({'already': f1, 'got': f2, 'one': f3}[key]())
    ```

4. lambda中嵌套选择逻辑或执行循环
    ``` python
    lower = (lambda x, y: x if x < y else y)
    print(lower('bb', 'aa'))

    import sys
    showall = lambda x: list(map(sys.stdout.write, x))
    t = showall(['spam\n', 'toast\n', 'eggs\n'])

    showall = lambda x: [sys.stdout.write(line) for line in x]
    t = showall(('bright\n', 'side\n', 'of\n', 'life\n'))
    ```

5. 作用域：lambda也能嵌套
    ``` python
    action = (lambda x: (lambda y: x + y))
    act = action(99) # lambda先获取变量x
    print(act(3))
    print(((lambda x: (lambda y: x + y))(99))(4))
    ```

6. lambda回调（Callbacks）
    - 你到一个商店买东西，刚好你要的东西没有货，于是你在店员那里留下了你的电话，过了几天店里有货了，店员就打了你的电话，然后你接到电话后就到店里去取了货。在这个例子里，你的电话号码就叫回调函数，你把电话留给店员就叫登记回调函数，店里后来有货了叫做触发了回调关联的事件，店员给你打电话叫做调用回调函数，你到店里去取货叫做响应回调事件。
    - Python的tkinker GUI API定义内联的回调函数
    - 例如：下面的代码创建了一个按钮，按钮按下会打印一行消息：
    ``` python
    import sys
    from tkinter import Button, mainloop
    x = Button(
        text='Press me',
        command=(lambda: sys.stdout.write('Spam\n')))
    x.pack()
    mainloop()
    ```
    - 回调处理器通过传递一个lambda函数作为command的关键字参数来注册的。

### 五、Functional Programming Tools
1. 函数式编程工具
    - Python混合支持多种编程范式：过程式procedural（使用基础语句）；面向对象式object-oriented（使用类）；函数式functional；
    - Python提供了一整套进行函数式编程的内置工具，把函数作用于序列和其他可迭代对象：
    - 比如`map`：在可迭代对象的各项上调用函数的工具；`filter`：使用一个测试函数来过滤项；`reduce`：把函数作用在成对的项上来允许结果。

2. 在可迭代对象上映射函数：map
    ``` python
    counters = [1, 2, 3, 4]
    def inc(x): return x + 10
    print(list(map(inc, counters)))
    print(list(map((lambda x: x + 10), counters)))
    ```
    - 编写自己的映射工具：
    ``` python
    def mymap(func, seq):
        res = []
        for x in seq: res.append(func(x))
        return res

    print(mymap(inc, [1, 2, 3]))
    ```
    - map可用于多个序列：
    ``` python
    print(list(map(pow, [1, 2, 3], [2, 3, 4])))
    ```

3. 选择可迭代对象中的元素：filter
    - filter也返回可迭代对象：
    ``` python
    print(list(filter((lambda x: x > 0), range(-5, 5))))
    ```

4. 合并可迭代对象中的元素：reduce
    - reduce位于functools模块，接受并处理一个迭代器，返回一个结果（非迭代器）：
    ``` python
    from functools import reduce
    print(reduce((lambda x, y: x + y), [1, 2, 3, 4]))
    print(reduce((lambda x, y: x * y), [1, 2, 3, 4]))
    ```
    - 用for循环模拟reduce：
    ``` python
    L = [1,2,3,4]
    res = L[0]
    for x in L[1:]:
        res = res + x
    print(res)
    ```
    - 编写自己的reduce函数：
    ``` python
    def myreduce(function, sequence):
        tally = sequence[0]
        for next in sequence[1:]:
            tally = function(tally, next)
        return tally

    print(myreduce((lambda x, y: x + y), [1, 2, 3, 4, 5]))
    print(myreduce((lambda x, y: x * y), [1, 2, 3, 4, 5]))
    ```
    - 内置的operator模块，提供了内置表达式对应的函数：
    ``` python
    import operator, functools
    print(functools.reduce(operator.add, [2, 4, 6]))
    print(functools.reduce((lambda x, y: x + y), [2, 4, 6]))
    ```


## chapter 20 Comprehensions and Generations
### 一、List Comprehensions and Functional Tools
1. list comprehensions apply an arbitrary expression to items in an iterable, rather than applying a function. 

2. List Comprehensions Versus map
    - 用循环收集字符串中字符的ASCII编码：
    ``` python
    res = []
    for x in 'spam':
        res.append(ord(x))
    print(res)
    ```
    - 用`map函数`：
    ``` python
    res = list(map(ord, 'spam'))
    print(res)
    ```
    - **列表推导表达式 list comprehension expression**：
    ``` python
    res = [ord(x) for x in 'spam']
    print(res)
    ```

3. Adding Tests and Nested Loops: filter
    ``` python
    print(list(filter((lambda x: x % 2 == 0), range(5))))
    print([x for x in range(5) if x % 2 == 0])
    ```
    - 等效的for循环：
    ``` python
    res = []
    for x in range(5):
        if x % 2 == 0:
            res.append(x)
    print(res)
    ```

4. Formal comprehension syntax标准推导语法
    - 通用的列表推导的结构如下：
    ``` python
    [ expression for target1 in iterable1 if condition1
                 for target2 in iterable2 if condition2 ...
                 for targetN in iterableN if conditionN ]
    ```
    - 例子：
    ``` python
    res = [x + y for x in [0, 1, 2] for y in [100, 200, 300]]
    print(res)

    print([x + y + z for x in 'spam' if x in 'sm'
                     for y in 'SPAM' if y in ('P', 'A')
                     for z in '123' if z > '1'])
    ```

5. List Comprehensions and Matrixes列表推导和矩阵
    ``` python
    M = [[1, 2, 3],
         [4, 5, 6],
         [7, 8, 9]]
    N = [[2, 2, 2],
         [3, 3, 3],
         [4, 4, 4]]

    print([row[1] for row in M])
    print([M[row][1] for row in (0, 1, 2)])
    print([M[i][i] for i in range(len(M))])
    print([M[i][len(M)-1-i] for i in range(len(M))])
    print([[col + 10 for col in row] for row in M])
    print([[M[row][col] * N[row][col] for col in range(3)] for row in range(3)]) # 这个语句是先for row再for col
    print([[col1 * col2 for (col1, col2) in zip(row1, row2)] for (row1, row2) in zip(M, N)])
    ```

6. 性能
    - `map调用比等效的for循环要快2倍，而列表推导往往比map调用要再快一些`；
    - map和列表推导是以C语言的速度来运行，而for循环在虚拟机PVM中运行。

### 二、Generator Functions
1. Generator Functions and Expressions 生成器函数和生成器表达式
    - 下面2种语言特性让用户可以推迟结果的计算：
        - **Generator functions生成器函数**：使用def编写，但是使用yield语句返回结果；
        - **Generator expressions生成器表达式**：它返回按需产生结果的对象。
    - 以上两者节省了内存空间，允许计算时间分摊到各次结果请求上。

2. Generator Functions: yield Versus return
    - State suspension状态挂起
        - 和返回一个值并退出的常规函数不同，**生成器函数**能够自动挂起并在生成值的时候恢复之前的状态并继续函数的执行。由于生成器函数在挂起时保存的状态包含它们的代码位置和整个局部作用域，因此当函数恢复时，它们的局部变量保持了信息并使其可用。
        - 生成器函数产生yield一个值，而不是返回return一个值，`yield语句`会挂起该函数并向调用者传回一个值，同时也保留了状态。
    - Iteration protocol integration与迭代协议集成
        - 函数若包含一条yield语句，该函数将被特别编译为**生成器**：它们不再是普通函数，而是作为通过特定的迭代协议方法来返回对象的函数。当调用时，它们返回一个生成器对象，该对象支持用一个自动创建的名为`__next__`的方法接口。
        - 生成器函数也可以有一条return语句，在def语句块的结尾，用于终止值的生成。

3. 生成器函数的应用
    ``` python
    def gensquares(N):
        for i in range(N):
            yield i ** 2
    ```
    - 这个函数在每次循环时都会yield一个值，之后将其返还给它的调用者。当它被暂停后，它的上一个状态被保存了下来，包括变量i和N，并且在yield语句之后被收回控制权。
    - 调用生成器函数：
    ``` python
    x = gensquares(4)
    print(x)
    ```
    - 返回的生成器对象有一个`__next__`方法，该方法可以开始这个函数，或者从它上次yield值后的地方恢复，并且在得到一系列值后，引发StopIteration异常：
    ``` python
    print(next(x))
    print(next(x))
    print(next(x))
    print(next(x))
    ```
    - *生成器函数本身就是迭代器*，所以不需要iter调用：
    ``` python
    y = gensquares(5)
    print(iter(y) is y)
    print(next(y))
    ```

4. 为什么要使用生成器函数
    - 生成器对于大型程序而言，在内存使用和性能方面都更好。生成器将产生一系列值的时间分散到每一次的循环迭代中；
    - 生成器函数是一种“穷人的”多线程机制：将操作拆分到每一次的yield之间。然而生成器仍然运行在一个单线程的控制内。

5. 拓展生成器函数协议：send vs next
    - 生成器函数协议有一个`send方法`，可以生成一系列结果的下一个元素，就像__next__方法一样，但是它也提供了一种调用者与生成器之间通讯的方式；
    - yield可以返回发送给send函数的元素（必须用A = yield X）；
    - 当使用这一额外的协议时，值可以通过调用G.send(value)发送给一个生成器G。之后恢复生成器代码的执行，并且生成器中的yield表达式返回了发送给send函数的值，如果提前调用了G.\_\_next\_\_()方法，yield返回None。
    ``` python
    def gen():
        for i in range(10):
            X = yield i
            print(X)
    G = gen()
    print(next(G)) # Must call next() first, to start generator,否则会出错
    print(G.send(77))
    print(G.send(77))
    print(next(G))
    ```
    运行结果如下：
    ``` python
    0
    77
    1
    77
    2
    None
    3
    ```

### 三、Generator Expressions
1. Generator Expressions: Iterables Meet Comprehensions生成器表达式：当可迭代对象遇见推导语法
    - **生成器表达式**跟列表推导很像，但是是在圆括号内，而不是方括号：
    ``` python
    print([x ** 2 for x in range(4)]) # List comprehension
    print((x ** 2 for x in range(4))) # Generator Expressions
    ```
    - 生成器表达式也是迭代器，不需要iter调用：
    ``` python
    G = (x ** 2 for x in range(4))
    print(iter(G) is G)
    print(next(G))
    print(next(G))
    print(next(G))
    print(next(G))
    ```
    - 我们一般不会在生成器表达式看到next机制的使用，因为for循环会自动触发：
    ``` python
    for num in (x ** 2 for x in range(4)):
        print('%s, %s' % (num, num / 2.0))
    ```
    - 每个迭代上下文都如上，包括for循环、sum、map和sorted等内置函数：
    ``` python
    print(''.join(x.upper() for x in 'aaa,bbb,ccc'.split(',')))
    print(sum(x ** 2 for x in range(4)))
    print(sorted(x ** 2 for x in range(4)))
    print(sorted((x ** 2 for x in range(4)), reverse=True))
    ```

2. 为什么使用生成器表达式
    - 就像生成器函数，生成器表达式是一种对内存空间的优化；
    - 生成器表达式在实际运行起来可能比列表推导稍慢一些，但对于非常大的运算或不能等待全部数据产生结果的应用来说是最优选择。

3. 生成器表达式 vs map
    - 生成器表达式通常等效于map调用：
    ``` python
    print(list(map(abs, (-1, -2, 3, 4))))
    print(list(abs(x) for x in (-1, -2, 3, 4)))
    ```
    - map和生成器都可以任意嵌套，下面的列表推导与map产生相同的结果，但是列表推导创建了2个列表：
    ``` python
    [x * 2 for x in [abs(x) for x in (-1, -2, 3, 4)]]
    list(map(lambda x: x * 2, map(abs, (-1, -2, 3, 4))))
    ```

4. 生成器表达式 vs filter
    - filter和一个带有if分句的生成器表达式是等价的：
    ``` python
    line = 'aa bbb c'
    print(''.join(x for x in line.split() if len(x) > 1))
    print(''.join(filter(lambda x: len(x) > 1, line.split())))
    ```

### 四、Generator Functions Versus Generator Expressions
1. 生成器函数 vs 生成器表达式
    - 生成器函数：一个使用了yield表达式的def语句是一个生成器函数。当被调用时，它返回一个新的生成器对象；
    - 生成器表达式：一个被包括在圆括号内的列表推导表达式被称为一个生成器表达式；
    - 这2个都包括一个返回自身的__iter__方法以及一个启动隐式循环或从上次运行离开的地方重新开始的__next__方法，都可以在能主动调用这些接口的迭代上下文中自动按需产生结果。

2. **生成器是单遍迭代对象Generators Are Single-Iteration Objects**
    - 生成器函数和生成器表达式本身都是迭代器，在一个生成器上调用iter没有实际效果。如果尝试手动使用多个迭代器来迭代结果，它们都将指向相同位置：
    ``` python
    G = (c * 4 for c in 'SPAM')
    I1 = iter(G)
    print(next(I1))
    print(next(I1))
    I2 = iter(G)
    print(next(I2))
    ```
    - 一旦任一迭代器运行结束，我们必须产生一个新的生成器以便重新开始。
    - 这与某些内置类型的行为不同。内置类型支持多个迭代器与多次迭代，并且在活跃迭代器中传递并反映它们的原位置修改：
    ``` python
    L = [1, 2, 3, 4]
    I1, I2 = iter(L), iter(L)
    print(next(I1))
    print(next(I1))
    print(next(I2))
    ```

3. `yield from`拓展
    ``` python
    def both(N):
        for i in range(N): yield i
        for i in (x ** 2 for x in range(N)): yield i
    print(list(both(5)))
    ```
    - 用`yield from`：
    ``` python
    def both(N):
        yield from range(N)
        yield from (x ** 2 for x in range(N))
    print(list(both(5)))
    print(' : '.join(str(i) for i in both(5)))
    ```

### 五、Generation in Built-in Types, Tools, and Classes
1. Generators and library tools: Directory walkers生成器和库工具：目录遍历器
    - `os.walk`在Python中的os.py标准库文件中被编写为一个迭代函数，它使用yield返回结果，因此它是一个迭代器：
    ``` python
    import os
    for (root, subs, files) in os.walk('.'):
        for name in files:
            if name.startswith('20'):
                print(root, name)
    ```

2. Generators and function application生成器和函数应用
    - 带“*”的参数可以将一个可迭代对象解包成单独的参数，详见第18章。
    ``` python
    def f(a, b, c): print('%s, %s, and %s' % (a, b, c))
    f(0, 1, 2)
    f(*range(3))
    f(*(i for i in range(3)))
    ```
    - 对字典的解包：
    ``` python
    D = dict(a='Bob', b='dev', c=40.5)
    f(a='Bob', b='dev', c=40.5)
    f(**D)
    f(*D)
    f(*D.values())
    ```

3. 类中用户定义的可迭代对象 详见第30章
    ``` python
    class SomeIterable:
        def __init__(...): ... 
        def __next__(...): ...
    ```

4. Permutations排列: All possible combinations
    - 用递归的方式实现排列（包括不同的顺序）
    ``` python
    def permute1(seq):
        if not seq:
            return [seq]
        else:
            res = []
            for i in range(len(seq)):
                rest = seq[:i] + seq[i+1:]
                for x in permute1(rest):
                    res.append(seq[i:i+1] + x)
            return res

    print(permute1('spam'))
    ```
    - 上述递归，直接看可能有点难理解，用数学来解释排列会好理解一点：
    - 如果让你弄出spam的所有的排列顺序，你会先提出来s，放到最前面，并对pam进行排序；再提出p，放到最前面，并对sam进行排序；...
    - 对应代码为 rest = seq[:i] + seq[i+1:]，即：把当前的节点数字去掉；
    - 对pam进行排序，需要先提出来p，放到最前面，并对am进行排序；对am进行排序，需要先提出来a；这也是为什么需要递归的原因，需要一层一层排序，并且拼接；
    - 对应代码为 for x in permute1(rest):，即：对剩余的数字进行排序；
    - 而 seq[i:i+1] + x 的意思就是把一个个提出来的字母拼接到前面去；
    - 最后返回所有排列组合。
    - 上面例子的生成器函数版本：
    ``` python
    def permute2(seq):
        if not seq:
            yield seq
        else:
            for i in range(len(seq)):
                rest = seq[:i] + seq[i+1:]
                for x in permute2(rest):
                    yield seq[i:i+1] + x

    print(list(permute2('spam')))
    print(permute1('spam') == list(permute2('spam')))
    G = permute2('spam')
    print(next(G))
    print(next(G))
    ```

5. 不要过度使用生成器
    - 生成器的隐式行为可能对于其他程序员难以理解，没必要强行使用让代码变复杂；
    - 列表能自动进行垃圾回收，同时更快地被编写（见下一章节）；
    - 生成器能减少内存和延迟。


### 六、Comprehension Syntax Summary
1. Scopes and Comprehension Variables 作用域及推导变量
    - 生成器推导、集合推导、字典推导以及列表推导中的临时循环变量名的作用域只局限于表达式内，不会与外部变量名冲突，这和for循环不一样：
    ``` python
    x = 99
    print([x for x in range(5)])
    print(x)

    y = 99
    for y in range(5): pass
    print(y)
    ```

2. 集合推导和字典推导
    - 集合推导和字典推导可视为把生成器表达式传递给类型名：
    ``` python
    print({x * x for x in range(10)})
    print(set(x * x for x in range(10)))
    print({x: x * x for x in range(10)})
    print(dict((x, x * x) for x in range(10)))
    ```
    - 与列表推导及生成器表达式一样，集合推导和字典推导接受if分句、嵌套for循环、在可迭代对象上迭代。


## chapter 21 The Benchmarking Interlude
### 一、Timing Iteration Alternatives
1. 计时迭代可选方案
    - 列表推导有时比for循环有速度优势，而map调用看情况，列表推导有时比for循环有速度优势，而map调用看情况：
    ``` python
    import time
    def timer(func, *args):
        start = time.perf_counter() 
        for i in range(1000): # 执行1000次
            func(*args)
        return time.perf_counter()  - start

    print(timer(pow, 2, 1000))
    ```
    - 上述代码有一些局限，包括额外计入了range的时间、不支持关键字参数，扩展上述代码，详见timer.py，如下：
    ``` python
    # timer.py
    """
    Homegrown timing tools for function calls.
    Does total time, best-of time, and best-of-totals time
    """

    import time, sys
    timer = time.perf_counter if sys.platform[:3] == 'win' else time.time

    def total(reps, func, *pargs, **kargs):
        """
        Total time to run func() reps times.
        Returns (total time, last result)
        """
        repslist = list(range(reps))
        start = timer()
        for i in repslist:
            ret = func(*pargs, **kargs)
        elapsed = timer() - start
        return (elapsed, ret)

    def bestof(reps, func, *pargs, **kargs):
        """
        Quickest func() among reps runs.
        Returns (best time, last result)
        """
        best = 2 ** 32 # 136 years seems large enough
        for i in range(reps): # range usage not timed here
            start = timer()
            ret = func(*pargs, **kargs)
            elapsed = timer() - start # Or call total() with reps=1
            if elapsed < best: best = elapsed # Or add to list and take min()
        return (best, ret)

    def bestoftotal(reps1, reps2, func, *pargs, **kargs):
        """
        Best of totals:
        (best of reps1 runs of (total of reps2 runs of func))
        """
        return bestof(reps1, total, reps2, func, *pargs, **kargs)
    ```
    运行代码如下：
    ``` python
    import timer
    print(timer.total(1000, pow, 2, 1000)[0])
    print(timer.total(1000, str.upper, 'spam'))
    print(timer.bestof(1000, str.upper, 'spam'))
    print(timer.bestof(1000, pow, 2, 1000000)[0])
    print(timer.bestof(50, timer.total, 1000, str.upper, 'spam'))
    print(timer.bestoftotal(50, 1000, str.upper, 'spam'))
    ```

2. 计时脚本
    - 计时迭代工具的计时器脚本，详见timeseqs.py，如下：
    ``` python
    "Test the relative speed of iteration tool alternatives."

    import sys, timer
    reps = 10000
    repslist = list(range(reps))

    def forLoop():
        res = []
        for x in repslist:
            res.append(abs(x))
        return res

    def listComp():
        return [abs(x) for x in repslist]

    def mapCall():
        return list(map(abs, repslist))

    def genExpr():
        return list(abs(x) for x in repslist)

    def genFunc():
        def gen():
            for x in repslist:
                yield abs(x)
        return list(gen())

    print(sys.version)
    for test in (forLoop, listComp, mapCall, genExpr, genFunc):
        (bestof, (total, result)) = timer.bestoftotal(5, 1000, test)
        print ('%-9s: %.5f => [%s...%s]' %
        (test.__name__, bestof, result[0], result[-1]))
    ```
    - 如代码所示，它报告的时间反应了这5个测试函数执行1000万部的快慢，每个函数创建了1000次带有10000个元素的列表；
    - 根据timeseqs.py的运行结果，生成器表达式比列表推导慢很多；map，即映射内置函数abs这样的内置函数，map会比列表推导快。

3. 无论如何，性能不是编写python代码的第一优先级。写出具有可读性和间接性的代码优先，然后在需要时优化。

### 二、Timing Iterations and Pythons with timeit
1. python标准库的timeit模块，自动化了代码计时
    - `timeit`，测试可以用可调用对象或语句字符串所指定；
    - 语句字符串：支持分隔符“；”或换行符“\n”分开的多条语句，以及用空格或制表符表示的缩进语句。

2. Interactive usage and API calls交互式用法与API调用
    - `timeit模块`中的`repeat`：表示调用运行若干组相同测试并返回一个列表，列表中的各项表示运行各组测试的总时间；
    - 每组测试由若干条相同测试语句（由number指定）组成；
    - 列表中的最小项（可通过min获取）就是各组运行的最佳时间；
    - stmt是statement的缩写。
    ``` python
    import timeit
    print(min(timeit.repeat(stmt="[x ** 2 for x in range(1000)]", number=1000, repeat=5)))
    ```
    - 上述代码运行5次，每次都执行了1000次的创建了拥有1000个元素的列表推导式。

3. Command-line usage命令行用法
    - timeit模块可以通过Python的 -m 标签自动地在路径上定位作为脚本运行：  
    `c:\code> python -m timeit -n 1000 "[x ** 2 for x in range(1000)]"`  
    `c:\code> py −3 -m timeit -n 1000 -r 5 "[x ** 2 for x in range(1000)]"`  
    `c:\code> C:\python33\Lib\timeit.py -n 1000 "[x ** 2 for x in range(1000)]"`  

4. Timing multiline statements计时对行语句
    - 在API调用模式下，可以使用换行符、制表符或空格写出多行代码：
    ``` python
    import timeit
    print(min(timeit.repeat(number=10000, repeat=3,
        stmt="L = [1, 2, 3, 4, 5]\nfor i in range(len(L)): L[i] += 1")))

    print(min(timeit.repeat(number=10000, repeat=3,
        stmt="L = [1, 2, 3, 4, 5]\ni=0\nwhile i < len(L):\n\tL[i] += 1\n\ti += 1")))
    ```
    - 在命令行模式下运行多行语句，可以把每条语句作为单独参数传入，并使用空白缩进；
    - timeit会把所有行拼接起来并插入换行符：  
    `c:\code> py −3 -m timeit -n 1000 -r 3 "L = [1,2,3,4,5]" "M = [x + 1 for x in L]"`

5. Other usage modes: Setup, totals, and objects其他使用模式：初始化、总时间和可运行对象
    - timeit允许让初始化的代码不计入要测试的语句的总时间:
        - 指定初始化代码，在命令行里用-s标签：  
        `c:\code> python -m timeit -n 1000 -r 3 -s "L = [1,2,3,4,5]" "M = [x + 1 for x in L]"` 这里把列表作为初始化语句分离了出来；
        - 或在API调用模式下使用setup参数字符串：
        ``` python
        import timeit
        print(min(timeit.repeat(number=10000, repeat=3,
            setup="L = [1, 2, 3, 4, 5]",
            stmt="for i in range(len(L)): L[i] += 1")))
        ```
    - timeit可以只计时总时间，使用模块的类API，计时可调用对象而不是字符串，详见库手册：
    ``` python
    import timeit
    print(timeit.timeit(stmt='[x ** 2 for x in range(1000)]', number=1000)) # Total time
    print(timeit.Timer(stmt='[x ** 2 for x in range(1000)]').timeit(1000)) # Class API
    print(timeit.repeat(stmt='[x ** 2 for x in range(1000)]', number=1000, repeat=3))

    def testcase():
        y = [x ** 2 for x in range(1000)]
    print(min(timeit.repeat(stmt=testcase, number=1000, repeat=3)))
    ```

### 三、Function Gotchas
1. Local Names Are Detected Statically局部变量是被静态检测的
    - python是在编译def代码时静态检测Python的局部变量的，而不是在运行时检测的：
    ``` python
    X = 99
    def selector(): 
        print(X)
    #   X = 88 如果这个语句在def里面，则出现未定义变量名错误
    selector()
    ```
    - 在编译时，python看到对X的赋值语句，决定了X会在函数中所有地方都是局部变量。但当函数运行时，执行print时，赋值并未发送，这时就会出现未定义变量名错误；
    - 产生这个问题的原因是，被赋值的变量名在函数内部的所有位置都被当做局部变量对待，不是在赋值以后的语句才被当作局部变量。
    - 如果想打印全局变量X，需要global语句：
    ``` python
    def selector():
        global X
        print(X)
        X = 88
    selector()
    print(X)
    ```
    - 但这样会改变全局变量X，如果不想改变，需要导入外围模块：
    ``` python
    X = 99
    def selector():
        import __main__
        print(__main__.X)
        X = 88
        print(X)
    selector()
    ```

2. Defaults and Mutable Objects默认值参数和可变对象
    - 用作默认值参数的可变值可以在调用之间保留状态；
    - 当def语句运行时，默认值参数就被求值并保存，而不是调用时；
    - python会将每一个默认值参数保存成一个对象，附加在函数本身；
    - 因为默认值参数在def时被求值，它能在外层作用域中保存值（详见工厂函数）。
    ``` python
    def saver(x=[]):
        x.append(1)
        print(x)
    saver([2])
    saver()
    saver()
    saver()
    ```