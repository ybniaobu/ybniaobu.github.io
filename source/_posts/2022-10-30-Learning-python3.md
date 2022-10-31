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

5. 
