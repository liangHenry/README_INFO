---
title: MySql-用法说明
date: 2017-07-22 10:28:42
tags: MySql
categories: tool 
---

# 安装
## Linux/UNIX上安装Mysql
Linux平台上推荐使用RPM包来安装Mysql,MySQL AB提供了以下RPM包的下载地址：  
* MySQL - MySQL服务器。你需要该选项，除非你只想连接运行在另一台机器上的MySQL服务器。
* MySQL-client - MySQL 客户端程序，用于连接并操作Mysql服务器。
* MySQL-devel - 库和包含文件，如果你想要编译其它MySQL客户端，例如Perl模块，则需要安装该RPM包。
* MySQL-shared - 该软件包包含某些语言和应用程序需要动态装载的共享库(libmysqlclient.so*)，使用MySQL。
* MySQL-bench - MySQL数据库服务器的基准和性能测试工具。

<!--more-->

## Centos 系统下使用 yum 命令安装 MySql： 
检测系统是否自带安装 mysql: `rpm -qa | grep mysql`   
如果你系统有安装，那可以选择进行卸载: `rpm -e mysql`,强力删除`rpm -e --nodeps mysql`  
安装mysql：  
`yum install mysql`  
`yum install mysql-server`  
`yum install mysql-devel`  
启动mysql：`service mysqld start`  
查看mysql版本号：mysqladmin --version
## window上查看mysql
`mysqld.exe --console`


# 命令

## 简单的命令

1. 连接服务器 `mysql`
2. 显示数据库信息 `show databases`
3. 设置密码 `mysqladmin -u root password "new_password"`
4. 登录附加密码`mysql -u root -p`
5. 查看服务是否启动`ps -ef | grep mysqld`
6. 启动mysql服务器`./mysql_safe &`
7. 关闭mysql服务器`./mysqladmin -u root -p shutdown`
8. 切换数据库`use mysql`
9. 重新载入授权表`FLUSH PRIVILEGES`
10. 给指定数据库添加用户 `GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP ON TUTORIALS.* TO 'liang'@'localhost' IDENTIFIED BY 'hello'`
11. 显示表`SHOW TABLES`
12. 显示表结构`SHOW COLUMNS FROM table`
13. 显示数据表的详细索引信息`SHOW INDEX FROM table`
14. 数据库性能`SHOW TABLE STATUS LIKE [FROM db_name] [LIKE 'pattern'] \G`
15. 用户退出`exit`
16. 创建数据库`mysqladmin -u root -p create hello`
17. 删除数据库`mysqladmin -u root -p drop hello`

## 表操作
### 创建表 
`CREATE TABLE table_name (column_name column_type)`
示例解析
```
CREATE TABLE IF NOT EXISTS `t_tbl`(
   `t_id` INT UNSIGNED AUTO_INCREMENT,
   `t_title` VARCHAR(100) NOT NULL,
   `t_author` VARCHAR(40) NOT NULL,
   `submission_date` DATE,
   PRIMARY KEY ( `t_id` )
)ENGINE=InnoDB DEFAULT CHARSET=utf8;
```
实例解析：  
* 如果你不想字段为`NULL`可以设置字段的属性为`NOT NULL`，在操作数据库时如果输入该字段的数据为`NULL`，就会报错。
* `AUTO_INCREMENT`定义列为自增的属性，一般用于主键，数值会自动加1。
* `PRIMARY KEY`关键字用于定义列为主键。 您可以使用多列来定义主键，列间以逗号分隔。
* `ENGINE`设置存储引擎，`CHARSET`设置编码。
* MySQL命令终止符为分号 (;)   

### 删除表
`drop table t_tbl`

### 删除列
` ALTER TABLE t_tbl  DROP i`

### 添加列
`ALTER TABLE t_tbl ADD i INT [FIRST] [AFTER a]`

### 修改列
`ALTER TABLE t_tbl MODIFY c CHAR(10) [not null] [default];`  
`ALTER TABLE t_tbl CHANGE i j BIGINT;`  
`ALTER TABLE t_tbl CHANGE j j INT`  
### 修改默认值
`ALTER TABLE testalter_tbl ALTER i SET DEFAULT 1000`  
`ALTER TABLE testalter_tbl ALTER i DROP DEFAULT `
### 修改表名
`ALTER TABLE testalter_tbl RENAME TO alter_tbl`
### 修改存储索引
`alter table tableName engine=myisam`
### 删除外键约束
`alter table tableName drop foreign key keyName`
### 修改位置
`alter table tableName modify name1 type1 first|after name2`

