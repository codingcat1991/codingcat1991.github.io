---
title: InnoDB Cluster部署
date: 2021-06-10 16:22:15
category: MySQL
tags:
  - MySQL
  - InnoDB Cluster
keywords: 'InnoDB Cluster,MySQL'
top_img: bg.jpg
cover: bg.jpg
description: 部署一个MySQL 8.0.25 InnoDB Cluster集群
---

部署一个MySQL 8.0.25 InnoDB Cluster集群

## InnoDB Cluster集群简介

![InnoDB Cluster架构](InnoDB Cluster架构.jpg)

## 环境信息

准备三台虚拟机

|                     | 服务器1            | 服务器2            | 服务器3            | 端口                 |
| ------------------- | ------------------ | ------------------ | ------------------ | -------------------- |
| 操作系统            | CentOS7            | CentOS7            | CentOS7            | /                    |
| IP                  | 192.168.165.156    | 192.168.165.157    | 192.168.165.225    | /                    |
| 主机名              | db156              | db156              | db225              | /                    |
| MySQL Server 8.0.25 | MGR 节点一         | MGR 节点二         | MGR 节点三         | 3306                 |
| MySQL Shell 8.0.25  | MySQL Shell节点一  | MySQL Shell节点二  | MySQL Shell节点三  | /                    |
| MySQL Router 8.0.25 | MySQL Router节点一 | MySQL Router节点二 | MySQL Router节点三 | 6446(写)、6447（读） |

## 部署说明

### 安装顺序

安装MySQL Server--->启用组复制--->安装MySQL Shell ---> 创建InnoDB Cluster集群---> 安装MySQL Router

### 安装包

| 安装包名                                            | MD5值                            | 下载连接                                                     |
| --------------------------------------------------- | -------------------------------- | ------------------------------------------------------------ |
| mysql-8.0.25-linux-glibc2.12-x86_64.tar.xz          | a12142a969531cebd845844620bb44a4 | https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.25-linux-glibc2.12-x86_64.tar.xz |
| mysql-shell-8.0.25-linux-glibc2.12-x86-64bit.tar.gz | 9192ea694ac97a5c5715806ddcf84932 | https://dev.mysql.com/get/Downloads/MySQL-Shell/mysql-shell-8.0.25-linux-glibc2.12-x86-32bit.tar.gz |
| mysql-router-8.0.25-linux-glibc2.12-x86_64.tar.xz   | bcd9b81f422667dbabd0a5d7bbf7638a | https://dev.mysql.com/get/Downloads/MySQL-Router/mysql-router-8.0.25-linux-glibc2.12-x86_64.tar.xz |

## 安装MySQL Server

### 安装依赖包

### 创建相关用户

```shell
[root@db156 ~]# groupadd mysql && useradd -r -g mysql -s /bin/false mysql
groupadd: group 'mysql' already exists
[root@db156 ~]# cat /etc/passwd | grep -iw  "mysql"
mysql:x:997:1000::/home/mysql:/bin/false
```

### 下载并解压安装包

