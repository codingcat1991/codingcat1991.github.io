---
title: CentOS7下如何设置MySQL开机自启
category: MySQL 
tags:
	- MySQL  
date: 2020-12-29
cover: 1611856804.jpg
top_img: 1611770404.jpg
description: 一般在部署完MySQL的时候都需要设置开机启动，本文讲解CentOS7下如何配置MySQL开机自启。
---

一般在部署完MySQL的时候都需要设置开机启动，本文讲解CentOS7下如何配置MySQL开机自启。

CentOS7和6及以前的版本不一样，下面推荐使用systemctl命令来管理服务而不是以前的service(虽然service还能用)，废话不多说，直接看怎么做吧

### 创建服务文件

```shell
touch /usr/lib/systemd/system/mysqld.service
```

mysqld是服务的名字，可自定义，不过必须以.service结尾。

服务文件内容如下：

```shell
[Unit]
Description=MySQL Server
Documentation=man:mysqld(8)
Documentation=https://dev.mysql.com/doc/refman/5.7/en/using-systemd.html
After=network.target
After=syslog.target

[Install]
WantedBy=multi-user.target

[Service]
User=mysql
Group=mysql
ExecStart=/usr/local/mysql/bin/mysqld --defaults-file=/etc/my.cnf
LimitNOFILE = 5000
```

内容还是通俗易懂的，就不细说啦，需要注意的是ExecStart是启动MySQL的脚本，这里需要使用mysqld文件来启动，而不是使用service命令时的support-files/mysql.server，这里有个好处就是，在多实例又不想使用MySQL官方自带的mysqlmulti的时候，就很方便。

### 配置开机自启

创建好服务文件之后，开机自启就简单了，一条命令即可：

```shell
systemctl enable mysqld
```

同时还可以使用systemctl命令启停MySQL实例，是不是很方便呀，赶紧试试吧！