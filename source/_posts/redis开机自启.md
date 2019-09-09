---
title: redis开机自启
date: 2019-08-08 15:27:25
tags: redis
categories: 工具
---
# window
```
redis-server --service-install redis.windows.conf --loglevel verbose 
```
# linux
## 设置redis.conf中daemonize为yes,确保守护进程开启。
 查找redis配置文件redis.conf
 <!-- more -->
```
[root@localhost /]# find / -name redis.conf
/usr/local/redis/redis.conf
```
 编辑redis配置文件
```
[root@localhost ~]# vim /usr/local/redis/redis.conf
```
命令行模式下输入 /daemonize 查找,将配置文件中daemonize为yes
## 编写开机自启动脚本
vi /etc/init.d/redis
```
# chkconfig: 2345 10 90  
# description: Start and Stop redis   
  
PATH=/usr/local/bin:/sbin:/usr/bin:/bin   
REDISPORT=6379  
  EXEC=/usr/local/redis/redis-server
  REDIS_CLI=/usr/local/redis/redis-cli

　PIDFILE=/var/run/redis.pid
  CONF="/usr/local/redis/redis.conf"

case "$1" in   
        start)   
                if [ -f $PIDFILE ]   
                then   
                        echo "$PIDFILE exists, process is already running or crashed."  
                else  
                        echo "Starting Redis server..."  
                        $EXEC $CONF   
                fi   
                if [ "$?"="0" ]   
                then   
                        echo "Redis is running..."  
                fi   
                ;;   
        stop)   
                if [ ! -f $PIDFILE ]   
                then   
                        echo "$PIDFILE exists, process is not running."  
                else  
                        PID=$(cat $PIDFILE)   
                        echo "Stopping..."  
                       $REDIS_CLI -p $REDISPORT  SHUTDOWN    
                        sleep 2  
                       while [ -x $PIDFILE ]   
                       do  
                                echo "Waiting for Redis to shutdown..."  
                               sleep 1  
                        done   
                        echo "Redis stopped"  
                fi   
                ;;   
        restart|force-reload)   
                ${0} stop   
                ${0} start   
                ;;   
        *)   
               echo "Usage: /etc/init.d/redis {start|stop|restart|force-reload}" >&2  
                exit 1  
esac
```

## 设置权限
```
[root@localhost init.d]# chmod 777 /etc/init.d/redis
```
