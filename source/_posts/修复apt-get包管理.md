---
title: 修复apt-get包管理
permalink: xiu-fu-apt-getbao-guan-li
id: 13
updated: '2016-03-24 11:35:19'
date: 2016-03-24 11:31:49
tags:
---

1.通过`wget http://oss.aliyuncs.com/aliyunecs/update_source.tgz` 下载update_source的压缩包。

2.`tar xvf update_source.tgz`解压后予执行权限` chmod 777 update_source.sh`。

3.执行该脚本`./update_source.sh`进行自动变更源操作。

![](https://img.alicdn.com/tps/TB1tGPGJFXXXXbwXVXXXXXXXXXX-878-324.jpg)
![](https://img.alicdn.com/tps/TB1Fsr1JFXXXXcFXpXXXXXXXXXX-810-184.jpg)

更新成功会提示”Success, exit now!“

如问题还未解决,请联系售后技术支持。

来源：[http://www.iisiis.com/web/vps/1884.html](http://www.iisiis.com/web/vps/1884.html)