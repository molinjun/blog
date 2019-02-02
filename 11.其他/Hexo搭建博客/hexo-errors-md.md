---
title: hexo常遇到的错误
comments: true
date: 2016-11-27 20:08:07
categories: Hexo
tags: Hexo
---
hexo使用node开发的，其中会依赖很多第三方依赖包。而一些依赖包的更新过快，经常会出现一些版本兼容的问题。下面就简单记录一些hexo搭建博客中遇到的错误，方便以后自己参考，也希望减少大家踩坑的时间。
<!--more-->
> 遇到问题，先看官网的[troubleshooting](https://hexo.io/docs/troubleshooting.html)

### MacOS下DTraceProviderBindings错误
**错误类型**
```shell
{ [Error: Cannot find module './build/Release/DTraceProviderBindings'] code: 'MODULE_NOT_FOUND' }
{ [Error: Cannot find module './build/default/DTraceProviderBindings'] code: 'MODULE_NOT_FOUND' }
{ [Error: Cannot find module './build/Debug/DTraceProviderBindings'] code: 'MODULE_NOT_FOUND' }
```
这种错误很奇怪，在ubuntu下是没有这个问题的。但是在macos下就总出这个错。
网上有很多说的方法，大体可能是dtrace-provider这个依赖包。
经过几次尝试，先把全局的hexo或者hexo-cli删除，再把项目目录下的node_moudles删除。然后，再使用--no-optional安装。问题解决。
```shell
# 删除全局的hexo
$ sudo npm uninstall hexo -g
# 安装全局hexo
$ npm install hexo --no-optional -g
#安装项目依赖
$ npm install --no-optional
```
至此，问题解决。

---
### 参考
[1] [mac osx 下 hexo DTraceProviderBindings 错误](http://www.ixirong.com/2016/08/30/solve-hexo-not-found-problem/)
[2] [更新Hexo 3.x 报错](http://codingpub.github.io/2016/09/07/%E6%9B%B4%E6%96%B0Hexo3-x%E6%8A%A5%E9%94%99/)
