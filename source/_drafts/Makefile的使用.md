---
title: Makefile的使用
date: 2024-08-30 13:09:00
tags: 
    - Tool
categories: Tool
---

## 概述

Makefile是一个程序构建工具。Makefile工具通过make指令读取项目目录下的`makefile`文件，按照其指定的规则编译链接构建项目应用程序。Makefile构建工具是一种通用构建工具，但实际应用中往往用于构建C、C++、Rust、Go程序。本文用Golang代码作为示例。

## 预备代码

创建示例项目：

```go
go mod init makefile
```

`makefile/pkg/hello/hello.go`:

```go
package hello

import (
	"fmt"
)

func Greet() {
	fmt.Println("Hello Makefile!")
}
```

`makefile/src/main.go`:

```go
package main

import (
	"makefile/pkg/hello"
)

func main() {
	hello.Greet()
}
```

在项目根目录下创建`makefile`文件，下面就开始makefile的编写

## 宏

### 自定义宏

makefile中可以定义宏，如下：

```makefile
GO = go
BINARY = myapp
PKG_DIR = pkg
CMD_DIR = src
```

之后可以通过`$(variable_name)`的形式使用宏

### 特殊宏

- `$@`：代表目标文件的名字。
- `$<`：代表第一个依赖文件的名字。
- `$^`：代表所有依赖文件的名字列表，用空格隔开。
- `$+`：同 `$^` 类似，但重复的依赖只出现一次。
- `$?`：代表所有比目标文件新的依赖文件的名字列表。
- `$*`：代表目标文件或依赖文件名中不包含扩展名的部分。

## 常规宏

可通过`make -p`查看常规宏的值。常规宏可分为两种：

- 作为程序名称的宏
- 包含程序参数的宏

## 规则

