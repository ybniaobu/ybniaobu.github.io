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
