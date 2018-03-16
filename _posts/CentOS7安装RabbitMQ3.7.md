---
title: CentOS 7 安装 RabbitMQ 3.7
date: 2018-03-16 17:18:36
tags: RabbitMQ
categories: CentOS 7 
---

# CentOS 7 安装 RabbitMQ 3.7
## 安装Erlang
### 安装依赖
```
sudo yum install -y gcc gcc-c++ glibc-devel make ncurses-devel openssl-devel autoconf java-1.8.0-openjdk-devel git
```
### 创建yum源 参考：(https://github.com/rabbitmq/erlang-rpm)
```
sudo vi /etc/yum.repos.d/rabbitmq-erlang.repo
```
<!--more-->
### 添加内容
```
[rabbitmq-erlang]
name=rabbitmq-erlang
baseurl=https://dl.bintray.com/rabbitmq/rpm/erlang/20/el/7
gpgcheck=1
gpgkey=https://dl.bintray.com/rabbitmq/Keys/rabbitmq-release-signing-key.asc
repo_gpgcheck=0
enabled=1
```
### 安装
```
sudo yum install -y erlang
```
### 进入erlang命令行表示成功
```
erl
```
## 安装 socat
```
yum install -y socat
```
## RabbitMQ 安装
官网下载地址：https://www.rabbitmq.com/install-rpm.html

```
sudo rpm -Uvh https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.7.4/rabbitmq-server-3.7.4-1.el7.noarch.rpm
```
>如果遇到erlang已安装且版本正确，但是RabbitMQ检测失败的情况
可以追加参数 —nodeps （不验证软件包依赖）

### systemctl 操作 RabbitMQ服务
```
systemctl start rabbitmq-server
systemctl status rabbitmq-server
systemctl restart rabbitmq-server
#设置为开机启动
systemctl enable rabbitmq-server
```

###开放端口
```
#增加rabbitMQ端口：5672
sudo firewall-cmd --add-port=5672/tcp --permanent
#重新加载防火墙设置
sudo firewall-cmd --reload
```

### 添加管理配置插件
```
#安装web管理页面插件（先启动rabbitmq服务）：
rabbitmq-plugins enable rabbitmq_management
#开放端口
sudo firewall-cmd --add-port=15672/tcp --permanent
#重新加载防火墙配置
sudo firewall-cmd --reload
```
>**访问安装好的服务器下的rabbitmq:http://localhost:15672/ 账号和密码:guest guest**

## Rabbit配置
###添加用户
```
#添加用户
sudo rabbitmqctl add_user admin passworld
#设置用户角色
sudo rabbitmqctl set_user_tags admin administrator
#tag（administrator，monitoring，policymaker，management）
#设置用户权限(接受来自所有Host的所有操作)
sudo rabbitmqctl  set_permissions -p "/" admin '.*' '.*' '.*'  
#查看用户权限
sudo rabbitmqctl list_user_permissions admin
```
### 配置远程访问
```
#修改配置文件
sudo vi /etc/rabbitmq/rabbitmq.config 
#保存以下内容
[
{rabbit, [{tcp_listeners, [5672]}, {loopback_users, ["admin"]}]}
].
```
# 备注
##RabbitMQ常用命令
```
# 添加用户
sudo rabbitmqctl add_user <username> <password>  
# 删除用户
sudo rabbitmqctl delete_user <username>  
# 修改用户密码
sudo rabbitmqctl change_password <username> <newpassword>  
# 清除用户密码（该用户将不能使用密码登陆，但是可以通过SASL登陆如果配置了SASL认证）
sudo rabbitmqctl clear_password <username> 
# 设置用户tags（相当于角色，包含administrator，monitoring，policymaker，management）
sudo rabbitmqctl set_user_tags <username> <tag>
# 列出所有用户
sudo rabbitmqctl list_users  
# 创建一个vhosts
sudo rabbitmqctl add_vhost <vhostpath>  
# 删除一个vhosts
sudo rabbitmqctl delete_vhost <vhostpath>  
# 列出vhosts
sudo rabbitmqctl list_vhosts [<vhostinfoitem> ...]  
# 针对一个vhosts给用户赋予相关权限；
sudo rabbitmqctl set_permissions [-p <vhostpath>] <user> <conf> <write> <read>  
# 清除一个用户对vhosts的权限；
sudo rabbitmqctl clear_permissions [-p <vhostpath>] <username>  
# 列出哪些用户可以访问该vhosts；
sudo rabbitmqctl list_permissions [-p <vhostpath>]   
# 列出用户访问权限；
sudo rabbitmqctl list_user_permissions <username>
```


# 参考：
https://ken.io/note/centos7-rabbitmq-install-setup
https://www.rabbitmq.com/download.html
https://github.com/judasn/Linux-Tutorial/blob/master/RabbitMQ-Install-And-Settings.md
http://blog.csdn.net/zyz511919766/article/details/42292655

