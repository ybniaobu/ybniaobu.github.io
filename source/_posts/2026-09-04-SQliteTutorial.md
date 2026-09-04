---
title: SQlite 学习笔记
date: 2026-09-04 13:02:01
categories: 
  - [计算机语言, 数据库]
tags:
  - 计算机语言
  - 数据库
  - SQlite
top_img: /images/black.jpg
cover: https://files.seeusercontent.com/2026/09/04/0qBc/SQlite.png
description: XXXXXXXXXXXXXXXXXXXXXXXXXXXXX
---

# SQLite 简介
**SQLite** 是一款轻量级的**内嵌式关系型数据库 embedded relational database**，由 D. Richard Hipp 于 2000 年开发并发布，它具有如下特质：  
①**无服务器 serverless**：诸如 MySQL 或 PostgreSQL 之类的**关系型数据库管理系统 RDBMS** 通常需要一个独立的服务器进程才能运行，需要访问数据库的应用程序通过 TCP/IP 协议来发送和接收请求，这种结构被称为**客户端/服务器架构 client/server architecture**。而 SQLite 数据库则直接与访问它的应用程序集成在一起，应用通过直接读写存储在磁盘上的数据库文件（`.db` 或 `.sqlite`）来与数据库交互，整个过程不需要任何服务器进程；  
②**自包含 self-contained**：SQLite 以 C 语言编写，几乎不依赖任何外部组件或第三方库，也无需安装，整个引擎被完整地封装在单个库文件（如 `sqlite3`）之中，开发时只需把这个库链接进自己的应用程序即可直接使用，无需进行任何配置或管理，这使得 SQLite 能够适用于任何运行环境；  
③**零配置 zero-configuration**：在使用 MySQL、PostgreSQL 等传统数据库之前，往往需要安装部署、编写配置文件、创建用户账户并设置权限等一系列繁琐步骤；而使用 SQLite 则无需任何配置即可开箱即用，既没有需要启动或停止的服务，也没有用户与权限需要管理；  
④**事务性 transactional**：SQLite 完整支持 **ACID 事务**，即所有操作是**原子性 atomicity**、**一致性 consistency**、**隔离性 isolation** 与**持久性 durability** 的。事务中的一系列操作要么全部成功提交，要么在任意一步出错时整体回滚，不会出现只执行一半的中间状态，即使发生断电或进程崩溃，数据库也能依靠日志恢复至一致状态，保证数据的完整与可靠。

> SQLite 官方提供了一个名为 `sqlite3`（window 中为 sqlite3.exe）的命令行交互工具 CLI，下载地址为：https://www.sqlite.org/download.html 。这里就不介绍如何配置 PATH 环境变量和 sqlite3 命令了，不想使用 CLI 完全可以不配置安装。SQLite GUI 工具推荐 [Letos](https://github.com/pawelsalawa/letos)（之前叫 SQLiteStudio）。

# 数据类型与约束
## 存储类
首先，SQLite 采用的是动态类型系统，即某个列的数据类型是由<u>存储在该列中的值</u>决定的，而不是由该列<u>声明时的数据类型</u>决定的。而要理解 SQLite 的类型系统，需注意区分**存储类 storage class** 和**数据类型 data type** 这两个概念，以及对应的**类型亲和性 type affinity**。存储类描述的是 SQLite 在磁盘上存储数据时所使用的格式，它比数据类型的概念更为宽泛。而数据类型则是更具体、更细节的分类。SQLite 提供了五种存储类，如下表所示：  

| 存储类 | 描述 | 说明 |
| :---- | :---- | :---- |
| NULL | 空值 | 表示该列没有存储任何数据 |
| INTEGER | 有符号整数 | 按数值大小以 1、2、3、4、6 或 8 字节存储 |
| REAL | 浮点数 | 以 8 字节的 IEEE 浮点数格式存储 |
| TEXT | 文本字符串 | 按数据库编码（UTF-8、UTF-16BE 或 UTF-16LE）存储 |
| BLOB | 二进制大对象 | 完全按输入的原样字节存储 |

