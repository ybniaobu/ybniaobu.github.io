---
title: SQLite 学习笔记
date: 2026-09-04 13:02:01
categories: 
  - [计算机语言, 数据库]
tags:
  - 计算机语言
  - 数据库
  - SQLite
top_img: /images/black.jpg
cover: https://files.seeusercontent.com/2026/09/04/0qBc/SQlite.png
description: 本笔记的主要内容有 SQLite 的数据类型、数据表操作、数据的查增改删，以及索引、事务等进阶特性，最后还有如何在 C/C++ 与 C# 程序中调用 SQLite【暂未完成】。
---

> 本文尽可能地介绍 SQLite 的大部分内容，但受限于篇幅，未提到的内容请查阅 [SQLite 官方文档](https://www.sqlite.org/docs.html)。本文书写时 SQLite 版本为 3.53.4（2026-07-24）。

# SQLite 简介
**SQLite** 是一款轻量级的**内嵌式关系型数据库 embedded relational database**，由 D. Richard Hipp 于 2000 年开发并发布，它具有如下特质：  
①**无服务器 serverless**：诸如 MySQL 或 PostgreSQL 之类的**关系型数据库管理系统 RDBMS** 通常需要一个独立的服务器进程才能运行，需要访问数据库的应用程序通过 TCP/IP 协议来发送和接收请求，这种结构被称为**客户端/服务器架构 client/server architecture**。而 SQLite 数据库则直接与访问它的应用程序集成在一起，应用通过直接读写存储在磁盘上的数据库文件（`.db` 或 `.sqlite`）来与数据库交互，整个过程不需要任何服务器进程；  
②**自包含 self-contained**：SQLite 以 C 语言编写，几乎不依赖任何外部组件或第三方库，也无需安装，整个引擎被完整地封装在单个库文件（如 `sqlite3`）之中，开发时只需把这个库链接进自己的应用程序即可直接使用，无需进行任何配置或管理，这使得 SQLite 能够适用于任何运行环境；  
③**零配置 zero-configuration**：在使用 MySQL、PostgreSQL 等传统数据库之前，往往需要安装部署、编写配置文件、创建用户账户并设置权限等一系列繁琐步骤。而使用 SQLite 则无需任何配置即可开箱即用，既没有需要启动或停止的服务，也没有用户与权限需要管理；  
④**事务性 transactional**：SQLite 完整支持 **ACID 事务**，即所有操作是**原子性 atomicity**、**一致性 consistency**、**隔离性 isolation** 与**持久性 durability** 的。事务中的一系列操作要么全部成功提交，要么在任意一步出错时整体回滚，不会出现只执行一半的中间状态，即使发生断电或进程崩溃，数据库也能依靠日志恢复至一致状态，保证数据的完整与可靠。

> SQLite 官方提供了一个名为 `sqlite3`（window 中为 sqlite3.exe）的命令行交互工具 CLI，下载地址为：https://www.sqlite.org/download.html 。这里就不介绍如何配置 PATH 环境变量和 sqlite3 命令了，不想使用 CLI 完全可以不配置安装。SQLite GUI 工具推荐 [Letos](https://github.com/pawelsalawa/letos)（之前叫 SQLiteStudio）。

# 数据类型
## 存储类
首先，SQLite 采用的是动态类型系统，即表的某个列的数据类型是由<u>存储在该列中的值</u>决定的，而不是由该列<u>声明时的数据类型</u>决定的。而要理解 SQLite 的类型系统，需注意区分**存储类 storage class** 和**数据类型 data type** 这两个概念，以及对应的**类型亲和性 type affinity**。存储类描述的是 SQLite 在磁盘上存储数据时所使用的格式，它比数据类型的概念更为宽泛，而数据类型则是更具体、更细节的分类，例如，INTEGER 存储类就包含了 7 种不同长度的整数数据类型。SQLite 提供了五种存储类，如下表所示：  

| <font size=2>存储类</font> | <font size=2>描述</font> | <font size=2>说明</font> |
| :---- | :---- | :---- |
| <font size=2>NULL</font> | <font size=2>空值</font> | <font size=2>表示该列没有存储任何数据</font> |
| <font size=2>INTEGER</font> | <font size=2>有符号整数</font> | <font size=2>按数值大小以 1、2、3、4、6 或 8 字节存储</font> |
| <font size=2>REAL</font> | <font size=2>浮点数</font> | <font size=2>以 8 字节的 IEEE 浮点数格式存储</font> |
| <font size=2>TEXT</font> | <font size=2>文本字符串</font> | <font size=2>按数据库编码（UTF-8、UTF-16BE 或 UTF-16LE）存储</font> |
| <font size=2>BLOB</font> | <font size=2>二进制大对象</font> | <font size=2>完全按输入的原样字节存储</font> |

SQLite 会根据**字面量 literal** 的书写形式，按照以下规则确定其所属的存储类：  
* 字面量不带引号且含有小数点或指数时，归为 REAL 存储类，例如 `3.14`、`1.5e3`；  
* 字面量被单引号括起来时，归为 TEXT 存储类，例如 `'Hello'`、`'你好'`。如果字符串本身包含单引号，需要写成<u>两个连续的单引号</u> `''` 来表示一个单引号，例如 `'O''Reilly'` 表示 O'Reilly。注意，不要使用双引号包裹字符串，在 SQLite 中双引号用于包裹标识符（如表名、列名）；  
* 字面量不带引号、不含小数点也不含指数（即纯整数形式）时，归为 INTEGER 存储类，例如 `123`、`-456`；  
* 不带引号的 NULL 字面量，归为 NULL 存储类；  
* 以 `X'…'` 或 `x'…'` 为前缀的字面量，归为 BLOB 存储类，例如 `X'53514C697465'` 表示文本 SQLite。

另外，SQLite 提供了 `typeof()` 函数，它可以根据值的书写格式查看其所属的存储类，如下所示：  

``` SQL
SELECT TYPEOF(100), TYPEOF(10.0), TYPEOF('100'), TYPEOF(x'1000'), TYPEOF(NULL);
```

输出：  

``` Console
TYPEOF(100) | TYPEOF(10.0) | TYPEOF('100') | TYPEOF(x'1000') | TYPEOF(NULL)
------------+--------------+---------------+-----------------+-------------
integer     | real         | text          | blob            | null
```

### Boolean 与 Date Time
SQLite 没有独立的 Boolean、Date/Time 存储类：  
* 布尔值在 SQLite 中实际上是被当作 INTEGER 来存储和处理的，TRUE 存储为整数 1，FALSE 存储为整数 0。从 SQLite 3.23.0 版本开始（2018-04-02），你可以直接使用 TRUE 和 FALSE 关键字，但它们只是整数字面量 1 和 0 的别名。  
* SQLite 能够把日期和时间存储为 TEXT、REAL 或 INTEGER 值，如下表。可以使用内置的日期和时间函数来自由转换不同格式。

| <font size=2>存储类</font> | <font size=2>日期格式</font> |
| :---- | :---- |
| <font size=2>TEXT</font> | <font size=2>格式为 "YYYY-MM-DD HH:MM:SS.SSS" 的日期</font> |
| <font size=2>REAL</font> | <font size=2>儒略日数（Julian day numbers），即从格林威治时间公元前 4714 年 11 月 24 日中午起经过的天数（基于前推格里高利历 proleptic Gregorian calendar）</font> |
| <font size=2>INTEGER</font> | <font size=2>Unix 时间戳，即从 1970-01-01 00:00:00 UTC 起经过的秒数</font> |

## 类型亲和性
当你创建一个表并为列指定数据类型时，实际上是在为这个列设置一个**类型亲和性 type affinity**，这个亲和性只是个推荐，告诉 SQLite 这一列最好存哪种类型的数据，但仍然可以在 INTEGER 亲和性的列中存入 TEXT。当插入的数据类型与列的亲和性不匹配时，SQLite 会尝试将数据隐式转换为亲和性所指示的类型。例如，你向一个具有 INTEGER 亲和性的列插入字符串 "123"，SQLite 会尝试把它转换为整数 123 再存储。共有 5 种类型亲和性，如下表：  

| <font size=2>类型亲和性</font> | <font size=2>描述</font> | <font size=2>说明</font> |
| :---- | :---- | :---- |
| <font size=2>TEXT</font> | <font size=2>文本</font> | <font size=2>使用 NULL、TEXT 和 BLOB 存储类存储数据。数值型数据在被插入之前，需要先被转换为文本格式，之后再插入到目标字段中</font> |
| <font size=2>NUMERIC</font> | <font size=2>数值</font> | <font size=2>可使用全部五种存储类。若文本能无损且可逆地转为整数或浮点数（优先整数）则转换后存储，否则仍按文本存储</font> |
| <font size=2>INTEGER</font> | <font size=2>整数</font> | <font size=2>行为与 NUMERIC 基本相同，只是通过 `CAST` 转换时文本会优先转为整数</font> |
| <font size=2>REAL</font> | <font size=2>浮点数</font> | <font size=2>与 NUMERIC 类似，但会把整数值强制转换为 8 字节浮点表示</font> |
| <font size=2>BLOB (NONE)</font> | <font size=2>二进制大对象</font> | <font size=2>不偏向任何存储类，也不做任何类型转换，数据按原样存储</font> |

列的亲和性由其声明的数据类型按如下顺序的规则匹配决定：  
1. 若声明的类型中包含字符串 "INT"，则该列具有 INTEGER 亲和性；  
2. 若声明的类型中包含 "CHAR"、"CLOB" 或 "TEXT" 中的任意一个，则该列具有 TEXT 亲和性；  
3. 若声明的类型中包含字符串 "BLOB"，或者根本没有指定类型，则该列具有 BLOB 亲和性；  
4. 若声明的类型中包含 "REAL"、"FLOA" 或 "DOUB" 中的任意一个，则该列具有 REAL 亲和性；  
5. 其他情况下，亲和性为 NUMERIC。  

注意这些规则的匹配顺序，例如，一个声明类型为 "CHARINT" 的列同时满足规则 1 和 2，但规则 1 优先，因此该列的亲和性为 INTEGER。

下面的表格展示了传统 SQL 实现中的许多常见数据类型名称所对应的亲和性，该表只列出了 SQLite 所能接受的类型名称中的一小部分。注意，SQLite 会忽略类型名后括号内的数字参数（例如 `VARCHAR(255)` 中的 `255`）：  

| <font size=2>亲和性</font> | <font size=2>数据类型</font> |
| :---- | :---- |
| <font size=2>INTEGER</font> | <font size=2>INT、INTEGER、TINYINT、SMALLINT、MEDIUMINT、BIGINT、UNSIGNED BIG INT、INT2、INT8</font> |
| <font size=2>TEXT</font> | <font size=2>CHARACTER、VARCHAR、VARYING CHARACTER、NCHAR、NATIVE CHARACTER、NVARCHAR、TEXT、CLOB</font> |
| <font size=2>BLOB</font> | <font size=2>BLOB、未指定类型</font> |
| <font size=2>REAL</font> | <font size=2>REAL、DOUBLE、DOUBLE PRECISION、FLOAT</font> |
| <font size=2>NUMERIC</font> | <font size=2>NUMERIC、DECIMAL、BOOLEAN、DATE、DATETIME</font> |

# 数据库操作
SQLite 通过其 C 语言 API（如 `sqlite3_open()` 等）负责打开或连接数据库文件，默认情况下，如果文件不存在，还会创建它。SQLite 并没有 SQL 层面的 `CREATE DATABASE` 与 `DROP DATABASE` 语句。要删除一个持久数据库，一般是先关闭连接，并在操作系统层面直接删除对应的数据库文件即可。同时，一个数据库连接可以通过 `ATTACH DATABASE` 附加多个数据库文件，从而同时访问多个数据库，并用 `DETACH DATABASE` 分离，但 DETACH 不等于删除文件。

## 附加数据库 ATTACH DATABASE
通常情况下，一个**数据库连接 database connection** 只打开一个数据库文件，这个数据库被称为**主数据库 main database**，其**模式名 schema name**（或称为别名）固定为 `main`。除此之外，每个连接还会拥有一个**临时数据库 temp database**，其模式名固定为 `temp`，用来存放临时表（通过 `CREATE TEMP TABLE` 创建）等仅在本连接内有效的对象。我们也可以使用 `ATTACH DATABASE` 语句把额外的数据库文件附加到当前连接上，可以在 SQL 语句中跨库查询、连接或复制数据，附加的基本语法如下：  

``` SQL
ATTACH DATABASE 'file_name' AS schema_name;
```

* `DATABASE` 关键字可以省略，写成 `ATTACH 'file_name' AS schema_name;` 也完全等价；  
* *'file_name'* 为要附加的数据库文件路径，使用相对路径或绝对路径。若指定的文件不存在，SQLite 会先创建一个空的数据库文件再附加；  
* *schema_name* 为给该数据库起的模式名或别名，之后引用其中的对象可以使用限定名 `schema_name.table_name`，若使用未限定名，SQLite 会先找 temp，再找 main，再找已附加的数据库。模式名不能是 `main` 或 `temp`，不能以 `sqlite_` 开头，也不能与已附加的其它模式名重复；  
* 文件名写成 `':memory:'` 时，附加的是一个内存数据库，完全驻留在内存中、不写入磁盘，连接断开后即消失；文件名写成空字符串 `''` 时，SQLite 会创建一个私有的临时数据库文件，该文件会在连接关闭时自动删除；  
* 同一连接上可附加的数据库数量是有限制的，默认最多 10 个（不含 `main` 与 `temp`），由编译时参数 `SQLITE_MAX_ATTACHED` 决定，其最大可达 125。运行时可用 C 接口 `sqlite3_limit()` 的 `SQLITE_LIMIT_ATTACHED` 参数查看或调整；  
* 可以通过 `PRAGMA database_list`，查询当前连接上所有已附加的数据库，返回序列号、名称和文件名三列。  

## 分离数据库 DETACH DATABASE
与附加相对的 `DETACH DATABASE` 语句用于把之前附加进来的数据库从当前连接上分离，其基本语法如下：  

``` SQL
DETACH DATABASE schema_name;
```

* `DATABASE` 关键字同样可以省略，写成 `DETACH schema_name;` 也完全等价；  
* 如果同一个数据库文件被用多个别名附加到了同一个连接上，DETACH 只会断开你指定的那一个别名对应的连接，其他别名仍然有效；  
* `main` 与 `temp` 不能被分离，它们并不是通过 `ATTACH` 附加进来的；  
* 如果被分离的数据库是一个内存数据库或临时数据库，DETACH 操作会摧毁这个数据库，其中的所有数据将会丢失。

# 数据表操作
数据表操作对应 SQL 中的**数据定义语言 data definition language (DDL)**，它与负责读写行数据的**数据操纵语言 data manipulation language (DML)** 相对，DDL 关注的是**模式对象 schema object** 本身的定义，而不关心对象里存放了什么数据。SQLite 的 DDL 语句以 `CREATE`、`ALTER`、`DROP` 三类关键字为主，可作用于的模式对象包括：**表 table**、**索引 index**、**视图 view** 和**触发器 trigger**。这几类对象之间存在依附关系：索引与触发器必须依附于某张表，删除表时它们会被一并删除，而视图虽然独立存在，一旦其定义引用的表被删除便会失效。因此这里先讲表相关操作，而索引、视图与触发器的语法则留到进阶特性章节中介绍。

> SQLite 会把每个模式对象的定义以**创建语句的原文**保存到内部系统表 `sqlite_schema`（旧名 `sqlite_master`）中，可以像查询普通表一样查询它。

## 创建表 CREATE TABLE
使用 `CREATE TABLE` 语句在任何给定的数据库创建一个新表，基本语法如下（其中中括号 `[]` 表示可选）：  

``` SQL
CREATE TABLE [IF NOT EXISTS] [schema_name].table_name (
    column_1 data_type PRIMARY KEY,
    column_2 data_type NOT NULL,
    column_3 data_type DEFAULT value,
    table_constraints
) [WITHOUT ROWID];
```

* 在 `CREATE TABLE` 关键字之后跟着表的唯一的名称或标识。表名 *table_name* 不能以 `sqlite_` 开头，因为该前缀被保留给 SQLite 内部使用；  
* 使用 `IF NOT EXISTS` 选项可以在表不存在时才创建新表。如果不使用该选项而尝试创建已存在的表，则会报错；  
* 可以指定新表所属的 *schema_name*，默认在主数据库 main 当中，也可以是临时数据库 temp 或任何附加 attached 的数据库；  
* 语句中可以包含多个**列定义 column definition**，由列名 *column_1*、数据类型 *data_type* 和**列约束 column constraint** 组成。这里的列约束还包括 `COLLATE` 和 `DEFAULT` 子句，尽管它们并不是真正意义上的约束，因为它们并不限制表中可以包含的数据。其他约束如 `PRIMARY KEY`、`FOREIGN KEY`、`UNIQUE`、`NOT NULL` 和 `CHECK` 会对表数据施加限制。表中的列数受编译时参数 `SQLITE_MAX_COLUMN` 的限制，表中单行数据不能超过 `SQLITE_MAX_LENGTH` 字节；  
* *table_constraints* 为一组**表级约束 table constraint**，如 `PRIMARY KEY`、`FOREIGN KEY`、`UNIQUE` 和 `CHECK` 约束；  
* 可选的 `WITHOUT ROWID` 选项（SQLite 3.8.2 及更高版本中支持）。表中的每一行都有一个隐式的 `rowid` 列，如果不希望 SQLite 创建 `rowid` 列，可以指定 `WITHOUT ROWID` 选项，但此时必须指定一个主键 PRIMARY KEY。`rowid` 和 PRIMARY KEY 的关系详见约束的 [PRIMARY KEY](#PRIMARY-KEY) 小节。

另外，也可以使用 `CREATE TABLE ... AS SELECT` 语句通过查询结果直接创建并填充新表，使用该语句创建的表没有主键，也没有任何约束，每一列的默认值都是 NULL，默认**排序规则 collation sequence** 都是 BINARY。

### 子句
#### DEFAULT 子句 
`DEFAULT` 子句用于指定当插入 INSERT 新行却没有为该列提供值时所采用的默认值。<u>如果列定义中不写 `DEFAULT`，该列的默认值就是 `NULL`</u>。显式的 `DEFAULT` 子句可以指定默认值为 `NULL`、字符串常量、BLOB 常量、带符号数值，或**用括号括起来的常量表达式**（如 `(1 + 2)`、`(datetime('now'))`），还可以是三个特殊关键字 `CURRENT_TIME`、`CURRENT_DATE` 与 `CURRENT_TIMESTAMP`，它们取的是 UTC 时间并以 TEXT 存储，格式分别为 `"HH:MM:SS"`、`"YYYY-MM-DD"` 和 `"YYYY-MM-DD HH:MM:SS"`。示例如下：  

> SQLite 判定常量表达式的标准是：其中不含子查询、不含列或表引用、不含绑定参数、也不含用双引号包裹的字符串。因此默认值不能引用同一张表的其它列，也不能引用其它表中的列。

``` SQL
CREATE TABLE orders (
    id       INTEGER PRIMARY KEY,                             -- 不写 DEFAULT，默认值即 NULL
    status   TEXT    NOT NULL DEFAULT 'pending',              -- 字面量形式的默认值
    quantity INTEGER NOT NULL DEFAULT 1 CHECK (quantity > 0), -- 默认值可与其它约束并用
    created  TEXT    NOT NULL DEFAULT CURRENT_TIMESTAMP,      -- 特殊关键字，UTC 时间
    updated  TEXT    DEFAULT (datetime('now', 'localtime')),  -- 括号表达式，每次插入时求值
    token    TEXT    DEFAULT (hex(randomblob(8))),            -- 非确定性函数也可以使用
    note     TEXT    DEFAULT NULL                             -- 显式写出默认值为 NULL
);
``` 

#### COLLATE 子句
`COLLATE` 子句用于为列指定**排序规则 collating sequence**，它决定该列的值在比较运算（`=`、`<`、`BETWEEN`、`IN` 等）、排序（`ORDER BY`）、分组与去重（`GROUP BY`、`DISTINCT`）以及建立索引时，如何判断两个值是否相等、谁大谁小。<u>若不写 `COLLATE`，列的排序规则默认为 `BINARY`</u>。SQLite 内置了三种排序规则，如下表所示：  

| <font size=2>排序规则</font> | <font size=2>说明</font> |
| :---- | :---- |
| <font size=2>BINARY</font> | <font size=2>默认值。直接按值的字节比较（与 UTF-8 编码顺序一致），大小写敏感，例如 `'A' < 'Z' < 'a'`</font> |
| <font size=2>NOCASE</font> | <font size=2>与 BINARY 相同，但比较时忽略ASCII 字母的大小写，因此 `'Alice' = 'ALICE'`；非 ASCII 字符（如 `'Ä'`、`'É'`）不受影响</font> |
| <font size=2>RTRIM</font> | <font size=2>Right Trim 的缩写，与 BINARY 相同，但忽略尾随空格，因此 `'abc' = 'abc '`</font> |

> 若需要中文拼音、多语言等更复杂的排序，可以借助 C 接口 `sqlite3_create_collation()` 注册自定义排序规则，官方 ICU (International Components for Unicode) 扩展正是通过它提供了 `UNICODE` 排序规则，注册后即可与内置规则一样使用，用 `PRAGMA collation_list;` 可以列出当前连接已注册的全部排序规则。

语法上，`COLLATE` 既可作为**列约束**写在列定义中，也可以作为**运算符**接在表达式之后，示例如下：  

``` SQL
CREATE TABLE users (
    id    INTEGER PRIMARY KEY,
    name  TEXT    COLLATE BINARY,                  -- 默认行为：区分大小写
    email TEXT    NOT NULL COLLATE NOCASE UNIQUE,  -- 忽略大小写，'a@x.com' 与 'A@X.com' 视为重复
    code  TEXT    COLLATE RTRIM                    -- 忽略尾随空格
);

SELECT * FROM users ORDER BY name COLLATE NOCASE;       -- 该次排序忽略大小写
```

> 一次比较究竟使用哪个排序规则，SQLite 按以下优先级决定：① 操作数上显式书写的 `COLLATE` 优先（两侧都写时从左到右）；② 否则采用操作数所属列上声明的排序规则；③ 两者都没有则使用 `BINARY`。

#### GENERATED ALWAYS AS 子句
`GENERATED ALWAYS AS` 子句用于定义**生成列 generated column**（也称为计算列 computed column），它的值由另外一个列通过表达式生成。其语法为 `列名 数据类型 [GENERATED ALWAYS] AS (表达式) [VIRTUAL | STORED]`，其中 `GENERATED ALWAYS` 两个词可以省略，只写 `AS (表达式)` 也可以。该特性从 SQLite 3.31.0（2020-01-22）开始支持，按数据的存放方式分为两种，如下表所示：  

| <font size=2>类型</font> | <font size=2>磁盘存储</font> | <font size=2>计算时机</font> | <font size=2>能否用 `ALTER TABLE ADD COLUMN` 添加</font> |
| :---- | :---- | :---- | :---- |
| <font size=2>VIRTUAL（默认）</font> | <font size=2>不存储，只保存表达式</font> | <font size=2>每次读取该列时即时计算</font> | <font size=2>可以</font> |
| <font size=2>STORED</font> | <font size=2>与其它列一起存入数据库文件</font> | <font size=2>插入或更新该行时计算一次</font> | <font size=2>不可以</font> |

生成列的表达式有若干限制：生成列不能有默认值，不能被用作 `PRIMARY KEY`。它只能引用同一张表中同一行的其它列，不能引用其它表或其它行的数据，也不能包含子查询，并且必须是确定性的，`random()` 这类非确定性函数，以及所有依赖当前时间的函数（如 `date('now')`）都不允许使用。示例如下：  

``` SQL
CREATE TABLE order_items (
    id         INTEGER PRIMARY KEY,
    name       TEXT    NOT NULL,
    price      REAL    NOT NULL CHECK (price >= 0),
    qty        INTEGER NOT NULL DEFAULT 1,
    subtotal   REAL    GENERATED ALWAYS AS (price * qty) VIRTUAL, -- 虚拟生成列：默认形式，读取时才计算，不占磁盘空间
    name_lower TEXT    AS (lower(name)) STORED -- 存储生成列：写入时算好并随行保存在文件中
);
```

### 约束
#### PRIMARY KEY
**主键 PRIMARY KEY** 是用于在表中标识每一行的一个列或一组列，每张表最多只能有一个主键。如果把 PRIMARY KEY 关键字加在某个列定义上，即**列约束**，那么该表的主键就由这一列单独构成。也可以写成**表级约束**来定义复合主键，如 `PRIMARY KEY (order_id, line_no)`。主键天然带有**唯一性约束** `UNIQUE`，即主键列（或列组合）中的值在表内必须互不重复。

``` SQL
CREATE TABLE users (
    email TEXT PRIMARY KEY,          -- 列约束：单列主键
    name  TEXT NOT NULL
);

CREATE TABLE order_items (
    order_id INTEGER,
    line_no  INTEGER,
    quantity INTEGER NOT NULL,
    PRIMARY KEY (order_id, line_no)  -- 表级约束：复合主键
);
```

默认情况下，表中的每一行都有一个隐式列，称为 `rowid`、`oid` 或 `_rowid_` 列。`rowid` 列存储一个 64 位有符号整数键，用于在表内唯一标识该行。含有 `rowid` 列的表被称为 rowid 表，rowid 表的数据以 B-Tree 结构存储，每个表行对应一个条目，并使用 rowid 值作为键。当主键并非 INTEGER PRIMARY KEY 时，SQLite 会为主键额外创建一个唯一索引，按主键查询时，先在该索引中查到对应的 rowid，再依据 rowid 在表的 B-Tree 中找到整行数据。如果不希望 SQLite 创建 `rowid` 列，可以指定之前提到的 `WITHOUT ROWID` 选项，同时需要显式指定主键，WITHOUT ROWID 表使用主键作为 B-Tree 的键，数据行直接按主键顺序存储。

如果一张表的主键由单列组成，且该列的声明类型为 `INTEGER`（大小写不限，但写成 `INT` 就不行），同时该表不是 WITHOUT ROWID 表，那么这个主键列就会成为 `rowid` 的别名，被称为 **INTEGER PRIMARY KEY**。在绝大多数情况下，应优先使用 INTEGER PRIMARY KEY，相对存储和读写效率较高，除非业务要求全局唯一字符串主键（比如邮箱），此时表会多一个隐藏 rowid 和主键唯一索引，此时如果表几乎没有辅助索引，且主要按主键查询，可以考虑 WITHOUT ROWID。

``` SQL
CREATE TABLE customers (
    id   INTEGER PRIMARY KEY, -- rowid 别名，插入 NULL 时自动分配
    name TEXT NOT NULL
);
```

当使用 INTEGER PRIMARY KEY 时，还可以额外增加一个 **AUTOINCREMENT 关键字**，用于管理当插入新行且未指定 `rowid` 或指定为 `NULL` 时 SQLite 自动分配值的行为。无论是否使用 AUTOINCREMENT 关键字，SQLite 会选择比当前表中最大 `rowid` 大 1 的值。但如果删除了具有最大 `rowid` 的行，那么该 `rowid` 可能会被后续插入的新行重用。若使用了 `AUTOINCREMENT` 关键字，新 `rowid` 将严格大于该表历史上曾经使用过的最大 `rowid`，即使删除了某些行，曾用过的 `rowid` 也永远不会被重用。为了实现这一点，SQLite 会在内部维护一个名为 sqlite_sequence 的系统表，用于记录每个使用了 AUTOINCREMENT 的表曾经分配过的最大 ROWID 值。同时，SQLite 官方文档明确指出，<u>AUTOINCREMENT 关键字会带来额外的开销，在大多数情况下并非必要</u>。

``` SQL
CREATE TABLE invoices (
    id     INTEGER PRIMARY KEY AUTOINCREMENT, -- 保证 id 一旦用过就不再复用
    amount REAL NOT NULL
);
```

#### FOREIGN KEY
SQLite 从 3.6.19 版本（2009-10-14）起开始支持外键约束，并且要求 SQLite 库在编译时不能定义 `SQLITE_OMIT_FOREIGN_KEY` 和 `SQLITE_OMIT_TRIGGER` 这两个编译选项，否则外键功能会被裁剪掉。出于向后兼容的考虑，<u>SQLite 默认不启用外键约束，必须通过 `PRAGMA foreign_keys = ON;` 为每个数据库连接单独开启</u>，如下：  

``` SQL
PRAGMA foreign_keys = ON;      -- 开启外键检查（对每个新连接都要执行一次！！！）
PRAGMA foreign_keys;           -- 查询当前连接是否已开启，返回 1 表示开启
PRAGMA foreign_key_list(employees); -- 列出某张表上定义的全部外键
```

**外键 FOREIGN KEY** 用于在两个表之间建立关联，外键引用的表称为**父表 parent table**，其被引用的列称为**父键 parent key**，使用外键约束的表称为**子表 child table**，其存放引用值的列称为**子键 child key**。与 PRIMARY KEY 一样，FOREIGN KEY 既可以写在列定义上作为**列约束**（此时只需 `REFERENCES` 子句），也可以写成**表级约束**。如果要引用由多列构成的复合父键，则必须使用表级约束，且子键与父键的列数、顺序必须一一对应。通常情况下，父键就是父表的 PRIMARY KEY，如果父键不是主键，那么这些父键列必须受到 UNIQUE 约束的限制，或者建有 UNIQUE 索引，同时 UNIQUE 索引必须使用 CREATE TABLE 中为父键列声明的排序规则。

``` SQL
CREATE TABLE departments (
    id   INTEGER PRIMARY KEY,
    name TEXT NOT NULL
);

CREATE TABLE employees (
    id      INTEGER PRIMARY KEY,
    name    TEXT    NOT NULL,
    dept_id INTEGER REFERENCES departments (id)   -- 列级外键约束
);

CREATE TABLE order_items (
    order_id INTEGER,
    line_no  INTEGER,
    quantity INTEGER NOT NULL,
    PRIMARY KEY (order_id, line_no)
);

CREATE TABLE item_notes (
    order_id INTEGER,
    line_no  INTEGER,
    note     TEXT,
    FOREIGN KEY (order_id, line_no) REFERENCES order_items (order_id, line_no) -- 表级复合外键
);
```

外键的 `ON DELETE` 和 `ON UPDATE` 子句用于配置当父表中删除行（`ON DELETE`）或修改已有行的父键值（`ON UPDATE`）时所采取的动作，共支持五种动作，默认动作为 `NO ACTION`，其它如下表所示：  

| <font size=2>动作</font> | <font size=2>说明</font> |
| :---- | :---- |
| <font size=2>NO ACTION</font> | <font size=2>默认值，不做任何处理</font> |
| <font size=2>RESTRICT</font> | <font size=2>只要存在引用该行的子行，立即拒绝删除或更新</font> |
| <font size=2>SET NULL</font> | <font size=2>删除或更新父行时，把子行中对应的子键设置为 NULL</font> |
| <font size=2>SET DEFAULT</font> | <font size=2>删除或更新父行时，把子行中对应的子键设置为其默认值</font> |
| <font size=2>CASCADE</font> | <font size=2>删除父行时级联删除所有引用它的子行；更新父键时级联更新子行中的子键值</font> |

``` SQL
CREATE TABLE comments (
    id      INTEGER PRIMARY KEY,
    post_id INTEGER NOT NULL REFERENCES posts (id) ON DELETE CASCADE, -- 删除帖子时级联删除其评论
    author  TEXT    REFERENCES users (name) ON UPDATE CASCADE,        -- 用户名变更时同步更新
    content TEXT    NOT NULL
);
```

> 外键约束并不会自动为子键建立索引。每当父表的父键被删除或更新时，即使动作为默认的 NO ACTION，SQLite 也需按子键回查子表以确认是否存在引用行，若子键无索引，就只能全表扫描，产生性能问题。因此<u>通常应手动为子表的子键列创建索引 INDEX</u>，同时也便于按子键进行连接查询。`CREATE INDEX` 相关内容详见进阶特性章节。
> 
> 外键约束默认是**立即的 immediate**，即每条语句执行结束时立即做外键约束检查。若在约束定义末尾加上 `DEFERRABLE INITIALLY DEFERRED`，该外键就成为**延迟的 deferred** 外键，检查会推迟到事务提交时才进行，只有在此时仍存在违规才会报错。

#### NOT NULL
**非空约束 NOT NULL** 用于限制某个列不允许存放 NULL 值，即插入或更新该列时必须给出一个非 NULL 的值，否则 SQLite 会直接拒绝该语句并报错。`NOT NULL` 只能作为列约束写在列定义上，跟在数据类型之后即可，不能写成表级约束。

``` SQL
CREATE TABLE employees (
    id      INTEGER PRIMARY KEY,
    name    TEXT    NOT NULL                 -- 必须提供姓名
);
```

#### UNIQUE
**唯一性约束 UNIQUE** 用于确保某个列（或一组列）中的值在表内互不重复，插入或更新时若出现重复值，SQLite 会拒绝执行并返回错误。`UNIQUE` 既可以作为列约束写在单个列定义上，也可以写成表级约束来约束多列组合的整体唯一性，此时只要组合中有一列不同即视为不重复。

``` SQL
CREATE TABLE users (
    id    INTEGER PRIMARY KEY,
    email TEXT NOT NULL UNIQUE             -- 列约束：邮箱不允许重复
);

CREATE TABLE order_items (
    order_id INTEGER,
    line_no  INTEGER,
    quantity INTEGER NOT NULL,
    UNIQUE (order_id, line_no)             -- 表级约束：两列组合不允许重复
);
```

对于 `UNIQUE` 约束来说，`NULL` 被视为互不相同的值，因此一个 UNIQUE 列中可以存在任意多个 `NULL`。另外，唯一性的比较遵循该列声明的排序规则，比如声明了 `COLLATE NOCASE` 的 UNIQUE 列中的 `'a@x.com'` 与 `'A@X.com'` 会被视为重复。

在实现层面，SQLite 会为每个 UNIQUE 约束自动创建一个**唯一索引 unique index**（在 `sqlite_schema` 中以 `sqlite_autoindex_表名_N` 命名），查询与去重判断都通过该索引完成。从功能上讲，UNIQUE 约束与手动执行的 `CREATE UNIQUE INDEX` 基本等价，区别在于 UNIQUE 约束是表定义的一部分，只能通过重建表等方式移除，而独立索引可以被显式删除。

#### CHECK
**检查约束 CHECK** 用于为列值附加一个布尔表达式作为限制条件，每当插入或更新一行数据时，SQLite 会对每个 `CHECK` 表达式求值，并将结果按 `CAST` 规则转换为 NUMERIC 值：若结果为整数 0 或实数 0.0，则视为违反约束并拒绝执行该语句；若结果为 NULL 或任意非零值，则视为通过。`CHECK` 既可以作为列约束，也可以作为表级约束，两种写法在功能上没有区别。另外，表达式可以引用同一行中的其它列，用于表达列与列之间的依赖关系。

``` SQL
CREATE TABLE projects (
    id         INTEGER PRIMARY KEY,
    begin_date TEXT NOT NULL,
    end_date   TEXT NOT NULL,
    budget     REAL NOT NULL CHECK (budget > 0),           -- 列约束
    CHECK (begin_date <= end_date)     -- 表级约束
);
```

约束表达式不能包含**子查询 subquery**，也不能引用其它表中的列或同一张表的其它行，只能在当前行的范围内进行比较与计算。在列定义中约束可以连续书写，多个 `CHECK` 之间是“与”的关系，全部通过才算合法。

#### ON CONFLICT 子句
`ON CONFLICT` 是 SQLite 特有的非 SQL 标准扩展，用于指定当 `UNIQUE`、`NOT NULL` 和 `PRIMARY KEY` 约束被违反时所采取的**冲突解决算法 conflict resolution algorithm**（`CHECK` 和 `FOREIGN KEY` 约束不支持该子句，冲突时固定按 ABORT 处理），共五种算法，默认为 `ABORT`，如下表所示：  

| <font size=2>算法</font> | <font size=2>说明</font> |
| :---- | :---- |
| <font size=2>ROLLBACK</font> | <font size=2>中止当前语句并报错，同时回滚整个当前事务</font> |
| <font size=2>ABORT</font> | <font size=2>默认值。中止当前语句并报错，撤销该语句已做的修改，但保留事务中之前的修改，事务仍然有效</font> |
| <font size=2>FAIL</font> | <font size=2>中止当前语句并报错，但该语句在出错前已完成的修改予以保留，事务仍然有效</font> |
| <font size=2>IGNORE</font> | <font size=2>跳过违反约束的那一行，继续处理语句中的其余行，不报错</font> |
| <font size=2>REPLACE</font> | <font size=2>先删除导致冲突的已存在行，再继续插入或更新。若违反的是 NOT NULL，则用该列默认值替换 NULL，无默认值时退化为 ABORT</font> |

`ON CONFLICT` 子句可以直接跟在约束定义之后。而在 `INSERT` 和 `UPDATE` 语句中，`ON CONFLICT` 关键字要换成 `OR`，写成 `INSERT OR IGNORE`、`UPDATE OR REPLACE` 等形式，语句级指定的算法会覆盖约束定义中指定的算法。示例如下：  

``` SQL
CREATE TABLE users (
    id    INTEGER PRIMARY KEY,
    email TEXT UNIQUE ON CONFLICT IGNORE,         -- 约束级：邮箱重复时静默跳过
    name  TEXT NOT NULL ON CONFLICT FAIL          -- 约束级：NULL 时报错但保留已修改的行
);

INSERT OR IGNORE INTO users (id, email, name) VALUES (1, 'a@x.com', 'Alice'); -- 重复时静默跳过
INSERT OR REPLACE INTO users (id, email, name) VALUES (1, 'b@x.com', 'Bob');  -- 删除旧行后插入
```

> 注意不要把这里的 `ON CONFLICT` 与 3.24.0（2018-06-04）新增的 `UPSERT` 语法（`INSERT ... ON CONFLICT ... DO ...`）相混淆，后者是 `INSERT` 语句的扩展。

## 修改表 ALTER TABLE
与其它数据库相比，SQLite 的 `ALTER TABLE` 语句功能较为有限，只支持五种修改操作：**重命名表**、**重命名列**、**添加列**、**删除列**和**修改列的 NOT NULL 约束**，基本语法如下。其它诸如修改列的数据类型、为已有列增删其它约束（如把某列改为 `UNIQUE`）之类的操作都无法通过 `ALTER TABLE` 直接完成，只能通过重建表的方式实现。  

``` SQL
ALTER TABLE [schema_name.]table_name RENAME TO new_table_name;                 -- 重命名表
ALTER TABLE [schema_name.]table_name RENAME [COLUMN] old_name TO new_name;     -- 重命名列
ALTER TABLE [schema_name.]table_name ADD  [COLUMN] column_definition;          -- 添加列
ALTER TABLE [schema_name.]table_name DROP [COLUMN] column_name;                -- 删除列
ALTER TABLE [schema_name.]table_name ALTER [COLUMN] column_name SET NOT NULL;  -- 设置 NOT NULL
ALTER TABLE [schema_name.]table_name ALTER [COLUMN] column_name DROP NOT NULL; -- 删除 NOT NULL
```

①重命名表，`RENAME TO` 语法把 *table-name* 的名称改为 *new-table-name*：  
* 该命令不能用于在已附加的多个数据库之间移动表，只能在同一个数据库内部重命名表；  
* 若被重命名的表上带有**触发器**或**索引**，重命名后这些对象仍然依附于该表。从 SQLite 3.25.0（2018-09-15）开始，**触发器**和**视图**定义中对该表的引用也会被一并重命名。从 SQLite 3.26.0（2018-12-01）开始，重命名表时，其它表的**外键约束**对该表的引用总会被同步转换；  

②重命名列，`RENAME COLUMN TO` 语法把表 *table-name* 的列名 *column-name* 改为 *new-column-name*：  
* 重命名列时，引用该列的**索引**、**触发器**、**视图**都会被同步改写；  

③添加列，`ADD COLUMN` 用于在表的末尾追加一个新列，无法指定插入位置，新列的定义规则与 `CREATE TABLE` 中的列定义基本一致，但存在如下限制：  
* 新列不能是 `PRIMARY KEY`，也不能带 `UNIQUE` 约束；  
* 若新列带 `NOT NULL` 约束，则必须为其指定一个**非 NULL 的默认值**，表中已有行的新列会被填充为该默认值；  
* 新列的默认值不能是 `CURRENT_TIME`、`CURRENT_DATE`、`CURRENT_TIMESTAMP`，也不能是括号表达式；  
* 若新列带有 `REFERENCES` 外键且外键约束已启用，其默认值必须为 NULL；  
* 生成列中只能添加 `VIRTUAL` 生成列，不能添加 `STORED` 生成列；    

④删除列，`DROP COLUMN` 用于删除一个已有列。若满足下列任一条件，删除会失败：  
* 该列是 `PRIMARY KEY` 或是其一部分，或带有 `UNIQUE` 约束，或建有**索引**；  
* 该列被表级 `CHECK` 约束、外键约束、生成列表达式，或触发器、视图、部分索引等其它模式对象所引用；  

⑤修改列，从 SQLite 3.53.0（2026-04-09）开始支持设置或删除列上的 `NOT NULL` 约束：  
* 若该列已经是 `NOT NULL`，`SET NOT NULL` 为空操作，不会添加重复的约束；  
* 若该列上存在多个 `NOT NULL` 约束（通过 `CREATE TABLE` 定义时可能出现），`DROP NOT NULL` 保证会移除其中一个或多个，但不保证全部移除；  

### 重建表
对于 `ALTER TABLE` 无法直接完成的修改（如更改列的数据类型、增删列约束等），官方推荐按**重建表**的流程操作，如下所示：  

``` SQL
PRAGMA foreign_keys = OFF;                 -- 关闭外键约束，避免迁移过程受干扰
BEGIN TRANSACTION;                         -- 开启事务

CREATE TABLE new_users (                   -- 按新结构创建临时表
    id    INTEGER PRIMARY KEY,
    email TEXT NOT NULL UNIQUE,            -- 例如这里为已有列新增了 UNIQUE 约束
    name  TEXT NOT NULL DEFAULT 'anonymous'
);

INSERT INTO new_users (id, email, name)    -- 迁移旧数据
SELECT id, email, name FROM users;

DROP TABLE users;                          -- 删除旧表
ALTER TABLE new_users RENAME TO users;     -- 新表改回原名

CREATE INDEX idx_users_email ON users (email);   -- 旧表上的索引已随 DROP TABLE 一并消失，需重新创建
CREATE VIEW  v_users AS SELECT id, email, name FROM users; -- 视图不会被 DROP TABLE 删除，但若其定义引用了已变的表结构则需重建
CREATE TRIGGER trg_users_email_lower             -- 旧表上的触发器同样已消失，需重新创建
AFTER INSERT ON users
BEGIN
    UPDATE users SET email = lower(NEW.email) WHERE id = NEW.id;
END;

PRAGMA foreign_key_check;                  -- 校验外键完整性
COMMIT;                                    -- 提交事务
PRAGMA foreign_keys = ON;                  -- 恢复外键约束
```

> 删除旧表会连带删除其上的索引与触发器，需在事后手动重建。视图虽不会被 `DROP TABLE` 删除，但若其定义引用了旧表结构也需一并重建。另外务必先关闭外键约束，否则删除旧表时可能触发其它表外键的级联动作。

## 删除表 DROP TABLE
使用 `DROP TABLE` 语句删除一个已存在的表，基本语法如下：  

``` SQL
DROP TABLE [IF EXISTS] [schema_name.]table_name;
```

* 若指定的表不存在则会报错，使用 `IF EXISTS` 选项可以在表不存在时静默跳过，不产生任何错误；  
* 删除表时，表中的全部数据、建立在该表上的**索引**与**触发器**都会被一并删除。引用该表的**视图**则不会被删除，但此后查询该视图时会报错；  
* 若外键约束已启用，`DROP TABLE` 命令会在把表从数据库模式中移除之前，先执行一次隐式的 `DELETE FROM` 命令。表上附着的任何**触发器**都会在隐式 `DELETE FROM` 执行之前先从数据库模式中删除，因此这一过程不可能触发任何触发器。但会引发所有已配置的**外键动作**。如果作为 `DROP TABLE` 命令的一部分而执行的隐式 `DELETE FROM` 违反了任何立即外键约束，则会返回错误并且表不会被删除。如果隐式 `DELETE FROM` 导致任何延迟外键约束被违反，且这些违反在事务提交时依然存在，则会在提交时报错；   
* 删除表只是把其占用的页放回**空闲列表 free list**，并不会缩小数据库文件，如需回收磁盘空间需另行执行 `VACUUM` 命令。

# 数据操作
数据操作语句即 **data manipulation language (DML)**，负责对表中已有的数据进行读写，包括查询 `SELECT`、插入 `INSERT`、更新 `UPDATE` 与删除 `DELETE` 四类。与之相对，前面介绍的 `CREATE TABLE`、`ALTER TABLE` 等属于 **data definition language (DDL)**。本节按查、增、改、删的顺序依次介绍这四类语句。

## 查询 SELECT
**查询 SELECT** 是 SQL 中使用频率最高的语句，用于从一个或多个表中检索数据，它只读取数据而不修改数据库。SQLite 中 `SELECT` 语句的完整语法非常复杂，核心形式大致如下（其中中括号 `[]` 表示可选）：  

``` SQL
[WITH [RECURSIVE] common_table_expression]
SELECT [ALL | DISTINCT] column_list
FROM table_list
[JOIN other_table ON join_condition]
[WHERE search_condition]
[GROUP BY column_list]
[HAVING search_condition]
[ORDER BY column_list [ASC | DESC]]
[LIMIT count [OFFSET skip]];
```

* `SELECT` 子句的 *column_list* 指定要返回的列或表达式。`FROM` 子句指定数据来源，可以是表、视图或子查询 subquery 等；  
* 各子句的书写顺序是固定的，不能随意调换，但书写顺序不等于逻辑语义顺序，基本的逻辑语义顺序大致为：① `WITH` 定义**公共表表达式 CTE**，供后面的主查询引用；② `FROM / JOIN` 确定数据来源并连接多表，得到候选行集合；③ `WHERE` 对候选行逐行过滤，仅保留满足条件的行；④ `GROUP BY` 把剩余行按分组列划分为若干组；⑤ `HAVING` 对每个组求值其过滤条件，丢弃不满足条件的组，条件中若含聚合函数，则在该组的全部行上求值；⑥ `SELECT` 计算输出表达式，生成结果集；⑦ `DISTINCT` 对结果集去重；⑧ `ORDER BY` 对结果集排序；⑨ `LIMIT` 与 `OFFSET` 截取要返回的行。

### SELECT 子句
`SELECT` 子句中可以使用 `*` 返回全部列，也可以列出具体的列名、使用表达式或调用内置函数。另外，可以使用 `AS` 关键字为列或表指定**别名 alias**，列别名会作为结果集的列名显示，表别名则用于在语句的其余部分引用该表。  

``` SQL
SELECT * FROM employees;                          -- 返回全部列
SELECT id, name FROM employees;                   -- 返回指定列
SELECT e.* FROM employees AS e;                   -- 返回指定表的所有列
SELECT e.id, e.name FROM employees AS e;          -- 返回指定表的指定列
SELECT DISTINCT name FROM employees;              -- DISTINCT 去重
SELECT COUNT(DISTINCT id) FROM employees;         -- 聚合函数 COUNT 统计不同 id 的行数

SELECT name AS n, salary * 12 AS annual_salary    -- 表达式 + 列别名
FROM employees e;                                 -- 表别名，省略 AS 关键字

SELECT 1 + 1, 'hello', datetime('now');           -- 无 FROM 时直接对表达式求值
```

* 若省略 `FROM` 子句，`SELECT` 直接对表达式列表求值并返回单行结果，常用于测试函数或查看环境信息；  
* 当 `SELECT` 的表达式列表中出现**聚合函数 aggregate function**（如 `count()`、`sum()`、`avg()`）时，该查询即为**聚合查询**。若不带 `GROUP BY` 子句，则聚合表达式会在整个数据集上求值一次，查询总是恰好返回一行结果，具体详见 [GROUP BY 子句](#GROUP-BY-子句)小节；  
* 指定别名时，`AS` 关键字可以省略。别名若为关键字或包含空格等特殊字符，需用双引号括起来，如 `AS "user name"`，注意标识符用双引号，与字符串的单引号不同；  
* `DISTINCT` 关键字用于去除结果集中的重复行，与之相对的 `ALL` 是默认行为，即返回全部行，通常省略不写。当列出多列时，`DISTINCT` 按所有列的组合整体判断是否重复。与 `UNIQUE` 约束不同，`DISTINCT` 判断重复时把 `NULL` 视为相等，结果中至多保留一个 `NULL`。 

### FROM 子句
`FROM` 子句用于指定查询的数据来源，常见的数据来源对象有如下：  

| <font size=2>数据来源</font> | <font size=2>写法</font> | <font size=2>说明</font> |
| :---- | :---- | :---- |
| <font size=2>表 table</font> | <font size=2>`FROM table_name [AS alias]`</font> | <font size=2>最常用，可用 `schema_name.table_name` 限定所属数据库</font> |
| <font size=2>视图 view</font> | <font size=2>`FROM view_name [AS alias]`</font> | <font size=2>用法与表完全一致</font> |
| <font size=2>子查询 subquery</font> | <font size=2>`FROM (SELECT ...) [AS alias]`</font> | <font size=2>可当作一张临时表使用</font> |
| <font size=2>表值函数 table-valued function</font> | <font size=2>`FROM func(args) [AS alias]`</font> | <font size=2>如 `json_each()`、`json_tree()`，会在结果中展开成多行</font> |
| <font size=2>常量列表 VALUES</font> | <font size=2>`FROM (VALUES (...), ...) [AS alias]`</font> | <font size=2>在查询中直接构造少量常量行，无需建表</font> |

* 数据来源可以指定所属的 *schema_name*，不写时 SQLite 会按 `temp`、`main`、已附加数据库的顺序依次查找，[附加数据库](#附加数据库-ATTACH-DATABASE)小节提过这点；    
* 可以在 `FROM` 中并列写出多个数据源并用逗号分隔，其效果等价于对它们做**交叉连接 CROSS JOIN**。可以将连接条件写在 `WHERE` 中，但更推荐显式的 `JOIN ... ON` 写法，详见 [JOIN 子句](#JOIN-子句)小节；  
* `VALUES` 子句生成的列有默认名称，依次为 column1、column2 …………，可以在外层 `SELECT` 引用并使用 `AS` 指定别名；  

``` SQL
SELECT * FROM users;                                  -- 单张表
SELECT * FROM user_summary_view;                      -- 视图
SELECT * FROM main.users;                             -- 指定 schema_name

SELECT e.name, d.name
FROM employees AS e, departments AS d                 -- 逗号即交叉连接，连接条件写在 WHERE 中
WHERE e.dept_id = d.id;

SELECT t.id, t.n                                      -- 子查询作数据源
FROM (SELECT dept_id AS id, name AS n FROM employees) AS t;

SELECT column1 AS id, column2 AS name
FROM (VALUES (1, 'Alice'), (2, 'Bob'));               -- VALUES 构造常量表

SELECT key, value FROM json_each('{"a": 1, "b": 2}'); -- 表值函数：把 JSON 展开成多行
```

#### JOIN 子句
`JOIN` 子句属于 `FROM` 子句的一部分，用于把两个数据源（表、视图、子查询等）组合起来。该子句主要包括**连接运算符 join-operator** 和**连接约束 join-constraint** 两个部分，语法大致为 `FROM table join-operator table join-constraint`，连接运算符前后的表可以称为左表与右表。连接约束包括 `ON` 和 `USING` 子句，连接运算符有多个类型，如下表所示：  

| <font size=2>连接类型</font> | <font size=2>关键字</font> | <font size=2>说明</font> |
| :---- | :---- | :---- |
| <font size=2>内连接 inner join</font> | <font size=2>`[INNER] JOIN`</font> | <font size=2>只保留两边能配上对的行</font> |
| <font size=2>左外连接 left outer join</font> | <font size=2>`LEFT [OUTER] JOIN`</font> | <font size=2>左表的行全部保留，当右表配不上时右表的列填 NULL</font> |
| <font size=2>右外连接 right outer join</font> | <font size=2>`RIGHT [OUTER] JOIN`</font> | <font size=2>右表的行全部保留，当左表配不上时左表的列填 NULL</font> |
| <font size=2>全外连接 full outer join</font> | <font size=2>`FULL [OUTER] JOIN`</font> | <font size=2>两边配不上的行都保留</font> |
| <font size=2>自然连接 natural join</font> | <font size=2>`NATURAL INNER/LEFT/RIGHT/FULL JOIN`</font> | <font size=2>不必写连接条件，自动按两表所有同名列做等值连接</font> |
| <font size=2>交叉连接 cross join</font> | <font size=2>`CROSS JOIN` 或 `FROM a, b`</font> | <font size=2>不做配对，两边行两两组合，即笛卡尔积</font> |

* 上表中的说明其实有一定误导性，所有连接本质上都是基于左右表的**笛卡尔积 cartesian product** 的，即若左表由 NL 行 ML 列组成，右表由 NR 行 MR 列组成，则笛卡尔积是一个 NL × NR 行、ML + MR 列的数据集。<u>在不使用 `ON` 和 `USING` 子句的前提下</u>，使用 `CROSS JOIN`、`INNER JOIN`、`JOIN` 和逗号 `,` 得到的都是笛卡尔积（我自己测试使用三个外连接得到的也是笛卡尔积，但官网只写了这几种）。`CROSS JOIN` 连接运算符产生的结果与 `INNER JOIN`、`JOIN` 和逗号 `,` 相同，区别在于它会阻止查询优化器调整连接中各表的顺序；  
* 若有 `ON` 子句，则会对笛卡尔积的每一行把 `ON` 表达式作为布尔表达式求值，只有求值为真的行才会被纳入结果数据集；  
* 若有 `USING` 子句，其语法为 `USING (column-name, ...)`，则其中指定的每一个列名都必须同时存在于左右表当中，对于每一对同名的列，都会对笛卡尔积的每一行对布尔表达式 `lhs.X = rhs.X` 求值，只有当所有这些表达式的求值结果都为真时，该行才会被纳入结果集，相当于一个特殊的 `ON` 子句；  
* 如果连接运算符中包含 `NATURAL` 关键字，则会向连接约束中添加一个隐式的 `USING` 子句。该隐式的 `USING` 子句包含同时出现在左右表中的每一个列名。指定了 `NATURAL` 关键字的连接不能同时添加 `USING` 或 `ON` 子句；   
* 连接 `JOIN` 本身并不会自动为连接列建立任何索引，若 `ON` 或 `USING` 子句中的等值条件所涉及的列上没有索引，SQLite 就只能对左表的每一行去全表扫描右表（即嵌套循环），被检查的行组合数恰好就是笛卡尔积的规模，时间复杂度为 O(NL × NR)。<u>为连接列建立索引后，SQLite 可以直接在该索引中定位到匹配的行，把逐行扫描变成定点查找，每处理左表一行只需 O(log NR) 的时间在 B-Tree 索引中检索，整体复杂度由 O(NL × NR) 降为 O(NL × log NR)</u>，往往是提升连接查询性能最直接的手段，详见进阶特性章节的[索引](#索引-Index)小节；  
* `RIGHT JOIN` 与 `FULL JOIN` 自 SQLite 3.39.0（2022-06-25）起才支持，更早的版本若需要这两种语义，前者可以把两张表对调后改写为 `LEFT JOIN`，后者可以用 `UNION` 合并左右连接的结果来模拟；  

``` SQL
-- 假设数据为 employees(name|id)：Alice|1、Bob|1、Carol|2、Dave|NULL、Eve|2
-- departments(id|dept_name)：1|研发部、2|市场部、3|财务部

-- 内连接：Alice|研发部、Bob|研发部、Carol|市场部、Eve|市场部
SELECT e.name, d.dept_name
FROM employees AS e
JOIN departments AS d ON e.id = d.id;

-- 左外连接：Alice|研发部、Bob|研发部、Carol|市场部、Dave|NULL、Eve|市场部
SELECT e.name, d.dept_name
FROM employees AS e
LEFT JOIN departments AS d ON e.id = d.id;

-- 右外连接：Alice|研发部、Bob|研发部、Carol|市场部、Eve|市场部、NULL|财务部
SELECT e.name, d.dept_name
FROM employees AS e
RIGHT JOIN departments AS d ON e.id = d.id;

-- 全外连接：Alice|研发部、Bob|研发部、Carol|市场部、Dave|NULL、Eve|市场部、NULL|财务部
SELECT e.name, d.dept_name
FROM employees AS e
FULL JOIN departments AS d ON e.id = d.id;

-- 交叉连接：笛卡尔积，共 5 × 3 = 15 行
SELECT e.name, d.dept_name
FROM employees AS e
CROSS JOIN departments AS d;

-- USING：相当于一个特殊的 ON 子句，结果等同于 ON e.id = d.id
SELECT e.name, d.dept_name
FROM employees AS e
JOIN departments AS d USING (id);

-- NATURAL：相当于 USING (id)
SELECT e.name, d.dept_name
FROM employees AS e
NATURAL JOIN departments AS d;
```

### WHERE 子句
如果指定了 `WHERE` 子句，则会针对输入数据中的每一行把 `WHERE` 表达式作为布尔表达式求值。只有使 `WHERE` 子句表达式的求值结果为真的行，才会被纳入数据集并继续后续处理，<u>如果 `WHERE` 子句的求值结果为假或为 `NULL`，该行都会被排除在结果之外</u>。

对于 JOIN、INNER JOIN 或 CROSS JOIN，`WHERE` 子句中与 `ON` 子句中没有区别。但对于 LEFT、RIGHT 或 FULL JOIN 这些外连接来说是有区别的。在外连接中，为没有匹配上的行补充的 NULL，是在 `ON` 子句处理之后、`WHERE` 子句处理之前加入的。因此写在 `ON` 子句中的形如 `left.x = right.y` 的约束，会允许这些补充进来的 NULL 行通过。但如果同一约束写在 `WHERE` 子句中，`right.y` 或 `left.x` 中的 NULL 会使表达式 `left.x = right.y` 不为真，从而把该行排除在输出之外。

布尔表达式中可以使用比较运算符、逻辑运算符以及若干谓词，常用运算符如下表：  

| <font size=2>运算符</font> | <font size=2>说明</font> |
| :---- | :---- |
| <font size=2>`=`、`==`、`!=`、`<>`、`<`、`<=`、`>`、`>=`</font> | <font size=2>`=` 与 `==` 等价，`!=` 与 `<>` 等价。任何值与 `NULL` 比较结果均为 `NULL` 而非真或假</font> |
| <font size=2>`IS`、`IS NOT`、`IS DISTINCT FROM`、`IS NOT DISTINCT FROM`</font> | <font size=2>`IS` 与 `IS NOT DISTINCT FROM` 等价，`IS NOT` 与 `IS DISTINCT FROM` 等价</font> |
| <font size=2>`x BETWEEN a AND b`</font> | <font size=2>闭区间，等价于 `x >= a AND x <= b`</font> |
| <font size=2>`[NOT] LIKE`、`[NOT] GLOB`、`REGEXP`、`MATCH`</font> | <font size=2>文本模式匹配</font> |
| <font size=2>`IN`、`NOT IN`</font> | <font size=2>判断值是否在给定列表或子查询结果中</font> |
| <font size=2>`ISNULL`、`IS NULL`、`IS NOT NULL`、`NOTNULL`、`NOT NULL`</font> | <font size=2>判断是否为 `NULL`，不能用 `= NULL` 判断</font> |
| <font size=2>`AND`、`OR`、`NOT`</font> | <font size=2>优先级为 `NOT` > `AND` > `OR`，可用括号改变优先级</font> |

* `IS`、`IS NOT` 等价于 `=`、`!=`，除非有操作数是 null。此时两个操作数都是 NULL 的话，`IS` 运算符求值为 1（真），`IS NOT` 运算符求值为 0（假）；如果其中一个操作数是 NULL 而另一个不是，则 `IS` 运算符求值为 0（假），`IS NOT` 运算符求值为 1（真）。`IS` 或 `IS NOT` 表达式的求值结果不可能是 NULL。
* `LIKE`、`GLOB`、`REGEXP`、`MATCH` 用于文本的**模式匹配 pattern matching**：  
&emsp;&emsp;①`LIKE` 使用两个通配符：`%` 匹配任意长度（含零长度）的任意字符序列，`_` 匹配任意单个字符。默认不区分大小写，但 SQLite 默认只理解 ASCII 字母的大小写，也就是说 `'a' LIKE 'A'` 为真，但 `'æ' LIKE 'Æ'` 为假。另外，可以用 `ESCAPE` 子句指定一个转义字符，让 `%` 与 `_` 变成普通字符；  
&emsp;&emsp;②`GLOB` 行为跟 `LIKE` 类似，但使用 Unix 风格的通配符：`*` 匹配任意字符序列，`?` 匹配任意单个字符，`[...]` 匹配字符集合，还有很多这里就不一一例举了。与 `LIKE` 不同，`GLOB` 始终区分大小写；    
&emsp;&emsp;③`REGEXP` 运算符是用户函数 `regexp()` 的一种特殊语法。默认情况下并没有定义 `regexp()` 用户函数，因此使用 `REGEXP` 运算符通常会得到一条错误信息。如果在运行时添加了一个名为 regexp 的应用程序自定义 SQL 函数，那么 `X REGEXP Y` 运算符就会被实现为对 `regexp(Y, X)` 的调用；  
&emsp;&emsp;④`MATCH` 运算符是应用程序自定义函数 `match()` 的一种特殊语法。默认的 `match()` 函数实现会抛出异常，实际上没什么用处。但扩展可以用更有用的逻辑来覆盖 `match()` 函数。  
* `IN` 与 `NOT IN` 运算符左侧接一个表达式，右侧接一个值列表或一个子查询。当右操作数是空集合时，无论左操作数是什么，甚至左操作数是 NULL，`IN` 的结果都是假，而 `NOT IN` 的结果都是真。`IN` 或 `NOT IN` 运算符的结果由下表格决定：  

| <font size=2>左操作数为 `NULL`</font> | <font size=2>右操作数含 `NULL`</font> | <font size=2>右操作数为空集</font> | <font size=2>左操作数在右操作数中被找到</font> | <font size=2>`IN` 运算符的结果</font> | <font size=2>`NOT IN` 运算符的结果</font> |
| :---- | :---- | :---- | :---- | :---- | :---- |
| <font size=2>否</font> | <font size=2>否</font> | <font size=2>否</font> | <font size=2>否</font> | <font size=2>假</font> | <font size=2>真</font> |
| <font size=2>无关紧要</font> | <font size=2>否</font> | <font size=2>是</font> | <font size=2>否</font> | <font size=2>假</font> | <font size=2>真</font> |
| <font size=2>否</font> | <font size=2>无关紧要</font> | <font size=2>否</font> | <font size=2>是</font> | <font size=2>真</font> | <font size=2>假</font> |
| <font size=2>否</font> | <font size=2>是</font> | <font size=2>否</font> | <font size=2>否</font> | <font size=2>NULL</font> | <font size=2>NULL</font> |
| <font size=2>是</font> | <font size=2>无关紧要</font> | <font size=2>否</font> | <font size=2>无关紧要</font> | <font size=2>NULL</font> | <font size=2>NULL</font> |

``` SQL
SELECT * FROM employees WHERE dept_id = 3 AND salary > 5000;
SELECT * FROM employees WHERE salary BETWEEN 3000 AND 8000;
SELECT * FROM employees WHERE name IN ('Alice', 'Bob', 'Carol');
SELECT * FROM employees WHERE phone IS NULL;
SELECT * FROM users WHERE email LIKE '%@gmail.com';     -- 以 @gmail.com 结尾
SELECT * FROM users WHERE name LIKE '_ike';             -- 恰好四个字母且后三个为 ike
SELECT * FROM files WHERE name LIKE '100\%' ESCAPE '\'; -- 匹配 100%
SELECT * FROM files WHERE name GLOB '*.txt';            -- 区分大小写的后缀匹配
```

### GROUP BY 子句
`GROUP BY` 子句主要用于**聚合查询 aggregate query**，即使用**聚合函数 aggregate function** 对多行数据进行计算，最终返回汇总结果的查询，通常会根据多行返回单行，SQLite 内置的常用聚合函数如下表：  

| <font size=2>函数</font> | <font size=2>说明</font> |
| :---- | :---- |
| <font size=2>`avg(X)`</font> | <font size=2>求非 NULL 行的平均值，不像数字的 BLOB 和 TEXT 值视作 0，并且始终返回浮点数，若都是 NULL 则返回 NULL</font> |
| <font size=2>`count(*)`、`count(X)`</font> | <font size=2>`count(*)` 统计所有行的行数，`count(X)` 只统计列中非 NULL 行的行数</font> |
| <font size=2>`max(X)`、`min(X)`</font> | <font size=2>求最大值或最小值，若都是 NULL 则返回 NULL</font> |
| <font size=2>`median(X)`</font> | <font size=2>返回所有非 NULL 值的中位数</font>  |
| <font size=2>`sum(X)`、`total(X)`</font> | <font size=2>对非 NULL 值求和。`SUM` 返回整数若非 NULL 行都是整数，否则返回浮点数，且都是 NULL 时返回 NULL。而 `TOTAL` 始终返回浮点数、都是 NULL 时返回 0.0</font> |
| <font size=2>`group_concat(X, Y)`</font> | <font size=2>把各行的非 NULL 值用分隔符 Y 拼接为一个字符串，Y 省去的话默认为逗号 `,`</font> |
| <font size=2>`string_agg(X, Y)`</font> | <font size=2>`group_concat(X, Y)` 的别名</font> |

* 如果 `SELECT` 语句是<u>不带 `GROUP BY` 子句的聚合查询</u>，那么每个聚合表达式都会在整个数据集上求值一次，每个非聚合表达式则会针对数据集中任意选取的一行求值一次，并且所有非聚合表达式使用的都是同一行。聚合表达式与非聚合表达式求值所得到的那唯一一行结果数据，就构成了不带 `GROUP BY` 子句的聚合查询的结果。不带 `GROUP BY` 子句的聚合查询总是恰好返回一行数据，即使输入数据零行也是如此；  
* 如果 `SELECT` 语句是<u>带 `GROUP BY` 子句的聚合查询</u>，那么每一行都会根据 `GROUP BY` 子句中的每一个表达式（不能是聚合表达式）的求值结果划入一个组。随后，`SELECT` 的每个表达式都会针对每一组行求值一次，如果该表达式是聚合表达式，它会在该组的所有行上求值，否则，它会针对从该组内任意选取的一行求值。如果结果集中存在多个非聚合表达式，那么这些表达式都针对同一行求值。最后每一组行都会为结果贡献一行；  
* 聚合函数还可以搭配三个可选内容：`DISTINCT` 关键字用于在聚合前先去重，例如 `count(DISTINCT X)` 返回列 X 不同值的数量；`ORDER BY` 子句写在最后一个参数之后，括号之内，用于指定聚合处理行的顺序，对 `max()`、`count()` 这类结果与顺序无关，但对 `group_concat()`、`string_agg()` 这类拼接字符串的函数会直接影响结果；`FILTER (WHERE expr)` 子句写在括号之后，只有使 `expr` 为真的行才会被纳入聚合。  

``` SQL
-- 假设 employees 表的数据为（name | dept_id | salary）：
-- Alice|1|12000、Bob|1|9000、Carol|2|15000、Dave|2|8000、Eve|NULL|11000

SELECT dept_id,
       count(*)                                   AS 人数,     -- 该组的行数
       sum(salary)                                AS 工资合计,  -- 该组 salary 之和
       avg(salary)                                AS 平均工资,  -- 该组 salary 平均值
       max(salary)                                AS 最高工资,  -- 该组 salary 的最大值
       sum(salary) FILTER (WHERE salary >= 10000) AS 高薪合计,  -- FILTER 子句：只累加该组中工资 ≥ 10000 的行
       group_concat(name, '/' ORDER BY salary)    AS 成员      -- ORDER BY 子句：按 salary 升序拼接该组姓名
FROM employees
GROUP BY dept_id;
```

输出：  

``` Console
dept_id | 人数 | 工资合计 | 平均工资 | 最高工资 | 高薪合计 |  成员
--------+------+----------+--------+---------+----------+-----------
1       | 2    | 21000    | 10500  | 12000   | 12000    | Bob/Alice
2       | 2    | 23000    | 11500  | 15000   | 15000    | Dave/Carol
NULL    | 1    | 11000    | 11000  | 11000   | 11000    | Eve
```

#### HAVING 子句
`HAVING` 子句在分组之后对组进行过滤，通常配合聚合函数使用，其地位相当于分组版的 `WHERE`，但 `WHERE` 中不能使用聚合函数。它会针对每一组行作为布尔表达式求值一次。如果 `HAVING` 子句的求值结果为假，该组就会被丢弃。如果 `HAVING` 子句是聚合表达式，它会在该组的所有行上求值；如果 `HAVING` 子句是非聚合表达式，则会针对从该组中任意选取的一行求值。沿用上面 `GROUP BY` 例子的数据，示例如下：  

> 如果条件既能放 `WHERE` 又能放 `HAVING`，优先放 `WHERE`，因为先过滤行可以减少分组数据量，提高性能。

``` SQL
SELECT dept_id,
       count(*)    AS 人数,
       avg(salary) AS 平均工资
FROM employees
GROUP BY dept_id
HAVING count(*) >= 2;
```

输出：  

``` Console
dept_id | 人数 | 平均工资
--------+------+----------
1       | 2    | 10500.0
2       | 2    | 11500.0
```

### ORDER BY 子句
`ORDER BY` 子句用于对 `SELECT` 查询返回的结果集排序，可以指定一个或多个排序键，每个键后可跟 `ASC`（升序，默认）或 `DESC`（降序），以及 `NULLS FIRST` 或 `NULLS LAST` 指定 NULL 排在最前还是最后。指定多个键时先按第一个键排，第一个键相同再按第二个键排，依此类推。另外，排序键还可以直接写列的序号（比如 1，2）或列别名来代替列名，还可以在排序键后接 `COLLATE` 临时指定排序规则（详见 [CREATE TABLE 的 COLLATE](#COLLATE-子句) 小节）。  

``` SQL
SELECT * FROM employees ORDER BY dept_id ASC, salary DESC; -- 先按部门升序，再按工资降序
SELECT * FROM employees ORDER BY dept_id ASC NULLS LAST; -- 显式指定 NULL 排在最后
SELECT name, salary * 12 AS annual_salary FROM employees ORDER BY annual_salary DESC; -- 按列别名排序
SELECT name, salary FROM employees ORDER BY 2 DESC; -- 按结果第 2 列排序
SELECT * FROM users ORDER BY name COLLATE NOCASE DESC; -- 忽略大小写排序
SELECT * FROM users ORDER BY lower(name) DESC; -- 表达式排序
```

* 不写 `ORDER BY` 时，结果行的返回顺序是不确定的，即使某次查询看似按插入顺序返回，也不应依赖这一行为；  
* 默认情况下 `NULL` 被视为最小值，因此升序时默认 `NULL` 排在最前，降序时默认排在最后。从 SQLite 3.30.0（2019-10-04）起，可以用 `NULLS FIRST` 或 `NULLS LAST` 显式指定 `NULL` 的位置；  
* 字符串按默认的 `BINARY` 排序规则逐字节比较，因此顺序为数字 < 大写字母 < 小写字母（如 `'A' < 'Z' < 'a'`），`DESC` 则与之完全相反。若想忽略大小写排序，可以使用 `COLLATE NOCASE`；  
* 时间若以 TEXT 存储，只有遵循 ISO 格式 `"YYYY-MM-DD HH:MM:SS.SSS"` 时，字典序才与时间先后一致，否则排序结果会错乱，例如 `'2026/09/04'` 就无法正确比较。而 REAL（儒略日）与 INTEGER（Unix 时间戳）本就是数值，直接按数值大小排序。  

### LIMIT 与 OFFSET 子句
`LIMIT` 子句用于限制 `SELECT` 查询返回的行数，可选的 `OFFSET` 子句用于跳过结果集中指定数量的行，两者结合常用于分页。`LIMIT` 有两种等价写法，`LIMIT 数量 OFFSET 偏移量` 先写取多少行，再写跳过多少行，或者 `LIMIT 偏移量, 数量` 先写跳过多少行，再写取多少行，如下：  

``` SQL
SELECT * FROM employees ORDER BY salary DESC LIMIT 10;     -- 工资最高的前 10 行
SELECT * FROM employees ORDER BY id LIMIT 10 OFFSET 20;    -- 跳过 20 行后取 10 行
SELECT * FROM employees ORDER BY id LIMIT 20, 10;          -- 与上一句等价：LIMIT offset, count

SELECT * FROM employees LIMIT -1 OFFSET 5;                 -- LIMIT 为负数表示不限制行数
```

* 建议分页查询配合 `ORDER BY` 使用，否则返回的行的顺序是不确定的，多次返回内容可能不一致；  
* `LIMIT` 与 `OFFSET` 的值都可以是标量表达式，如 `LIMIT 2 * 5`。若表达式的值为 NULL 或无法无损转换为整数则会报错；  
* 若 `LIMIT` 表达式求值为负数，则返回的行数没有上限。若 `OFFSET` 与 `LIMIT` 子句的值分别为 M 和 N，且 `SELECT` 返回的行数少于 M + N 行，则跳过前 M 行，并返回剩余的行（如果还有的话）；  
* `OFFSET` 子句是 `LIMIT` 的可选语句，故不允许单独写 `OFFSET` 而不写 `LIMIT`，此时应使用 `LIMIT -1 OFFSET n`。  

### CASE 表达式
`CASE` 表达式扮演的角色类似于其它编程语言中的 IF-THEN-ELSE，根据一系列条件的真假返回不同的值。它可以出现在任何允许标量表达式的地方，例如 `SELECT`、`WHERE`、`GROUP BY`、`ORDER BY`、`UPDATE`、`INSERT`、`DELETE` 等。它有**简单形式 simple CASE** 和**搜索形式 searched CASE** 两种写法：  

``` SQL
-- 简单形式：把 CASE 后的表达式依次与各 WHEN 的值做相等比较
CASE base_expr 
    WHEN value_1 THEN result_1 
    [WHEN value_2 THEN result_2]
    ...
    [ELSE default_result] 
END

-- 搜索形式：各 WHEN 后为独立的布尔表达式
CASE 
    WHEN expr_1 THEN result_1 
    [WHEN expr_2 THEN result_2]
    ...
    [ELSE default_result] 
END

-- 简单形式等价于搜索形式的以下写法
CASE
    WHEN base_expr = value1 THEN result1
    [WHEN base_expr = value2 THEN result2]
    ...
    [ELSE default_result]
END
```

* `CASE` 是按顺序依次求值的，并且具有短路特性，一旦某个 `WHEN` 匹配成功就立即返回对应的 `THEN` 结果，后续分支不再求值。若没有任何分支匹配，则返回 `ELSE` 的结果，省略 `ELSE` 时返回 `NULL`；  
* simple CASE 和 searched CASE 的唯一区别就是，simple CASE 只计算一次 base 表达式，而 searched CASE 可能会求多次；  
* 若基础表达式为 NULL，那么 `CASE` 的结果总是 `ELSE` 表达式的求值结果，并且 `NULL = NULL` 的结果是 NULL。要想判断 NULL，必须使用 `IS NULL` 或 `IS NOT NULL`；  
* 内置函数 `iif(x,y,z)` 在逻辑上等价于 `CASE WHEN x THEN y ELSE z END`。  

一个简单的示例如下（沿用 [GROUP BY 子句](#GROUP-BY-子句)小节的示例中的数据）：  

``` SQL
SELECT name, salary,
       CASE
           WHEN salary < 10000 THEN '低'
           WHEN salary < 12000 THEN '中'
           ELSE '高'
       END AS level
FROM employees;

SELECT SUM(CASE WHEN salary > 10000 THEN 1 ELSE 0 END) AS high_salary
FROM employees;
```

输出：  

``` Console
name  | salary | level |
------+--------+-------+
Alice | 12000  | 高    |
Bob   | 9000   | 低    |
Carol | 15000  | 高    |
Dave  | 8000   | 低    |
Eve   | 11000  | 中    |

high_salary |
------------+
3           |
```

### 子查询 Subquery
**子查询 subquery** 是嵌套在其它 SQL 语句中的 `SELECT` 语句，通常必须用括号 `()` 包起来。子查询可以出现在 `SELECT`、`FROM`、`WHERE`、`HAVING`、`JOIN`、`INSERT`、`UPDATE`、`DELETE` 等位置。根据返回结果可以大致分为返回单个值的**标量子查询 scalar subquery**、返回列的**列子查询 column subquery**、返回表的**表子查询 table subquery**，下面尽可能得多给出相关示例：  

``` SQL
-- 标量子查询，常见于 SELECT、WHERE、HAVING
SELECT * FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees); -- 查询高于平均工资的员工

SELECT name,
       (SELECT COUNT(*) FROM employees AS e WHERE e.dept_id = d.id) AS employee_count
FROM departments AS d; -- 查询每个部门中员工的数量

-- 列子查询，常见于 IN、NOT IN
SELECT * FROM employees
WHERE dept_id IN (SELECT id FROM departments WHERE location = '北京'); -- 查询部门在北京的员工

-- 表子查询，常见于 FROM、JOIN
SELECT t.dept_id, t.avg_sal
FROM (SELECT dept_id, AVG(salary) AS avg_sal FROM emp GROUP BY dept_id) AS t
WHERE t.avg_sal > 10000; -- 查询员工平均工资大于 10000 的部门

SELECT d.name, t.cnt
FROM departments AS d
JOIN (SELECT dept_id, COUNT(*) AS cnt FROM employees GROUP BY dept_id) AS t
ON d.id = t.dept_id; -- 查询部门中员工的数量
```

当子查询出现在 `SELECT` 语句的 `FROM` 子句中时，SQLite 可能会将子查询的结果存储在一个临时表中，然后对这个临时表执行外部 SELECT 语句。但因为临时表没有任何索引，而外部查询（很可能是一个连接）将被迫对临时表进行全表扫描，或者在临时表上创建查询时索引，这两种方法的速度都不够快。为此，SQLite 会尽可能**展平 flatten** 子查询：把子查询的 `FROM` 子句并入外层查询，并改写外层中引用子查询结果的表达式，使二者一起被优化。当子查询无法展平时，SQLite 可能会改用**协程 co-routine** 逐行向外层提供结果，比物化成临时表更省内存、首行输出也更早。同时，还会尝试**条件下推 predicate push-down**，把外层 `WHERE` 条件推进子查询内部以缩小结果集。

子查询还可以分为**相关子查询 correlated subquery** 和**非相关子查询 uncorrelated subquery**，引用外层查询中列的子查询，即称为相关子查询，常见于标量子查询或用作 `IN`、`NOT IN`、`EXISTS`、`NOT EXISTS` 表达式右操作数的 `SELECT` 语句。相关子查询每处理一行，子查询就执行一次，性能通常更差。而非相关子查询可以独立执行，不引用外层列，并且只求值一次，其结果按需重复使用。

`EXISTS` 运算符是一个布尔函数，主要用于检查一个子查询是否返回了至少一行数据，如果子查询返回了任何行，则结果为 1，反之为 0。子查询的结果每一行有多少列、以及具体返回什么值，都不影响 `EXISTS` 运算符的结果。特别是，含有 `NULL` 值的行与不含 `NULL` 值的行没有任何区别。因此子查询的选择列表通常习惯写成 `SELECT 1` 或 `SELECT *`。另外，`EXISTS` 几乎总是与相关子查询配合使用。

``` SQL
SELECT * FROM departments AS d
WHERE EXISTS (SELECT 1 FROM employees AS e WHERE e.dept_id = d.id); -- 存在员工的部门

SELECT * FROM departments AS d
WHERE NOT EXISTS (SELECT 1 FROM employees AS e WHERE e.dept_id = d.id); -- 不存在员工的部门
```

> `EXISTS` 与 `IN` 在多数情况下可以互相改写，但当子查询结果中可能含有 `NULL` 时两者语义不同：`IN` 可能返回 NULL 导致 WHERE 不成立查不到任何行（详见 [WHERE 子句](#WHERE-子句)小节），而 `EXISTS` 只看行是否存在，不受 NULL 干扰。因此表达是否存在语义的检查，优先使用 `EXISTS` 更稳妥。另外，`EXISTS` 还是短路判断，子查询一旦返回第一行就立即停止，不再继续扫描剩余行，当子查询是相关子查询时，`EXISTS` 通常效率比 `IN` 更高。

### 复合查询 Compound query
多个 `SELECT` 语句可以使用集合运算符 `UNION`、`UNION ALL`、`INTERSECT` 或 `EXCEPT` 连接在一起，构成一个**复合查询 compound query**`，如下表：  

| <font size=2>运算符</font> | <font size=2>说明</font> |
| :---- | :---- |
| <font size=2>`UNION`</font> | <font size=2>并集，去除重复行</font> |
| <font size=2>`UNION ALL`</font> | <font size=2>并集，保留重复行，无需去重故性能更好</font> |
| <font size=2>`INTERSECT`</font> | <font size=2>交集，只保留两边都出现的行，并去重</font> |
| <font size=2>`EXCEPT`</font> | <font size=2>差集，保留只在左边出现的行，并去重</font> |

``` SQL
-- 汇总正式员工与合同工的联系方式，可能出现同名的行会被去重
SELECT name, phone FROM employees
UNION
SELECT name, phone FROM contractors;

-- 在旧表中存在、但已被新表删除的用户（数据迁移核对）
SELECT email FROM old_users
EXCEPT
SELECT email FROM users;

-- 只返回一行，因为 NOCASE 下 'a' 和 'A' 相等
SELECT 'a' COLLATE NOCASE
UNION
SELECT 'A';
```

* 参与复合查询的每个 `SELECT` 必须返回相同的列数，结果集的列名取自最左边第一个 `SELECT`；  
* 为了判定复合 `SELECT` 运算符结果中的重复行，对应列的值按从左到右各 `SELECT` 所声明的排序规则 `COLLATE` 进行比较，即把左侧和右侧 `SELECT` 语句的对应列分别当作 `=` 运算符的左右操作数，同时 NULL 值被视为与其它 NULL 值相等；  
* `ORDER BY` 与 `LIMIT` 只能写在复合查询的最末尾，作用于整个结果集，排序键只能引用结果集的列（即第一个 `SELECT` 的列名或序号）。

### WITH 子句
**公共表表达式 common table expression (CTE)** 是通过在 `SELECT`、`INSERT`、`DELETE` 或 `UPDATE` 语句前置一个 `WITH` 子句定义的临时命名结果集，仅在所属的单个 SQL 语句执行期间有效，可理解为临时的视图 VIEW，并且单个 `WITH` 子句可以指定一个或多个 CTE。CTE 又分为**普通 ordinary** 与**递归 recursive** 两种，普通 CTE 有助于把子查询从主 SQL 语句中提取出来，使查询更易于理解，而递归 CTE 则提供了对树与图进行层次化或递归查询的能力，这是 SQL 语言本身原本不具备的。其基本语法形式如下：  

> CTE 和 WITH 子句从 SQLite 3.8.3（2014-02-03）起支持，其中 MATERIALIZED 和 NOT MATERIALIZED 提示自 SQLite 3.35.0（2021-03-12）起支持。

``` SQL
WITH [RECURSIVE] 
cte_name [(column_name, ...)] AS [MATERIALIZED | NOT MATERIALIZED] (SELECT ...)
[, cte_name [(column_name, ...)] AS [MATERIALIZED | NOT MATERIALIZED] (SELECT ...)] ...
```

`AS` 后可以添加**物化提示 materialization hint**（借鉴自 PostgreSQL 的非标准 SQL 语法），它只是对查询优化器如何实现 CTE 的一种提示。`MATERIALIZED` 会把 CTE 物化成内存或临时磁盘上的临时表再引用，因此会阻断扁平化（子查询与外层查询合并）与条件下推（外层查询的 WHERE 条件推进子查询）等优化，常用作一道优化屏障。`NOT MATERIALIZED` 则当作子查询代入每一处引用，但名字虽含 "NOT" 也并不禁止物化，如果查询优化器认为物化是最佳方案，它仍可自由地对这个子查询采用物化，其含义更接近为当作普通视图或子查询看待。如果两个提示都没有写，那么 SQLite 可以自由选择它认为最合适的实现策略，这也是官方推荐的做法。除非有充分的理由，否则不要给公共表表达式加上 `MATERIALIZED` 或 `NOT MATERIALIZED` 关键字。

#### Ordinary CTE
普通 CTE 用于简化查询逻辑，将复杂的子查询提取出来，使 SQL 语句更易读和维护。它的作用类似于一个临时的视图，但生命周期仅限于当前语句。另外，即使某个 `WITH` 子句带有 `RECURSIVE` 关键字，其中也可以包含普通 CTE，使用 `RECURSIVE` 并不会强制所有公共表表达式都变成递归的，它只是允许 CTE 引用自身。下面演示一个 `WITH` 定义多个 CTE，且后面的 CTE 可以引用前面的例子：  

``` SQL
WITH dept_headcount(dept_id, headcount) AS ( -- 第一步：算出每个部门的人数
    SELECT dept_id, count(*) FROM employees GROUP BY dept_id
),
avg_headcount(avg_count) AS ( -- 第二步：引用前面的 CTE，算出各部门人数的平均值
    SELECT avg(headcount) FROM dept_headcount
)
SELECT d.name, h.headcount -- 主查询中 dept_headcount 被再次使用
FROM dept_headcount AS h
JOIN departments AS d ON d.id = h.dept_id
WHERE h.headcount > (SELECT avg_count FROM avg_headcount) -- 用第二个 CTE 的结果做筛选
ORDER BY h.headcount DESC;
```

> 普通 CTE 与子查询在能力上等价，区别主要在于可读性：CTE 把中间结果提到语句开头统一命名，可被引用多次而只需书写一次。而层层嵌套的子查询则需自内向外阅读，当同一子查询反复出现或嵌套超过两层时，就应考虑改用 CTE。

#### Recursive CTE
递归 CTE 用于处理层级或图状结构的数据，例如组织架构、分类目录或依赖关系。`WITH` 后应写上 `RECURSIVE` 关键字来允许查询体引用自身，并且 CTE 体必须由两部分通过 `UNION` 或 `UNION ALL` 组合而成，即必须是复合查询，其中第一部分不引用自身提供递归的起点称为**初始查询 initial-select**，第二部分引用自身进行迭代称为**递归查询 recursive-select**。初始查询不可以包含 `ORDER BY`、`LIMIT` 或 `OFFSET`，而递归查询中可以包含 `ORDER BY`、`LIMIT` 或 `OFFSET`，但不能使用聚合函数与窗口函数。

计算递归 CTE 的基本逻辑顺序为：①先运行初始查询，并将结果写入临时队列，当队列非空时反复执行后面的 ② ~ ④；②从临时队列提取一行；③将提取出来的一行写入 CTE 表；④假装提取出来的一行是 CTE 表唯一的一行，并运行递归查询，并将所有结果写入临时队列。上述过程会被以下规则修改：  
* 如果连接初始查询与递归查询的运算符是 `UNION`，重复的行在加入队列之前就会被丢弃。如果运算符是 `UNION ALL`，生成的所有行都会被加入队列。在判断一行是否重复时，`NULL` 值彼此相等，而与任何其它值都不相等；  
* `LIMIT` 子句若存在，则决定步骤 ③ 中最多有多少行会被加入 CTE 表。一旦达到该上限，递归就会停止。上限为负数表示加入 CTE 表的行数不受限制；  
* `OFFSET` 子句若存在且其值为正数 N，则前 N 行不会被加入 CTE 表。这前 N 行仍然会被递归查询处理，只是不会被加入 CTE 表，同时这 N 行不计入 `LIMIT` 的数量；  
* `ORDER BY` 子句若存在，则决定步骤 ② 中从队列取出行的顺序。若没有 `ORDER BY` 子句，则取出行的顺序是未定义的。在当前实现中，省略 `ORDER BY` 子句时队列表现为 FIFO，但应用程序不应当依赖这一事实，因为它可能发生变化。  

下面给出 Recursive CTE 的三个示例，分别为①生成一个从 1 到 10 的数字序列；②树的深度优先搜索与广度优先搜索；③有向无环图 DAG 的查询：  


***①生成一个从 1 到 10 的数字序列***  

``` SQL
-- 生成 1 ~ 10 的整数序列
WITH RECURSIVE seq(n) AS (
    SELECT 1                  -- 初始查询：起点
    UNION ALL
    SELECT n + 1 FROM seq     -- 递归查询：引用自身，逐行递归
    WHERE n < 10              -- 终止条件，也可以改为 LIMIT 10
)
SELECT n FROM seq;
```

***②树的深度优先搜索与广度优先搜索***  
递归查询中的 `ORDER BY` 子句可以用来控制树的搜索方式是**深度优先 depth-first** 还是**广度优先 breadth-first**，下面以一个简单的小顶堆作为示例：  

``` SQL
CREATE TABLE heap (
    idx INTEGER PRIMARY KEY,
    val INTEGER
);

--             10
--          /      \
--        20        30
--       /  \      /  \
--     40    50  60    70
INSERT INTO heap(idx, val) VALUES
(1, 10), (2, 20), (3, 30), (4, 40), (5, 50), (6, 60), (7, 70);

-- 深度优先搜索 depth-first search
WITH RECURSIVE dfs(idx, val, depth) AS (
    SELECT idx, val, 0 FROM heap WHERE idx = 1 -- 根节点
    UNION ALL
    SELECT heap.idx, heap.val, dfs.depth + 1 FROM heap
    JOIN dfs ON heap.idx = dfs.idx * 2 OR heap.idx = dfs.idx * 2 + 1
    ORDER BY 3 DESC -- 等价于 ORDER BY dfs.depth + 1 DESC，DESC 优先处理 depth 更大的行
)
SELECT idx, val, depth FROM dfs; -- 输出顺序：10 - 20 - 40 - 50 - 30 - 60 - 70

-- 广度优先搜索 breadth-first search
WITH RECURSIVE bfs(idx, val, depth) AS (
    SELECT idx, val, 0 FROM heap WHERE idx = 1 -- 根节点
    UNION ALL
    SELECT heap.idx, heap.val, bfs.depth + 1 FROM heap
    JOIN bfs ON heap.idx = bfs.idx * 2 OR heap.idx = bfs.idx * 2 + 1
    ORDER BY 3 -- 等价于 ORDER BY dfs.depth + 1，默认 ASC 优先处理 depth 更小的行
)
SELECT idx, val, depth FROM bfs; -- 输出顺序：10 - 20 - 30 - 40 - 50 - 60 - 70
```

***③有向无环图 DAG***  

> 对于大型图而言，强烈建议使用[索引](#索引-Index)，有助于提升查询性能。

``` SQL
PRAGMA foreign_keys = ON;

CREATE TABLE node (
    id   INTEGER PRIMARY KEY,
    name TEXT NOT NULL UNIQUE
);

CREATE TABLE edge (
    xfrom INTEGER NOT NULL REFERENCES node(id),
    xto   INTEGER NOT NULL REFERENCES node(id),
    PRIMARY KEY (xfrom, xto)
);

-- 建议加索引
CREATE INDEX idx_edge_from ON edge(xfrom);
CREATE INDEX idx_edge_to   ON edge(xto);

INSERT INTO node(id, name) 
VALUES (1, 'A'), (2, 'B'), (3, 'C'), (4, 'D'), (5, 'E');

INSERT INTO edge(xfrom, xto) 
VALUES (1, 2), (1, 3), (2, 4), (3, 4), (4, 5);

WITH RECURSIVE paths(current_id, path, depth) AS (
    SELECT id, name, 0 FROM node WHERE name = 'A'
    UNION ALL
    SELECT edge.xto, paths.path || ' -> ' || node.name, paths.depth + 1 FROM paths
    JOIN edge ON edge.xfrom = paths.current_id
    JOIN node ON node.id = edge.xto
)
SELECT path, depth FROM paths
WHERE NOT EXISTS (SELECT 1 FROM edge WHERE edge.xfrom = paths.current_id);
```

输出：  

``` Console
path             | depth
-----------------+-------
A -> B -> D -> E | 3
A -> C -> D -> E | 3
```

另外，若图中存在环，递归可能永远停不下来，大致可以使用这些方法来解决：使用 `UNION`（而非 `UNION ALL`）自动去重，让每个节点只出现一次；或者记录已经走过的节点，并检查下一个节点是否已经在路径中；又或者限制递归深度，超过指定深度就停止。

## 插入 INSERT
**插入 INSERT** 语句用于向表中添加新行，基本语法形式如下：  

``` SQL
[WITH [RECURSIVE] common_table_expression]
INSERT [OR conflict_algorithm] 
INTO [schema_name.]table_name AS alias [(column_list [, ...])]
{ VALUES (expr [, ...]) [, (expr [, ...])] ...
| SELECT ... 
| DEFAULT VALUES }
[ON CONFLICT ... DO ...]
[RETURNING result_list];
```

* `INSERT` 语句有三种基本形式：`INSERT INTO table VALUES(...);` 用于创建一行或多行新数据，`INSERT INTO table SELECT ...;` 用于把查询结果整体插入，`INSERT INTO table DEFAULT VALUES;` 用于插入全部为默认值的一行；  
* 列清单 *column_list* 可以省略。如果指定了列清单，要新增数据的列数必须与列清单中的项数相同，未出现在列清单中的列取默认值（无默认值时为 `NULL`）。如果未指定列清单，则要新增数据的列数必须与表中的列数相同。实际开发中建议总是显式写出列清单；  
* 冲突算法 *conflict_algorithm* 在列约束的 [ON CONFLICT 子句](#ON-CONFLICT-子句)小节中提到过，包含 `ABORT`、`ROLLBACK`、`FAIL`、`IGNORE` 和 `REPLACE`，具体含义这里就不重复了。总之，`INSERT` 语句插入的值会按目标列的类型亲和性进行隐式转换，并受列上各类约束的检查，冲突时按约束定义的 `ON CONFLICT` 子句指定的算法处理，但若在 `INSERT` 语句指定了冲突算法，则会覆盖约束定义中指定的算法。另外，冲突算法中 `REPLACE` 比较特殊，允许直接将 `REPLACE` 作为 `INSERT OR REPLACE` 的别名；  
* 注意区分列约束的 `ON CONFLICT` 子句与 `INSERT` 语句当中的 `ON CONFLICT` 子句，后者又称为 **UPSERT 子句**，详见下一小节。`UPSERT` 子句只允许出现在 `VALUES` 形式或 `SELECT` 形式中，`DEFAULT VALUES` 形式不支持。若在 `SELECT` 形式中，为了避免解析歧义，`SELECT` 语句应始终包含 `WHERE` 子句，即使该子句只是 `"WHERE true"`，若没有 `WHERE` 子句时，解析器无法判断标记 `"ON"` 究竟是 `SELECT` 上 `JOIN` 的一部分，还是 `UPSERT` 子句的开头；  
* 向 `INTEGER PRIMARY KEY`（rowid 别名）列插入 `NULL` 时，SQLite 会自动分配 rowid 值，前面也提到过；  
* `RETURNING` 子句从 SQLite 3.35.0（2021-03-12）起支持，用于在插入后返回受影响行的列值或表达式。

``` SQL
INSERT INTO employees (name, dept_id) VALUES ('Alice', 1), ('Bob', 2); -- VALUES 形式：一次插入多行
INSERT INTO employees (name, dept_id)
SELECT name, dept_id FROM candidates WHERE interview_status = 'passed'; -- SELECT 形式：把面试合格的候选者插入员工表
INSERT INTO employees DEFAULT VALUES; -- DEFAULT VALUES 形式：整行均为默认值

INSERT OR IGNORE INTO employees (name, dept_id) VALUES ('Alice', 1); -- INSERT OR IGNORE：冲突时静默跳过
REPLACE INTO employees (name, dept_id) VALUES ('Alice', 2); -- REPLACE：冲突时先删除旧行再插入新行，等同于 INSERT OR REPLACE

INSERT INTO employees (name, dept_id) VALUES ('Bob', 2)
RETURNING name, dept_id, salary; -- RETURNING 返回插入行对应列名的值
INSERT INTO employees (name, dept_id) VALUES ('Bob', 2)
RETURNING *; -- -- RETURNING * 返回插入行所有列
```

### UPSERT 子句
正如上面所说，`UPSERT` 子句就是在 `INSERT` 之后追加的一个或多个 `ON CONFLICT` 子句。当 `INSERT` 会违反唯一性约束时，它会让该 `INSERT` 表现为一次 `UPDATE`（详见下面更新 [UPDATE 小节](#更新-UPDATE)），或者什么都不做，其语法形式有两种如下：  
``` SQL
INSERT INTO table_name (column_list) VALUES (value_list)
ON CONFLICT [(conflict_target)] DO UPDATE SET column = expr [, ...] [WHERE condition];

INSERT INTO table_name (column_list) VALUES (value_list)
ON CONFLICT [(conflict_target)] DO NOTHING;
```

> `UPSERT` 不是标准 SQL，SQLite 的 `UPSERT` 遵循 PostgreSQL 的设计并做了一定泛化，并从 SQLite 3.24.0（2018-06-04）加入，最初的实现严格遵循 PostgreSQL 语法，即只允许单个 ON CONFLICT 子句，并且要求 DO UPDATE 必须带冲突目标。在 SQLite 3.35.0 版本（2021-03-12）中，该语法得到了推广，允许使用多个 ON CONFLICT 子句，并允许 DO UPDATE 在不带冲突目标的情况下进行。

* `UPSERT` 只针对唯一性约束，即 `CREATE TABLE` 语句中显式的 `UNIQUE` 或 `PRIMARY KEY` 约束，或者一个唯一索引。对于失败的 `NOT NULL`、`CHECK` 或外键约束，以及通过触发器实现的约束，`UPSERT` 不会介入；  
* 冲突目标 *conflict_target* 用于指定带有唯一性约束的单个列或多个列。在 `INSERT` 语句中，冲突目标可以在最后一个 `ON CONFLICT` 子句上省略，但对其它所有 `ON CONFLICT` 子句都是必需的，因为最后一个子句主要起到兜底的作用；  
* 当一条 `INSERT` 语句中有多个 `ON CONFLICT` 子句时，它们按出现的先后顺序依次检查。当插入某一行触发唯一性约束冲突时，SQLite 会按顺序找到第一个匹配该冲突的子句并执行其动作。并且允许最后一个 `ON CONFLICT` 子句省略冲突目标，如果前面的子句都没有匹配到冲突，这个兜底子句就会在任何唯一性约束失败时触发；  
* 如果插入操作会导致冲突目标所指定的唯一性约束失败，那么该次插入会被省略，转而执行相应的 `DO NOTHING` 或 `DO UPDATE` 操作；  
* 在 `DO UPDATE` 中，列名默认引用的是表中原始未修改的值（即 INSERT 尝试之前的值），可以用 `excluded.列名` 引用本次尝试插入（但因冲突被排除）的值。末尾可选的 `WHERE` 用于为更新附加额外条件。  

``` SQL
INSERT INTO vocabulary(word) VALUES('jovial')
ON CONFLICT(word) DO UPDATE SET count = count + 1; -- 单词库中已存在单词，则记忆次数加一

INSERT INTO phonebook(name, phonenumber) VALUES('Alice', '704-555-1212')
ON CONFLICT(name) DO UPDATE SET phonenumber = excluded.phonenumber; -- 号码簿中已存在号码，则更新为新号码

INSERT INTO phonebook2(name, phonenumber, validDate)
VALUES('Alice', '704-555-1212', '2018-05-08')
ON CONFLICT(name) DO UPDATE SET
    phonenumber = excluded.phonenumber,
    validDate = excluded.validDate
WHERE excluded.validDate > phonebook2.validDate; -- 仅当新日期的 validDate 更大时才更新
```

> 注意区分三个易混淆的 `ON CONFLICT`：① `CREATE TABLE` 中列约束定义上的 `ON CONFLICT` 子句指定的是默认冲突算法；② `INSERT OR REPLACE` 中的 `OR REPLACE` 是语句级的冲突算法；③ UPSERT 的 `ON CONFLICT ... DO UPDATE/DO NOTHING` 是语句级的显式指令，优先级最高。

## 更新 UPDATE
**更新 UPDATE** 语句用于修改表中已有的行，基本语法形式如下：  

``` SQL
[WITH [RECURSIVE] common_table_expression]
UPDATE [OR conflict_algorithm] [schema_name.]table_name
SET column_1 = expr_1 [, column_2 = expr_2, ...]
[FROM table_or_subquery]
[WHERE search_condition]
[RETURNING result_list]
[ORDER BY ...] [LIMIT count [OFFSET skip]];
```

* 与 `INSERT` 一样，`UPDATE` 也支持 `RETURNING` 子句与 `OR` 约束冲突解决算法，这里就不再重复了；  
* `UPDATE` 语句对每一受影响行所做的修改，由 `SET` 关键字后的一系列赋值决定，每个赋值在等号左侧指定一个列名，右侧是一个标量表达式，对每一受影响的行，相应列被设置为对应标量表达式的求值结果，标量表达式可以引用被更新行的列，此时所有标量表达式都在进行任何赋值之前先行求值。若同一列名在赋值列表中多次出现，除最右侧的一次外其余均被忽略，未出现在赋值列表中的列保持不修改；  
* 从 SQLite 3.15.0（2016-10-14）开始，`SET` 子句中的赋值可以写成等号左侧为括号括起的列名列表、右侧为同等大小的**行值 row value** 的形式；  
* `UPDATE-FROM` 是遵循 PostgreSQL 的非 SQL 标准扩展，从 SQLite 3.33.0（2020-08-14）起支持，它允许 `UPDATE` 基于其它表或子查询的数据进行更新；  
* 若 SQLite 在编译时启用了宏 `SQLITE_ENABLE_UPDATE_DELETE_LIMIT`，则可以使用 `ORDER BY` 与 `LIMIT` 子句，用于把更新限制在前若干行；  

``` SQL
UPDATE employees SET salary = salary * 1.1 WHERE dept_id = 1; -- 按条件更新（省略 WHERE 则更新全部行）
UPDATE t SET a = b, b = a; -- 同时赋值，交换两列

UPDATE employees -- UPDATE-FROM 使用另一张表的数据
SET salary = d.base_salary
FROM (SELECT dept_id, base_salary FROM dept_standard) AS d
WHERE employees.dept_id = d.dept_id;

UPDATE jobs SET status = 'done' -- 每次只处理最早的两条任务
WHERE status = 'pending'
RETURNING id; -- 返回被更新的行
ORDER BY created_at
LIMIT 2
```

## 删除 DELETE
**删除 DELETE** 语句用于删除表中的行，基本语法形式如下：  

``` SQL
[WITH [RECURSIVE] common_table_expression]
DELETE FROM [schema_name.]table_name
[WHERE search_condition]
[RETURNING result_list];
[ORDER BY ...] [LIMIT count [OFFSET skip]]
```

* 如果 `WHERE` 子句不存在，则表中的全部行都将被删除。如果提供了 `WHERE` 子句，则仅删除 `WHERE` 子句布尔表达式为 TRUE 的行，表达式为 FALSE 或 NULL 的行将被保留。另外，省略 `WHERE` 时删除表的全部行时，表本身及其关联的索引、触发器仍然保留，这与 `DROP TABLE` 有本质不同；  
* `ORDER BY`、`LIMIT` 与 `RETURNING` 的用法与 `UPDATE` 相同；  
* 当 `DELETE` 语句同时省略 `WHERE` 子句和 `RETURNING` 子句，且被删除的表没有触发器时，SQLite 会采用截断优化，即跳过逐行访问直接丢弃整表数据以加快删除操作的速度。可以通过宏 `SQLITE_OMIT_TRUNCATE_OPTIMIZATION` 禁用所有查询的截断优化；  
* `DELETE` 删除数据后，空间通常只是在 SQLite 内部变成可重用空间，例如页内空闲空间或 freelist，数据库文件默认不会自动缩小。要回收磁盘空间，通常需要执行 `VACUUM`，详见进阶特性 [VACUUM 小节](#VACUUM)。   

``` SQL
DELETE FROM employees WHERE dept_id = 1; -- 按条件删除

DELETE FROM logs -- 删除限制数量的过期日志
WHERE created_at < datetime('now', '-30 days')
ORDER BY created_at
LIMIT 1000;
```

# 进阶特性
## 视图 VIEW
**视图 VIEW** 用于保存一条 `SELECT` 语句，并为其指定名称，它本身不存储任何数据，每次查询视图时都会实时执行其定义中的 `SELECT`。并且<u>视图通常会被当作子查询处理</u>，SQLite 为了优化会尽量把视图展平进外层查询。其创建 `CREATE VIEW` 和删除 `DROP VIEW` 的基本语法形式如下：  

``` SQL
CREATE [TEMP | TEMPORARY] VIEW [IF NOT EXISTS] [schema_name.]view_name [(column_name, ...)]
AS select_statement;

DROP VIEW [IF EXISTS] [schema_name.]view_name;
```

* 视图与表、索引、触发器一样属于模式对象，其定义会以创建语句的原文保存在内部系统表 `sqlite_schema`（旧名 `sqlite_master`）中，可以像查询普通表那样查看已建视图的定义，例如 `SELECT sql FROM sqlite_schema WHERE type = 'view';`；  
* 如果使用 `TEMP` 或 `TEMPORARY` 关键字，则创建的视图仅对当前数据库连接可见，并在数据库连接关闭时自动删除，并且创建的视图在临时数据库 `temp` 当中，此时不能显式指定 `temp` 以外的数据库模式名 *schema_name*；  
* 可以指定视图所属的数据库 *schema_name*，当未指定模式且不存在 `TEMP` 或 `TEMPORARY` 关键字时，默认在主数据库 `main` 中创建。另外，视图名 *view_name* 与表名在同一个 schema 内不能重名；  
* 视图是**只读**的，不能对视图执行 `DELETE`、`INSERT` 或 `UPDATE` 操作，除非创建 `INSTEAD OF` 触发器，详见[触发器](#触发器-Trigger)小节；  
* 若视图名后跟着列名清单，则该清单决定视图各列的名称。若省略列名清单，则视图中的列名由 `SELECT` 结果集的列名派生而来。推荐使用列名清单，或者在 `SELECT` 语句中指定结果列的别名，因为虽然 SQLite 允许创建依赖自动生成列名的视图，但生成列名所使用的规则在未来版本中可能发生变化；  
* 正如之前[修改表](#修改表-ALTER-TABLE)和[删除表](#删除表-DROP-TABLE)小节所说，表被 `DROP TABLE` 删除后视图仍然存在，但此后查询该视图会报错。而 `ALTER TABLE` 的重命名表/重命名列会自动改写视图定义中对它们的引用。  

``` SQL
CREATE VIEW v_emp_dept (id, name, dept_name, salary) AS -- 创建视图
SELECT e.id, e.name, d.name, e.salary
FROM employees AS e
LEFT JOIN departments AS d ON e.dept_id = d.id;

SELECT dept_name, COUNT(*) FROM v_emp_dept GROUP BY dept_name; -- 像查表一样查视图
PRAGMA table_info(v_emp_dept); -- 查看视图的列定义

DROP VIEW v_emp_dept; -- 删除视图
```

## 索引 INDEX
**索引 INDEX** 是依附于某张表，并由表中的一个或多个列（或表达式）派生出来的辅助数据结构，在 SQLite 中以 B-Tree 的形式单独存储，使 SQLite 无需**全表扫描 full table scan** 而通过**二分查找 binary search** 快速定位到符合查询条件的行。创建索引 `CREATE INDEX` 与删除索引 `DROP INDEX` 的基本语法形式如下：  

``` SQL
CREATE [UNIQUE] INDEX [IF NOT EXISTS] [schema_name.]index_name
ON table_name (column_name [COLLATE collation] [ASC | DESC], ...)
[WHERE expr];

DROP INDEX [IF EXISTS] [schema_name.]index_name;
```

* 索引同样是一个模式对象，其定义同样以创建语句的原文保存在 `sqlite_schema` 中，查询方式为：`SELECT name, sql FROM sqlite_schema WHERE type = 'index';`。可以用 `PRAGMA index_list(表名);` 查看表上有哪些索引，或 `PRAGMA index_info(索引名);` 返回构成索引键的列，又或 `PRAGMA index_xinfo(索引名);` 返回索引中所有列的信息。另外，SQLite 会为 `UNIQUE` 与非 INTEGER 的 `PRIMARY KEY` 约束自动创建索引（在 `sqlite_schema` 中命名为 `sqlite_autoindex_表名_N`），自动索引不能用 `DROP INDEX` 删除；  
* 索引必须依附于某张表，创建时用 `ON table_name` 指定，单个表可以附加的索引数量没有特别的限制。正如之前[修改表](#修改表-ALTER-TABLE)和[删除表](#删除表-DROP-TABLE)小节所说，重命名表或重命名列会自动改写索引定义中对它们的引用，删除表时其上的索引会被一并删除；  
* `UNIQUE` 关键字用于创建唯一索引，让索引同时承担唯一性约束的作用，效果与表定义中的 `UNIQUE` 约束基本一致，详见 [UNIQUE](#UNIQUE) 小节；  
* 索引键既可以是列名，也可以是表达式，后者称为**表达式索引 index on expression**，从 SQLite 3.9.0（2015-10-14）起支持。引用虚拟列（详见 [GENERATED ALWAYS AS 子句](#GENERATED-ALWAYS-AS-子句)）的索引也是表达式索引。表达式只能引用被索引表自身的列，不能包含子查询、对其它表的引用、非确定性函数，自定义的 SQL 函数默认被视为非确定性函数，除非在注册函数时使用了 `SQLITE_DETERMINISTIC` 标志。SQLite 查询规划器会在以下情况下考虑使用表达式索引：当索引的表达式出现在查询的 `WHERE` 子句或 `ORDER BY` 子句中，且与 `CREATE INDEX` 语句中的表达式完全一致时，仅允许存在一些细微的语法差异，例如空格的改变；  
* 每个索引键后可以跟 `ASC` 或 `DESC` 指定升序或降序（默认 `ASC`），从 SQLite 3.7.10（2012-01-16）起才会真正区分这两种排序方向。SQLite 不支持索引使用 `NULLS FIRST` 和 `NULLS LAST`，出于排序的目的，SQLite 将 NULL 值视为小于所有其他值，因此，NULL 值始终出现在 `ASC` 索引的开头和 `DESC` 索引的结尾。也可以跟 `COLLATE` 为文本键指定排序规则，不写时沿用列上指定的排序规则；  
* 带可选的 `WHERE` 子句的索引称为**部分索引 partial index**，SQLite 3.8.0（2013-08-26）起支持，它只为表中满足条件的部分行建立索引条目，比如用于省略被索引列为 NULL 的条目。如果使用得当，部分索引可以减小数据库文件的大小，并提高查询和写入性能。部分索引的 `WHERE` 子句可以包含运算符、字面量与被索引表自身的列，但不能包含子查询、对其它表的引用、非确定性函数和绑定参数。查询能否用上部分索引，取决于查询的 `WHERE` 条件是否逻辑蕴含索引的 `WHERE` 条件。

### 索引加速查询原理

> 关于查询计划与索引更多、更深入的原理，可参阅官方文档 [Query Planning](https://www.sqlite.org/queryplanner.html)。

在 `rowid` 或 `INTEGER PRIMARY KEY` 表中，行的数据按 rowid 递增的顺序逻辑存放在一棵 B-Tree 上。当我们查询非 rowid 列（也非主键或 UNIQUE 键）时，SQLite 只能读取表中的每一行进行查询，这就是**全表扫描 full table scan**，其时间复杂度为 O(N)。相对的，如果按 rowid 查询，因为数据本身就按 rowid 有序存放，SQLite 可以用**二分查找 binary search** 找到正确的行，把时间复杂度降到 O(log N)，但是 rowid 通常并不是我们关心的业务键。为了既能按业务键查询又能利用二分查找，可以建立业务键上的索引。索引本质上可以理解为另一张表，它把被索引的列放在 rowid 之前，并按被索引列的值排序存放。于是 SQLite 可以先在索引上二分查找，查询业务键对应的 rowid，再拿着 rowid 回到原表上二分查找，从而读取业务键对应的行，从而用两次 O(log N) 的二分查找替代一次 O(N) 的全表扫描，达到加速查询的目的。

索引同样可以用来加速排序，即 `ORDER BY` 子句。当没有合适的索引可用时，SQLite 需要先收集查询的所有输出，所有输出都会累积在临时存储空间中，然后再将这些输出通过排序器进行处理，时间复杂度为 O(N log N)。若 `ORDER BY` 列上有索引，只需沿着索引从头到尾（`DESC` 则从尾到头）顺序扫描，然后按 rowid 回到原表上二分查找取行即可，时间复杂度仍然是 O(N log N)，但可以因此节省临时存储空间。

如果索引中已经包含了查询所需的全部列，SQLite 就不必拿着 rowid 回到原表上二分查找了，直接在索引上就能得到结果，这称为**覆盖索引 covering index**。对排序而言，覆盖索引还能让 `ORDER BY` 变为纯粹的 O(N) 顺序扫描，且无需任何临时缓冲区。

最后，注意索引不是免费的，它会占用额外的磁盘空间，并且每次 `INSERT`、`UPDATE`、`DELETE` 都必须同步维护所有受影响的索引，索引越多，写入越慢，因此应当只为确实需要的查询建立索引。 

> 上述原理对普通 rowid 表和 `WITHOUT ROWID` 表同样成立，区别仅在于充当键的 `rowid` 被替换成了主键。

### INDEXED BY
`INDEXED BY` 子句是 SQLite 的扩展语法，用于<u>强制</u>查询规划器在 `SELECT`、`UPDATE`、`DELETE` 语句中使用指定的索引。注意，`INDEXED BY` 子句并非提示机制，而是<u>明确要求</u>优化器使用哪个索引。还有一个 `NOT INDEXED` 子句指定上述语句不得使用任何索引，包括由 `UNIQUE` 和 `PRIMARY KEY` 约束创建的隐式索引。`INDEXED BY` 子句和 `NOT INDEXED` 子句的示例如下，关键字要紧跟在表名或别名之后：  

``` SQL
SELECT name FROM employees INDEXED BY idx_emp_dept WHERE dept_id = 1; -- 强制使用指定索引
SELECT name FROM employees NOT INDEXED WHERE dept_id = 1; -- 强制不使用任何索引

SELECT e.name, d.name AS dept_name
FROM departments AS d
LEFT JOIN employees AS e INDEXED BY idx_emp_dept ON e.dept_id = d.id;
``` 

### REINDEX
`REINDEX` 命令用于删除并从头开始重新建立索引。当修改了自定义排序规则函数定义，或者表达式索引所依赖的函数定义发生了变化时非常有用。其基本语法形式如下：  

``` SQL
REINDEX;                           -- 重建所有已附加数据库中的全部索引
REINDEX collation_name;            -- 重建所有使用该排序规则的索引
REINDEX [schema_name.]table_name;  -- 重建指定表上的全部索引
REINDEX [schema_name.]index_name;  -- 重建指定索引
REINDEX EXPRESSIONS;               -- 重建所有表达式索引（SQLite 3.53.0，2026-04-09 起）
```

对于 `REINDEX name` 中的 name 可能同时是排序规则、表或索引的名称。此时，SQLite 会优先匹配排序规则名称，为避免歧义，建议显式指定 schema 名称。另外，如果 name 是 EXPRESSIONS，则除了重建所有表达式索引之外，该命令还会按照上述优先级规则重建使用名为 “expressions” 的排序规则的任何索引，或名为 “expressions” 的表的索引，或名为 “expressions” 的索引。

## 触发器 TRIGGER
**触发器 TRIGGER** 是绑定在表（或视图）上的，当指定的 `INSERT`、`UPDATE`、`DELETE` 事件发生时自动执行的数据库操作。其创建 `CREATE TRIGGER` 和 `DROP TRIGGER` 与删除的基本语法形式如下：  

``` SQL
CREATE [TEMP | TEMPORARY] TRIGGER [IF NOT EXISTS] [schema_name.]trigger_name
[BEFORE | AFTER | INSTEAD OF] {DELETE | INSERT | UPDATE [OF column_name, ...]}
ON table_name
[FOR EACH ROW]
[WHEN expr]
BEGIN
    [UPDATE | INSERT | DELETE | SELECT] statement; -- 注意这个分号
    ...
END;

DROP TRIGGER [IF EXISTS] [schema_name.]trigger_name;
```

* `BEFORE` 或 `AFTER` 关键字决定了触发操作的执行时机，如果两个关键字都未指定，则默认值为 `BEFORE`。`BEFORE` 和 `AFTER` 触发器仅适用于普通表，`INSTEAD OF` 触发器仅适用于视图。如果你在视图上创建了 `INSTEAD OF` 触发器，那么当你对视图执行这些操作时，SQLite 不会真的去修改视图，而是转去执行触发器里的 SQL，常用于把对视图的增删改替换为对视图绑定表的增删改；  
* 对于 `UPDATE OF column-name`，只有当指定的列名同时出现在 `UPDATE` 语句的 `SET` 子句当中时，触发器才会触发。但是由于历史遗留问题，`UPDATE OF` 指定的列名未出现在表中也不会报错，会被静默忽略；  
* SQLite 只支持行级 `FOR EACH ROW` 触发器，没有语句级 `FOR EACH STATEMENT` 触发器。`FOR EACH ROW` 可以省略，但语义上始终是对每一行触发一次；  
* 如果提供了 `WHEN` 子句，则仅当 `WHEN` 子句为真时才会执行指定的 SQL 语句；  
* `WHEN` 子句和 `BEGIN ... END` 内触发体均可使用 `NEW.列名` 和 `OLD.列名` 形式的引用访问正在插入、删除或更新的行的元素。但 `INSERT` 只能使用 `NEW.列名`，而 `DELETE` 只能使用 `OLD.列名`，而 `UPDATE` 两者即可使用；  
* 当与其关联的表被删除时，触发器也会自动删除。但是，如果触发器操作引用了其他表，则即使这些表被删除或修改，触发器也不会被删除或修改；  
* 触发器中触发体的 `UPDATE`、`DELETE` 和 `INSERT` 语句不支持完整的语法，存在一些限制：在上述语句中要修改的表名必须是不带限定符的表名；被修改或查询的表必须与触发器所附加的表或视图位于同一数据库中，但临时触发器不受此限制；不支持 `INSERT` 语句的 `DEFAULT VALUES` 形式；不支持 `INDEXED BY` 和 `NOT INDEXED` 子句；不支持 `UPDATE` 和 `DELETE` 语句的 `ORDER BY` 和 `LIMIT` 子句；触发器内部的语句不支持直接使用 CTE，但可以将 CTE 嵌入到触发器内部语句使用的子查询中；  
* 如果 `BEFORE UPDATE` 或 `BEFORE DELETE` 触发器修改或删除了原本应该更新或删除的行，则后续的更新或删除操作的结果未定义。此外，如果 `BEFORE` 触发器修改或删除了行，则原本应该对这些行运行的 `AFTER` 触发器是否实际运行也未定义。在 `BEFORE INSERT` 触发器中，如果 `rowid` 没有显式设置为整数，则 `NEW.rowid` 的值未定义。由于上述行为，建议程序员优先使用 `AFTER` 触发器而不是 `BEFORE` 触发器；  
* 触发体中可执行 `INSERT`、`UPDATE`、`DELETE`、`SELECT`，因此一个触发器可能引发另一个触发器，形成连锁甚至递归。递归触发器默认关闭，由 `PRAGMA recursive_triggers` 控制；  
* `RAISE(conflict_algorithm, 'msg')` 是在触发器内部的特殊函数，用来主动抛出错误，并执行指定的冲突解决算法和返回指定的错误消息。  

``` SQL
CREATE TRIGGER trg_users_email_lower            -- 插入后把邮箱统一转成小写
AFTER INSERT ON users
WHEN NEW.email <> lower(NEW.email)
BEGIN
    UPDATE users SET email = lower(NEW.email) WHERE id = NEW.id;
END;

CREATE TRIGGER trg_employees_check_salary       -- BEFORE 中校验待写入的值，并返回错误信息
BEFORE UPDATE OF salary ON employees
WHEN NEW.salary < 0
BEGIN
    SELECT RAISE(ABORT, 'salary must not be negative');
END;

CREATE TRIGGER trg_view_insert                  -- INSTEAD OF，让视图的更改写入原表
INSTEAD OF INSERT ON v_emp_dept
BEGIN
    INSERT INTO employees (id, name, dept_id) VALUES (NEW.id, NEW.name, NEW.dept_id);
END;
```

## 事务 TRANSACTION
**事务 TRANSACTION** 是一组数据库操作的工作单元，用于保证数据的完整性和可靠性。事务具有以下四个标准，称为 **ACID 标准**，即使事务因程序崩溃、操作系统转储或计算机断电而中断，SQLite 也能保证所有事务符合 ACID 标准：  
1. **原子性 atomicity**：事务中的所有操作要么全部成功，要么全部失败回滚，不存在部分完成的中间状态；  
2. **一致性 consistency**：事务的执行不会破坏数据库的完整性约束，数据在事务前后都处于合法状态。  
3. **隔离性 isolation**：并发执行的事务之间互不干扰。比如当一个会话启动事务并执行 `INSERT` 或 `UPDATE` 语句来更改数据时，这些更改仅对当前会话可见，其他会话不可见；  
4. **持久性 durability**：一旦事务提交，其修改就是永久性的，即使发生系统故障也不会丢失。

<u>默认情况下，SQLite 以自动提交模式运行事务，即对于任意语句或命令（少数 `PRAGMA` 语句除外）SQLite 都会自动启动、处理并提交事务</u>。但有两种方式手动启动事务的方式：一种是使用 `BEGIN` 和 `COMMIT` 语句，一种是使用 `SAVEPOINT` 语句，基本代码形式如下：   

``` SQL
BEGIN [DEFERRED | IMMEDIATE | EXCLUSIVE] [TRANSACTION]; -- 开启事务
ROLLBACK [TRANSACTION]; -- 回滚并取消事务
[COMMIT | END] [TRANSACTION]; -- 提交/结束事务

SAVEPOINT savepoint_name; -- 设置保存点
ROLLBACK [TRANSACTION] TO [SAVEPOINT] savepoint_name; -- 回滚到保存点，事务仍然继续
RELEASE [SAVEPOINT] savepoint_name; -- 释放保存点
```

### BEGIN / COMMIT
* 使用 `BEGIN` 启动事务，事务通常会持续到下一次 `COMMIT` 或 `ROLLBACK` 命令执行为止。如果数据库关闭，或者发生冲突且冲突解决算法指定为 `ROLLBACK`，事务也会回滚；  
* 使用 `BEGIN` 创建的事务不能嵌套，`SAVEPOINT` 支持嵌套与部分回滚，`ROLLBACK` 命令的 `TO SAVEPOINT name` 子句仅适用于 `SAVEPOINT` 事务。无论事务是由 `SAVEPOINT` 还是之前的 `BEGIN` 启动，尝试在事务中调用 `BEGIN` 命令都会失败并返回错误；  
* SQLite 支持来自不同数据库连接的多个并发读取事务（可能在不同的线程或进程中），但仅支持一个并发写入事务。**读事务 read transaction** 仅用于读取操作。**写事务 write transaction** 允许读取和写入操作，读事务由 `SELECT` 语句启动，写事务由 `CREATE`、`DELETE`、`DROP`、`INSERT` 或 `UPDATE` 等语句启动。如果在读事务处于活动状态时执行写语句，则读事务会尽可能升级为写事务。如果其他数据库连接已经修改了数据库或正在修改数据库，则无法升级为写事务，写语句将失败并返回 `SQLITE_BUSY` 错误。当读取事务处于活动状态时，其他数据库连接对数据库所做的任何更改，发起读取事务的数据库连接都无法看到；  
* 事务可以是 `DEFERRED`、`IMMEDIATE` 或 `EXCLUSIVE`，默认行为是 `DEFERRED`。`DEFERRED` 即事务不立即开始直到第一次读写操作才自动启动，并且它本质上只是设置一个标签，让自动开始的事务持续执行直到显式执行 `COMMIT` 或 `ROLLBACK`，亦或是发生错误等。如果 `BEGIN DEFERRED` 之后的第一个语句是 `SELECT`，则会启动一个读事务，后续的写语句会尽可能将事务升级为写事务。如果 `BEGIN DEFERRED` 之后的第一个语句是写语句，则会启动一个写事务。`IMMEDIATE` 立即执行写事务，并且如果另一个数据库连接上已有正在执行的写入事务，会直接 `SQLITE_BUSY` 报错。`EXCLUSIVE` 与 `IMMEDIATE` 类似，都会立即启动写事务，在 `WAL` 模式下，`EXCLUSIVE` 和 `IMMEDIATE` 的效果相同，但在其他日志模式下，`EXCLUSIVE` 会阻止其他数据库连接在事务进行期间读取数据库；  
* `PRAGMA foreign_keys` 属于不能在事务中修改的操作，在事务中为空操作，因此使用时需要注意顺序，需与 `重建表` 小节的顺序一致。  

``` SQL
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
```

### SAVEPOINT
* `SAVEPOINT` 会启动一个带名称的事务，`SAVEPOINT` 可以在 `BEGIN ... COMMIT` 内部或外部启动，在外部时，行为与 `DEFERRED` 事务一致；  
* `ROLLBACK TO` 会将数据库状态恢复到相应 `SAVEPOINT` 之后的状态。注意，与不带 `TO` 的 `ROLLBACK` 不同，ROLLBACK TO 命令不会取消事务，而是从头开始重新启动事务，但所有中间的 SAVEPOINT 操作都会被取消；  
* `RELEASE` 类似于 `SAVEPOINT` 的 `COMMIT` 命令。`RELEASE` 会将所有与其名称匹配的保存点从事务栈中移除。对于内部事务的 `RELEASE` 不会将任何更改写入数据库文件，它只是从事务栈中移除保存点，使得无法再回滚到这些保存点。但如果 `RELEASE` 释放了最外层的保存点，导致事务栈为空，则 `RELEASE` 与 `COMMIT` 的效果相同。即使事务最初是由 `SAVEPOINT` 而不是 `BEGIN` 启动的，也可以使用 `COMMIT` 释放所有保存点并提交事务。需要注意的是：内部事务可能已提交（使用 `RELEASE` 命令），但随后其工作可能被外部事务中的 `ROLLBACK` 撤销，断电、程序崩溃或操作系统崩溃都会导致最外层事务回滚，撤销该外部事务中发生的所有更改，即使是那些看似已被 `RELEASE` 提交的更改，只有最外层事务提交后，内容才会真正写入磁盘；  
* 上述事务嵌套现象的规则本质有：①`BEGIN` 仅在事务栈为空时有效，如果在调用时事务栈不为空，则失败；②`COMMIT` 命令会提交所有未完成的事务，并将事务栈清空；③`RELEASE` 从事务栈中最新添加的事务开始，按时间顺序释放之前的保存点，直到释放到名称匹配的保存点为止。如果 `RELEASE` 导致事务栈为空，则该事务提交；④不带 `TO` 子句的 `ROLLBACK` 会回滚所有事务，并将事务堆栈清空；⑤带有 `TO` 子句的 `ROLLBACK` 命令会将事务回滚到时间上最近的、名称匹配的 `SAVEPOINT`，但该 `SAVEPOINT` 仍然保留在事务栈中。

``` SQL
BEGIN; -- 外层事务开始

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
SAVEPOINT sp; -- 内层保存点，内层事务开始

UPDATE accounts SET balance = balance + 100 WHERE id = 2;
ROLLBACK TO sp; -- 回滚内层

UPDATE accounts SET balance = balance + 50 WHERE id = 2;
RELEASE sp; -- 释放保存点（可选）

...

COMMIT; -- 提交整个外层事务
```

### 日志模式
SQLite 的日志模式 `journal_mode` 是其实现 ACID 事务的核心机制，SQLite 一共支持 6 种日志模式：`DELETE`、`TRUNCATE`、`PERSIST`、`MEMORY`、`OFF` 和 `WAL`。其中前 5 种属于**回滚日志 rollback journal** 体系，而 WAL 是**预写日志 write-ahead log** 模式，可以通过 `PRAGMA schema.journal_mode` 选择，如下表：  

| <font size=2>日志模式</font> | <font size=2>说明</font> |
| :---- | :---- |
| <font size=2>DELETE（默认）</font> | <font size=2>回滚日志在事务提交后被删除</font> |
| <font size=2>TRUNCATE</font> | <font size=2>将回滚日志截断为零长度来提交事务，而不是删除它。在许多系统中，截断文件比删除文件快得多，因为无需更改包含该文件的目录</font> |
| <font size=2>PERSIST</font> | <font size=2>防止回滚日志在每次事务结束后被删除，只把头标记为无效防止其他数据库连接回滚日志，在某些平台上是一种有效的优化手段</font> |
| <font size=2>MEMORY</font> | <font size=2>回滚日志放在内存中，速度快，但数据库文件可能会损坏</font> |
| <font size=2>WAL</font> | <font size=2>预写日志，修改先追加到 `-wal` 文件</font> |
| <font size=2>OFF</font> | <font size=2>关闭日志，不创建回滚日志，禁用 SQLite 的原子提交和回滚功能，同时 `ROLLBACK` 命令不再有效</font> |

* **回滚日志 rollback journal** 是一个普通的磁盘文件，始终位于与数据库文件相同的目录或文件夹中，并且文件名与数据库文件相同，只是在文件名后添加了 `-journal` 后缀。当进程想要修改数据库文件（且未处于 `WAL` 模式）时，它首先会将原始的、未更改的数据库内容记录到回滚日志中，然后将更改直接写入数据库文件，ROLLBACK 时，回滚日志中包含的原始内容会被还原到数据库文件中；  
* **预写日志 write-ahead log** 从 SQLite 3.7.0（2010-07-21）起支持。`WAL` 模式在大多数场景下效率都优于传统的回滚日志模式，特别是在并发场景下，并且还支持读写操作并发（其它模式读写互斥，只支持读读并发）。它的工作原理是，将原始内容保留在数据库文件中，而更改则追加到 WAL 文件中，一次事务提交可以在不写入原始数据库的情况下，允许其它读取在原始数据库上操作，并同时提交修改。多个事务可以追加到单个 WAL 文件的末尾。最终 WAL 文件中附加的所有事务都需要传输回原始数据库，这个过程称为检查点 checkpoint，默认情况下当 WAL 文件达到 1000 页的阈值时，SQLite 会自动执行检查点操作。这个阈值或者自动检查点间隔可以通过 `PRAGMA wal_autocheckpoint = N;` 设置，另外还可以通过 `PRAGMA schema.wal_checkpoint[(PASSIVE | FULL | RESTART | TRUNCATE | NOOP)];` 手动触发检查点操作，这里篇幅有限，相关内容详见官方文档 [Write-Ahead Logging](https://www.sqlite.org/wal.html)；  
* 补充一个跟 WAL 相关的设置，即同步模式 `PRAGMA schema.synchronous = 0 | OFF | 1 | NORMAL | 2 | FULL | 3 | EXTRA;`，用于控制数据库文件在磁盘上的写入同步级别，在断电或系统崩溃时的安全性与写入性能之间进行取舍，四种取值的含义如下：①`OFF | 0` 几乎不做同步，把数据交给操作系统后立即继续，应用自身崩溃时数据是安全的，但操作系统崩溃或断电可能导致数据库损坏，提交速度最快；②`NORMAL | 1` 在最关键的时机同步，但没有 `FULL` 那么频繁，在 WAL 模式下不会损坏数据库，是官方推荐的性能与安全平衡点，代价是断电后可能丢失最近若干已提交的事务（即牺牲持久性 durability）；③`FULL | 2` 为默认值，使用 xSync 确保所有内容在继续之前都安全写入磁盘，操作系统崩溃或断电不会损坏数据库，在 WAL 模式下完全满足 ACID，但在回滚日志模式下是否具有持久性仍取决于底层文件系统；④`EXTRA | 3` 在 `FULL` 的基础上，额外在 DELETE 模式下删除回滚日志后再同步一次该日志所在的目录，以提供更强的持久性，在 WAL 模式下与 `FULL` 没有区别。 

``` SQL
PRAGMA journal_mode = WAL;          -- 切换日志模式
PRAGMA synchronous  = NORMAL;       -- WAL 下建议使用 NORMAL
...
PRAGMA wal_checkpoint(TRUNCATE);    -- 手动把 WAL 内容合并回主库并截断 wal 文件
```

## 窗口函数 window function
**窗口函数 window function** 是在 `SELECT` 结果集的若干行上求值的函数，这些行构成一个**窗口 window**。窗口函数和其它函数（标量函数或聚合函数）的区别在于 `OVER` 子句，一个函数只要带有 `OVER` 子句就是窗口函数，在函数和 `OVER` 子句之间还可能有个 `FILTER` 子句。窗口函数从 SQLite 3.25.0（2018-09-15）起支持，其行为以 PostgreSQL 为参照设计，基本语法形式如下：  

> 窗口函数的作用很像 `GROUP BY` 子句，但它们的核心区别在于，聚合查询会把同一组的多行合并为一行，改变结果集，而在窗口函数计算阶段，它所作用的结果集行数保持不变，每一行都会得到一个计算结果。它们的作用可以简单理解为，`GROUP BY` 负责压缩汇总，窗口函数负责明细分析。

``` SQL
function_name(expr [, ...]) [FILTER (WHERE expr)] OVER (
    [base_window_name]
    [PARTITION BY expr, ...]
    [ORDER BY ordering_term, ...]
    [frame_spec]
)
```

* `OVER` 后括号中的内容称为**窗口定义 window definition** (*window-defn*)，由可选的基础窗口名  *base_window_name*、`PARTITION BY` 子句、`ORDER BY` 子句与**窗口帧规范 frame specification** (*frame-spec*) 四部分组成。其中 `PARTITION BY` 子句将结果集划分为多个分区，窗口函数会在每个分区内独立计算。`ORDER BY` 子句决定分区内行的处理顺序。帧规范定义窗口函数可以读取分区内哪些行；  
* 与普通函数不同，窗口函数不能使用 `DISTINCT` 关键字。并且，窗口函数只能出现在 `SELECT` 的结果列与 `SELECT` 的 `ORDER BY` 子句中，不能出现在其它子句当中，诸如 `WHERE`、`GROUP BY`、`HAVING` 等子句，因为窗口函数是基于结果集求值的，其它子句的逻辑求值时机早于窗口函数的计算；  
* 窗口函数可以分为**聚合窗口函数 aggregate window function** 和**内置窗口函数 built-in window function**（详见后面小节）。SQLite 的所有内置聚合函数都可以通过添加适当的 `OVER` 子句用作聚合窗口函数，常用内置聚合函数详见 [GROUP BY](#GROUP-BY-子句) 子句小节。另外还可以使用 `sqlite3_create_window_function ()` API 创建自定义聚合窗口函数。  

### PARTITION BY 子句
`PARTITION BY` 子句用于把结果集划分为一个或多个**分区 partition**，而窗口函数会针对每个分区独立计算。如果不写 `PARTITION BY`，那么整个结果集就是一个分区。下面以示例解释该子句的作用，更直观一点，先使用聚合窗口函数，如下：  

``` SQL
-- 假设 employees 表的数据为（name | dept_id | salary）：
-- Alice|1|12000、Bob|1|9000、Carol|2|15000、Dave|2|12000、Eve|NULL|11000

SELECT name, dept_id, salary,
    avg(salary) OVER (PARTITION BY dept_id) AS 部门平均,
    avg(salary) OVER ()                     AS 全表平均,
    sum(salary) OVER (PARTITION BY dept_id) AS 部门总计,
    sum(salary) OVER ()                     AS 全表总计,
    round(100.0 * salary / sum(salary) OVER (PARTITION BY dept_id), 1) AS 员工工资占部门比
FROM employees
ORDER BY dept_id;
```

输出：  

``` Console
name  | dept_id | salary | 部门平均 | 全表平均 | 部门总计 | 全表总计 | 员工工资占部门比
------+---------+--------+---------+---------+----------+---------+-----------------
Eve   | NULL    | 11000  | 11000   | 11800   | 11000    | 59000   | 100.0
Alice | 1       | 12000  | 10500   | 11800   | 21000    | 59000   | 57.1
Bob   | 1       | 9000   | 10500   | 11800   | 21000    | 59000   | 42.9
Carol | 2       | 15000  | 13500   | 11800   | 27000    | 59000   | 55.6
Dave  | 2       | 12000  | 13500   | 11800   | 27000    | 59000   | 44.4
```

### 窗口帧与 ORDER BY 子句
**窗口帧规范 frame specifications** 用于精确指定窗口函数要读取分区内的哪些行，它的语法由**帧类型 frame type**、**起始帧边界 starting frame boundary**、**结束帧边界 ending frame boundary** 和可选的 `EXCLUDE` 子句四部分组成，其语法形式如下：  

``` SQL
{ ROWS | RANGE | GROUPS } BETWEEN frame_start AND frame_end [EXCLUDE ...]
{ ROWS | RANGE | GROUPS } frame_start [EXCLUDE ...] -- 省略结束帧边界时，结束帧边界默认为 CURRENT ROW
```

***①帧类型 frame type***  
共有三个帧类型，决定了起始帧边界和结束帧边界的判定方式，如下表：  

| <font size=2>帧类型</font> | <font size=2>说明</font> |
| :---- | :---- |
| <font size=2>`ROWS`</font> | <font size=2>起始帧边界和结束帧边界按行数相对当前行计数</font> |
| <font size=2>`GROUPS`</font> | <font size=2>起始帧边界和结束帧边界按相对于当前组的“组”数量来确定的，组是 `ORDER BY` 取值相同的行集合，称为**对等行 peer**</font> |
| <font size=2>`RANGE`</font> | <font size=2>以 `ORDER BY` 指定列的值为基准，取与当前行的取值相差一定范围内的行，这要求 `ORDER BY` 只有一个排序项</font> |

***②起始帧边界 starting frame boundary 与结束帧边界 ending frame boundary***  
起始与结束边界共有五种写法，如下表：  

| <font size=2>边界</font> | <font size=2>说明</font> |
| :---- | :---- |
| <font size=2>`UNBOUNDED PRECEDING`</font> | <font size=2>分区的第一行</font> |
| <font size=2>`expr PRECEDING`</font> | <font size=2>当前行之前 expr 个单位处，`0 PRECEDING` 与 `CURRENT ROW` 等价</font> |
| <font size=2>`CURRENT ROW`</font> | <font size=2>当前行，<u>对于 `RANGE` 与 `GROUPS`，当前行的 peer 也包含在内</u></font> |
| <font size=2>`expr FOLLOWING`</font> | <font size=2>当前行之后 expr 个单位处</font> |
| <font size=2>`UNBOUNDED FOLLOWING`</font> | <font size=2>分区的最后一行</font> |

***③`EXCLUDE` 子句***  
可选的 `EXCLUDE` 子句用于在帧边界已确定的范围上再排除若干行，如下表：  

| <font size=2>写法</font> | <font size=2>说明</font> |
| :---- | :---- |
| <font size=2>`EXCLUDE NO OTHERS`</font> | <font size=2>默认值，不排除任何行</font> |
| <font size=2>`EXCLUDE CURRENT ROW`</font> | <font size=2>把当前行排除（`RANGE`、`GROUPS` 下当前行的其它 peer 仍保留）</font> |
| <font size=2>`EXCLUDE GROUP`</font> | <font size=2>把当前行及其全部 peer 一并排除，<u>注意若没有 `ORDER BY` 所有分区的行视为 peer</u></font> |
| <font size=2>`EXCLUDE TIES`</font> | <font size=2>保留当前行，但排除它的 peer</font> |

* 若帧类型为 `RANGE` 或 `GROUPS`，则所有 `ORDER BY` 取值都相同的行被视为**对等行 peer**。或者，若没有 `ORDER BY` 项，则所有行都互为 peer，且 peer 始终位于同一帧内；  
* `expr PRECEDING` / `expr FOLLOWING` 中的 *expr* 必须是非负常量数值表达式。在 `ROWS` 下它必须是整数（按行计数），在 `GROUPS` 下也必须是整数（按 peer 组计数），而在 `RANGE` 下可以是任意非负实数，因为 `RANGE` 是按值范围确定边界的，同时若值是非整数，比如 NULL，那么此时帧范围就是当前 peer。`GROUPS`、`EXCLUDE` 以及 `RANGE` 下的 `<expr> PRECEDING / FOLLOWING` 均自 SQLite 3.28.0（2019-04-16）起支持；  
* 若省略整个帧规范，默认规范为 `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW EXCLUDE NO OTHERS`，即从分区首行到当前行（含当前行及其 peer）。<u>需要注意的是，若没有 `ORDER BY` 子句，则所有行互为 peer，又因为 `RANGE` 帧类型下 `CURRENT ROW` 包括所有 peer，所以此时默认帧边界为整个分区</u>；  
* 结束帧边界不能使用排在起始帧边界之前的写法，例如 `BETWEEN CURRENT ROW AND 1 PRECEDING` 是非法的；  
* 帧类型只对聚合窗口函数以及 `first_value()`、`last_value()`、`nth_value()` 内置窗口函数有意义，大多数内置窗口函数会忽略帧规范；  
* `ORDER BY` 子句决定分区内各行的处理顺序，它会直接影响 `row_number()`、`rank()` 等排名类内置窗口函数的结果，也会影响窗口帧的默认范围。需要特别注意，这个 `ORDER BY` 只作用于窗口内部，并不会改变结果集的最终输出顺序，最终顺序仍然由 `SELECT` 语句末尾的 `ORDER BY` 决定，不要混淆了；  
* `FILTER` 子句只适用于聚合窗口函数，若提供了 `FILTER` 子句，则只有那些使 `WHERE expr` 为真的行才会被纳入窗口帧。

为更好展示窗口帧与 ORDER BY 子句的效果，下面会有多个示例，都沿用之前的数据：  

***示例①：行号与累计求和***  

``` SQL
SELECT name, salary,
       row_number() OVER (ORDER BY salary) AS 行号, -- 仅由 ORDER BY 决定编号顺序
       sum(salary)  OVER (ORDER BY salary) AS 累计  -- 默认帧规范：从分区首行到当前行
FROM employees
ORDER BY salary; -- 决定结果集的最终输出顺序
```

输出：  

``` Console
name  | salary | 行号 | 累计
------+--------+------+-------
Bob   | 9000   | 1    | 9000
Eve   | 11000  | 2    | 20000
Alice | 12000  | 3    | 44000
Dave  | 12000  | 4    | 44000
Carol | 15000  | 5    | 59000
``` 

***示例②：ROWS、GROUPS、RANGE 三种帧类型的对比***  

``` SQL
SELECT name, dept_id, salary,
       sum(salary) OVER (ORDER BY salary
                         ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING) AS 前后一行之和, -- 按行计数
       sum(salary) OVER (ORDER BY salary
                         GROUPS BETWEEN 1 PRECEDING AND 1 FOLLOWING) AS 前后一组之和, -- 按 peer 组计数
       sum(salary) OVER (ORDER BY salary
                         RANGE BETWEEN 2000 PRECEDING AND 2000 FOLLOWING) AS 前后范围之和 -- 按 ORDER BY 的取值计数
FROM employees
ORDER BY salary; -- salary 有重复值，会产生 peer 行
```

输出：  

``` Console
name  | dept_id | salary | 前后一行之和 | 前后一组之和 | 前后范围之和
------+---------+--------+-------------+-------------+-------------
Bob   | 1       | 9000   | 20000       | 20000       | 20000
Eve   | NULL    | 11000  | 32000       | 44000       | 44000
Alice | 1       | 12000  | 35000       | 50000       | 35000
Dave  | 2       | 12000  | 39000       | 50000       | 35000
Carol | 2       | 15000  | 27000       | 39000       | 15000
```

***示例③：FILTER 子句***  

``` SQL
SELECT name, dept_id, salary,
       count(*) OVER (PARTITION BY dept_id)                                AS 部门人数,
       count(*) FILTER (WHERE salary >= 12000) OVER (PARTITION BY dept_id) AS 部门高薪人数
FROM employees
ORDER BY dept_id;
```

输出：  

``` Console
name  | dept_id | salary | 部门人数 | 部门高薪人数
------+---------+--------+----------+--------------
Eve   | NULL    | 11000  | 1        | 0
Alice | 1       | 12000  | 2        | 1
Bob   | 1       | 9000   | 2        | 1
Carol | 2       | 15000  | 2        | 2
Dave  | 2       | 12000  | 2        | 2
```

***示例④：EXCLUDE 子句***  

``` SQL
SELECT name, salary,
       group_concat(name, '/') OVER (ORDER BY salary
                                     ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING) AS 默认,
       group_concat(name, '/') OVER (ORDER BY salary
                                     ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING
                                     EXCLUDE CURRENT ROW) AS 排除当前行,
       group_concat(name, '/') OVER (ORDER BY salary
                                     ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING
                                     EXCLUDE GROUP) AS 排除当前组,
       group_concat(name, '/') OVER (ORDER BY salary
                                     ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING
                                     EXCLUDE TIES) AS 排除并列
FROM employees
ORDER BY salary;
```

输出：  

``` Console
name  | salary | 默认             | 排除当前行      | 排除当前组      | 排除并列
------+--------+------------------+----------------+----------------+---------------
Bob   | 9000   | Bob/Eve          | Eve            | Eve            | Bob/Eve
Eve   | 11000  | Bob/Eve/Alice    | Bob/Alice      | Bob/Alice      | Bob/Eve/Alice
Alice | 12000  | Eve/Alice/Dave   | Eve/Dave       | Eve            | Eve/Alice
Dave  | 12000  | Alice/Dave/Carol | Alice/Carol    | Carol          | Dave/Carol
Carol | 15000  | Dave/Carol       | Dave           | Dave           | Dave/Carol
```

### 内置窗口函数
除聚合窗口函数外，SQLite 还参照 PostgreSQL 提供了 11 个内置窗口函数。其中一些窗口函数（`rank()`、`dense_rank()`、`percent_rank()` 与 `ntile()`）使用了 peer 的概念，在这些情况下，帧规范写 `ROWS`、`GROUPS` 还是 `RANGE` 都无所谓。并且，大多数内置窗口函数会忽略帧规范，例外是 `first_value()`、`last_value()` 与 `nth_value()`。

| <font size=2>函数</font> | <font size=2>说明</font> |
| :---- | :---- |
| <font size=2>`row_number()`</font> | <font size=2>当前行在其分区内的行号，按窗口 `ORDER BY` 从 1 开始连续编号</font> |
| <font size=2>`rank()`</font> | <font size=2>当前行所属 peer 组中第一行的行号，是有间隔的排名，并列后会跳号（如 1、1、3）</font> |
| <font size=2>`dense_rank()`</font> | <font size=2>当前行所属 peer 组的序号，是无间隔的排名，并列后不跳号（如 1、1、2）</font> |
| <font size=2>`percent_rank()`</font> | <font size=2>等于 (rank - 1) / (分区行数 - 1)，取值恒在 0.0 到 1.0 之间，分区只有一行时返回 0.0</font> |
| <font size=2>`cume_dist()`</font> | <font size=2>累计分布，等于当前行所在 peer 组最后一行的 row_number 除以分区行数</font> |
| <font size=2>`ntile(N)`</font> | <font size=2>把分区尽可能均匀地分成 N 组并返回当前行所属组号（1 到 N）</font> |
| <font size=2>`lag(expr[, offset[, default]])`</font> | <font size=2>返回分区内当前行之前第 offset 行（默认 1）的 expr 值，不存在时返回 default（默认 NULL）</font> |
| <font size=2>`lead(expr[, offset[, default]])`</font> | <font size=2>返回分区内当前行之后第 offset 行（默认 1）的 expr 值，不存在时返回 default（默认 NULL）</font> |
| <font size=2>`first_value(expr)`</font> | <font size=2>窗口帧中第一行的 expr 值</font> |
| <font size=2>`last_value(expr)`</font> | <font size=2>窗口帧中最后一行的 expr 值</font> |
| <font size=2>`nth_value(expr, N)`</font> | <font size=2>窗口帧中第 N 行（从 1 开始计数）的 expr 值，不存在时返回 NULL</font> |

下面用一个示例演示这些内置窗口函数（沿用上面的 employees 表，按 salary 从高到低排序）：  

``` SQL
SELECT name, salary,
       row_number()        OVER (ORDER BY salary DESC) AS 行号, -- 1、2、3、4、5
       rank()              OVER (ORDER BY salary DESC) AS 排名, -- 并列后跳号：1、2、2、4、5
       dense_rank()        OVER (ORDER BY salary DESC) AS 密集排名, -- 并列后不跳号：1、2、2、3、4
       percent_rank()      OVER (ORDER BY salary DESC) AS 百分位, -- (rank - 1) / (5 - 1)
       cume_dist()         OVER (ORDER BY salary DESC) AS 累计分布, -- 到最后一名时恒为 1.0
       ntile(2)            OVER (ORDER BY salary DESC) AS 分组, -- 尽量平均分成 2 组
       lag(salary, 1, -1)  OVER (ORDER BY salary DESC) AS 上一名, -- 没有上一名时用 -1 兜底
       lead(salary, 1, -1) OVER (ORDER BY salary DESC) AS 下一名, -- 没有下一名时用 -1 兜底
       first_value(salary) OVER (ORDER BY salary DESC
                                 GROUPS BETWEEN 1 PRECEDING AND 1 FOLLOWING) AS 帧内最高, -- 帧内第一行，降序下即帧内最高工资
       last_value(salary)  OVER (ORDER BY salary DESC
                                 GROUPS BETWEEN 1 PRECEDING AND 1 FOLLOWING) AS 帧内最低 -- 帧内最后一行，降序下即帧内最低工资
FROM employees
ORDER BY salary DESC;
```

输出：  

``` Console
name  | salary | 行号 | 排名 | 密集排名 | 百分位 | 累计分布 | 分组 | 上一名 | 下一名 | 帧内最高 | 帧内最低
------+--------+------+-----+---------+--------+---------+------+--------+-------+---------+----------
Carol | 15000  | 1    | 1   | 1       | 0.0    | 0.2     | 1    | -1     | 12000 | 15000   | 12000
Alice | 12000  | 2    | 2   | 2       | 0.25   | 0.6     | 1    | 15000  | 12000 | 15000   | 11000
Dave  | 12000  | 3    | 2   | 2       | 0.25   | 0.6     | 1    | 12000  | 11000 | 15000   | 11000
Eve   | 11000  | 4    | 4   | 3       | 0.75   | 0.8     | 2    | 12000  | 9000  | 12000   | 9000
Bob   | 9000   | 5    | 5   | 4       | 1.0    | 1.0     | 2    | 11000  | -1    | 11000   | 9000
```

### 具名 WINDOW 子句
可以将 `OVER` 后的窗口定义，单独拎出来写到 `SELECT` 语句中的 `WINDOW` 子句里命名，然后再在窗口函数中使用。当一个查询中有多个窗口函数，且它们的窗口定义相同时，可以避免重复书写。<u>`WINDOW` 子句位于`SELECT` 语句的 `HAVING` 子句之后、`ORDER BY` 子句之前</u>，可以定义多个具名窗口并用逗号分隔：  

``` SQL
SELECT name, dept_id, salary,
       row_number() OVER w_dept AS 部门行号, -- 引用第一个具名窗口 w_dept
       rank()       OVER w_dept AS 部门排名, 
       sum(salary)  OVER w_dept AS 部门累计,
       sum(salary)  OVER w_all  AS 全表累计  -- 换用第二个具名窗口
FROM employees
WINDOW w_dept AS (PARTITION BY dept_id ORDER BY salary DESC), -- 第一个具名窗口：按部门分区
       w_all  AS (ORDER BY salary DESC)                       -- 第二个具名窗口：不分区，整个结果集是一个分区
ORDER BY salary DESC;
```

还允许用具名窗口定义另一个窗口，称为**窗口链 window chaining**。此时新的窗口定义中不能出现 `PARTITION BY`，只能写在基础窗口里，也不能在基础窗口已有 `ORDER BY` 时再写 `ORDER BY`，并且帧规范只能写在新的窗口定义中，例如：  

``` SQL
SELECT name, salary,
       sum(salary) OVER (w ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS 部门累计
FROM employees
WINDOW w AS (PARTITION BY dept_id ORDER BY salary DESC)
ORDER BY salary DESC;
```

> 窗口链在我使用的当前版本 3.53.4 好像有 BUG，行为和不使用窗口链不一致，不知道是我理解的问题还是真有 BUG，但问了 AI 它也觉得行为不应该不一致。

## EXPLAIN
`EXPLAIN` 是 SQLite 提供的调试与分析工具，用于在不修改数据的前提下查看某条语句将会如何执行。单独出现 `EXPLAIN` 关键字时，该语句返回原本用于执行该命令的虚拟机指令序列，SQLite 的工作原理就是将 SQL 语句翻译成字节码，然后在虚拟机中运行该字节码，一般只在排查极端问题时才会查看。而 `EXPLAIN QUERY PLAN` 返回的则是**查询计划 query plan** 的高级信息，说明如何扫描各表、如何使用索引等，日常排查性能问题主要用它，下面也只介绍 `EXPLAIN QUERY PLAN`。它们的基本语法形式如下：  

``` SQL
EXPLAIN statement;             -- 输出低级虚拟机指令（字节码）
EXPLAIN QUERY PLAN statement;  -- 输出高级查询计划（日常排查用这个）
```

`EXPLAIN QUERY PLAN` 的结果集为 `id`、`parent`、`notused`、`detail` 四列，真正需要关注的是 `detail` 列。CLI 中默认不会输出结果集表格，而是渲染为树状结构，方便查看。`detail` 以 `SCAN` 开头表示全表扫描，会遍历表中的全部记录，以 `SEARCH` 开头则表示只访问满足条件的一部分记录，通常意味着用上了索引。常见片段如下表：  

| <font size=2>detail 示例</font> | <font size=2>说明</font> |
| :---- | :---- |
| <font size=2>`SCAN t1`</font> | <font size=2>对 t1 全表扫描</font> |
| <font size=2>`SEARCH t1 USING INDEX i1 (a=?)`</font> | <font size=2>用索引 i1 定位，`WHERE` 中的 `a=?` 用到了该索引</font> |
| <font size=2>`SEARCH t1 USING COVERING INDEX i2 (a=?)`</font> | <font size=2>用覆盖索引，查询所需列全在索引里，无需回表</font> |
| <font size=2>`SEARCH t1 USING INTEGER PRIMARY KEY (rowid=?)`</font> | <font size=2>按 rowid 精确定位</font> |
| <font size=2>`USE TEMP B-TREE FOR ORDER BY`</font> | <font size=2>排序无法借助索引，额外建了临时 B 树，通常是值得优化的信号</font> |

另外，`detail` 中子查询会作为外层查询的子节点显示，还会显示是否是 `CORRELATED` 相关子查询或 `SCALAR` 标量子查询，出现 `CO-ROUTINE` 表示子查询以协程逐行提供结果，`MATERIALIZE` 表示子查询被物化进了临时表，而没有独立的子查询节点则说明它被展平进了外层查询。另外，复合查询会显示为 `COMPOUND QUERY`。更多详细内容，详见官方文档 [EXPLAIN QUERY PLAN](https://www.sqlite.org/eqp.html)。下面给出一个建立索引前后的简单示例：  

``` SQL
-- 未建索引：只能全表扫描
EXPLAIN QUERY PLAN
SELECT name FROM employees WHERE dept_id = 1;
```

输出：  

``` Console
QUERY PLAN
`--SCAN employees
```

``` SQL
CREATE INDEX idx_emp_dept ON employees (dept_id); -- 建索引后，同样的查询改为用索引定位

EXPLAIN QUERY PLAN
SELECT name FROM employees WHERE dept_id = 1;
```

输出：  

``` Console
QUERY PLAN
`--SEARCH employees USING INDEX idx_emp_dept (dept_id=?)
```

## ANALYZE
SQLite 使用**基于代价的查询计划器 cost-based query planner**，对同一个查询它会估算多种执行方案的总代价，选择代价最低的一种，而估算的依据是数据库中记录的统计信息。而 `ANALYZE` 命令可以收集有关表和索引的统计信息，并将收集到的信息存储在数据库的内部表中（默认情况下，写入 `sqlite_stat1`），以便查询优化器可以访问这些信息并利用它们来做出更好的查询规划选择。若从未执行过 `ANALYZE`，SQLite 没有真实统计信息，只能按内置的默认猜测值估算各方案代价，因此选出的索引可能偏离实际最优，带有较大任意性。<u>`ANALYZE` 并非必需，但是如果应用程序执行复杂的查询，且这些查询可能存在多个查询计划，那么运行 `ANALYZE` 后，查询规划器将能够更好地选择最佳计划，这可以显著提升某些查询的性能</u>。  

``` SQL
ANALYZE;                           -- 分析所有已附加数据库中的全部表与索引
ANALYZE schema_name;               -- 只分析指定数据库中的全部表与索引
ANALYZE [schema_name.]table_name;  -- 只分析指定表及其关联索引
ANALYZE [schema_name.]index_name;  -- 只分析指定索引
```

直接运行 `ANALYZE` 命令会扫描整个数据库，在大型数据库上可能非常耗时，SQLite 官方推荐定期运行 `PRAGMA optimize` 命令，它通常是一个空操作 no-op，但如果对查询规划器有用，它偶尔会对数据库中的个别表运行一个或多个 `ANALYZE` 子命令。官方同时建议，针对不同连接时长的数据库，应采取不同的策略：  
1. 对于数据库连接持续时间较短的应用程序，应在关闭每个数据库连接之前运行一次 `PRAGMA optimize;`；  
2. 使用长期数据库连接的应用程序应在首次打开连接时运行 `PRAGMA optimize=0x10002;`，然后定期运行 `PRAGMA optimize;`，例如每天一次，但如果数据库数据增长很快，则运行次数应更多；  
3. 所有应用程序在模式更改后都应该运行 `PRAGMA optimize;`，尤其是在执行一个或多个 `CREATE INDEX` 语句之后。  

`PRAGMA optimize` 命令通常只会考虑对之前被同一数据库连接查询过的表或 `sqlite_stat1` 表中没有条目的表运行 `ANALYZE` 操作。但如果运行 `PRAGMA optimize=0x10000;`，将检查所有表，以确定它们是否可以从 `ANALYZE` 中受益，若数据库连接首次打开没有查询历史记录时，建议运行它。

## VACUUM
`VACUUM` 命令会重建数据库文件，并将其重新打包到尽可能小的磁盘空间中，以回收未使用的空间、减少碎片并优化性能。需要此操作的主要原因有：①删除数据只是把相应的页标记为可复用并放入**空闲列表 free list**，数据库文件会比实际需要的空间大，只有 `VACUUM` 才会真正回收这些空间；②频繁的增删改操作会导致数据库碎片化，表与索引的数据散落在文件各处，运行 `VACUUM` 可以确保它们大致上连续存储，提高 I/O 性能；③因为数据库的 `page_size` 或 `auto_vacuum` 属性通常情况下需要在创建数据库文件之前配置，运行时配置无效，需要在调用 `PRAGMA` 命令之后调用 `VACUUM` 重建数据库文件。其基本语法形式如下：  

``` SQL
VACUUM;                     -- 重建主数据库 main
VACUUM schema_name;         -- 重建指定的已附加数据库
VACUUM INTO 'file_name';    -- 把整理后的副本写入新文件，常用来做备份
```

`VACUUM` 命令的工作原理是将数据库内容复制到一个临时数据库文件中，然后用该临时文件的内容覆盖原始文件，覆盖原始文件时，会使用回滚日志或 WAL 文件。也就是说，执行 VACUUM 操作时，需要占用相当于原始数据库文件大小两倍的可用磁盘空间。`VACUUM INTO` 直接使用指定的文件代替临时数据库，同时省略了复制的步骤。另外，`VACUUM`（非 `VACUUM INTO`）是一个写入操作，因此如果另一个数据库连接持有阻止写入的锁，则 `VACUUM` 操作将失败。

除了使用 `VACUUM` 命令在数据删除后回收空间外，还可以使用自动清理模式，通过 `PRAGMA schema.auto_vacuum` 设置，可以自动回收空闲页面，但仅截断文件中的空闲列表页，不会对数据库进行碎片整理或重新打包单个数据库页，由于它会在文件内移动页面，自动清理反而可能会加剧碎片问题。其三种模式如下：  

``` SQL
PRAGMA auto_vacuum;                -- 查询当前模式
PRAGMA auto_vacuum = NONE;         -- 0，关闭自动清理（默认值）
PRAGMA auto_vacuum = FULL;         -- 1，每次事务提交都截断文件以移除空闲列表页
PRAGMA auto_vacuum = INCREMENTAL;  -- 2，空间回收在后续的 PRAGMA incremental_vacuum 命令中进行
```

另外，只有在数据库是新建的（尚未创建任何表），才能将 `auto_vacuum` 从 NONE 改为 FULL 或 INCREMENTAL（完全清理和增量清理模式可以随时切换）。要修改自动清理模式，需要先执行 `PRAGMA auto_vacuum = ...;` 指定新模式，再执行一次 `VACUUM` 重组数据库才会真正应用。  

## PRAGMA
`PRAGMA` 是 SQLite 特有的语句，用来查询或修改 SQLite 自身环境变量。<u>有些编译指示 `PRAGMA` 在 SQL 编译阶段生效，而非执行阶段。也就是说，如果使用 C 语言的 API 或类似的封装接口，编译指示可能需要在 `sqlite3_prepare()` 调用期间执行，而不是像普通 SQL 语句那样在 `sqlite3_step()` 调用期间执行。编译指示是在 `sqlite3_prepare()` 还是 `sqlite3_step()` 调用期间执行，取决于编译指示本身以及 SQLite 的具体版本</u>。其语法有查询与设置两种形式：  

``` SQL
PRAGMA [schema_name.]name;         -- 查询当前值
PRAGMA [schema_name.]name = value; -- 设置
PRAGMA [schema_name.]name(value);  -- 带参数调用，多为查询类，如 PRAGMA table_info(employees);
```

这里只简单介绍较为常用的 `PRAGMA` 语句，具体内容详见 SQLite 官方文档 [PRAGMA Statements](https://www.sqlite.org/pragma.html#syntax)：  

| <font size=2>语句</font> | <font size=2>说明</font> |
| :---- | :---- |
| <font size=2>`PRAGMA page_size;`</font> | <font size=2>查看或设置数据库的页大小，SQLite 3.12.0（2016-03-29）以后默认 4096 字节，之前默认 1024 字节</font> |
| <font size=2>`PRAGMA encoding;`</font> | <font size=2>查看或设置数据库的文本编码（UTF-8、UTF-16等），使用 `sqlite3_open()` 创建默认 UTF-8</font> |
| <font size=2>`PRAGMA user_version;`</font> | <font size=2>读写数据库文件头中的用户自定义版本号，常用于数据库迁移时判断版本</font> |
| <font size=2>`PRAGMA journal_mode = WAL;`</font> | <font size=2>切换日志模式，详见[日志模式](#日志模式)小节</font> |
| <font size=2>`PRAGMA synchronous = NORMAL;`</font> | <font size=2>设置同步级别，在安全性与写入性能之间取舍，详见[日志模式](#日志模式)小节</font> |
| <font size=2>`PRAGMA foreign_keys = ON;`</font> | <font size=2>开启外键约束检查，注意它默认是关闭的，详见 [FOREIGN KEY](#FOREIGN-KEY)小节</font> |
| <font size=2>`PRAGMA busy_timeout = 5000;`</font> | <font size=2>设置遇到锁冲突时的等待毫秒数，避免直接返回 `SQLITE_BUSY` 错误，默认为 0 不会进行任何等待</font> |
| <font size=2>`PRAGMA cache_size;`</font> | <font size=2>查看或设置当前连接在内存中的页数或页缓存大小，正数时为页数，负数时为 KB 大小，默认 -2000，即 2,048,000 字节</font> |
| <font size=2>`PRAGMA table_info(表名);`</font> | <font size=2>查看表的列定义，包括列名、类型、是否允许 NULL、默认值以及是否为主键，是代码中读取表结构最常用的方式</font> |
| <font size=2>`PRAGMA index_list(表名);`</font> | <font size=2>查看表上已经建立了哪些索引</font> |
| <font size=2>`PRAGMA page_count;`</font> | <font size=2>查看数据库文件当前占用的总页数</font> |
| <font size=2>`PRAGMA optimize;`</font> | <font size=2>让查询优化器按需收集统计信息，详见 [ANALYZE](#ANALYZE) 小节</font> |

另外，大多数 `PRAGMA` 是连接级的，只对当前连接生效（如 `foreign_keys`、`busy_timeout`、`cache_size`），关闭连接后即失效。而 `journal_mode`、`auto_vacuum`、`page_size`、`user_version` 这类会写进数据库文件头，是持久化的。

## 内置函数
SQLite 内置函数按返回值分类，可以分为**标量函数 scalar function**（每行返回一个值）、**聚合函数 aggregate function**（一组行返回一个值，加上 `OVER` 就是窗口函数）、**表值函数 table-valued function**（返回多行多列的临时结果集）。官方文档将它们分在了不同的文档里面，包括[核心函数 core function](https://www.sqlite.org/lang_corefunc.html)、[日期时间函数 Date & Time function](https://www.sqlite.org/lang_datefunc.html)、[聚合函数 aggregate function](https://www.sqlite.org/lang_aggfunc.html)、[窗口函数 window function](https://www.sqlite.org/windowfunctions.html)、[数学函数 math function](https://www.sqlite.org/lang_mathfunc.html) 和 [JSON 函数 JSON function](https://www.sqlite.org/json1.html)。其中聚合函数在 [GROUP BY 子句](#GROUP-BY-子句)小节有提及，窗口函数在[窗口函数 window function](#窗口函数-window-function) 小节有提及。另外，自定义函数可以通过 C API `sqlite3_create_function()` 注册（各语言绑定通常都封装了对应的注册 API）。下面仅简单介绍 core function 中的常用函数，其它函数请看官方文档。

> 需要注意的是，这些内置函数的可用性取决于 SQLite 的版本与编译选项。math function 自 SQLite 3.35.0（2021-03-12）起才引入，且需要在编译时定义宏 `SQLITE_ENABLE_MATH_FUNCTIONS` 才会被编入。JSON function 自 SQLite 3.38.0（2022-02-22）起默认内置，在此之前只是可选的 JSON1 扩展，需要定义宏 `SQLITE_ENABLE_JSON1`，而 3.38.0 起若定义了宏 `SQLITE_OMIT_JSON` 则仍会被整体裁掉。另外，SQLite 通常随语言运行时一起分发，其版本与编译选项都取决于具体的运行时，因此同一个函数在不同环境中未必都存在，使用前最好先用 `SELECT sqlite_version();` 查看版本、用 `PRAGMA compile_options;` 查看编译选项（如是否启用 `ENABLE_MATH_FUNCTIONS`），再用 `PRAGMA function_list;` 列出当前实际可用的函数确认一下。

***①数值函数***  
* `abs(X)`：返回数值 X 的绝对值；X 为 NULL 时返回 NULL，若 X 是无法转换为数值的字符串或 BLOB 则返回 0.0；  
* `round(X)`、`round(X, Y)`：把 X 四舍五入到小数点后 Y 位，Y 省略或为负数时均按 0 处理；  
* `sign(X)`：X 为正数、零、负数时分别返回 +1、0、-1，X 为 NULL、BLOB 或无法无损转为数值的字符串时返回 NULL；  
* `random()`：返回一个范围在 -9223372036854775807 到 +9223372036854775807 之间的伪随机整数；   

***②字符串函数***  
* `length(X)`：对字符串返回其 Unicode 码点数（而非字节数），通常可以理解为返回实际字符数。对 BLOB 返回字节数，X 为 NULL 时返回 NULL；  
* `lower(X)`、`upper(X)`：按 ASCII 规则把字符串转换为全小写或全大写，非 ASCII 字符需借助 ICU 扩展处理；  
* `substr(X, Y)`、`substr(X, Y, Z)`、`substring(...)`：截取 X 中从第 Y 个字符（Y 为负表示从右计数）起始的 Z 个字符，Z 省略时截取至末尾。`substring()` 是 `substr()` 的别名；  
* `format(FORMAT, ...)`、`printf(FORMAT, ...)`：类似 C 语言的 `printf()`，%n 格式被忽略，%p 等价于 %X，%z 等价于 %s。`printf()` 是 `format()` 的别名；  
* `concat(X, ...)`、`concat_ws(SEP, X, ...)`：把各参数拼接为一个字符串，`concat_ws()` 以 SEP 作为分隔符；  

> 对于字符串拼接，SQLite 还支持 SQL 标准的字符串连接运算符 `||`，它把左右两个操作数拼接成一个字符串，例如 `'Hello' || ' ' || 'World'` 得到 `'Hello World'`。它与 `concat()` 的关键区别在于对 `NULL` 的处理：`||` 的任一操作数为 `NULL` 时，整个表达式的结果就是 `NULL`，而 `concat()` 会直接忽略 `NULL` 参数。另外需要注意优先级，`||` 的优先级高于 `*`、`/`、`%`、`+`、`-` 等算术运算符，因此 `1 + 2 || 3` 会先算 `2 || 3` 而不是 `1 + 2`，书写时最好用括号明确结合顺序。

***③表达式函数***  
* `iif(X, Y, Z)`、`if(X, Y, Z)`：等价于表达式 `CASE WHEN X THEN Y ELSE Z END`，`if()` 是 `iif()` 的别名；  
* `like(X, Y)`、`like(X, Y, Z)`：等价于表达式 `Y LIKE X [ESCAPE Z]`，注意 X 是模式、Y 是待匹配字符串，参数顺序与 `LIKE` 运算符相反；  
* `glob(X, Y)`：等价于表达式 `Y GLOB X`，参数顺序与 `GLOB` 运算符相反；  

***④信息与状态函数***  
* `typeof(X)`：返回 X 所属存储类的名称，可能为 "null"、"integer"、"real"、"text" 或 "blob"；  
* `sqlite_version()`、`sqlite_source_id()`：分别返回当前 SQLite 库的版本号与源码标识；   

# 在程序中调用
## C/C++ API
SQLite 对外暴露的是一套 **C 语言 API**（头文件 `sqlite3.h`），C++ 可以直接调用。整个接口有超过 255 个 API，但大部分很少用到，初学者只需关注核心 API 即可，这里也只介绍核心 API，其它内容详见官方文档 [The SQLite C/C++ Interface](https://www.sqlite.org/c3ref/intro.html)。整个接口围绕两个核心对象以及八个函数展开：**数据库连接对象 `sqlite3`** 与**预编译语句对象 `sqlite3_stmt`**（stmt 即 statement），八个函数如下：  

【书签】

1. `sqlite3_open()`：用于打开数据库文件（不存在则创建），得到**数据库连接对象**，；  
2. `sqlite3_prepare()`：
3. `sqlite3_bind()`：
4. `sqlite3_step()`：
5. `sqlite3_column()`：
6. `sqlite3_finalize()`：
7. `sqlite3_close()`：
8. `sqlite3_exec()`：

## C# API

**System.Data.SQLite** 与 **Microsoft.Data.Sqlite**
