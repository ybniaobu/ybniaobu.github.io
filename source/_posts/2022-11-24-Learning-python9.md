---
title: 《Learning Python》读书笔记（九）
date: 2022-11-24 14:08:00
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
> 本篇主要内容为：第7部分异常和第8部分高级主题：unicode编码、字节字符串、被管理的属性、装饰器、元类。

<div  align="center">  
<img src="https://s2.loli.net/2022/09/17/ri9Ue6nguJdq1Ca.jpg" width = "80%" height = "80%" alt="Learning Python"/>
</div>

**<font size=5>PART VII Exceptions and Tools</font>**
## Chapter 33 Exception Basics
### 一、Exception Basics
**1、异常的5类语句**  
① `try/except`：捕捉异常并从中恢复；  
② `try/finally`：无论异常是否发生，执行清理行为；  
③ `raise`：手动在代码中触发异常；  
④ `assert`：有条件地在代码中触发异常；  
⑤ `with/as`：实现环境管理器，见下一章。  

**2、异常的作用**  
错误处理；事件通知；特殊情况处理；终止行为；非常规控制流程

**3、捕获异常**

``` python
def fetcher(obj, index):
    return obj[index]

x = "spam"

try:
    fetcher(x, 4)
except IndexError: # Catch and recover
    print('got exception')
```

**4、引发异常： `raise` 语句**

``` python
try:
    raise IndexError # Trigger exception manually
except IndexError:
    print('got exception')
```

**5、用户定义的异常**  
用户定义的异常通过类来编写，它们继承自一个**内置异常类 Exception** ：

``` python
class AlreadyGotOne(Exception): pass

def grail():
    raise AlreadyGotOne() # Raise an instance

try:
    grail()
except AlreadyGotOne:
    print('got exception')
```

可以定制出错信息文本：

``` python
class Career(Exception):
    def __str__(self): return 'So I became a waiter...'

raise Career()
```

**6、终止动作**  
`try/finally`组合指明结束时一定会执行的终止动作：

``` python
try:
    fetcher(x, 3)
finally:
    print('after fetch')
```

触发异常也会执行 `finally` 代码块：

``` python
def after():
    try:
        fetcher(x, 4)
    finally:
        print('after fetch')
    print('after try?')

after()
```

## Chapter 34 Exception Coding Details
### 一、The try/except/else Statement
**1、 `try/except/else` 语句**  
① `except`: 捕捉所有异常类型；  
② `except name`: 只捕捉指定的异常；  
③ `except name as value`: 捕捉所列异常并将该异常实例赋值给名称 value ；  
④ `except (name1, name2)`: 捕捉任何列出的异常；   
⑤ `except (name1, name2) as value`: 捕捉任何列出的异常，并将该异常实例元组赋值给名称 value ；  
⑥ `else`: 如果没有引发异常，就会运行；  
⑦ `finally`: 总是在退出 try 语句时运行此代码块；  

可以这么理解：尝试一个可能会出错的语句，除了（except）XX 错误，运行定制的语句，若没有（else）错误，运行没发生异常要执行的程序；如果 try 后面的语句执行时引发了异常，python 会回到 try 并搜索第一个与异常名称匹配的 except。

python会从上到下以及由左至右地检测 except 分句。

**2、合并 try 语句的语法**  
`try` 语句必须至少有一个 except 或一个 finally ，顺序为 *try -> except -> else -> finally* ；如果要有 else ，必须有至少一个 except ：

``` python
sep = '-' * 45 + '\n'

print(sep + 'EXCEPTION RAISED AND CAUGHT')
try:
    x = 'spam'[99]
except IndexError:
    print('except run')
finally:
    print('finally run')
print('after run')

print(sep + 'NO EXCEPTION RAISED')
try:
    x = 'spam'[3]
except IndexError:
    print('except run')
finally:
    print('finally run')
print('after run')

print(sep + 'NO EXCEPTION RAISED, WITH ELSE')
try:
    x = 'spam'[3]
except IndexError:
    print('except run')
else:
    print('else run')
finally:
    print('finally run')
print('after run')

print(sep + 'EXCEPTION RAISED BUT NOT CAUGHT')
try:
    x = 1 / 0
except IndexError:
    print('except run')
finally:
    print('finally run')
print('after run')
```

### 二、The raise Statement
**1、引发异常**  
① `raise instance`  
② `raise class`  
异常总是类的实例，如果传入一个类，python 会创建被引发的一个异常实例：即 `raise IndexError` 等价于 `raise IndexError()`；

如果 try 中包含了 `except name as X` ，那么 raise 中的异常实例会赋值给变量 X 。

**2、作用域和 `try except` 变量**

``` python
try:
    1 / 0
except Exception as X:
    print(X)
```

X 会被局限在 except 块中，而且该变量 X 会在 except 块退出后被移除：

``` python
X = 99
try:
    1 / 0
except Exception as X:
    print(X)

print(X) # 会出错：NameError: name 'X' is not defined
```

若要在 try 语句后引用该异常实例，需要赋值给另一个变量：

``` python
try:
    1 / 0
except Exception as X:
    print(X)
    Saveit = X

print(Saveit)
```

**3、异常链：`raise from`**  
`raise newexception from otherexception`  
from 后面跟的表达式指定了另一个异常类或实例，该异常会附加到 newexception 的 `__cause__` 属性；如果 newexception 没有被捕获，那么 python 会把2个异常都作为标准出错消息打印出来：

``` python
try:
    1 / 0
except Exception as E:
    raise TypeError('Bad') from E
```

当异常处理程序内部，程序错误地引发一个异常，一个相似的过程会自动发生：

``` python
try:
    1 / 0
except:
    badname
```

### 三、The assert Statement
**1、assert 可视为条件式的 raise 语句**  
`assert test, data`  
如果 test 为假， python 就引发异常 data 项：

``` python
def f(x):
    assert x < 0, 'x must be negative'
    return x ** 2

f(1) # 出现错误：AssertionError: x must be negative
```

assert 语句中 data 是可选的；  
assert 几乎是用来捕捉用户定义的约束条件，而不是捕捉实际的程序设计错误；  
因为 python 会自行捕获错误，通常没必要写 assert 去捕捉：

``` python
def reciprocal(x):
    assert x != 0
    return 1 / x
# 上述的assert一般都是多余的
```

### 四、with/as Context Managers
**1、基本用法**

``` python
with expression [as variable]:
    with-block
```
这里的 expression 要返回一个支持上下文管理协议的对象：

``` python
with open(r'C:\misc\data') as myfile:
    for line in myfile:
        print(line)
```

可以使用 `try/finally` 语句来实现类似的效果：

``` python
myfile = open(r'C:\misc\data')
try:
    for line in myfile:
        print(line)
finally:
    myfile.close()
```

**2、上下文管理协议**  
可以自己编写一个上下文管理器（未摘抄，有兴趣再查阅）

## Chapter 35 Exception Objects
### 一、Class-Based Exceptions
**1、编写异常类**  
异常类拥有状态信息和行为，支持继承：

``` python
class General(Exception): pass
class Specific1(General): pass
class Specific2(General): pass

def raiser0():
    X = General()
    raise X 

def raiser1():
    X = Specific1()
    raise X

def raiser2():
    X = Specific2()
    raise X

for func in (raiser0, raiser1, raiser2):
    try:
        func()
    except General:
        import sys
        print('caught: %s' % sys.exc_info()[0])
```

`sys.exc_info` 用于抓取最近发生的异常，它的结果第一个元素是被引发的异常类，第二个元素是实际被引发的实例。

### 二、Built-in Exception Classes
**1、Python 能够引发的所有内置异常都是预定义的类对象，可通过 builtin 模块中的内置名称使用**  
① `BaseException`：异常的顶层根父类，该类不应由用户定义的类直接继承，它提供了子类可继承的默认打印和状态保持行为；  
② `Exception`：用户定义的异常的根父类，它是 BaseException 类的一个直接子类，并且是除系统退出事件类外，所有其他内置异常的父类；  
③ `ArithmeticError`：Exception 的子类，数字错误的根父类，它的子类包括 `OverflowError` ,  `ZeroDivisionError` 和 `Floating PointError` ；  
④ `LookupError`：Exception 的子类，索引错误的根父类，它的子类包括 `IndexError` ,  `KeyError` 等等。  

**2、默认打印和状态**  
传递给内置异常类的参数，都会被自动保存在实例的 args 元组属性中，并且在打印该实例的时候自动显示：

``` python
raise IndexError
raise IndexError('spam')
I = IndexError('spam')
print(I.args)
print(I)
```

用户定义的异常类同样如此：

``` python
class E(Exception): pass
raise E
raise E('spam')
I = E('spam')
print(I.args)
print(I)

try:
    raise E('spam')
except E as X:
    print(X)
    print(X.args)
    print(repr(X))
```

**3、定制的打印显示**  
除了像上面传递参数来定制显示，也可以定义 `__str__` 或 `__repr__` 来返回希望显示的字符串：

``` python
class MyBad(Exception):
    def __str__(self):
        return 'Always look on the bright side of life...'

try:
    raise MyBad()
except MyBad as X:
    print(X)

raise MyBad()
```

### 三、Custom Data and Behavior
**1、内置异常父类提供了一个默认的构造函数，把构造函数参数自动存储到了一个名为 args 的实例元组属性中**  

``` python
class FormatError(Exception):
    def __init__(self, line, file):
        self.line = line
        self.file = file

def parser():
    raise FormatError(42, file='spam.txt')

try:
    parser()
except FormatError as X:
    print('Error at: %s %s' % (X.file, X.line))
```

**2、提供异常方法**

``` python
class FormatError(Exception):
    logfile = 'formaterror.txt'
    def __init__(self, line, file):
        self.line = line
        self.file = file
    def logerror(self):
        log = open(self.logfile, 'a')
        print('Error at:', self.file, self.line, file=log)

def parser():
    raise FormatError(40, 'spam.txt')

try:
    parser()
except FormatError as exc:
    exc.logerror()
```

## Chapter 36 Designing with Exceptions
### 一、Nesting Exception Handlers
**嵌套异常：**  
①示例：控制流嵌套：

``` python
def action2():
    print(1 + []) # Generate TypeError

def action1():
    try:
        action2()
    except TypeError:
        print('inner try')

try:
    action1()
except TypeError:
    print('outer try')
```

②示例：语法嵌套化：

``` python
def raise1(): raise IndexError
def noraise(): return
def raise2(): raise SyntaxError

for func in (raise1, noraise, raise2):
    print('<%s>' % func.__name__)
    try:
        try:
            func()
        except IndexError:
            print('caught IndexError')
    finally:
        print('finally run')
    print('...')
```

### 二、Exception Idioms
**跳出多重循环嵌套**  
可以用 raise 来跳出循环：

``` python
class Exitloop(Exception): pass

try:
    while True:
        while True:
            for i in range(10):
                if i > 3: raise Exitloop
                print('loop3: %s' % i)
            print('loop2')
        print('loop1')
except Exitloop:
    print('continuing')
```

如果 raise 换成 break ，会无线循环，因为只是跳出了 for 循环，没有跳出 while 循环。


**<font size=5>PART VIII Advanced Topics</font>**
## Chapter 37 Unicode and Byte Strings
### 一、String Basics
**1、python 3.X 中的字符串修改**  
python 2.X的 str 和 unicode 类型以及融入了python 3.X的 bytes 和 str 类型，而且新增了 bytearray 可变类型：  
①如果使用 ASCII 或 UTF-8 ，普通的字符串 str 对象和文本文件能够应对；  
②处理非 ASCII 的 Unicode 文本，3.X 比 2.X 对其的支持更直接好用；  
③处理二进制数据，例如图像或音频文件，需要理解 bytes 对象。

**2、字符编码方案**  
**ASCII 标准**定义了从0到127的字符编码，并且允许每个字符存储在一个8位的字节中，实际上只有7位被用到（2^7=128)；

