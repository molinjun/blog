<!-- TOC -->

- [设置一个适合自己的 VSCode 环境](#设置一个适合自己的-vscode-环境)
  - [安装](#安装)
  - [按键映射](#按键映射)
    - [按键重映射](#按键重映射)
    - [多光标模式](#多光标模式)
    - [vim-airline](#vim-airline)
    - [vim-easymotion](#vim-easymotion)
    - [vim-surround](#vim-surround)
  - [主题](#主题)
  - [基础操作](#基础操作)
    - [窗口操作](#窗口操作)
    - [编辑](#编辑)
  - [常用插件](#常用插件)
  - [卸载 VSCode](#卸载-vscode)

<!-- /TOC -->

<a id="markdown-设置一个适合自己的-vscode-环境" name="设置一个适合自己的-vscode-环境"></a>

# 设置一个适合自己的 VSCode 环境

[Visual Studio Code(VSCode)](https://code.visualstudio.com/) 是一款微软出的代码编辑器。它免费、轻便而且功能强大。你可以从扩展市场下载各种有用的插件，定制一款适合自己的编辑器。

> 截止到本篇文章发布时，官网的版本为 1.33。行文中的操作可能会随着版本的更新有细微差异，具体以[官网](https://code.visualstudio.com/)为准。

<a id="markdown-安装" name="安装"></a>

## 安装

VSCode 的安装非常简单。进入[官网下载页](https://code.visualstudio.com/download)，根据自己的系统，选择相应的版本下载安装。具体的安装可以参考[官网安装文档](https://code.visualstudio.com/docs/setup/setup-overview)。  
安装完成后，可以配置 VSCode 在命令行启动。通过使用`code .`来打开当前目录。打开 VSCode，运行命令`shift+cmd+p`打开命令面板，输入`shell command`,安装 code 到 PATH。
具体参见[官网 mac 安装](https://code.visualstudio.com/docs/setup/mac)。

<a id="markdown-按键映射" name="按键映射"></a>

## 按键映射

VSCode 支持很多种按键映射，习惯其他编辑器的同学可以根据选择相应的插件。我是使用 vim 操作较多，需要安装插件[VSCodeVim](https://marketplace.visualstudio.com/items?itemName=vscodevim.vim)。在菜单栏【Code】 > 【Preferences】 > 【Keymaps】下，点击 vim 安装，重载生效。

> VSCodeVim 是一个在 VSCode 环境下的 vim 模拟器，并不支持 vimscript。所以主机上的.vimrc 配置和 plugin 都不可用。

Mac 环境下连续按键(key repeating)有点问题，可以执行以下指令解决。

```
$ defaults write com.microsoft.VSCode ApplePressAndHoldEnabled -bool false         # For VS Code
$ defaults write com.microsoft.VSCodeInsiders ApplePressAndHoldEnabled -bool false # For VS Code Insider
$ defaults delete -g ApplePressAndHoldEnabled                                      # If necessary, reset global default
```

由于 VSCode 的 vim 只是一个模拟器，所以没有原生 vim 那么强大和丰富的插件。不过 VSCodeVim 可以通过重映射相关按键，并且集成了几个不错的 vim 插件。

<a id="markdown-按键重映射" name="按键重映射"></a>

### 按键重映射

我比较常用的几个重映射就是 leader 键的设置，以及通过 jk 替代 ESC,退出编辑模式。这里不过多介绍。以下是我常用的 vim 相关配置：

```
    "vim.easymotion": true,
    "vim.sneak": true,
    "vim.incsearch": true,
    "vim.useSystemClipboard": true,
    "vim.useCtrlKeys": true,
    "vim.hlsearch": true,
    "vim.insertModeKeyBindings": [
        {
            "before": [ "k", "j" ],
            "after": [ "<Esc>" ]
        }
    ],
    "vim.normalModeKeyBindingsNonRecursive": [
        {
            "before": [ "<leader>", "w" ],
            "commands": [ ":w" ]
        },
        {
            "before": [ "<C-n>" ],
            "commands": [ ":nohl" ]
        },
        {
            "before": [ ">" ],
            "commands": [ "editor.action.indentLines" ]
        },
        {
            "before": [ "<" ],
            "commands": [ "editor.action.outdentLines" ]
        },
    ],
    "vim.leader": ",",
    "vim.handleKeys": {
        "<C-a>": false,
        "<C-f>": false
    }
```

<a id="markdown-多光标模式" name="多光标模式"></a>

### 多光标模式

多光标模式在一些多行编辑的场景非常好用。通过【cmd/ctrl + d】进入多行编辑模式，通过【ga】在下一匹配单词处增加光标。点击【ESC】进入多光标 normal 模式。再次点击【ESC】退出多光标，进入 normal 模式。

<a id="markdown-vim-airline" name="vim-airline"></a>

### vim-airline

设置状态栏颜色。

```
 "vim.statusBarColorControl": true,
 "vim.statusBarColors.normal": ["#8FBCBB", "#434C5E"],
 "vim.statusBarColors.insert": "#BF616A",
 "vim.statusBarColors.visual": "#B48EAD",
 "vim.statusBarColors.visualline": "#B48EAD",
 "vim.statusBarColors.visualblock": "#A3BE8C",
 "vim.statusBarColors.replace": "#D08770"
```

<a id="markdown-vim-easymotion" name="vim-easymotion"></a>

### vim-easymotion

单词的快捷跳转。

```
<leader><leader>s<搜索字母> 向前搜索
<leader><leader>f<搜索字母> 向后搜索
```

操作之后，匹配的字母会高亮，按相应字母完成跳转。
我常用这两个，[VSCodeVim 文档](https://marketplace.visualstudio.com/items?itemName=vscodevim.vim)有更多操作，可以去查看。

<a id="markdown-vim-surround" name="vim-surround"></a>

### vim-surround

括号的操作。

```
d s  <现有的括号类型>  删除括号
c s <现有括号><更改括号类型>  修改括号类型
ysaw <更改的括号类型> 添加括号，aw是整个单词
S <更改的括号类型>  visual模式操作，添加括号
```

<a id="markdown-主题" name="主题"></a>

## 主题

在 VSCode 界面，Code -> Preferences -> Color Theme，打开主题选择器，通过上下键选择主题，界面会实时预览，按下 ENTER 键选中。我选择【monokai】和我的 vim 主题一致。也可以从插件库安装相应主题。打开插件界面`shift+cmd+x`，输入`theme`,会列出丰富的插件，选择安装。

```
// 字体
"editor.fontFamily": "Roboto Mono Light for Powerline"
"editor.fontSize": 12,
// tab设置
"editor.tabSize": 2,
// 关闭minimap
"editor.minimap.enabled": false
```

<a id="markdown-基础操作" name="基础操作"></a>

## 基础操作

<a id="markdown-窗口操作" name="窗口操作"></a>

### 窗口操作

```
打开问题面板：ctrl/cmd + shift + m
侧边栏打开关闭： ctrl/cmd + b
分屏： ctrl/cmd + \
分屏切换：ctrl/cmd + 1/2/3
打开 console 面板： cmd + j
```

<a id="markdown-编辑" name="编辑"></a>

### 编辑

```
快速打开：cmd/ctrl + p
```

<a id="markdown-常用插件" name="常用插件"></a>

## 常用插件

VSCode [插件市场](https://marketplace.visualstudio.com/)有各种丰富的插件提供下载。

- [Settings Sync](https://marketplace.visualstudio.com/items?itemName=Shan.code-settings-sync) 使用 github gist 来同步多台设备的配置。非常好用。
- [Auto Close Tag](https://marketplace.visualstudio.com/items?itemName=formulahendry.auto-close-tag) 自动关闭匹配的标签。对于 HTML 很实用。
- [Auto Rename Tag](https://marketplace.visualstudio.com/items?itemName=formulahendry.auto-rename-tag) 同步修改开闭合标签。
- [GitLens](https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens) Git blame 注解
- [ESLint]() JavaScript 语法检查工具。
- [Prettier]() 格式化工具。
- [setting sync]() 同步 VSCode 配置到 github gist。

这里特别要说的是，对于代码的规范和格式化非常重要。所以对于 ESLint 和 Prettier 的安装， 请查看 [使用 ESLint 和 Prettier 规范代码](./project/20190417-build-eslint-prettier.md)

<a id="markdown-卸载-vscode" name="卸载-vscode"></a>

## 卸载 VSCode

Mac 下彻底卸载 VSCode。

```
1. rm -rf $HOME/Library/Application\ Support/Code
2. rm -rf $HOME/.vscode
3. Application 里删掉.app
```
