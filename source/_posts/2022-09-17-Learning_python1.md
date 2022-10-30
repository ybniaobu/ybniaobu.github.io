---
title: 《Learning Python》读书笔记（一）
date: 2022-09-17 17:24:20
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
> 本篇主要内容为：Python解释器；Python对象类型的初步介绍；数值类型。

![learning_python.jpg](https://s2.loli.net/2022/09/17/ri9Ue6nguJdq1Ca.jpg)

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
    | :-----| :-----|
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
    | x | y | 按位与、集合交集Bitwise OR, set union |
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