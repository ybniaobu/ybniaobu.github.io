---
title: 《Learning Python》读书笔记（三）
date: 2022-10-30 13:36:03
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
> 本篇主要内容为：XXXXXXXXXXXXXXXXXXXXXXX

<div  align="center">  
<img src="https://s2.loli.net/2022/09/17/ri9Ue6nguJdq1Ca.jpg" width = "80%" height = "80%" alt="Learning Python"/>
</div>

# PART II Types and Operations
## chapter 9 Tuples, Files, and Everything Else
### 一、Tuples
1. 元组的特性
任意对象的有序集合；通过偏移量存取；属于**不可变序列**；固定长度、多样性、任意嵌套；对象引用的数组。

2. 元组字面量和运算

| Operation | Interpretation |
| :---- | :---- |
| () | 空元组 |
| T = (0,) | 单个元素的元组 |
| T = (0, 'Ni', 1.2, 3) | 多元素元组 |
| T = 0, 'Ni', 1.2, 3 | 同上(无括号) |
| T = ('Bob', ('dev', 'mgr')) | 嵌套元组 |
| T = tuple('spam') | 可迭代对象的元素组成的元组 |
| T[i] | 索引 |
| T[i][j] | 嵌套索引 |
| T[i:j] | 切片 |
| len(T) | 长度 |
| T1 + T2 | 拼接 |
| T * 3 | 重复 |
| for x in T: print(x) | 迭代 |
| 'spam' in T | 成员关系 |
| [x ** 2 for x in T] | 列表推导 |
| T.index('Ni') | 查找方法 |
| T.count('Ni') | 计数 |
| namedtuple('Emp', ['name', 'jobs']) | 有名元组拓展类型 |

### 二、Tuples in Action
1. 元组的基本操作
``` python
T = (1, 2, 3, 4)
print(T[0], T[1:3])
```

2. 逗号和圆括号  
python允许忽略元组的圆括号，但建议一直使用圆括号；*单个元素的元组如果没逗号，不是元组*：
``` python
x = (40)
print(type(x)) # 不加逗号，x为整数
y = (40,)
print(type(y))
```

3. 转换、方法或不可变性
    - 因为元组是**不可变对象**，需要转为列表再用`sort方法`排序：
    ``` python
    T = ('cc', 'aa', 'dd', 'bb')
    tmp = list(T)
    tmp.sort()
    T = tuple(tmp)
    print(T)
    ```
    - `sorted函数`接受任何序列对象，返回的是一个新的list：
    ``` python
    T = ('cc', 'aa', 'dd', 'bb')
    print(sorted(T))
    ```
    - **列表推导**总是会创建新的列表，可以用来遍历元组、字符串在内的任何序列对象：
    ``` python
    T = (1, 2, 3, 4, 5)
    L = [x + 20 for x in T]
    print(L)
    X = (x + 20 for x in T)
    print(X)
    ```
    - `index方法`和`count方法`：
    ``` python
    T = (1, 2, 3, 2, 4, 2)
    print(T.index(2)) # 第一个2的偏移量
    print(T.index(2, 2)) # 偏移量2后面的2的偏移量
    print(T.count(2))
    ```
    - 元组的不可变性只适用于本组本身顶层而并非其内容，比如*元组内列表可以修改*：
    ``` python
    T = (1, [2, 3], 4)
    T[1][0] = 'spam'
    print(T)
    ```

4. 有名元组Named Tuples
    - `namedtuple`工具（`collections`模块）实现了一个增加了逻辑的元组拓展类型，能同时支持使用序号和属性名访问组件，也可以使用键的类字典形式访问；有名元组的属性名来自类，因此与键不完全相同：
    ``` python
    from collections import namedtuple
    Rec = namedtuple('Rec', ['name', 'age', 'jobs'])
    bob = Rec('Bob', age=40.5, jobs=['dev', 'mgr'])
    print(bob)
    print(bob[0], bob[2])
    print(bob.name, bob.jobs)
    ```
    运行结果如下：
    ``` python
    Rec(name='Bob', age=40.5, jobs=['dev', 'mgr'])
    Bob ['dev', 'mgr']
    Bob ['dev', 'mgr']
    ```
    - 在需要时，可以转换成一个类字典的基于键的形式：
    ``` python
    O = bob._asdict()
    print(O['name'], O['jobs'])
    print(O)
    ```
    运行结果如下：
    ``` python
    Bob ['dev', 'mgr']
    {'name': 'Bob', 'age': 40.5, 'jobs': ['dev', 'mgr']}
    ```
    - 元组赋值，元组和有名元组都支持**解包元组赋值**，详见第13章：
    ``` python
    bob = Rec('Bob', 40.5, ['dev', 'mgr'])
    name, age, jobs = bob
    print(name, jobs)
    for x in bob: print(x)
    ```
    运行结果如下：
    ``` python
    Bob ['dev', 'mgr']
    Bob
    40.5
    ['dev', 'mgr']
    ```


