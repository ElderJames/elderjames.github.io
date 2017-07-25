---
title: ASP.NET Core中使用依赖注入获得请求生命周期内唯一的DbContext
tags: |-

  - .NET
  - .NET Core
permalink: asp-net-corezhong-shi-yong-qing-qiu-sheng-ming-zhou-qi-nei-wei-yi-de-dbcontext
id: 2
updated: '2016-09-22 14:57:06'
date: 2016-09-22 13:58:52
---

很高兴，我的博客又回来了。

在使用.NET Core来开发项目已经有两个多月了，想在这里记录一些在.NET Core中比较方便的知识。

这里讲一讲在ASP.NET Core中使用依赖注入的方式请求生命周期内唯一的DbContext。
按照以往.NET Framework中的Web应用程序使用EF，我们会把DbContext实例放入CallContext中，这是在一个线程周期中可共享的存储对象，来达到获得线程内唯一的DbContext等其它对象。但是在.NET Core时代，已经不需要CallContext了，而是使用最突出的特点——依赖注入。

首先，我们在Startup.cs中配置好EF Core:

```asp.net
services.AddDbContext<YourContext>(option =>
     option.UseMySql(Configuration.GetConnectionString("YunGoConllection"))
);
```
然后，在需要的类中以构造方法注入的方式注入YourContext即可。注意：不管是在MVC项目中还是被应用的类库中，都可以使用注入方法！这就是.NET Core的强大~

```asp.net
  public class TestController : Controller
    {
        YourContext _db;

        public TestController(YourContext db)
        {
            _db=db;
        }

        public async Task<IActionResult> Test()
        {
            var _users=await _db.Users.ToListAsync();
            return Ok(_users);
        }
     }
```

其实在services.AddDbContext()内部，是已经用services.AddScoped()方法注册了YourContext，用这种方式注册后可以在一次请求生命周期内获得同一个实例，所以，我们可以把所有的注入都使用这种方式，节省因为每次都new对象（等同于注册方法services.AddTransient()）而带来的内存占用~