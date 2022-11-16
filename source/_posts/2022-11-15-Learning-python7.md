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
> 本篇主要内容为：XXXXXXXXXXXXXXXXXX。

<div  align="center">  
<img src="https://s2.loli.net/2022/09/17/ri9Ue6nguJdq1Ca.jpg" width = "80%" height = "80%" alt="Learning Python"/>
</div>

# PART V Modules and Packages
## chapter 24 Module Packages
### 一、Package Import Basics
1. 除了模块名之外，导入还可以指定目录路径
    - Python代码的目录被称为包package；
    - 包导入是把目录变成Python命名空间，其属性对应目录中所包含的子目录和模块文件。

2. 包导入基础
    - `import dir1.dir2.mod` 或 `from dir1.dir2.mod import x`
    - 上述语句表明了dir1里有子目录dir2，dir2里包含mod.py

3. 包和搜索路径设置
    - import语句中的目录路径只能是以点号间隔的变量，不能import C:\mycode\dir1\dir2\mod。

4. \_\_init\_\_.py包文件
    - 如果选择使用包导入，包导入语句的路径中的*每个目录*都必须有\_\_init\_\_.py这个文件，否则包导入会失败；
    - 也就是说上面dir1和dir2里都必须包含\_\_init\_\_.py文件，而容器目录dir0不需要\_\_init\_\_.py文件，dir0必须在模块搜索路径的sys.path列表中；
    - 即dir0\dir1\dir2\mod.py 对应 import dir1.dir2.mod
    - \_\_init\_\_.py可以包含代码，也可以是空的；
    - 它们的代码将在python第一次导入一个路径的时候被自动运行，所以可以被作为执行包的初始化的钩子。

5. Package initialization file roles包初始化文件的角色
    - \_\_init\_\_.py文件用作包初始化的钩子hook，将目录声明成一个Python包，替目录生成一个模块命名空间以及在目录导入时实现from\* 语句
    - 包的初始化：python在首次导入目录时，会自动执行该目录下\_\_init\_\_.py文件中的代码；
    - 模块使用的声明：包的\_\_init\_\_.py文件即声明一个路径是Python的包；
    - 模块命名空间的初始化：导入表达式dir1.dir2运行后，会返回一个模块对象，该对象的命名空间包含了dir2的\_\_init\_\_.py文件中赋值的所有名称；
    - from\* 语句的行为：可以在\_\_init\_\_.py文件内定义\_\_all\_\_列表来规定目录以from\*语句形式导入什么。\_\_all\_\_列表指包名称使用from\*时，应该导入的子模块的名称清单。如果没有设定\_\_all\_\_，from\*语句不会自动加载嵌套于该目录内的子模块。\_\_init\_\_.py可以是空白，但必须存在。

### 二、Package Import Example
1. 包导入示例
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

2. 包的from语句 vs 包的import语句
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

3. 为什么使用包导入
    - 包导入提供了程序文件的目录信息；
    - 包导入也可以简化PYTHONPATH和.pth文件搜索路径设置；
    - 包能解决多个同名文件安装在同一个机器上所引发的模糊性。

### 三、Package Relative Imports
1. 包相对导入
    - 包内部导入同一个包中的内容时，可以使用和外部导入相同的完整路径语法，也可以利用特殊的包内搜索规则来简化import语句：
    - 对于包中导入：
        - 默认跳过包自己的目录。导入只检查sys.path列表中的搜索路径。称为**绝对导入**：`import XX`；
        - from语句允许显式地要求导入只搜索包的目录（以点号开始）。称为**相对导入**：`from . import XX`。

2. 相对导入基础知识
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

3. 相对导入的实际应用
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

4. 包相对导入的陷阱
    - 不能随意使用from . 的相对导入语法，除非发起导入的文件本身是包的一部分；
    - 导入不会搜索一个包模块自身的路径，除非使用from . 的相对导入语法；
    - 最好能使用完整包路径导入，即`from system.section.mypkg import mod`，来替代包相对导入或简单导入（import mode）
    - 详见书720-724

5. \_\_name\_\_ == "\_\_main\_\_"
    - 当一个模块作为顶级脚本被运行时，它的\_\_name\_\_属性的值是“\_\_main\_\_”字符串，但当它被导入却不是；
    - 即`if __name__ == "__main__"`后的语句在被导入后不被执行。

### 四、Namespace Packages
1. 命名空间包
    - 4种导入模型：
        - **基础模块导入**：`import mod`, `from mod import attr`
        - **包导入**：`import dir1.dir2.mod`, `from dir1.mod import attr`
        - **包相对导入**：`from . import mod` (相对), `import mod` (绝对)
        - **命名空间包**：`import splitdir.mod`：运行包横跨多个目录，不需要__init__.py初始化文件。
    - 两种风格的包：
        - 原始的模型，现在称为**常规包regular packages**；
        - 可选的模型，称为**命名空间包namespace packages**。
    - 命名空间包模型常常被作为一种后备选项进行使用。

2. 命名空间包的语义
    - 常规包必须拥有一个__init__.py文件，而且必须位于一个独立的目录里；
    - 命名空间包可以横跨多个路径，这些路径在被导入时被收集；
    - 所有能够成为一个命名空间包组成部分的目录都不能包含一个__init__.py文件，但是可以被嵌套在里面当作一个单独的包。

3. 命名空间包导入算法
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

4. 对常规包的影响：可选的\_\_init\_\_.py
    - 如果一个单独目录包没有该文件，它将被当作一个单独目录命名空间包，而且不会引发任何警告；
    - 同时，原始的常规包模型仍然支持，而且作为一个初始化钩子会自动运行\_\_init\_\_.py文件中的代码；
    - 而且常规包有性能上的优势。

5. 命名空间包的实际应用
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

6. 命名空间包嵌套
    - 命名空间包支持任意嵌套，并成为低层级的“父路径”；
    - 较低层的组件可以是一个模块、常规包也可以是另一个命名空间包。


## chapter 25 Advanced Module Topics
