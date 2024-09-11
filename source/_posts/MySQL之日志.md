---
title: MySQL之日志
date: 2024-08-29 12:14:00
tags: 
    - MySQL
    - Ops
categories: 数据库
---

## 错误日志

错误日志记录了MySQL启动和停止时，服务器运行过程发生的错误相关信息；该日志默认开启。

查看日志位置，一般位于`/var/log`下

```sql
show variables like '%log_error%'
```

## 二进制日志

BINLOG，记录了所有DDL和DML

作用：

- 灾难时的数据恢复
- MySQL的主从复制

MySQL8中二级制日志默认开启

查看日志位置：

```sql
show variables like '%log_bin%'
```

日志格式：

- STATEMENT
  - 基于SQL语句的记录
- ROW
  - 基于行的日志记录，记录每行的数据变更
- MIXED
  - 混合STATEMENT和ROW

查看格式：

```sql
show variables like '%binlog_format%';
```

可在MySQL配置文件使用`binlog_format=xxx`指定日志格式

查看binlog需要使用`mysqlbinlog`来查看，具体语法：

```sql
mysqlbinlog [-arguments] logfile
```

清理日志：

- `restart master` 删除所有binlog
- `purge master logs to 'binlog.****'` 删除***编号之前的所有日志
- `purge master logs before 'yyyy-mm-dd hh:mi:ss'`删除指定时间之前产生的日志

## 查询日志

记录客户端所有的操作语句

查看日志参数：

```sql
show variables like '%general%'
```

在MySQL配置文件指定`general_log=1`开启查询日志，`general_log_file=xxx`指定日志路径

## 慢查询日志

记录了所有执行时间超过参数long_query_time并扫描记录不小于min_examined_row_limit的所有SQL语句日志

在MySQL配置文件指定`slow_query_log=1`开启日志，`long_query_time=xxx`指定执行时间

默认不会记录管理语句，也不会记录不使用所有进行查找的查询，在MySQL配置文件指定`log_slow_admin_statements`开启日志，`log_queries_not_using_indexes`指定执行时间

查看日志位置：

```sql
SHOW VARIABLES LIKE  '%slow_query_log%';
```

