---
title: 《Learning Python》读书笔记（八）
date: 2022-11-18 13:59:19
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

# PART VI Classes and OOP
## chapter 29 Class Coding Details
### 一、The class Statement
1、Class 语句是对象的创建者并且是一个隐含的赋值运算：当它执行时产生类对象，并把其引用值存储到前面所使用的名称中
- class语句的一般形式：

``` python
class name(superclass,...):
    attr = value 
    def method(self,...):
        self.attr = value
```

2、示例
- 当 python 执行 class 语句时，会从头到尾执行其主体内语句；
- 在 class 语句中的赋值语句所创建的名称，位于 class 的局部作用域中，会成为类对象中的属性。

``` python
class SharedData:
    spam = 42
    print(spam)

x = SharedData()
y = SharedData()
print(x.spam, y.spam)
```

- ***可以通过类名称修改类属性***：

``` python
SharedData.spam = 99
print(x.spam, y.spam, SharedData.spam)
```

- ***对实例对象赋值修改实例属性***：

``` python
x.spam = 88
print(x.spam, y.spam, SharedData.spam)
```

- 以下例子能更好解释这种把同一名称存储在两个位置的行为：

``` python
class MixedNames:
    data = 'spam'
    def __init__(self, value):
        self.data = value
    def display(self):
        print(self.data, MixedNames.data)

x = MixedNames(1)
y = MixedNames(2)
x.display(); y.display()
```

运行结果如下：

``` python
1 spam
2 spam
```

- data 位于 2 个地方：在实例对象内（由 \_\_init\_\_ 中的 self.data 赋值运算所创建以及在实例继承的类中（由类中的data赋值运算所创建）


### 二、Methods
1、在一个类方法中，第一位参数为 self 。这个参数给方法提供了一个钩子 hook ，从而返回调用的主体，即**实例对象**。
- 这个名称的存在是为了明确脚本中使用的是实例的属性名称，而不是局部作用域或全局作用域中的名称。

2、示例

``` python
class NextClass:
    def printer(self, text):
        self.message = text
        print(self.message)

x = NextClass()
x.printer('instance call')
print(x.message)
```

- 当对实例进行点号运算来调用它时，printer 会先通过继承进行定位，然后它的 self 参数会被自动赋值为实例对象；
- text 参数会得到在调用时传入的字符串（'instance call'）；
- *方法能通过实例或类本身2种方式的任意一种进行调用*：

``` python
NextClass.printer(x, 'class call')
print(x.message)
```


### 三、Inheritance
1、在 Python 中，当对对象进行点号运算时就会触发继承，以及搜索属性定义树，即在一个或多个相互链接的命名空间中搜索
- 实例属性是由对方法内的 self 属性进行赋值运算产生的；
- 类属性是通过 class 语句内的语句创建的；
- 父类的连接是通过 class 语句首行的括号内列出的类而产生的；
- 结果就是连接实例的属性命名空间树，到产生它的类、再到类首行中列出的所有父类。

2、子类可以完全替代继承的属性，也可以拓展父类方法

``` python
class Super:
    def method(self):
        print('in Super.method')

class Sub(Super):
    def method(self):
        print('starting Sub.method')
        Super.method(self)
        print('ending Sub.method')
```

- 这类 sub 替代了 super 的方法，同时又调用了 super 的方法，即拓展：

``` python
x = Super()
x.method()

x = Sub()
x.method()
```

3、抽象父类 Abstract Superclasses

``` python
class Super:
    def delegate(self):
        self.action()

class Provider(Super):
    def action(self):
        print('in Provider.action')

x = Provider()
x.delegate()
```

- 上述代码的调用 delegate 方法时，会发生 2 次独立的继承搜索：
    - x.delegate 调用时，python 会搜索 Provider 实例和类树中更上层的类对象，直到 Super 中找到 delegate ，实例 x 传给该方法的 self 参数；
    - 在 Super.delegate 方法中， self.action 会对 self 以及它上层的对象发起另一次继承搜索。因为 self 引用了一个 Provider 实例，所以 action 方法会在 Provider 子类中找到；
- 这个例子中的父类有时被称为**抽象父类 abstract superclass**，即类的部分行为预期由其子类来提供；
- 如果所预期的方法没有在子类中定义，那么当继承搜索失败时， python 会引发名称未定义的异常。


### 四、Namespaces: The Conclusion
1、**命名空间 Namespaces**与**作用域 Scopes**
- 带点号和无点号的名称采用不同的处理方式；
- 无点号运算的名称( X )对应于作用域；
- 带点号的属性名( object.X )使用的是对象的命名空间。

``` python
X = 11

def f():
    print(X)

def g():
    X = 22
    print(X)

class C:
    X = 33
    def m(self):
        X = 44
        print(X)
        self.X = 55
```

- 上述代码产生了5个 X 变量：模块属性（11）、函数内的局部变量（22）、类属性（33）、方法中的局部变量（44）以及实例属性（55）：

``` python
print(X)
f()
g()
print(X)

obj = C()
print(obj.X)

obj.m()
print(obj.X)
print(C.X)
```

运行结果如下：

``` python
11
11
22
11
33
44
55
33
```

2、嵌套的类：重温 LEGB 作用域规则

（1）

``` python
X = 1

def nester():
    print(X)                         # Global: 1
    class C:
        print(X)                     # Global: 1
        def method1(self):
            print(X)                 # Global: 1
        def method2(self):
            X = 3
            print(X)                 # Local: 3
    I = C()
    I.method1()
    I.method2()

print(X)
nester()
print('-'*40)
```

（2）

``` python
X = 1

def nester():
    X = 2
    print(X)                         # Local: 2
    class C:
        print(X)                     # In enclosing def (nester): 2
        def method1(self):
            print(X)                 # In enclosing def (nester): 2
        def method2(self):
            X = 3
            print(X)                 # Local: 3
    I = C()
    I.method1()
    I.method2()

print(X)
nester()
print('-'*40)
```

（3）

``` python
X = 1

def nester():
    X = 2
    print(X)                        # Local: 2
    class C:
        X = 3
        print(X)                    # Local: 3
        def method1(self):
            print(X)                # In enclosing def (not 3 in class!): 2
            print(self.X)           # Inherited class local: 3
        def method2(self):
            X = 4
            print(X)                # Local: 4
            self.X = 5
            print(self.X)           # Located in instance: 5
    I = C()
    I.method1()
    I.method2()

print(X)
nester()
print('-'*40)
```

上面3个运行结果如下：

```
1
1
1
1
3
----------------------------------------
1
2
2
2
3
----------------------------------------
1
2
3
2
3
4
5
----------------------------------------
```

- 像 X 这样简单名称的查找规则从来不会搜索外层 class 语句，只搜索 def 语句、模块和内置的；
- 是 LEGB 规则，不是 CLEGB ！！
- 所以在 method1 中， X 在位于其外层类外部的 def 语句中被找到；
- 想要访问类中被赋值的名称，必须将其作为类对象或者实例对象的属性来获取。


### 五、Namespace Dictionaries: Review
1、命名空间字典
- 如同模块的命名空间，类对象和实例对象的命名空间被实现为字典，即内置的 `__dict__ 属性`；
- 属性继承就是搜索链接的字典；实例对象和类对象就是互相之间带有链接的字典：

``` python
class Super:
    def hello(self):
        self.data1 = 'spam'

class Sub(Super):
    def hola(self):
        self.data2 = 'eggs'
```

- 当我们创建子类的实例时，该实例一开始是空的命名空间字典，但是有指向它的类的链接，让继承搜索能顺着寻找；
- 继承树可在特殊的属性中显式获取；
- 实例对象的 `__class__ 属性`链接到了它的类，而类有一个 `__bases__ 属性`（一个元组），其中包含通往更高的父类的链接：

``` python
X = Sub()
print(X.__dict__)
print(X.__class__)
print(Sub.__bases__)
print(Super.__bases__)
```

