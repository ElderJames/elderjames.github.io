---
layout: post
title: 如何用 Blazor 实现 Ant Design 组件库（二）
permalink: how-to-implement-ant-design-components-using-blazor-2
date: 2020-04-05 13:54:10
tags:
    - Blazor
    - AntDesignBlazor
categories:
    - .NET Core
    - Blazor
---

# 前言

前两周，我发表了上一篇文章《如何用 Blazor 实现 Ant Design 组件库？》，得到了很多朋友的响应，也有很多朋友加入我的钉钉群，并收听了我在第二天的直播。

这次直播是我人生第一次做直播，以至于没做什么准备，主要介绍了 AntBlazor 项目的情况，和介绍了 Blazor 框架的一些知识点，其中穿插了我之前用 Blazor 实现二维码多端登录。结果没有重点地一讲讲了4个小时。

这次直播我有录播，地址在：https://www.bilibili.com/video/BV1uE411c7Hc。大家也可以在钉钉群看回放。

两周过去了，在这个清明假期，我准备在假期的最后一天再来直播，但不会讲那么久了，免得占用大家太多的时间。同样在钉钉群和B站同时直播，并开启录播。

- 时间：4月6日 下午2:00 - 5:00
- 直播间：https://live.bilibili.com/22040564
- 钉钉群：项目 README 底部二维码（推荐）

好了，预告说完了，本篇主要讲一讲关于开发 Blazor 的价值和我从 AntBlazor 项目中获得的不少益处。

项目地址: https://github.com/ElderJames/ant-design-blazor

# Blazor 是否值得关注

Blazor 是一个全新的框架，目前 WebAssembly 的版本还在 Preview 阶段，将于5月19日Build大会期间发布正式。

所以在我推广 AntBlazor 时就有很多人质疑：现在投入精力开发组件库是否值得？会不会跟 Webform 一样只有微软自己玩最后被放弃？是否需要等成熟了再去了解？

首先，大家要了解 Blazor 是一个全能的框架：

- 服务端：使用 WebSocket 实现 dom 的节点渲染
- 浏览器端：利用 WebAssembly 托管.NET Runtime，运行完整的 SPA 应用
- PWA：支持离线运行
- 移动端：基于Xamarin，实现原生控件的绑定
- 原生应用：.NET 5 正式发布时会有 Blazor 原生应用的预览版本

相对于三大框架的初期版本，Blazor 基于 .NET 生态，已经有很多完备的组件。如路由、依赖注入、状态管理、国际化/本地化、权限控制、GC、调试工具、模板等……这些组件在已有前端框架中，哪些是一发布就全都有的？

当然，要是跟现在的前端框架比起来，Blazor 在生态上还是有很大的发展空间，但是我们依然能看到它的巨大潜力。作为一个受欢迎的开源项目，自然会吸引到开源社区的开发者一起来丰富生态。而且相对于后端，前端的生态发展应该是更容易的。

在这种情况下，我们除了等待官方开发组开发新特性之外，围绕已有的核心组件，还是可以做很多生态上的补充的。

AntBlazor 就是致力于丰富 Blazor 生态的众多开源项目之一。

# 开发 Blazor 项目的价值

AntBlazor 是一个让大家在开源实践的同时学习 Blazor 以及其他技能的项目，它在 Blazor WebAssembly 3.1 preview 阶段被创建，有一段时间是实验性地开发组件和基于 WebAssembly 的文档项目。而在 3.2 preview 1 之后开始伴随着每个 preview 版本，都会在项目中尽量把最新特性都用上。

- 从最开始的普通类库项目转换成 Razor Class Library，因发布文件的目录结构变更修改 Github Actions 的 CI 脚本
- 加入 PWA 特性支持，我们的文档页面可以安装成 PWA。
- 调整 Debugging 支持，现在文档的 WebAssembly 项目也支持断点调试了。

除了官方发布的新特性，我自己也在创造性地让项目更具“前端气息”，利用 Blazor 做更多有用的实践。

## 改造路由

路由组件是最基础的组件，官方的路由实现了最基本的事情——路由表和导航，而为了让它更易于使用，我一直在改进它。

- 既支持原来用特性指定的路由，也支持用目录的路径作为路由的约定方式
- 支持 Url QueryString 的属性绑定
- 支持默认终结点
- 支持切换语言，实现本地化

路由的改造是在分析过源码后才实现的，这部分将来会统一以源码分析专题介绍给大家。

## 添加多语言

目前版本还没官方的全球化/本地化支持，我自己就先实现了一个多语言服务，动态地切换当前使用语言。

这个多语言服务值得注意的地方：

- 无需强制刷新页面，只需导航一下路由就能切换
- 用一个事件来触发需要刷新状态的位置，使组件重新渲染不同语言的字符串
- 使用 yaml 来配置资源文件

