---
title: 使用AppVeyor持续集成你的Hexo博客
permalink: Use-Appveyor-to-continuously-integrate-your-Hexo-blog
tags: Hexo,Appveyor,ci
updated: '2017-08-01 01:19:08'
date: '2017-08-01 01:19:08'
categories: Hexo
---

炎热的盛夏，真的是让人难以入眠。在这种不免之夜，如果还加上停电……简直就是惨绝人寰了！

还好，今晚没停电，可以让我吹着风扇安心去写这篇文章。

距离我建立这个博客都过去一周了，一直都在做这个站点的建设。首先是找到个好看的主题，之后绑定自己的https域名，然后是寻找更多玩法，比如放了只可爱的猫咪在右下角，它会跟随者你的鼠标摇头摆尾~比如在底部贴了我的GitHub贡献马赛克，在我的朋友页面抓取我的Github followers，还有就是这篇文章的重点，用AppVeyor去持续集成Hexo博客。

AppVeyor是目前唯一支持.NET开源项目的持续集成功能，运行环境是Windows，身为.NET开发者和软粉的我，怎么不支持一下这个有情怀的服务商呢？

太晚了，挖坑，明天继续写。可以先看看我的[配置文件](https://github.com/ElderJames/elderjames.github.io/blob/files/appveyor.yml)，以及底部的两篇参考文章~

{% aplayer '剩下的盛夏' 'TF Boys' http://dl.stream.qqmusic.qq.com/133751958.mp3?vkey=65AE8DE8D14CBE02883EBE1033D6E9127E077DBCAA4DAB55CF6DE2FA33A803BEAE739F1D56B6892BCA0EBCBF8C84BF7D47EC65652B06DEF0&guid=4082350140&fromtag=30 https://y.gtimg.cn/music/photo_new/T002R300x300M000001rvkpT0so9Uk.jpg?max_age=2592000 'narrow:true' 'width:400px' 'autoplay:true' %}

参考文章：
- [Hexo利用AppVeyor持续集成](http://www.shong.win/blog/2017/02/19/hexo-ci/)
- [Hexo的版本控制与持续集成](https://formulahendry.github.io/2016/12/04/hexo-ci/)