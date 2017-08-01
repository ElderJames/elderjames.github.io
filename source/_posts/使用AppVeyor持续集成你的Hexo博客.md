---
title: 使用AppVeyor持续集成你的Hexo博客
permalink: Use-Appveyor-to-continuously-integrate-your-Hexo-blog
tags: Hexo,Appveyor,ci
updated: '2017-08-01 01:19:08'
date: '2017-08-01 01:19:08'
categories: Hexo
thumbnail: 
---

炎热的盛夏，真的是让人难以入眠。在这种不免之夜，如果还加上停电……简直就是惨绝人寰了！

还好，今晚没停电，可以让我吹着风扇安心去写这篇文章。

距离我建立这个博客都过去一周了，一直都在做这个站点的建设。首先是找到个好看的主题，之后绑定自己的https域名，然后是寻找更多玩法，比如放了只可爱的猫咪在右下角，它会跟随者你的鼠标摇头摆尾~比如在底部贴了我的GitHub贡献马赛克，在我的朋友页面抓取我的Github followers，还有就是这篇文章的重点，用AppVeyor去持续集成Hexo博客。

AppVeyor是目前唯一支持.NET开源项目的持续集成功能，运行环境是Windows，身为.NET开发者和软粉的我，怎么不支持一下这个有情怀的服务商呢？

<del>太晚了，挖坑，明天继续写。</del> 

#### 第一步，创建CI项目

访问[AppVeyor网站](https://ci.appveyor.com/projects)，注册或使用Github帐号授权登录，在PROJECTS页面点击【NEW PROJECT】，然后在右侧选择你的Hexo博客所在的仓库。

![](/images/1.png)

#### 第二步，配置项目和环境变量

点击项目里的[SETTING]选项卡，配置以下几项：

1. Default branch -> [你存放原文件的分支名]
2. Branches to build -> 选择Only branches specified below,并填入[你存放原文件的分支名]

如下图，然后点击一下下方的Save按钮。
![](/images/2.png)

点击右侧的Environment标签，添加环境变量：

| 左对齐标题 | 右对齐标题 |
| :------| :------ 
| `STATIC_SITE_REPO` | 你的仓库地址 |
| `TARGET_BRANCH` | 编译后文件存放的分支 | 
| `GIT_USER_EMAIL`| Github的用户邮箱 |
| `GIT_USER_NAME` | Github用户名 |

如下图，然后点击一下下方的Save按钮。

![](/images/3.png)

#### 第三步，获取Access Token

访问个人设置页面：

![](/images/4.png)

点击边栏下方的【Personal access tokens】选项卡，并点击右上方的【Generate new token】按钮。Token description任意填写，下方的选项中全选repo即可。

最后，点击下方绿色的【Generate token】按钮。此时就能得到你的Access Token。

也可以参考[官方文档](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/)

#### 第四步，加密Access Token

由于这个AccessToken是可以直接操作你的仓库的，而且配置文件是公开的，所以这时就要求对AccessToken进行加密。可到[AppVeyor Token加密页面](https://ci.appveyor.com/tools/encrypt)进行加密。把加密后的字符串填入下一步中的配置文件里。

#### 第五步，配置文件

在Hexo的**根目录**下新建一个`appveyor.yml`文件，写下以下内容

```yaml

clone_depth: 5

# 这里是限制了只编译有源文件的分支，这样在创建了其它分支时就不会再次编译了
branches:
  only:
  - files

environment:
  access_token:
    secure: #加密后的access_token

install:
  - ps: Install-Product node 6.0 #原始环境的node 是4.x的版本，最好升级到最新的版本，防止Hexo的插件无法安装
  - node --version
  - npm --version
  - npm install
  - npm install hexo-cli -g

build_script:
  - hexo generate

artifacts:
  - path: public

on_success:
  - git config --global credential.helper store
  - ps: Add-Content "$env:USERPROFILE\.git-credentials" "https://$($env:access_token):x-oauth-basic@github.com`n"
  - git config --global user.email "%GIT_USER_EMAIL%"
  - git config --global user.name "%GIT_USER_NAME%"
  - git clone --depth 5 -q --branch=%TARGET_BRANCH% %STATIC_SITE_REPO% %TEMP%\static-site
  - cd %TEMP%\static-site
  - del * /f /q
  - for /d %%p IN (*) do rmdir "%%p" /s /q
  - SETLOCAL EnableDelayedExpansion & robocopy "%APPVEYOR_BUILD_FOLDER%\public" "%TEMP%\static-site" /e & IF !ERRORLEVEL! EQU 1 (exit 0) ELSE (IF !ERRORLEVEL! EQU 3 (exit 0) ELSE (exit 1))
  - git add -A
  - git commit -m "Update Static Site"
  - git push origin %TARGET_BRANCH%
  - appveyor AddMessage "Static Site Updated"

```

大致的意思是从github仓库的当前分支拉取下来，编译成静态文件后，在push到目标分支。由于AppVeyor环境中是通过Access Token访问我们的仓库的，而Hexo自带的部署则在访问的过程中需要我们输入帐号密码，所以Hexo g -d的命令就不适合在这里使用。需要先编译成静态文件，再把public文件夹的静态文件push到目标分支。

最后，把这个文件提交到Github上就可以测试了！在AppVeyor的首页可以看到部署的过程和结果~

![](/images/5.png)

别说我没告诉你，还可以添加一枚徽章装装逼哦！[![Build status](https://ci.appveyor.com/api/projects/status/b0wack7uxrvifijj/branch/files?svg=true)](https://ci.appveyor.com/project/ElderJames/elderjames-github-io/branch/files)


可以看看我的[配置文件](https://github.com/ElderJames/elderjames.github.io/blob/files/appveyor.yml)，以及底部的两篇参考文章~

参考文章：
- [Hexo利用AppVeyor持续集成](http://www.shong.win/blog/2017/02/19/hexo-ci/)
- [Hexo的版本控制与持续集成](https://formulahendry.github.io/2016/12/04/hexo-ci/)


{% aplayer '剩下的盛夏' 'TF Boys' http://dl.stream.qqmusic.qq.com/133751958.mp3?vkey=65AE8DE8D14CBE02883EBE1033D6E9127E077DBCAA4DAB55CF6DE2FA33A803BEAE739F1D56B6892BCA0EBCBF8C84BF7D47EC65652B06DEF0&guid=4082350140&fromtag=30 https://y.gtimg.cn/music/photo_new/T002R300x300M000001rvkpT0so9Uk.jpg?max_age=2592000 'narrow:true' 'width:400px' 'autoplay:true' %}