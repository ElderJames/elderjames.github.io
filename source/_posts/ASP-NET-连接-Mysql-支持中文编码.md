---
title: ASP.NET 连接 Mysql 支持中文编码
permalink: make-mysql-support-chinese-encoding
id: 10
updated: '2015-11-08 12:56:43'
date: 2015-11-06 16:44:28
tags:
- Mysql
---

1、关闭mysql服务
`service mysql stop `

2、修改 /etc/mysql/my.cnf  （默认的安装路径）
`vim /etc/mysql/my.cnf `

3、打开my.cnf后，在文件内的[mysqld]下增加如下两行设置:
```
character-set-server=utf8
collation-server=utf8_general_ci
```
![](http://)

保存并退出。

4、重新启动mysql服务
`service mysql start  `

5、连接字符串加入`CharSet=utf8;`