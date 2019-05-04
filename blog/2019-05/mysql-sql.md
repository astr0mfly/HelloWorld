---
date: "2019-05-03T11:20:15-07:00"
draft: true
title: "Mysql-Sql"
#tags: []
#series: []
#categories: []
toc: true
---
# 基础知识

核心通用语句：`SELECT col_name1,col_name2 FROM table_name WHERE condition`

- 所要查询的列名可以是多个，如果要全部的话，直接使用 *
- 可以使用 DISTINCT 关键词去重，变为 SELECT DISTINCT xxx FROM xxx
- WHERE 之后的语句是过滤条件，格式为 WHERE col_name operator value，其中运算符可以是:
    - 等于 =
    - 不等于 <>
    - 大于 >
    - 小于 <
    - 大于等于 >=
    - 小于等于 <=
    - 某个范围内 BETWEEN value1 AND value2
    - 符合某种模式 LIKE pattern
    - 在一个列表中 IN (value1, value2)
- 用单引号来环绕文本值，数值不需要引号
- WHERE 中支持多个条件，可以用 AND 和 OR 组合，如果要一起用的话，使用圆括号注明优先级
- 排序使用 ORDER BY，如果需要倒序或者顺序，可以指定 DESC 或 ASC，比如 ORDER BY col1 DESC, col2 ASC
- 用 LIMIT 来限制获取结果的数量
- LIKE 的模式
    - % 代表一个或多个字符，比如 w% 表示以 w 开头的字符串，%x 表示以 x 结尾的字符串，w%x 表示以 w 开头且以 x 结尾的字符串
    - `_` 仅替代一个字符
    - [charlist] 包含列表中的任意一个
    - [^charlist] 不包含列表中的任意一个
- 要结合多个表进行查询，则使用 JOIN，其中有几种不同的方法，需要注意


查询的数据进行一些计算: `SELECT function(col) FROM table`

一类是 Aggregate 函数，把一系列的值处理成一个值；

- avg()
- count()
- first()
- last()
- max()
- min()
- sum()
  
另一类是 Scalar 函数，接受一个值，返回一个值。

- ucase()
- lcase()
- mid()
- len()
- round()
- now()
- format()

# 重点剖析
## SQL 是一种声明式语言

`SELECT first_name, last_name FROM employees WHERE salary > 100000`

## SQL 的语法并不按照语法顺序执行
语法顺序:
SELECT[DISTINCT] -> FROM -> WHERE -> GROUP BY -> HAVING -> UNION -> ORDER BY

执行顺序：
FROM -> WHERE -> GROUP BY -> HAVING -> SELECT -> DISTINCT -> UNION -> ORDER BY


1、 FROM 才是 SQL 语句执行的第一步，并非 SELECT 。数据库在执行 SQL 语句的第一步是将数据从硬盘加载到数据缓冲区中，以便对这些数据进行操作。（译者注：原文为“The first thing that happens is loading data from the disk into memory, in order to operate on such data.”，但是并非如此，以 Oracle 等常用数据库为例，数据是从硬盘中抽取到数据缓冲区中进行操作。）

2、 SELECT 是在大部分语句执行了之后才执行的，严格的说是在 FROM 和 GROUP BY 之后执行的。理解这一点是非常重要的，这就是你不能在 WHERE 中使用在 SELECT 中设定别名的字段作为判断条件的原因。

3、 无论在语法上还是在执行顺序上， UNION 总是排在在 ORDER BY 之前。很多人认为每个 UNION 段都能使用 ORDER BY 排序，但是根据 SQL 语言标准和各个数据库 SQL 的执行差异来看，这并不是真的。尽管某些数据库允许 SQL 语句对子查询（subqueries）或者派生表（derived tables）进行排序，但是这并不说明这个排序在 UNION 操作过后仍保持排序后的顺序。

**我们学到了什么？**

