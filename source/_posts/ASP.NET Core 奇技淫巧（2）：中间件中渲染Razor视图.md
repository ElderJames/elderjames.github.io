---
layout: post
title: Core奇技淫巧（2）：中间件中渲染Razor视图
date: 2017-11-23 21:50:39
permalink: A-Middleware-Implement-For-Rendering-Razor-Views-In-AspNetCore
tags:
- ASP.NET Core
- .NET Core
categories:
- .NET Core
thumbnail:
---
## 前言

上一篇文章[《ASP.NET Core 奇技淫巧（1）：中间件实现服务端静态化缓存》](A-Middleware-Implement-For-Server-Side-Static-Caching-In-AspNetCore.html)中介绍了中间件的使用方法、以及使用中间件实现服务端静态化缓存的功能。本系列文章取名“奇技淫巧”不是没道理的，因为这写技巧都是我最近在做的公司实际项目中的一些奇怪的需求之后总结而来的……

## 要解决的问题

好了，本篇说说如何在中间件中渲染Razor视图。之所以会有这个技巧，是因为我们有个需求：

> 需要在所有返回404状态的路由都输出一个特定视图。
> 比如当有id=1的文章，而没有id=2的文章时，那么`/url/1.html`展示文章详情页，`/url/2.html`展示404视图。

所以，要实现这个需求只有两种办法：
> 1. 当文章查找不到时直接执行`return View("404")`返回404视图。
> 2. 在中间件中执行完MVC的处理之后检查返回状态，如果是错误状态就直接渲染视图并输出。

由于CMS系统中不止一处需要返回404状态，所以因为用代码整洁作为懒惰的借口，决定尝试第二个方法。

## 原理

待续...

## 实现

直接上代码：

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ICompositeViewEngine engine)
{
    app.Use(async (context, next) =>
    {
        await next();
        if (context.Response.StatusCode >= 400)
        {
            context.Response.StatusCode = (int)HttpStatusCode.NotFound;
            context.Response.ContentType = "text/html";

            var viewResult = engine.GetView("~/", "~/Views/Default/Home/Error.cshtml", true);

            if (!viewResult.Success)
                await context.Response.WriteAsync("OMG! 连错误视图都找不到了。。");

            using (var output = new StringWriter())
            {
                var viewContext = new ViewContext()
                {
                    HttpContext = context,
                    Writer = output,
                    RouteData = new Microsoft.AspNetCore.Routing.RouteData()
                    {
                        Routers = { new RouteProvider() },
                    },
                    View = viewResult.View,
                    FormContext = new FormContext(),
                    ActionDescriptor = new Microsoft.AspNetCore.Mvc.Abstractions.ActionDescriptor()
                };

                await viewResult.View.RenderAsync(viewContext);

                await context.Response.WriteAsync(output.ToString());
            }
        }
    });

    app.UseMvc(routes =>
    {
        //添加 自定义路由匹配与url生成组件
        routes.Routes.Add(new RouteProvider());
    });
}
```

## 总结

这个技巧还能用于单页面应用程序的路由重定向，把所有路由都输出入口页面代码。

待续...