- 当类为 self 属性赋值时，属性会位于实例的属性命名空间字典内：

``` python
Y = Sub()
X.hello()
print(X.__dict__)
X.hola()
print(X.__dict__)
print(list(Sub.__dict__.keys()))
print(list(Super.__dict__.keys()))
print(Y.__dict__)
```

- 因为属性在 python 内部是字典键，所以有2种方式进行访问或赋值：

``` python
print(X.data1, X.__dict__['data1'])
X.data3 = 'toast'
print(X.__dict__)
X.__dict__['data3'] = 'ham'
print(X.__dict__)
```

- 但是属性的点号运算会执行继承搜索，而字典索引无法访问继承属性；
- 比如继承的属性 X.hello 就无法由 `X.__dict__['hello']` 来访问。


### 六、Documentation Strings Revisited
1、如同模块，文档字符串 Documentation Strings 也可以用于类
- 文本字符串由 python 自动保存到相应对象的 \_\_doc\_\_ 属性中；
- 文档字符串适用于模块文件、函数定义，以及类和方法：

``` python
# docstr.py
"I am: docstr.__doc__"

def func(args):
    "I am: docstr.func.__doc__"
    pass

class spam:
    "I am: spam.__doc__ or docstr.spam.__doc__ or self.__doc__"
    def method(self):
        "I am: spam.method.__doc__ or self.method.__doc__"
        print(self.__doc__)
        print(self.method.__doc__)
```

``` python
import docstr
print(docstr.__doc__)
print(docstr.func.__doc__)
print(docstr.spam.__doc__)
print(docstr.spam.method.__doc__)

x = docstr.spam()
x.method()
```

- 第15章还讨论了 PyDoc 工具，该工具能将这些文档字符串整理成报告和网页：`help(docstr)`。


## chapter 30 Class Coding Details
### 一、The Basics
1、运算符重载 operator overloading 基础知识
- 运算符重载让类拦截常规的 Python 操作；
- 类可重载所有 python 表达式运算符；
- 类也可以重载打印、函数调用、属性访问等内置运算；
- 重载使类实例的行为更接近内置类型；
- 重载是通过在一个类中提供特殊名称的方法来实现的。

2、构造函数和捕捉减法的表达式： \_\_init\_\_ 和 \_\_sub\_\_

``` python
class Number:
    def __init__(self, start):
        self.data = start
    def __sub__(self, other):
        return Number(self.data - other)

X = Number(5)
Y = X - 2
print(Y.data)
```

- \_\_sub\_\_ 跟第27章的 \_\_add\_\_ 方法类似，会拦截减法表达式并返回一个新的对象实例作为结果；
- 在一个实例被创建的过程中，首先触发是 \_\_new\_\_ 方法，这一方法将创建并返回一个新的实例对象，并传入 \_\_init\_\_ 函数以供初始化，见第40章。

3、常见的运算符重载方法

| 方法名 Method | 实现功能 Implements | 触发调用的形式 Called for |
| :---- | :---- | :---- |
| \_\_init\_\_ | Constructor | Object creation: X = Class(args) |
| \_\_del\_\_ | Destructor | Object reclamation of X |
| \_\_add\_\_ | Operator + | X + Y, X += Y if no \_\_iadd\_\_ |
| \_\_or\_\_ | Operator &#124; (bitwise OR) | X &#124; Y, X &#124;= Y if no \_\_ior\_\_ |
| \_\_repr\_\_, \_\_str\_\_ | Printing, conversions | print(X), repr(X), str(X) |
| \_\_call\_\_ | Function calls | X(\*args, \*\*kargs) |
| \_\_getattr\_\_ | Attribute fetch | X.undefined |
| \_\_setattr\_\_ | Attribute assignment | X.any = value |
| \_\_delattr\_\_ | Attribute deletion | del X.any |
| \_\_getattribute\_\_ | Attribute fetch | X.any |
| \_\_getitem\_\_ | Indexing, slicing, iteration | X[key], X[i:j], for loops and other iterations if no \_\_iter\_\_ |
| \_\_setitem\_\_ | Index and slice assignment | X[key] = value, X[i:j] = iterable |
| \_\_delitem\_\_ | Index and slice deletion | del X[key], del X[i:j] |
| \_\_len\_\_ | Length | len(X), truth tests if no \_\_bool\_\_ |
| \_\_bool\_\_ | Boolean tests | bool(X), truth tests (named \_\_nonzero\_\_ in 2.X) |
| \_\_lt\_\_, \_\_gt\_\_ | Comparisons | X < Y, X > Y |
| \_\_le\_\_, \_\_ge\_\_ | Comparisons | X <= Y, X >= Y |
| \_\_eq\_\_, \_\_ne\_\_ | Comparisons | X == Y, X != Y |
| \_\_radd\_\_ | Right-side operators | Other + X |
| \_\_iadd\_\_ | In-place augmented operators | X += Y (or else \_\_add\_\_) |
| \_\_iter\_\_, \_\_next\_\_ | Iteration contexts | I=iter(X), next(I); for loops, in if no \_\_contains\_\_, all comprehensions, map(F,X), others |
| \_\_contains\_\_ | Membership test | item in X (any iterable) |
| \_\_index\_\_ | Integer value | hex(X), bin(X), oct(X), O[X], O[X:] (replaces 2.X \_\_oct\_\_, \_\_hex\_\_) |
| \_\_enter\_\_, \_\_exit\_\_ | Context manager | with obj as var: |
| \_\_get\_\_, \_\_set\_\_, \_\_delete\_\_ | Descriptor attributes | X.attr, X.attr = value, del X.attr |
| \_\_new\_\_ | Creation | Object creation, before \_\_init\_\_ |

- 运算符重载方法是可选的，如果没有给出相应的运算符重载方法的话，大多数内置函数会对类实例失效。

### 二、Indexing and Slicing: \_\_getitem\_\_ and \_\_setitem\_\_
1、`__getitem__`：如果类定义或继承了该方法，该方法会自动被调用并进行实例的索引运算
- 即实例 X 出现在 `X[i]` 的运算中，python 会调用实例继承的 `__getitem__` 方法，把 X 作为第一个参数传入，索引值传给第二个参数：

``` python
class Indexer:
    def __getitem__(self, index):
        return index ** 2

X = Indexer()
print(X[2])
```

2、拦截分片 Intercepting Slices
- 除了索引，`__getitem__` 也会被分片表达式调用：
- 以下为分片相关，详见第7章：

``` python
L = [5, 6, 7, 8, 9]
print(L[2:4], L[1:], L[:-1], L[::2])
```

- 也可以手动地传入分片对象：

``` python
print(L[slice(2, 4)], L[slice(1, None)], L[slice(None, -1)], L[slice(None, None, 2)])
```

- 而 `__getitem__` 即能被基础索引（带有一个索引）调用，又能被分片（带有一个分片对象）调用：

``` python
class Indexer:
    data = [5, 6, 7, 8, 9]
    def __getitem__(self, index):
        print('getitem:', index)
        return self.data[index]

X = Indexer()
X[0]
X[1]
X[-1]
X[2:4]
X[1:]
X[:-1]
X[::2]
```

- `__getitem__` 可以检测它接收的参数类型，并提取分片对象的边界。分片对象拥有 start 、 stop 和 step 这些属性，任意一项被省略的话都是 None ：

``` python
class Indexer:
    def __getitem__(self, index):
        if isinstance(index, int):
            print('indexing', index)
        else:
            print('slicing', index.start, index.stop, index.step)

X = Indexer()
X[99]
X[1:99:2]
X[1:]
```

3、`__setitem__` 方法是对索引进行赋值，可以拦截索引赋值或分片赋值

``` python
class IndexSetter:
    data = [5, 6, 7, 8, 9]
    def __setitem__(self, index, value):
        self.data[index] = value ** 2

X = IndexSetter()
X[3] = 4
print(X.data)
```

4、`__index__` 方法不是索引
- `__index__` 方法会为一个实例返回一个整数值，供转化数字串的内置函数使用：

