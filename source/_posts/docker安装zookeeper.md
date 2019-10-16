---
title: docker安装zookeeper
date: 2019-10-16 14:33:42
tags:
  - docker
  - zookeeper
---
# 下载zookeeper镜像
`docker pull zookeeper`

# 启动容器并添加映射
`docker run --privileged=true -d --name zookeeper --publish 2181:2181  -d zookeeper:latest`

# 查看容器是否启动
`docker ps`