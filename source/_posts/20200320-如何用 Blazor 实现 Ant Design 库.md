---
layout: post
title: 如何用 Blazor 实现 Ant Design 组件库？
permalink: how-to-implement-ant-design-components-using-blazor
date: 2020-03-21 13:54:10
tags:
    - Blazor
    - AntDesignBlazor
categories:
    - .NET Core
    - Blazor
---

本文主要分享我创建 Ant Design of Blazor 项目的心路历程，已经文末有一个 Blazor 线上分享预告。

# Blazor WebAssembly 来了！

Blazor 这个新推出的前端 Web 框架，想必是去年 .NET Core 3.0 发布时才进入 .NET 开发者的视线的。但其实，那时发布的是服务端托管版，而真正具有跨时代意义的，是即将在今年5月发布的 WebAssembly 托管版。

我早在两年前，2017年底，就已经在 Steve Sanderson 的[一篇博客](https://blog.stevensanderson.com/2017/11/05/blazor-on-mono/)中看到这个能完全运行在浏览器的前端框架。那时我还是个只会 vue 和一点 Angular 的年轻人，看到只用 C# 就能写一样的单页应用，哪能经得住这种激动。那时就很期待能快点正式发布。后来真的被纳入 AspNetCore 项目的一部分，不断发展完善。没想到最后会是服务端托管版本先发布，而一直发展了接近3年的 WebAssembly 版本，也终于将在今年5月正式发布！

发展至今，Blazor 已经从通过 SignalR 实现双向绑定的服务端托管模式、到通过 WebAssembly 在浏览器上运行 .NET 运行时，再到现在还实现了 PWA、移动端原生控件绑定等技术，Blazor 在未正式发布时就已经有如此多的功能，可见 ASP.NET 团队和整个社区在 Blazor 上付出了有多大的精力、诚意与期待。ASP.NET 团队还有一系列的开发计划，今年底预计还会发布基于原生UI渲染系统的预览版本。

也就是说，Blazor 目前还是前端 Web 框架，但是等到了明年，就会成为一个跨设备平台的客户端框架了！

# Blazor 现状

那么，现在我们就能马上用起来了吗？

能，也不能，这取决于你想怎么用。如果像官方给出的开发模板那样，使用 Bootstrap 作为样式库，而自己实现所有交互效果的话，完全可以用起来了。但是，如果希望有更具交互性更美观的组件库，能拿来就用，快速实现 UI 需求的话，能选择的组件库还不多，在 Awesome Blazor 列表中，完成度较高的开源的组件库，最好的就有 Material Design 的实现 MatBlazor，其他的基于Bootstrap 的，大多是靠JS实现交互，Blaozr只是渲染最基本的一个Html标签。再者还有收费的 Telerik UI、Radzen.Blazor 等等……国内还有好几个原来做 WPF、MVC组件的厂商也开始了Blazor组件的贩卖……

这种形势，就等于是手上只有一个 angluar-cli，是能创建空白的组件，但离搭建出高水准的UI组件就还差十万八千里了。那么，该让谁来填补组件库的空白呢？我们参考一下已有的前端框架吧。

# 为什么想做 Blazor 组件库

目前我们用在项目中的前端框架组件库，能数出名的 Ant Design、ElementUI、iView、Vuetify 莫不是依靠开源社区的力量打造出来的。所以，Blazor 的组件库也需要靠开源社区贡献！

那我们怎么做 Blazor 的组件库呢？

现代的前端组件库大体上可以分两个部分，一是设计语言与理论，二是具体框架实现。设计语言与理论更多地是 UI/UX 方面的设计。对我来说，最喜欢的设计以此是微软的 Fluent Design、谷歌的 Material Design、再到蚂蚁的 Ant Design。而具体框架实现就是由 React、Angular、Vue 等框架的实现了。

经过几年来的实践，还有对开源社区的观察，分别用过 Bootstrap、iView、Material、PrimeNG 等组件库之后，最终还是觉得 Ant Design 在 UI 设计、组件功能和实用性、项目工程、社区活跃度等方面总体上都做得很好。作为一个开源项目， 2019 年 7 月，Ant Design的 Github star 数超过 Material UI，成为全球 star 数最高的 React 组件库项目。

还翻到篇文章，蚂蚁UED接受采访时说：

>选择 Design 这个域名，是期待我们能将设计价值观、原则和模式逐步传承下去，因为前端框架、设计风格都会过时，antd （Ant Design of React：用 React 前端框架实现的 Ant Design）一定会被淘汰。但如果我们有 Ant Design ，下个潮流来的时候，我们能快速搭建新的 Ant Design of XXX。非常感动的是，除了 antd，我们已经有 10+ 不同前端框架实现的 Ant Design。 我们的 Design 梦得到了真正的延续。<p align="right"> —— 蚂蚁UED</p>

于是一语成谶，WebAssembly 这一潮流来了，Ant Design of Blazor 应运而生。

# 我是如何做 Ant Design of Blazor

## 与 Ant Design 官方一致

Ant Design of Blazor （以下简称 AntBlazor）从一开始我就定位在与官方 React 实现的代码仓库同规模的项目，并参考它的其他实现，如 NG-ZORRO、ant-design-vue 是如何做另一个框架的实现的。

- 参考 ant-design-vue 的命名，我把 Blazor 仓库名改为 ant-design-blazor。
- 参考 NG-ZORRO 的 logo 改编，我把 Blazor 官网的 logo 抠下来，也把 Ant Design 的 logo 挖空并旋转 -45°，把 Blazor logo 填上并修改颜色。
- 参考 NG-ZORRO 的样式同步机器人，我也自己写了样式同步的 Github Action，定时检查 ant-design 是否有新版本发布，当有新版本时自动提交PR。
- 参考 ant-design 写了 README 和 demo 文档。
- 参考 NG-ZORRO，使用 Angular 的 commit lints。
- 为了项目工程质量，使用 TS 编写互操作脚本，编译脚本都是开源的，对外部贡献者尽量友好，而不是像已有的几个 Blazor 组件库那样只给出混淆后的 js、css。
- 还有一些在文档构建时衍生出的副产品，如约定式路由、组件渲染服务等。

在为了与官方高度一致上的努力，还会继续。希望有一天能在丰富 Blazor 生态的同时，还能成为被 Ant Design 生态认可的框架实现，能成为他们 Design 梦的一个延续。

## 发起 Blazor 中文社区

对与社区来说，Blazor 还是新鲜事物，就算发布半年了，真正有在项目中运用的还是极少数。因此我建了个微信群，希望对 Blazor 真有兴趣的朋友加入，为了发广告的就免了。想加入，可找张队帮忙邀请，他加的人多😂。

当然，社区运营不是拉个群就完成的，还需要定期地举办线上线下的分享，还有邀请不同领域的大佬一起交流。目前来自前端圈的于航老师和敖天羽大大也被邀请进来了。

## 制定开发路线

在今年年中，我们打算能发布第一个版本 0.1.0，其中完成以下开发项：

- 基本实现 Ant Design 组件
- 组件的独立发布脚本
- 完整文档的构建脚本

如果进展顺利，在年末 .NET 5 发布时，我们能发布生产可用的 1.0 版。

## 寻找贡献者

要实现这些目标，光靠我一个人是不行的，我需要在社区中寻找到其他开发者一起贡献这个项目。在社区中，目前已经为 AntBlazor 项目邀请到几个开发者参与了代码贡献，并希望有更多对 Ant Design 和 Blazor 有兴趣的开源爱好者加入。特别是希望能有企业用户在有需求的情况下慷慨贡献。

我们欢迎感兴趣的前端开发者加入，Blazor 是一个完完全全的前端框架，我们在做的也是前端组件库，是十分需要与感激有前端开发者加入到我们，推动 Blazor 在前端的发展。

我能做的是完善开发流程和引入一些好用的工具，尽量让社区贡献更方便。

## 开展技术分享

考虑到很多朋友还不太了解 Blazor，因此我会在社区中不定期举办分享和培训。会找 Microsoft Reactor 的老师帮忙安排场地，也会在线上举办直播会议，给大家分享一下 Blazor 与参与开源项目的知识，让他们能更好地入手给 AntBlazor 提交代码。

在本周日（即3月21日）下午，我将在钉钉发起直播会议，讲解 Blazor 的官方文档以及 ant-design-blazor 的项目结构以及示范写 Ant Design 组件。钉钉群入口就是ant-design-blazor 项目README最下面的二维码，大家扫码之前记得点个 Star！


项目地址: https://github.com/ElderJames/ant-design-blazor


---

参考：

- [专访蚂蚁金服体验技术UED：Ant Design希望成为世界级设计体系](https://www.zcool.com.cn/article/ZOTU0NjA0.html)
- [https://zhuanlan.zhihu.com/p/110863773](https://zhuanlan.zhihu.com/p/110863773)
