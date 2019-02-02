---
title: hexo的站内链接问题
comments: true
date: 2016-11-27 21:59:39
categories: Hexo
tags: Hexo
toc: true
---
在写博客的时候，经常需要进行站内文章的跳转。原来的方式就是直接写上文章的外部链接。这样非常的不合理，因为如果网站外部链接换了，就需要每篇文章修改，就像硬编码似的。
<!--more-->
### 原来的方式
就像我要引用[AWS申请免费EC2主机](http://blog.gezhiqiang.com/2016/11/25/ec2-build-md/)。
```
[AWS申请免费EC2主机](http://blog.gezhiqiang.com/2016/11/25/ec2-build-md/)
```
### 新方法
但是，从hexo3.0引入了[Render Pipeline Changed](https://github.com/hexojs/hexo/wiki/Breaking-Changes-in-Hexo-3.0#render-pipeline-changed)特性，可以优雅的实现站内跳转。
```
{% post_path hello-world %}
// /2015/01/16/hello-world/
{% post_link hello-world %}
// <a href="/2015/01/16/hello-world/">Hello World</a>
{% post_link hello-world Custom Title %}
// <a href="/2015/01/16/hello-world/">Custom Title</a>
```
所以上面的引用{% post_link ec2-build-md %}。
```
{% post_link ec2-build-md %}
```

### 参考
[1] [Breaking Changes in Hexo 3.0](https://github.com/hexojs/hexo/wiki/Breaking-Changes-in-Hexo-3.0#render-pipeline-changed)
[2] [Hexo使用内链及文章中加入图片的方法](http://marshal.ohtly.com/2015/09/12/internal-link-and-image-for-hexo/)

