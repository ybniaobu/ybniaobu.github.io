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
description: XXXXXXXXXXXXXXXXXXXXXXXXXXXXX
---

# SQLite 简介
**SQLite** 是一款轻量级的**内嵌式关系型数据库 embedded relational database**，由 D. Richard Hipp 于 2000 年开发并发布，它具有如下特质：  
①**无服务器 serverless**：诸如 MySQL 或 PostgreSQL 之类的**关系型数据库管理系统 RDBMS** 通常需要一个独立的服务器进程才能运行，需要访问数据库的应用程序通过 TCP/IP 协议来发送和接收请求，这种结构被称为**客户端/服务器架构 client/server architecture**。而 SQLite 数据库则直接与访问它的应用程序集成在一起，应用通过直接读写存储在磁盘上的数据库文件（`.db` 或 `.sqlite`）来与数据库交互，整个过程不需要任何服务器进程；  
②**自包含 self-contained**：SQLite 以 C 语言编写，几乎不依赖任何外部组件或第三方库，也无需安装，整个引擎被完整地封装在单个库文件（如 `sqlite3`）之中，开发时只需把这个库链接进自己的应用程序即可直接使用，无需进行任何配置或管理，这使得 SQLite 能够适用于任何运行环境；  
③**零配置 zero-configuration**：在使用 MySQL、PostgreSQL 等传统数据库之前，往往需要安装部署、编写配置文件、创建用户账户并设置权限等一系列繁琐步骤；而使用 SQLite 则无需任何配置即可开箱即用，既没有需要启动或停止的服务，也没有用户与权限需要管理；  
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

    TYPEOF(100) | TYPEOF(10.0) | TYPEOF('100') | TYPEOF(x'1000') | TYPEOF(NULL)
    ------------+--------------+---------------+-----------------+-------------
    integer     | real         | text          | blob            | null

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
数据表操作对应 SQL 中的**数据定义语言 DDL (Data Definition Language)**，它与负责读写行数据的**数据操纵语言 DML (Data Manipulation Language)** 相对，DDL 关注的是**模式对象 schema object** 本身的定义，而不关心对象里存放了什么数据。SQLite 的 DDL 语句以 `CREATE`、`ALTER`、`DROP` 三类关键字为主，可作用于的模式对象包括：**表 table**、**索引 index**、**视图 view** 和**触发器 trigger**。这几类对象之间存在依附关系：索引与触发器必须依附于某张表，删除表时它们会被一并删除，而视图虽然独立存在，一旦其定义引用的表被删除便会失效。因此这里先讲表相关操作，而索引、视图与触发器的语法则留到进阶特性章节中介绍。

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
* 可选的 `WITHOUT ROWID` 选项（SQLite 3.8.2 及更高版本中支持）。表中的每一行都有一个隐式的 `rowid` 列，如果不希望 SQLite 创建 `rowid` 列，可以指定 `WITHOUT ROWID` 选项，但此时必须指定一个主键 PRIMARY KEY。`rowid` 和 PRIMARY KEY 的关系详见约束的 PRIMARY KEY 小节。

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
**主键 PRIMARY KEY** 是用于在表中唯一标识每一行的一个列或一组列，每张表最多只能有一个主键。如果把 PRIMARY KEY 关键字加在某个列定义上，即**列约束**，那么该表的主键就由这一列单独构成。也可以写成**表级约束**来定义复合主键，如 `PRIMARY KEY (order_id, line_no)`。

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
**唯一约束 UNIQUE** 用于确保某个列（或一组列）中的值在表内互不重复，插入或更新时若出现重复值，SQLite 会拒绝执行并返回错误。`UNIQUE` 既可以作为列约束写在单个列定义上，也可以写成表级约束来约束多列组合的整体唯一性，此时只要组合中有一列不同即视为不重复。

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
| <font size=2>REPLACE</font> | <font size=2>先删除导致冲突的已存在行，再继续插入或更新；若违反的是 NOT NULL，则用该列默认值替换 NULL，无默认值时退化为 ABORT</font> |

`ON CONFLICT` 子句可以直接跟在约束定义之后；而在 `INSERT` 和 `UPDATE` 语句中，`ON CONFLICT` 关键字要换成 `OR`，写成 `INSERT OR IGNORE`、`UPDATE OR REPLACE` 等形式，语句级指定的算法会覆盖约束定义中指定的算法。示例如下：  

``` SQL
CREATE TABLE users (
    id    INTEGER PRIMARY KEY,
    email TEXT UNIQUE ON CONFLICT IGNORE,         -- 约束级：邮箱重复时静默跳过
    name  TEXT NOT NULL ON CONFLICT FAIL          -- 约束级：NULL 时报错但保留已修改的行
);

INSERT OR IGNORE INTO users (id, email, name) VALUES (1, 'a@x.com', 'Alice'); -- 重复时静默跳过
INSERT OR REPLACE INTO users (id, email, name) VALUES (1, 'b@x.com', 'Bob');  -- 删除旧行后插入
```

> 注意不要把这里的 `ON CONFLICT` 与 3.24.0（2018-06-04）新增的 UPSERT 语法（`INSERT ... ON CONFLICT (列名) DO UPDATE ...`）相混淆，后者是 `INSERT` 语句的扩展。

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
数据操作语句即 **DML (Data Manipulation Language)**，负责对表中已有的数据进行读写，包括查询 `SELECT`、插入 `INSERT`、更新 `UPDATE` 与删除 `DELETE` 四类。与之相对，前面介绍的 `CREATE TABLE`、`ALTER TABLE` 等属于 **DDL (Data Definition Language)**。本节按查、增、改、删的顺序依次介绍这四类语句。

## 查询 SELECT
**查询 SELECT** 是 SQL 中使用频率最高的语句，用于从一个或多个表中检索数据，它只读取数据而不修改数据库。SQLite 中 `SELECT` 语句的完整语法非常复杂，核心形式大致如下（其中中括号 `[]` 表示可选）：  

``` SQL
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
* 各子句的书写顺序是固定的，不能随意调换，但书写顺序不等于逻辑执行顺序，基本的逻辑执行顺序大致为：① `FROM / JOIN` 确定数据来源并连接多表，得到候选行集合；② `WHERE` 对候选行逐行过滤，仅保留满足条件的行；③ `GROUP BY` 把剩余行按分组列划分为若干组；④ `HAVING` 对每个组求值其过滤条件，丢弃不满足条件的组，条件中若含聚合函数，则在该组的全部行上求值；⑤ `SELECT` 计算输出表达式，生成结果集；⑥ `DISTINCT` 对结果集去重；⑦ `ORDER BY` 对结果集排序；⑧ `LIMIT` 与 `OFFSET` 截取要返回的行。

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
FROM employees AS e;                              -- 表别名

SELECT 1 + 1, 'hello', datetime('now');           -- 无 FROM 时直接对表达式求值
```

* 若省略 `FROM` 子句，`SELECT` 直接对表达式列表求值并返回单行结果，常用于测试函数或查看环境信息；  
* 别名若为关键字或包含空格等特殊字符，需用双引号括起来，如 `AS "user name"`，注意标识符用双引号，与字符串的单引号不同；  
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

* 数据来源可以指定所属的 *schema_name*，不写时 SQLite 会按 `temp`、`main`、已附加数据库的顺序依次查找，附加数据库小节提过这点；    
* 可以在 `FROM` 中并列写出多个数据源并用逗号分隔，其效果等价于对它们做**交叉连接 CROSS JOIN**。可以将连接条件写在 `WHERE` 中，但更推荐显式的 `JOIN ... ON` 写法，详见 JOIN 子句小节；  
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
* `IN` 与 `NOT IN` 运算符左侧接一个表达式，右侧接一个值列表或一个子查询。当右操作数是空集合时，无论左操作数是什么，甚至左操作数是 NULL，`IN` 的结果都是假，而 `NOT IN` 的结果都是真。

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

``` SQL
-- 假设 employees 表的数据为（name | dept_id | salary）：
-- Alice|1|12000、Bob|1|9000、Carol|2|15000、Dave|2|8000、Eve|NULL|11000

SELECT dept_id,
       count(*)                AS 人数,     -- 该组的行数
       sum(salary)             AS 工资合计,  -- 该组 salary 之和
       avg(salary)             AS 平均工资,  -- 该组 salary 平均值
       max(salary)             AS 最高工资,  -- 该组 salary 的最大值
       group_concat(name, '/') AS 成员      -- 把该组的姓名拼成一个字符串
FROM employees
GROUP BY dept_id;
```

输出：  

    dept_id | 人数 | 工资合计 | 平均工资 | 最高工资 |  成员
    --------+------+----------+----------+----------+-----------
    1       | 2    | 21000    | 10500.0  | 12000    | Alice/Bob
    2       | 2    | 23000    | 11500.0  | 15000    | Carol/Dave
    NULL    | 1    | 11000    | 11000.0  | 11000    | Eve

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

    dept_id | 人数 | 平均工资
    --------+------+----------
    1       | 2    | 10500.0
    2       | 2    | 11500.0

### 排序 ORDER BY
【书签】

`ORDER BY` 子句用于对结果集排序，可以指定一个或多个排序键，每个键后可跟 `ASC`（升序，默认）或 `DESC`（降序）；指定多个键时先按第一个键排，第一个键相同再按第二个键排，依此类推。排序键还可以直接写**结果列的序号**（从 1 开始）或**列别名**来代替列名，也可以在排序键后接 `COLLATE` 临时指定排序规则（详见 `CREATE TABLE` 的 `COLLATE` 小节）。  

``` SQL
SELECT * FROM employees ORDER BY dept_id ASC, salary DESC; -- 先按部门升序，再按工资降序
SELECT name, salary AS s FROM employees ORDER BY s DESC;   -- 按列别名排序
SELECT name, salary FROM employees ORDER BY 2 DESC;        -- 按结果第 2 列排序
SELECT * FROM users ORDER BY name COLLATE NOCASE;          -- 忽略大小写排序
```

* 默认情况下 `NULL` 被视为最小值，因此升序时 `NULL` 排在最前，降序时排在最后。从 SQLite 3.30.0（2019-10-04）开始，可以用 `NULLS FIRST` 或 `NULLS LAST` 显式指定 `NULL` 的位置；  
* <u>不写 `ORDER BY` 时，结果行的返回顺序是不确定的</u>，即使某次查询看似按插入顺序返回，也不应依赖这一行为。  

### 限制行数 LIMIT 与 OFFSET
`LIMIT` 子句用于限制返回的最大行数，`OFFSET` 子句用于指定先跳过结果集中的多少行，两者结合常用于分页。`LIMIT` 有两种等价写法，如下：  

``` SQL
SELECT * FROM employees ORDER BY salary DESC LIMIT 10;     -- 工资最高的前 10 行
SELECT * FROM employees ORDER BY id LIMIT 10 OFFSET 20;    -- 跳过 20 行后取 10 行（第 3 页）
SELECT * FROM employees ORDER BY id LIMIT 20, 10;          -- 与上一句等价：LIMIT offset, count
SELECT * FROM employees LIMIT -1 OFFSET 5;                 -- LIMIT 为负数表示不限制行数
```