``` python
class C:
    def __index__(self):
        return 255

X = C()
print(hex(X))
print(bin(X))
print(oct(X))
```

- 也可以作为索引：

``` python
print(('C' * 256)[255])
print(('C' * 256)[X])
print(('C' * 256)[X:])
```

5、索引迭代：`__getitem__`
- `__getitem__` 也可以是 Python 中一种重载迭代的方式；
- 可以重复对序列进行索引运算，直到检测到超出边界的 IndexError 异常。

``` python
class StepperIndex:
    def __getitem__(self, i):
        return self.data[i]
    
X = StepperIndex()
X.data = "Spam"

print(X[1])  # Indexing calls __getitem__

for item in X:  # for loops call __getitem__
    print(item, end=' ')
```

- 成员关系测试 in 、列表推导、内置函数 map 、列表和元组赋值运算以及类型构造方法都会自动调用 `__getitem__` ：

``` python
print('p' in X)
print([c for c in X])
print(list(map(str.upper, X)))
(a, b, c, d) = X
print(a, c, d)
print(list(X), tuple(X), ''.join(X))
print(X)
```

### 三、Iterable Objects: \_\_iter\_\_ and \_\_next\_\_
1、Python 中所有的迭代上下文都先尝试 `__iter__` 方法，再尝试 `__getitem__`

2、用类定义用户自定义可迭代对象
- 迭代上下文是通过将一个可迭代对象传入内置函数 iter ，并尝试调用 \_\_iter\_\_ 方法来实现的，该方法返回一个迭代器对象；
- 如果提供了这种方法，python 会重复调用这个迭代器对象的 \_\_next\_\_ 方法来产生元素，直到引发 StopIteration 异常。

``` python
class Squares:
    def __init__(self, start, stop):
        self.value = start - 1
        self.stop = stop
    def __iter__(self):
        return self # __iter__方法返回的迭代器对象就是实例对象self
    def __next__(self):
        if self.value == self.stop:
            raise StopIteration # 迭代通过raise语句来结束
        self.value += 1
        return self.value ** 2 # 改为生成平方

for i in Squares(1, 5):
    print(i, end=' ')

X = Squares(1, 5)
I = iter(X)
print(next(I))
print(next(I))
print(next(I))
print(next(I))
print(next(I))
```

3、单遍扫描和多遍扫描
- 类的 `__iter__` 被设计为只遍历一次，而不是多次；
- 类在代码中显式地选择扫描行为；
- 由于上面 Squares 类的 \_\_iter\_\_ 通常返回只带有一份迭代状态复制的 self ，因此是一个一次性的迭代；
- 一旦迭代过这个类的一个实例，它就变为空；
- 所以需要为每一次新的迭代创建一个新的可迭代实例对象。

``` python
X = Squares(1, 5)
print([n for n in X]) # Exhausts items: __iter__ returns self
print([n for n in X]) # Now it's empty: __iter__ returns same self
print([n for n in Squares(1, 5)]) # Make a new iterable object
print(list(Squares(1, 3))) # A new object for each new __iter__ call
```

4、单个对象上的多个迭代器
- 例如14章提到的：生成器函数、生成器表达式以及 map 和 zip 这样的内置函数，是单次迭代对象，只能支持唯一一个活跃扫描；
- range 内置函数和其他的内置类型（如列表），则支持多个带有独立位置的活跃迭代器。

（1）单次遍历迭代对象

``` python
S = map(list, 'ace') # 得到的是['a'],['b'],['c']的迭代器
for x in S:
    for y in S:
        print(x + y, end=' ')
print('-')
```

（2）多次遍历迭代对象

``` python
S = list('ace') # 得到的是['a','b','c']
for x in S:
    for y in S:
        print(x + y, end=' ')
print('-')
```

- 当用类编写用户定义的可迭代对象，要达到多个迭代器的效果，\_\_iter\_\_ 只需替迭代器定义一个新的对象状态，而不是在每次迭代器请求中都返回 self ；
- 下面的 SkipObject 类定义了一个可迭代对象。由于它的迭代器对象会在每次迭代时都被一个支持类重新创建，因此能够支持多个循环：

``` python
class SkipObject:
    def __init__(self, wrapped):
        self.wrapped = wrapped
    def __iter__(self):
        return SkipIterator(self.wrapped) # New iterator each time

class SkipIterator:
    def __init__(self, wrapped):
        self.wrapped = wrapped # Iterator state information
        self.offset = 0
    def __next__(self):
        if self.offset >= len(self.wrapped):
            raise StopIteration
        else:
            item = self.wrapped[self.offset]
            self.offset += 2
            return item

alpha = 'abcdef'
skipper = SkipObject(alpha)
I = iter(skipper)
print(next(I), next(I), next(I))

for x in skipper:
    for y in skipper:
        print(x + y, end=' ')

print('-')
```

5、编程备选方案：`__iter__` 加 yield
- 包含 yield 语句的函数都会被转换成一个生成器函数；
- 当被调用时，它返回一个新的生成器对象；
- 一个被自动创建的 \_\_iter\_\_ 方法返回它本身；
- 而另一个自动创建的 \_\_next\_\_ 方法要么开始函数的执行，要么回到上一次执行的位置：

``` python
def gen(x):
    for i in range(x): yield i ** 2

G = gen(5)
print(G.__iter__() == G)
I = iter(G)
print(next(I), next(I))
```

- 这个带有 yield 的生成器函数可以作为类的 \_\_iter\_\_ 重载方法；
- 这样的方法会放回带有 \_\_next\_\_方法的新生成器对象；
- 在类中作为方法的生成器函数有权访问存储在实例属性和局部作用域变量中的状态：

``` python
class Squares:
    def __init__(self, start, stop):
        self.start = start
        self.stop = stop
    def __iter__(self):
        for value in range(self.start, self.stop + 1):
            yield value ** 2

for i in Squares(1, 5): print(i, end=' ')
print('-')
```

- \_\_iter\_\_ 返回了一个生成器对象，该生成器对象带有一个自动创建的 \_\_next\_\_ 类，如果调用结果对象的 next 接口，就能按需产生结果：

``` python
S = Squares(1, 5)
print(S)
I = iter(S)
print(I)
print(next(I), next(I))
```

- 除了将生成器函数作为 \_\_iter\_\_ 方法，可以手动访问属性和调用，如`Squares(1,5).gen()`

``` python
class Squares:
    def __init__(self, start, stop):
        self.start = start
        self.stop = stop
    def gen(self):
        for value in range(self.start, self.stop + 1):
            yield value ** 2

for i in Squares(1, 5).gen(): print(i, end=' ')
print('-')
```

①当有 \_\_iter\_\_ 时，迭代触发 \_\_iter\_\_ 并返回一个新的带有 \_\_next\_\_ 的生成器；
②当没有 \_\_iter\_\_ 时，代码会调用一个生成器，自动创建 \_\_iter\_\_ 方法返回它本身。

6、使用 yield 的多重迭代器
- 之前的 \_\_iter\_\_ 加 yield 组合自动支持多重活跃迭代器；
- 因为每次对\_\_iter\_\_的调用都是返回一个新的生成器：

``` python
class Squares:
    def __init__(self, start, stop):
        self.start = start
        self.stop = stop
    def __iter__(self):
        for value in range(self.start, self.stop + 1):
            yield value ** 2

S = Squares(1, 5)
I = iter(S)
print(next(I), next(I))
J = Squares(1, 5)
J = iter(S)
print(next(J), next(J))

S = Squares(1, 3)
for i in S:
    for j in S:
        print('%s:%s' % (i, j), end=' ')
print('-')
```

### 四、Membership: \_\_contains\_\_, \_\_iter\_\_, and \_\_getitem\_\_
1、in 成员关系
- 类通常把 in 成员关系运算符实现为一个迭代，使用 `__iter__` 方法或 `__getitem__` 方法；
- 类还可以通过编写 `__contain__` 方法来支持更加特定具体的成员关系。
- 即当 `__contain__` 方法存在时，它优先于 `__iter__` 方法，而 `__iter__` 方法优先于 `__getitem__` 方法；
- `__contain__` 方法应该把成员关系定义为对一个键值做映射，或定义为对序列的搜索。

