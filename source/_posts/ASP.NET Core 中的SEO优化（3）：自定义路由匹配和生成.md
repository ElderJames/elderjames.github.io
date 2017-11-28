---
title: ASP.NET Core 中的SEO优化（3）:自定义路由匹配和生成
permalink: A-Middleware-Implement-For-Customized-Routing-In-AspNetCore
date: 2017-11-29 18:31:34
tags:
- ASP.NET Core
- .NET Core
categories:
- .NET Core
---

## 前言

前两篇文章主要总结了CMS系统两个技术点在**ASP.NET Core**中的应用：
- [《ASP.NET Core 中的SEO优化（1）：中间件实现服务端静态化缓存》](A-Middleware-Implement-For-Server-Side-Static-Caching-In-AspNetCore.html)
- [《ASP.NET Core 中的SEO优化（2）：中间件中渲染Razor视图》](A-Middleware-Implement-For-Rendering-Razor-Views-In-AspNetCore)

而本篇文章，继续介绍另一个技术点：自定义路由匹配和生成。

## 背景

在MVC5时代，默认的路由可能就是简单的约定`/{controller}/{action}/{id}`,第一节对应控制器（Controller）名，第二节对应操作（Action）名,第三节是参数名。
在WebApi和ASP.NET Core时代，有了`Route`特性来指定相应操作的路由指向，可以很灵活地配置RESTful Api。

但是，路由灵活性在注重SEO的CMS系统中有更苛刻的要求。例如：

- 栏目的列表 ->`/{父栏目名}/{子栏目名}-{页码}/`
- 文章详情页 ->`/{栏目名}/{文章名}.html`
- 标签页 ->`/{标签名}`

这些友好的url链接使用默认的路由约定是很难实现的，当然是可以配置路由规则去传递参数：

```csharp
app.UseMvc(routes =>
{
    routes.MapRoute(
        name: "article_list",
        template: "{parentCategory}/{category}-{page}/",
        defaults: new { controller = "Article", action = "Index" });

    routes.MapRoute(
        name: "article_detail",
        template: "{category}/{article}.html",
        defaults: new { controller = "Article", action = "Detail" });

    routes.MapRoute(
        name: "tags",
        template: "{tag}/",
        defaults: new { controller = "Article", action = "Tag" });
    });
```

但是，这样配置很繁琐，也不灵活，如果还要传入更多规则，比如不向下匹配，就显得捉襟见肘了。那有没有更灵活的方案呢？当然有，就是本文接下来要介绍的IRouter接口。

## 原理