* `LIMIT` 与 `OFFSET` 的值可以是常量表达式，如 `LIMIT 2 * 5`；若表达式的值为 NULL 或无法无损转换为整数则会报错；  
* 不允许单独写 `OFFSET` 而不写 `LIMIT`，此时应使用 `LIMIT -1 OFFSET n`；  
* 分页查询务必配合 `ORDER BY` 使用，否则每次返回的"页"内容可能不一致。  

### 条件表达式 CASE
**CASE 表达式**用于在 SQL 中实现条件分支，根据条件的真假返回不同的值，它有**简单形式 simple CASE** 和**搜索形式 searched CASE** 两种写法：  

``` SQL
-- 简单形式：把 CASE 后的表达式依次与各 WHEN 的值比较
CASE case_expr WHEN value_1 THEN result_1 [WHEN value_2 THEN result_2 ...] [ELSE default_result] END

-- 搜索形式：各 WHEN 后为独立的布尔表达式
CASE WHEN condition_1 THEN result_1 [WHEN condition_2 THEN result_2 ...] [ELSE default_result] END
```

* 两种形式都按从上到下的顺序依次求值，<u>一旦某个 `WHEN` 匹配成功就立即返回对应的 `THEN` 结果，后续分支不再求值</u>，因此分支的排列顺序会影响结果；若没有任何分支匹配，则返回 `ELSE` 的结果，省略 `ELSE` 时返回 `NULL`；  
* CASE 是**表达式**而非语句，其结果是一个值，可以用在任何允许表达式的地方：`SELECT` 列表、`WHERE`、`ORDER BY`、`GROUP BY`、`HAVING`、`UPDATE ... SET` 等；  

``` SQL
SELECT name, salary,
       CASE
           WHEN salary < 5000  THEN '低'
           WHEN salary < 10000 THEN '中'
           ELSE '高'
       END AS level
FROM employees;

-- 配合聚合函数实现条件统计（行转列的常用手法）
SELECT dept_id,
       COUNT(CASE WHEN salary >= 10000 THEN 1 END) AS high_cnt,  -- 不匹配时返回 NULL，COUNT 忽略
       SUM(CASE WHEN gender = 'F' THEN 1 ELSE 0 END) AS female_cnt
FROM employees
GROUP BY dept_id;
```

> 注意 `CASE x WHEN NULL ...` 永远无法匹配，因为与 `NULL` 的比较结果不是真，判断 `NULL` 应写成搜索形式 `WHEN x IS NULL`。

### 子查询 Subquery
**子查询 subquery** 是嵌套在其它 SQL 语句中的 `SELECT`，按返回结果的形态可分为三类：返回单个值的**标量子查询 scalar subquery**、返回单列多行的**列子查询**、返回多行多列的**表子查询**。  

``` SQL
SELECT * FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);   -- 标量子查询：高于平均工资的员工

SELECT d.name, t.cnt
FROM departments AS d
JOIN (SELECT dept_id, COUNT(*) AS cnt FROM employees GROUP BY dept_id) AS t
  ON d.id = t.dept_id;                                -- 表子查询：写在 FROM 中必须取别名
```

* 标量子查询只允许返回单列；若返回多行则只有第一行被采用，若没有返回任何行则其结果为 `NULL`；  
* `EXISTS (子查询)` 是一个谓词运算符，当子查询返回**至少一行**时结果为真，`NOT EXISTS` 则正好相反。它只关心子查询是否返回行，<u>不关心返回的内容是什么</u>，因此子查询的选择列表通常习惯写成 `SELECT 1` 或 `SELECT *`；`EXISTS` 几乎总是与**相关子查询 correlated subquery** 配合使用：子查询中引用外层查询的列时，外层每处理一行，子查询就重新求值一次；  

``` SQL
SELECT * FROM departments AS d
WHERE EXISTS (SELECT 1 FROM employees AS e WHERE e.dept_id = d.id);     -- 存在员工的部门
```

> `EXISTS` 与 `IN` 在多数情况下可以互相改写，但当子查询结果中可能含有 `NULL` 时两者语义不同：`IN` 受三值逻辑影响可能整体返回非真，而 `EXISTS` 只看行是否存在，不受 `NULL` 干扰。因此表达"是否存在"语义的检查，优先使用 `EXISTS` 更稳妥。

### 复合查询
**复合查询 compound query** 用集合运算符把多个 `SELECT` 的结果合并为一个结果集，共支持四种运算符，如下表：  

| <font size=2>运算符</font> | <font size=2>说明</font> |
| :---- | :---- |
| <font size=2>`UNION`</font> | <font size=2>并集，去除重复行</font> |
| <font size=2>`UNION ALL`</font> | <font size=2>并集，保留重复行，无需去重故性能更好</font> |
| <font size=2>`INTERSECT`</font> | <font size=2>交集，只保留两边都出现的行</font> |
| <font size=2>`EXCEPT`</font> | <font size=2>差集，保留只在左边出现的行</font> |

``` SQL
-- 汇总正式员工与合同工的联系方式，可能出现同名的行会被去重
SELECT name, phone FROM employees
UNION
SELECT name, phone FROM contractors;

-- 在旧表中存在、但已被新表删除的用户（数据迁移核对）
SELECT email FROM old_users
EXCEPT
SELECT email FROM users;

SELECT name FROM employees WHERE dept_id = 1
UNION
SELECT name FROM employees WHERE dept_id = 2
ORDER BY name;                          -- ORDER BY 只能写在最后一个 SELECT 之后
```

* 参与复合的每个 `SELECT` 必须返回相同的列数，对应列的值按从左到右各 `SELECT` 所声明的排序规则进行比较，结果集的列名取自最左边第一个 `SELECT`；  
* `INTERSECT` 的优先级高于 `UNION` 与 `EXCEPT`，`UNION` 与 `EXCEPT` 同级且从左到右结合，可用括号改变求值顺序；`EXCEPT` 是**不可交换**的，`A EXCEPT B` 与 `B EXCEPT A` 的结果一般不同；  
* `ORDER BY` 与 `LIMIT` 只能写在复合查询的最末尾，作用于整个结果集，排序键只能引用结果集的列（第一个 `SELECT` 的列名或序号），若要对某个 `SELECT` 单独排序需将其改写为子查询；集合运算判断重复的规则与 `DISTINCT` 一致，`NULL` 被视为相等。  

## 插入 INSERT
**插入 INSERT** 语句用于向表中添加新行，基本语法如下：  

``` SQL
INSERT [OR conflict_algorithm] INTO [schema_name.]table_name [(column_list)]
{VALUES (expr_list) [, ...] | select_stmt | DEFAULT VALUES}
[RETURNING result_list];
```

* `VALUES` 形式插入一行或多行；`INSERT ... SELECT` 形式把查询结果整体插入；`DEFAULT VALUES` 插入一行全部为默认值的行；  
* 列清单 *column_list* 可以省略，此时按表中列的定义顺序为全部列提供值，<u>实际开发中建议总是显式写出列清单</u>，避免表结构变化后顺序错位；未出现在列清单中的列取默认值（无默认值时为 `NULL`）；  
* 插入的值会按目标列的类型亲和性进行隐式转换，并受列上各类约束的检查，冲突时按 `ON CONFLICT` 子句指定的算法处理，详见 `CREATE TABLE` 的约束小节；  
* 向 `INTEGER PRIMARY KEY`（rowid 别名）列插入 `NULL` 时，SQLite 会自动分配 rowid 值；`RETURNING` 子句（3.35.0，2021-03-12 起支持）用于在插入后直接返回受影响行的列值或表达式，配合它可以方便地取回自动分配的 id；  

``` SQL
INSERT INTO departments (id, name) VALUES (1, '研发部'), (2, '市场部');   -- 一次插入多行

INSERT INTO employees (name, dept_id) VALUES ('Alice', 1);               -- 未列出的列取默认值

INSERT INTO employees DEFAULT VALUES;                                    -- 整行均为默认值

INSERT INTO employees_archive (id, name, dept_id)                        -- 把查询结果插入另一张表
SELECT id, name, dept_id FROM employees WHERE hire_date < '2010-01-01';

INSERT INTO employees (name, dept_id) VALUES ('Bob', 2)                  -- RETURNING 返回实际插入的列值
RETURNING id, name;
```

### UPSERT
**UPSERT** 是 SQLite 3.24.0（2018-06-04）起支持的 `INSERT` 扩展语法（借鉴自 PostgreSQL），用于在单条语句中原子地实现"**存在则更新、不存在则插入**"，其语法是在普通 `INSERT` 之后追加一个 `ON CONFLICT` 子句：  

``` SQL
INSERT INTO table_name (column_list) VALUES (...)
ON CONFLICT (conflict_target) DO UPDATE SET column = expr [, ...] [WHERE expr];

INSERT INTO table_name (column_list) VALUES (...)
ON CONFLICT [(conflict_target)] DO NOTHING;
```

* *conflict_target* 指定用于检测冲突的列清单（可以包含多列），这些列上必须建有**唯一索引**或 **UNIQUE 约束**，否则报错；`DO NOTHING` 形式可以省略冲突目标，表示任何唯一性冲突都直接忽略；  
* 执行时 SQLite 先尝试正常插入；若在冲突目标上发生唯一性冲突，则改为对**已存在的冲突行**执行 `DO UPDATE SET` 更新，或按 `DO NOTHING` 直接跳过。整个操作是原子的，不会出现"先查再插"两条语句之间的竞态问题；  
* 在 `DO UPDATE SET` 子句中，可以用 `excluded.列名` 引用本次尝试插入（但因冲突被排除）的值；末尾可选的 `WHERE` 用于为更新附加额外条件，条件不满足时该冲突行保持不变；  

``` SQL
CREATE TABLE visit_stats (
    page  TEXT PRIMARY KEY,
    views INTEGER NOT NULL DEFAULT 0
);

INSERT INTO visit_stats (page, views) VALUES ('/index', 1)
ON CONFLICT (page) DO UPDATE SET views = views + 1;     -- 页面已存在则计数加一

INSERT INTO visit_stats (page, views) VALUES ('/about', 1)
ON CONFLICT (page) DO UPDATE SET views = excluded.views + visit_stats.views; -- excluded 引用待插入值
```

> 注意区分三个易混淆的 `ON CONFLICT`：① `CREATE TABLE` 中约束定义上的 `ON CONFLICT` 子句（如 `email TEXT UNIQUE ON CONFLICT IGNORE`）指定的是默认冲突算法；② `INSERT OR REPLACE` 中的 `OR REPLACE` 是语句级的冲突算法；③ UPSERT 的 `ON CONFLICT ... DO UPDATE/DO NOTHING` 是语句级的显式指令，优先级最高。另外 UPSERT 也可以配合 `INSERT ... SELECT` 使用，把查询结果批量"插入或更新"到目标表，实现简易的数据同步。

