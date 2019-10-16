---
title: Centos关闭防火墙
date: 2019-10-16 14:02:59
tags: Centos
---
# 查看防火墙是否开启
```
 firewall-cmd --state
```
# 关闭防火墙
`
systemctl stop firewalld.service
`
# 禁止firewall开机启动
`systemctl disable firewalld.service `