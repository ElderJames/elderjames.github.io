---
layout: post
title: Angular 利用 marked.js 添加 Markdown + HTML 同时渲染的 Pipe
permalink: add-Markdown-and-html-rendering-pipe-in-Angular-using-markedjs
date: 2019-12-27 17：51
tags:
- Angular
- Typescript
categories:
- Angular
---

## 背景

最近在公司开发的一个项目需要在 Angular 上展示图文，并且需要同时支持 Markdown 和 HTML

对于同时支持 Markdown 和 HTML ，应该要分为编辑和渲染两部分考虑。

对于编辑，目前尚未找到同时支持两种格式的编辑器。我个人认为 Markdown 最好的开源编辑器是 [Editor.md](https://pandao.github.io/editor.md/)，最好的 HTML 编辑器是 [UEditor](https://github.com/fex-team/ueditor)，虽然他们俩都已经很久很久没更新过……

所以在编辑页面就只能提供两个编辑器的切换，对于 Markdown 和 HTML 分部用各自的编辑器。 但是，我可以存到同一个字段吗？

这就要考虑到渲染了，如果能找到同时支持渲染 Markdown 和 HTML 的组件，我就不需要在后端把 Markdown 原文和渲染后的 HTML 分开字段存储了，还有利于对 Markdown 文本的修改。

于是找到了 marked.js

## marked.js 介绍

[官方文档](https://marked.js.org/)

marked.js 是一个普通的js库，并不是 Angular 特有的组件，所以我们在集成时还是需要进行一些编码。同时也说明它能支持其他前端框架，甚至是普通 HTML 的直接引用，非常轻量。

## 给 Angular 添加一个用于 Markdown 渲染的 Pipe

Angular Pipe 是 Angular 中用于字符格式转换的组件，专门处理输出字符的转换，如日期、翻译等等在Angular开发中都比较常用。今天我们就给项目添加一个用于 Markdown 渲染的 Pipe。

### 一、npm 引用

需要引用marked.js和它的 typescript 类型库来辅助模块声明

```bash
    npm install marked
    npm install @types/marked
```

### 二、创建 Pipe

通过 angular-cli 创建 Pipe 文件

```bash
ng generate pipe marked
```

这样它能自动创建一个名为 "marked" 的 Pipe (`marked.pipe.ts`)，并导入到你的 `app.module.ts` 文件中了。

### 修改 `marked.pipe.ts`

生成出来的 `marked.pipe.ts` 文件如下：

```ts
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'marked'
})
export class MarkedPipe implements PipeTransform {

  transform(value: any, args?: any): any {
    return null;
  }

}
```

我们需要导入 marked , 然后修改 transform 方法来解析 Markdown 并返回 HTML 字符串。

更改之后的代码如下：

```ts
import { Pipe, PipeTransform } from "@angular/core";
import * as marked from "marked";

@Pipe({
  name: "marked"
})
export class MarkedPipe implements PipeTransform {
  transform(value: any): any {
    if (value && value.length > 0) {
      return marked(value);
    }
    return value;
  }
}

```

其中的 `transform` 方法将接受文本作为值，如果它有长度，则返回解析后的 HTML。否则，它只返回传递的值。

### 使用方法

使用方法跟其他 Pipe 几乎差不多，但是有一点，就是如果渲染出来的是 HTML 字符串，需要让 Angular 自己在渲染成最终的 HTML，我们需要把 HTML 字符串传入到 Angular 模版中一个 HTML 元素的 `innerHTML` 属性中，举个例子：

```ts
// app.component.ts
...
public markdownString: string = 'This is text with **markdown**';
...

// app.component.html
...
<div [innerHTML]="markdownString | marked"></div>
...
```

### 总结

这样就大功告成啦！你可以尝试使用同时含有 Markdown 和 HTML 的字符串，就像 Github 里众多开源项目的 README 文档一样，使用穿插着 HTML 排版的文档来渲染美观的图文。