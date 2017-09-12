---
title: ASP.NET Core中POCO控制器特性：把普通类变成控制器
date: 2017-09-12 12:00:00
permalink: add-the-webapi-service-to-the-general-class-using-the-POCO-controller-feature-of-ASP.NET-Core
tags: 
- ASP.NET Core
- .NET Core
- WebApi
categories:
- .NET Core
description: 这篇文章主要就是说说我最近在为自己的框架写基础设施的时候，实现一个类库，能把引用它的类库中指定接口的实现类变成Controller，实现无MVC依赖添加WebApi服务的过程。
thumbnail: 
---

POCO Controller是 **ASP.NET Core** 中的一个特性，虽然在2015年刚发布的时候就有这个特性了，可是大多数开发者都只是按原有的方式去写，而没有用到这个特性，但是把这个特性稍微封装后用在SOA架构中Service层的场景中是极其便利的。这篇文章主要就是说说我最近在为自己的框架写基础设施的时候，实现一个类库，能把引用它的类库中指定接口的实现类变成Controller，实现无MVC依赖添加WebApi服务的过程。

### POCO控制器简介

POCO控制器就是ASP.NET Core项目中所有带有`Controller`后缀的类、或者标记了`[Controller]`特性的类，虽然没有像模版项目中那样继承自**Controller类**，也会被识别为控制器,拥有跟普通控制器一样的功能，像下面这段代码中，两个类都会被识别成控制器：

```
public class PocoController
{
    public IActionResult Index()
    {
        return new ContentResult() { Content = “Hello from POCO controller!” };
    }
}

[Controller]
public class Poco
{
    public IActionResult Index()
    {
        return new ContentResult() { Content = “Hello from POCO controller!” };
    }
}
```

### POCO控制器原理

