---
title: ASP.NET Core使用Session方法
permalink: as
id: 17
updated: '2016-07-08 14:23:41'
date: 2016-07-08 14:18:16
tags:
- .NET Core
- .NET
---

- 在project.json配置文件中引入程序集：`"Microsoft.AspNetCore.Session": "1.0.0"`

- 在Startup.cs的ConfigureService方法中加入以下代码：
```
services.AddSession(options => {
    options.IdleTimeout = TimeSpan.FromMinutes(30);
 } );
```
- 在Configure方法中加： `app.UseSession();`

于是，Session就可以用啦！