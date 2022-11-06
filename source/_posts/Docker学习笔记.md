---
title: 云原生学习笔记
date: 2022/9/26 20:14:12
categories: learning-notes
excerpt: 主要记录作者学习云原生的笔记，笔记类主要是给作者看，因此访客不必在笔记上花费太多时间阅读理解
hide: false
tag: markdown
---
# 概述：

# docker
## 常用指令
运行在虚拟机ubuntu20.04上
```bash
# 构建容器镜像，.表示在当前文件夹内
docker build -t getting-started .

# docker run运行docker，-d在后台运行，-p 6379:6379，将主机的端口80映射到容器中的端口80
# --name给容命名
docker run -d -p 6379:6379 --name redis redis:lastest
```
更新源码后，需要先删除旧容器，再启动新容器
```bash
# 获取容器的 ID 
docker ps

# 使用docker stop命令停止容器。
# 用docker ps查询的ip更换<the-container-id>
docker stop <the-container-id>

# 容器停止后，使用docker rm命令将其删除
docker rm <the-container-id>

# 将停止和删除命令合并，通过force指令
docker rm -f <the-container-id>

# 启动更新后的容器
docker run -dp 3000:3000 getting-started
```

使用卷实现持久化数据，更换容器后被存储的数据不会改变
```bash
# 使用docker volume create命令创建卷
docker volume create todo-db

# 再次停止并删除应用程序容器docker rm -f <id>，因为它仍在运行而不使用持久

# 启动 todo 应用程序容器，添加-v标志以指定卷安装，使用命名卷并将其挂载到/etc/todos，捕获在该路径创建的所有文件
docker run -dp 3000:3000 -v todo-db:/etc/todos getting-started
```

docker compose使用
```shell
# 检查是否安装上
docker compose version

# 启动应用程序栈，-d在后台运行所有内容
docker compose up -d
```

## 安装与使用镜像

1. 找镜像。在[docker hub](https://hub.docker.com/)找所需的镜像，找不同版本可以点击tag查找所需版本
```bash
# 下载最新版本
docker pull nginx

docker pull nginx:1.20.1

# 查看所有镜像
docker images

# 删除镜像
docker rmi 镜像名:版本号/镜像id

docker rm 镜像
```

2. 启动容器
```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
# 常用的OPTIONS
# OPTIONS:开机自启
--restart-always 
# OPTIONS:端口映射
-p 88:80

# 查看正在运行的容器
docker ps
# 查看所有
docker ps -a 
# 删除停止的容器
docker rm 容器id/名称
```

3. 进入容器修改内容
```bash
# 进入容器内部
docker exec -it 容器id /bin/bash
```

4. 提交改变
```bash
# 总命令,，打包成一个镜像，细节使用--help查看
docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]

# 常用形式
docker commit -a "作者" -m "提交变化" 容器id 自定义名称
```
5. 远程传输

- 打包成一个压缩包tar后传输给另一个机器，另一个机器再解压运行即可
```bash
# 打包成一个压缩包tar后传输
docker save -o 自定义名称 镜像

# 对压缩包解压运行即可
docker load -i 上一步定义的名称
```
- 推送到远程仓库，需要先`docker login`登录(`docker logout`登出)
```bash
docker tag <local-image:tagname> <new-repo:tagname>
docker push <new-repo:tagname>
```
6. 将数据挂载到外部
```bash
docker run --name some-nginx -v /some/content:/usr/share/nginx/html:ro -d nginx
# --name some-nginx 定义名称
# -v /some/content:/usr/share/nginx/html:ro 将/usr/share/nginx/html与主机/some/content目录挂载，且为只读模式
```
挂载一个redis
```bash
docker run -v /data/redis/redis.conf:/etc/redis/redis.conf \
-v /data/redis/data:/data  \
-d --name myredis  \
-p 6379:6379 \
redis:latest redis-server /etc/redis/redis.conf
# 指令解释
# -v 进行挂载，将本地/data/redis/redis.conf与容器/etc/redis/redis.conf相挂载
# redis-server /etc/redis/redis.conf 启动自定义配置文件,指定配置文件必须是容器内的
```
7. 杂项
```bash
# 查看日志
docker logs 容器id

# 把容器指定位置的东西复制出来，将命令返回来是将外部复制到容器内
docker cp 容器id:文件绝对路径 外部路径
```