在这个期间，更深入地学习到了状态刷新的要点。在下次直播中会分享给大家。


## 文档构建的改进

本项目用 Blazor WebAssembly 来构建静态文档站点，为了让源码目录兼顾 .NET 项目与开放性，源文件会放置在项目目录外，在构建时会用到 MSBuild Tasks 来做一些文件拷贝和调用 Node.js 的步骤。Node.js 主要用来调用 gulp 来构建 less 文件和 typescript 文件。但是开发这个项目需要安装 Node.js 还是会让人难受的。所以我一直在寻找可用 C# 替代的方案。

我最近就在用 .NET Core 写一个 CLI，用来从分散在各个组件目录中的文件生成文档所需的元数据，比如菜单和组件示例页面的文档与源码。这部分在 Ant Design 三大框架的版本中，都是用 Node.js 运行 js 脚本实现的。而我还是想用 .NET 来实现构建过程。目前已经取得不错的进展，已经在 PR #64 进行中了。后期我会再继续寻找用 .NET 构建 less 和 typescript 的方法，集成进这个 CLI 项目中，这样我们在开发时也不需要用到 Node 了。

## CI 的不断改进

为了做更开放的开源项目，吸引更多开发者加入到项目中，让大家容易入手，并把关注点集中在组件开发上，我在优化开发流程上做了很多努力，特别是在 CI 上花了好多功夫。在维护 AntBlazor 的这段时间，也让我获得了很多 Github Actions 的使用经验和一些 shell 脚本的应用技巧：

- **ant-design 仓库的样式同步：**

    ant-design 的样式一直在改进，所以对于我们这个从最开始就定位为一个 Ant Design 可持续发展的 Blazor 实现，样式同步并不断改进功能是很有必要的。
    
    我于是写了一个定时触发的 shell 脚本，克隆 ant-design 仓库，并把 components 目录下的所有 less 文件复制到 ant-design-blazor 项目，最后把变更以 PR 方式提交到仓库中。

- **PR 预览页面 :**

    我看到 Ant Design 官方的文章中介绍了他们的CI，我也觉得里面的 PR 预览很有意思，于是我也做了一个类似的功能。不同点是，他们用了Azure Pipelines，而我依然只用 Github Actions。
    
    在 Github Actions 中，Fork 的仓库没有对原仓库进行写操作的权限，导致 PR 脚本中机器人的 token 不能创建issue 评论来显示浏览页面的链接。于是我就用手动出发的方式，先发一个 `/preview` 的评论，触发脚本运行预览页面的部署和链接显示。
    
    后期考虑写一个 Github Bot，这样就能给他授权写操作权限了。

- **Gitee 同步与 Gitee Pages 刷新**

    如上篇文章提到，由于网络原因，国内用户访问 Github 和 Github Pages 都会有不同程度的困难。于是我在Gitee 创建了一个镜像仓库，并且也开通了 Gitee Pages 的服务。

    一开始每次推送代码到 Github 之后，我都要去 Gitee 点一下强制同步，再点一下更新 Gitee Pages，好辛苦。 Gitee Pages 的刷新还需要 99 元的 Pro 版本才支持，对于个人开源项目来说，是极为不友好的。于是我各处搜索，终于找到了 Gitee 同步和 Gitee Pages 刷新的 Github Action。虽然 Gitee Pages 刷新是需要获取 Cookie 的（连 SSH 都不能操作），但是总比手动进去点刷新强很多。具体的脚本大家可以到项目中参考，我认为这个对于使用 Github Pages 做博客或者的项目文档的开发者来说，是非常值得运用的。


# 总结：学习-反哺是一个良性循环

可以看出，Blazor 让开源的 .NET 点亮了前端技能，从此用 .NET 就能做更多前端的事情了。

以上的这些实践，都让我们学习到了更多，有的甚至是不只作用于 Blazor 的更通用的技能；而每一项有意义的改进，又都可以成为丰富 Blazor 生态的重要元素。

其实不是只在 AntBlazor 项目中，在其他需要我们学习的技能上，创造性的实践通常都会让我们得到更多的进步。

欢迎关注 Blazor，欢迎参与 AntBlazor 和加入 Blazor 中文社区。

Github: https://github.com/ElderJames/ant-design-blazor
文档/Demo: https://ant-design-blazor.gitee.io
Blazor文档：https://docs.microsoft.com/zh-cn/aspnet/core/blazor

---

_本项目以 MIT 协议开源，为了能得到够更好的且可持续的发展，我们期望获得更多的支持者，我们将把所得款项用于社区活动和推广。你可以通过文章的 **【赞赏】** 和 **README** 中的捐赠方式支持我们。我们会把详细的捐赠记录登记在捐赠者名单中。_