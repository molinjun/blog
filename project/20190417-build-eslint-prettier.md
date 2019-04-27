<!-- TOC -->

- [使用 ESLint 和 Prettier 规范代码](#使用-eslint-和-prettier-规范代码)
  - [ESLint](#eslint)
    - [安装使用](#安装使用)
      - [本地安装](#本地安装)
      - [全局安装](#全局安装)
    - [配置](#配置)
      - [配置方式](#配置方式)
      - [配置项](#配置项)
    - [集成 VSCode](#集成-vscode)
  - [Prettier](#prettier)
    - [安装使用](#安装使用-1)
    - [配置](#配置-1)
    - [ESLint 与 Prettier 集成](#eslint-与-prettier-集成)
      - [1. 禁用 ESLint 中格式化相关的规则](#1-禁用-eslint-中格式化相关的规则)
      - [2. 使用 ESLint 来运行 Prettier](#2-使用-eslint-来运行-prettier)
      - [推荐配置](#推荐配置)
  - [最佳实践](#最佳实践)
    - [启用 ESLint](#启用-eslint)
    - [启用 Prettier](#启用-prettier)
    - [参考配置](#参考配置)
  - [参考](#参考)

<!-- /TOC -->

<a id="markdown-使用-eslint-和-prettier-规范代码" name="使用-eslint-和-prettier-规范代码"></a>

# 使用 ESLint 和 Prettier 规范代码

JavaScript 是一种动态的弱类型语言。它的语法特性非常灵活，但没有约束同时也带来了很多头疼的问题。不容易捕捉的运行时错误，总是不让人放心。另外，团队的合作开发，成员代码质量的高低以及不同的代码风格，都给代码审查以及项目的质量带来很大的问题。所以，必须有效地制定规则和约定，来规范项目的代码。  
[ESLint](https://cn.eslint.org/) 就是这样一个 JavaScript 的代码检查工具。它的目的就是保证代码的一致性和避免错误。它可以非常方便的集成到编辑器中，同时支持个性化定制自己的规则。 但是，ESLint 只会帮我们提示错误，我们需要另外一个工具帮我们根据规则自动修复一部分错误。 [Prettier](https://prettier.io/) 就是我们需要的一款代码格式化工具。下面我们将介绍这两款工具的基本使用，以及如何集成到我们的编辑器中。

> 如果不想关心细节，可以直接查看篇尾的【最佳实践】章节完成配置。

<a id="markdown-eslint" name="eslint"></a>

## ESLint

ESLint 是一个开源的 JavaScript 代码检查工具，由 Nicholas C. Zakas 于 2013 年 6 月创建，使用 Node.js 编写。ESLint 的所有规则都被设计成可插入的，所以可以非常灵活的制定适合自己的检测规则。

<a id="markdown-安装使用" name="安装使用"></a>

### 安装使用

首先确保系统安装了 Node.js 和 npm。可以在本地安装或者全局安装。

<a id="markdown-本地安装" name="本地安装"></a>

#### 本地安装

如果想让 ESLint 成为构建系统的一部分，建议本地安装。

```
$ npm i -D eslint
```

此时 eslint 指令并不能工作。

```
$ ./node_modules/.bin/eslint app.js

Oops! Something went wrong! :(

ESLint couldn't find a configuration file. To set up a configuration file for this project, please run:

    eslint --init

ESLint looked for configuration files in /Users/dennis/learn/demo and its ancestors.

If you think you already have a configuration file or if you need more help, please stop by the ESLint chat room: https://gitter.im/eslint/eslint
```

要使用 `eslint --init` 来创建一个配置文件。`eslint init` 需要有 package.json 文件。如果没有，首先使用`npm init`创建。

```
$ ./node_modules/.bin/eslint --init
```

可以根据提示常见自己的配置。

```
$ ./node_modules/.bin/eslint --init
? How would you like to use ESLint? To check syntax, find problems, and enforce code style
? What type of modules does your project use? JavaScript modules (import/export)
? Which framework does your project use? None of these
? Where does your code run? Node
? How would you like to define a style for your project? Answer questions about your style
? What format do you want your config file to be in? JSON
? What style of indentation do you use? Spaces
? What quotes do you use for strings? Single
? What line endings do you use? Unix
? Do you require semicolons? Yes
? What format do you want your config file to be in? JSON
Successfully created .eslintrc.json file in /Users/dennis/learn/demo
```

当然大家不要完全依赖提示来创建，你可以根据需求再次修改生成的配置。接下来，我们就可以对我们的文件进行 lint 校验了。

```
$ ./node_modules/.bin/eslint yourfile.js
```

如果有 lint 错误，可以 fix 部分错误。

```
$ ./node_modules/.bin/eslint --fix demo.js
```

> 注意：使用本地的 ESLint，使用的插件和配置都需要本地安装。

使用上面的命令还是很不方便，我们可以配置我们的 npm 指令。在 package.json 中加入指令。

```
 "scripts": {
    "lint": "eslint *.js --fix"
  }
```

这样就可以通过 npm run lint 来 lint 代码了。

```
$ npm run lint
```

<a id="markdown-全局安装" name="全局安装"></a>

#### 全局安装

如果想使 ESLint 使用于所有的项目，建议全局安装。安装和本地安装类似。

```shell
$ npm i -g eslint
$ eslint --init
$ eslint yourfile.js
```

全局安装的 ESLint，使用的任何插件或配置都必须全局安装。

<a id="markdown-配置" name="配置"></a>

### 配置

<a id="markdown-配置方式" name="配置方式"></a>

#### 配置方式

ESLint 有两种配置方式：  
**1. Configuration Comments：在注释中加入配置**  
这种方式不常用，可以用来在某些场景需要对某个文件或者某一代码行做特殊处理。

文件头尾使用：

```
/* eslint-disable no-console */

console.log('bar');

/* eslint-enable no-console */
```

针对特定语句:

```
// eslint-disable-next-line
alert('foo');

/* eslint-disable-next-line */
alert('foo');

alert('foo'); /* eslint-disable-line */
```

**2. Configuration Files：使用配置文件**  
使用 JavaScript、JSON 或者 YAML 文件为整个目录和它的子目录指定配置信息。可以配置一个独立的 `.eslintrc.*`文件，或者直接在 `package.json` 文件里的 `eslintConfig` 字段指定配置，推荐使用。目前 `.eslintrc` 文件格式将被废弃，所以可以选择 `.eslintrc.json`。

<a id="markdown-配置项" name="配置项"></a>

#### 配置项

**Environments**  
指定脚本的运行环境。每种环境都有一组特定的预定义全局变量。常见的如下：

```
browser - 浏览器环境中的全局变量。
node - Node.js 全局变量和 Node.js 作用域。
es6 - 启用除了 modules 以外的所有 ECMAScript 6 特性（该选项会自动设置 ecmaVersion 解析器选项为 6）。
jest - Jest 全局变量。
```

**Globals**  
脚本在执行期间访问的额外的全局变量。如果你想在一个源文件里使用全局变量，推荐你在 ESLint 中定义这些全局变量，这样 ESLint 就不会发出警告了。  
要在你的 JavaScript 文件中，用注释指定全局变量，格式如下：

```
/* global var1:false, var2:false */
```

定义全局变量 `var1` 和 `var2`。var1 允许被重写，var2 不允许被重写。
在配置文件里配置全局变量时

```
{
    "globals": {
        "var1": true,
        "var2": false
    }
}
```

**Parser Options**  
指定 JavaScript 的语言选项。默认情况下，ESLint 支持 ECMAScript 5 语法。

```
{
    "parserOptions": {
        "ecmaVersion": 6,
        "sourceType": "module",
        "ecmaFeatures": {
            "jsx": true
        }
    },
    "rules": {}
}
```

**Parser**  
ESLint 默认使用 Espree 作为其解析器，你可以在配置文件中指定一个不同的解析器。

**Rules 启用的规则及其各自的错误级别**  
Rules 用来定义一些规则。Eslint 有很多默认的规则，这个不用全部去记，只需要使用的过程中逐个去了解就可以了。可以参考[官网规则](http://eslint.cn/docs/rules/)。
每个规则可以定义三种错误级别：

- "off" 或 0 - 关闭规则
- "warn" 或 1 - 开启规则，使用警告级别的错误：warn (不会导致程序退出)
- "error" 或 2 - 开启规则，使用错误级别的错误：error (当被触发的时候，程序会退出)

可以直接定义规则的等级，有些规则支持一些额外选项。如下：

```
//定义等级
'class-methods-use-this': 0,
// 支持选项
'no-unused-expressions': [2, {
    allowShortCircuit: true,
    allowTernary: true
}]
```

第一个参数为错误等级。第二个参数为可选项。

**extends**  
用来继承一些通用的配置，如官网的`eslint-recommended`。也可以使用 [AirBnB](https://github.com/airbnb/javascript) 规范。AirBnB 会相对比较严格一些，可以根据自己的需求选择。AirBnB 可以根据自己是否使用 react 和 jsx 来选择使用[通用版](https://www.npmjs.com/package/eslint-config-airbnb)或者 [airbnb-base](https://www.npmjs.com/package/eslint-config-airbnb-base) 版。
使用通用版需要安装一下依赖。

```
$ npm i -D eslint-plugin-react
$ npm i -D eslint-plugin-jsx-a11y
```

**Plugin 第三方插件**  
插件是一个 npm 包，通常输出规则。一些插件也可以输出一个或多个命名的配置。使用插件之前，需要 npm 安装。配置文件中的插件名称可以省略 `eslint-plugin-` 前缀。

```
{
    "plugins": [
        "plugin1",
        "eslint-plugin-plugin2"
    ]
}
```

extends 属性值可以由以下组成：

- plugin:
- 包名 (省略了前缀，比如，react)
- /
- 配置名称 (比如 recommended)

```
{
    "plugins": [
        "react"
    ],
    "extends": [
        "eslint:recommended",
        "plugin:react/recommended"
    ],
    "rules": {
       "no-set-state": "off"
    }
}
```

<a id="markdown-集成-vscode" name="集成-vscode"></a>

### 集成 VSCode

ESLint 可以非常容易地集成到各种编辑器。这里介绍一下集成到 VSCode，别的编辑器请参看[官网](https://cn.eslint.org/docs/user-guide/integrations)。  
VSCode 提供了 ESLint 的[插件](https://github.com/Microsoft/vscode-eslint)，在 VSCode 安装即可。安装完成后，VSCode 会查找最近安装的 ESLint 和配置文件来 lint 代码，并显示错误的格式。

<a id="markdown-prettier" name="prettier"></a>

## Prettier

[Prettier](https://prettier.io/) 是一款代码格式化工具。Prettier 和 ESLint 是两种不同功能的工具，不要混淆，可以参考[这里](https://prettier.io/docs/en/comparison.html)。

<a id="markdown-安装使用-1" name="安装使用-1"></a>

### 安装使用

具体请参考[安装说明](https://prettier.io/docs/en/install.html)。

```
npm install --save-dev --save-exact prettier
# or globally
npm install --global prettier
```

如果我们在 VSCode 中使用，需要在 VSCode 安装 [prettier-vscode](https://github.com/prettier/prettier-vscode) 插件。安装完成后，可以使用指令格式化代码。

```
1. CMD + Shift + P -> Format Document
OR
1. Select the text you want to Prettify
2. CMD + Shift + P -> Format Selection
```

当然，也可以修改 `editor.formatOnSave` 配置，在文件保存时，运行 format。

```
// Set the default
"editor.formatOnSave": false,
// Enable per-language
"[javascript]": {
    "editor.formatOnSave": true
}
```

<a id="markdown-配置-1" name="配置-1"></a>

### 配置

Prettier 的配置通常放在 `.prettierrc`文件中，当然也支持 json、toml 等文件格式。更多的配置说明请看[这里](https://prettier.io/docs/en/configuration.html)。
常见的一些配置如下：

```
{
  "printWidth": 80,        // 每行字符数
  "trailingComma": "es5",  // 是否允许尾部的逗号
  "tabWidth": 4,           // 每个缩进的空格数
  "semi": true,            // 是否需要分号
  "singleQuote": true      // 单引号或双引号
}
```

<a id="markdown-eslint-与-prettier-集成" name="eslint-与-prettier-集成"></a>

### ESLint 与 Prettier 集成

Prettier 可以集成到 ESLint 中，这样就可以使用 Prettier 来负责格式化相关问题，同时 ESLint 来负责检测代码的错误语法问题。  
将 Prettier 集成到 ESLint 主要就两个步骤：

<a id="markdown-1-禁用-eslint-中格式化相关的规则" name="1-禁用-eslint-中格式化相关的规则"></a>

#### 1. 禁用 ESLint 中格式化相关的规则

ESLint 中有些格式化规则可能与定义的 Prettier 规则有冲突。为了确保格式化规则全由 Prettier 决定，我们需要禁用 ESLint 中的相关规则。
[eslint-config-prettier ](https://github.com/prettier/eslint-config-prettier) 可以用来完成这个功能。首先安装这个依赖：

```
$ npm i -D eslint-config-prettier
```

然后在 `.eslintrc.json` 中`extends` 数组的最尾添加 `prettier`。

```
{
  "extends": [
    "some-other-config-you-use",
    "prettier"
  ]
}
```

<a id="markdown-2-使用-eslint-来运行-prettier" name="2-使用-eslint-来运行-prettier"></a>

#### 2. 使用 ESLint 来运行 Prettier

使用 [eslint-plugin-prettier](https://github.com/prettier/eslint-plugin-prettier) 来将 Prettier 的格式化内容，作为一个规则（rule）加到 ESLint。

```
$ npm i -D eslint-plugin-prettier
```

修改 `.eslintrc.json`。

```
{
  "plugins": ["prettier"],
  "rules": {
    "prettier/prettier": "error"
  }
}
```

<a id="markdown-推荐配置" name="推荐配置"></a>

#### 推荐配置

eslint-plugin-prettier 提供一个 `recommended` 配置项，来一步完成上述配置。

```
$ npm i -D eslint-config-prettier eslint-plugin-prettier
```

然后在 `.eslintrc.json` 中修改。

```
{
  "extends": ["plugin:prettier/recommended"]
}
```

<a id="markdown-最佳实践" name="最佳实践"></a>

## 最佳实践

本方法适用于 VSCode 的场景。大家可以设计适合自己的场景。

<a id="markdown-启用-eslint" name="启用-eslint"></a>

### 启用 ESLint

- 本地目录安装 ESLint。不要全局安装，减少复杂度。

```
npm i -D eslint
```

- VSCode 安装 [ESLint 插件]()。
- 添加 `.eslintrc.json`配置文件  
  如果没有 `package.json` 先 `npm init` 生成。

```
./node_modules/.bin/eslint --init
```

可以根据提示生成适合自己的配置。最好根据不同的项目设置不同的配置，例如前端 React 项目可能要启用 React 插件等。

<a id="markdown-启用-prettier" name="启用-prettier"></a>

### 启用 Prettier

- 本地安装 Prettier

```
$ npm install --save-dev --save-exact prettier
```

- VSCode 安装 Prettier 插件。
- 创建 `.prettierrc` 文件添加配置。
- Prettier 集成到 ESLint。  
  安装 [eslint-config-prettier ](https://github.com/prettier/eslint-config-prettier) 覆盖 ESLint 的格式化配置。 安装 [eslint-plugin-prettier](https://github.com/prettier/eslint-plugin-prettier) 并把 Prettier 设置成 ESLint 的 rule。

```
npm i -D eslint-config-prettier eslint-plugin-prettier
```

修改 `.eslintrc.json` 配置。

```
{
  "extends": ["plugin:prettier/recommended"]
}
```

- 设置 `editor.formatOnSave`配置，在文件保存时，运行 format。

<a id="markdown-参考配置" name="参考配置"></a>

### 参考配置

`.eslintrc.json` 文件如下：

```
{
  "env": {
    "es6": true,
    "node": true
  },
  "extends": ["eslint:recommended", "prettier"],
  "globals": {
    "Atomics": "readonly",
    "SharedArrayBuffer": "readonly"
  },
  "parserOptions": {
    "ecmaVersion": 2018,
    "sourceType": "module"
  },
  "rules": {
    "linebreak-style": ["error", "unix"],
    "quotes": ["error", "single"],
    "semi": ["error", "always"],
    "prettier/prettier": "error"
  }
}
```

`.prettierrc` 文件如下：

```
{
  "trailingComma": "all",
  "tabWidth": 2,
  "semi": true,
  "printWidth": 80,
  "singleQuote": true
}
```

<a id="markdown-参考" name="参考"></a>

## 参考

[1][eslint官网](https://cn.eslint.org/)  
[2][prettier官网](https://prettier.io/)  
[3][integrating prettier + eslint + airbnb style guide in vscode](https://blog.echobind.com/integrating-prettier-eslint-airbnb-style-guide-in-vscode-47f07b5d7d6a)  
[4][write cleaner code using prettier and eslint in vscode](https://medium.com/@pgivens/write-cleaner-code-using-prettier-and-eslint-in-vscode-d04f63805dcd)  
[5][深入浅出eslint——关于我学习eslint的心得](https://juejin.im/post/5bab946cf265da0ae92a75ca)
