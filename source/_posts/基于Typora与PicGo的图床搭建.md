---
title: 基于Typora与PicGo的图床搭建
#tags: [hexo建站,hexo部署,github部署,个人博客]      #添加的标签
categories: 
  - 优雅写作
  - 图床搭建
description: 
#cover: 
---



## 前言

图床，就是指一些可以把图片存放到网上并且引用到其他网站使用的服务，就像以前的网络相册。本文介绍搭建的图床是基于Typora与PicGo而实现的。



## 搭建

1. 在github上（或者其他网站）建立一个仓库专门用来存放图片，仓库一定要设置成**public**的，否则会导致图片外链不成功。

2. 在github中生成一个Token，用来给PicGo上传图片赋予权限。

3. 在PicGo中，设置github的信息，设定仓库名处要设置成用户名+仓库名。

   ![](https://raw.githubusercontent.com/OverCookkk/PicBed/master/blogImg/picgo.png)

4. 设置完后，图片就可以通过PicGo上传到github的仓库里了，然后通过复制PicGo里的链接到用的地方显示图片了。

