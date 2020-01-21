---
title: ASP.NET Core Support gRPC-Web
date: 2020-01-21 15:21:00
permalink: asp_net_core-support-grpc-web
tags: 
- ASP.NET Core
- .NET Core
- gRPC-Web
categories:
- .NET Core
---

> grpc-dotnet 项目在 PR #695 完成了 ASP.NET Core 服务与 .NET Core gRPC 客户端的 gRPC-Web 实现。
> 虽然目前还是实验性项目，但是并不阻碍我们为之兴奋。下面我们来看看如何使用。

## gRPC-Web 简介

gRPC-Web 允许从浏览器应用程序使用 gRPC，gRPC-Web 支持在新场景中使用 gRPC：

- JavaScript 浏览器应用程序可以使用 gRPC-Web JavaScript 客户端调用 gRPC 服务。
- Blazor WebAssembly 应用程序可以使用.Net Core gRPC 客户端调用 gRPC 服务。
- 可以让 gRPC 服务被用于不完全支持 HTTP/2 的环境中。
- 可以让 gRPC 用于 HTTP/2 中没有的技术，例如 Windows 身份验证。

Grpc.AspNetCore.Web 和 Grpc.Net.Client.Web 提供了扩展来为 .NET Core 支持端到端的 gRPC-Web。

## 服务端使用 Grpc.AspNetCore.Web

Grpc.AspNetCore.Web 提供了中间件使 ASP.NET Core gRPC 服务接受 gRPC- web 调用。

_Startup.cs_

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddGrpc();
    services.AddGrpcWeb(o => o.GrpcWebEnabled = true);
}

public void Configure(IApplicationBuilder app)
{
    app.UseRouting();
    app.UseGrpcWeb();
    app.UseEndpoints(endpoints =>
    {
        endpoints.MapGrpcService<GreeterService>();
    });
}
```

gRPC-Web 可以通过设置 `GrpcWebOptions.GrpcWebEnabled = true` 来被应用于所有 gRPC 服务，或者通过`EnableGrpcWeb()`方法被应用于单个服务：

```c
app.UseEndpoints(endpoints =>
{
    endpoints.MapGrpcService<GreeterService>().EnableGrpcWeb();
});
```

## 客户端使用 Grpc.Net.Client.Web

Grpc.Net.Client.Web 提供了一个 HttpClient delegating handler 来配置 .NET Core gRPC 客户端发送 gRPC-Web 请求。

```c
// Create channel
var handler = ew GrpcWebHandler(GrpcWebMode.GrpcWebText, new HttpClientHandler());
var channel = GrpcChannel.ForAddress("https://localhost:5001", new GrpcChannelOptions
    {
        HttpClient = new HttpClient(handler)
    });

// Make call with a client
var client = Greeter.GreeterClient(channel);
var response = await client.SayHelloAsync(new GreeterRequest { Name = ".NET" });
```

## 总结

可以看出，得益于 ASP.NET Core 3.0 以来对 gRPC 的深度集成，增加 gRPC 相关的新特性已经非常容易，使.NET Core 成为云原生家族的重要成员。

近期，观测分析平台 SkyWalking 的 .NET 自动探针 (SkyAPM-dotnet) 也已经支持了 grpc-dotnet 远程调用的链路跟踪采集，欢迎大家使用！如果喜欢，也请大家给点个星星！

项目地址：https://github.com/SkyAPM/SkyAPM-dotnet
