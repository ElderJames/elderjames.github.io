---
title: ASP.NET Core 中的SEO优化（4）：自定义视图路径及主题切换
date: 2017-11-30 21:44:19
permalink: Customized-View-Path-And-Theme-Switching-In-AspNetCore
description:
thumbnail:
tags:
- ASP.NET Core
- .NET Core
categories:
- .NET Core
---
## 系列回顾

- [《ASP.NET Core 中的SEO优化（1）：中间件实现服务端静态化缓存》](A-Middleware-Implement-For-Server-Side-Static-Caching-In-AspNetCore.html)
- [《ASP.NET Core 中的SEO优化（2）：中间件中渲染Razor视图》](A-Middleware-Implement-For-Rendering-Razor-Views-In-AspNetCore.html)
- [《ASP.NET Core 中的SEO优化（3）:自定义路由匹配和生成》](A-Middleware-Implement-For-Customized-Routing-In-AspNetCore.html)

## 背景

切换主题，是博客、CMS等系统的必备功能，一般来说，有三种切换主题的需求。

1. 在管理后台上传主题包，并选择主题
2. 前端自动按照频道、栏目等切换模版
3. 用户在前端切换主题，并记录用户的选择

这三种需求，其实核心原理都是一样，就是制定一套主题的目录，切换主题等于切换目录名。主题内的页面模版都是按照一定的规则存放的。

下面是两个主题包的目录示例:

```shell
  .
  ├── theme0
  |   ├── Assets
  |   |   ├── js
  |   |   ├── css
  |   |   └── img
  |   ├── Home
  |   |   ├── Index.cshtml
  |   |   └── About.cshtml
  |   ├── Article
  |   |   ├── Index.cshtml
  |   |   └── Detail.cshtml
  |   └── Shared
  |       ├── Page.cshtml
  |       └──  _Layout.cshtml
  └── theme1
      ├── Assets
      |   ├── js
      |   ├── css
      |   └── img
      ├── Home
      |   ├── Index.cshtml
      |   └── About.cshtml
      ├── Article
      |   ├── Index.cshtml
      |   └── Detail.cshtml
      └── Shared
          ├── Page.cshtml
          └──  _Layout.cshtml
  ```

  大家一定注意到了，上面每个主题包里都按照传统ASP.NET MVC的约定来划分目录：控制器名为文件夹，操作名为视图文件。其实这里只是方便起见，按照接下来介绍的方法，是可以完全地自定义这个目录划分的。

## 原理

当ASP.NET MVC从控制器处理完数据返回视图的时候，ASP.NET MVC会按照默认的多个路径去查找文件，如果文件存在，则使用该文件渲染，如果不存在，则寻找下一个路径，比如默认的路径会有`/{Area}/{Controller}/{Action}.cshtml`、`/{Controller}/{Action}.cshtml`、`/Shared/{Action}.cshtml`等等我们熟悉的约定，那么在查找视图文件时，会安装从左往右的路径去查询，如果都查询不出来，是会报错的。

而如果要做到切换主题文件夹名来切换主题，我们就需要在默认规则上加主题的目录占位符，使的查询时用主题文件夹名来替换占位符，例如`/{theme}/{Controller}/{Action}.cshtml`、`/{theme}/Shared/{Action}.cshtml`等等，这样，当查询视图文件时，就能匹配到对应的主题文件夹，并且找到相应的视图了。

总结起来，切换主题功能有两个重点需要我们去实现:

1. 在原有规则中加入占位符
2. 每次请求都获取当前的主题名，并改变视图查询路径

## 实现

最简单的实现，在操作（action）的最后`return View(viewPath)`时传入视图路径，直接就能指向对应视图，但是，这样做一点都不灵活，而且每个操作都要传路径也是不够简洁，不容易维护，所以我们需要更好的解决方案。

#### ASP.NET MVC 实现

在ASP.NET MVC时代，我们可通过继承RazorViewEngine类，在基类的`ViewLocationFormats`和`PartialViewLocationFormats`两个属性中加入有主题目录名占位符的路径，并重写`CreateView`、`CreatePartialView`、`FileExists`三个方法，使每次请求都能获取最新的主题名，如下面的例子中从路由数据对象中获取主题名：