既然并不是所有的数据库都按照上述方式执行 SQL 预计，那我们的收获是什么？我们的收获是永远要记得： SQL 语句的语法顺序和其执行顺序并不一致，这样我们就能避免一般性的错误。如果你能记住 SQL 语句语法顺序和执行顺序的差异，你就能很容易的理解一些很常见的 SQL 问题。

## SQL 语言的核心是对表的引用（table references）

由于 SQL 语句语法顺序和执行顺序的不同，很多同学会认为SELECT 中的字段信息是 SQL 语句的核心。其实真正的核心在于对表的引用。

`<from clause> ::= FROM <table reference> [ { <comma> <table reference> }... ]`

FROM 语句的“输出”是一张联合表，来自于所有引用的表在某一维度上的联合。

## 灵活引用表能使 SQL 语句变得更强大
```sql
<table reference> ::=
    <table name>
  | <derived table>
  | <joined table>
```
在 SQL 语句中派生表的应用甚至比表连接更加强大，下面我们就要讲到表连接。

**我们学到了什么？**

思考问题时，要从表引用的角度出发，这样就很容易理解数据是怎样被 SQL 语句处理的，并且能够帮助你理解那些复杂的表引用是做什么的。

更重要的是，要理解 JOIN 是构建连接表的关键词，并不是 SELECT 语句的一部分。有一些数据库允许在 INSERT 、 UPDATE 、 DELETE 中使用 JOIN 。

## SQL 语句中推荐使用表连接

高级 SQL 程序员也许学会给你忠告：尽量不要使用逗号来代替 JOIN 进行表的连接，这样会提高你的 SQL 语句的可读性，并且可以避免一些错误。

- 安全。 JOIN 和要连接的表离得非常近，这样就能避免错误。
- 更多连接的方式，JOIN 语句能去区分出来外连接和内连接等。

记着要尽量使用 JOIN 进行表的连接，永远不要在 FROM 后面使用逗号连接表。


## SQL 语句中不同的连接操作
SQL 语句中，表连接的方式从根本上分为五种：

- EQUI JOIN (内连接，外连接)
- SEMI JOIN (这种连接方式是只连接目标表的一部分。)
- ANTI JOIN (这种连接的关系跟 SEMI JOIN 刚好相反。在 IN 或者 EXISTS 前加一个 NOT 关键字就能使用这种连接。)
- CROSS JOIN (这个连接过程就是两个连接的表的乘积：即将第一张表的每一条数据分别对应第二张表的每条数据。)
- DIVISION (如果 JOIN 是一个乘法运算，那么 DIVISION 就是 JOIN 的逆过程。)

## SQL 中如同变量的派生表
在这之前，我们学习到过 SQL 是一种声明性的语言，并且 SQL 语句中不能包含变量。但是你能写出类似于变量的语句，这些就叫做派生表,所谓的派生表就是在括号之中的子查询：
```sql
-- A derived table
FROM (SELECT * FROM author)
```
我们反复强调，大体上来说 SQL 语句就是对表的引用，而并非对字段的引用。要好好利用这一点，不要害怕使用派生表或者其他更复杂的语句。


## SQL 语句中 GROUP BY 是对表的引用进行的操作

**GROUP BY A.x, A.y, B.z**

上面语句的结果就是产生出了一个包含三个字段的新的表的引用。我们来仔细理解一下这句话：当你应用 GROUP BY 的时候， SELECT 后没有使用聚合函数的列，都要出现在 GROUP BY 后面。（译者注：原文大意为“当你是用 GROUP BY 的时候，你能够对其进行下一级逻辑操作的列会减少，包括在 SELECT 中的列”）。

GROUP BY，再次强调一次，是在表的引用上进行了操作，将其转换为一种新的引用方式。

## SQL 语句中的 SELECT 实质上是对关系的映射

我个人比较喜欢“映射”这个词，尤其是把它用在关系代数上。（译者注：原文用词为 projection ，该词有两层含义，第一种含义是预测、规划、设计，第二种意思是投射、映射，经过反复推敲，我觉得这里用映射能够更直观的表达出 SELECT 的作用）。一旦你建立起来了表的引用，经过修改、变形，你能够一步一步的将其映射到另一个模型中。 SELECT 语句就像一个“投影仪”，我们可以将其理解成一个将源表中的数据按照一定的逻辑转换成目标表数据的函数。
通过 SELECT语句，你能对每一个字段进行操作，通过复杂的表达式生成所需要的数据。

