---
layout: post
title: 【译】探索更轻量的Electron替代品来托管Blazor桌面应用程序
permalink: exploring-lighter-alternatives-to-electron-for-hosting-a-blazor-desktop-app
date: 2020-02-23 23:54:10
tags:
    - Blazor
    - .NET Core
categories:
    - .NET Core
---

_本文翻译自 ASP.NET 项目组的 Steve Sanderson 的博客，发表于 2019年11月1日。Steve Sanderson 是 Blazor 最早的创造者。这篇文章发布后还有一篇后续，是介绍一个在本文提到的跨平台 webview 概念的落地项目 WebWindow ，我也会接着翻译过来。_

---

我们能否以更少的资源消耗，获得 Electron 的利用 web 技术构建的桌面应用程序的优势?

Electron 在 2014 首次开源，作为一种使用 Web 技术（HTML+CSS+JS）构建桌面应用程序的方式，它迅速流行起来。其设计的核心思想是将可预测的环境捆绑在一起:

- **它捆绑了自己的 Chromium 副本，** 因此，你可以确定你的HTML/CSS将如何被渲染，不必担心 IE浏览器（等） 随机的旧版本。
- **它捆绑了自己的 Node.js 副本，** 因此，你拥有已知的一个可移植编程平台的版本，它超越了浏览器沙箱，可以直接与本机系统交互。

这些选择在五年前很有价值，但到了 2019 年底，你可能会做出不同的选择。这些选择也是为什么 Electron 对资源极度渴求却也会闻名的关键。默认的空白Electron 8.0.0 应用程序需要下载164MB（压缩后66MB），并作为4个单独的进程运行，总共消耗150MB。

这些数字在您看来完全可以满足您的场景。如果是这样，那太好了！这篇文章并不是要抨击 Electron，它是一个运行良好的项目，人们显然已经成功地使用了它。在这篇文章中，我只是想思考一下我们还有什么其他的选择。

# 它能有多轻?

假如...

- **...我们没有绑定 Chromium，** 而是使用了操作系统中已经存在的 webview？到了 2019 年，几乎所有的桌面操作系统都将拥有一个足够现代的、通常基于 Chromium 的浏览器可以拿来即用。对于你的应用程序来说，它是上周的Chromium 还是去年的都不重要。
- **... 我们没有绑定 Node，** 而是使用了操作系统中已经存在的编程环境，或者选择引入一个不同的？一个框架相关的 .NET Core 应用程序的大小可以很容易地在 1MB 以下，而一个独立的 .NET Core 应用程序（即绑定了自己的 .NET Core 副本）经链接和压缩后可以减少到约 20MB。

各种 Electron 的轻量替代项目已经如雨后春笋般涌现出来[1]。当然，也包括 PWA，但这不是这篇文章的目的，因为 PWA 不能进行对底层操作系统的本机访问。

# Electron 上的 Blazor

我们对使用 Blazor 构建跨平台的桌面应用程序很感兴趣。这并不奇怪：将 C#/.NET 的性能和生产率，与熟悉的 HTML/CSS UI 渲染结合在一起是强大的和有吸引力的。

