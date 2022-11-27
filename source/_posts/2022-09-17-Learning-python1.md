---
title: 《Learning Python》读书笔记（一）
date: 2022-09-17 17:24:20
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

# PART I Getting Started
## Chapter 2 How Python Runs Programs
### 一、Python Interpreter 解释器
* Python即是计算机语言，又是叫解释器的安装包
* Python语言代码 → python解释器 → 执行
* the interpreter is a layer of software logic between your code and the computer hardware on your machine.


### 二、Byte code compilation 字节码编译
1. python interpreter将source code（源代码）编译成byte code（字节码），再转发到 → virtual machine（虚拟机）；
2. byte code
    - 字节码比源代码运行速度快；
    - python会将程序的字节码保持在.pyc的拓展名文件，并存储在__pycache__的子目录中（在源文件同一路径）；
    - 若上次保存的字节码之后没修改过源代码，则直接加载.pyc文件；
    - 字节码不是机器的2进制代码，是特定于python的一种表达形式。这也是python比c慢的原因，字节码比cpu指令需要更多的工作。
3. python virtual machine(PVM)
    - 字节码发送至虚拟机的程序上执行，虚拟机是runtime engine of Python，是python系统的一部分。

### 三、Python编译器的变种或者实现方式（implementation）
* 5个主要的变体：CPython, Jython, IronPython, Stackless, and PyPy。
1. CPython
    - The original, and standard, implementation of Python is usually called CPython；
    - cpython就是大部分人使用的；
    - Cpython允许python脚本化C和C++。

2. Jython: Python for Java
    - Jython包含Java类，这些类将python源代码编译成Java字节码，在到Java虚拟机(JVM)；
    - Jython让Python代码能够脚本化Java应用程序，python代码被翻译成Java字节码，运行起来像Java程序。

3. IronPython: Python for .NET
    - 让python程序与windows的.NET框架以及linux的开源的Mono编写的应用相集成（integrate with）；
    - IronPython allows Python programs to act as both client and server components, gain accessibility both to and from other .NET languages。

4. Stackless: Python for concurrency（并发性）
    - Because it does not save state on the C language call stack, Stackless Python can make Python easier to port to small；
    - stack architectures, provides efficient multiprocessing options, and fosters novel programming structures such as coroutines；
    - 上面的翻译：因为它不会在C语言调用栈上保存状态，让它更易移植到较小的栈架构中，提供了高效的多处理选项，并且促进了协程(coroutine)的编程结构的出现。

5. PyPy: Python for speed
    - PyPy是Cpython的另一种实现，It provides a fast Python implementation with a JIT (just-in-time) compiler；
    - JIT：字节码转换和程序运行同时进行；
    - PyPy is the successor（继任者） to the original Psyco JIT；
    - PyPy currently claims a 5.7X speedup over CPython，让某些情况下跟C一样快，有时超越C。


## Chapter 3 How You Run Programs
### 一、Interactive command line交互式命令行；interactive prompt交互式提示
1. 就是Windows上的DOS console window（DOS控制台窗口）—a program named cmd.exe and usually known as Command Prompt(命令提示符)；
2. DOS：Disk Operating System磁盘操作系统(一种面向磁盘的系统软件)，DOS就是人给机器下达命令的集合，是存储在操作系统中的命令集。

### 二、The System Path系统路径
1. if you have not set your system’s PATH environment variable to include Python’s install directory, you may need to replace；
2. the word “python” with the full path to the Python executable on your machine；
3. 比如c:\Users> c:\python33\python而不是C:\Users\\> python

### 三、Running Code Interactively交互式地运行代码
1. When working interactively, the results of your code are displayed below the >>> input lines after you press the Enter key.

### 四、程序program、模块module、脚本script的区别
1. Terminology in this domain can vary somewhat；
2. module files are often referred to as programs in Python—a program is considered to be a series of precoded statements stored in a file for repeated execution；
3. Module files that are run directly are also sometimes called scripts—an informal term usually meaning a top-level顶层 program file；
4. Some reserve the term “module” for a file imported from another file, and “script” for the main file of a program; we generally will here, too.

### 五、IDLE GUI (Integrated Development and Learning Environment; Graphical User Interface)集成开发和学习环境的图形用户界面
1. shell （计算机壳层）指为使用者提供操作界面的软件，类似于DOS下的COMMAND.COM和后来的cmd.exe；
2. 基本上shell分两大类： GUI shell（图形界面shell）和Command Line Interface shell（命令行式shell）；
3. 在命令行shell中python draft.py > saveit.txt 可以让输出行都存储在该txt文件中，这叫流重定向 stream redirection。

### 六、Module的属性（Attributes）
1. a module is mostly just a package of variable names, known as a namespace, and the names within that package are called attributes；
2. 模块是变量名的包，即命名空间，在包内的变量名称为属性；
3. An attribute is simply a variable name that is attached to a specific object (like a module).


# PART II Types and Operations
## chapter 4 Introducing Python Object Types
### 一、Python object types
1. OOP Object Oriented Programming 面向对象编程
2. Python’s Core Data Types或built-in object types（内置对象类型）
    - 数字Numbers；字符串Strings；列表Lists；字典Dictionaries；元组Tuples；文件Files；集合Sets；
    - Other core types：布尔型Booleans；类型types  Program unit types：函数Functions；模块modules；类classes；
    - Implementation-related types：已编译代码Compiled code, 调用栈跟踪stack tracebacks；
    - 列表是对象的有序集合，字典通过键key存储对象。
3. 作用于多种类型的通用操作都是以内置函数或表达式的形式出现的（如len(X)、X[0])；类型特定的操作是以方法调用的形式出现的（如string.upper())

### 二、numbers and strings
1. 数字Numbers

    - 数字没有长度，要先转变为字符串再用len函数：
    ``` python 
    print(len(str(12345)))
    ```
    - math module和random module:    
    ``` python
    import math
    print(math.pi)
    print(math.sqrt(85))
    import random
    print(random.random())
    print(random.choice([1, 2, 3, 4]))
    ```

2. 字符串Strings
    - 严格来说字符串是由单字符的字符串所组成的序列sequence；
    - （1）序列操作：字符串支持位置顺序的操作，可通过索引indexing操作
    ``` python
    S = 'Spam'
    print(len(S))
    print(S[0])
    print(S[1])
    print(S[-1])
    print(S[len(S)-1])
    print(S[1:3])  # 这个叫分片（slice）X[I:J]，注意不包括J。
    print(S[1:])
    print(S[:-1])
    print(S[:])
    ```
    - （2）拼接concatenation：
    ``` python
    print(S + 'xyz')
    print(S * 8)
    ```
    - 加号既可以用作数字加号，也可以字符串拼接，这就是一种多态性 polymorphism：the meaning of an operation depends on the objects being operated on；
    - （3）不可变性Immutability：
    - you can’t change a string by assigning to one of its positions, but you can always build a new one and assign it to the same name；
    - 不能通过对位置进行赋值（assign）而改变字符串，但是可以对整个变量赋值；
    - 比如不能 S[0] = 'z'
    ``` python
    S = 'z' + S[1:]
    print(S)
    ```
    - （4）数字、字符串和元组是不可变的，列表、集合、字典是可以变的；
    - （5）基于位置改变文本数据：
    ``` python
    S = 'shrubbery'
    L = list(S)
    print(L)
    L[1] = 'c'
    print('-'.join(L)) # join()方法 str.join(sequence) 将序列中的元素以指定的字符连接生成一个新的字符串
    B = bytearray(b'spam') # 用bytearray()方法进行拼接
    B.extend(b'eggs')
    print(B)
    print(B.decode())
    ```
    - （6）其他字符串的方法：
    ``` python
    S = 'Spam'
    print(S.find('pa')) # find()方法 str.find(str, beg=0, end=len(string)) 返回一个查找的字符的偏移量offset，没有找到就返回-1
    print(S.replace('pa', 'XYZ')) #  replace() 方法把字符串中的 old（旧字符串） 替换成 new(新字符串)
    print(S) # 因为字符不可变，所以上面方法只是创建新的字符串作为结果
    line = 'aaa,bbb,ccccc,dd'
    print(line.split(','))
    S = 'spam'
    print(S.upper())
    print(S.isalpha()) # isalpha()方法检测字符串是否只由字母组成
    line = 'aaa,bbb,ccccc,dd\n'
    print(line.rstrip())
    print(line.rstrip().split(','))
    ```
    - （7）字符串formatting的替代操作（包括f字符串），以下都可以：
    ``` python  
    a = 'spam'
    b = 'SPAM!'
    print('%s, eggs, and %s' % (a, b)) # Formatting expression (all)
    print('{0}, eggs, and {1}'.format(a, b))
    print('{}, eggs, and {}'.format(a, b))
    print(f'{a}, eggs, and {b}')
    print('{:,.2f}'.format(296999.2567))
    print('%.2f | %+05d' % (3.14159, -42))
    ```
    - （8）内置的dir函数，这个函数会列出再调用者作用域内，形式参数的默认值。 dir：directory（目录）
    - 这个函数也会返回一个列表，包含对象的所有属性。由于方法是函数属性，也会在列表里。
    ``` python
    print(dir(S))
    # 例如字符串的_add_方法是真正执行字符串拼接的函数。但尽量避免使用下面的第二个方法，因为运行更慢
    print(S + 'NI!')
    print(S.__add__('NI!'))
    ```
    - help函数可以查询方法是做什么的 比如help(S.replace)
    - （9） ord()函数是chr()函数（对于8位的ASCII字符串）或unichr()函数（对于Unicode对象）的配对函数
    - 它以一个字符（长度为1的字符串）作为参数，返回对应的ASCII数值，或者Unicode数值，是ordinal的缩写
    ``` python
    print(ord('a'))
    ```
    - （10）原始raw字符串，去掉反斜线转义机制
    ``` python
    print(r'\nhaha')
    print('\nhaha')
    ```
    - （11）Unicode字符串(ASCII是一种简单的Unicode)
    ``` python
    print('sp\xc4m') # 3.X: normal str strings are Unicode text    \x为16进制转义符
    print(b'a\x01c') # bytes字符串表示原始字节值
    print(u'sp\u00c4m') # The 2.X Unicode literal works in 3.3+: just str   \u为Unicode转义符，可百度查表\u00c4对应Ä
    ```
    - （12）encode()方法、decode()方法进行编码解码
    ```
    str = "菜鸟教程"
    str_utf8 = str.encode("UTF-8")
    print("UTF-8 编码：", str_utf8)
    print("UTF-8 解码：", str_utf8.decode('UTF-8'))
    ```
    - （13）★正则表达式(regular expression) Python的re模块 用于文本的模式匹配
    - 正则表达式描述了一种字符串匹配的模式（pattern），可以用来检查一个串是否含有某种子串(搜索)、将匹配的子串替换或者从某个串中取出符合某个条件的子串(分割)等。
    ``` python
    import re
    match = re.match('Hello[ \t]*(.*)world', 'Hello Python world')
    print(match.group(1))
    match = re.match('[/:](.*)[/:](.*)[/:](.*)', '/usr/home:lumberjack')
    print(match.groups())
    ```
    - 大致了解一下，37章会更详细说明