SELECT 语句有很多特殊的规则，至少你应该熟悉以下几条：

- 你仅能够使用那些能通过表引用而得来的字段；
- 如果你有 GROUP BY 语句，你只能够使用 GROUP BY 语句后面的字段或者聚合函数；
- 当你的语句中没有 GROUP BY 的时候，可以使用开窗函数代替聚合函数；
- 当你的语句中没有 GROUP BY 的时候，你不能同时使用聚合函数和其它函数；
- 有一些方法可以将普通函数封装在聚合函数中；
- ……

原因如下：

- 凭直觉，这种做法从逻辑上就讲不通。
- 如果直觉不能够说服你，那么语法肯定能。 SQL : 1999 标准引入了 GROUPING SETS，SQL： 2003 标准引入了 group sets : GROUP BY() 。无论什么时候，只要你的语句中出现了聚合函数，而且并没有明确的 GROUP BY 语句，这时一个不明确的、空的 GROUPING SET 就会被应用到这段 SQL 中。因此，原始的逻辑顺序的规则就被打破了，映射（即 SELECT ）关系首先会影响到逻辑关系，其次就是语法关系。（译者注：这段话原文就比较艰涩，可以简单理解如下：在既有聚合函数又有普通函数的 SQL 语句中，如果没有 GROUP BY 进行分组，SQL 语句默认视整张表为一个分组，当聚合函数对某一字段进行聚合统计的时候，引用的表中的每一条 record 就失去了意义，全部的数据都聚合为一个统计值，你此时对每一条 record 使用其它函数是没有意义的）。

**我们学到了什么？**

SELECT 语句可能是 SQL 语句中最难的部分了，尽管他看上去很简单。其他语句的作用其实就是对表的不同形式的引用。而 SELECT 语句则把这些引用整合在了一起，通过逻辑规则将源表映射到目标表，而且这个过程是可逆的，我们可以清楚的知道目标表的数据是怎么来的。

想要学习好 SQL 语言，就要在使用 SELECT 语句之前弄懂其他的语句，虽然 SELECT 是语法结构中的第一个关键词，但它应该是我们最后一个掌握的。

## DISTINCT ， UNION ， ORDER BY 和 OFFSET
在学习完复杂的 SELECT 之后，我们再来看点简单的东西：

- 集合运算（ DISTINCT 和 UNION ）
- 排序运算（ ORDER BY，OFFSET…FETCH）
- 集合运算（ set operation）：


集合运算主要操作在于集合上，事实上指的就是对表的一种操作。从概念上来说，他们很好理解：

- DISTINCT 在映射之后对数据进行去重
- UNION 将两个子查询拼接起来并去重
- UNION ALL 将两个子查询拼接起来但不去重
- EXCEPT 将第二个字查询中的结果从第一个子查询中去掉
- INTERSECT 保留两个子查询中都有的结果并去重


排序运算（ ordering operation）：

排序运算跟逻辑关系无关。这是一个 SQL 特有的功能。排序运算不仅在 SQL 语句的最后，而且在 SQL 语句运行的过程中也是最后执行的。使用 ORDER BY 和 OFFSET…FETCH 是保证数据能够按照顺序排列的最有效的方式。其他所有的排序方式都有一定随机性，尽管它们得到的排序结果是可重现的。

OFFSET…SET是一个没有统一确定语法的语句，不同的数据库有不同的表达方式，如 MySQL 和 PostgreSQL 的 LIMIT…OFFSET、SQL Server 和 Sybase 的 TOP…START AT 等。

## 版本控制

| Version | Action                   | Time       |
| ------- | ------------------------ | ---------- |
| 1.0     | Init                     | 2019-05-03T11:20:15-07:00|