因此，我们发布了[一个示例和一个实验性的包来托管 Electron 上的Blazor](https://github.com/aspnet/AspLabs/tree/master/src/ComponentsElectron)。这里的关键创新之处在于它不是运行在 WebAssembly 上，而是使用普通的跨平台的 .NET Core 运行时来实现完全的原生 .NETb 性能，并在不受任何浏览器沙箱限制的情况下完全访问本机操作系统。

你今天可以试试。但要注意，这只是一个“asplabs”项目，因为我们还没有作出任何承诺，以推出和支持这项技术。
# 纯 webview上的 Blazor

不难想象，一个 Blazor 混合桌面应用程序将如何进一步大幅减小体积。我们可以用一个纯粹的系统原生的 web view 替换掉 Electron，因为在 2019 年，在你的目标机器上总会有一个足够好的来用的。另外，我们并不需要 Node 作为跨平台的编程环境，因为 .NET Core 已经为我们扮演了这样的角色。

为了验证这一点，我构建了 [一项名为 BlazorDesktop 的实验](https://github.com/steveSandersonMS/BlazorDesktop)。这与 [BlazorElectron](https://github.com/aspnet/AspLabs/tree/master/src/ComponentsElectron) 非常相似，实际上大部分代码都是复制粘贴过来的。同样，它运行在原生的 .NET Core 上（所以不是在WebAssembly 上），但是现在它运行在一个更小的渲染堆栈上，没有任何的 Chromium 或 Node.js 绑定。

![](https://blog.stevensanderson.com/wp-content/uploads/2019/11/01/blazor-desktop.png)

这是一个功能齐全的 Blazor 应用程序，你可以在其中添加任何基于 .NET Core 的原生功能，并作为一个非常轻量级的桌面应用程序运行——具体数字见下文。与 PWA 不同，它不局限于浏览器沙箱。

如果您有兴趣尝试一下，请注意这只是一个概念验证，目前仅适用于Windows。将其扩展为跨平台并不难（我将使用诸如 webview 之类的东西来添加 Mac+Linux 支持），但是我现在还没有在积极地做这件事。如果您感兴趣，请提交一份 PR。

# 统计数据

毫不奇怪，这个最小的 Blazor + webview 应用程序比构建在整个Chromium + Node 技术栈上的应用程序要小得多，也不需要太多内存:

![](https://blog.stevensanderson.com/wp-content/uploads/2019/11/01/download-size-chart.png)

![](https://blog.stevensanderson.com/wp-content/uploads/2019/11/01/memory-use-chart.png)

.NET Core 的好处之一是，通过一个简单的开关，您可以控制生成的输出是否绑定了它自己的 .NET Core 运行时副本（也称为独立的），或者它是否假设运行时已经安装在目标OS上（也称为依赖于框架）。在上面的第一个图中可以看到这一点，您可以看到它对输出大小有很大的影响，因为 Blazor 库本身非常紧凑。

在企业环境中，如果您知道已经安装了某些软件，那么可以安全地分发与框架相关的小于 1MB 的应用程序，其中相同的二进制文件可以在任何受支持的操作系统上运行。但是对于公共发行版，您很可能会发布一个独立的应用程序——为每个 Windows、Mac 和 Linux 用户分别生成不同的二进制文件。

请注意，上述压缩的 Blazor webview 应用程序大约有 200KB 的大小，其中还包含了 Bootstrap 样式，因此如果您正在使用其他的则可以删除它。

# 需要做什么才能使其可行

正如我所说的，Blazor Desktop目前只是一个快速的概念验证，完全是在我从 [NDC Sydney](https://ndcsydney.com/) 返回的旅途中开发的。要成为可行的产品还有很长的路要走。

它需要：
- **跨平台支持，** 例如使用 webview 
- **Edge (Chromium) 支持，**因此，在 Windows 10 上，我们使用的是操作系统自己的基于 Chromium 的浏览器，而不是我的原型中使用的基于 Edge 的 webview。
- **处理找不到合适的webview的情况。**在极少数情况下，用户的操作系统没有提供可接受的 webview 技术，我们可以提示用户并下载一个独立的 Chromium 发行版（可能通过CEF）。
- 、**桌面 API** ——一个重大的需求，需要付出高成本地从零开始实现。Electron 不只是使用 Node 作为通用的编程环境;它还提供了一组跨平台的 api，用于与桌面操作系统进行交互，执行诸如复制到剪贴板、更改任务栏或 dock 图标、显示本机下拉菜单、显示本机提示或对话框等任务。.NET Core 本身提供了大量应用程序需要的 API，但是目前它并没有解决很多桌面应用的问题，因为现在还没有任何标准的基于.NET Core 的跨平台 UI 技术。但是任何实际的应用程序都需要这些功能。可以想象，仅仅为了获得这些 API 的跨平台实现而绑定 Node 也是值得的（但仍然不绑定 Chromium）。
- **自动发布和下载更新**

# 反馈要求

我写这篇文章的原因主要是为了更多地了解开发人员社区对这些技术的感受。你有使用 .NET+HTML+CSS 构建混合桌面应用程序的场景吗？你会乐意将 Blazor 与 Electron 一起使用，还是认为有必要更轻量？

# 注脚：现有的 Electron 替代品

多年来，许多人都在考虑制造更轻的 Electron 替代品。现在有各种各样的开源项目，尽管还不清楚是否有关键的真正去大规模采用的动力。其中一些，如 [Chromely](https://github.com/chromelyapps/Chromely)，移除了 Node，只捆绑了 Chromium。其他的，如 [Neutralino](https://github.com/neutralinojs/neutralinojs)，则消除了Chromium，只将基于 Node 的编程模型与本机操作系统的浏览器技术捆绑在一起。在最基本的方面，[webview](https://github.com/zserge/webview) 只是对 webview 概念的一个抽象：它以构建在主机操作系统中的任何浏览器技术来显示 HTML/CSS，并且不提供任何自己的跨平台编程模型。