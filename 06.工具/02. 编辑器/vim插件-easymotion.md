---
title: vim插件1-easymotion
comments: true
date: 2016-11-03 22:31:41
toc: true
categories:
- vim
tags:
- vim

---

在vim操作中，经常需要跳转。而easymotion是一款非常好的用于实现快速跳转的插件。下面介绍几个常用的操作。详细的安装及使用教程请戳[官网][1]。
<!--more-->
```shell
leder键为逗号。
```
#### 向后word跳转
```shell
leader leader w
```
光标之后的首字母会高亮。点击需要跳转
到的字母，即可跳转到该单词。

#### 向后区域字母跳转
```shell
leader leader f
```
然后输入要跳转到的字符。光标之后单词的首字母会高亮。点击需要跳转到的字母，即可跳转到该字母。

#### 向后区域前一字母跳转
```shell
leader leader t
```
然后输入要跳转到的字符。光标之后单词的首字母会高亮。点击需要跳转到的字母，即可跳转到该字母**前一字母**。

#### 向后行跳转
```shell
leader leader j
```
光标之后行的首字母会高亮。点击需要跳转到的字母，即可跳转到该行。

#### 向前区域行跳转
```shell
leader leader k
```
光标之前行的首字母会高亮。点击需要跳转到的字母，即可跳转到该行。

#### 前后区域字母跳转
```shell
leader leader s
```
然后输入要跳转到的字符。光标前后区域内的该字母会高亮。点击需要跳转到的字母，即可跳转到该字母。

#### 向后区域行跳转
```shell
leader leader j
```
然后输入要跳转到的字符。光标之后单词的首字母会高亮。点击需要跳转到的字母，即可跳转到该单词。

#### 行内前后字母跳转
```shell
leader leader h/l
```
然后输入要跳转到的字符。光标之后单词的首字母会高亮。点击需要跳转到的字母，即可跳转到该字母。
[1]: https://github.com/easymotion/vim-easymotion