`ord` 函数返回字符的二进制识别值（Unicode码点序数），`chr` 函数返回给定整数编码值的对应字符：

``` python
print(ord('a'))
print(ord('啊'))
print(hex(97))
print(chr(97))
```

各种符号和重音字符并不在 ASCII 所定义的字符范围内。为了容纳特殊字符，一些标准使用一个8位字节所有可能的值（0到255），并把 ASCII 范围以外的128-255分配给特殊字符，其中一个标准叫 **Latin-1 字符集**。

``` python
print(chr(196))
```

**Unicode 文本**用以表示欧洲、亚洲和其他非英语的字符集，拥有比8位字节更多的字符；

字节和字符串之间的转换由2个术语定义：  
①**编码 Encoding** ：把字符串翻译为原始字节形式的过程；  
②**解码 Decoding** ：把一个原始字符串翻译为字符串的过程。  

**UTF-8 编码**，采用可变的字节数 byte（8 bit 比特、位）来表示众多字符：  
小于128的字符码为1个字节；128和 0x7ff (2047)之间的字符码转换为两个字节，其中每个字节的值都位于128-255之间， 0x7ff 以上的代码转换为3个或4个字节序列，序列每个字节的值都位于128-255之间。

ASCII 是 Latin-1 和 UTF-8 的子集，即 ASCII 字符串也是有效的 Latin-1 和 UTF-8 编码字符串；ASCII 、Latin-1 、UTF-8 以及很多其他的编码，都被认为是 Unicode ；

**UTF-16** 和 **UTF-32** 分别按照每字符固定大小的2个和4个字节来格式化文本。  
2个字符的 ASCII 字符串是2个字节，但是在 UTF-16 和 UTF-32 中它会更宽，并包含头部的字节：

``` python
S = 'ni'
print(S.encode('ascii'), S.encode('latin1'), S.encode('utf8'))
print(S.encode('utf16'), len(S.encode('utf16')))
print(S.encode('utf32'), len(S.encode('utf32')))
```

**3、Python 如何在内存中存储字符串的**  
Python 3.3 以后，在内存中 python 为每个字符分配1、2或4个字节。

**4、Python 的字符串类型**  
Python 3.X 带有3中字符串对象类型：  
① `str` 表示解码的 Unicode 文本（包括 ASCII ）；  
② `bytes` 表示二进制数据（包括编码的文本）；  
③ `bytearray`，一种可变的 bytes 类型。  

**5、文本和二进制文件**  
**文本文件**：当一个文件以文本模式打开时，读取其数据会自动将内容解码，并且将解码的内容返回为一个 **str** ：写入内容需要一个 **str** ，并且将其传输到文件之前自动编码它；  
**二进制文件**：通过内置 `open` 函数的模式字符串参数添加一个 **b** ，就能以二进制模式打开文件，此时读取其数据不会解码它，而是将其作为一个 **bytes** 对象；写入同理，接受 **bytes** 对象。  

如果处理图像文件、经网络传输的数据、必须解压的打包二进制数据等等，使用 bytes 和二进制模式文件处理合适；  
如果要处理的内容本质是文本化的，例如程序输出、 HTML 、电子邮件内容或 CSV 或 XML 文件，使用 str 和文本模式合适。

### 二、Coding Basic Strings
**1、字符串字面量**  
`'xxx'`、`"xxx"`、`'''xxx'''`都会产生一个 str ；在它们任何一个前面添加一个 b 或 B ，则会创建一个 bytes ：

``` python
B = b'spam'
S = 'eggs'
print(type(B), type(S))
```

bytes 对象实际上是一个**短整数序列**，但它尽可能地将自己打印为字符：

``` python
print(B[0], B[1:], list(B))
```

bytes 对象也是**不可修改**的。

**2、Unicode 字面量，它们被当作普通的 str 字符串（为了兼容2.X而存在）**

``` python
U = u'spam'
print(type(U))
print(U)
```

**3、字符串类型转换**  
① `str.encode()` 和 `bytes(S, encoding)` 把字符串转换为其原始字节形式；  
② `bytes.decode()` 和 `str(B, encoding)` 把原始字节转换为其字符串形式。  

``` python
S = 'eggs'
print(S.encode())
print(bytes(S, encoding='ascii'))
B = b'spam'
print(B.decode())
print(str(B, encoding='ascii'))
```

encode 和 decode 方法根据使用者的平台使用默认编码名称：

``` python
import sys
print(sys.platform)
print(sys.getdefaultencoding())
```

str 可以省略编码名称参数，但是不带编码名称的 str 会直接返回 bytes 对象的打印字符串：

``` python
print(str(B))
print(len(str(B)))
```

### 三、Coding Unicode Strings
**1、Python的字符串字面量支持 "\xNN" 十六进制字节值转义以及 "\uNNNN" 和 "\UNNNNNNNN" Unicode 转义**  
在 Unicode 转义中，第一种形式用4位十六进制数编码2字节字符码点，第二种形式用8位十六进制数编码4字节码点。

**2、编写 ASCII 文本**  
ASCII 文本是一个简单的 Unicode 类型，作为字节值序列存储：

``` python
print(ord('X'))
S = 'XYZ'
print(S.encode('ascii')) # Values 0..127 in 1 byte (7 bits) each
print(S.encode('latin-1')) # Values 0..255 in 1 byte (8 bits) each
print(S.encode('utf-8')) # Values 0..127 in 1 byte, 128..2047 in 2, others 3 or 4
```

**3、编写非 ASCII 文本**  
①在 str 里，通过十六进制转义或者 Unicode 转义( escapes )；  
②在 byte 中，通过十六进制转义。

十六进制转义 x 要求2个16进制数位，而 Unicode 转义的 u 和 U 分别为4个和8个十六进制数位，例如：十六进制值 0xCD 和 0xE8 。

``` python
print(chr(0xc4), chr(0xe8))
S = '\xc4\xe8' # Single 8-bit value hex escapes: two digits，16^2即2^8，即8位
print(S)
S = '\u00c4\u00e8' # 16-bit Unicode escapes: four digits each，16^4即2^16，即16位
print(S)
S = '\U000000c4\U000000e8' # 32-bit Unicode escapes: eight digits each，16^8即2^32，即32位
print(S)
```

**4、编码和解码非 ASCII 文本**  
①编码

``` python
S = '\u00c4\u00e8'
print(S.encode('latin-1')) # 1 byte per character when encoded
print(S.encode('utf-8')) # 2 bytes per character when encoded
print(len(S.encode('latin-1')))
print(len(S.encode('utf-8')))
```

②解码

``` python
B = b'\xc4\xe8'
print(len(B)) # 2 raw bytes, two encoded characters
print(B.decode('latin-1'))

B = b'\xc3\x84\xc3\xa8'
print(len(B))
print(B.decode('utf-8'))
```

**5、混用非 ASCII 字符和 ASCII 字符**

``` python
S = 'A\u00c4B\U000000e8C'
print(S)
print(S.encode('latin-1'))
print(len(S.encode('latin-1')))
print(S.encode('utf-8'))
print(len(S.encode('utf-8')))
```

对于 UTF-16 和 UTF-32 ，使用每字符2字节和4字节方案，并有着相同大小的编码头：

``` python
S = 'spam'
print(S.encode('utf-16'))
print(S.encode('utf-32'))
```

**6、byte 字节串的转义**  
Python 允许特殊字符以十六进制和 Unicode 转义的方式编码到 str 字符串中，但只能以十六进制转义的方式编码到 bytes 字符串中。

在 bytes 中，Unicode 转义序列会被当作字符处理：

``` python
B = b'A\xC4B\xE8C'
print(B)
B = b'A\u00C4B\U000000E8C'
print(B)
```

字节串的字面量要求字符是 ASCII 字符， str 字符串允许字面量包含任何字符：

``` python
S = 'AÄBèC'
print(S)
B = b'AÄBèC' # 会出错：SyntaxError: bytes can only contain ASCII literal characters
B = b'A\xC4B\xE8C' # Chars must be ASCII, or escapes
print(B.decode('latin-1'))
```

### 四、Using bytes Objects
**1、`Bytes` 对象是一个小整数序列，每个整数在0到255之间，显示时打印为 ASCII 字符**  
它支持序列操作以及 str 对象上可用的大多数方法，但不支持格式化方法或 % 格式化表达式。与 str 一样，是不可改变对象。

**2、方法调用**

``` python
print(set(dir('abc')) - set(dir(b'abc')))
print(set(dir(b'abc')) - set(dir('abc')))
```

bytes 的方法需要 bytes 类型的参数：

``` python
B = b'spam'
print(B.find(b'pa'))
print(B.replace(b'pa', b'XY'))
print(B.split(b'pa'))
```

**3、序列运算**  
bytes 是一个8位的序列，对其索引会返回表示其二进制值的整数：

``` python
B = b'spam'
print(B[0], B[-1])
print(chr(B[0]))
print(list(B))
print(B[1:], B[:-1], len(B), B + b'lmn', B * 4)
```

**4、创建 bytes 对象的其他方式**
bytes 构造函数，可以传递 str 和编码名称参数，也可以传递一系列整数的可迭代对象作为参数：

``` python
B = bytes('abc', 'ascii')
print(B)
B = bytes([97, 98, 99])
print(B)
```

### 五、Using bytearray Objects
**1、`bytearray` ，是范围0到255之间的整数的一个可变序列，是 bytes 的可变的变体**  
它支持和 bytes 相同的字符串方法和序列操作，并且支持和列表同样多的可变的原位置修改操作。

**2、bytearray 内置函数来创建 bytearray 对象**  
需要传入编码名称和 str 字符串，或者直接传入字节串 bytes ：

``` python
S = 'spam'
C = bytearray(S, 'latin1')
print(C)

B = b'spam'
C = bytearray(B)
print(C)
```

bytearry 对象也是小整数序列，像列表一样可以修改：

``` python
print(list(C))
print(C[0])
C[0] = ord('x') # 索引复制要提供一个整数
print(C)
C[1] = b'Y'[0]
print(C)
```

可以用列表的方法来原处修改 bytearray ：

``` python
C.append(ord('L')) # 接受的也是数字
print(C)
C.extend(b'MNO') # extend接受可迭代对象
print(C)
```

序列操作和字符串方法也在 bytearray 上有效：

``` python
print(C + b'!#')
print(len(C))
C.replace(b'xY', b'sp')
print(C)
print(C * 4)
```

### 六、Using Text and Binary Files
**1、文本模式意味着 str 对象，而二进制模式意味着 bytes 对象**  
①**文本模式文件 Text-mode files** 根据 Unicode 编码来解释文件内容，要么是平台的默认编码名，要么是传递进的编码名；  
②**二进制模式文件 Binary-mode files** 返回原始的文件内容，作为表示字节值的一个整数序列。  

open 内置函数的第二个参数决定了要处理文本文件还是二进制文件，比如 rb 就是读取二进制文件，默认模式是 rt ，等同于 r ；文本文件返回一个 str 供读取，需要 str 来写入；二进制文件返回 bytes 供读取，需要一个 bytes 或 bytearray 供写入。

**2、文本文件基础**

``` python
file = open('temp', 'w')
size = file.write('abc\n')
file.close()

file = open('temp')
text = file.read()
print(text)
```

**3、文本和二进制模式**  
写入时提供 str ，并根据 open 的模式，读取是获得 str 或 bytes ：

``` python
open('temp', 'w').write('abc\n')
print(open('temp', 'r').read())
print(open('temp', 'rb').read())
```

文本模式在输出时把 \n 转换为 \r\n ，在输入时把 \r\n 转换回 \n ，但二进制模式不会这么做。  

使用二进制文件，运行类似代码：

``` python
open('temp', 'wb').write(b'abc\n')
print(open('temp', 'r').read())
print(open('temp', 'rb').read())
```