```
[root@db156 opt]# wget https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.25-linux-glibc2.12-x86_64.tar.xz
--2021-06-10 04:43:21--  https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.25-linux-glibc2.12-x86_64.tar.xz
Resolving dev.mysql.com (dev.mysql.com)... 137.254.60.11
Connecting to dev.mysql.com (dev.mysql.com)|137.254.60.11|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://cdn.mysql.com//Downloads/MySQL-8.0/mysql-8.0.25-linux-glibc2.12-x86_64.tar.xz [following]
--2021-06-10 04:43:22--  https://cdn.mysql.com//Downloads/MySQL-8.0/mysql-8.0.25-linux-glibc2.12-x86_64.tar.xz
Resolving cdn.mysql.com (cdn.mysql.com)... 104.71.212.231
Connecting to cdn.mysql.com (cdn.mysql.com)|104.71.212.231|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 896219776 (855M) [text/plain]
Saving to: ‘mysql-8.0.25-linux-glibc2.12-x86_64.tar.xz’

100%[==================================================================================================================================================================>] 896,219,776 14.2MB/s   in 66s    

2021-06-10 04:44:29 (13.0 MB/s) - ‘mysql-8.0.25-linux-glibc2.12-x86_64.tar.xz’ saved [896219776/896219776]

[root@db156 opt]# wget https://dev.mysql.com/get/Downloads/MySQL-Shell/mysql-shell-8.0.25-linux-glibc2.12-x86-32bit.tar.gz
--2021-06-10 04:44:39--  https://dev.mysql.com/get/Downloads/MySQL-Shell/mysql-shell-8.0.25-linux-glibc2.12-x86-32bit.tar.gz
Resolving dev.mysql.com (dev.mysql.com)... 137.254.60.11
Connecting to dev.mysql.com (dev.mysql.com)|137.254.60.11|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://cdn.mysql.com//Downloads/MySQL-Shell/mysql-shell-8.0.25-linux-glibc2.12-x86-32bit.tar.gz [following]
--2021-06-10 04:44:41--  https://cdn.mysql.com//Downloads/MySQL-Shell/mysql-shell-8.0.25-linux-glibc2.12-x86-32bit.tar.gz
Resolving cdn.mysql.com (cdn.mysql.com)... 104.71.212.231
Connecting to cdn.mysql.com (cdn.mysql.com)|104.71.212.231|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 41487987 (40M) [application/x-tar-gz]
Saving to: ‘mysql-shell-8.0.25-linux-glibc2.12-x86-32bit.tar.gz’

100%[==================================================================================================================================================================>] 41,487,987  11.6MB/s   in 3.8s   

2021-06-10 04:44:45 (10.4 MB/s) - ‘mysql-shell-8.0.25-linux-glibc2.12-x86-32bit.tar.gz’ saved [41487987/41487987]

[root@db156 opt]# wget https://dev.mysql.com/get/Downloads/MySQL-Router/mysql-router-8.0.25-linux-glibc2.12-x86_64.tar.xz
--2021-06-10 04:44:53--  https://dev.mysql.com/get/Downloads/MySQL-Router/mysql-router-8.0.25-linux-glibc2.12-x86_64.tar.xz
Resolving dev.mysql.com (dev.mysql.com)... 137.254.60.11
Connecting to dev.mysql.com (dev.mysql.com)|137.254.60.11|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://cdn.mysql.com//Downloads/MySQL-Router/mysql-router-8.0.25-linux-glibc2.12-x86_64.tar.xz [following]
--2021-06-10 04:44:54--  https://cdn.mysql.com//Downloads/MySQL-Router/mysql-router-8.0.25-linux-glibc2.12-x86_64.tar.xz
Resolving cdn.mysql.com (cdn.mysql.com)... 104.71.212.231
Connecting to cdn.mysql.com (cdn.mysql.com)|104.71.212.231|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 65288920 (62M) [text/plain]
Saving to: ‘mysql-router-8.0.25-linux-glibc2.12-x86_64.tar.xz’

100%[==================================================================================================================================================================>] 65,288,920  12.5MB/s   in 6.1s   

2021-06-10 04:45:01 (10.3 MB/s) - ‘mysql-router-8.0.25-linux-glibc2.12-x86_64.tar.xz’ saved [65288920/65288920]

[root@db156 opt]# tar -xf mysql-8.0.25-linux-glibc2.12-x86_64.tar.xz
[root@db156 opt]# mv mysql-8.0.25-linux-glibc2.12-x86_64 /usr/local/mysql/
[root@db156 opt]# chown -R mysql:mysql /usr/local/mysql/
[root@db156 opt]# chmod -R 755 /usr/local/mysql/
```

### 创建相关目录

```
[root@db156 opt]# mkdir -p /data/mysql/{data,binlog,relay}
[root@db156 opt]# chown -R mysql:mysql /data/mysql/
[root@db156 opt]# chmod -R 750 /data/mysql/
```



### 准备配置文件

```
[mysqld]
server_id=13306
gtid_mode=on
enforce_gtid_consistency=1
transaction_write_set_extraction=XXHASH64
loose-group_replication_group_name='aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa'
loose-group_replication_start_on_boot=off
loose-group_replication_ip_whitelist='192.168.165.0/24'
loose-group_replication_local_address='192.168.165.157:33061'
loose-group_replication_group_seeds='192.168.165.156:33061,192.168.165.157:33061,192.168.165.225:33061'
loose-group_replication_bootstrap_group=off
loose-group_replication_single_primary_mode=on
loose-group_replication_enforce_update_everywhere_checks=off
loose-group_replication_recovery_get_public_key=on
loose-group_replication_recovery_use_ssl=off
loose-group_replication_ssl_mode='DISABLED'
loose-group_replication_consistency='EVENTUAL'
loose-group_replication_member_expel_timeout=5
report_host='192.168.165.157'
report_port=3306
plugin_load_add ='group_replication.so'
auto_increment_increment=1
auto_increment_offset=1
mysqlx_port=33060
admin_port=33062
datadir=/data/mysql/data
basedir=/usr/local/mysql
user=mysql
```

### 初始化MySQL Server

