---
layout: post
title: 时序数据库InfluxDB在.NET Core中的使用——（2）IoC和仓储模式
date: 2017-09-26 00:44:01
permalink: influxdb-in-dotNEt-Core-2-ioc-and-repository-pattern
tags:
- .NET Core
- InfluxDB
categories:
- .NET Core
description:
thumbnail: https://www.msolution.io/wp-content/uploads/2015/12/influxdb.png
---

我上一篇文章[《InfluxDB在.NET Core中的使用——（1）InfluxDB简介》](/influxdb-in-dotNEt-Core-by-ioc-and-repository-pattern.html)中简单介绍了InfluxDB的相关概念，本文将继续介绍下半部分，就是在.NET Core中的使用方法。

## .NET 客户端

正因为InfluxDB是通过http请求来操作数据库，所以在.NET Core中使用很简单，只需要用HttpClient封装应该客户端、并且按照InfluxDB的数据结构去解析数据就行了。而且，已经有大神开发出了一个可用于.NET Core的InfluxDB客户端开源项目 [InfluxData.Net](https://github.com/pootzko/InfluxData.Net) ，本文将利用这个客户端项目结合.NET Core自带的依赖注入框架和仓储模式，去封装一个事件存储的InfluxDB实现的类库。

---
未完待续。