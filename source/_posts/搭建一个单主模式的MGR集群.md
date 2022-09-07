---
title: 搭建一个单主模式的MGR集群
tags:
  - MySQL
  - 组复制
category:
  - MySQL
cover: 1605722404.jpg
'top_img:': 1605722404.jpg
date: 2021-05-10 10:55:23
keywords: ‘MySQL,组复制’
description: 搭建一个单主模式的MGR集群
---

搭建一个单主模式的MGR集群

# 环境说明

实验基于MySQL8.0.24版本，操作系统为CentOS7，共三个节点，分别如下：

| 主机名 | IP              | Port | Server Id |
| ------ | --------------- | ---- | --------- |
| db156  | 192.168.165.156 | 3306 | 1         |
| db157  | 192.168.165.157 | 3306 | 2         |
| db225  | 192.168.165.225 | 3306 | 3         |

架构如下：

![单节点单主组复制架构](单节点单主组复制架构.png)

# 修改hosts配置

修改每个服务器的/etc/hosts文件并增加每个服务器的映射：

```
192.168.165.156 db156
192.168.165.157 db157
192.168.165.225 db225
```

# 部署MySQL实例

不做赘述

# 第一个节点配置

修改my.cnf文件并重启MySQL服务

在mysqld下增加如下配置：

```
disabled_storage_engines="MyISAM,BLACKHOLE,FEDERATED,ARCHIVE,MEMORY"
server_id = 1
gtid_mode=ON
enforce_gtid_consistency=ON
binlog_checksum=NONE
log_bin = /data/mysql/logs/mysql-bin
log_slave_updates=ON
binlog_format=ROW
master_info_repository=TABLE
relay_log_info_repository=TABLE
transaction_write_set_extraction=XXHASH64
plugin_load_add='group_replication.so'
group_replication_group_name="1eab024d-afdd-11eb-a2d1-005056a0bb1a"
group_replication_start_on_boot=off
group_replication_local_address= "192.168.165.156:33061"
group_replication_group_seeds= "192.168.165.156:33061,192.168.165.157:33061,192.168.165.225:33061"
group_replication_bootstrap_group=off
```

重启MySQL服务后执行如下操作：

```sql
[root@db156 ~]# mysql
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.24 MySQL Community Server - GPL

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> SET SQL_LOG_BIN=0;
Query OK, 0 rows affected (0.00 sec)

mysql> CREATE USER repl@'%' IDENTIFIED BY '123456';
Query OK, 0 rows affected (0.02 sec)

mysql> GRANT REPLICATION SLAVE ON *.* TO repl@'%';
Query OK, 0 rows affected (0.00 sec)

mysql> GRANT BACKUP_ADMIN ON *.* TO repl@'%';
Query OK, 0 rows affected (0.00 sec)

mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)

mysql> SET SQL_LOG_BIN=1;
Query OK, 0 rows affected (0.00 sec)

mysql> CHANGE REPLICATION SOURCE TO SOURCE_USER='repl', SOURCE_PASSWORD='123456' FOR CHANNEL 'group_replication_recovery';
Query OK, 0 rows affected, 2 warnings (0.02 sec)

mysql> SET GLOBAL group_replication_bootstrap_group=ON;
Query OK, 0 rows affected (0.00 sec)

mysql> START GROUP_REPLICATION USER='repl', PASSWORD='123456';
Query OK, 0 rows affected (2.24 sec)

mysql> SET GLOBAL group_replication_bootstrap_group=OFF;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT * FROM performance_schema.replication_group_members;
+---------------------------+--------------------------------------+-------------+-------------+--------------+-------------+----------------+
| CHANNEL_NAME              | MEMBER_ID                            | MEMBER_HOST | MEMBER_PORT | MEMBER_STATE | MEMBER_ROLE | MEMBER_VERSION |
+---------------------------+--------------------------------------+-------------+-------------+--------------+-------------+----------------+
| group_replication_applier | 9b4c6994-afd5-11eb-beab-005056a0bb1a | db156       |        3306 | ONLINE       | PRIMARY     | 8.0.24         |
+---------------------------+--------------------------------------+-------------+-------------+--------------+-------------+----------------+
1 row in set (0.00 sec)

# 创建测试库
mysql> CREATE DATABASE test;
Query OK, 1 row affected (0.00 sec)

mysql> USE test;
Database changed
mysql> CREATE TABLE t1 (c1 INT PRIMARY KEY, c2 TEXT NOT NULL);
Query OK, 0 rows affected (0.01 sec)

mysql> INSERT INTO t1 VALUES (1, 'Luis');
Query OK, 1 row affected (0.01 sec)

mysql> 

```

