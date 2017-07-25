---
title: Visual Studio 2017 正式版安装问题及解决方案记录
permalink: visual-studio-2017-zheng-shi-ban-an-zhuang-wen-ti-ji-jie-jue-fang-an-ji-lu
id: 27
updated: '2017-03-14 13:48:30'
date: 2017-03-14 10:05:42
tags:
---

##Visual Studio 2017正式版终于出来啦！

下载地址：[请到主页](https://www.visualstudio.com/zh-hans/)

推荐使用安装器下载，速度比以前快了很多！必装的Web和.NET Core组件只需要15分钟就可以安装完（当然，视网速而定，15分钟是用100M/bs的宽带下载的，下载峰值在5M/s左右）

以下是记录了一些安装中遇到的问题，会不定时更新。

#### Win7 无法安装Cordova
因为Cordova组件需要PowerShell 6.0，而Win7是不自带的（Win8-Win10自带）,所以先下载PowerShell 6.0吧！

[PowerShell 6.0下载地址](https://www.microsoft.com/en-us/download/details.aspx?id=34595)

#### Cordova 无法编译成Android
可能是Gradle压缩包损坏导致，重新下载压缩包放到目录C:\Users\Administrator\.gradle\wrapper\dists\[gradle-2.13-all]下
[单独下载Gradle](https://services.gradle.org/distributions/)