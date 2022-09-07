---
title: SQL注入介绍
category: MySQL
tags:
	- MySQL
	- SQL注入
date: 2020-12-5
description: SQL 注入攻击是通过将恶意的SQL语句如添加、删除等插入到应用的输入参数中，经过后台解析后发送到数据库服务器上解析执行进行的攻击。本文以mysql为例，讨论SQL注入以及在Django中如何防止SQL注入。
---

SQL 注入攻击是通过将恶意的SQL语句如添加、删除等插入到应用的输入参数中，经过后台解析后发送到数据库服务器上解析执行进行的攻击。本文以mysql为例，讨论SQL注入以及在Django中如何防止SQL注入。

## SQL注入介绍

在Web程序中，一般都会有后台根据用户输入内容查找或者执行相关动作的场景，如登录时查询用户是否存在。后台在处理的时候可能是根据用户输入的用户名，拼接SQL之后到数据库查询来判断，这时，如果用户恶意输入不正确内容或内容本身存在问题，会导致应用程序崩溃，甚至是丢失数据等导致相关损失。即通过把SQL命令插入到Web表单提交或输入域名或页面请求的查询字符串，最终达到欺骗服务器执行恶意的SQL命令。这种场景就称为SQL注入。

### 一个SQL注入的简单例子

如执行一条SQL语句:

```sql
select * 
  from tb_user 
 where username = 'jacobzhou';
```

其中jacobzhou是用户输入的值，此时可以得到正确的值：

```sql
mysql> select * 
    ->   from tb_user 
    ->  where username = 'jacobzhou';
+------+-----------+----------+------+
| id   | username  | password | age  |
+------+-----------+----------+------+
|    1 | jacobzhou | 123456   |   29 |
+------+-----------+----------+------+
1 row in set (0.00 sec)
```

但是，当用户输入错误的值，如jacobzhou';drop table  tb_test;，如果使用的是字符串拼接的方式去执行，SQL语句就变成了：

```mysql
select * 
  from tb_user 
 where username = 'jacobzhou';drop table tb_test;';
```

执行结果为：

```sql
mysql> select * 
    ->   from tb_user 
    ->  where username = 'jacobzhou';drop table tb_test;';
+------+-----------+----------+------+
| id   | username  | password | age  |
+------+-----------+----------+------+
|    1 | jacobzhou | 123456   |   29 |
+------+-----------+----------+------+
1 row in set (0.00 sec)

ERROR 1051 (42S02): Unknown table 'db-platform.tb_test'
    '> 
```

以上就是一个SQL注入的例子，可以看到，如果db-platform.tb_test表存在，那就会被恶意删除。例子相对较极端，对于Django自带的connection来说，使用execute函数执行SQL语句的时候，每次只执行一条语句，后一条语句不会执行。因此上面的例子在Django中是不成立的，但是足以说明SQL注入所带来的安全风险。那SQL注入都有哪些方式，如何才能防止SQL注入呢？

## SQL注入原理

Web应用程序对于用户输入的数据和合法性没有严谨的判断，前端用户的输入直接传输给后端，攻击者通过构造不同的参数，形成不同的SQL语句来实现对数据库的任意操作。
SQL注入产生需要满足两个条件：

- 参数用户可控：前端传给后端的参数内容是用户可以控制的
- 参数带入数据库查询：传入的参数直接拼接到SQL语句，且带入数据库查询

## SQL注入类型

SQL注入的分类有很多，如POST注入、Cookie注入、延时注入、搜索注入等，但是归根结底也是数字型和字符型注入的不同展现形式或者是注入的位置不同。

### 数字型

用户输入为整数，假设SQL语句为：

```sql
select * from home_application_database where id = 3; 
```

其中3为数字，是用户正常输入。

当满足如下条件，则可能存在数字型注入：

- 输入3' 页面报错（SQL语法错误）
- 输入3 and 1 = 1 页面正常返回结果
- 输入3 and 1=2 页面返回错误（SQL语句返回空数据）

