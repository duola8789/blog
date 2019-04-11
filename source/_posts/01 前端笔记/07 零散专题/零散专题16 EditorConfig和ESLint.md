---
title: 零散专题16 EditorConfig和ESLint
top: false
date: 2017-12-05 10:33:30
updated: 2019-03-21 09:12:28
tags:
- EditorConfig
- ESLint
- WebStorm
categories: 其他
---

编程好帮手：EditorConfig + ESLint

<!-- more -->

## EditorConfig

为了保持项目中代码缩进风格的一致，可以使用EditorConfig来定义和维护一致的编码风格，例如范缩进风格，缩进大小，Tab长度以及字符集等。

标准的`.editorconfig`文件中，第一行是EditorConfig的官网，第二行是用于指明`.editorconfig`文件的位置，*代表通配符，下面是对应的规则，现在项目中使用的配置文件是这样的：

```BASH
root = true

[*]
#缩进风格：空格
indent_style = space

#缩进大小2
indent_size = 2

#字符集utf-8
charset = utf-8

#行尾允许空格
trim_trailing_whitespace = true

#结尾总是插入新的一行
insert_final_newline = true

[*.md]
trim_trailing_whitespace = false

```
没有加入的属性还有：

```BASH
#换行符lf
end_of_line = lf
```

## ESLint

