---
title: Yarn Spinner 剧情脚本语言
date: 2026-04-07 20:56:48
categories: 
  - [游戏开发, Gameplay]
  - [游戏开发, Unity]
tags:
  - 游戏开发
  - 剧本编写
  - Unity
  - Gameplay
top_img: /images/black.jpg
cover: https://files.seeusercontent.com/2026/04/07/hFz6/YarnSpinner.png
description: XXXXXXXXXXXXXXXXXXXXXX。
---

# 前言
最近一直在研究游戏的对话系统，也调查了一些知名的插件，诸如：[Dialogue System](https://assetstore.unity.com/packages/tools/behavior-ai/dialogue-system-for-unity-11672) 和 [Node Canvas](https://assetstore.unity.com/packages/tools/visual-scripting/nodecanvas-14914)，从本质上来说这两个插件都是在 Unity 编辑器内部运行的可视化**对话树 Dialogue Tree** 编辑器。只是 Dialogue System 功能比较齐全，也支持导入我后面要说的几种对话脚本或工具，但该插件我感觉过于复杂笨重了，而 Node Canvas 相对轻量一点，还有行为树、状态机等功能。这两个插件都会将对话树中的对话内容序列化至 ScriptableObject 的资产文件 .asset 中，Node Canvas 是将对话树序列化为 JSON，以字符串的形式存储在 ScriptableObject 中，Dialogue System 则更复杂一点，存储了更多信息。

上述插件在处理轻量级的游戏剧本时，可以说是完全够用的，但是一旦文本量变大或者分支很多时，维护、修改以及协作都会成为难题。因为往往编剧更希望在 Word 文档或者 Excel 中写剧情，也方便多人协作修改，那么此时大量剧本及分支逻辑就需要程序员在插件中一个一个去添加或者修改，不利于开发效率。让编剧去使用插件更是不太可能，还不能相互协作，更不要说可能还需要额外的席位费（笑）。所以最好的方式就是编写一个独立于游戏引擎的剧情编辑工具，将程序工作和编剧工作解耦，提高效率。对于大的游戏工作室来说，这样做最好，但是小工作室或者独立工作室可能就没有这么多的精力，只能使用其他开发者开发的工具了，我也找到了一些专门针对游戏分支剧情的对话工具，诸如：[Twine](https://twinery.org/)、[ink](https://www.inklestudios.com/ink/)、[Yarn Spinner](https://yarnspinner.dev/) 以及 [articy:draft X](https://www.articy.com/en/)。具体介绍可以查看这篇文章：[游戏分支剧情创作中的挑战和工具](https://www.gcores.com/articles/139398)，写的非常详细，我就不再赘述了。在调查了这些工具后，我觉得 **Yarn Spinner** 是最适合我的想法的，首先它是开源免费的，并且无论是在写作、多人协作、可视化上还是在与 Unity 或其他游戏引擎的集成上，都非常完善，本篇文章主要翻译并记录它的语法，方便以后查看，顺便记录一下以及如何在 Unity 中使用。其实 articy:draft X 我感觉也可以，只不过它要收费，多人协作还需要购买多人许可证和观众许可证。

# Yarn Spinner 脚本语法
Yarn Spinner 可以在网页上进行编辑：https://try.yarnspinner.dev ，提供了文本编辑和播放功能，但是功能没有 VS Code 中强大，Yarn Spinner 为 VS Code 开发了专属插件，为脚本提供了语法高亮，还有一个树状图形视图可以查看，如下图：  

<div align="center">  
<img src="https://files.seeusercontent.com/2026/04/08/iiH8/YarnSpinner01_InVSCode.png" width = "75%" height = "75%" alt="图1 - VS Code editing a Yarn script"/>
</div>

插件下载安装直接在 VS Code Extensions 中搜索 Yarn Spinner 就行。每个 Yarn Spinner 项目都需要一个 <kbd>.yarnproject</kbd> 文件，可以通过 Command Palette（通过 <kbd>Ctrl+Shift+P</kbd> 打开）的 Yarn Spinner: Create New Yarn File 命令创建。其他关于 Yarn Spinner 插件功能的介绍我就不赘述了，详见[官方文档](https://docs.yarnspinner.dev/)。这里就额外提一下，在 VS Code 可以通过微软官方的 Live Share 插件进行协作。

## 基础语法
### Node
Yarn Spinner 脚本由**节点 nodes** 组成，一个 node 包含一组**抬头 headers** 和**主体 body**，其中一组抬头至少要包含**标题 title**。这个 title 比较重要，是 node 的名字，游戏程序需要使用 title 进入一段对话，或者跳跃到另外一段对话，title 不会向玩家展示。而主体 body 则包含文本对话，基本单位是**行 line**，node 会一行一行地将文本数据传输给游戏，其中行开头在冒号前（冒号前无空格）的内容会被标记为角色名，角色名信息也会传输给游戏，当然行也可以没有角色名，如下：  

<div style="background-color: #f2faf4; margin-bottom: 1em; padding: 10px;">
title: Start<br>
---<br>
This is a line of dialogue, without a character name.<br>
Speaker: This is another line of dialogue said by a character called "Speaker".<br>
===<br>
</div>

其中 headers 可以包含多行 <kbd>key: value</kbd> 结构，`---` 标识表示主体 body 开始，`===` 标识表示节点 node 结束。

<div style="background-color: #fff5ed; margin-bottom: 1em; padding: 10px;">
<span style="color: #ff9900;">Warning：</span> title 不支持数字开头，也不支持空格和 <kbd>.</kbd>。<kbd>First Node</kbd>、<kbd>First.Node</kbd>、<kbd>1stNode</kbd> 都是非法的。
</div>

### 编辑器 Header
除了 title 这个 header 外，还有一些 header 是用于在编辑器的树状图形视图 Graph View 中改变 node 的显示效果的，有如下几个 header：  

**①Color**：支持的颜色有 <kbd>red</kbd>, <kbd>green</kbd>, <kbd>blue</kbd>, <kbd>orange</kbd>, <kbd>yellow</kbd>, <kbd>purple</kbd>，也支持十六进制颜色码 Hex，如下图：  

<div align="center">  
<img src="https://files.seeusercontent.com/2026/04/08/Hqo1/YarnSpinner02_ColorHeader.png" width = "40%" height = "40%" alt="图2 - Color Header"/>
</div>

**②Position**：决定了当前 node 在 Graph View 中的位置。

**③Cluster/Group**：可以将多个 node 作为一个分组，所有使用同一个 cluster/group 值的 node 都会在编辑器中被打包成组，<kbd>group:</kbd> 和 <kbd>cluster:</kbd> 好像都支持：  

<div align="center">  
<img src="https://files.seeusercontent.com/2026/04/08/rh4P/YarnSpinner03_ClusterHeader.png" width = "55%" height = "55%" alt="图3 - Cluster Header"/>
</div>

**④Image**：用于在 node 中展示一张图片，图片路径默认就是当前文件的路径，可以在 VS Code 中点击 <kbd>.yarnproject</kbd> 文件，在 Project MetaData 中设置 Image Path。

<div align="center">  
<img src="https://files.seeusercontent.com/2026/04/08/Q0sy/YarnSpinner04_ImageHeader.png" width = "35%" height = "35%" alt="图4 - Image Header"/>
</div>

**⑤Note**：除了常规的 node 外，可以添加 <kbd>style: note</kbd> 将 node 改造为 note 标签，用于提示：  

<div align="center">  
<img src="https://files.seeusercontent.com/2026/04/08/ev6I/YarnSpinner05_NoteHeader.png" width = "50%" height = "50%" alt="图5 - Note Header"/>
</div>

## 进阶语法
### Options
**option** 是为了给对话提供选项让玩家进行选择，任意前缀带有 <kbd>-></kbd> 的行 line，都视作 option。option 还支持嵌套，并且 option 可以有属于它们自己的行，只有选择了该选项才会出现，<u>属于 option 的行需要进行缩进</u>，示例如下：  

<div align="center">  
<img src="https://files.seeusercontent.com/2026/04/09/sF8f/YarnSpinner06_Option.gif" width = "65%" height = "65%" alt="图6 - Option 演示"/>
</div>

### Jump
跳转命令 <kbd>&lt;&lt;jump title&gt;&gt;</kbd> 用于从一个 node 跳转到另外一个 node，注意该命令中 jump 与要跳转的 title 中间有个空格。跳转命令还可以跳转到另外一个 <kbd>.yarn</kbd> 文件里的 node（注意 node 的 title 在项目全局内是不能重复的）。跳转命令的好处就是可以减少 option 的相互嵌套，示例如下：  

<div align="center">  
<img src="https://files.seeusercontent.com/2026/04/09/p6Hh/YarnSpinner07_Jump.gif" width = "65%" height = "65%" alt="图7 - Jump 演示"/>
</div>

### Detour
巡回命令 <kbd>&lt;&lt;detour title&gt;&gt;</kbd> 跟跳转命令很类似，只不过跳转命令跳转到其他 node 不会回来了，而巡回命令会回到调用该命令的位置。触发回去有两种情形，一种是到达了 node 的底部，一种是到达了 <kbd>&lt;&lt;return&gt;&gt;</kbd> 命令，示例如下：  

<div align="center">  
<img src="https://files.seeusercontent.com/2026/04/09/fDo4/YarnSpinner08_Detour.gif" width = "65%" height = "65%" alt="图8 - Detour 演示"/>
</div>

### Variables
很多时候，游戏对话需要根据玩家之前的选择抑或是之前的行为进行一定的调整，这就需要使用**变量 variables** 去控制对话，改变需要呈现的选项或者选项下内容。Yarn Spinner 脚本支持三种变量：**数字 numbers**、**字符串 strings** 和**布尔值 booleans**，声明变量的格式如下（变量名前有个 <kbd>$</kbd> 符号）：  

<div style="background-color: #f2faf4; margin-bottom: 1em; padding: 10px;">
&lt;&lt;declare $playerName = "Reginald the Wizard"&gt;&gt;<br>
&lt;&lt;declare $gold = 42&gt;&gt;<br>
&lt;&lt;declare $doorUnlocked = false&gt;&gt;<br>
</div>

变量还支持隐式声明：即直接使用一个变量，Yarn Spinner 会猜测该变量的类型，并设置为默认值，对于数字默认值为 0，对于字符串默认值为空字符串，对于布尔值默认值为 false，但 Yarn Spinner 还是建议声明变量，并且建议在一个特殊的节点中（比如 title 为 Setup 的节点）声明所有变量，便于项目管理。变量赋值的格式如下：  

<div style="background-color: #f2faf4; margin-bottom: 1em; padding: 10px;">
&lt;&lt;set $myCoolNumber to 8&gt;&gt;<br>
&lt;&lt;set $myFantasticString to "incredible!"&gt;&gt;<br>
</div>

当然变量还支持**表达式 expression**，比如字符串拼接，数学运算以及逻辑运算符：  

<div style="background-color: #f2faf4; margin-bottom: 1em; padding: 10px;">
&lt;&lt;set $numberOfSidesInATriangle = 2 + 1&gt;&gt;<br>
&lt;&lt;set $numberOfSidesInASquare = $numberOfSidesInATriangle + 1&gt;&gt;<br>
&lt;&lt;set $aString = "A" + "B"&gt;&gt;<br>
</div>

支持的**运算符 operators** 有如下，首先是**逻辑运算符 logical operators**：  

- Equality: <kbd>eq</kbd> 或 <kbd>is</kbd> 或 <kbd>==</kbd>
- Inequality: <kbd>neq</kbd> 或 <kbd>!</kbd>
- Greater than: <kbd>gt</kbd> 或 <kbd>&gt;</kbd>
- Less than: <kbd>lt</kbd> 或 <kbd>&lt;</kbd>
- Less than or equal to: <kbd>lte</kbd> 或 <kbd>&lt;=</kbd>
- Greater than or equal to: <kbd>gte</kbd> 或 <kbd>&gt;=</kbd>
- Boolean 'or': <kbd>or</kbd> 或 <kbd>||</kbd>
- Boolean 'xor': <kbd>xor</kbd> 或 <kbd>^</kbd>
- Boolean 'not': <kbd>not</kbd> 或 <kbd>!</kbd>
- Boolean 'and': <kbd>and</kbd> 或 <kbd>&&</kbd>

**数学运算符 maths operators** 如下：  

- Addition: <kbd>+</kbd>
- Subtraction: <kbd>-</kbd>
- Multiplication: <kbd>*</kbd>
- Division: <kbd>/</kbd>
- Truncating Remainder Division: <kbd>%</kbd>
- Brackets：<kbd>(</kbd><kbd>)</kbd>

<div style="background-color: #edf7fa; margin-bottom: 1em; padding: 10px;">
<span style="color: #316f8f;">NOTE：</span>实际上我在使用时，发现 <kbd>+=</kbd> 、<kbd>-=</kbd>、<kbd>*=</kbd>、<kbd>/=</kbd> 也都是支持的。
</div>

Yarn Spinner 还支持变量类型的**转换 Conversion**，有三个对应的内置函数：<kbd>string()</kbd>、<kbd>number()</kbd>、<kbd>bool()</kbd>，字符串的示例如下：  

<div style="background-color: #f2faf4; margin-bottom: 1em; padding: 10px;">
&lt;&lt;declare $aNumber = 42&gt;&gt;<br>
&lt;&lt;declare $aString = "This is my string."&gt;&gt;<br>
&lt;&lt;set $aString = string($aNumber)&gt;&gt;<br>
</div>

在行内使用变量，就是使用大括号 <kbd>{</kbd> <kbd>}</kbd>：  

<div style="background-color: #f2faf4; margin-bottom: 1em; padding: 10px;">
&lt;&lt;set $variableName to "a string value"&gt;&gt;<br>
The value of variableName is {$variableName}.<br>
</div>

### Flow Control
Yarn Spinner 支持两种控制流：if statements 和 conditional options。

**①if statements**  
支持 if、elseif、else，基本上所有语言都一个样，格式如下：  

<div style="background-color: #f2faf4; margin-bottom: 1em; padding: 10px;">
&lt;&lt;if $gold_amount &gt;= 5 and $reputation &gt;= 8&gt;&gt;<br>
&emsp;&emsp;Merchant: You're rich enough and popular enough for me to serve you!<br>
&lt;&lt;elseif $gold_amount &gt;= 10 or $reputation &gt;= 10&gt;&gt;<br>
&emsp;&emsp;Merchant: I wouldn't normally, but I'll serve you!<br>
&lt;&lt;else&gt;&gt;<br>
&emsp;&emsp;Merchant: You're neither rich enough nor important enough for me to serve!<br>
&lt;&lt;endif&gt;&gt;<br>
</div>

**②conditional options**  
这个比较特殊，用于控制是否向玩家呈现某个特殊的选项。但是所有的选项都会被 Yarn Spinner 传递到游戏当中，决定是否呈现是游戏内部的事情，因为有时候我们会需要这个未被触发的选项呈现给玩家，但是处于无法选择的状态。

<div style="background-color: #f2faf4; margin-bottom: 1em; padding: 10px;">
Guard: You're not allowed in!<br>
-> Sure I am! The boss knows me! &lt;&lt;if $reputation &gt; 10&gt;&gt;<br>
-> Please?<br>
-> I'll come back later.<br>
</div>

示例如下：  

<div align="center">  
<img src="https://files.seeusercontent.com/2026/04/09/hU8x/YarnSpinner09_ControlFlow.gif" width = "65%" height = "65%" alt="图9 - Flow Control 演示"/>
</div>

### Once
Once 主要用于只触发一次的对话。比如主角第一次遇到铁匠，铁匠就会吹嘘一通自己的手艺，但之后再找铁匠，就不会再次触发吹嘘的对话了。Once 有两种使用方式：一种是将多个行包裹在 <kbd>&lt;&lt;once&gt;&gt;</kbd> 和 <kbd>&lt;&lt;endonce&gt;&gt;</kbd> 当中，还支持使用 <kbd>&lt;&lt;else&gt;&gt;</kbd> 分句，以及内嵌条件 <kbd>if</kbd>，如下：  

<div style="background-color: #f2faf4; margin-bottom: 1em; padding: 10px;">
&lt;&lt;once if $player_is_adventurer&gt;&gt;<br>
&emsp;&emsp;Guard: I used to be an adventurer like you, but then I took an arrow in the knee.<br>
&lt;&lt;else&gt;&gt;<br>
&emsp;&emsp;Guard: Greetings.<br>
&lt;&lt;endonce&gt;&gt;<br>
</div>

另外一种用法就是在行 line 或者选项 option 的最后添加 <kbd>once</kbd> 或者 <kbd>once if</kbd>。若添加了，行只会显示一次，而选项只能被选择一次：  

<div style="background-color: #f2faf4; margin-bottom: 1em; padding: 10px;">
Guard: Who are you? &lt;&lt;once&gt;&gt;<br>
-> Where can I park my horse? &lt;&lt;once if $has_horse&gt;&gt;<br>
&emsp;&emsp;Guard: Over by the tavern.<br>
</div>

<div style="background-color: #edf7fa; margin-bottom: 1em; padding: 10px;">
<span style="color: #316f8f;">NOTE：</span><kbd>once</kbd> 会以 variable 的形式存储在 Dialogue Runner’s Variable Storage 里，跟其他 variable 一样。只不过在 Yarn Spinner 脚本中无法获取到这些变量。Dialogue Runner’s Variable Storage 后面讲代码时会详细说明。
</div>

***TODO：添加演示***

### Smart Variables
这玩意其实就是 C# 的属性，声明方式就是比变量多了一些表达式，比如：  

<div style="background-color: #f2faf4; margin-bottom: 1em; padding: 10px;">
&lt;&lt;declare $has_enough_materials = $wood &gt;= 5 && $nails &gt;= 10&gt;&gt;<br>
&lt;&lt;declare $has_required_skill = $carpentry_level &gt;= 3&gt;&gt;<br>
&lt;&lt;declare $can_build_chair = $has_enough_materials && $has_required_skill&gt;&gt;<br>
</div>

### Enums
枚举也不必多说了，声明方式如下：  

<div style="background-color: #f2faf4; margin-bottom: 1em; padding: 10px;">
&lt;&lt;enum Food&gt;&gt;<br>
&emsp;&emsp;&lt;&lt;case Apple&gt;&gt;<br>
&emsp;&emsp;&lt;&lt;case Orange&gt;&gt;<br>
&emsp;&emsp;&lt;&lt;case Pear&gt;&gt;<br>
&lt;&lt;endenum&gt;&gt;<br>
<br>
&lt;&lt;declare $favouriteFood = Food.Apple&gt;&gt;<br>
&lt;&lt;if $favouriteFood == Food.Apple&gt;&gt;<br>
&emsp;&emsp;I love apples!<br>
&lt;&lt;endif&gt;&gt;<br>
<br>
// You can even skip the name of the enum if Yarn Spinner can <br>
// figure it out from context!<br>
&lt;&lt;set $favouriteFood = .Pear&gt;&gt;<br>
</div>

### Line Groups
line group 是让 Yarn Spinner 随机选择的 option，如下：  

<div style="background-color: #f2faf4; margin-bottom: 1em; padding: 10px;">
=> Guard: Halt!<br>
=> Guard: No entry!<br>
=> Guard: Stop!<br>
=> Guard: Hail, adventurer! &lt;&lt;if $player_is_adventurer&gt;&gt;<br>
</div>

每次触发这段对话，Yarn Spinner 都会为你随机选择一条播放。

### Commands
**commands** 是在游戏内使用的特殊命令，跟行和选项一样，会被传递给游戏内的 Dialogue Runner。内置的 commands 有两个，<kbd>wait</kbd> 和 <kbd>stop</kbd>，其中 wait 可以暂停对话几秒，而 stop 可以停止对话，相当于到达了对话的底部：

<div style="background-color: #f2faf4; margin-bottom: 1em; padding: 10px;">
// Wait for 2 seconds<br>
&lt;&lt;wait 2&gt;&gt;<br>
<br>
// Leave the dialogue now<br>
&lt;&lt;stop&gt;&gt;<br>
<br>
// Leave the dialogue if we don't have enough money<br>
&lt;&lt;if $money &lt; 50&gt;&gt;<br>
&emsp;&emsp;Shopkeeper: You can't afford my pies!<br>
&emsp;&emsp;&lt;&lt;stop&gt;&gt;<br>
&lt;&lt;endif&gt;&gt;<br>
</div>

还可以通过 C# 代码创建自己的 command，

***TODO：之后回来再补充***

### Functions
**function** 就是带返回值的 command，function 可以让我们从游戏中获得一定的数据，又或者说生成一定的随机值：

<div style="background-color: #f2faf4; margin-bottom: 1em; padding: 10px;">
// Inside an if statement:<br>
&lt;&lt;if dice(6) == 6&gt;&gt;<br>
&emsp;&emsp;You rolled a six!<br>
&lt;&lt;endif&gt;&gt;<br>
<br>
// Inside a line:<br>
Gambler: My lucky number is {random_range(1,10)}!<br>
</div>

内置的 function 有如下：  

| Built-In Functions | 作用 |
| :----: | :----: |
| `visited(string node_name)` | 返回 title 为 node_name 的 node 是否至少进入出去一次 |
| `visited_count(string node_name)` | 返回进入出去 title 为 node_name 的 node 的次数 |
| `random()` | 返回 0 - 1 的随机值 |
| `random_range(number a, number b)` | 返回 a - b 的随机值 |
| `dice(number sides)` | 返回 1 - side 的随机整数 |
| `min(number a, number b)` | 返回最小值 |
| `max(number a, number b)` | 返回最大值 |
| `round(number n)` | 返回最近的整数 |
| `round_places(number n, number places)` | 将 n 四舍五入到保留指定小数位数（places）的最近数字 |
| `floor(number n)` | 返回‌小于或等于 n 的最大整数 |
| `ceil(number n)` | 返回大于或等于 n 的最小整数 |
| `inc(number n)` | 向上取整，如果已经是整数则加 1，inc 是 increment 的缩写 |
| `dec(number n)` | 向下取整，如果已经是整数则减 1，dec 是 decrement 的缩写 |
| `decimal(number n)` | 返回小数部分 |
| `int(number n)` | 返回整数部分 |

跟 command 一样，也可以通过 C# 代码创建自己的 function，

***TODO：之后回来再补充***

## 高级语法
### Node Groups
**node group** 跟 line group 类似，line group 自动选择的是 line，而 node group 自动选择 node。要创建 node group，需要将多个 node 设置为同一个名字，并且至少包含一个 <kbd>when:</kbd> header，注意是 header，如下：  

<div style="background-color: #f2faf4; margin-bottom: 1em; padding: 10px;">
title: Guard<br>
when: once<br>
---<br>
Guard: You there, traveller!<br>
Player: Who, me?<br>
Guard: Yes! Stay off the roads after dark!<br>
===<br>
title: Guard<br>
when: always<br>
---<br>
Guard: I hear the king has a new advisor.<br>
===<br>
title: Guard<br>
when: $has_sword<br>
---<br>
Guard: No weapons allowed in the city!<br>
===<br>
</div>

### Storylets & Saliency


# Yarn Spinner in Unity
https://github.com/YarnSpinnerTool/YarnSpinner-Unity.git#current