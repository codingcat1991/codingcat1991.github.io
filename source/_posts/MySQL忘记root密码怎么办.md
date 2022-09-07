---
title: MySQL忘记root密码怎么办
date: 2020-10-23
tags:
	- MySQL
category: MySQL
description: 使用MySQL过程中我们偶尔会遇到忘记密码的情况，怎么办呢？能不能吧密码找回来或者重新设置密码呢？
---

使用MySQL过程中我们偶尔会遇到忘记密码的情况，怎么办呢？能不能吧密码找回来或者重新设置密码呢？

MySQL忘记root密码之后，可以重新设置一个新的密码，怎么做呢？

1. 修改配置文件my.cnf，在[mysqld]下面增加参数：skip-grant-tables（启动 MySQL 服务的时候跳过权限表认证。启动后，连接到 MySQL 的 root 将不需要口令。），重启MySQL服务，免密码登录。

   ```
   [root@ocp11g~]# vi /etc/my.cnf    
   [root@ocp11g~]# service mysql restart
   Shuttingdown MySQL.... SUCCESS!
   StartingMySQL.. SUCCESS!
   [root@ocp11g~]# mysql -uroot
   Welcometo the MySQL monitor. Commands end with; or \g.
   YourMySQL connection id is 2
   Serverversion: 5.7.26-log MySQL Community Server (GPL)
   Copyright(c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.
   Oracleis a registered trademark of Oracle Corporation and/or its
   affiliates.Other names may be trademarks of their respective
   owners.
   Type'help;' or '\h' for help. Type '\c' to clear the current input statement.
   mysql>
   ```

2. 修改root用户密码为空

   ```
   mysql>alter user root@localhost identified by '123456';
   ERROR1290 (HY000): The MySQL server is running with the --skip-grant-tables optionso it cannot execute this statement
   mysql>update user set authentication_string = NULL where user = 'root';
   ERROR1046 (3D000): No database selected
   mysql>use mysql
   Readingtable information for completion of table and column names
   Youcan turn off this feature to get a quicker startup with -A
   Databasechanged
   mysql>update user set authentication_string = NULL where user = 'root';
   QueryOK, 1 row affected (0.01 sec)
   Rowsmatched: 1 Changed: 1 Warnings: 0
   mysql>
   ```

3. 修该配置文件删除skip-grant-tables参数，重启MySQL服务，使用空密码登录，修改root密码

   ```
   [root@ocp11g~]# vi /etc/my.cnf    
   [root@ocp11g~]# service mysql restart
   Shuttingdown MySQL.. SUCCESS!
   StartingMySQL. SUCCESS!
   [root@ocp11g~]# mysql -uroot
   Welcometo the MySQL monitor. Commands end with; or \g.
   YourMySQL connection id is 3
   Serverversion: 5.7.26-log MySQL Community Server (GPL)
   Copyright(c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.
   Oracleis a registered trademark of Oracle Corporation and/or its
   affiliates.Other names may be trademarks of their respective
   owners.
   Type'help;' or '\h' for help. Type '\c' to clear the current input statement.
   mysql>alter user root@localhost identified by '123456';
   QueryOK, 0 rows affected (0.00 sec)
   mysql>
   ```

4. 密码修改完成，MySQL root密码已修改，MySQL正常使用

   ```
   [root@ocp11g~]# mysql -uroot
   ERROR1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)
   [root@ocp11g~]# mysql -uroot -p
   Enterpassword:
   Welcometo the MySQL monitor. Commands end with; or \g.
   YourMySQL connection id is 5
   Serverversion: 5.7.26-log MySQL Community Server (GPL)
   Copyright(c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.
   Oracleis a registered trademark of Oracle Corporation and/or its
   affiliates.Other names may be trademarks of their respective
   owners.
   Type'help;' or '\h' for help. Type '\c' to clear the current input statement.
   mysql>exit
   Bye
   ```

   