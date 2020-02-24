---
layout: post
title: 【译】来看看WebWindow，一个跨平台的.NET Core webview 库
permalink: webwindow-a-cross-platform-webview-for-dotnet-core
date: 2020-02-23 23:54:10
tags:
    - Blazor
    - .NET Core
categories:
    - .NET Core
---

_本文翻译自 ASP.NET 项目组的 Steve Sanderson 的博客，发表于 2019年11月18日。Steve Sanderson 是 Blazor 最早的创造者。_

---

它类似于 Electron，但没有捆绑 Node.js 和 Chromium，也没有大部分 API。

我的上一篇文章研究了如何用 web 渲染的 UI 构建一个 .NET Core 桌面或控制台应用程序，而不需要引入全部的 Electron 部件。这似乎引起了很多人的兴趣，所以我决定将其升级到新的技术并添加跨平台支持。

最终成果是一个名为 [WebWindow](https://www.nuget.org/packages/WebWindow) 的小 NuGet 包，你可以将它添加到任何 .NET Core 控制台应用程序中。它可以打开一个包含基于 web 的 UI 的本机操作系统窗口（Windows/Mac/Linux），而无需您的应用程序绑定 Node 或 Chromium。

我还把它和 Blazor 解耦了。您现在可以在窗口内托管任何类型的 web UI。repo 中包含一个使用Vue 的示例，还有一个使用了 Blazor。

> ***注意：*** *这个库是超前 alpha 质量的。如果你想用它来构建一些真实的东西，请看这篇文章末尾的注释。到目前为止，这只是另一个原型。*

# “Hello World” 示例

创建一个新的 .NET Core 3 C# 控制台应用程序，然后添加一个引用到 `WebWindow` NuGet包:

```xml
<ItemGroup>
  <PackageReference Include="WebWindow" Version="0.1.0-20191120.3" />
</ItemGroup>
```
接下来，将代码添加到 `Program`  类中的 `Main` 方法。

```cs
static void Main(string[] args)
{
    var window = new WebWindow("My super app");
    window.NavigateToString("<h1>Hello, world!</h1> This window is from a .NET Core app.");
    window.WaitForExit();
}
```

这样就完成了！现在根据你运行的操作系统，你的应用程序会显示如下窗口：

![](https://blog.stevensanderson.com/wp-content/uploads/2019/11/18/hello-windows.png)

![](https://blog.stevensanderson.com/wp-content/uploads/2019/11/18/hello-macos.png)

![](https://blog.stevensanderson.com/wp-content/uploads/2019/11/18/hello-ubuntu.png)

这个示例使用 `NavigateToString(html)` 从硬编码的 .NET 字符串渲染 HTML。你也可以：

- 使用 `NavigateToUrl(url)` 来显示来自HTTP服务器的内容(本地或远程)
- 使用 `NavigateToLocalFile(path)` 来显示来自本地磁盘的 HTML 文件，其中 `path` 是绝对路径或相对于当前工作目录的路径。[示例在这里](https://github.com/SteveSandersonMS/WebWindow/blob/a01537a9328b085075866a965191d6323ad2cf7d/samples/HelloWorldApp/Program.cs#L11)

作为一个稍微高级一些的选项，您可以配置 `WebWindow` 来处理自定义协议，如 `myapp://`，并指定一个委托（回调），它可以为该协议中的每个 URL 返回任意内容。示例在[这里](https://github.com/SteveSandersonMS/WebWindow/blob/a01537a9328b085075866a965191d6323ad2cf7d/testassets/HelloWorldApp/Program.cs#L14)和[这里](https://github.com/SteveSandersonMS/WebWindow/blob/a01537a9328b085075866a965191d6323ad2cf7d/testassets/HelloWorldApp/wwwroot/index.html#L11)。

一旦您的 web 内容正在运行，JavaScript 和 .NET 之间的底层通信方式就是在 JS 中使用 `window.external.sendMessage/receiveMessage` API（[示例](https://github.com/SteveSandersonMS/WebWindow/blob/a01537a9328b085075866a965191d6323ad2cf7d/testassets/HelloWorldApp/wwwroot/index.html#L14-L20)）和在.NET中使用 `webWindowInstance.SendMessage` 和 `webWindowInstance.OnWebMessageReceived` API -（[示例](https://github.com/SteveSandersonMS/WebWindow/blob/a01537a9328b085075866a965191d6323ad2cf7d/testassets/HelloWorldApp/Program.cs#L21-L24)）。但是，如果您正在构建一个 Blazor 应用程序，则不需要使用这些底层 API，而可以使用 Blazor 常规的 JS 互操作特性。

# 托管一个 Blazor 应用程序

WebWindow 并没有与 Blazor 耦合。这里是一个[使用 Vue.js 在 WebWindow 内呈现一个简单的目录资源管理器应用程序的例子](https://github.com/SteveSandersonMS/WebWindow/tree/master/samples/VueFileExplorer)。

但如果你真的想用 Blazor，那是非常整洁和简单的。我还做了一个小插件包，[WebWindow.Blazor](https://www.nuget.org/packages/WebWindow.Blazor)，它可以让你在你的 `Program.Main` 中只用一行代码就能运行一个 Blazor 应用程序。

```
static void Main(string[] args)
{
    ComponentsDesktop.Run<Startup>("My Blazor App", "wwwroot/index.html");
}
```

概括地说，这 _并不_ 涉及 WebAssembly、Node.js 或者是一份私有绑定的 Chromium。它只是在本地运行 .NET Core，直接与操作系统自己的 web 渲染技术进行通信。这一次在 macOS 上的运行结果是:

![](https://blog.stevensanderson.com/wp-content/uploads/2019/11/18/blazor-macos.jpg)

完整的 [WebWindow+Blazor 示例在这里](https://github.com/SteveSandersonMS/WebWindow/tree/master/samples/BlazorDesktopApp)。

# 他是如何工作的

- 在 **Windows** 上，WebWindow [通过 `webview2` 使用新的基于 Chromium 的 Edge](https://docs.microsoft.com/en-us/microsoft-edge/hosting/webview2)，假设你已经安装了那个浏览器（如果你不安装，它可能会退回到旧的Edge，但我还没有实现）
- 在 **Mac** 上，它使用了操作系统内置的[WKWebView](https://developer.apple.com/documentation/webkit/wkwebview)，这与 Safari 背后的技术相同
- 在 **Linux** 上，它使用了 [WebKitGTK+2](https://webkitgtk.org/)，这也是一种基于 WebKit 的技术

这一切的关键在于，与使用 Electron 相比，开发出下载体积更小和内存使用更少的应用程序。但事实果真如此吗？以下是下载大小的统计数据：

![](https://blog.stevensanderson.com/wp-content/uploads/2019/11/18/download-size-chart.png)

正如您所看到的，无论您选择“独立”（捆绑了.NET Core 运行时的一个副本）还是“依赖于框架”（依赖于已安装在目标系统中的 .NET Core），都会对最终的应用程序大小产生巨大的影响。依赖于框架的 WebWindow 应用程序真的可以很小，因为它们只包含您自己的应用程序的二进制文件，并且没有捆绑运行时或浏览器。

接下来，是内存使用统计:

![](https://blog.stevensanderson.com/wp-content/uploads/2019/11/18/memory-use-chart.png)

在 Windows 上，WebWindow 和 Electron 使用的是相同的浏览器技术（Chromium），这种技术占用了大部分内存。这就解释了为什么他们之间的差异并不是很大。在 Linux 和 Mac 上，使用自绑定浏览器（指 Electron）与使用操作系统内置技术之间的区别更为明显。

# 这个项目会得到支持和维护吗？

目前我还没有做出任何承诺！现在最好把它看作是另一个实验。如果有足够多的人想参与进来，就有可能创建一个合适的开源社区项目。

最迫切需要的是有 C++ 经验的人来重写我的原型质量的 C++ 和 Objective-C 代码，使其真正完善。我已完成所有内存管理的概率接近于0。也许它也应该使用 CMake 或其他健全的构建配置系统。（注意：它确实有一个[基于 Azure DevOps 的跨平台 CI](https://dev.azure.com/SteveSandersonMS/WebWindow/_build?definitionId=2)。）

如果您打算在生产中使用它，那么您还需要添加大量的特性。例如，设置应用程序图标、添加本地菜单栏等功能。如果您有兴趣贡献这样的功能，并将使其跨平台工作，请[来这个 repo](https://github.com/SteveSandersonMS/WebWindow)！