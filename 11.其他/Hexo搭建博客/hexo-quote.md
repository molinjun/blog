---
title: maupassant主题的引用问题
comments: true
date: 2016-11-27 21:31:59
categories: Hexo
tags: Hexo
toc: true
---
Maupassant是我很喜欢的一款hexo的主题，简约素雅。但是，原版本上还是有些不符合我口味的地方。例如，索引的问题。markdown原有的索引，是左边的竖杠，而Maupassant 却是一个大引号，看着还不得劲啊。所以参考着试着修改了样式。
<!--more-->
### 修改方法
由于原Maupassant·的索引样式就是引号。所以需要修改主题的css样式。
css文件放在【maupassant】-【source】-【css】下的style.scss文件下。
原来的索引样式：
```css
blockquote,.stressed {
    -moz-box-sizing: border-box;
    box-sizing: border-box;
    margin: 2.5em 0;
    padding: 0 0 0 50px;
    color: #555;
    border-left: none;
}
blockquote:before,.stressed-quote:before {
    content: "\201C";
    display: block;
    font-family: times;
    font-style: normal;
    font-size: 48px;
    color: #444;
    font-weight: bold;
    line-height: 30px;
    margin-left: -50px;
    position: absolute;
}
```
修改后的：
```css
blockquote, .stressed {
  border-left: 5px solid #2B4DD5;
  padding: 0 15px;
  color: #33CC70;
  margin-left: 5px;
  margin-right: 15px;
  font-family: times;
  font-style: normal;
  font-weight: bold;
  line-height: 30px;
  font-size: 16px; }

blockquote, .stressed-quote > :first-child {
  margin-top: 0; }

blockquote, .stressed-quote > :last-child {
  margin-bottom: 0; }
```
> 这就是修改后的索引效果。
虽然整体配色还不是太满意，不过总算回到了markdown的索引。

### 参考
[1] [修改hexo的索引](http://t.tiany.me/2015/12/08/hexo-config/)
