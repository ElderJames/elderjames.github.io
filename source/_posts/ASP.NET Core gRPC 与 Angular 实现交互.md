---
title: 前端 JS/TS 调用 ASP.NET Core gRPC-Web
date: 2020-02-07 14:42:00
permalink: call-asp_net_core-grpc-web-with-js-and-ts
tags:
    - ASP.NET Core
    - .NET Core
    - gRPC-Web
categories:
    - .NET Core
---

# 前言

在上两篇文章中，介绍了[ASP.NET Core 中的 gRPC-Web 实现](/asp_net_core-support-grpc-web.html) 和 [在 Blazor WebAssembly 中使用 gRPC-Web](/Using-gRPC-Web-with-Blazor-WebAssembly.html)，实现了 Blazor WebAssembly 调用 ASP.NET Core gRPC-Web。虽然 ASP.NET Core 中的 gRPC-Web 实现目前还是试验性项目，但是鉴于它在生态上的重大意义，说不定我们很快就能在正式版本中使用。

虽然 Blazor WebAssembly 现在已经是 .NET 进军前端的大热门，但有同学说，只介绍了 Blazor WebAssembly 的调用方法还不够呀，现在比较常用的还是 JS/TS 前端，那么本篇，我就介绍一下在前端 JS/TS 中调用 ASP.NET Core gRPC-Web。

其实 gRPC-Web 项目本身，就是为 JS/TS 提供 gRPC 能力的，让不支持 HTTP/2 的客户端和服务端也能使用 gRPC 的大部分特性。gRPC-Web 项目提供了一个 protoc CLI 插件，可用于把 proto 协议文件转换为 JS/TS 语言可导入的对应 gRPC 服务的客户端，还生成了 `.d.ts` 文件来支持 Typescript。

# 示例

接下来，我就来展示一下，用 Visual Studio 自带的 ASP.NET Core + Angular 模板创建的项目，把原来的 WebApi 调用改造成 gRPC-Web 调用。

本示例基于 .NET Core 3.1，请安装好最新的 .NET Core SDK 和 Visual Studio 2019。

## 创建项目

打开 Visual Studio 2019，创建新项目 -> 选择"ASP.NET Core Web 应用程序" -> 填写项目名 -> 选择 "Angular" 项目模板。如图：

![](/images/aspnetcore-grpc-web/1.png)

我们就是用这个项目，把 fetch-data 页面获取数据的方式修改为 gRPC-Web 。

## 添加 gRPC proto 文件

在项目中新建一个目录 `Protos`，创建文件 `weather.proto` :

```
syntax = "proto3";

import "google/protobuf/empty.proto";
import "google/protobuf/timestamp.proto";

option csharp_namespace = "AspNetCoreGrpcWeb";

package WeatherForecast;

service WeatherForecasts {
  rpc GetWeather (google.protobuf.Empty) returns (WeatherReply);
}

message WeatherReply {
  repeated WeatherForecast forecasts = 1;
}

message WeatherForecast {
  google.protobuf.Timestamp dateTimeStamp = 1;
  int32 temperatureC = 2;
  int32 TemperatureF = 3;
  string summary = 4;
}

```

可以看到 proto 中导入了官方库的其他 proto 文件，它们是编译用的辅助文件。对于.NET Core 项目，可以通过引用 `Google.Protobuf` 这个包引入。

## 修改 ASP.NET Core 服务端

我们先修改服务端，让 ASP.NET Core 提供 gRPC-Web 服务。

由于 gRPC-Web 包还没有发布到 NuGet.org，现在你需要添加一个临时的包管理源来获得 nightly 预览。在你的解决方案的根目录下添加`NuGet.config`文件：

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <!--To inherit the global NuGet package sources remove the <clear/> line below -->
    <clear />
    <add key="nuget" value="https://api.nuget.org/v3/index.json" />
    <add key="gRPC-nightly" value="https://grpc.jfrog.io/grpc/api/nuget/v3/grpc-nuget-dev" />
  </packageSources>
</configuration>
```

再添加必要的 Nuget 包引用：

```xml
<PackageReference Include="Grpc.AspNetCore" Version="2.27.0-dev202001100801" />
<PackageReference Include="Grpc.AspNetCore.Web" Version="2.27.0-dev202001100801" />
```

接着，修改原来的 `WeatherForecastController` 改为 `WeatherForecastService`:

```csharp
    public class WeatherForecastsService : WeatherForecasts.WeatherForecastsBase
    {
        private static readonly string[] Summaries = {
            "Freezing", "Bracing", "Chilly", "Cool", "Mild", "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
        };

        public override Task<WeatherReply> GetWeather(Empty request, ServerCallContext context)
        {
            var reply = new WeatherReply();

            var rng = new Random();
            reply.Forecasts.Add(Enumerable.Range(1, 5).Select(index =>
            {
                var temperatureC = rng.Next(-20, 55);
                return new WeatherForecast
                {
                    DateTimeStamp = Timestamp.FromDateTime(DateTime.UtcNow.AddDays(index)),
                    TemperatureC = temperatureC,
                    TemperatureF = 32 + (int)(temperatureC / 0.5556),
                    Summary = Summaries[rng.Next(Summaries.Length)]
                };
            }));

            return Task.FromResult(reply);
        }
    }
```

现在，在你的服务器的 `Startup.cs` 文件中，修改 `ConfigureServices` 以添加以下行:

```csharp
  services.AddGrpc();
