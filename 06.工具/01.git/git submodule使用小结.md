---
title: git submodule 使用小结
comments: true
date: 2017-03-08 22:50:17
categories: git
tags: git submodule
toc: true
---
Git Submodule 允许一个git仓库，作为另一个git仓库的子目录，并且保持父项目和子项目相互独立。
<!--more-->
## 添加子仓库
```bash
$ git submodule add <仓库地址> <本地路径>
```
新建一个父仓库main，一个子仓库sub。将父仓库克隆到本地。
```
$ git clone ssh://git@10.2.237.56:23/dennis/main.git
```
进入父仓库，并添加子仓库。
```
$ git submodule add ssh://git@10.2.237.56:23/dennis/sub.git lib
```

添加成功后，在父仓库根目录增加了.gitmodule文件。
```
[submodule "sub"]
    path = lib
    url = ssh://git@10.2.237.56:23/dennis/sub.git
```
并且在父仓库的git 配置文件中加入了submodule段。
```
$ cat .git/config

// 加了submodule段
[submodule "sub"]
    url = ssh://git@10.2.237.56:23/dennis/sub.git
```
注意：添加子仓库之后，主仓库的对应目录下（这里为lib），并不是sub仓库的文件，而是对应的commit id。如图所示：

![添加子仓库](http://obv0ef5sf.bkt.clouddn.com/git-submodule-add.png)

## 检出(checkout)
克隆一个包含子仓库的仓库目录，并不会clone下子仓库的文件，只是会克隆下.gitmodule描述文件，需要进一步克隆子仓库文件。
```
// 初始化本地配置文件
$ git submoudle init

// 检出父仓库列出的commit
$ git submodule update
```
或者使用组合指令。
```
$ git submodule update --init --recursive
```
此时子目录在一个未命名分支,此时子仓库有改动并没有检测到。
```
$ git branch
* (HEAD detached at 46a27af)
  master
```
在子仓库，切换到master分支，并git pull最新代码之后，回到主仓库目录，会显示子仓库修改，需要在主仓库提交修改，即修改指定的commit id。
```
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   lib (new commits)

no changes added to commit (use "git add" and/or "git commit -a")
```
## 更新
如果在本地修改子仓库，在主仓库 git status会显示子仓库有修改。
```
$git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)
  (commit or discard the untracked or modified content in submodules)

	modified:   lib (modified content)

no changes added to commit (use "git add" and/or "git commit -a")
```
需要现在子仓库提交，然后再到主仓库提交代码。
## 删除子仓库
- 删除.gitsubmodule里相关部分
- 删除.git/config 文件里相关字段
- 删除子仓库目录。

```
$ git rm --cached <本地路径>
```
> 如果未按照上述步骤删除，可能残留在.git/modudles文件夹内。

## 参考
[1] [Git submodule实战](http://blog.jqian.net/post/git-submodule.html)
