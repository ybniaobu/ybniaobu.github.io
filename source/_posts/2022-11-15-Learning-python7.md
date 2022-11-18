---
title: 《Learning Python》读书笔记（七）
date: 2022-11-15 14:09:30
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
> 本篇主要内容为：包导入；class基础。

<div  align="center">  
<img src="https://s2.loli.net/2022/09/17/ri9Ue6nguJdq1Ca.jpg" width = "80%" height = "80%" alt="Learning Python"/>
</div>

# PART V Modules and Packages
## chapter 24 Module Packages
### 一、Package Import Basics
1、除了模块名之外，导入还可以指定目录路径
- Python代码的目录被称为**包 package**；
- 包导入是把目录变成Python命名空间，其属性对应目录中所包含的子目录和模块文件。

2、包导入基础
- `import dir1.dir2.mod` 或 `from dir1.dir2.mod import x`
- 上述语句表明了dir1里有子目录dir2，dir2里包含mod.py

3、包和搜索路径设置
- import语句中的目录路径只能是以点号间隔的变量，不能import C:\mycode\dir1\dir2\mod。

4、\_\_init\_\_.py包文件
- 如果选择使用包导入，包导入语句的路径中的*每个目录*都必须有\_\_init\_\_.py这个文件，否则包导入会失败；
- 也就是说上面dir1和dir2里都必须包含\_\_init\_\_.py文件，而容器目录dir0不需要\_\_init\_\_.py文件，dir0必须在模块搜索路径的sys.path列表中；
- 即dir0\dir1\dir2\mod.py 对应 import dir1.dir2.mod
- \_\_init\_\_.py可以包含代码，也可以是空的；
- 它们的代码将在python第一次导入一个路径的时候被自动运行，所以可以被作为执行包的初始化的钩子。

5、Package initialization file roles包初始化文件的角色
- \_\_init\_\_.py文件用作包初始化的钩子hook，将目录声明成一个Python包，替目录生成一个模块命名空间以及在目录导入时实现from\* 语句
- 包的初始化：python在首次导入目录时，会自动执行该目录下\_\_init\_\_.py文件中的代码；
- 模块使用的声明：包的\_\_init\_\_.py文件即声明一个路径是Python的包；
- 模块命名空间的初始化：导入表达式dir1.dir2运行后，会返回一个模块对象，该对象的命名空间包含了dir2的\_\_init\_\_.py文件中赋值的所有名称；
- from\* 语句的行为：可以在\_\_init\_\_.py文件内定义\_\_all\_\_列表来规定目录以from\*语句形式导入什么。\_\_all\_\_列表指包名称使用from\*时，应该导入的子模块的名称清单。如果没有设定\_\_all\_\_，from\*语句不会自动加载嵌套于该目录内的子模块。\_\_init\_\_.py可以是空白，但必须存在。

### 二、Package Import Example
1、包导入示例
- 在dir1和dir2文件夹里设置\_\_init\_\_.py文件：

``` python
# dir1\__init__.py
print('dir1 init')
x = 1

# dir1\dir2\__init__.py
print('dir2 init')
y = 2

# dir1\dir2\mod.py
print('in mod.py')
z = 3

import dir1.dir2.mod
import dir1.dir2.mod # 第二次import不再执行

from importlib import reload
reload(dir1)
reload(dir1.dir2)

print(dir1)
print(dir1.dir2)
print(dir1.dir2.mod)

print(dir1.x)
print(dir1.dir2.y)
print(dir1.dir2.mod.z)
```

2、包的from语句 vs 包的import语句
``` python
from dir1.dir2 import mod
print(mod.z)

from dir1.dir2.mod import z
print(z)

import dir1.dir2.mod as mod
print(mod.z)

from dir1.dir2.mod import z as modz
print(modz)
```

3、 为什么使用包导入
- 包导入提供了程序文件的目录信息；
- 包导入也可以简化PYTHONPATH和.pth文件搜索路径设置；
- 包能解决多个同名文件安装在同一个机器上所引发的模糊性。

