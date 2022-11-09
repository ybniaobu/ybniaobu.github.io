---
title: 《Learning Python》读书笔记（五）
date: 2022-11-08 14:00:31
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
> 本篇主要内容为：XXXXXXXXXXXXXXXXXXXXXXXXXXXX

<div  align="center">  
<img src="https://s2.loli.net/2022/09/17/ri9Ue6nguJdq1Ca.jpg" width = "80%" height = "80%" alt="Learning Python"/>
</div>

# PART IV Functions and Generators
## chapter 16 Function Basics
### 一、Coding Functions
1. 函数相关的语句和表达式
    - `def`创建了一个函数对象并将其赋值给了某一变量名；
    - `lambda`创建一个函数对象并将其作为结果返回，见第19章；
    - `return`将一个结果对象传回给调用者，默认返回None；
    - `yield`向调用者发回一个结果对象，并挂起它们的状态；
    - `global`声明了一个模块级的可被赋值的变量；
    - `nonlocal`声明了一个需要被赋值的外层函数变量；
    - 参数通过赋值(对象引用)传递给函数，除非你显式指明形式参数与实际参数的对应，否则实际参数按位置赋值给形式参数。参数、返回值与变量不需要被声明。

2. def语句
    - 一般格式一下：
    ``` python
    def name(arg1, arg2,... argN):
        statements
    ```
    - def的头部定义了被赋值函数对象的函数名，圆括号parentheses中包含了**形式参数arguments (sometimes called parameters)**，简称**形参**；
    - 在函数调用时，括号内的传入对象将赋值给头部的形式参数；
    - Python的return语句将结束函数调用并把结果返回至函数调用处，return语句是可选的，一个没有返回值的函数自动返回None对象；
    - 可以将函数赋值给一个不同的变量名，并通过新的变量名进行调用：
    ``` python
    othername = func
    othername()
    ```
    - 函数也是对象，在程序运行时被明确地记录在内存中。除了调用以外，函数允许将任意属性附加到其中以供之后使用：
    ``` python
    def func(): ... 
    func() 
    func.attr = value
    ```

### 二、A First Example: Definitions and Calls
1. 定义Definition
``` python
def times(x, y):
    return x * y
```

2. 调用Calls
    - 参数是通过赋值传入的，函数头部的形式参数x被赋值为2，y被赋值为4：
    ``` python
    print(times(2, 4))
    print(times('Ni', 4))
    ```
    - *函数体（嵌套在函数定义语句中的代码）在函数被一个调用表达式调用时才会执行*；
    - 不调用是不执行函数体的，可以试试def一个有错误的函数，不调用它去运行。

3. Python中的**多态Polymorphism**
    - 如上所示，times函数中表达式x*y完全取决于x和y的对象类型，这种依赖类型的行为称为**多态Polymorphism**；
    - 函数可以自动地应用到所有类别的对象上。只要对象支持所预期的接口（也称为协议，即预期的方法和表达式运算符），函数就能处理它们；
    - 从宏观上来说，Python为对象接口object interfaces编程，不是为数据类型编程。


### 三、A Second Example: Intersecting Sequences
1. 定义
    - 将代码封装**package**（或者**wrap**）在函数中：
    ``` python
    def intersect(seq1, seq2):
        res = [] 
        for x in seq1: 
            if x in seq2: 
                res.append(x) 
        return res
    ```

2. 调用
    ``` python
    s1 = "SPAM"
    s2 = "SCAM"
    print(intersect(s1, s2))
    ```

3. 多态
    - 对于intersect函数，只要第一个参数支持for循环，第二个参数支持in成员测试，就能正常工作：
    ``` python
    x = intersect([1, 2, 3], (1, 4))
    print(x)
    ```