二进制模式在输出中，\n 没有扩展为 \r\n 。

\x00 是二进制0字节并且不是一个可打印的字符：

``` python
open('temp', 'wb').write(b'a\x00c')
print(open('temp', 'r').read())
print(open('temp', 'rb').read())
```

二进制模式文件用 bytes 对象返回其内容，但接受一个 bytes 或 bytearray 对象写入；
大多数 python API 接受 bytes 的同时也会接受 bytearray ：

``` python
BA = bytearray(b'\x01\x02\x03')
open('temp', 'wb').write(BA)
print(open('temp', 'r').read())
print(open('temp', 'rb').read())
```

以文本模式读取默认字符集以外的二进制数据会出现 UnicodeDecodeError 。

因为文本模式的输入文件必须能够依据 Unicode 编码来解码内容，所以没有办法在文本模式下读取真正的二进制内容（区分 Unicode 码点和字节值）。

``` python
open('temp', 'wb').write(b'\xFF\xFE\xFD')
print(open('temp', 'rb').read())
print(open('temp', 'r').read())
```

所以最好文本文件以文本模式读取，二进制文件以二进制模式读取。

### 七、Using Unicode Files
**1、open 内置函数接受一个编码名称，可以自动解码和编码**

**2、读写 Unicode 文件**  
把字符串以特定编码写入一个文本文件：

``` python
S = 'A\xc4B\xe8C'
open('latindata', 'w', encoding='latin-1').write(S)
open('utf8data', 'w', encoding='utf-8').write(S)
print(open('latindata', 'rb').read())
print(open('utf8data', 'rb').read())
```

以特定编码读取文本文件：

``` python
print(open('latindata', 'r', encoding='latin-1').read())
print(open('utf8data', 'r', encoding='utf-8').read())
```

手动解码：

``` python
X = open('latindata', 'rb').read()
print(X.decode('latin-1'))
X = open('utf8data', 'rb').read()
print(X.decode())
```

**3、处理 BOM**  
一些编码方式在文件开始处存储了一个特殊的**字节顺序标记（BOM，byte order marker）**序列，来指定数据的大小尾方式 endianness (which end of a string of bits is most significant to its value)，或者声明编码类型。

如果编码名暗示了 BOM ，python 在输入时会忽略该标记，在输出时写入该标记；例如，在 UTF-16 和 UTF-32 中，BOM 指定大尾或小尾格式；一个 UTF-8 文本可能也会包含一个 BOM ，通常只是申明 UTF-8 格式：  
①在 UTF-16 中，总是对 utf-16 进行 BOM 处理，而更为特定的编码名称 "utf-16-le" 标示小尾格式；  
②在 UTF-8 中，更为特定的编程名称 "utf-8-sig" 迫使 python 在输入和输出时分别跳过和写入 BOM ，但是常用的 "utf-8" 不会这样做。

小尾 Little Endian 方式：低位字节放在内存的低地址处，高位字节放在高地址处；  
大尾 Big Endian 方式：低位字节放在内存的高地址处，高位字节放在低地址处。

**4、记事本 BOM 示例**  
如果 spam.txt 保存为 UTF-8 模式（默认模式）：

``` python
import sys
print(sys.getdefaultencoding())
print(open('spam.txt', 'rb').read())
print(open('spam.txt', 'r').read())
print(open('spam.txt', 'r', encoding='utf-8').read())
```

如果 spam.txt 保存为 UTF-8 BOM 模式：

``` python
print(open('spam.txt', 'rb').read())
print(open('spam.txt', 'r').read())
print(open('spam.txt', 'r', encoding='utf-8').read())
print(open('spam.txt', 'r', encoding='utf-8-sig').read())
```

如果 spam.txt 保存为 UTF-16 BE（大尾）模式：

``` python
print(open('spam.txt', 'rb').read())
print(open('spam.txt', 'r', encoding='utf-16').read())
print(open('spam.txt', 'r', encoding='utf-16-be').read())
```

**5、用 Python 代码让 UTF-8 带有 BOM（使用 utf-8-sig ）**

``` python
open('temp.txt', 'w', encoding='utf-8').write('spam\nSPAM\n')
print(open('temp.txt', 'rb').read())

open('temp.txt', 'w', encoding='utf-8-sig').write('spam\nSPAM\n')
print(open('temp.txt', 'rb').read())
print(open('temp.txt', 'r', encoding='utf-8').read()) # Keeps BOM
print(open('temp.txt', 'r', encoding='utf-8-sig').read()) # Skips BOM
```

对于 UTF-16 ，BOM 被自动处理：在输出时，数据以平台本地的大小尾方式书写，并且 BOM 总是存在；

在输入时，数据根据 BOM 解码，并且总是去掉 BOM ：

``` python
import sys
print(sys.byteorder) # 显示little
open('temp.txt', 'w', encoding='utf-16').write('spam\nSPAM\n')
print(open('temp.txt', 'rb').read())
print(open('temp.txt', 'r', encoding='utf-16').read())
```

UTF-16 编码名称可以指定不同的大小尾：

``` python
open('temp.txt', 'w', encoding='utf-16-be').write('\ufeffspam\nSPAM\n')
print(open('temp.txt', 'rb').read())
print(open('temp.txt', 'r', encoding='utf-16').read())
print(open('temp.txt', 'r', encoding='utf-16-be').read())
```

### 八、Other String Tool
本节内容较为简单，需额外查阅其他资料。

**1、re 模式匹配模块 re Pattern-Matching Module（正则表达式）**  
这里介绍得太简单了，详见菜鸟教程。

re 模块使 Python 语言拥有全部的正则表达式功能，可用于 str 、 bytes 和 bytearray ；`re.match`函数；在模式字符串中，(.\*)表示任意除换行符（\n、\r）之外字符，(.)重复0或多次(\*)，并作为匹配的子字符串单独保存；groups() 方法返回一个包含所有小组字符串的元组：

``` python
import re
S = 'Bugger all down here on earth!'
B = b'Bugger all down here on earth!'

print(re.match('(.*) down (.*) on (.*)', S).groups())
print(re.match(b'(.*) down (.*) on (.*)', B).groups())
```

**2、struct 二进制数据模块**  
struct 模块用来从字符串中创建和提取打包的二进制数据：

``` python
import struct
B = struct.pack('>i4sh', 7, b'spam', 8)
print(B)
vals = struct.unpack('>i4sh', B)
```

也可以创建和读取二进制文件：

``` python
F = open('data.bin', 'wb')
import struct
data = struct.pack('>i4sh', 7, b'spam', 8)
print(data)
F.write(data)
F.close()

F = open('data.bin', 'rb')
data = F.read()
print(data)
values = struct.unpack('>i4sh', data)
print(values)
```

**3、pickle 对象序列化模块**  
pickle 总是创建一个 bytes 对象（不管传入的协议），使用该模块的 dumps 调用来返回对象的pickle 字符串：

``` python
import pickle
print(pickle.dumps([1, 2, 3])) # default protocol=3=binary
print(pickle.dumps([1, 2, 3], protocol=0)) # ASCII protocol 0
```

存储 pickle 化对象的文件必须总是以二进制模式打开：

``` python
pickle.dump([1, 2, 3], open('temp', 'wb'))
print(pickle.load(open('temp', 'rb')))
print(open('temp', 'rb').read())
```

**4、XML 解析工具 Parsing Tools**  
XML 是一种基于标签的语言，用于定义结构化信息，通常用来定义通过Web传输的文档和数据；Python 自身附带一个完整的 XML 解析工具包，支持 SAX 和 DOM 解析模式。

不抄写，学了 XML 后可以再来了解。


## Chapter 38 Managed Attributes
### 一、Why Manage Attributes
**1、为什么使用被管理的属性**  
对工具构建者来说，被管理属性的访问是灵活的 API 的一个重要部分；

在整个程序对使用了某名称的所有地方都进行修改，不是个小任务，可以选择编写方法来管理对属性值的访问。

**2、属性访问器 attribute accessor**  
四种访问器技术：  
① `__getattr__` 和 `__setattr__` 方法，用于把未定义的属性获取和所有属性赋值路由到通用的处理方法；  
② `__getattribute__` 方法，用于把所有属性获取都路由到一个泛化的处理方法；  
③ `property` 内置函数，用于把特定属性访问路由到 `get` 和 `set` 函数；  
④描述符协议，用于把特定属性访问路由到具有任意访问和修改处理方法的类的实例，是 `property` 和 `slot` 工具的基础。  

这4中属性拦截技术都用于把任意属性路由到被包装对象的、基于委托的代理类。

### 二、Properties  
**property** 协议允许我们把一个特定属性的获取、设置和修改操作指向我们所提供的函数或方法，使得我们能够插入在属性访问时自动运行的代码，或是拦截属性的删除；

property 内置函数可以创建 property 并将其赋值给类属性，跟方法函数一样；同时也是可以被子类和实例继承的属性；  
property 的访问拦截函数带有 self 实例参数，可以在主体实例上访问状态信息和类属性；  
一个 property 管理一个单一的、特定的属性；  
property 就是描述符 descriptors 的一种受限制的形式。

**1、基础知识**  
将 property 赋值给类属性：`attribute = property(fget, fset, fdel, doc)`  

参数都不是必需的：  
① `fget` 传入函数（或类方法）用于拦截属性访问；  
② `fset`（或类方法）传入函数用于属性赋值；  
③ `fdel`（或类方法）传入函数用于属性删除；  
④ `fget` 函数返回被计算好的属性值，`fset` 和 `fdel` 返回 None ；  
⑤ `doc` 参数接受该属性的一个文档字符串。

property 函数返回一个 property 对象，将其赋予要被管理的类属性名称，它又被类的所有实例继承。

**2、第一个示例**

``` python
class Person:
    def __init__(self, name):
        self._name = name
    def getName(self):
        print('fetch...')
        return self._name
    def setName(self, value):
        print('change...')
        self._name = value
    def delName(self):
        print('remove...')
        del self._name
    name = property(getName, setName, delName, "name property docs")

bob = Person('Bob Smith')
print(bob.name) # 属性访问Runs getName
bob.name = 'Robert Smith' # 属性赋值Runs setName，打印出change...
print(bob.name) # 属性访问Runs getName
del bob.name # 属性删除Runs delName

sue = Person('Sue Jones')
print(sue.name)
print(Person.name.__doc__)
```

**3、动态地计算属性的值**

``` python
class PropSquare:
    def __init__(self, start):
        self.value = start
    def getX(self):
        return self.value ** 2
    def setX(self, value): 
        self.value = value
    X = property(getX, setX)

P = PropSquare(3)
Q = PropSquare(32)

print(P.X)
P.X = 4
print(P.X)
print(Q.X)
```

**4、使用装饰器编写 property**  
property 对象也有 getter 、 setter 和 deleter 方法，这些方法赋值了相应的 property 访问器方法，并且返回了 property 自身的副本。

可以通过装饰器来指定这些组件，getter 组件由创建 property 自身的行为来自动填充：

``` python
class Person:
    def __init__(self, name):
        self._name = name
    
    @property # name = property(name)
    def name(self):
        "name property docs"
        print('fetch...')
        return self._name

    @name.setter # name = name.setter(name)
    def name(self, value):
        print('change...')
        self._name = value
    
    @name.deleter # name = name.deleter(name)
    def name(self):
        print('remove...')
        del self._name

bob = Person('Bob Smith')
print(bob.name)
bob.name = 'Robert Smith'
print(bob.name)
del bob.name
```

### 三、Descriptors
**描述符**协议允许把一个特定的属性的获取、设置和删除操作指向一个单独类对象的方法； property 是描述符的一种。

描述符编写成独立的类，它们就像方法函数一样赋值给类属性，会被子类和实例继承；通过为描述符自身提供一个 self ，或通过让客户类实例的属性引用描述符对象。因此它们可以保留和使用自身的状态信息，以及主体实例的状态信息。