### 三、Package Relative Imports
1、包相对导入
- 包内部导入同一个包中的内容时，可以使用和外部导入相同的完整路径语法，也可以利用特殊的包内搜索规则来简化import语句：
- 对于包中导入：
    - 默认跳过包自己的目录。导入只检查sys.path列表中的搜索路径。称为**绝对导入**：`import XX`；
    - from语句允许显式地要求导入只搜索包的目录（以点号开始）。称为**相对导入**：`from . import XX`。

2、相对导入基础知识
- from语句可以使用以点号开头的子句来导入位于同一包中的模块（包相对导入），而不是位于模块导入搜索路径上某处的模块（绝对导入）；
- 比如：`from . import spam` 即将文件相同包路径中名为spam的一个模块导入；
- 类似：`from .spam import name` 在文件所位于的包内，找到spam的模块并导入其中变量name；
- 而import string则总是在sys.path上的某处查找一个string模块，而不会查找该包中的同名模块，即绝对导入。
- 例：在一个名为mypkg的包目录下的一个模块文件：
    - `from .string import name1, name2` 即Imports names from mypkg.string；
    - `from . import string` 即Imports mypkg.string；
    - `from .. import string` 即从mypkg的父目录导入string模块。
- *相对导入中的 “.” 用来表示包含当前文件的包目录；“..” 表示当前包的父目录的相对导入。*
    - `from . import D` 即 Imports A.B.D (. means A.B)
    - `from .. import E` 即 Imports A.E (.. means A)
    - `from .D import X` 即 Imports A.B.D.X (. means A.B)
    - `from ..E import X` 即 Imports A.E.X (.. means A)

3、相对导入的实际应用
- 包外导入
    ``` python
    import string
    print(string)
    ```
    - 上述代码，如果当前工作目录下，没有string.py，则会导入标准库的string模块；
    - 但是因为模块搜索路径的第一项就是当前工作目录（current working directory，CWD）。
- 包内导入
    - 在pkg文件夹下设置空的\_\_init\_\_.py：  
    ``` python
    # code\pkg\spam.py
    from . import eggs  # import eggs会出错，因为import是绝对导入，会查找该包中的模块
    print(eggs.X)

    # code\pkg\eggs.py
    X = 99999
    import string
    print(string)

    import pkg.spam
    ```

4、包相对导入的陷阱
- 不能随意使用from . 的相对导入语法，除非发起导入的文件本身是包的一部分；
- 导入不会搜索一个包模块自身的路径，除非使用from . 的相对导入语法；
- 最好能使用完整包路径导入，即`from system.section.mypkg import mod`，来替代包相对导入或简单导入（import mode）
- 详见书720-724

5、\_\_name\_\_ == "\_\_main\_\_"
- 当一个模块作为顶级脚本被运行时，它的\_\_name\_\_属性的值是“\_\_main\_\_”字符串，但当它被导入却不是；
- 即`if __name__ == "__main__"`后的语句在被导入后不被执行。

### 四、Namespace Packages
1、命名空间包
- 4种导入模型：
    - **基础模块导入**：`import mod`, `from mod import attr`
    - **包导入**：`import dir1.dir2.mod`, `from dir1.mod import attr`
    - **包相对导入**：`from . import mod` (相对), `import mod` (绝对)
    - **命名空间包**：`import splitdir.mod`：运行包横跨多个目录，不需要__init__.py初始化文件。
- 两种风格的包：
    - 原始的模型，现在称为**常规包regular packages**；
    - 可选的模型，称为**命名空间包namespace packages**。
- 命名空间包模型常常被作为一种后备选项进行使用。

2、命名空间包的语义
- 常规包必须拥有一个__init__.py文件，而且必须位于一个独立的目录里；
- 命名空间包可以横跨多个路径，这些路径在被导入时被收集；
- 所有能够成为一个命名空间包组成部分的目录都不能包含一个__init__.py文件，但是可以被嵌套在里面当作一个单独的包。

