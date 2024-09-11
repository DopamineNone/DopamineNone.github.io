---
title: MySQL之索引
date: 2024-08-24 15:50:00
tags: 
    - MySQL
    - SQL
    - 后端
categories: 数据库
---

## 索引概述

索引（Index）是MySQL**高效**获取数据的数据结构

优点：

- 提高数据检索的效率，降低数据库的IO成本
- 通过索引列对数据进行排序，降低数据排序的成本，降低CPU的消耗

劣势：

- 索引列要占空间
- 降低更新表的速度

## 索引结构

MySQL索引由存储引擎实现，不同存储引擎有不同结构：

- B+Tree 最常见
- Hash
- R-Tree
- Full-text

![image-20240824145049683](./pics/image-20240824145049683.png)

> 数据结构补充：
>
> 搜索二叉树：任意节点都满足: 左子树最大值 < 根节点值 < 右子树最大值
>
> 红黑树： 平衡的搜索二叉树
>
> B-Tree(多路平衡查找树): 一个节点最多可以存储4个key5个指针；每个指针指向的子树的值均在该指针相邻的key值表示的区间

### B+Tree

- 所有元素会出现在叶子节点
- 根元素会出现在右节点第一个
- MySQL中对经典B+Tree基础上，在叶子节点增加了指向相邻叶子节点的指针

> - 相对二叉树，层级更少，搜索效率高
> - 相对于B-Tree，要保存叶子节点和非叶子节点，导致一页中存储的键值减少，指针跟着减少，要同样保存大量数据，只能增加树的高度，导致性能降低

### Hash

用hash算法将键值换算城信hash值，映射在对应槽位上，再存储在hash表中

- 只能用于对等比较，不支持范围查询
- 无法利用索引完成排序操作
- 查询效率高

## 索引分类

![image-20240824152438057](./pics/image-20240824152438057.png)

InnoDB中，索引又可以分为**聚集索引**和**二级索引**

![image-20240824152630129](./pics/image-20240824152630129.png)

聚集索引选取规则：

- 存在主键，主键索引就是聚集索引
- 不存在主键，将使用第一个唯一索引作为聚集索引
- 都没有，InnoDB会自动生成一个rowid作为隐藏的聚集索引

> 回表查询：先从二级索引中拿到行id，再用id从聚集索引找到该行的记录

## 索引语法

### 创建索引

```sql
CREATE [UNIQUE|FULLTEXT] INDEX index_name ON table_name (column_name);
```

> 一个索引只关联一个字段：单列索引
>
> 一个索引只关联多个字段：联合索引/组合索引

### 查询索引

查询现有索引

```sql
show index from table_name;
```

### 删除索引

```sql
drop index index_name on table_name;
```

## SQL性能分析

### SQL执行频率

```sql
show {global|session} status like 'Com_______';
```

### 慢查询日志

慢查询日志会保存执行时间超过某一阈值的SQL语句。

查看慢日志开启状态

```sql
show variables like 'slow_query_log';
```

开启慢查询日志:

```cmd
# /etc/my.cnf

slow_query_log=1
# 超时阈值（秒）
long_query_time=2
```

慢查询日志位置：`/var/lib/mysql/localhost-slow.log`

### Profile详情

profiles能在做SQL优化是帮助我们了解时间耗费的分布情况。

查询是否支持profiles

```sql
show @@have_profiling;
```

查询是否开启profiles

```sql
show @@profiling;
```

开启profiles

```sql
set profiling=1;
```

查询当前会话SQL的耗时情况

```sql
show profiles;

-- 指定query_id的SQL语句耗时情况
show profile for query query_id;

-- ...的CPU使用情况
show profile cpu for query query_id
```

### explain执行计划

用`explain/desc`命令获取MySQL如何执行SELECT语句的信息

 ```sql
 {explain|desc} SQL查询语句
 ```

各字段含义：

- id
  - 表示查询中执行select子句的顺序（越大先执行，相同则从上到下执行）
- select_type
  - SIMPLE
  - PRIMARY 外层查询
  - UNION
  - SUBQUERY 子查询
- type (由好到坏)
  - NULL 不访问任何表的查询
  - system 访问系统表
  - const 查询常量表或表只有一行数据
  - eq_ref 使用唯一索引
  - ref 使用非唯一索引
  - range 使用索引的一部分查询
  - index 全索引查询
  - all 全表查询
- possible_key
  - 可能应用这张表的索引
- Key
  - 实际用到的索引
- Key_len
  - 表示索引中引用的字节数
- rows
  - MySQL认为必须要执行查询的行数
- filtered
  - 返回结果的函数占需读取的行数的百分比

## 索引的使用

- 最左前缀法则（**联合索引**）
  - 从索引的最左列开始，并且不跳过索引中的列（如果跳跃某一列，后面的索引会失效）
- 范围查询
  - 联合索引中，出现范围查询（`<`, `>`）会使右侧的列索引失效
  - 可以使用`>=`或`<=`解决
- 索引列运算
  - 在索引列上进行运算操作，索引将失效
- 字符串不加引号
  - 字符串不加引号（导致隐式转换）也会导致索引失效
- 模糊查询
  - 头部模糊匹配会导致索引失效
- or连接的条件
  - 如果or之前有用到索引，后面没有用到索引，会导致索引失败
  - 给原本没有索引的字段创建索引就解决。
- 数据分布影响
  - 如果使用索引比全表搜索还慢，则不会使用索引
- SQL提示
  - use index(index_name) 建议
  - ignore index(index_name) 忽略
  - force index(index_name) 强制 
  - 在from子句之后使用
- 覆盖索引
  - 指查询使用索引，且返回的列在该索引中都能找到
  - 不推荐用`select *`

> extra信息：
>
> - using index condition: 查找使用了索引，但是需要回表查询
> - using where, using index: 直接从索引返回结果

- 前缀索引
  - 可以用字符串的一部分前缀建立索引，可以大大节约索引空间，提高索引效率
  - `create index index_name on table_name(column(n))`, 前缀长度为n
- 单列索引与联合索引
  - 业务场景中，如果涉及多个字段的查询，则建议使用联合索引
  - 可以通过SQL提示来指定查询时使用联合索引

## 索引设计原则

1. 数据量较大（>= 一百万条），查询频繁
2. 针对常作为where, order by, group by操作的字段建立索引
3. 尽量选择区分度高的列作为索引
4. 对于长字符串字段建立索引，建议使用前缀索引
5. 尽量使用联合索引，可以覆盖索引
6. 控制索引数量，索引越多，维护索引的代价也越大，影响增删改的效率
7. 建表时使用NOT NULL可以让优化器更好的确定哪个索引最有效地用于查询

