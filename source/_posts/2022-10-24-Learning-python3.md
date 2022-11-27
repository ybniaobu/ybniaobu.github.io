---
title: 《Learning Python》读书笔记（三）
date: 2022-10-24 13:30:33
categories: 
  - [python, python读书笔记]
tags:
  - python
  - 读书笔记
top_img: /images/black.jpg
cover: https://s2.loli.net/2022/11/27/R9htYK5LyFuEGTk.jpg
---

> 该笔记为 **《Learning Python》** 的读书笔记，由于是早期未搞熟博客系统时所写，笔记结构较为混乱；  
> 该书涉及的内容可能过于啰嗦，但包含一些python背后的逻辑和机制，可以粗略过一遍，但若仔细阅读就是在坑自己；  
> 该笔记内容过多，所以不展示部分代码的结果，需复制到编辑器中查看；  
> 学习完成日期为2022年10月20日。  
> 本篇主要内容为：Python解释器；Python对象类型的初步介绍；数值类型。

<div  align="center">  
<img src="https://s2.loli.net/2022/09/17/ri9Ue6nguJdq1Ca.jpg" width = "80%" height = "80%" alt="Learning Python"/>
</div>

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
        - 以上5个组件组合成了`sys.path`。

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
1. `import语句`和`from语句`
    ``` python
    import module
    module.method()

    from module import method
    method()
    ```

2. `from*语句`
    - 当我们使用*代替特定名称时，会取得模块顶层被赋值的所有名称的副本：
    ``` python
    from module import *
    method()
    ```

3. Imports Happen Only Once导入只发生一次
    - simple.py如下：
    ``` python
    print('hello')
    spam = 1
    ```
    - 导入执行从头到尾执行一次module文件：
    ``` python
    import simple # 执行了代码，打印了hello
    print(simple.spam)
    ```
    - 第二次导入并不会重新执行该模块的代码，只是从内部模块表取出已创建的模块对象：
    ``` python
    simple.spam = 2
    import simple
    print(simple.spam)
    ```
    - 如果需要再一次运行，需要内置函数`reload`，见后面。

4. import和from是赋值语句
    - import和from是可执行语句，可以被嵌套在if测试里，在def里等等；
    - import和from是隐式的赋值语句；
    - *import将整个模块对象赋值给一个单独名称*；
    - *from将一个或多个名称赋值给另一个模块中的同名对象*；
    - *以from复制的名称会变成对共享对象的引用，所以要小心共享的可变对象*：
    - 比如small.py，如下：
    ``` python
    x = 1
    y = [1, 2]
    ```
    导入small.py：
    ``` python
    from small import x, y
    x = 42 # Changes local x only
    y[0] = 42
    import small
    print(small.x)
    print(small.y)
    ```
    - 若想修改另一文件的全局变量名，必须用import：
    ``` python
    import small
    small.x = 42 # Changes x in other module
    print(small.x)
    ```

5. import and from Equivalence等价性
    - from只是把名称从一个模块复制到另一个模块，但是不会对模块名本身进行赋值；
    - 所以从概念上来说，以下2段代码是等价的
        - `from module import name1, name2`；
        - import module  
            name1 = module.name1  
            name2 = module.name2  
            del module
    - 跟所有赋值语句一样，from语句会在导入者中创建新的变量，而这些变量在初始化时引用了被导入文件中的同名对象，不过，只复制了名称，没复制引用的对象。

6. Potential Pitfalls潜在陷阱 of the from Statement
    - 因为from语句会让变量的位置更隐式和模糊，所以推荐使用import而不是from，虽然使用from也没什么太多可怕的后果。


### 二、Module Namespaces
1. 模块命名空间
    - 理解模块的一种方式就是把它看作名称的封装；
    - 模块就是命名空间，存在于一个模块内的名称被称为模块对象的属性。
        - 模块语句会在首次导入时执行：模块被导入时，python会建立空的模块对象，并逐一执行该模块文件内的语句；
        - 顶层的赋值语句会创建模块属性：在导入时，文件顶层（即不在def与class之内）的赋值名称语句会建立对象的属性，存储在模块的命名空间里；
        - 模块的命名空间可以通过属性__dict__会dir(M)获取；
        - 模块是一个独立的作用域：模块文件的作用域在模块导入后就成为模块对象属性的命名空间。
    - 模块在首次导入，从头到尾执行语句：
    ``` python
    # module2.py
    print('starting to load...')
    import sys
    name = 42

    def func(): pass

    class klass: pass

    print('done loading.')
    ```
    ``` python
    import module2
    ```
    - 模块被加载后，它的作用域就变成了返回的模块对象的一个属性命名空间：
    ``` python
    print(module2.sys)
    print(module2.name)
    print(module2.func)
    print(module2.klass)
    ```