3、命名空间包导入算法
- 当对每个模块搜索路径中的directory搜索名为spam的被导入包时，python会按照下面的顺序测试一系列匹配条件：
  - 如果找到directory\spam\\_\_init\_\_.py，便会导入一个常规包并返回；
  - 如果找到directory\spam.{py, pyc, or other module extension}，便会导入一个简单模块并返回；
  - 如果找到文件夹directory\spam，便把它记录下来，而扫描将从搜索路径中的下一个目录继续；
  - 如果上述的所有都没找到，扫描将从搜索路径中的下一个目录继续。
- 如果搜索路径没有从上述步骤1和步骤2中返回一个模块或包，但在步骤3至少记录了一个路径，就会创建一个命名空间包；
- python只会在一个模块或常规包被找到的时候，或者整个路径已经被完全扫描过以后才停止搜索；
- 命名空间包只有在整个过程中没有找到其他同名的模块、包或文件才会被返回；
- 在模块搜索路径上任意位置的模块文件和常规包都优先于命名空间包目录；
- 命名空间包的创建会立即发生，不会推迟到子层级的导入发生之时。新的命名空间包有一个\_\_path\_\_属性；
- 该属性被设置为在上述步骤3中扫描并记录的目录路径字符串的可迭代对象，但是没有__file__属性；
- \_\_path\_\_属性在随后更深的访问过程中用于搜索所有包组件；
- 命名空间包是访问更低层次项目的“父路径”。

4、对常规包的影响：可选的\_\_init\_\_.py
- 如果一个单独目录包没有该文件，它将被当作一个单独目录命名空间包，而且不会引发任何警告；
- 同时，原始的常规包模型仍然支持，而且作为一个初始化钩子会自动运行\_\_init\_\_.py文件中的代码；
- 而且常规包有性能上的优势。

5、命名空间包的实际应用
- 在这一结构中，2个名为sub的子目录位于2个不同的父目录dir1和dir2中：
    - C:\code\ns\dir1\sub\mod1.py
    - C:\code\ns\dir2\sub\mod2.py
- 如果将dir1和dir2都添加进模块搜索路径，sub将成为一个横跨2个目录的命名空间包

``` python
# ns\dir1\sub\mod1.py
print(r'dir1\sub\mod1')

# ns\dir2\sub\mod2.py
print(r'dir2\sub\mod2')

import sys
sys.path.append(r'XXXXX\\ns\\dir1')
sys.path.append(r'XXXXX\\ns\\dir2')

import sub.mod1 # 无错误，可以运行
import sub.mod2

print(sub.mod1)
print(sub.mod2)

print(sub)
print(sub.__path__)
```

- 相对导入也适用于命名空间包：
    - 可以把mod1里的代码改为
    
    ``` python
    from . import mod2
    print(r'dir1\sub\mod1')
    ```
    
    - 这样import sub.mod1会得到dir2\sub\mod2和dir1\sub\mod1。

6、命名空间包嵌套
- 命名空间包支持任意嵌套，并成为低层级的“父路径”；
- 较低层的组件可以是一个模块、常规包也可以是另一个命名空间包。


> **分界线（之前的笔记格式要调整）**

## chapter 25 Advanced Module Topics
### 一、Module Design Concepts
1、模块设计概念
- 在python中用户总是位于某个模块中，即使在交互式命令行下输入的代码实际上也存在于*\_\_main\_\_*的内置模块中；
- 最小化模块*耦合coupling*：全局变量，模块应该尽可能地独立于其他模块内使用的全局变量；
- 最大化模块*内聚cohesion*：统一的目标；
- 模块尽可能不去更改其他模块的变量；
- 模块不仅可以被导入，而且还可以导入和使用其他用python或者诸如C的其他语言编写的模块。

2、使 \* 的破坏最小化：*\_X* 和 *\_\_all\_\_*
- 可以在名称前面加上下划线，防止导入时把名称复制出来，最小化对命名空间的破坏；
- `from *` 是复制出所有的名称；

``` python
# unders.py
a, _b, c, _d = 1, 2, 3, 4
```

``` python
from unders import *
print(a, c)
print(_b) # 会出现NameError
```

- 但 import 仍然可以获取并修改这类名称：

``` python
import unders
print(unders._b)
```