其实，在ASP.NET Core中，已经不像ASP.NET WebApi 那样，通过ControllerFactory来创建Controller，多亏于ASP.NET Core一脉相承的IoC框架 `Microsoft.Extensions.DependencyInjection`,ASP.NET Core中的内部实现变得更优雅。其中POCO控制器的核心原理就在[IApplicationFeatureProvider<ControllerFeature>](https://github.com/aspnet/Mvc/blob/2bacb6003f3bc2c3b58107c1118346dca3f5fa13/src/Microsoft.AspNetCore.Mvc.Core/ApplicationParts/IApplicationFeatureProviderOfT.cs)这个接口的实现[ControllerFeatureProvider](https://github.com/aspnet/Mvc/blob/760c8f38678118734399c58c2dac981ea6e47046/src/Microsoft.AspNetCore.Mvc.Core/Controllers/ControllerFeatureProvider.cs#L15)。

通过[aspnet/Mvc项目的Github源码仓库](https://github.com/aspnet/Mvc/search?p=1&q=IApplicationFeatureProvider&type=&utf8=%E2%9C%93)中查询得知，Mvc里把Controller、ViewComponent、TagHelper、Views等组件定义为特性(Feature)，如ControllerFeature，特性里就存放了应用中被识别为相组件的类型的集合，如如ControllerFeature中就存放了所有Controller类型。`IApplicationFeatureProvider<ControllerFeature>`这个接口是用来给MVC应用提供的控制器类型识别的接口，当把这个接口的实现注册到服务配置中，就能为其中识别的类型提供控制器功能。

[ControllerFeatureProvider](https://github.com/aspnet/Mvc/blob/760c8f38678118734399c58c2dac981ea6e47046/src/Microsoft.AspNetCore.Mvc.Core/Controllers/ControllerFeatureProvider.cs#L15)是这个接口的默认实现，其中有一个方法`IsController(TypeInfo typeInfo)`的功能就是判断某类型是否为控制器的。而接口方法`PopulateFeature(IEnumerable<ApplicationPart> parts,ControllerFeature feature)`则为把传入的 “Mvc应用部分（ApplicationPart，大概是指Mvc的作用程序集）”中的类型都一一判断，如果是控制器，那么就加入控制器特性对象中。

### 实现自定义判断规则

通过上面的剖析，我们就知道要实现自定义的控制器判断规则，只需要重写ControllerFeature类或者重新实现IApplicationFeatureProvider<ControllerFeature>接口，但是由于PopulateFeature不是虚方法或抽象方法，所以不能被重写，那么只能重新写一个类来实现IApplicationFeatureProvider<ControllerFeature>接口了。为了兼容原来规则，我把原来的规则照搬过来，复制了`IsController`的方法（开源的好处），并且在PopulateFeature中加入了自己的规则。先贴代码,避免篇幅过长，`IsController`方法的实现就直接链接到[源码](https://github.com/aspnet/Mvc/blob/760c8f38678118734399c58c2dac981ea6e47046/src/Microsoft.AspNetCore.Mvc.Core/Controllers/ControllerFeatureProvider.cs#L41)了：

```CSharp

 internal class ServiceControllerFeatureProvider : IApplicationFeatureProvider<ControllerFeature>
    {
        private const string ControllerTypeNameSuffix = "Controller";
        private readonly IEnumerable<Type> ServiceTypes;

        public ServiceControllerFeatureProvider(IEnumerable<Type> ServiceTypes)
        {
            this.ServiceTypes = ServiceTypes;
        }

        public void PopulateFeature(IEnumerable<ApplicationPart> parts,
            ControllerFeature feature)
        {
            foreach (var type in Reflection.CurrentAssembiles.SelectMany(o => o.DefinedTypes))
            {
                if (IsController(type) || ServiceTypes.Any(o => type.IsClass && o.IsAssignableFrom(type)) && !feature.Controllers.Contains(type))
                {
                    feature.Controllers.Add(type);
                }
            }
        }

        protected bool IsController(TypeInfo typeInfo)
        {
            //...
        }
    }

```

上面代码的原理，是按照我的框架的需求来改写的，构造方法传入的参数ServiceTypes是定义了服务方法的接口的类型，接口和对应实现类似于以下代码，这些代码可以写在一个.NET Core控制台项目中。

```CSharp
    public interface ITestService
    {
        string Test(string name);
    }

    [Route("test")]
    public class TestService : ITestService
    {
        [Route("{name}"), HttpGet]
        public string Test(string name)
        {
            return "Hello " + name;
        }
    }
```

其中TestService类就是会被识别为控制器的类，但是接口和实现是可以分开在不同程序集的。通过原本`ControllerFeatureProvider`类中`PopulateFeature`方法的`parts`参数中的类型是不包括除了引用了Mvc的程序集的其它程序集的，所以我这里用自己实现的类型扫描类`Reflection`中的`CurrentAssembiles`静态变量来获取当前应用程序的所有引用的（自己创建的项目）程序集的，具体实现的代码在我的框架[Shriek]的[源码中](https://github.com/ElderJames/shriek-fx/blob/master/src/Shriek/Utils/Reflection.cs)。


### 看看效果

现在，在TestService类所在项目文件中引入以下Nuget包：

```xml
    <PackageReference Include="Microsoft.AspNetCore.Hosting" Version="2.0.0" />
    <PackageReference Include="Microsoft.AspNetCore.Mvc.Core" Version="2.0.0" />
    <PackageReference Include="Microsoft.AspNetCore.Mvc.Formatters.Json" Version="2.0.0"/>
```

创建一个Startup.cs文件,写入以下代码：

```CSharp
    public class Startup
    {
        public void ConfigureServices(IServiceCollection services)
        {
            var builder = services.AddMvcCore()
                  .AddJsonFormatters()
                  .AddFormatterMappings();

            builder.ConfigureApplicationPartManager(manager =>
            {
                var featureProvider = new ServiceControllerFeatureProvider(typeof(ITest));
                manager.FeatureProviders.Add(featureProvider);
                builder.Services.AddSingleton<IApplicationFeatureProvider<ControllerFeature>>(featureProvider);
            });
        }

        public void Configure(IApplicationBuilder app, IHostingEnvironment env)
        {
            app.UseMvc();
        }
    }
```

然后在控制台程序的入口文件Program.cs的Main方法中写入一下代码：

```CSharp
    internal class Program
    {
        public static void Main(string[] args)
        {
            new WebHostBuilder()
                .UseKestrel()
                .UseStartup<Startup>()
                .UseUrls("http://*:8080")
                .Build()
                .Start();
        }
    }
```

一切编译通过后，点击运行，在浏览器中访问"[http://localhost:8080/test/elderjames](http://localhost:8080/test/elderjames)”,如果看到返回了“Hello elderjames”，那么就大功告成啦！



### 总结

这篇文章中主要介绍了通过实现`IApplicationFeatureProvider<ControllerFeature>`接口实现自定义控制器的方法。其实这个方法的发现源于我在社区交流中Lemon同学的指点。在接下来的文章中，我会介绍如何从接口获取自定义特性标签，实现从接口获得mvc特性，而接口和实现类都不依赖mvc的方法。敬请期待！