### 三、lists
1. 列表Lists
    - （1）列表是可变的mutable，与字符串不同，通过对对应偏移量offset进行赋值或者其他方法对列表进行操作，可对列表进行改变：
    ``` python
    L = [123, 'spam', 1.23] 
    print(len(L))
    print(L[0])
    (L[:-1])
    print(L + [4, 5, 6])
    print(L * 2)
    print(L) # 列表没变，因为没有赋值

    L.append('NI') # append方法增加了列表的大小
    print(L)
    print(L.pop(2)) # 弹出并返回 改变了列表 pop：出栈
    print(L)

    M = ['bb', 'aa', 'cc']
    M.sort() # sort方法 按升序修改列表排序
    print(M)
    M.reverse() # reverse方法 对列表进行反转修改
    print(M)
    ```
    - （2）嵌套Nesting、矩阵matrixes或者多维数组multidimensional arrays:
    ``` python
    M = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
    print(M)
    print(M[1])
    print(M[1][2])
    ```
    - （3）list comprehension expression列表解析(推导)表达式：
    ```python
    col2 = [row[1] for row in M] # row可以为任意变量名
    print(col2)
    print([row[1] + 1 for row in M])
    print([row[1] for row in M if row[1] % 2 == 0])
    diag = [M[i][i] for i in [0, 1, 2]]
    print(diag)
    doubles = [c * 2 for c in 'spam']
    print(doubles)
    ```
    - 用range函数来表示循环的次数
    ``` python
    print(list(range(4)))
    print(list(range(-6, 7, 2)))
    print([[x ** 2, x ** 3] for x in range(4)])
    print([[x, x / 2, x * 2] for x in range(-6, 7, 2) if x > 0])
    ```
    - 下面的内容比较复杂，下面只是作为前瞻：
    ``` python
    G = (sum(row) for row in M) # 创建★生成器generator，把一个列表生成式的[]改成()，generator也是可迭代对象
    print(next(G)) # iter(G) not required here   iter()函数：生成迭代器
    print(next(G)) # Run the iteration protocol next()  next()：返回迭代器的下一个项目
    print(list(map(sum, M))) # map()会根据提供的函数对指定序列做映射
    ```
    - comprehension syntax推导语法也可用于字典和集合，以及之前的列表和生成器generators
    ``` python
    print({sum(row) for row in M})
    print({i : sum(M[i]) for i in range(3)})
    print([ord(x) for x in 'spaam'])
    print({ord(x) for x in 'spaam'})
    print({x: ord(x) for x in 'spaam'})
    print((ord(x) for x in 'spaam'))
    ```

### 四、dictionaries
1. 字典Dictionaries
    - （1）字典不是序列，而是映射（mapping）：
    -  映射操作，字典编写在大括号里，包含一系列键值对（“key: value” pairs）。不能通过相对位置进行索引。
    ``` python
    D = {'food': 'Spam', 'quantity': 4, 'color': 'pink'}
    print(D['food'])
    D['quantity'] += 1
    print(D)
    ```
    - （2）对键赋值assignments创建新的键值对或修改键值对
    ``` python
    D = {}
    D['name'] = 'Tom' 
    D['job'] = 'dev'
    D['age'] = 40
    print(D)
    D['name'] = 'Bob'
    print(D)
    ```
    - （3）其他方式创建字典
    ``` python
    bob1 = dict(name='Bob', job='dev', age=40)
    print(bob1)
    bob2 = dict(zip(['name', 'job', 'age'], ['Bob', 'dev', 40])) # zip()函数将对象中对应的元素打包成一个个元组
    print(bob2)
    ```
    - （4）nesting嵌套
    ``` python
    rec = {
        'name': {'first': 'Bob', 'last': 'Smith'},
        'jobs': ['dev', 'mgr'],
        'age': 40.5,
    }
    print(rec['name'])
    print(rec['name']['last'])
    print(rec['jobs'][-1])
    rec['jobs'].append('janitor')
    print(rec)
    ```
    - （5）用if语句测试键是否在字典里
    ``` python
    D = {'a': 1, 'b': 2, 'c': 3}
    if not 'f' in D: # 'f' in D是False，not False = True
        print('missing')
    # 其他方法测试，大致介绍，后面会详细
    value = D.get('x', 0) #  get()函数返回指定键的值，如果指定的键不存在时返回指定的默认值
    print(value)
    value = D['x'] if 'x' in D else 0
    print(value)
    ```
    - （6）键的排序
    ``` python
    D = {'a': 1, 'c': 3, 'b': 2}
    Ks = list(D.keys())
    Ks.sort() # 列表的sort()方法
    print(Ks)
    for key in Ks: 
        print(key, '=>', D[key])

    for key in sorted(D): # sorted()函数
        print(key, '=>', D[key])
    ```
    - （7）迭代和优化Iteration and Optimization
    - for loop（for循环）是通用迭代工具，遵守迭代协议；
    - an object is iterable if it is either a physically stored sequence in memory, or an object that generates one item at a time in the context of an iteration operation — a sort of “virtual” sequence；
    - 一个可迭代对象可以是在内存物理存储的序列，也可以是迭代操作下产生的对象（一种虚拟的序列）；
    - 前面的生成器（generator）就是一个可迭代对象，它在值并非存储在内存中，而是通过迭代工具在被请求时生成；
    - 这里初步认识一下，后面会详细说明迭代协议iteration protocol；
    - 任何一个从左到右扫描一个对象的Python工具都使用迭代协议；
    - 下面2种工具在迭代协议内部发挥作用，并产生相同结果：
    ``` python
    squares = [x ** 2 for x in [1, 2, 3, 4, 5]] # 列表推导表达式
    print(squares)
    squares = []
    for x in [1, 2, 3, 4, 5]: 
        squares.append(x ** 2)
    print(squares)
    ```
    - 列表推导和相关的函数编程工具（如map和filter）通常运行得比for循环快；
    - “iterable,” means either a physical sequence, or a virtual one that produces its items on request.

### 五、tuples
1. 元组Tuples
    - （1）元组是序列，但它具有不可变性，和字符串类似。
    ``` python
    T = (1, 2, 3, 4)
    print(len(T))
    print(T + (5, 6))
    print(T[0])
    ```
    - （2）元组还有2个专有的可调用方法
    ``` python
    print(T.index(4)) # T中4的索引是3
    print(T.count(4)) # T中4出现了1次
    ```
    - （3）元组不能对索引进行赋值，因为具有不可变性
    - 比如T[0] = 2会错误
    - 只含一个元素的元组需要逗号作为结尾
    ``` python
    T = (1, 2, 3, 4)
    T = (2,) + T[1:]
    print(T)
    ```
    - （4）嵌套nesting
    - 元组元素的圆括号通常可以被省略
    ``` python
    T = 'spam', 3.0, [11, 22, 33]
    print(T[1])
    print(T[2][1])
    ```

### 六、files
1. 文件Files
    - （1）File objects are Python code’s main interface（接口） to external files on your computer；
    - open函数实例如下：
    ``` python
    f = open('data.txt', 'w')
    f.write('Hello\n') # 该方法会返回6，因为有6bytes被写入，utf-8中一个英文字符等于一个字节，该源文件由utf-8编码。
    f.write('world\n') # 该方法会返回6，因为有6bytes被写入，utf-8中一个英文字符等于一个字节
    f.close()
    f = open('data.txt')
    text = f.read()
    print(text)
    print(text.split())
    ```
    - 如今读取一个文件的最佳方式就是根本不读它，而是通过文件提供的一个迭代器（iterator）在for循环或其他上下文自动逐行读取：
    ``` python
    for line in open('data.txt'): print(line)
    print(dir(f))
    ```
    - （2）二进制字节文件Binary Bytes Files
    - 文本文件把内容显示为正常的str字符串，并且在写入读取数据时自动执行Unicode编码和解码；
    - 而二进制文件把内容显示为一个特定的字节字符串，并且允许你不修改地访问文件内容；
    - Python的struct模块可以同时创建和解析被打包过的二进制数据来写入一个二进制模式的文件；
    ``` python
    import struct
    packed = struct.pack('>i4sh', 7, b'spam', 8)
    print(packed) # 10 bytes, not objects or text，显示出的为特定的字节字符串
    file = open('data.bin', 'wb')
    file.write(packed) # 该方法会返回10，因为有10bytes被写入
    file.close()
    data = open('data.bin', 'rb').read()
    print(data)
    print(data[4:8])
    print(list(data)) # 小写s的Unicode编码为115，p为112，a为97，m为109
    print(struct.unpack('>i4sh', data))
    ```
    - （3）Unicode文本文件Unicode Text Files
    - 为了访问文件中的非ASCII编码的Unicode文本，可以传入一个编码名参数。读取和写入时按指定的编码范式进行解码和编码；
    ``` python
    S = 'sp\xc4m'
    print(S)
    file = open('unidata.txt', 'w', encoding='utf-8') # Write/encode UTF-8 text
    file.write(S) # 该方法会返回4，因为有4bytes被写入
    file.close()
    text = open('unidata.txt', encoding='utf-8').read()
    print(text)
    print(len(text))

    raw = open('unidata.txt', 'rb').read()
    print(raw)
    print(len(raw))
    print(text.encode('utf-8'))
    print(raw.decode('utf-8'))
    print(text.encode('latin-1'))
    print(text.encode('utf-16'))
    print(len(text.encode('latin-1')), len(text.encode('utf-16')))
    print(b'\xff\xfes\x00p\x00\xc4\x00m\x00'.decode('utf-16'))
    ```
    - （4）其他类文件工具：Python comes with additional file-like tools: pipes(管道), FIFOs, sockets(套接字), keyed-access files(键值访问文件), persistent object shelves(持久化对象shelve), descriptor-based files(基于描述符的文件), relational and object-oriented database interfaces(关系型数据库接口和面向对象数据库接口等), and more.

### 七、Other core types
1. 集合Sets
    - 集合不是映射也不是序列，它是不可变的唯一对象的无序集合；
    - 集合是大括号{}，更像是一个无值的字典的键；

    ``` python
    X = set('spam') # set()函数
    Y = {'h', 'a', 'm'}
    Z = X, Y
    print(Z)
    print(X & Y) # Intersection
    print(X | Y) # Union
    print(X - Y) # Difference
    print(X > Y) # Superset
    print({n ** 2 for n in [1, 2, 3, 4]}) # Set comprehensions 集合推导式
    print(list(set([1, 2, 1, 3, 1])))
    print(set('spam') - set('ham'))
    print(set('spam') == set('asmp'))
    print('p' in set('spam'), 'p' in 'spam', 'ham' in ['eggs', 'spam', 'ham'])
    ```

2. 数值类型：十进制数decimal和分数fraction
    ``` python
    print(type(1/3))
    import decimal
    d = decimal.Decimal('3.141')
    print(d + 1)
    print(type(d+1))

    decimal.getcontext().prec = 2 # prec:precision精度
    e = decimal.Decimal('1.00') / decimal.Decimal('3.00')
    print(e)
    print(type(e))

    from fractions import Fraction
    f = Fraction(2, 3)
    print(f)
    print(type(f))
    print(f + Fraction(1, 2))
    ```

3. 布尔值 Booleans 包括True、False、None
    ``` python
    print(1 > 2, 1 < 2)
    print(bool('spam'))  # Object's Boolean value
    X = None
    print(X)
    L = [None] * 10
    print(L)
    print(type(L))
    print(type(type(L)))

    if type(L) == type([]): 
        print('yes')
    if type(L) == list: 
        print('yes')
    if isinstance(L, list): # 判断一个对象是否为一个类型
        print('yes')
    ```

4. 定义的类class
    ``` python
    class Worker:
        def __init__(self, name, pay): # Initialize when created
            self.name = name # self is the new object
            self.pay = pay
        def lastName(self):
            return self.name.split()[-1] # Split string on blanks
        def giveRaise(self, percent):
            self.pay *= (1 + percent) # Update pay in place

    bob = Worker('Bob Smith', 50000) # Make two instances
    sue = Worker('Sue Jones', 60000) 
    print(bob.lastName()) # Call method: bob is self
    print(sue.lastName()) # sue is the self subject
    sue.giveRaise(.10) # Updates sue's pay
    print(sue.pay)
    ```
    - name和pay是类的属性attributes (sometimes called 状态信息state information)；
    - lastName和giveRaise是类的方法functions (normally called methods)。


## chapter 5 Numeric Types
### 一、Numeric Type Basics
1. 完整的Python数值类型工具包括：
    - Integer and floating-point objects 整数和浮点对象；
    - Complex number objects 复数对象；
    - Decimal: fixed-precision objects 小数：固定精度对象；
    - Fraction: rational number objects 分数：有理数对象；
    - Sets: collections with numeric operations 集合：带有数值运算的集合体；
    - Booleans: true and false 布尔值：真和假；
    - Built-in functions and modules: round, math, random, etc. 内置函数和模块；
    - Expressions; unlimited integer precision; bitwise operations; hex, octal, and binary formats 表达式；无限制整数精度；位运算；十六进制，八进制和二进制格式；
    - Third-party extensions: vectors, libraries, visualization, plotting, etc. 第三方拓展：向量、库、可视化、作图等。