上一节最后介绍的一般路由配置方法，其实是`MapRoute`方法创建一个个`IRouter`的实例，添加到`IRouteBuilder`实例中，具体方法可以[看看源码](https://github.com/aspnet/Routing/blob/032bcf43b2cefe641fc6ee9ef3ab0769024a182c/src/Microsoft.AspNetCore.Routing/MapRouteRouteBuilderExtensions.cs#L97)。

所以我们可以实现一个自定义的`IRouter`，就能不用默认的约定，去实现我们的苛刻需求。

首先要看看`IRouter`这个接口的定义，源码在[`aspnet/Routing/src/Microsoft.AspNetCore.Routing.Abstractions/IRouter.cs`](https://github.com/aspnet/Routing/blob/032bcf43b2cefe641fc6ee9ef3ab0769024a182c/src/Microsoft.AspNetCore.Routing.Abstractions/IRouter.cs)

```csharp
namespace Microsoft.AspNetCore.Routing
{
    public interface IRouter
    {
        Task RouteAsync(RouteContext context);

        VirtualPathData GetVirtualPath(VirtualPathContext context);
    }
}
```

## 实现

`IRouter`接口只有两个方法，本文分别给出代码示例来介绍。

### `RouteAsync`

这个方法中实现路由匹配，从路由上下文中获取所需要的数据，存入RouteData字典中，再从控制器取出做相应的处理。

```csharp
public async Task RouteAsync(RouteContext context)
{
    var requestedUrl = context.HttpContext.Request.Path.Value.TrimStart('/').ToLower();
    var split = requestedUrl.Split('/');

    if (secoend != null && secoend.EndsWith(".html") && split.Length == 2)
    {
        var title = secoend.Replace(".html", "");
        context.RouteData.Values["controller"] = "Article";
        context.RouteData.Values["action"] = "Detail";
        context.RouteData.Values["category"] = first;
        context.RouteData.Values["title"] = title;
    }
    //...对请求路径进行一系列的判断

    //最后注入`MvcRouteHandler`示例执行`RouteAsync`方法，表示匹配成功
    await context.HttpContext.RequestServices.GetService<MvcRouteHandler>().RouteAsync(context);
}
```

这样，当请求路由为`/news/asp.net.html`时，就会匹配到上面的规则，请求进入`Article`控制器中的`Detail`操作被处理，并且可以从控制器中的`RouteData.Values["category"]?.ToString();`方法拿到所需的数据。

### `GetVirtualPath`

这个方法实现路由生成，可以从路由上下文中获取RouteData字典中的数据，进行虚拟路径（区别与真实目录）的生成。

```csharp
public VirtualPathData GetVirtualPath(VirtualPathContext context)
{
    var path = string.Empty;
    var hasController = context.Values.TryGetValue("controller", out var controller);
    var hasAction = context.Values.TryGetValue("action", out var action);
    var hasCategory = context.Values.TryGetValue("category", out var category);
    var hasTitle = context.Values.TryGetValue("title", out var title);

    if (hasController && hasAction && hasCategory && hasTitle)
    {
        path = $"/{category/{title}.html";
    }

    return path != string.Empty ? new VirtualPathData(this, path) : null;
}
```

这样，当调用路径生成方法`@Url.Action("Detail","Article",new { title="asp.net", category="news" })`,就会生成"/news/asp.net.html"这样的路径来。

### IRouter的设置生效

自定义实现的`IRouter`如何设置到原有的MVC项目中呢？方法很简单，上面已经简单说道了，其实`app.UseMvc`这个方法就有添加`IRouter`的方法:

```csharp
app.UseMvc(routes =>
{
    //添加 自定义路由匹配与url生成组件
    routes.Routes.Add(new RouteProvider());
});
```

这里`RouteProvider`是`IRouter`的实现，这样自定义的路由提供对象就生效啦！

## 相关小技巧

1. SEO中会有一条铁规则，就是不是有效链接的情况下只能返回404，比如，手动输入了一个路径，能匹配到文章详情的操作（action），如果数据库中查不出文章，本来应该不能匹配的，但是，如果在匹配的时候查询数据库确认是否存在的话，会加大系统的压力，所以，可以放在操作（action）中查询，查不到再返回`NotFound`，让上一篇文章[《ASP.NET Core 中的SEO优化（2）：中间件中渲染Razor视图》](A-Middleware-Implement-For-Rendering-Razor-Views-In-AspNetCore)中介绍的中间件一并处理输出404页面。

2. 站内链接如果使用带域名的绝对路径，能够提高优化效果，我们可以自己写一个`UrlHelper`的扩展方法:

```csharp
public static class UrlHelperExtensions
{
    public static string AbsoluteAction(
        this IUrlHelper helper,
        string actionName,
        string controllerName,
        object routeValues = null)
    {
        string scheme = helper.ActionContext.HttpContext.Request.Scheme;
        return helper.Action(actionName, controllerName, routeValues, scheme);
    }

    public static string AbsoluteContent(
        this IUrlHelper helper,
        string contentPath)
    {
        return new Uri(helper.ActionContext.HttpContext.Request.GetUri(), helper.Content(contentPath)).ToString();
    }

    public static string AbsoluteRouteUrl(
        this IUrlHelper helper,
        string routeName,
        object routeValues = null)
    {
        string scheme = helper.ActionContext.HttpContext.Request.Scheme;
        return helper.RouteUrl(routeName, routeValues, scheme);
    }
}
```

## 总结

本文主要介绍了自定义路由匹配和生成的解决方案，把路由相关的处理集中到一个类中，避免分散在各个视图进行维护。下篇文章，将会介绍自定义视图搜索目录及主题切换。