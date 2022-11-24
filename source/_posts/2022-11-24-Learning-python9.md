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
