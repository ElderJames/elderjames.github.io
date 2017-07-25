---
title: Framework7多视图（View）工具栏布局
tags: |-

  - Framework7
permalink: the-multiple-views-structure-on-framework7
id: 5
updated: '2015-10-11 00:48:05'
date: 2015-10-10 23:33:25
---

HiApp 可以算是一个比较完整的App，其中有很多能够学习的地方，比如它用的RequireJS 、它的界面交互等等，而对于初学者来说，它的布局结构是很值得学习的。因为它的结构是目前绝大多数App的常用布局，分为顶部导航栏Navbar，中 部多个视图View，底部是切换view的工具栏Toolbar。

它的特殊之处在于在界面首层显示了工具栏，而且4个view的底部都是给工具栏空出了位置，而进入到第二层以后，就自动隐藏了工具栏，view内的页 面没有给工具栏留出位置。这是一个很巧妙的结构。

在文档里，介绍的布局实例里都没有提及这种布局，所以我才说应该去学习。

但是由于HiApp的模块化结构使得它不易看懂，经过我自己的试验，终于做出了效果大致相同的结构。

用到的结构html是：

    <div class="views  navbar-through tabs">
        <div id="view-1" class="view tab active">
            <div class="navbar">
            </div>
            <div class="pages">
            </div>
        </div>
        <div id="view-2" class="view tab">
            <div class="navbar">
            </div>
            <div class="pages">
            </div>
        </div>
        <div id="view-3" class="view tab">
            <div class="navbar">
            </div>
            <div class="pages">
            </div>
        </div>
        <div id="view-4" class="view tab">
            <div class="navbar">
            </div>
            <div class="pages">
            </div>
        </div>
        <!-- Bottom Toolbar-->
        <div class="bottom-toolbar toolbar tabbar tabbar-labels toolbar-hidden">
            <div class="toolbar-inner">
                <a href="#view-1" class="link tab-link active">
                    <i class="icon-table icon-2x"></i>
                    <span class="tabbar-label">课表</span>
                </a>
                <a href="#view-2" class="link tab-link">
                    <i class="icon icon-book icon-2x">
                        <span class="badge bg-red">4</span>
                    </i>
                    <span class="tabbar-label">读书</span>
                </a>
                <a href="#view-3" class="link tab-link">
                    <i class="icon icon-compass icon-2x"></i>
                    <span class="tabbar-label">发现</span>
                </a>
                <a href="#view-4" class="link tab-link">
                    <i class="icon icon-user icon-2x"></i>
                    <span class="tabbar-label">我</span>
                </a>
            </div>
        </div>
    </div>
这里注意几个地方：

- views里添加 navbar-through类,这使得各个view中的页面默认有工具栏布局

- toolbar里添加 toolbar-hidden类，这使得toolbar默认关闭，可以给欢迎页面或者登陆页面有全屏的效果。

那么，如何使进入第二层以上的页面隐藏工具栏呢？

首先看看页面布局：

    <div class="pages navbar-through">
        <div class="navbar">
            <div class="navbar-inner">
                <div class="center">Framework7</div>
            </div>
        </div>
        <div id="me" data-page="Me" class="page navbar-through no-swipeback">
            <div class="page-content contacts-content">
                ...
            </div>
        </div>
    </div>

page中添加no-toolbar类，这使得这个页面有无工具栏布局（底部没有留空），而这仅仅是对page布局有作用，而不会有文档中说的自动隐 藏toolbar效果，因为no-toolbar只会对跟该页面同view下的toolbar起作用，而我们的toolbar不是在view内，而是在各个view之外，所以要隐藏toolbar，还是要靠JS来实现。

因为本人是初学者，看不懂RequireJS ，所以JS部分是我自己写的。
而JS的重点就在于，如何监听当前页面是否在首层呢？很容易想到，要Page添加事件，检验一下当前页是否为第一层的页面。但是要监听什么事件呢？

如果是pageInit事件，那么在后退时会出现问题，因为F7为了呈现页面动画切换效果，在返回时会提前初始化前一个页面，比如，页面跳转顺序是A —>B—>C，现在要从C返回B，在返回前，view内会有BC两个页面以供切换，而切换到B的时候，F7会销毁C页面，加载A页面，这时会触发pageInit事件，如果用在pageInit触发时隐藏工具栏，那么当你从C切换到B时，工具栏就隐藏了，显然不符合我们要在A页面隐藏的要求。
而pageAfterBack事件是在返回动作完成后触发，这就会使得动画效果不连贯，不美观。

那么，应该用哪个事件呢？

我这里用的是pageBeforeAnimation，在页面加载完成后不触发，等要切换了动画要开始前才触发。这样，我们就能够在进行页面切换 动画的同时进行工具栏显示动画。

而另一个重点是，如何检测页面是否为第一层的页面？

大家都知道，页面事件会传入一个参数e，通过e.detail.page获得pageData，里面包括了name，url等。但是，这个e传的是触发事件的参数，比如从B切换到A，传入的是B页面的参数，所以用name、url等都不能检测出要切换到的页面是否为A页面。但是，当然这个是有解决的办法的。我们从文档中可以找到，PageData中的page.view可以获取当前view，而Views中的View.activePage属性却可以获取当前view的活动页面，也就是说，B切换到A，在动画是在A页面上进行的，活动页是A，所以，这样就能获取到我们要切换到的那个页面的数据了，再进行检验，就OK啦！

好啦，分析完，下面是JS代码

     //初始化后动画开始前检测当前视图的活动页，如果是第一层的4个页面，则显示工具栏
    $$(document).on('pageBeforeAnimation', function (e) {
        var page = e.detail.page.view.activePage;
        if (page.name === 'Course' || page.name == 'Book' || page.name == 'Discover' || page.name == 'Me') {
            myApp.showToolbar('.toolbar');
        }
        else {
            myApp.hideToolbar('.toolbar');
        }
    });

好啦，这个问题就解决啦！希望大家多多分享你的经验哦！另外，有问题的话记得先看看文档，因为文档会解决很多问题的！