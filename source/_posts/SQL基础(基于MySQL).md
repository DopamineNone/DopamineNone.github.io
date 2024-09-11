---
layout: passage
title: SQL基础(基于MySQL)
date: 2024-08-19 9:13:00
tags:
  - SQL
  - MySQL
categories: 数据库
---

## DDL(Data Definition Language)

### 表查询

- 查询当前所有表

```sql
SHOW TABLES;
```

- 查询表结构

```sql
DESC TABLE_NAME;
```

- 查询指定表建表语句

```SQL
SHOW CREATE TABLE TABLE_NAME;
```

### 表创建

```sql
CREATE TABLE TABLE_NAME (
	COLUMN_NAME TYPE [COMMENT] [CONDITION],
    ...
);
```

### 数据类型

- 数值类型
  - TINYINT 	1byte
  - SMALLINT      2bytes
  - MEDIUMINT  3bytes
  - INT / INTEGER 4bytes
  - SIGINT            8bytes
  - FLOAT             4bytes  float(m, d) 类似decimal
  - DOUBLE          8bytes  double(m,d) 类似decimal
  - DECIMAL        decimal(a,b) a表示小数点左边能存储的十进制数字最大数，b表示小数点右边...

> 对于数值类型，可以在字段约束条件中使用unsigned来声明无符号类型。

- 字符串类型
  - CHAR CHAR(N) 定长为n的字符串
  - VARCHAR VARCHAR(N) 最长为n的字符串
  - TINYTEXT, BLOB, TEXT, ... 等长文本或二进制数据
- 日期类型
  - DATE
  - TIME
  - YEAR
  - DATETIME 日期+时间
  - TIMESTAMP 时间戳

### 表修改

#### 修改表名

```sql
-- 方法一
ALTER TABLE TABLE_NAME RENAME AS NEW_TABLE_NAME;
-- 方法二
RENAME TABLE OLD_TABLE_NAME TO NEW_TABLE_NAME
```

#### 添加字段

```sql
ALTER TABLE TABLE_NAME ADD COLUMN TYPE [COMMENT] [CONDITION];
```

#### 修改字段类型

```sql
ALTER TABLE TABLE_NAME MODIFY COLUMN NEWTYPE;
```

#### 修改字段名

```sql
ALTER TABLE TABLE_NAME RENAME OLD_COLUMN TO NEW_COLUMN;
```

#### 修改字段名和类型

```sql
ALTER TABLE TABLE_NAME CHANGE OLD_COLUMN NEW_COLUMN NEWTYPE [COMMENT] [CONDITION];
```

#### 删除字段

```sql
ALTER TABLE TABLE_NAME DROP COLUMN;
```

#### 删除表

```sql
DROP TABLE [IF EXISTS] TABLE_NAME;
```

#### 删除并重新创建表

```sql
TRUNCATE TABLE TABLE_NAME;
```

## DML (Data Manipulation Language)

### 添加数据

```sql
INSERT INTO TABLE_NAME [(COLUMN1, COLUMN2, ...)] VALUES (VALUE1, VALUE2,...), (VALUE1, VALUE2,...), ...;
```

> 添加数据时忽略未给出字段，则表明给全部字段添加数据

### 修改数据

```sql
UPDATE TABLE_NAME SET COLUMN1=VALUE1, COLUMN2=VALUE2,...
[WHERE CONDITION];
```

### 删除数据

```sql
DELETE FROM TABLE [WHERE CONDITION];
```

## DQL(Data Query Language)

关键知识点：

- 选择查询
- 别名
- where条件
- order by排序
- group by分组
- limit 截断和偏移
- case when then ... end条件分支
- having条件（针对group by 分组）
- 内连接、外连接
- 聚合函数
- 开窗函数
- 子查询, exist, in
- with as语句

## DCL(Data Control Language)

### 查询用户

```sql
USE mysql;
SELECT * FROM user;
```

### 创建用户

```sql
CREATE USER 'username'@'hostname' IDENTIFIED BY 'password';
```

> 可以使用通配符%来表示任意主机。

### 修改用户密码

```sql
ALTER USER 'username'@'hostname' IDENTIFIED BY 'newpassword';
```

### 删除用户

```sql
DROP USER 'username'@'hostname';
```

### 查询权限

```sql
SHOW GRANTS FOR 'username'@'hostname';
```

### 赋予权限

```sql
GRANT ALL|OTHER PRIVILEGE ON DATABASE.TABLE TO 'username'@'hostname';
```

### 撤销权限

```sql
REVOKE ALL|OTHER PRIVILEGE ON DATABASE.TABLE FROM 'username'@'hostname';
```

