---
title: 在 Blazor WebAssembly 中使用 gRPC-Web
date: 2020-01-21 15:21:00
permalink: Using-gRPC-Web-with-Blazor-WebAssembly
tags: 
- ASP.NET Core
- .NET Core
- Blazor
- gRPC-Web
categories:
- .NET Core
---

> 对于单页面应用程序，gRPC-Web 是 JSON-over-HTTP 的一种方便、高性能的替代方案。

*如果你已经了解关于 gRPC 和 gRPC-Web 的一切，你可以跳到 添加 gRPC 服务到一个Blazor WebAssembly 应用程序 一节。如果你只是想要一些简单的 Blazor WebAssembly + gRPC-Web 应用程序，请看这个仓库 https://github.com/SteveSandersonMS/BlazorGrpcSamples。*

# 现状

与其他所有基于浏览器的单页应用程序（SPA）技术一样，在 Blazor WebAssembly 中，数据交互和触发服务器端操作的最常见方式是 JSON-over-HTTP。它很简单：客户端使用预先约定的 HTTP 方法向某个预先约定的 URL 发出 HTTP 请求，然后服务器执行操作并使用预先约定的 HTTP 状态码和预先约定格式的 JSON 数据进行应答。

这种方法通常很有效，这样做的人往往能过上充实的生活。然而，它也有两个显而易见的弱点：

- JSON 是一种非常冗长的数据格式。它没有优化带宽。
- 没有任何机制可以保证所有这些预先约定的 url、HTTP 方法、状态码 等等的细节在服务器和客户端之间实际上是一致的。

## 什么是 gRPC?

gRPC 是一种远程过程调用（RPC）机制，最初由谷歌开发。对于 SPA，你可以将其视为 JSON-over-HTTP 的替代方案。它直接修复了上面列出的两个弱点:
- 它被优化为最小的网络流量，发送有效的二进制序列化消息
- 你可以在编译时保证服务器和客户端对端点的存在、数据的发送和接收形式达成一致，而不需要指定任何URL、状态码 等等。

## 这是怎么做到的呢？

要编写gRPC服务，你需要编写一个`.proto`文件，它是一组 RPC 服务及其数据形状的独立于语言的描述。从这里，你可以用任何语言生成强类型的服务器和客户端类，从而保证在编译时符合你的协议。然后在运行时，gRPC 处理(反)序列化数据并以有效的格式（默认为 protobuf ）发送/接收消息。
另一个很大的好处是它不是 REST，所以你不必与同事经常争论哪些 HTTP 方法和状态代码在你的场景中是最幸福和幸运的。它只是简单的 RPC，在我们还是小孩子的时候就真情实感想要的。

## 为什么不是每个人都在他们的SPA中使用 gRPC ?

传统上，从基于浏览器的应用程序中使用 gRPC 是不可能的，因为 gRPC 需要 HTTP/2，而且浏览器不公开任何 api，让 JS、WASM 代码直接控制 HTTP/2 请求。
但是现在有一个解决方案！gRPC-Web 是 gRPC的扩展，它使 gRPC 与基于浏览器的代码兼容（从技术上讲，它是通过 HTTP/1.1 请求执行 gRPC 的一种方式）。gRPC-Web 还没有流行起来，因为到目前为止还没有多少服务器或客户端框架提供对它的支持。
ASP.NET Core 从 3.0 版本开始就提供了强大的 gRPC 支持。现在，在此基础上，我们将在服务器和客户端提供对 gRPC-Web的预览支持。如果你想深入了解细节，可以在来自 James Newton-King 的优秀的 pull request 查看全部实现。

# 添加 gRPC 服务到一个Blazor WebAssembly 应用程序

目前还没有这方面的项目模板，所以将 gRPC 支持添加到 Blazor WebAssembly 应用程序需要很多步骤，本文是详细的介绍。但好消息是你只需要做一次这样的设置。当你完成起步与运行起来后，添加更多的 gRPC 端点并调用它们是非常简单的。
首先，由于 gRPC-Web 包还没有发布到 NuGet.org，现在你需要添加两个临时的包管理源来获得 nightly 预览。
你可以在你的解决方案的根目录下添加`NuGet.config`文件。希望一两个月后就不需要了。

