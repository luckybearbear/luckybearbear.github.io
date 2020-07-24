---
title: YApi可视化接口管理平台centos7部署
date: 2020-07-24 11:45:18
tags: 工具
---


## YApi 可视化接口管理平台centos7部署

# 安装nodejs

## 下载

```
wget https://npm.taobao.org/mirrors/node/v12.4.0/node-v12.4.0-linux-x64.tar.xz
```

## 解压

```
tar -xvf node-v12.4.0-linux-x64.tar.xz
```

## 进入bin目录，执行ls命令

```
cd node-v12.4.0-linux-x64/bin && ls
```
<!-- more -->
### 测试

```
./node -v
```

安装成功，这是node 和 npm还不能全局使用，需要做关联。

## 关联

```
ln -s /node-v12.4.0-linux-x64/bin/node /usr/local/bin/node
ln -s /node-v12.4.0-linux-x64/bin/npm /usr/local/bin/npm
```

 这样就可以在任何目录下执行 `node` 和 `npm` 命令。

# 安装MongoDB

## 下载

`wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-4.2.8.tgz`

## 解压与转移

```
 tar -zxvf mongodb-linux-x86_64-rhel70-4.2.8.tgz
```

```
mv mongodb-linux-x86_64-rhel70-4.2.8 /usr/local/mongodb
```

## 配置conf与目录

### 进入mongodb目录

```
cd /usr/local/mongodb/
```

### 创建db目录和日志文件

```
mkdir -p ./data/db

mkdir -p ./logs

touch ./logs/mongodb.log
```

### 创建mongodb.conf文件

vim mongodb.conf

```
#端口号
port=27017
#db目录
dbpath=/usr/local/mongodb/data/db
#日志目录
logpath=//usr/local/mongodb/logs/mongodb.log
#后台
fork=true
#日志输出
logappend=true
#允许远程IP连接
bind_ip=0.0.0.0
```

## 启动

```
./bin/mongod --config mongodb.conf
```

## 连接

```
./bin/mongo
```



# 安装git

```
yum -y install git
```

# 安装YApi

参考官网