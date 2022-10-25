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
    - \n换行符newline character，\t制表符tab character
    ``` python
    s = 'a\nb\tc'
    print(s)
    print(len(s)) # 这个字符串有5个字符，包含了一个ASCII a字符、一个换行字符、一个ASCII b字符等。
    ```
    - 字符串反斜杠字符合集String backslash characters

    | Escape | Meaning |
    | :---- | :---- |
    | \newline | 行的延续，一句代码太长了，直接回车会报错，写个\再换行，就会当做同一行处理 |
    | \\ | 反斜杠 Backslash (保留一个\) |
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

3. 原始字符串Raw Strings
    - 例如：`myfile = open('C:\new\text.dat', 'w')`
    - 这里因为有\n、\t存在转义机制，所以要用原始字符串：`myfile = open(r'C:\new\text.dat', 'w')`或者用2个反斜杠代替一个反斜杠：`myfile = open('C:\\new\\text.dat', 'w')`

4. 三引号编写多行块字符串Triple Quotes Code Multiline Block Strings
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
    