描述符也管理一个单一的、指定的属性。

**1、基础知识**

``` python
class Descriptor:
    "docstring goes here"
    def __get__(self, instance, owner): ...
    def __set__(self, instance, value): ...
    def __delete__(self, instance): ...
```

所有带有这些方法（ `__get__` 、 `__set__` 和 `__delete__` ）的类都可以看作描述符。

与 property 不同，省略了 `__set__` 意味着允许被管理的属性通过赋值重新定义，这样就会隐藏描述符；要使一个属性是只读的，必须定义 `__set__` 来捕获赋值并引发一个异常；带有 `__set__` 的描述符被称为数据描述符 data descriptor ，相比于其他正常继承规则而定位的属性拥有优先权。

区分描述符的 `__delete__` 方法和常见的 `__del__` 方法，前者会在试图删除被管理属性的名称时被调用；而后者是通用的实例析构函数方法，会在任何类的实例将要进行垃圾回收时被调用。

**2、描述符方法参数**  
`__get__` 访问方法额外接受一个 owner 参数，指定了描述符实例所依附的类；instance 参数要么是被访问属性的实例，要么当访问的属性是类属性的时候是 None 。

属性获取自动传递到 `__get__` 方法中的参数，`X.attr -> Descriptor.__get__(Subject.attr, X, Subject)`

``` python
class Descriptor:
    def __get__(self, instance, owner):
        print(self, instance, owner, sep='\n')

class Subject:
    attr = Descriptor()

X = Subject()
X.attr
Subject.attr
```

**3、只读描述符 Read-only descriptors**  
在描述符类中捕获赋值操作并引发一个异常来阻止属性赋值：

``` python
class D:
    def __get__(*args): print('get')
    def __set__(*args): raise AttributeError('cannot set')

class C:
    a = D()

X = C()
X.a
X.a = 99 # 会出错：AttributeError: cannot set
```

**4、第一个示例**

``` python
class Name:
    "name descriptor docs"
    def __get__(self, instance, owner):
        print('fetch...')
        return instance._name
    def __set__(self, instance, value):
        print('change...')
        instance._name = value
    def __delete__(self, instance):
        print('remove...')
        del instance._name

class Person:
    def __init__(self, name):
        self._name = name
    name = Name()

bob = Person('Bob Smith')
print(bob.name) # Runs Name.__get__
bob.name = 'Robert Smith' # Runs Name.__set__
print(bob.name) # Runs Name.__get__
del bob.name # Runs Name.__delete__

sue = Person('Sue Jones')
print(sue.name)
print(Name.__doc__)
```

描述符的 `__get__ `方法里，self 是 Name 类实例，instance 是 Person 类实例，owner 是 Person 类。描述符类实例是一个类的属性，因此被客户类和所有实例和子类继承。

**5、动态地计算属性的值**

``` python
class DescSquare:
    def __init__(self, start):
        self.value = start
    def __get__(self, instance, owner):
        return self.value ** 2
    def __set__(self, instance, value):
        self.value = value

class Client1:
    X = DescSquare(3)

class Client2:
    X = DescSquare(32)

c1 = Client1()
c2 = Client2()

print(c1.X)
c1.X = 4
print(c1.X)
print(c2.X)
```

**6、在描述符中使用状态信息**  
在上面2个描述符的例子中，第一个例子（ name 属性）使用了存储在客户实例中的数据，第二个例子（属性平方）使用了附加到描述符对象本身的数据；

描述符可以使用实例状态和描述符状态，或者二者的任意组合：  
①描述符状态用于管理描述符内部使用的数据，或是横跨所有实例的数据；  
②实例状态记录了和客户类相关、或是被客户类创建的信息。  

描述符状态基于描述符的数据，实例状态基于客户类实例的数据。

下面的描述符把信息附加到了它自己的实例：

``` python
class DescState:
    def __init__(self, value):
        self.value = value
    def __get__(self, instance, owner):
        print('DescState get')
        return self.value * 10
    def __set__(self, instance, value):
        print('DescState set')
        self.value = value

class CalcAttrs:
    X = DescState(2)
    Y = 3
    def __init__(self):
        self.Z = 4

obj = CalcAttrs()
print(obj.X, obj.Y, obj.Z)
obj.X = 5
CalcAttrs.Y = 6
obj.Z = 7
print(obj.X, obj.Y, obj.Z)

obj2 = CalcAttrs()
print(obj2.X, obj2.Y, obj2.Z)
```

这段代码的内部 value 信息仅存在于描述符之中；这里只管理了描述符的属性，即对 X 的获取和设置访问被拦截，对 Y 和 Z 的访问没被拦截。

对描述符存储或使用附加到客户类实例中的一个属性：

``` python
class InstState:
    def __get__(self, instance, owner):
        print('InstState get')
        return instance._X * 10
    def __set__(self, instance, value):
        print('InstState set')
        instance._X = value

class CalcAttrs:
    X = InstState()
    Y = 3
    def __init__(self):
        self._X = 2
        self.Z = 4

obj = CalcAttrs()
print(obj.X, obj.Y, obj.Z)
obj.X = 5
CalcAttrs.Y = 6
obj.Z = 7
print(obj.X, obj.Y, obj.Z)

obj2 = CalcAttrs()
print(obj2.X, obj2.Y, obj2.Z)
```

可以同时使用这2种状态信息：

``` python
class DescBoth:
    def __init__(self, data):
        self.data = data
    def __get__(self, instance, owner):
        return '%s, %s' % (self.data, instance.data)
    def __set__(self, instance, value):
        instance.data = value

class Client:
    def __init__(self, data):
        self.data = data
    managed = DescBoth('spam')

I = Client('eggs')
print(I.managed)
I.managed = 'SPAM'
print(I.managed)
print(I.data)
```

### 四、\_\_getattr\_\_ and \_\_getattribute\_\_
① `__getattr__` 针对未定义的属性运行，只能为不存储在实例中或是不继承自它的类的属性运行；  
② `__getattribute__` 针对所有的属性运行，要避免把属性访问传递给父类而导致递归循环。

这两个方法更加通用，更适合基于委托 delegation-based 的编码模式：用于实现包装器 wrapper （或代理 proxy ）来管理对一个内嵌对象的所有属性访问。

这两种方法拦截属性获取；`__setattr__` 方法捕获赋值对属性的更改；`__delattr__` 方法拦截属性删除。

**1、基础知识**  
① `def __getattr__(self, name):` On undefined attribute fetch [obj.name]  
② `def __getattribute__(self, name):` On all attribute fetch [obj.name]  
③ `def __setattr__(self, name, value):` On all attribute assignment [obj.name=value]  
④ `def __delattr__(self, name):` On all attribute deletion [del obj.name]  
2个 get 方法返回属性的值，另外2个返回 None 。

``` python
class Catcher:
    def __getattr__(self, name):
        print('Get: %s' % name)
    def __setattr__(self, name, value):
        print('Set: %s %s' % (name, value))

X = Catcher()
X.job
X.pay
X.pay = 99
```

**2、避免属性拦截方法的循环**  
由于 `__getattribute__` 和 `__setattr__` 针对所有的属性运行，要避免自己调用自己而触发递归循环 recursive loop 。

①比如在 `__getattribute__` 方法内的属性获取，会再次触发 `__getaatribute__` ：

``` python
def __getattribute__(self, name):
    x = self.other
```

当属性访问被编写在 `__getattribute__` 自身中，要避免循环，就需要另外把获取指向更高的父类，从而跳过这个层级的版本。因为object类总是新式类的父类，选择它比较好：

``` python
def __getattribute__(self, name):
    x = object.__getattribute__(self, 'other')
```

②在 `__setattr__` 方法内赋值任何属性，都会再次触发 `__setattr__` ：

``` python
def __setattr__(self, name, value):
    self.other = value
```

为解决这个问题，可以把属性赋值为实例的 `__dict__` 命名空间字典中的一个键赋值：

``` python
def __setattr__(self, name, value):
    self.__dict__['other'] = value
```

也可以把自己属性赋值传递给一个更高的父类而避免循环：

``` python
def __setattr__(self, name, value):
    object.__setattr__(self, 'other', value)
```

**3、第一个示例**

``` python
class Person:
    def __init__(self, name):
        self._name = name # Triggers __setattr__！！
    
    def __getattr__(self, attr):
        print('get: ' + attr)
        if attr == 'name':
            return self._name
        else:
            raise AttributeError(attr)
    
    def __setattr__(self, attr, value):
        print('set: ' + attr)
        if attr == 'name':
            attr = '_name'
        self.__dict__[attr] = value
    
    def __delattr__(self, attr):
        print('del: ' + attr)
        if attr == 'name':
            attr = '_name'
        del self.__dict__[attr]

bob = Person('Bob Smith')
print(bob.name) # Runs __getattr__
bob.name = 'Robert Smith' # Runs __setattr__
print(bob.name)
del bob.name # Runs __delattr__

sue = Person('Sue Jones')
print(sue.name)
```

**4、使用 `__getattribute__`**

``` python
def __getattribute__(self, attr):
    print('get: ' + attr)
    if attr == 'name':
        attr = '_name'
    return object.__getattribute__(self, attr)
```

把 `__getattr__` 替换为这个，结果是类似的，但是在 `__setattr__` 的获取中会触发一次额外的 `__getattribute__` 调用。

**5、计算出的属性**

``` python
class AttrSquare:
    def __init__(self, start):
        self.value = start

    def __getattr__(self, attr):
        if attr == 'X':
            return self.value ** 2
        else:
            raise AttributeError(attr)
    
    def __setattr__(self, attr, value):
        if attr == 'X':
            attr = 'value'
        self.__dict__[attr] = value
    
A = AttrSquare(3)
B = AttrSquare(32)

print(A.X)
A.X = 4
print(A.X)
print(B.X)
```

**6、\_\_getattr\_\_和\_\_getattribute\_\_的区别**
attr1 是一个类属性，attr2 是一个实例属性，attr3 是一个在获取时被管理的属性：

``` python
class GetAttr:
    attr1 = 1
    def __init__(self):
        self.attr2 = 2
    def __getattr__(self, attr):
        print('get: ' + attr)
        if attr == 'attr3':
            return 3
        else:
            raise AttributeError(attr)

X = GetAttr()
print(X.attr1)
print(X.attr2)
print(X.attr3)

class GetAttribute:
    attr1 = 1
    def __init__(self):
        self.attr2 = 2
    def __getattribute__(self, attr):
        print('get: ' + attr)
        if attr == 'attr3':
            return 3
        else:
            return object.__getattribute__(self, attr)

X = GetAttribute()
print(X.attr1)
print(X.attr2)
print(X.attr3)
```

`__getattr__` 只拦截 attr3 的访问，因为 attr3 是未定义的。


## Chapter 39 Decorators
### 一、The Basics
**装饰 Decoration** 是一种为函数和类指定管理或扩增代码的一种方式；即分为**函数装饰器 Function decorators**和**类装饰器 Class decorators** 。

**1、函数装饰器**

①用法：

``` python
@decorator
def F(arg):
    ...
F(99)
```

等同于：

``` python
def F(arg):
    ...
F = decorator(F)
F(99)
```

②实现：  
装饰器自身是一个返回可调用对象的可调用对象（函数和类的任何组合都可以使用）。

``` python
def decorator(F):
    return F
```

可以用一个装饰器返回和最初函数不同的一个对象；

用类来实现类似的装饰器，可以重载调用操作：

``` python
class decorator:
    def __init__(self, func):
        self.func = func
    def __call__(self, *args):
@decorator
def func(x, y)
```

③支持方法装饰：  
对类的方法进行装饰，上面的方式就不行了，用嵌套函数会更好：