### 三、Files
1. 常见的文件操作

| Operation | Interpretation |
| :---- | :---- |
| output = open(r'C:\spam', 'w') | 创建输出文件，防止出现字符串转义要用原始字符串r |
| input = open('data', 'r') | 创建输入文件 |
| input = open('data') | 与上一行一样，r是默认值 |
| aString = input.read() | 把整个文件读入一个字符串 |
| aString = input.read(N) | 读取接下来的N个字符到一个字符串 |
| aString = input.readline() | 读取下一行（包括\n换行符）到一个字符串 |
| aList = input.readlines() | 读取整个文件到一个字符串列表（包括\n换行符） |
| output.write(aString) | 把字符串写入文件 |
| output.writelines(aList) | 把列表内所以字符串写入文件 |
| output.close() | 手动关闭（当文件收集完成时会替你关闭文件） |
| output.flush() | 把输出缓冲区刷入硬盘中，但不关闭文件 |
| anyFile.seek(N) | 将文件位置移动到偏移量N处以便进行下一个操作 |
| for line in open('data'): use line | 文件迭代器逐行读取 |
| open('f.txt', encoding='latin-1') | Unicode文本文件 |
| open('f.bin', 'rb') | 字节码文件 |

2. 打开文件
    - `open函数`：`afile = open(filename, mode)`
    - `open函数`的第二个参数是处理模式，`'r'`以输入模式打开文件（默认值），`'w'`以输出模式生成并打开文件，`'a'`表示在文件尾部追加内容并打开文件；
    - 在模式字符串中加上`'b'`可以进行二进制数据处理；加上`'+'`意味着被打开文件同时支持输入输出；
    - `open函数`的前2个参数必须是字符串。第三个为可选参数，它能够用来控制输出缓冲：传入`0`意味着输出无缓冲（写入方法调用时立即传入外部文件）；
    - `open函数`的其他参数可用于特殊种类的文件。

3. 使用文件的基础用法的提示：
    - 文件迭代器最适合逐行读取，见第14章；
    - 内容是字符串，不是对象：从文件读取的数据回到脚本时是一个字符串；
    - 文件是被缓冲的以及可定位的：默认情况下，输出文件总是被缓冲的，这意味着写入的文本不能不会立即自动从内存转移到硬盘，除非关闭一个文件，或者运行其flush方法，才能强制被缓冲的数据进入硬盘；
    - `close`通常是可选的：回收时自动关闭：但手动关闭调用在大型程序中通常是不错的习惯，当循环中有许多文件被打开时，可能需要在垃圾回收机制发挥作用前，调用close释放资源。


### 四、Files in Action
1. 文件的基本操作
``` python
myfile = open('9-Tuples, Files, and Everything Else\myfile.txt', 'w')
myfile.write('hello text file\n') # 该方法会返回写入的字符数16
myfile.write('goodbye text file\n') # 该方法会返回写入的字符数18
myfile.close()
myfile = open('9-Tuples, Files, and Everything Else\myfile.txt')
print(myfile.readline())
print(myfile.readline())
print(myfile.readline())
# read方法
print(open('9-Tuples, Files, and Everything Else\myfile.txt').read())
# 文件迭代器
for line in open('9-Tuples, Files, and Everything Else\myfile.txt'):
    print(line, end='')
```

2. 获取当前文件的路径：`os模块`
    - `os.getcwd()方法`，cwd：current working directory：
    ``` python
    import os
    print(os.getcwd())
    ```
    - `os.chdir()方法`改变默认路径，chdir：change directory：`os.chdir('C:\Users\XXXXXXXXXXXXXXXXXX')`

