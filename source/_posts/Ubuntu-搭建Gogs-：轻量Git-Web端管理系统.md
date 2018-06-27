---
title: Ubuntu 搭建Gogs ：轻量Git Web端管理系统
tags:
- git
- gogs
- Linux
- Ubuntu
permalink: ubuntu-da-jian-gogs-qing-liang-git-webduan-guan-li-xi-tong
id: 14
updated: '2016-03-28 18:13:06'
date: 2016-03-26 22:36:31
---

Gogs是国人开发的开源git管理系统，类似gitlab，可在服务器上搭建私人的git版本管理系统。内建很多功能，但俾比gitlab安装更方便，占用更少的资源。
详细介绍还是看看官方网站吧：[https://gogs.io](https://gogs.io)

现在结合本人的安装过程，在这里做一个记录。适合小白参考。

安装方法有三种：二进制安装，源码安装，包管理安装，我认为最方便的还是二进制安装。

首先下载[二进制包](https://gogs.io/docs/installation/install_from_binary)，在这个页面上也有安装教程。

然后在解压后进入目录，执行 ./gogs web ，则能够启动服务进程，但这时的服务是临时的，一退出就会关闭，所以需要持续化进程配置。

因为gogs的安装设置页面里的设置只能设置一次，以后要修改则要新建一个文件，所以这里最好先做好准备工作。

- 反向代理 代理ip为 http://localhost:3000/
- 持续进程 使用supervisor

`sudo apt-get -y install supervisor`

`sudo mkdir -p /var/log/gogs`

`sudo nano /etc/supervisor/supervisord.conf`

在这个文件最下面添加应用信息
```
[program:gogs]
directory=/home/gogs/
command=/home/gogs/gogs web
autostart=true
autorestart=true
startsecs=10
stdout_logfile=/var/log/gogs/stdout.log
stdout_logfile_maxbytes=1MB
stdout_logfile_backups=10
stdout_capture_maxbytes=1MB
stderr_logfile=/var/log/gogs/stderr.log
stderr_logfile_maxbytes=1MB
stderr_logfile_backups=10
stderr_capture_maxbytes=1MB
environment = HOME="/home/git", USER="git"
user = git
```
其中directory和command中的路径是gogs目录的路径

最后执行重启服务
`sudo service supervisor restart`

提示Starting supervisor: supervisord.

然后执行`ps -ef | grep gogs`查看服务是否已经启动

如果有如下类似信息则表示服务启动成功：

root      1344  1343  0 08:55 ?        00:00:00 /home/gogs/gogs web

这个时候，可以在浏览器上访问`http://your_server_ip:3000/`或者在反向代理中设置好的地址，自动跳转到/install设置页面。其中要注意的是网址设置，如果已经使用反向代理，则添加设置的地址，应用域名不需要http:// ，而后面的应用url则需要http:// ，否则路由会出错。