``` python
def decorator(F):
    def wrapper(*args):
    return wrapper

@decorator
def func(x, y):
    ...
func(6, 7) # calls wrapper(6, 7)

class C:
    @decorator
    def method(self, x, y):
        ...

X = C()
X.method(6, 7) # calls wrapper(X, 6, 7)
```

这样子 wrapper 在其第一个参数里接收了 C 类的实例。


**2、类装饰器**  
类装饰器是管理类的一种方法，或是使用额外逻辑来完成实例构造调用的一种方式：

①用法：

``` python
@decorator
class C:
    ...
x = C(99)
```

等同于：

``` python
class C:
    ...
C = decorator(C)
x = C(99)
```

之后用类名调用创建一个实例时，最终会触发装饰器返回的可调用对象。

②实现：

``` python
def decorator(C):
    # Save or use class C；Return a different callable: nested def, class with __call__, etc.

@decorator
class C: ...
```

这样一个类装饰器返回的可调用对象，通常创建并返回最初类的一个新实例，并以某种方式扩展以管理接口。

比如下面的装饰器插入一个对象来拦截类实例的未定义属性：

``` python
def decorator(cls):
    class Wrapper:
        def __init__(self, *args):
            self.wrapped = cls(*args) # self.wrapped = C(6, 7)即一个实例
        def __getattr__(self, name):
            return getattr(self.wrapped, name)
    return Wrapper

@decorator
class C:
    def __init__(self, x, y): # Run by Wrapper.__init__
        self.attr = 'spam'

x = C(6, 7) # Really calls Wrapper(6, 7)
print(x.attr) # Runs Wrapper.__getattr__, 返回了self.wrapped.attr，由于self.wrapped为C类的实例，所以prints "spam"
```

类装饰器通常可以编写为一个创建并返回可调用对象的工厂函数，或是创建并返回类的工厂函数，使用 `__init__` 或 `__call__` 方法来拦截调用操作。

工厂函数通常在外层作用域引用中保持状态，而类在属性中保持状态。

**3、装饰器嵌套**

``` python
@A
@B
@C
def f(...):
    ...
```

等同于：

``` python
def f(...):
    ...
f = A(B(C(f)))
```

实例：

``` python
def d1(F): return lambda: 'X' + F()
def d2(F): return lambda: 'Y' + F()
def d3(F): return lambda: 'Z' + F()

@d1
@d2
@d3
def func(): # func = d1(d2(d3(func)))
    return 'spam'

print(func())
```

**4、装饰器参数**  
函数装饰器和类装饰器都能接受参数，这些参数传递给了返回装饰器的装饰器，而装饰器再返回可调用对象。

``` python
@decorator(A, B)
def F(arg):
    ...
F(99)
```

等同于：

``` python
def F(arg):
    ...
F = decorator(A, B)(F)
F(99)
```

例子中的装饰器函数的例子如下：

``` python
def decorator(A, B):
    # Save or use A, B
    def actualDecorator(F):
        # Save or use function F
        # Return a callable: nested def, class with __call__, etc.
        return callable
    return actualDecorator
```

### 二、Coding Function Decorators
编写函数装饰器的示例：

**1、跟踪调用**
使用一个函数装饰器，统计被装饰函数的调用次数：

``` python
class tracer:
    def __init__(self, func):
        self.calls = 0
        self.func = func
    def __call__(self, *args):
        self.calls += 1
        print('call %s to %s' % (self.calls, self.func.__name__))
        self.func(*args)

@tracer # 在def末尾触发tracer的__init__，创建了实例
def spam(a, b, c):
    print(a + b + c)

spam(1, 2, 3) # 触发tracer的__call__
spam('a', 'b', 'c')
print(spam.calls)
print(spam)
```

**2、装饰器状态保持方案**  
实例属性、全局变量、非局部闭包变量和函数属性，都可以用于保持状态：

①类实例属性  
上个例子的扩展版本：

``` python
class tracer:
    def __init__(self, func): 
        self.calls = 0 # 类实例属性保存状态
        self.func = func
    def __call__(self, *args, **kwargs):
        self.calls += 1
        print('call %s to %s' % (self.calls, self.func.__name__))
        return self.func(*args, **kwargs)

@tracer
def spam(a, b, c):
    print(a + b + c)

@tracer
def eggs(x, y):
    print(x ** y)

spam(1, 2, 3)
spam(a=4, b=5, c=6)

eggs(2, 16)
eggs(4, y=4)
```

②外层作用域和全局变量  
闭包函数（带有外围def作用域引用和嵌套的def）可以实现同样的效果：

``` python
calls = 0
def tracer(func):
    def wrapper(*args, **kwargs):
        global calls
        calls += 1
        print('call %s to %s' % (calls, func.__name__))
        return func(*args, **kwargs)
    return wrapper

@tracer
def spam(a, b, c):
    print(a + b + c)

@tracer
def eggs(x, y):
    print(x ** y)

spam(1, 2, 3)
spam(a=4, b=5, c=6)

eggs(2, 16)
eggs(4, y=4)
```

上面的方案因为计数器是全局变量，意味着被每个被包装函数所共享，对于任何函数调用，计数器都会累计，而不是各自独立计数。

③外层作用域和非局部变量  
修改上面方案，改为外层作用域的非局部变量，允许拥有各自的状态：

``` python
def tracer(func):
    calls = 0
    def wrapper(*args, **kwargs):
        nonlocal calls
        calls += 1
        print('call %s to %s' % (calls, func.__name__))
        return func(*args, **kwargs)
    return wrapper

@tracer
def spam(a, b, c):
    print(a + b + c)

@tracer
def eggs(x, y):
    print(x ** y)

spam(1, 2, 3)
spam(a=4, b=5, c=6)

eggs(2, 16)
eggs(4, y=4)
```

④函数属性  
使用 `func.attr = value` 也可以实现于 nonlocal 版本一样的结果：

``` python
def tracer(func):
    def wrapper(*args, **kwargs):
        wrapper.calls += 1
        print('call %s to %s' % (wrapper.calls, func.__name__))
        return func(*args, **kwargs)
    wrapper.calls = 0
    return wrapper

@tracer
def spam(a, b, c):
    print(a + b + c)

@tracer
def eggs(x, y):
    print(x ** y)

spam(1, 2, 3)
spam(a=4, b=5, c=6)

eggs(2, 16)
eggs(4, y=4)
```

⑤错误：对方法进行装饰
最上面跟踪调用的例子无法对类方法进行装饰，会发生错误；  
根源在于 tracer 的 `__call__` 方法的 self 参数，是一个 tracer 的实例，而并未在参数列表传递被装饰的类的主体。  
被装饰的主体类的实例没有包括在 *args 中。

``` python
class tracer:
    def __init__(self, func):
        self.calls = 0
        self.func = func
    def __call__(self, *args, **kwargs):
        self.calls += 1
        print('call %s to %s' % (self.calls, self.func.__name__))
        return self.func(*args, **kwargs)

class Person:
    def __init__(self, name, pay):
        self.name = name
        self.pay = pay
    
    @tracer
    def giveRaise(self, percent):
        self.pay *= (1.0 + percent)

bob = Person('Bob Smith', 50000)
bob.giveRaise(.25) # 会出错，因为tracer(giveRaise)(bob, .25)中的bob不会被*args接收
```

⑥使用嵌套函数装饰方法  
可以在简单函数和类级别的方法上都能工作：

``` python
def tracer(func):
    calls = 0
    def onCall(*args, **kwargs):
        nonlocal calls
        calls += 1
        print('call %s to %s' % (calls, func.__name__))
        return func(*args, **kwargs)
    return onCall

@tracer
def spam(a, b, c):
    print(a + b + c)

@tracer
def eggs(N):
    return 2 ** N

spam(1, 2, 3)
spam(a=4, b=5, c=6)
print(eggs(32))

class Person:
    def __init__(self, name, pay):
        self.name = name
        self.pay = pay
    
    @tracer
    def giveRaise(self, percent): # giveRaise = tracer(giveRaise)，调用时返回onCall(sue, .10)
        self.pay *= (1.0 + percent)
    
    @tracer
    def lastName(self):
        return self.name.split()[-1]

print('methods...')
bob = Person('Bob Smith', 50000)
sue = Person('Sue Jones', 100000)
print(bob.name, sue.name)
sue.giveRaise(.10) # Runs onCall(sue, .10)，返回了giveRaise(sue, .10)
print(int(sue.pay))
print(bob.lastName(), sue.lastName())
```

⑦使用描述符装饰方法
由于描述符的 `__get__` 方法在调用时接受描述符类实例和主体类实例，非常适合装饰类方法：

``` python
class tracer:
    def __init__(self, func):
        self.calls = 0
        self.func = func

    def __call__(self, *args, **kwargs):
        self.calls += 1
        print('call %s to %s' % (self.calls, self.func.__name__))
        return self.func(*args, **kwargs)
    
    def __get__(self, instance, owner):
        return wrapper(self, instance)

class wrapper:
    def __init__(self, desc, subj): # save了主体类实例和装饰器（描述符）实例
        self.desc = desc # 装饰器（描述符）实例
        self.subj = subj # 主体实例
    def __call__(self, *args, **kwargs):
        return self.desc(self.subj, *args, **kwargs) # Runs tracer.__call__

@tracer
def spam(a, b, c):
    print(a + b + c) # Uses __call__ only

class Person:
    def __init__(self, name, pay):
        self.name = name
        self.pay = pay
    
    @tracer
    def giveRaise(self, percent):
        self.pay *= (1.0 + percent)
    
    @tracer
    def lastName(self):
        return self.name.split()[-1]

spam(1, 2, 3)
spam(a=4, b=5, c=6)

print('methods...')
bob = Person('Bob Smith', 50000)
sue = Person('Sue Jones', 100000)
print(bob.name, sue.name)
sue.giveRaise(.10)
print(int(sue.pay))
print(bob.lastName(), sue.lastName())
```

上面的理解：  
①被装饰的函数（ spam ）只调用 tracer 类的 `__call__` ，而不会调用其 `__get__` ；
②被装饰的类方法（ giveRaise 、 lastName ）首先调用 tracer 类的 `__get__` 来解析方法名获取，返回 wrapper 实例调用，触发了 wrapper 对象的 `__call__` 方法，转而调用了 `tracer.__call__` ；
③比如 `sue.giveRaise(.10)` ，首先运行 `tracer.__get__` ，返回 `wrapper(tracer(giveRaise), sue)(sue, .10)` ，然后因为调用返回 `wrapper.__call__` ，即 `tracer(giveRaise)(sue, .10)` ，因为 sue 不会被 \*args 接收，所以不会重复 sue ，见v、错误：对方法进行装饰。最后调用 `tracer.__call__` ，返回了 `giveRaise(sue, .10)` 。

下面的版本和上面的效果一样：

``` python
class tracer(object):
    def __init__(self, func):
        self.calls = 0
        self.func = func
    def __call__(self, *args, **kwargs):
        self.calls += 1
        print('call %s to %s' % (self.calls, self.func.__name__))
        return self.func(*args, **kwargs)
    def __get__(self, instance, owner):
        def wrapper(*args, **kwargs):
            return self(instance, *args, **kwargs)
        return wrapper
```

### 三、Coding Function Decorators 2
**1、函数装饰器的第二个例子：调用计时**  
包括单次调用计时，也包括全部调用总时间：

``` python
import time

class timer:
    def __init__(self, func):
        self.func = func
        self.alltime = 0
    def __call__(self, *args, **kargs):
        start = time.perf_counter()
        result = self.func(*args, **kargs)
        elapsed = time.perf_counter() - start
        self.alltime += elapsed
        print('%s: %.5f, %.5f' % (self.func.__name__, elapsed, self.alltime))
        return result

@timer
def listcomp(N):
    return [x * 2 for x in range(N)]

@timer
def mapcall(N):
    return list(map((lambda x: x * 2), range(N)))

result = listcomp(5)
listcomp(50000)
listcomp(500000)
listcomp(1000000)
print(result)
print('allTime = %s' % listcomp.alltime) # listcomp是timer的实例

print('')
result = mapcall(5)
mapcall(50000)
mapcall(500000)
mapcall(1000000)
print(result)
print('allTime = %s' % mapcall.alltime)
print('\n**map/comp = %s' % round(mapcall.alltime / listcomp.alltime, 3))
```