``` python
class Iters:
    def __init__(self, value):
        self.data = value

    def __getitem__(self, i):
        print('get[%s]:' % i, end='')
        return self.data[i]
    
    def __iter__(self):
        print('iter=> ', end='')
        self.ix = 0
        return self
    
    def __next__(self):
        print('next:', end='')
        if self.ix == len(self.data): raise StopIteration
        item = self.data[self.ix]
        self.ix += 1
        return item
    
    def __contains__(self, x):
        print('contains: ', end='')
        return x in self.data

X = Iters([1, 2, 3, 4, 5])
print(3 in X)

for i in X: 
    print(i, end=' | ')
print()
print([i ** 2 for i in X])
print( list(map(bin, X)) )

I = iter(X)
while True:
    try:
        print(next(I), end=' @ ')
    except StopIteration:
        break

print('\n' + '-' * 128)
```

2、上述代码去掉 `__contains__` 方法后，成员关系由 `__iter__` 执行

``` python
class Iters:
    def __init__(self, value):
        self.data = value

    def __getitem__(self, i):
        print('get[%s]:' % i, end='')
        return self.data[i]
    
    def __iter__(self):
        print('iter=> ', end='')
        self.ix = 0
        return self
    
    def __next__(self):
        print('next:', end='')
        if self.ix == len(self.data): raise StopIteration
        item = self.data[self.ix]
        self.ix += 1
        return item

X = Iters([1, 2, 3, 4, 5])
print(3 in X)

for i in X: 
    print(i, end=' | ')
print()
print([i ** 2 for i in X])
print( list(map(bin, X)) )

I = iter(X)
while True:
    try:
        print(next(I), end=' @ ')
    except StopIteration:
        break

print('\n' + '-' * 128)
```

3、上述代码去掉 `__contains__` 和 `__iter__` 方法后，成员关系和其他迭代上下文，由 `__getitem__` 方法调用

``` python
class Iters:
    def __init__(self, value):
        self.data = value

    def __getitem__(self, i):
        print('get[%s]:' % i, end='')
        return self.data[i]

X = Iters([1, 2, 3, 4, 5])
print(3 in X)

for i in X: 
    print(i, end=' | ')
print()
print([i ** 2 for i in X])
print( list(map(bin, X)) )

I = iter(X)
while True:
    try:
        print(next(I), end=' @ ')
    except StopIteration:
        break

print('\n' + '-' * 128)
```

- `__getitem__` 方法更加通用，除了迭代，还拦截显式索引和分片：

``` python
X = Iters('spam')
print(X[0])

print(X[1:])
print(X[:-1])

print(list(X))
```

### 五、Attribute Access: \_\_getattr\_\_ and \_\_setattr\_\_
1、类在需要的时候也可以拦截基本的属性访问（点号运算），即 `object.contribute` 

2、属性引用
- `__getattr__ `方法可以用来拦截属性引用；
- 每当用一个未定义的（不存在的）属性名称字符串对一个实例对象做点号运算时，它就会被调用；
- 正因为如此，`__getattr__` 可以用作以泛化形式响应属性请求的钩子；
- 它通常用于将代理控制对象的调用委托给内嵌（被包装）的对象；
- 这个方法也可以用于让类适配一个接口，或是为数据属性添加一个访问方法。

``` python
class Empty:
    def __getattr__(self, attrname):
        if attrname == 'age':
            return 40
        else:
            raise AttributeError(attrname)

X = Empty()
print(X.age)
```

3、属性赋值和删除
- `__setattr__` 会拦截所有的属性赋值；
- 如果定义或继承了这个方法，`self.attr = value` 会变成 `self.__setattr__('attr', value)`
- 注意事项：如果在 `__setattr__` 中对任何 self 属性做赋值，都将再次调用 `__setattr__` ，会导致无限递归循环（最终结果是相对快速的栈溢出异常）；
- 记住：`__setattr__` 会捕获所有的属性赋值；
- 如果你想使用该方法，可以把实例属性的赋值编写成对属性字典键的赋值来避免循环；
- 即，使用 `self.__dict__['name'] = x` ，而不是 `self.name = x` ，来避免循环：

``` python
class Accesscontrol:
    def __setattr__(self, attr, value):
        if attr == 'age':
            self.__dict__[attr] = value + 10
        else:
            raise AttributeError(attr + ' not allowed')

X = Accesscontrol()
X.age = 40
print(X.age)
```

- 第三个属性管理方法 `__delattr__` 被传入属性名称字符串并在所有属性删除操作被调用；
- 它也必须通过 `__dict__` 来进行属性删除操作，从而避免循环调用。

4、其他属性管理工具
- `__getattribute__` 方法拦截所有的属性访问，不只是未定义的；
- property 内置函数允许把方法对指定类属性上的访问和修改操作关联起来；
- 描述符 Descriptors 提供了一个协议，把一个类的 `__get__` 和 `__set__` 方法对指定类属性上的访问关联起来；
- slot 属性在类中被声明，但在每个实例中都会创建隐式的存储。

### 六、String Representation: \_\_repr\_\_ and \_\_str\_\_
1、`__repr__` 和 `__str__`
- `__str__` 会首先被打印操作和 str 内置函数尝试；
- `__repr__` 用于交互式命令行、 repr 函数、嵌套的显示，以及没有 \_\_str\_\_ 时的 print 和 str 调用
- `__str__` 用于程序的用户友好的显示，而 `__repr__` 通常用于代码或底层的显示：

``` python
class adder:
    def __init__(self, value=0):
        self.data = value
    def __add__(self, other):
        self.data += other

class addrepr(adder):
    def __repr__(self):
        return 'addrepr(%s)' % self.data

x = addrepr(2)
x + 1
```

- 在交互式命令行中，x 显示 addrepr(3) ，与 print(x) 一样：

``` python
print(x) 

class addstr(adder):
    def __str__(self):
        return '[Value: %s]' % self.data

x = addstr(3)
x + 1
```

- 在交互式命令行中， x 显示 `<__main__.addstr object at 0x00000000029738D0>`，与 print(x) 不一样：

``` python
print(x)

class addboth(adder):
    def __str__(self):
        return '[Value: %s]' % self.data
    def __repr__(self):
        return 'addboth(%s)' % self.data

x = addboth(4)
x + 1
```

- 在交互式命令行中， x 显示 addboth(5) ，运行 `__repr__` 方法：

``` python
print(x) # 运行__str__方法 
print(str(x), repr(x))
```

2、使用提示
- `__str__` 和 `__repr__` 都必须返回字符串；
- `__str__` 只会应用在对象出现在打印操作顶层时；在对象中内嵌的对象仍然使用 `__repr__` 方法打印
- 我的理解是 `__repr__` 表示对象本身显示什么，`__str__` 表示对象打印出什么。
- 打印容器对象的 `__str__` 方法：

``` python
class Printer:
    def __init__(self, val):
        self.val = val
    def __str__(self):
        return str(self.val)

objs = [Printer(2), Printer(3)]
for x in objs: print(x)
print(objs)
```

- 打印容器对象的 `__repr__` 方法：

``` python
class Printer:
    def __init__(self, val):
        self.val = val
    def __repr__(self):
        return str(self.val)

objs = [Printer(2), Printer(3)]
for x in objs: print(x)
print(objs)
```

### 七、Right-Side and In-Place Uses: \_\_radd\_\_ and \_\_iadd\_\_
1、右侧加法
- 目前编写的 `__add__` 方法并不支持把实例对象写在 + 右侧；
- 为了实现更通用的表达式，需要同时编写 `__radd__` 方法；
- 只有当 + 右侧是实例对象且左侧不是实例对象时，python 才会调用 `__radd__`：