## 添加 gRPC 服务到一个托管部署的Blazor WebAssembly 应用程序

如果你已经在 ASP.NET Core 服务端上托管了一个Blazor WebAssembly 应用程序，默认情况下，你有三个项目：客户端、服务端和共享项目。我发现定义 gRPC 服务最方便的地方是在共享项目中，因为这样生成的类对服务器和客户机都可用。
首先，编辑你的共享项目的`.csproj`添加必要的 gRPC 包引用：

```xml
  <ItemGroup>
    <PackageReference Include="Google.Protobuf" Version="3.11.2" />
    <PackageReference Include="Grpc.Net.Client" Version="2.27.0-dev202001100801" />
    <PackageReference Include="Grpc.Tools" Version="2.27.0-dev202001081219">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
    </PackageReference>
  </ItemGroup>
```
如果这些包不能正确恢复，请确保添加了 nightly 源。

现在可以创建`.proto`文件来定义服务了。例如，在共享项目中添加一个名为`greet.proto`的文件。包含如下内容：

```proto
syntax = "proto3";
option csharp_namespace = "GrpcGreeter";
package greet;

service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply);
}

message HelloRequest {
  string name = 1;
}

message HelloReply {
  string message = 1;
}
```

要使gRPC工具从这里生成服务器和客户端类，请进入共享项目的`.csproj`并添加以下内容:

```xml
  <ItemGroup>
    <Protobuf Include="greet.proto" />
  </ItemGroup>
```

此时，解决方案应该可以没有错误地编译通过。

### 从服务端公开gRPC服务

在你的服务器项目中，创建一个名为`GreeterService`的新类，包含以下内容:

```csharp
using Grpc.Core;
using GrpcGreeter;

public class GreeterService : Greeter.GreeterBase
{
    public override Task<HelloReply> SayHello(HelloRequest request, ServerCallContext context)
    {
        var reply = new HelloReply { Message = $"Hello from gRPC, {request.Name}!" };
        return Task.FromResult(reply);
    }
}
```
这个类继承自 `Greeter.GreeterBase`，它是从 `.proto` 文件自动生成的。因此，当你将新断点添加到 `.proto` 中时，你将有新方法可以在这里进行重写来提供具体实现。

服务端的最后一部分是使用新的 api 将其公开为一个 `gRPC-Web` 服务。你需要为此引用一个包，因此在你的服务端项目的`.csproj`中，添加以下包引用:

```xml
<PackageReference Include="Grpc.AspNetCore" Version="2.27.0-dev202001100801" />
<PackageReference Include="Grpc.AspNetCore.Web" Version="2.27.0-dev202001100801" />
```
现在，在你的服务器的`Startup.cs`文件中，修改`ConfigureServices`以添加以下行:

```csharp
  services.AddGrpc();
```

*注意:如果你只打算公开gRPC服务，你可能不再需要MVC控制器，在这种情况下，你可以从下面删除`services.AddMvc() `和`endpoints.MapDefaultControllerRoute()`。*

只是在 `app.AddRouting();` 的下面添加以下内容，它会处理将传入的gRPC-web请求映射到服务端，使其看起来像 gRPC请求:

```
  app.UseGrpcWeb();
```

最后，在`app.UseEndpoints`语句块中注册你的 gRPC-Web 服务类，并在该语句块的顶部使用以下代码行:
```csharp
  endpoints.MapGrpcService<GreeterService>().EnableGrpcWeb();
```

就这样，你的gRPC-Web服务端已经准备好了!

### 在客户端中使用gRPC服务

在你的客户端项目的`.csproj`中，你需要添加以下两个 nightly 包的引用:

