---
title: CentOS 7 安装 MongoDB 3.6
date: 2018-03-16 10:18:36
tags: MongoDB
categories: centOS 7 
---

# CentOS 7 安装 MongoDB 3.6
## 安装步骤

### 创建文件
```
vi /etc/yum.repos.d/mongodb-org-3.6.repo
```
### 文件内容
```
[mongodb-org-3.6]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/amazon/2013.03/mongodb-org/3.6/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-3.6.asc
```
### 安装
```
sudo yum install -y mongodb-org
```
要安装特定版本的MongoDB，请分别指定每个组件包并将版本号附加到包名称，如下例所示：  

```
sudo yum install -y mongodb-org-3.6.3 mongodb-org-server-3.6.3 mongodb-org-shell-3.6.3 mongodb-org-mongos-3.6.3 mongodb-org-tools-3.6.3
```
您可以指定任何可用的MongoDB版本。然而，yum会在新版本可用时升级软件包。为防止意外升级，请钉住包装。要固定软件包，请将以下exclude指令添加到/etc/yum.conf文件中：   

```
exclude=mongodb-org,mongodb-org-server,mongodb-org-shell,mongodb-org-mongos,mongodb-org-tools
```
<!--more-->
### 国内服务器启动报错  这个问题可以不解决直接启动 mongodb
问题:

```
2018-03-16T10:59:40.008+0800 W NETWORK  [thread1] Failed to connect to 127.0.0.1:27017, in(checking socket for error after poll), reason: Connection refused
2018-03-16T10:59:40.008+0800 E QUERY    [thread1] Error: couldn't connect to server 127.0.0.1:27017, connection attempt failed :
connect@src/mongo/shell/mongo.js:251:13
@(connect):1:6
exception: connect failed
```
运行命令：  

```
mongod -repair
```
会看到：

```
exception in initAndListen: NonExistentPath: Data directory /data/db not found., terminating
```
创建db:

```
cd /data
mkdir db
```
安装完成后记得启动服务，才能用mongo 登录。


## 常用命令
### 查看mongo安装位置
```
whereis mongod
```
### 查看修改配置文件
```
vi /etc/mongod.conf
```
### 启动mongodb
```
systemctl start mongod.service
or
service mongod start
```
### 重新启动mongodb
```
systemctl restart mongod.service
or
service mongod restart
```
### 停止mongodb
```
systemctl stop mongod.service
or
service mongod stop
```
### 查看mongodb的状态
```
systemctl status mongod.service
```
### mongo登录
```
mongo
```
#### 查看数据库
```
show dbs
```

## 配置安全机制和对外访问权限
### 创建管理员账号
```
mongo
use admin
db.createUser({user:"root",pwd:"root",roles:[{role: "userAdminAnyDatabase", db: "admin"}]})
```
用管理员账号登录
```
mongo -u "root" -p "root" --authenticationDatabase "admin"
```


### 创建某个数据库的允许访问账号
```
mongo
use admin
db.getSiblingDB("spring_boot_test").createUser({"user" : "liang","pwd" : "hahaha", roles: [{"role" : "readWrite", "db" : "spring_boot_test"}]})
```
用这个账号登录
```
mongo  -u liang -password test2 -authenticationDatabase spring_boot_test 
```
### 验证用户是否创建成功
```
db.auth('root','root')
```
### 配置mongoDB登录路径
```
mongodb://user:password@ip:port/db
```
### 修改配置文件
```
vi /etc/mongod.conf
```
找到bindIp:127.0.0.1 改为:

```
bindIp:127.0.0.1,192.168.22.210
```
找到#security,改为:
```
security:
  authorization: enabled
```
保存退出，重启mongo服务

### 添加完用户可能之间出现这个错误
错误1:

```
pam_unix(runuser:session): session opened for user mongod by (uid=0)
mongod.service: control process exited, code=exited status=1
```
解决办法：

```
chown -R mongod:mongod /var/lib/mongo
chown -R mongod:mongod /var/log/mongodb
```

错误2:

```
Error starting mongod. /var/run/mongodb/mongod.pid exists.
```
解决办法：

```
rm /var/run/mongodb/mongod.pid -f
```

## 卸载mongodb
### 停止服务
```
service mongod stop
```
### 删除软件
```
yum erase $(rpm -qa | grep mongodb-org)
```
### 移除日志
```
rm -r /var/log/mongodb
rm -r /var/lib/mongo
```
### 删除数据
```
rm -rf /data/db
```
## 常用命令
```
> show dbs  #显示数据库列表 
> show collections  #显示当前数据库中的集合（类似关系数据库中的表）
> show users  #显示用户
> use <db name>  #切换当前数据库，如果数据库不存在则创建数据库。 
> db.help()  #显示数据库操作命令，里面有很多的命令 
> db.foo.help()  #显示集合操作命令，同样有很多的命令，foo指的是当前数据库下，一个叫foo的集合，并非真正意义上的命令 
> db.foo.find()  #对于当前数据库中的foo集合进行数据查找（由于没有条件，会列出所有数据） 
> db.foo.find( { a : 1 } )  #对于当前数据库中的foo集合进行查找，条件是数据中有一个属性叫a，且a的值为1
> db.dropDatabase()  #删除当前使用数据库
> db.cloneDatabase("127.0.0.1")   #将指定机器上的数据库的数据克隆到当前数据库
> db.copyDatabase("mydb", "temp", "127.0.0.1")  #将本机的mydb的数据复制到temp数据库中
> db.repairDatabase()  #修复当前数据库
> db.getName()  #查看当前使用的数据库，也可以直接用db
> db.stats()  #显示当前db状态
> db.version()  #当前db版本
> db.getMongo()  ＃查看当前db的链接机器地址
> db.serverStatus()  #查看数据库服务器的状态
```

