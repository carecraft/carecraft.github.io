---
layout:     post
title:      "在 PostgreSQL 查找结果中添加序列号"
subtitle:   "add sequence number in postgresql select list"
category : language-instrument
date:       2017-07-18
author:     "Max"
header-img: "img/post-dk-2016.jpg"
catalog:    true
tags:
    - sql
---

## 问题描述

需要在 PostgreSQL 的查询结果中添加一个递增的序列号。

## 解决方案

### 调用内置窗口函数

在[网上](http://www.postgresqltutorial.com/postgresql-row_number/)找到了内置窗口函数 row_number() 可以解决该问题。

函数处理的行的集合可以称之为一个窗口（window）。row_number() 函数可以为窗口中的每一行分配一个唯一整数，语法如下：
```
ROW_NUMBER() OVER(
    [PARTITION BY column_1, column_2,…]
    [ORDER BY column_3,column_4,…]
)
```
PARTITION BY 从句可以将窗口分成更小的集合/分区，每个集合/分区的行号从 1 开始重新增长。省略该从句，则将整个窗口视为一个集合/分区。

ORDER BY 从句决定序号的分配依据。

示例如下：
```
SELECT product_id, product_name, group_id, ROW_NUMBER () OVER (ORDER BY product_name) FROM products;
```

可惜在 8.2.4 版本的数据库中似乎不支持。

### 创建序列号生成器

SEQUENCE 序列对象又可称序列生成器，是一种特殊的单行表，一般用来为表的行生成唯一标识。通过 CREATE SEQUENCE 命令创建一个新的序列号生成器可以解决该问题。

[CREATE SEQUENCE 语法](https://www.postgresql.org/docs/current/static/sql-createsequence.html)如下：
```
CREATE [ TEMPORARY | TEMP ] SEQUENCE [ IF NOT EXISTS ] name [ INCREMENT [ BY ] increment ]
    [ MINVALUE minvalue | NO MINVALUE ] [ MAXVALUE maxvalue | NO MAXVALUE ]
    [ START [ WITH ] start ] [ CACHE cache ] [ [ NO ] CYCLE ]
    [ OWNED BY { table_name.column_name | NONE } ]
```

序列对象的相关函数有：

Function | Return Type | Description
--- | --- | ---
currval(regclass) | bigint | 返回指定序列对象的最近一个 nextval() 的值
lastval() | bigint | 返回任意序列对象的最近一个 nextval() 的值
nextval(regclass) |  bigint | 使序列对象增长并返回增长后的值
setval(regclass, bigint) | bigint | 设置序列对象的当前值
setval(regclass, bigint, boolean) | bigint | 设置序列对象的当前值并修改 is_called 标记


示例如下：
```
CREATE SEQUENCE row_number START 1;
SELECT product_id, product_name, group_id, nextval('row_number') FROM products ORDER BY product_name;
```




