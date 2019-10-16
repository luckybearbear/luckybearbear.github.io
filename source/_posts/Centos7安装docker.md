---
title: Centos7安装docker
date: 2019-10-16 12:52:20
tags:
  - docker
  - Centos
---
# 前言
## Docker需求配置
lunix内核，要求3.8以上
centos7
Docker是一个进程，一启动就两个进程，一个服务，一个守护进程。占用资源就非常少，启动速度非常快，1s。
一台机器上vm，3到10个实例。docker 100到10000。
<!-- more -->
# 安装Docker
## 安装工具包
```
sudo yum install -y yum-utils 		#安装工具包，缺少这些依赖将无法完成
```
执行结果如下
```
Loaded plugins: fastestmirror, langpacks
base                                                                                          | 3.6 kB  00:00:00
epel                                                                                          | 4.3 kB  00:00:00
extras                                                                                        | 3.4 kB  00:00:00
update                                                                                        | 3.4 kB  00:00:00
(1/3): epel/7/x86_64/updateinfo                                                               | 797 kB  00:00:00
(2/3): epel/7/x86_64/primary_db                                                               | 4.7 MB  00:00:00
(3/3): update/7/x86_64/primary_db                                                             | 4.8 MB  00:00:00
Loading mirror speeds from cached hostfile
Package yum-utils-1.1.31-40.el7.noarch already installed and latest version
Nothing to do
```
## 设置远程仓库
```
$sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
执行结果：
Loaded plugins: fastestmirror, langpacks
adding repo from: https://download.docker.com/linux/centos/docker-ce.repo
grabbing file https://download.docker.com/linux/centos/docker-ce.repo to /etc/yum.repos.d/docker-ce.repo
repo saved to /etc/yum.repos.d/docker-ce.repo
```
## 安装
```
yum install docker-io
```
## 启动
```
$ sudo systemctl start docker
或者
$ sudo service docker start
service docker start        #启动docker
chkconfig docker on         #加入开机启动
```
# 查看Docker版本
```
docker --help #帮助
docker –v #简单查看版本
docker version #查看版本
```