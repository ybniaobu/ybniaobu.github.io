---
title: 《Learning Python》读书笔记（六）
date: 2022-11-10 14:44:00
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
## chapter 20 Comprehensions and Generations
### 一、List Comprehensions and Functional Tools
1. list comprehensions apply an arbitrary expression to items in an iterable, rather than applying a function.

2. List Comprehensions Versus map
    - 用循环收集字符串中字符的ASCII编码：
    ``` python
    res = []
    for x in 'spam':
        res.append(ord(x))
    print(res)
    ```
    - 用`map函数`：
    ``` python
    res = list(map(ord, 'spam'))
    print(res)
    ```
    - **列表推导表达式 list comprehension expression**：
    ``` python
    res = [ord(x) for x in 'spam']
    print(res)
    ```

3. Adding Tests and Nested Loops: filter
    ``` python
    print(list(filter((lambda x: x % 2 == 0), range(5))))
    print([x for x in range(5) if x % 2 == 0])
    ```
    - 等效的for循环：
    ``` python
    res = []
    for x in range(5):
        if x % 2 == 0:
            res.append(x)
    print(res)
    ```

4. Formal comprehension syntax标准推导语法
    - 通用的列表推导的结构如下：
    ``` python
    [ expression for target1 in iterable1 if condition1
                 for target2 in iterable2 if condition2 ...
                 for targetN in iterableN if conditionN ]
    ```
    - 例子：
    ``` python
    res = [x + y for x in [0, 1, 2] for y in [100, 200, 300]]
    print(res)

    print([x + y + z for x in 'spam' if x in 'sm'
                     for y in 'SPAM' if y in ('P', 'A')
                     for z in '123' if z > '1'])
    ```

5. List Comprehensions and Matrixes列表推导和矩阵
    ``` python
    M = [[1, 2, 3],
         [4, 5, 6],
         [7, 8, 9]]
    N = [[2, 2, 2],
         [3, 3, 3],
         [4, 4, 4]]

    print([row[1] for row in M])
    print([M[row][1] for row in (0, 1, 2)])
    print([M[i][i] for i in range(len(M))])
    print([M[i][len(M)-1-i] for i in range(len(M))])
    print([[col + 10 for col in row] for row in M])
    print([[M[row][col] * N[row][col] for col in range(3)] for row in range(3)]) # 这个语句是先for row再for col
    print([[col1 * col2 for (col1, col2) in zip(row1, row2)] for (row1, row2) in zip(M, N)])
    ```

6. 性能
    - `map调用比等效的for循环要快2倍，而列表推导往往比map调用要再快一些`；
    - map和列表推导是以C语言的速度来运行，而for循环在虚拟机PVM中运行。

### 二、Generator Functions
1. Generator Functions and Expressions 生成器函数和生成器表达式
    - 下面2种语言特性让用户可以推迟结果的计算：
        - **Generator functions生成器函数**：使用def编写，但是使用yield语句返回结果；
        - **Generator expressions生成器表达式**：它返回按需产生结果的对象。
    - 以上两者节省了内存空间，允许计算时间分摊到各次结果请求上。

2. Generator Functions: yield Versus return
    - State suspension状态挂起
        - 和返回一个值并退出的常规函数不同，**生成器函数**能够自动挂起并在生成值的时候恢复之前的状态并继续函数的执行。由于生成器函数在挂起时保存的状态包含它们的代码位置和整个局部作用域，因此当函数恢复时，它们的局部变量保持了信息并使其可用。
        - 生成器函数产生yield一个值，而不是返回return一个值，`yield语句`会挂起该函数并向调用者传回一个值，同时也保留了状态。
    - Iteration protocol integration与迭代协议集成
        - 函数若包含一条yield语句，该函数将被特别编译为**生成器**：它们不再是普通函数，而是作为通过特定的迭代协议方法来返回对象的函数。当调用时，它们返回一个生成器对象，该对象支持用一个自动创建的名为`__next__`的方法接口。
        - 生成器函数也可以有一条return语句，在def语句块的结尾，用于终止值的生成。

3. 生成器函数的应用
    ``` python
    def gensquares(N):
        for i in range(N):
            yield i ** 2
    ```
    - 这个函数在每次循环时都会yield一个值，之后将其返还给它的调用者。当它被暂停后，它的上一个状态被保存了下来，包括变量i和N，并且在yield语句之后被收回控制权。
    - 调用生成器函数：
    ``` python
    x = gensquares(4)
    print(x)
    ```
    - 返回的生成器对象有一个`__next__`方法，该方法可以开始这个函数，或者从它上次yield值后的地方恢复，并且在得到一系列值后，引发StopIteration异常：
    ``` python
    print(next(x))
    print(next(x))
    print(next(x))
    print(next(x))
    ```
    - *生成器函数本身就是迭代器*，所以不需要iter调用：
    ``` python
    y = gensquares(5)
    print(iter(y) is y)
    print(next(y))
    ```