3. 文本文件和二进制文件Text and Binary Files  
文本文件把内容表示为常规的str字符串，自动执行Unicode编码和解码，并且默认执行末行转换；二进制文件把内容表示为一个特殊的bytes字节串类型，并且允许程序不修改地访问内容；二进制文件不会对数据执行任何换行符转换，而文本文件会默认执行对\n的来回转换，并采用Unicode编码。详见第37章：
``` python
data = open('9-Tuples, Files, and Everything Else\data.bin', 'rb').read()
print(data)
print(data[4:8])
print(data[4:8][0]) # s的ASCII码是115
print(bin(data[4:8][0]))
```

4. 在文件中存储Python对象：转换
    - 文件数据在脚本中一定是字符串，需要使用转换工具把对象转为字符串：
    ``` python
    X, Y, Z = 43, 44, 45
    S = 'Spam'
    D = {'a': 1, 'b': 2}
    L = [1, 2, 3]
    F = open('9-Tuples, Files, and Everything Else\datafile.txt', 'w')
    F.write(S + '\n')
    F.write('%s,%s,%s\n' % (X, Y, Z))
    F.write(str(L) + '$' + str(D) + '\n')
    F.close()
    chars = open('9-Tuples, Files, and Everything Else\datafile.txt').read()
    print(chars)
    ```
    - `rstrip方法`，删除\n：
    ``` python
    F = open('9-Tuples, Files, and Everything Else\datafile.txt')
    line = F.readline()
    print(line.rstrip())
    ```
    - `spilt方法`：
    ``` python
    line = F.readline()
    print(line)
    parts = line.split(',')
    print(parts)
    numbers = [int(P) for P in parts] # int函数去除\n
    print(numbers)
    ```
    - `eval函数`，来执行一个字符串表达式，并返回表达式的值。把字符串当作可执行程序代码：
    ``` python
    line = F.readline()
    print(line)
    parts = line.split('$')
    print(parts)
    print(eval(parts[0]))
    objects = [eval(P) for P in parts] # eval函数去除\n
    print(objects)
    ```

5. 存储原生对象：pickle模块 (Storing Native Python Objects: pickle)  
`pickle模块`能够让我们直接在文件中存储任何python对象的高级工具，同时并不需要对字符串进行来回转换。可视为通用的数据格式化和解析工具；pickle模块会执行所谓的对象序列化，也就是对象与字节字符串之间的相互转换：
``` python
D = {'a': 1, 'b': 2}
F = open('9-Tuples, Files, and Everything Else\datafile.pkl', 'wb')
import pickle
pickle.dump(D, F)
F = open('9-Tuples, Files, and Everything Else\datafile.pkl', 'rb')
E = pickle.load(F)
print(E)
```

6. 用JSON格式存储Python对象：json标准库  
由于JSON与Python中字典和列表在语法上的相似性，因此`json标准库`能够很容易地在python对象与JSON格式之间来回转换：
``` python
name = dict(first='Bob', last='Smith')
rec = dict(name=name, job=['dev', 'mgr'], age=40.5)
print(rec)
import json
S = json.dumps(rec)
print(S)
O = json.loads(S)
print(O)
print(O == rec)

json.dump(rec, fp=open(r'9-Tuples, Files, and Everything Else\testjson.txt', 'w'), indent=4)
print(open(r'9-Tuples, Files, and Everything Else\testjson.txt').read())
P = json.load(open(r'9-Tuples, Files, and Everything Else\testjson.txt'))
print(P)
```

7. 存储打包二进制数据：struct模块  
`struct模块`能够构造并解析打包二进制数据，要生成一个打包二进制数据文件，可以用'wb'（写入二进制）模式打开它，并将一个格式化字符串和几个对象传给struct，python会创建一个我们通常写入文件的二进制bytes数据字节码，主要由不可打印字符的十六进制转义组成：
``` python
F = open('9-Tuples, Files, and Everything Else\data.bin', 'wb')
import struct
data = struct.pack('>i4sh', 7, b'spam', 8)
print(data)
F.write(data)
F.close()

F = open('9-Tuples, Files, and Everything Else\data.bin', 'rb')
data = F.read()
print(data)
values = struct.unpack('>i4sh', data)
print(values)
```

8. 文件上下文管理器File Context Managers  
*它可以把文件处理代码包装到一个逻辑层中，以确保在退出后自动关闭文件，而不是依赖于垃圾回收时的自动关闭*，详见34章：
``` python
with open(r'C:\code\data.txt') as myfile:
    for line in myfile:
```

