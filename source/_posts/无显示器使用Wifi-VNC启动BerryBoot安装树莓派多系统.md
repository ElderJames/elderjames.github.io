---
title: 无显示器使用Wifi-VNC启动BerryBoot安装树莓派多系统
date: 2018-06-05 13:36:41
permalink: boot-the-berryboot-installer-for-raspberry-pi-with-wifi-and-vnc-but-no-screen
tags: 
- 树莓派
categories:
- 树莓派
thumbnail:
---

网络连接部分参考官方文档：https://www.berryterminal.com/doku.php/berryboot/headless_installation

网络信息中填写与当前用于远程访问树莓派的电脑处在同一局域网，同一个路由器就只是ip不同，掩码和网关都一样。

VNC下载地址：https://www.realvnc.com/en/connect/download/viewer/ 选择Standalone EXE单文件版本即可。

VNC中服务器ip填写在树莓派cmdline.txt文件中设置的ip，设置里选择24位颜色。即可访问。