实际上如果 map 没有将其包装在一个 list 调用中迫使结果生成， map 测试在 Python 中几乎不花时间，它返回一个可迭代对象而没有进行迭代。

**2、添加装饰器参数**  
提供一个输出标签并且可以打开或关闭跟踪消息：

``` python
import time

def timer(label='', trace=True):
    class Timer:
        def __init__(self, func):
            self.func = func
            self.alltime = 0
        def __call__(self, *args, **kargs):
            start = time.perf_counter()
            result = self.func(*args, **kargs)
            elapsed = time.perf_counter() - start
            self.alltime += elapsed
            if trace:
                format = '%s %s: %.5f, %.5f'
                values = (label, self.func.__name__, elapsed, self.alltime)
                print(format % values)
            return result
    return Timer

# 外层的 timer 函数，返回 Timer 类作为实际的装饰器。装饰时，它记住了被装饰的函数自身，还能访问位于外围函数作用域中的装饰器参数（ label 和 trace ）

@timer(label='[CCC]==>')
def listcomp(N):
    return [x * 2 for x in range(N)]

@timer(trace=True, label='[MMM]==>')
def mapcall(N):
    return list(map((lambda x: x * 2), range(N)))

for func in (listcomp, mapcall):
    result = func(5)
    func(50000)
    func(500000)
    func(1000000)
    print(result)
    print('allTime = %s\n' % func.alltime)
print('**map/comp = %s' % round(mapcall.alltime / listcomp.alltime, 3))
```

### 四、Coding Class Decorators
类装饰器可以用于管理类自身，或者用来拦截实例创建调用以管理实例。

**1、单例类 Singleton Classes**
下面的代码实现了传统的单例编程模式 classic singleton coding pattern ，其中每个类最多只有一个实例：

``` python
instances = {}

def singleton(aClass): # 接受被包装类，并保留状态信息，返回onCall
    def onCall(*args, **kwargs):
        if aClass not in instances:
            instances[aClass] = aClass(*args, **kwargs)
        return instances[aClass]
    return onCall

@singleton
class Person: # Person = singleton(Person) = onCall，onCall再接受实例参数
    def __init__(self, name, hours, rate):
        self.name = name
        self.hours = hours
        self.rate = rate
    def pay(self):
        return self.hours * self.rate

@singleton
class Spam:
    def __init__(self, val):
        self.attr = val

bob = Person('Bob', 40, 10) # 即onCall('Bob', 40, 10)，并返回Person实例
print(bob.name, bob.pay())

sue = Person('Sue', 50, 20) # 无法再创建新的实例，所以sue仍然是bob实例
print(sue.name, sue.pay())

print(instances)

X = Spam(val=42)
Y = Spam(99)
print(X.attr, Y.attr)
```

**2、编写替代方案**
①使用 nonlocal 语句改写上面例子，并实现了同样效果：

``` python
def singleton(aClass):
    instance = None
    def onCall(*args, **kwargs):
        nonlocal instance
        if instance == None:
            instance = aClass(*args, **kwargs)
        return instance
    return onCall
```

②使用函数属性，效果同上：

``` python
def singleton(aClass):
    def onCall(*args, **kwargs):
        if onCall.instance == None:
            onCall.instance = aClass(*args, **kwargs)
        return onCall.instance
    onCall.instance = None
    return onCall
```

③使用类，为每次装饰使用一个实例，效果同上，这个例子会在后面看到一个常见的装饰器类错误。这里只想要一个实例，实际不是这样。

``` python
class singleton:
    def __init__(self, aClass):
        self.aClass = aClass
        self.instance = None
    def __call__(self, *args, **kwargs):
        if self.instance == None:
            self.instance = self.aClass(*args, **kwargs)
        return self.instance
```

**3、跟踪对象接口**  
类装饰器的另一个常用场景是为每个生成的实例扩展接口；  
类装饰器可以在实例上安装一个包装器 wrapper 和代理 proxy 逻辑层。

①非装饰器版的委托示例：

``` python
class Wrapper:
    def __init__(self, object):
        self.wrapped = object
    def __getattr__(self, attrname):
        print('Trace:', attrname)
        return getattr(self.wrapped, attrname)

x = Wrapper([1,2,3])
x.append(4)
print(x.wrapped)

x = Wrapper({"a": 1, "b": 2})
print(list(x.keys()))
```

②类装饰器版：  
上面的类示例可以编写为一个类装饰器，能够触发被包装实例的创建；  
通过拦截实例创建调用，下面的类装饰器允许跟踪整个对象接口（即跟踪对任何属性的访问）；  
每个实例都会生成一个新的 Wrapper 实例，并拥有自己的访问计数器：

``` python
def Tracer(aClass):
    class Wrapper:
        def __init__(self, *args, **kargs):
            self.fetches = 0
            self.wrapped = aClass(*args, **kargs)
        def __getattr__(self, attrname):
            print('Trace: ' + attrname)
            self.fetches += 1
            return getattr(self.wrapped, attrname)
    return Wrapper

@Tracer
class Spam:
    def display(self):
        print('Spam!' * 8)

@Tracer
class Person:
    def __init__(self, name, hours, rate):
        self.name = name
        self.hours = hours
        self.rate = rate
    def pay(self):
        return self.hours * self.rate

food = Spam()
food.display()
print([food.fetches])

bob = Person('Bob', 40, 50)
print(bob.name)
print(bob.pay())

sue = Person('Sue', rate=100, hours=60)
print(sue.name)
print(sue.pay())

print(bob.name)
print(bob.pay())
print([bob.fetches, sue.fetches])
```

**4、类错误二：保持多个实例**
修改上面例子，使用装饰器类来装饰类，而不是装饰器函数：

``` python
class Tracer:
    def __init__(self, aClass):
        self.aClass = aClass
    def __call__(self, *args):
        self.wrapped = self.aClass(*args)
        return self
    def __getattr__(self, attrname):
        print('Trace: ' + attrname)
        return getattr(self.wrapped, attrname)

@Tracer
class Spam:
    def display(self):
        print('Spam!' * 8)

food = Spam() # Triggers __init__
food.display() # Triggers __getattr__
```

上面不能处理给定类的多个实例：每个实例构建调用都会触发 `__call__` ，会覆盖前面的实例，所以 Tracer 只能保存最后创建的实例：

``` python
@Tracer
class Person:
    def __init__(self, name):
        self.name = name

bob = Person('Bob')
print(bob.name)
Sue = Person('Sue')
print(sue.name) # sue overwrites bob
print(bob.name)
```

### 五、Managing Functions and Classes Directly  
**1、 之前的示例，都设计来拦截函数和实例创建调用；下面的示例用于管理函数和类本身**  
下面定义了一个装饰器，把函数或类对象添加到一个基于字典的注册表，并返回对象本身：

``` python
registry = {}
def register(obj):
    registry[obj.__name__] = obj
    return obj

@register
def spam(x):
    return(x ** 2)

@register
def ham(x):
    return(x ** 3)

@register
class Eggs:
    def __init__(self, x):
        self.data = x ** 4
    def __str__(self):
        return str(self.data)

print('Registry:') 
for name in registry:
    print(name, '=>', registry[name], type(registry[name]))

print('\nManual calls:')
print(spam(2))
print(ham(2))
X = Eggs(2)
print(X)

print('\nRegistry calls:')
for name in registry:
    print(name, '=>', registry[name](2))
```

**2、函数装饰器也能用来处理函数属性，并且类装饰器可以动态地插入新的类属性或方法**

``` python
def decorate(func):
    func.marked = True
    return func

@decorate
def spam(a, b):
    return a + b

print(spam.marked)

def annotate(text):
    def decorate(func):
        func.label = text
        return func
    return decorate

@annotate('spam data')
def spam(a, b):
    return a + b

print(spam(1, 2), spam.label)
```

### 六、“Private” and “Public” Attributes
**1、实现私有属性**  
下面的类装饰器实现一个用于类实例属性的 Private 声明；

不接受被装饰类的外部对属性的获取或修改访问，但仍允许类自身在自己的方法中自由地访问这些名称：

``` python
"""
Privacy for attributes fetched from class instances.
See self-test code at end of file for a usage example.

Decorator same as: Doubler = Private('data', 'size')(Doubler).
Private returns onDecorator, onDecorator returns onInstance,
and each onInstance instance embeds a Doubler instance.
"""
traceMe = False
def trace(*args):
    if traceMe: print('[' + ' '.join(map(str, args)) + ']')

def Private(*privates):
    def onDecorator(aClass):
        class onInstance:
            def __init__(self, *args, **kargs):
                self.wrapped = aClass(*args, **kargs)
            
            def __getattr__(self, attr):
                trace('get:', attr)
                if attr in privates:
                    raise TypeError('private attribute fetch: ' + attr)
                else:
                    return getattr(self.wrapped, attr)
            
            def __setattr__(self, attr, value):
                trace('set:', attr, value)
                if attr == 'wrapped':
                    self.__dict__[attr] = value # Avoid looping
                elif attr in privates:
                    raise TypeError('private attribute change: ' + attr)
                else:
                    setattr(self.wrapped, attr, value)
        return onInstance
    return onDecorator

if __name__ == '__main__':
    traceMe = True

    @Private('data', 'size')
    class Doubler:
        def __init__(self, label, start):
            self.label = label
            self.data = start
        def size(self):
            return len(self.data)
        def double(self):
            for i in range(self.size()):
                self.data[i] = self.data[i] * 2
        def display(self):
            print('%s => %s' % (self.label, self.data))
    
X = Doubler('X is', [1, 2, 3]) # self.wrapped = aClass(*args, **kargs)触发了__setattr__方法
Y = Doubler('Y is', [-10, -20, -30]) # 同上

print(X.label)
X.display(); X.double(); X.display()
print(Y.label)
Y.display(); Y.double()
Y.label = 'Spam'
Y.display()
```

下面都会出现错误：

``` python
print(X.size()) # prints "TypeError: private attribute fetch: size"
print(X.data)
X.data = [1, 1, 1]
X.size = lambda S: 0
print(Y.data)
print(Y.size())
```

在上述代码，用到了三个层面的状态保存：  
①传递给 Private 的参数在装饰发生前，作为一个外层作用域保持，供 onDecorator 和 onInstance 使用；  
② onDecorator 的类参数在装饰时，作为一个外层作用域保持，供实例构建时使用；  
③被包装的实例对象保存为 onInstance 代理对象中的一个实例属性，以便从类外部访问属性。

**2、公有声明**  
Public 声明一个类的实例属性，可以从任何地方自由地访问，而没有声明为 Public 的任何名称，不能从类的外部访问。

当使用了 Private ，所有未声明的名称就是 Public ；当使用了 Public ，所有未声明的名称就是 Private 。