```
[root@db156 opt]# /usr/local/mysql/bin/mysqld --defaults-file=/etc/my.cnf --initialize-insecure
2021-06-10T08:12:00.192163Z 0 [System] [MY-013169] [Server] /usr/local/mysql/bin/mysqld (mysqld 8.0.25) initializing of server in progress as process 10863
2021-06-10T08:12:00.364089Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2021-06-10T08:12:05.120563Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2021-06-10T08:12:05.548074Z 0 [Warning] [MY-013501] [Server] Ignoring --plugin-load[_add] list as the server is running with --initialize(-insecure).
2021-06-10T08:12:06.717610Z 0 [Warning] [MY-000067] [Server] unknown variable 'loose-group_replication_group_name=aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa'.
2021-06-10T08:12:06.718217Z 0 [Warning] [MY-000067] [Server] unknown variable 'loose-group_replication_start_on_boot=off'.
2021-06-10T08:12:06.718823Z 0 [Warning] [MY-000067] [Server] unknown variable 'loose-group_replication_ip_whitelist=192.168.165.0/24'.
2021-06-10T08:12:06.719416Z 0 [Warning] [MY-000067] [Server] unknown variable 'loose-group_replication_local_address=192.168.165.156:33061'.
2021-06-10T08:12:06.719969Z 0 [Warning] [MY-000067] [Server] unknown variable 'loose-group_replication_group_seeds=192.168.165.156:33061,192.168.165.157:33061,192.168.165.225:33061'.
2021-06-10T08:12:06.720542Z 0 [Warning] [MY-000067] [Server] unknown variable 'loose-group_replication_bootstrap_group=off'.
2021-06-10T08:12:06.721103Z 0 [Warning] [MY-000067] [Server] unknown variable 'loose-group_replication_single_primary_mode=on'.
2021-06-10T08:12:06.721683Z 0 [Warning] [MY-000067] [Server] unknown variable 'loose-group_replication_enforce_update_everywhere_checks=off'.
2021-06-10T08:12:06.722250Z 0 [Warning] [MY-000067] [Server] unknown variable 'loose-group_replication_recovery_get_public_key=on'.
2021-06-10T08:12:06.722873Z 0 [Warning] [MY-000067] [Server] unknown variable 'loose-group_replication_recovery_use_ssl=off'.
2021-06-10T08:12:06.723485Z 0 [Warning] [MY-000067] [Server] unknown variable 'loose-group_replication_ssl_mode=DISABLED'.
2021-06-10T08:12:06.724060Z 0 [Warning] [MY-000067] [Server] unknown variable 'loose-group_replication_member_expel_timeout=5'.
2021-06-10T08:12:06.727059Z 6 [Warning] [MY-010453] [Server] root@localhost is created with an empty password ! Please consider switching off the --initialize-insecure option.
```



### 配置启动文件并启动MySQL

```
[root@db156 opt]# vi /usr/lib/systemd/system/mysqld.service
[root@db156 opt]# cat /usr/lib/systemd/system/mysqld.service
[Unit]
Description=MySQL Server
Documentation=man:mysqld(8)
Documentation=https://dev.mysql.com/doc/refman/8.0/en/using-systemd.html
After=network.target
After=syslog.target

[Install]
WantedBy=multi-user.target

[Service]
User=mysql
Group=mysql
ExecStart=/usr/local/mysql/bin/mysqld_safe --defaults-file=/etc/my.cnf
LimitNOFILE = 5000
[root@db156 opt]# systemctl start mysqld 
[root@db156 opt]# systemctl status mysqld      
● mysqld.service - MySQL Server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; disabled; vendor preset: disabled)
   Active: active (running) since Thu 2021-06-10 04:57:12 EDT; 10s ago
     Docs: man:mysqld(8)
           https://dev.mysql.com/doc/refman/8.0/en/using-systemd.html
 Main PID: 14232 (mysqld_safe)
   CGroup: /system.slice/mysqld.service
           ├─14232 /bin/sh /usr/local/mysql/bin/mysqld_safe --defaults-file=/etc/my.cnf
           └─14626 /usr/local/mysql/bin/mysqld --defaults-file=/etc/my.cnf --basedir=/usr/local/mysql --datadir=/data/mysql/data --plugin-dir=/usr/local/mysql/lib/plugin --log-error=db156.err --pid-fil...

Jun 10 04:57:12 db156 systemd[1]: Started MySQL Server.
Jun 10 04:57:13 db156 mysqld_safe[14232]: 2021-06-10T08:57:13.594106Z mysqld_safe Logging to '/data/mysql/data/db156.err'.
Jun 10 04:57:13 db156 mysqld_safe[14232]: 2021-06-10T08:57:13.689214Z mysqld_safe Starting mysqld daemon with databases from /data/mysql/data
```

在执行`systemctl start mysqld `如果报错`Failed to start mysqld.service: Unit not found.`,可以执行`systemctl daemon-reload`解决。

### 修改本地账号密码并设置不过期

```
[root@db156 opt]# /usr/local/mysql/bin/mysql -uroot -S /tmp/mysql.sock -e "SET SQL_LOG_BIN=0;SET global super_read_only=OFF; SET global read_only=OFF; alter user 'root'@'localhost' password expire never; set password for 'root'@'localhost'='123456';flush privileges; SET SQL_LOG_BIN=1;"
```

### 创建复制账号并授权

