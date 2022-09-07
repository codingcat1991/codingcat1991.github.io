---
title: MySQL查询未区分大小写
tags:
  - MySQL
category: MySQL
top_img: 1606845605.jpg
cover: 1606845605.jpg
date: 2021-07-05 17:45:23
description: MySQL中，查询条件匹配的时候是否区分大小写？什么情况下区分？什么情况下不区分？遇到需要区分但是没有区分的场景该如何处理？
---

MySQL中，查询条件匹配的时候是否区分大小写？什么情况下区分？什么情况下不区分？遇到需要区分但是没有区分的场景该如何处理？

## 场景

考虑如下场景：

```sql
mysql> use test_0705;
Database changed
mysql> select * from tb_test where name = 'CodingCat';
+----+-----------+
| id | name      |
+----+-----------+
|  1 | CodingCat |
|  2 | CODINGCAT |
|  3 | codingcat |
+----+-----------+
3 rows in set (0.02 sec)

mysql> 
```

从上图我们看到，查询条件中指定要查询的是`name='CodingCat'`，但是结果把`CODINGCAT`和`codingcat`一起给查询出来了，可见此查询结果并未区分大小写。

## 探究

我们知道，在MySQL中有排序规则这个概念，排序规则表示在规定的存储的数据编码格式下的比较规则，如是否区分大小写等。针对上面的场景，查看表结构如下：

```plsql
mysql> show create table tb_test;
+---------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table   | Create Table                                                                                                                                                                |
+---------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| tb_test | CREATE TABLE `tb_test` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8 |
+---------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.04 sec)

mysql> 
```

该表继承了数据库的排序规则，数据库建表语句如下：

```sql
mysql> show create database test_0705;
+-----------+--------------------------------------------------------------------+
| Database  | Create Database                                                    |
+-----------+--------------------------------------------------------------------+
| test_0705 | CREATE DATABASE `test_0705` /*!40100 DEFAULT CHARACTER SET utf8 */ |
+-----------+--------------------------------------------------------------------+
1 row in set (0.03 sec)

mysql> 
```

从数据库建表语句来看，并没有指定排序规则，则该数据库继承了全局的排序规则，查看数据库的排序规则如下：

```sql
mysql> show variables like 'collation%';
+----------------------+--------------------+
| Variable_name        | Value              |
+----------------------+--------------------+
| collation_connection | utf8mb4_general_ci |
| collation_database   | utf8_general_ci    |
| collation_server     | utf8mb4_general_ci |
+----------------------+--------------------+
3 rows in set (0.38 sec)

mysql> 
```

从全局的配置来看，数据库服务器使用的是`utf8mb4_general_ci`排序规则，后缀有ci，意思是case insensitive，即大小写不敏感，所以查询出来的结果不区分大小写。

在MySQL中，排序规则常见有带后缀ci和不带ci两种，如utf8_bin和utf8_genera_ci，带ci表示区分大小写，否则不区分。

## 解决

对于上面的问题，如果在设置了字符排序规则是带ci后缀的，但是又想要查询结果是区分大小写，怎么做？有两种解决方法。

- 查询时在需要区分大小写的字段条件上使用binary关键字

  ```sql
  mysql> select * from tb_test where binary name = 'CodingCat';
  +----+-----------+
  | id | name      |
  +----+-----------+
  |  1 | CodingCat |
  +----+-----------+
  1 row in set (0.04 sec)
  
  mysql> 
  ```

- 修改表中字段的排序规则

  ```sql
  mysql> ALTER TABLE tb_test MODIFY COLUMN name VARCHAR(255) BINARY CHARACTER SET utf8 COLLATE utf8_bin DEFAULT NULL;
  Query OK, 3 rows affected (0.08 sec)
  Records: 3  Duplicates: 0  Warnings: 0
  
  mysql> show create table tb_test;
  +---------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
  | Table   | Create Table                                                                                                                                                                                                    |
  +---------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
  | tb_test | CREATE TABLE `tb_test` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `name` varchar(255) CHARACTER SET utf8 COLLATE utf8_bin DEFAULT NULL,
    PRIMARY KEY (`id`)
  ) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8 |
  +---------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
  1 row in set (0.05 sec)
  
  mysql> select * from tb_test where binary name = 'CodingCat';
  +----+-----------+
  | id | name      |
  +----+-----------+
  |  1 | CodingCat |
  +----+-----------+
  1 row in set (0.05 sec)
  
  mysql> 
  ```

  





