---
title: windows安裝VMware
date: 2019-10-16 12:21:09
tags:  
  - VMware
  - Centos
---
# 安装VMware12
 <!-- more -->
傻瓜式安装！忽略
# 安装Centos7
## 右击刚创建的虚拟机，选择设置
进入阿里云站点：[http://mirrors.aliyun.com/centos/7/isos/x86_64/](http://mirrors.aliyun.com/centos/7/isos/x86_64/)，选择 CentOS-7-x86_64-DVD-1804.iso下载
![](https://raw.githubusercontent.com/luckybearbear/img/master/hexo/20191016122705.png)
## 点击开启此虚拟机
执行常规安装操作
![](https://raw.githubusercontent.com/luckybearbear/img/master/hexo/20191016123425.png)
## 桥接模式下设置固定ip
```
vi   /etc/sysconfig/network-scripts/ifcfg-ens33
```
修改如下
```
ONBOOT=yes  # 该网卡是否随网络服务启动
IPADDR=192.168.220.101  # 该网卡ip地址就是你要配置的固定IP，如果你要用xshell等工具连接，220这个网段最好和你自己的电脑网段一致，否则有可能用xshell连接失败
GATEWAY=192.168.220.2   # 网关
NETMASK=255.255.255.0   # 子网掩码
```
## 重启网络服务
```
service network restart
``` 
输入ifconfig显示如下代表成功
![](https://raw.githubusercontent.com/luckybearbear/img/master/hexo/20191016123949.png)
