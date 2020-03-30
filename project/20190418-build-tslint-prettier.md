# 使用 TSLint 和 Prettier 规范 TypeScript 项目

上一篇[《使用 ESLint 和 Prettier 规范代码》](./20190417-build-eslint-prettier.md)我们介绍了对 JavaScript 项目的 lint 配置，这一篇我们介绍一下 TypeScript 项目的格式化配置。具体的步骤和上文基本一致，所以这里不说太多理论，直接介绍步骤。

> 建议先看上一篇。

## 创建 TypeScript 项目

对已有的项目，可以直接安装 TSLint。新建 TypeScript 项目可以按如下步骤。

### 创建一个项目

```
$ mkdir ts-demo
$ cd ts-demo
$ npm init -y
```

### 配置 TypeScript

```
# Install TypeScript locally
$ npm install -D typescript

# Initialise a TypeScript config file
$ node_modules/.bin/tsc --init
```

这样就生成了一个配置文件 `tsconfig.json` 。关于配置的详细信息，查看[这里](https://www.typescriptlang.org/docs/handbook/compiler-options.html)。

### VSCode 启用本地 TypeScript

VSCode 是自带 TypeScript 的。我们设置为本地安装的。

- 创建一个 `.vscode` 文件夹。
- 文件夹下创建 `.vscode/settings.json`文件。

```
{
  "typescript.tsdk": "./node_modules/typescript/lib"
}
```

- 在 VSCode 打开文件，查看状态栏确保使用的 TypeScript 版本是本地安装的。

## 启用 TSLint

TypeScript 已经对 JavaScript 做了很好的类型检查，但对一些代码规范有所欠缺。[TSLint](https://palantir.github.io/tslint/) 是 TypeScript 的标准静态代码分析工具，可以用来弥补这一不足。

- 本地目录安装 TSLint。

```
$ npm install -D tslint
```

- 添加配置文件 `.tslint.json` 配置文件。

```
$ ./node_modules/.bin/tslint --init
```

根据需求修改自己的配置，这里是全部的[规则列表](https://palantir.github.io/tslint/rules/)。

- VSCode 安装 [TSLint 插件](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-typescript-tslint-plugin)。

## 启用 Prettier

**本地安装 Prettier**

```
npm install --save-dev --save-exact prettier
```

**VSCode 安装 Prettier 插件**

**创建 `.prettierrc` 文件添加配置**

**Prettier 集成到 TSLint**  
官方说明查看[这里](https://prettier.io/docs/en/integrating-with-linters.html#tslint)。安装 [tslint-config-prettier ]() 覆盖 TSLint 的格式化配置。 安装 [tslint-plugin-prettier]() 并把 Prettier 设置成 TSLint 的 rule。

```
$ npm i -D tslint-config-prettier tslint-plugin-prettier
```

修改 `tslint.json` 配置。

```
{
  "extends": ["tslint-plugin-prettier", "tslint-config-prettier"],
  "rules": {
    "prettier": true
  }
}
```

**修改`editor.formatOnSave`配置**  
设置 `true`在文件保存时，运行 format。

## 参考配置

`tslint.json` 文件如下：

```
{
  "extends": [
    "tslint:recommended",
    "tslint-config-prettier",
    "tslint-plugin-prettier"
  ],
  "rules": {
    "prettier": true,
    "quotemark": [true, "single"]
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

## 参考

[1][tslint官网](https://palantir.github.io/tslint/)  
[2][prettier官网](https://prettier.io/)  
[3][vscode + typescript + prettier = happy developer](https://www.fiznool.com/blog/2018/12/07/vscode---typescript---prettier--happy-developer/)
