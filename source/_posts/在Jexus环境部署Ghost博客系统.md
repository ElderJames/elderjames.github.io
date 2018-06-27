---
title: 在Jexus环境部署Ghost博客系统
tags: 
- 环境部署
- Linux
- Jexus
- Blog
permalink: install-ghost-blog-on-jexus-successfully
id: 3
updated: '2015-10-17 15:13:50'
date: 2015-10-05 01:46:13
---

###建博感言
很久以前就想有自己的博客，但由于无暇顾及而荒废了。之前使用过WordPress，它的功能十分强大，安装之后就一直手痒弄着主题和插件。但是这实在太麻烦了，也不是我最终需要的，。我自认不是一个设计师，不能做出个漂亮的东西，但我喜欢什么样的博客我还是知道的。

###本文主题
Ghost博客系统由node.js写成，它的轻便以及高性能不言而喻，界面用bootstrap风格，原生多种客户端，默认使用Markdown编辑器，完全就是为书写而打造，实在是漂亮，很早之前就想使用，但由于没有自己的服务器，所以一直没用上，现在阿里云有学生优惠，买了配置很低的服务器，就打算使用Ghost来作为我的博客。

而我的服务器环境有点特殊。

我本身是个.NET开发者，但由于还是学生，囊中羞涩，又因为相对与Windows服务器，Linux在性能和安全性上确更胜一筹，而.NET的跨平台技术已经进行很长一段时间了，微软去年也宣布了.NET开源和在跨平台上努力着，所以我认为.NET跨平台将是一个行业的趋势，希望能学习在Linux服务器上运行我的.NET程序。

于是我用的服务器程序是Jexus，一个强大的.NET服务器。

可能使用Jexus的.NET极客们大多使用自己写的.NET博客系统（已经有大神写出一个开源系统）或者是WordPress，网上并没有关于在Jexus上部署Ghost的教程，但是可以发现网上的教程有一个共通点，就是都使用Web Server的反向代理，而不是直接运行在其之上（而是运行在node.js上的），正好Jexus上有强大的反向代理功能，正好这个功能没使用过，于是今天我就在Jexus上部署了Ghost博客程序。

###过程
node.js和Ghost的安装步骤在官网上都有，我们只需要安装ghost的部骤，不安装和配置服务器程序，因为我们已经安装好jexus了。其中要注意的是，如果直接下载node.js最新版本，就会下载版本号为4.1.1的node.js，而Ghost只能是0.10.* - 0.12.*,官方推荐使用0.10.40，所以记得下载特定版本。

###步骤

 **安装必要的Ubuntu软件包：**

    sudo apt-get update

    sudo apt-get upgrade -y

    sudo aptitude install -y build-essential zip vim wget

**获取node.js**

    sudo wget http://nodejs.org/dist/v0.10.40/node-v0.10.40.tar.gz

    tar -zxf node-v0.10.40.tar.gz 

    cd node-v0.10.40

    $ ./configure 

    $ make 

    $ sudo make install 

**安装Ghost中文版**

    cd /var/www/myblog/

    wget http://dl.ghostchina.com/Ghost-0.7.0-zh-full.zip

    unzip -d ghost Ghost-0.7.0-zh-full.zip

    cd ghost

    sudo npm install --production

    npm start

此时Ghost已安装完成，但是一旦使用`Ctrl+C`退出到控制台，Ghost进程就会关闭。为了使我们的程序一直保持在后台运行，我们需要使用进程保护程序*PM2*。

    //需要先进入上面Ghost的安装目录，执行：
      sudo npm install pm2 -g

    //创建一个守护进程，其中“ghost”是对这个进程的命名，可自行修改
      NODE_ENV=production pm2 start index.js --name "ghost"

    //让PM2知道在开机后自动运行我们的网站
      pm2 startup ubuntu
      pm2 save
  
**pm2相关命令:**

`pm2 kill ghost `（清除所有ghost进程）

`pm2 <start|stop|restart> ghost `（启动|停止|重启ghost进程）

`pm2 startup <centos|ubuntu|amazon>` （让pm2能够在这3个系统上自动启动）

至此，pm2 已经可以守护 Ghost 博客永远在线。

**Jexus配置**

等Ghost安装之后，就到了Jexus出场了！我们在这里使用的是Jexus强大的的反向代理功能。将我们自己的域名绑定到Ghost在服务器上的本地IP，只需在Jexus新建一个配置文件，并加入一行：`reproxy=/  http://127.0.0.1:2368`即可，完整的配置是：

    port=80
    root=/ /var/www/ghost/ghost #ghost所在目录
    hosts=yangshunjie.com #你需要的域名
    reproxy=/  http://127.0.0.1:236

最后记得在配置文件`config.js`里设置你的域名哦！

###总结

这个部署过程让我学会node.js的部署，以及Jexus反向代理的一个用处，就是绑定域名到其他web Server的程序上，这样以后就可以在同一服务器上运行多种语言的程序了（我的服务器目前是.NET+PHP+Node.js）

>[Ghost博客平台：在Ubuntu上安装Ghost](http://www.applecho.com/installing-ghost-ubuntu/)

>[Ghost博客平台：安装pm2](http://www.applecho.com/installing-ghost-pm2/)

>[Jexus 负载均衡](http://www.cnblogs.com/shanyou/archive/2013/05/04/3059950.html)