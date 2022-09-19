---
title: Python3安装
author: ZQ7
date: 2022-09-19 12:00:00 +0800
categories: [Linux,Python3]
tags: [Linux]
---

Centos7系统默认安装的python版本为python2.x，目前所开发的Python脚本都是基于3.x，我们要额外安装Python3。

先来查看python安装位置，一般是位于/usr/bin/python目录下

```shell
[root@localhost ~]# which python
/usr/bin/python
```

## 安装Python3

### 1.安装依赖包

```shell
[root@localhost ~]# yum -y groupinstall "Development tools"
[root@localhost ~]# yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel libffi-devel
```

> 注意 安装python3.7以上版本，需要安装libffi-devel

### 2.下载Python3源码文件

官网选择所需要的[Python](https://www.python.org/downloads)版本的，这里用的是Python3.10.6

```shell
[root@localhost ~]# cd /usr/local/src/
[root@localhost ~]# wget https://www.python.org/ftp/python/3.10.6/Python-3.10.6.tgz
```

### 3.安装Python3

```shell
[root@localhost ~]# tar -zxvf Python-3.10.6.tgz
[root@localhost ~]# mkdir /usr/local/python3
[root@localhost ~]# cd Python-3.10.6
[root@localhost ~]# ./configure --prefix=/usr/local/python3
[root@localhost ~]# make && make install
```

解释：

> 解压压缩包
> 创建安装目录
> 进入解压后的安装包文件夹
> 配置安装目录
> 编译 安装

### 4.创建软链接

```shell
[root@localhost ~]# ln -s /usr/local/python3/bin/python3 /usr/bin/python3
[root@localhost ~]# ln -s /usr/local/python3/bin/pip3 /usr/bin/pip3
```

### 5.配置动态链接库

```shell
[root@localhost ~]# echo "/usr/local/python3/lib" >/etc/ld.so.conf.d/python3.conf
```

### 6.验证Python3版本号

```shell
[root@localhost ~]# python3 -V
Python 3.10.6
```