``` python
"""
Class decorator with Private and Public attribute declarations.

Controls external access to attributes stored on an instance, or
Inherited by it from its classes. Private declares attribute names
that cannot be fetched or assigned outside the decorated class,
and Public declares all the names that can.

Caveat: this works in 3.X for explicitly named attributes only: __X__
operator overloading methods implicitly run for built-in operations
do not trigger either __getattr__ or __getattribute__ in new-style
classes. Add __X__ methods here to intercept and delegate built-ins.
"""
traceMe = False
def trace(*args):
    if traceMe: print('[' + ' '.join(map(str, args)) + ']')

def accessControl(failIf):
    def onDecorator(aClass):
        class onInstance:
            def __init__(self, *args, **kargs):
                self.__wrapped = aClass(*args, **kargs)
            
            def __getattr__(self, attr):
                trace('get:', attr)
                if failIf(attr):
                    raise TypeError('private attribute fetch: ' + attr)
                else:
                    return getattr(self.__wrapped, attr)
            
            def __setattr__(self, attr, value):
                trace('set:', attr, value)
                if attr == '_onInstance__wrapped': # 见31-2伪私有属性，__wrapped变成了_onInstance__wrapped
                    self.__dict__[attr] = value
                elif failIf(attr):
                    raise TypeError('private attribute change: ' + attr)
                else:
                    setattr(self.__wrapped, attr, value)
        return onInstance
    return onDecorator

def Private(*attributes):
    return accessControl(failIf=(lambda attr: attr in attributes))

def Public(*attributes):
    return accessControl(failIf=(lambda attr: attr not in attributes))

@Private('age')
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

X = Person('Bob', 40)
print(X.name)
X.name = 'Sue'
print(X.name)
# X.age # TypeError: private attribute fetch: age
# X.age = 'Tom' # TypeError: private attribute change: age

@Public('name')
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

X = Person('bob', 40)
print(X.name)
X.name = 'Sue'
print(X.name)
X.age # TypeError: private attribute fetch: age
X.age = 'Tom' # TypeError: private attribute change: age
```

虽然控制了对实例及类属性的访问控制，但是仍然可以显示地调用：

``` python
print(X._onInstance__wrapped.age)
```

内置操作隐式运行地 `__X__` 运算符重载方法不会在新式类触发 `__getattr__` 或 `__getattribute__` ：

``` python
@Private('age')
class Person:
    def __init__(self):
        self.age = 42
    def __str__(self):
        return 'Person: ' + str(self.age)
    def __add__(self, yrs):
        self.age += yrs

X = Person()
print(X.age) # TypeError: private attribute fetch: age
print(X)
X + 10 # TypeError: unsupported operand type(s) for +: 'onInstance' and 'int'
```


## Chapter 40 Metaclasses
### 一、Metaclass Basics
**元类**是创建类的类。  

某种程度上来说，元类只是扩展了装饰器的代码插入模型，元类允许拦截并扩展类的创建，提供了一种在 class 语句结束时运行插入额外逻辑的 API 。元类主要由构建API工具的程序员使用。  

类装饰器在被装饰类创建完成之后运行；而元类在类创建过程中就运行，创建并返回新的客户类。

**1、类是类型的实例**
用户定义的类对象是名为 **type** 的对象的实例，**type** 本身是一个类；  
类继承自 **object** ， **object** 是 **type** 的一个子类；  

内置类型的实例的类型是内置的类型，例如列表的实例的类型是 list ，而列表类型的类型是 type 本身。

**类即类型，类型即类**。

``` python
print(type([]), type(type([])))
print(type(list), type(type))
```

用户定义的类是产生它们自己的实例的类型；

类有链接到 type 的一个 `__class__` 属性，就像实例有链接到创建它的类的 `__class__` 一样：

``` python
class C: pass
X = C()
print(type(X))
print(X.__class__)
print(type(C))
print(C.__class__)
```

**2、元类是 Type 的子类**  
因为类是 type 类的实例，所以从 type 的定制的子类创建类允许我们实现各种定制的类；

①元类是 type 类的子类；  
②类对象是 type 类的实例或子类；

为了控制创建类以及扩展其行为的方式，可以指定一个用户定义的类创建自一个用户定义的元类，而不是常规的 type 类。

主要上面的类型实例关系与继承不同，但不会暴露在正常的继承搜索中，即不出现在类的 `__bases__` 元组中，见后面。

**3、class 语句协议**  
在一条 class 语句的末尾，Python 遵循一个标准协议，运行了所有内嵌的代码后，python 会调用 type 对象来创建 class 对象：  
`class = type(classname, superclasses, attributedict)`  

type 对象定义了一个 `__call__` 运算符重载方法，当 type 被调用时，该方法运行2个其他方法：  
① `type.__new__(typeclass, classname, superclasses, attributedict)`  
② `type.__init__(class, classname, superclasses, attributedict)`  
`__new__` 方法创建并返回新的 class 对象，然后 `__init__` 方法初始化新创建的对象。

例如：

``` python
class Eggs: ...

class Spam(Eggs):
    data = 1
    def meth(self, arg):
        return self.data + arg
```

这里会在 class 语句末尾调用 type 对象来产生 class 对象：  
`Spam = type('Spam', (Eggs,), {'data': 1, 'meth': meth, '__module__': '__main__'})`

也可以显示地调用 type 来动态地创建一个类：

``` python
x = type('Spam', (), {'data': 1, 'meth': (lambda x, y: x.data + y)}) # 空的父类元组会自动添加object父类
i = x()
print(x, i)
print(i.data, i.meth(2))
print(x.__bases__)
```

**4、声明元类**  
在类头部把想要使用的元类作为关键字参数列出来：  
`class Spam(metaclass=Meta):`  
与继承的父类同时列在头部，父类必须列在元类之前：  
`class Spam(Eggs, metaclass=Meta):`

当一个特定的元类按照上面的语法声明时，运行在 class 语句末尾来创建 class 对象的调用被修改为了调用元类而不是默认的 type ：`class = Meta(classname, superclasses, attributedict)`

因为元类是 type 的一个子类，所以如果元类定义了 `__new__` 和 `__init__` 方法的话，那么 type 的 `__call__` 会把创建和初始化新的 class 对象的调用委托给元类：  
`Meta.__new__(Meta, classname, superclasses, attributedict)`  
`Meta.__init__(class, classname, superclasses, attributedict)`

例如：

``` python
class Spam(Eggs, metaclass=Meta):
    data = 1
    def meth(self, arg):
        return self.data + arg
```

在 class 语句末尾，python 内部会自动运行如下代码：  
`Spam = Meta('Spam', (Eggs,), {'data': 1, 'meth': meth, '__module__': '__main__'})`

如果元类定义了自己的 `__new__` 或 `__init__` ，在调用期间，它们会依次由所继承的 type 类的 `__call__` 方法调用，以创建并初始化新类。

### 二、Coding Metaclasses
**1、一个基础的元类**  
简单的示例：一个带有 `__new__` 方法的 type 的子类：

``` python
class Meta(type):
    def __new__(meta, classname, supers, classdict):
        """Run by inherited type.__call__"""
        return type.__new__(meta, classname, supers, classdict)
```

元类的 `__new__` 方法通常执行所需的定制并且调用 type 父类的 `__new__` 方法来创建并返回新的类对象。

稍微复杂的示例：

``` python
class MetaOne(type):
    def __new__(meta, classname, supers, classdict):
        print('In MetaOne.new:', meta, classname, supers, classdict, sep='\n...')
        return type.__new__(meta, classname, supers, classdict)

class Eggs:
    pass

print('making class')
class Spam(Eggs, metaclass=MetaOne):
    data = 1
    def meth(self, arg):
        return self.data + arg

print('making instance')
X = Spam()
print('data:', X.data, X.meth(2))
```

在 class 语句末尾调用元类，Spam 继承自 Eggs 并且是 Metaone 的一个实例，X 是 Spam 的一个实例并且继承自 Spam 。

**2、定制构建和初始化**
`__new__` 创建并返回了类对象，而 `__init__` 初始化了作为参数被传入的已经创建的类：

``` python
class MetaTwo(type):
    def __new__(meta, classname, supers, classdict):
        print('In MetaTwo.new: ', classname, supers, classdict, sep='\n...')
        return type.__new__(meta, classname, supers, classdict)

    def __init__(Class, classname, supers, classdict):
        print('In MetaTwo.init:', classname, supers, classdict, sep='\n...')
        print('...init class object:', list(Class.__dict__.keys()))
    
class Eggs:
    pass

print('making class')
class Spam(Eggs, metaclass=MetaTwo):
    data = 1
    def meth(self, arg):
        return self.data + arg

print('making instance')
X = Spam()
print('data:', X.data, X.meth(2))
```

类初始化方法（ `__init__` ）在类构建方法（ `__new__` ）之后运行；  
Spam 的 `__init__` 会在实例创建的时候运行，而不会被元类的 `__init__` 影响。

**3、其他元类编写技巧**  
①使用工厂函数

``` python
def MetaFunc(classname, supers, classdict):
    print('In MetaFunc: ', classname, supers, classdict, sep='\n...')
    return type(classname, supers, classdict)

class Eggs:
    pass

print('making class')
class Spam(Eggs, metaclass=MetaFunc):
    data = 1
    def meth(self, arg):
        return self.data + arg

print('making instance')
X = Spam()
print('data:', X.data, X.meth(2))
```

②用普通类重载类创建调用  
下面的类没有继承自 type ，而是提供了一个 `__call__` 方法；  
`__new__` 和 `__init__` 要重新命名为其他名称（比如下面的 \_\_New\_\_ 和 \_\_Init\_\_ ），否则会在 Meta 实例创建时运行，而不是之后在元类的角色中被调用：

``` python
class MetaObj:
    def __call__(self, classname, supers, classdict):
        print('In MetaObj.call: ', classname, supers, classdict, sep='\n...')
        Class = self.__New__(classname, supers, classdict)
        self.__Init__(Class, classname, supers, classdict)
        return Class
    
    def __New__(self, classname, supers, classdict):
        print('In MetaObj.new: ', classname, supers, classdict, sep='\n...')
        return type(classname, supers, classdict)
    
    def __Init__(self, Class, classname, supers, classdict):
        print('In MetaObj.init:', classname, supers, classdict, sep='\n...')
        print('...init class object:', list(Class.__dict__.keys()))

class Eggs:
    pass

print('making class')
class Spam(Eggs, metaclass=MetaObj()): # MetaObj是一个类实例
    data = 1
    def meth(self, arg):
        return self.data + arg

print('making instance')
X = Spam()
print('data:', X.data, X.meth(2))
```

三个方法通过正常实例继承来的 `__call__` 方法被分发。

使用父类继承来扮演 type 类似的角色：

``` python
class SuperMetaObj:
    def __call__(self, classname, supers, classdict):
        print('In SuperMetaObj.call: ', classname, supers, classdict, sep='\n...')
        Class = self.__New__(classname, supers, classdict)
        self.__Init__(Class, classname, supers, classdict)
        return Class

class SubMetaObj(SuperMetaObj):
    def __New__(self, classname, supers, classdict):
        print('In SubMetaObj.new: ', classname, supers, classdict, sep='\n...')
        return type(classname, supers, classdict)

    def __Init__(self, Class, classname, supers, classdict):
        print('In SubMetaObj.init:', classname, supers, classdict, sep='\n...')
        print('...init class object:', list(Class.__dict__.keys()))

class Spam(Eggs, metaclass=SubMetaObj()):
    data = 1
    def meth(self, arg):
        return self.data + arg

print('making instance')
X = Spam()
print('data:', X.data, X.meth(2))
```

③用元类重载 `__call__`  
对 `__new__` 和 `__call__` 的重载需调用 type 来启动：

``` python
class SuperMeta(type):
    def __call__(meta, classname, supers, classdict):
        print('In SuperMeta.call: ', classname, supers, classdict, sep='\n...')
        return type.__call__(meta, classname, supers, classdict)

    def __init__(Class, classname, supers, classdict):
        print('In SuperMeta init:', classname, supers, classdict, sep='\n...')
        print('...init class object:', list(Class.__dict__.keys()))

print('making metaclass')
class SubMeta(type, metaclass=SuperMeta):
    def __new__(meta, classname, supers, classdict):
        print('In SubMeta.new: ', classname, supers, classdict, sep='\n...')
        return type.__new__(meta, classname, supers, classdict)
    def __init__(Class, classname, supers, classdict):
        print('In SubMeta init:', classname, supers, classdict, sep='\n...')
        print('...init class object:', list(Class.__dict__.keys()))

class Eggs:
    pass

print('making class')
class Spam(Eggs, metaclass=SubMeta): # Invoke SubMeta, via SuperMeta.__call__
    data = 1
    def meth(self, arg):
        return self.data + arg

print('making instance')
X = Spam()
print('data:', X.data, X.meth(2))
```

