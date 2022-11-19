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

