---
title: github常用命令
date: 2019-08-08 14:18:42
tags: github
categories: 工具
---
## 查看这个月star最多的项目
```aidl
https://github.com/trending/java?since=monthly
[java]可以替换成其他语言
```
<!-- more -->
## 常用命令
git init 新建一个空的仓库
git status 查看状态
git add . 添加文件
git commit -m '注释' 提交添加的文件并备注说明
git remote add origin git@github.com:jinzhaogit/git.git 连接远程仓库
git push -u origin master 将本地仓库文件推送到远程仓库
git log 查看变更日志
git reset --hard 版本号前六位 回归到指定版本
git branch 查看分支
git branch newname 创建一个叫newname的分支
git checkout newname 切换到叫newname的分支上
git merge newname 把newname分支合并到当前分支上
git pull origin master 将master分支上的内容拉到本地上