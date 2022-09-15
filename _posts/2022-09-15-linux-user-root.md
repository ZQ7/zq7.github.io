---
title: 亚马逊服务器开启密码远程，启用root账号
author: ZQ7
date: 2022-09-15 15:00:00 +0800
categories: [Linux,User,Root]
tags: [Linux]
---

有些服务器系统版本初始化之后，默认是没有开启root用户远程登录的，使用中会有一些权限限制，有需要的时候，需要手动开启一下。

## 启用Root账号

### 1.登录服务器

使用默认用户，通过管理后台，或者使用证书(默认用户名)通终端、第三方工具登录服务器

### 2.切换Root用户

```shell
[root@localhost ~]# sudo -i
```

### 3.设置root账户密码

```shell
[root@localhost ~]# passwd root
```

### 4.编辑配置文件

如果vim没有安装需要先安装

```shell
[root@localhost ~]# yum install -y vim
```

辑sshd_config文件

```shell
[root@localhost ~]# vim /etc/ssh/sshd_config
```

PermitRootLogin          直接删除掉前面的#号就开启了root账户登陆

PasswordAuthentication   设置为yes，开启密码登陆

ESC退出编辑模式  :wq保存并退出

### 5.重启ssh

```shell
[root@localhost ~]# service sshd restart
```

ssh -p 22 root@xxx.xxx.xxx.xxx
登录成功~
