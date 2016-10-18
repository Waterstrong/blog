---
title: MySQL在Linux下默认区分大小写
date: 2015-12-20 22:28:42
category: Techniques
tags: [数据库, MySQL, Linux, Hibernate, JPA]
thumbnailImage: /assets/mysql-case/mysql.png
---

前段时间遇到一个Hibernate/JPA自动映射MySQL Schema时报错问题，然后查了一下官方文档，原来是MySQL在Linux下默认区分大小写导致的，大致了解了一下，主要涉及两个变量lower_case_file_system和lower_case_table_names。
<!-- more -->

#### 默认大小写敏感
MySQL数据库名、表名、别名在Linux下默认区分大小写，root登录通过命令查看其配置：
mysql> show variables like 'lower%';
+------------------------+-------+
| Variable_name          | Value |
+------------------------+-------+
| lower_case_file_system | OFF   |
| lower_case_table_names | 0     |
+------------------------+-------+

#### 变量名和值解释
lower_case_file_system为只读属性，显示出系统的文件系统是否大小写敏感，OFF表示大小写敏感，ON表示大小写不敏感。
lower_case_table_names默认为0，表示大小写敏感；设置1表示大小写不敏感，创建的表/数据库以小写形式存放在磁盘上，对于sql语句转换为小写操作；设置2表示创建的表/数据库依据语句上格式存放，但查找都是转换为小写进行。

lower_case_file_system解释可参见[官方文档](http://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_lower_case_file_system)说明：

> This variable describes the case sensitivity of file names on the file system where the data directory is located. OFF means file names are case sensitive,ON means they are not case sensitive. This variable is read only because it reflects a file system attribute and setting it would have no effect on the file system.


lower_case_table_names解释可参见[官方文档](http://dev.mysql.com/doc/refman/5.6/en/server-system-variables.html#sysvar_lower_case_table_names)的说明：

> [lower_case_table_names](http://dev.mysql.com/doc/refman/5.6/en/server-system-variables.html#sysvar_lower_case_table_names), If set to 0, table names are stored as specified and comparisons are case sensitive. If set to 1, table names are stored in lowercase on disk and comparisons are not case sensitive. If set to 2, table names are stored as given but compared in lowercase. This option also applies to database names and table aliases. For additional information, see [Section 9.2.2](http://dev.mysql.com/doc/refman/5.6/en/identifier-case-sensitivity.html), “Identifier Case Sensitivity”.

> On Windows the default value is 1. On OS X, the default value is 2.

> You should *not* set **lower_case_table_names** to 0 if you are running MySQL on a system where the data directory resides on a case-insensitive file system (such as on Windows or OS X). It is an unsupported combination that could result in a hang condition when running an INSERT INTO ... SELECT ... FROM **tbl_name** operation with the wrong **tbl_name** letter case. With MyISAM, accessing table names using different letter cases could cause index corruption.


#### 解决方案
关于Hibernate/JPA数据库schema自动映射时，对于Linux上MySQL大小写敏感解决方案为：
* **方案一: 设计时在数据库中命名都采用 小写字母或小写字母+下划线 的方式。**
* **方案二: 用root登录，修改/etc/mysql/my.cnf, 在[mysqld]下加入一行：lower_case_table_names=1，重启数据库。**

**但是特别注意：**
> As of MySQL 5.6.27, an error message is printed and the server exits if you attempt to start the server with --lower_case_table_names=0 on a case-insensitive file system.

若需要设置lower_case_table_names = 1时，在重启数据库实例之前就需要将原来的数据库和表转换为小写，否则将找不到数据库名。而数据库名无法直接更名，可以新建一个小写的数据库名，然后rename table到新的数据库，完成表的迁移。