- 可以通过在模块顶层把变量名的字符串列表赋值给变量 *\_\_all\_\_* ，从而达到类似于 *\_X* 命名惯例的隐藏效果；
- 使用此功能时，`from *` 语句只会把列在 *\_\_all\_\_ 列表*中的这些名称复制出来；
- *\_\_all\_\_* 是指明要复制的名称，而 *\_X* 是指明不被复制的名称。

``` python
# alls.py
__all__ = ['a', '_c']
a, b, _c, _d = 1, 2, 3, 4
```

``` python
from alls import *
print(a, _c)
print(b) # 会出现NameError

from alls import a, b, _c, _d
print(a, b, _c, _d)

import alls
print(alls.a, alls.b, alls._c, alls._d)
```

- 就像 *\_X* 一样，*\_\_all\_\_ 列表*只对 `from *` 语句有效；其他导入语句仍然能访问全部名称。

3、启用未来语言特性：*\_\_future\_\_ 模块*
- 可以在Python 2.X中使用`from __future__ import featurename`来获取3.X的语言特性。

### 二、Built-in Attribute \_\_name\_\_
1、*\_\_name\_\_* 和 *\_\_main\_\_*
- 每个模块都有一个名为 *\_\_name\_\_* 的内置属性：
    - 如果文件作为顶层程序文件执行，在启动时 *\_\_name\_\_* 就会被设置为字符串*"\_\_main\_\_"*；
    - 如果文件被导入， *\_\_name\_\_* 就会设为客户程序所了解的模块名。

2、混合使用模式

``` python
# runme.py
def tester():
    print("It's Christmas in Heaven...")

if __name__ == '__main__':
    tester()
```

``` python
import runme # 因为是导入，不是顶层文件执行，所以导入语句执行模块代码时，不直接调用函数
runme.tester() # 正常调用函数
```

- 这里的 *\_\_name\_\_* 允许模块被同时编写成一个可导入的库和一个顶层文件；
- 因为`if __name__ == '__main__'`后面是调用，不是定义函数或者 print ；
- 所以 *\_\_name\_\_* 可以用于自我测试，判断是在执行还是在导入。

3、以 *\_\_name\_\_* 进行测试

``` python
# minmax.py
# 直接执行minmax得到I am: __main__；1；6
print('I am:', __name__)

def minmax(test, *args):
    res = args[0]
    for arg in args[1:]:
        if test(arg, res):
            res = arg
    return res

def lessthan(x, y): return x < y
def grtrthan(x, y): return x > y

if __name__ == '__main__':
    print(minmax(lessthan, 4, 2, 1, 5, 6, 3))
    print(minmax(grtrthan, 4, 2, 1, 5, 6, 3))
```

``` python
import minmax # 得到I am: minmax
print(minmax.minmax(minmax.lessthan, 's', 'p', 'a', 'a')) # 得到'a'
```

### 三、Other Advanced Module-related Topics
1、修改模块搜索路径

``` python
import sys
print(sys.path)
```

- `sys.path.append('C:\\sourcedir')` 添加路径；
- `sys.path = [r'd:\temp']` 修改路径；
- `sys.path.insert(0, '..')` 插入路径；
- `sys.path`是暂时的，只存在于发生的 python 会话，修改不会被保留下来。

2、 import 语句和 from 语句的* as 拓展*
- 重命名 import 的模块：

``` python
import reallylongmodulename as name
name.func()
```

- 同理 from 也可以：

``` python
from module1 import utility as util1
from module2 import utility as util2
util1(); util2()
```

3、用名称字符串导入模块

``` python
X = 'string'
import X
```

- 这样代码会尝试导入 X.py ，而不是 string 模块。
- 要用`exec内置函数`解决， *exec* 会编译代码字符串，传给解释器执行：

``` python
modname = 'minmax'
exec('import ' + modname)
```

- 或者用内置的`__import__函数`，但是这个函数返回模块对象：

``` python
modname = 'minmax'
minmax = __import__(modname)
print(minmax)
```

- 或者用`importlib.import_module`：

``` python
import importlib
modname = 'minmax'
minmax = importlib.import_module(modname)
print(minmax)
```