```
[root@db156 opt]# /usr/local/mysql/bin/mysql -uroot -S /tmp/mysql.sock -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 8.0.25 MySQL Community Server - GPL

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> SET SQL_LOG_BIN=0;
Query OK, 0 rows affected (0.00 sec)

mysql> CREATE USER IF NOT EXISTS `repl`@`192.168.165.156` IDENTIFIED BY '123456' PASSWORD EXPIRE NEVER;
CREATE USER IF NOT EXISTS `repl`@`192.168.165.157` IDENTIFIED BY '123456' PASSWORD EXPIRE NEVER;
CREATE USER IF NOT EXISTS `repl`@`192.168.165.225` IDENTIFIED BY '123456' PASSWORD EXPIRE NEVER;
Query OK, 0 rows affected (0.04 sec)

mysql> CREATE USER IF NOT EXISTS `repl`@`192.168.165.157` IDENTIFIED BY '123456' PASSWORD EXPIRE NEVER;
Query OK, 0 rows affected (0.02 sec)

mysql> CREATE USER IF NOT EXISTS `repl`@`192.168.165.225` IDENTIFIED BY '123456' PASSWORD EXPIRE NEVER;
GRANT REPLICATION SLAVE, REPLICATION CLIENT, BACKUP_ADMIN ON *.* TO `repl`@`192.168.165.156`; 
Query OK, 0 rows affected (0.01 sec)

mysql> GRANT REPLICATION SLAVE, REPLICATION CLIENT, BACKUP_ADMIN ON *.* TO `repl`@`192.168.165.156`; 
Query OK, 0 rows affected (0.01 sec)

mysql> GRANT REPLICATION SLAVE, REPLICATION CLIENT, BACKUP_ADMIN ON *.* TO `repl`@`192.168.165.157`; 
Query OK, 0 rows affected (0.01 sec)

mysql> GRANT REPLICATION SLAVE, REPLICATION CLIENT, BACKUP_ADMIN ON *.* TO `repl`@`192.168.165.225`; 
Query OK, 0 rows affected (0.00 sec)

mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.01 sec)

mysql> SET SQL_LOG_BIN=1;
Query OK, 0 rows affected (0.00 sec)

mysql> exit
Bye
[root@db156 opt]# 
```

### 创建管理账号并授权

```
[root@db156 opt]# /usr/local/mysql/bin/mysql -uroot -S /tmp/mysql.sock -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.25 MySQL Community Server - GPL

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> SET SQL_LOG_BIN=0;
Query OK, 0 rows affected (0.00 sec)

mysql> CREATE USER IF NOT EXISTS `icadmin`@`192.168.165.%` IDENTIFIED BY '123456' PASSWORD EXPIRE NEVER;
Query OK, 0 rows affected, 1 warning (0.01 sec)

mysql> GRANT SELECT, RELOAD, SHUTDOWN, PROCESS, FILE, SUPER, EXECUTE, REPLICATION SLAVE, REPLICATION CLIENT, CREATE USER ON *.* TO `icadmin`@`192.168.165.%` WITH GRANT OPTION;
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> GRANT BACKUP_ADMIN,CLONE_ADMIN,PERSIST_RO_VARIABLES_ADMIN,REPLICATION_APPLIER,SYSTEM_VARIABLES_ADMIN ON *.* TO `icadmin`@`192.168.165.%` WITH GRANT OPTION;
GRANT INSERT, UPDATE, DELETE ON `mysql`.* TO `icadmin`@`192.168.165.%` WITH GRANT OPTION;
Query OK, 0 rows affected (0.01 sec)

mysql> GRANT INSERT, UPDATE, DELETE ON `mysql`.* TO `icadmin`@`192.168.165.%` WITH GRANT OPTION;
Query OK, 0 rows affected (0.00 sec)

mysql> GRANT INSERT, UPDATE, DELETE, CREATE, DROP, REFERENCES, INDEX, ALTER, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, EVENT, TRIGGER ON `mysql_innodb_cluster_metadata`.* TO `icadmin`@`192.168.165.%` WITH GRANT OPTION;
Query OK, 0 rows affected (0.00 sec)

mysql> GRANT INSERT, UPDATE, DELETE, CREATE, DROP, REFERENCES, INDEX, ALTER, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, EVENT, TRIGGER ON `mysql_innodb_cluster_metadata_bkp`.* TO `icadmin`@`192.168.165.%` WITH GRANT OPTION;
Query OK, 0 rows affected (0.00 sec)

mysql> GRANT INSERT, UPDATE, DELETE, CREATE, DROP, REFERENCES, INDEX, ALTER, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, EVENT, TRIGGER ON `mysql_innodb_cluster_metadata_previous`.* TO `icadmin`@`192.168.165.%` WITH GRANT OPTION;
FLUSH PRIVILEGES;
SET SQL_LOG_BIN=1;
Query OK, 0 rows affected (0.32 sec)

mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)

mysql> SET SQL_LOG_BIN=1;
Query OK, 0 rows affected (0.00 sec)

mysql> exit
Bye
[root@db156 opt]# 
```



### 安装组复制插件

```
[root@db156 opt]# /usr/local/mysql/bin/mysql -uroot -S /tmp/mysql.sock -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 8.0.25 MySQL Community Server - GPL

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> INSTALL PLUGIN group_replication SONAME 'group_replication.so';
ERROR 1125 (HY000): Function 'group_replication' already exists
mysql> SELECT PLUGIN_NAME, PLUGIN_STATUS FROM INFORMATION_SCHEMA.PLUGINS  WHERE PLUGIN_NAME LIKE '%group_replication%';
+-------------------+---------------+
| PLUGIN_NAME       | PLUGIN_STATUS |
+-------------------+---------------+
| group_replication | ACTIVE        |
+-------------------+---------------+
1 row in set (0.00 sec)

mysql> exit
Bye
[root@db156 opt]# 
```



### 设置复制通道

