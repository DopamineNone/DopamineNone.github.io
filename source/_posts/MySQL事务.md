---
layout: passenge
title: MySQL事务
date: 2024-08-20 10:24:00
categories: 数据库
tags: 
    - MySQL
    - SQL
---

## 事务

事务是一组操作的集合，是不可分割的工作单位。事务中的所有操作要么同时成功，要么同时失败。事务保证了MySQL的ACID特性。

## 事务操作

```sql
-- 方法一
start transation;
...
-- 提交|回滚
commit|rollback;

-- 方法二
set autocommit=0;
...
commit|rollback;
set autocommit=1;
```

## 事务特性

- Atomicity 原子性， 事务是不可分割的最小操作单元，要么全部成功，要么全部失败
- Consistency 一致性， 事务完成，必须使所有数据保持一致性
- Isolation 隔离性， 数据库系统提供隔离机制，保证事务在不受外部并发操作影响的独立环境下运行
- Durablitiy 持久性， 事务一旦提交或回滚，对数据的改变是永久的

## 事务隔离级别

并发事务一般会有以下问题：

- 脏读：一个事务读到另一个事务还没提交的数据
- 不可重复读：一个事务先后读取同一记录，但两次读取的数据不同
- 幻读：一个事务按照条件查询数据时，没有对应的数据行，但在插入数据时，又发现该数据已经存在

MySQL隔离级别：

- Read uncommitted
- Read committed 解决了脏读
- Repeatable Read (default) 解决了脏读和不可重复读
- Serializable 解决了脏读+不可重复+幻读

```sql
-- 查询事务隔离级别
select @@TRANSACTION_ISOLATION;

-- 设置隔离级别
set session transaction isolation level [level];
```