4. **局部变量Local Variables**
    - intersect函数中的res变量在Python中被称为局部变量，仅在函数运行时存在；
    - 在函数内部进行赋值的变量名都默认为**局部变量**；
    - res被赋值过，所以是局部变量；参数也是通过赋值被传入的，所以seq1和seq2也是局部变量；for循环中的变量x也是局部变量；
    - 所有的局部变量在函数调用时出现，在函数退出时消失。详见第17章。


## chapter 17 Scopes
### 一、Python Scope Basics
1. Python作用域基础
    - Python创建、改变或查找变量名都是在所谓的**命名空间namespace**中进行的，**作用域scope**就是命名空间；
    - 在代码中给一个变量赋值的地方决定了这个变量将存在于哪个命名空间；
    - 一个函数内赋值的所有变量名都与该函数的命名空间相关联；
    - 在def内赋值的变量名与在def外赋值的变量名不冲突，即使是相同的变量名。
    - 变量可以在3个不同的地方被赋值，分别对应3种不同的作用域：
        - If a variable is assigned inside a def, it is **local** to that function.
        - If a variable is assigned in an enclosing def, it is **nonlocal** to nested functions.
        - If a variable is assigned outside all defs, it is **global** to the entire file.
    - 我们将其称为语义作用域lexical scoping，因为变量的作用域由源代码的位置决定，而不是由函数调用决定：
    ``` python
    X = 99 # Global (module) scope X
    def func():
        X = 88 # Local (function) scope X: a different variable
    ```

2. 作用域细节  
函数提供了嵌套的命名空间（作用域），使其内部使用的变量名局部化。而模块定义了全局作用域。

3. 变量名解析Name Resolution：**LEGB机制**
    - 在默认情况下，变量名赋值会创建或改变局部变量；
    - 变量名引用至多在4种作用域内查找：*先局部local，再外层的函数enclosing functions，再全局global，最后内置built-in（这四个就是LEGB）*；
    - 使用`global`和`nonlocal`语句声明的名称将赋值的变量名分别映射到外围的模块和函数的作用域；
    - 所有在函数def语句内赋值的变量名默认均为局部变量。函数能够任意使用在外层函数内或全局作用域中的变量名，但必须声明为非局部变量和全局变量来改变其属性。

4. 作用域实例
    ``` python
    X = 99
    def func(Y):
        Z = X + Y
        return Z
    print(func(1))
    ```
    - 全局变量名：X，func；局部变量名：Y，Z。
    - 当函数调用结束时，局部变量会从内存中移除。

5. 内置作用域The Built-in Scope
    - 内置作用域仅仅是一个名为`builtins`的内置模块，要导入builtins才能使用内置作用域：
    ``` python
    import builtins
    print(dir(builtins))
    ```
    - 这个列表中的变量名组成了python中的内置作用域。前一半是内置的异常，后一半是内置函数；
    - Python会在LEGB查找中的最后自动查找这个模块；
    - 因此有2种方式引用一个内置函数：利用LEGB法则，或者手动导入builtins模块：
    ``` python
    print(zip)
    import builtins
    print(builtins.zip)
    print(zip is builtins.zip)
    ```

6. 不要重定义内置名称
    - 由于LEGB查找的流程，会使它在第一处找到变量名的地方生效。
    - 在局部作用域中的变量名可能会覆盖在全局作用域和内置作用域中有着相同变量名的变量，而全局变量名可能会覆盖内置的变量名：
    ``` python
    def hider():
        open = 'spam'
        open('data.txt')
    # 这样的话，open在函数内就不能打开文件了
    ```


### 二、The global Statement
1. global语句
    - 全局变量是在外层模块文件的顶层被赋值的变量名；
    - 全局变量如果是在函数内被赋值的话，必须经过声明；
    - 全局变量名在函数的内部不经过声明也可以被引用。
    - global允许我们修改一个def外的模块文件顶层的名称；nonlocal适用于外层def的局部作用域内的名称：
    ``` python
    X = 88
    def func():
        global X
        X = 99 # Global X: outside def
    func()
    print(X)

    y, z = 1, 2
    def all_global():
        global x
        x = y + z
    # 上面x、y、z都是all_global函数内的全局变量
    ```

