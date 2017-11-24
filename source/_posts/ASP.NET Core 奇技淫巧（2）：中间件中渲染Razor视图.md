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

## 实现

实现方式很简单，就是在Configure中注入ICompositeViewEngine实例，构造视图上下文，再渲染视图为字符串，最后输出。其它的分析就在代码注释中说明吧

直接上代码：

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ICompositeViewEngine engine)
{
    app.Use(async (context, next) =>
    {
        //因为只是在请求最后处理，所以这里直接就运行下一个中间件
        await next();
        //返回后检查是否出现错误的状态
        if (context.Response.StatusCode >= 400)
        {
            context.Response.StatusCode = (int)HttpStatusCode.NotFound;
            //ContentType设置为text/html，使浏览器以正常页面的格式显示
            context.Response.ContentType = "text/html";
            //指向特定的视图
            var viewResult = engine.GetView("~/", "~/Views/Default/Home/Error.cshtml", true);

            if (!viewResult.Success)
                await context.Response.WriteAsync("OMG! 连错误视图都找不到了。。");
            //创建临时的StringWriter实例，用来配置到视图上下文中
            using (var output = new StringWriter())
            {
                //视图上下文对于视图渲染来说很重要，视图中的前后台交互都需要它
                var viewContext = new ViewContext()
                {
                    HttpContext = context,
                    Writer = output,
                    RouteData = new Microsoft.AspNetCore.Routing.RouteData()
                    {
                        //RouteData在这里传入视图，这样视图可以显示错误信息之类的数据
                    },
                    View = viewResult.View,
                    FormContext = new FormContext(),
                    ActionDescriptor = new Microsoft.AspNetCore.Mvc.Abstractions.ActionDescriptor()
                };
                //渲染
                await viewResult.View.RenderAsync(viewContext);
                //输出到响应体
                await context.Response.WriteAsync(output.ToString());
            }
        }
    });

    //后面是Mvc的中间件，执行Mvc的处理
    //...app.UseMvc
}
```

## 总结

这个技巧还能用于单页面应用程序的路由重定向，把所有路由都输出入口页面代码。