---
title: hexo安装与简介-hexo搭建博客1
date: 2016-10-26 22:53:32
categories: Hexo
tags: Hexo
comments: true
toc: true
---

Hexo是一个快速、方便的博客框架。使用markdown解析文章，通过几个简单命令快速生成静态页面，并快速部署。

<!--more-->
*详细介绍，请参见hexo的[官方网站](https://hexo.io/)或者[中文网站](https://hexo.io/zh-cn/docs/)*

### 安装hexo

安装hexo之前，需要先安装好**nodejs**和**git**。如果尚未安装，可以参考如下[安装教程](https://hexo.io/zh-cn/docs/)。
使用npm安装Hexo。

```shell
$ npm install -g hexo-cli
```
安装成功之后，在命令行输入hexo version即可查看hexo的版本信息。

```shell
dennis@iZ2390d0j3fZ:~$ hexo version
hexo-cli: 1.0.2
os: Linux 3.13.0-95-generic linux x64
http_parser: 2.7.0
node: 6.3.0
v8: 5.0.71.52
uv: 1.9.1
zlib: 1.2.8
ares: 1.10.1-DEV
icu: 57.1
modules: 48
openssl: 1.0.2h
```
### 建立网站

安装完hexo-cli之后，可以通过指令快速生成脚手架。

```shell
$ hexo init <项目名称>
```
系统会生成一系列文件目录，并且安装依赖。
*这里需要注意的事，默认是从npm官网下载依赖，速度极慢。可以选择淘宝镜像来安装。*

```shell
安装cnpm
$ npm install -g cnpm --registry=https://registry.npm.taobao.org
安装nrm
$ sudo cnpm install nrm
选择淘宝镜像
nrm use taobao
```
### 目录结构

生成的文件目录如下所示：

```shell
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
```
- \_config.yml
yaml格式的配置文件。
- package.json
应用程序的信息。
- scaffolds
模板文件夹。当新建文章时，hexo会根据scaffold来建立文件。
- source
source文件夹存放用户资源。除\_post文件夹外，所有的以\_为开头的文件和文件夹以及隐藏文件将会被忽略。
markdown文件和html文件会被解析并放到**public**文件夹，其他文件会被拷贝过去。
- themes
主题文件夹。hexo会根据主题来生成静态页面。
默认的主题为**landscape**。不过我选用**maupassant**。

此时，在命令窗口执行**hexo server**，便可以在本地搭建web服务器，用于测试预览。
访问[localhost:4000]() 即可查看效果。
