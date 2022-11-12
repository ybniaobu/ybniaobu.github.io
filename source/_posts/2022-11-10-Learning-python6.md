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
    - 列表推导有时比for循环有速度优势，而map调用看情况，列表推导有时比for循环有速度优势，而map调用看情况：
    ``` python
    import time
    def timer(func, *args):
        start = time.perf_counter() 
        for i in range(1000): # 执行1000次
            func(*args)
        return time.perf_counter()  - start

    print(timer(pow, 2, 1000))
    ```
    - 上述代码有一些局限，包括额外计入了range的时间、不支持关键字参数，扩展上述代码，详见timer.py，如下：
    ``` python
    # timer.py
    """
    Homegrown timing tools for function calls.
    Does total time, best-of time, and best-of-totals time
    """

    import time, sys
    timer = time.perf_counter if sys.platform[:3] == 'win' else time.time

    def total(reps, func, *pargs, **kargs):
        """
        Total time to run func() reps times.
        Returns (total time, last result)
        """
        repslist = list(range(reps))
        start = timer()
        for i in repslist:
            ret = func(*pargs, **kargs)
        elapsed = timer() - start
        return (elapsed, ret)

    def bestof(reps, func, *pargs, **kargs):
        """
        Quickest func() among reps runs.
        Returns (best time, last result)
        """
        best = 2 ** 32 # 136 years seems large enough
        for i in range(reps): # range usage not timed here
            start = timer()
            ret = func(*pargs, **kargs)
            elapsed = timer() - start # Or call total() with reps=1
            if elapsed < best: best = elapsed # Or add to list and take min()
        return (best, ret)

    def bestoftotal(reps1, reps2, func, *pargs, **kargs):
        """
        Best of totals:
        (best of reps1 runs of (total of reps2 runs of func))
        """
        return bestof(reps1, total, reps2, func, *pargs, **kargs)
    ```
    运行代码如下：
    ``` python
    import timer
    print(timer.total(1000, pow, 2, 1000)[0])
    print(timer.total(1000, str.upper, 'spam'))
    print(timer.bestof(1000, str.upper, 'spam'))
    print(timer.bestof(1000, pow, 2, 1000000)[0])
    print(timer.bestof(50, timer.total, 1000, str.upper, 'spam'))
    print(timer.bestoftotal(50, 1000, str.upper, 'spam'))
    ```

2. 计时脚本
    - 计时迭代工具的计时器脚本，详见timeseqs.py，如下：
    ``` python
    "Test the relative speed of iteration tool alternatives."

    import sys, timer
    reps = 10000
    repslist = list(range(reps))

    def forLoop():
        res = []
        for x in repslist:
            res.append(abs(x))
        return res

    def listComp():
        return [abs(x) for x in repslist]

    def mapCall():
        return list(map(abs, repslist))

    def genExpr():
        return list(abs(x) for x in repslist)

    def genFunc():
        def gen():
            for x in repslist:
                yield abs(x)
        return list(gen())

    print(sys.version)
    for test in (forLoop, listComp, mapCall, genExpr, genFunc):
        (bestof, (total, result)) = timer.bestoftotal(5, 1000, test)
        print ('%-9s: %.5f => [%s...%s]' %
        (test.__name__, bestof, result[0], result[-1]))
    ```
    - 如代码所示，它报告的时间反应了这5个测试函数执行1000万部的快慢，每个函数创建了1000次带有10000个元素的列表；
    - 根据timeseqs.py的运行结果，生成器表达式比列表推导慢很多；map，即映射内置函数abs这样的内置函数，map会比列表推导快。

3. 无论如何，性能不是编写python代码的第一优先级。写出具有可读性和间接性的代码优先，然后在需要时优化。

### 二、Timing Iterations and Pythons with timeit
1. python标准库的timeit模块，自动化了代码计时
    - `timeit`，测试可以用可调用对象或语句字符串所指定；
    - 语句字符串：支持分隔符“；”或换行符“\n”分开的多条语句，以及用空格或制表符表示的缩进语句。

2. Interactive usage and API calls交互式用法与API调用
    - `timeit模块`中的`repeat`：表示调用运行若干组相同测试并返回一个列表，列表中的各项表示运行各组测试的总时间；
    - 每组测试由若干条相同测试语句（由number指定）组成；
    - 列表中的最小项（可通过min获取）就是各组运行的最佳时间；
    - stmt是statement的缩写。
    ``` python
    import timeit
    print(min(timeit.repeat(stmt="[x ** 2 for x in range(1000)]", number=1000, repeat=5)))
    ```
    - 上述代码运行5次，每次都执行了1000次的创建了拥有1000个元素的列表推导式。