[Eslint](https://eslint.org/)是一个用来识别ECMAScript并且按照规则给出报告的代码检测工具，使用它可以避免低级错误和统一代码的风格。ESLint被设计为完全可配置的，一般使用配置文件来配置ESLint，在项目的根目录下创建`.eslintrc.js`

可以被配置的信息主要分为3类：

- Environments：你的Javascript脚本将要运行在什么环境（如：Node，浏览器等）中。
- Globals：执行代码时脚步需要访问的额外全局变量。
- Rules：开启某些规则，也可以设置规则的等级。

### 安装

安装推荐局部安装：

```BASH
npm i -D eslint
```
安装完毕后，接下来新建一个配置文件`.eslintrc.js`，或者使用如下的命令行来自动生成。

```BASH
./node_modules/.bin/eslint --init

# 如果是全局安装的话可以执行
eslint --init
```
配置文件中的[配置规则](https://eslint.org/docs/user-guide/configuring)分为三种等级：

- off" 或者 0：关闭规则。
- "warn" 或者 1：打开规则，并且作为一个警告（不影响exit code）。
- "error" 或者 2：打开规则，并且作为一个错误（exit code将会是1）。

```JS
// .eslintrc.js
module.exports = {
  env: {
    browser: true,
    node: true,
  },
};
```
具体的配置规则非常多，具体见[这里](https://eslint.org/docs/rules/)。

### 在Webstorm中使用

在Webstorm中配置ESLint，需要在`Language & Frameworks` → `Javascript` → `Code Quality Tools` → `ESLint`中进行配置，勾选`Enable`

一般情况下下面的对应的配置和地址都会自动填好。

### AlloyTeam ESLint规则

也可以使用腾讯的[AlloyTeam的ESLint规则](https://github.com/AlloyTeam/eslint-config-alloy)，有[详细的规则描述和示例](https://github.com/AlloyTeam/eslint-config-alloy)，可以在此基础上进行定制。

使用时需要首先安装`eslint`、`babel-eslint`、`eslint-config-alloy`，这是AlloyTeam的ESLint规则的最基础的所需要安装的依赖

```BASH
npm install --save-dev eslint babel-eslint eslint-config-alloy
```
然后在项目根目录下的`eslintrc.js`中复制以下内容：

```JS
module.exports = {
  extends: [
    'eslint-config-alloy',
  ],
  globals: {
    // 这里填入你的项目需要的全局变量
    // 这里值为 false 表示这个全局变量不允许被重新赋值，比如：
    //
    // jQuery: false,
    // $: false
  },
  rules: {
    // 这里填入你的项目需要的个性化配置，比如：
    //
    // // @fixable 一个缩进必须用两个空格替代
    // 'indent': [
    //     'error',
    //     2,
    //     {
    //         SwitchCase: 1,
    //         flatTernaryExpressions: true
    //     }
    // ]
  }
};
```
### 配合Vue使用

可以在使用AlloyTeam预置的Vue的规则：

```BASH
npm install eslint 
            babel-eslint 
            vue-eslint-parser@2.0.1-beta.2 
            eslint-plugin-vue@3 
            eslint-config-alloy --save-dev 
```

然后将`.eslintrc.js`中的`extends`的值更改为`['eslint-config-alloy/vue']`

下面是在CRM系统中使用的，一份配合Vue使用的配置文件：

```JS
module.exports = {
  "env": {
    "browser": true,
    "amd": true,
    "jquery": true
  },
  "rules": {
    // 禁止出现alert, prompt 和 confirm, 可以在发布的时候进行检测
    "no-alert": 2,
    // 禁止出现console, 可以在发布的时候进行检测
    "no-console": 2,
    // 禁止出现debugger, 可以在发布的时候进行检测
    "no-debugger": 2,
    // 禁止不写分号
    "semi": [2, "always"],
    // 禁止出现tab之外的缩进
    "indent": [2, "tab"],
    // 允许定义前使用
    "no-use-before-define": 0,
    // 允许if (!!foo) 这种形式
    "no-extra-boolean-cast": 0,
    // 允许对函数声明进行覆盖赋值
    "no-func-assign": 0,
    // 允许使用caller或callee
    "no-caller": 0,
    // 允许函数在不同的情况下返回不同类型的值
    "consistent-return": 0,
    // 允许在switch的case中不加break
    "no-fallthrough": 0,
    // 允许使用__proto__
    "no-proto": 0,
    // 允许覆盖外部变量
    "no-shadow": 0,
    // 允许文件的最后一行不是空白行
    "eol-last": 0,
    // 允许使用下划线开头命名变量
    "no-underscore-dangle": 0,
    // 建议将操作符放到行尾, 而不是行首
    "operator-linebreak": [1, "after"],
    // 建议使用已定义的变量
    "no-undef": 1,
    // 建议return语句中不要包含赋值表达式
    "no-return-assign": 1,
    // 建议代码列数不能超过120行
    "max-len": [1, 120],
    // 建议启用严格模式
    "global-strict": 0,
    "strict": 1,
    // 建议使用单引号
    "quotes": [1, "single"],
    // 建议注释符要有空白隔开
    "spaced-comment": [1, "always"]
  },
  // 如果使用vue单文件组件
  "extends": [
    "plugin:vue/essential"
  ],
}
```
### 配合React使用

同样可以在使用AlloyTeam预置的React的规则：

```同样可以
npm install eslint 
            babel-eslint 
            eslint-plugin-react 
            eslint-config-alloy  --save-dev
```
然后将`.eslintrc.js`中的`extends`的值更改为`['eslint-config-alloy/react']`。

在Exam项目中使用的一份很简单的配置文件：

```JS
module.exports = {
  extends: [
    'plugin:react/recommended',
  ],
  "ecmaFeatures": {
    "jsx": true,
    "modules": true
  },
  "env": {
    "browser": true,
    "node": true
  },
  "parser": "babel-eslint",
  "rules": {
    "quotes": [2, "single"],
    "react/jsx-uses-react": 2,
    "react/jsx-uses-vars": 2,
    "react/react-in-jsx-scope": 2
  },
  "plugins": [
    "react"
  ]
}
```
需要注意的是，如果使用的Create React App脚手架工具来搭建React项目，由于它将默认的构建配置封装了起来，而ESLint仅仅开启了最基本的规则，更重要的是默认情况下，ESLint仅仅会在IDE中对违反规则的情况进行提示，并不会在构建时在终端的输出进行终端和提示。

如果这种情况可以满足需要，而只需要开启更多的规则，那么就可以在根目录下新建一个文件`.eslintrc.json`，然后添加：

```JS
{
  "extends": "react-app"
}
```
但是如果要起到更强制性的提示作用（中断构建、终端提示），Create React App建议使用[Prettier](https://github.com/prettier/prettier)代替ESLint。如果要使用ESLint，那么就需要使用`npm run eject`，将配置文件吐出，按照AlloyTeam的提示进行配置即可。

### ESlint对`Async`报错的解决方法

在ESlint配置文件中增加

```BASH
parserOptions: {
  "ecmaVersion": 8,
}
```


## 参考

- [rules@ESLint](https://eslint.org/)
- [在WebStorm中使用editorConfig插件@CSDN](https://blog.csdn.net/itpinpai/article/details/53887818)
- [ESLint与EditorConfig@CSDN](https://blog.csdn.net/qq_35809834/article/details/72758528)
- [ESLint - 简介@简书](https://www.jianshu.com/p/2bcdce1dc8d4)
- [Displaying Lint Output in the Editor@Create React App](https://facebook.github.io/create-react-app/docs/setting-up-your-editor#displaying-lint-output-in-the-editor)
- [AlloyTeam/eslint-config-alloy@github](https://github.com/AlloyTeam/eslint-config-alloy)
