---
title: CentOS 7 安装 MySql7-5
date: 2018-02-26 10:18:36
tags: MySql
categories: CentOS 7 
---

# CentOS 7 安装 mysql7-5

##用yum安装
\# yum install mysql
\# wget http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm
\# rpm -ivh mysql-community-release-el7-5.noarch.rpm
\# yum install mysql-community-server
\# yum install mysql-devel
##重新启动
\# service mysqld restart
<!--more-->
##设置密码
mysql -u root
mysql> set password for 'root'@'localhost' =password('root');
##远程设置

```
    把在所有数据库的所有表的所有权限赋值给位于所有IP地址的root用户。
    mysql> grant all privileges on *.* to root@'%'identified by 'root';
```


##开启防火墙设置
\# firewall-cmd --zone=public --add-port=3306/tcp --permanent
\# firewall-cmd --reload

##设置 /etc/my.cnf
[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
###*在后面增加：*
[mysql]
default-character-set=utf8
[mysqld]
wait_timeout=31536000
interactive_timeout=31536000
default-storage-engine=INNODB
character-set-server=utf8
collation-server=utf8_general_ci
[client]
default-character-set=utf8

##设置开机启动mysql
systemctl enable mysqld.service;
systemctl daemon-reload