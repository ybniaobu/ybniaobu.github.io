---
title: 《Learning Python》读书笔记（二）
date: 2022-10-24 13:30:33
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
> 本篇主要内容为：共享引用；字符串；列表；字典。

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