``` python
class Commuter1:
    def __init__(self, val):
        self.val = val
    def __add__(self, other):
        print('add', self.val, other)
        return self.val + other
    def __radd__(self, other): # 顺序反转：self在 + 的右侧，other在左侧
        print('radd', self.val, other)
        return other + self.val

x = Commuter1(88)
y = Commuter1(99)

print(x + 1)
print(1 + y)
print(x + y)
```

- 2个实例对象相加后，python首先运行 `__add__` ，在 `__add__` 内部的 return 中又有加号即 88 + y，这个 + 触发了 `__radd__` 方法；
- 所以上面 `__add__` 方法中，`return self.val + other` 改为 `return other + self.val` ，就会看到2次 add ；

2、在 `__radd__` 中使用 `__add__`
- 可以在 `__radd__` 中直接调用 `__add__` ；要么交换位置，间接触发 `__add__` ；要么在类的顶层直接把 `__radd__` 赋值成 `__add__` 的别名：

``` python
class Commuter2:
    def __init__(self, val):
        self.val = val
    def __add__(self, other):
        print('add', self.val, other)
        return self.val + other
    def __radd__(self, other):
        return self.__add__(other)

class Commuter3:
    def __init__(self, val):
        self.val = val
    def __add__(self, other):
        print('add', self.val, other)
        return self.val + other
    def __radd__(self, other):
        return self + other

class Commuter4:
    def __init__(self, val):
        self.val = val
    def __add__(self, other):
        print('add', self.val, other)
        return self.val + other
    __radd__ = __add__
```

 3、return 类类型，需要 isinstance 测试来避免嵌套

``` python
class Commuter5:
    def __init__(self, val):
        self.val = val
    def __add__(self, other):
        if isinstance(other, Commuter5): # 如果不测试，会得到嵌套在Commuter5中的Commuter5对象
            other = other.val
        return Commuter5(self.val + other)
    def __radd__(self, other):
        return Commuter5(other + self.val)
    def __str__(self):
        return '<Commuter5: %s>' % self.val

x = Commuter5(88)
y = Commuter5(99)
print(x + 10)
print(10 + y)
print(x + y)
print(x + y + 10)
```

4、原位置加法 In-Place Addition
- 为了实现 += ，可以编写一个 `__iadd__` 或 `__add__`；
- `__iadd__` 更高效，若不存在，则会使用 `__add__`：

``` python
class Number:
    def __init__(self, val):
        self.val = val
    def __iadd__(self, other):
        self.val += other
        return self

x = Number(5)
x += 1
x += 1
print(x.val)

y = Number([1])
y += [2]
y += [3]
print(y.val)
```

5、每个运算符都有右侧和原位置重置方法，例如乘法的 `__mul__` 、 `__rmul__` 、 `__imul__`。

### 八、Other Operator Overloading
1、Call Expressions: `__call__`
- 调用实例会使用 `__call__` 方法：

``` python
class Callee:
    def __call__(self, *pargs, **kargs):
        print('Called:', pargs, kargs)

C = Callee()
C(1, 2, 3)
C(1, 2, 3, x=4, y=5)
```

2、Comparisons: `__lt__` , `__gt__` , and Others
- 类可以定义方法来捕获<、>、<=、>=、==、!=
- 比较运算符没有隐含关系，比如 == 为真并不意味着 != 为假。因此 `__eq__` 和 `__ne__` 的定义要确保两个运算符都正确地工作；
- 以下为简单介绍：

``` python
class C:
    data = 'spam'
    def __gt__(self, other):
        return self.data > other
    def __lt__(self, other):
        return self.data < other

X = C()
print(X > 'ham') # runs __gt__
print(X < 'ham') # runs __lt__
```

3、Boolean Tests: `__bool__` and `__len__`
- Python会首先尝试 `__bool__` 来获取一个直接的布尔值；如果没有该方法，则尝试 `__len__` 来根据对象长度确定真值：

``` python
class Truth:
    def __bool__(self): return True

X = Truth()
if X: print('yes!')

class Truth:
    def __bool__(self): return False

X = Truth()
print(bool(X))
```

- 如果没有上述方法，python 会使用长度，非零长度为真，零长度为假：

``` python
class Truth:
    def __len__(self): return 0

X = Truth()
if not X: print('no!')
```

- 如果上述2者都没有定义，对象会被看作真：

``` python
class Truth:
    pass

X = Truth()
print(bool(X))
```

4、Object Destruction: `__del__` 对象析构函数
- 每当实例空间被收回时，`__del__`（析构函数方法）会自动执行：

``` python
class Life:
    def __init__(self, name='unknown'):
        print('Hello ' + name)
        self.name = name
    def live(self):
        print(self.name)
    def __del__(self):
        print('Goodbye ' + self.name)

brian = Life('Brian')
brian.live()
brian = 'loretta'
```

- 当 brain 被赋值为一个字符串时，我们会失去 Life 实例的最后一个引用并因此触发其析构函数；
- python 在回收实例时，会自动回收该实例拥有的内存空间，所以析构函数并不需要考虑空间管理。


## chapter 31 Designing with Classes
### 一、Python and OOP
该章内容是一些常用的 OOP 的设计模式，本书在这一块只是抛砖引玉，大致了解一下就行，按需学习，需要深入了解的时候再去深入了解。  

大致包括*继承*、*组合*、*委托*和*工厂*，以及*伪私有属性*、*多重继承*和*绑定方法*

1、OOP and Inheritance

``` python
class Employee:
    def __init__(self, name, salary=0):
        self.name = name
        self.salary = salary
    def giveRaise(self, percent):
        self.salary = self.salary + (self.salary * percent)
    def work(self):
        print(self.name, "does stuff")
    def __repr__(self):
        return "<Employee: name=%s, salary=%s>" % (self.name, self.salary)

class Chef(Employee):
    def __init__(self, name):
        Employee.__init__(self, name, 50000)
    def work(self):
        print(self.name, "makes food")

class Server(Employee):
    def __init__(self, name):
        Employee.__init__(self, name, 40000)
    def work(self):
        print(self.name, "interfaces with customer")

class PizzaRobot(Chef):
    def __init__(self, name):
        Chef.__init__(self, name)
    def work(self):
        print(self.name, "makes pizza")

bob = PizzaRobot('bob')
print(bob)
bob.work()
bob.giveRaise(0.20)
print(bob)

for klass in Employee, Chef, Server, PizzaRobot:
    obj = klass(klass.__name__)
    obj.work()

print()
```

2、OOP and Composition 组合
- 组合涉及把其他对象嵌入容器对象内，促使其实现容器方法；
- 有些书称组合为聚合 aggregation ：

``` python
class Customer:
    def __init__(self, name):
        self.name = name
    def order(self, server):
        print(self.name, "orders from", server)
    def pay(self, server):
        print(self.name, "pays for item to", server)

class Oven:
    def bake(self):
        print("oven bakes")

class PizzaShop:
    def __init__(self):
        self.server = Server('Pat')
        self.chef = PizzaRobot('Bob')
        self.oven = Oven()

    def order(self, name):
        customer = Customer(name)
        customer.order(self.server)
        self.chef.work()
        self.oven.bake()
        customer.pay(self.server)

scene = PizzaShop()
scene.order('Homer')
print()
scene.order('Shaggy')
print()
```

- 以上的 PizzaShop 类就是容器和控制器；
- 每份订单都会创建新的 Customer 对象，并把内嵌的 Server 对象传给 Customer 的方法。

3、OOP and Delegation 委托：包装器代理对象 “Wrapper” Proxy Objects
- 委托通常是指控制器对象内嵌其他对象，并把操作请求传递给那些内嵌的对象；
- 控制器负责管理类活动，例如记录日志和验证访问，为接口组件添加额外步骤，或监视活跃实例；
- 委托是组合的一种特殊形式，它使用包装器（代理）类管理单一的内嵌对象，而包装器类保留了内嵌对象的大多数或全部的接口。
- 通常使用 `__getattr__` 方法钩子来实现委托，来把任意的访问转发给被包装的对象：

