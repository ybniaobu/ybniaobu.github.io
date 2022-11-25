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
> 本篇主要内容为：XXXXXXXXXXXXXXXXXX。

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
① fget 传入函数（或类方法）用于拦截属性访问；  
② fset（或类方法）传入函数用于属性赋值；  
③ fdel（或类方法）传入函数用于属性删除；  
④ fget 函数返回被计算好的属性值，fset 和 fdel 返回 None ；  
⑤ doc 参数接受该属性的一个文档字符串。

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
