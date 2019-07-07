---
title: JS31 JS中的事件队列（Event Loop）
top: false
date: 2017-12-06 19:19:04
updated: 2019-07-07 20:16:50
tags:
- 队列
categories: JavaScript
---
JS中的事件队列（Event Loop）学习笔记及练习。

<!-- more -->

## 同步和异步

首先要明确：

**JS是单线程语言**

也就是说，JS一次只能做一件事情。

CPU处理指令速度非常快，远比磁盘I/O和网络I/O速度快，所以一些CPU直接执行的任务就成了优先执行主线任务（即同步任务），然后需要I/O返回数据的任务就成了等待被执行的任务（即异步任务）

- 同步任务（Asynchrono）：在主线程上排队执行的任务，前一个任务执行完毕，才能执行后一个任务；
- 异步任务（Synchrono）：不进入主线程、而进入“任务队列”（task queue）的任务，只有“任务队列”通知主线程，某个异步任务可以执行了，该任务才会进入主线程执行。

所以：

**当要主线程任务完成会后，就会去读取异步任务的“任务队列”，这就是JavaScript的运行机制**

## Microtasks和Macrotasks

具体到任务队列，又分为宏任务（Microtasks）和微任务（Macrotasks）

属于微任务的任务有：

- `Process.nextTick`
- `Promise`
- `Object.observe`（已被废弃）
- `MutationObserver`

属宏任务的任务有：

- `setTimeout`
- `setInterval`
- `setImmediate`
- `MessageChannel`
- `I/O`
- `UI渲染`

具体的执行顺序：

（1） 代码开始第一次循环，执行所有主线程的同步任务，遇到异步函数，分别添加到微任务队列和宏任务队列。

（2）所有同步任务执行完成后，开始执行异步任务。

（3）首先执行微任务队列中的全部任务，在执行过程中，如果遇到新的微任务，那么会**加入到当前的微任务队列中，继续执行，直到所有的微任务执行完毕**

（4）微任务执行完成后，开始执行宏任务中的任务，在执行过程中，如果遇到微任务，会将微任务将入到为任务队列，优先执行微任务队列中的任务，微任务执行完成后返回继续执行宏任务

（5）直到所有宏任务执行完毕。

也就是说，**JavaScript在执行完主线程的同步任务后，开始执行异步任务。首先执行异步任务中的微任务队列，然后执行宏任务。在执行过程中，每次执行宏任务之前都会检查微任务队列，如果微任务队列未清空，则总会优先执行微任务**。

## 异步中的异步

面试题一般都会在异步中再次遇到异步的问题上搞事情，我比较容易犯糊涂的有下面两点。

（1）在微任务中又遇到了微任务，举例子来说明吧：

```JS
console.log(1);

setTimeout(() => {
  console.log(2);
});

Promise.resolve().then(() => {
  console.log(3);
  process.nextTick(() => {
    console.log(4);
  })
});
```

按照上面的分析，首先打印出`1`，然后将`console.log(2)`放到宏任务的队列，在然后将`console.log(3)`和`process.nextTick`放入了微任务队列：