### 四、Module Gotchas
1、模块名称冲突：包和包相对导入
- 如果有2个同名模块，默认情况下， python 总是选择搜索路径 *sys.path* 中最左边的那一项，要么避免这问题，要么使用包导入功能。

2、顶层代码中语句次序很重要
- 当模块被导入（或重载）， python 会从头到尾执行它的代码语句。

3、from 复制名称，而不是链接，和 import 不太一样
-  import 是将模块对象赋值到名称。

4、reload 不能作用于 from 导入
- 因为 from 是复制，不会链接到名称所在模块，所以重载无效。

5、递归形式的 from 导入可能无法工作
- 相互导入的模块，称为*递归导入 recursive imports*；
- 用 from 可能会导致递归不会实际发生，有可能在导入时，名称尚不存在，用 import 可能没什么影响。


# PART VI Classes and OOP
## chapter 26 OOP: The Big Picture
### 一、OOP and Class
1、*类 Classes* 是*面向对象程序设计 object-oriented programming (OOP)* 的主要工具
- 类支持**继承 inheritance** ，一种代码定制和复用的机制，真正的OOP，对象需要有继承层次。

2、类 Classes
- *多重实例 Multiple instances*
    - 类是产生对象的工厂。每当调用类，就会产生一个带有独立命名空间的新对象，该对象可以读取类的属性，并用自己的命名空间来存储数据；
- *通过继承进行定制 Customization via inheritance*
    - 可以在类的外部以编写子类的方式，来重新定义其属性进而扩充类；
- *运算符重载 Operator overloading*
    - python 提供了一些可以让类使用的钩子，从而能够拦截并实现任何的内置类型运算。

3、属性继承搜索
- `object.attribute`
- 当对 class 语句产生的对象使用上述表达式时，会启用一次搜索，搜索对象连接的类树；
- 即：找到 attribute 首次出现的地方，先搜索 object ，然后是该对象上的所有类，由下往上；
- *属性访问就是搜索类树，这种搜索即继承*，因为树中较低位置的对象继承了较高位置的对象的所有属性；
- 树中较高位置的类称为**父类 superclass**，较低位置称为**子类 subclass**；父类提供了所有子类共享的行为，子类可以重新定义父类的名称，从而覆盖父类定义的行为。

4、编写类树

``` python
class C2: ... # 创建父类
class C3: ... # 创建父类
class C1(C2, C3): ... # 继承父类，括号内从左到右的顺序决定了树中次序

I1 = C1() # 类调用，创建实例
I2 = C1()
```

- 类的属性是在 class 语句的顶层语句块中通过赋值语句添加到类的；
- 属性是通过 **self** 的赋值，来附加给实例的。 **self** 提供了被处理的实例的引用；
- 类通过方法函数为实例提供行为（即 class 里的 def 语句）：

``` python
class C2: ... 
class C3: ...

class C1(C2, C3):
    def setname(self, who):
        self.name = who

I1 = C1()
I2 = C1()

I1.setname('bob')
I2.setname('sue')

print(I1.name)
```

- 上面的代码，直到 setname 调用前，C1 类都不会把 name 属性附加到实例上。
- `__init__方法`（又称为**构造函数**），如果没有`__init__方法`，类调用将返回一个空实例，而不会将其初始化：

``` python
class C2: ... 
class C3: ...

class C1(C2, C3):
    def __init__(self, who):
        self.name = who

I1 = C1('bob')
I2 = C1('sue')
print(I1.name)
```

5、在很多应用领域，可以获取或购买父类的集合，即软件框架 framework  
这些软件框架可能提供一些数据库接口、测试协议、GUI工具包等


## chapter 27 Class Coding Basics
### 一、Classes Generate Multiple Instance Objects
1、类对象和实例对象
- *类对象 class objects* 提供默认行为，*实例对象 instance objects* 是程序处理的实际对象；
- class 语句创建类并赋值给一个名称，class 语句内的赋值语句会创建类的属性；
- 类属性提供了对象的**状态信息**和**行为**；
- 调用类对象会创建实例对象，每个实例对象继承了类的属性并获得了自己的命名空间；
- 在方法内对 self 属性做赋值运算会产生每个实例自己的属性。

