---
title: Yarn Spinner 剧情脚本语言（待删除）
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
description: 【待删除文章】本文章记录了如何使用 Yarn Spinner 脚本进行叙事文本创作，包括基础至高级语法。
---

> Yarn Spinner 官网为 https://yarnspinner.dev/ ，本文章发表时版本为 3.2。

# 前言
最近一直在研究游戏的对话系统，也调查了一些知名的插件，诸如：[Dialogue System](https://assetstore.unity.com/packages/tools/behavior-ai/dialogue-system-for-unity-11672) 和 [Node Canvas](https://assetstore.unity.com/packages/tools/visual-scripting/nodecanvas-14914)，从本质上来说这两个插件都是在 Unity 编辑器内部运行的可视化**对话树 Dialogue Tree** 编辑器。只是 Dialogue System 功能比较齐全，也支持导入我后面要说的几种对话脚本或工具，但该插件我感觉过于复杂笨重了，而 Node Canvas 相对轻量一点，还有行为树、状态机等功能。这两个插件都会将对话树中的对话内容序列化至 ScriptableObject 的资产文件 .asset 中，Node Canvas 是将对话树序列化为 JSON，以字符串的形式存储在 ScriptableObject 中，Dialogue System 则更复杂一点，存储了更多信息。

上述插件在处理轻量级的游戏剧本时，可以说是完全够用的，但是一旦文本量变大或者分支很多时，维护、修改以及协作都会成为难题。因为往往编剧更希望在 Word 文档或者 Excel 中写剧情，也方便多人协作修改，那么此时大量剧本及分支逻辑就需要程序员在插件中一个一个去添加或者修改，不利于开发效率。让编剧去使用插件更是不太可能，还不能相互协作，更不要说可能还需要额外的席位费（笑）。所以最好的方式就是编写一个独立于游戏引擎的剧情编辑工具，将程序工作和编剧工作解耦，提高效率。对于大的游戏工作室来说，这样做最好，但是小工作室或者独立工作室可能就没有这么多的精力，只能使用其他开发者开发的工具了，我也找到了一些专门针对游戏分支剧情的对话工具，诸如：[Twine](https://twinery.org/)、[ink](https://www.inklestudios.com/ink/)、[Yarn Spinner](https://yarnspinner.dev/) 以及 [articy:draft X](https://www.articy.com/en/)。具体介绍可以查看这篇文章：[游戏分支剧情创作中的挑战和工具](https://www.gcores.com/articles/139398)，写的非常详细，我就不再赘述了。在调查了这些工具后，我觉得 **Yarn Spinner** 是最适合我的想法的，首先它是开源免费的（MIT License），并且无论是在写作、多人协作、可视化上还是在与 Unity 或其他游戏引擎的集成上，都非常完善，本篇文章主要翻译并记录它的语法，方便以后查看。其实 articy:draft X 我感觉也可以，只不过它要收费，多人协作还需要购买多人许可证和观众许可证。

> 我后来发现 articy:draft X 有免费商用版，只不过有一定限制，详见 https://www.articy.com/en/articydraft/free/ 。

# Yarn Spinner 编辑
Yarn Spinner 可以在网页上进行编辑：https://try.yarnspinner.dev ，提供了文本编辑和播放功能，但是功能没有 VS Code 中强大，Yarn Spinner 为 VS Code 开发了专属插件，为脚本提供了语法高亮，还有一个树状图形视图可以查看，如下图：  

<div align="center">  
<img src="https://files.seeusercontent.com/2026/04/08/iiH8/YarnSpinner01_InVSCode.png" width = "75%" height = "75%" alt="图1 - VS Code editing a Yarn script"/>
</div>

插件下载安装直接在 VS Code Extensions 中搜索 Yarn Spinner 就行。每个 Yarn Spinner 项目都需要一个 <kbd>.yarnproject</kbd> 文件，可以通过 Command Palette（通过 <kbd>Ctrl+Shift+P</kbd> 打开）的 Yarn Spinner: Create New Yarn File 命令创建。其他关于 Yarn Spinner 插件功能的介绍我就不赘述了，详见[官方文档](https://docs.yarnspinner.dev/)。这里额外提一下，在 VS Code 可以通过微软官方的 Live Share 插件进行协作。

# 基础语法
## Node
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

## 编辑器 Header
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

## Options
**option** 是为了给对话提供选项让玩家进行选择，任意前缀带有 <kbd>-></kbd> 的行 line，都视作 option。option 还支持嵌套，并且 option 可以有属于它们自己的行，只有选择了该选项才会出现，<u>属于 option 的行需要进行缩进</u>，示例如下：  

<div align="center">  
<img src="https://files.seeusercontent.com/2026/04/09/sF8f/YarnSpinner06_Option.gif" width = "65%" height = "65%" alt="图6 - Option 演示"/>
</div>

## Jump
跳转命令 <kbd>&lt;&lt;jump title&gt;&gt;</kbd> 用于从一个 node 跳转到另外一个 node，注意该命令中 jump 与要跳转的 title 中间有个空格。跳转命令还可以跳转到另外一个 <kbd>.yarn</kbd> 文件里的 node（注意 node 的 title 在项目全局内是不能重复的）。跳转命令的好处就是可以减少 option 的相互嵌套，示例如下：  

<div align="center">  
<img src="https://files.seeusercontent.com/2026/04/09/p6Hh/YarnSpinner07_Jump.gif" width = "65%" height = "65%" alt="图7 - Jump 演示"/>
</div>

## Detour
巡回命令 <kbd>&lt;&lt;detour title&gt;&gt;</kbd> 跟跳转命令很类似，只不过跳转命令跳转到其他 node 不会回来了，而巡回命令会回到调用该命令的位置。触发回去有两种情形，一种是到达了 node 的底部，一种是到达了 <kbd>&lt;&lt;return&gt;&gt;</kbd> 命令，示例如下：  

<div align="center">  
<img src="https://files.seeusercontent.com/2026/04/09/fDo4/YarnSpinner08_Detour.gif" width = "65%" height = "65%" alt="图8 - Detour 演示"/>
</div>

# 进阶语法

## Variables
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

## Flow Control
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

## Once
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

<div align="center">  
<img src="https://files.seeusercontent.com/2026/04/10/9eWu/YarnSpinner10_Once.gif" width = "65%" height = "65%" alt="图10 - Once 演示"/>
</div>

## Smart Variables
这玩意其实就是 C# 的属性，声明方式就是比变量多了一些表达式，比如：  

<div style="background-color: #f2faf4; margin-bottom: 1em; padding: 10px;">
&lt;&lt;declare $has_enough_materials = $wood &gt;= 5 && $nails &gt;= 10&gt;&gt;<br>
&lt;&lt;declare $has_required_skill = $carpentry_level &gt;= 3&gt;&gt;<br>
&lt;&lt;declare $can_build_chair = $has_enough_materials && $has_required_skill&gt;&gt;<br>
</div>

## Enums
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

# 特殊语法
## Commands
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

除了内置 command 外，还支持创建自己的 command。在 Unity 中有两种方式可以添加自定义 command，一种是通过给方法添加 <kbd>YarnCommand</kbd> 特性，另一种是调用 DialogueRunner 的 <kbd>AddCommandHandler</kbd> 方法。

<div style="background-color: #fff5ed; margin-bottom: 1em; padding: 10px;">
<span style="color: #ff9900;">Warning：</span> 我强烈不推荐使用自定义 command，我看了一下源码，自定义 command 的 GC 问题挺严重的，会频繁进行字符串操作、 new list&lt;T&gt; 或者 array。将静态方法作为 command 会比将 MonoBehaviour 方法作为 command 相对好一点，但仍然存在该问题。若还是需要自定义，建议不要使用 YarnCommand 特性，而是通过 AddCommandHandler 手动注册，首先 YarnCommand 特性本质上也是通过 AddCommandHandler 注册的，其次 YarnCommand 的实现是基于反射的，有一定的性能开销，虽然注册所有 command 方法只发生在游戏 startup 阶段。
</div>

### \[YarnCommand\]
YarnCommand 特性可以添加到继承 MonoBehaviour 的类的方法中，也可以添加到纯 C# 类的静态方法中。先说继承 MonoBehaviour 的方式：  

``` C#
public class CharacterMovement : MonoBehaviour 
{
    [YarnCommand("leap")]
    public void Leap() 
    {
        Debug.Log($"{name} is leaping!");
    }
}
```

注意：如果 CharacterMovement.cs 拖拽到了场景中的 MyCharacter 下面，那么在 Yarn Spinner 的文本中调用该 command 时，<u>还需要额外指定 gameobject 的名字</u>，即 MyCharacter，以便让 DialogueRunner 知道哪个 gameobject 需要使用该 command，如下：  

<div style="background-color: #f2faf4; margin-bottom: 1em; padding: 10px;">
&lt;&lt;leap MyCharacter&gt;&gt;&emsp;&emsp;// will print "MyCharacter is leaping!" in the console<br>
</div>

纯 C# 类中的方法要添加 YarnCommand 特性，必须是静态方法：  

``` C#
public class FadeCamera 
{
    [YarnCommand("fade_camera")]
    public static void FadeCamera() 
    {
        Debug.Log("Fading the camera!");
    }
}
```

Yarn Spinner 脚本中调用无需提供 gameobject 的名字：  

<div style="background-color: #f2faf4; margin-bottom: 1em; padding: 10px;">
&lt;&lt;fade_camera&gt;&gt;&emsp;&emsp;// will print "Fading the camera!" in the console<br>
</div>

带有 YarnCommand 特性的方法支持 6 种参数：<kbd>string</kbd>、<kbd>int</kbd>、<kbd>float</kbd>、<kbd>bool</kbd>、<kbd>GameObject</kbd>、<kbd>Component</kbd>（或者继承 Component 的类），对于 GameObject 和 Component，Yarn Spinner 会查找所有激活的场景下的所有 GameObject 是否跟传递进来的名字匹配（通过 `GameObject.Find()`），若匹配，该 GameObject 就会被传递为 YarnCommand 方法的参数。示例如下：  

``` C#
[YarnCommand("walk")]
public void Walk(GameObject destination, bool dancing = false) 
{
    var position = destination.transform.position;
    if (dancing) 
    {
        // set animation to a dance
    }
    else 
    {
        // set animation to a regular walk
    }
    // walk the character to 'position'
}
```

Yarn Spinner 脚本中调用该 command 的方式如下：  

<div style="background-color: #f2faf4; margin-bottom: 1em; padding: 10px;">
&lt;&lt;walk MyCharacter StageLeft&gt;&gt;&emsp;&emsp;// walk to the position of the object named 'StageLeft'<br>
&lt;&lt;walk MyOtherCharacter StageRight dancing&gt;&gt;<br>
</div>

<div style="background-color: #edf7fa; margin-bottom: 1em; padding: 10px;">
<span style="color: #316f8f;">NOTE：</span>Unity 中，可以通过打开 Window -> Yarn Spinner -> Commands 看到你注册的所有 command。
</div>

### AddCommandHandler
使用 AddCommandHandler API 的方法如下：  

``` C#
public class CustomCommands : MonoBehaviour 
{
    // Drag and drop your Dialogue Runner into this variable.
    public DialogueRunner dialogueRunner;

    public void Awake() 
    {
        // Create a new command called 'camera_look', which looks at a target. 
        // Note how we're listing 'GameObject' as the parameter type.
        dialogueRunner.AddCommandHandler<GameObject>(
            "camera_look",     // the name of the command
            CameraLookAtTarget // the method to run
        );
    }

    // The method that gets called when '<<camera_look>>' is run.
    private void CameraLookAtTarget(GameObject target) 
    {
        if (target == null) debug.Log("Can't find the target!");
        // Make the main camera look at this target
        Camera.main.transform.LookAt(target.transform);
    }    
}
```

上述 Command 在 Yarn Spinner 脚本中的调用方式如下：  

<div style="background-color: #f2faf4; margin-bottom: 1em; padding: 10px;">
&lt;&lt;camera_look LeftMarker&gt;&gt;&emsp;&emsp;// make the camera look at an object named LeftMarker<br>
</div>

### Async Command
Command 还支持 Unity 的协程：

``` C#
public class CustomWaitCommand : MonoBehaviour 
{
    [YarnCommand("custom_wait")]
    static IEnumerator CustomWait() 
    {
        // Wait for 1 second
        yield return new WaitForSeconds(1.0);
        
        // Because this method returns IEnumerator, it's a coroutine. 
        // Yarn Spinner will wait until onComplete is called.
    }    
}
```

除了协程也支持 Unity Awaitables、Tasks 或者 UniTasks。Yarn Spinner 提供了一个 awaitable 类叫做 YarnTask，当项目中安装了 UniTask，默认使用 [UniTask](https://github.com/Cysharp/UniTask) ，若没安装则使用 [Unity Awaitables](https://docs.unity3d.com/Manual/async-await-support.html)。

``` C#
using Yarn.Unity;

public class CustomWaitCommand : MonoBehaviour 
{
    [YarnCommand("custom_wait")]
    static async YarnTask CustomWait() 
    {
        // Wait for 1 second
        await YarnTask.Delay(1000);

        // Yarn Spinner will wait until this method
        // returns, before continuing the dialogue.
    }    
}
```

## Functions
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

跟 command 一样，也可以通过 C# 代码创建自己的 function，也有两种方式，一种是 <kbd>YarnFunction</kbd> 特性，另一个是 DialogueRunner 的 <kbd>AddFunction</kbd> 方法。与 command 不同的是，function 只支持注册静态方法，并且只支持 string、int、float、bool 这几个参数。

### \[YarnFunction\]
示例如下：  

``` C#
public class AdderFunction
{
   [YarnFunction("add_numbers")]
   public static int AddNumbers(int first, int second)
   {
       return first + second;
   }
}
```

上述 function 示例在 Yarn Spinner 脚本中的调用方式如下：

<div style="background-color: #f2faf4; margin-bottom: 1em; padding: 10px;">
&lt;&lt;if add_numbers(1,1) == 2&gt;&gt;<br>
&emsp;&emsp;One plus one is {add_numbers(1, 1)}<br>
&lt;&lt;endif&gt;&gt;<br>
</div>

# 高级语法
在进入高级语法之前，先要介绍一下不同的游戏叙事，游戏叙事可以分为线性叙事、分支叙事以及**故事块 Storylets** 叙事，如下图：  

<div align="center">  
<img src="https://files.seeusercontent.com/2026/04/10/I9rb/YarnSpinner11_Narratives.png" width = "45%" height = "45%" alt="图11 - Storytelling Narratives"/>
</div>

故事块叙事就是上图中的第三种，它的故事块与故事块之间没有一个绝对的连接，任意故事块可以跳转至任意故事块，连接是动态的。那么如何连接故事块？这就牵扯到了**显著性 Saliency‌** 机制。显著性决定了当前时间点哪些故事块跟当前情形更加相关，比如，当前你在卧室，那么跟卧室相关的故事块的显著性更高。说白了，故事块叙事强调一种<u>随机性</u>，只不过随着故事的发展，某一些或一类故事块的触发概率更高（即显著性更高）。

显著性的**颗粒度 Granularity** 可以分为 Collection 级别、Line 级别和 Sub-Line 级别，为此 Yarn Spinner 提供了两种语法：**Line Groups** 和 **Node Groups**。

## Line Groups
**line group** 是让 Yarn Spinner 随机选择的 option，如下：  

<div style="background-color: #f2faf4; margin-bottom: 1em; padding: 10px;">
=> Alice: Yep.<br>
=> Alice: Of course it is!<br>
=> Alice: Why would you think otherwise?<br>
</div>

每次触发这段对话，Yarn Spinner 都会为你随机选择一条播放。但这样并没有显著性的概念，我们可以为 line group 添加条件，如下：  

<div style="background-color: #f2faf4; margin-bottom: 1em; padding: 10px;">
=> Alice: Yep.<br>
=> Alice: Of course it is! &lt;&lt;if $barry_suspicion &gt; 3&gt;&gt;<br>
=> Alice: Why would you think otherwise? &lt;&lt;if $barry_suspicion &gt; 5&gt;&gt;<br>
=> Alice: I am not talking without my lawyer &lt;&lt;if $barry_suspicion &gt; 5 && $knows_barry_is_cop&gt;&gt;<br>
</div>

假设高 suspicion 并且知道是个警察，对话就更有可能呈现最后一句，最后一句是显著性最高的，但是其他选项也满足条件。

## Node Groups
**node group** 跟 line group 类似，line group 自动选择的是 line，而 node group 自动选择 node。要创建 node group，需要将多个 node 设置为同一个名字，并且<u>至少</u>包含一个 <kbd>when:</kbd> header，注意是 header，如下：  

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
when: $suspicion &gt; 5<br>
---<br>
Guard: No weapons allowed in the city!<br>
===<br>
</div>

## Saliency Strategies
Yarn Spinner 内置了四种**显著性策略 Saliency Strategies**：①**First**；②**Best**；③**Best Least Recently Viewed**；**Random Best Least Recently Viewed**。这些策略使用了一个叫做 **complexity** 的概念，如果 line 或 node 的条件是 always，complexity 为 0；如果条件包含 once，则 complexity + 1；如果条件包含其他条件判断，每有一个表达式再 + 1。拿 <kbd>when:</kbd> header 举例：  

- <kbd>when: always</kbd> 的 complexity 为 0；
- <kbd>when: $a</kbd> 的 complexity 为 1；
- <kbd>when: $a or $b</kbd> 的 complexity 为 2；
- <kbd>when: once</kbd> 的 complexity 为 1；
- <kbd>when: once if $a or $b</kbd> 的 complexity 为 3；

**First** 策略就是选择符合条件的第一个 line 或 node，根据其在文件中的顺序；**Best** 策略就是选择符合条件的 line 或 node 中 complexity 最大的一个；**Best Least Recently Viewed** 策略会将所有符合条件的根据选择的次数排序，若次数一致则根据 complexity 排序，若最优解有多个，则选择第一个；**Random Best Least Recently Viewed** 这个策略的排序跟上面那个一样，就是最优解有多个时，随机选择。

要修改显著性策略，需要点开 <kbd>.yarnproject</kbd> 文件，在 Project Metadata 里改变 Saliency Strategy。还支持自定义显著性策略，需要创建一个类继承 `IContentSaliencyStrategy` 接口，并赋值给 Dialogue 类的 ContentSaliencyStrategy 属性，具体查看官方 API 文档。

## Tags and Metadata
**tag** 用于给 line 或 node 一些额外的信息，tag 不会展示给玩家，主要是被游戏引擎使用。tag 可以被加在行 line 和选项 option 的最后，以 <kbd>#</kbd> 为开头，不能包含空格，可以有多个 tag，但必须放置在同一行中，如下：  

<div style="background-color: #f2faf4; margin-bottom: 1em; padding: 10px;">
Homer: Hi, I'd like to order a tire balancing. #tone:sarcastic #duplicate<br>
</div>

这些 tag 可以在游戏引擎中通过 `LocalizedLine.Metadata` 属性获取。具体如何使用，取决于你自己。还有些特殊的 Yarn Spinner 内部使用的 tag：  
①<kbd>#lastline</kbd>：Yarn Spinner 编译脚本时会在 option 前的最后一行添加这个 tag，你可以自由使用它：

<div style="background-color: #f2faf4; margin-bottom: 1em; padding: 10px;">
Hello there. #lastline<br>
-> Hi!<br>
-> What's up?<br>
</div>

在 Unity 中，Yarn Spinner 自带的 Options Presenter，可以选择是否 Show Last Line，就是因为使用了这个 tag。于是在选择选项时，我们可以看到上一行讲了什么。

②<kbd>#line</kbd>：这个是用于做多语言**本地化 Localization** 的，相当于每句话的 id，这个 id 会被自动添加，可以在 Unity 项目中导出带有每句话 id 的 CSV 文件，去做翻译工作。

<div style="background-color: #f2faf4; margin-bottom: 1em; padding: 10px;">
Mechanic: You're in orbit of Jupiter, at a rest station along the main tourism lines. There's a meteorite headed towards here that'll completely destroy this station in three days. #line:4c49c5<br>
Mechanic: And you're a wayfinding robot bolted to the floor of said Jupiter Tourist Station. #line:5b6256<br>
-> Bolted to the floor?! #line:f65d07<br>
&emsp;&emsp;Mechanic: Yeah, like all tourist helper bots. #line:1b159b<br>
-> Three days?! #line:40eaf7<br>
&emsp;&emsp;Mechanic: More or less. I wouldn't make any long-term plans. #line:3a6c94<br>
</div>

node 的 tag 需要使用 header 添加，无需以 <kbd>#</kbd> 为开头，当然也可以添加 #，也支持多个 tag，如下：  

<div style="background-color: #f2faf4; margin-bottom: 1em; padding: 10px;">
title: Train_Dialogue<br>
tags: #camera2 background:conductor_cabin<br>
---<br>
Why did you stop the train?<br>
Now we won't arrive in time at the next stop!<br>
===<br>
</div>

line 的 **metadata** 只有 tag，而 node 的 metadata 还包含一个叫做 <kbd>tracking</kbd> 的 header，编译器会为所有 <kbd>visited()</kbd> function 调用的 node，创建这个 tracking header。当然你也可以自主在 header 中添加 tracking 并携带 <kbd>always</kbd> 值（还支持 <kbd>never</kbd> 值），即使没有被 <kbd>visited()</kbd> 调用：  

<div style="background-color: #f2faf4; margin-bottom: 1em; padding: 10px;">
title: Node_Name<br>
tracking: always<br>
---<br>
I know how many times you've been here.<br>
===<br>
</div>

**metadata** 跟文本 id 一样，可以在 Unity 中导出 CSV 文件，通过在 Unity 点击 <kbd>.yarnproject</kbd> 文件，就可以看到。

## Markup
**标记 markup** 可以理解为特性 attribute，一种用法是用于标记一段文字做特殊的文字效果或者动画的，如下：  

<div style="background-color: #f2faf4; margin-bottom: 1em; padding: 10px;">
Oh, [wave]hello[/wave] there!<br>
Oh, [wave]hello [bounce]there![/bounce][/wave]&emsp;&emsp;//overlapping attributes<br> 
[wave/]&emsp;&emsp;//self-closing attributes<br>
[wave][bounce]Hello![/]&emsp;&emsp;//close-all marker<br>
[wave size=2]Wavy![/wave]&emsp;&emsp;//attributes can have properties<br>
[wave=2]Wavy![/wave]&emsp;&emsp;//this attribute 'wave' has a property called 'wave', which has an integer value of 2.<br>
</div>

但是文字效果或动画，应该是要安装 Text Animator 插件的（我没装插件，在 Unity 中尝试了一下没有效果）。markup 除了上述用法，还有个替换文本的功能，有三个内置的 replacement markers：<kbd>select</kbd>、<kbd>plural</kbd>、<kbd>ordinal</kbd>，这三个 markers 都有个内置的 property 叫做 <kbd>value</kbd>：  

①<kbd>select</kbd> marker 用于替换文本，比如根据设定的性别替换文本，注意 select 是 self-closing attribute，后面有个 <kbd>/</kbd>：  

<div style="background-color: #f2faf4; margin-bottom: 1em; padding: 10px;">
&lt;&lt;declare $gender= ""&gt;&gt;<br>
选择你的性别：  <br>
-> 男<br>
&emsp;&emsp;&lt;&lt;set $gender = "male"&gt;&gt;<br>
-> 女<br>
&emsp;&emsp;&lt;&lt;set $gender = "female"&gt;&gt;<br>
[select value={$gender} male="他" female="她" /]从梦中醒来~<br>
</div>

②<kbd>plural</kbd>和<kbd>ordinal</kbd>，第一个是处理复数的，第二个是处理序数的（即 1st、2nd）。除了 <kbd>value</kbd> 外，它们还有 5 个 property：<kbd>one</kbd>、<kbd>two</kbd>、<kbd>few</kbd>、<kbd>many</kbd>、<kbd>other</kbd>，示例如下：  

<div style="background-color: #f2faf4; margin-bottom: 1em; padding: 10px;">
PieMaker: Hey, look! [plural value={$pie_count} one="A pie" other="Some pies" /]!<br>
PieMaker: I just baked [plural value={$pie_count} one="a pie" other="% pies" /]!<br>
Runner: The race is over! I came in [ordinal value={$race_position} one="%st" two="%nd" few="%rd" other="%th" /] place!<br>
</div>

<div style="background-color: #edf7fa; margin-bottom: 1em; padding: 10px;">
<span style="color: #316f8f;">NOTE：</span>实际上 Yarn Spinner 查找行中角色名的方式也是通过 markup 来实现的。markup parser 会对每行第一个 <kbd>:</kbd> 加一个空格之前的内容添加 <kbd>character</kbd> attribute，类似 <code>[character name="CharacterA"]CharacterA: [/character]Hello!</code> 。
</div>

## Shadow Lines
**shadow line** 主要用于<u>让相同的 line 拥有同一个 id，便于共享资源，比如配音音频文件</u>，同时共享 id 的 line 不会在字符串表中拥有不同的入口。源 line 需要有 <kbd>#line:</kbd> tag，共享源 line id 的 line 需要使用 <kbd>#shadow:</kbd> tag：  

<div style="background-color: #f2faf4; margin-bottom: 1em; padding: 10px;">
title: Tavern<br>
---<br>
Ava: Hello, barkeep!  <br>
Guy: Hi there, how can I help? <br>
Ava: I should go. #line:departure<br>
===<br>
<br>
title: Kitchen<br>
---<br>
Ava: Greetings, chef!<br>
Guy: What are you doing back here? <br>
Ava: I should go. #shadow:departure<br>
===<br>
</div>
