---
title: Ghost博客邮件SMTP服务配置
tags: |-

  - ghost
  - smtp
permalink: the-smtp-setting-for-ghost-blog
id: 7
updated: '2015-10-13 17:25:43'
date: 2015-10-13 16:57:04
---

前天由于回到学校，用回了我的Pocker2机械键盘，在输入密码时连续不知道大小写切换键换了地方，连续输错了密码，帐号被封了……

Ghost也是的，连博客主人的号都能封，还不能（也可能是我不会）通过操作服务器（比如Wordpress能通过删除配置文件）来重置密码，只能通过发送邮件，好吧，之前就一直配置不成功，现在好了，只能硬着头皮给它设置好了。

看过几篇关于配置邮箱的文章，但还是不行，我就以为是我人品不好了。但是今天重新看了配置文件之后，终于发现了我错在哪，所以现在记录一下吧。

其实问题就在于**填写mail配置的地方不对**

之前我是替换了` development`节点里的配置。其实，正确的地方是在开头第一个节点`production`里的`mail:{}`,把它替换成配置的信息就好了。（这可能跟node.js运行模式的选择有关，我在第一篇文章里有提过，在守护进程里设置的是**production**）

配置代码片段如下：

    mail: {
             transport: 'SMTP',
             from: '杨舜杰博客 <blog@yangshunjie.com>', //发件人
             options: {
                  host: 'smtp.yangshunjie.com',//我已经将自己的域名绑定到了阿里云的企业邮箱
                  secureConnection: false, //不使用SSL                     
                  port: 25,//端口
                  auth: {
                      user: 'blog@yangshunjie.com', //邮箱地址
                      pass: '********'  //密码
                  }
              }
          },

最后重启一下ghost,一按找回密码，好了，收到邮件了，密码重置了，这篇博文也写完了。
