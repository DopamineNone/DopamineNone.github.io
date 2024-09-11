---
title: MySQL之主从复制
date: 2023-12-45 13:13:00
tags: 
    - MySQL
    - Ops
categories: 数据库
---

## 概述

主从复制指主数据库的DDL和DML操作通过二进制日志传送到从库服务器，在从库上对这些日志重新执行，从而使得从库和主库的数据保持同步

MySQL支持链状复制，一个主库可以向多个从库进行复制

作用：

- 主库故障，可以快速切换到从库提供服务
- 实现读写分离，降低主库的访问压力
- 从主库执行备份，避免备份期间影响主库服务

## 原理

![image-20240829125228774](./pics/image-20240829125228774.png)

1. Master将DDL/DML操作写入本地binlog
2. Slave的IO线程读取Master的binlog，并将其内容写入Slave本地的Relay log
3. Slave的SQL线程从中继日志Relay Log读取内容，并重做其中的时间，将改变反映到它自己的数据

## 搭建

### 主库配置

修改配置文件

```cmd
# 要保证id在整个集群环境唯一
server-id=1
# 1表示只读，0表示读写
read-only=0
```

> 这里read-only仅针对普通用户，super用户仍有权限对数据库进行写操作；可以使用`super-read-only=1`限制super用户进行写操作

重启服务后，主库还需要创建远程连接的账号，并赋予主从复制的权限

```sql
CREATE USER 'XXX'@'XXX' IDENTIFIED WITH mysql_native_password BY 'XXX'
GRANT REPLICATION SLAVE ON *.* TO 'XXX'@'XXX'
```

查看二进制日志坐标

```sql
show master status;
```

字段说明：

- file:从哪个文件开始推送日志文件
- position:从哪个位置开始推送日志
- binlog_ignore_db:指定不需要同步的数据库

### 从库配置

修改配置文件

```cmd
server-id=xxx
read-only=1
```

重启服务后，执行如下SQL语句

```sql
# 8.0.23之后
CHANGE REPLICATION SOURCE TO SOURCE_HOST='XXX', SOURCE_USER='XXX', SOURCE_PASSWORD='XXX', SOURCE_LOG_FILE='XXX', SOURCE_LOG_POS='XXX';
# 8.0.23之前
CHANGE MASTER TO MASTER_HOST='XXX', MASTER_USER='XXX', MASTER_PASSWORD='XXX', MASTER_LOG_FILE='XXX', MASTER_LOG_POS=XXX;
```

启动主从复制：

```sql
start replica; # 8.0.22之后
start slave; # 8.0.23之前
```

查看主从复制状态(在从库执行)

```sql
show replica status;
```