2. 数值字面量Numeric Literals
    - 浮点数包括小数和带科学计数标志e即幂的数字；
    - 十六进制以0X或0x开头；八进制以0o或0O开头；二进制以0b或0B开头：
    ``` python
    print(hex(18)) # 转换为十六进制hexadecimal
    print(oct(18)) # 转换为八进制octonary
    print(bin(18)) # 转换为二进制binary
    print(int('0b100', base=2)) # 将字符串以base进制转换为整数
    ```
    - 复数字面量被写成实部realpart和虚部imaginarypart，虚部以j或J结尾：
    ``` python
    aa=123-12j
    print(aa.real)
    print(aa.imag)
    print(aa)
    print(complex(123, -12)) # complex(real, imag)来创建复数
    ```
    - 导入模块调入小数分数：
    ``` python
    import decimal, fractions
    print(decimal.Decimal('1.0')) 
    print(fractions.Fraction(1,3))
    ```

3. 内置built-in数值工具
    - 包括表达式运算符Expression Operators；内置数学函数；工具模块
    - 下面的运算符从上到下优先级逐渐升高：

| Operators | Description |
| :---- | :---- |
| yield x | 生成器函数send协议 |
| lambda args: expression | 创建匿名函数 |
| x if y else z | 三元选择表达式（仅当y为真时，x才会被计算）Ternary selection |
| x or y | 逻辑或（仅当x为假时，y才会被计算）Logical OR |
| x and y | 逻辑与（仅当x为真时，y才会被计算）Logical AND |
| not x | 逻辑非Logical negation |
| x in y, x not in y | 成员关系 (iterables, sets) |
| x is y, x is not y | 对象同一性测试 |
| x < y, x <= y, x > y, x >= y | 大小比较、集合的子集和超集subset and superset |
| x == y, x != y | 值等价性运算符 |
| x &#124; y | 按位与、集合交集Bitwise OR, set union |
| x ^ y | 按位异或、集合对称差集Bitwise XOR, set symmetric difference |
| x & y | 按位与、集合交集Bitwise AND, set intersection |
| x << y, x >> y | 将x左移或右移y位Shift x left or right by y bits |
| x + y | 加法、拼接Addition, concatenation |
| x – y | 减法、集合差集Subtraction, set difference |
| x * y | 乘法、重复Multiplication, repetition |
| x % y | 取余数、格式化字符串Remainder, format |
| x / y, x // y | 真除法、向下取整除法Division: true and floor |
| −x, +x | 取负、取正Negation, identity |
| ˜x | 按位非（取反码）Bitwise NOT (inversion) |
| x ** y | 幂运算（指数）Power (exponentiation) |
| x[i] | 索引（序列、映射等）Indexing (sequence, mapping, others) |
| x[i:j:k] | 分片Slicing |
|x(...) | 调用（函数、方法、类，其他可调用对象）Call (function, method, class, other callable) |
| x.attr | 属性引用Attribute reference |
| (...) | 元组、表达式、生成器表达式 Tuple, expression, generator expression |
| [...] | 列表、列表推导List, list comprehension |
| {...} | 字典、集合、集合与字典推导Dictionary, set, set and dictionary comprehensions |

4. 运算符重载和多态Operator overloading and polymorphism
    - Python itself automatically overloads some operators, such that they perform different actions depending on the type of built-in objects being processed.
    - 就是比如加号在数字上就是加法，字符串上就是拼接，用在自己定义的类的对象上时，可以执行任意操作；
    - 这种特性被成为多态，即操作的意义由操作对象来决定。

### 二、Numbers in Action
1. 数值的显示格式
    - 以下的字符串格式化string formatting在第7章介绍

    ``` python
    num = 1 / 3.0
    print('%e' % num)
    print('%4.2f' % num)
    print('{0:4.2f}'.format(num))
    ```

2. 普通比较和链式比较Chained
    ``` python
    X = 2
    Y = 4
    Z = 6
    print(X < Y < Z) # 链式比较，等于X < Y and Y < Z
    ```
    - ★注意：浮点数的比较可能会出错，需要其他处理

    ``` python
    print(1.1 + 2.2 == 3.3)
    print(1.1 + 2.2) # 浮点数因为有限的比特位数，而不能精确地表示某些值
    print(int(1.1 + 2.2) == int(3.3))
    ```

3. 经典除法
    - 在Python 2.X或之前的版本中，5/2得到的结果为2，这个就是经典除法，因为整数除以整数得到整数；
    - 在Python 3.X中总是执行真除法，不管操作数的类型，都返回包括任何小数部分的一个浮点结果，甚至包括9/3得到3.0。

    ``` python
    print((9 / 3), (9.0 / 3), (9 // 3), (9 // 3.0))
    ```

4. 向下取整除法vs截断除法Floor versus truncation
    - //运算符地非正式别名为截断除法，但向下舍入不是严格地截断；
    - 用math模块可以看出上述区别：

    ``` python
    import math
    print(math.floor(2.5))
    print(math.floor(-2.5))
    print(math.trunc(2.5)) # truncation截断
    print(math.trunc(-2.5))
    print(5 // 2, 5 // -2) 
    print(5 // 2.0, 5 // -2.0) # //的结果的数据类型总是依赖于操作数的类型
    ```

5. 十六进制、八进制和二进制：字面量Literals与转换
    ``` python
    print(0o1, 0o20, 0o377) # 八进制字面量
    print(0x01, 0x10, 0xFF) # 十六进制字面量
    print(0b1, 0b10000, 0b11111111) # 二进制字面量
    # 十进制转换为其他进制
    print(oct(64), hex(64), bin(64))
    # 通过int函数将字符串转换为选定进制的整数
    print(int('64'), int('100', 8), int('40', 16), int('1000000', 2))
    # eval()函数用来执行一个字符串表达式，并返回表达式的值
    print(eval('64'), eval('0o100'), eval('0x40'), eval('0b1000000'))
    # 用字符串格式化将整数转换为指定的字符串，详见第七章
    print('{0:o}, {1:x}, {2:b}'.format(64, 64, 64))
    print('%o, %x, %x, %X' % (64, 64, 255, 255))
    ```

6. 按位操作Bitwise Operations
    - bit：位、比特、比特位；byte：字节
    - 这里不涉及位运算的细节。工作中若涉及二进制数据包会有用，需要再学习。

    ``` python
    x = 1
    print(x << 2) # 1为0001 in bits 向左移2 bits 即0100，相当于十进制的4
    print(x | 2) # Bitwise OR 按位或：0001|0010 = 0011，相当于十进制的3
    print(x & 1) # Bitwise AND 按位和：0001&0001 = 0001，相当于十进制的1
    X = 0xFF
    print(bin(X))
    print(bin(X ^ 0b10101010)) # Bitwise XOR 按位异或：either but not both：11111111^10101010 = 01010101
    # 用bit_length方法获取数字的位数，len函数也可以
    print(bin(256), (256).bit_length(), len(bin(256)) - 2)
    ```

7. 其他内置数值工具Other Built-in Numeric Tools
    ``` python
    # 内置函数pow和abs，内置模块math
    import math
    print(math.pi, math.e)
    print(math.sin(2 * math.pi / 180)) # math.pi = 3.14 = 180°，故这里结果为sin(2)
    print(math.sqrt(144), math.sqrt(2))

    print(pow(2, 4), 2 ** 4, 2.0 ** 4.0)
    print(abs(-42.0), sum((1, 2, 3, 4)))
    print(min(3, 1, 2, 4), max(3, 1, 2, 4))

    print(math.floor(2.567), math.floor(-2.567))
    print(math.trunc(2.567), math.trunc(-2.567))
    print(round(2.567), round(2.467), round(2.567, 2)) # round(2.5)为2
    print('%.1f' % 2.567, '{0:.2f}'.format(2.567))
    # 平方根的3种方式
    import math
    print(math.sqrt(144))
    print(144 ** .5)
    print(pow(144, .5))
    # random模块
    import random
    print(random.random()) # 0-1的随机浮点数
    print(random.randint(1, 10)) # 2个数字之间挑选一个随机整数
    # random模块从一个序列中随机选取一项，或随机打乱shuffle列表中的元素，shuffle：洗牌、打乱次序
    print(random.choice(['Life of Brian', 'Holy Grail', 'Meaning of Life']))
    suits = ['hearts', 'clubs', 'diamonds', 'spades']
    random.shuffle(suits)
    print(suits)
    ```

### 三、Other Numeric Types
1. Decimal Type小数类型
    - 区分Decimal与浮点数，Decimal有固定的位数和小数点，可以理解为精度固定的浮点数。
    - 浮点数运算缺乏精确性，所有小数对象虽然带来了性能上的损失，但精度更大。
    ``` python
    print(0.1 + 0.1 + 0.1 - 0.3)
    from decimal import Decimal
    print(Decimal('0.1') + Decimal('0.1') + Decimal('0.1') - Decimal('0.3'))
    print(type(Decimal('0.1') + Decimal('0.1') + Decimal('0.1') - Decimal('0.3')))
    # Decimal最好传入字符串，也可以传入浮点数，但是会产生默认且庞大的小数位数，所以最好用str函数变为字符串
    print(Decimal(0.1) + Decimal(0.1) + Decimal(0.1) - Decimal(0.3))
    # 设置全局小数精度
    import decimal
    print(decimal.Decimal(1) / decimal.Decimal(7))
    decimal.getcontext().prec = 4 # 全局小数精度
    print(decimal.getcontext())
    print(decimal.Decimal(1) / decimal.Decimal(7))
    print(Decimal(0.1) + Decimal(0.1) + Decimal(0.1) - Decimal(0.3))
    # 用with上下文管理器语句context manager statement来临时重置小数精度
    # 在with语句退出后，精度会重置为初始值。with语句详见第34章
    print(decimal.Decimal('1.00') / decimal.Decimal('3.00'))
    with decimal.localcontext() as ctx: # 注意是local
        ctx.prec = 2
        print(decimal.Decimal('1.00') / decimal.Decimal('3.00'))
    print(decimal.Decimal('1.00') / decimal.Decimal('3.00'))
    ```

2. Fraction Type分数类型
    - 也可处理浮点类型的数值不确定性
    ``` python
    from fractions import Fraction
    x = Fraction(1, 3)
    y = Fraction(4, 6)
    print(x, y)
    print(x + y)
    print(x * y)
    # 分数对象也可以用浮点数字符串来创建
    print(Fraction('.25'))
    ```

3. Fraction和Decimal都提供了得到精确结果的方式，但这需要付出一些速度和代码冗余性的代价。

4. 分数转换和混用类型
    - 浮点数对象的as_integer_ratio()方法：将一个float用分数表示出来，返回的是一个二元元组。
    ``` python
    print((2.5).as_integer_ratio())
    f = 2.5
    z = Fraction(*f.as_integer_ratio()) # 这里的*是一种特殊的语法syntax，可以把一个元组展开为独立的参数。18章会更详细
    print(z)
    print(float(z))

    print(Fraction.from_float(1.75)) # 分数的from_float方法
    print(Fraction(*(1.75).as_integer_ratio()))
    # 表达式允许某些类型的互用
    print(x + 2) # Fraction + int -> Fraction
    print(x + 2.0) # Fraction + float -> float
    print(x + (1./3)) # Fraction + float -> float
    print(x + Fraction(4, 3)) # Fraction + Fraction -> Fraction
    # 尽管可以把浮点数转换为分数，但会出现精度损失
    print(4.0 / 3)
    print((4.0 / 3).as_integer_ratio())
    a = Fraction(*(4.0 / 3).as_integer_ratio())
    print(6004799503160661 / 4503599627370496) # 非常接近于4/3
    print(a.limit_denominator(10)) # 限制分母的最大值
    ```

