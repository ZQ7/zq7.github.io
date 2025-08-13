---
title: CentOS 7 OpenSSL升级
author: ZQ7
date: 2023-09-18 12:00:00 +0800
categories: [Linux,OpenSSL]
tags: [Linux]
---

Centos7默认OpenSSL版本为1.0.2，使用中会出现各种版本兼容性问题，所以需要升级OpenSSL版本。

先来查看openssl版本和备份现有的配置

```shell
[root@localhost ~]# cat /etc/centos-release
CentOS Linux release 7.9.2009 (Core)
[root@localhost ~]# openssl version
OpenSSL 1.0.2k-fips  26 Jan 2017
```

```shell
[root@localhost ~]# find /usr -name openssl
/usr/bin/openssl
[root@localhost ~]# cp -r /usr/bin/openssl /usr/bin/openssl_backup
```

## 安装OpenSSL

### 1.安装依赖包

```shell
[root@localhost ~]# yum install openssl-devel -y
[root@localhost ~]# yum install perl -y
[root@localhost ~]# yum install gcc -y
```

### 2.下载OpenSSL源码文件

官网选择所需要的[openssl](https://www.openssl.org/source/old/)版本的，这里选择的是[openssl-1.1.1v](https://www.openssl.org/source/old/1.1.1/openssl-1.1.1v.tar.gz)

```shell
[root@localhost ~]# cd /usr/local/src/
[root@localhost ~]# wget https://www.openssl.org/source/old/1.1.1/openssl-1.1.1v.tar.gz
```

### 3.安装OpenSSL

```shell
[root@localhost ~]# tar -zxvf openssl-1.1.1v.tar.gz
[root@localhost ~]# cd openssl-1.1.1v
[root@localhost ~]# ./config --prefix=/usr/local/openssl
[root@localhost ~]# make && make install
```

> 命令解释：
> 解压压缩包
> 进入解压后的安装包文件夹
> 配置安装目录
> 编译 安装

### 4.创建软链接

```shell
[root@localhost ~]# rm -rf  /usr/bin/openssl
[root@localhost ~]# ln -s /usr/local/openssl/bin/openssl /usr/bin/openssl
```

### 5.配置动态链接库

```shell
[root@localhost ~]# echo "/usr/local/openssl/lib" >> /etc/ld.so.conf
[root@localhost ~]# ldconfig -v
```
> 命令解释：
> 写入openssl库文件的搜索路径
> 使修改后的/etc/ld.so.conf生效

### 6.验证版本号

```shell
[root@localhost ~]# openssl version
OpenSSL 1.1.1v  1 Aug 2023
```
