---
title: React提高08 Create React App
top: false
date: 2019-03-01 09:38:04
update: 2019-03-01 09:38:04
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

## 参考

- [Docs@Create React App](https://facebook.github.io/create-react-app/docs/documentation-intro)
- [从React脚手架工具学习React项目的最佳实践（上）：前端基础配置@掘金](https://juejin.im/post/59dcd87451882578c2084515)
