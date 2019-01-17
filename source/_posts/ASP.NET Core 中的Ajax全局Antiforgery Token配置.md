---
layout: post
title: ASP.NET Core 中的Ajax全局Antiforgery Token配置
permalink: The-Ajax-global-Antiforgery-Token-configuration-in-ASP.NET-Core
date: 2018-09-11 18:24:34
tags:
- ASP.NET Core
- .NET Core
- JQuery
categories:
- .NET Core
---



# 前言

本文基于官方文档 [《在 ASP.NET Core 防止跨站点请求伪造 (XSRF/CSRF) 攻击》](https://docs.microsoft.com/zh-cn/aspnet/core/security/anti-request-forgery?view=aspnetcore-2.1)扩展另一种全局配置Antiforgery方法，适用于使用ASP.NET Core Razor + JQuery Ajax的项目，喜欢玩前后端分离的同学可以酌情参考，但希望不要对XSRF/CSRF掉以轻心，更不要不做处理。

# Antiforgery Token 介绍

跨站点请求伪造(XSRF/CSRF)攻击跟浏览器中登录验证之后保存的Cookie有关，恶意站点通过向攻击目标站点发起非法请求时，浏览器按规则是会带上Cookie信息的，此时被攻击站点就会认为是用户操作行为，如果被利用在修改密码等操作上，对用户的信息安全就会带来威胁。为抵御 CSRF 攻击最常用的方法是使用同步器标记模式(STP)。 而Antiforgery Token（防伪令牌）是ASP.NET Core中的STP实现方案。

STP的防御过程：

1. 服务器发送到客户端的当前用户的标识相关联的令牌。
2. 客户端返回将令牌发送到服务器进行验证。
3. 如果服务器收到与经过身份验证的用户的标识不匹配的令牌，将拒绝请求。

熟悉ASP.NET和ASP.NET Core的同学应该都不陌生，因为在ASP.NET时期就有防止XSRF攻击的方法，ASP.NET MVC中，IHtmlHelper.BeginForm默认情况下生成防伪令牌，而ASP.NET Core中，使用FormTagHelper默认也会生成防伪令牌的。

# 解决方案

## Form表单提交

TagHelper用法：

```html
<form asp-controller="Manage" asp-action="ChangePassword" method="post">
    ...
</form>
```

HtmlHelper 生成Form的用法：
```cs
@using (Html.BeginForm("ChangePassword", "Manage"))
{
    ...
}
```

普通html form表单用法：

```html
<form action="/" method="post">
    @Html.AntiForgeryToken()
</form>
```

那么在Razor渲染之后，表单中就会生成一个隐藏的表单字段：

```html
    <input name="__RequestVerificationToken" type="hidden" value="CfDJ8NrAkS ... s2-m9Yw">
```

## 那么Ajax中要怎么处理呢？

官方文档虽然有提到Ajax的处理方法，但是它指的Ajax是js原生实现`XMLHttpRequest`，而不是我们一般所认识的`JQuery.ajax`。所以本文就要介绍一下在使用`JQuery.ajax`时的全局配置。

### 场景一、从普通表单获取Antiforgery Token

这种方法跟上面提到的Form表单提交一致，只要把所生成的隐藏的表单字段也一并提交到服务器即可。

```js
$.ajax({
    url:"/Manage/ChangePassword",
    type:"post"
    data: { "__RequestVerificationToken":"CfDJ8NrAkS ... s2-m9Yw" }
})
```

但是这种方法有个弊端，就是需要配置的东西很多，又要在Contorller中加`[ValidateAntiForgeryToken]`特性，又要在表单中处理使其生成隐藏字段。

有没有更方便的方法？当然有！而且即使是文档中号称自动防范 XSRF/CSRF的Razor Pages都同样需要！因为它并没有处理Ajax的场景。

### 场景二、全局配置，自动处理

#### 全局获取Forgery Token

全局（每个页面）获取Forgery Token就是文档中提到的注入`Microsoft.AspNetCore.Antiforgery.IAntiforgery`并调用`GetAndStoreTokens`方法，但是由于需要达到全局获取，我需要把这个方法的调用写到布局页,如默认MVC模版的`Views/Shared/_Layout.cshtml`

```aspnet
@inject Microsoft.AspNetCore.Antiforgery.IAntiforgery Xsrf
@functions{
    public string GetAntiXsrfRequestToken()
    {
        return Xsrf.GetAndStoreTokens(Context).RequestToken;
    }
}

<script>
    var csrfToken = '@GetAntiXsrfRequestToken()';
</script>
```

#### Ajax全局配置

`JQuery.ajax`里全局设置头部的方法是`$.ajaxSetup`，按照文档，把所需的头部字段`RequestVerificationToken`配置上上面获取到的令牌变量`csrfToken`，即可实现在每个Ajax请求都带有Forgery Token。

```js
(function (window, document, $) {
    $.ajaxSetup({
        headers: {
            'RequestVerificationToken': csrfToken
        }
    });
})(window, document, jQuery);
```

# 总结

虽然现在流行前后端分离了，包括我在内，也用上高大上的React、Angular、Vue等优秀框架，Github前端也把JQuery去掉了，但是Razor在ASP.NET Core中的份量有增无减，2.0版本带来了更轻量的Razor Pages, 因此Razor+JQuery的热度不会那么快退去，希望这篇文章能给大家在Razor+JQuery技术的使用过程中带来一点参考价值。