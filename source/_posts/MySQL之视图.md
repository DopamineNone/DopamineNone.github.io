---
title: MySQL之视图
date: 2024-08-27 10:47:00
tags: 
    - MySQL
    - SQL
categories: 数据库
---

## 简介

视图是一种虚拟表，它并不存储实际的数据，而是通过 SELECT 语句定义了一个结果集。视图可以简化复杂的查询，提供数据的安全访问机制，并且可以让用户看到多个表的组合结果。

- 视图的定义本质上是一个 `SELECT` 语句。
- 当查询视图时，实际上是执行了视图定义中的 `SELECT` 语句。
- 视图还具有持久性、安全性、可重用性等特点，这使得它不仅仅是简单的 `SELECT` 语句别名。
  - 持久性：视图定义保存在数据库中，即使数据库重启后仍然存在；视图本身不存储数据，而是指向数据库中实际存储的数据
  - 安全性：视图可以用来限制用户访问敏感数据；用户只能通过视图访问数据，而不能直接访问基础表
  - 可重用性：视图可以被多次使用，就像使用表一样

## 视图的增删改查

### 创建

```sql
CREATE [OR REPLACE] VIEW view_name AS SELECT语句;
```

### 查询

查询视图的`SELECT`语句：

```sql
show create view view_name;
```

查询视图数据用法和查询表数据用法一致

### 修改

- 方法一：`CREATE OR REPLACE VIEW view_name AS ...`
- 方法二：`ALTER VIEW view_name AS ...`

### 删除

```sql
DROP VIEW IF EXISTS view_name;
```

## 检查选项

视图的检查选项的作用是通过视图检查正在修改的每个行是否符合视图的定义。MySQL提供了两个选项：

- WITH CASCADED CHECK OPTION (默认值) 会递归检查操作是否满足当前视图以及其引用的所有视图的定义条件
- WITH LOCAL CHECK OPTION 仅检查操作是否满足当前视图以及**其引用且有检查选项的**的视图的定义

> - MySQL允许基于视图再创建视图
> - 不建议通过视图进行增删改操作

## 视图更新

视图可更新必须满足视图的行和基础表的行必须存在一对一关系。若视图包含以下任意一项，该视图就不可更新：

- 聚合函数/窗口函数
- DISTINCT
- GROUP BY
- HAVING
- UNION/ UNION ALL