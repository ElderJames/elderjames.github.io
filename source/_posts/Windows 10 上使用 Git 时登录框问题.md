---
layout: post
title: Windows 10 上使用 Git 时登录框问题
permalink: the-basic-credential-prompt-for-git-on-windows-10
date: 2017-11-23 18:31:34
tags:
- Git
- Windows10
categories:
- 笔记
---

如果在Pull或者Push时弹出的是Windows的提示框，并且输入正确的账号密码后还是现实验证不通过，解决办法如下：

在输入框点击取消，Git提示`Logon failed, use ctrl+c to cancel basic credential prompt`，此时再次在命令行输入账号密码即可。