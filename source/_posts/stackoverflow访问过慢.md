---
title: stackoverflow访问过慢
date: 2019-10-11 15:49:59
tags: 论坛
---
  # 介绍
  stackoverflow是开发常用的提问和解决代码问题网站，但自己访问总是非常的慢，几十秒甚至几分钟。
  网站：[https://stackoverflow.com/](https://stackoverflow.com/)
  # 分析
  ## 墙太厚
  stackoverflow只是访问速度很慢，但终归可以打开，如果被墙不可能最后打开的，所以排除。
  ## 请求阻塞
  访问jquery.min.js资源阻塞。
  host添加一下
  ```
  127.0.0.1       ajax.googleapis.com
  ```