2. 命名空间字典：__dict__
    - 在内部，模块命名空间被存储为字典对象：
    ``` python
    print(list(module2.__dict__.keys()))
    ```
    - 里面的__file__指明模块是从哪个文件加载，__name__指明导入者的名称。

3. Attribute Name Qualification属性名称的点号运算  
**qualification点号运算** (a.k.a. attribute fetch属性获取) syntax object.attribute

4. 导入vs作用域
    - moda.py：
    ``` python
    X = 88
    def f():
        global X
        X = 99
    ```
    ``` python
    X = 11

    import moda 
    moda.f() 
    print(X, moda.X)
    ```
    - moda.f()修改了moda中的X，而不是这个文件的X。moda.f的全局作用域一定是所在文件；
    - 一段代码的作用域完全由该代码在文件中所处的实际位置决定。作用域绝不会被函数调用或模块导入影响。

5. 命名空间的嵌套Namespace Nesting
    - 虽然导入不会使命名空间发送向上的嵌套，但确实会发生向下的嵌套。
    ``` python
    # mod3.py
    X = 3
    ```
    ``` python
    # mod2.py
    X = 2
    import mod3

    print(X, end=' ')
    print(mod3.X)
    ```
    ``` python
    X = 1
    import mod2 # 打印出2 3

    print(X, end=' ') # 打印1
    print(mod2.X, end=' ') # 打印2
    print(mod2.mod3.X) # 打印3
    ```
    - 无法编写`import mod2.mod3`。这会启用包导入，在下一章介绍。


### 三、Reloading Modules
1. Reloading Modules
    - 要强制使模块代码重新载入并重新运行，要调用`reload内置函数`。

2. reload基础
    - reload是函数，不是语句；reload传入的参数是一个存在的模块对象；reload在python 3.X中位于模块之中，需要导入才能使用；
    - 一般的用法是导入一个模块，在文本编辑器内修改其代码，然后将其重新加载；
    - 当调用reload时，Python会重读模块文件的源代码，重新执行其顶层语句；
    - reload会在原位置修改模块对象，reload并不会删除并重新创建模块对象。
        - reload会在模块当前命名空间内执行模块文件的新代码：覆盖其现有命名空间而不是删除重建；
        - 文件中顶层赋值语句会将名称替换为新的值；
        - 重新加载只会对以后使用from的用户程序造成影响，之前使用from来读取属性的程序不会受到重新加载影响；
        - 重新加载会影响所以使用import读取了模块的用户程序；
        - 重新加载只适用于单一的模块。

3. reload示例
    - 要用python解释器演示，上述函数打印出来后，不要关掉解释器，此时修改changer.py文件的代码，然后调用reload：
    ``` python
    # changer.py
    message = "First version"
    def printer():
        print(message)
    ```
    ``` python
    import changer
    changer.printer()
    ```
    修改后：
    ``` python
    from importlib import reload
    reload(changer) # reload会返回模块对象<module 'changer' from XXXXXXXXXX>
    changer.printer() # 这时候会显示修改后的结果
    ```
    - 除了在交互式命令行下重新加载模块外，模块重新加载在较大程序也十分有用，重新加载使程序能够提供高度动态的接口。

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
1、静态方法和类方法介绍
- 静态方法大致与类中简单无实例函数的工作方法类似，而类方法被传入一个类而不是一个实例；
- 要使用这些方法，要在类中调用特殊的内置函数：`staticmethod` 和 `classmethod` ，或使用 `“@name”` 装饰语法；
- 在 python 3.X 中，无实例的方法只通过一个类名调用，而不需要一个 `staticmethod` 申明，但要实例来调用，仍然需要。

①静态方法：嵌套在类中的没有 self 参数的简单函数；
②类方法：传入方法的第一个参数不是实例对象，而是类对象；
③实例方法：即常规方法，需要接受实例。