SQLite 会根据**字面量 literal** 的书写形式，按照以下规则确定其所属的存储类：  
① 字面量不带引号且含有小数点或指数时，归为 REAL 存储类，例如 `3.14`、`1.5e3`；  
② 字面量被单引号括起来时，归为 TEXT 存储类，例如 `'Hello'`、`'你好'`。如果字符串本身包含单引号，需要写成<u>两个连续的单引号</u> `''` 来表示一个单引号，例如 `'O''Reilly'` 表示 O'Reilly。注意，不要使用双引号包裹字符串，在 SQLite 中双引号用于包裹标识符（如表名、列名）；  
③ 字面量不带引号、不含小数点也不含指数（即纯整数形式）时，归为 INTEGER 存储类，例如 `123`、`-456`；  
④ 不带引号的 NULL 字面量，归为 NULL 存储类；  
⑤ 以 `X'…'` 或 `x'…'` 为前缀的字面量，归为 BLOB 存储类，例如 `X'53514C697465'` 表示文本 SQLite。

另外，SQLite 提供了 `typeof()` 函数，它可以根据值的书写格式查看其所属的存储类，如下所示：  

``` SQL
SELECT TYPEOF(100), TYPEOF(10.0), TYPEOF('100'), TYPEOF(x'1000'), TYPEOF(NULL);
```

输出：  

    TYPEOF(100) | TYPEOF(10.0) | TYPEOF('100') | TYPEOF(x'1000') | TYPEOF(NULL)
    ------------+--------------+---------------+-----------------+-------------
    integer     | real         | text          | blob            | null

### 类型亲和性
当你创建一个表并为列指定数据类型时，实际上是在为这个列设置一个**类型亲和性 type affinity**，这个亲和性只是个推荐，告诉 SQLite 这一列最好存哪种类型的数据，但仍然可以在 INTEGER 亲和性的列中存入 TEXT。当插入的数据类型与列的亲和性不匹配时，SQLite 会尝试将数据隐式转换为亲和性所指示的类型。例如，你向一个具有 INTEGER 亲和性的列插入字符串 "123"，SQLite 会尝试把它转换为整数 123 再存储。共有 5 种类型亲和性，如下表：  

| 类型亲和性 | 描述 | 说明 |
| :---- | :---- | :---- |
| TEXT | 文本 | 使用 NULL、TEXT 和 BLOB 存储类存储数据。数值型数据在被插入之前，需要先被转换为文本格式，之后再插入到目标字段中 |
| NUMERIC | 数值 | 可使用全部五种存储类。若文本能无损且可逆地转为整数或浮点数（优先整数）则转换后存储，否则仍按文本存储 |
| INTEGER | 整数 | 行为与 NUMERIC 基本相同，只是通过 `CAST` 转换时文本会优先转为整数 |
| REAL | 浮点数 | 与 NUMERIC 类似，但会把整数值强制转换为 8 字节浮点表示 |
| BLOB (NONE) | 二进制大对象 | 不偏向任何存储类，也不做任何类型转换，数据按原样存储 |

### Boolean 与 Date/Time





## 约束 Constraint
### 主键与自增
### 外键
### 其他约束（NOT NULL、UNIQUE、CHECK、DEFAULT）

# 建表与修改表
## 创建表 CREATE TABLE
## 修改表 ALTER TABLE
## 删除表 DROP TABLE

# 查询数据 SELECT
## WHERE 条件过滤
## ORDER BY 排序与 LIMIT 分页
## 聚合函数与 GROUP BY
## 连接查询 JOIN
## 子查询 Subquery

# 数据的增删改 DML
## 插入 INSERT
## 更新 UPDATE
## 删除 DELETE

# 事务 Transaction
## 事务控制语句
## ACID 与并发控制

# 进阶特性
## 视图 View
## 索引 Index
## 触发器 Trigger
## 窗口函数与公共表表达式（CTE）

# 在程序中调用
## C/C++ API
## C#（Unity）中的使用