2、第一个示例

``` python
class FirstClass:
    def setdata(self, value):
        self.data = value
    def display(self):
        print(self.data)

x = FirstClass() # 调用产生实例
y = FirstClass()
x.setdata("King Arthur")
y.setdata(3.14159)

x.display()
y.display()
```

- 类中的函数称为**方法 methods** ；
- setdata 函数中，传入的值会赋给 self.data 。 self 会自动引用当前处理的实例，所以赋值语句会把值存储在实例的命名空间；
- 可以修改实例属性：

``` python
x.data = "New value"
x.display()
```

- 也可以在类方法函数外，在实例命名空间内产生全新的属性（很少见，不常用）：

``` python
x.anothername = "spam"
print(x.anothername)
```

### 二、Classes Are Customized by Inheritance
1、类产生的实例对象继承类的属性。也可以让类继承其他类。
- 父类列在 class 语句头部的括号里；
- 类从其父类中继承属性；
- 实例会继承所有可访问类的属性；
- 每个 object.attribute 引用都会启动一个独立的搜索；
- 逻辑的修改是通过创建子类，而不是修改父类。

2、第二个示例

``` python
class FirstClass:
    def setdata(self, value):
        self.data = value
    def display(self):
        print(self.data)

class SecondClass(FirstClass):
    def display(self): # Changes display
        print('Current value = "%s"' % self.data)
```

- 继承搜索会从实例往上进行，首先到子类，然后到父类；
- 所以 SecondClass 覆盖了 FirstClass 中的 display ，有时称这种重新定义取代属性的动作为*重载 overloading* ：

``` python
z = SecondClass()
z.setdata(42) # FirstClass的setdata
z.display()
```

3、类是模块内的属性
- 若 FirstClass 从其他模块导入：

``` python
from modulename import FirstClass
class SecondClass(FirstClass):
    def display(self): ...
```

- 或者，其等效写法如下：

``` python
import modulename
class SecondClass(modulename.FirstClass):   
    def display(self): ...
```

### 三、Classes Can Intercept Python Operators
1、**运算符重载**，即让类编写的对象，截获并响应用在内置类型上的运算：加法、切片、打印和点号运算等，运算符重载能让对象拥有内置对象那样的行为。
- 以双下划线命名的方法（*\_\_X\_\_*）是特殊钩子，来拦截运算；
- 当实例出现在内置运算中，这类方法会自动被调用；
- 类可以重载绝大多数内置类型运算；
- 默认的运算符不存在，如果类没有定义或继承运算符重载方法，那么类的实例将不能支持相应运算；
- 新式类有一些默认的运算符重载方法。新式类见后面；
- 运算符将类与对象模型结合到一起，让类获得与内置对象一样的行为；
- 运算符重载主要被 Python 工具开发人员使用，而不是应用程序开发人员。所以不应被随意使用；
- 但是`__init__方法`，几乎每个类都会用到，也称为构造函数方法，用于初始化对象状态。

2、第三个示例
- `__init__`会在创建实例时被调用： self 是新的 ThirdClass 对象；`__add__`会在 ThirdClass 实例出现在 + 表达式时被调用；`__str__`会在打印对象时被调用：

``` python
class FirstClass:
    def setdata(self, value):
        self.data = value
    def display(self):
        print(self.data)

class SecondClass(FirstClass):
    def display(self): # Changes display
        print('Current value = "%s"' % self.data)

class ThirdClass(SecondClass):
    def __init__(self, value):
        self.data = value
    def __add__(self, other):
        return ThirdClass(self.data + other)
    def __str__(self):
        return '[ThirdClass: %s]' % self.data
    def mul(self, other):
        self.data *= other

a = ThirdClass('abc')
a.display()
print(a) # __str__: returns display string

b = a + 'xyz' # __add__: makes a new instance。实例对象a传给了__add__中的self，右侧传给了other
b.display()
print(b)

a.mul(3)
print(a)
```

### 四、The World’s Simplest Python Class
1、最简单的类

``` python
class Rec: pass
```
- 可以在 class 语句外，通过赋值变量名给这个类增加属性：

