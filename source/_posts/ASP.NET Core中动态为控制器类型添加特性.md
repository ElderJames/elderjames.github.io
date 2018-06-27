---
title: ASP.NET Core中动态为控制器类型添加特性
date: 2017-09-13 17:59:22
permalink: dynamically-add-features-to-the-controller-type-in-dot-net-core
tags: 
- ASP.NET Core
- .NET Core
- WebApi
- MVC
categories:
- .NET Core
thumbnail: /images/add_webapi_to_class/title.png
---
我在上一篇文章[《ASP.NET Core中为指定类添加WebApi服务功能》](https://yangshunjie.com/add-the-webapi-service-to-the-specified-class-in-asp-net-core.html)中介绍了如何为指定类型增加WebApi服务功能，达到了一定的解耦效果（不再需要继承Controller类，不再需要Controller后缀名），但是，`Route`、`HttpGet`之类的特性类型还是需要用Mvc的，那么今天这篇文章，就介绍如何把这些标签的依赖也消灭掉。

如果还没看上一篇文章又不了解如何指定类型为控制器的朋友，最好先看一下上一篇文章，因为要实现本文介绍的功能，是需要先实现上一篇文章所介绍的功能的。

### 实现原理

在没有MVC引用的类库中自定义`RouteAttribute`、`HttpGetAttribute`、`HttpPostAttribute`等等跟MVC配对的特性标签类型；其次，在定义服务接口时把这些特性标记到方法声明上面；最后在ASP.NET Core 服务启动时自动按照指定的接口上的特性给相应的接口添加MVC的标签，使得动态指定的类型也能获得MVC的路由和请求方法限制的功能。

用到的技术点主要是**反射**和利用MVC框架给出的**配置点**。

MVC框架给出的配置点在[`IControllerModelConvention`](https://github.com/aspnet/Mvc/blob/de1b763d963f8be612d93889e08e43f67c98d0d5/src/Microsoft.AspNetCore.Mvc.Core/ApplicationModels/IControllerModelConvention.cs)、[`IActionModelConvention`](https://github.com/aspnet/Mvc/blob/b6a6b50776bb1ee49c0bca1353375e964101bb8a/src/Microsoft.AspNetCore.Mvc.Core/ApplicationModels/IActionModelConvention.cs)、[`IParameterModelConvention`](https://github.com/aspnet/Mvc/blob/16c267d95eafbf310f17bc938d00048a541be0d0/src/Microsoft.AspNetCore.Mvc.Core/ApplicationModels/IParameterModelConvention.cs)三个接口。顾名思义，它们分别对应控制器、操作、参数的类型配置，在它们的`Apply`方法中，传入了一个MVC启动阶段扫描到的类型，开发者可以通过给这个类型添加各种MVC特性，如`RouteData`、`Attributes`、`Filters`等。但是，这些特性组合是有规则的，不能一股脑儿地都添加进去，MVC的机制中会用`SelectorModel`来将特性进行分组。

例如有以下的操作方法和它的特性：

```csharp
[HttpGet]
[AcceptVerbs("POST", "PUT")]
[HttpPost("Api/Things")]
public void DoThing()
```

那么MVC中就需要把他们分为两组：

1. `[HttpPost("Api/Things")]`
2. `[HttpGet], [AcceptVerbs("POST", "PUT")]`

所以，这个分组的规则，我们怎么去处理呢？其实，在MVC的源码中就有这样的方法[`IList<SelectorModel> CreateSelectors(IList<object> attributes)`](https://github.com/aspnet/Mvc/blob/de389226011fb15140ee0a8d6a2429b2eb943f3a/src/Microsoft.AspNetCore.Mvc.Core/Internal/DefaultApplicationModelProvider.cs#L439)，我们把它复制过来就好了。

### 学以致用

回到我们的需求，我们需要把指定类型的方法都加上MVC特性，就需要自定义一些`ModelConvention`。由于我们的需求比较简单，只需特性作用按类型来使指定类型获得MVC特性，我们只需在构造方法中传入指定类型，分别为控制器、操作和参数这三个作用类型都定义一个。由于篇幅问题，下面只贴Action的实现来讲解，另外两个大家可以看我[项目中的源码](https://github.com/ElderJames/shriek-fx/tree/master/src/Shriek.ServiceProxy.Http.Server/Internal)。

```csharp
using HttpGet = Microsoft.AspNetCore.Mvc.HttpGetAttribute;
using HttpPost = Microsoft.AspNetCore.Mvc.HttpPostAttribute;
using Route = Microsoft.AspNetCore.Mvc.RouteAttribute;

internal class ActionModelConvention : IActionModelConvention
{
    //构造方法传入指定接口类型
    public ActionModelConvention(Type serviceType)
    {
        this.serviceType = serviceType;
    }

    private Type serviceType { get; }

    public void Apply(ActionModel action)
    {
        //判断是否为指定接口类型的实现类
        if (!serviceType.IsAssignableFrom(action.Controller.ControllerType)) return;

        var actionParams = action.ActionMethod.GetParameters();

        //这串linq是查询出接口类型中与当前action相对应的方法，从中获取特性
        var method = serviceType.GetMethods().FirstOrDefault(mth =>
        {
            var mthParams = mth.GetParameters();
            return action.ActionMethod.Name == mth.Name
                   && actionParams.Length == mthParams.Length
                   && actionParams.Any(x => mthParams.Where(o => x.Name == o.Name).Any(o => x.GetType() == o.GetType()));
        });

        var attrs = method.GetCustomAttributes();
        var actionAttrs = new List<object>();

        foreach (var att in attrs)
            {
                //下面的HttpMethodAttribute是我们自己写的特性类型
                if (att is HttpMethodAttribute methodAttr)
                {
                    var httpMethod = methodAttr.Method;
                    var path = methodAttr.Path;

                    if (httpMethod == HttpMethod.Get)
                    {
                        //添加的HttpGet和HttpPost使用了命名空间别名
                        actionAttrs.Add(new HttpGet(path));
                    }
                    else if (httpMethod == HttpMethod.Post)
                    {
                        actionAttrs.Add(new HttpPost(path));
                    }
                }
                 //下面的RouteAttribute是我们自己写的特性类型
                if (att is RouteAttribute routeAttr)
                {
                    actionAttrs.Add(new Route(routeAttr.Template));
                }
            }

        if (actionAttrs.Any())
        {
            action.Selectors.Clear();
            //AddRange静态方法就是从源码中复制过来的
            ModelConventionHelper.AddRange(action.Selectors, ModelConventionHelper.CreateSelectors(actionAttrs));
        }
    }
}
```

上面代码其实还省略了其它的请求方式，我的源码中是有的，大家也可以前去查看。除了[`ActionModelConvention`](https://github.com/ElderJames/shriek-fx/blob/master/src/Shriek.ServiceProxy.Http.Server/Internal/ActionModelConvention.cs)，还需要写[`ControllerModelConvention`](https://github.com/ElderJames/shriek-fx/blob/master/src/Shriek.ServiceProxy.Http.Server/Internal/ControllerModelConvention.cs)和[`ParameterModelConvention`](https://github.com/ElderJames/shriek-fx/blob/master/src/Shriek.ServiceProxy.Http.Server/Internal/ParameterModelConvention.cs)。

### 配置到MVC框架

核心的代码写好了，那么怎么让它起作用呢？其实官方的源码已经提供了示例：[Mvc/test/WebSites/ApplicationModelWebSite/Startup.cs](https://github.com/aspnet/Mvc/blob/760c8f38678118734399c58c2dac981ea6e47046/test/WebSites/ApplicationModelWebSite/Startup.cs#L16)，我们只需在`services.AddMvc()`里的`setupAction`委托中将我们的配置类型添加到[MvcOptions.Conventions](https://github.com/aspnet/Mvc/blob/b4fe715c71f472180069f674bde3ef8014064d64/src/Microsoft.AspNetCore.Mvc.Core/MvcOptions.cs#L59)属性里就好了，这个属性是用来添加所有模型配置的，然后MVC启动后会把这些配置都扫描处理一遍。

来看看我这里的实现：

```csharp
//假设我们定义了这样的接口
[Route("test")]
public interface ITestService
{
    [Route("{name}"), HttpGet]
    string Test(string name);
}

//AddMvc也一样，这里用AddMvcCore只是为了减少依赖
services.AddMvcCore(opt=>
    {
        opt.Conventions.Add(new ControllerModelConvention(typeof(ITestService)));
        opt.Conventions.Add(new ActionModelConvention(typeof(ITestService)));
        opt.Conventions.Add(new ParameterModelConvention(typeof(ITestService)));
    })
```

当然了，我这里是因为指定类型是通过反射获取的，所以用了Type类型作为参数，大家也可以用泛型，在构造方法里获取对象类型。

### 完整代码实现

接下来，除了这三个`ModelConvention`类型，我把整个实现代码贴一下，让大家看得比较直观，最后会跟上一篇文章的实现加入进来。因为，如果没有昨天的工作，我们指定的类型不被MVC识别为控制器的话，我们是无法实现为这些类型添加MVC属性的。

首先，先创建一个控制台程序，引入一下Nuget包

```xml
    <PackageReference Include="Microsoft.AspNetCore.Hosting" Version="2.0.0" />
    <PackageReference Include="Microsoft.AspNetCore.Mvc.Core" Version="2.0.0" />
```

接着，定义一个接口以及它的实现，接口中标记了一些自定义特性，而实现类中完全没有：

```csharp
[Route("test")]
public interface ITestService
{
    [Route("{name}"), HttpGet]
    string Test(string name);
}

public class TestService : ITestService
{
    public string Test(string name)
    {
        return "Hello " + name;
    }
}
```

然后在控制台程序的入口文件Program.cs的Main方法中写入一下代码：

```csharp
internal class Program
{
    public static void Main(string[] args)
    {
       new WebHostBuilder()
            .UseKestrel()
            .UseUrls("http://localhost:8080")
            .ConfigureServices(services =>
            {
                //使用AddMvc亦可
                services.AddMvcCore(opt=>
                {
                    opt.Conventions.Add(new ControllerModelConvention(typeof(ITestService)));
                    opt.Conventions.Add(new ActionModelConvention(typeof(ITestService)));
                    opt.Conventions.Add(new ParameterModelConvention(typeof(ITestService)));
                })
                //下面这段是上一篇文章里的内容
                .ConfigureApplicationPartManager(manager =>
                {
                    var featureProvider = new ServiceControllerFeatureProvider(typeof(ITestService));
                    manager.FeatureProviders.Add(featureProvider);
                });
            })
            .Configure(app => app.UseMvc())
            .Build()
            .Start();
    }
}
```

一切编译通过后，点击运行，在浏览器中访问"[http://localhost:8080/test/elderjames](http://localhost:8080/test/elderjames)”，如果看到返回了“Hello elderjames”，那么就大功告成啦！

![](/images/add_webapi_to_class/1.png)

### 总结

这篇文章中主要介绍了通过实现[`IControllerModelConvention`](https://github.com/aspnet/Mvc/blob/de1b763d963f8be612d93889e08e43f67c98d0d5/src/Microsoft.AspNetCore.Mvc.Core/ApplicationModels/IControllerModelConvention.cs)、[`IActionModelConvention`](https://github.com/aspnet/Mvc/blob/b6a6b50776bb1ee49c0bca1353375e964101bb8a/src/Microsoft.AspNetCore.Mvc.Core/ApplicationModels/IActionModelConvention.cs)、[`IParameterModelConvention`](https://github.com/aspnet/Mvc/blob/16c267d95eafbf310f17bc938d00048a541be0d0/src/Microsoft.AspNetCore.Mvc.Core/ApplicationModels/IParameterModelConvention.cs)三个接口实现为指定为控制器的类型添加MVC特性的方法。

本篇文章发现源码的部分受到[*max zhang*](https://github.com/maxzhang1985) 和他的群里的群友*福州 | Today*的帮助，在此表示衷心的感谢。

在接下来的文章中，会介绍使用功能强大的.NTE Core开源AOP框架[**AspectCore**](https://github.com/dotnetcore/AspectCore-Framework)实现的动态代理客户端，注册以上所说的接口，即可获得可以调用对应的WebApi服务的功能。这些工作的源码可以在[我的框架示例项目](https://github.com/ElderJames/shriek-fx/tree/master/samples/Shriek.Samples.WebApiProxy)中运行，大家有兴趣可以看看效果。

**感谢阅读和批评指教！**

[温馨提示：为方便大家看文中提到的源码，原文中有大量指向源码链接，如果您看的是没有链接的转载，可以再来看我的原文。]