---
title: docker安装并使用Nexus3
date: 2019-12-03 15:23:39
tags: Nexus
---
# 安装
## 拉取nexus3镜像
```
docker pull docker.io/sonatype/nexus3
```
 <!-- more -->
## 运行nexus容器
`docker run -d -p 8081:8081 --restart=always --name nexus -v nexus-data:/home/nexus sonatype/docker-nexus3`
```
注释：
docker run   -d            ##后台启动 
-p 8081:8081               ##映射端口：容器端口
--restart=always           ##容器重启策略
--name nexus               ##容器命名
-v nexus-data:/home/nexus  ##数据挂载到 /home/nexus 例如nexus下载包放入宿主机该位置
sonatype/docker-nexus3     ##指定nexus3镜像
```
## 浏览器访问
`http://ip:8081`
登录账号：admin
登录密码：admin123

