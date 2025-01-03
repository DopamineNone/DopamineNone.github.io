---
title: SSL / TLS 工作原理
date: 2024-09-09 16:52:09
tags: 
    - Security
    - 计算机网络
categories: 计算机网络
---

## 简介

**SSL**和**TLS**都是Web安全传输协议，在某个实际网络会话中，往往只会选择其中的一种使用。**SSL**（Security Sockets Layer，安全套接层）是早期解决HTTP明文传输带来的安全问题：信息嗅探和篡改。**TLS**（Transport Layer Security, 传输层安全)则是由SSL演变而来。目前主要使用的是**TLS**。

## CA数字证书

**CA**（Certificate Authority）数字证书是由权威认证机构颁发的，是SSL/TLS握手中必不可少的一部分。CA的作用是用于验证服务器身份的，申请流程如下：

1. **生成密钥对**：服务器管理员会生成一对公钥和私钥。这个操作通常是在服务器上通过诸如OpenSSL这样的工具完成的。
2. **证书请求**：服务器会使用生成的公钥创建一个证书签名请求（CSR）。CSR中包含了服务器的公钥以及其他一些信息如域名等。
3. **证书签发**：将CSR提交给一个受信任的CA，CA会对服务器的身份进行验证。一旦身份验证通过，CA会签发一个数字证书，这个证书包含了服务器的公钥以及CA的签名。
4. **安装证书**：服务器安装由CA签发的证书，这样当客户端连接到服务器时，就可以看到这个证书，并且通过验证CA的签名来确认服务器的身份。

## SSL/TLS握手

1. **客户端Hello**: 客户端向服务器发送一个“Client Hello”消息，其中包含了客户端支持的TLS版本、加密算法套件列表、随机数**random1**（用于生成密钥）以及其它参数。
2. **服务器Hello**: 服务器接收到“Client Hello”后，会从提供的加密算法套件中选择一个，并向客户端发送一个“Server Hello”消息，其中包含它选择的参数和自己的随机数**random2**。
3. **证书交换**: 服务器发送其数字证书给客户端，证书中包含了公钥等信息，这允许客户端验证服务器的身份。
4. **服务器密钥交换（可选）**: 在某些情况下，比如使用DHE或ECDHE密钥交换算法时，服务器还需要发送临时的公钥给客户端，以便进行密钥协商。
5. **服务器Hello完成**: 服务器发送一个“Server Hello Done”消息，表示它已经完成了握手的第一阶段。
6. **客户端密钥交换**: 客户端使用之前获得的服务器公钥来加密一个被称为“**pre-master secret**”的值，并发送给服务器。
7. **变更密码规范**: 双方都发送一个“Change Cipher Spec”消息，表示从现在开始所有的通信都将使用协商好的加密算法。
8. **完成握手**: 客户端和服务器各自独立地使用之前交换的信息(**random1**, **random2**, **pre-master secret**)计算出相同的主密钥（master secret），并用这个密钥生成实际的会话密钥，然后双方互发一个带签名的“Finished”消息，确认握手成功。

**SSL/TLS**可应用在双向认证上，可以在客户端-服务端通信中避免**中间人攻击**。

## 协议升级

传统的http服务加上SSL证书配置后可以升级为https服务；类似的，ws协议也可以升级为wss协议以提高安全性。以下为在nginx上配置https接口的过程：