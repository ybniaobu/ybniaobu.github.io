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
Yarn Spinner 脚本由**节点 nodes** 组成，一个 node 包含一组**抬头 headers** 和**主体 body**，其中一组抬头至少要包含**标题 title**。这个 title 比较重要，是 node 的名字，游戏程序需要使用 title 进入一段对话，或者跳跃到另外一段对话，title 不会向玩家展示。而主体 body 则包含文本对话，基本单位是**行 line**，node 会一行一行地将文本数据传输给游戏，其中行开头在冒号前（冒号前无空格）的内容会被标记为角色名，角色名信息也会传输给游戏，当然行也可以没有角色名。

    title: Start
    ---
    This is a line of dialogue, without a character name.
    Speaker: This is another line of dialogue said by a character called "Speaker".
    ===

其中 headers 可以包含多行 <kbd>key: value</kbd> 结构，`---` 标识表示主体 body 开始，`===` 标识表示节点 node 结束。

<div style="background-color: #fff5ed; margin-bottom: 1em; padding: 10px;">
<span style="color: #ff9900;">Warning：</span> title 不支持数字开头，也不支持空格和 <kbd>.</kbd>。<kbd>First Node</kbd>、<kbd>First.Node</kbd>、<kbd>1stNode</kbd> 都是非法的。
</div>

### 编辑器 Header

除了 title 这个 header 外，其他 header 基本都是用于在编辑器的树状图形视图 Graph View 中改变 node 的显示效果的，有如下几个 header：  

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

    title: 嵌套Option
    ---
    // 注意缩进！！！！！！
    请选择：
    -> 选项1
        选择了选项1
        请再次选择：
        -> 选项1-1
            选择了选项1-1
        -> 选项1-2
            选择了选项1-2
    -> 选项2
        选择了选项2
        请再次选择：
        -> 选项2-1
            选择了选项2-1
        -> 选项2-2
            选择了选项2-2
    选择完毕
    ===

<div align="center">  
<img src="https://files.seeusercontent.com/2026/04/08/d0jL/YarnSpinner06_Option.gif" width = "30%" height = "30%" alt="图6 - Option 演示"/>
</div>

### Jump
跳转命令 <kbd>&lt;&lt;jump title&gt;&gt;</kbd> 用于从一个 node 跳转到另外一个 node，注意该命令中 jump 与要跳转的 title 中间有个空格。跳转命令还可以跳转到另外一个 <kbd>.yarn</kbd> 文件里的 node（注意 node 的 title 在项目全局内是不能重复的）。跳转命令的好处就是可以减少 option 的相互嵌套，示例如下：  

    title: Jump命令
    ---
    请选择跳转场景：
    -> 跳转node1
        <<jump node1>>
    -> 跳转node2
        <<jump node2>>
    ===

    title: node1
    ---
    跳转至了node1
    ===

    title: node2
    ---
    跳转至了node2
    ===

<div align="center">  
<img src="https://files.seeusercontent.com/2026/04/08/hkG2/YarnSpinner07_Jump.gif" width = "30%" height = "30%" alt="图7 - Jump 演示"/>
</div>

### Detour
迂回命令 <kbd>&lt;&lt;detour title&gt;&gt;</kbd> 跟跳转命令很类似，只不过跳转命令跳转到其他 node 不会回来了，而迂回命令会回到调用该命令的位置。触发回去有两种情形，一种是到达了 node 的底部，一种是到达了 <kbd>&lt;&lt;return&gt;&gt;</kbd> 命令，示例如下：  

    title: Detour命令
    ---
    A: 你想听我讲故事吗？
    -> B: 不想。
        A: 好吧。
    -> B: 想。
        <<detour 讲故事>>
    A: 你可以去睡了。
    ===

    title: 讲故事
    ---
    A: 你想要听详细的版本还是简短的版本？
    -> B: 简短的。
        A: 吧啦吧啦。
    -> B: 详细的
        <<detour 讲详细故事>>
        A: 希望你听的开心。
    ===

    title: 讲详细故事
    ---
    A: 吧啦吧啦吧啦吧啦。
    A: 还想再听吗？
    -> 想。
    -> 不想。
        <<return>>
    A: 吧啦吧啦吧啦吧啦吧啦吧啦。
    ===

<div align="center">  
<img src="https://files.seeusercontent.com/2026/04/08/rD5p/YarnSpinner08_Detour.gif" width = "30%" height = "30%" alt="图8 - Detour 演示"/>
</div>

### Variables


## 高级语法

# Yarn Spinner in Unity
https://github.com/YarnSpinnerTool/YarnSpinner-Unity.git#current