如果后台使用的是`select * from home_application_database where id = `和未经验证的用户输入做拼接后去数据库查询，就满足了上面的三个条件，存在数字型注入。

这里可以看到用户输入必须是整数，后端验证用户输入必须是整数才会继续执行即可解决。

### 字符型

用户输入是字符串，如SQL语句：

```sql
select * from home_application_database where name = '154'
```

其中154是字符串，是用户正常输入。

当满足如下条件时，则可能存在字符型注入：

- 输入154'，页面异常（SQL语句语法错误）
- 输入154' and 1 = 1 -- ，页面正常返回
- 输入154' and 1 = 2 -- ，页面错误（查询结果为空）

同数字型，如果后台直接使用拼接语句的形式去数据库执行，则就满足了上面单个条件，存在字符型注入。攻击者使用单引号的方式提前结束前一个单引号，并使用and来添加其他操作。

## 如何防止SQL注入

在开发时应该秉持一种**外部参数皆不可信**的原则来进行开发。

- 加强参数验证

  开发时，验证所有来自前端的输入，必须是符合要求的数据类型，符合指定规则的数据才允许继续往下执行。

- SQL语句参数化处理

  减少使用或不使用字符串拼接的方式执行SQL，而是将用户输入当着参数传给执行SQL的方法，如Django中的cursor.execute()函数就支持在SQL语句中使用占位符，将输入作为参数传递给方法执行。

- 存储过程

  使用存储过程也可以有效防止SQL注入，不过在存储过程中，需使用占位符，并且使用输入参数来预编译SQL语句后再执行。

## Django中防止SQL注入

Django中使用ORM可以有效防止SQL注入，所以应该尽可能使用ORM。但是ORM对于复杂查询就无能为力了，这时就需要执行原生SQL时，可以使用如下方式：

1. 使用extra（不建议使用这种方式执行SQL）
2. 使用raw
3. 使用django.db执行自定义SQL
4. 直接使用pymysql

在使用原生SQL语句时，应避免直接使用用户输入拼接SQL语句，上面三种执行原生SQL的方式均提供了占位符来进行参数替换，防止SQL注入。我们比较常用的是3、和4，两种方法都是使用cursor.execue()方法，具体如下：

- django.db

  ```python
  from django.db import connection
  cursor = connection.cursor()
  cursor.execute(sql)
  result = cursor.fetchall()
  ```

- pymysql

  ```python
  connection = pymysql.connect(**mysql_server)
  cursor = connection.cursor()
  cursor.execute(sql)
  result = cursor.fetchall()
  ```

  execute中的sql语句使用占位符，并传入相应参数即可防止SQL注入：

  ```
  sql="select * from home_application_database where id = %s" 
  cursor.execute(sql,3)
  1
  cursor.execute(sql,"3'")
  1
  E:\venv\python2\lib\site-packages\pymysql\cursors.py:297: Warning: Truncated incorrect DOUBLE value: '3''
    self._do_get_result()
  cursor.execute(sql,"3 and 1 = 1 ")
  E:\venv\python2\lib\site-packages\pymysql\cursors.py:297: Warning: Truncated incorrect DOUBLE value: '3 and 1 = 1 '
  1
    self._do_get_result()
  cursor.execute(sql,"3 and 1 = 2 ")
  E:\venv\python2\lib\site-packages\pymysql\cursors.py:297: Warning: Truncated incorrect DOUBLE value: '3 and 1 = 2 '
    self._do_get_result()
  1
  ```

  从上面执行结果看出，对于整数型注入，使用execute中带参数执行的方式，并不满足注入条件，当使用3'，3 and 1 = 1 ，3 and 1 = 2 作为输入传给execute执行时，程序报错。

  同理对于字符型注入也一样。

  > 注意：在使用execute函数执行时，SQL语句中的占位符，不管是字符还是整型，都使用%s，且对于字符型的数据，在SQL语句里面不能使用'%s'，否则会报错。使用参数替换本质上是对输入的参数进行转义处理，防止输入中的引号。

