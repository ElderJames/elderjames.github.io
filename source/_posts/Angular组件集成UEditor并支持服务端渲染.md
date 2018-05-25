---
layout: post
title: Angular组件集成UEditor并支持服务端渲染
permalink: Angular-integrating-with-UEditor-and-ssr-supporting
date: 2018-05-26 00:35:00
tags:
- Angular
- Typescript
categories:
- Angular
---

Angular、React和Vue 被誉为现今最强大的前端三大框架，其中React和Vue在移动端（web、rn）开发方面如鱼得水，而Angular凭借优秀的模块化设计、工程化构建以及强类型语言Typescript加持，在大中型规模的应用开发上相比另外两个框架更能大展拳脚。

本人作为偏向后端（大部分是.NET）的开发者，对Angular的最主要需求就是中台(管理后台)应用了。随着.NET Core的开源和壮大，它以很快的速度与“现代”技术接轨，比如跨平台特性使得在微服务、容器技术以及DevOps上与其他平台有着平等的地位；更高的性能、更轻的体量使得它有着比一直以此特点著称的Node.js更加有优势……自从了解到ASP.NET Core框架下的JavascriptService项目后，我就一直在关注着他的发展，这是因为它支持了Angular、React和Vue 这类单页应用（single page application, spa）的服务端渲染（server-side rendering, ssr）的特性。服务端渲染主要是在完全支持前后端分离开发的优点的同时，弥补了spa需要在浏览器完全加载后才开始渲染页面导致的首屏加载慢、难以进行SEO优化的缺点。

所谓服务端渲染，其实并不是什么新技术，就是从服务端渲染出HTML数据返回给浏览器的过程，这在spa应用大行其道之前，随着网页的诞生就发展起来了的。到了现在，.Net、Java、PHP、Node.j、Python等等都有自己的服务端渲染模版框架，技术点也是大同小异，无非就是老掉牙的MVC模式，将后端数据与前端模版一起渲染出HTML代码输送到浏览器上再展示出图文并茂的网页来。

但是，到了Spa时代，Spa都是把模版、静态文件等都下载到本地后请求服务端数据，再结合模版渲染出html,最后才能在浏览器展示出页面。虽然首屏加载会慢，但通过页面上的内部路由（链接）去跳转页面就很快，因为已经没有了模版加载的过程，只需请求服务器获取小量的数据即可。不过，这样的应用对首屏加载速度和SEO优化要求比较苛刻的产品就无法满足需求了，于是各家都发现还是老技术好，开始了对服务端渲染的支持。其中Angular就有专门的Universal项目来做这方面的支持。

我一直在关注Angular的支持ssr中台组件库项目，但在差不多两年的时间里，我找遍整个Github的很多组件库项目，都很少有比较完整且支持ssr的，大多数都只是“Demo”、“Starter”类项目。我甚至给几个组件比较完整的项目提了issue，得到的答复大都是由于第三方组件依赖window、document等浏览器特有对象，不予实现。

没办法，得知Angular 6.0和Material2 6.0发布之际，决定自己来对某个组件库做改造吧！

早在去年的Angular 4.0版本，Angular Universal项目就组件完善了。还在beta版本的Material组件库也有消息逐步支持了。如今6.0发布，我独钟的Material已经有很漂亮的基本组件了，决定马上开干！

请大家Star一下我两天前才发布的项目[ElderJames/aspnetcore-material-universal](https://github.com/ElderJames/aspnetcore-material-universal)

此项目修改自angular-material-app项目，因为此项目大部分组件都是来自原生的Material组件，所以让它支持SSR会比较方便。

要支持SSR，就必须不能有对window、document等浏览器对象的引用，否则就会报错无法运行。所以对存在需要引用这些对象的情况，一般会是先判断是否在浏览器环境执行。

而到了集成功能最完善的富文本编辑器UEditor时遇到了棘手问题，我花了一天时间才算是解决了。

#### 解决难点1：*避免浏览器对象引用* 

UEditor的js库“日久失修”，本身就不支持模块化，window引用直接暴露在全局环境，也就是一加载就会引用……所以必须对它本身进行修改，在原有代码前加入判断。



先把js库下载并放在assets（或其他用来放置静态文件的）目录，分别在[neditor.all.js](https://github.com/ElderJames/aspnetcore-material-universal/blob/master/ClientApp/assets/neditor/neditor.all.js)、[neditor.copnfig.js](https://github.com/ElderJames/aspnetcore-material-universal/blob/master/ClientApp/assets/neditor/neditor.config.js)、[zh-cn.js](https://github.com/ElderJames/aspnetcore-material-universal/blob/master/ClientApp/assets/neditor/i18n/zh-cn/zh-cn.js)文件源程序前加入以下代码：

```js

if (typeof window === 'undefined')
  return;

```

#### 解决难点2：*模块化引用*

UEditor本身的使用方式是在html文件通过`<script>`标签引入以上3个js文件，然后操作一个由这些文件构造的**UE**对象。而在spa中，还使用`<script>`方式引入js的话，就会导致js暴露到外部，每个页面都会加载，达不到模块化的要求，所以我们必须用模块化的f方式引入这些文件。没错，就是`import`关键字。但是import之后，怎么在模块中取得UE对象呢？通过一翻查询，在`import`前先使用`declare`关键字声明这个对象，`import`后就能引用了。如下所示：

```js
declare let UE: any;

import '../../../assets/neditor/neditor.config.js';
import '../../../assets/neditor/neditor.all.js';
import '../../../assets/neditor/i18n/zh-cn/zh-cn.js';
```

注意，使用`import`方式引入js文件的话，内部对静态文件的引入就会错乱，我们需要在UEditor的配置项指定静态资源路径:


```js
UEDITOR_HOME_URL: '/assets/neditor/'`
```

#### 解决难点3：*绑定Dom元素*

UEditor的初始化要求传入一个元素的id，来让UEditor能绑定到指定的元素。但是Angular的模板渲染是通过虚拟Dom的方式，显然UEditor无法绑定Dom，只能等虚拟Dom渲染出真实Dom时才能绑定，那么就还要留意Angular渲染相关的生命周期钩子了。

通过注入ElementRef对象来获取当前模板的真实dom元素：

```ts
export class UEditorComponent implements ControlValueAccessor, OnInit, OnDestroy {
    
    isBowser: boolean;
    editor_text: string;
    elementRef: ElementRef;

    constructor(elementRef: ElementRef, render: Renderer2) {
        this.isBowser = typeof window !== 'undefined';

        if (!this.isBowser)
            return;

        this.elementRef = elementRef;
    }
}
```

`ngOnInit`钩子中进行每次组建渲染后的UEditor初始化，`ngOnDestroy`钩子进行UEditor实例的释放，防止内存泄漏。
```
    ngOnDestroy() {
        if (!this.isBowser)
            return;

        console.log('destroy。。。');

        this.ue && this.ue.destroy();
        this.ue = null;

        this.isReady = false;
    }

    ngOnInit() {
        if (!this.isBowser)
            return;

        var id = new Date().getUTCMilliseconds() + '';
        this.elementRef.nativeElement.id = id;

        let con: any = _.merge({}, this.defaultConfig, this.config);
        this.ue = UE.getEditor(id, con);

        this.registerEvents();

    }
```

好了，先写到这里，相信这次实践能对其他类似的js库的ssr支持集成带来有用的经验。