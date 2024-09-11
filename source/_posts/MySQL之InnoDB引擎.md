---
title: MySQL之InnoDB引擎
date: 2024-08-28 13:33:00
tags: 
    - MySQL
categories: 数据库
---

## 逻辑存储结构

- TableSpace 表空间
  - ibd文件，一个mysql实例可以对应多个表空间，用于存储记录、索引等数据
- Segment 段
  - 分为数据段、索引段、回滚段，InnoDB是索引组织表，数据段就是B+树的叶子节点，索引是B+树的非叶子节点。
- Extent
  - 区，表空间的单元结构，每个区大小为1M
- Page
  - 页，InnoDB存储引擎磁盘管理的最小单元，每个页的大小默认为16KB，为了保证页的连续性，InnoDB存储引擎每次从磁盘申请4-5个区
- Row
  - 行

## 架构

### 内存结构

#### Buffer Pool

执行增删改操作时，先操作缓冲池数据；若没有数据，则从磁盘加载并缓存；然后再以一定频率刷新到磁盘从而减少磁盘IO

缓存池以Page为单位，用链表管理Page。

- free page 即空闲Page
- clean page 被使用但违背修改过的page
- dirty page 脏页，数据被修改过，可能与磁盘数据不一致

#### Change Buffer

更改缓冲区，针对非谓唯一二级索引页，在执行DML语句时，如果这些数据Page没有在Buffer Pool中，不会直接操作磁盘，而回将数据变更至Change Buffer中。在未来数据被读取时，数据再合并恢复到Buffer Pool中，再将数据刷到磁盘中。

> 意义：
>
> - 减少磁盘I/O操作
> - 提高写入效率

#### Adaptive Hash Index

自适应hash索引，用于优化对Buffer Pool数据的查询。无需人工干预，是系统根据情况自动完成。

查看是否开启：

```sql
show variables like '%hash_index%'
```

#### Log Buffer

日志缓冲区，保存redo log, undo log数据。日志缓冲区的日志回定期刷新到磁盘中。

innodb_log_buffer_size: 缓冲区大小

innodb_flush_log_at_trx_commit: 日志刷新到磁盘时机

### 磁盘结构

#### System Tablespace

更改缓冲区的存储区域。

参数：`innodb_data_file_path`

#### File-Per-Table Tablespaces

每个表的文件表空间包括单个InnoDB表的数据和索引，并存储在文件系统上的单个数据文件(`.ibd`)

参数：`innodb_file_per_table`

#### General Tablespaces

通用表空间，需要通过CREATE TABLESPACE语法创建通用表空间，在创建表时，可以指定该表空间。

```sql
create tablespace space_name add datafile 'xxx.ibd' engine = innodb;

create table xxx (
	...
) engine=innodb tablespace space_name;
```

#### Undo Tablespaces

撤销表空间，MySQL实例在初始化时会自动创建两个默认的undo表空间，用于存储undo log

#### Temporary Tablespaces

存储临时表

#### Doublewrite Buffer Files

双写缓冲区，innodb从buffer pool刷新到磁盘前，写将数据页写入双写缓冲区文件，用于系统异常时恢复数据(`.dblwr`)

#### Redo log

实现事务持久性，又重做日志缓冲区和重做日志文件组成。事务提交后会把所有修改信息都存在日志中。

### 后台线程

作用：在合适的时机将数据刷新到磁盘中

- Master Thread
  - 负责将缓冲区数据异步刷新到磁盘中，保持数据一致性；脏页刷新，合并插入缓存，undo页回收
- IO Thread
  - Read thread
  - Write thread
  - Log thread 日志缓冲区刷入磁盘
  - Insert buffer thread 写缓冲区刷入磁盘
- Purge Thread
  - 回收事务已经提交的undo log
- Page Cleaner Thread
  - 协助Master Thread刷新脏页到磁盘的线程

## 事务原理

### 事务

不可分割的工作单位，事务的所有操作要么同时成功，要么同时失败

特点：ACID

> 原子性、一致性、持久性由redo log、undo log保证， 隔离性由锁、MVCC保证

### Redo Log

重做日志由重做日志缓冲以及重做日志文件组成，前者在内存中，后缀在磁盘中。

### Undo Log

回滚日志，记录数据被修改前的信息。作用：提供回滚和MVCC

undo log是逻辑日志，记录与当前操作效果相反的记录。

undo log在事务提交后，并不会删除undo log，这些日志可能会用在MVCC

undo log采用段的方式进行管理和记录，存放在rollback segment回滚段

## MVCC

- 当前读

读取的记录是最新版本，且要保证其他并发事务不能修改当前记录

```sql
select ... lock in share mode
select ... for update
update
insert
delete
```

- 快照读

简单的select就是快照读，读取的是记录数据的可见版本

- Read Committed 每次select都会生成一个快照读
- Repeatable Read 开启事务后第一个select语句才是快照读的地方
- Serializable 快照读会退化为当前读

- MVCC

全程：多版本并发控制，指维护一个数据的多个版本，是读写操作没有冲突。

MVCC具体实现依赖于数据库记录的三个隐式字段、undo log日志、readView

### 隐藏字段

- `DB_TRX_ID` 最近改动事务ID，记录插入这条记录或最后一次修改该记录的事务ID
- `DB_ROLL_PTR`回滚指针，指向这条记录的上一个版本
- `DB_ROW_ID` 如果表结构没有主键，将会生成该隐藏字段

### undo log

当insert的时候，产生的undo log日志只在回滚是需要，事务提交后可以立即删除

而update、delete的时候，产生的undo log日志不仅在回滚需要，快照读时也需要

对同一事务的修改会导致undolog生成一条记录版本链表，头部时最新的旧记录，尾部是最早的旧记录

### ReadView

是快照读SQL指向时MVCC提取数据的依据

四个字段：

- m_ids 当前活跃事务ID集合/尚未提交的事务ID集合
- min_trx_id 最小活跃事务ID/尚未提交的事务ID值
- max_trx_id 预分配事务ID/下次要生成的事务ID值
- creator_trx_id ReadView创建者的事务ID

规则：

![image-20240828150848272](./pics/image-20240828150848272.png)

- RC隔离级别下，在事务中每一次执行快照读时生成ReadView
- RR隔离级别下，每一个事务最多生成一次ReadView

关于事务、MVCC，这里推荐一个介绍视频：[面试官：你怎么理解MySQL事务和MVCC的？](https://www.bilibili.com/video/BV1FW421d7DA/?spm_id_from=333.1007.top_right_bar_window_history.content.click&vd_source=dd291d3bf50e92741817346c92c84f80)