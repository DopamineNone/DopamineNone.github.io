---
title: Kitex初体验
tags: 
    - Golang
    - Kitex
    - 后端
categories: Web 开发
---

第一次体验Kitex框架，记录一下学习过程。

## 前置知识

### RPC

全程Remote Procedure Call(远程过程调用)，即调用某个远程服务方法，获取其响应。

[RPC调用过程](https://www.cloudwego.io/zh/docs/kitex/getting-started/pre-knowledge/#rpc-%E8%B0%83%E7%94%A8%E7%9A%84%E6%B5%81%E7%A8%8B):

1. 客户端构造参数、发现并与服务端建立连接、序列化参数并发送。
2. 服务端反序列化参数、处理响应、将结果序列化并发送客户端。
3. 客户端接收响应后反序列化响应，得到结果。

### Thrift

这里主要是指一种IDL(Interface Description Language)协议，用于定义接口交互的数据类型以及生成客户端/服务端模板代码。

主要数据类型有：

- BaseType: 'bool' | 'byte' | 'i8' | 'i16' | 'i32' | 'i64' | 'double' | 'string' | 'binary' | 'uuid'
- enum
- Container: 'list' | 'set' | 'map'
- Struct: `struct StructName { FieldID: [required/optional] DataType ItemName\n ...}`

> 1. 注意这里的struct的field可选类型有三种：required/optional/default。当没有在field下加required/optional字段就是default模式。当发送方发送的某个filed为nil时，default模式下服务端会为该field构造默认值，optional模式下为nil。  
> 2. FieldID不可重复

- Service:  `service ServiceName { RetnType FuncName(FieldID: ParamType param, ...)\n ...}`

> 代码工具通过service的定义实现client/server接口的实现

关键标识符:

| Identifier | Function | How to Use | Demo |
|:---:|:---:|:---:|:---:|
| namespace  | 定义命名空间 | `namespace [language] [route]` | `namespace go com.example` |
| include    | 包含外部thrift文件 | `include "filename.suffix"` | `include "base.thrift"` |
| typedef   | 重定义类型 | `typedef type YourType` | `typedef i64 MyInt` |
| const    | 定义常量 | 略 | 略|
> 通过include包含外部文件后，想使用该文件内的变量需要通过**文件名**引用，而不是**命名空间**

### kitex

kitex是Kitex框架提供的代码生成命令行工具，支持thrift和protobuf的IDL。

用法： `kitex [options] path/to/idl/file`

常用选项：

- module 参数表明生成代码的 go mod 中的 module name指定模板代码的模块
- service 表明我们要生成手脚架代码。
- use 指定kitex-gen目录，kitex-gen下为框架运行必需的代码

## 初体验下

看看这两个demo就能很快上手：

- [基础示例](https://www.cloudwego.cn/zh/docs/kitex/getting-started/quick_start/)
- [进阶教程](https://www.cloudwego.cn/zh/docs/kitex/getting-started/tutorial/)

## 实操

我用Kitex重写5G_AKA实体实例(将原生net改成Kitex罢了)，这里仅贴出部分代码，完整代码见仓库[5G_AKA](https://github.com/DopamineNone/5G-AKA-go)

先定义各实体间的通信接口

---

**学习资料**：

- [Kitex官方文档](https://www.cloudwego.io/zh/docs/kitex)
- [Thrift简介与IDL基本语法](https://blog.csdn.net/weixin_41519463/article/details/108042629)