```

_注意:如果你只打算公开 gRPC 服务，你可能不再需要 MVC 控制器，在这种情况下，你可以从下面删除`services.AddMvc()` 和 `endpoints.MapDefaultControllerRoute()`。_

只是在 `app.AddRouting();` 的下面添加以下内容，它会处理将传入的 gRPC-web 请求映射到服务端，使其看起来像 gRPC 请求:

```csharp
  app.UseGrpcWeb();
```

最后，在`app.UseEndpoints`语句块中注册你的 gRPC-Web 服务类，并在该语句块的顶部使用以下代码行:

```csharp
  endpoints.MapGrpcService<WeatherForecastsService>().EnableGrpcWeb();
```

就这样，你的 gRPC-Web 服务端已经准备好了!

## 修改 Angular 项目

项目的 `ClientApp` 目录中，就是 Angular 的项目文件，使用 ASP.NET Core 托管，是这个项目模板约定的。

我们需要先通过 `proto` 生成需要的 js 文件。

### 安装 npm 包

运行命令：

```bash
 npm i protoc google-protobuf ts-protoc-gen @improbable-eng/grpc-web -s
```

### 生成 js 文件

在 ClientApp 目录下创建目录 `proto` ，再运行命令：

```bash
./node_modules/protoc/protoc/bin/protoc --plugin="protoc-gen-ts=.\node_modules\.bin\protoc-gen-ts.cmd" --js_out="import_style=commonjs,binary:./../Protos" --ts_out="service=grpc-web:src/app/proto" -I ./../Protos ../Protos/*.proto
```

可以在`proto`目录中看到生成了 4 个文件（每个 proto 会生成 4 个）

-   `weather_pb_service.js`: 包含了 rpc 调用客户端 `WeatherForecastsClient`。
-   `weather_pb.js`: 包含了传输对象 `WeatherForecast`。这个文件在原 proto 的目录下，需要手动移过来。
-   两个 `*.d.ts` 文件是对应以上两个文件的类型描述，用于 TS。

![](/images/aspnetcore-grpc-web/2.png)

### 修改 fetch-data 页面组件

接下来，需要引用生成的文件，创建一个 `WeatherForecastsClient` 来调用 gRPC-Web 服务端。

-   fetch-data.component.ts

    ```ts
    import { Component } from '@angular/core';
    import { WeatherForecast } from '../proto/weather_pb';
    import { WeatherForecastsClient } from '../proto/weather_pb_service';
    import { Empty } from 'google-protobuf/google/protobuf/empty_pb';

    @Component({
        selector: 'app-fetch-data',
        templateUrl: './fetch-data.component.html',
    })
    export class FetchDataComponent {
        public forecasts: WeatherForecast[];

        private client: WeatherForecastsClient;
        constructor() {
            this.client = new WeatherForecastsClient('https://localhost:5001');
            this.client.getWeather(new Empty(), (error, reply) => {
                if (error) {
                    console.error(error);
                }

                this.forecasts = reply.getForecastsList();
            });
        }
    }
    ```

-   fetch-data.component.html

    ```html
    <h1 id="tableLabel">Weather forecast</h1>

    <p>This component demonstrates fetching data from the server.</p>

    <p *ngIf="!forecasts"><em>Loading...</em></p>

    <table class="table table-striped" aria-labelledby="tableLabel" *ngIf="forecasts">
        <thead>
            <tr>
                <th>Date</th>
                <th>Temp. (C)</th>
                <th>Temp. (F)</th>
                <th>Summary</th>
            </tr>
        </thead>
        <tbody>
            <tr *ngFor="let forecast of forecasts">
                <td>{{ forecast.getDatetimestamp().toDate() }}</td>
                <td>{{ forecast.getTemperaturec() }}</td>
                <td>{{ forecast.getTemperaturef() }}</td>
                <td>{{ forecast.getSummary() }}</td>
            </tr>
        </tbody>
    </table>
    ```

可以看到：

-   创建 `WeatherForecastsClient` 对象需要传入服务端的 HostName，要注意不要用 `/`后缀。
-   生成出来的 `WeatherForecast` 类型包含`getter/setter`, 而 `WeatherForecast.AsObject` 类才是值对象，可以直接访问属性值，需要调用 `.toObject()` 方法进行转换。
-   `datetimestamp` 属性的类型 `proto.google.protobuf.Timestamp` 是 protobuf 里的关键字，调用 `toDate()` 方法可转换为 TS 的 `Date` 类型。

## 运行项目

完事具备，我们可以运行项目了。访问 `https://localhost:5001/fetch-data`，就可以看到前端是通过 gRPC-Web 获取数据了。

![](/images/aspnetcore-grpc-web/3.png)

# 总结

可以看出，要在前端 JS/TS 使用 gRPC-Web 虽然在开发工具上没有 Blazor WebAssembly 方便，但是从 `proto` 生成客户端之后，前端的 TS 代码就能直接获得强类型的调用方法和路由，很简单地得到 gRPC 带来的好处。另外，gRPC-Web 项目本身已经 GA，所以我们可以先在后端使用它的 gRPC 代理，而前端可以放心大胆地在我们的生产项目中使用它。等到 ASP.NET Core 正式支持 gRPC-Web 后，就可以不需要代理了，比其他平台和语言都更有优势。

本示例的源码已发布到 Github：https://github.com/ElderJames/AspNetCoreGrpcWeb