![](http://image.oldzhou.cn/FvFPJrqlOCzMaqZhgjKXanYKZQR4)

执行完成同步任务后，首先执行微任务队列，打印出`3`之后，遇到了另外一个微任务`process.nextTick`，所以正确的顺序是，将`process.nextTick`中的代码`conosle.log(4)`**再次加入微任务队列**：

![](http://image.oldzhou.cn/Flu3jvAxoUwLMprikd87JZ3hHKML)

然后继续执行微任务队列，打印`4`，此时微任务队列已经清空，这个时候才会去执行宏任务，打印`2·

所以，正确的打印顺序是`1` → `3` → `4` → `2`

（2）在宏任务又遇到了微任务

```JS
console.log(1);

setTimeout(() => {
  console.log(2);
  Promise.resolve().then(() => {
    console.log(3);
  });
});

setTimeout(() => {
  console.log(4);
});
```

同样首先先打印了`1`，然后将`console.log(2)`以及`Promise.resolve()`后面这一堆都加入了宏任务队列，然后将`console.log(4)`也加入宏任务队列

![](http://image.oldzhou.cn/FqAC_k1q6layQhk7jboZq6W3zFFa)

由于此时没有微任务，开始执行宏任务队列，首先打印了`2`，然后执行`Promise.resovle`的代码，由于这是一个微任务，所以会将`console.log(3)`加入了微任务队列

![](http://image.oldzhou.cn/Fo1mY5lRR5mOx7zoii5DaYU0aOVn)

此时还未执行的任务中，由于`console.log(3)`是微任务，所以会优先执行，所以会先打印`3`，最后打印`4`

所以，正确的打印顺序是`1` → `2` → `3` → `4`

> 写Dome的时候，发现浏览器环境（Chrome 75）与Node（10.16）执行处的结果并不完全相同，Node环境本身执行的结果也不相同，大部分时间结果是`1243`，不知道为何，因为对Node的时间循环并不了解，留下疑问（2019-07-07）

## 练习

在面试中经常会遇到考察输出顺序的题目。

第一题

```JS
setTimeout(function() {
  console.log(4)
}, 0);
new Promise(function(resolve) {
  console.log(1)
  for (var i = 0; i < 10000; i++) {
    i == 9999 && resolve()
  }
  console.log(2)
}).then(function() {
  console.log(5)
});
requestAnimationFrame(function () {
  console.log(6)
})
console.log(3);
```

之所以`6`在`4`前面，因为`setTimeOut`即便第二个参数是0，但是HTML5标准规定了其最小值不能低于`4`毫秒，并且浏览器设置的最短间隔都在`10`毫秒左右，而`requestAnimationFrame`采用系统时间间隔，浏览器自动确定刷新频率，优先执行

> 但是今天在重温这道题目时，发现浏览（Chrome 75）的执行结果有时是`4`在`6`的前面，有时是`6`在`4`的前面，实际上这个顺序也是不确定的，不知道为何，存疑（2019-07-07）

第二题

```JS
console.log('start')
const interval = setInterval(() => {
  console.log('setInterval')
}, 0)
setTimeout(() => {
  console.log('setTimeout 1')
  Promise.resolve().then(() => {
    console.log('promise 3')
  }).then(() => {
    console.log('promise 4')
  }).then(() => {
    setTimeout(() => {
      console.log('setTimeout 2')
      Promise.resolve().then(() => {
        console.log('promise 5')
      }).then(() => {
        console.log('promise 6')
      }).then(() => {
        clearInterval(interval)
      })
    }, 0)
  })
}, 0)
Promise.resolve().then(() => {
  console.log('promise 1')
}).then(() => {
  console.log('promise 2')
})
```

第三题

```JS
console.log('start');
setTimeout(() => { console.log('s1') }, 0);
new Promise((resolve) => {
  console.log('p1');
  resolve()
}).then(v => {
  console.log('t1');
  setTimeout(() => { console.log('s2') }, 0);
  new Promise((resolve) => {
    console.log('p2');
    resolve()
  }).then(v => {
    console.log('t2')
  });
  console.log('t3');
  setTimeout(() => { console.log('s3') }, 0);
});
console.log('end');
```

第四题

面试快手时遇到的，对于在执行微任务时又遇到微任务，突然又糊涂了，傻逼一个。

```JS
console.log(1);

setTimeout(() => {
  console.log(2);
});

process.nextTick(() => {
  console.log(3);
});

setImmediate(() => {
  console.log(4);
});

new Promise(resolve => {
  console.log(5);
  resolve();
  console.log(6);
}).then(() => {
  console.log(7);
});

Promise.resolve().then(() => {
  console.log(8);
  process.nextTick(() => {
    console.log(9);
  })
})
```

## 参考

- [event loop js事件循环 microtask macrotask@CSDN](http://blog.csdn.net/sjn0503/article/details/76087631)
- [Promise的队列与setTimeout的队列有何关联？@知乎](https://www.zhihu.com/question/36972010)