### REPLACE 语句
`REPLACE` 是 SQLite 对标准 SQL 的扩展，它是 `INSERT OR REPLACE` 的简写形式，即把语句开头的 `INSERT` 直接换成 `REPLACE`，语法与 `INSERT` 完全一致。其语义为"**如果发生冲突就先删除旧行、再插入新行**"：当待插入的行与表中已有行在 `PRIMARY KEY` 或 `UNIQUE` 约束上发生冲突时，SQLite 会先删除造成冲突的已有行，然后再插入新行，因此语句总是能够成功执行；若没有任何冲突，则与普通 `INSERT` 完全等价。这里的"替换"发生在**行**的层面，与只修改个别列的 `UPDATE` 有本质区别。  

``` SQL
CREATE TABLE positions (
    code TEXT PRIMARY KEY,     -- 股票代码作为主键
    name TEXT NOT NULL,
    shares INTEGER NOT NULL CHECK (shares >= 0)
);

INSERT INTO positions (code, name, shares) VALUES ('AAPL', 'Apple', 100);
REPLACE INTO positions (code, name, shares) VALUES ('AAPL', 'Apple Inc.', 200);
-- 主键 'AAPL' 已存在：先删除旧行再插入新行，最终表中只有 (AAPL, Apple Inc., 200) 一行
```

由于其内部是"删除 + 插入"两步操作，使用时需注意以下副作用：<u>未出现在列清单中的列会被重置为默认值</u>（`UPDATE` 与 UPSERT 的 `DO UPDATE` 则不会动这些列）；<u>rowid 会改变</u>，若主键不是 `INTEGER PRIMARY KEY`，新行会获得一个全新的隐式 rowid，AUTOINCREMENT 计数也随之增长；删除旧行还会<u>连带触发删除侧的连锁反应</u>，即触发 `DELETE` 触发器以及外键的 `ON DELETE` 动作（如 `CASCADE` 级联删除子表数据），可能导致意料之外的数据丢失。  

> 若只想在冲突时更新部分列，应优先使用语义更精确、副作用更小的 UPSERT（`INSERT ... ON CONFLICT ... DO UPDATE`），REPLACE 更适合"整行覆盖"且确认无副作用影响的场景。

## 更新 UPDATE
**更新 UPDATE** 语句用于修改已有行的数据，基本语法如下：  

``` SQL
UPDATE [OR conflict_algorithm] [schema_name.]table_name
SET column_1 = expr_1 [, column_2 = expr_2, ...]
[FROM table_or_subquery]
[WHERE search_condition]
[ORDER BY ...] [LIMIT count [OFFSET skip]]
[RETURNING result_list];
```

* `SET` 子句中各列的赋值同时进行，右侧表达式引用的是**更新前的旧值**，因此交换两列的值无需借助中间变量；  
* 省略 `WHERE` 时会更新表中的**全部行**，务必确认这是预期行为；  
* `FROM` 子句（3.33.0，2022-09-29 起支持）允许基于其它表或子查询的数据进行更新，效果类似其它数据库的 `UPDATE ... JOIN`；  
* `ORDER BY` 与 `LIMIT` 配合可以把更新限制在前若干行，常用于分批处理；与 `INSERT` 一样，`UPDATE` 也支持 `RETURNING` 子句与 `OR` 冲突算法；  

``` SQL
UPDATE employees SET salary = salary * 1.1 WHERE dept_id = 1;      -- 按条件更新（省略 WHERE 则更新全部行）

UPDATE t SET a = b, b = a;                                         -- 同时赋值，交换两列

UPDATE employees                                                   -- 借助 FROM 用另一张表的数据更新
SET salary = d.base_salary
FROM (SELECT dept_id, base_salary FROM dept_standard) AS d
WHERE employees.dept_id = d.dept_id;

UPDATE jobs SET status = 'done'                                    -- 每次只处理最早的两条任务
WHERE status = 'pending'
ORDER BY created_at
LIMIT 2
RETURNING id;                                                      -- 返回被更新的行
```

## 删除 DELETE
**删除 DELETE** 语句用于移除表中的行，基本语法如下：  

``` SQL
DELETE FROM [schema_name.]table_name
[WHERE search_condition]
[ORDER BY ...] [LIMIT count [OFFSET skip]]
[RETURNING result_list];
```

* 省略 `WHERE` 时删除表中的**全部行**，表本身及其索引、触发器仍然保留（这与 `DROP TABLE` 不同）；  
* 删除全部行时，SQLite 内部可能采用**截断优化 truncate optimization**：跳过逐行扫描直接丢弃整表数据，速度极快，但此时不会触发 DELETE 触发器，`RETURNING` 也会返回空结果集。把 `WHERE` 写成恒真条件（如 `WHERE 1`）即可禁用该优化；  
* `ORDER BY`、`LIMIT` 与 `RETURNING` 的用法与 `UPDATE` 相同，用于分批删除或取回被删除的行；  
* 删除的行所占空间只是被放回空闲列表，数据库文件不会自动缩小，需要执行 `VACUUM` 回收磁盘空间；  

``` SQL
DELETE FROM employees WHERE id = 42;                 -- 按条件删除

DELETE FROM logs                                     -- 分批删除过期日志，避免长事务
WHERE created_at < datetime('now', '-30 days')
ORDER BY created_at
LIMIT 1000;

DELETE FROM employees;                               -- 删除全部行（可能触发截断优化）
DELETE FROM employees WHERE 1;                       -- 恒真条件，禁用截断优化以触发触发器
```

> `DELETE` 语句不带跨表连接能力，若要"按另一张表的条件删除"，需借助子查询改写，如 `DELETE FROM t1 WHERE id IN (SELECT id FROM t2)`。另外，频繁大批量删除会放大 **WAL 模式**下的日志体积，必要时考虑分批提交事务。

# 进阶特性
前面几章讲清了 SQLite 的类型系统以及表、数据与各类模式对象的增删改查；本章补上建立在这些基础之上的能力：把查询保存下来复用（视图）、让它跑得更快（索引）、让数据变更自动发生（触发器）、把多条语句绑成一个整体（事务），以及在不写应用代码的前提下表达复杂计算（窗口函数与 CTE）；最后是维护调优最常用的两件工具（`VACUUM` 与 `PRAGMA`）与一份内置函数速查。

## 视图 View
**视图 view** 是把一条 `SELECT` 语句保存为具名对象的机制，它本身<u>不存储任何数据</u>，每次查询视图时都会实时执行其定义中的 `SELECT`，因此可以把视图理解为一张"虚拟表"。其基本语法如下：  

``` SQL
CREATE [TEMP | TEMPORARY] VIEW [IF NOT EXISTS] [schema_name.]view_name [(column_name, ...)]
AS select_stmt;

DROP VIEW [IF EXISTS] [schema_name.]view_name;
```

* 视图不是表，SQLite 只在 `sqlite_schema` 中保存这条 `SELECT` 的原文，因此<u>建视图几乎不占存储空间</u>，查询视图等价于把视图定义内联进查询语句执行；  
* 可以给视图指定列名清单；若不指定则沿用 `SELECT` 结果集的列名，此时<u>结果集中出现重名列会导致建视图失败</u>，应改用列名清单或 `AS` 别名；  
* 视图定义中可以包含 `WHERE`、`JOIN`、`GROUP BY`、子查询甚至引用其它视图，也可以加 `TEMP` 关键字建到 `temp` 库中（仅当前连接有效）；视图名与表名共享同一命名空间，同一个 schema 内不能重名；  
* 视图是**只读**的：<u>直接对普通视图执行 `INSERT`、`UPDATE`、`DELETE` 会报错</u>，确实需要"通过视图写数据"时必须在该视图上创建 `INSTEAD OF` 触发器（见触发器小节）；  
* 视图依赖的是**表结构**而非表数据：表被 `DROP TABLE` 删除后视图仍然存在，但此后查询该视图会报错；而 `ALTER TABLE` 的重命名表/重命名列会自动改写视图定义中对它们的引用；  

``` SQL
CREATE VIEW v_emp_dept AS
SELECT e.id, e.name, d.name AS dept_name, e.salary
FROM employees AS e
LEFT JOIN departments AS d ON e.dept_id = d.id;

SELECT dept_name, COUNT(*) FROM v_emp_dept GROUP BY dept_name;  -- 像查表一样查视图
PRAGMA table_info(v_emp_dept);                                  -- 查看视图的列定义

DROP VIEW v_emp_dept;
```

> 视图只是"保存好的查询"，<u>既不缓存结果、也不能建索引</u>，所以它不会让查询变快；某条视图查询慢，应优化视图内部的写法或为底层表建索引。需要把结果"物化"存下来时应改用 `CREATE TABLE ... AS SELECT`，代价是数据不再随源表变化。
> 
> 视图可以嵌套，但层数过多会让查询计划难以优化、排查问题时也难以看清最终执行的 SQL，实践中建议控制在两三层以内。

## 索引 Index
**索引 index** 是建立在表的一列或多列之上的辅助数据结构（B-Tree），用于把"全表扫描"变成"定点查找"，同时也承担 `UNIQUE` / `PRIMARY KEY` 约束的唯一性检查。其基本语法如下：  

``` SQL
CREATE [UNIQUE] INDEX [IF NOT EXISTS] [schema_name.]index_name
ON table_name (indexed_column [, indexed_column, ...]);

DROP INDEX [IF EXISTS] [schema_name.]index_name;

-- indexed_column 可以为下列形式之一
column_name [COLLATE collation_name] [ASC | DESC]
expression  [COLLATE collation_name] [ASC | DESC]
```

* 索引与表一样属于模式对象，<u>索引名在同一个数据库内必须唯一</u>（不是"每张表内唯一"）；索引只能建在表上，不能建在视图上；  
* `UNIQUE` 索引要求索引列的组合在表内唯一，重复插入会失败；而<u>`NULL` 被视为互不相同</u>，所以多行可以同时取 `NULL`；  
* 索引列除列名外还可以是**表达式**（3.9.0 起），此时查询中必须出现<u>与索引定义完全一致的表达式</u>才能命中，例如 `lower(name)`；  
* **部分索引 partial index**（3.8.0 起）带 `WHERE` 子句，只为满足条件的行建索引，既省空间，又能让查询优化器确认"目标行必然落在索引覆盖范围内"；  
* 索引可以服务于 `WHERE` 的等值与范围条件、`JOIN ... ON` 的连接列、`ORDER BY` 与 `GROUP BY` 的排序需求；<u>列上声明了 `COLLATE` 时，查询与索引必须使用同一种排序规则才能匹配</u>；  
* 索引是**有代价**的：占磁盘空间，且每次 `INSERT`、`UPDATE`、`DELETE` 都要同步维护，<u>不要盲目给每一列都建索引</u>，选择性低（重复值多）的列上建索引往往得不偿失；  

