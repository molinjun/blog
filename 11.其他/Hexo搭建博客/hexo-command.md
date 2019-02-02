---
title: hexo常用命令-hexo搭建博客3
date: 2016-10-30 22:53:32
categories: Hexo
tags: Hexo
comments: true
toc: true
---
hexo提供了一系列命令来完成项目创建、创建文章、静态化文件、调试以及部署的指令。
以下就介绍几种常用的命令。
<!-- more -->

### init
```
$ hexo init <目录名>
```
新建一个网站
### new
```
$ hexo new [layout] <title>
```
新建一篇文章。
若不设置layout，则使用\_config.yml中设置的default_layout来新建。

### generate
```
$ hexo generate
```
生成静态文件。生成html、css和jss等静态文件，并放于publich目录下。

### server
```
$ hexo server
-p, --port  重设端口
-s, --static  只使用静态文件
-l, --log 启动日记记录，使用覆盖记录格式
```
启动服务器。默认情况下，访问网址为： http://localhost:4000/。

### deploy
```
$ hexo deploy
```
部署网站。
