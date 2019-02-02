---
title: linux指令学习—— find 
comments: true
date: 2017-02-19 22:56:12
categories: ubuntu
tags: ubuntu shell
toc: true
---
find 指令可以用来遍历路径下的文件。
<!--more-->
## 命令格式
```
find [path] -option [- print] [-exec -ok command {} \]
```
path为要查找的目录路径。
## 常用参数
### name
按文件名查找。
```
find . -name *.js  // 查找后缀名为.js的文件 
```
可以使用通配符来查找。
```
* 代表1到多个任意字符
？代表单个字符
[A-Z]*  大写字母开头的文件
```
### type
按文件类型查找。
```
find . -type f // 查找当前目录的所有文件
```
常见的文件类型：
```
f  普通文件
d  目录
b  块设备文件
c  字符设备
p  管道文件
l  符号链接
```
### exec 
对查找的文件执行指令。
```
find . -type f -exec 命令 {} \;
```
> 注意：大括号{} 和 \ 之间有个空格，而 \ 和 分好之间没有空格。
