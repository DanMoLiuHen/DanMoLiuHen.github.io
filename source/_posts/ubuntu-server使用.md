---
title: socket网络编程
date: 2022/11/26 20:14:12
categories: learning-notes
excerpt: 记录在虚拟机上使用ubuntu-server镜像的一些琐事，包括但不限于换源、使用root账户、vim行号设置
hide: false
tag: markdown
---
# ubuntu-server使用
## 换源

使用`vim /etc/apt/sources.list`修改源配置文件，在首行添加以下内容
```bash
# 阿里源
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
```


## 使用root账户
ubuntu初始没有root账户，需要登录后创建，输入指令`sudo passwd root`即可创建root用户，后续登录可以使用root登录，（在root角色下不需要执行`sudo`获取权限）

可以使用`su`进入root用户，退出root用户

若需要禁用root账户，执行即可`sudo passwd -l root`

## vim行号设置
输入`vim`进入vim界面，按`esc`+`:`进入底行模式，输入指令`echo $VIM`查看`vim`的环境变量

我的是`/usr/share/vim`，因此进入该目录修改`vimrc`文件即可，在文件末尾加上`set number`即可永久查看行号

## 时区设置

# 一些指令
`top`指令

