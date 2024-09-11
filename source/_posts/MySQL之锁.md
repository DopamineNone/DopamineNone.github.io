---
title: MySQL之锁
date: 2024-08-28 08:46:00
tags: 
    - MySQL
categories: 数据库
---

## 概述

锁是计算机协调多进程/线程访问临界资源的机制。MySQL中，按锁的的粒度可分为三类锁：

- 全局锁：数据库中所有表
- 表级锁：每次操作锁整张表
- 行级锁：每次操作锁相应的行

## 全局锁

加锁后整个实例处于只读状态。

应用场景：全库的逻辑备份

```sql
-- 加全局锁
flush tables with read lock;

-- 备份数据
mysqldump -u root -p your_database > xxx.sql

-- 解锁
unlock tables;
```

存在的问题：

- 主库备份：备份期间业务停摆
- 从库备份：不能执行主库同步过来的binlog，导致主从延迟

故推荐用参数`--single-transaction`代替全局锁完成一致性数据备份

```sql
mysqldump --single-transaction ...
```

## 表级锁

表级锁又分为：

- 表锁
  - 表共享读锁
  - 表独占写锁
- 元数据锁
- 意向锁

### 表锁

```sql
-- 加锁
lock tables table_name [read|write];
-- 释放锁
unlock tables;
```

表共享读锁, 表独占写锁与Golang的读写锁`sync.RWMutex`概念类似

### 元数据锁

meta data lock(MDL), 加锁过程由系统自动控制，MDL锁主要作用是维护元数据的一致性；避免DML和DDL冲突，保证读写正确性。

![image-20240828090653972](./pics/image-20240828090653972.png)

查看当前数据库使用的元素据锁：

```sql
select object_type, object_schema, object_name, lock_type, lock_duration from performance_schema.metadata_locks;
```

### 意向锁

作用：避免DML加的行锁和表锁的冲突；使用意向锁来减少表锁的检查;也是系统自动追加的锁。

- 防止死锁和冲突：通过使用意向锁，系统可以更好地协调不同事务之间的并发操作，避免出现死锁情况。
- 提供锁升级路径：意向锁允许从较低级别的锁（如行级锁）升级到更高级别（如表级锁），而无需释放所有较低级别的锁再重新加锁。

- 意向共享锁（IS）：读锁兼容，写锁互斥
- 意向排他锁（IX）：读写锁都互斥

查看意向锁和行锁情况：

```sql
select object_schema, object_name, index_name, lock_type, lock_mode, lock_data from performance_schema.data_locks;
```

## 行级锁

锁定粒度最小，并发度最高。

行锁是通过索引上的索引项加锁来实现的，而不是对记录加的锁。

InnoDB下行级锁又分为三类：

- 行锁： 锁定单个行记录的锁，防止其他事务对此进行update和delete
- 间隙锁：锁定索引记录间隙，防止其他事务在足够间隙进行insert，产生幻读
- 临键锁：行锁和间隙锁的组合

### 行锁

- 共享锁（S）
- 排他锁（X）

![image-20240828093124913](./pics/image-20240828093124913.png)

- 针对唯一索引进行检索，对已存在的记录进行等值匹配，将会自动优化为行锁。
- InnoDB的行锁是针对索引加的锁，不通过索引条件检索数据则InnoDB将对表中的所有记录加锁，即升级为表锁

### 间隙锁

REPEATABLE READ事务隔离级别下，InnoDB使用next-keys锁进行搜索和索引扫描，防止幻读

- 对索引上的等值查询（唯一索引），给不存在的记录加锁，优化为间隙锁
- 对索引上的等值查询（普通索引），向右遍历最后一个值不满足查询需求时，next-key lock退化为间隙锁
- 对索引上的范围查询（唯一索引），回访问到不满足条件的第一个值为止

### 临键锁

行锁+间隙锁