``` python
class Wrapper:
    def __init__(self, object):
        self.wrapped = object
    def __getattr__(self, attrname):
        print('Trace: ' + attrname)
        return getattr(self.wrapped, attrname)
```

- `__getattr__` 方法会获取属性名称的字符串，`getattr` 内置函数可通过名称字符串获取被包装对象的属性：`getattr(X,N)` 就是 `X.N`
- 这里 Wrapper 类只是在每次属性访问时打印跟踪消息，并把属性请求委托给内嵌的 wrapped 对象：

``` python
x = Wrapper([1, 2, 3])
x.append(4)
print(x.wrapped)
x = Wrapper({'a': 1, 'b': 2})
print(list(x.keys()))
```

- 总的效果就是通过 Wrapper 类内的额外代码来扩充被包装对象的全部接口；
- 可以利用这种方法记录方法调用，把方法调用路由到额外或定制的逻辑，使类适应一个新接口。

### 二、Pseudoprivate Class Attributes
1、类的伪私有属性 Pseudoprivate Class Attributes
- python 支持名称重整 mangling ，使类中的某些名称局部化；
- 重整后的名称会被误以为是私有属性，但名称重整并不能阻止来自类外部代码的访问；
- 这个功能主要是为了避免实例内的命名空间冲突，而不是限制名称的访问，所以重整后的变量名称为伪私有。

2、名称重整的工作方式
- 只在 class 内部，任意开头双下划线，结尾没双下划线的名称，会自动在前面包含外围类的名称从而进行扩展；
- 比如 Spam 类中的 \_\_X 会自动变成 \_Spam\_\_X ，这样就不会和其他类中相同的变量名相冲突；
- 实例属性引用也需要使用 `self._Spam__X`：

``` python
class C1:
    def meth1(self): self.__X = 88
    def meth2(self): print(self.__X)
class C2:
    def metha(self): self.__X = 99
    def methb(self): print(self.__X)

class C3(C1, C2): pass
I = C3()

I.meth1(); I.metha()
print(I.__dict__)
I.meth2(); I.methb()
```

- 这样可避免实例中潜在的名称冲突，如下：

``` python
class C1:
    def meth1(self): self.X = 88
    def meth2(self): print(self.X)
class C2:
    def metha(self): self.X = 99
    def methb(self): print(self.X)

class C3(C1, C2): pass
I = C3()
I.meth1(); I.metha()
print(I.__dict__)
```

### 三、Methods Are Objects: Bound or Unbound
1、类的方法可以通过实例或类来访问
①未绑定 Unbound 的类方法对象（无 self ）：直接对类进行点号运算从而获取类的函数属性。
②绑定 Bound 的实例方法对象（ self +函数）：对实例进行点号运算从而获取类的函数属性。

``` python
class Spam:
    def doit(self, message):
        print(message)

object1 = Spam()
x = object1.doit # Bound method object: instance+function
x('hello world')
```

- 如果对类进行点号运算来获取 doit ，就会得到未绑定的方法对象；
- 要调用该方法，需要传入实例作为最左侧的参数。

``` python
t = Spam.doit # Unbound method object
object1 = Spam()
t(object1, 'howdy')
```

- 同理可以在类的方法内引用 self 的属性，而该属性指向类中另外的方法：

``` python
class Eggs:
    def m1(self, n):
        print(n)
    def m2(self):
        x = self.m1
        x(42)

Eggs().m2()
```

2、在Python 3.X 中，未绑定方法是函数
- 打印一个非实例类的方法的类型，在 python 2.X 显示为未绑定方法，在 python 3.X 显示为函数：

``` python
class Selfless:
    def __init__(self, data):
        self.data = data
    def selfless(arg1, arg2):
        return arg1 + arg2
    def normal(self, arg1, arg2):
        return self.data + arg1 + arg2

X = Selfless(2)
print(X.normal(3, 4))
print(Selfless.normal(X, 3, 4))
print(Selfless.selfless(3, 4)) # No instance: works in 3.X, fails in 2.X!
X.selfless(3, 4) # 会出现TypeError: selfless() takes 2 positional arguments but 3 were given
```

3、绑定方法和其他可调用对象
- 绑定方法可以作为一般对象处理，比如用列表存储绑定方法对象：

``` python
class Number:
    def __init__(self, base):
        self.base = base
    def double(self):
        return self.base * 2
    def triple(self):
        return self.base * 3
    
x = Number(2)
y = Number(3)
z = Number(4)
acts = [x.double, y.double, y.triple, z.double]
for act in acts:
    print(act())
```

- 绑定函数对象也有自己的内省信息，包括一些属性能够让其访问配对的实例对象和方法函数：

``` python
bound = x.double
print(bound.__self__, bound.__func__)
print(bound.__self__.base)
```

4、绑定方法只是可调用对象的一种，以下都可以用类似方式处理

``` python
def square(arg):
    return arg ** 2

class Sum:
    def __init__(self, val):
        self.val = val
    def __call__(self, arg):
        return self.val + arg

class Product:
    def __init__(self, val):
        self.val = val
    def method(self, arg):
        return self.val * arg

sobject = Sum(2)
pobject = Product(3)
actions = [square, sobject, pobject.method]

for act in actions:
    print(act(5))

print(actions[-1](5))
print([act(5) for act in actions])
print(list(map(lambda act: act(5), actions)))
```

### 四、Classes Are Objects: Generic Object Factories
1、The factory design pattern
- 工厂：将任意可调用对象，比如类，传递给生成其他种类对象的函数。

``` python
def factory(aClass, *pargs, **kargs):
    return aClass(*pargs, **kargs)

class Spam:
    def doit(self, message):
        print(message)

class Person:
    def __init__(self, name, job=None):
        self.name = name
        self.job = job

object1 = factory(Spam)
object2 = factory(Person, "Arthur", "King")
object3 = factory(Person, name='Brian')

object1.doit(99)
print(object2.name, object2.job)
print(object3.name, object3.job)
```

- 工厂允许代码与动态配置的对象构造细节相隔绝。

### 五、Multiple Inheritance: “Mix-in” Classes
1、多继承的缺点就是当相同的方法名称在不止一个父类中定义时，就会造成冲突
- 搜索属性时，python 会从左到右遍历搜索类首行的父类
    - 在经典类 classic classes 中，所有情形下的属性搜索始终实行**深度优先 depth-first** 搜索，直到继承树的顶端，然后从左至右进行，这顺序称为 **DFLR（depth-first, left-to-right path）**
    - 在新式类 new-style classes 中，属性搜索通常是一样的，但在**钻石模式 diamond patterns** 下以**广度优先**方式进行，搜索向上移动之前沿着继承树的同一级搜索，这顺序称为新式 **MRO（method resolution order）**
- 当继承树中的多个类共享一个共同父类时，钻石模式就会出现；
- 新式搜索顺序旨在访问完全部子类后，仅访问一次这样的共享父类；
- 当冲突产生时，而不愿意使用继承的第一个名称时，会是个问题，详见下一章的新式类、MRO和新式工具。

2、编写 mix-in 类
- 多继承的最常见方法就是 to “mix in” general-purpose methods from superclasses
- 就是编写一个通用的类，然后通过多继承去使用它，跟模块作用类似；
- 下面的代码定义了一个名为 ListInstance 的 mix-in 类，它为继承了它的所有类都重载了 `__str__` 方法：

``` python
class ListInstance:
    def __attrnames(self):
        result = ''
        for attr in sorted(self.__dict__):
            result += '\t%s=%s\n' % (attr, self.__dict__[attr])
        return result
    
    def __str__(self):
        return '<Instance of %s, address %s:\n%s>' % (
            self.__class__.__name__,
            id(self),
            self.__attrnames())
```

- 每个实例都有一个内置的 `__class__` 属性，引用创建本实例的类；每个类都有一个 `__name__` 属性，引用类头部的名称；
- id 内置函数显示实例的内存地址。