### 索引操作
1. 创建索引`CREATE INDEX indexName ON mytable(username(length))`;
2. 修改索引`ALTER mytable ADD INDEX [indexName] ON (username(length)) `
3. 创建表时直接增加索引`CREATE TABLE mytable(username VARCHAR(16) NOT NULL,INDEX [indexName] (username(length)))`
4. 删除索引`DROP INDEX [indexName] ON mytable`
5. 创建唯一索引`CREATE UNIQUE INDEX indexName ON mytable(username(length)) `
6. 修改唯一索引`ALTER table mytable ADD UNIQUE [indexName] (username(length))`
7. 创建表时直接创建唯一索引`CREATE TABLE mytable(username VARCHAR(16) NOT NULL,UNIQUE [indexName] (username(length)))`
8. 添加和删除索引
    8.1. `ALTER TABLE tbl_name ADD PRIMARY KEY (column_list)`: 该语句添加一个主键，这意味着索引值必须是唯一的，且不能为NULL。
    8.2. `ALTER TABLE tbl_name ADD UNIQUE index_name (column_list)`: 这条语句创建索引的值必须是唯一的（除了NULL外，NULL可能会出现多次）
    8.3. `ALTER TABLE tbl_name ADD INDEX index_name (column_list)`: 添加普通索引，索引值可出现多次
    8.4. `ALTER TABLE tbl_name ADD FULLTEXT index_name (column_list)`:该语句指定了索引为 FULLTEXT ，用于全文索引  
9. 添加和删除主键
    9.1. `ALTER TABLE testalter_tbl MODIFY i INT NOT NULL;`添加主键索引时
    9.2. `ALTER TABLE testalter_tbl ADD PRIMARY KEY (i)`添加主键索引时
    9.3. `ALTER TABLE testalter_tbl DROP PRIMARY KEY`删除主键
10. 显示索引 `SHOW INDEX FROM table_name; \G`  

## 数据操作
1. 插入数据:`INSERT INTO table_name ( field1, field2,...fieldN ) VALUES ( value1, value2,...valueN );`
2. 查询数据:`SELECT column_name,column_name FROM table_name [WHERE Clause] [OFFSET M ] [LIMIT N]`
3. 更新数据:`UPDATE table_name SET field1=new-value1, field2=new-value2 [WHERE Clause]`
4. 删除数据:`DELETE FROM table_name [WHERE Clause]`
5. 模糊查询:`SELECT * FROM table_name WHERE field1 LIKE condition1 [AND [OR]] filed2 = 'somevalue'`
6. 连接查询结果:`SELECT * FROM tables  UNION [ALL | DISTINCT] SELECT * FROM tables `
7. 排序:`SELECT fieldN table_name1ORDER BY field1 [ASC [DESC]]`
8. 分组:`SELECT column_name, function(column_name) FROM table_name WHERE column_name operator value GROUP BY column_name;`
9. 内外连接
  9.1 INNER JOIN（内连接,或等值连接）：获取两个表中字段匹配关系的记录
  9.2 LEFT JOIN（左连接）：获取左表所有记录，即使右表没有对应匹配的记录
  9.3 RIGHT JOIN（右连接）： 与 LEFT JOIN 相反，用于获取右表所有记录，即使左表没有对应匹配的记录
