---
title: React提高09 Hooks
top: false
date: 2019-03-27 18:20:38
updated: 2019-03-29 09:02:35
tags:
- Hooks
categories: React
---

React Hooks学习笔记。

这篇文章真的写了好久，从1月多，Hooks还是实验特性时就开始看，中间断断续续，再加上一开始看英文的文档，拖到了现在。

总结的太磨叽了，又弄了一个精简版，那这个做分享吧。

心里也没底。

[文中涉及到的代码在这里。](https://github.com/duola8789/react-learning)

做了一个[分享的PPT](/files/React-Hooks.pptx)，如果有人需要的话可以拿走。

<!-- more -->

# 精简版

React Hooks是V16.8的新特性，是一个向后兼容的新特性（不会引入破坏性的改变）。

## 引入的原因

实现比现有方案（HOC/Render Props）更优雅的代码复用，为纯组件引入状态，能够将组件划分为更细的粒度。

## 内置Hooks

### 1 `useState Hook`

#### （1）用法

内置的`useState`用来为纯组件添加状态变量和更新方法，可以认为是`this.state`和`this.setState`的简化版，以数组的形式获取状态变量，第一个变量是一个状态，第二个是更新方法，`useState`接受的参数是初始值

```JS
import { useState } from 'react';
const [count, setCount] = useState(0);
```
#### （2）函数式更新

在使用`setState`更新时，可以直接传递一个结果，这个结果将直接赋值给`state`，如果更新的结果与上一个状态有关系，那可以使用函数式更新，`setState`的参数是一个函数，函数的参数是之前的值，返回的是更新的值

#### （3）手动合并对象

`useState`不会自动合并更新对象，所以需要手动进行合并，采取函数式赋值的方式：`setState(prevState => ({ ...prevState, a: 100 }))`

#### （4）延迟初始化

`useState`的参数`initialState`是首次渲染期间使用的状态，如果这个状态是一个高开销的计算结果，可以改为提供函数，这个函数仅在初始渲染时执行，可以避免性能浪费：

```
const [state, setState] = useState(() => ({ a: initialize(), b: 2 }));
```

### 2 `useEffect Hook`

#### （1）用法

内置的`useEffect`会在每次渲染（DOM更新）后（包括首次）后执行才作为参数的方法，可以认为是`componentDidMount`和`componentDidUpdate`的合集，分为需要销毁的和不需要销毁的两种

```JS
useEffect(() => {
  document.title = `You clicked ${count} times`
});
```

上面是不需要销毁的`useEffect`，它会在每次渲染后执行内部的函数。

#### （2）清理effect

如果需要销毁，那么只需要在这个函数内部返回一个另一个函数，在返回的函数中执行销毁操作即可。注意和执行一样，销毁也是在每次渲染后都会执行，可以防止内存泄漏。

```JS
useEffect(() => {
  document.title = `You clicked ${count} times`;
  return () => {
    document.title = `ok`;  
  }
});
```
#### （3）避免重复渲染

`useEffect`默认的表现是在每次渲染后触发，当组件的任何一个状态发生改变时，更新函数都会执行，可以给`useEffect`传递第二个参数，这时只有当数组中的任一一项的值发生变化，`useEffect`的更新函数才会执行。

这个数组并不会作为参数传递给更新函数内部，但是从概念将，更新函数中引用的每个值都应该出现在输入数组中，这样才能避免更新函数依赖的某个值发生了变化，而函数没有重新执行。ESLint的插件会自动检测并插入这个数组，推荐使用。

如果参数一个空数组，那么任何变量的变化都不会引起`useEffect`的执行，这时候与只在`componentDidMount`和`componentWillUnmount`执行代码是类似的。

它的最大作用是就是减少因为没有在`componentDidUpdate`处理更新前的状态而导致的bug。

#### （4）`useEffect`的执行时机

`useEffect`中的更新函数会延迟到layout和paint后触发，也就是说在浏览器更新屏幕之后才会触发，但是有一些事件不能推迟，对于这个类型的事件需要在`useLayoutEffect`中触发，它与`useEffect`的不同就是在触发时机上的不同。

虽然`useEffect`延迟到浏览器绘制完成之后执行，但是它保证在任何新渲染之前触发。

#### （5）`useEffect`中获取本次渲染更新后的值

==在`useEffect`的更新函数中，拿到的`state`和`props`总是当次渲染的初始值==，==即便执行了`setState`之后仍是这样==。

可以认为每次渲染时通过`useState`声明的状态是不可变的（Immutable），每次渲染都会对它拍一个快照保存下来，当状态更而重新渲染时就会形成N个状态。不光是`state`和`props`通过快照的形式保存，组件的事件处理和`useEffect`都是同样的形式

如果要在`useEffect`中使用更新后的`state`，需要使用`useRef`

### 3 `useRef`

`useRef`返回一个可变的对象，其`current`属性被初始化为传递的参数，返回的这个对象就保留在组件的生命周期中。

`useRef`返回的`ref`对象在所有Render过程中保持着唯一引用，如果认为`state`是不可变的数据，那么`ref`对象就可以认为是可变对象，对`ref.current`的赋值和取值，拿到的额都是一个状态。

要注意，避免在渲染过重中（`return`中）设置直接饮用，可能会导致预料之外的结果，相反，应该只在事件处理程序和`useEffect`中修改、使用引用。

### 4 `useContext`

用来创建context对象，当使用来传递数据时有用。

### 5 `useReducer`

useState的替代方案，当组件使用flux架构组织管理数据时有用。

用它配合`useContext`可以避免在多层组件中深度传递回调的需要。

### 6 `useCallback`

主要是用来处理在`useEffect`之外的定义函数无法管理依赖，也无法成为`useEffect`的依赖，每次渲染都会生成新的快照的情况，使用之后只有在函数的依赖发生变化时才会生成新的函数，有利于提高性能，依赖也更清晰。

### 7 `useMemo`

与`useCallback`类似，返回的是一个不生成快照的对象，而非函数。

`useMemo`只会在其中一个输入发生更改时重新计算，此优化有助于避免在每个渲染上进行高开销的计算。

### 8 `useLayoutEffect`

前面介绍过，与`useEffect`的不同点仅仅在于执行时机不同，`useLayoutEffect`在绘制前同步触发，`useEffect`会推迟到绘制后触发

## Hooks的使用规定

（1）只在最顶层调用Hooks，不要再内部循环、条件语句或嵌套函数中调用Hooks（这是因为React是通过多个Hooks的调用顺序来确定多个`useState`中状态变量的对应关系），如果想要有条件的运行一个effect，可以将条件判断放在Hook内部

（2）只在React函数中调用Hooks，不在普通的JavaScript函数中调用

可以通过ESLint的`eslint-plugin-react-hooks`插件来检查、规范Hooks的使用，避免不规范的使用而导致的bug。

## 编写自定义Hook

自定义Hook实际上就是一个函数，将公用的逻辑提取进去，可以调用其他的Hook，必须以`use`开头。

不同的组件使用同一个自定义的Hook，状态不会共享，只是将逻辑复用，Hook使用时内部状态和effect都是完全隔离的。



# 啰嗦版

## 1 简介

React Hooks是v16.8推出的一个新特性，它提供给开发者一种能力，让开发者不用`class`的形式就能够使用`state`和其他的特性

在进一步了解Hooks之前，有几点注意事项：

- 不是必选项，你可以现在现有的组件中添加代码来试用Hooks，但是如果你不想使用它也是完全可以的
- 100%向后兼容，Hooks不会引入任何破坏性的改变（breaking changes）
- 已经提供使用，在16.8版本正式引入

Hooks的引入并不意味着在React中要放弃使用`class`，后面会介绍渐进式使用Hooks的策略

Hooks也不会取代你已了解的React中的概念，相反，Hooks为`props`、`state`、`context`、`refs`和生命周期等等你所熟悉的概念提供了一个更加直接的API。Hooks提供了一个更好的方式来复合使用上述概念。

## 2 为什么要引入Hooks

Hooks解决了React中一系列看似毫不相关的问题，这些问题在我们编写、维护大量组件时非常常见。

### 2.1 在组件间复用有状态的逻辑是非常困难的

React并没有提供一种方式来为组件添加可复用的接口（例如将一个组件连接到store）。React中经常使用一些编程范式来解决这类问题，例如Render Props、高阶组件。但是使用这些范式需要对组件进行重构，这个过程非常的麻烦，并且将导致代码难以维护。当你在React DevTools中查看一个典型的React应用，你会发现充满了“包裹地狱”（wrapper hell），组件嵌套在多层的provider、consumer、高阶组件、Render和其他抽象结构中。虽然我们可以公国开发工具来进行梳理，但是这也表明了一个更底层的问题：React需要一个更好原生方法来实现状态逻辑的复用。

使用Hooks，你可以将状态逻辑从组件中抽离出来，这样就可以对它单独进行测试和复用。**Hooks为开发者提供了在不改变组件层次的基础上进行状态逻辑复用的能力**。这使得我们能够轻而易举的在组件之间或者社区间共享Hooks。


### 2.2 复杂的组件变的难以理解

我们常常要维护这样的组件：开始很简单，慢慢就充满了大量难以维护的状态逻辑和副作用。每个生命周期方法常常包含了大量的不相关逻辑的组合。例如，组件常常在`componentDidMoun`和`componentDidUpdate`来获取数据，而在一个`componentDidMount`中常包含许多无关的逻辑，比如建立事件监听器，这些事件监听器需要在`componentWillUnmount`中销毁。这些相互关联、同时更改的代码被分隔开了，但是完全没有关系的却出现在同一个方法中。这将带来大量的bug和不一致性。

在大多数情况下，我们无法将这些组件分割成为更小的组件，因为状态逻辑贯穿了整个组件，我们也很难测试这些组件。这也是许多开发者更倾向于为React配套使用一个单独的状态管理框架的原因。但是这也常常带来太多的抽象，让你在不同的文件之间反复跳转，让组件的复用更加困难。

为了解决这个问题，Hooks可以根据依赖（例如建立订阅器或者获取数据）而非生命周期钩子，将一个组件分割为更小的函数。为了让组件更容易预测，你可以选择通过reducer来管理组件的内部状态。

### 2.3 `Class`让开发者和计算机都感到迷惑

为了实现代码复用，代码组织难度加大了。我们发现`Class`成为了学习React的一大障碍。你必须搞清楚在JavaScript中`this`的工作原理，这与`this`在其他大多数语言中的原理都不相同。你必须记得为事件处理程序绑定`this`。在[关于`class`的语法提案](https://babeljs.io/docs/en/babel-plugin-proposal-class-properties)没有正式通过前，`class`的代码显得非常啰嗦。人们能够很好地理解`props`、`state`、自顶向下的数据流，但是仍很难理解`class`。React中函数和`class`组件自身以及应用场景的区别在充满经验的React开发者间都存在着分歧。

此外，React已经诞生5年了，我们希望在接下来的5年保持连贯性。像Sevlte、Angular、Glimmer等显示的，AOT编译（运行前编译）有着很大的潜力，尤其是不仅用于与模板时。我们最近在实验使用[Prepack](https://prepack.io/)进行[组件折叠（component folding）](https://note.youdao.com/)的尝试，并且初见成效。但是我们也发现`class`组件带来的某些无目的范式会降低这些优化效果。`Cass`在当前的工具中也存在着问题，比如`Class`的压缩效果并不好，热更新编的颗粒化且不可靠。我们希望能够提供新的API让代码得到最大程度的优化。

为了解决这些问题，**Hooks提供了绕过`class`使用React更多的特性的能力**。从概念上将，React组件更接近于函数。Hooks对函数更友好，但也不会未被React所提倡的理念。Hooks提供了一个非常迫切的“逃生舱口”，并且不需要您去学习复杂的函数式编程或者响应式编程的知识。

### 2.4 小结

啰里吧嗦翻译了一堆，总结起来，Hooks出现的原因有三：

1. 现有的React的状态组件复用方式（高阶组件、Render Props）有各自的问题， 而Hooks可以优雅的（不改变组件层次）实现代码复用
2. Hooks可以将组件根据功能，将组件划分为更小的粒度，便于调试、测试和维护
3. Hooks可以不使用`Class`来编写组件，提高代码性能，降低React的使用难度

## 3 渐进式策略

> React并不准备移除`class`

我们都知道，Recat开发者都忙于搬砖而无暇关注新发布的API。Hooks非常新，可能在更多的实例和教程出现之后，再考虑学习或者使用Hooks，可能更加稳妥。

我们也知道，为React添加一个新的原语遇到的阻力会很大。我们为好奇的读者准备了一份详细的[请求意见稿](https://github.com/reactjs/rfcs/pull/68)，它介绍了Hooks出现的更多细节，提供了额外的视角来审视具体的设计决定和相关技术。

**最重要的是，Hooks与现有代码是共存的，所以可以渐进式的应用到代码中**。我们分享了试验性的API，目的是得到社区中对亲自参与React未来发展的人们的早期的反馈，我们将不断更新Hooks。

最后，不必急于迁移到Hooks。我们建议避免任何的“大重构”，特别是针对现有的、复杂的`class`组件。Hooks的思维习惯还需要一定的思维转变。根据我们的经验，最好首先在全新的、不重要的组件中练习使用Hooks，并且确保你的团队中的每个人都欣然接受。

我们为Hooks准备了全部现有的针对`class`的用例，但是**我们将在可预见的未来继续提供对`class`组件的支持**。Facebook中有大量的用`class`写成的组件，我们绝没有移除它们的计划。相反，我们将会在新的代码中同时使用Hooks和`class`

## 4 State Hook

React默认提供了两种Hooks，一种用来声明状态变量的State Hook，另外一种是用来执行副作用函数的Effect Hook。另外，用户可以自定义个性化的Hooks，进行灵活的应用。首先来了解State Hook。

### 4.1 例子

下面的例子渲染了一个计数器，当点击按钮式，计算器的值会递加

先看传统的`class`组件怎么写：

```
class Count extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }

  setCount() {
    const { count } = this.state;
    this.setState({
      count: count + 1
    })
  };

  render() {
    return (
      <div>
        <p>You clicked {this.state.count} times</p>
        <button onClick={() => this.setCount()}>Click Me</button>
      </div>
    )
  }
}
```

用Hooks的实现：

```
// 引入useState，用来在函数组件中保存局部变量
import { useState } from 'react';

function Example() {
  // Declare a new state variable, which we'll call "count"
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

上面，`useState`就是一个Hook。我们在一个函数组件中调用Hook，来为函数组件添加一些内部状态。React在重复渲染时会保留这些状态。`useState`返回两个变量：当前状态值和更新此状态的函数。你可以在一个事件处理程序等任意位置调用这个方法。它和`class`中的`this.setState`也有点类似，除了Hook返回的函数并不会将新旧状态进行合并（后面会有一个例子来比较`useState`和`this.setState`的区别）

`useState`的唯一一个参数是初始状态值。上面的例子中，这个值是`0`，因为我们的计数器是从0开始计数的。注意，与`this.state`不同，Hooks中的状态值类型可以不是对象，当时对象类型也是支持的。初始状态参数旨在第一次渲染时用到。

### 4.2 声明状态变量

上面的`class`是一个有状态的组件，React中又有一种无状态的组件，也叫做函数组件（function component）长的像这样：

```
const Example = (props) => {
  // You can use Hooks here!
  return <div />;
}
```
如果想要用函数组件来实现计数器这个例子，在以前是行不通的，因为在函数组件中是没有`this`的，所以没有办法维护`this.state`这个状态变量。

这时候就是Hooks的用武之地了：当你想为一个函数组件添加一个状态变量（`state`），以前只能改写成为`class`的形式，现在就可以在函数组件中使用Hooks来达到你的目的。

在函数组件中可以使用`useState`：

```JS
import { useState } from 'react';

function Example() {
  // Declare a new state variable, which we'll call "count"
  const [count, setCount] = useState(0);
}
```
**`useState`到底做了什么？**

`useState`声明了一个“状态变量”（例子中的`count`），这个变量名是可以随意定义的。这是在函数调用时保存变量值的方法--`useState`的作用就像`class`中的`this.state`一样。一般来说，当函数退出后变量会被清除，但是React会将状态变量保存起来。

**`useState`的参数是什么？**

`useState`只有一个参数，那就是Hook的初始值。与`class`不同，`useState`定义的状态变量可以不是`object`，`number`或者`string`都是可以的。在计数器的例子中我们需要一个数值来记录用户点击的次数，所以我们为变量传入了`0`作为初始值（如果想要在`state`中保存两个不同的值，需要调用`useState()`两次）

**`useState`的返回值是什么？**

返回两个变量：当前值和这个值的更新方法，上面的代码`const [count, setCount] = useState()`和`class`中的`this.state.count`和`this.setState`是类似的，不同点是Hook将它们作为一组数据同时返回

当再次渲染时，React会在函数组件中获取`count`的最新值，如果想要更新`count`可以调用`setCount`

### 4.3 读取状态变量

在`class`中我们通过读取`this.state.count`来获取最新的`count`值：

```
<p>You clicked {this.state.count} times</p>
```
在函数组组件中可以直接使用`count`值：

```
<p>You clicked {count} times</p>
```

### 4.4 更新状态变量

在`class`中通过调用`this.setState`来更新`count`值

```
<button onClick={() => this.setState({ count: this.state.count + 1 })}>
  Click me
</button>
```
在函数组件中，我们已经声明了变量`setCount`和`count`，所以也就不需要`this`了


```
<button onClick={() => setCount(count + 1)}>
  Click me
</button>
```

### 4.5 声明多个状态变量

在一个组件中可以多次使用状态Hook：

```JS
function ExampleWithManyStates() {
  // 声明了多个状态变量
  const [age, setAge] = useState(42);
  const [fruit, setFruit] = useState('banana');
  const [todos, setTodos] = useState([{ text: 'Learn Hooks' }]);
  // ...
}
```

通过数组的[解构赋值](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment/)，我们可以在调用`useState`声明多个不同名称的状态变量。变量名并不是`useState`API的一部分，相反，React假设你会在每次渲染时，以相同的顺序多次调用`useState`。后面我们会回过头再来解释为什么可以这样做以及应用场景。

### 4.6 究竟什么是Hook？

通过上面的例子，我们可以发现：

Hook是一种能够“侵入”React的函数组件的状态和生命周期特性的函数。Hook不能用在`class`中--Hook让你能够抛弃`class`来使用React（我们并不建议立刻对现有代码进行重构，但是如果你愿意的话，可以在新代码中开始使用Hooks）

React提供了一些内嵌的Hooks，例如`useState`。你也可以创建自己的Hook，达到在不同的组件间复用有状态的行为。后面会首先讲解内嵌的Hook。


## 5 Effect Hook

我们将React组件中的某些操作称为具有副作用的effect（或者简称为effect），例如获取数据、订阅事件或者手动更改DOM，因为这些行为可能会影响其他的组件，并且无法在渲染期间完成。

`useEffect`就是一种Effect Hook，它可以为函数组件添加副作用行为。它提供的功能和`class`中的`componentDidMount`、 `componentDidUpdate`、`componentWillUnmount`是相同的，只是被合并为一个单独的API。

React组件中有两种常见的effect，一种是需要销毁的，另外一种是不需要销毁的。

### 5.1 不需要销毁的effect

有些时候，我们希望在React更新DOM之后运行一些额外的代码。注入网络请求、手动更新DOM以及打印日志都是常见的不需要销毁的effect。这么认为的原因是我们可以运行它们，并且立刻忘记它们。下面看一下，用Class组件和Hook组件分别是如何实现effect的。

#### 5.1.1 用Class组件实现

在React的Class组件中，`render`方法并不会导致effect，因为在`render`中为时尚早--我们是在React更新DOM后才执行effect。

所以在React的Class组件中，我们将effect放在`componentDidMount`和`componentDidUpdate`生命周期函数中。下面的例子中，组件会在React更新DOM之后设置文档的标题。

```
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }

  componentDidMount() {
    document.title = `You clicked ${this.state.count} times`;
  }

  componentDidUpdate() {
    document.title = `You clicked ${this.state.count} times`;
  }

  render() {
    return (
      <div>
        <p>You clicked {this.state.count} times</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Click me
        </button>
      </div>
    );
  }
}
```

我们必须在class的两个生命周期函数中编写相同的代码。

这是因为很多情况下，不管组件是刚刚渲染完成还是更新完成，我们想要执行相同的effect。也就是说，我们想要在每次渲染后都执行effect。但是React的Class组件并没有提供这样的生命周期。我们能够提取出一个单独的方法，但是仍需要在两个地方调用它。

而如果使用`useEffect`Hook来完成同样的行为：

#### 5.1.2 用Hook组件实现

如果改成使用`useEffect`的形式：

```
import { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  // Similar to componentDidMount and componentDidUpdate:
  useEffect(() => {
    // Update the document title using the browser API
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

你可能会有这样的疑问：

（1）`useEffect`究竟做了什么？

通过使用这个Hook，你告诉React这个组件需要在每次渲染后执行一些操作。React会保存你传入的函数（这个函数就是上面提到的effect）并且在每次DOM更新后进行调用。在上面的Effect中，我们更改了文档的标题，当然也可以执行获取数据等其他必要的操作。

（2）为什么在组件中调用`useEffect`？

将`useEffect`放在组件中让我们有能力从effect中获取`count`状态变量（以及任何prop）我们不需要特殊的API来读取它--因为它已经在函数的作用域中了。Hooks鼓励JavaScript闭包，并且尽量避免在JavaScript已经提供解决方案的情况下引入额外的React特有的API

（3）`useEffect`在每次渲染后都会运行吗？

是的。`useEffect`默认在首次渲染以及每次更新后都会运行（当然后办法修改这种行为）。这使得你函数无所谓是在mount还是update后被调用，因为他们会在每次渲染后都被统一调用。Reasct会保证在DOM更新完成后才会调用effect。

#### 5.1.3 细节讲解

```JS
function Example() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });
```

我们声明了`count`状态变量，随后需要使用effect来完成某些行为。我们给`useEffect`Hook传递了一个函数，这个函数就是effect，在其中我们设置文档的标题。我们在effect可以读取罪行的`count`因为它位于函数作用域内。当React渲染组件时会保存我们的effect，并且在每次更新DOM后运行--包括首次渲染。

有经验的卡开发者或者会注意到，传递给`useEffect`的函数在每次渲染后都是不同的。这是有意而为之。实际上，这让我们在effect中读取`count`的值而不必担心它的值没有刷新。每次渲染，我们都会使用新的effect代替之前的。折让effect看起来更像render的结果--每个effect都“属于”一次特定的render。这是很有用的，后面会提到。

与`componentDidMount`和`componentDidUpdate`，`useEffect`的effect不会阻塞浏览器更新显示信息。这会让你的app更加响应式。大部分的effect不需要同步完成，在少数情况下需要（例如测量布局），可以使用`useLayoutEffect`API。

### 5.2 需要销毁的effect

有一些effect需要销毁，例如我们可能会针对默写外部数据源建立监听器。这种情况下就必须销毁effect，以避免内存泄漏。同样使用Class组件和Hook组件都可以时间。

#### 5.2.1 使用Class组件

在React的Class组件中，需要在`componentDidMount`生命周期中建立监听器，在`componentWillUnmount`销毁，例如，我们通过`ChatAPI`来订阅好友的上线状态，在React的Class组件中的实现如下：

```
class FriendStatus extends React.Component {
  constructor(props) {
    super(props);
    this.state = { isOnline: null };
    this.handleStatusChange = this.handleStatusChange.bind(this);
  }

  componentDidMount() {
    ChatAPI.subscribeToFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  componentWillUnmount() {
    ChatAPI.unsubscribeFromFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  handleStatusChange(status) {
    this.setState({
      isOnline: status.isOnline
    });
  }

  render() {
    if (this.state.isOnline === null) {
      return 'Loading...';
    }
    return this.state.isOnline ? 'Online' : 'Offline';
  }
}
```
在`componentDidMount`和`componentWillUnmount`生命周期中需要执行相反的操作。生命周期方法让我们不得不将逻辑上相关联的操作分割开来。

#### 5.2.2 使用Hooks组件

你或许认为我们需要一个独立的effect来执行销毁。但是由于添加和移除订阅的关系如此紧密，所以`useEffect`在设计时就将而二者联系在了一起。在effect中返回一个函数，React就会在销毁时执行这个函数。

```
mport { useState, useEffect } from 'react';

function FriendStatus(props) {
  const [isOnline, setIsOnline] = useState(null);

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);

    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
```

在上面的例子中，React会在组件销毁时通过`ChatAPI`执行取消订阅，就如同在随后的渲染中再次运行Effect Hook一样（如果需要，当传递给`ChatAPI`的`props.friend.id`没有变化时，可以让React不再重复订阅）

（1）为什么在effect中返回了一个函数？

这是effect的可选的销毁机制。每一个effect都可以返回一个函数来进行销毁。这让我们能够统一维护保持添加和移除的逻辑。它们都是同一个effect的组成部分。

返回的函数可以使命名函数也可以是匿名函数、箭头函数。

（2）React究竟是何时销毁一个effect？

React在组件unmount时销毁effect。但是正如我们上面介绍过的，effect在每次渲染都会运行，而不只运行一次。所以React也会在上一次渲染结束后、运行下一次的effect之前来执行清理。后面会介绍为什么这有利于减少bug以及为了减少可能带来的性能问题如何退出这种机制。


### 5.3 为什么在每次更新后effect都会运行

为什么effect的销毁在每次重新渲染后都会执行，而不是只在unmount时执行一次。下面的例子会展示这样设计的好处--编写bug更少的组件。

前面的订阅好友上线状态的组件会展示好友是否上线，组件从`this.props`中读取`friend.id`，在组件mount后订阅好友状态，在unmount后取消订阅

```
componentDidMount() {
  ChatAPI.subscribeToFriendStatus(
    this.props.friend.id,
    this.handleStatusChange
  );
}

componentWillUnmount() {
  ChatAPI.unsubscribeFromFriendStatus(
    this.props.friend.id,
    this.handleStatusChange
  );
}
```
但是，如果prop中的firend变化了，而组件仍然显示在界面上，这时会发生什么？我们的组件会继续展示之前的朋友（与当前不同）的上线状态。这就是一个bug。而且当我们在取消订阅时用了错误的好友ID是，会造成内存的泄露甚至崩溃。

在Class组件中，我们需要在`componentDidUpdate`中处理这种情况：

```
componentDidUpdate(prevProps) {
  // Unsubscribe from the previous friend.id
  ChatAPI.unsubscribeFromFriendStatus(
    prevProps.friend.id, 
    this.handleStatusChange
  );
    
  // Subscribe to the next friend.id
  ChatAPI.subscribeToFriendStatus(
    this.props.friend.id, 
    this.handleStatusChange
  );
}
```
忘记正确的处理`componentDidUpdate`中的逻辑是React产生bug最多的原因之一。

如果使用Hooks的组件：

```
function FriendStatus(props) {
  // ...
  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });
```
这种情况就能避免上面的bug。

没有代码专门处理更新的情况因为`useEffect`默认会自动处理。它会在应用下一个effect之前清理上一个effect。下面的订阅和取消订阅的顺序可以证明这一点：

```JS
// Mount with { friend: { id: 100 } } props
ChatAPI.subscribeToFriendStatus(100, handleStatusChange);     // Run first effect

// Update with { friend: { id: 200 } } props
ChatAPI.unsubscribeFromFriendStatus(100, handleStatusChange); // Clean up previous effect
ChatAPI.subscribeToFriendStatus(200, handleStatusChange);     // Run next effect

// Update with { friend: { id: 300 } } props
ChatAPI.unsubscribeFromFriendStatus(200, handleStatusChange); // Clean up previous effect
ChatAPI.subscribeToFriendStatus(300, handleStatusChange);     // Run next effect

// Unmount
ChatAPI.unsubscribeFromFriendStatus(300, handleStatusChange); // Clean up last effect
```
这种行为默认保证了一致性，避免了Class组件中因为忽略更新逻辑而导致的bug

### 5.4 使用effect的注意事项

#### 5.4.1 使用多个effect来分离关注点

在我们讨论为什么要引入Hooks的时候提到了一个问题，Class组件的生命周期方法常常包含无关的逻辑，而相关逻辑却常常被分散在不同的方法中。下面的组件包含了前面例子中的计数器和好友上线提示的逻辑：

```
class FriendStatusWithCounter extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0, isOnline: null };
    this.handleStatusChange = this.handleStatusChange.bind(this);
  }

  componentDidMount() {
    document.title = `You clicked ${this.state.count} times`;
    ChatAPI.subscribeToFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  componentDidUpdate() {
    document.title = `You clicked ${this.state.count} times`;
  }

  componentWillUnmount() {
    ChatAPI.unsubscribeFromFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  handleStatusChange(status) {
    this.setState({
      isOnline: status.isOnline
    });
  }
  // ...
```

上面的代码中，设置文档标题的逻辑分散在`componentDidMount`和`componentDidUpdate`中，订阅逻辑分散在`componentDidMount`和`componentWillUnmount`中，而`componentDidMount`中包含了所有的逻辑代码。

使用Hooks如何解决这个问题？就像可以多次使用状态Hook一样，也可以使用多个effect。这让我们可以分离无关逻辑到不动的effect中。

```
function FriendStatusWithCounter(props) {
  const [count, setCount] = useState(0);
  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  const [isOnline, setIsOnline] = useState(null);
  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }
  // ...
}
```
==Hooks让我们能够基于代码的行为来分割代码==，而不是基于生命周期。React会按照effect声明的顺序，运行组件中的每一个effect


#### 5.4.2 跳过effect以改善性能

某些情况下，每次渲染都销毁或者应用effect会造成性能问题。在Class组件中，我们可以通过在`componentDidUpdate`中对比`prevProps`和`prevState`来解决这个问题

```JS
componentDidUpdate(prevProps, prevState) {
  if (prevState.count !== this.state.count) {
    document.title = `You clicked ${this.state.count} times`;
  }
}
```

这种需求很常见，所以它被内置到了`useEffect`Hook的API中。如果在重复渲染时某些特定值未发生改变，你可以让React不再运行effect。具体做法是将一个数组作为可选的第二个参数传递给`useEffect`

```JS
useEffect(() => {
  document.title = `You clicked ${count} times`;
}, [count]); // Only re-run the effect if count changes
```

上面的例子中，将`[count]`作为第二个参数传给了`useEffect`。当`count`值未发生变化时，React会跳过`effect`。如果数组中有多个值，React会在任何一个发生改变时再次执行effect。

注意，如果进行这种优化，确保数组中包含了被effect使用的、会随时间变化的外部变量。否则你的代码从上次渲染中获取的参考值不会改变。

如果想只执行、销毁effet一次（mount和unmount时），可以传递一个空数组作为第二个参数。这告诉了React你的effect不依赖任何从`props`和`state`中任何变量，所以不需要重复执行。这种情况与只在`componentDidMount`和`componentWillUnmount`执行代码是类似的，我们建议谨慎使用，因为容易导致bug

## 6 Hooks的使用规定

Hooks是函数，在使用时需要注意两条规定，可以引入ESLint的一个[插件](https://www.npmjs.com/package/eslint-plugin-react-hooks)来确保遵守

### 6.1 只在最顶层调用Hooks

==不要在内部循环、条件语句或者嵌套函数中调用Hooks==。相反，要式中在React函数的最顶层使用Hooks。这样才能确保组件每次渲染时都以同样的顺序调用Hook。这是保证React在多个`useState`和`useEffect`中正确保存状态的前提。

### 6.2 只在React函数中调用Hooks

不要在普通的JavaScript函数中调用Hooks。正确的做法是：

- 在React的函数组件中调用Hooks
- 在自定义的Hooks中调用Hooks

遵守这条规定，能够确保组件中的所有状态逻辑在源码中都是清晰可见的。

### 6.3 ESLint插件

通过这个[ESLint插件](https://www.npmjs.com/package/eslint-plugin-react-hooks)可以检测在使用Hooks时是否遵守了上面两条规定。安装：

```BASH
npm install eslint-plugin-react-hooks
```

ESLint的配置文件：

```JS
{
  "plugins": [
    // ...
    "react-hooks"
  ],
  "rules": {
    // ...
    "react-hooks/rules-of-hooks": "error"
  }
}
```
将来这个插件会默认集成在Create React App和类似工具中。

### 6.4 为什么

看这样一个例子：

```JS
function Form() {
  // 1. Use the name state variable
  const [name, setName] = useState('Mary');

  // 2. Use an effect for persisting the form
  useEffect(function persistForm() {
    localStorage.setItem('formData', name);
  });

  // 3. Use the surname state variable
  const [surname, setSurname] = useState('Poppins');

  // 4. Use an effect for updating the title
  useEffect(function updateTitle() {
    document.title = name + ' ' + surname;
  });

  // ...
}
```
React是通过Hooks的调用顺序来确定多个`useState`中状态变量的对应关系。上面的例子之所以能够正常运行是因为每次渲染时Hooks的执行顺序是相同的。


```JS
// ------------
// First render
// ------------
useState('Mary')           // 1. 初始化状态变量name为'Mary'
useEffect(persistForm)     // 2. 添加effect来保存表单数据
useState('Poppins')        // 3. 初始化状态变量surname为Poppins'
useEffect(updateTitle)     // 4. 添加effect来更新标题

// -------------
// Second render
// -------------
useState('Mary')           // 1. 读取状态变量name (忽略了参数)
useEffect(persistForm)     // 2. 更新effect
useState('Poppins')        // 3. 读取状态变量surname(忽略了参数)
useEffect(updateTitle)     // 4. 更新effect
```
只要Hook的调用顺序不变，React就能正确辨别多个变量的关系。但是如果我们将一个Hook放到一个条件语句中呢？


```JS
// 🔴 违反了第一条规定，在条件语句中使用了Hook
if (name !== '') {
  useEffect(function persistForm() {
    localStorage.setItem('formData', name);
  });
}
```
条件语句第在第一次渲染时条件判断为`true`，Hooks的调用情况与上面相同。但是当第二次渲染时判断条件如果变为了`false`，这个Hook会被跳过，这是Hook的调用顺序发生了变化：

```
useState('Mary')           // 1. 读取状态变量name (忽略了参数)
// useEffect(persistForm)  // 🔴 这个Hook被跳过了
useState('Poppins')        // 🔴 2 (原来是第3步). 无法读取状态变量surname
useEffect(updateTitle)     // 🔴 3 (原来是第4步). 无法更新effect
```
React不会知道第二个`useState`返回值是什么，React认为这个组件的第二个Hook是`persistForm`这个effect，就如同上次渲染相同，但是实际情况并不是这样。从这之后的Hooks调用都会错位，导致了bug。

==这就是必须在组件中的顶层中调用Hook的原因==。如果想要有条件的运行一个effect，可以将条件判断放在Hook内部：

```JS
useEffect(function persistForm() {
  // 👍 We're not breaking the first rule anymore
  if (name !== '') {
    localStorage.setItem('formData', name);
  }
});
```

## 编写个性化Hook

### 例子

编写个性化Hook可以将组件的逻辑提取为可复用的函数。

上面例子中一个订阅好友状态的组件我们命名为`FriendStatus`，现在假设我们的聊天程序有一个通讯录的功能，我们想要将在线好友的名字渲染为绿色。我们可以粘贴复制相同逻辑的代码到`FriendListItem`组件中，但是这很不优雅：

```
import React, { useState, useEffect } from 'react';

function FriendListItem(props) {
  const [isOnline, setIsOnline] = useState(null);

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  return (
    <li style={{ color: isOnline ? 'green' : 'black' }}>
      {props.friend.name}
    </li>
  );
}
```

更优雅的做法是在`FriendStatus`和`FriendListItem`组件中共用相同的逻辑。

在以前，React中想要在组件中复用状态逻辑有两种方法：`render props`和HOC。使用这两种方法都需要添加一些包裹组件，现在使用Hooks可以不向组件树中添加新的组件来解决这个问题。

### 提取自定义Hook

如果我们想在两个JavaScript函数中复用逻辑，可以将这部分提取到单独的函数中。React组件和Hooks都是函数，所以也可以使用这种方法。

自定义的Hook是这样的一种函数：函数名以`use`开头，可以调用其他的Hooks。在下面我们编写一个自定义的Hook：`useFeinedStatus`

```JS
import React, { useState, useEffect } from 'react';

function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(friendID, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(friendID, handleStatusChange);
    };
  });

  return isOnline;
}
```
在`useFeinedStatus`中没有新的代码，逻辑都是从旧的组件中复制而来。和组件中调用一样，在我们自己编写的Hooks中，仍然需要遵守在顶层调用、不能在条件语句中调用的原则。

与React的组件不同，自定义的Hook不需要特殊标记。我们可以决定它的参数和返回值，就如果普通的函数一样。它的名字应该以`use`开头，这样就能一眼分辨出它遵守的Hook的规定。

我们的`useFeinedStatus`Hook的目的是订阅好友的状态，所以参数是`friendID`，返回值是好友是否在线的状态值

```JS
function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  // ...

  return isOnline;
}
```

### 使用自定义Hook

在一开始，我们的既定目标是从`FriendStatus`和`FriendListItem`组件中移除重复的逻辑代码。二者中都包含了好友上线通知的逻辑。

现在我们已经将这部分逻辑提取到了`useFriendStatus`这个Hook中，我们可以这样使用：

```JS
function FriendStatus(props) {
  const isOnline = useFriendStatus(props.friend.id);

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
```

```
function FriendListItem(props) {
  const isOnline = useFriendStatus(props.friend.id);

  return (
    <li style={{ color: isOnline ? 'green' : 'black' }}>
      {props.friend.name}
    </li>
  );
}
```

**优化后的代码与原来的代码相同吗？**

是的。优化后的代码实现了一样的功能。但是我们将复用的部分提取到了单独的函数中。自定义的Hook的设计思路与Hooks相同，与React的特性不同。


**自定义的Hook的名字一定要要以`use`开头吗？**

是的，这个习惯很重要。如果不这样做，我们就无法自动的侦测违反Hooks使用规定的行为，因为我们无法分别一个函数中是否调用了Hooks。

**使用同一个Hook的两个组件的会共享状态码？**

并不是，自定义Hooks是一种复用状态逻辑的方式（就像建立一个监听器并且保存当前值），但是每次使用一个自定义Hook，所有其内部的状态和effect都是完全隔离的。


**自定义Hook是如何隔离状态的？**

Hook的每次调用都会隔离状态，因为对于React而言，我们直接调用`useFriendStatus`，与调用`useState`和`useEffect`是一样的。前面介绍过，同一个组件中`useState`和`useEffect`的多次调用完全是相互独立的。

### 提示：在Hooks之间传递参数

既然Hooks是函数，那么我们就可以在它们中间传递参数。

为了说明这一点，我们为上面的聊天APP的例子再添加一个组件，它是一个聊天信息接受选择器，用来显示当前选择的好友是否在线。

```
const friendList = [
  { id: 1, name: 'Phoebe' },
  { id: 2, name: 'Rachel' },
  { id: 3, name: 'Ross' },
];

function ChatRecipientPicker() {
  const [recipientID, setRecipientID] = useState(1);
  const isRecipientOnline = useFriendStatus(recipientID);

  return (
    <div>
      <Circle color={isRecipientOnline ? 'green' : 'red'} />
      <select
        value={recipientID}
        onChange={e => setRecipientID(Number(e.target.value))}
      >
        {friendList.map(friend => (
          <option key={friend.id} value={friend.id}>
            {friend.name}
          </option>
        ))}
      </select>
    <div/>
  );
}
```
我们将当前选择的好友ID保存在变量`recipientID`中，如果用户在`<select>`中选择了另外的好友我们会更新这个变量

因为`useState`Hook调用时会提供`recipientID`的最新之，我们将它作为参数传给`useFriendStatus`这个Hook

```JS
const [recipientID, setRecipientID] = useState(1);
const isRecipientOnline = useFriendStatus(recipientID);
```

这就实现了通知我们当前选择的好友是否上线的目的。当我们选择了另外一个好友并且更新了`recipientID`后，我们自定义的Hook`useFriendStatus`就会取消对上一个朋友的订阅，订阅选择后的新朋友的状态

### 另一个例子：`useYourImagination()`

> 这部分的内容，由于我没有接触过Redux，所以不能很好的理解代码的功能。下一个任务就是学习Redux，需要回过头来再重新学习这部分

自定义Hook提供了React组件之前不具备的共享逻辑的灵活性。自定义Hook可以实现很多的功能，比如表单处理、动画、事件订阅、计时器等等。此外，Hooks使用起来就想使用React内置的特性一样轻松。

不要过早开始抽象逻辑。现在函数式组件功能更强大，你代码库中的函数式组件的平均长度会因此增长不少。这很正常，你没有必要立刻将它们分割为Hooks。但我们还是鼓励在有需要隐藏简单接口后的辅助逻辑时、或者分解复杂组件时开始使用自定义Hook

举个例子，下面是一个通过Hooks方式管理的复杂的组件，包含了许多局部变量。`useState`并没有将更新逻辑集中变得更容易，所以也许你更倾向于使用Redux的reducer的形式来实现：

```JS
function todosReducer(state, action) {
  switch (action.type) {
    case 'add':
      return [...state, {
        text: action.text,
        completed: false
      }];
    // ... other actions ...
    default:
      return state;
  }
}
```

reducer非常易于独立测试以及表达复杂的更新逻辑，如果需要还可以被分解为粒度更小的reducer。你也许习惯了使用React局部变量带来的优势，并不愿意再引入另外的库。

但如果编写一个`useReducer`的Hook帮助我们管理组件中的reducer中的局部变量呢？简单的实现如下：

```JS
function useReducer(reducer, initialState) {
  const [state, setState] = useState(initialState);

  function dispatch(action) {
    const nextState = reducer(state, action);
    setState(nextState);
  }

  return [state, dispatch];
}
```
可以在组件中使用这个Hook，使用reducer进行它的状态管理：

```JS
function Todos() {
  const [todos, dispatch] = useReducer(todosReducer, []);

  function handleAddClick(text) {
    dispatch({ type: 'add', text });
  }

  // ...
}
```
在复杂组件中利用reducer管理局部变量是非常常见的需求，所以React提供了的内置的`useReducer`Hook。

## Hooks API

React提供了一系列内置Hooks，这里简单介绍一下吧，详细的参考[文档](https://reactjs.org/docs/hooks-reference.html)。

分为两大类，基础Hook和附加Hook

基础Hook包括：

- `useState`
- `useEffect`
- `useContext`

附加Hook包括：

- `useReducer`
- `useCallback`
- `useMemo`
- `useRef`
- `useImperativeHandle`
- `useLayoutEffect`
- `useDebugValue`

### `useState`

```JS
const [state, setState] = useState(initialState);
```
返回一个状态变量和更新函数。

要注意的是，在每次渲染期间，`useState`返回的第一个值将始终是应用更新后的最新的状态。

#### 函数式更新

在使用`setState`更新时，可以直接传递一个结果，这个结果将直接赋值给`state`，如果更新的结果与上一个状态有关系，那可以使用函数式更新，`setState`的参数是一个函数，函数的参数是之前的值，返回的是更新的值

```JS
setState((prevState => prevState + 1)
```

#### 手动合并对象

注意，与class组件中的`setState`方法不同，`useState`不会自动合并更新对象：

比如，在class组件中：

```
class Test extends React.Component {
  state = { a: 1, b: 2, };

  render() {
    console.log(this.state);
    return (
      <div>
        <button onClick={() => this.setState({ a: 100 })}>click</button>
      </div>
    );
  }
}
```
点击按钮，`setState`会自动将对象合并，打印结果是：

```BASH
{a: 100, b: 2}
```

而在使用Hooks的组件中中：

```
function Test() {
  const [state, setState] = useState({ a: 1, b: 2 });
  console.log(state);
  return (
    <div>
      <button onClick={() => setState({ a: 100 })}>click</button>
    </div>
  );
}
```

`useState`在更新时不会将对象合并，所以打印的结果是：

```BASH
{a: 100}
```

所以需要手动进行合并，采取函数式赋值的方式：

```
setState(prevState => ({ ...prevState, a: 100 }))
```
这样才能保证更新后的对象是我们想要的对象。

#### 延迟初始化

`useState`的参数`initialState`是首次渲染期间使用的状态，在后续的更新渲染过程中，它会被忽略，因为`state`会采用上一次更新后最新的值，但是如果这个初始状态仍然会被计算一次：

```
function initialize() {
  console.log('initialize');
  return 1;
}

function Test() {
  const [state, setState] = useState({ a: initialize(), b: 2 });
  console.log(state);
  return (
    <div>
      <button onClick={() => setState(prevState => ({ ...prevState, a: 100 }))}>click</button>
    </div>
  );
}
```
在首次渲染时`initialize`会被调用，当点击按钮组件更新时，`initialize`仍然会被调用，但是结果却被遗弃了。如果这个状态是一个高开销的计算结果，可以改为提供函数，这个函数仅在初始渲染时执行，可以避免性能浪费：

```
const [state, setState] = useState(() => ({ a: initialize(), b: 2 }));
```
将`useState`的参数改为函数，在后续渲染时，初始状态计算就会被跳过了。

### `useEffect`

```
useEffect(didUpdate);
```
接受一个函数，默认在每次渲染后执行。

#### 清理effect

如果在effect中创建的一些事件需要在组件卸载时清理，比如定时器或者事件订阅等，可以为effect的更新函数返回一个新的函数，这个函数可以作为清理函数，在每次渲染时组件删除前执行，防止内存泄漏。

```JS
useEffect(() => {
  const subscription = props.source.subscribe();
  return () => {
    // Clean up the subscription
    subscription.unsubscribe();
  };
});
```

如果组件渲染多次，在执行下一个effect之前都会先执行清理函数。

#### `useEffect`的执行时机

`useEffect`中的更新函数会延迟到layout和paint后触发，也就是说在浏览器更新屏幕之后才会触发，因为它所针对的事件是订阅等事件处理程序不应该组织UI界面的更新。

但是有一些事件不能推迟，比如用户可见的DOM改变必须在下一次绘制之前同步触发，避免用户感觉到操作与视觉的不一致性。对于这个类型的事件需要在`useLayoutEffect`中触发，它与`useEffect`的不同就是在触发时机上的不同。

虽然`useEffect`延迟到浏览器绘制完成之后执行，但是它保证在任何新渲染之前触发。


#### 避免重复渲染

`useEffect`默认的表现是在每次渲染后触发，当组件的任何一个状态发生改变时，更新函数都会执行：

```
export default function () {
  const [count1, setCount1] = useState(0);
  const [count2, setCount2] = useState(0);

  useEffect(() => {
    console.log(count2);
  });

  return (
    <div>
      <p>You clicked {count1} times</p>
      <button onClick={() => setCount1(count1 + 1)}>Add Count1</button>
      <button onClick={() => setCount2(count1 + 1)}>Add Count2</button>
    </div>
  );
}
```
每更新`count1`的值时，`useEffect`都会执行，如果我们只希望打印`count2`的`useEffect`只在`count2`更新时执行，可以给`useEffect`传递第二个参数，它是更新时所依赖的值组成的数组

```JS
useEffect(() => {
  console.log(count2);
}, [count2]);
```
这样，这个更新函数只有在`count2`发生改变时才会执行。

如果传入一个空数组`[]`那就意味着告诉React这个更新函数不依赖于组件中的任何值，仅仅在首次渲染时执行，在组件销毁时执行清理，从不在更新时运行。

这个数组并不会作为参数传递给更新函数内部，但是从概念将，更新函数中引用的每个值都应该出现在输入数组中，这样才能避免更新函数依赖的某个值发生了变化，而函数没有重新执行。ESLint的插件会自动检测并插入这个数组，推荐使用。

#### `useEffect`中获取本次渲染更新后的值

==在`useEffect`的更新函数中，拿到的`state`和`props`总是当次渲染的初始值==，牢记这一点很重要，例如：

```
export default function () {
  const [count1, setCount1] = useState(0);
  const [count2, setCount2] = useState(0);

  // useEffect1
  useEffect(() => {
    setCount1(100);
    console.log(count1, 'useEffect1');
  }, [count1, count2]);

  // useEffect2
  useEffect(() => {
    console.log(count1, 'useEffect2');
  });

  return (
    <div>
      <p>{count1}</p>
      <button onClick={() => setCount2(count1 + 1)}>Add Count2</button>
    </div>
  );
}
```
分析一下这个组件的执行结果：

（1）首次渲染，`useEffect1`执行，`setCount1(100)`，这个时候打印的结果是多少？

（2）继续向下，`useEffect2`执行，打印的结果是多少？

（3）此时屏幕上显示的多少？

（4）是否会继续执行？

再重复一遍，在==在`useEffect`的更新函数中，拿到的`state`和`props`总是当次渲染的初始值==，==即便执行了`setState`之后仍是这样==，所以：

（1）首次渲染，`useEffect1`执行，`setCount1(100)`，这时`count1`的值仍然是初始值`0`，所以这个时候打印的结果是`0 "useEffect1"`

（2）继续向下，`useEffect2`执行，同样获取的`count1`的值仍然是当次渲染的初始值，所以打印的结果是`0 "useEffect2"`

（3）此时屏幕上显示的结果，在`return`时获取的`count1`的值才是经过了`useEffect`中更新后的值，所以屏幕上显示`0`

（4）由于在这轮渲染中，`count1`的值由`0`变为了`100`，而`useEffect1`依赖了`count1`的值，`useEffect2`默认每次更新后执行，所以会仍然执行。所以在下一轮次的渲染时会打印出`100 "useEffect1"`和`100 "useEffect2"`，屏幕上显示`100`。

所以完整的打印结果是：

```BASH
0       "useEffect1"
0       "useEffect2"
100     "useEffect1"
100     "useEffect2"
```

实际上可以认为每次渲染时通过`useState`声明的状态是不可变的（Immutable），每次渲染都会对它拍一个快照保存下来，当状态更而重新渲染时就会形成N个状态。不光是`state`和`props`通过快照的形式保存，组件的事件处理和`useEffect`都是同样的形式，例如：

```
export default function () {
  const [count, setCount] = useState(1);

  const log = () => {
    setTimeout(() => {
      console.log(count);
    }, 3000);
  };

  return (
    <div>
      <p>{count}</p>
      <button onClick={() => {
        log();
        setCount(100);
      }}>
        Set future count
      </button>
    </div>
  );
}
```
最终代码打印的结果是`1`，而不是`100`，屏幕上显示的`100`，这是因为在首次渲染时，`log`方法被执行，执行延时函数，这时候的`count`快照值是`1`，继续执行`setCount(100)`导致重新渲染，这一次渲染时`count`变为了`100`，显示在屏幕，3s后打印的`count`是首次渲染时保存的快照，所以结果是`1`

我犯过的一个错误就是，在`useEffect1`中通过`setState`更新了`count1`的值，而在`useEffect2`中要使用更新后的`count1`的值，这就会导致错误，因为在任何一个`useEffect`中拿到的`count1`的值是当次更新的`count1`的初始值，而不会是在`useEffect1`中更新后的值。

如何解决这个问题呢？我觉得有两个方法，一个是更好的组织`useEffect`，一个`useEffect`中不要完成过多的功能呢，更不要成为一个中间过程，为最终渲染的结果提供中间数据，而是让每个`useEffect`都提供渲染需要的最终数据。

如果确实有这种需要，那么就需要使用React提供了另外一种内置Hook了，`useRef`。

> 注意，在渲染结果中拿到的都是更新后的最新的`props`和`state`，如果在渲染结果中出现了旧的`props`和`state`，那么很可能是遗漏了一些依赖，导致对应的`useEffect`没有按照预期执行。还是推荐使用前面提到的ESLint的插件来帮助我们发现和解决问题。

### `useRef`

```JS
const refContainer = useRef(initialValue);
```

`useRef`返回一个可变的对象，其`current`属性被初始化为传递的参数，返回的这个对象就保留在组件的生命周期中。

`useRef`返回的`ref`对象在所有Render过程中保持着唯一引用，如果认为`state`是不可变的数据，那么`ref`对象就可以认为是可变对象，对`ref.current`的赋值和取值，拿到的额都是一个状态：

```
export default function () {
  const count = useRef(1);

  // useEffect1
  useEffect(() => {
    // 赋值
    count.current = 100;
    console.log(count, 'useEffect1');
  });

  // useEffect2
  useEffect(() => {
    // 赋值
    count.current = 200;
    console.log(count, 'useEffect2');
  });
  return (
    <div />
  );
}
```

使用`useRef`就可以再当次渲染获取到改变后的值，所以打印结果是：

```BASH
{current: 100} "useEffect1"
{current: 200} "useEffect2"
```

要注意，避免在渲染过重中（`return`中）设置直接饮用，可能会导致预料之外的结果，相反，应该只在事件处理程序和`useEffect`中修改、使用引用。

其他内置的Hook目前还没用到，简单看下：

### `useContext`

```JS
const context = useContext(Context);
```

用来创建context对象，当使用[Context API](https://duola8789.github.io/2019/01/25/01%20%E5%89%8D%E7%AB%AF%E7%AC%94%E8%AE%B0/03%20React/React%E6%8F%90%E9%AB%9807%20Context/)来传递数据时有用。

### `useReducer`

```JS
const [state, dispatch] = useReducer(reducer, initialState，initialAction);
```

useState的替代方案，接受类型为`(state, action) => newState`的Reducer，返回与`dispatch`方法匹配的当前状态。`initialAction`是可选的，提供初始的`action`

```
import React, { useReducer } from 'react';

const initialState = { count: 0 };

function reducer(state, action) {
  const { type } = action;
  switch (type) {
    case 'reset': {
      return initialState;
    }
    case 'increment': {
      return { count: state.count + 1 };
    }
    case 'decrement': {
      return { count: state.count - 1 };
    }
    default: {
      return state;
    }
  }
}

export default function () {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <div>
      <p>{state.count}</p>
      <button onClick={() => dispatch({ type: 'reset' })}>Reset</button>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
    </div>
  );
}
```

这个时候的`state`有reducer得来，更新方法dispatch是匹配reducer的`dispatch({type: 'type'})`更新方法。

[用它配合`useContext`可以避免在多层组件中深度传递回调的需要](http://react.html.cn/docs/hooks-faq.html#how-to-avoid-passing-callbacks-down)。

### `useCallback`

```JS
const memoizedCallback = useCallback(
  () => {
    doSomething(a, b);
  },
  [a, b],
);
```
主要的目的是，如果一个函数依赖了组件的`state`，并且由于复用的原因，不能放在`useEffect`中，就将这个函数用`useCallback`包装，返回的变量可以作为对应的`useEffect`的依赖，当其依赖发生变化时，返回新的函数引用，同时触发对应的`useEffect`重新执行。

我理解使用的原因主要出于性能优化和便于维护，例如：

```
export default function () {
  const [count1, setCount1] = useState(0);
  const [count2, setCount2] = useState(100);

  function fetch() {
    console.log('fetch');
    return count1 + 1000;
  }

  useEffect(() => {
    console.log(fetch());
  });

  return (
    <div>
      <button onClick={() => setCount1(count1 + 1)}>Add Count1</button>
      <button onClick={() => setCount2(count2 + 1)}>Add Count2</button>
    </div>
  );
}
```
其中的`useEffect`无法添加`fetch`作为依赖，因为它是一个普通的函数，而且每次渲染`fetch`都会生成一个快照，如果使用了`useCallback`：

```
export default function () {
  const [count1, setCount1] = useState(0);
  const [count2, setCount2] = useState(100);

  const fetch = useCallback(() => {
    console.log('fetch');
    return count1 + 1000;
  }, [count1]);

  useEffect(() => {
    console.log(fetch());
  }, [fetch, count1, count2]);

  return (
    <div>
      <button onClick={() => setCount1(count1 + 1)}>Add Count1</button>
      <button onClick={() => setCount2(count2 + 1)}>Add Count2</button>
    </div>
  );
}
```
使用了`useCallback`之后，依赖更清晰，并且在`count1`未发生变化时不会生成新的快照，有助于性能的提高。

### `useMemo`

```JS
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

与`useCallback`类似，返回的是一个不生成快照的对象，而非函数。

`useMemo`只会在其中一个输入发生更改时重新计算，此优化有助于避免在每个渲染上进行高开销的计算。

### `useLayoutEffect`

前面介绍过，与`useEffect`的不同点仅仅在于执行时机不同，`useLayoutEffect`在绘制前同步触发，`useEffect`会推迟到绘制后触发




## 参考

- [Hooks API Reference@React](https://reactjs.org/docs/hooks-reference.html)
- [Hooks API 参考@React中文文档](http://react.html.cn/docs/hooks-reference.html)
- [精读《useEffect 完全指南》@掘进](https://juejin.im/post/5c9827745188250ff85afe50#heading-8)
- [useEffect 完整指南@overreacted](https://overreacted.io/zh-hans/a-complete-guide-to-useeffect/)







https://reactjs.org/docs/hooks-overview.html

