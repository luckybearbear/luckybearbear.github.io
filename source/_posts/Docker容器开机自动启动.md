---
title: Docker容器开机自动启动
date: 2019-10-16 13:09:20
tags: docker
---
#前言
 部署项目服务器时，为了应对停电等情况影响正常web项目的访问，会把Docker容器设置为开机自动启动。
 <!-- more -->
 # 初次运行容器
 使用--restart参数来设置
 ```
 docker run -m 512m --memory-swap 1G -it -p 58080:8080 --restart=always   
 --name bvrfis --volumes-from logdata mytomcat:4.0 /root/run.sh  
 ```
 restart具体参数值详细信息：
        no -  容器退出时，不重启容器；
        on-failure - 只有在非0状态退出时才从新启动容器；
        always - 无论退出状态是如何，都重启容器；
# 创建时未指定 --restart=always
可通过update 命令设置
```
docker update --restart=always xxx 
```