4. 为什么要使用生成器函数
    - 生成器对于大型程序而言，在内存使用和性能方面都更好。生成器将产生一系列值的时间分散到每一次的循环迭代中；
    - 生成器函数是一种“穷人的”多线程机制：将操作拆分到每一次的yield之间。然而生成器仍然运行在一个单线程的控制内。

5. 拓展生成器函数协议：send vs next
    - 生成器函数协议有一个`send方法`，可以生成一系列结果的下一个元素，就像__next__方法一样，但是它也提供了一种调用者与生成器之间通讯的方式；
    - yield可以返回发送给send函数的元素（必须用A = yield X）；
    - 当使用这一额外的协议时，值可以通过调用G.send(value)发送给一个生成器G。之后恢复生成器代码的执行，并且生成器中的yield表达式返回了发送给send函数的值，如果提前调用了G.\_\_next\_\_()方法，yield返回None。
    ``` python
    def gen():
        for i in range(10):
            X = yield i
            print(X)
    G = gen()
    print(next(G)) # Must call next() first, to start generator,否则会出错
    print(G.send(77))
    print(G.send(77))
    print(next(G))
    ```
    运行结果如下：
    ``` python
    0
    77
    1
    77
    2
    None
    3
    ```

### 三、Generator Expressions
1. Generator Expressions: Iterables Meet Comprehensions生成器表达式：当可迭代对象遇见推导语法
    - **生成器表达式**跟列表推导很像，但是是在圆括号内，而不是方括号：
    ``` python
    print([x ** 2 for x in range(4)]) # List comprehension
    print((x ** 2 for x in range(4))) # Generator Expressions
    ```
    - 生成器表达式也是迭代器，不需要iter调用：
    ``` python
    G = (x ** 2 for x in range(4))
    print(iter(G) is G)
    print(next(G))
    print(next(G))
    print(next(G))
    print(next(G))
    ```
    - 我们一般不会在生成器表达式看到next机制的使用，因为for循环会自动触发：
    ``` python
    for num in (x ** 2 for x in range(4)):
        print('%s, %s' % (num, num / 2.0))
    ```
    - 每个迭代上下文都如上，包括for循环、sum、map和sorted等内置函数：
    ``` python
    print(''.join(x.upper() for x in 'aaa,bbb,ccc'.split(',')))
    print(sum(x ** 2 for x in range(4)))
    print(sorted(x ** 2 for x in range(4)))
    print(sorted((x ** 2 for x in range(4)), reverse=True))
    ```

2. 为什么使用生成器表达式
    - 就像生成器函数，生成器表达式是一种对内存空间的优化；
    - 生成器表达式在实际运行起来可能比列表推导稍慢一些，但对于非常大的运算或不能等待全部数据产生结果的应用来说是最优选择。

3. 生成器表达式 vs map
    - 生成器表达式通常等效于map调用：
    ``` python
    print(list(map(abs, (-1, -2, 3, 4))))
    print(list(abs(x) for x in (-1, -2, 3, 4)))
    ```
    - map和生成器都可以任意嵌套，下面的列表推导与map产生相同的结果，但是列表推导创建了2个列表：
    ``` python
    [x * 2 for x in [abs(x) for x in (-1, -2, 3, 4)]]
    list(map(lambda x: x * 2, map(abs, (-1, -2, 3, 4))))
    ```

4. 生成器表达式 vs filter
    - filter和一个带有if分句的生成器表达式是等价的：
    ``` python
    line = 'aa bbb c'
    print(''.join(x for x in line.split() if len(x) > 1))
    print(''.join(filter(lambda x: len(x) > 1, line.split())))
    ```

### 四、Generator Functions Versus Generator Expressions
1. 生成器函数 vs 生成器表达式
    - 生成器函数：一个使用了yield表达式的def语句是一个生成器函数。当被调用时，它返回一个新的生成器对象；
    - 生成器表达式：一个被包括在圆括号内的列表推导表达式被称为一个生成器表达式；
    - 这2个都包括一个返回自身的__iter__方法以及一个启动隐式循环或从上次运行离开的地方重新开始的__next__方法，都可以在能主动调用这些接口的迭代上下文中自动按需产生结果。

2. **生成器是单遍迭代对象Generators Are Single-Iteration Objects**
    - 生成器函数和生成器表达式本身都是迭代器，在一个生成器上调用iter没有实际效果。如果尝试手动使用多个迭代器来迭代结果，它们都将指向相同位置：
    ``` python
    G = (c * 4 for c in 'SPAM')
    I1 = iter(G)
    print(next(I1))
    print(next(I1))
    I2 = iter(G)
    print(next(I2))
    ```
    - 一旦任一迭代器运行结束，我们必须产生一个新的生成器以便重新开始。
    - 这与某些内置类型的行为不同。内置类型支持多个迭代器与多次迭代，并且在活跃迭代器中传递并反映它们的原位置修改：
    ``` python
    L = [1, 2, 3, 4]
    I1, I2 = iter(L), iter(L)
    print(next(I1))
    print(next(I1))
    print(next(I2))
    ```