``` SQL
CREATE INDEX idx_emp_dept        ON employees (dept_id);                -- 普通索引：加速过滤与连接
CREATE UNIQUE INDEX idx_users_email ON users (email COLLATE NOCASE);    -- 唯一索引：忽略大小写去重
CREATE INDEX idx_emp_lower_name  ON employees (lower(name));            -- 表达式索引
CREATE INDEX idx_users_active    ON users (email) WHERE active = 1;     -- 部分索引：只覆盖活跃用户

EXPLAIN QUERY PLAN SELECT * FROM employees WHERE dept_id = 3;           -- 查看是否命中索引
```

`EXPLAIN QUERY PLAN` 输出中的 `SEARCH ... USING INDEX 索引名` 表示用到了索引，`SCAN 表名` 则表示全表扫描。查看与维护索引常用的语句如下：  

``` SQL
PRAGMA index_list(employees);        -- 表上的全部索引（含 sqlite_autoindex_*）
PRAGMA index_info(idx_emp_dept);     -- 某个索引包含哪些列
PRAGMA index_xinfo(idx_emp_dept);    -- 附带上排序方向、是否随 rowid 等细节
REINDEX;                             -- 重建当前数据库的全部索引（也可写 REINDEX 索引名）
ANALYZE;                             -- 收集统计信息写入 sqlite_stat1，供查询优化器参考
```

> 当查询没有可用索引时，SQLite 可能为当前语句临时建立**自动索引 automatic index**（执行计划中显示 `USING AUTOMATIC INDEX`），它只在本次语句内有效——<u>看到自动索引通常意味着该查询缺少一个持久索引</u>。另外可用 `INDEXED BY 索引名` 强制指定索引、用 `NOT INDEXED` 禁用索引，一般只用于排查问题，不建议写进业务代码。

## 触发器 Trigger
**触发器 trigger** 是绑定在表（或视图）上的、由 `INSERT`、`UPDATE`、`DELETE` 语句自动触发的动作，它把"数据变更时应当顺带完成的另一件事"交给数据库自己保证，而不是依赖应用代码记得去做。其基本语法如下：  

``` SQL
CREATE [TEMP | TEMPORARY] TRIGGER [IF NOT EXISTS] [schema_name.]trigger_name
[BEFORE | AFTER | INSTEAD OF] {DELETE | INSERT | UPDATE [OF column_name, ...]}
ON table_name
[FOR EACH ROW]
[WHEN expr]
BEGIN
    statement;
    ...
END;

DROP TRIGGER [IF EXISTS] [schema_name.]trigger_name;
```

* SQLite <u>只支持行级触发器，没有语句级触发器</u>：`FOR EACH ROW` 可以省略，但语义上始终是对每一行触发一次；`BEGIN ... END` 是语法必需部分，哪怕触发体只有一条语句也要写，语句之间用 `;` 分隔；  
* 触发时机上，`BEFORE` / `AFTER` 用于普通表，<u>`INSTEAD OF` 只能用于视图</u>，它是实现"可写视图"的唯一手段；  
* 触发体中用 `NEW.列名` 引用**新行**、`OLD.列名` 引用**旧行**：`INSERT` 只有 `NEW`，`DELETE` 只有 `OLD`，`UPDATE` 两者都有。<u>在 `BEFORE` 触发器中对 `NEW.列名` 赋值可以改写即将写入的值</u>，而 `NEW` 在 `AFTER` 触发器中已不可改；  
* `WHEN 表达式` 用于附加触发条件，`UPDATE OF 列名` 用于限定只有指定列被更新时才触发；  
* 触发体中可执行 `INSERT`、`UPDATE`、`DELETE`、`SELECT`，因此一个触发器可能引发另一个触发器，形成连锁甚至递归；<u>递归触发器默认关闭，由 `PRAGMA recursive_triggers` 控制</u>；  

``` SQL
CREATE TRIGGER trg_users_email_lower            -- 插入后把邮箱统一转成小写
AFTER INSERT ON users
FOR EACH ROW
WHEN NEW.email <> lower(NEW.email)
BEGIN
    UPDATE users SET email = lower(NEW.email) WHERE id = NEW.id;
END;

CREATE TRIGGER trg_employees_check_salary       -- BEFORE 中校验待写入的值
BEFORE UPDATE OF salary ON employees
FOR EACH ROW
WHEN NEW.salary < 0
BEGIN
    SELECT RAISE(ABORT, 'salary must not be negative');
END;

CREATE TRIGGER trg_view_insert                  -- INSTEAD OF：让视图可写
INSTEAD OF INSERT ON v_emp_dept
BEGIN
    INSERT INTO employees (id, name, dept_id) VALUES (NEW.id, NEW.name, NEW.dept_id);
END;

DROP TRIGGER trg_users_email_lower;
```

> 触发器的副作用是"隐式"的，调试与维护成本较高，<u>能用约束（`CHECK`、`UNIQUE`、`FOREIGN KEY`）或应用层逻辑表达清楚的规则，就不要写成触发器</u>。另外需注意与 `删除 DELETE` 小节的呼应：当清空整表走了**截断优化**时，`DELETE` 触发器不会被触发，行为与逐行删除不一致。

## 事务 Transaction
**事务 transaction** 是一组要么全部生效、要么全部不生效的数据库操作，它既是 SQLite 满足 ACID 的基础，也是多步写入"不能出现中间状态"时唯一的正确做法。控制事务边界的语句（TCL）如下：  

``` SQL
BEGIN [DEFERRED | IMMEDIATE | EXCLUSIVE] [TRANSACTION];
COMMIT [TRANSACTION];                                   -- END 是 COMMIT 的同义词
ROLLBACK [TRANSACTION];

SAVEPOINT savepoint_name;                               -- 设置保存点
RELEASE [SAVEPOINT] savepoint_name;                     -- 释放保存点（最外层保存点等同提交）
ROLLBACK [TRANSACTION] TO [SAVEPOINT] savepoint_name;   -- 回滚到保存点，事务仍然继续
```

* <u>单条 SQL 语句本身就是一个隐式事务</u>：SQLite 会自动为它开启事务、执行并提交（失败则回滚）；只有用 `BEGIN` 显式开启之后，事务边界才由你掌控，可以有条件地 `COMMIT` 或 `ROLLBACK`；  
* 三种 `BEGIN` 的区别在于**何时获取锁**：`DEFERRED`（默认）直到第一次读或写才加锁，`IMMEDIATE` 一开始就获取写锁，`EXCLUSIVE` 直接获取排他锁（在 WAL 模式下与 `IMMEDIATE` 基本等价）。<u>若事务中会先读后写，用 `BEGIN IMMEDIATE` 可避免"锁升级"失败导致的 `SQLITE_BUSY`</u>；  
* `SAVEPOINT` 支持嵌套与部分回滚：`ROLLBACK TO` 只撤销到该保存点之后的修改，事务本身仍然有效，可以继续操作后统一 `COMMIT`；  
* 事务中某条语句出错时，默认只撤销该语句自身的修改，<u>不会自动回滚整个事务</u>；要整体回滚需显式 `ROLLBACK`，或把约束的冲突算法设为 `ON CONFLICT ROLLBACK`；  
* `PRAGMA foreign_keys` 属于<u>不能在事务中修改</u>的开关（此时它是空操作），因此迁移脚本必须先关外键、再 `BEGIN`，与 `重建表` 小节的顺序一致；  

``` SQL
BEGIN IMMEDIATE;                                       -- 转账：要么都成功，要么都不发生
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;

BEGIN;                                                 -- 保存点：只撤销一半的操作
INSERT INTO logs (msg) VALUES ('step 1');
SAVEPOINT sp1;
UPDATE counters SET n = n + 1;
ROLLBACK TO sp1;                                       -- 撤销 UPDATE，前面的 INSERT 留在事务中
RELEASE sp1;
COMMIT;
```

### 日志模式与并发控制
SQLite 的所有修改都依赖**回滚日志 rollback journal**或**预写日志 write-ahead log**来保证原子性与持久性，由 `PRAGMA journal_mode` 选择，可选值如下表：  

| <font size=2>日志模式</font> | <font size=2>说明</font> |
| :---- | :---- |
| <font size=2>DELETE（默认）</font> | <font size=2>回滚日志在提交后被删除，兼容性最好，但读写互相阻塞</font> |
| <font size=2>TRUNCATE</font> | <font size=2>与 DELETE 类似，但把日志文件截断为 0 字节而不删除</font> |
| <font size=2>PERSIST</font> | <font size=2>与 DELETE 类似，但保留日志文件（只把头标记为无效），避免反复创建文件</font> |
| <font size=2>MEMORY</font> | <font size=2>回滚日志放在内存中，速度快，断电时无法恢复</font> |
| <font size=2>WAL</font> | <font size=2>预写日志，修改先追加到 `-wal` 文件，读操作可同时进行</font> |
| <font size=2>OFF</font> | <font size=2>关闭日志，不做原子性与持久性保证，仅用于可随时重建的临时数据</font> |

* <u>WAL 是 SQLite 并发能力最好的一种模式</u>：同一时刻可以有多个读者与一个写者并存，读者看到的是事务开始时的**快照**，因此"读写不互相阻塞"；但它会额外产生 `-wal` 与 `-shm` 两个文件，且<u>WAL 数据库只能被同一台主机上的进程访问</u>，不能放在网络文件系统上；  
* `PRAGMA synchronous` 控制每次提交的"落盘"程度：`FULL`（默认）最安全，`NORMAL` 在 WAL 模式下兼顾安全与性能（崩溃可能丢失最近若干次提交，但不会损坏数据库文件），`EXTRA` 更严格但仅适用于回滚日志模式；  
* SQLite 的锁是**数据库级**而非行级的：写操作会锁住整个数据库文件，因此多进程写入时应设置 `PRAGMA busy_timeout = 5000;`，让遇到 `SQLITE_BUSY` 的连接自动等待重试而不是立刻报错；  
* SQLite 的事务隔离级别是**串行化 serializable**，不会出现脏读、不可重复读与幻读；四个 ACID 特性与日志模式的关系可以概括为：<u>原子性与一致性由日志保证，隔离性由锁与快照保证，持久性由 `synchronous` 与文件系统落盘保证</u>；  

``` SQL
PRAGMA journal_mode = WAL;          -- 切换日志模式，返回切换后的模式名
PRAGMA synchronous  = NORMAL;       -- WAL 下的常用组合
PRAGMA busy_timeout = 5000;         -- 遇到锁时等待 5 秒再报错
PRAGMA wal_checkpoint(TRUNCATE);    -- 把 WAL 内容合并回主库并截断 wal 文件
```

## 窗口函数与公共表表达式（CTE）
这两个特性都能让复杂查询更清晰：**公共表表达式 CTE** 解决"子查询嵌套太深、同一段逻辑要写多遍"的问题，**窗口函数 window function** 解决"既要保留明细行、又要同时算出排名/占比/环比"的问题。

### CTE（WITH）
**公共表表达式 common table expression（CTE）** 用一个名字把子查询"提"到主查询之前，使其在同一条语句中可以被引用（并可多次引用）。该特性从 SQLite 3.8.3（2014-02-03）开始支持，语法如下：  

