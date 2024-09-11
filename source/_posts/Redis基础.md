---
title: Redis基础
date: 2024-08-20 11:58:25
tags: 
    - Redis
    - NoSQL
categories: 数据库
---

## Redis介绍

redis是一个键值存储系统，其数据存储在内存中，故常作为后端应用的缓存数据库来提高存储层的数据吞吐量。

特点：

- 键值型存储
- 单线程（指命令执行）
- 低延迟，速度快

## 通用命令

通用命令查询指令：`help @generic`

常见redis通用命令：

- keys 查询键值（支持模糊查询，生产环境不适用）
- del 删除键值，返回成功删除的键值对数
- exists 判断key是否存在
- expire 给一个key设置有效期
- ttl 查看一个key的剩余有效期

## String

redis中最简单的存储类型。根据字符串格式可分为3类：

- string 普通字符串
- int 整型
- float 浮点型

int和float可以做增减操作；字符串类型最大空间不能超过512MB

常用命令：

```cmd
set name john

get name # john

mset k1 v1 k2 v2 k3 1 k4 0.5

mget name k1 k2 
#1) john
#2) v1
#3) v2

incr k3 # 2

incrby k3 2 # 4

decrby k3 2 # 2

decr k3 # 1

incrbyfloat k4 0.6

setnx k1 v4 # 0
get k1 # v1

setex k5 10 1 # expire为10
```

> 推荐的key命名方式：项目名:业务名:类型:id；可实现key的分级存储

## Hash

hash类型允许在redis中存储键值对，每个哈希都有一个唯一的键，并且包含多个字段（field）到值（value）的映射对，类似golang中的map，python的dict。

常见命令：

```cmd
# hset key field value
hset web:user:1 name john
hset web:user:1 email 123@qq.com

hget web:user:1 name # john
hget web:user:1 email # 123@qq.com

# 还有类似mset mget的 hmset hmget

hgetall web:user:1
#1) "name"
#2) "john"
#3) "email"
#4) "123@qq.com"

hkeys web:user:1
#1) "name"
#2) "email"

hvals web:user:1
#1) "john"
#2) "123@qq.com"

# 还有类似的hincrby hsetnx等命令
```

## List

list通常被用来模拟队列（FIFO，先进先出）或栈（LIFO，后进先出）的行为。列表支持的操作使得它可以非常方便地用作消息队列、任务队列等场景。

```cmd
lpush list a
lpush list b
rpush list c

lrange list 0 2
#1) "b"
#2) "a"
#3) "c"

lpop list # b
rpop list # c
rpop list # a
lpop list # nil

# blpop brpop 类似lpop rpop,但会在没元素时等待指定时间，不会直接返回nil
blpop list 10
```

## Set

集合，类似python的set。特点如下：

- 无序
- 元素不可重复
- 查找快
- 支持交集、并集

常用命令：

```cmd
sadd s1 1
sadd s1 2
sadd s2 2
sadd s2 3

srem s1 1

scard s2 # 2
sismember s2 2 # yes
smembers s2
#1) 2
#2) 3

sinter s1 s2
# 2

sdiff s2 s1
# 3

sunion s1 s2
#1) 2
#2) 3
```

## SortedSet

可排序的set，底层每个元素都拥有一个score属性，可以基于score属性对元素进行排序

常见命令：

```cmd
zadd z1 10 john
zrem z1 john

zadd z1 5 john
zadd z1 6 blob
zscore z1 john # 5
zrank z1 john # 0
zcard z1 # 2
zcount z1 3 6 # 2
zincrby z1 2 john # 7
zrange z1 0 1
#1) blob
#2) john
zrangebyscore z1 5 6
#1) blob
zdiff/zinter/zunion
```

## BitMap

BitMap时用于存储和操作位级别的数据，它通常用于表示大量布尔值的状态。

```cmd
setbit status 0 1
setbit status 1 0

getbit status 0 # 1
getbit status 1 # 0

# 本质上bitmap就是字符串
set status "\x0F"

getbit status 3 # 0
getbit status 4 # 1

bitcount status # 4

bitpos status 0 # 0
bitpos status 1 # 4
bitpos status 0 3 4 bit # 3
bitpos status 0 3 4 [byte]# -1
```

## HyperLogLog

HyperLogLog (HLL) 是一种用于估计大量数据中不同元素（即基数）数量的数据结构。它牺牲了一定的精确性，换来了更小的内存消耗。

常用命令：

```cmd
pfadd product1 fruit vegetable
pfcount product1

pfadd product2 meat milk

pfmerge result product1 product2

pfcount result
# 4
```



