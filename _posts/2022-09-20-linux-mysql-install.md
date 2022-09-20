---
title: Linux/UNIX安装MySQL
author: ZQ7
date: 2022-09-20 12:00:00 +0800
categories: [Linux,MySQL]
tags: [Linux]
---

Linux/UNIX 上安装 [MySQL](https://dev.mysql.com/downloads/mysql)
Linux平台上推荐使用RPM包来安装Mysql,MySQL AB提供了以下RPM包的下载地址：

MySQL          - MySQL服务器。你需要该选项，除非你只想连接运行在另一台机器上的MySQL服务器。
MySQL-client - MySQL 客户端程序，用于连接并操作Mysql服务器。
MySQL-devel  - 库和包含文件，如果你想要编译其它MySQL客户端，例如Perl模块，则需要安装该RPM包。
MySQL-shared - 该软件包包含某些语言和应用程序需要动态装载的共享库(libmysqlclient.so*)，使用MySQL。
MySQL-bench  - MySQL数据库服务器的基准和性能测试工具。

安装前，我们可以检测系统是否自带安装 MySQL:
rpm -qa | grep mysql
如果你系统有安装，那可以选择进行卸载:

rpm -e mysql　　            // 普通删除模式
rpm -e --nodeps mysql    // 强力删除模式，如果使用上面命令删除时，提示有依赖的其它文件，则用该命令可以对其进行强力删除

## 安装MySQL

### 1.安装依赖包

在 Centos7 系统下使用 yum 命令安装 MySQL，需要注意的是 CentOS7 版本中 MySQL数据库已从默认的程序列表中移除，所以在安装前我们需要先去官网下载 Yum 资源包，下载地址为：https://dev.mysql.com/downloads/repo/yum/

```shell
[root@localhost ~]# cd /usr/local/src/
[root@localhost ~]# wget http://repo.mysql.com/mysql80-community-release-el7-7.noarch.rpm
[root@localhost ~]# rpm -ivh mysql80-community-release-el7-7.noarch.rpm
[root@localhost ~]# yum update
[root@localhost ~]# yum install mysql-server
```

### 2.权限设置

```shell
[root@localhost ~]# chown -R mysql:mysql /var/lib/mysql/
```

> 注意 启动MySQL如果遇到权限不足的情况，
> cat /etc/my.cnf // 查看mysqld配置

datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

根据log文件路径，查看错误日志，调整相关权限
例 chown -R mysql:mysql /var/run/mysqld/mysqld.pid

### 3.启动 MySQL

初始化root用户的密码已设置并存储在错误日志文件中，查看并自行保存初始密码
grep 'temporary password' /var/log/mysqld.log 

```shell
[root@localhost ~]# mysqld --initialize
[root@localhost ~]# systemctl start mysqld
[root@localhost ~]# systemctl status mysqld
```

命令解释：

> 初始化 MySQL
> 启动   MySQL
> 查看 MySQL 运行状态

### 4.初始化

MySQL服务器初始化（从MySQL 5.7开始）

在 MySQL 服务器初始启动时，如果服务器的数据目录为空，则会发生以下情况：

- MySQL 服务器已初始化。
- 在数据目录中生成SSL证书和密钥文件。
- 安装并启用该 validate_password 插件。
- 将创建一个超级用户 帐户’root’@‘localhost’。并会设置超级用户的密码，将其存储在错误日志文件/var/log/mysqld.log中。

```shell
[root@localhost ~]# grep 'temporary password' /var/log/mysqld.log
[root@localhost ~]# mysql -uroot -p
[root@localhost ~]# ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass4!';
```

命令解释：

> 查看初始化密码
> root用户登录(使用初始化密码)
> 修改root新密码
> 求密码至少包含1个大写字母、1个小写字母、1个数字和1个特殊字符，密码总长度至少为8个字符。

### 5.验证MySQL安装

```shell
[root@localhost ~]# mysqladmin --version
mysqladmin  Ver 8.0.30 for Linux on x86_64 (MySQL Community Server - GPL)
```

### 6.登录 MySQL

当 MySQL 服务已经运行时, 我们可以通过 MySQL 自带的客户端工具登录到 MySQL 数据库中, 首先打开命令提示符, 输入以下格式的命名:

mysql -h 主机名 -u 用户名 -p
参数说明：

-h : 指定客户端所要登录的 MySQL 主机名, 登录本机(localhost 或 127.0.0.1)该参数可以省略;
-u : 登录的用户名;
-p : 告诉服务器将会使用一个密码来登录, 如果所要登录的用户名密码为空, 可以忽略此选项。
如果我们要登录本机的 MySQL 数据库，只需要输入以下命令即可：

mysql -u root -p
按回车确认, 如果安装正确且 MySQL 正在运行, 会得到以下响应:

Enter password:
若密码存在, 输入密码登录, 不存在则直接按回车登录。登录成功后你将会看到 Welcome to the MySQL monitor... 的提示语。

然后命令提示符会一直以 mysql> 加一个闪烁的光标等待命令的输入, 输入 exit 或 quit 退出登录。

## 重置MySQL密码

### 1.无密码登陆

编辑mysql配置文件:my.cnf，
vim /etc/my.cnf

在[mysqld]配置下添加：
skip-grant-tables   
保存退出；

### 2.使配置生效

重启mysql服务
service mysqld restart；

### 3.将旧密码置空

1. 连接mysql
    mysql -u root -p 
    提示输入密码时直接敲回车。
2. 选择数据库
    use mysql
3. 将密码置空
    update user set authentication_string = '' where user = 'root';
4. 退出
    quit

### 4.去除免密码登陆

删掉步骤1的语句(或者以#注释)'skip-grant-tables'

重启服务  service mysqld restart

### 5.修改密码

mysql -u root -p
提示输入密码时直接敲回车，刚刚已经将密码置空了

ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass4!';
此句命令若报错， 可能是localhost在实际应用中作了修改，可以改为以下命令行 

ALTER USER 'root'@'%' IDENTIFIED BY 'MyNewPass4!';
密码形式过于简单则会报错
