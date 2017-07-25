---
title: .NET Core以依赖注入的方式配置MySql的DbContext
permalink: yi-zhu-ru-de-fang-shi
id: 23
updated: '2016-07-31 11:57:00'
date: 2016-07-30 11:04:28
tags:
---

使用ASP.NET Core开发Web项目，在很多方面都跟之前的.NET版本有很大不同，在Startup.cs的配置方面,一言不合就使用依赖注入。当然，这对于整个项目的解耦是极好的。这次踩的坑在DbContext的配置上。

在[柚子大神的博客](http://www.1234.sh/post/pomelo-data-mysql)中,有介绍他移植过来的mysql for ef core的使用方法是在Startup中注册服务：
```aspnet
services.AddDbContext<YourContext>(x => x.UseMySql("server=localhost;database=yourdb;uid=root;pwd=yourpwd"));
```
但是，这样配置之后，又需要在YourContext中配置：
![](/content/images/2016/07/0e1ef77c-b03e-439f-a894-cb1de1811695.png)

这样的话，其实是等于配置了两次，在DbContext中是直接写ConnectionString的，不能（我还不会）从配置文件中读取。因此，需要寻找一个从Startup中配置的有效方法。

通过对比MVC6项目模版的sqlserver配置方式，和[官方文档的介绍](https://docs.efproject.net/en/latest/miscellaneous/configuring-dbcontext.html)，发现DbContext里是可以通过注入的方式注册ConnectionString的，方法就是设置构造函数注入DbContextOptions：

```aspnet
  public YourContext(DbContextOptions<YourContext> options)
            : base(options)
        { }
```