---
title: Web之Http基础
date: 2023-08-25 21:39:46
categories: 计算机网络
tags: 
- CTF
- 计算机网络
---

超文本传输协议（Hypertext Transfer Protocol，HTTP）是一个简单的请求-响应协议。

打开浏览器，随便访问一个网站，按F12打开开发者工具，点击**网络/Network**,再次刷新网页，就可以看到你向这个网站服务器发送的HTTP请求数据包了。

![Alt text](images/http/image.png)

## HTTP请求格式

HTTP请求包包含三个部分：请求行、请求头、请求体

### HTTP请求行

- 请求行的内容
  - 方法类型
  - 资源路径+查询参数
  - 协议版本

直观地分析一个请求行  
`GET https://somewebsite.com/index.php?id=1 HTTP/1.1`

- GET 是请求方式中的一种
- `https://somewebsite.com/index.php?id=1` 是URL，既资源路径+查询参数
- HTTP/1.1 协议版本

> **请求方式**  
> 只要有GET和POST两种方式，两者最直观的区别在于**数据参数的位置**
>
> - GET
> 数据参数往往直接写在URL的尾部，比如访问`https://somewebsite.com/index.php?usr=1&pwd=1`时,就向目标网站传递了参数usr=1和pwd=1。
> - POST  
> 数据参数往往写在请求体中  
>GET和POST的使用用途也往往不同，GET往往用来申请访问相关网页资源，POST往往用于向指定资源提交数据进行处理请求（例如提交表单或者上传文件）

### HTTP请求头

有一些常见的字段

- Host: 主机域名
- User-Agent: 客户端相关信息
- Accept：客户端想要的响应数据类型
- Content-Type： 客户端告诉服务器实际发送的数据类型
- X-Forwarded-For： 表示 HTTP 请求端真实 IP
- Cookie: 某些网站为了辨别用户身份、进行session跟踪而储存在用户本地终端上的数据
- Referer: 用于告知服务器用户的来源页面

### HTTP请求体

客户端传给服务器的其他任意数据，POST方法下表单数据在此处  
（使用GET方法的HTTP请求往往没有请求体）

## HTTP响应包

和请求包类似，由三个部分组成：响应行，响应头，响应体

### 响应行

由协议版本，状态码，状态消息组成，比如
`HTTP/1.1 200 OK`

- 协议版本  HTTP/版本号
- 状态码:   XXX
  - 1xx 表示【临时响应】并需要请求者继续执行操作的状态代码
  - 2xx 表示【成功】处理了请求的状态代码
  - 3xx 表示要完成请求，需要进一步操作。通常，这些状态代码用来【重定向】
  - 4xx 表示客户端错误--处理发生错误，责任在客户端
  - 5xx 表示【服务器】在尝试处理请求时发生内部错误。这些错误可能是服务器本身的错误，而不是请求出错  
  
  常见状态码
  - 200 客户端请求成功，即处理成功
  - 302 只是所请求的资源已移动到由Location响应头给定的URL，浏览器会自动重新访问到这个页面
  - 404 请求资源不存在，一般是URL输入有误，或者网站资源被删除了
- 状态消息 描述状态码

### 响应头

告知客户端的信息

- Date
- Content-type

### 响应体

服务器给客户端的数据内容，如HTML文件等。

## 发送自定义HTTP请求

我们可以通过一些工具来发送自定义的HTTP请求

有些网站有配置防止爬虫爬取的技术，所以发送HTTP请求最好要设置HTTP请求头中的User-Agent等字段，让我们发送的请求“看上去”像是浏览器发送的。

### curl命令

Windows和Linux命令行下都能用，是访问URL的计算机逻辑语言的工具。

- 向目标网址发送请求

```Bash
curl http://target.com 
```

- 用指定方式发送请求

```Bash
curl http://target.com -X GET
curl http://target.com -X POST -d "id=1"
```

当使用参数 -d，-X POST 可以省略，因为会隐式发起 POST 请求。

- 设置HTTP请求头

```Bash
curl http://target.com -H "User-Agent:MyBrowser"
```

- 设置表单数据

```Bash
curl http://target.com -F "username=admin" -F "password=123456"
curl http://target.com -F "file=@/path/to/file"
```

- 设置cookie

```Bash
curl http://target.com -b "character=admin"
```

### HackBar浏览器插件

下载名为HackBar的浏览器插件后，可以按F12在开发者工具中打开
![Alt text](images/http/image-1.png)
图形化界面使得自定义http请求很容易，最后点左上角EXECUTE发送

### Python的requests库

可以写Python脚本来实现http数据包的发送

```Python
import requests
URL = 'http://target.com'
# 参数、表单、文件头、cookie都以字典的形式作为函数参数
param = {
    'id': 1
}
data = {
    'username': 'admin',
    'password': '12345'
}
header = {
    'User-Agent': 'MyBrowser'
    # 'Cookie': 'character=admin' #cookie可以在这里自定义
}
cookie = {
    'character': 'admin'
}

# requests.get、requests.post方法分别用GET方法和POST方法发送http请求,返回response类
response1 = requests.get(URL, params=param, headers=header, cookies=cookie)
repsonse2 = requests.post(URL, data=data, headers=header,cookies=cookie)

# response类的常用属性有response.status_code（网站返回的状态码）,response.text（网站返回的响应体)
print(response1.text)
print(response2.text)
```

### Burp Suite

Burp Suite是一款强大的网络渗透工具，可以通过代理http请求包，修改相关字段，实现自定义数据包，可以自行搜索相关教程

## 例题

现在有了上述基础知识，可以来点例题练练手

### 攻防世界 Web get_post

![Alt text](images/http/image-2.png)

要求1： 请用GET方式提交一个名为a,值为1的变量  
要求2： 请再以POST方式随便提交一个名为b,值为2的变量

用HackBar插件可以轻松秒杀  
![Alt text](images/http/image-6.png)
EXECUTE后可得到flag

### 攻防世界 Web cookie

题目描述：你知道什么是cookie吗？

打开开发者工具，查看cookie
![Alt text](images/http/image-4.png)
访问目录下的cookie.php试试  
出现提示See the http response
![Alt text](images/http/image-5.png)
发现flag

### 2023MoeCTF Web http

题目描述：用GET方法，完成五个任务

1. use parameter: UwU=u
2. post **form**: Luv=u
3. use admin character
4. request from 127.0.0.1
5. use browser 'MoeBrowser'

关于这题我有写一篇详细的writeup[我的writeup](#)
