---
title: 零散专题32 生成PDF
top: false
date: 2019-06-13 16:59:43
updated: 2019-06-13 16:59:45
tags:
- PDF
- PDFKit
categories: 其他
---

JavaScript生成PDF方案调研，以及PDFKit调研。

<!-- more -->

## 生成pdf的方案

（1）JSPDF(前端生成 )

- 优点：不需要服务端安装无头浏览器，使用CSS方便控制样式，生成的PDF文字可复制
- 缺点：对中文支持不好
 
（2）PDFKit( 服务端生成)

- 优点：服务端直接解决，生成的PDF文字可复制，通过引入字体，可支持中文
- 缺点：样式控制复杂

（3）node-html-pdf(服务端生成)

- 优点：服务端通过控制HTML模板生成PDF，支持中文，样式控制方便
- 缺点：不支持图片，需要安装无头浏览器，性能有隐患

（4）JSPDF + HTMLToCanvas(前端生成)

- 优点：样式控制方便，支持中文，比较美观
- 缺点：生成的PDF内容是图片，无法复制

（5） 使用打印(前端生成)

- 优点：简单，代码量少
- 缺点：需要引导用户，且不美观

## PDFKit

### 简介

[PDFKit](http://pdfkit.org/)使用来在Node服务端生成PDF文件的JS包（也支持在浏览器使用），它可以轻松生成复杂的、多页的、可打印的PDF文档。

它的API是链式语法，与操作Canvas的API有一些类似

安装：

```BASH
npm install pdfkit -S
```

创建PDFKit文档很容易：

```JS
const PDFDocument = require('pdfkit');
const doc = new PDFDocument();
```

PDFDocument实例是可读的Node流，它不会自动的保存，但是可以使用`pipe`方法将输出的PDF文档传递给另一个可写的Node流。当PDF文档编写成功后，调用`end`方法来结束流程。下面的例子来展示如何将生成的文档传递给PDF文件或者HTTP响应：

```JS
doc.pipe(fs.createWriteStream('/path/to/file.pdf')); // 写入PDF文件
doc.pipe(res);                                       // 传递给HTTP响应

// 用后面介绍的API来为PDF添加内容

// 结束编辑，关闭流
doc.end();
```

### 在浏览器中使用

PDFKit的0.6版本后支持在浏览器中使用，有两种方法在浏览器端使用PDFKit，一种是在浏览器端使用[browserify](http://browserify.org/)加载Node模块，另一种方法是直接使用PDFKit的预编译版本，可以从[Github上下载](https://github.com/foliojs/pdfkit/releases)。

在浏览器端使用PDFKit和Node中使用的唯一区别就是输出结果，在浏览器端输出结果必须是浏览器支持的格式，比如[Blob](https://developer.mozilla.org/en-US/docs/Web/API/Blob)。Blob格式可以允许浏览器在一个iframe中直接展示生成的PDF文档，或者将PDF上传到服务器，或者让用户下载它。

将PDFDcument输出为Blob格式，需要将他传递给[blob-stream](https://github.com/devongovett/blob-stream)，这个模块可以将Node的流转换为Blob。下面的例子使用了Browserify来加载PDFKit和blob-stram（如果没有使用Browserify可以直接使用`<script>`标签代替）

```JS
// 引入依赖
const PDFDocument = require('pdfkit');
const blobStream  = require('blob-stream');

// 创建文档
const doc = new PDFDocument();

// 传递给Blob
const stream = doc.pipe(blobStream());

// 在这里添加PDF的内容

// 结束时得到的是Blob
doc.end();

stream.on('finish', function() {
  // 将blob转换为PDF文档
  const blob = stream.toBlob('application/pdf');

  // 或者得到Blob URL，直接在浏览器中展示
  const url = stream.toBlobURL('application/pdf');
  iframe.src = url;
});
```

### 添加页面

PDFKit文档的第一页是在创建文档时自动添加的，除非使用了`autoFirstPage: false`选项。后续的页面必须手动添加：

```JS
doc.addPage()
```

可以使用`pageAdded`事件，让每个页面创建后都添加上相同的内容：

```JS
doc.on('pageAdded', () => doc.text("Page Title"));
```

可以通过为`addPage`传递参数设置页面的尺寸、方向：

- `layout`，页面方向，可取值`portrait`（默认值）/`landscape`
- `size`，页面尺寸，取值是一个数组`[宽, 高]`，单位是PDF的点（1/72英寸）。可以传入字符串指定一些[预设值](https://github.com/foliojs/pdfkit/blob/b13423bf0a391ed1c33a2e277bc06c00cabd6bf9/lib/page.coffee#L72-L122)，默认值是`letter`
- `margin`，设定页边距，可以设为一个数字，那么各边距都等于它，也可以设为一个对象，分`top`/`left`/`bottom`/`right`四个属性分别设定各边距

```JS
// Add a 50 point margin on all sides
doc.addPage({
  margin: 50});


// Add different margins on each side
doc.addPage({
  margins: {
    top: 50,
    bottom: 50,
    left: 72,
    right: 72
  }
});
```

给PDFDocument构造函数传递页面参数对象，可以设置每一页的默认尺寸和布局，**它会被每个`addPage`传递的页面参数覆盖**。

### `bufferPage`

有的时候需要在后面的页面完成后回到前面的页面，对前面的页面进行修改，可以在PDFDocument构造函数传递一个参数`bufferPages: true`，来手动控制页面流。

一般这个情况可能不多见，具体可以参考[文档这部分内容](http://pdfkit.org/docs/getting_started.html#switching_to_previous_pages)。

### 设定文档基础信息

基础信息包括标题、作者等，可以通过对`doc.info`赋值，也可以在创建文档时传递参数实现。可以设定的基础信息包括（需要首字母大写）`Title`/`Author`/`Subject`/`Keywords`/`CreationDate`/`ModDate`

### 加密和访问权限

可以对PDF加密，并且使用密码打开文件。也可以设定PDF文件的访问权限。具体内容[参考文档](http://pdfkit.org/docs/getting_started.html#encryption_and_access_privileges)。


### 矢量图形

PDF格式兼容矢量图形，PDFKit提供了类似于HTML5 Canvas的API来创建矢量图形。图形通过一些列的直线和曲线构成，看一个例子：


```JS
doc.moveTo(0, 20)                              // 设定起点
  .lineTo(100, 160)                            // 直线
  .quadraticCurveTo(130, 200, 150, 120)        // 二次曲线
  .bezierCurveTo(190, -40, 200, 200, 300, 150) // 贝塞尔曲线
  .lineTo(400, 90)                             // 直线
  .stroke();                                   // 画
```

![](http://image.oldzhou.cn/FgCmy8HU_S_RSbq0ZY_oEy3WxNmU)

### SVG路径

PDFKit包含了SVG路径解析器，所以可以使用SVG路径画出一个图形，上面的图形使用SVG路径同样可以实现：

```JS
doc.path('M 0,20 L 100,160 Q 130,200 150,120 C 190,-40 200,200 300,150 L 400,90')
  .stroke()
```

### 图形小助手

PDFKit提供了一些封装好的方法来画出一些常用的图形，包括：

```JS
// 长方形 
rect(x, y, width, height) 
// 圆角长方形
roundedRect(x, y, width, height, cornerRadius) 
// 椭圆
ellipse(centerX, centerY, radiusX, radiusY = radiusX) 
// 圆形
circle(centerX, centerY, radius) 
// 多边形
polygon(points...) 
```
使用`ploygon`方法通过传输一系列由横纵坐标组成的数组，会创建一系列的直线，并连起来成为多边形：

```JS
doc.polygon([100, 100], [200, 200], [200, 300])
  .stroke()
```

![](http://image.oldzhou.cn/Fk5M94-QfYdoMI7tgsssP9V5uOOc)

## 边框和填充样式

使用`storke`是画线，使用`fill`画出来的是填满空间的实体，使用`fillAndStroke`同时实现：

```JS
doc.polygon([100, 100], [200, 200], [200, 300]).fillAndStroke('green', 'red')
```

![](http://image.oldzhou.cn/FsxXEsYK8TDrObBVxVyTo0X9Efbb)

PDFKit可以设定的属性有：

- `lineWidth`
- `lineCap`
- `lineJoin`
- `miterLimit`
- `dash`
- `fillColor`
- `strokeColor`
- `opacity`
- `fillOpacity`
- `strokeOpacity`

其中`lineCap`用来设定直线的端点形状，可取值有`butt`/`round`/`square`，`lineJoin`用来设定直线交汇处的形状，可取值有`miter`/`round`/`bevel`

![](http://image.oldzhou.cn/FnONkQ8gOQcW9qpcUvm1dzFriA-X)

### 虚线

使用`dash`方法画虚线：

```JS
doc.circle(100, 50, 50)
  .dash(5, { space: 10 })
  .stroke();
```
dash接受的第一个参数是每一段虚线的长度，第二个参数是一个选项对象，其中`space`属性用来指定每个虚线段的间隔，默认值与虚线段长度相等，`phase`属性指定虚线段的起点（不知道有什么用）

当使用`dash`方法后，后续的直线都是虚线的，可以使用`undash`方法恢复实现：

```JS
doc.moveTo(100, 50)
  .lineTo(100, 150)
  .dash(5, { space: 10, phase: 10 })
  .stroke();

doc.moveTo(100, 150)
  .lineTo(300, 150)
  .undash()
  .stroke();
```

![](http://image.oldzhou.cn/Fk5xy4jbX0W9miIfYazZaXJo9W2e)

### 颜色

可以使用一个数组表示RGB或者CMYK颜色，或者字符串的16进制颜色或者CSS颜色的名称

`fill`和`stroke`方法的参数可以指定颜色，也可以使用`fillColor`和`strokeColor`来指定颜色，第一个参数是颜色，第二个参数是透明度

也可以使用`fillOpacity`、`strokeOpacity`或者`opacity`来单独指定透明度

### 渐变

使用`linearGradient`和`radialGradient`实现渐变色，详见[文档](http://pdfkit.org/docs/vector.html#gradients)。


### 保存恢复图形堆栈

图形堆栈是所有创建的样式和移动的快照，每次调用`save`方法当前的图形堆栈就会被推入一个堆栈中，当调用`restore`方法后，堆栈中的最后一个状态就会被应用到环境中。

所以，你可以保存状态，改变一些样式，然后恢复到之前的状态。


### 移动

通过移动，可以再不改变图形本身的基础上，改变图形的样式。有三种移动的类型可用：`translate`/`rotate`/`sacle`

详见[文档](http://pdfkit.org/docs/vector.html#transformations)。

### 剪切

途径剪切与填充（fill）和连线（stroke）不同，它是一个蒙版，会隐藏掉图形中不想要的部分。

所有落在剪切路径内部的图形都是可见的，外部的都是不可见的。

```JS
// Create a clipping path
doc.circle(100, 100, 100)
   .clip();

// Draw a checkerboard pattern
for (let row = 0; row < 10; row++) {
  for (let col = 0; col < 10; col++) {
    const color = (col % 2) - (row % 2) ? '#eee' : '#4183C4';
    doc.rect(row * 20, col * 20, 20, 20)
       .fill(color);
  }
}
```

结果：

![](http://image.oldzhou.cn/FgVNwesxXwj9PE028GP8y_5eHoFe)

想要取消剪切，需要在`clip`之前调用`save`方法，完成剪切部分的操作后调用`restore`方法。

### 文字

使用`text`方法添加文字

```JS
doc.text('Hello world!')
```

每次调用`text`方法都会另起一行，并且自动与之前行的位置对其。可以为`text`方法传两个参数，指定其位置。

```JS
doc.text('hello1');
doc.text('hello2').text('hello3', 0, 0)
```

![](http://image.oldzhou.cn/Fr8W2cCb6LWOlefYslkMJ_lzOHxF)

可以调用`moveDown`或`moveUp`方法来按行移动

### 文字换行和对其

`text`方法接受一个对象作为参数，用来指定一些配置项。

在不传递任何参数的情况下，`text`方法生成的文字横向沿着页面的左边距排放，纵向沿着页面上边距摆放，后续文字排在已有文字的下方。

PDFKit会根据文字内容，自动添加下一页，无需手动控制。

PDFKit还提供了文字折行的功能。文字会自动换行，除非指定`lineBreak`为`false`。默认情况下文字遇到页面边距会换行，但是指定`width`属性会让文字按照不同的宽度换行。如果指定了`height`属性，文字会调整到能放下的最多的行数，多余的行会被剪切

```JS
doc.text(t, {
  width: 200
})
```

![](http://image.oldzhou.cn/FlAFrCFiMipWd__j5l6uAo2LQSig)

```JS
doc.text(t, {
  width: 200,
  height 200,
})
```
![](http://image.oldzhou.cn/FlAFrCFiMipWd__j5l6uAo2LQSig)


当文字在允许换行时，可以通过`align`属性指定对其方式，可取的值有`left`/`right`/`center`/`justify`

### 文字样式

`text`接受一系列的参数来指定文字样式。

![](http://image.oldzhou.cn/FrZ0qQEPazKcik5NHqAe5Xu6YGJb)

`ellipsis`为`true`时用来指定当文字太长时用省略号来代替多余的文字，可以传入字符串指定代替的字符

使用`columns`和`columnGap`来将文字按列排布

```JS
doc.fillColor('red')
  .text(t, {
  columns: 3,
  columnGap: 15,
  height: 100,
  width: 300,
  align: 'justify'
});
```
![](http://image.oldzhou.cn/FldaGQ86hOl_VV55eP3B9DWfZTQv)

### 文字测量

当文档需要精确的布局时，需要知道一段文字的尺寸，可以使用`widthOfString(text, options)`和`heightOfString(text, options)`方法

这两个方法不会绘制文字，只会返回测量后的尺寸。

### 列表

使用`list`方法可以创建无需列表，第一个参数是一个由各项文字组成的数组，后续参数可以指定横纵坐标。

可以通过嵌套数组创建嵌套列表

### 富文本

在`text`的选项参数里面参数`continued`为`true`，可以生成连续的文字

```JS
doc.fillColor('red').text('123', { continued: true })
  .fillColor('green').text('456');
```

![](http://image.oldzhou.cn/FlohKekaDPiBLfLmIkpf5QZR89lW)

### 字体

PDFKit默认支持14种字体，使用`font`方法可以直接使用字符串指定这14种字体：

- 'Courier'
- 'Courier-Bold'
- 'Courier-Oblique'
- 'Courier-BoldOblique'
- 'Helvetica'
- 'Helvetica-Bold'
- 'Helvetica-Oblique'
- 'Helvetica-BoldOblique'
- 'Symbol'
- 'Times-Roman'
- 'Times-Bold'
- 'Times-Italic'
- 'Times-BoldItalic'
- 'ZapfDingbats'

除了这14种字体，PEFKit也支持外嵌字体，支持的字体格式有`.ttf`/`.otf`/`.ttc`/`.dfont`

在默认情况下，PDFKit是**不支持中文**的，所以需要上传包含中文的字体并指定，下载了开源的思源宋体并上传到文件夹中：

```JS
doc.font('./fonts/Source-Han.otf');

doc.text('你好');
doc.text('你也好');
```

PDFKit也支持注册字体，这样就不必每次使用字体时都输入一大串的路径名了：

```JS
// 注册字体
doc.registerFont('source font','./fonts/Source-Han.otf');

// 使用字体
doc.font('source font');

doc.text('你好');
doc.text('你也好');
```

### 图像

通过`image`方法来创建图像，图像的形式可以是路径、buffer对象、BASE64编码后的data uri。**PDFKit支持`JPEG`和`PNG`格式**

如果没有提供`X`/`Y`参数，图像会在当前文字流的位置（在最后一行文字后）渲染。如果提供了坐标，图像会在指定的位置渲染。

`image`方法可配置的参数有：

- `width`/`height`，指定图像的宽高，当都未指定时会按照图片的实际尺寸渲染，如果指定了某一方向的尺寸，图像会按照指定方向的尺寸按原比例缩放，如果都指定了，图像会拉伸至指定尺寸
- `sacle`，对图像进行缩放
- `fit`，接受一个数组，图像以这个数组提供的宽度和高度中较小的尺寸进行缩放（类似CSS的`background-size`的`contain`属性），会留白
- `cover`，接受一个数组，图像以这个数组提供的宽度和高度中较大的尺寸进行缩放（类似CSS的`background-size`的`cover`属性），会剪切

![](http://image.oldzhou.cn/FlTa2RQUmatR2mxjGxYk4mQG5V1-)

### 注释

PDF中的注释是具有可交互特性的文字，比如可点击的链接、注释或者高亮、下划线、删除线等文字样式。支持的注释有：

![](http://image.oldzhou.cn/FmhmIgKC8zb_csCziYnBeIHM7L55)

要注意的是，这些文字样式都不是直接加载文字上，而是以矩形的方式覆盖到文字上。所以使用的时候需要使用`widthOfString`计算出要添加注释的文字的宽度，使用`currentLineHeight`计算出当前的行高。

还要注意注释的堆栈顺序，比如添加`link`，需要保证`link`是最后一个被添加的，否则会被其他的注释覆盖，导致无法点击。


```JS
// Add the link text
doc.fontSize(25)
   .fillColor('blue')
   .text('This is a link!', 20, 0);

// Measure the text
const width = doc.widthOfString('This is a link!');
const height = doc.currentLineHeight();

// Add the underline and link annotations
doc.underline(20, 0, width, height, {color: 'blue'})
   .link(20, 0, width, height, 'http://google.com/');
```

对于比较常用的链接、下划线，可以直接在`text`方法的选项中添加，更为方便：

```JS
doc.fontSize(20)
  .fillColor('red')
  .text('Another link!', 20, 0, {
      link: 'http://apple.com/',
      underline: true
    }
  );
```

## 参考

- [pdfKit](http://pdfkit.org/)
