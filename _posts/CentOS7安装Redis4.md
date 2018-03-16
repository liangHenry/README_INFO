---
title: CentOS 7 安装 Redis4.0.2
date: 2018-03-16 10:18:36
tags: Redis
categories: CentOS 7
---

# CentOS 7 安装 Redis4.0.2
## 安装步骤

### 安装基础依赖
```
sudo yum install -y gcc gcc-c++ make jemalloc-devel epel-release
```
### 下载Redis（ https://redis.io/download ）
```
wget http://download.redis.io/releases/redis-4.0.2.tar.gz
```
### 解压tar到指定目录
```
mkdir /usr/redis
tar -zvxf redis-4.0.2.tar.gz -C /usr/redis
```
### 编译&安装
```
cd /usr/redis/redis-4.0.2
make & make install
```
### 启动redis-server
```
cd /usr/redis/redis-4.0.2/src
./redis-server
```
### 启动redis客户端测试
```
cd /usr/redis/redis-4.0.2/src
./redis-cli
```

## Redis配置

### 修改配置文件
```
vi /usr/redis/redis-4.0.2/redis.conf
```
#### 更改绑定
```
#将bind 127.0.0.1 更换为本机IP，例如：192.168.22.210
bind 192.168.22.210
```
#### 关闭保护模式
```
protected-mode no
```
#### 增加密码
```
#requirepass foobared
#去掉前面的注释，并修改为所需要的密码：（其中myPassword就是要设置的密码）
requirepass myPassword
```
##### 密码登录
```
./redis-cli -h 127.0.0.1 -p 6379

127.0.0.1:6379>auth myPassword
```
### 开发端口
```
firewall-cmd --add-port=6379/tcp --permanent
firewall-cmd --reload
```
### Redis 指定配置文件启动
```
cd /usr/redis/redis-4.0.2/src
./redis-server redis.conf
```
### Redis客户端连接指定Redis Server
```
cd /usr/redis/redis-4.0.2/src
./redis-cli -h 192.168.11.11
```
### 配置Redis开机启动
#### 创建服务文件
```
vi /usr/lib/systemd/system/redis.service
```
##### 文件内容
```
[Unit]
Description=Redis Server
After=network.target

[Service]
ExecStart=/usr/redis/redis-4.0.2/src/redis-server /usr/redis/redis-4.0.2/redis.conf --daemonize no
ExecStop=/usr/redis/redis-4.0.2/src/redis-cli -p 6379 shutdown
Restart=always

[Install]
WantedBy=multi-user.target
```

#### 设置Redis服务开机启动
```
systemctl enable redis
```

#### 启动Redis服务
```
sudo systemctl start redis
```

