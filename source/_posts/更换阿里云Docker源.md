---
title: 更换阿里云Docker镜像源
date: 2017-09-12 11:54:17
permalink: change-the-docker-image-origin-to-aliyun
tags: 
- Docker
categories:
- Docker
description: 由于国内网络连接Docker官方源很慢，所以这篇文章介绍如何把Docker源更换为阿里云的。
thumbnail: https://www.docker.com/sites/default/files/group_5622_0.png
---

由于Docker技术比虚拟机技术更为轻便、快捷。自 2013 年 3 月以Apache 2.0授权协议开源后，到近两年成为当下分布式系统最热门的开发运维方式。由于国内网络连接Docker官方源很慢，甚至无法从官方源拉取镜像创建容器，所以这篇文章介绍如何把Docker源更换为阿里云的，使中国开发者也能学习并运用到Docker带来的DevOps变革。

#### 运行环境 (与换源无关)：
- Windows 10
- Docker for Windows [下载](https://download.docker.com/win/stable/Docker%20for%20Windows%20Installer.exe)
- Kitematic [下载](https://github.com/docker/kitematic/releases/download/v0.17.1/Kitematic-0.17.1-Windows.zip)

安装Docker的同时，我们可以先登录阿里云管理平台，通过前台页面进入【容器服务控制台】(https://cs.console.aliyun.com/#/repo)

#### 登录阿里云容器服务控制台

容器控制台的入口不是在一开始登录控制台的页面里的，所以如果以后记不住这个地址，可以按以下页面进入。

1. 指向阿里云首页的导航栏“产品”选项，从下拉的列表中点击“容器服务”

    ![](/images/aliyun-docker/1.png)

2. 从容器服务页面的上宣传图中点击“管理控制台”按钮进入。

    ![](/images/aliyun-docker/2.png)


#### 获取镜像地址

进入控制台后，点击副侧边栏的“镜像”选项卡，在进入的页面中点击右上方的“镜像仓库控制台”链接。

![](/images/aliyun-docker/3.png)

再点击副侧边栏的“Docker Hub 镜像站点”选项卡，就能看到阿里云分配的“专属加速器地址”：

![](/images/aliyun-docker/4.png)

#### 配置到Docker

这个地址怎么用呢？当然是配置到Docker的设置里了。运行Docker后，右键点击Docker Logo，点击setting按钮。在进入设置面板后，点击“Daemon”选项卡，点击“Basic”左边的开关，在下边输入框中会出现空的配置json字符串，这时就把从阿里云获取到的镜像地址填入“registry-mirrors”节点，如图：

![](/images/aliyun-docker/5.png)

然后点击Apply，大功告成！

我们可以通过在Kitematic中选择一个镜像创建一个容器试试看。

![](/images/aliyun-docker/6.png)