2. 在Python中使用多线程Multithreading进行并行计算的程序也要依赖全局变量
    - 因为全局变量在并行线程中在不同的函数之间成为共享内存；
    - 多线程与程序的其余部分并行地执行函数调用，由Python的标准库模块_thread、threading和queue提供支持。多线程不在本书探讨内容，详见其他书籍。

3. 其他访问全局变量的方法
    - 一个模块文件的全局作用域一旦被导入就成了这个模块对象的一个属性命名空间：
    ``` python
    # thismod.py
    var = 99

    def local():
        var = 0 # 局部变量，不改变全局的var

    def glob1():
        global var
        var += 1

    def glob2():
        var = 0 # 局部变量，不改变全局的var
        import thismod # 导入自己
        thismod.var += 1 # 改变全局变量var

    def glob3():
        var = 0
        import sys
        glob = sys.modules['thismod'] # 得到模组对象
        glob.var += 1 # 改变全局变量var

    def test():
        print(var)
        local(); glob1(); glob2(); glob3()
        print(var)
    ```
    - 可以通过导入外层模块并对其属性进行赋值来模拟global语句，见thismod.py的glob2()、glob3()函数：
    ``` python
    import thismod
    thismod.test()
    print(thismod.var)
    ```


### 三、Scopes and Nested Functions
1. 嵌套作用域Nested Scope
    - 接下来深入学习LEGB查找规则中的E，即任意外层函数局部作用域的形式：
    ``` python
    X = 99
    def f1():
        X = 88
        def f2():
            print(X)
        f2() # f2是一个临时函数，仅在f1内部执行的过程中存在
    f1()

    def f1():
        X = 88
        def f2():
            print(X)
        return f2 # Return f2 but don't call it
    action = f1()
    action()
    ```
    - 对action名称的调用本质上运行了f1运行时我们命名为f2的函数；
    - 因为Python中的函数与其他一切一样是对象，因此可以作为其他函数的返回值传递回来；
    - f2也记住了f1的嵌套作用域中的X，尽管f1已经不处于激活状态，详见下面。

2. **工厂函数Factory Functions：闭包Closures**
    - 这种代码行为可以叫做**Closures**，也可以**Factory Functions**；
    - 即函数对象能够记忆外层作用域里的值，不管嵌套作用域是否还在内存中存在：
    ``` python
    def maker(N):
        def action(X): 
            return X ** N 
        return action
    ```
    - 上面定义了一个外层函数，用来生成并返回一个嵌套的函数，却不调用这个内嵌的函数；
    - maker创造出action，但只是简单地返回action而不执行它。
    ``` python
    f = maker(2) # 调用maker函数，执行maker内的代码，创建了action函数，并返回action函数
    print(f)
    ```
    - f是生成的内嵌函数即action的一个引用，因为是return action，maker函数创建并返回action函数。
    ``` python
    print(f(3)) # f(3)调用action函数，将3传递进X，返回3**2
    print(f(4))
    ```
    - 上面调用了maker函数创建的并传回的一个内嵌函数。这里不平常的地方在于，在调用action时，maker已经退出，但是内嵌的函数记住了整数2。实际上，在外层嵌套局部作用域内的N被作为执行的状态信息保留了下来，并附加到生成的action函数上。如果再调用外层的函数，可以得到一个新的有不同状态信息的嵌套函数
    ``` python
    g = maker(3)
    print(g(4))
    print(f(4))
    ```
    - 名称g的函数记住了3，名称f的函数记住了2；每个函数都有自己的状态信息state information。
    - 上述是一种相对高级的技术，在代码中不太常见；另外嵌套作用域常常被lambda函数创建表达式利用：
    ``` python
    def maker(N):
        return lambda X: X ** N
    h = maker(3)
    print(h(4))
    ```