3、单继承模型下，混合上述的类

``` python
class Spam(ListInstance):
    def __init__(self):
        self.data1 = 'food'

x = Spam()
print(x)
```

4、多继承：

``` python
class Super:
    def __init__(self):
        self.data1 = 'spam'
    def ham(self):
        pass

class Sub(Super, ListInstance):
    def __init__(self):
        Super.__init__(self)
        self.data2 = 'eggs'
        self.data3 = 42
    def spam(self):
        pass

X = Sub()
print(X)
```

## chapter 32 Advanced Class Topics
### 一、Extending Built-in Types
1、内嵌方式扩展类型

``` python
class Set:
    def __init__(self, value = []):
        self.data = []
        self.concat(value)

    def intersect(self, other):
        res = []
        for x in self.data:
            if x in other:
                res.append(x)
        return Set(res)
    
    def union(self, other):
        res = self.data[:]
        for x in other:
            if not x in res:
                res.append(x)
        return Set(res)
    
    def concat(self, value):
        for x in value:               # Removes duplicates
            if not x in self.data:
                self.data.append(x)

    def __len__(self): return len(self.data)
    def __getitem__(self, key): return self.data[key] 
    def __and__(self, other): return self.intersect(other)
    def __or__(self, other): return self.union(other)
    def __repr__(self): return 'Set:' + repr(self.data)
    def __iter__(self): return iter(self.data) # 生成迭代操作
```

- 通过索引和迭代操作，能让上述定义的 Set 类充当真正的列表：

``` python
x = Set([1, 3, 5, 7])
print(x.union(Set([1, 4, 7])))
print(x | Set([1, 4, 6]))
```

2、通过子类扩展类型
- 在 python 中， list ,  str ,  dict , 和 tuple 这样的类型转换函数实际上是调用了类型的对象构造函数；
- 因此可以通过建立类型名称的子类来定制或扩展内置类型的行为，比如把列表的偏移量从1开始算起而不是0：

``` python
class MyList(list):
    def __getitem__(self, offset):
        print('(indexing %s at %s)' % (self, offset))
        return list.__getitem__(self, offset - 1)

print(list('abc'))
x = MyList('abc')  # __init__ inherited from list
print(x)  # __repr__ inherited from list

print(x[1])
print(x[3])

x.append('spam'); print(x) # # Attributes from list superclass
x.reverse(); print(x)
```

- 上述的编码方式可以提供编写第一个例子的 Set 的另一种方式，即作为内置 list 的子类：

``` python
class Set(list):
    def __init__(self, value = []):
        list.__init__([]) # 继承父类
        self.concat(value)
    
    def intersect(self, other):
        res = []
        for x in self:
            if x in other:
                res.append(x)
        return Set(res)
    
    def union(self, other):
        res = Set(self)
        res.concat(other)
        return res
    
    def concat(self, value):
        for x in value:
            if not x in self:
                self.append(x)
    
    def __and__(self, other): return self.intersect(other)
    def __or__(self, other): return self.union(other)
    def __repr__(self): return 'Set:' + list.__repr__(self)

x = Set([1,3,5,7])
y = Set([2,1,4,5,6])
print(x, y, len(x))
print(x.intersect(y), y.union(x))
print(x & y, x | y)
x.reverse(); print(x)
```

### 二、The “New Style” Class Model
1、介绍
- 本书之前谈到的类和新式类相比，称为经典类，但在 python 3.X 中类的区分已经融合了；
- 在 python 3.X 中，所有的类都是所谓的新式类，不管是否显式地继承自 object ；
- 所有的类都继承自 object ，不管是显式的还是隐式的，所有的类都隐含是 object 的子类；
- 所以在 python 3.X 中新式类的功能成为了常规的类功能。
- 新式类要么从内置类型（如list）派生，要么从一个叫做 object 的特殊内置类派生。

2、新式类的变化
- ①针对内置属性的获取：跳过实例
    - `__getattr__` 和 `__getattribute__` 通用属性拦截方法仍然通过显式名称访问属性，但不再适用于那些被内置运算隐式获取的属性。
- ②类和类型的合并：类型检验
    - **类就是类型，类型就是类**； `type(instance)` 内置函数返回一个实例对应的类，与 `instance.__class__` 是相同的。
- ③ object 自动根类：默认情况
    - 所有的新式类继承自 object 类。该类在 3.X 中被自动添加为用户定义类继承树的根（最顶级夫）类，并且不需要被显式地指定为父类。
- ④继承搜索顺序：MRO与钻石模式
    - 多继承的钻石模式的搜索顺序更像广度优先搜索，先横向搜索再纵向搜索。这种属性搜索顺序称为 **MRO** ，可以用新式类中的 `__mro__` 属性来跟踪。
- ⑤继承算法：第40章
    - 新式类所使用的继承算法比经典类的深度优先模式更加复杂，包括了描述符、元类和内置函数的特殊情况。
- ⑥新的高级工具：代码的影响
- 新式类有一组新的类工具，包括 **slot** 、 **property** 、**描述符**、 **super** 和 `__getattribute__` 方法

3、内置属性的获取将跳过实例
- 即在新式类中，通用实例属性拦截方法 `__getattr__` 和 `__getattribute__` 不能再拦截下以 `__X__` 命名的运算符重载方法名的调用；
- 也就是说对 `__X__` 这一类名称的搜索是从类开始，而非从实例开始；
- 为什么引入搜索改变：它反映了一个由元类模型引入的难题。类现在是元类 metaclass 的实例，又因为元类可以定义内置运算符方法来处理它们生成的类；
- 所以在类上运行的方法调用必须跳过类本身，并在更高层次选择处理该类的方法，而不是选取类本身的版本；
- 类本身的版本可能会导致非绑定方法调用，因为类自身方法会处理低一层次的实例；
- 结果是类本身即是类型又是实例，因此实例在内置运算方法搜索的时候都被跳过了；
- 但非内置名称和内置名称的直接显式调用仍旧会检测实例：

``` python
class C(object): pass # object可有可无
X = C()

X.normal = lambda: 99 # 在方法外修改实例属性
print(X.normal())

X.__add__ = lambda y: y + 88
print(X.__add__(1)) # 内置方法的显式调用
```

- `print(X + 1)` # 3.X 会出现 TypeError ，因为不会搜索实例的内置方法，在 2.X 得到结果89

4、类型模型改变
- **类即类型** Classes are types；Types are classes
- 类是由元类生成，元类要么是 type 自身，要么是由 type 定制来扩展或管理生成的类的一个子类；
- 内置的类型（比如列表、字符串）和用户定义的编写为类的类型之间没有真正的区别；
- 可以编写元类：使用 class 语句编写的用户定义 type 子类，控制作为它们的实例的类的创建：

``` python
class C(object): pass

I = C()
print(type(I), I.__class__) # 在2.X中，类实例的类型是instance，<type 'instance'>, I.__class__结果一样
print(type(C), C.__class__) # 在2.X中，type(C)结果为<type 'classobj'>，而C.__class__会出错，class C has no attribute '__class__'

print(type([1, 2, 3]), [1, 2, 3].__class__)
print(type(list), list.__class__)
```

- 元类，详见第40章：

> ① object 类是所有新式类的父类；  
> ② type 是所有类的类；  
> ③ object 类是由元类 type 创建的，但是 type 类又继承了 object 类， type 元类的类则是由 type 元类自身创建的；  
> ④所以任何元素都是对象（都是 type 的实例对象），一切都继承 object ，一切皆对象。

5、所有对象派生自 “object”
- 所有的类都隐式或显式地继承自 object 类，并且由于所有的类型都是类，所以每个对象都派生自 object 内置类；
- 一个类实例的类型就是产生它的类，一个类的类型就是 type 类；
- 实例和类都继承自内置的 object 类：

``` python
print(isinstance(X, object))
print(isinstance(C, object))
print(C.__bases__)
```

