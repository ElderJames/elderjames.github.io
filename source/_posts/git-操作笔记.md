---
title: git 操作笔记
permalink: git-cao-zuo-bi-ji
id: 15
updated: '2016-03-28 23:52:38'
date: 2016-03-28 14:17:16
tags:
- Git
---

`$ cd myproject`  你建立的项目文件夹
 
`$ git init`   执行git的本地初始化
 
`$ git add .`  将所有的文件添加到版本控制系统
 
`$ git commit -m "initial commit"`  在本地提交到版本库
 
`$ git remote add origin git@116.255.160.144://srv/gitserver/jewels.git` 添加远程仓库(jewels是服务器端项目管理到名字，与本地项目名字无关)
 
`$ git push origin master` 将本地版本库推送到远程仓库

`git checkout master` //进入master分支

`git checkout -b frommaster` //以master为源创建分支frommaster

// 把本地仓库提交到远程仓库的master分支中

`git push ssh://git@dev.lemote.com/rt4ls.git master` 

或使用标记

`$ git remote add origin ssh://git@dev.lemote.com/rt4ls.git`

`$ git push origin master`

`$ git push origin test:master`         // 提交本地test分支作为远程的master分支

`$ git push origin test:test `             // 提交本地test分支作为远程的test分支

`$ git push origin :test`              // 刚提交到远程的test将被删除，但是本地还会保存的，不用担心

文章来源：[git 远程分支创建与推送](http://www.cnblogs.com/wangkangluo1/archive/2011/09/02/2164313.html)