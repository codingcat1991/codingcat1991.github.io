---
title: Oracle数据库关闭的情况下如何修改参数
category: Oracle
tags:
	- Oracle
date: 2020-10-20
description: 很多时候由于误操作，修改了某些参数后，重启数据库发现无法启动，需要将参数修改回来或者其他操作后才能正常启动，但是在数据库关闭的情况下，无法使用alter system等命令来修改参数，到底需要怎么做呢？
---

很多时候由于误操作，修改了某些参数后，重启数据库发现无法启动，需要将参数修改回来或者其他操作后才能正常启动，但是在数据库关闭的情况下，无法使用alter system等命令来修改参数，到底需要怎么做呢？

## 场景

在Oracle中很多参数在修改的时候都有一定的规定，如在设置快速恢复区时，db_recovery_file_dest和db_recovery_file_dest_size需修改后重启数据库，如果只修改了db_recovery_file_dest，db_recovery_file_dest_size并不设定值，则无法重启数据库，启动时报错如下：

```
SQL> startup 
ORA-19802: cannot use DB_RECOVERY_FILE_DEST without DB_RECOVERY_FILE_DEST_SIZE
```

由于数据库已关闭，这时候也无法修改参数db_recovery_file_dest_size：

```
SQL> alter system set db_recovery_file_dest_size=4G scope=spfile;
alter system set db_recovery_file_dest_size=4G scope=spfile
*
ERROR at line 1:
ORA-01034: ORACLE not available
Process ID: 0
Session ID: 0 Serial number: 0
```

## 解决方法

遇到上面类似修改参数之后无法启动数据库的情况，可以通过修改参数文件来解决。在Oracle中，有spfile和pfile两种参数文件，spfile是不可编辑的，正常情况下数据库使用的是spfile，pfile可以编辑，Oracle中可以通过spfile和pfile来相互创建，因此可以通过这个方法来解决此问题。

首先，使用已有的pfile来启动数据库，如果没有，可以根据其他数据库的内容来手动创建：startup pfile=''

启动后，使用create pfile='新的pfile文件' from spfile='原来的spfile文件'，此时的数据库状态不一定非得OPEN。

然后修改新建的pfile文件，确保参数都正确修改，如上面的例子，删除db_recovery_file_dest所在的行

关闭数据库并使用新的pfile文件启动数据库：startup pfile='新的pfile';

创建spfile文件：create spfile='spfile文件' from pfile='新的pfile文件';

关闭数据库，正常启动：startup

## 实战

```
SQL> show parameter db_recovery            

NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
db_recovery_file_dest                string
db_recovery_file_dest_size           big integer 0
SQL> alter system set db_recovery_file_dest='/u01/app/oracle/fast_recovery_area' scope=spfile;

System altered.
SQL> shutdown immediate
Database closed.
Database dismounted.
ORACLE instance shut down.
SQL> startup 
ORA-19802: cannot use DB_RECOVERY_FILE_DEST without DB_RECOVERY_FILE_DEST_SIZE
SQL> alter system set db_recovery_file_dest_size=4G scope=spfile;
alter system set db_recovery_file_dest_size=4G scope=spfile
*
ERROR at line 1:
ORA-01034: ORACLE not available
Process ID: 0
Session ID: 0 Serial number: 0


SQL> startup pfile='/u01/app/oracle/product/19.3.0/db_1/dbs/init.ora'
ORACLE instance started.

Total System Global Area 1073737800 bytes
Fixed Size                  8904776 bytes
Variable Size             616562688 bytes
Database Buffers          440401920 bytes
Redo Buffers                7868416 bytes
ORA-00205: error in identifying control file, check alert log for more info


SQL> select status from v$instance;

STATUS
------------
STARTED

SQL> create pfile='/u01/app/oracle/product/19.3.0/db_1/dbs/init-20201020.ora' from spfile='/u01/app/oracle/product/19.3.0/db_1/dbs/spfileorcl.ora';

File created.

SQL> shutdown immediate 
ORA-01507: database not mounted


ORACLE instance shut down.
SQL> 
```

修改文件/u01/app/oracle/product/19.3.0/db_1/dbs/init-20201020.ora，删除db_recovery_file_dest所在的行。

```
SQL> startup pfile='/u01/app/oracle/product/19.3.0/db_1/dbs/init-20201020.ora'
ORACLE instance started.

Total System Global Area 1593832624 bytes
Fixed Size                  9135280 bytes
Variable Size             973078528 bytes
Database Buffers          603979776 bytes
Redo Buffers                7639040 bytes
Database mounted.
Database opened.

SQL> create spfile='/u01/app/oracle/product/19.3.0/db_1/dbs/spfileorcl.ora' from pfile='/u01/app/oracle/product/19.3.0/db_1/dbs/init-20201020.ora';

File created.

SQL> 
SQL> shutdown immediate 
Database closed.
Database dismounted.
ORACLE instance shut down.
SQL> startup 
ORACLE instance started.

Total System Global Area 1593832624 bytes
Fixed Size                  9135280 bytes
Variable Size             973078528 bytes
Database Buffers          603979776 bytes
Redo Buffers                7639040 bytes
Database mounted.
Database opened.
```