``` python
Rec.name = 'Bob'
Rec.age = 40
```

- 此时创建实例，会继承附加在类上的属性：

``` python
x = Rec()
y = Rec()
print(x.name, y.name)
```

- 可以再对实例对象赋值属性，因为属性引用会启动继承搜索，所以实例会先获得实例属性：

``` python
x.name = 'Sue'
print(Rec.name, x.name, y.name)
```

- 命名空间对象的属性通常以字典形式实现的，类继承树就是互相连接的字典，详见第29章。
- `__dict__属性`是大多数基于类的对象的命名空间字典：

``` python
print(list(Rec.__dict__.keys()))
print(list(name for name in Rec.__dict__ if not name.startswith('__')))
print(list(x.__dict__.keys()))
print(list(y.__dict__.keys()))
```

- `__class__`代表实例指向其类的链接：

``` python
print(x.__class__)
```

- 类也有个`__bases__属性`，它是其父类对象引用的元组，在这里是隐含的 object 根类：

``` python
print(Rec.__bases__)
```

2、python的类模型是相当动态的
- 类和实例只是*命名空间对象*，可以在任何地方使用它们的属性；
- 方法也可以独立地创建在任意类对象的外部：

``` python
def uppername(obj):
    return obj.name.upper()
```

- 只要传入一个带 name 属性（该属性自带有 upper 方法）的 obj 对象就可以调用
- x 这个类实例适合 uppername 的接口：

``` python
print(uppername(x))
```

- 我们也可以把函数赋值成类的属性，该函数就变成了类的方法：

``` python
Rec.method = uppername
print(x.method())
print(y.method())
print(Rec.method(x))
```


## chapter 28 A More Realistic Example
### 一、Example of Classes

``` python
# person.py
class Person:
    # 步骤1
    def __init__(self, name, job=None, pay=0):
        self.name = name
        self.job = job
        self.pay = pay
    
    # 步骤2
    def lastName(self): 
        return self.name.split()[-1]

    def giveRaise(self, percent):
        self.pay = int(self.pay * (1 + percent))
    
    # 步骤3
    def __repr__(self):
        return '[Person: %s, %s]' % (self.name, self.pay)
    

    # 步骤4
class Manager(Person):
    # 步骤5
    def __init__(self, name, pay):
        Person.__init__(self, name, 'mgr', pay)

    # 步骤4
    def giveRaise(self, percent, bonus=.10):
        Person.giveRaise(self, percent + bonus) # 也可以复制父类的代码self.pay = int(self.pay * (1 + percent + bonus))，坏处就是修改代码就需要修改2次了
```

**步骤1**：创建实例
- 实例对象属性在类方法函数中的 self 属性赋值来创建。常见方法是在*\_\_init\_\_构造函数方法*中赋值给 self ；
- self 就是新创建的实例对象。job 参数是 *\_\_init\_\_* 函数作用域里的一个局部变量，而 self.job 是实例的一个属性；
- `self.job = job` 就是把局部 job 赋给 self.job 属性。
- 生成几个实例：

``` python
from person import Person

bob = Person('Bob Smith')
sue = Person('Sue Jones', job='dev', pay=100000)
print(bob.name, bob.pay)
print(sue.name, sue.pay)
```

**步骤2**：添加行为方法
- **封装encapsulation**：封装思想就是把操作逻辑包装到接口之后；
- 把操作对象的代码编写到类方法，即封装的一种方式：

``` python
print(bob.lastName(), sue.lastName())
sue.giveRaise(.10)
print(sue.pay)
```

**步骤3**：运算符重载
- 排在 *\_\_init\_\_* 之后第二常用的运算符重载 *\_\_repr\_\_* ，以及 *\_\_str\_\_* ；
- 打印对象会显示该对象的 *\_\_str\_\_* 或 *\_\_repr\_\_* 方法所返回的内容；
- 这2者被用在不同的上下文实现不同的显示，但是只编写 *\_\_repr\_\_* 足以支持几乎所有的场景：

``` python
print(bob)
print(sue)
```

**步骤4**：通过编写子类定制行为
- 编写名为 Manager 的子类：