5. Sets集合
    - 集合是无序的unordered，可迭代的iterable，既不是序列sequence也不是映射mapping类型，是一些唯一的、不可变的对象的一个无序集合体。同数学集合。
    - 集合是无值的字典，可以使用集合字面量形式set literal form，即花括号（大括号）curly braces
    ``` python
    # set函数
    x = set('abcde')
    print(x)
    y = set('bdxyz')
    # 集合通过表达式运算符支持一般的数学集合运算。
    print(x - y) # Difference
    print(x | y) # Union
    print(x & y) # Intersection
    print(x ^ y) # Symmetric difference (XOR)
    print(x > y, x < y) # Superset, subset
    # in测试
    print('e' in x)
    # 集合的intersection方法；union方法；add方法；update方法；remove方法；issubst方法
    z = x.intersection(y)
    print(z)
    z.add('SPAM')
    print(z)
    z.update(set(['X', 'Y']))
    print(z)
    z.remove('b')
    print(z)
    S = set([1, 2, 3])
    print(S.union([3, 4]))
    print(S.intersection((1, 3, 5)))
    print(S.issubset(range(-5, 5)))
    ```
    - 集合是可迭代的容器，可以用于len、for循环和列表推导的操作
    - {}是个字典，空的集合需要用函数set来创建
    - Immutable constraints and frozen sets不可变性限制与冻结集合
    - 集合只能包含不可变的immutable (a.k.a. “hashable”)对象类型，列表和字典不能嵌入到集合里，但元组可以嵌入集合
    - 比如print(S.add([1, 2, 3])) 会出来TypeError: unhashable type: 'list'
    - 若要在另一个集合中存储一个集合，可以用内置函数frozenset创建一个不可变的集合，该集合不可修改，并且可以嵌套到其他集合中。
    ``` python
    for item in set('abc'): print(item * 3)
    S = {1.23}
    S.add((1, 2, 3))
    print(S)
    # Set comprehensions集合推导式
    print({x ** 2 for x in [1, 2, 3, 4]})
    print({c * 4 for c in 'spam'})
    # 集合可以用于提取列表、字符串以及可迭代对象的差异
    print(set(dir(bytes)) - set(dir(bytearray))) # In bytes but not bytearray
    print(set(dir(bytearray)) - set(dir(bytes))) # In bytearray but not bytes
    # 可以借助集合进行顺序无关的等价性测试
    L1, L2 = [1, 3, 5, 2, 4], [2, 5, 3, 4, 1]
    print(L1 == L2)
    print(set(L1) == set(L2))
    print(sorted(L1))
    print(sorted(L1) == sorted(L2))
    print('spam' == 'asmp', set('spam') == set('asmp'), sorted('spam') == sorted('asmp'))
    ```

6. Booleans布尔型
    - True和False可看作整数1和0
    ``` python
    print(type(True))
    print(isinstance(True, int)) # isinstance() 函数来判断一个对象是否是一个已知的类型，bool实际上是只是内置整数类型int的子类。
    print(True == 1)
    print(True + 4)
    ```


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
### 一、String Basics
常见的字符串字面量和操作：

| Operation | Interpretation |
| :---- | :---- |
| S = '' | 空字符串 |
| S = "spam's" | 双引号Double quotes |
| S = 's\np\ta\x00m' | 转义序列Escape sequences |
| S = """...multiline...""" | 三引号块字符串Triple-quoted block strings |
| S = r'\temp\spam' | 原始字符串（不进行转义）Raw strings (no escapes) |
| B = b'sp\xc4m' | 字节串Byte strings in 2.6, 2.7, and 3.X (Chapter 4, Chapter 37) |
| U = u'sp\u00c4m' | unicode字符串Unicode strings in 2.X and 3.3+ (Chapter 4, Chapter 37) |
| S1 + S2 | 拼接Concatenate |
| S * 3 | 重复repeat |
| S[i] | 索引Index |
| S[i:j] | 切片slice |
| len(S) | 长度length |
| "a %s parrot" % S | 字符串格式化表达式String formatting expression |
| "a {0} parrot".format(S) | 字符串格式化方法String formatting method in 2.6, 2.7, and 3.X |
| S.find('pa') | 字符串方法String methods |
| S.rstrip() | 移除右侧空白remove whitespace |
| S.replace('pa', 'xx') | 替换replacement |
| S.split(',') | 用分隔符分组split on delimiter | 
| S.isdigit() | 内容测试content test检测字符串是否只由数字组成 |
| S.lower() | 大小写转换case conversion |
| S.endswith('spam') | 尾部测试end test |
| '-'.join('s', 'p', 'a', 'm') | 分隔符连接delimiter join |
| S.encode('latin-1') | 编码Unicode encoding |
| B.decode('utf8') | 解码Unicode decoding |
| for x in S: print(x) | 迭代Iteration |
| 'spam' in S | 成员关系membership |
| [c * 2 for c in S] | 推导式comprehensions |
| map(ord, S) | ord返回单个字符的ASCII序号 |
| re.match('sp(.*)am', '-') | 模式匹配：库模块Pattern matching: library module |

### 二、String Literals
1. 字符串字面量
    - 字符串之间没有逗号会自动拼接相邻的字符串字面量；
    - 通过反斜杠转义来嵌入引号字符：
    ``` python
    title = "Meaning " 'of' " Life"
    print(title)
    print('knight\'s', "knight\"s")
    ```

2. 转义序列***Escape*** Sequences
    - `\n`换行符newline character，`\t`制表符tab character
    ``` python
    s = 'a\nb\tc'
    print(s)
    print(len(s)) # 这个字符串有5个字符，包含了一个ASCII a字符、一个换行字符、一个ASCII b字符等。
    ```
    - 字符串反斜杠字符合集String backslash characters

| Escape | Meaning | 
| :---- | :---- | 
| \newline | 行的延续，一句代码太长了，直接回车会报错，写个\再换行，就会当做同一行处理 | 
| \\ | 反斜杠 Backslash (保留一个\\) | 
| \' | 单引号 Single quote (保留 ') | 
| \" | 双引号 Double quote (保留 ") | 
| \a | 响铃 Bell | 
| \b | 退格 Backspace | 
| \f | 换页 Formfeed | 
| \n | 换行 Newline (linefeed) | 
| \r | 回车 Carriage return | 
| \t | 水平制表符 Horizontal tab | 
| \v | 垂直制表符 Vertical tab | 
| \xhh | 十六进制 Character with hex value hh (exactly 2 digits) | 
| \ooo | 八进制 Character with octal value ooo (up to 3 digits) | 
| \0 | 空字符：二进制的0字符 | 
| \N{ id } | Unicode数据库ID（不包括前面的r） | 
| \uhhhh | 16位十六进制值的Unicode值 Unicode character with 16-bit hex value | 
| \Uhhhhhhhh | 32位的十六进制的Unicode值 |  
| \other | Not an escape (keeps both \ and other) | 

3. ***原始字符串Raw Strings***
    - 例如：`myfile = open('C:\new\text.dat', 'w')`
    - 这里因为有\n、\t存在转义机制，所以要用原始字符串：`myfile = open(r'C:\new\text.dat', 'w')`或者用2个反斜杠代替一个反斜杠：`myfile = open('C:\\new\\text.dat', 'w')`

4. **三引号编写多行块字符串Triple Quotes Code Multiline Block Strings**
    - 三引号会保留所有包围的文本，包括你以为的注释的文本：
    ``` python
    menu = """spam # comments here added to string!
    ... eggs # ditto
    ... """
    print(menu)
    ```

### 三、Strings in Action
1. 索引和分片***Indexing and Slicing***
    - `分片（S[i:j]）`：不包括上(右)边界，即S[1:3]获取从 ***偏移量(offset)*** 1到不包括偏移量为3之间的元素；`拓展的分片（S[i:j:k]）`：接受一个步长k，其默认值为+1。
    ``` python
    S = 'spam'
    print(S[0], S[-2])
    print(S[1:3], S[1:], S[:-1])
    S = 'abcdefghijklmnop'
    print(S[1:10:2])
    print(S[::2])
    ```
    - 倒序收集元素，使用负数步幅，2个边界实际进行了反转：
    ``` python
    S = 'hello'
    print(S[::-1])
    S = 'abcedfg'
    print(S[5:1:-1]) # 获取了从2到5的元素
    ```
    - `slice()`函数
    ``` python
    print('spam'[1:3])
    print('spam'[slice(1, 3)])
    print('spam'[::-1])
    print('spam'[slice(None, None, -1)])
    ```

2. 字符串转换工具
    - `int`函数；`str`函数；`repr`函数：return the canonical string representation of the object将对象转化为供解释器读取的string形式：
    ``` python
    print(int("42"), str(42))
    print(repr(42))
    print(str('spam'), repr('spam')) # repr() 的输出追求明确性，除了对象内容，还需要展示出对象的数据类型信息
    ```

3. 字符串代码转换
    - `ord()`函数是`chr()`函数对于8位的ASCII字符串或`unichr()`函数（对于Unicode对象）的配对函数，它以一个字符（长度为1的字符串）作为参数，返回对应的ASCII数值，或者Unicode数值，返回值是对应的十进制整数。ord全称为***ordinal(序数)***、chr全称为***character***。
    ``` python
    print(ord('s'), chr(115))
    ```
    - `int()`函数、`bin()`函数：int函数接受字符串或数字以及一个base（进制数，默认十进制）；bin即binary，返回整数的二进制：
    ``` python
    print(int('1101', 2))
    print(bin(13))
    ```

4. 修改字符串
    - ***字符串是不可变序列***，不能对索引进行赋值，不能`S[0]='x'`，要用拼接或分片赋值一个新的字符串或者用`replace`函数：
    ``` python
    S = 'spam'
    S = S + 'SPAM!'
    print(S)
    S = S[:4] + 'Burger' + S[-1]
    print(S)

    S = 'splot'
    S = S.replace('pl', 'pamal')
    print(S)
    ```
    - ***字符串格式化***，生成一个新的字符串：
    ``` python
    print('That is %d %s bird!' % (1, 'dead'))
    print('That is {0} {1} bird!'.format(1, 'dead'))
    ```

### 四、String Methods
1. 字符串方法
    - **方法**与特定对象相关联，是依附于对象的**属性attribute**，而这些属性引用了可调用的函数：
    ``` python
    S = 'spam'
    print(dir(S)) # 打印出来是字符串的所有属性或方法
    ```
    - 显式结果为：
    ``` python
    ['__add__', '__class__', '__contains__', '__delattr__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getitem__', '__getnewargs__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__iter__', '__le__', '__len__', '__lt__', '__mod__', '__mul__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__rmod__', '__rmul__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', 'capitalize', 'casefold', 'center', 'count', 'encode', 'endswith', 'expandtabs', 'find', 'format', 'format_map', 'index', 'isalnum', 'isalpha', 'isascii', 'isdecimal', 'isdigit', 'isidentifier', 'islower', 'isnumeric', 'isprintable', 'isspace', 'istitle', 'isupper', 'join', 'ljust', 'lower', 'lstrip', 'maketrans', 'partition', 'removeprefix', 'removesuffix', 'replace', 'rfind', 'rindex', 'rjust', 'rpartition', 'rsplit', 'rstrip', 'split', 'splitlines', 'startswith', 'strip', 'swapcase', 'title', 'translate', 'upper', 'zfill']
    ```

2. 用方法修改字符串
    - `replace`方法：
    ``` python
    S = 'spammy'
    S = S.replace('mm', 'xx')
    print(S)

    S = 'xxxxSPAMxxxxSPAMxxxx'
    print(S.replace('SPAM', 'EGGS')) # replace方法会生成一个新的字符串，因为字符串不可变。
    print(S.replace('SPAM', 'EGGS', 1))
    ```
    - `find`方法：
    ``` python
    S = 'xxxxSPAMxxxxSPAMxxxx'
    where = S.find('SPAM')
    print(where)
    S = S[:where] + 'EGGS' + S[(where+4):]
    print(S)
    ```
    - `join`方法，join将列表的字符串连在一起，并在元素间用分隔符delimiter隔开：
    ``` python
    S = 'spammy'
    L = list(S)
    print(L)
    L[3] = 'x'
    L[4] = 'x'
    print(L)
    S = ''.join(L)
    print(S)
    print('SPAM'.join(['eggs', 'sausage', 'ham', 'toast'])) # 用分隔符SPAM加入后面列表
    ```

