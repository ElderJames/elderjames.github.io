---
layout: post
title: Asp.Net Core 3.0 Identity 集成 Microsoft Account OAuth
permalink: AspNet-Core-3_0-Identity-For-Microsoft-Account-OAuth
date: 2019-12-06 15:59:00
tags:
    - ASP.NET Core
    - .NET Core
categories:
    - .NET Core
---

最近在给使用 Identity Server 开发的登录中心添加第三方授权登录，集成了微信、钉钉、Github，最后还想到使用Windows 10系统的用户，大多数都有 Microsoft 帐号了，如果集成了 Microsoft 帐号，会带来更多方便！于是就研究了 Microsoft Account OAuth 授权的集成。

使用 Microsoft Account 授权的方式有几种:

- Azure Active Directory
- Microsoft Graph
- Azure WebApp

本示例就用注册简单的 WebApp 实现吧！

### 一、注册 Azure WebApp

- 访问应用注册页面 https://ms.portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredApps

![](/images/msaccount/1.png)

- 选择个人账户中的应用程序选项卡，点击新注册，选择“仅与个人账户关联”，输入名称，点击注册。

![](/images/msaccount/2.png)

- 注册后，看到应用列表中有这个新注册的应用，点击进入设置页面，可以应用程序(客户端) ID。

![](/images/msaccount/3.png)

![](/images/msaccount/4.png)

![](/images/msaccount/5.png)

- 进入证书与密码页面，点击新客户端密码，创建一个客户端密码。

![](/images/msaccount/6.png)

### 二、在ASP.NET Core 应用中注册一个ExternalProvider

- 安装Nuget包

    ```xml
        <PackageReference Include="Microsoft.AspNetCore.Authentication.MicrosoftAccount" Version="3.0.0" />
    ```

- 注册 Microsoft Account OAuth 验证组件，配置ClientId和ClientSecret，以及回调路径

    ```cs
    services.AddAuthentication()
        .AddMicrosoftAccount(options =>
        {
            //options.SignInScheme = IdentityServerConstants.ExternalCookieAuthenticationScheme;
            options.ClientId = Configuration["Microsoft:ClientId"];
            options.ClientSecret = Configuration["Microsoft:ClientSecret"];
            options.CallbackPath = new PathString("/signin-microsoft");
        });
    ```

    PS. 集成 Identity Server 4 之后，可能需要设置 `options.SignInScheme = IdentityServerConstants.ExternalCookieAuthenticationScheme;`

    ![](/images/msaccount/7.png)

### 三、完成！