```
[root@db156 opt]# /usr/local/mysql/bin/mysql -uroot -S /tmp/mysql.sock -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 10
Server version: 8.0.25 MySQL Community Server - GPL

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> CHANGE MASTER TO MASTER_USER='repl', MASTER_PASSWORD='123456' FOR CHANNEL 'group_replication_recovery';
Query OK, 0 rows affected, 5 warnings (0.01 sec)

mysql> SELECT User_name,Channel_name FROM mysql.slave_master_info WHERE user_name = 'repl';
+-----------+----------------------------+
| User_name | Channel_name               |
+-----------+----------------------------+
| repl      | group_replication_recovery |
+-----------+----------------------------+
1 row in set (0.00 sec)

mysql> SET global group_replication_recovery_get_public_key=ON;
Query OK, 0 rows affected (0.00 sec)

mysql> exit
Bye
[root@db156 opt]# 
```



### 查看MySQL进程

```
[root@db156 opt]# ps -ef | grep mysql
mysql    14232     1  0 04:57 ?        00:00:00 /bin/sh /usr/local/mysql/bin/mysqld_safe --defaults-file=/etc/my.cnf
mysql    14626 14232  0 04:57 ?        00:00:03 /usr/local/mysql/bin/mysqld --defaults-file=/etc/my.cnf --basedir=/usr/local/mysql --datadir=/data/mysql/data --plugin-dir=/usr/local/mysql/lib/plugin --log-error=db156.err --pid-file=db156.pid
root     15500  9686  0 05:12 pts/0    00:00:00 grep --color=auto mysql
[root@db156 opt]# 
```



> 说明：其他两个节点也按照以上步骤部署

## 启用MGR组复制

三个MySQL Server实例部署完成之后，就可以启用组复制了。

对于组复制，数据必须存储在InnoDB事务存储引擎中，使用其他存储引擎（包括临时 MEMORY存储引擎）可能会导致组复制中的错误。disabled_storage_engines 如下设置 系统变量以防止其使用：

disabled_storage_engines="MyISAM,BLACKHOLE,FEDERATED,ARCHIVE,MEMORY"

修改配置文件，添加上面的配置并重启即可：

```
[root@db156 opt]# vi /etc/my.cnf 
[root@db156 opt]# cat /etc/my.cnf
[mysqld]
server_id=13306
gtid_mode=on
enforce_gtid_consistency=1
transaction_write_set_extraction=XXHASH64
loose-group_replication_group_name='aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa'
loose-group_replication_start_on_boot=off
loose-group_replication_ip_whitelist='192.168.165.0/24'
loose-group_replication_local_address='192.168.165.156:33061'
loose-group_replication_group_seeds='192.168.165.156:33061,192.168.165.157:33061,192.168.165.225:33061'
loose-group_replication_bootstrap_group=off
loose-group_replication_single_primary_mode=on
loose-group_replication_enforce_update_everywhere_checks=off
loose-group_replication_recovery_get_public_key=on
loose-group_replication_recovery_use_ssl=off
loose-group_replication_ssl_mode='DISABLED'
loose-group_replication_consistency='EVENTUAL'
loose-group_replication_member_expel_timeout=5
report_host='192.168.165.156'
report_port=3306
plugin_load_add ='group_replication.so'
auto_increment_increment=1
auto_increment_offset=1
mysqlx_port=33060
admin_port=33062
datadir=/data/mysql/data
basedir=/usr/local/mysql
user=mysql
bled_storage_engines="MyISAM,BLACKHOLE,FEDERATED,ARCHIVE,MEMORY"
[root@db225 opt]# systemctl restart mysqld
```



首次启动组复制的过程称为引导，，可以使用group_replication_bootstrap_group系统变量来引导组，且只能在一个实例上执行并只能执行一次。

我们选择节点一上执行：

```
[root@db156 opt]# /usr/local/mysql/bin/mysql -uroot -S /tmp/mysql.sock -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.25 MySQL Community Server - GPL

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> SET GLOBAL group_replication_bootstrap_group=ON;
Query OK, 0 rows affected (0.01 sec)

mysql> START GROUP_REPLICATION USER='repl', PASSWORD='123456';
Query OK, 0 rows affected, 1 warning (2.22 sec)

mysql> SET GLOBAL group_replication_bootstrap_group=OFF;
Query OK, 0 rows affected (0.00 sec)

mysql> 
```

节点二及节点三：

```
[root@db225 opt]# /usr/local/mysql/bin/mysql -uroot -S /tmp/mysql.sock -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.25 MySQL Community Server - GPL

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> START GROUP_REPLICATION USER='repl', PASSWORD='123456';
Query OK, 0 rows affected, 1 warning (2.64 sec)

mysql> 
```

在将其他节点添加至复制组时，如果报一下错误：

```
mysql> START GROUP_REPLICATION USER='repl', PASSWORD='123456';
ERROR 3092 (HY000): The server is not configured properly to be an active member of the group. Please see more details on error log.
```

查看到日志如下：