2、Python 2.X 和 3.X 中的静态方法
①在 Python 2.X 和 3.X 中，当一个方法通过实例被获取的时候，会产生一个绑定方法；
②在 Python 2.X 中，从一个类中获取一个方法会产生一个非绑定方法，如果不手动地传入一个实例就不能调用这个方法；
③在 Python 3.X 中，从一个类中获取一个方法会产生一个简单函数，该函数在没有传入一个实例的时候也可以正常被调用；
④在 Python 2.X 中，必须总是把一个方法声明为静态的，才能不传入实例来调用它，不管是通过类还是实例调用；
⑤在 Python 3.X 中，如果一个方法只通过一个类调用，不需要声明为静态的。但是要通过实例来调用，必须声明为静态的。

``` python
class Spam: # 类实例计数器
    numInstances = 0
    def __init__(self):
        Spam.numInstances = Spam.numInstances + 1
    def printNumInstances():
        print("Number of instances created: %s" % Spam.numInstances)

a = Spam()
b = Spam()
c = Spam()

Spam.printNumInstances()
a.printNumInstances() # 会出现错误：TypeError: Spam.printNumInstances() takes 0 positional arguments but 1 was given
```

- 以下为上述例子的常规方法：

``` python
class Spam:
    numInstances = 0
    def __init__(self):
        Spam.numInstances = Spam.numInstances + 1
    def printNumInstances(self):
        print("Number of instances created: %s" % Spam.numInstances)

a, b, c = Spam(), Spam(), Spam()
a.printNumInstances()
Spam.printNumInstances(a)
Spam().printNumInstances()
```

3、内置函数 `staticmethod` 和 `classmethod`
- 静态方法不需要实例，类方法需要一个类参数：

``` python
class Methods:
    def imeth(self, x):
        print([self, x])
    
    def smeth(x):
        print([x])

    def cmeth(cls, x):
        print([cls, x])
    
    smeth = staticmethod(smeth)
    cmeth = classmethod(cmeth)
# 常规实例方法
obj = Methods()
obj.imeth(1)
Methods.imeth(obj, 2)
# 静态方法
Methods.smeth(3)
obj.smeth(4)
# 类方法：python自动将类传入类方法的第一位，不管是类调用还是实例调用
Methods.cmeth(5)
obj.cmeth(6)
```

4、子类继承并定制静态方法

``` python
class Spam:
    numInstances = 0
    def __init__(self):
        Spam.numInstances += 1
    def printNumInstances():
        print("Number of instances: %s" % Spam.numInstances)
    printNumInstances = staticmethod(printNumInstances)

class Sub(Spam):
    def printNumInstances():
        print("Extra stuff...")
        Spam.printNumInstances()
    printNumInstances = staticmethod(printNumInstances)

a = Sub()
b = Sub()
a.printNumInstances()
Sub.printNumInstances()
Spam.printNumInstances() # 调用父类的方法

class Other(Spam): pass

c = Other()
c.printNumInstances() # 这里会打印出3，是因为子类都是继承了父类的构造方法
```

5、继承类方法
- 类方法接受的是调用主体的最直接的类：

``` python
class Spam:
    numInstances = 0
    def __init__(self):
        Spam.numInstances += 1
    def printNumInstances(cls):
        print("Number of instances: %s %s" % (cls.numInstances, cls))
    printNumInstances = classmethod(printNumInstances)

class Sub(Spam):
    def printNumInstances(cls):
        print("Extra stuff...", cls)
        Spam.printNumInstances()
    printNumInstances = classmethod(printNumInstances)

class Other(Spam): pass

x = Sub()
y = Spam()
x.printNumInstances() # 子类的实例调用类方法，传入了sub，又由于Sub显式调用了Spam父类，故父类方法接受了父类自己
Sub.printNumInstances()
y.printNumInstances()

z = Other()
z.printNumInstances()
```

6、由于类总是接受实例树中最底层（lowest）的类
- 因此更适合处理同一类树中的各个类之间相互区别的数据，比如为每个类管理实例计数器：

``` python
class Spam:
    numInstances = 0
    def count(cls):
        cls.numInstances += 1
    def __init__(self):
        self.count()
    count = classmethod(count)

class Sub(Spam):
    numInstances = 0
    def __init__(self):
        Spam.__init__(self)

class Other(Spam):
    numInstances = 0

x = Spam()
y1, y2 = Sub(), Sub()
z1, z2, z3 = Other(), Other(), Other()
print(x.numInstances, y1.numInstances, z1.numInstances)
print(Spam.numInstances, Sub.numInstances, Other.numInstances)
```