``` python
from person import Manager
tom = Manager('Tom Jones', 50000)
tom.giveRaise(.10)
print(tom.lastName())
print(tom)
```

- `super内置函数`也可以调用父类，详见第32章。
- 多态的应用：

``` python
print('--All three--')
for obj in (bob, sue, tom):
    obj.giveRaise(.10)
    print(obj)
```

- 对象可能是一个 Person 或 Manager ，而 Python 自动运行相应的 giveRaise ，这就是 Python 中的多态概念

**步骤5**：定制构造函数
- 为 Manager 定制构造函数，初始化对象的状态信息属性。
- 组合类 Combine Classes 的其他方式
- `__getattr__运算符重载方法`，能拦截未定义属性的访问并把这些访问委托给一个带有 getattr 内置调用的内嵌对象，详见第30章。第31章将讨论组合 composition 的设计问题。

**步骤6**：使用*内省工具 Introspection*
- Python 的内省工具允许我们访问对象实现的内部机制的一些特殊属性和函数，语言框架工具的开发者会比应用程序开发者更广泛地使用它们：
    - 内置的 *instance.\_\_class\_\_ 属性*提供了一个从实例到创建它的类的链接。同时类还有个 *\_\_name\_\_* ，还有一个 *\_\_base\_\_* 序列来提供父类的访问；
    - 内置的 *object.\_\_dict\_\_* 属性提供了一个字典，将所有命名空间对象中的属性都存储为键值对。

``` python
bob = Person('Bob Smith')
print(bob)
print(bob.__class__)
print(bob.__class__.__name__)

print(list(bob.__dict__.keys()))

for key in bob.__dict__:
    print(key, '=>', bob.__dict__[key])

for key in bob.__dict__:
    print(key, '=>', getattr(bob, key))

print(dir(bob))
```

**步骤7**：把对象存储到数据库中
- 我们关闭 python ，实例会消失，因为它是内存中临时性对象；
- 因为应用程序往往需要永久性改变，所以需要把对象持久化 object persistence/permanence；
    - `pickle 模块`：实现任意 python 对象与字节串之间的序列化和解序列化；
    - `dbm 模块`：实现一个通过键访问的文件系统，以存储字节串；
    - `shelve 模块`：使用以上2个模块，按照键把 Python 对象存储到一个文件中；
    - shelve 使用 pickle 把一个对象转换为其 pickle 化的字节串，并将其存储在一个 dbm 文件中的键之下；pickle：腌制、酸菜 shelve：搁置。
- 在 shelve 数据库中存储对象，详见 makedb.py ：

``` python
# makedb.py
from person import Person, Manager
bob = Person('Bob Smith')
sue = Person('Sue Jones', job='dev', pay=100000)
tom = Manager('Tom Jones', 50000)

import shelve
db = shelve.open('persondb') # Filename where objects are stored
for obj in (bob, sue, tom): # 把对象的名称用做键，把它们赋给shelve
    db[obj.name] = obj
db.close()
```

- 交互式地探索 shelve ：

``` python
import shelve
db = shelve.open('persondb')

print(len(db))
print(list(db.keys()))

bob = db['Bob Smith']
print(bob)

print(bob.lastName())

for key in db:
    print(key, '=>', db[key]) # db[key]即对应的对象，打印出来就是运行了__repr__方法

for key in sorted(db):
    print(key, '=>', db[key])
```

- 以上代码即导入了实例对象，并且将 bob 连接到它。
- 更新 shelve 中的对象：
    - 编写一个程序，在每次运行的时候都更新一个实例，以证实对象是持久的，即每次运行时，它们当前的值都是可用的，详见 updatedb.py ：

``` python
# updatedb.py
import shelve
db = shelve.open('persondb')

for key in sorted(db):
    print(key, '\t=>', db[key])

sue = db['Sue Jones']
sue.giveRaise(.10)
db['Sue Jones'] = sue
db.close()
# 每次运行都会永久改变 sue 的工资，可以证明对象的持久化。
```

- 如果要了解 pickle 和 shelve 的更多细节，见官方手册。