3. 闭包vs类
    - 类是一个更好的实现这种状态记忆的选择，因为它们用属性赋值来更加显式地创建它们的内存；
    - 当记忆状态是唯一目标时，闭包函数经常提供一个轻量级的可行的替代方案；
    - 它为每一次调用提供局部化存储空间，来存储一个单独的内嵌函数所需的数据；
    - nonlocal语句允许嵌套作用域状态改变；
    - 当class嵌套在def中，闭包也可以被创建，详见第29章关于嵌套类的说明。

4. 使用默认值参数defaults,来保存外层作用域的状态
    ``` python
    def f1():
        x = 88
        def f2(x=x):
            print(x)
        f2()
    f1()
    ```
    - x=x意味着参数x会默认使用外层作用域中x的值，由于第二个x在python进入内嵌的def之前就已经完成其求值，仍引用f1中的x；实际上嵌套作用域查找规则之所以加入到python中就是为了让默认值参数不再扮演这种角色；所以无需x=x，因为python会自动记住所需要的外层作用域的任意值，从而在内嵌的def中使用。
    - 避免在def中嵌套def，会让程序更简单：
    ``` python
    def f1():
        x = 88
        f2(x)

    def f2(x):
        print(x)

    f1()
    ```

5. 嵌套作用域，lambda
    - `lambda`表达式也为其创建的函数引入新的局部作用域：
    ``` python
    def func():
        x = 4
        action = (lambda n: x ** n)
        return action
    x = func()
    print(x(2))
    ```

6. 循环变量可能需要默认值参数，而不是作用域
    - 如果在函数中定义的lambda或者def嵌套在一个循环之中，而这个内嵌函数又引用了一个外层作用域的变量，该变量被循环所改变，那么所有在这个循环中产生的函数会有相同的值，也就是最后一次循环中完成时被引用变量的值。在这种情形下，需要使用默认值参数来保存变量的当前值：
    ``` python
    def makeActions():
        acts = []
        for i in range(5): # Tries to remember each i
            acts.append(lambda x: i ** x)
        return acts

    T = makeActions()
    print(T[0])
    ```
    运行结果如下为`<function makeActions.<locals>.<lambda> at 0x0000020EEE2D9F30>`
    - 上面的代码会出问题：因为外层作用域中的变量在嵌套的函数被调用时才进行查找，而此时i=4。所以它们实际上记住的是同样的值，也就是最后一次循环迭代中循环变量的值。所以当下面的所有调用传入底数2时，列表中每个结果都是2的4次方：
    ``` python
    print(T[0](2)) # 因为acts列表里面都是lambda函数，所以要先索引再传递参数
    print(T[1](2))
    print(T[2](2))
    print(T[3](2))
    print(T[4](2))
    ```
    运行结果如下：
    ``` python
    0
    1
    4
    9
    16
    ```
    - 为了让这类代码能够工作，必须使用默认值参数来传入当前外层作用域中的值。因为默认值参数的求值是在嵌套函数创建时就发生的（而不是该函数之后被调用时），所以每一个函数都记住了属于自己的变量i：
    ``` python
    def makeActions():
        acts = []
        for i in range(5):
            acts.append(lambda x, i=i: i ** x)
        return acts

    T = makeActions()
    print(T[0](2))
    print(T[1](2))
    print(T[2](2))
    print(T[3](2))
    print(T[4](2))
    ```
    - 更详细见第18章对默认值参数和第19章对lambda的介绍。


### 四、The nonlocal Statement
1. nonlocal语句
    - `nonlocal`语句可以使内嵌的def对外层函数中的名称进行读取和写入访问；
    - `nonlocal`语句通过提供可改写的状态信息，让嵌套作用域闭包变得更加有用；
    - 声明nonlocal名称的时候，必须以及存在于该外层函数的作用域中，不能由内嵌def中的第一次赋值来创建。

2. nonlocal基础