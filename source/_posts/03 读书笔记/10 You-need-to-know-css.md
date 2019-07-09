---
title: 10 You-need-to-know-css
top: false
date: 2019-06-30 08:58:00
updated: 2019-07-09 20:35:25
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

> 参考：
> - [半透明边框@You-need-to-know-css](https://lhammer.cn/You-need-to-know-css/#/zh-cn/translucent-borders)
> - [background-clip@MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/background-clip)

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

> 参考
> - [多重边框@You-need-to-know-css](https://lhammer.cn/You-need-to-know-css/#/zh-cn/multiple-borders)
> - [box-shadow@MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/box-shadow)
> - [box-shadow详解@掘金](https://juejin.im/post/5ce290cc6fb9a07ea8039d83)
> - [outline@MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/outline)
> - [outline-offset@MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/outline-offset)

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

所以这里取值最大值不能超过`outline`的宽度`w`，而最小宽度根据勾股定理不能小于`$\sqrt{2}r - r$`，即`$\sqrt{2-1}r$`，在我们上面的设置中，扩展半径的取值范围是`[2.07, 5)`


> 参考：
> - [边框内圆角@You-need-to-know-css](https://lhammer.cn/You-need-to-know-css/#/zh-cn/inner-rounding)

## 背景定位

### [`background-position`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/background-position)

![](http://image.oldzhou.cn/FrnFBglSd0nXpgo1VAOqqMONVf8R)

我比较熟悉的背景定位是使用`background-position`，但是都是只指定了两个属性值：

```CSS
.backgroundPosition {
  background-position: right bottom;
}
```

有三个点要注意：

1. `background-position`指定的位置是相对于由`background-origin`定义的位置图层的，默认定位于元素的左上角
2. `background-position`在一个方向上，可以指定一个值（例如上面），也可以指定两个值，可以在`right`后面跟一个数值，表示相对于边缘的位置
3. 如果指定的是百分比，`0%`代表图片的左（上）边界和容器的左（上）边界重合，`100%`代表图片的右（下）边界与容器的右（下）边界重合，`50%`代表图片的中心与容器的中心重合

```CSS
.inner {
  width: 50%;
  height: 200px;
  margin: 0 auto;
  padding: 20px;
  background: #fff url('../assets/images/css-tricks.png') no-repeat;
  background-size: 50px;CSS
}

.backgroundPosition {
  background-position: right 20px bottom 20px;
}
```
另外，这个属性是可以融合到`background`属性中的：

```CSS
.backgroundPosition {
 background: #fff url('../assets/images/css-tricks.png') no-repeat right 20px bottom 20px;
}
```

### [`background-origin`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/background-origin)

刚才也提到了，`background-position`指定的位置是相对于由`background-origin`定义的位置图层的。`background-origin`规定了背景图片的属性的原点位置与容器的关系（与`background-clip`类似）

可以取值有：

- `border-box`: 图片边界与`border`重合
- `padding-box`: 图片边界与`padding`区域重合
- `content-box`: 图片边界与`content`区域重合

![](http://image.oldzhou.cn/Fk6jnaJ2R33O7q7j93Z-lMh4gnZ0)

所以，可以取值`content-box`，借用`padding`定位实现。

```CSS
.inner {
  padding: 20px;
  border: 10px solid red;
  background: #fff url('../assets/images/css-tricks.png') no-repeat right bottom;
  background-origin: content-box;
}
```

> 参考：
> - [背景定位@You-need-to-know-css](https://lhammer.cn/You-need-to-know-css/#/zh-cn/extended-bg-position)
> - [background-position@MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/background-position)
> - [background-origin@MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/background-origin)

## 条纹进度条

可以使用`background`的`repeating-linear-gradient`属性，来生成一个静态的进度条，再结合动画属性，实现动态的进度条样式。

![](http://image.oldzhou.cn/FhYUUEkaUI0ZKKsS-9Uz_s4ItDWm)

[`linear-gradient`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/linear-gradient)可以实现线形重复渐变效果。

[`repeating-linear-gradient`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/repeating-linear-gradient)类似`linear-gradient`，并且采用相同的参数，但是它会在所有方向上重复渐变以覆盖其整个容器。

它分为两组参数，第一组参数用来描述渐变线的起点位置，`to top`这样的值会转换为0度，`to left`会被转换为270度。第二组参数可以包含多个值，用逗号分隔，每个值都是一个色值，后面跟随一个可选的终点位置，终点位置可以是百分比也可以是绝对值

```CSS
/* 一个倾斜45度的重复线性渐变,
   从蓝色开始渐变到红色 */
linear-gradient(45deg, blue, red);
```

![](http://image.oldzhou.cn/FrfiO8WADiybGVVVZEaqCzegt6Pt)

```CSS
/* 一个从右下角到左上角的重复线性渐变,
   从蓝色开始渐变到红色 */
linear-gradient(to left top, blue, red);
```

![](http://image.oldzhou.cn/Fk3UAB64mjf_QRIBHnCjpd9OJSxM)

```CSS
/* 一个由下至上的重复线性渐变,
   从蓝色开始，40%后变绿，
   最后渐变到红色 */
linear-gradient(0deg, blue, green 40%, red);
```

![](http://image.oldzhou.cn/FmQdq_CuWwjIoiPrNLZDG7ygKc3y)

要想实现突然变色，而非渐变色，需要为两个颜色制定同一个终点位置，那么这两个颜色会产生一个无线小的过渡区域，从效果上看，颜色会在那个位置突然变化，而不是一个平滑的过程。

```CSS
background: linear-gradient(90deg, blue 50%, red 50%);
```

![](http://image.oldzhou.cn/Fu8bj-9f2y3nll4eB0pa23e7-WIX)

上面的写法的可维护性不太好，因为每次修改尺寸时都需要修改两处，但是CSS规范规定，如果某个色标的位置值比整个列表在它之前的位置都要小，则改色标的位置会被设置为它前面的额所有色标位置值的最大值

所以上面的代码也可以改为：

```CSS
background: linear-gradient(90deg, blue 50%, red 0);
```

色块的大小是可以通过`background-size`来控制的：

```CSS
background: linear-gradient(90deg, blue 50%, red 0);
background-size: 16px;
```

![](http://image.oldzhou.cn/FmbtnK0ucf0g_dQjb2KsLOB1S6d2)

在就将角度倾斜，通过`animation`控制`background-position`，就可以实现动起来的效果了：

```
.inner:after {
  background: linear-gradient(90deg, blue 50%, red 0);
  background-size: 16px;
  animation: loading 10s linear infinite;
}
@keyframes loading {
  to {
    background-position: 100% 0 ;
  }
}
```

这样可以实现水平方向条纹的横向移动

![](http://image.oldzhou.cn/FvfGizAT795CYaX8dkenO_Oi7GTe)

但是如果要是倾斜45度就出现问题了

```
.inner:after {
  background: linear-gradient(45deg, blue 50%, red 0);
  background-size: 16px;
  animation: loading 10s linear infinite;
}
```

![](http://image.oldzhou.cn/Fgpe2VMO_K0CPIc8uLoeG-Nr6KAA)

这个时候应该改为使用`repeating-linear-gradient`属性，在所有方向上重复渐变以覆盖其整个容器，使用这个属性时，色标值需要写完整：

```CSS
background: repeating-linear-gradient(90deg, blue 0 ,blue 50%, red 50%, red 100%);
```
所以这样可以实现斜着45度的进度条：

```CSS
.inner:after {
  background: repeating-linear-gradient(45deg, red 0, red 25%, blue 0, blue 50%,
  red 0, red 75%, blue 0, blue 100%);
  background-size: 16px;
  animation: loading 10s linear infinite;
}
```

> 参考：
> - [条纹背景@You-need-to-know-css](https://lhammer.cn/You-need-to-know-css/#/zh-cn/stripes-background?id=%e8%bf%9b%e5%ba%a6%e6%9d%a1)
> - [linear-gradient@MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/linear-gradient)
> - [repeating-linear-gradient@MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/repeating-linear-gradient)
> - [聊一聊CSS3的渐变——gradient@掘金](https://juejin.im/post/5bc2fb09f265da0ac55e75e7)