```
2021-06-10T10:42:54.535154Z 10 [System] [MY-010597] [Repl] 'CHANGE MASTER TO FOR CHANNEL 'group_replication_applier' executed'. Previous state master_host='', master_port= 3306, master_log_file='', master_log_pos= 4, master_bind=''. New state master_host='<NULL>', master_port= 0, master_log_file='', master_log_pos= 4, master_bind=''.
2021-06-10T10:42:57.045266Z 0 [ERROR] [MY-011526] [Repl] Plugin group_replication reported: 'This member has more executed transactions than those present in the group. Local transactions: 689d5d1d-c9cc-11eb-8243-005056a0d28b:1 > Group transactions: aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:1'
2021-06-10T10:42:57.056782Z 0 [ERROR] [MY-011522] [Repl] Plugin group_replication reported: 'The member contains transactions not present in the group. The member will now exit the group.'
2021-06-10T10:42:57.056911Z 0 [System] [MY-011503] [Repl] Plugin group_replication reported: 'Group membership changed to 192.168.165.157:3306, 192.168.165.156:3306 on view 16233217077427225:2.'
2021-06-10T10:43:00.730039Z 0 [System] [MY-011504] [Repl] Plugin group_replication reported: 'Group membership changed: This member has left the group.'
2021-06-10T10:43:00.758719Z 9 [System] [MY-011566] [Repl] Plugin group_replication reported: 'Setting super_read_only=OFF.'
```

则在该节点上执行`reset master`可以解决。

查看组复制状态(任何一个实例均可查询)：

```
mysql> select * FROM performance_schema.replication_group_members;
+---------------------------+--------------------------------------+-----------------+-------------+--------------+-------------+----------------+
| CHANNEL_NAME              | MEMBER_ID                            | MEMBER_HOST     | MEMBER_PORT | MEMBER_STATE | MEMBER_ROLE | MEMBER_VERSION |
+---------------------------+--------------------------------------+-----------------+-------------+--------------+-------------+----------------+
| group_replication_applier | 689d5d1d-c9cc-11eb-8243-005056a0d28b | 192.168.165.157 |        3306 | ONLINE       | SECONDARY   | 8.0.25         |
| group_replication_applier | 71d57215-c9cc-11eb-a5ac-005056a098ec | 192.168.165.225 |        3306 | ONLINE       | SECONDARY   | 8.0.25         |
| group_replication_applier | 8b5f525d-c9c3-11eb-bf4e-005056a0bb1a | 192.168.165.156 |        3306 | ONLINE       | PRIMARY     | 8.0.25         |
+---------------------------+--------------------------------------+-----------------+-------------+--------------+-------------+----------------+
3 rows in set (1.54 sec)

mysql> 
```

至此，MGR三节点的组复制搭建完成。

## 安装MySQL Shell

安装过程如下：

```
[root@db156 opt]# tar -xf mysql-shell-8.0.25-linux-glibc2.12-x86-64bit.tar.gz 
[root@db156 opt]# mv mysql-shell-8.0.25-linux-glibc2.12-x86-64bit /usr/local/mysqlshell
[root@db156 opt]# chown -R mysql:mysql /usr/local/mysqlshell/
[root@db156 opt]# chmod -R 755 /usr/local/mysqlshell/
[root@db156 opt]# echo "export PATH=\$PATH:/usr/local/mysqlshell/bin" >> /etc/profile
[root@db156 opt]# source /etc/profile
```

三个节点均执行以上步骤安装。



## 创建InnoDB Cluster集群

通过mysqlsh命令使用安装MySQL Server时创建的管理账号连接到MySQL 实例，执行`dba.createCluster()`创建集群，并过过`dba.getCluster("testCluster").status()`查看集群状态：

