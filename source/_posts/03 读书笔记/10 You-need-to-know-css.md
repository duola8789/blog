---
title: 10 You-need-to-know-css
top: false
date: 2019-06-30 08:58:00
updated: 2019-07-01 11:24:00
tags: 
- CSS
categories: 读书笔记
---

CSS练习项目，每天练一练。前期按照[CSS Tricks](https://lhammer.cn/You-need-to-know-css/#/)进行练习。

<!-- more -->

## 透明边框

默认情况下，背景的颜色会延伸至边框下层，所以如果边框设置为透明色，会被背景色覆盖掉。

可以设置CSS3的属性[`background-clip`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/background-clip)设置元素的背景（背景图片或颜色）是否延伸到边框下面。

`background-clip`取值有四个：

- `border-box`: 背景延伸至边框外沿（但是在边框下层）
- `padding-box`: 背景延伸至内边距（`padding`）外沿，不会绘制到边框外
- `content-box`: 背景延伸至内容区（`content box`）外沿
- `text`：背景剪裁成为文字的前景色。（只有Chrome支持，需加`-webkit-`前缀）

![](http://image.oldzhou.cn/FpKGstS3QYG5aEbxFSCWYfrP7VlE)

所以将`background-clip`设置为`padding-box`就可以实现透明边框。

*参考*：

- [半透明边框@You-need-to-know-css](https://lhammer.cn/You-need-to-know-css/#/zh-cn/translucent-borders)
- [background-clip@MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/background-clip)

## 多重边框

### `box-shadow`

`box-shadow`用来产生阴影效果，如果只给出两个数值，那么浏览器解析为`x`方向偏移量和`y`方向偏移量；如果给出第三个值，将被解释为模糊半径的大小；如果给出第四个值，将被监事未扩展半径的大小。

另外两个属性是`inset`属性（声明式）和颜色值。

一直对模糊半径和扩展半径有一些糊涂，从[下面这个例子](https://juejin.im/post/5ce290cc6fb9a07ea8039d83)了解`box-shadow`的生成过程

首先定义出`box-shadow`属性：

```CSS
.box {
  width: 300px;
  height: 200px;
  background-color: yellowgreen;
  box-shadow: 30px 50px 20px #000;
}
```
（1）首先在`box`元素生成一个同`box`大小、形状完全相同的阴影，颜色为设置的颜色`#000`

![](http://image.oldzhou.cn/FsV6IWhYI7_Q1bgL6TF7OHma9E0Z)

此时，阴影在元素的上层

（2）根据`offset-x`和`offset-y`，将阴影从上层向右移动`30px`，向上移动`50px`

![](http://image.oldzhou.cn/FrF8Xmp5uh3UukOsaYQiaLaG9vwo)

（3）然后根据设置的`20px`的模糊半径，然后在四个方向上，每个方向的阴影边缘为中心，向两侧各扩展`10px`的区域，作为高斯模糊的半径

以右侧为例：

![](http://image.oldzhou.cn/FkPYqMg8GKKPhsJ9sajTyZhABVEw)

（4）最后一步，将阴影与元素重叠的区域剪裁掉，如下图：

![](http://image.oldzhou.cn/FtG6m6JEX1SUeZn6p7SgdTuJpcvq)

（5）得到最终效果：

![](http://image.oldzhou.cn/Fo2YmjK0hJYLXI_R1LH5Ll4iZXE7)

从上面的例子看出，模糊半径是是以阴影边缘为中心，向两侧扩展一半，作为半径，进行高斯模糊。只能取正值。

扩展半径，是将阴影的面积增大，在它取值为`0`时，阴影面积是等于元素的面积的，它不等于`0`时，取值是四个方向增大或者减小的尺寸

```CSS
.inner {
  box-shadow: 30px 50px 0 30px #000;
}
```

![](http://image.oldzhou.cn/Fn4GZlUhMnmHGxpUWPlUJd0Xx6Q9)

上图的虚线尺寸就是增加的扩展半径`30px`，当设置了扩展半径，模糊半径也会从增大后的阴影边缘向两侧模糊。

可以通过将`x-offset`、`y-offset`、模糊半径都设为`0`，设置不同尺寸的扩展半径，来实现多重边框的效果：

```CSS
.inner {
  box-shadow: 0 0 0 5px darkgoldenrod, 0 0 0 10px darkolivegreen, 0 0 0 15px red;
}
```

![](http://image.oldzhou.cn/FswhNgLNUD7PxJLo2woa0kCXqvXa)

优点是很容易实现2条以上的边框，且可以实现圆角边框，缺点是无法实现非实线的边框。


### `outline` + `outline-offset`

`outline`可以设置一个或者或者多个单独轮廓属性，轮廓不占据空间（`box-shadow`和`border`都占据空间）

`outline-offset`用来设置一个`outline`与一个元素边缘或者边框的间隙

将两者结合，就可以实现多种边框，并且可以实现`box-shadow`无法实现的，非实线的多重边框

```CSS
.inner2 {
  border: aqua dotted 5px;
  outline: red dotted 5px;
  outline-offset: -15px;
}
```

![](http://image.oldzhou.cn/FrXgZTN2joXiIvqj7YStdIl2XUA2)

优点是可以实现非实线的边框，缺点是实现2条以上的边框不方便，且无法实现圆角边框。


*参考*

- [多重边框@You-need-to-know-css](https://lhammer.cn/You-need-to-know-css/#/zh-cn/multiple-borders)
- [box-shadow@MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/box-shadow)
- [box-shadow详解@掘金](https://juejin.im/post/5ce290cc6fb9a07ea8039d83)
- [outline@MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/outline)
- [outline-offset@MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/outline-offset)

## 边框内圆角

要实现的效果：

![](http://image.oldzhou.cn/FluW1mYFqLlYY8D1bBtJLU4cJHmL)

边框外围没有圆角，内部有圆角，所以单独使用`border-radius`是不行的。上一部分内容中，我们可以发现，`box-shadow`是跟随边框有圆角的，而`outline`则是没有圆角的，所以我们可以使用三者的组合实现边框内圆角。

利用`border-radius`设置圆角，假设为`r`，但是不设置`border`，设置`outline`为`w`，这样外边框是没有圆角的，


```CSS
.inner {
  background: darkgray;
  border-radius: 5px;
  outline: 5px solid darkgreen;
}
```
这时候效果是：

![](http://image.oldzhou.cn/FpHhAgjiahF6PeuxsF_XaqwZb0-i)

背景会由于`border-radius`的设置出现圆角，然后我们用`box-shadow`将这部分补上，只设置其扩展半径，颜色与`outline`的颜色相同

```CSS
.inner {
  background: darkgray;
  border-radius: 5px;
  outline: 5px solid darkgreen;
  box-shadow: 0 0 0 5px darkgreen;
}
```
扩展半径的取值范围见下图：

![](http://image.oldzhou.cn/FgsHRYQ-1rkRVFYStGFTgFUGFWCl)

复习一下扩展半径的值：

![](http://image.oldzhou.cn/Fn4GZlUhMnmHGxpUWPlUJd0Xx6Q9)

所以这里取值最大值不能超过`outline`的宽度`w`，而最小宽度根据勾股定理不能小于`$\sqrt{2}r - r$`，即`$\sqrt{2-1}r$`，在我们上面的设置中，扩展半径的取值范围是`[5, 2.07]`


*参考*：

- [边框内圆角@You-need-to-know-css](https://lhammer.cn/You-need-to-know-css/#/zh-cn/inner-rounding)
