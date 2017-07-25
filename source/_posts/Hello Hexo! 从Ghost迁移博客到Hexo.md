---
title: Hello Hexo! 从Ghost迁移博客到Hexo
permalink: migrate-blog-from-ghost-to-hexo-and-github
tags: hexo
updated: '2017-07-26 01:19:08'
date: '2017-07-26 01:19:08'
---

很高兴用上了hexo来搭建博客！

因为最近使用学习angular、vue、cordova等框架和技术，慢慢喜欢上了用node来构建应用程序的过程。我原来的博客系统是Ghost,部署在亚马逊AWS上。最近AWS一年免费体验准备到期了，博客也要找个别的地方安放。

早就知道了github上能搭建博客，但是之前是觉得构建起来很复杂，就没有尝试。这次，我就要尝试一下了！就是你现在看到的模样！

这篇文章要记下来整个迁移过程。从hexo的安装、构建、部署、安装主题、从Ghost迁移文章。

## 安装Hexo  

*[官方文档](https://hexo.io/zh-cn/docs/index.html)*

0. 安装 [Node.js](http://nodejs.org/) 命令行输入 `npm`,有相关信息则安装成功 
1. 安装 [Git](http://git-scm.com/) 命令行输入`Git`，有相关信息则安装成功
2. 在命令行输入以下命令安装Hexo
```bash
$ npm install -g hexo-cli
```
3. 初始化博客

```bash
# 初始化一个目录，<folder>不填则为当前目录
$ hexo init <folder>
$ cd <folder>

# 安装依赖包
$ npm install
```

新建完成后，指定文件夹的目录如下：

```
.
├── _config.yml   -> 全局配置文件
├── package.json  -> node的项目配置文件，用来配置依赖包和一些npm命令
├── scaffolds     -> 模板文件夹。当您新建文章或页面时，Hexo 会根据 scaffold 来建立文件
├── source        -> 资源文件夹是存放用户资源的地方,在里面创建文章、页面、图片等博客的内容
|   ├── _drafts   -> 草稿文件夹
|   └── _posts    -> 文章文件夹
└── themes        -> 主题文件夹。Hexo 会根据主题来生成静态页面
```

4. 输入以下命令，启动站点查看效果

```bash
$ hexo server
```
如果想更换端口，可以在命令后追加参数

```bash
$ hexo server -p 8080
```
此时在浏览器访问 [http://localhost:4000](http://localhost:4000) 就可以看到博客效果啦！

什么?默认主题不好看？别急，后面会讲如何安装主题~ 接下来介绍如何发布到Github！

## 发布到GitHub

接下来介绍如何把这个博客发布到Github Page。

Github Pages是Github为开源项目提供展示文档和示例的功能，有300M的空间，但只是放置静态文件，所以Hexo就必须先编译成静态文件，再push到Github Pages仓库。

1. 创建Github仓库，名称必须是[Github用户名].github.io,例如本博客的仓库 [elderjames.github.io](https://elderjames.github.io),然后获得git仓库的地址，如https://github.com/ElderJames/elderjames.github.io.git

2. 部署配置

在开始之前，您必须先在 _config.yml 中修改参数

```yml
deploy:
  type: git
  repo: https://github.com/ElderJames/elderjames.github.io.git
```
3. 命令行输入以下命令，安装部署工具 hexo-deployer-git

```bash
$ npm install hexo-deployer-git --save
```

4. 输入以下命令，编译并发布到Github

```bash
$ hexo deploy --generate
```
*这个命令是可以分开的，一个是编译静态文件，一个是发布，具体请[参阅文档](https://hexo.io/zh-cn/docs/generating.html)*

5. 如果成功提交到Github,则可以马上访问仓库名的那个地址 https://elderjames.github.io

## 尝试写一篇新文章

``` bash
$ hexo new "My New Post"
```

## 安装主题

主题在Github上有很多了，比如我现在安装这个，只要把主题文件复制到 themes 目录下，每个子目录代表一个主题。主题还能有很多扩展的设置，具体可能要参考主题的文档。

## 从Ghost迁移文章

由于我之前的博客部署在 AWS 上，是 Ghost 博客系统，好处是文章用 markdown 编写，可以很好的迁移到 Hexo。

0. 从Ghost博客导出博客数据，几下保存的路径

访问 http://yourblog.com/ghost/debug/ , 点击 【导出】按钮下载一个 Json 文件。

1. 安装迁移工具

```bash
$ npm install -g hexo-migrator-ghost
```

2. 输入以下命令导入文章到 Hexo, `<source>` 为第0步下载的json文件的路径。

```bash
$ hexo migrate ghost <source>
```

然后，可以愉快得看到 `./source/_posts`（如果有草稿，则包括`./source/_dragts`）文件夹里已经出现了原来在 Ghost 中的文章。

好了，再执行一次 `hexo deploy --generate` 发布文章吧！！

别忘了互相关注哦！晚安！