``` SQL
WITH cte_name [(column_name, ...)] AS ( select_stmt )
     [, cte_name2 AS ( select_stmt2 ) ...]
select_stmt;                                            -- 主查询

WITH RECURSIVE cte_name AS ( 初始查询 UNION ALL 递归查询 ) ...   -- 递归形式
```

* CTE 的作用域<u>仅限紧跟其后的那一条语句</u>，它与视图的区别正在于此：视图是持久化的模式对象，CTE 只是"这条 SQL 里的临时命名子查询"；  
* 可以一次定义多个 CTE，后定义的可以引用先定义的；用 CTE 替代多层嵌套子查询，可以让阅读顺序变成"从上到下"；  
* `WITH RECURSIVE` 用于表达递归：把 CTE 拆成**初始查询**（不引用自身）与**递归查询**（引用自身）两部分并用 `UNION ALL` 连接，常用于生成序列、遍历组织树/分类树；<u>递归必须有终止条件，否则会无限递归下去</u>；  

``` SQL
-- 生成 1~10 的序列
WITH RECURSIVE seq(n) AS (
    SELECT 1                                              -- 初始行
    UNION ALL
    SELECT n + 1 FROM seq WHERE n < 10                    -- 递归行，WHERE 是终止条件
)
SELECT group_concat(n) FROM seq;

-- 遍历部门树（id, parent_id 邻接表）
WITH RECURSIVE tree(id, name, depth) AS (
    SELECT id, name, 0 FROM departments WHERE parent_id IS NULL
    UNION ALL
    SELECT d.id, d.name, t.depth + 1
    FROM departments AS d JOIN tree AS t ON d.parent_id = t.id
)
SELECT depth, name FROM tree ORDER BY depth;
```

### 窗口函数 window function
**窗口函数**从 SQLite 3.25.0（2018-09-15）开始支持，它在一组相关的行（称为**窗口 window**）上计算出一个值并附加到<u>每一行</u>上，语法如下：  

``` SQL
function_name(expr_list) OVER (
    [PARTITION BY expr_list]                -- 把结果集划分为若干窗口
    [ORDER BY expr_list [ASC | DESC]]       -- 窗口内排序
    [ROWS | RANGE frame_spec]               -- 窗口帧，默认值随是否写 ORDER BY 而不同
);
```

* 内置窗口函数可分为三类：**排名类** `ROW_NUMBER()`、`RANK()`、`DENSE_RANK()`、`NTILE(n)`，**取值类** `LAG()`、`LEAD()`、`FIRST_VALUE()`、`LAST_VALUE()`、`NTH_VALUE()`，以及<u>把普通聚合函数当作窗口函数使用</u>（如 `SUM(x) OVER (...)`、`AVG(x) OVER (...)`）；  
* `PARTITION BY` 相当于"分组但不折叠行"，`ORDER BY` 决定窗口内的顺序；只写 `OVER ()` 时整个结果集就是同一个窗口；  
* 窗口帧用 `ROWS BETWEEN ... AND ...` 指定参与计算的行范围，如 `ROWS BETWEEN 2 PRECEDING AND CURRENT ROW` 表示近三行；<u>省略时，写了 `ORDER BY` 的默认帧是"从窗口起点到当前行"，没写 `ORDER BY` 时默认覆盖整个窗口</u>；  
* 与 `GROUP BY` 的关键区别是：<u>聚合会"折叠行"，窗口函数不会——明细行全部保留</u>，因此可以在同一行里同时看到明细值、组内排名与组内合计；  
* 窗口函数<u>只能出现在 `SELECT` 列表与 `ORDER BY` 中</u>，不能直接写在 `WHERE`、`GROUP BY`、`HAVING` 里；若需要按其结果过滤，要把整个查询包成子查询或 CTE；  
* 从 3.28.0 起可以把窗口定义提取为命名窗口，在主查询末尾用 `WINDOW w AS (...)` 声明后反复引用；  

``` SQL
SELECT name, dept_id, salary,
       ROW_NUMBER() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rn,    -- 组内排名
       RANK()       OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rk,    -- 并列会跳号
       salary * 1.0 / SUM(salary) OVER (PARTITION BY dept_id)        AS ratio, -- 组内占比
       AVG(salary)  OVER (PARTITION BY dept_id ORDER BY hire_date
                          ROWS BETWEEN 2 PRECEDING AND CURRENT ROW)  AS ma3    -- 近三人均值
FROM employees;
```

> 若只想取"每个部门工资最高的那个人"，窗口函数需要包一层子查询（`SELECT * FROM (...) WHERE rn = 1`）。SQLite 也支持 `SELECT max(salary), name FROM ... GROUP BY dept_id` 这种"裸列"写法，虽然更短但不可移植，可移植的正式写法仍是窗口函数或相关子查询。

## Vacuum
`VACUUM` 是 SQLite 的空间整理命令：它<u>按当前表结构重新生成一份数据库文件，把空闲页与碎片回收掉</u>，是前面 `DROP TABLE`、`DELETE` 小节中"空间被放回空闲列表、文件不会自动缩小"的最终解法。其语法如下：  

``` SQL
VACUUM;                            -- 整理当前连接的 main 数据库
VACUUM schema_name;                -- 整理指定数据库，如 VACUUM temp;
VACUUM INTO 'file_name';           -- 把整理后的副本导出为新文件（3.27.0，2019-02-07 起）
```

* `DROP TABLE` 与 `DELETE` 只是把页放回**空闲列表 free list**，文件大小不变；执行 `VACUUM` 才会真正缩小文件、减少碎片，也让后续扫描更快；  
* `VACUUM INTO` 把数据库整理后写出到另一个文件，<u>目标文件必须不存在</u>，它既是"整理成副本"的手段，也是一种不需要停止写入的**在线备份**方式；  
* 代价是需要约等于数据库大小的临时空间并做全量读写，<u>对大库执行会长时间占用 IO</u>，且不能在事务中执行；  
* `VACUUM` 会重建表，因此<u>没有显式 `INTEGER PRIMARY KEY` 的表中，rowid 可能发生变化</u>；同时 `sqlite_sequence` 中记录的 AUTOINCREMENT 计数会被保留，不会让"已用过的 id"重新可用；  
* 若希望 SQLite 平时就回收空间，可把 `auto_vacuum` 设为 `FULL` 或 `INCREMENTAL`（前者在每次提交时尝试搬移页，后者需手动调用 `PRAGMA incremental_vacuum;`）；<u>`auto_vacuum` 的修改需配合一次 `VACUUM` 才真正生效</u>；  

``` SQL
PRAGMA freelist_count;              -- 空闲页数量，大于 0 说明有空间可回收
VACUUM;                             -- 重建文件，回收全部空闲页
VACUUM INTO 'backup_20260914.db';   -- 导出一份整理后的副本

PRAGMA auto_vacuum = INCREMENTAL;   -- 设置自动清理模式（需 VACUUM 生效）
PRAGMA incremental_vacuum(100);     -- 增量回收，最多处理 100 页
PRAGMA wal_checkpoint(TRUNCATE);    -- WAL 模式：合并回主库并清空 wal 文件
```

## PRAGMA
`PRAGMA` 是 SQLite 特有的语句，用来**查询或修改引擎自身的参数**，功能上等价于 C 接口中的 `sqlite3_...` 配置与查询函数；它既不是 DDL 也不是 DML，而是"引擎的控制面"。语法有查询与设置两种形式：  

``` SQL
PRAGMA name;                       -- 查询当前值
PRAGMA name = value;               -- 设置
PRAGMA name(value);                -- 带参数调用，多为查询类，如 PRAGMA table_info(employees);
PRAGMA schema_name.name;           -- 指定数据库，如 PRAGMA main.journal_mode;
```

按用途，常用的 `PRAGMA` 可分为下面几类：  

| <font size=2>类别</font> | <font size=2>常用 PRAGMA</font> | <font size=2>说明</font> |
| :---- | :---- | :---- |
| <font size=2>连接与文件</font> | <font size=2>`journal_mode`、`synchronous`、`foreign_keys`、`busy_timeout`、`cache_size`、`temp_store`</font> | <font size=2>影响事务与并发行为，详见事务小节</font> |
| <font size=2>结构查询</font> | <font size=2>`table_info`、`table_xinfo`、`index_list`、`index_info`、`foreign_key_list`、`database_list`、`collation_list`</font> | <font size=2>查看表、索引、外键、排序规则等模式信息</font> |
| <font size=2>完整性校验</font> | <font size=2>`integrity_check`、`quick_check`、`foreign_key_check`</font> | <font size=2>校验物理与逻辑一致性，迁移后必跑</font> |
| <font size=2>版本与标识</font> | <font size=2>`schema_version`、`user_version`、`application_id`</font> | <font size=2>`user_version` 与应用自定义整数常用于记录数据库版本</font> |
| <font size=2>空间与统计</font> | <font size=2>`page_size`、`page_count`、`freelist_count`、`optimize`、`analysis_limit`</font> | <font size=2>查看并优化存储与查询计划统计</font> |
| <font size=2>环境信息</font> | <font size=2>`compile_options`、`function_list`、`module_list`</font> | <font size=2>查看编译选项、可用函数与虚拟表模块</font> |

* <u>大多数 `PRAGMA` 是连接级的，只对当前连接生效</u>（如 `foreign_keys`、`busy_timeout`、`cache_size`），换一个连接就要重新设置；而 `journal_mode`、`auto_vacuum`、`page_size`、`user_version` 这类会<u>写进数据库文件头</u>，是持久化的；  
* 设置类 PRAGMA 大多在语句执行后立即生效且不返回错误码，例如<u>`PRAGMA foreign_keys = ON;` 在事务中会被静默忽略</u>；  
* 查询类 PRAGMA 以结果集形式返回，可以当普通查询使用（例如把 `table_info` 的结果与其它表 `JOIN`）；  
* `PRAGMA optimize;` 会让查询优化器按需更新统计信息，适合在应用关闭数据库前执行一次；  
* 前缀 `schema_name.` 可限定作用的数据库，例如 `PRAGMA main.journal_mode;` 与 `PRAGMA temp.store;`；  

``` SQL
PRAGMA compile_options;                 -- 这个库编译时启用了哪些特性
PRAGMA function_list;                   -- 当前可用的全部函数（含聚合与窗口函数）
PRAGMA table_info(employees);           -- 列名、类型、NOT NULL、默认值、是否主键
PRAGMA index_list(employees);           -- 表上的索引
PRAGMA integrity_check;                 -- 校验整个数据库，返回 ok 表示正常
PRAGMA foreign_key_check;               -- 校验外键引用是否都存在
PRAGMA user_version = 3;                -- 记录应用的 schema 版本号
```

## 内置函数
SQLite 内置了一批函数，可分为**标量函数 scalar function**（每行返回一个值）、**聚合函数 aggregate function**（一组行返回一个值，加上 `OVER` 就是窗口函数），以及日期时间、JSON 等专用函数族。常用函数按用途归类如下：  