3. 用方法解析文本Parsing Text
    - 方法`split()`将一个字符串从分隔符处切成一系列子串，默认分隔符为空格、制表符或换行符：
    ``` python
    line = 'aaa bbb ccc'
    cols = line.split()
    print(cols)
    line = 'bob,hacker,40'
    print(line.split(','))
    line = "i'mSPAMaSPAMlumberjack"
    print(line.split("SPAM"))
    ```

4. 其他常见字符串方法
    - 方法`rstrip()`，`upper()`，`isalpha()`，`endswith()`，`startswith()`，`find()`：
    ``` python
    line = "The knights who say Ni!\n"
    print(line.rstrip())
    print(line.upper())
    print(line.isalpha()) # isalpha()检测字符串是否只由字母或文字组成
    print(line.endswith('Ni!\n'))
    print(line.startswith('The'))
    print(line.find('Ni')) # find方法返回字符串的索引值
    ```
    - 可以使用`help(S.method)`的结果来得到关于方法的更多提示。

### 五、String Formatting Expressions
1. **字符串格式化表达式String Formatting Expressions**
    - `'...%s...' % (values)`：这一形式是基于C语言的printf模型，并且在现有编码中广泛使用。

2. **字符串格式化方法调用String formatting method calls**
    - `'...{}...'.format(values)`：这是Python 3.0新增的技术，起源于C#/.NET中的同名工具。

3. **f字符串f-strings**
    - `f'...{values}...'`：Python 3.6新增的。

4. 格式化表达式基础
    - 在%运算符的左侧放置一个或多个需要进行格式化的字符串，如%d，%s；在%运算符的右侧放置一个或多个（内嵌在元组中）对象；
    - %s的意思是把他们转换为字符串，因此每种对象类型都适用于%s转换，一般只需记得用%s转换就行：
    ``` python
    print('That is %d %s bird!' % (1, 'dead')) # 前面的%后面的字母见后面类型码type codes
    exclamation = 'Ni'
    print('The knights who say %s!' % exclamation)
    print('%d %s %g you' % (1, 'spam', 4.0))
    print('%s -- %s -- %s' % (42, 3.14159, [1, 2, 3]))
    ```
    运行结果如下：
    ```
    That is 1 dead bird!
    The knights who say Ni!
    1 spam 4 you
    42 -- 3.14159 -- [1, 2, 3]
    ```

5. 类型码String formatting type codes

| Code | Meaning |
| :---- | :---- |
| s | 字符串 |
| r | Same as s, but uses repr, not str |
| c | 字符Character (int or str) |
| d | 十进制数字 |
| i | 整数 |
| u | 与d相同（已废弃：不再是无符号整数） |
| o | 八进制整数（以8为底） |
| x | 十六进制整数（以16为底） |
| X | 与x相同 |
| e | 带指数的浮点数 |
| E | 与e相同 |
| f | 十进制浮点数 |
| F | 与f相同 |
| g | 浮点数e或f，g根据数字内容选择格式（如果指数小于-4或不小于精度，则使用e，其他则使用f，默认总位数精度为6） |
| G | 浮点数E或F |
| % | %字面量 Literal % (coded as %%) |

6. 格式化字符串表达式左侧的转换目标支持多种转换操作
    - 一般结构为：`%[(keyname)][flags][width][.precision]typecode`
    - `keyname`：为索引在表达式右侧使用的字典所提供的键名称
    - `flags`：说明格式的标签，如左对齐(-)、数值符号(+)、正数前的空白以及负数前的-(空格)、零填充(0)
    - `width`：最小字段宽度
    - `precision`：为浮点数设置小数点后显示的数位
    - width和precision部分都可以编写成一个*，以指定他们应该从表达式右侧的输入值中的下一项取值。
    ``` python
    x = 1234
    res = 'integers: ...%d...%-6d...%06d' % (x, x, x) # 6位的左对齐格式化和6位补零的格式化
    print(res)
    x = 1.23456789
    print('%e | %f | %g' % (x, x, x))
    print('%E' % x)
    print('%-6.2f | %05.2f | %+06.1f' % (x, x, x))
    print('%s' % x, str(x))
    ```
    运行结果如下：
    ```
    integers: ...1234...1234  ...001234
    1.234568e+00 | 1.234568 | 1.23457
    1.234568E+00
    1.23   | 01.23 | +001.2
    1.23456789 1.23456789
    ```
    - 可以在格式化字符中用一个`*`来指定宽度和精度，迫使它们的值从%运算符右边的输入中的下一项获取：
    ``` python
    print('%f, %.2f, %.*f' % (1/3.0, 1/3.0, 4, 1/3.0))
    ```
    运行结果如下：
    ```
    0.333333, 0.33, 0.3333
    ```

7. 基于字典的格式化表达式    
    - `print('%(qty)d more %(food)s' % {'qty': 1, 'food': 'spam'})`
    - 生成类似HTML或XML文本的程序往往利用这一技术:
    ``` python
    reply = """
    Greetings...
    Hello %(name)s!
    Your age is %(age)s
    """
    values = {'name': 'Bob', 'age': 40}
    print(reply % values)
    ```
    - 内置函数`vars()`，这个函数返回的字典包含了在它被调用的地方所有存在的变量，可以利用这个来访问变量：
    ``` python
    food = 'spam'
    qty = 10
    print(vars())
    print('%(qty)d more %(food)s' % vars())
    ```

### 六、String Formatting Method Calls
1. 字符串格式化方法基础
    ``` python
    template = '{0}, {1} and {2}' # 通过位置
    print(template.format('spam', 'ham', 'eggs'))
    template = '{motto}, {pork} and {food}' # 通过键
    print(template.format(motto='spam', pork='ham', food='eggs')) # 跟上一节的不一样，后面不是字典，见下面
    template = '{motto}, {0} and {food}' # 通过位置和键
    print(template.format('ham', motto='spam', food='eggs'))
    template = '{}, {} and {}' # 通过相对位置
    print(template.format('spam', 'ham', 'eggs'))
    ```

2. 比较：上一节的格式化表达式
    ``` python
    template = '%s, %s and %s'
    print(template % ('spam', 'ham', 'eggs'))
    template = '%(motto)s, %(pork)s and %(food)s'
    print(template % dict(motto='spam', pork='ham', food='eggs')) # 通过dict()函数转为字典
    ```
    - 格式化方法可以让任意的对象类型在目标上替换，有点像格式化表达式中的%s目标码：
    ``` python
    print('{motto}, {0} and {food}'.format(42, motto=3.14, food=[1, 2]))
    X = '{motto}, {0} and {food}'.format(42, motto=3.14, food=[1, 2])
    print(X.split(' and ')) # 以空格and空格为分隔符
    Y = X.replace('and', 'but under no circumstances')
    print(Y)
    ```

3. 添加键、属性和偏移量
    ``` python
    import sys
    print('My {1[kind]} runs {0.platform}'.format(sys, {'kind': 'laptop'}))
    print('My {map[kind]} runs {sys.platform}'.format(sys=sys, map={'kind': 'laptop'}))
    somelist = list('SPAM')
    print(somelist)
    print('first={0[0]}, third={0[2]}'.format(somelist))
    print('first={0}, last={1}'.format(somelist[0], somelist[-1]))
    parts = somelist[0], somelist[-1], somelist[1:3]
    print('first={0}, last={1}, middle={2}'.format(*parts)) # *是一种特殊的语法syntax，可以把一个元组展开为独立的参数。18章会更详细
    ```
    运行结果如下：
    ```
    My laptop runs win32
    My laptop runs win32
    ['S', 'P', 'A', 'M']
    first=S, third=A
    first=S, last=M
    first=S, last=M, middle=['P', 'A']
    ```

4. 高级格式化方法语法Advanced Formatting Method Syntax
    - 可以在格式化字符串中添加额外的语法来实现更具体的层级，以下是是一个字符串中的形式化格式，四个部分都为可选的，中间不能有空格：
    - `{fieldname component !conversionflag :formatspec}`
    - `fieldname`是辨识参数，位置参数或键值number or keyword；`component`是大于等于零个的.name或[index]；`converionflag`是以!开始的，后面跟着r、s或a，分别调用repr、str或ascii内置函数；`formatspec`是以:开始的，后面跟着文本指定字段宽度、对齐方式、补零、小数精度等细节；
    - `formatspec`的形式：`[[fill]align][sign][#][0][width][,][.precision][typecode]`
    - `fill`可以是除{和}之外的任意填充字符；`align`可以是<、>、=、^，分别代表左对齐、右对齐、符号字符后的填充、居中对齐；`sign`可以是+、-或空格；`逗号(,)`请求千分位分隔符；`width`、`precision`和`typecode`与%表达式一样，但`typecode`多一个二进制格式b

5. 高级格式化方法举例
    ``` python
    print('{0:10} = {1:10}'.format('spam', 123.4567)) # {0:10}指10字符宽的字段的第一个位置参数
    print('{0:>10} = {1:<10}'.format('spam', 123.4567)) # >右对齐
    print('{0.platform:>10} = {1[kind]:<10}'.format(sys, dict(kind='laptop')))
    # 省略位置参数
    print('{:10} = {:10}'.format('spam', 123.4567))
    print('{:>10} = {:<10}'.format('spam', 123.4567))
    print('{.platform:>10} = {[kind]:<10}'.format(sys, dict(kind='laptop')))
    # 浮点数
    print('{0:e}, {1:.3e}, {2:g}'.format(3.14159, 3.14159, 3.14159))
    print('{0:f}, {1:.2f}, {2:06.2f}'.format(3.14159, 3.14159, 3.14159)) # {2:06.2f}添加一个6字符宽度的字段，并在左边补充0
    # 十六进制、八进制、二进制
    print('{0:X}, {1:o}, {2:b}'.format(255, 255, 255))
    print(bin(255), int('11111111', 2), 0b11111111)
    print(hex(255), int('FF', 16), 0xFF)
    print(oct(255), int('377', 8), 0o377)
    # 格式化的参数可以通过嵌套的格式化语法从参数列表动态获取，很想格式化表达式的*语法（见7-5-4）
    print('{0:.{1}f}'.format(1 / 3.0, 4))
    print('%.*f' % (4, 1 / 3.0))
    # format函数(字符串方法的替代方案)
    print('{0:.2f}'.format(1.2345))
    print(format(1.2345, '.2f'))
    print('%.2f' % 1.2345)
    # 千分位分隔符
    print('{0:,d}'.format(999999999999))
    print('{:,.2f}'.format(296999.2567))
    ```
    运行结果如下：
    ```
    spam       =   123.4567
          spam = 123.4567
         win32 = laptop
    spam       =   123.4567
          spam = 123.4567
         win32 = laptop
    3.141590e+00, 3.142e+00, 3.14159
    3.141590, 3.14, 003.14
    FF, 377, 11111111
    0b11111111 255 255
    0xff 255 255
    0o377 255 255
    0.3333
    0.3333
    1.23
    1.23
    1.23
    999,999,999,999
    296,999.26
    ```