### 五、Core Types Review and Summary
1. 类型分类和要点
    - 字符串、列表和元组都拥有如拼接、长度和索引等序列操作；
    - 只有可变对象（列表、字典和集合）可以在原位置修改；不能原位置修改数字、字符串或元组；
    - 文件只导出方法，因此可变性并不真的适用它们；
    - 数字包括整数、浮点数、复数、小数和分数；
    - 字符串包括str，以及bytes字节串；bytearray字符串类型是可变的；
    - 集合是一个没有值只有键的字典，不能映射，没有顺序，因此不是映射或序列类型，frozenset是集合的一种拥有不可变性的变体。

2. 对象类型

| 对象类型 | 分类 | 是否可变 |
| :---- | :---- | :---- |
| 数字 | 数值 | 否 |
| 字符串 | 序列 | 否 |
| 列表 | 序列 | 是 |
| 字典 | 映射 | 是 |
| 元组 | 序列 | 否 |
| 文件 | 拓展 | N/A |
| 集合 | 集合 | 是 |
| frozenset | 集合 | 否 |
| bytearray | 序列 | 是 |

3. 对象灵活性  
列表、字典和元组可以包括任何种类的对象；集合可以包含任意的**不可变类型immutable对象（可哈希的hashable）**；列表、字典和元组可以任意嵌套；列表、字典和集合可以动态地扩大和缩小。
``` python
L = ['abc', [(1, 2), ([3], 4)], 5]
print(L[1])
print(L[1][1])
print(L[1][1][0])
print(L[1][1][0][0])
```

4. 引用vs复制
    - 由于赋值会产生相同对象的多个引用，因此原位置修改可变对象时，可能会影响其他程序，特别是当涉及嵌套时，关系会复杂：
    ``` python
    X = [1, 2, 3]
    L = ['a', X, 'b']
    D = {'x':X, 'y':2}
    上面为第一行的列表创建了3个引用，由于列表是可变的，修改对象会改变另外2个
    X[1] = 'surprise'
    print(L)
    print(D)
    ```
    - 复制的方法：i、无参数分片表达式(L[:])；ii、字典、集合或列表的copy方法；iii、list、dict、set函数；iv、copy标准库模块。补：无参数的分片和字典的copy方法只能进行顶层复制，即不能复制嵌套的数据结构，如果要复制深层嵌套的数据结构，要用copy模块的deepcopy方法，见6-2章节。
    ``` python
    L = [1,2,3]
    D = {'a':1, 'b':2}
    A = L[:]
    B = D.copy()
    A[1] = 'Ni'
    B['c'] = 'spam'
    print(L, D)
    print(A, B)
    # 就中最开始的例子来说，用分片即可避免共同引用
    X = [1, 2, 3]
    L = ['a', X[:], 'b']
    D = {'x':X[:], 'y':2}
    ```

5. 比较、等价性和真值Comparisons, Equality, and Truth  
当嵌套对象存在时，python能够自动遍历数据结构，并从左到右地应用比较，这被称为**递归比较recursive comparison**，见第19章。就核心类型而言，递归功能是默认实现的。例如，比较列表对象将自动比较所有内容：
    - ==运算符测试值的等价性；is表达式测试对象的同一性：
    ``` python
    L1 = [1, ('a', 3)]
    L2 = [1, ('a', 3)]
    print(L1 == L2, L1 is L2) # L1和L2相等但不是同一个对象
    ```
    - 理论上下面应该是不同对象相同值，因为python内部会对临时存储并重复使用字符串做优化，所以事实上内存中只有一个字符串'spam'：
    ``` python
    S1 = 'spam'
    S2 = 'spam'
    print(S1 == S2, S1 is S2)
    ```
    - 相对大小比较也能递归地应用于嵌套的数据结构：
    ``` python
    L1 = [1, ('a', 3)]
    L2 = [1, ('a', 2)]
    print(L1 < L2, L1 == L2, L1 > L2)
    ```

6. 比较不同类型
    - 数字比较数值的相对大小；
    - 字符串按照字母字典顺序比较（按照ord函数返回字符集编码顺序）字符从左到右比较（'abc'<'ac'）；
    - 列表和元组从左到右，而是对嵌套结构是递归的（[2]>[1,2]）；
    - 集合相对大小采用子集超集的标准；
    - 字典通过比较后的(key,value)来判断是否相同，但不支持相对大小比较。