| <font size=2>类别</font> | <font size=2>函数</font> |
| :---- | :---- |
| <font size=2>数值</font> | <font size=2>`abs()`、`round()`、`sign()`、`max()` / `min()`（多参数形式）、`random()`、`randomblob()`、`hex()`、`unhex()`；数学函数 `pow()`、`sqrt()`、`exp()`、`ln()`、`log()`、`floor()`、`ceil()`、`trunc()`（3.35.0 起，需启用 `SQLITE_ENABLE_MATH_FUNCTIONS`）</font> |
| <font size=2>文本</font> | <font size=2>`length()`、`octet_length()`、`upper()`、`lower()`、`trim()` / `ltrim()` / `rtrim()`、`substr()` / `substring()`、`replace()`、`instr()`、`printf()` / `format()`、`quote()`、`char()`、`unicode()`、`concat()` / `concat_ws()`（3.44.0 起）</font> |
| <font size=2>类型与空值</font> | <font size=2>`typeof()`、`CAST`、`ifnull()`、`coalesce()`、`nullif()`、`iif()`（3.32.0 起）、`likely()` / `unlikely()`</font> |
| <font size=2>日期时间</font> | <font size=2>`date()`、`time()`、`datetime()`、`julianday()`、`unixepoch()`、`strftime()`、`timediff()`（3.43.0 起）</font> |
| <font size=2>聚合</font> | <font size=2>`count()`、`sum()`、`total()`、`avg()`、`max()` / `min()`、`group_concat()` / `string_agg()`（3.44.0 起）</font> |
| <font size=2>JSON</font> | <font size=2>`json()`、`json_extract()`、`json_array()`、`json_object()`、`json_set()`、`json_patch()`、`json_type()`，以及 `->`、`->>` 运算符（3.38.0 起内置）</font> |
| <font size=2>元信息</font> | <font size=2>`sqlite_version()`、`sqlite_source_id()`、`last_insert_rowid()`、`changes()`、`total_changes()`</font> |

* `date()` / `datetime()` / `strftime()` 系列既可以格式化也可以做时间计算，常用修饰符有 `'now'`、`'localtime'`、`'-30 days'`、`'start of month'` 等，例如 `date('now', 'start of month', '+1 month')`；<u>`CURRENT_TIMESTAMP` 与 `datetime('now')` 取到的都是 UTC 时间</u>，需要本地时间要显式加上 `'localtime'`，与 `DEFAULT 子句` 小节的说明一致；  
* `ifnull(x, y)` 只能接受两个参数（等价于 `COALESCE(x, y)`），`coalesce()` 可以给一长串候选值；`nullif(a, b)` 在两者相等时返回 `NULL`，常用于避免除零或过滤空串；  
* `iif(condition, true_value, false_value)` 是 `CASE WHEN condition THEN true_value ELSE false_value END` 的简写；  
* 聚合函数可以搭配 `FILTER (WHERE ...)` 只对满足条件的行聚合，例如 `sum(amount) FILTER (WHERE status = 'paid')`（3.30.0 起），比在 `CASE` 里绕一圈更直观；  
* 聚合函数加上 `OVER (...)` 之后即成为窗口函数，用法见窗口函数小节；<u>SQLite 不支持用 SQL 自定义函数</u>，新增函数只能通过 C 接口 `sqlite3_create_function()` 注册（各语言绑定通常都封装了对应的注册 API）；  

``` SQL
SELECT round(3.4567, 2), abs(-5), hex(randomblob(4));                        -- 数值
SELECT upper(name), substr(email, instr(email, '@') + 1), length(name);      -- 文本
SELECT iif(salary >= 10000, '高', '其它'), coalesce(phone, '未登记');         -- 条件与空值
SELECT date('now'), datetime('now', 'localtime'), strftime('%Y-%m', created); -- 日期时间
SELECT dept_id,
       sum(amount) FILTER (WHERE status = 'paid') AS paid_sum,               -- 3.30.0 起的 FILTER
       group_concat(name, '/') AS names
FROM orders JOIN employees USING (id) GROUP BY dept_id;
SELECT json_extract(payload, '$.user.id'), payload -> '$.tags' FROM events;  -- JSON
```

> 使用函数时需特别注意**版本与编译开关**：JSON 函数在 3.38.0（2022-02-22）之前是名为 JSON1 的可选扩展，数学函数依赖 `SQLITE_ENABLE_MATH_FUNCTIONS` 编译选项。由于 SQLite 通常随语言运行时捆绑分发，<u>同一个函数在不同环境中未必可用</u>，动手前最好用 `SELECT sqlite_version();` 与 `PRAGMA function_list;` 确认一下。

# 在程序中调用
前面几章都在 SQL 语句这一层展开，而 SQLite 的定位是**嵌入式数据库 embedded database**：它最终总是以库的形式被链接进宿主程序，由程序通过 API 来驱动。本章先介绍官方的 **C 接口**（`sqlite3` 库，它既供 C/C++ 直接调用，也是所有第三方语言绑定的共同底座），再介绍 .NET 侧的 **C# API**。

## C/C++ API
SQLite 对外暴露的是一套 **C 语言 API**（头文件 `sqlite3.h`），C++ 可以直接调用。整个接口围绕两个核心对象展开：**数据库连接 `sqlite3`** 与**预编译语句 `sqlite3_stmt`**，绝大多数操作都遵循同一套固定流程：  
①`sqlite3_open_v2()` 打开数据库文件（不存在则创建），得到连接对象；  
②`sqlite3_prepare_v2()` 把一条带占位符的 SQL 文本**编译**为预编译语句，编译一次即可反复执行，这也是防注入的根本手段；  
③`sqlite3_bind_*()` 为语句中的占位符绑定具体值，<u>占位符索引从 1 开始</u>；  
④`sqlite3_step()` 执行语句：查询返回 `SQLITE_ROW` 表示读到一行，返回 `SQLITE_DONE` 表示执行结束，增删改则直接返回 `SQLITE_DONE`；  
⑤`sqlite3_column_*()` 在当前行上按列序号（<u>从 0 开始</u>）读取各列的值；  
⑥`sqlite3_finalize()` 释放语句对象，之后该 `sqlite3_stmt` 不可再使用；  
⑦`sqlite3_close()`（或 `sqlite3_close_v2()`）关闭连接并释放资源。  

### 常用接口一览
| <font size=2>阶段</font> | <font size=2>常用函数</font> | <font size=2>说明</font> |
| :---- | :---- | :---- |
| <font size=2>打开与关闭</font> | <font size=2>`sqlite3_open_v2()`、`sqlite3_close()`、`sqlite3_close_v2()`</font> | <font size=2>能用 `_v2` 就不要用旧版 `sqlite3_open()`，旧接口的错误语义已被废弃</font> |
| <font size=2>错误处理</font> | <font size=2>`sqlite3_errmsg()`、`sqlite3_errcode()`、`sqlite3_extended_errcode()`、`sqlite3_extended_result_codes()`</font> | <font size=2>返回码为整数，`errmsg` 给出可读描述；打开扩展错误码可区分 `SQLITE_CONSTRAINT_NOTNULL` 等细分原因</font> |
| <font size=2>一次性执行</font> | <font size=2>`sqlite3_exec()`</font> | <font size=2>回调式便捷接口，适合 DDL 或无结果语句；<u>它无法绑定参数，只应用于可信任的常量 SQL</u></font> |
| <font size=2>编译与执行</font> | <font size=2>`sqlite3_prepare_v2()`、`sqlite3_step()`、`sqlite3_reset()`、`sqlite3_finalize()`</font> | <font size=2>`reset` 可复用语句（重置到编译后状态），`finalize` 才是彻底释放</font> |
| <font size=2>绑定参数</font> | <font size=2>`sqlite3_bind_int()`、`sqlite3_bind_int64()`、`sqlite3_bind_double()`、`sqlite3_bind_text()`、`sqlite3_bind_blob()`、`sqlite3_bind_null()`、`sqlite3_bind_parameter_index()`</font> | <font size=2>占位符写作 `?`、`?NNN`、`:name`、`@name` 或 `$name`</font> |
| <font size=2>读取列</font> | <font size=2>`sqlite3_column_int()`、`sqlite3_column_double()`、`sqlite3_column_text()`、`sqlite3_column_blob()`、`sqlite3_column_bytes()`、`sqlite3_column_type()`、`sqlite3_column_count()`</font> | <font size=2>`column_type()` 返回某个 `SQLITE_INTEGER`/`TEXT`/`BLOB` 之类的存储类，对应 `数据类型` 一节的五种存储类</font> |
| <font size=2>结果与统计</font> | <font size=2>`sqlite3_last_insert_rowid()`、`sqlite3_changes()`、`sqlite3_total_changes()`</font> | <font size=2>取最近一次插入自动分配的 rowid、本次/累计受影响行数</font> |
| <font size=2>连接配置</font> | <font size=2>`sqlite3_busy_timeout()`、`sqlite3_limit()`、`sqlite3_config()`、`sqlite3_db_config()`</font> | <font size=2>分别对应 `PRAGMA busy_timeout`、各类编译时限值的运行期调整等</font> |
| <font size=2>扩展能力</font> | <font size=2>`sqlite3_create_function()`、`sqlite3_create_collation()`、`sqlite3_create_module()`、`sqlite3_load_extension()`</font> | <font size=2>注册标量/聚合函数、自定义排序规则、虚拟表模块</font> |
| <font size=2>备份</font> | <font size=2>`sqlite3_backup_init()`、`sqlite3_backup_step()`、`sqlite3_backup_finish()`</font> | <font size=2>在线备份到另一个连接</font> |

要点补充：  
* 打开时用标志组合控制行为，如 `SQLITE_OPEN_READWRITE | SQLITE_OPEN_CREATE` 允许读写并在文件不存在时创建，再加 `SQLITE_OPEN_URI` 后文件名可以写成 `file:test.db?mode=memory&cache=shared` 这样的 URI 形式；  
* <u>每个返回值都要检查</u>：`SQLITE_OK`、`SQLITE_ROW`、`SQLITE_DONE` 是正常码，`SQLITE_BUSY`（被占用）、`SQLITE_CONSTRAINT`（违反约束）、`SQLITE_CORRUPT` 等才是错误码；  
* 绑定文本/二进制时最后一个参数决定内存所有权：`SQLITE_STATIC` 表示调用方保证数据在语句执行期间有效（零拷贝，性能好但易踩坑），`SQLITE_TRANSIENT` 表示让 SQLite 立刻复制一份（安全但多一次拷贝），<u>不确定时一律用 `SQLITE_TRANSIENT`</u>；  
* `sqlite3_column_text()` 返回的指针<u>在下一次 `sqlite3_step()`、`sqlite3_reset()` 或 `sqlite3_finalize()` 之后即失效</u>，需要留存就必须自己复制；BLOB 的长度用 `sqlite3_column_bytes()` 获取，其中可能含有 `\0`，不能当字符串处理；  
* 每条 `prepare` 出来的语句都必须 `finalize`；若关闭连接时仍有未释放的语句，`sqlite3_close()` 会返回 `SQLITE_BUSY` 而不真正关闭，用 `sqlite3_close_v2()` 则可以延迟到语句释放后再关闭；  
* 一条语句可以反复"绑定 → step → `sqlite3_reset()`"，这在循环写入时能省下重复编译的开销；`sqlite3_clear_bindings()` 可把参数重置为 `NULL`；  
* 线程模型上，默认编译（`SQLITE_THREADSAFE=1`，串行化模式）下一个连接同一时刻只应由一个线程使用；<u>不要把同一个连接交给多个线程并发使用</u>，并发应当通过多个连接 + `busy_timeout` + WAL 模式来解决；  
* 编译集成最省事的方式是使用官方提供的 **amalgamation 单文件**（`sqlite3.c` 与 `sqlite3.h`）直接编进项目，也可以链接系统或预编译的 `sqlite3` 库。  

