---
title: Docker笔记
date: 2023-11-28 15:57:12
tags: docker
categories: Tool
---

## Docker是什么

Docker是一种应用容器，docker容器内部是运行独立的环境。与虚拟机相比，docker可移植性高，性能开销低。开发者可以将他们的应用打包在docker中，打包，然后快速发布到不同环境的主机上中。

## Docker架构

由三部分组成

- Clients  
  用户通过客户端向Docker Host 发送命令
- Hosts  
  以daemon守护进程的形式存在，以root权限运行，可以完成普通用户无法执行的命令如挂载文件系统。daemon下又有两个内容：images和container，两者如类和实例的关系，docker可以由镜像开启相应的一个容器。
- Registries  
  仓库，用于存放docker images。

## Docker基本命令

### 服务相关

- 启动docker服务`systemctl start docker`
- 停止docker服务`systemctl stop docker`
- 查看docker服务`systemctl status docker`
- 重启docker服务`systemctl restart docker`
- 开机启动docker`systemctl enable docker`

### 镜像相关

- 查看本地镜像`docker images`
- 搜索仓库镜像`docker search <image-name>`
- 拉取远程镜像`docker pull <image-name>:<version>`
- 删除本地镜像`docker rmi <image-name>:<version>`或`docker rmi <image-ID>`
- 删除所有本地镜像`docker rmi ˋdocker images -qˋ`

### 容器相关

- 创建并运行容器 `docker run -it --name=<docker-name> <image-name>:<version> /bin/bash`
  1. -it 表示交互和伪终端，创建交互式容器
  2. -id 以守护模式运行容器, 创建守护式容器，用exec交互，用stop停止运行
  3. /bin/bash进入shell  
  4. exit 可以退出docker环境
- 查看容器`docker ps -a`  a表示all
- 进入容器环境`docker exec -it <docker-name> /bin/bash`
- 停止运行容器`docker stop <docker-name>`
- 运行容器`docker start <docker-name>`
- 删除容器`docker rm <docker-name>`或`docker rm <docker-id>`
- 删除所有容器`docker rm ˋdocker ps -aqˋ`
- 查看容器信息`docker inspect <docker-name>`
- 容器与主机之间的数据拷贝`docker cp <path> <path>`, 容器的路径前加上`容器名:`

## 数据卷

### 概念&作用

- 数据卷是宿主机的文件或目录
- 容器目录和数据卷绑定后，两边数据修改会同步
- 一个数据卷可以被多个容器同时挂载

### 配置

`docker run`的-v选项：`-v <real-route>:<docker-route>`  
`<real-route>`不存在则会自动创建  
**一个容器可以挂载多个目录**

```Bash
docker run -it --name=hello hello-docker \
-v <real-route1>:<docker-route1> \
-v <real-route2>:<docker-route2> \
-v ... \
-v <real-routen>:<docker-routen> \
```

### 数据卷容器

```Bash
# 创建数据卷容器
docker run --name=z -v /volume -it <docker-name> /bin/bash
# 创建容器绑定数据卷容器
docker run --name=x -v --volumes-from z <docker-name> /bin/bash
```

## 应用部署

### 步骤

1. 搜索镜像`docker search <docker-name>`
2. 拉取镜像`docker pull <docker-name>`
3. 创建容器，映射目录、端口

    ```Bash
    docker run -id \
    -p <host-port>:<docker-port> \
    ...
    ```

### MySQL部署

```Bash
# 创建目录
makedir mysql
# 进入目录
cd mysql
# 创建容器
docker run \
--name testsql \
-d \
-p 3306:3306 \
--restart unless-stopped \
-v $PWD/log:/var/log/mysql \
-v $PWD/data:/var/lib/mysql \
-v $PWD/conf:/etc/mysql/conf.d \
# 初始化mysql密码
-e MYSQL_ROOT_PASSWORD=123456 \
mysql:8.0
```

### Nginx部署

```Bash
# 创建挂载目录
mkdir -p /home/nginx/conf
mkdir -p /home/nginx/log
mkdir -p /home/nginx/html
# 将nginx配置文件复制到宿主机
docker run --name temp_nginx -p 80:80 -d nginx
docker cp temp_nginx:/etc/nginx/nginx.conf /home/nginx/conf/nginx.conf
docker cp temp_nginx:/etc/nginx/conf.d /home/nginx/conf/conf.d
docker cp temp_nginx:/usr/share/nginx/html /home/nginx/
docker rm -f temp_nginx
# 创建容器
docker run \
-p 80:80 \
--name=nginx \
-v /home/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
-v /home/nginx/conf/conf.d:/etc/nginx/conf.d \
-v /home/nginx/log:/var/log/nginx \
-v /home/nginx/html:/usr/share/nginx/html \
-d nginx
```

## 容器转为镜像

```Bash
# 容器转镜像
docker commit <docker-id> <image-name>:<version>
# 镜像转压缩文件
docker save -o <tar-filename> <image-name>:<version>
# 压缩文件转镜像
docker load -i <tar-filename> 
```

## Dockerfile

### 关键字

- **FROM** 指定父镜像
- **MAINTAINER** 作者信息
- **LABEL** docker标签
- **RUN** 执行命令，格式有RUN command p1 p2和["command","p1","p2"]
- **CMD** 容器启动时执行的命令，格式同RUN
- **COPY** 复制文件
- **ADD** 添加文件，支持远程文件下载  
- **ENV** 环境变量
等等

### Dockerfile构建镜像

```Bash
docker build -f <your-dockerfile-route> -t <image-name>:<version> .
# 若在Dockerfile目录下执行该指令且Dockerfile文件名就是Dockerfile,则可省略-f选项
```

## Docker Compose

编排多容器分布式部署的工具。

### 安装

```Bash
# 安装
sudo curl -L https://github.com/docker/compose/releases/download/v2.21.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
# 设置权限
sudo chmod +x /usr/local/bin/docker-compose
```

### 配置docker-compose.yml