### 六、Decorators and Metaclasses: Part 1
1、装饰器介绍
①**函数装饰器 Function decorators**：同时为简单函数和类方法指明了特殊运算模式，通过把简单函数和类方法包在一层额外的逻辑实现中，也称为元函数 metafunction；
②**类装饰器Class decorators**：为类添加管理全体对象和其接口的支持。  

python 提供一些内置的函数装饰器，程序员也可以自己编写定制装饰器，装饰器不是严格地被要求写成类，但是通常被编写成类

2、函数装饰器基础
- 函数装饰器可以看作是它跟在后面的函数的运行时声明；
- 它包含 **“@”** 符号，和跟着后面的元函数 metafunction（管理另一函数的函数）：

``` python
class Spam:
    numInstances = 0
    def __init__(self):
        Spam.numInstances = Spam.numInstances + 1
    
    @staticmethod # 跟写在后面的printNumInstances = staticmethod(printNumInstances)一样
    def printNumInstances():
        print("Number of instances created: %s" % Spam.numInstances)

a = Spam()
b = Spam()
c = Spam()
Spam.printNumInstances()
a.printNumInstances()
```

- 因为 `classmethod` 和 `property` 内置函数也接受和返回函数，也能用作装饰器：

``` python
class Methods(object):
    def imeth(self, x):
        print([self, x])
    
    @staticmethod
    def smeth(x):
        print([x])
    
    @classmethod
    def cmeth(cls, x):
        print([cls, x])
    
    @property
    def name(self):
        return 'Bob ' + self.__class__.__name__

obj = Methods()
obj.imeth(1)
obj.smeth(2)
obj.cmeth(3)
print(obj.name)
```

3、用户定义函数装饰器
- 下面的代码在实例中存储被装饰的函数，并捕获对原来函数名的调用：

``` python
class tracer:
    def __init__(self, func):
        self.calls = 0
        self.func = func
    def __call__(self, *args):
        self.calls += 1
        print('call %s to %s' % (self.calls, self.func.__name__))
        return self.func(*args)

@tracer # Same as spam = tracer(spam)
def spam(a, b, c):
    return a + b + c

print(spam(1, 2, 3))
print(spam('a', 'b', 'c'))
```

4、类装饰器和元类
- 同理，在类前面的装饰器 `@decorator` 等同于在 class 语句后面的 `Class = decorator(Class)`：

``` python
def count(aClass):
    aClass.numInstances = 0
    return aClass

@count
class Spam:
    def __init__(self):
        Spam.numInstances += 1

@count
class Sub(Spam):
    def __init__(self):
        Spam.__init__(self)
        Sub.numInstances += 1

a = Spam()
b = Sub()
print(a.numInstances, Spam.numInstances)
print(b.numInstances, Sub.numInstances)
```

- 类装饰器也可以通过拦截构造函数，并将实例对象包在一个代理中，管理实例的全部接口；
- 详见第39章，下面是模型的预习：

``` python
def decorator(cls):
    class Proxy:
        def __init__(self, *args):
            self.wrapped = cls(*args)
        def __getattr__(self, name):
            return getattr(self.wrapped, name)
    return Proxy

@decorator
class C: ...
X = C()
```

- 元类能把一个类对象的创建路由到顶级 type 类的一个子类，见第40章。

### 七、The super Built-in Function
1、传统的父类调用形式

``` python
class C:
    def act(self):
        print('spam')

class D(C):
    def act(self):
        C.act(self)
        print('eggs')

X = D()
X.act()
```

2、super 的基础用法
- super 通过检测调用栈来自动定位 self 参数和寻找父类，并且将 self 参数和父类配对在一个特殊的代理对象中，从而将之后的调用路由到父类方法；
- 但 super 函数后面的没有 self ：

``` python
class C:
    def act(self):
        print('spam')

class D(C):
    def act(self):
        super().act()
        print('eggs')

X = D()
X.act()
```

3、局限性：多继承
- super 函数是钻石多继承树中的协同多继承分发协议，依赖于 MRO 算法；
- 钻石多继承树中的协同多继承分发协议：cooperative multiple inheritance dispatch protocols in diamond multiple-inheritance trees。