### 完整示例
``` C++
#include <sqlite3.h>
#include <stdio.h>

int main(void) {
    sqlite3 *db = NULL;

    // ① 打开连接：读写 + 不存在则创建；失败时 db 可能非空，仍可用 errmsg 取错误信息
    if (sqlite3_open_v2("app.db", &db,
                        SQLITE_OPEN_READWRITE | SQLITE_OPEN_CREATE, NULL) != SQLITE_OK) {
        fprintf(stderr, "open failed: %s\n", sqlite3_errmsg(db));
        sqlite3_close(db);
        return 1;
    }
    sqlite3_busy_timeout(db, 5000);                                     // 等价于 PRAGMA busy_timeout = 5000
    sqlite3_exec(db, "PRAGMA foreign_keys = ON;", NULL, NULL, NULL);    // 外键要按连接开启

    // ② 无结果的 DDL 用 exec 最省事
    sqlite3_exec(db,
        "CREATE TABLE IF NOT EXISTS employees ("
        "  id      INTEGER PRIMARY KEY,"
        "  name    TEXT    NOT NULL,"
        "  dept_id INTEGER,"
        "  salary  REAL);",
        NULL, NULL, NULL);

    // ③④⑤ 插入：编译 → 绑定 → 执行。? 也可写成 :name / @name / $name
    const char *insert_sql =
        "INSERT INTO employees (name, dept_id, salary) VALUES (?, ?, ?);";
    sqlite3_stmt *stmt = NULL;
    if (sqlite3_prepare_v2(db, insert_sql, -1, &stmt, NULL) != SQLITE_OK) {
        fprintf(stderr, "prepare failed: %s\n", sqlite3_errmsg(db));
        sqlite3_close(db);
        return 1;
    }
    sqlite3_bind_text  (stmt, 1, "Alice", -1, SQLITE_TRANSIENT);   // 索引从 1 开始
    sqlite3_bind_int   (stmt, 2, 1);
    sqlite3_bind_double(stmt, 3, 12000.0);
    if (sqlite3_step(stmt) != SQLITE_DONE) {                       // 增删改期望 SQLITE_DONE
        fprintf(stderr, "step failed: %s\n", sqlite3_errmsg(db));
    }
    sqlite3_finalize(stmt);                                        // 释放语句
    printf("last insert rowid = %lld\n", sqlite3_last_insert_rowid(db));

    // ⑥ 查询：step 返回 SQLITE_ROW 时读取当前行，列序号从 0 开始
    sqlite3_prepare_v2(db, "SELECT id, name, salary FROM employees WHERE dept_id = ?;",
                       -1, &stmt, NULL);
    sqlite3_bind_int(stmt, 1, 1);
    while (sqlite3_step(stmt) == SQLITE_ROW) {
        int         id     = sqlite3_column_int(stmt, 0);
        const char *name   = (const char *)sqlite3_column_text(stmt, 1);  // 下次 step 后失效
        double      salary = sqlite3_column_double(stmt, 2);
        printf("%d\t%s\t%.2f\n", id, name, salary);
    }
    sqlite3_finalize(stmt);                                        // 语句可复用，但用完必须释放

    // ⑦ 事务：直接用 SQL 控制边界，任何一步失败都应回滚
    sqlite3_exec(db, "BEGIN IMMEDIATE;", NULL, NULL, NULL);
    if (sqlite3_exec(db, "UPDATE employees SET salary = salary * 1.1;", NULL, NULL, NULL)
        != SQLITE_OK) {
        sqlite3_exec(db, "ROLLBACK;", NULL, NULL, NULL);
    } else {
        sqlite3_exec(db, "COMMIT;", NULL, NULL, NULL);
    }

    sqlite3_close(db);      // 仍有未 finalize 的语句时会返回 SQLITE_BUSY
    return 0;
}
```

> C 接口是 SQLite 唯一的"原生"接口，其它语言的绑定都只是它的包装：因此这里的原则（预编译 + 绑定参数、检查返回码、及时 finalize）在各语言通用。<u>最容易被忽略的是 `finalize` 与返回码检查</u>，前者造成资源泄漏，后者会把"约束冲突""数据库被锁"等问题悄悄吞掉。

## C# API
.NET 上有两个成熟的 ADO.NET Provider 可以访问 SQLite：历史悠久的 **System.Data.SQLite** 与微软自家的 **Microsoft.Data.Sqlite**。选型建议先看下表，本章的 API 详解只针对 **Microsoft.Data.Sqlite**。

### 选型：Microsoft.Data.Sqlite 与 System.Data.SQLite

| <font size=2>对比项</font> | <font size=2>System.Data.SQLite</font> | <font size=2>Microsoft.Data.Sqlite</font> |
| :---- | :---- | :---- |
| <font size=2>维护方</font> | <font size=2>SQLite 官方开发团队（最早由 Robert Simpson 发起）</font> | <font size=2>微软 .NET 数据团队，与 EF Core 同源</font> |
| <font size=2>NuGet 包</font> | <font size=2>`System.Data.SQLite`（含原生库）、`System.Data.SQLite.Core`、`System.Data.SQLite.Linq`</font> | <font size=2>`Microsoft.Data.Sqlite`（含原生库）、`Microsoft.Data.Sqlite.Core`（仅托管层，需自行选择 bundle）</font> |
| <font size=2>原生库分发</font> | <font size=2>传统上是**混合模式程序集**或按平台/位数分发的 interop DLL，部署与升级时较易踩坑</font> | <font size=2>通过 SQLitePCLRaw bundle 自动携带对应平台的原生 SQLite，也可替换为 SQLCipher、系统 SQLite 等</font> |
| <font size=2>平台与框架</font> | <font size=2>以 .NET Framework 起家，.NET Core / .NET 5+ 支持较晚且配置繁琐</font> | <font size=2>面向 .NET Standard 2.0 与现代 .NET，跨平台、单文件发布、裁剪与 AOT 更友好</font> |
| <font size=2>API 覆盖面</font> | <font size=2>更全：`SQLiteDataAdapter`、`DataSet`/`DataTable`、`SQLiteCommandBuilder`、LINQ to SQLite、EF6 Provider、托管版虚拟表</font> | <font size=2>精简：只做 ADO.NET 核心（Connection/Command/Reader/Transaction），配合 EF Core 使用</font> |
| <font size=2>加密</font> | <font size=2>提供独立的 SEE / SQLCipher 发行版本</font> | <font size=2>默认 bundle 不含加密，需换成 SQLCipher 的 bundle 并使用 `Password`</font> |
| <font size=2>现状</font> | <font size=2>仍在维护，但生态重心已转移</font> | <font size=2>EF Core 官方 SQLite Provider 的底层实现，事实上的主流选择</font> |

* <u>新项目一律优先 Microsoft.Data.Sqlite</u>：跨平台开箱即用、部署简单、与 EF Core 无缝衔接，且更新节奏跟着 .NET 走；  
* 只有下列情况才考虑 System.Data.SQLite：需要 `DataAdapter`/`DataSet` 直接把结果灌进 `DataTable`、需要 **EF6**（不是 EF Core）的 SQLite Provider，或维护中的老 .NET Framework 项目已经在大量使用它；  
* 两者的核心用法（`CreateCommand()`、参数、`ExecuteReader()`、事务）高度相似，迁移成本主要在零散的高级特性与原生库分发方式上；  

### 安装与连接
安装 `Microsoft.Data.Sqlite` 即可，它默认依赖 `SQLitePCLRaw.bundle_e_sqlite3`，<u>原生库随包分发、无需手工安装 SQLite</u>：在项目目录执行 `dotnet add package Microsoft.Data.Sqlite`，或在 IDE 的 NuGet 管理器中安装。若想自己控制原生库（例如改用系统 SQLite 或 SQLCipher），则安装 `Microsoft.Data.Sqlite.Core` 并另行引入对应的 bundle。  

``` C#
using Microsoft.Data.Sqlite;

// Data Source 为必填项；":memory:" 表示内存数据库
using var conn = new SqliteConnection(
    "Data Source=app.db;Mode=ReadWriteCreate;Foreign Keys=True;Pooling=True");
conn.Open();

Console.WriteLine(conn.ServerVersion);   // 底层 SQLite 版本，如 3.45.0
Console.WriteLine(conn.DataSource);      // 实际打开的文件路径

using var cmd = conn.CreateCommand();
cmd.CommandText = "SELECT name FROM sqlite_schema WHERE type = 'table';";
```

连接字符串由一组 `关键字=值` 组成，常用关键字如下表：  

| <font size=2>关键字</font> | <font size=2>取值</font> | <font size=2>说明</font> |
| :---- | :---- | :---- |
| <font size=2>`Data Source`</font> | <font size=2>文件路径、`:memory:` 或 URI</font> | <font size=2>数据源，也可写作 `Filename`；URI 形式（`file:xxx?mode=memory&cache=shared`）可用于共享内存库等场景</font> |
| <font size=2>`Mode`</font> | <font size=2>`ReadWriteCreate`（默认）、`ReadWrite`、`ReadOnly`、`Memory`</font> | <font size=2>打开模式，对应 C 接口的 `SQLITE_OPEN_*` 标志</font> |
| <font size=2>`Cache`</font> | <font size=2>`Default`、`Shared`、`Private`</font> | <font size=2>页缓存是否跨连接共享，多个连接访问同一个内存库时必须用 `Shared`</font> |
| <font size=2>`Foreign Keys`</font> | <font size=2>`True` / `False`</font> | <font size=2>打开连接后自动执行 `PRAGMA foreign_keys = ON`，省去每次手动开启</font> |
| <font size=2>`Recursive Triggers`</font> | <font size=2>`True` / `False`</font> | <font size=2>等价于 `PRAGMA recursive_triggers`，即是否允许触发器递归</font> |
| <font size=2>`Default Timeout`</font> | <font size=2>秒数</font> | <font size=2>命令默认等待锁的时间，对应 `PRAGMA busy_timeout` 的行为</font> |
| <font size=2>`Pooling`</font> | <font size=2>`True`（默认）/ `False`</font> | <font size=2>是否启用连接池，即 `Close()` 后底层连接是否被复用</font> |
| <font size=2>`Password`</font> | <font size=2>密钥文本</font> | <font size=2>仅在替换为带加密的 bundle（如 SQLCipher）之后才可用</font> |

