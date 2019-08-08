---
title: nginx配置ssl证书
date: 2019-08-08 15:06:07
tags: nginx
---
配置如下
<!-- more -->
```
server {
        listen 443;
        server_name sm.moldcio.com;  # localhost修改为您证书绑定的域名。
        #error_page 497  https://$host$request_uri; #正常错误反馈转换到https
        ssl on;   #设置为on启用SSL功能。
        #root html;
        #index index.html index.htm;
        ssl_certificate /tomcat/ssl_key/2225091_sm.moldcio.com.pem;   #将domain name.pem替换成您证书的文件名。
        ssl_certificate_key /tomcat/ssl_key/2225091_sm.moldcio.com.key;   #将domain name.key替换成您证书的密钥文件名。
        ssl_session_timeout 5m;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;  #使用此加密套件。
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;   #使用该协议进行配置。
        ssl_prefer_server_ciphers on;   
        location / {
            proxy_set_header Host  $host;
            proxy_set_header X-Real-Ip $remote_addr;
            proxy_set_header X-Forwarded-For $remote_addr;
            proxy_pass http://119.3.9.188;
        }
    }
    server {
    listen       80;
    server_name  sm.moldcio.com;
    
    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Real-Ip $remote_addr;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_pass http://119.3.9.188:8101;
    }
```