6. 与%格式化表达式做比较
    ``` python
    print('%s=%s' % ('spam', 42))
    print('{0}={1}'.format('spam', 42))
    print('{}={}'.format('spam', 42))
    # 和上面5进行比较
    print('%-10s = %10s' % ('spam', 123.4567))
    print('%10s = %-10s' % ('spam', 123.4567))
    print('%(plat)10s = %(kind)-10s' % dict(plat=sys.platform, kind='laptop'))
    print('%e, %.3e, %g' % (3.14159, 3.14159, 3.14159))
    print('%f, %.2f, %06.2f' % (3.14159, 3.14159, 3.14159))
    print('%x, %o' % (255, 255))
    # 其他
    print('My {1[kind]:<8} runs {0.platform:>8}'.format(sys, {'kind': 'laptop'}))
    print('My %(kind)-8s runs %(plat)8s' % dict(kind='laptop', plat=sys.platform))

    data = dict(platform=sys.platform, kind='laptop')
    print('My {kind:<8} runs {platform:>8}'.format(**data)) # **data见第十八章，它把字典解包成一组name=value的关键字参数
    print('My %(kind)-8s runs %(platform)8s' % data)
    ```
    运行结果如下：
    ```
    spam=42
    spam=42
    spam=42
    spam       =   123.4567
          spam = 123.4567
         win32 = laptop
    3.141590e+00, 3.142e+00, 3.14159
    3.141590, 3.14, 003.14
    ff, 377
    My laptop   runs    win32
    My laptop   runs    win32
    My laptop   runs    win32
    My laptop   runs    win32
    ```

### 七、Why the Format Method?
1. 字符串格式化方法支持表达式一些额外的功能
    - 例如**二进制类型码binary type codes**和**千分位分组thousands groupings**，但字符串表达式不支持二进制，即没有2进制的类型码，见7-5，`print('%b' % ((2 ** 16) - 1))`
    ``` python
    print('{0:b}'.format((2 ** 16) - 1))
    print(bin((2 ** 16) - 1))
    print('%s' % bin((2 ** 16) - 1))
    print('{}'.format(bin((2 ** 16) - 1)))
    print('%s' % bin((2 ** 16) - 1)[2:])
    # 字符串表达式不支持千分位分组
    print('{:,d}'.format(999999999999))
    ```
    运行结果如下：
    ```
    1111111111111111
    0b1111111111111111
    0b1111111111111111
    0b1111111111111111
    1111111111111111
    999,999,999,999
    ```

2. 基于字典，2种方法之间的比较
    ``` python
    print('{name} {job} {name}'.format(name='Bob', job='dev'))
    print('%(name)s %(job)s %(name)s' % dict(name='Bob', job='dev'))
    D = dict(name='Bob', job='dev')
    print('{0[name]} {0[job]} {0[name]}'.format(D))
    print('{name} {job} {name}'.format(**D))
    print('%(name)s %(job)s %(name)s' % D)
    ```
    运行结果全为`Bob dev Bob`。

## chapter 8 Lists and Dictionaries
### 一、Lists
1. 列表的特性
    - 任意对象的有序集合；通过偏移访问；可变长度、异构以及任意嵌套Variable-length, heterogeneous, and arbitrarily nestable；
    - 属于**可变序列**的分类；对象引用数组。

2. 常用列表字面量和操作

| Operation | Interpretation |
| :---- | :---- |
| L = [] | 空列表 |
| L = [123, 'abc', 1.23, {}] | Four items: indexes 0..3 |
| L1 = ['Bob', 40.0, ['dev', 'mgr']] | 嵌套子列表Nested sublists |
| L2 = list('spam') | 可迭代对象元素的列表List of an iterable’s items |
| L2 = list(range(-4, 4)) | list of successive integers |
| L[i] | 索引 index |
| L[i][j] | 索引的索引 index of index |
| L[i:j] | 分片 slice |
| len(L) | length |
| L1 + L2 | 拼接 Concatenate |
| L * 3 | 重复 Repeat |
| for x in L: print(x) | 迭代 Iteration |
| 3 in L | 成员关系 membership |
| L.append(4) | 列表末尾添加新的对象 |
| L.extend([5,6,7]) | 用新列表扩展原来的列表 |
| L.insert(i, X) | 插入 |
| L.index(X) | 查找索引 |
| L.count(X) | 统计元素出现次数 |
| L.sort() | 排序 |
| L.reverse() | 反转 |
| L.copy() | 复制 |
| L.clear() | 清除 |
| L.pop(i) | 弹出并返回元素 |
| L.remove(X) | 删除元素 |
| del L[i] | 删除 |
| del L[i:j] | 删除切片 |
| L[i:j] = [] | 删除切片 |
| L[i] = 3 | 索引赋值 |
| L[i:j] = [4,5,6] | 分片赋值 |
| L = [x**2 for x in range(5)] | 列表推导List comprehensions |
| list(map(ord, 'spam')) | 映射 map |

### 二、Lists in Action
1. 基本列表操作：拼接与重复
``` python
print(len([1, 2, 3]))
print([1, 2, 3] + [4, 5, 6])
print(['Ni!'] * 4)
```

2. **列表迭代和推导List Iteration and Comprehensions**
``` python
print(3 in [1, 2, 3])
for x in [1, 2, 3]:
    print(x, end=' ')
res = [c * 4 for c in 'SPAM']
print(res)
# map()会根据提供的函数对指定序列做映射
print(list(map(abs, [-1, -2, 0, 1, 2])))
```
显式结果如下：
```
True
1 2 3 ['SSSS', 'PPPP', 'AAAA', 'MMMM']
[1, 2, 0, 1, 2]
```

3. **索引、分片和矩阵Indexing, Slicing, and Matrixes**
    - 对列表进行**分片**会返回一个新的列表：
    ``` python
    L = ['spam', 'Spam', 'SPAM!']
    print(L[2])
    print(L[-2])
    print(L[1:])
    ```
    运行结果如下：
    ```
    SPAM!
    Spam
    ['Spam', 'SPAM!']
    ```
    - **矩阵和嵌套**：
    ``` python
    matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
    print(matrix[1])
    print(matrix[1][1])
    print(matrix[2][0])
    matrix = [[1, 2, 3],
            [4, 5, 6],
            [7, 8, 9]]
    print(matrix[1][1])
    ```

4. 原位置修改列表
    - **索引**和**分片**的**赋值**都直接修改列表，而不是生成一个新的列表；
    - 分片赋值slice assignment可分成2步理解：（1）删除等号左边指定分片；（2）将等号右边插入被删除的部分；
    - 分片赋值可用于替换、增长、删除
    ``` python
    L = ['spam', 'Spam', 'SPAM!']
    L[1] = 'eggs'
    print(L)
    L[0:2] = ['eat', 'more']
    print(L)

    L = [1, 2, 3, 4, 5, 6, 7]
    L[2:5] = L[3:6] # 右侧在左侧被执行删除前就被取出来了
    print(L)
    L = [1, 2, 3]
    L[1:2] = [4, 5] # 插入的元素数目不需要与被删除的数目相匹配
    print(L)
    L[1:1] = [6, 7] # 将1：1之间的空白切片插入6、7
    print(L)
    L[1:2] = []
    print(L)
    L = [1]
    L[:0] = [2, 3, 4] # 在最前面插入
    print(L)
    L[len(L):] = [5, 6, 7] # 在最后面插入
    print(L)
    L.extend([8, 9, 10])
    print(L)
    ```
    运行结果如下：
    ``` python
    ['spam', 'eggs', 'SPAM!']
    ['eat', 'more', 'SPAM!']
    [1, 2, 4, 5, 6, 6, 7]
    [1, 4, 5, 3]
    [1, 6, 7, 4, 5, 3]
    [1, 7, 4, 5, 3]
    [2, 3, 4, 1]
    [2, 3, 4, 1, 5, 6, 7]
    [2, 3, 4, 1, 5, 6, 7, 8, 9, 10]
    ```

5. 列表方法调用
    - `L.append(X)`跟`L+[X]`类似，不同的是前者原位置修改L，而后者会生成新的列表：
    ``` python
    L = ['eat', 'more', 'SPAM!']
    L.append('please')
    print(L)
    ```
    - `sort`方法，可以传入关键词参数对sort行为进行修改：
    ``` python
    L = ['abc', 'ABD', 'aBe']
    L.sort()
    print(L)
    L = ['abc', 'ABD', 'aBe']
    L.sort(key=str.lower) # 小写后排序
    print(L)
    L = ['abc', 'ABD', 'aBe']
    L.sort(key=str.lower, reverse=True)
    print(L)
    ```
    - `append`和`sort`方法不会返回列表对象，返回值都是`None`。
    - `sorted`函数，返回一个新的列表：
    ``` python
    L = ['abc', 'ABD', 'aBe']
    print(sorted(L, key=str.lower, reverse=True))
    L = ['abc', 'ABD', 'aBe']
    print(sorted([x.lower() for x in L], reverse=True))
    ```

6. 其他常见列表方法或操作
    - `extend`方法是循环访问传入的可迭代对象，并逐个把产生的元素添加到列表尾部，而append是直接将传入的可迭代对象添加到尾部。详见第14章。
    ``` python
    L = [1, 2]
    L.extend([3, 4, 5])
    print(L)
    ```
    - `pop`方法弹出并返回最后最后一个元素，并且能够接受被删除并返回的元素的偏移量（默认为-1）：
    ``` python
    print(L.pop())
    print(L)
    ```
    - `reverse`方法和`reversed`函数：
    ``` python
    L.reverse()
    print(L)
    print(reversed(L)) # 结果是一个能够按需产生值的迭代器
    print(list(reversed(L)))
    ```
    - `pop`方法和`append`方法联用，以实现快速的**后进先出栈结构last in-first-out (LIFO) stack structure**，列表的末端作为**栈**的顶端：
    ``` python
    L = []
    L.append(1) # Push onto stack
    L.append(2)
    print(L)
    print(L.pop()) # Pop off stack
    print(L)
    ```
    - `remove`、`insert`、`count`、`index`方法：
    ``` python
    L = ['spam', 'eggs', 'ham']
    print(L.index('eggs')) # 查找某元素的偏移
    L.insert(1, 'toast') # 在偏移处插入某元素
    print(L)
    L.remove('eggs')
    print(L)
    print(L.count('spam')) # 计算元素出现的次数
    ```
    - `del`语句del statement：
    ``` python
    L = ['spam', 'eggs', 'ham', 'toast']
    del L[0]
    print(L)
    del L[1:]
    print(L)
    ```

### 三、Dictionaries
1. 字典的特性
    - 字典有时叫做关联数组associative arrays或散列表hashes；
    - 通过键而不是偏移量来读取；任意对象的无序集合（以前无序，从python3.6开始，dict的插入变为有序，即字典整体变的有序）；
    - 长度可变、异构、任意嵌套Variable-length, heterogeneous, and arbitrarily nestable；属于可变映射类型Of the category “mutable mapping”；对象引用表（散列表）。

2. 常见字典字面量和操作

| Operation | Interpretation |
| :---- | :---- |
| D = {} | 空字典 |
| D = {'name': 'Bob', 'age': 40} | 字典 |
| E = {'cto': {'name': 'Bob', 'age': 40}} | 嵌套 Nesting |
| D = dict(name='Bob', age=40) | dict函数 |
| D = dict([('name', 'Bob'), ('age', 40)]) | 键值对 |
| D = dict(zip(keyslist, valueslist)) | 拉链式键值对zipped key/value pairs |
| D = dict.fromkeys(['name', 'age']) | 键列表 |
| D['name'] | 键索引 |
| E['cto']['age'] |  嵌套索引 |
| 'age' in D | 成员关系：键存在测试 |
| D.keys() | 方法：所有键 |
| D.values() | 所有值 |
| D.items() | 所有键值元组 |
| D.copy() | 复制copy (top-level) |
| D.clear() | 清除 |
| D.update(D2) | 通过键合并 merge by keys |
| D.get(key, default) | 通过键获取，如果不存在返回default或None |
| D.pop(key, default) | 通过键删除，如果不存在返回default或错误 |
| D.setdefault(key, default) | 通过键获取，如果不存在default设置为None |
| D.popitem() | 删除/返回键值对 |
| len(D) | 长度，存储的键值对的对数 |
| D[key] = 42 | 增加键值对 |
| del D[key] | 删除键值对 |
| list(D.keys()) | 查看键 |
| D = {x: x*2 for x in range(10)} | 字典推导 |