- 内置类型 built-in types 也是如此，内置类型也是类，它们的实例继承自 object ：

``` python
print(isinstance('spam', object))
print(isinstance(str, object))
print(str.__bases__)
```

- 实际上， type 自身继承自 object ，而且 object 继承自 type ，即使二者是不同对象：

``` python
print(type(type))
print(type(object))
print(isinstance(type, object))
print(isinstance(object, type))
print(type is object)
```

6、钻石继承改变
- 钻石模式指：有多于一个的父类指向同个更高级父类的树状模式（长得像钻石）；
- ① DFLR 搜索顺序：经典类
    - 深度优先 depth first ，然后从左到右 left to right （首字母：DFLR）：python 一路向上搜索，深入树的左侧，返回后才开始找右侧
- ② MRO 搜索顺序：新式类
    - 广度优先 breadth-first ，先搜索当前父类右侧的所有其他父类，再一路往上至顶端( MRO:method resolution order )
- 新式的 MRO 允许较底层的父类覆盖高层父类的属性：

``` python
class A(object): attr = 1
class B(A): pass
class C(A): attr = 2
class D(B, C): pass

x = D()
print(x.attr) # 在2.X中，结果为1，完整的DFLR搜索顺序为x、D, B, A, C, A
# 而完整的MRO搜索顺序为x、D、B、C、A
```

- 可以显式得解决冲突：

``` python
class D(B, C): attr = B.attr
x = D()
print(x.attr)
```

- object 父类为各种内置运算提供了默认方法，在没有 MRO 的搜索模式下，多继承中 object 的默认方法总是覆盖用户编写的类中的定制代码。

7、 `__mro__` 方法
- `class.__mro__` 方法可以跟踪新式类的默认继承方式，将返回类的 MRO 顺序；
- 一个给定类的 MRO 列表包括了类本身，它的父类，以及直到继承树顶端 object 的所有高级父类；
- 在这个列表中，每个类出现在它的父类之前，而且多个父类保持了它们在 `__bases__` 父类元组中出现的顺序
- super 函数调用可以使用 MRO 列表中的下一个类，而这个类不一定是一个父类。
- 注意 MRO 顺序只适用于钻石继承模式，如下：

``` python
class A: pass
class B(A): pass
class C(A): pass
class D(B, C): pass
print(D.__mro__)
```

- 对于非钻石继承模式，还是 DFLR ，如下：

``` python
class A: pass
class B(A): pass
class C: pass
class D(B, C): pass
print(D.__mro__)
```

- `class.mro()` 方法在每次类进行实例化时被调用，返回值是一个列表：

``` python
print(D.mro() == list(D.__mro__))
```

### 三、New-Style Class Extensions - Slots
1、Slots : 属性声明
- 通过将一系列的字符串属性名称赋值给特殊的 `__slots__` 类属性，可以让新式类限制实例会得到的属性集，又能优化内存和速度性能

2、slot 基础
- 只有 `__slots__` 列表内的名称可赋值为实例属性：

``` python
class limiter(object):
    __slots__ = ['age', 'name', 'job']

x = limiter()
x.age = 40
x.spam = 40 # 会出现错误：AttributeError: 'limiter' object has no attribute 'spam'
print(x.age)
```

- slots 最好只在有大量实例出现的、内存密集型应用的情况下使用。

3、slot 与命名空间字典
- slot 会使类模型复杂化；
- 有些带有 slot 的实例会没有 `__dict__` 属性命名空间字典（替换命名空间字典存储），有些可能会拥有这个字典不包含的数据属性（共存）；
- 这是新式类模型和传统类模型的不兼容性，导致通用访问属性的代码复杂化，甚至让程序失败：

``` python
class C:
    __slots__ = ['a', 'b']

X = C()
X.a = 1
print(X.a)
print(X.__dict__) # 会出现错误：AttributeError: 'C' object has no attribute '__dict__'
```

- 仍然可以使用 getattr 、 setattr （不仅查找 `__dict__` ，也会查找例如 slot 的类一级名称）和 dir（会收集整个类树上所有被继承的名称）；

``` python
print(getattr(X, 'a'))
setattr(X, 'b', 2)
print(X.b)
print('a' in dir(X))
print('b' in dir(X))
```

- 如果没有一个属性命名空间字典，不能给不在 slot 列表中的实例赋值新的名称：

``` python
class D:
    __slots__ = ['a', 'b']
    def __init__(self):
        self.d = 4 # 无法赋值，创建实例后会出错

X = D() # 会出现错误：AttributeError: 'D' object has no attribute 'd'
```

- 可以在 `__slots__` 中显式包含 `__dict__` 来创建一个属性命名空间字典：

``` python
class D:
    __slots__ = ['a', 'b', '__dict__']
    c = 3
    def __init__(self):
        self.d = 4

X = D()
print(X.d)
print(X.c)
X.a = 1
X.b = 2
print(X.__dict__)
print(X.__slots__)
```

4、父类中的多个 `__slots__` 列表
- slot 列表可能会不止一次出现在类树；
- 因为 slot 名称是类的一级属性（ class-level attributes ），实例按一般继承规则，获得了类树中其他位置的所有 slot 名称的并集：

``` python
class E:
    __slots__ = ['c', 'd']
class D(E):
    __slots__ = ['a', '__dict__']

X = D()
X.a = 1; X.b = 2; X.c = 3
print(X.a, X.c)
```

- 如果只检测被直接继承的 slot 列表，则不能获取在类树的更高层次定义的 slot ：

``` python
print(E.__slots__)
print(D.__slots__)
print(X.__slots__)
print(X.__dict__)
```

- 但是 dir 包含所有的 slot ：`print(dir(X))`

5、slot 使用规则
- slot 声明可以出现在一个类树中的多个类
    - 若父类没有 slot ，子类的 slot 就没有意义：如果子类继承自一个没有 `__slots__` 的父类，为父类创建的 `__dict__` 实例属性将总是可访问的。而避免 `__dict__` 是使用 slot 的主要原因；
    - 如果子类没有 slot ，父类的 slot 也没有意义：同上
    - 重新定义让父类的 slot 变得没有意义：如果一个类定义了父类中相同的 slot 名称，它的重新定义会根据一般继承规则隐藏父类中的 slot 。需要从父类获取描述符来访问父类的 slot ；
    - slot 会阻止类一级的默认名称：slot 被实现成类一级的描述符，不能在类一级中对同名类属性进行赋值。

### 四、New-Style Class Extensions - Properties
1、property ：属性访问器
- property 能自动调用（动态地）方法来访问或者赋值实例属性。功能与 Java 和 C# 语言的属性类似，但在 python 最好少用；
- property 和 slot 都属于类一级描述符 class-level attribute descriptors ，见第38章。

2、property 基础
- property 是一种被赋值给类属性名称的对象；
- 产生 property 的方式：调用内置函数 property ，同时传入三个访问器方法（分别用于处理获取、设置和删除操作），以及一个可选的 property 文档字符串；
- 如果任一参数以 None 传入，特性就不能支持对应操作；
- 最终得到的 property 对象，一般是在 class 顶层赋值给名称（ `name = property()` ），以及后面会说的“@”来自动化这一个步骤：

``` python
class properties(object):
    def getage(self):
        return 40
    age = property(getage, None, None, None) # (get, set, del, docs), or use @

x = properties()
print(x.age) # 对property名称的访问，会被自动路由到property调用的一个访问器方法
```

- 新增对属性赋值运算的支持：

``` python
class properties(object):
    def getage(self):
        return 40
    def setage(self, value):
        print('set age: %s' % value)
        self._age = value
    age = property(getage, setage, None, None)

x = properties()
print(x.age)
x.age = 42
print(x._age)
print(x.age)
x.job = 'trainer'
print(x.job)
```

- 用函数装饰器 function decorator 来编写 property 见第38章；
- `__getattribute__` and Descriptors 描述符也是类扩展，见第38章。

### 五、Static and Class Methods
