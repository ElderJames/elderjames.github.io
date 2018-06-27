---
title: EF Core For Mysql 的DataBase First（基于已有数据库）方式连接数据库
permalink: ef-core-for-mysql-de-databasefang-shi-lian-jie-shu-ju-ku
id: 24
updated: '2016-07-30 12:25:49'
date: 2016-07-30 11:30:16
tags:
- .NET Core
- Mysql
---

.NET要跨平台，就一定要使用同样可以跨平台的数据库，而轻量的MySQL自然是首选。.NET Core发展到现在，虽然官方的Entity Framework还没发布MySQL版本，但是刚认识的柚子大神已经首先把它做出来了（[他的博客](http://www.1234.sh/post/pomelo-data-mysql)）。

同时，他也开源了它使用MySql for Entity Framework Core写的轻量博客系统，可惜在数据库连接是使用Code First。Code First当时是好，也是微软官方推荐的数据操作方式，可是，对于已有数据库的项目，希望创建基于已有数据库的.NET Core应用，或者对已有项目进行移植的开发者，就显得力不从心了。

虽然官方有从数据库创建模型的方法（[地址](https://docs.efproject.net/en/latest/platforms/aspnetcore/existing-db.html)），但是它使用了下面这三个只适用于SqlServer的工具包：

- Microsoft.EntityFrameworkCore.SqlServer
- Microsoft.EntityFrameworkCore.Tools
- Microsoft.EntityFrameworkCore.SqlServer.Design

目前MySQL方面还没有这些工具包，就自然不能执行从数据库创建模型的命令（如下）了。

```bash
Scaffold-DbContext "Server=(localdb)\mssqllocaldb;Database=Blogging;Trusted_Connection=True;" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models
```
因此，我们目前只能手动创建模型，并且在DbContext中将模型与数据库中相应的表绑定起来，具体方法就是在DbContext中重写OnModelCreating方法：

```aspnet
        protected override void OnModelCreating(ModelBuilder builder)
        {
            builder.Entity<SliderPicture>(e =>
            {
                e.HasKey(x => x.Id);
                e.ToTable("yg_slider");
            });

            builder.Entity<ClientVersion>(e =>
            {
                e.HasKey(x => x.Id);
                e.ToTable("yg_version")
                .HasIndex(x => x.CreateTime);
            });
        }
```

希望.NET Core能尽快完善并增加更强大的特性，我想这也是所以.NETer的希望。