### 四、Dictionaries in Action
1. 字典的基本操作
``` python
D = {'spam': 2, 'ham': 1, 'eggs': 3}
print(D['spam'])
print(D)
print(len(D))
print('ham' in D) # 直接测试键是否存在
print(D.keys()) # 返回的是一个可迭代对象
print(list(D.keys()))
```

2. 原位置修改字典
``` python
D['ham'] = ['grill', 'bake', 'fry'] # 修改元素
print(D)
del D['eggs'] # 删除元素
print(D)
D['brunch'] = 'Bacon' # 新增元素
print(D)
```

3. 其他字典方法
    - `values`方法和`items`方法，都返回可迭代对象，values返回值，items返回键值对的元组：
    ``` python
    D = {'spam': 2, 'ham': 1, 'eggs': 3}
    print(list(D.values()))
    print(list(D.items()))
    ```
    - `get`方法通过键获取值，若键不存在，则返回默认值None或指定的默认值：
    ``` python
    print(D.get('spam')) # get方法通过键获取值，若键不存在，则返回默认值None或指定的默认值
    print(D.get('toast'))
    print(D.get('toast', 88))
    ```
    - `update`方法，类似于拼接：
    ``` python
    D = {'eggs': 3, 'spam': 2, 'ham': 1}
    D2 = {'toast':4, 'muffin':5}
    D.update(D2)
    print(D)
    ```
    - `pop`方法删除一个键并返回它的值：
    ``` python
    print(D.pop('muffin'))
    print(D.pop('toast'))
    print(D)
    ```
    - 遍历字典的键列表：
    ``` python
    table = {'1975': 'Holy Grail', 
        '1979': 'Life of Brian',
        '1983': 'The Meaning of Life'}
    for year in table:
        print(f'{year}\t{table[year]}')
    ```
    - 推导语法，从值到键；在字典中可能一个值会有多个键与之对应：
    ``` python
    table = {'Holy Grail': '1975', 
        'Life of Brian': '1979',
        'The Meaning of Life': '1983'}
    print(list(table.items()))
    print([title for (title, year) in table.items() if year == '1975']) # 详见14章、32章
    ```
    - copy方法在第9章进行讨论。

4. 用整数、元组作键
``` python
table = {1975: 'Holy Grail',
    1979: 'Life of Brian', 
    1983: 'The Meaning of Life'}
Matrix = {}
Matrix[(2, 3, 4)] = 88
Matrix[(7, 8, 9)] = 99
print(Matrix)
```

5. 字典的嵌套
``` python
rec = {'name': 'Bob',
    'jobs': ['developer', 'manager'],
    'web': 'www.bobs.org/˜Bob',
    'home': {'state': 'Overworked', 'zip': 12345}}
print(rec['jobs'][1])
print(rec['home']['zip'])
```

6. dict函数创建字典
``` python
print(dict(name='Bob', age=40))
print(dict([('name', 'Bob'), ('age', 40)]))
# zip函数，详见13、14章
keyslist = ['name', 'age']
valueslist = ['Bob', 40]
print(dict(zip(keyslist, valueslist)))
print(zip(keyslist, valueslist))
print(list(zip(['a', 'b', 'c'], [1, 2, 3])))
print(dict(zip(['a', 'b', 'c'], [1, 2, 3])))
print({k: v for (k, v) in zip(['a', 'b', 'c'], [1, 2, 3])}) # 字典推导表达式
print({x: x ** 2 for x in [1, 2, 3, 4]})
# fromkeys方法将同一值传入键列表，默认值为None
print(dict.fromkeys(['a', 'b'], 0))
print(dict.fromkeys('spam'))
```

7. 字典视图Dictionary views  
字典的keys, values和items方法都返回***视图对象view objects***：**视图对象**是**可迭代对象**，每次只产生一个结果项的对象，而不是在内存中立即产生结果列表。
``` python
D = dict(a=1, b=2, c=3)
print(D.keys())
print(D.values())
print(D.items())
print(list(D.keys()), list(D.values()), list(D.items()))
for k in D.keys(): print(k)
for key in D: print(key)
# 对字典的改变会改变视图对象
D = {'a': 1, 'b': 2, 'c': 3}
K = D.keys()
V = D.values()
del D['b']
print(list(K), list(V))
```

8. 字典视图和集合
    - `keys`方法返回的视图对象类似于集合，支持交集和并集等操作：
    ``` python
    D = {'a': 1, 'b': 2, 'c': 3}
    print(D.keys() & D.keys())
    print(D.keys() & {'b'})
    print(D.keys() & {'b': 1})
    print(D.keys() | {'b', 'c', 'd'})
    ```
    运行结果如下：
    ``` python
    {'c', 'b', 'a'}
    {'b'}
    {'b'}
    {'a', 'c', 'd', 'b'}
    ```
    - `values`视图不支持，因为值不唯一，但`items`可以，因为键值对是唯一的，并且是**可散列的hashable**(具有**不变性的immutable**)：
    ``` python
    D = {'a': 1, 'b': 1, 'c': 1}
    print(D.values() & {1})  # TypeError: unsupported operand type(s) for &: 'dict_values' and 'set'
    print(D.items() & D.items())
    ```
    运行结果如下：
    ``` python
    {('c', 1), ('b', 1), ('a', 1)}
    ```
    - 如果字典项视图是**可散列**的话，也就是，只包括**不可变的对象**的话，他们类似于集合；Items views are set-like too if they are **hashable**—that is, if they contain only **immutable** objects：
    ``` python
    D = {'a': 1}
    print(list(D.items()))
    print(D.items() | D.keys())
    print(D.items() | D)
    print(D.items() | {('c', 3), ('d', 4)})
    print(dict(D.items() | {('c', 3), ('d', 4)}))
    ```
    运行结果如下：
    ``` python
    [('a', 1)]
    {'a', ('a', 1)}
    {'a', ('a', 1)}
    {('c', 3), ('a', 1), ('d', 4)}
    {'c': 3, 'a': 1, 'd': 4}
    ```
    - 视图对象不能用方法：
    ``` python
    D = {'b': 2, 'c': 3, 'a': 1}
    Ks = D.keys()
    Ks.sort() # AttributeError: 'dict_keys' object has no attribute 'sort'
    # 要排序用sorted函数
    for k in sorted(Ks): print(k, D[k])
    for k in sorted(D): print(k, D[k])
    ```


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
1. 元组的基本操作
``` python
T = (1, 2, 3, 4)
print(T[0], T[1:3])
```

2. 逗号和圆括号  
python允许忽略元组的圆括号，但建议一直使用圆括号；*单个元素的元组如果没逗号，不是元组*：
``` python
x = (40)
print(type(x)) # 不加逗号，x为整数
y = (40,)
print(type(y))
```

3. 转换、方法或不可变性
    - 因为元组是**不可变对象**，需要转为列表再用`sort方法`排序：
    ``` python
    T = ('cc', 'aa', 'dd', 'bb')
    tmp = list(T)
    tmp.sort()
    T = tuple(tmp)
    print(T)
    ```
    - `sorted函数`接受任何序列对象，返回的是一个新的list：
    ``` python
    T = ('cc', 'aa', 'dd', 'bb')
    print(sorted(T))
    ```
    - **列表推导**总是会创建新的列表，可以用来遍历元组、字符串在内的任何序列对象：
    ``` python
    T = (1, 2, 3, 4, 5)
    L = [x + 20 for x in T]
    print(L)
    X = (x + 20 for x in T)
    print(X)
    ```
    - `index方法`和`count方法`：
    ``` python
    T = (1, 2, 3, 2, 4, 2)
    print(T.index(2)) # 第一个2的偏移量
    print(T.index(2, 2)) # 偏移量2后面的2的偏移量
    print(T.count(2))
    ```
    - 元组的不可变性只适用于本组本身顶层而并非其内容，比如*元组内列表可以修改*：
    ``` python
    T = (1, [2, 3], 4)
    T[1][0] = 'spam'
    print(T)
    ```

4. 有名元组Named Tuples
    - `namedtuple`工具（`collections`模块）实现了一个增加了逻辑的元组拓展类型，能同时支持使用序号和属性名访问组件，也可以使用键的类字典形式访问；有名元组的属性名来自类，因此与键不完全相同：
    ``` python
    from collections import namedtuple
    Rec = namedtuple('Rec', ['name', 'age', 'jobs'])
    bob = Rec('Bob', age=40.5, jobs=['dev', 'mgr'])
    print(bob)
    print(bob[0], bob[2])
    print(bob.name, bob.jobs)
    ```
    运行结果如下：
    ``` python
    Rec(name='Bob', age=40.5, jobs=['dev', 'mgr'])
    Bob ['dev', 'mgr']
    Bob ['dev', 'mgr']
    ```
    - 在需要时，可以转换成一个类字典的基于键的形式：
    ``` python
    O = bob._asdict()
    print(O['name'], O['jobs'])
    print(O)
    ```
    运行结果如下：
    ``` python
    Bob ['dev', 'mgr']
    {'name': 'Bob', 'age': 40.5, 'jobs': ['dev', 'mgr']}
    ```
    - 元组赋值，元组和有名元组都支持**解包元组赋值**，详见第13章：
    ``` python
    bob = Rec('Bob', 40.5, ['dev', 'mgr'])
    name, age, jobs = bob
    print(name, jobs)
    for x in bob: print(x)
    ```
    运行结果如下：
    ``` python
    Bob ['dev', 'mgr']
    Bob
    40.5
    ['dev', 'mgr']
    ```


### 三、Files
1. 常见的文件操作

| Operation | Interpretation |
| :---- | :---- |
| output = open(r'C:\spam', 'w') | 创建输出文件，防止出现字符串转义要用原始字符串r |
| input = open('data', 'r') | 创建输入文件 |
| input = open('data') | 与上一行一样，r是默认值 |
| aString = input.read() | 把整个文件读入一个字符串 |
| aString = input.read(N) | 读取接下来的N个字符到一个字符串 |
| aString = input.readline() | 读取下一行（包括\n换行符）到一个字符串 |
| aList = input.readlines() | 读取整个文件到一个字符串列表（包括\n换行符） |
| output.write(aString) | 把字符串写入文件 |
| output.writelines(aList) | 把列表内所以字符串写入文件 |
| output.close() | 手动关闭（当文件收集完成时会替你关闭文件） |
| output.flush() | 把输出缓冲区刷入硬盘中，但不关闭文件 |
| anyFile.seek(N) | 将文件位置移动到偏移量N处以便进行下一个操作 |
| for line in open('data'): use line | 文件迭代器逐行读取 |
| open('f.txt', encoding='latin-1') | Unicode文本文件 |
| open('f.bin', 'rb') | 字节码文件 |

2. 打开文件
    - `open函数`：`afile = open(filename, mode)`
    - `open函数`的第二个参数是处理模式，`'r'`以输入模式打开文件（默认值），`'w'`以输出模式生成并打开文件，`'a'`表示在文件尾部追加内容并打开文件；
    - 在模式字符串中加上`'b'`可以进行二进制数据处理；加上`'+'`意味着被打开文件同时支持输入输出；
    - `open函数`的前2个参数必须是字符串。第三个为可选参数，它能够用来控制输出缓冲：传入`0`意味着输出无缓冲（写入方法调用时立即传入外部文件）；
    - `open函数`的其他参数可用于特殊种类的文件。

3. 使用文件的基础用法的提示：
    - 文件迭代器最适合逐行读取，见第14章；
    - 内容是字符串，不是对象：从文件读取的数据回到脚本时是一个字符串；
    - 文件是被缓冲的以及可定位的：默认情况下，输出文件总是被缓冲的，这意味着写入的文本不能不会立即自动从内存转移到硬盘，除非关闭一个文件，或者运行其flush方法，才能强制被缓冲的数据进入硬盘；
    - `close`通常是可选的：回收时自动关闭：但手动关闭调用在大型程序中通常是不错的习惯，当循环中有许多文件被打开时，可能需要在垃圾回收机制发挥作用前，调用close释放资源。


