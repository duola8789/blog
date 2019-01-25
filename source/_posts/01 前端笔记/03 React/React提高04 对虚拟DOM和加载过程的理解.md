---
title: React提高04 对虚拟DOM和加载过程的理解
top: false
date: 2019-01-25 16:05:43
tags:
- React
- 坑
categories: React
---

也是面试时比较常遇到的React的问题之一。

了解了之后还是能够加深对React的理解。

<!-- more -->

![image](http://upload-images.jianshu.io/upload_images/1814354-4bf62e54553a32b7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)


## React虚拟DOM的理解

虚拟DOM是用JS的对象结构模拟出html中DOM的结构，批量的增删改查，由于直接操作的是JS对象，所以速度要比操作真实DOM要快，最后更新到真正的额DOM中。

虚拟DOM构建的对象，除了dom相关属性，还包括了React自身需要的属性，比如ref，key等，大概如下结构：


```
{
  type: 'div',
  props: {
    className: 'xxx',
    children: [{
      type: 'span',
      props: {
        children: ['CLICK ME']
      },
      ref: key:
    }, {
      type: Form,
      props: {
        children: []
      },
      ref: key:
    }] | Element
  }
  ref: 'xxx',
  key: 'xxx'
}
```

## react何时将虚拟DOM渲染为真实DOM

render这个步骤就是react组件挂载的步骤

> react组件挂载：将组件渲染，并构建DOM元素然后插入页面的过程

render的步骤是创建虚拟DOM，挂载组件，

在render之后，将这个虚拟 dom 树真正渲染成一个 dom 树，插入了页面，可以认为是在componentDidMount这个步骤完成的，该方法被调用时，已经渲染出真实的 DOM

在组件存在期，componentDidUpdate与componentDidMount类似，在组件被重新渲染后，此方法被调用，真实DOM已经渲染完成

## react不能setState的步骤

shouldComponentUpdate和componentWillUpdate就会造成循环调用，使得浏览器内存占满后崩溃

## virtual DOM 的理解

1. 用 JavaScript 对象结构表示 DOM 树的结构；然后用这个树构建一个真正的 DOM 树，插到文档当中
2. 当状态变更的时候，重新构造一棵新的对象树。然后用新的树和旧的树进行比较，记录两棵树差异
3. 把2所记录的差异应用到步骤1所构建的真正的DOM树上，视图就更新了

Virtual DOM 本质上就是在 JS 和 DOM 之间做了一个缓存。可以类比 CPU 和硬盘，既然硬盘这么慢，我们就在它们之间加个缓存：既然 DOM 这么慢，我们就在它们 JS 和 DOM 之间加个缓存。CPU（JS）只操作内存（Virtual DOM），最后的时候再把变更写入硬盘（DOM）。


## 为什么虚拟DOM比原生DOM性能更高

React 的基本思维模式是每次有变动就整个重新渲染整个应用，相当于设置 innerHTML，只不过它设置的是内存里面 Virtual-DOM，而不是真实的 DOM。

React.js 厉害的地方并不是说它比 DOM 快(这句话本来就是错的)，而是说不管你数据怎么变化，我都可以以最小的代价来更新DOM。方法就是在内存里面用新的数据刷新一个虚拟的DOM数，然以后新旧DOM树进行比较，找出差异，再将差异更新到真正的DOM 树上。

原生DOM性能低，因为DOM 的规范迫使浏览器在实现的时候为每一个 DOM 元素添加了非常多的属性，然而这其中很多我们都用不到

而React对虚拟DOM的属性进行了精简，非常轻量化，并且使用了diff的算法，比较前后差异，最后只把变化的部分一次性应用到真实的DOM树

> React 的变动检查由于是 DOM 结构层面的，即使是全新的数据，只要最后渲染结果没变，那么就不需要做无用功。

更正确的说法应该是：虚拟DOM不一定比原生DOM性能高，但是让开发者更省心的更新数据

> 首先, 虚拟dom并没有比直接原生操作更快, 所谓"快"是有条件的
比如点赞, 数字+1, 直接操作dom会更快.
> 
> 如果你能自己捋请规则, 每回手动操作dom的时候, 都只改动应该改变的, 那dom操作永远比虚拟dom快.
> 
> 但如果你的改动勾连的地方很多, 而且要保持状态, 那虚拟dom的自动diff无疑会让你省更多心.
> 
> 比如一个列表, 列表项有点赞等状态, 回复数量等信息, 有动态新增, 有动态加载, 这时候你要直接操作dom会很繁琐.
> 
> 虚拟dom的核心在于diff, 它自动帮你计算那些应该调整, 然后只修改该修改的区域, 省下的不是运行速度这种小速度, 而是开发速度/维护速度/逻辑简练程度等"总体速度"

## diff算法

比较两棵DOM树的差异是 Virtual DOM 算法最核心的部分这也是所谓的 Virtual DOM 的 diff 算法。两个树的完全的 diff 算法是一个时间复杂度为 O(n^3) 的问题。但是在前端当中，你很少会跨越层级地移动DOM元素。所以 Virtual DOM 只会对同一个层级的元素进行对比：

![image](https://pic2.zhimg.com/50/6d64b0b7889e7f020bb020aea5947a09_hd.jpg)

上面的div只会和同一层级的div对比，第二层级的只会跟第二层级对比。这样算法复杂度就可以达到 O(n)。

## 参考
- https://www.zhihu.com/question/31809713?sort=created
- https://segmentfault.com/q/1010000009401008?_ea=1922362/*&^%$
- https://www.zhihu.com/question/29504639
