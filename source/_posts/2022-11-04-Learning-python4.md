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
> 本篇主要内容为：while和for循环；可迭代对象、迭代器和推导；Python文档资源

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


## chapter 14 Iterations and Comprehensions
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
