---
title: NPM 国内镜像
permalink: 'n'
id: 26
updated: '2016-09-28 23:31:38'
date: 2016-09-22 15:18:28
tags:
- NPM
---

npm由于很多时候都需要从国外的服务器下载组件，由于**众所周知**的原因，国内安装时往往会很慢甚至卡住安装不上的情况，这时就需要替换一下安装源，使用国内服务商提供的镜像：

- http://npm.hacknodejs.com/
- http://registry.npmjs.vitecho.com/
- https://registry.npm.taobao.org

####使用方法
永久使用镜像命令： 

`npm config set registry https://registry.npm.taobao.org`

临时使用镜像命令：

`npm --registry "http://npm.hacknodejs.com/" install underscore`