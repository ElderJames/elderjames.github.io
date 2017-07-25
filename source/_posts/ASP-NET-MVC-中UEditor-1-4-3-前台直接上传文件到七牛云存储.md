---
title: ASP.NET MVC 中UEditor 1.4.3 前台直接上传文件到七牛云存储
tags: |-

  - 七牛
permalink: use-qiniu-upload-file-service-on-ueditor-for-asp-net-mvc
id: 9
updated: '2015-10-30 13:35:52'
date: 2015-10-30 12:52:35
---

最近在做一个ASP.NET MVC 的项目，当然是离不开富文本编辑器，这里我选择的是百度的UEditor，虽然不够轻量，界面也不好看，但是功能还是挺多的。而因为项目主要面对移动端的用户，所以为了减轻低配服务器的压力，也为了用户体验更佳，决定在项目中接入七牛云存储。

选择七牛是看重他的CDN、上传加速和图片处理功能，最神奇的是在获取图片时只需在Url中加入几个参数即刻，而且还可以自己定图片的大小，大大减少了服务器在图片处理上的压力。

好吧，在这里就不继续安利了。

我第一次直接在项目中接入七牛，所以还是碰到了很多障碍。过程我就先不说了，我直接说怎么修改UEditor的相关代码吧。

###前台部分

- 在配置文件config.json中，修改替换所有的`upfile` 为`file`，这里是因为七牛的上传接口要求文件上传控件（input）的name为file。

- 然后在最后加上几个配置项：
```

    /*七牛配置*/
    "access_key": "",
    "secret_key": "",
    "bucket": "mybucket",
    "domain": "http://xxx.com/",//七牛默认域名或者自定义域名
    "uploadUrl":"http://upload.qiniu.com/",//七牛上传文件的域名
    "imageFieldName": "file", /* 提交的图片表单名称 */
    "tokenAction": "/Admin/Attachment/Token",//获取上传凭证token的Action
```

- 在路径 ueditor/dialogs/image/image.js 中找到` header['X_Requested_With'] = 'XMLHttpRequest';` 在下面添加以下代码：

```
  var filename = file.file.name;
  var token = "";
  console.log(filename);
  $.ajax({
       dataType: 'text',
       async: false,
       url: editor.getOpt('tokenAction') + "?file=" + filename+"&type=img",
       success: function (data) {
           oken = data;
       }
   });
   data['token'] = token;
```

- 在这段代码的上面，找到`uploader.option('server', url);`
注释并修改为`uploader.option('server',editor.getOpt('uploadUrl'));`

- 在这段代码下面寻找`  uploader.on('uploadSuccess', function (file, ret) {`，并在这个方法中的`   if (json.state == 'SUCCESS') {`增加一行代码：
```
      //在返回的地址加上七牛设置里的域名
      json.url = editor.getOpt('domain') + json.url;
```

- 另外在attachment.js、video.js 的相同地方按上面所说的修改。

###后台部分

- 引入Qiniu SDK,添加一个获取token的服务:

- 添加获取token的Action（事先已经含有UEditor SDK的代码）：
![ ](http://cdn.blog.yangshunjie.com/image/2/80/e2d2079f10dae2c8d51d0b96fe131.png)

- OssService.cs 中相关代码

![](http://cdn.blog.yangshunjie.com/image/d/b4/cbafea85536df7484629196a29cd6.png)
![](http://cdn.blog.yangshunjie.com/image/4/18/4538ae76b6e196c09489c059a8651.png)

这样就基本可以实现将图片上传到七牛的空间了。而如果想在自己的服务器保留一份文件，可以在token中加如CallbackUrl 和 CallbackBody，使七牛在上传完成后对本地的服务器发出请求，把文件的参数发到本地服务器，这时本地服务器即刻按照返回的文件路径将文件抓取到本地服务器中。这部分我还没着手做，下次再继续写吧！