10. 空值判断:`is null` `is not null` `null <=> null` is true `null=null` is false.
11. 正则表达式:`SELECT * FROM table_name WHERE field1 REGEXP 'mar'



 
# 数据类型
## 数值类型
| 类型		|大小 		|范围（有符号） 	|范围（无符号） 	|用途|
|:---		|:---		|:---			|:---			|:---|
|TINYINT 	|1 字节 	|(-128，127) 		|(0，255) 		|小整数值|
|SMALLINT 	|2 字节 	|(-32,768，32,767) 	|(0，65 535) 		|大整数值|
|MEDIUMINT 	|3 字节 	|(-8,388,608，8 388 607) 	|(0，16 777 215) 	|大整数值|
|INT或INTEGER 	|4 字节 	|(-2,147,483,648，2,147,483,647) 	|(0，4 294 967 295) 	|大整数值|
|BIGINT 	|8 字节 	|(-9,233,372,036,854,775,808，<br />9,223,372,036,854,775,807) 	|(0，18 446 744 073 709 551 615) 	|极大整数值|
|FLOAT 		|4 字节 	|(-3.402 823 466 E+38<br />，-1.175 494 351 E-38)，<br />0，(1.175 494 351 E-38，<br />3.402 823 466 351 E+38) 	|约值0，(1.175 E-38，<br />3.402 E+38) 	|单精度浮点数值|
|DOUBLE 	|8 字节 	|约值(-1.797 E+308，<br />-2.225 E-308)，0，<br />(2.225 E-308，<br />1.797 E+308) 	|约值0，(2.225 E-308，<br />1.797 E+308) 	|双精度浮点数值|
|DECIMAL 	|对DECIMAL(M,D) ，<br />如果M>D，<br />为M+2否则为D+2 	|依赖于M和D的值 	|依赖于M和D的值 	|小数值| 

## 日期和时间类型
|类型 	|大小(字节) 	|范围 	|格式 	|用途|
|:---	|:---		|:---	|:---	|:---|
|DATE 	|3 	|1000-01-01/9999-12-31 	|YYYY-MM-DD 	|日期值|
|TIME 	|3 	|'-838:59:59'<br />'838:59:59' 	|HH:MM:SS 	|时间值或持续时间|
|YEAR 	|1 	|1901/2155 	|YYYY 	|年份值|
|DATETIME 	|8 	|1000-01-01 00:00:00<br />9999-12-31 23:59:59 	|YYYY-MM-DD HH:MM:SS 	|混合日期和时间值|
|TIMESTAMP 	|4 	|1970-01-01 00:00:00<br />2037 年某时 	|YYYYMMDD HHMMSS 	|混合日期和时间值，时间戳|

## 字符串类型

|类型 	|大小 	|用途|
|:---	|:---	|:---|
|CHAR 	|0-255字节 	|定长字符串|
|VARCHAR 	|0-65535 字节 	|变长字符串|
|TINYBLOB 	|0-255字节 	|不超过 255 个字符的二进制字符串|
|TINYTEXT 	|0-255字节 	|短文本字符串|
|BLOB 	|0-65 535字节 	|二进制形式的长文本数据|
|TEXT 	|0-65 535字节 	|长文本数据|
|MEDIUMBLOB 	|0-16 777 215字节 	|二进制形式的中等长度文本数据|
|MEDIUMTEXT 	|0-16 777 215字节 	|中等长度文本数据|
|LONGBLOB 	|0-4 294 967 295字节 	|二进制形式的极大文本数据|
|LONGTEXT 	|0-4 294 967 295字节 	|极大文本数据|
说明:
CHAR和VARCHAR类型类似，但它们保存和检索的方式不同。它们的最大长度和是否尾部空格被保留等方面也不同。在存储或检索过程中不进行大小写转换。  
BINARY和VARBINARY类类似于CHAR和VARCHAR，不同的是它们包含二进制字符串而不要非二进制字符串。也就是说，它们包含字节字符串而不是字符字符串。这说明它们没有字符集，并且排序和比较基于列值字节的数值值。  
BLOB是一个二进制大对象，可以容纳可变数量的数据。有4种BLOB类型：TINYBLOB、BLOB、MEDIUMBLOB和LONGBLOB。它们只是可容纳值的最大长度不同。  
有4种TEXT类型：TINYTEXT、TEXT、MEDIUMTEXT和LONGTEXT。这些对应4种BLOB类型，有相同的最大长度和存储需求。   

## 正则表达式
|模式	|描述|
|:---	|:---	|
|^	|匹配输入字符串的开始位置。如果设置了 RegExp 对象的 Multiline 属性，^ 也匹配 '\n' 或 '\r' 之后的位置。|
|\$	|匹配输入字符串的结束位置。如果设置了RegExp 对象的 Multiline 属性，$ 也匹配 '\n' 或 '\r' 之前的位置。|
|.	|匹配除 "\n" 之外的任何单个字符。要匹配包括 '\n' 在内的任何字符，请使用象 '[.\n]' 的模式。|
|[...]	|字符集合。匹配所包含的任意一个字符。例如， '[abc]' 可以匹配 "plain" 中的 'a'。|
|[^...]	|负值字符集合。匹配未包含的任意字符。例如， '[^abc]' 可以匹配 "plain" 中的'p'。|
|p1｜p2｜p3	|匹配 p1 或 p2 或 p3。例如，`'z｜food'` 能匹配 "z" 或 "food"。`'(z｜f)ood'` 则匹配 "zood" 或 "food"。|
|*	|匹配前面的子表达式零次或多次。例如，zo* 能匹配 "z" 以及 "zoo"。* 等价于{0,}。|
|+	|匹配前面的子表达式一次或多次。例如，'zo+' 能匹配 "zo" 以及 "zoo"，但不能匹配 "z"。+ 等价于 {1,}。|
|{n}	|n 是一个非负整数。匹配确定的 n 次。例如，'o{2}' 不能匹配 "Bob" 中的 'o'，但是能匹配 "food" 中的两个 o。|
|{n,m}	|m 和 n 均为非负整数，其中n <= m。最少匹配 n 次且最多匹配 m 次。|

# mysql事物

 MySQL 事务主要用于处理操作量大，复杂度高的数据。比如说，在人员管理系统中，你删除一个人员，你即需要删除人员的基本资料，也要删除和该人员相关的信息，如信箱，文章等等，这样，这些数据库操作语句就构成一个事务！
  * 在 MySQL 中只有使用了 Innodb 数据库引擎的数据库或表才支持事务。
  * 事务处理可以用来维护数据库的完整性，保证成批的 SQL 语句要么全部执行，要么全部不执行。
  * 事务用来管理 insert,update,delete 语句

一般来说，事务是必须满足4个条件（ACID）： Atomicity（原子性）、Consistency（稳定性）、Isolation（隔离性）、Durability（可靠性）
 1. 事务的原子性：一组事务，要么成功；要么撤回。  
 2. 稳定性:有非法数据（外键约束之类），事务撤回。
 3. 隔离性:事务独立运行。一个事务处理后的结果，影响了其他事务，那么其他事务会撤回。事务的100%隔离，需要牺牲速度。
 4. 可靠性:软、硬件崩溃后，InnoDB数据表驱动会利用日志文件重构修改。可靠性和高速度不可兼得， `innodb_flush_log_at_trx_commit` 选项 决定什么时候吧事务保存到日志里。

>在 MySQL 命令行的默认设置下，事务都是自动提交的，
即执行 SQL 语句后就会马上执行 COMMIT 操作。
因此要显式地开启一个事务务须使用命令 BEGIN 或 START TRANSACTION，
或者执行命令 SET AUTOCOMMIT=0，用来禁止使用当前会话的自动提交。


## 事物控制语句：
  * BEGIN或START TRANSACTION；显式地开启一个事务；
  * COMMIT；也可以使用COMMIT WORK，不过二者是等价的。COMMIT会提交事务，并使已对数据库进行的所有修改称为永久性的；
  * ROLLBACK；有可以使用ROLLBACK WORK，不过二者是等价的。回滚会结束用户的事务，并撤销正在进行的所有未提交的修改；
  * SAVEPOINT identifier；SAVEPOINT允许在事务中创建一个保存点，一个事务中可以有多个SAVEPOINT；
  * RELEASE SAVEPOINT identifier；删除一个事务的保存点，当没有指定的保存点时，执行该语句会抛出一个异常；
  * ROLLBACK TO identifier；把事务回滚到标记点；
  * SET TRANSACTION；用来设置事务的隔离级别。InnoDB存储引擎提供事务的隔离级别有READ UNCOMMITTED、READ COMMITTED、REPEATABLE READ和SERIALIZABLE。

## MYSQL 事务处理主要有两种方法：

1、用 BEGIN, ROLLBACK, COMMIT来实现
  * BEGIN 开始一个事务
  * ROLLBACK 事务回滚
  * COMMIT 事务确认

2、直接用 SET 来改变 MySQL 的自动提交模式:
  * SET AUTOCOMMIT=0 禁止自动提交
  * SET AUTOCOMMIT=1 开启自动提交

# 临时表
```
CREATE TEMPORARY TABLE SalesSummary (
    product_name VARCHAR(50) NOT NULL, 
    total_sales DECIMAL(12,2) NOT NULL DEFAULT 0.00, 
    avg_unit_price DECIMAL(7,2) NOT NULL DEFAULT 0.00,
    total_units_sold INT UNSIGNED NOT NULL DEFAULT 0
);
```

# 元数据
|命令	|描述|
|:---	|:---	|
|SELECT VERSION( )|服务器版本信息|
|SELECT DATABASE( )|当前数据库名 (或者返回空)|
|SELECT USER( )	|当前用户名|
|SHOW STATUS	|服务器状态|
|SHOW VARIABLES|	服务器配置变量|
    
    
# 序列
```
CREATE TABLE insect
    -> (
    -> id INT UNSIGNED NOT NULL AUTO_INCREMENT,
    -> PRIMARY KEY (id),
    -> name VARCHAR(30) NOT NULL, # type of insect
    -> date DATE NOT NULL, # date collected
    -> origin VARCHAR(30) NOT NULL # where collected
)engine=innodb auto_increment=100 charset=utf8;
```
```
mysql> ALTER TABLE insect DROP id;
mysql> ALTER TABLE insect
    -> ADD id INT UNSIGNED NOT NULL AUTO_INCREMENT FIRST,
    -> ADD PRIMARY KEY (id);