``` python
class A:
    def act(self): print('A')
class B:
    def act(self): print('B')
class C(A, B):
    def act(self):
        super().act()
class D(B, A):
    def act(self):
        super().act()

X = C()
X.act() # super根据MRO顺序，找到最左边的带有该方法的类
Y = D()
Y.act() # 同理
```

- 在多继承中，能用传统继承就用传统继承，因为 super 只能继承一个。

4、super的优势：继承树的修改与分发
①在运行时改变父类：可以通过super来分发调用：

``` python
class X:
    def m(self): print('X.m')
class Y:
    def m(self): print('Y.m')
class C(X):
    def m(self): super().m()

i = C()
i.m()
C.__bases__ = (Y,)
i.m()
```

②协同多继承方法的分发：当多继承树对多个类的同名函数进行分发时，super 提供一种顺序调用路由的协议；
- 钻石类树模型下：

``` python
class A:
    def __init__(self): print('A.__init__')
class B(A):
    def __init__(self): print('B.__init__'); A.__init__(self)
class C(A):
    def __init__(self): print('C.__init__'); A.__init__(self)
class D(B, C): pass

x = D() # 运行B
```

- 相比之下，如果所有类都使用super，方法调用将按照 MRO 类顺序分发：

``` python
class A:
    def __init__(self): print('A.__init__')
class B(A):
    def __init__(self): print('B.__init__'); super().__init__()
class C(A):
    def __init__(self): print('C.__init__'); super().__init__()
class D(B, C): pass

x = D()
```

- 上面先运行 B ，而 B 中的 super() 是按照 D 的 MRO 顺序（D、B、C、A）来的，所以 B 中的 super() 会运行 C ，再 C 的 super() 运行 A ；
- 故结果是 `B.__init__、C.__init__、A.__init__`：

``` python
print(D.__mro__)
```

- 只要所有类都采用了 super 调用，通过在 MRO 序列下选择下一个类，一个类方法的 super 调用就能在类树上传递调用；
- 总之， super 要么不用，要么全用，最好不用。

5、相同参数限制
- 若方法参数随着类不同而变化时，使用 super 会让程序员很难确定 super 选择的版本（实际上会随着类树而变化）：

``` python
class Employee:
    def __init__(self, name, salary):
        self.name = name
        self.salary = salary
    
class Chef(Employee):
    def __init__(self, name):
        super().__init__(name, 50000)

class Server(Employee):
    def __init__(self, name):
        super().__init__(name, 40000)

bob = Chef('Bob')
sue = Server('Sue')
print(bob.salary, sue.salary)
```

- 上面没问题，因为是单继承树；
- 但如果`class TwoJobs(Chef, Server): pass`，再`tom = TwoJobs('Tom')`，会出错误：`TypeError: __init__() takes 2 positional arguments but 3 were given`。

### 八、Class Gotchas
1、修改类属性
- 所有从类产生的实例都共享这个类的命名空间，所以对类层次的修改都会反映在实例里：

``` python
class X:
    a = 1

I = X()
print(I.a)
print(X.a)
# class语句外修改类属性
X.a = 2
print(I.a)
print(X.a)
```

2、修改可变的类属性，比如列表
- 因为类属性被所有实例共享，如果一个类属性引用一个可变对象，那么任何实例在原位置修改该对象会影响到所有实例和类：

``` python
class C:
    shared = []
    def __init__(self):
        self.perobj = []

x = C()
y = C()
print(y.shared, y.perobj)
x.shared.append('spam')
x.perobj.append('spam')
print(x.shared, x.perobj)
print(y.shared, y.perobj)
print(C.shared)
```

3、方法和类中的作用域
- 类 Spam 是在 generate 函数的局部作用域中赋值的，所以能被内嵌的函数看到，即 LEGB 作用域的 E ：

``` python
def generate():
    class Spam:
        count = 1
        def method(self):
            print(Spam.count)
    return Spam()

generate().method()
```

- 尽管如此， method 方法是看不到外层类的局部作用域， method 方法只看得到外层 def 的局部作用域；
- 这也是为什么方法得通过 self 实例，或类名称来引用外层类定义得方法或属性；
- 即必须使用 self.count 或 Spam.count ，而不是 count ；
- method 方法能够访问：它自己的作用域、外层函数的作用域、外围模块的全局作用域、所有存储在类的 self 实例的数据，以及它的非局部名称的类本身。