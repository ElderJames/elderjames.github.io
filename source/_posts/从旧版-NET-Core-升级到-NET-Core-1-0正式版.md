---
title: 从旧版.NET Core 升级到.NET Core 1.0正式版
permalink: cong-jiu-ban-net-core-sheng-ji-dao-net-core-1-0zheng-shi-ban
id: 16
updated: '2016-06-30 15:11:08'
date: 2016-06-30 14:00:33
tags:
- .NET Core
- .NET
categories:
- .NET Core
---

6月28日凌晨，RedHat 开发者大会上微软正式发布.NET Core正式版，我作为.NET爱好者，马上就想安装来玩玩，不过因为之前电脑上就装有旧版本，在这次正式版的安装中遇到了一些坑，装了n遍，死活装不上，停留在旧版本。

下载地址：

1. [.NET Core for Visual Studio official MSI Installer](https://go.microsoft.com/fwlink/?LinkId=817245) 
(需要先安装[Visual Studio 2015 Update 3](https://www.visualstudio.com/downloads/download-visual-studio-vs))
2. [.NET Core SDK for Windows](https://go.microsoft.com/fwlink/?LinkID=809122)

然后，如果你电脑之前没安装过.NET Core,有装了Visual Studio 2015 的话，需要把VS升级到Update3，再安装.NET Core for Visual Studio official MSI Installer（里面已经包含.NET Core SDK）； 如果没装VS，也不打算使用VS的话，可以只安装.NET Core SDK。

但是，如果以前安装过旧版的.NET Core，就需要先在【控制面板\所有控制面板项\程序和功能】中卸载以前的版本，并且==需要删除所有的环境变量==，在【控制面板\所有控制面板项\系统】中的左侧边栏中点击【高级系统设计】，在弹出的窗口中点击下方的环境变量按钮，在弹出的【环境变量】窗口中的【系统变量】一栏中找到“Path”变量，删除所有指向dotnet、dnx、dnu、dnvm目录的值，然后按“确定”保存退出。

这样，安装程序才能重新安装新版的.NET Core SDK。在命令提示符窗口中输入`dotnet`命令即可查看到当前版本是否为正式版1.0.1。

谢谢大家！

