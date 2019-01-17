---
layout: post
title: 利用Topshelf把.NET Core Generic Host管理的应用程序部署为Windows服务
permalink: Using-Topshelf-To-Deploy-Net-Core-Generic-Host-App-To-Windows-Services
date: 2019-01-12 18:31:34
tags:
- .NET Core
categories:
- .NET Core
---

# 背景

2019第一篇文章。

此文源于前公司在迁移项目到.NET Core的过程中，希望使用Generic Host来管理定时任务程序时，没法部署到Windows服务的问题，而且官方也没给出解决方案，只能关注一下[官方issue #809](https://github.com/aspnet/Extensions/issues/809) 等他们方解决了。

官方文档只提供了一个[《在 Windows 服务中托管 ASP.NET Core》](https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/windows-service?spm=a2c4e.11153940.blogcont676413.12.65753eb0BIzfmE&view=aspnetcore-2.1)的方案，可以使用`Microsoft.AspNetCore.Hosting.WindowsServices`类库来把Web应用部署为Windows服务。但是ASP.NET Core虽然是控制台程序，但是它本身是使用了含有HTTP管道的Web Host来负责应用程序的生命周期管理，用它来作为定时任务的话，会有很多不必要的工作负载，例如占用端口、增加了很多依赖等等。

官方意识到这个问题之后，在.NET Core 2.1版本新增了Generic Host通用主机，剥离了原来WebHost的Http管道相关的API，源码中可以发现Web Host已经基于Generic Host实现。它才是作为纯粹定时任务程序的最佳拍档。

但是由于Generic Host本身非常简单，用它运行的程序设置在注册为Windows服务启动之后会自动停止。研究很久之后才知道，想在Windows上启动服务，还是不能像Linux上那么简单——

于是尝试结合Topshelf来创建Windows服务，最终成功了。

## 实现方法

1. 先实现`IHostLifetime`接口来接管应用程序的生命周期，其实就是用空的实现来替换掉默认的ConsoleLifetime，这样就可以在之后由Topshelf框架内部去管理生命周期。

```cs
    internal class TopshelfLifetime : IHostLifetime
    {
        public TopshelfLifetime(IApplicationLifetime applicationLifetime, IServiceProvider services)
        {
            ApplicationLifetime = applicationLifetime ?? throw new ArgumentNullException(nameof(applicationLifetime));
        }

        private IApplicationLifetime ApplicationLifetime { get; }

        public Task WaitForStartAsync(CancellationToken cancellationToken)
        {
            return Task.CompletedTask;
        }

        public Task StopAsync(CancellationToken cancellationToken)
        {
            return Task.CompletedTask;
        }
    }
```

2. 然后实现`IHostedService`接口，把后台任务逻辑写到`StartAsync`方法中，参见官方文档[《在 ASP.NET Core 中使用托管服务实现后台任务》](https://docs.microsoft.com/zh-cn/aspnet/core/fundamentals/host/hosted-services?view=aspnetcore-2.1#ihostedservice-interface)，本文示例使用定时写入文本到一个文件来测试定时任务是否成功运行。

```cs
    internal class FileWriterService : IHostedService, IDisposable
    {
        private static string path = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, @"test.txt");

        private Timer _timer;

        public Task StartAsync(CancellationToken cancellationToken)
        {
            if (cancellationToken.IsCancellationRequested) return Task.FromCanceled(cancellationToken);

            _timer = new Timer(
                (e) => WriteTimeToFile(),
                null,
                TimeSpan.Zero,
                TimeSpan.FromSeconds(10));

            return Task.CompletedTask;
        }

        public void WriteTimeToFile()
        {
            if (!File.Exists(path))
            {
                using (var sw = File.CreateText(path))
                {
                    sw.WriteLine(DateTime.Now);
                }
            }
            else
            {
                using (var sw = File.AppendText(path))
                {
                    sw.WriteLine(DateTime.Now);
                }
            }
        }

        public Task StopAsync(CancellationToken cancellationToken)
        {
            _timer?.Change(Timeout.Infinite, 0);

            return Task.CompletedTask;
        }

        public void Dispose()
        {
            _timer?.Dispose();
        }
    }
```

3. 构建Generic Host，在`ConfigureServices`方法中注册`TopshelfLifetime`，并且注册一个托管服务`FileWriterService`，就能完成Generic Host的简单构建，当然完整的项目应该还包含配置、日志等等。最后，使用Topshelf来接管Generic Host，创建Windows服务。

```cs
    internal class Program
    {
        private static void Main(string[] args)
        {
            var builder = new HostBuilder()
                .ConfigureServices((hostContext, services) =>
                {
                    services.AddSingleton<IHostLifetime, TopshelfLifetime>();
                    services.AddHostedService<FileWriterService>();
                });

            HostFactory.Run(x =>
            {
                x.SetServiceName("GenericHostWindowsServiceWithTopshelf");
                x.SetDisplayName("Topshelf创建的Generic Host服务");
                x.SetDescription("运行Topshelf创建的Generic Host服务");

                x.Service<IHost>(s =>
                {
                    s.ConstructUsing(() => builder.Build());
                    s.WhenStarted(service =>
                    {
                        service.Start();
                    });
                    s.WhenStopped(service =>
                    {
                        service.StopAsync();
                    });
                });
            });
        }
    }
```

3. 最后发布应用程序，并安装到Windows服务。

以管理员权限开启终端，执行命令：

```bash
  dotnet publish -c release -r win-x64
  
  cd path-to-project/bin/release/netcoreapp2.1/win-x64/publish

  ./project-name install

  net start GenericHostWindowsServiceWithTopshelf

```

![](/images/generic-host/generic-host-install.png)

这样这个Windows服务就启动了！查看输出文件，可以看到定时写入成功，服务也一直没关闭~

![](/images/generic-host/generic-host-result.png)

## 示例代码

https://github.com/ElderJames/GenericHostWindowsServiceWithTopshelf

## 参考链接

官方文档[《.NET 通用主机》](https://docs.microsoft.com/zh-cn/aspnet/core/fundamentals/host/generic-host?view=aspnetcore-2.1)

官方文档[《在 ASP.NET Core 中使用托管服务实现后台任务》](https://docs.microsoft.com/zh-cn/aspnet/core/fundamentals/host/hosted-services?view=aspnetcore-2.1)