```csharp
public class TemplateViewEngine : RazorViewEngine
{
    public TemplateViewEngine() : base()
    {
        ViewLocationFormats = new[] {
            "~/Views/{1}%1/{0}.cshtml",
            "~/Views/{1}/{0}.cshtml",//默认路径
            "~/Views/Shared%1/{0}.cshtml",
            "~/Views/Shared/{0}.cshtml",
        };

        PartialViewLocationFormats = new[] {
            "~/Views/{1}%1/{0}.cshtml",
            "~/Views/{1}/{0}.cshtml",//默认路径
            "~/Views/Shared%1/{0}.cshtml",
            "~/Views/Shared/{0}.cshtml",
        };
    }

    protected override IView CreatePartialView(ControllerContext controllerContext, string partialPath)
    {
        var template = controllerContext.RouteData.Values["template"] != null ? "/" + controllerContext.RouteData.Values["template"].ToString() : "";

        return base.CreatePartialView(controllerContext, partialPath.Replace("%1", template));
    }

    protected override IView CreateView(ControllerContext controllerContext, string viewPath, string masterPath)
    {
        var template = controllerContext.RouteData.Values["template"] != null ? "/" + controllerContext.RouteData.Values["template"].ToString() : "";

        return base.CreateView(controllerContext, viewPath.Replace("%1", template), masterPath);
    }

    protected override bool FileExists(ControllerContext controllerContext, string virtualPath)
    {
        var template = controllerContext.RouteData.Values["template"] != null ? "/" + controllerContext.RouteData.Values["template"].ToString() : "";

        return base.FileExists(controllerContext, virtualPath.Replace("%1", template));
    }
}
```
事实上，如果是需要实现不同用户不同主题的功能，主题信息可以存储在Session中，还能从`controllerContext`实例获取`Session`中存储的主题名。

那么，在ASP.NET Core中如何实现呢？

#### ASP.NET Core 实现

ASP.NET Core 相比ASP.NET MVC框架，虽然使用上为了开发者平滑过渡，很多约定都相同，但是架构本身是做了翻天覆地的重构和优化，得益于一脉相承的MSDI框架，ASP.NET Core框架实现了组件化，很多功能都通过IoC的方式修改或扩展。例如本文介绍的主题情况功能，就是实现`IViewLocationExpander`接口来达到扩展配置的目的，而且还比ASP.NET MVC的更加简洁：

```csharp
public class TemplateViewLocationExpander : IViewLocationExpander
{
    public IEnumerable<string> ExpandViewLocations(ViewLocationExpanderContext context, IEnumerable<string> viewLocations)
    {
        var template = context.Values["template"] ?? "Default";

        string[] locations = { "/Views/" + template + "/{1}/{0}.cshtml", "/Views/" + template + "/{0}.cshtml", "/Views/" + template + "/Shared/{0}.cshtml" };
        return locations.Union(viewLocations);
    }

    public void PopulateValues(ViewLocationExpanderContext context)
    {
        context.Values["template"] = context.ActionContext.RouteData.Values["Template"]?.ToString() ?? "Default";
    }
}
```

这个接口里面，`PopulateValues`方法主要用来获取实时的主题信息，`context.ActionContext`中除了`RouteData`可获得实时数据，还有`HttpContext`实例可获得用户信息，甚至能利用`RequestServices`实例注入服务。而只有在`PopulateValues`中修改了`context`，`ExpandViewLocations`方法才会从`context`中获得主题信息，从而达到修改视图查找路径的目的。

当我们实现了`IViewLocationExpander`接口后，还需要在`Startup`类的`services.AddMvc();`下修改MVC的配置:

```csharp
services.AddMvc();
//配置模版视图路径
services.Configure<RazorViewEngineOptions>(options =>
{
    options.ViewLocationExpanders.Add(new TemplateViewLocationExpander());
});
```

*PS:这种修改MVC内部配置的方式很有趣，以后有空会研究一番。*

## 总结

本文主要介绍了在ASP.NET Core中利用修改视图查询路径实现主题切换的功能，虽然只介绍了核心部分，但是其它部分如管理主题、前端切换等功能，都是很容易实现的，以后我会在我的框架样例中实现，敬请大家关注啦！