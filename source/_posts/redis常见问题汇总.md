---
title: redis常见问题汇总
date: 2019-10-15 14:20:34
tags: redis
categories: redis
---
# 服务没有及时响应启动或控制请求
![问题1](https://raw.githubusercontent.com/luckybearbear/img/master/hexo/20191015142101.png)
redis.windows.conf配置文件存在空格，去除空格即可。
![](https://raw.githubusercontent.com/luckybearbear/img/master/hexo/20191015142241.png)