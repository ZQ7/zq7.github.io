---
title: Nginx安装
author: ZQ7
date: 2022-09-03 15:00:00 +0800
categories: [Linux,Nginx]
tags: [Linux,Nginx]
---



对于Linux，可以使用来自nginx.org的[nginx软件包](http://nginx.org/en/linux_packages.html "nginx软件包")，也可以从源代码编译来安装nginx。

## Nginx编译安装

### 1. 安装前工作

首先更新系统软件源，使用以下命令更新系统 -

```shell
[root@localhost ~]# yum update
```

> 有关两个命令的一点解释：  
> `yum -y update` - 升级所有包，改变软件设置和系统设置,系统版本内核都升级  
> `yum -y upgrade` - 升级所有包，不改变软件设置和系统设置，系统版本升级，内核不改变

**依赖包安装**

```shell
[root@localhost src]# yum -y install gcc gcc-c++ autoconf automake libtool make cmake
[root@localhost src]# yum -y install zlib zlib-devel openssl openssl-devel pcre-devel
```

### 2. 下载Nginx安装源文件

源码下载，可官网下载地址：http://nginx.org/en/download.html 下载并上传到服务器(选择最新稳定版本：`nginx-1.22.0`)


或直接在服务上执行以下命令下载 -

```shell
[root@localhost ~]# cd /usr/local/src
[root@localhost src]# wget -c http://nginx.org/download/nginx-1.22.0.tar.gz
```

解压上面下载的文件 -

```shell
[root@localhost src]# tar zxvf nginx-1.22.0.tar.gz
```

在编译之前还要做一些前期的准备工作，如：依懒包安装，Nginx用户和用户组等。

### 3. 新建nginx用户及用户组

使用 root 用户身份登录系统，执行以下命令创建新的用户。

```shell
[root@localhost src]# groupadd nginx
[root@localhost src]# useradd -g nginx -M nginx
```

`useradd`命令的`-M`参数用于不为`nginx`建立`home`目录  
修改`/etc/passwd`，使得`nginx`用户无法bash登陆(nginx用户后面由`/bin/bash`改为`/sbin/nologin`)，

```shell
[root@localhost src]# vi /etc/passwd
```

然后找到有 nginx 那一行，把它修改为(后面由`/bin/bash`改为`/sbin/nologin`)：

```
nginx:x:1002:1003::/home/nginx:/sbin/nologin
```

### 4. 编译配置、编译、安装

下面我们进入解压的nginx源码目录：`/usr/local/src/` 执行以下命令 -

```
[root@localhost ~]# cd /usr/local/src/nginx*
[root@localhost nginx-1.22.0]# pwd
/usr/local/src/nginx-1.22.0
[root@localhost nginx-1.22.0]#
[root@localhost nginx-1.22.0]# ./configure --prefix=/usr/local/nginx \
--pid-path=/usr/local/nginx/run/nginx.pid \
--with-http_ssl_module \
--user=nginx \
 --group=nginx \
--with-pcre \
--without-mail_pop3_module \
--without-mail_imap_module \
--without-mail_smtp_module
```

> 注意：上面的反斜杠`\` 表示换行继续。

`--prefix=/usr/local/nginx` 指定安装到 `/usr/local/nginx` 目录下。

上面配置完成后，接下来执行编译 -

```shell
[root@localhost nginx-1.22.0]# make
[root@localhost nginx-1.22.0]# make install
... ...
cp conf/nginx.conf '/usr/local/nginx/conf/nginx.conf.default'
test -d '/usr/local/nginx/run' \
        || mkdir -p '/usr/local/nginx/run'
test -d '/usr/local/nginx/logs' \
        || mkdir -p '/usr/local/nginx/logs'
test -d '/usr/local/nginx/html' \
        || cp -R html '/usr/local/nginx'
test -d '/usr/local/nginx/logs' \
        || mkdir -p '/usr/local/nginx/logs'
make[1]: Leaving directory `/usr/local/src/nginx-1.22.0'
[root@localhost nginx-1.22.0]#
```

上面编译时间跟你的电脑配置相关，所以可能需要一些等待时间。

**查看安装后的程序版本：**

```shell
[root@localhost nginx-1.22.0]# /usr/local/nginx/sbin/nginx -v
nginx version: nginx/1.22.0
```

**修改Nginx默认端口(可选)：**

```
[root@localhost nginx-1.22.0]# vi /usr/local/nginx/conf/nginx.conf
```

找到 -

```shell
... ...
   
    server {
        listen       80;
        server_name  localhost;
... ...
```

把上面的 `80` 修改为你想要的端口，如：`8080` 。  
修改配置后验证配置是否合法：

```shell
[root@localhost nginx-1.22.0]# /usr/local/nginx/sbin/nginx -t
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
```

**启动Nginx程序、查看进程** -

```shell
[root@localhost nginx-1.22.0]# /usr/local/nginx/sbin/nginx
[root@localhost nginx-1.22.0]# ps -ef | grep nginx
root      4352     1  0 14:00 ?        00:00:00 nginx: master process /usr/local/nginx/sbin/nginx
nginx     4353  4352  0 14:00 ?        00:00:00 nginx: worker process
root      4735 10590  0 17:01 pts/0    00:00:00 grep --color=auto nginx
[root@localhost nginx-1.22.0]#
```

**nginx停止、重启**  
未添加nginx服务前对nginx的管理只能通过一下方式管理：

```
#  nginx 管理的几种方式 -
# 启动Nginx 
/usr/local/nginx/sbin/nginx 
# 从容停止Nginx：
kill -QUIT 主进程号 # 如上一步中的 ps 命令输出的 4352，就是 Nginx的主进程号
# 快速停止Nginx：
kill -TERM 主进程号
# 强制停止Nginx：
pkill -9 nginx
# 平滑重启nginx
/usr/nginx/sbin/nginx -s reload
```

现在我们来看看安装的Nginx的运行结果，可以简单地使用`curl`命令访问localhost测试，结果如下 -

```
[root@localhost nginx-1.22.0]# curl localhost
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
[root@localhost nginx-1.22.0]#
```

或者也可以打开浏览访问目标服务器的IP+端口号查看。



## 设置Nginx开机自启动

### 进入到/lib/systemd/system/目录

cd /lib/systemd/system/
创建nginx.service文件，并编辑
vi nginx.service
内容：

    [Unit]
    Description=nginx service
    After=network.target
    
    [Service] 
    Type=forking 
    ExecStart=/usr/local/nginx/sbin/nginx
    ExecReload=/usr/local/nginx/sbin/nginx -s reload
    ExecStop=/usr/local/nginx/sbin/nginx -s quit
    PrivateTmp=true
    
    [Install] 
    WantedBy=multi-user.target

### 加入开机自启

设置开机自启

[root@localhost ~]# systemctl enable nginx

取消开机自启

[root@localhost ~]# systemctl disable nginx



### 服务的启动/停止/刷新配置文件/查看状态

systemctl start nginx.service　         启动

systemctl stop nginx.service　          停止

systemctl restart nginx.service　       重启

systemctl list-units --type=service     查看所有已启动的服务

systemctl status nginx.service          查看服务状态

systemctl enable nginx.service          设置开机自启

systemctl disable nginx.service         关闭开机自启
