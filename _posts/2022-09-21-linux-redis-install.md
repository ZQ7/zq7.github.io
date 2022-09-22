---
title: Linux/UNIX安装Redis7.0
author: ZQ7
date: 2022-09-21 18:00:00 +0800
categories: [Linux,Redis]
tags: [Linux]
---

## 

## 源码安装Redis

### 1.下载压缩包

在[Redis](https://redis.io)版本库：https://download.redis.io/releases/ 可根据自己的需求选择下载对应的版本

```shell
[root@localhost ~]# cd /usr/local/src/
[root@localhost ~]# wget https://download.redis.io/releases/redis-7.0.4.tar.gz
[root@localhost ~]# tar zvxf redis-7.0.4.tar.gz
```

### 2.安装

```shell
[root@localhost ~]# yum -y install gcc gcc-c++
[root@localhost ~]# mkdir /usr/local/redis
[root@localhost ~]# cd /usr/local/src/redis-7.0.4
[root@localhost ~]# make
[root@localhost ~]# make install PREFIX=/usr/local/redis
```

> 注意 由于redis是c语言编写的，所以我们需要先安装gcc(yum -y install gcc gcc-c++)

### 3.启动

上述步骤如果没有问题，Redis已经安装在/usr/local/redis/目录下了，相关启动脚本在bin目录下

```shell
[root@localhost ~]# /usr/local/redis/bin/redis-server
```

有启动页面说明启动成功

## 设置服务(service管理)

首先将redis-7.0.4/utils/redis_init_script文件复制到/etc/init.d下，同时命名为redis，查看redis文件，根据相关路径创建启动配置文件(也可自行更改 对应即可)。

```shell
[root@localhost ~]# cp /usr/local/src/redis-7.0.4/utils/redis_init_script /etc/init.d/redis
[root@localhost ~]# cat /etc/init.d/redis
```

部分代码

```shell
...省略...
REDISPORT=6379
EXEC=/usr/local/bin/redis-server
CLIEXEC=/usr/local/bin/redis-cli
PIDFILE=/var/run/redis_${REDISPORT}.pid
CONF="/etc/redis/${REDISPORT}.conf"
...省略... 
```

复制配置文件

```shell
[root@localhost ~]# mkdir /etc/redis
[root@localhost ~]# cp /usr/local/src/redis-7.0.4/redis.conf /etc/redis/6379.conf
```

根据相关需求，编辑/etc/redis/6210.conf配置文件即可
#是否在后台执行，yes：后台运行；no：不是后台运行
daemonize yes
#指定 redis 只接收来自于该 IP 地址的请求，如果不进行设置，那么将处理所有请求
bind * -::*
#配置可以让用户使用AUTH命令来认证密码，才能使用其他命令。
requirepass password

## 加入系统服务

```shell
[root@localhost ~]# vi /lib/systemd/system/redis.service
```

配置代码

```shell
[Unit]
Description=redis
After=network.target

[Service]
Type=forking
PIDFile=/var/run/redis_6379.pid
ExecStart=/usr/local/redis/bin/redis-server /etc/redis/6379.conf
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

刷新配置&启动

```shell
[root@localhost ~]# systemctl daemon-reload
[root@localhost ~]# systemctl start redis
[root@localhost ~]# systemctl enable redis.service
```

参数说明：

[Unit] 表示这是基础信息
Description 是描述
After 是在那个服务后面启动，一般是网络服务启动后启动
[Service] 表示这里是服务信息
ExecStart 是启动服务的命令
ExecReload 是重启服务的指令
ExecStop 是停止服务的指令
[Install] 表示这是是安装相关信息
WantedBy 是以哪种方式启动：multi-user.target表明当系统以多用户方式（默认的运行级别）启动时，这个服务需要被自动运行。



防火墙的设置
firewall-cmd --state        #查看防火墙是否开启
systemctl start firewalld    #开启
systemctl stop firewalld    #关闭
