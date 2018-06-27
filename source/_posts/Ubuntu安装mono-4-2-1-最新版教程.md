---
title: Ubuntu安装mono 4.2.1 最新版教程
tags:
- mono
- Linux
- Ubuntu
permalink: the-lastest-way-to-install-mono-4-2-1-on-ubuntu
id: 11
updated: '2015-12-03 14:39:39'
date: 2015-12-03 12:00:04
---

现在安装mono已经不用make命令来编译，可以直接执行`apt-get mono-complete`来安装最新的mono版本。

下面看看怎么安装：

- 首先，在mono-project的安装指南页面获得最新的注册信息，
[这是地址](http://www.mono-project.com/docs/getting-started/install/linux/#debian-ubuntu-and-derivatives)。

然后按照上面的命令执行

`sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF`

`echo "deb http://download.mono-project.com/repo/debian wheezy main" | sudo tee /etc/apt/sources.list.d/mono-xamarin.list`

`sudo apt-get update`

然后，再执行

`sudo apt-get install mono-complete  `

OK,完成。不用再等久久的编译咯！