# 第二个节点配置

将第一个节点的配置文件复制到第二个节点并修改server-id和group_replication_local_address即可

```
server_id = 2
binlog_format = ROW
disabled_storage_engines="MyISAM,BLACKHOLE,FEDERATED,ARCHIVE,MEMORY"
gtid_mode=ON
enforce_gtid_consistency=ON
binlog_checksum=NONE
log_slave_updates=ON
master_info_repository=TABLE
relay_log_info_repository=TABLE
transaction_write_set_extraction=XXHASH64
plugin_load_add='group_replication.so'
group_replication_group_name="1eab024d-afdd-11eb-a2d1-005056a0bb1a"
group_replication_start_on_boot=off
group_replication_local_address= "192.168.165.157:33061"
group_replication_group_seeds= "192.168.165.156:33061,192.168.165.157:33061,192.168.165.225:33061"
group_replication_bootstrap_group=off
```

重启后执行如下操作，注意和第一个节点的区别：

```sql
[root@db157 ~]# mysql
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.24 MySQL Community Server - GPL

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> SET SQL_LOG_BIN=0;
Query OK, 0 rows affected (0.00 sec)

mysql> CREATE USER repl@'%' IDENTIFIED BY '123456';
Query OK, 0 rows affected (0.03 sec)

mysql> GRANT REPLICATION SLAVE ON *.* TO repl@'%';
Query OK, 0 rows affected (0.00 sec)

mysql> GRANT BACKUP_ADMIN ON *.* TO repl@'%';
Query OK, 0 rows affected (0.01 sec)

mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)

mysql> SET SQL_LOG_BIN=1;
Query OK, 0 rows affected (0.00 sec)

mysql> CHANGE REPLICATION SOURCE TO SOURCE_USER='repl', SOURCE_PASSWORD='123456' FOR CHANNEL 'group_replication_recovery';
Query OK, 0 rows affected, 2 warnings (0.01 sec)

mysql> START GROUP_REPLICATION USER='repl', PASSWORD='123456';
Query OK, 0 rows affected (3.11 sec)

mysql> SELECT * FROM performance_schema.replication_group_members;
+---------------------------+--------------------------------------+-------------+-------------+--------------+-------------+----------------+
| CHANNEL_NAME              | MEMBER_ID                            | MEMBER_HOST | MEMBER_PORT | MEMBER_STATE | MEMBER_ROLE | MEMBER_VERSION |
+---------------------------+--------------------------------------+-------------+-------------+--------------+-------------+----------------+
| group_replication_applier | 89999650-afd6-11eb-893e-005056a0d28b | db157       |        3306 | RECOVERING   | SECONDARY   | 8.0.24         |
| group_replication_applier | 9b4c6994-afd5-11eb-beab-005056a0bb1a | db156       |        3306 | ONLINE       | PRIMARY     | 8.0.24         |
+---------------------------+--------------------------------------+-------------+-------------+--------------+-------------+----------------+
2 rows in set (0.01 sec)

mysql> SELECT * FROM performance_schema.replication_group_members;
+---------------------------+--------------------------------------+-------------+-------------+--------------+-------------+----------------+
| CHANNEL_NAME              | MEMBER_ID                            | MEMBER_HOST | MEMBER_PORT | MEMBER_STATE | MEMBER_ROLE | MEMBER_VERSION |
+---------------------------+--------------------------------------+-------------+-------------+--------------+-------------+----------------+
| group_replication_applier | 89999650-afd6-11eb-893e-005056a0d28b | db157       |        3306 | ONLINE       | SECONDARY   | 8.0.24         |
| group_replication_applier | 9b4c6994-afd5-11eb-beab-005056a0bb1a | db156       |        3306 | ONLINE       | PRIMARY     | 8.0.24         |
+---------------------------+--------------------------------------+-------------+-------------+--------------+-------------+----------------+
2 rows in set (0.00 sec)

mysql> 
```



# 第三个节点配置

与第二个节点相同