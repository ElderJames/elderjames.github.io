---
layout: post
title: .NET Core 3.0中用 Code-First 方式创建 gRPC 服务与客户端
permalink: code-first-generate-gRPC-services-and-clients-in-dotnet-Core-3_0
date: 2019-11-24 23:54:10
tags:
    - ASP.NET Core
    - .NET Core
categories:
    - .NET Core
---

## .NET Core love gRPC

千呼万唤的 .NET Core 3.0 终于在 9 月份正式发布，在它的众多新特性中，除了性能得到了大大提高，比较受关注的应该是 ASP.NET Core 3.0 对 gRPC 的集成了。
它的源码托管在 grpc-dotnet 这个 Github 库中，由微软 .NET 团队与谷歌 gRPC 团队共同维护.

.NET Core 对 gRPC 的支持在 grpc 官方仓库早已有实现（grpc/csharp），但服务端没有很好地与 ASP.NET Core 集成，使用起来还需要自己进行一些集成扩展。
而 ASP.NET Core 3.0 新增了 gRPC 服务的托管功能，能让 gRPC 与 ASP.NET Core 框架本身的特性很好地结合，如日志、依赖注入、身份认证和授权，并由 Kestrel 服务器提供 HTTP/2 链接，性能上得到充分保障。

推荐把项目中已有的 RPC 框架或者内部服务间 REST 调用都迁移到 gRPC 上，因为它已经是云原生应用的标准 RPC 框架，在整个 CNCF 主导下的云原生应用开发生态里 gRpc 有着举足轻重的地位。

对于 gRPC 的使用方式，前段时间已经有其他大神写的几篇文章了，这里就不再赘述了。
本文主要介绍的是区别于标准使用规范的，但对.NET 应用更加友好的使用方式，最后会提供源码来展示。

作为对比，还是要列一下标准的使用步骤：

1. 定义 proto 文件，包含服务、方法、消息对象的定义
2. 引入 `Grpc.Tools` Nuget 包并添加指定 proto 路径和生成模式
3. 生成项目，得到服务端的抽象类或客户端的调用客户端组件
4. 实现服务端抽象类，并在 ASP.NET Core 注册这个服务的路由端点
5. DI 注册 gRPC 服务。
6. 客户端用 `Grpc.Net.ClientFactory` Nuget 包进行统一配置和依赖注入

.NET Core 对 gRPC 的大力支持使开发者开发效率大大提高，入门难度也减少了许多，完全可以成为跟 WebApi 等一样的 .NET Core 技术栈的标配。

## proto 在单一语言系统架构中的局限性

使用 proto 文件的好处是多语言支持，同一份 proto 可以生成各种语言的服务和客户端，可以让用不同语言开发的微服务直接互相远程调用。但 proto 文件作为不同服务间的契约，不可以经常修改，否则就会对使用了它的服务造成不同程度的影响，因此对 proto 文件的版本控制需要得到重视。

另外，我们的应用程序还不应该与 gRPC 耦合，否则就会导致系统架构被这些实现细节所绑架。直接依赖 proto 文件和由它生成的代码，就是对 gRPC 的强耦合。

例如，当应用程序在演进的过程中，复杂度还未达到完全部署隔离的必要时，为了避免因“完全边界”引入的部署运维复杂性，又能预留隔离的可能性，需要有一层接口层作为“不完全边界”。

又比如，目前在 windows 系统的 iis 上还不支持 grpc-dotnet，当有 windows 上的应用程序需要使用 RPC，就需要换成 REST 的实现了。

因此，为了不让应用程序对 gRPC 过于依赖，还应该使用一层抽象（接口）层与其解耦，用接口来隔离对 RPC 实现的依赖，这样在需要使用不同的实现时，可以通过注册不同的实现来方便地切换。

在这些场景下，本文要介绍的 Code-First gRPC 使用方法就发挥作用了。

## Code-First gRPC

说了这么久，我好像还没正式介绍 Code-First gRPC，到底他有多适合在单一语言系统架构中实现 gRPC 呢？下面要介绍的就是基于大名鼎鼎的 protobuf-net 实现的 gRPC 框架，**protobuf-net.Grpc**。

protobuf-net 是在过去十几年前到现在一直在 .NET 中有名的 Protobuf 库，想用 Protobuf 序列化时就会用到这个库。他的特性就是可以把 C# 代码编写的类能以 Protobuf 的协议进行序列化和反序列化，而不是 proto 文件再生成这些类。而 protobuf-net.Grpc 则是一脉相承，可以把 C# 写的接口，在服务端方便地把接口的实现类注册成 ASP.NET Core 的 gRPC 服务，在客户端把接口动态代理实现为调用客户端，调用前面的这个服务端。

用法很简单，只要声明一个接口为您的服务契约：

```cs
[ServiceContract]
public interface IMyAmazingService {
    ValueTask<SearchResponse> SearchAsync(SearchRequest request);
    // ...
}
```

然后实现该接口的服务端：

```cs
public class MyServer : IMyAmazingService {
    // ...
}
```

或者向系统获取客户端：

```cs
var client = http.CreateGrpcService<IMyAmazingService>();
var results = await client.SearchAsync(request);
```

这相当于以下 .proto 中的服务：

```proto
service MyAmazingService {
    rpc Search (SearchRequest) returns (SearchResponse) {}
	// ...
}
```

`protobuf-net.Grpc` 同样通过普通类型定义支持 gRPC 的四种模式，把 C# 8.0 中最新的 `IAsyncEnumerable` 类型识别成 proto 中的 `stream`，单向流、双向流都可以实现！而且用 `IAsyncEnumerable` 实现可比 proto 生成的类方便很多。

例如 proto 双向流定义：

```
rpc chat(stream ChatRequest) returns ( stream ChatResponse);
```

生成出来的方法是：

```cs
Task BathTheCat(IAsyncStreamReader<ChatRequest> requestStream, IServerStreamWriter<ChatResponse> responseStream)
```

而 `protobuf-net.Grpc` 只要定义一个方法：

```cs
IAsyncEnumerable<ChatResponse> SubscribeAsync(IAsyncEnumerable<ChatRequest> requestStream);
```

由此可见，`protobuf-net.Grpc` 无需在契约层引入第三方库，充分运用了 C# 类型系统，把方法、类型映射到兼容了 gRPC 的服务定义上。

上文所说的 proto 局限也迎刃而解了，函数调用、gRPC、REST 都能方便切换。（REST 实现可以参考我的开源框架 shriek-fx 中的 Shriek.ServiceProxy.Http ）组件。

下一篇，我将主要介绍利用 `protobuf-net.Grpc` 的 gRPC 双向流模式与 `Blazor` 实现一个简单的在线即时聊天室。

![](https://raw.githubusercontent.com/ElderJames/GrpcChat/master/grpc-chat.gif)

相关链接：

-   protobuf-net.Grpc：https://github.com/protobuf-net/protobuf-net.Grpc
-   shriek-fx：https://github.com/Shriek-Projects/shriek-fx
-   GrpcChat: https://github.com/ElderJames/GrpcChat
