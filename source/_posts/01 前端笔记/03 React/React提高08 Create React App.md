---
title: React提高08 Create React App
top: false
date: 2019-03-01 09:38:04
update: 2019-03-21 13:51:57
tags:
- 脚手架
- Create React App
categories: React
---

一个由Facebook官方出品的React脚手架工具，无需额外配置，迅速搭建React应用脚手架。

这里只对它进行简单的尝试和入门，如果需要进一步的学习，[官网在这里](https://facebook.github.io/create-react-app/)，[文档在这里](https://facebook.github.io/create-react-app/docs/getting-started)，也可以参考[这篇文章](https://juejin.im/post/59dcd87451882578c2084515)进行更高阶更深入的配置和学习。

<!-- more -->

使用Create React App开发React应用不必再安装Webpack或者Babel，它们已经被内置在脚手架中了

它提供的功能：

1. 开箱即用的React支持
2. 开发模式和生产模式的编译
3. 开发模式的热更新
4. 提供了单元测试测试的接口支持
5. 其他配置工具的默认配置（亦可以个性化配置）


## 快速开始

```BASH
npx create-react-app my-app
cd my-app
npm start
```
React应用会运行在`http://localhost:3000/`，开发完成后使用`npm run build`打包


## 安装

安装需要Node的版本在8.10.0以上

```BASH
# 使用npx
npx create-react-app my-app

# npm在5.1版本下不能使用npx
# 需要使用npm，首先进行全局安装
npm install -g create-react-app
# 创建应用
create-react-app my-app
```

## 运行

```BASH
npm start
```

在`http://localhost:3000`以开发模式运行React应用，页面提供了热跟新功能，在控制台会显示错误和警告。

## 单元测试

### 基本使用

```BASH
npm test
```

使用Jest运行单元测试，默认情况下会运行从上次提交commit后有改动的文件的单元测试。

需要`react-scripts@0.3.0`及更高版本，老项目开启单元测试看[这里](https://github.com/facebook/create-react-app/blob/master/CHANGELOG-0.x.md#migrating-from-023-to-030)。

Jest是基于Node的运行期，速度很快，并且动过jsdom提供了浏览器的全局变量，比如`window`，但Jest对于DOM的测试是不准确的，它的目的是对逻辑和组件进行单元测试，而非测试DOM

在运行时Jest会自动寻找以`test.js`/`spec.js`或者`__tests__`文件夹下以`.js`结尾的文件，==这些文件可以位于`src`目录下任意深度的文件夹内==。

建议将测试文件（或`__test__`文件夹）和北侧文件放在一起，有两个好处：

1. 便于管理，一眼就能看到文件的单元测试文件
2. 引入组件的时候更简洁, 例如`import App from './App'`

### 命令行接口

使用`npm test`时，Jest会以`watch`模式运行，每次更改文件都会重新运行测试文件

这个模式下的命令行接口提供了各种能力，可以一直开着这个窗口进行快速的重复测试

### 版本管理接口

使用`npm test`默认情况下会运行从上次提交commit后有改动的文件的单元测试。可以在`watch`模式下按`a`来要求Jest执行全部的测试

如果当前的工程没有使用版本管理，那么Jest会默认运行全部的测试

### 测试组件

关于Jest的使用，以前学习[在Vue中使用Jest时总结过](https://blog.csdn.net/duola8789/article/details/80434962/)，Jest的基本用法是相同的，在测试组件时有所区别，Vue测试组件使用的将组件和Jest进行连接的工具是Vue-test-utils，而React则是[Enzyme](https://airbnb.io/enzyme/)（更准确些，[jest-enzyme](https://github.com/FormidableLabs/enzyme-matchers)更接近于Vue-test-utils，封装了很多方便的API）

内容比较多，这里不展开，[文档在这里](https://facebook.github.io/create-react-app/docs/running-tests)，慢慢单独学习。

## 构建生产文件

```BASH
npm run build
```

打包出的文件是经过压缩的，文件名带有Hash值

## 个性化配置

因为Create-React-App将Webpack、Babel、ESLit的配置隐藏起来，简化了用户的配置操作，可以快速开始开发。

但是这只适用于一些小型的、没有特殊需求的应用的开发，如果构建大型应用还需要对上面这些工具进行个性化的配置：

```BASH
npm run eject
```

运行后，Create-React-App会将上面工具的配置文件复制到项目中，以后对配置文件进行修改后，项目式中会采用项目中复制修改后的配置文件

要注意，这个操作是不可逆的。

## CSS-Loader

在新版本的Create React App中增加了对CSS Modules的支持，要求`react-scripts`版本高级2.0.0。

CSS文件的命名形式为`[name].module.css`，对应的类名会通过添加后缀的形式来实现局部作用域，类名的格式是`[filename]\_[classname]\_\_[hash]`

详情参考[文档](https://facebook.github.io/create-react-app/docs/adding-a-css-modules-stylesheet)。

如果需要在老版本的Create React App中增加了对CSS Modules的支持，则首先需要先通过`eject`命令暴露配置文件，参考[这篇文章](https://medium.com/nulogy/how-to-use-css-modules-with-create-react-app-9e44bec2b5c2)。

## ESLint

由于Create React App将默认的构建配置封装了起来，而ESLint仅仅开启了最基本的规则，更重要的是默认情况下，ESLint仅仅会在IDE中对违反规则的情况进行提示，并不会在构建时在终端的输出进行终端和提示。

如果这种情况可以满足需要，而只需要开启更多的规则，那么就可以在根目录下新建一个文件`.eslintrc.json`，然后添加：

```JS
{
  "extends": "react-app"
}
```

但是如果要起到更强制性的提示作用（中断构建、终端提示），Create React App建议使用[Prettier](https://github.com/prettier/prettier)代替ESLint。如果要使用ESLint，那么就需要使用`npm run eject`，将配置文件吐出，按照AlloyTeam的提示进行配置即可，参考[这篇笔记](https://duola8789.github.io/2017/12/05/01%20%E5%89%8D%E7%AB%AF%E7%AC%94%E8%AE%B0/07%20%E9%9B%B6%E6%95%A3%E4%B8%93%E9%A2%98/%E9%9B%B6%E6%95%A3%E4%B8%93%E9%A2%9816%20EditorConfig%E5%92%8CESLint/)。

## Ant Design按需引入

安装antd：

```BASH
npm install antd -S
```

然后进行按需引入分为两种情况：

（1）未eject出所有配置：

参考[antd的文档](https://ant.design/docs/react/use-with-create-react-app-cn)，

安装[react-app-rewired](https://github.com/timarney/react-app-rewired)和[customize-cra](https://github.com/arackaf/customize-cra)。

```BASH
npm install react-app-rewired customize-cra -D
```
然后修改`package.json`文件的启动命令：

```
"scripts": {
  "start": "react-app-rewired start",
  "build": "react-app-rewired build",
  "test": "react-app-rewired test",
}
```
然后安装[babel-plugin-import](https://github.com/ant-design/babel-plugin-import)

```BASH
npm install babel-plugin-import -D
```

然后在根目录下创建`config-overrides.js`，用来修改默认配置：

```JS
const { override, fixBabelImports } = require('customize-cra');

module.exports = override(
  fixBabelImports('import', {
    libraryName: 'antd',
    libraryDirectory: 'es',
    style: 'css', // true
  }),
);
```

然后按按照下面的格式按需引入模块：

```JS
import { Button } from 'antd';
```

（2）已经eject出所有配置文件：

这个时候直接按照[babel-plugin-import文档](https://github.com/ant-design/babel-plugin-import)的说明配置即可。

安装`babel-plugin-import`：

```BASH
npm install babel-plugin-import -D
```
然后在`package.json`中找到`babel`选项，修改为：

```
"babel": {
  "presets": [
    "react-app"
  ],
  "plugins": [
    [
      "import",
      {
        "libraryName": "antd",
        "style": "css"
      }
    ]
  ]
}
```

引入方式与上面相同：

```JS
import { Button } from 'antd';
```

## 配置Less

首先安装`less`和`less-loader`：

```
npm install less less-loader -D
```

然后同样分为是否eject配置两种情况：

（1）未eject出所有配置，仍遵循上面的步骤，安装`react-app-rewired`和`customize-cra`,修改`package.json`中的启动脚本。

然后修改`config-overrides.js`文件：

```JS
const { override, fixBabelImports, addLessLoader } = require('customize-cra');

module.exports = override(
  fixBabelImports('import', {
    libraryName: 'antd',
    libraryDirectory: 'es',
    style: true, 
  }),
  addLessLoader({
    javascriptEnabled: true,
    modifyVars: { '@primary-color': '#1DA57A' },
  }),
);
```
这里利用了`less-loader`的`modifyVars`来进行主题配置，变量和其他配置方式可以参考[配置主题](https://ant.design/docs/react/customize-theme-cn)文档。

（2）已经eject出所有配置的情况，参考[这篇文章](https://juejin.im/post/5c3d67066fb9a049f06a8323)：

在`config`目录下的`webpack.config.js`文件，找到`// style files regexes`注释位置，添加：

```JS
// 添加 less 解析规则
const lessRegex = /\.less$/;
const lessModuleRegex = /\.module\.less$/;
```

然后找到`rules`属性，在其中添加less解析配置：

```JS
// Less 解析配置
{
  test: lessRegex,
  // exclude: lessModuleRegex,
  use: getStyleLoaders(
    {
      importLoaders: 2,
      sourceMap: isEnvProduction && shouldUseSourceMap,
    },
    'less-loader'
  ),
  sideEffects: true,
},
{
  test: lessModuleRegex,
  use: getStyleLoaders(
    {
      importLoaders: 2,
      sourceMap: isEnvProduction && shouldUseSourceMap,
      modules: true,
      getLocalIdent: getCSSModuleLocalIdent
    },
    'less-loader'
  ),
},
```

要注意的是，新添加的`less-loader`必须在`file-loader`的前面才能生效，因为Webpack在解析Loader是从右至左进行的（从下到上），只有先经过`file-loader`对文件路径的处理，`less`文件才能够被正确引入。


## 参考

- [Docs@Create React App](https://facebook.github.io/create-react-app/docs/documentation-intro)
- [从React脚手架工具学习React项目的最佳实践（上）：前端基础配置@掘金](https://juejin.im/post/59dcd87451882578c2084515)
- [在create-react-app中使用@Ant Design](https://ant.design/docs/react/use-with-create-react-app-cn)
- [ant-design/babel-plugin-import@github](https://github.com/ant-design/babel-plugin-import)
- [在 Create React App 中启用 Sass 和 Less@掘进](https://juejin.im/post/5c3d67066fb9a049f06a8323)
