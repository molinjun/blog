<!-- TOC -->

- [打造高效的 Mac 开发环境](#打造高效的-mac-开发环境)
  - [通用设置](#通用设置)
  - [软件安装](#软件安装)
    - [基础软件](#基础软件)
    - [效率工具](#效率工具)
  - [开发环境配置](#开发环境配置)
    - [XCode](#xcode)
    - [Homebrew 包管理工具](#homebrew-包管理工具)
    - [iTerm2](#iterm2)
    - [zsh](#zsh)
      - [安装](#安装)
      - [修改为默认的 shell](#修改为默认的-shell)
      - [安装 oh-my-zsh](#安装-oh-my-zsh)
      - [插件安装](#插件安装)
    - [Git 配置](#git-配置)
    - [Node.js 环境](#nodejs-环境)
      - [node 版本管理](#node-版本管理)
    - [VSCode](#vscode)

<!-- /TOC -->

<a id="markdown-打造高效的-mac-开发环境" name="打造高效的-mac-开发环境"></a>

# 打造高效的 Mac 开发环境

<a id="markdown-通用设置" name="通用设置"></a>

## 通用设置

**系统更新**
左上角 apple 图标 -> 【关于本机】-> 【软件更新】。

**通用配置**
左上角 apple 图标 -> 【偏好设置】-> 【通用】。可以修改主题为 Dark。

**Dock**
可以在左上角 apple 图标 -> 【偏好设置】-> 【Dock】对 Dock 进行相关修改。如勾选【Automatically hide and show the Dock】。

**安全设置**
apple 图标 -> 【偏好设置】-> 【Security & Privacy】。可以修改密码。

**键盘设置**
apple 图标 -> 【偏好设置】-> 【Keyboard】。为了利于 vim 操作，在【Modifier Keys】中将【control 键】与【caps lock】互换位置。

**触摸板 Trackpad**
根据自己系统勾选。我默认全勾选。

**App store**
申请账号并登陆 AppId。

<a id="markdown-软件安装" name="软件安装"></a>

## 软件安装

<a id="markdown-基础软件" name="基础软件"></a>

### 基础软件

- [Chrome 浏览器](https://www.google.com/chrome/)
- [Microsoft Office](https://www.office.com)
- [Slack](https://slack.com/)
- [Foxmail](https://www.foxmail.com/)
- [有道云笔记](https://note.youdao.com/)
- [有道翻译](https://cidian.youdao.com/multi.html)

<a id="markdown-效率工具" name="效率工具"></a>

### 效率工具

- [Alfred](https://www.alfredapp.com/) 快速搜索工具
- [cmdTap](http://www.yingdev.com/projects/cmdtap) 人物切换
- [shadowsocks](https://github.com/shadowsocks/ShadowsocksX-NG) 翻墙工具

<a id="markdown-开发环境配置" name="开发环境配置"></a>

## 开发环境配置

<a id="markdown-xcode" name="xcode"></a>

### XCode

在 App store 安装 XCode 开发套件，时间较长，可选择安装。Xcode command line tools 可以通过以下指令安装。

```
$ xcode-select --install
```

这样就可以使用一些 linux 指令。

<a id="markdown-homebrew-包管理工具" name="homebrew-包管理工具"></a>

### Homebrew 包管理工具

[Homebrew](https://brew.sh/) 是 OSX 系统的包管理工具。安装按照官网指令执行：

```
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

安装完成之后，可以执行`brew doctor`验证安装。
常用的使用方法：

```
安装包：brew install <package>
卸载：brew uninstall <package>
更新包：brew upgrade <package>
检查是否需要更新：brew outdated
查看已安装包：brew list --versions
```

<a id="markdown-iterm2" name="iterm2"></a>

### iTerm2

[iTerm2](https://www.iterm2.com/)是 Mac 上非常强大的一个终端，直接官网下载安装就可以。  
使用指令`Command + ,`进入设置窗口。

- 更改字体：【Profiles】-> 【Text】-> 【Font】
- 修改配色：【Profiles】-> 【Colors】-> 【Color Presets】

[Solarized](https://ethanschoonover.com/solarized/)确实比较流行，大家可以安装。新版的 iTerm2 已经自带了。不过还是喜欢黑色的【Tango Dark】。当然大家可以在[这里](https://github.com/mbadolato/iTerm2-Color-Schemes)查看更多的配色方案。

<a id="markdown-zsh" name="zsh"></a>

### zsh

zsh 是更强大的 shell。

<a id="markdown-安装" name="安装"></a>

#### 安装

默认已经安装了 zsh，如果没有安装，可以按一下指令安装。

```
$ brew install zsh
```

更多的安装方式，查看[这里](https://github.com/robbyrussell/oh-my-zsh/wiki/Installing-ZSH)。

<a id="markdown-修改为默认的-shell" name="修改为默认的-shell"></a>

#### 修改为默认的 shell

编辑/etc/shells，在末尾添加/usr/local/bin/zsh 或者
/bin/zsh，然后将其配置为默认的 shell。

```
$ chsh -s /usr/local/bin/zsh 或 /bin/zsh
```

<a id="markdown-安装-oh-my-zsh" name="安装-oh-my-zsh"></a>

#### 安装 oh-my-zsh

默认的 zsh 显示并不酷炫，[oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh)可以傻瓜式的配置 zsh。具体参看 oh-my-zsh 的 github 发布页。

```
$ sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

安装完成之后，会在\$HOME 目录下生成`.zshrc`文件。可以通过修改此文件，更改配置。例如修改主题配置：

```
ZSH_THEME="agnoster"
```

更多的配置请看这个[网址](https://github.com/robbyrussell/oh-my-zsh/wiki/Themes)。  
zsh 的箭头效果需要特殊字体支持，使用[Powerline 字体](https://github.com/powerline/fonts),可以下载安装。

```
# clone
$ git clone https://github.com/powerline/fonts.git --depth=1
# install
$ cd fonts
$ ./install.sh
```

<a id="markdown-插件安装" name="插件安装"></a>

#### 插件安装

**zsh-autosuggestions**  
 [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions) 可以用来完成历史指令的自动补全。具体安装参考这篇[说明](https://github.com/zsh-users/zsh-autosuggestions/blob/master/INSTALL.md)。

```
// 1. download
$ git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
// 2. configure .zshrc
plugins=(zsh-autosuggestions)
// 3. make it work
$ source ~/.zshrc
```

**autojump**
[autojump](https://github.com/wting/autojump)用来快速跳转目录。

```
$ brew install autojump
或者
$ git clone git://github.com/wting/autojump.git
$ cd autojump
$ ./install.py or ./uninstall.py
```

然后安装提示在.zshrc 文件中加入相应指令，打开新的窗口生效。
常用的命令：

```
j foo:  跳转到包含foo的目录
jc bar： 跳转到子目录
jo music： 打开子目录
jco images： 跳转病打开子目录
```

**zsh-syntax-highlighting**
[zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting) 用来做指令高亮。错误指令红色高亮，正确指令绿色高亮。

```
$ brew install zsh-syntax-highlighting
$ source /usr/local/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
```

更多安装方法，查看[安装说明](https://github.com/zsh-users/zsh-syntax-highlighting/blob/master/INSTALL.md)。

<a id="markdown-git-配置" name="git-配置"></a>

### Git 配置

系统默认已经安装了 git，可以验证是否安装。

```
$ git version
```

基本设置

```
# create your username
$ git config --global user.name "Dennis Ge"

# create your email
$ git config --global user.email "dennis.ge@prometheanworld.com"
```

可以添加一下常用的重命名，例如 pull，push。在.zshrc 中添加以下语句：

```shell
alias push='git push origin $(git rev-parse --abbrev-ref HEAD)'
alias pull='git pull --rebase origin $(git rev-parse --abbrev-ref HEAD)'
```

为了规范 commit message, 我通常习惯使用 [commitzen](https://github.com/commitizen/cz-cli)。安装请参考官网，或按如下步骤：

```
$ npm i -g commitzen
$ npm install -g cz-conventional-changelog
$ echo '{ "path": "cz-conventional-changelog" }' > ~/.czrc
```

安装完成，可是使用`git cz`代替`git commit`指令。

<a id="markdown-nodejs-环境" name="nodejs-环境"></a>

### Node.js 环境

<a id="markdown-node-版本管理" name="node-版本管理"></a>

#### node 版本管理

通常使用 [nvm](https://github.com/creationix/nvm) 来管理安装的 node 版本。直接下载安装：

```
$ curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash
```

node 将下载到\$HOME 下的`.nvm目录`，并在`.zshrc`中加入以下指令。

```
$ export NVM_DIR="${XDG_CONFIG_HOME/:-$HOME/.}nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm
```

`source ~/.zshrc`生效。常用的指令如下：

```
安装制定版本：nvm install 版本
使用制定版本：nvm use 版本
查看已安装的版本：nvm ls
查看远端版本：nvm ls-remote
重命名：nvm alias
```

<a id="markdown-vscode" name="vscode"></a>

### VSCode

VSCode 的安装配置, 请参考我的一篇 [配置一个适合自己的 VSCode 环境](./20190416-setup-own-vscode.md)。

至此，基本搭建好了一个比较好用的开发环境了，愉快的编程吧！