3. `yield from`拓展
    ``` python
    def both(N):
        for i in range(N): yield i
        for i in (x ** 2 for x in range(N)): yield i
    print(list(both(5)))
    ```
    - 用`yield from`：
    ``` python
    def both(N):
        yield from range(N)
        yield from (x ** 2 for x in range(N))
    print(list(both(5)))
    print(' : '.join(str(i) for i in both(5)))
    ```

### 五、Generation in Built-in Types, Tools, and Classes
1. Generators and library tools: Directory walkers生成器和库工具：目录遍历器
    - `os.walk`在Python中的os.py标准库文件中被编写为一个迭代函数，它使用yield返回结果，因此它是一个迭代器：
    ``` python
    import os
    for (root, subs, files) in os.walk('.'):
        for name in files:
            if name.startswith('20'):
                print(root, name)
    ```

2. Generators and function application生成器和函数应用
    - 带“*”的参数可以将一个可迭代对象解包成单独的参数，详见第18章。
    ``` python
    def f(a, b, c): print('%s, %s, and %s' % (a, b, c))
    f(0, 1, 2)
    f(*range(3))
    f(*(i for i in range(3)))
    ```
    - 对字典的解包：
    ``` python
    D = dict(a='Bob', b='dev', c=40.5)
    f(a='Bob', b='dev', c=40.5)
    f(**D)
    f(*D)
    f(*D.values())
    ```

3. 类中用户定义的可迭代对象 详见第30章
    ``` python
    class SomeIterable:
        def __init__(...): ... 
        def __next__(...): ...
    ```

4. Permutations排列: All possible combinations
    - 用递归的方式实现排列（包括不同的顺序）
    ``` python
    def permute1(seq):
        if not seq:
            return [seq]
        else:
            res = []
            for i in range(len(seq)):
                rest = seq[:i] + seq[i+1:]
                for x in permute1(rest):
                    res.append(seq[i:i+1] + x)
            return res

    print(permute1('spam'))
    ```
    - 上述递归，直接看可能有点难理解，用数学来解释排列会好理解一点：
    - 如果让你弄出spam的所有的排列顺序，你会先提出来s，放到最前面，并对pam进行排序；再提出p，放到最前面，并对sam进行排序；...
    - 对应代码为 rest = seq[:i] + seq[i+1:]，即：把当前的节点数字去掉；
    - 对pam进行排序，需要先提出来p，放到最前面，并对am进行排序；对am进行排序，需要先提出来a；这也是为什么需要递归的原因，需要一层一层排序，并且拼接；
    - 对应代码为 for x in permute1(rest):，即：对剩余的数字进行排序；
    - 而 seq[i:i+1] + x 的意思就是把一个个提出来的字母拼接到前面去；
    - 最后返回所有排列组合。
    - 上面例子的生成器函数版本：
    ``` python
    def permute2(seq):
        if not seq:
            yield seq
        else:
            for i in range(len(seq)):
                rest = seq[:i] + seq[i+1:]
                for x in permute2(rest):
                    yield seq[i:i+1] + x

    print(list(permute2('spam')))
    print(permute1('spam') == list(permute2('spam')))
    G = permute2('spam')
    print(next(G))
    print(next(G))
    ```

5. 不要过度使用生成器
    - 生成器的隐式行为可能对于其他程序员难以理解，没必要强行使用让代码变复杂；
    - 列表能自动进行垃圾回收，同时更快地被编写（见下一章节）；
    - 生成器能减少内存和延迟。


### 六、Comprehension Syntax Summary
1. Scopes and Comprehension Variables 作用域及推导变量
    - 生成器推导、集合推导、字典推导以及列表推导中的临时循环变量名的作用域只局限于表达式内，不会与外部变量名冲突，这和for循环不一样：
    ``` python
    x = 99
    print([x for x in range(5)])
    print(x)

    y = 99
    for y in range(5): pass
    print(y)
    ```

2. 集合推导和字典推导
    - 集合推导和字典推导可视为把生成器表达式传递给类型名：
    ``` python
    print({x * x for x in range(10)})
    print(set(x * x for x in range(10)))
    print({x: x * x for x in range(10)})
    print(dict((x, x * x) for x in range(10)))
    ```
    - 与列表推导及生成器表达式一样，集合推导和字典推导接受if分句、嵌套for循环、在可迭代对象上迭代。


## chapter 21 The Benchmarking Interlude
### 一、Timing Iteration Alternatives
1. 计时迭代可选方案
