---
title: docker常用命令
date: 2019-08-08 14:40:11
tags: docker
---

# docker logs 查看实时日志
```
docker logs -f -t --since="2019-01-12" --tail=200 upms-web
```
--since : 此参数指定了输出日志开始日期，即只输出指定日期之后的日志。

-f : 查看实时日志

-t : 查看日志产生的日期

-tail=10 : 查看最后的10条日志。

upms-web : 容器名称
<!-- more -->

# 进入tomcat容器 查看jdk版本

```linux
docker  exec -it upms-web /bin/bash

java -version
```

# 重启指定容器

```
docker restart [容器名称]
```

# 查看挂载

```
docker inspect nginx
```

![](https://raw.githubusercontent.com/luckybearbear/img/master/hexo/Image.png)

# docker 生成镜像

```
docker-compose up
```

# 复制日志

```
docker cp mes-web:/usr/local/logs/mes-web.log ~
```