```
[root@db225 opt]# mysqlsh icadmin@192.168.165.156:3306
Please provide the password for 'icadmin@192.168.165.156:3306': ******
Save password for 'icadmin@192.168.165.156:3306'? [Y]es/[N]o/Ne[v]er (default No): 
MySQL Shell 8.0.25

Copyright (c) 2016, 2021, Oracle and/or its affiliates.
Oracle is a registered trademark of Oracle Corporation and/or its affiliates.
Other names may be trademarks of their respective owners.

Type '\help' or '\?' for help; '\quit' to exit.
Creating a session to 'icadmin@192.168.165.156:3306'
Fetching schema names for autocompletion... Press ^C to stop.
Your MySQL connection id is 28
Server version: 8.0.25 MySQL Community Server - GPL
No default schema selected; type \use <schema> to set one.
 MySQL  192.168.165.156:3306 ssl  JS > var cluster = dba.createCluster('testCluster', {adoptFromGR: true});
A new InnoDB cluster will be created based on the existing replication group on instance '192.168.165.156:3306'.

Creating InnoDB cluster 'testCluster' on '192.168.165.156:3306'...

Adding Seed Instance...
Adding Instance '192.168.165.157:3306'...
Adding Instance '192.168.165.225:3306'...
Adding Instance '192.168.165.156:3306'...
Resetting distributed recovery credentials across the cluster...
NOTE: User 'mysql_innodb_cluster_13306'@'%' already existed at instance '192.168.165.156:3306'. It will be deleted and created again with a new password.
NOTE: User 'mysql_innodb_cluster_13306'@'%' already existed at instance '192.168.165.156:3306'. It will be deleted and created again with a new password.
Cluster successfully created based on existing replication group.

 MySQL  192.168.165.156:3306 ssl  JS > dba.getCluster("testCluster").status();
{
    "clusterName": "testCluster", 
    "defaultReplicaSet": {
        "name": "default", 
        "primary": "192.168.165.156:3306", 
        "ssl": "DISABLED", 
        "status": "OK", 
        "statusText": "Cluster is ONLINE and can tolerate up to ONE failure.", 
        "topology": {
            "192.168.165.156:3306": {
                "address": "192.168.165.156:3306", 
                "instanceErrors": [
                    "NOTE: The required parallel-appliers settings are not enabled on the instance. Use dba.configureInstance() to fix it."
                ], 
                "memberRole": "PRIMARY", 
                "mode": "R/W", 
                "readReplicas": {}, 
                "replicationLag": null, 
                "role": "HA", 
                "status": "ONLINE", 
                "version": "8.0.25"
            }, 
            "192.168.165.157:3306": {
                "address": "192.168.165.157:3306", 
                "instanceErrors": [
                    "NOTE: The required parallel-appliers settings are not enabled on the instance. Use dba.configureInstance() to fix it."
                ], 
                "memberRole": "SECONDARY", 
                "mode": "R/O", 
                "readReplicas": {}, 
                "replicationLag": null, 
                "role": "HA", 
                "status": "ONLINE", 
                "version": "8.0.25"
            }, 
            "192.168.165.225:3306": {
                "address": "192.168.165.225:3306", 
                "instanceErrors": [
                    "NOTE: The required parallel-appliers settings are not enabled on the instance. Use dba.configureInstance() to fix it."
                ], 
                "memberRole": "SECONDARY", 
                "mode": "R/O", 
                "readReplicas": {}, 
                "replicationLag": null, 
                "role": "HA", 
                "status": "ONLINE", 
                "version": "8.0.25"
            }
        }, 
        "topologyMode": "Single-Primary"
    }, 
    "groupInformationSourceMember": "192.168.165.156:3306"
}
 MySQL  192.168.165.156:3306 ssl  JS > 
```

其中：

- "status": "OK" 表示集群状态是正常的
- "topologyMode": "Single-Primary" 表示是单主模式
- "mode": "R/W" 表示可读可写
- "mode": "R/O" 表示只读



## 安装MySQL Router

安装：

```
[root@db156 opt]# tar -xf mysql-router-8.0.25-linux-glibc2.12-x86_64.tar.xz 
[root@db156 opt]# mv mysql-router-8.0.25-linux-glibc2.12-x86_64 /usr/local/mysqlrouter
[root@db156 opt]# chown -R mysql:mysql /usr/local/mysqlrouter/
[root@db156 opt]# chmod -R 755 /usr/local/mysqlrouter/
[root@db156 opt]# mkdir -p /data/mysqlrouter
[root@db156 opt]# chown -R mysql:mysql /data/mysqlrouter/
[root@db156 opt]# echo "export PATH=\$PATH:/usr/local/mysqlrouter/bin" >> /etc/profile
[root@db156 opt]# source /etc/profile
```

bootstrap引导:

```
[root@db156 opt]# /usr/local/mysqlrouter/bin/mysqlrouter --bootstrap icadmin@192.168.165.156:3306 --directory /data/mysqlrouter --conf-use-sockets --user=mysql --name=mysql_router_13306 --conf-bind-address=192.168.165.156 --account-host="192.168.165.%"
Please enter MySQL password for icadmin: 
# Bootstrapping MySQL Router instance at '/data/mysqlrouter'...

- Creating account(s) (only those that are needed, if any)
- Verifying account (using it to run SQL queries that would be run by Router)
- Storing account in keyring
- Adjusting permissions of generated files
- Creating configuration /data/mysqlrouter/mysqlrouter.conf

# MySQL Router 'mysql_router_13306' configured for the InnoDB Cluster 'testCluster'

After this MySQL Router has been started with the generated configuration

    $ /usr/local/mysqlrouter/bin/mysqlrouter -c /data/mysqlrouter/mysqlrouter.conf

the cluster 'testCluster' can be reached by connecting to:

## MySQL Classic protocol

- Read/Write Connections: localhost:6446, /data/mysqlrouter/mysql.sock
- Read/Only Connections:  localhost:6447, /data/mysqlrouter/mysqlro.sock

## MySQL X protocol

- Read/Write Connections: localhost:6448, /data/mysqlrouter/mysqlx.sock
- Read/Only Connections:  localhost:6449, /data/mysqlrouter/mysqlxro.sock

[root@db156 opt]# 
```

引导成功后会在/data/mysqlrouter目录下生成所需的数据文件，目录结构如下：

```
[root@db156 opt]# tree /data/mysqlrouter/
/data/mysqlrouter/
├── data
│   ├── ca-key.pem
│   ├── ca.pem
│   ├── keyring
│   ├── router-cert.pem
│   ├── router-key.pem
│   └── state.json
├── log
│   └── mysqlrouter.log
├── mysqlrouter.conf
├── mysqlrouter.key
├── run
├── start.sh
└── stop.sh

3 directories, 11 files
```

