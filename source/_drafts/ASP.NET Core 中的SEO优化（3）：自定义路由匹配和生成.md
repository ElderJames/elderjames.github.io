---
title: ASP.NET Core 中的SEO优化（3）:自定义路由匹配和生成
permalink: A-Middleware-Implement-For-Customized-Routing-In-AspNetCore
date: 2017-11-23 18:31:34
tags:
- ASP.NET Core
- .NET Core
categories:
- .NET Core
---

前两篇文章主要总结了CMS系统两个技术点在**ASP.NET Core**中的应用：
- [《ASP.NET Core 中的SEO优化（1）：中间件实现服务端静态化缓存》](A-Middleware-Implement-For-Server-Side-Static-Caching-In-AspNetCore.html)
- [《ASP.NET Core 中的SEO优化（2）：中间件中渲染Razor视图》](A-Middleware-Implement-For-Rendering-Razor-Views-In-AspNetCore)

而本篇文章，继续介绍另一个技术点：自定义路由匹配和生成

在MVC5时代，默认的路由可能就是简单的约定`/{controller}/{action}/{id}`,第一节对应控制器（Controller）名，第二节对应操作（Action）名,第三节是参数名。
在WebApi和ASP.NET Core时代，有了`Route`特性来指定相应操作的路由指向，可以很灵活地配置RESTful Api。

但是，路由灵活性在注重SEO的CMS系统中有更苛刻的要求。例如"

- 栏目的列表路由格式会定义为`/{父栏目名}/{子栏目名}-{页码}/`
- 