3. Command-line usage命令行用法
    - timeit模块可以通过Python的 -m 标签自动地在路径上定位作为脚本运行：  
    `c:\code> python -m timeit -n 1000 "[x ** 2 for x in range(1000)]"`  
    `c:\code> py −3 -m timeit -n 1000 -r 5 "[x ** 2 for x in range(1000)]"`  
    `c:\code> C:\python33\Lib\timeit.py -n 1000 "[x ** 2 for x in range(1000)]"`  

4. Timing multiline statements计时对行语句
    - 在API调用模式下，可以使用换行符、制表符或空格写出多行代码：
    ``` python
    import timeit
    print(min(timeit.repeat(number=10000, repeat=3,
        stmt="L = [1, 2, 3, 4, 5]\nfor i in range(len(L)): L[i] += 1")))

    print(min(timeit.repeat(number=10000, repeat=3,
        stmt="L = [1, 2, 3, 4, 5]\ni=0\nwhile i < len(L):\n\tL[i] += 1\n\ti += 1")))
    ```
    - 在命令行模式下运行多行语句，可以把每条语句作为单独参数传入，并使用空白缩进；
    - timeit会把所有行拼接起来并插入换行符：  
    `c:\code> py −3 -m timeit -n 1000 -r 3 "L = [1,2,3,4,5]" "M = [x + 1 for x in L]"`

5. Other usage modes: Setup, totals, and objects其他使用模式：初始化、总时间和可运行对象
    - timeit允许让初始化的代码不计入要测试的语句的总时间:
        - 指定初始化代码，在命令行里用-s标签：  
        `c:\code> python -m timeit -n 1000 -r 3 -s "L = [1,2,3,4,5]" "M = [x + 1 for x in L]"` 这里把列表作为初始化语句分离了出来；
        - 或在API调用模式下使用setup参数字符串：
        ``` python
        import timeit
        print(min(timeit.repeat(number=10000, repeat=3,
            setup="L = [1, 2, 3, 4, 5]",
            stmt="for i in range(len(L)): L[i] += 1")))
        ```
    - timeit可以只计时总时间，使用模块的类API，计时可调用对象而不是字符串，详见库手册：
    ``` python
    import timeit
    print(timeit.timeit(stmt='[x ** 2 for x in range(1000)]', number=1000)) # Total time
    print(timeit.Timer(stmt='[x ** 2 for x in range(1000)]').timeit(1000)) # Class API
    print(timeit.repeat(stmt='[x ** 2 for x in range(1000)]', number=1000, repeat=3))

    def testcase():
        y = [x ** 2 for x in range(1000)]
    print(min(timeit.repeat(stmt=testcase, number=1000, repeat=3)))
    ```

### 三、Function Gotchas
1. Local Names Are Detected Statically局部变量是被静态检测的
    - python是在编译def代码时静态检测Python的局部变量的，而不是在运行时检测的：
    ``` python
    X = 99
    def selector(): 
        print(X)
    #   X = 88 如果这个语句在def里面，则出现未定义变量名错误
    selector()
    ```
    - 在编译时，python看到对X的赋值语句，决定了X会在函数中所有地方都是局部变量。但当函数运行时，执行print时，赋值并未发送，这时就会出现未定义变量名错误；
    - 产生这个问题的原因是，被赋值的变量名在函数内部的所有位置都被当做局部变量对待，不是在赋值以后的语句才被当作局部变量。
    - 如果想打印全局变量X，需要global语句：
    ``` python
    def selector():
        global X
        print(X)
        X = 88
    selector()
    print(X)
    ```
    - 但这样会改变全局变量X，如果不想改变，需要导入外围模块：
    ``` python
    X = 99
    def selector():
        import __main__
        print(__main__.X)
        X = 88
        print(X)
    selector()
    ```

2. Defaults and Mutable Objects默认值参数和可变对象
    - 用作默认值参数的可变值可以在调用之间保留状态；
    - 当def语句运行时，默认值参数就被求值并保存，而不是调用时；
    - python会将每一个默认值参数保存成一个对象，附加在函数本身；
    - 因为默认值参数在def时被求值，它能在外层作用域中保存值（详见工厂函数）。
    ``` python
    def saver(x=[]):
        x.append(1)
        print(x)
    saver([2])
    saver()
    saver()
    saver()
    ```