生成的MySQL Router配置文件如下：

```
[root@db156 opt]# cat /data/mysqlrouter/mysqlrouter.conf 
# File automatically generated during MySQL Router bootstrap
[DEFAULT]
name=mysql_router_13306
user=mysql
logging_folder=/data/mysqlrouter/log
runtime_folder=/data/mysqlrouter/run
data_folder=/data/mysqlrouter/data
keyring_path=/data/mysqlrouter/data/keyring
master_key_path=/data/mysqlrouter/mysqlrouter.key
connect_timeout=15
read_timeout=30
dynamic_state=/data/mysqlrouter/data/state.json
client_ssl_cert=/data/mysqlrouter/data/router-cert.pem
client_ssl_key=/data/mysqlrouter/data/router-key.pem
client_ssl_mode=PREFERRED
server_ssl_mode=AS_CLIENT
server_ssl_verify=DISABLED

[logger]
level = INFO

[metadata_cache:testCluster]
cluster_type=gr
router_id=1
user=mysql_router1_7d9sq1vuggg9
metadata_cluster=testCluster
ttl=0.5
auth_cache_ttl=-1
auth_cache_refresh_interval=2
use_gr_notifications=0

[routing:testCluster_rw]
bind_address=192.168.165.156
bind_port=6446
socket=/data/mysqlrouter/mysql.sock
destinations=metadata-cache://testCluster/?role=PRIMARY
routing_strategy=first-available
protocol=classic

[routing:testCluster_ro]
bind_address=192.168.165.156
bind_port=6447
socket=/data/mysqlrouter/mysqlro.sock
destinations=metadata-cache://testCluster/?role=SECONDARY
routing_strategy=round-robin-with-fallback
protocol=classic

[routing:testCluster_x_rw]
bind_address=192.168.165.156
bind_port=6448
socket=/data/mysqlrouter/mysqlx.sock
destinations=metadata-cache://testCluster/?role=PRIMARY
routing_strategy=first-available
protocol=x

[routing:testCluster_x_ro]
bind_address=192.168.165.156
bind_port=6449
socket=/data/mysqlrouter/mysqlxro.sock
destinations=metadata-cache://testCluster/?role=SECONDARY
routing_strategy=round-robin-with-fallback
protocol=x

[http_server]
port=8443
ssl=1
ssl_cert=/data/mysqlrouter/data/router-cert.pem
ssl_key=/data/mysqlrouter/data/router-key.pem

[http_auth_realm:default_auth_realm]
backend=default_auth_backend
method=basic
name=default_realm

[rest_router]
require_realm=default_auth_realm

[rest_api]

[http_auth_backend:default_auth_backend]
backend=metadata_cache

[rest_routing]
require_realm=default_auth_realm

[rest_metadata_cache]
require_realm=default_auth_realm

[root@db156 opt]# 
```

启动MySQL路由：

```
[root@db156 opt]# bash /data/mysqlrouter/start.sh 2>&1 > /dev/null &
[root@db156 opt]# ps -ef | grep mysqlrouter
root     24911     1  0 08:00 pts/0    00:00:00 sudo ROUTER_PID=/data/mysqlrouter/mysqlrouter.pid /usr/local/mysqlrouter/bin/mysqlrouter -c /data/mysqlrouter/mysqlrouter.conf --user=mysql
mysql    24913 24911  2 08:00 pts/0    00:00:01 /usr/local/mysqlrouter/bin/mysqlrouter -c /data/mysqlrouter/mysqlrouter.conf --user=mysql
root     24998  9686  0 08:01 pts/0    00:00:00 grep --color=auto mysqlrouter
```

读写分离测试：使用不同的端口

```
[root@db156 opt]# /usr/local/mysql/bin/mysql -uicadmin -p -h 192.168.165.156 -P 6446    
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 196
Server version: 8.0.25 MySQL Community Server - GPL

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> select @@hostname as hostname, @@port as port;
+----------+------+
| hostname | port |
+----------+------+
| db156    | 3306 |
+----------+------+
1 row in set (0.00 sec)

mysql> exit
Bye
[root@db156 opt]# /usr/local/mysql/bin/mysql -uicadmin -p -h 192.168.165.156 -P 6447
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 30
Server version: 8.0.25 MySQL Community Server - GPL

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> select @@hostname as hostname, @@port as port;
+----------+------+
| hostname | port |
+----------+------+
| db225    | 3306 |
+----------+------+
1 row in set (0.00 sec)

mysql> exit
Bye
[root@db156 opt]# /usr/local/mysql/bin/mysql -uicadmin -p -h 192.168.165.156 -P 6447
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1517
Server version: 8.0.25 MySQL Community Server - GPL

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> select @@hostname as hostname, @@port as port;
+----------+------+
| hostname | port |
+----------+------+
| db157    | 3306 |
+----------+------+
1 row in set (0.00 sec)

mysql> 
```

可以看到使用读写端口6446时连接的是主节点，而使用只读端口6447连接时是在两个从节点之间切换操作。