```
```
ALTER TABLE t AUTO_INCREMENT = 100;
```

# 删除重复数据
```
mysql> CREATE TABLE tmp SELECT last_name, first_name, sex FROM person_tbl  GROUP BY (last_name, first_name, sex);
mysql> DROP TABLE person_tbl;
mysql> ALTER TABLE tmp RENAME TO person_tbl;
```  

# 导出数据
```
SELECT * FROM runoob_tbl 
INTO OUTFILE '/tmp/tutorials.txt';
```
以下实例为导出 CSV 格式：
```
SELECT * FROM passwd INTO OUTFILE '/tmp/tutorials.txt'
FIELDS TERMINATED BY ',' ENCLOSED BY '"'
LINES TERMINATED BY '\r\n';
```
```
SELECT a,b,a+b INTO OUTFILE '/tmp/result.text'
FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\n'
FROM test_table;
```

导出表作为原始数据
```
mysqldump -u root -p --no-create-info \
            --tab=/tmp RUNOOB runoob_tbl
password ******
```

导出SQL格式的数据到指定文件，如下所示：
```
$ mysqldump -u root -p RUNOOB runoob_tbl > dump.txt
password ******
```
如果你需要导出整个数据库的数据，可以使用以下命令：
```
$ mysqldump -u root -p RUNOOB > database_dump.txt
password ******
```
如果需要备份所有数据库，可以使用以下命令：
```
$ mysqldump -u root -p --all-databases > database_dump.txt
password ******
```
将数据表及数据库拷贝至其他主机
如果你需要将数据拷贝至其他的 MySQL 服务器上, 你可以在 mysqldump 命令中指定数据库名及数据表。
在源主机上执行以下命令，将数据备份至 dump.txt 文件中:
```
$ mysqldump -u root -p database_name table_name > dump.txt
password *****
```
如果完整备份数据库，则无需使用特定的表名称。
如果你需要将备份的数据库导入到MySQL服务器中，可以使用以下命令，使用以下命令你需要确认数据库已经创建：
```
$ mysql -u root -p database_name < dump.txt
password *****
```
你也可以使用以下命令将导出的数据直接导入到远程的服务器上，但请确保两台服务器是相通的，是可以相互访问的：</p>
```
$ mysqldump -u root -p database_name \
       | mysql -h other-host.com database_name
```  

# 导入数据
```
LOAD DATA LOCAL INFILE 'dump.txt' INTO TABLE mytbl;
```
```
mysql> LOAD DATA LOCAL INFILE 'dump.txt' INTO TABLE mytbl
  -> FIELDS TERMINATED BY ':'
  -> LINES TERMINATED BY '\r\n';
```
```
mysql> LOAD DATA LOCAL INFILE 'dump.txt' 
    -> INTO TABLE mytbl (b, c, a);
```
```
$ mysqlimport -u root -p --local database_name dump.txt
password *****
```
```
$ mysqlimport -u root -p --local --fields-terminated-by=":" \
   --lines-terminated-by="\r\n"  database_name dump.txt
password *****
```
```
$ mysqlimport -u root -p --local --columns=b,c,a \
    database_name dump.txt
password *****
```
