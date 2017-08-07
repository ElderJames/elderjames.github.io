---
title: 给你的域名加把锁：免费Https申请
date: 2017-08-02 15:58:36
permalink: how-to-request-a-free-https
tags: Github Page,Https,SSL,Hexo
categories: Github Page,HTTPS
description: 企业级ssl证书费用高昂，如果只是个人网站没必要用企业级的，使用一些免费的就好了。接下来，我就以我的个人域名为例，介绍下申请免费https的流程。
thumbnail: https://img.alicdn.com/tps/TB1sfvIMXXXXXcgXXXXXXXXXXXX-2800-1100.jpg
---

关于Https的介绍，先引用一段维基百科的描述：

> 超文本传输安全协议（英语：Hypertext Transfer Protocol Secure，缩写：HTTPS，常称为HTTP over TLS，HTTP over SSL或HTTP Secure）是一种通过计算机网络进行安全通信的传输协议。HTTPS经由HTTP进行通信，但利用SSL/TLS来加密数据包。HTTPS开发的主要目的，是提供对网站服务器的身份认证，保护交换数据的隐私与完整性。这个协议由网景公司（Netscape）在1994年首次提出，随后扩展到互联网上。
历史上，HTTPS连接经常用于万维网上的交易支付和企业信息系统中敏感信息的传输。在2000年代晚期和2010年代早期，HTTPS开始广泛使用于保护所有类型网站上的网页真实性，保护账户和保持用户通信，身份和网络浏览的私密性。

除了平常我们看不见的安全性外，拥有Https前缀的域名被浏览器加了把锁，看起来就比没有的牛~而且，苹果在去年已经规定所有应用内链接都需要有https了！

企业级ssl证书费用高昂，如果只是个人网站没必要用企业级的，使用一些免费的就好了。接下来，我就以我的个人域名为例，介绍下申请免费https的流程。

此处炫耀一下：  ![](/images/15.png)

#### 提前准备

1. 一个自己的域名，可以在国内的域名注册商购买注册，如阿里云、腾讯云，然后最好是做好备案，因为微信等一些客户端的链接分享都要求备案过的域名才能正常访问。当然，不备案也不妨碍绑定Github Pages和国外服务器。

#### 教程目标

1. 给自己的域名设置 Flexible SSL 认证，即网站到解析服务器间的加密传输，让域名可以使用https
2. 访问http域名，自动301跳转到https
3. 访问http://www.yourdomain.com ,会自动跳转到https://yourdomain.com（当然，可以反过来）

#### 步骤

1. 注册CloudFlare帐户，并添加域名解析

    访问[CloudFlare注册页面](https://www.cloudflare.com/a/sign-up),简单地输入邮箱和密码就能注册成功好。

2. 添加域名

    登录后，访问[Add Websites页面](https://www.cloudflare.com/a/add-site)添加一个域名，如我的是yangshunjie.com，然后点击【Begin Scan】扫描。大概需要等待1分钟，完成后点击【Continue Setup】继续下一步。

    ![](/images/6.png)

3. 设置解析项

    这时，我们需要添加域名解析了，将域名解析到指向的服务器。这跟普通的域名解析绑定是一样的。如果要绑定Github Pages，就需要参考[Github Help官方指南](https://help.github.com/articles/setting-up-an-apex-domain/#configuring-a-records-with-your-dns-provider)添加A记录分别指向`192.30.252.153`和`192.30.252.154`。另外再设置一个www的CNAME记录，指向yangshunjie.com。

    ![](/images/7.png)

4. 修改域名解析服务器

    登录你的域名注册商的控制后台，把DNS解析服务器改为CloudFlare提供的两个地址。然后等待转移完成。我的域名是在阿里云注册的，所以这里截图演示一下：
    
    ![](/images/8.png)
    ![](/images/9.png)

    > 注：官方说明，域名服务器修改最长需要72小时生效 ，用了两个域名测试，大约需要 5~30 分钟，看到 Status: Active 即可。

    ![](/images/10.png)

5. 回到CloudFlare，点击上部的 crypto 菜单 , 然后设置 Flexible SSL。

    ![](/images/11.png)

6. 接着点击 Page Rules 菜单，添加两个规则，一个是将`http://yangshunjie.com/*`设为 Always Use HTTPS,一个是讲`http://www.yangshunjie.com/`重定向到`https://yangshunjie.com/`(如果需要默认跳转到www开头的域名，则这里就重定向到`https://www.yangshunjie.com/`即可)。

    ![](/images/12.png)
    ![](/images/13.png)
    ![](/images/14.png)

#### 注意事项

1. 以后你需要绑定其它服务器，都需要在这里配置。DNS记录视缓存的时间一般需要5~30分钟生效。

