---
title: go-redis基本使用
date: 2024-08-27 16:28:09
tags: 
    - Go
    - Redis
categories: Go Tool
---

## 连接

如果redis在虚拟机上，应先修改`redis.cnf`中的几个配置项

```go
protected-mode no
bind 0.0.0.0
```

连接demo如下:

```go
package main

import (
	"context"
	"fmt"
	"github.com/redis/go-redis/v9"
)

var rdb *redis.Client

func initDB() (err error) {
	rdb = redis.NewClient(&redis.Options{
		Addr:     "192.168.131.134:6379",
		Password: "", // no password set
		DB:       0,  // use default DB
	})
	_, err = rdb.Ping(context.Background()).Result()
	return
}

func main() {
	err := initDB()
	if err!= nil {
		panic(err)
	}
	fmt.Println("Hello, Redis!")
}
```

go-redis中操作redis的函数和redis命令用法很接近，以下是不同指令的例子。

## String

```go
func main() {
    ctx := context.Background()
    
    // set
    err := rdb.Set(ctx, "name", "john").Err()
    if err != nil {
        panic(err)
    }
    
    // get
    if value, err := rdb.Get(ctx, "name").Result(); err != nil {
        panic(err)
    } else {
        fmt.Println("Name=", value.(string))
    }
    
    // getset: set newvalue and get oldvalue
    if old, err := rdb.GetSet(ctx, "name", "bob"); err != nil {
        panic(err)
    } else {
        fmt.Println("Old value=", old)
    }
    
    // setnx
    if err = rdb.SetNX(ctx, "name", "john"); err != nil {
        panic(err)
    } else {
        val, _ := rdb.Get(ctx, "name")
        fmt.Println("New name=", val.(string))
    }
}
```