### 执行命令与参数化
``` C#
// 增删改：ExecuteNonQuery 返回受影响的行数
using (var cmd = conn.CreateCommand())
{
    cmd.CommandText = "INSERT INTO employees (name, dept_id, salary) VALUES ($name, $dept, $salary);";
    cmd.Parameters.AddWithValue("$name",   "Alice");
    cmd.Parameters.AddWithValue("$dept",   1);
    cmd.Parameters.AddWithValue("$salary", 12000.0);
    int rows = cmd.ExecuteNonQuery();          // 1
}

// 取单个值：ExecuteScalar 返回首行首列，无结果时为 null
using (var cmd = conn.CreateCommand())
{
    cmd.CommandText = "SELECT count(*) FROM employees;";
    long count = (long)cmd.ExecuteScalar()!;
}
```

* 占位符支持 `$name`、`@name`、`:name` 三种前缀，它们<u>完全等价</u>，实际项目里统一一种写法即可；<u>任何时候都不要把用户输入用字符串拼接进 SQL</u>，参数化不只是防注入，也省去手工转义引号与类型；  
* 需要在循环里反复执行同一条 SQL 时，先调用 `cmd.Prepare()` 让它预编译一次，后续执行复用，对应 C 接口的 `sqlite3_prepare_v2`；  
* 参数值按 .NET 类型自动映射（见下节表格）；需要显式指定 SQLite 类型时可用 `cmd.Parameters.Add("$data", SqliteType.Blob)`；  
* 一条 `SqliteCommand` 建议只放一条 SQL 语句，不要依赖"多语句执行时返回值来自哪一条"的语义；`ExecuteNonQuery()` 返回的行数对 `SELECT` 没有意义；  

### 查询与结果读取
``` C#
using var cmd = conn.CreateCommand();
cmd.CommandText = "SELECT id, name, salary, phone FROM employees WHERE dept_id = $dept ORDER BY id;";
cmd.Parameters.AddWithValue("$dept", 1);

using var reader = cmd.ExecuteReader();
while (reader.Read())
{
    long    id     = reader.GetInt64(0);                        // 按列序号读取，序号从 0 开始
    string  name   = reader.GetString(1);
    double  salary = reader.GetDouble(2);
    string? phone  = reader.IsDBNull(3) ? null : reader.GetString(3);   // NULL 判断
    // 也可以用列名：reader.GetOrdinal("name") 或 reader["name"]
}
```

`AddWithValue` 写入时按 .NET 类型选择存储类，读取时再用对应的 `GetXxx()` 取回。常用的映射关系如下：  

| <font size=2>.NET 类型</font> | <font size=2>SQLite 存储类</font> | <font size=2>说明</font> |
| :---- | :---- | :---- |
| <font size=2>`bool`、`byte`、`short`、`int`、`long`</font> | <font size=2>INTEGER</font> | <font size=2>布尔值存为整数 0 与 1，与 `Boolean 与 Date Time` 小节的结论一致</font> |
| <font size=2>`double`、`float`</font> | <font size=2>REAL</font> | <font size=2>浮点数按 IEEE 格式存储</font> |
| <font size=2>`string`、`char`</font> | <font size=2>TEXT</font> | <font size=2>默认 UTF-8 编码</font> |
| <font size=2>`byte[]`</font> | <font size=2>BLOB</font> | <font size=2>按原始字节存取</font> |
| <font size=2>`DateTime`</font> | <font size=2>TEXT</font> | <font size=2>默认格式为 `"yyyy-MM-dd HH:mm:ss.FFFFFFF"`，用 `GetDateTime()` 读回</font> |
| <font size=2>`DateTimeOffset`</font> | <font size=2>TEXT</font> | <font size=2>带时区偏移量的文本形式</font> |
| <font size=2>`DateOnly`、`TimeOnly`、`TimeSpan`</font> | <font size=2>TEXT</font> | <font size=2>日期、时间与时长的文本形式</font> |
| <font size=2>`Guid`</font> | <font size=2>TEXT</font> | <font size=2>以文本形式存取，便于直接在 SQL 中比较与查看</font> |
| <font size=2>`decimal`</font> | <font size=2>TEXT</font> | <font size=2>以文本形式存取，避免浮点数带来的精度损失</font> |

* <u>时间与 GUID 默认都以 TEXT 存储</u>：好处是可以直接用字符串比较、肉眼可读，`DateTime` 的这种格式也恰好能按字典序正确排序；需要按时间范围查询时照常写 `WHERE created >= $start` 即可；  
* 读取时优先使用强类型方法 `GetInt64()`、`GetString()`、`GetDouble()`、`GetDateTime()` 或泛型的 `GetFieldValue<T>()`；不确定列类型时用 `GetValue()` 配合 `IsDBNull()`；  
* 动态读取时可用 `reader.FieldCount`、`reader.GetName(i)`、`reader.GetOrdinal("列名")`；`reader.HasRows` 适合"只判断有没有数据"的场景；  
* <u>`SqliteDataReader` 与 `SqliteCommand` 都要用 `using` 及时释放</u>，它们持有原生语句对象；`Read()` 返回 `false` 并不代表 reader 已释放；  

### 事务
``` C#
using var tx = conn.BeginTransaction();     // 默认对应 BEGIN，SQLite 只有 Serializable 一种隔离级别
try
{
    using (var cmd = conn.CreateCommand())
    {
        cmd.Transaction = tx;               // 命令必须挂上事务
        cmd.CommandText = "UPDATE accounts SET balance = balance - 100 WHERE id = 1;";
        cmd.ExecuteNonQuery();
    }
    tx.Commit();
}
catch
{
    tx.Rollback();
    throw;
}
```

* <u>事务中的命令必须把 `cmd.Transaction` 指到该事务上</u>，否则 Microsoft.Data.Sqlite 会直接抛 `InvalidOperationException`——这是它与 System.Data.SQLite 的一个行为差异；  
* 支持嵌套：在事务内再次调用 `conn.BeginTransaction()` 会用 `SAVEPOINT` 实现，内层 `Rollback()` 只撤销内层，外层事务仍可继续，与 `事务 Transaction` 一节的保存点语义一致；  
* SQLite 只有**串行化**隔离级别，`BeginTransaction(IsolationLevel.XXX)` 传入其它值不会得到"读未提交"之类的效果；  
* 默认的 `BeginTransaction()` 相当于 `BEGIN DEFERRED`，如果事务里是"先读后写"，<u>可以在打开事务后立刻执行一条写语句</u>先拿到写锁，避免锁升级失败导致 `SQLITE_BUSY`（这正是 `BEGIN IMMEDIATE` 的作用）；  

### 扩展能力：自定义函数、排序规则与大对象
``` C#
// 自定义标量函数：注册后可在 SQL 中直接调用，如 SELECT to_hex(data) FROM files;
conn.CreateFunction("to_hex", (byte[] data) => Convert.ToHexString(data));

// 自定义聚合函数：种子 + 逐行累积 + 收尾，如 SELECT product(rate) FROM ...
conn.CreateAggregate("product", 1.0,
    (double acc, double x) => acc * x,
    acc => acc);

// 自定义排序规则：注册后照用 COLLATE，如 ORDER BY name COLLATE PINYIN
conn.CreateCollation("PINYIN", (x, y) => string.Compare(x, y, StringComparison.OrdinalIgnoreCase));
```

* `CreateFunction` / `CreateAggregate` 有带状态的重载（`CreateFunction<TState, TResult>`），可以给函数携带常量状态；还可以标注 `isDeterministic: true`，告诉查询优化器该函数在相同输入下结果稳定；  
* <u>自定义函数、排序规则都只对当前连接有效</u>，连接从池中复用或重新打开后都需要重新注册，建议把它们封装成"打开连接后的初始化"方法；  
* 加载以动态库形式分发的 SQLite 扩展（如 `csv`、`spellfix1`）可用 `LoadExtension()`，前提是允许加载扩展；更底层的需求（如自定义虚拟表）可以直接用 `SqliteConnection.Handle` 拿到原生 `sqlite3` 句柄，调用 SQLitePCLRaw 的接口；  
* 大 BLOB 建议走流式接口，避免整块数据进出内存：读取用 `reader.GetStream(列序号)`，就地读写用 `SqliteBlob`（需要表上有 `INTEGER PRIMARY KEY`，用 `rowid` 定位行）；  
* 备份用 `BackupDatabase()`，它是**在线备份**：源库可以继续读写，比在文件层面复制 `.db` 文件安全得多，尤其在 WAL 模式下；  

``` C#
// 流式读取 BLOB，避免一次性载入内存
using (var reader = cmd.ExecuteReader())
{
    while (reader.Read())
    {
        using var stream = reader.GetStream(0);
        // 例如 stream.CopyTo(fileStream);
    }
}

// 就地读写大对象：按区间写入指定行的列
using (var blob = new SqliteBlob(conn, "files", "content", rowid: 1, readOnly: false))
{
    blob.Write(bytes, offset: 0, count: bytes.Length);
}

// 在线备份到另一个数据库文件
using var source      = new SqliteConnection("Data Source=app.db");
using var destination = new SqliteConnection("Data Source=backup.db");
source.Open();
destination.Open();
source.BackupDatabase(destination);      // 等价于 C 接口的 sqlite3_backup_* 系列
```

### 使用注意
* <u>异步方法是"假异步"</u>：SQLite 本身没有异步 IO，`ExecuteNonQueryAsync()`、`ExecuteReaderAsync()` 等实际是同步执行后立即返回，既不会释放调用线程，也不要用它们来"提升并发"；  
* 连接对象**不是线程安全**的：一个连接同一时刻只应由一个线程使用；需要并发时应使用多个连接，并配合 `Pooling`、`Default Timeout`（`busy_timeout`）与 WAL 模式来缓解写锁竞争；  
* <u>把连接级配置写进连接字符串</u>（如 `Foreign Keys=True`、`Recursive Triggers=True`），不要依赖"打开后执行一次 `PRAGMA`"，否则连接池复用底层连接时状态不可预期；  
* `Password` 之类的加密能力取决于所选的 SQLitePCLRaw bundle，默认 bundle 不带加密；需要加密时改用 `SQLitePCLRaw.bundle_e_sqlcipher` 等，并注意密钥的保管方式；  
* 需要 `DataTable`/`DataSet` 时，Microsoft.Data.Sqlite 不提供 `DataAdapter`，可以手动用 reader 填充，或直接改用 System.Data.SQLite；不想手写 SQL 则可用 EF Core 的 `Microsoft.EntityFrameworkCore.Sqlite`——它的底层正是本节的 `Microsoft.Data.Sqlite`，因此本章的连接、事务与类型映射知识同样适用；  

> 其它语言的绑定（Python 的 `sqlite3`、Java 的 JDBC 驱动、Rust 的 `rusqlite` 等）本质上都是对 C 接口的封装，因此"预编译语句 + 绑定参数 + 检查返回码"这套流程在各语言中是一致的。<u>把这套流程理解透，比记住某个语言的具体方法名更有价值</u>。