7. True和False  
***整数0为真，整数1为假；空数据结构为假，非空数据结构为真***，比如`if X != ''`:是为了测试对象是否包括内容：
``` python
print(bool(1))
print(bool('spam'))
print(bool({}))
```

8. None对象
`None`为一个特殊对象，可被认为是假。一般起到一个空占位符的作用；但`None`不意味着未定义，`None`是一个对象，而不是没有内容。

9. 类型测试
    - 严格来说，对象的类型本身，也属于type类型的对象：
    ``` python
    print(type(type(1)))
    ```
    - 类型测试及`isinstance函数`：
    ``` python
    print(type([1]) == type([]))
    print(type([1]) == list)
    print(isinstance([1], list))
    ```
    - `types模块`提供了不能作为内置类型使用的类型的名称：
    ``` python
    import types
    def f(): pass
    print(type(f) == types.FunctionType)
    ```

### 六、Built-in Type Gotchas
1. 序列重复
    - 序列重复就是多次将序列加在自己身上：
    ``` python
    L = [4, 5, 6]
    X = L * 4
    Y = [L] * 4
    print(X)
    print(Y)
    ```
    - 由于L在赋值给Y时是被嵌套的，因此Y中包含了指向原本L的列表的引用：
    ``` python
    L[1] = 0
    print(X)
    print(Y)
    ```
    - 解决上面的问题与之前一样，复制个副本：
    ``` python
    L = [4, 5, 6]
    Y = [list(L)] * 4
    L[1] = 0
    print(Y)
    ```
    - 尽管Y与L不再共用同一个列表对象，但Y中的嵌套的列表都指向同一对象：
    ``` python
    Y[0][1] = 99 # 所有四个都被改变了
    print(Y)
    ```
    - 避免上述共享，需要保证每个嵌套都有一个单独副本：
    ``` python
    L = [4, 5, 6]
    Y = [list(L) for i in range(4)]
    print(Y)
    Y[0][1] = 99 # 这样就只改变了第一个
    print(Y)
    ```

2. 循环数据结构Cyclic Data Structures
如果一个复合对象包含指向自身的引用，就称之为循环对象。python在对象中检测到循环，都会打印成[...]，除非真的需要，不建议使用循环引用：
``` python
L = ['grail']
L.append(L)
print(L)
```

3. 不可变类型不可以在原位置改变
不能在原位置改变不可变对象，必须通过分片、拼接等操作来创建一个新的对象，再赋值回原来的引用


# PART III Statements and Syntax
## chapter 10 Introducing Python Statements
### 一、Python’s Statements
1. Python程序结构  
程序由模块构成；模块包含语句；语句包含表达式；表达式创建并处理对象  
Programs are composed of modules；Modules contain statements；Statements contain expressions；Expressions create and process objects

2. Python’s Statements语句

| Statement | Role | Example |
| :---- | :---- | :---- |
| 赋值语句 | 创建引用reference | a, b = 'good', 'bad' |
| 调用和其他表达式 | 运行函数 | log.write("spam, ham") |
| print语句 | 打印对象 | print('The Killer', joke) |
| if/elif语句 | 选择行为 | if "python" in text: print(text) |
| for语句 | 迭代 | for x in mylist: print(x) |
| while语句 | 循环loop | while X > Y: print('hello') |
| pass | 空占位符Empty placeholder | while True: pass |
| break | 退出循环 | while True: if exittest(): break |
| continue | 循环继续 | while True: if skiptest(): continue |
| def | 函数与方法 | def f(a, b, c=1, *d): print(a+b+c+d[0]) |
| return | 函数结果 | def f(a, b, c=1, *d): return a+b+c+d[0] |
| yield | 生成器函数 | def gen(n): for i in n: yield i*2 |
| global | 命名空间Namespaces | def function(): global x |
| nonlocal | 非局部声明 | def outer(): x = 'old' def function(): nonlocal x; x = 'new' |
| import | 获取模块 | import sys |
| from | 获取模块属性 | from sys import stdin |
| class | 构建对象 | class Subclass(Superclass): |
| try/except/finally | 捕捉异常Catching exceptions | try: |
| raise | 触发异常 | raise EndSearch(location) |
| assert | 调试异常Debugging checks | assert X > Y, 'X too small' |
| with/as | 上下文管理器 | with open('data') as myfile: |
| del | 删除引用 | del data[k] |