# PART V Modules and Packages
## chapter 22 Modules: The Big Picture
### 一、Python Program Architecture
1. 一个模块文件顶层定义的所有变量都变成了被导入的模块对象的属性。

2. Python Program Architecture程序架构
    - 一个python拥有一个主体的顶层文件，辅以数个被称为模块的支持文件；
    - 顶层文件包含了程序的主要控制流程：即用来启动应用程序的文件；
    - 模块文件是工具库。

3. Imports and Attributes导入与属性
    - `import语句`会逐行运行在目标文档中对的语句从而构建其中对象，使其变成模块的属性即`module.attribute`。

4. How Imports Work导入语句如何工作
    - 导入会执行3个步骤：找到模块文件；编译成字节码；执行模块的代码来创建其所定义的对象；
    - 这3步骤只会在程序第一次导入才会进行，之后导入相同的模块时，会跳过这3个步骤，只提取内存中已加载的模块对象；
    - python会把载入的模块存储在一个叫sys.modules的表中，每次导入操作先检查该表，不存在，则启动上述3个步骤。
    - Python使用标准模块搜索路径来找出import语句所对应的模块文件，详见第24章；
    - python会把模块编译为字节码，如果发现字节码文件比源代码旧，则会自动生成新的字节代码。字节码被存在__pycache__子目录里，只有被导入的文件才会在机器上留下.pyc字节码文件，顶层文件的字节码在内部使用后就丢弃了；
    - import的最后步骤就是执行，文件中的语句会从头到尾被执行。

### 二、The Module Search Path
1. 模块搜索路径
    - 多数情况下，我们可以依赖模块导入搜索路径的自动特性，完全不需要配置这些路径；
    - python的模块搜索路径是下面这些主要组件拼接的结果，其中有些需要自定义：
        - 程序的主目录；（自动的）  
        即包含程序的顶层脚本文件所在的目录；这个目录总是会优先被搜索，所以会覆盖其他目录中相同名称的模块。
        - PYTHONPATH目录（可配置的configurable）；  
        python会从左到右搜索PYTHONPATH环境变量设置中罗列出的所有目录；PYTHONPATH是设置包含python程序文件的目录列表，可以把想导入的目录都加进来。
        - 标准库目录（自动的）；
        - 任何.pth文件中的内容（可配置的）；  
        不常用，python允许用户把需要的目录写在后缀名为.pth的文本文件中一行一行列出来，作为PYTHONPATH设置的一种替代方案；可以把文件放在python安装目录的顶层（C:\Python33）或者标准库所在位置的sitepackages子目录（C:\Python33\Lib\site-packages）。
        - 第三方扩展应用的site-packages主目录（自动的）；  
        python会自动将标准库的site-packages子目录添加到模块搜索路径。
        - 以上5个组件组合成了sys.path。

2. Configuring the Search Path配置搜索路径
    - 可以通过【我的电脑】-【属性】-【高级系统设置】-【环境变量】-【新建】，变量名写PYTHONPATH，变量值就是你要导入模块的路径；或者在python安装目录下创建.pth文本文件。
    - 上面的方法是永久设置模块的搜索路径。
    
3. sys.path列表
    - 暂时设置模块的搜索路径；
    - Python在程序启动时配置sys.path，自动将上述5个目录合并，形成一个列表；
    - 这个列表提供一种让脚本手动定制其搜索路径的方式，这种修改只在脚本执行时保持，可以采用`sys.path.append`或`sys.path.insert`来改变列表：
    ``` python
    import sys
    print(sys.path)
    sys.path.append(r'XXXX')
    print(sys.path)
    ```

4. 模块文件选择
    - python会在搜索路径中选择第一个能够匹配导入名称的文件；
    - 导入语句的本质是外部组件暴露的接口，包括以下类型：源代码文件b.py; 字节码文件b.pyc; 优化字节码文件b.pyo（不常见）; 目录b（对于包导入而言，见24章）；编译拓展模块（c或c++编写），导入时使用动态链接；用c编写的编译好的内置模块，变被静态链接至python；ZIP文件组件，导入时自动解压缩（标准库路径就是一个.zip文件）
    - 更多细节参考Python标准库手册中的内置__import__函数的说明，这个函数是import语句的可定制工具。


## chapter 23 Module Coding Basics
### 一、Module Creation and Usage
1. import语句