### 四、Files in Action
1. 文件的基本操作
``` python
myfile = open('9-Tuples, Files, and Everything Else\myfile.txt', 'w')
myfile.write('hello text file\n') # 该方法会返回写入的字符数16
myfile.write('goodbye text file\n') # 该方法会返回写入的字符数18
myfile.close()
myfile = open('9-Tuples, Files, and Everything Else\myfile.txt')
print(myfile.readline())
print(myfile.readline())
print(myfile.readline())
# read方法
print(open('9-Tuples, Files, and Everything Else\myfile.txt').read())
# 文件迭代器
for line in open('9-Tuples, Files, and Everything Else\myfile.txt'):
    print(line, end='')
```

2. 获取当前文件的路径：`os模块`
    - `os.getcwd()方法`，cwd：current working directory：
    ``` python
    import os
    print(os.getcwd())
    ```
    - `os.chdir()方法`改变默认路径，chdir：change directory：`os.chdir('C:\Users\XXXXXXXXXXXXXXXXXX')`

3. 文本文件和二进制文件Text and Binary Files  
文本文件把内容表示为常规的str字符串，自动执行Unicode编码和解码，并且默认执行末行转换；二进制文件把内容表示为一个特殊的bytes字节串类型，并且允许程序不修改地访问内容；二进制文件不会对数据执行任何换行符转换，而文本文件会默认执行对\n的来回转换，并采用Unicode编码。详见第37章：
``` python
data = open('9-Tuples, Files, and Everything Else\data.bin', 'rb').read()
print(data)
print(data[4:8])
print(data[4:8][0]) # s的ASCII码是115
print(bin(data[4:8][0]))
```

4. 在文件中存储Python对象：转换
    - 文件数据在脚本中一定是字符串，需要使用转换工具把对象转为字符串：
    ``` python
    X, Y, Z = 43, 44, 45
    S = 'Spam'
    D = {'a': 1, 'b': 2}
    L = [1, 2, 3]
    F = open('9-Tuples, Files, and Everything Else\datafile.txt', 'w')
    F.write(S + '\n')
    F.write('%s,%s,%s\n' % (X, Y, Z))
    F.write(str(L) + '$' + str(D) + '\n')
    F.close()
    chars = open('9-Tuples, Files, and Everything Else\datafile.txt').read()
    print(chars)
    ```
    - `rstrip方法`，删除\n：
    ``` python
    F = open('9-Tuples, Files, and Everything Else\datafile.txt')
    line = F.readline()
    print(line.rstrip())
    ```
    - `spilt方法`：
    ``` python
    line = F.readline()
    print(line)
    parts = line.split(',')
    print(parts)
    numbers = [int(P) for P in parts] # int函数去除\n
    print(numbers)
    ```
    - `eval函数`，来执行一个字符串表达式，并返回表达式的值。把字符串当作可执行程序代码：
    ``` python
    line = F.readline()
    print(line)
    parts = line.split('$')
    print(parts)
    print(eval(parts[0]))
    objects = [eval(P) for P in parts] # eval函数去除\n
    print(objects)
    ```

5. 存储原生对象：pickle模块 (Storing Native Python Objects: pickle)  
`pickle模块`能够让我们直接在文件中存储任何python对象的高级工具，同时并不需要对字符串进行来回转换。可视为通用的数据格式化和解析工具；pickle模块会执行所谓的对象序列化，也就是对象与字节字符串之间的相互转换：
``` python
D = {'a': 1, 'b': 2}
F = open('9-Tuples, Files, and Everything Else\datafile.pkl', 'wb')
import pickle
pickle.dump(D, F)
F = open('9-Tuples, Files, and Everything Else\datafile.pkl', 'rb')
E = pickle.load(F)
print(E)
```

6. 用JSON格式存储Python对象：json标准库  
由于JSON与Python中字典和列表在语法上的相似性，因此`json标准库`能够很容易地在python对象与JSON格式之间来回转换：
``` python
name = dict(first='Bob', last='Smith')
rec = dict(name=name, job=['dev', 'mgr'], age=40.5)
print(rec)
import json
S = json.dumps(rec)
print(S)
O = json.loads(S)
print(O)
print(O == rec)

json.dump(rec, fp=open(r'9-Tuples, Files, and Everything Else\testjson.txt', 'w'), indent=4)
print(open(r'9-Tuples, Files, and Everything Else\testjson.txt').read())
P = json.load(open(r'9-Tuples, Files, and Everything Else\testjson.txt'))
print(P)
```

7. 存储打包二进制数据：struct模块  
`struct模块`能够构造并解析打包二进制数据，要生成一个打包二进制数据文件，可以用'wb'（写入二进制）模式打开它，并将一个格式化字符串和几个对象传给struct，python会创建一个我们通常写入文件的二进制bytes数据字节码，主要由不可打印字符的十六进制转义组成：
``` python
F = open('9-Tuples, Files, and Everything Else\data.bin', 'wb')
import struct
data = struct.pack('>i4sh', 7, b'spam', 8)
print(data)
F.write(data)
F.close()

F = open('9-Tuples, Files, and Everything Else\data.bin', 'rb')
data = F.read()
print(data)
values = struct.unpack('>i4sh', data)
print(values)
```

8. 文件上下文管理器File Context Managers  
*它可以把文件处理代码包装到一个逻辑层中，以确保在退出后自动关闭文件，而不是依赖于垃圾回收时的自动关闭*，详见34章：
``` python
with open(r'C:\code\data.txt') as myfile:
    for line in myfile:
```

### 五、Core Types Review and Summary
1. 类型分类和要点
    - 字符串、列表和元组都拥有如拼接、长度和索引等序列操作；
    - 只有可变对象（列表、字典和集合）可以在原位置修改；不能原位置修改数字、字符串或元组；
    - 文件只导出方法，因此可变性并不真的适用它们；
    - 数字包括整数、浮点数、复数、小数和分数；
    - 字符串包括str，以及bytes字节串；bytearray字符串类型是可变的；
    - 集合是一个没有值只有键的字典，不能映射，没有顺序，因此不是映射或序列类型，frozenset是集合的一种拥有不可变性的变体。

2. 对象类型

| 对象类型 | 分类 | 是否可变 |
| :---- | :---- | :---- |
| 数字 | 数值 | 否 |
| 字符串 | 序列 | 否 |
| 列表 | 序列 | 是 |
| 字典 | 映射 | 是 |
| 元组 | 序列 | 否 |
| 文件 | 拓展 | N/A |
| 集合 | 集合 | 是 |
| frozenset | 集合 | 否 |
| bytearray | 序列 | 是 |

3. 对象灵活性  
列表、字典和元组可以包括任何种类的对象；集合可以包含任意的**不可变类型immutable对象（可哈希的hashable）**；列表、字典和元组可以任意嵌套；列表、字典和集合可以动态地扩大和缩小。
``` python
L = ['abc', [(1, 2), ([3], 4)], 5]
print(L[1])
print(L[1][1])
print(L[1][1][0])
print(L[1][1][0][0])
```

4. 引用vs复制
    - 由于赋值会产生相同对象的多个引用，因此原位置修改可变对象时，可能会影响其他程序，特别是当涉及嵌套时，关系会复杂：
    ``` python
    X = [1, 2, 3]
    L = ['a', X, 'b']
    D = {'x':X, 'y':2}
    上面为第一行的列表创建了3个引用，由于列表是可变的，修改对象会改变另外2个
    X[1] = 'surprise'
    print(L)
    print(D)
    ```
    - 复制的方法：i、无参数分片表达式(L[:])；ii、字典、集合或列表的copy方法；iii、list、dict、set函数；iv、copy标准库模块。补：无参数的分片和字典的copy方法只能进行顶层复制，即不能复制嵌套的数据结构，如果要复制深层嵌套的数据结构，要用copy模块的deepcopy方法，见6-2章节。
    ``` python
    L = [1,2,3]
    D = {'a':1, 'b':2}
    A = L[:]
    B = D.copy()
    A[1] = 'Ni'
    B['c'] = 'spam'
    print(L, D)
    print(A, B)
    # 就中最开始的例子来说，用分片即可避免共同引用
    X = [1, 2, 3]
    L = ['a', X[:], 'b']
    D = {'x':X[:], 'y':2}
    ```

5. 比较、等价性和真值Comparisons, Equality, and Truth  
当嵌套对象存在时，python能够自动遍历数据结构，并从左到右地应用比较，这被称为**递归比较recursive comparison**，见第19章。就核心类型而言，递归功能是默认实现的。例如，比较列表对象将自动比较所有内容：
    - ==运算符测试值的等价性；is表达式测试对象的同一性：
    ``` python
    L1 = [1, ('a', 3)]
    L2 = [1, ('a', 3)]
    print(L1 == L2, L1 is L2) # L1和L2相等但不是同一个对象
    ```
    - 理论上下面应该是不同对象相同值，因为python内部会对临时存储并重复使用字符串做优化，所以事实上内存中只有一个字符串'spam'：
    ``` python
    S1 = 'spam'
    S2 = 'spam'
    print(S1 == S2, S1 is S2)
    ```
    - 相对大小比较也能递归地应用于嵌套的数据结构：
    ``` python
    L1 = [1, ('a', 3)]
    L2 = [1, ('a', 2)]
    print(L1 < L2, L1 == L2, L1 > L2)
    ```

6. 比较不同类型
    - 数字比较数值的相对大小；
    - 字符串按照字母字典顺序比较（按照ord函数返回字符集编码顺序）字符从左到右比较（'abc'<'ac'）；
    - 列表和元组从左到右，而是对嵌套结构是递归的（[2]>[1,2]）；
    - 集合相对大小采用子集超集的标准；
    - 字典通过比较后的(key,value)来判断是否相同，但不支持相对大小比较。

7. True和False  
***整数0为真，整数1为假；空数据结构为假，非空数据结构为真***，比如`if X != ''`:是为了测试对象是否包括内容：
``` python
print(bool(1))
print(bool('spam'))
print(bool({}))
```

8. None对象
`None`为一个特殊对象，可被认为是假。一般起到一个空占位符的作用；但`None`不意味着未定义，`None`是一个对象，而不是没有内容。

9. 类型测试
    - 严格来说，对象的类型本身，也属于type类型的对象：
    ``` python
    print(type(type(1)))
    ```
    - 类型测试及`isinstance函数`：
    ``` python
    print(type([1]) == type([]))
    print(type([1]) == list)
    print(isinstance([1], list))
    ```
    - `types模块`提供了不能作为内置类型使用的类型的名称：
    ``` python
    import types
    def f(): pass
    print(type(f) == types.FunctionType)
    ```

### 六、Built-in Type Gotchas
1. 序列重复
    - 序列重复就是多次将序列加在自己身上：
    ``` python
    L = [4, 5, 6]
    X = L * 4
    Y = [L] * 4
    print(X)
    print(Y)
    ```
    - 由于L在赋值给Y时是被嵌套的，因此Y中包含了指向原本L的列表的引用：
    ``` python
    L[1] = 0
    print(X)
    print(Y)
    ```
    - 解决上面的问题与之前一样，复制个副本：
    ``` python
    L = [4, 5, 6]
    Y = [list(L)] * 4
    L[1] = 0
    print(Y)
    ```
    - 尽管Y与L不再共用同一个列表对象，但Y中的嵌套的列表都指向同一对象：
    ``` python
    Y[0][1] = 99 # 所有四个都被改变了
    print(Y)
    ```
    - 避免上述共享，需要保证每个嵌套都有一个单独副本：
    ``` python
    L = [4, 5, 6]
    Y = [list(L) for i in range(4)]
    print(Y)
    Y[0][1] = 99 # 这样就只改变了第一个
    print(Y)
    ```

2. 循环数据结构Cyclic Data Structures
如果一个复合对象包含指向自身的引用，就称之为循环对象。python在对象中检测到循环，都会打印成[...]，除非真的需要，不建议使用循环引用：
``` python
L = ['grail']
L.append(L)
print(L)
```

3. 不可变类型不可以在原位置改变
不能在原位置改变不可变对象，必须通过分片、拼接等操作来创建一个新的对象，再赋值回原来的引用