```xml
<PackageReference Include="Grpc.Net.Client.Web" Version="2.27.0-dev202001100801" />
<PackageReference Include="Microsoft.AspNetCore.Blazor.Mono" Version="3.2.0-preview1.20052.1" />
```

后者是为了修复 Blazor WebAssembly 中的一个问题的变通方案，这个问题将在几周后的下一次预览中得到修复。如果这些包不能正常恢复，请确保添加了 nightly 源。

现在设置你的客户端应用程序的依赖项注入系统，以便能够提供`GreeterClient`的实例。这将允许你在客户端应用程序的任何位置调用 gRPC 服务。在客户端项目的`Startup.cs`中的`ConfigureServices`方法中添加以下内容:

```csharp
services.AddSingleton(services =>
{
    // Create a gRPC-Web channel pointing to the backend server
    var httpClient = new HttpClient(new GrpcWebHandler(GrpcWebMode.GrpcWeb, new HttpClientHandler()));
    var baseUri = services.GetRequiredService<NavigationManager>().BaseUri;
    var channel = GrpcChannel.ForAddress(baseUri, new GrpcChannelOptions { HttpClient = httpClient });

    // Now we can instantiate gRPC clients for this channel
    return new Greeter.GreeterClient(channel);
});
```

需要注意的是`Greeter.GreeterClient`是从`.proto file`文件中为你生成的代码。你不必手动实现它！但你还需要添加以下使用语句，使上述代码编译通过：

```csharp
using GrpcGreeter;
using Grpc.Net.Client;
using Grpc.Net.Client.Web;
using Microsoft.AspNetCore.Components;
using System.Net.Http;
```

我们快完成了！现在有一个可工作的服务端，并希望还有一个可工作的客户端。我们只需要从 UI 调用一些 gRPC 服务。例如，在你的`Index.razor`文件中，替换成如下内容： 

```csharp
@page "/"
@using GrpcGreeter
@inject Greeter.GreeterClient GreeterClient

<h1>Invoke gRPC service</h1>

<p>
    <input @bind="yourName" placeholder="Type your name" />
    <button @onclick="GetGreeting" class="btn btn-primary">Call gRPC service</button>
</p>

Server response: <strong>@serverResponse</strong>

@code {
    string yourName = "Bert";
    string serverResponse;

    async Task GetGreeting()
    {
        var request = new HelloRequest { Name = yourName };
        var reply = await GreeterClient.SayHelloAsync(request);
        serverResponse = reply.Message;
    }
}
```

现在在浏览器中尝试一下。用户界面看起来是这样的:

![](https://blog.stevensanderson.com/wp-content/uploads/2020/01/15/greeter-ui.png)

如果你查看浏览器开发者工具的 network 选项卡中的请求，你会看到它在发送和接收二进制protobuf消息:

![](https://blog.stevensanderson.com/wp-content/uploads/2020/01/15/greeter-request.png)

### 总结和示例

现在你已经有了基础，如果你愿意，还可以进一步使用 gRPC 来进行服务器和客户机之间的所有数据交换。gRPC 工具将为你生成所有的数据传输类，提高网络流量的效率，并消除 url、HTTP 方法、状态代码和序列化等 HTTP-over-JSON 的问题。

还有一个更详细的示例（https://github.com/SteveSandersonMS/BlazorGrpcSamples/tree/master/Hosted），它是一个完整的 Blazor WebAssembly 托管应用程序，使用 gRPC 获取“天气预报”数据。如果你对从默认的基于json的解决方案升级到基于gRPC-Web的解决方案所需的具体步骤感兴趣，请参阅这个准确地显示了我所做的更改的差异对比（https://github.com/SteveSandersonMS/BlazorGrpcSamples/commit/72544c54085a35cd89aae20030d7f91d75317a2f）。

## 添加 gRPC 服务到一个独立部署的 Blazor WebAssembly 应用程序

如果你正在构建一个纯独立的 Blazor WebAssembly 应用程序，而不是托管在 ASP.NET Core，那么我们就不能对你将拥有什么样的服务器做任何假设（意思是你只开发客户端，而服务端是由别人开发的场景）。我们只能假设你要调用一些与 gRPC-Web 兼容的服务端点，它们可能是在其他主机上的 ASP.NET Core 服务上暴露，或者是一个在另一个 gRPC 服务周围的 Envoy gRPC-Web 包装器（https://blog.envoyproxy.io/envoy-and-grpc-web-a-fresh-new-alternative-to-rest-6504ce7eb880）。在这里我们唯一关心的是配置你的 Blazor WebAssembly 应用程序来使用它。
设置客户端应用程序的大多数步骤与上面的“托管部署”情况相同。然而，在某些方面，它有点棘手，因为我们不能依赖于我们在“托管”情况下所做的一些假设。区别如下:

- 获取和使用.proto文件

  假定你的外部 gRPC 服务维护者可以为你提供定义该服务的`.proto`文件。你可以将其复制到你的客户端项目中，并在`.csproj`中添加一个引用它的`<Proto>`项。为了让工具工作，你还需要添加类似这样的包引用:

  ```xml
    <!-- Needed temporarily until the next Blazor WebAssembly preview release -->
    <PackageReference Include="Microsoft.AspNetCore.Blazor.Mono" Version="3.2.0-preview1.20052.1" />

    <!-- gRPC-Web packages -->
    <PackageReference Include="Google.Protobuf" Version="3.11.2" />
    <PackageReference Include="Grpc.Net.Client" Version="2.27.0-dev202001100801" />
    <PackageReference Include="Grpc.Net.Client.Web" Version="2.27.0-dev202001100801" />
    <PackageReference Include="Grpc.Tools" Version="2.27.0-dev202001081219">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
    </PackageReference>
  ```
- 配置客户端DI服务

  与“托管托管”情况类似，你将向Startup.cs中添加代码以添加gRPC-Web客户端服务。这里的主要区别是，你必须知道用于访问外部服务的 Base URL 是什么，因为我们不能再假设它的 Base URL 跟托管你的客户端应用程序的服务端的 Base URL 一样。因此，你的DI服务注册可能看起来更像以下内容:
  
  ```csharp
  services.AddSingleton(services =>
  {
  #if DEBUG
      var backendUrl = "https://localhost:5001"; // Local debug URL
  #else
      var backendUrl = "https://some.external.url:12345"; // Production URL
  #endif

      // Now we can instantiate gRPC clients for this channel
      var httpClient = new HttpClient(new GrpcWebHandler(GrpcWebMode.GrpcWeb, new HttpClientHandler()));
      var channel = GrpcChannel.ForAddress(backendUrl, new GrpcChannelOptions { HttpClient = httpClient });
      return new Greeter.GreeterClient(channel);
  });
  ```
  这将根据你是否在调试模式下构建而改变URL。
  
### 总结和示例

就是这样！现在，你的独立部署的 Blazor WebAssembly 应用程序可以使用外部的 gRPC-Web 服务。对于完整的可运行示例，下面是一个示例独立应用程序（`https://github.com/SteveSandersonMS/BlazorGrpcSamples/tree/master/Standalone`），它在外部 URL 上调用一个 gRPC-Web 服务。 这个示例附带了一个实际的用于测试的 gRPC-Web 服务器，但是你可以分开考虑它。如果你想确切地看到我更改了什么，这里是与默认项目模板输出的差异（`https://github.com/SteveSandersonMS/BlazorGrpcSamples/commit/d6ec609f2b7e6591958d38e4a207c9b4f52f0feb`）。

# 你的反馈要求

如果你想进一步了解 gRPC，看看 ASP.NET Core gRPC 文档。请给我们关于你对 gRPC-Web 的意见和经验的反馈，因为这将帮助我们选择如何以及是否在未来的 ASP.NET Core 版本中使 gRPC-Web 成 一个标准特性。你可以在这里发布评论，或者在 GitHub 上发布标题中带有“反馈”的 issue。