元类用于创造类对象，但是只在作为一个元类角色被调用的时候才产生它们的实例；  
元类也可以从其他元类继承名称，元类间的继承只适用于显式的名称获取，而不能作用于内置操作的调用的隐式名称查找；  
内置操作的调用的隐式名称可以在它的 `__class__` 中找到，要么是默认的 type ，要么是一个元类。

SubMeta 中的 metaclass 是必要的；  
SuperMeta 的 `__call__` 方法不会运行在 SubMeta 创建时的 SuperMeta 调用，而是会运行在 Spam 创建时的 SubMeta 调用；  
SubMeta 的创建会路由到 type 。

如下：

``` python
class SuperMeta(type):
    def __call__(meta, classname, supers, classdict):
        print('In SuperMeta.call:', classname)
        return type.__call__(meta, classname, supers, classdict)
    
class SubMeta(SuperMeta): # Created by type default，普通的父类被内置操作跳过，但显式的获取调用不会跳过
    def __init__(Class, classname, supers, classdict): 
        print('In SubMeta init:', classname)

print(SubMeta.__class__)
print([n.__name__ for n in SubMeta.__mro__])
print()
print(SubMeta.__call__) # 显式调用
print()
SubMeta.__call__(SubMeta, 'xxx', (), {}) # 显式调用：运行了SuperMeta的__call__，元类继承
print()
SubMeta('yyy', (), {}) # 隐式内置调用：没运行SuperMeta的__call__，元类的类是type
```

更深刻的理解见后面。

### 三、Inheritance and Instance
**1、元类和父类继承**  
区分元类的指定方式和父类继承，相似但不一样。

①元类继承自 type 类  
元类通常重新定义 type 类的 `__new__` 和 `__init__` 方法，也可以重新定义 `__call__` （不常见，而且会出现上一节中看到的复杂性）

②元类声明会被子类继承  
`metaclass=M` 会被该类的普通子类继承，继承了该声明的类的构建都会运行该元类。

③元类属性不会被类实例继承  
因为类是元类的实例，所以元类中定义的行为适用于类，但不适用于类的实例；  
实例从类和父类获取行为，但不会从元类获取行为；  
普通实例属性继承通常只查找该实例、对应的类、所有父类的 `__dict__` 字典；不包括元类。

④元类属性会被类获取  
类能通过继承关系从元类获得方法；  
类通过类的 `__class__` 链接来获取元类属性，这跟普通实例从它们的类获取名称是一样的，但是通过 `__dict__` 继承的名称会被优先搜索；  
当同一名称同时出现在元类和父类，父类的版本（通过继承）会被优先使用，而元类（作为元类的实例）会被忽略；  
而类的 `__class__` 不会被它本身的实例所继承。

``` python
class MetaOne(type):
    def __new__(meta, classname, supers, classdict):
        print('In MetaOne.new:', classname)
        return type.__new__(meta, classname, supers, classdict)

    def toast(self):
        return 'toast'

class Super(metaclass=MetaOne):
    def spam(self):
        return 'spam'

class Sub(Super):
    def eggs(self):
        return 'eggs'
```

上面代码运行时，元类会同时处理2个客户类的构建，并且实例继承类属性而不是元类：

``` python
X = Sub()
print(X.eggs(), X.spam())
print(X.toast()) # 会出错：AttributeError: 'Sub' object has no attribute 'toast'
```

但类可以从父类继承名字，也可以从元类获取名字；  
从元类获取的方法被绑定到了主体类上，从普通类获取的方法通过类获取是非绑定的，而通过实例获取是绑定的：

``` python
print(Sub.eggs(X), Sub.spam(X))
print(Sub.toast())
print(Sub.toast)
print(Sub.spam)
print(X.spam)
```

**2、元类 vs 父类**
简单地来说，类作为元类的实例继承了元类的属性，但是这个属性不能被类自己的实例继承：

``` python
class A(type): attr = 1
class B(metaclass=A): pass
I = B()
print(B.attr)
print(I.attr) # 出现错误：AttributeError: 'B' object has no attribute 'attr'
print('attr' in B.__dict__, 'attr' in A.__dict__)
```

把 A 从元类改为父类：

``` python
class A: attr = 1
class B(A): pass
I = B()
print(B.attr)
print(I.attr)
print('attr' in B.__dict__, 'attr' in A.__dict__)
```

同一名称出现在父类和元类中的情形：

``` python
class M(type): attr = 1
class A: attr = 2
class B(A, metaclass=M): pass
I = B()
print(B.attr, I.attr)
print('attr' in B.__dict__, 'attr' in A.__dict__, 'attr' in M.__dict__)
```

python 会优先通过 MRO （通过继承）检查每个类的 `__dict__` ，之后再到元类（作为实例）中获取：

``` python
class M(type): attr = 1
class A: attr = 2
class B(A): pass
class C(B, metaclass=M): pass
I = C()
print(I.attr, C.attr)
print([x.__name__ for x in C.__mro__])
```

类通过自身的 `__class__` 链接来获取元类属性（即实例通过 `__class__` 获取类属性的方式）；  
但是实例继承是将其作用域限制在 MRO 顺序搜索到的每一个类的 `__dict__` 中，即跟随每个类的 `__bases__` ，而且只使用实例的 `__class__` 链接一次：

``` python
print(I.__class__)
print(C.__bases__)
print(C.__class__)
print(C.__class__.attr)
```

**3、继承：完整的例子**
①实例从它的类继承；类则从类和元类继承；元类从父元类继承：  

``` python
class M1(type): attr1 = 1
class M2(M1): attr2 = 2

class C1: attr3 = 3
class C2(C1,metaclass=M2): attr4 = 4

I = C2()
print(I.attr3, I.attr4)
print(C2.attr1, C2.attr2, C2.attr3, C2.attr4)
print(M2.attr1, M2.attr2)

print(I.__class__, C2.__bases__)
print(C2.__class__, M2.__bases__)

print(M2.__class__)
print([x.__name__ for x in C2.__mro__]) # __bases__ tree from I.__class__
print([x.__name__ for x in M2.__mro__]) # __bases__ tree from C2.__class__
```

综上所述，继承会在利用 `__class__` 之前先利用 `__bases__` ，因为 `__bases__` 被用于创造类的时候建立 `__mro__` 顺序，而继承基于 MRO 。  
普通实例没有 `__bases__` ；而类二者都有，包括类和元类。

②继承算法顺序  
- 从实例 I 出发，先搜索该实例，再搜索它的类，之后搜索所有父类：  
    - 实例 I 的 `__dict__`；   
    - 所有在 I 的 `__class__` 中的 `__mro__` 找到的类的 `__dict__`。  
- 从类 C 出发，先搜索该类，再搜索它的所有父类，之后搜索它的元类树：  
    - 所有 C 的 `__mro__` 中的类的 `__dict__` ； 
    - 所有在 C 的 `__class__` 中的 `__mro__` 找到的类的 `__dict__` ；  
- 步骤 b 出现的数据描述符具有优先权；
- 内置操作跳过步骤 a ，从步骤 b 开始搜索。

③描述符特例  

``` python
class D:
    def __get__(self, instance, owner): print('__get__')
    def __set__(self, instance, value): print('__set__')

class C: d = D() # Data descriptor attribute
I = C()
I.d
I.d = 1
I.__dict__['d'] = 'spam'
I.d # 命名空间的d不会覆盖数据描述符
```

④内置操作特例  
实例和类都会跳过内置操作；  
比如，str 是内置操作，`__str__` 是它等价的显式名称，实例在内置操作的搜索被跳过：

``` python
class C:
    attr = 1
    def __str__(self): return('class')

I = C()
print(I.__str__(), str(I)) # 都来自类C

I.__str__ = lambda: 'instance'
print(I.__str__(), str(I)) # 前者来自实例的命名空间，后者来自C的命名空间，即显式调用来自实例，内置操作来自类
```

同样规则也适用于类与元类，显式名称搜索从类开始，内置操作从类的类开始，即元类（默认是 type ）：

``` python
class D(type):
    def __str__(self): return('D class')
class C(D):
    pass
print(C.__str__(C), str(C)) # C的元类是type

class C(D):
    def __str__(self): return('C class')
print(C.__str__(C), str(C))

class C(metaclass=D):
    def __str__(self): return('C class')
print(C.__str__(C), str(C))
```

所有的类也继承自 object ，包括默认的 type 元类。

比如下面，C 按照继承（ MRO ）从 object 获取了默认的 `__str__` ，而非从元类中：

``` python
class C(metaclass=D):
    pass
print(C.__str__(C), str(C))
print(C.__str__)

for k in (C, C.__class__, type): print([x.__name__ for x in k.__mro__])
```

### 四、Metaclass Methods
**1、元类方法**  
元类方法能够处理对应的实例类，不是普通实例对象 self ，而是类本身：

``` python
class A(type):
    def x(cls): print('ax', cls) # 注意：不是self
    def y(cls): print('ay', cls)

class B(metaclass=A):
    def y(self): print('by', self)
    def z(self): print('bz', self)

print(B.x, B.y, B.z)
B.x()

I = B()
I.y()
I.z()
I.x() # 出现错误：AttributeError: 'B' object has no attribute 'x'
```

**2、元类方法中的运算符重载**

``` python
class A(type):
    def __getattr__(cls, name):
        return getattr(cls.data, name)

class B(metaclass=A):
    data = 'spam'

print(B.upper())
print(B.upper)
print(B.__getattr__)
I = B()
I.upper # 出现错误：AttributeError: 'B' object has no attribute 'upper'
I.__getattr__ # 出现错误：AttributeError: 'B' object has no attribute '__getattr__'
```

### 五、Examples
**1、实例：向类添加方法**

``` python
def eggsfunc(obj):
    return obj.value * 4

def hamfunc(obj, value):
    return value + 'ham'

class Extender(type):
    def __new__(meta, classname, supers, classdict):
        classdict['eggs'] = eggsfunc
        classdict['ham'] = hamfunc
        return type.__new__(meta, classname, supers, classdict)

class Client1(metaclass=Extender):
    def __init__(self, value):
        self.value = value
    def spam(self):
        return self.value * 2

class Client2(metaclass=Extender):
    value = 'ni?'

X = Client1('Ni!')
print(X.spam())
print(X.eggs())
print(X.ham('bacon'))

Y = Client2()
print(Y.eggs())
print(Y.ham('bacon'))
```

上述示例中的元类把2个已知的方法添加到了声明了元类的每个类中。

**2、基于装饰器的上述代码**

``` python
def eggsfunc(obj):
    return obj.value * 4

def hamfunc(obj, value):
    return value + 'ham'

def Extender(aClass):
    aClass.eggs = eggsfunc
    aClass.ham = hamfunc
    return aClass

@Extender
class Client1:
    def __init__(self, value):
        self.value = value
    def spam(self):
        return self.value * 2

@Extender
class Client2:
    value = 'ni?'

X = Client1('Ni!')
print(X.spam())
print(X.eggs())
print(X.ham('bacon'))

Y = Client2()
print(Y.eggs())
print(Y.ham('bacon'))
```

**3、元类 vs 类装饰器**
①类装饰器可以管理类和实例，但是通常不能创建类，需要额外步骤来创建新的类；  
②元类可以管理类和实例，但是管理实例需要一些额外的工作； 

类装饰器在 class 语句末尾，把类名绑定到装饰器函数或类的结果；  
元类通过一条 class 语句末尾把类对象的创建路由到一个对象，从而创建新的类。

本章后面有元类和类装饰器比较以及结合的例子，因为暂无学习的必要性，未作摘抄，大致了解即可，之后面向需求学习。