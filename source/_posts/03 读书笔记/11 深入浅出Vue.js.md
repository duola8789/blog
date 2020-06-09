---
title: 深入浅出Vue.js
top: false
date: 2020-04-03 09:37:45
updated: 2020-06-09 09:42:28
tags:
- Vue
categories: 读书笔记
---

[《深入浅出Vue.js》](https://book.douban.com/subject/32581281/)读书笔记。非常好的一本书，对Vue源码和原理介绍的确实是深入浅出，值得阅读。

<!-- more -->

# Object的变化侦测

Vue中一个状态的依赖不再是具体的DOM节点，而是一个组件。当状态变化后，会通知到组件，组件内部再使用虚拟DOM进行比对。这样可以大大降低依赖数量，从而降低依赖追踪所消耗的内存。

逐步回答下列问题：

## 如何追踪变化？

使用`Object.defineProperty`劫持对象的`get`和`set`属性，封装成为`defineReactive(data, key, value)`这样一个函数

## 如何收集依赖？

在`getter`中收集依赖，在`setter`中触发更新

## 依赖收集在哪里

首先将依赖保存在一个全局变量上（比如`window.target`），然后用一个数组来保存收集的依赖。

上述代码封装为`Dep`类，专门用来管理依赖，它包括`addSub`、`removeSub`、`depend`、`notify`方法。

`Dep`会在`defineReactive`中实例化，在`get`中`depend`，在`set`中`notify`

```JS
class Dep {
  constructor() {
    this.subs = [];
  }

  addSub(sub) {
    this.subs.push(sub);
  }

  removeSub(sub) {
    if (this.subs.length > 0) {
      const index = this.subs.indexOf(sub);
      if (index > -1) {
      }
    }
  }

  depend() {
    if (window.target) {
      this.addSub(window.target);
    }
  }

  notify() {
    const subs = this.subs.slice();
    for (let i = 0; i < subs.length; i++) {
      subs[i].update();
    }
  }
}
```

## 依赖是谁

`Watcher`类，它是抽象出来，用来集中处理各种情况的类。依赖收集阶段只收集这个封装好的类的实例，通知只通知它一个，再由他通知其他地方。

## `Watcher`是什么

`Watcher`的实现非常巧妙，需要结合代码体会：

```JS
class Watcher {
  constructor(vm, expOrFn, cb) {
    this.vm = vm;
    this.cb = cb;
    this.getter = parsePath(expOrFn);
    this.value = this.get();
  }

  get() {
    window.target = this;
    this.value = this.getter.call(this.vm, this.vm);
    window.target = undefined;
    return this.value;
  }

  update() {
    const oldValue = this.value;
    this.value = this.get();
    this.cb.call(this.vm, this.value, oldValue);
  }
}
```

（1）在构造方法中，把`parsePath`的返回值赋值给`getter`，`parsePath`的返回值是一个函数，给这个函数传入一个对象，就可以访问这个对象在对应的`expOrFn`路径下的属性值

（2）在构造方法中，调用`get`为`value`赋值，`get`方法中，将`Watcher`实例赋值给`window.target`，然后通过调用上一步的`getter`方法，获取对象在`expOrFn`路径下的属性值，并将属性值赋给`value`。

（3）在读取这个属性值（`expOrFn`）的时候，会触发属性值的`getter`，在`getter`中我们会将`window.target`收集到依赖`dep`中，此时`window.target`已经是`watcher`实例了，这样就将`watcher`实例收集到依赖中

（4）`update`方法是在`Dep`中的`notify`方法中被调用的，用来触发`Watcher`中真正要执行的方法

## 如何侦测所有属性

上面只能侦测单个属性 ，所以封装了`Observer`类来递归侦测数据所有的属性（包括子属性），将所有的属性都转换为`getter/setter`的形式，追踪其变化

```JS
class Observer {
  constructor(value) {
    this.value = value;

    if (!Array.isArray(value)) {
      // 遍历所有属性
      this.walk(value);
    }
  }

  walk(value) {
    const keys = Object.keys(value);
    for (let i = 0; i < keys.length; i++) {
      defineReactive(value, keys[i], value[keys[i]]);
    }
  }
}


const defineReactive = (data, key, value) => {
  // 递归子属性
  if (typeof value === 'object') {
    new Observer(value);
  }
  let dep = new Dep();
  Object.defineProperty(data, key, {
    enumerable: true,
    configurable: true,
    get() {
      dep.depend();
      return value;
    },
    set(newVal) {
      if (newVal === value) {
        return;
      }
      value = newVal;
      dep.notify();
    }
  });
};
```

## 总结

（1）通过封装`Object.defineProperty`方法得到的`defineReactive`函数来劫持属性的`getter`和`setter`，在`getter`中收集依赖，在`setter`中触发依赖更新

（2）`Dep`类用来收集和管理依赖，在`getter`中`depned`，在`setter`中`notify`

（3）`Watcher`类就是收集的依赖，实际上是一个订阅器，`Watcher`会将自己的实例赋值给`window.target`（全局变量）上，然后去主动访问属性，触发属性的`getter`，将`window.target`收集到`Dep`中，`Watcher`的`update`方法会在`Dep`的`notify`方法中被调用，触发更新

（4）`Obersver`类用来将一个对象的所有属性和子属性都变成响应式的，通过递归调用`defineReactive`来实现


# Array的变化侦测

## 如何追踪变化

使用拦截器，拦截数组可以改变自身的方法，覆盖`Array.prototype`

## 拦截器是什么

拦截器要具能够实现原生数组方法的功能，但是提供机会让我们可以插入一些操作：

```JS
const arrayProto = Array.prototype;
const arrayMethods = Object.create(arrayProto)

['push', 'pop', 'shift', 'unshift', 'reverse', 'sort', 'splice'].forEach((method) => {
  // 缓存原始方法
  const original = arrayProto[method];
  Object.defineProperty(arrayMethods, method, {
    value: function mutator(...args) {
      return original.apply(this, args)
    },
    enumerable: false,
    writable: true,
    configurable: true
  })
});
```

## 使用拦截器覆盖Array原型

有两种方法来覆盖，一种是使用`__proto__`属性，另外一种是当`__proto__`不被支持的情况下，直接接改写的方法直接设置到被侦测的数组上：

```JS
const hasProto = `__proto__` in {};
const arrayKeys = Object.getOwnPropertyNames(arrayMethods);

function def(obj, key, val, enumerable) {
  Object.defineProperty(obj, key, {
    value: val,
    enumerable: !!enumerable,
    writable: true,
    configurable: true
  });
}

export class Observer {
  constructor(value) {
    this.value = value;

    if (Array.isArray(value)) {
      const augment = hasProto ? protoAugment : copyAugment;
      augment(value, arrayMethods, arrayKeys);
    }
  }
}

function protoAugment(target, src, keys) {
  target.__proto__ = src;
}

function copyAugment(target, src, keys) {
  for (let i = 0, l = keys.length; i < l; i++) {
    const key = keys[i];
    def(target, key, src[key])
  }
}
```

## 如何收集数组的依赖

这里谈论的数组，本身也是对象的一个属性值，例如：

```JS
{
  data: {
    arr: []
  }
}
```

数组本身的依赖，也是在`arr`这个`key`的响应式监听中收集依赖的，所以数组本身的依赖收集也是在`getter`中收集的，但是触发依赖的过程不光是在`setter`中，还有在上面改写的数组方法的拦截器中触发

## 在哪里那保存数组的依赖

数组触发依赖不是在`definedProperty`和`setter`中完成的，而是在拦截器中完成的，数组拦截器中无法访问在`defineReactive`中定义的`let dep = new Dep()`这个变量，所以Vue将数组的依赖保存了在了`Observer`的实例属性上：

```JS
class Observer {
  constructor(value) {
    this.value = value;
    this.dep = new Dep();

    // ......
  }
}
```

这样，在`getter`中和拦截器中都可以访问以依赖了，至于如何依赖，后面介绍。

## 收集依赖

在`getter`中收集数组的依赖时，需要访问到`Obersver`的实例上保存的`dep`属性，Vue中是通过定义了一个`observe`方法实现的

```JS
function observe(val) {
  if (!isObject(val)) {
    return;
  }
  let ob;
  if (Object.prototype.hasOwnProperty.call(val, '__proto__') && val instanceof Observer) {
    ob = val.__proto__;
  } else {
    ob = new Observer(val);
  }
  return ob;
}
```

首先根据`__proto__`属性判断数组是不是响应式数据，如果已经是了就不需要再重复创建`Obersver`的实例，如果不是的话则会创建一个`Obersver`实例

`observe`方法会在`definedReactive`中被调用：

```JS
const defineReactive = (data, key, value) => {
  let childOb = observe(value);
  let dep = new Dep();
  Object.defineProperty(data, key, {
    enumerable: true,
    configurable: true,
    get() {
      dep.depend();
      // 将数组的依赖保存在`Observer`实例上
      if (childOb) {
        childOb.dep.depend();
      }
      return value;
    },
    set(newVal) {
      if (newVal === value) {
        return;
      }
      value = newVal;
      dep.notify();
    }
  });
};
```

这样数组的依赖就被保存在了`Observer`实例上（我理解，这个`childOb`就是为了将数组的依赖保存在`Observer`实例上存在的，如果没有它数组的依赖无法挂载到`Observer`实例上）

## 在拦截器中获取Observer实例

在拦截器中的`this`指向的当前数组的实例，我们希望在`this`上读取到`Observer`的实例，Vue的做法是在`Observer`定义时为数组实例添加了一个`__ob__`属性，将Observer实例保存到了这个属性上

```JS
class Observer {
  constructor(value) {
    this.value = value;
    this.dep = new Dep();

    // 将 observer 实例赋到 __ob__属性
    def(value, '__ob__', this);

    // ...
  }
}
```

`__ob__`的作用不仅仅是在拦截器中访问`Observer`的实例，还可以用来标记当前`value`是否已经被`Observer`转换为了响应式的数据

在`value`上标记了`__ob__`之后，就可以通过`value.__ob__`来访问`Observer`的实例，然后通过`Observer`实例上保存的`dep`属性来访问收集的数组的依赖：

```JS
// 改写数组的原生方法
['push', 'pop', 'shift', 'unshift', 'reverse', 'sort', 'splice'].forEach((method) => {
  const original = arrayProto[method];
  def(arrayMethods, method, function mutator(...args) {
    const result = original.apply(this, args);
    // 访问 Observer 实例
    const ob = this.__ob__;

    // 访问依赖并发送通知
    ob.dep.notify();
    return result;
  });
});
```

## 侦测数组成员的变化

在`Observer`中针对数组类型定义了`observerArray`方法，这个方法中遍历数组的每个成员，调用前面定义的`observe`方法，将每个成员都转换为响应式数据：

```JS
class Observer {
  constructor(value) {
    this.value = value;
    this.dep = new Dep();

    def(value, '__ob__', this);

    if (Array.isArray(value)) {
      // 拦截数组原生方法
      const augment = hasProto ? protoAugment : copyAugment;
      augment(value, arrayMethods, arrayKeys);
      this.observeArray(value);
    } else {
      this.walk(value);
    }
  }

  walk(value) {
    const keys = Object.keys(value);
    for (let i = 0; i < keys.length; i++) {
      defineReactive(value, keys[i], value[keys[i]]);
    }
  }

  // 侦测数组中的每一项
  observeArray(items) {
    for (let i = 0, l = items.length; i < l; i++) {
      observe(items[i]);
    }
  }
}
```

## 侦测数组新增成员的变化

数组的`push`/`unshift`等方法时可以新增数组内容的，新增数组的内容也需要转换为响应式数据，这个过程是在数组拦截器中完成的

首先要获取到新增的元素，然后对新增的元素通过上面定义的`Observer`实例上的`observeArray`方法将新增成员转换为响应式数据：

```JS
// 改写数组的原生方法
['push', 'pop', 'shift', 'unshift', 'reverse', 'sort', 'splice'].forEach((method) => {
  const original = arrayProto[method];
  def(arrayMethods, method, function mutator(...args) {
    const result = original.apply(this, args);
    const ob = this.__ob__;

    let inserted;
    switch (method) {
      case 'push':
      case 'unshift': {
        inserted = args;
        break;
      }
      case 'splice': {
        inserted = args.slice(2);
        break;
      }
    }
    if (inserted) {
      ob.observeArray(inserted);
    }

    ob.dep.notify();
    return result;
  });
});
```

## 数组的限制

对Array的变化时通过拦截原型的方式实现的，这样直接使用下标修改数组，Vue是无法侦测到的，同样直接修改数组的`length`属性也是不行的

实际上如果使用`Object.defineProperty`对数组的`key`做监听也是可以实现上面两种方式的响应式变化的，没有这样做，根据尤雨溪的解释是：

> 性能代价和获得的用户体验收益不成正比

具体的讨论可以看[这篇文章](https://segmentfault.com/a/1190000015783546?_ea=4074035)。


## 总结

Array追踪变化的方式与Object不一样，它是通过方法来改变内容的，所以通过创建拦截器去覆盖数组原型的方式来追踪变化

Array中收集依赖的方式和Object一样，都是在`getter`中手机，但由于使用依赖的位置不同，数组要在拦截器中向依赖发消息，所以依赖不能像Objet一样保存在`defineRactive`中，而是保存在了`Obersever`实例上

Observer中，把每个侦测了变化的数据都标记`__ob__`，并把`this`（Observer实例）保存在`__ob__`上，这样①可以标记数据是否被侦测了变化，保证每个数据只被侦测一次②通过数数据获取到`__ob__`，从而拿到Observer上保存的依赖，当拦截到数组发生变化时，向依赖发送通过

除了侦测数组自身的变化外，数组中元素的变化也要侦测。在Observer中判断如果当前被侦测的数据是数组，则调用`observerArray`方法将数组每个元素都转换为响应式

除了侦测已有数据外，当用户使用`push`等方法向数组中新增数据时，新增的数据也要进行变化侦测。如果操作数组的是`push`、`unshift`、`splice`方法，则从参数中将数据提取出来，使用`observer`对新增数据进行变化侦测。

# 变化侦测的API实现原理

## `vm.$watch`

```JS
const unwatch = vm.$watch(expOrFn, callback, [options]);

// 取消观察
unwatch();
```

### `expOrFn`支持函数的改造

`expOrFn`表达式只接受以点分割的路径，如果是复杂的表示，可以用函数代替。

`vm.$watch`是对Watcher的一种封装，目前的Watcher结构不支持`expOrFn`为函数，进行一点改造：

```JS
class Watcher {
  constructor(vm, expOrFn, cb) {
    this.vm = vm;
    this.cb = cb;
    // expOrFn 支持函数
    if (typeof expOrFn === 'function') {
      this.getter = expOrFn;
    } else {
      this.getter = parsePath(expOrFn);
    }
    this.value = this.get();
  }

  get() {
    window.target = this;
    this.value = this.getter.call(this.vm, this.vm);
    window.target = undefined;
    return this.value;
  }

  update() {
    const oldValue = this.value;
    this.value = this.get();
    this.cb.call(this.vm, this.value, oldValue);
  }
}
```

当`expOrFn`为函数时，函数中所读取的所有Vue实例上的响应式数据都会被Watcher所观察（因为或在函数中获取数据的值，在所有数据的`get`中会将这个Watcher收集到`dep`中）

> 实际上Vue的计算属性实现原理与`expOrFn`支持函数关系很大

### `watch`的实现

这样Watcher可以实现`vm.$watch`的功能，但是参数`deep`和`immediate`需要对Watcher进行改造才可以实现：

```JS
Vue.prototype.$watch = function (expOrFn, cb, options) {
  const vm = this;
  options = options || {};

  const watcher = new Watcher(vm, expOrFn, cb, options);

  if (options.immediate) {
    cb.call(vm, watcher.value);
  }

  return function unwatch() {
    watcher.teardown();
  };
};
```

### Watcher的`teardown`方法

`teardown`方法用来取消观察数据，本质是把`watcher`从观察的所有状态的依赖列表中移除，要实现这个方法，需要：

1. 在Watcher中记录自己都订阅了谁，也就是`watcher`实例被收集进入了哪些Dep里
2. 不想继续订阅这些Dep时，循环自己的订阅列表通过Dep来将自己从Dep的依赖列表中移除

这就需要对收集依赖部分的代码进行改造，首先在Watcher中添加`addDep`方法，用来在`watcher`中记录自己都订阅过哪些Dep：

```JS
class Watcher {
  constructor(vm, expOrFn, cb) {
    this.vm = vm;
    this.cb = cb;

    // 记录自己订阅的依赖列表
    this.deps = [];
    // 依赖列表的 Id
    this.depIds = new Set();

    // expOrFn 支持函数
    if (typeof expOrFn === 'function') {
      this.getter = expOrFn;
    } else {
      this.getter = parsePath(expOrFn);
    }
    this.value = this.get();
  }

  addDep(dep) {
    const id = dep.id;
    if(!this.depIds.has(id)) {
      this.depIds.add(id);
      this.deps.push(dep);
      // 将自己（watcher实例）添加的 dep 的 subs 列表中
      dep.addSub(this);
    }
  }
}
```

Dep中对应的收集依赖的逻辑也要有所改变：

```JS
let uid = 0;

class Dep {
  constructor() {
    this.id = uid++;
    this.subs = [];
  }

  depend() {
    if (window.target) {
      // this.addSub(window.target);
      window.target.addDep(this)
    }
  }
}
```

`window.target`就是`watcher`实例，执行`addDep`方法时，Watcher会记录自己会被哪些Dep通知，Dep也会记录数据变化时需要通知哪些Watcher，这是一个多对多的关系（正因为`expOrFn`可能是函数，里面可能会订阅多个响应式数据）

这样在Watcher中就可以添加`teardown`方法来解除观察，通知自己订阅的Dep，让它们把自己从依赖列表中移除掉：

```JS
class Watcher {
  //  从所有依赖项的 Dep 列表中把自己移除
  teardown() {
    let i = this.deps.length;
    while(i--) {
      this.deps[i].removeSub(this)
    }
  }
}
```

在Dep中的`removeSub`方法会把watcher实例从自己的subs中移除掉：

```JS
class Dep {
  removeSub(sub) {
    if (this.subs.length > 0) {
      const index = this.subs.indexOf(sub);
      if (index > -1) {
        return this.subs.splice(index, 1)
      }
    }
  }
}
```

### `deep`参数的实现原理

`deep`参数的功能实现，就是除了要触发当前这个被监听数据的收集依赖的逻辑之外，还要把当前监听的这个值在内的所有子值都要触发一遍收集依赖逻辑。

```JS
class Watcher {
  constructor(vm, expOrFn, cb, options) {
    this.vm = vm;
    this.cb = cb;

    if (options) {
      this.deep = !!options.deep;
    } else {
      this.deep = false;
    }

    // ...
  }

  get() {
    window.target = this;
    this.value = this.getter.call(this.vm, this.vm);

    // deep 逻辑
    if (this.deep) {
      traverse(this.value);
    }

    window.target = undefined;
    return this.value;
  }

  // ...
}
```

`traverse`方法就是用来递归`value`的所有子值来触发它们依赖的功能：

```JS
// 递归收集依赖
function traverse(val) {
  _traverse(val, seenObjects);
  seenObjects.clear();
}

function _traverse(val, seen) {
  let i, keys;
  const isA = Array.isArray(val);

  if ((!isA && !isObject(val)) || Object.isFrozen(val)) {
    return;
  }

  if (val.__ob__) {
    const depId = val.__ob__.dep.id;
    if (seen.has(depId)) {
      return;
    }
    seen.add(depId);
  }

  if (isA) {
    i = val.length;
    while (i--) {
      _traverse(val[i], seen);
    }
  } else {
    keys = Object.keys(val);
    i = keys.length;
    while (i--) {
      _traverse(val[[keys[i]]], seen);
    }
  }
}
```

上面的关键是，当数据的类型为`Object`时，循环`Object`中所有的`key`，然后执行了读取操作，然后再递归子值：

```JS
while (i--) {
 _traverse(val[[keys[i]]], seen);
}
```

其中`val[[keys[i]]`会触发`getter`，也就是说会触发收集依赖的操作，这时候`window.target`还未被清空，会将当前的`watcher`收集进去。

## `vm.$set`

```JS
vm.$set(target, key, value);
// target 需要时响应式数据，但不能是Vue实例或者Vue实例的根数据对象
```

这个API主要是用来避开Vue无法侦测属性被添加的限制。

### `target`是数组

```JS
function set(target, key, val) {
  // 处理数组的情况
  if (Array.isArray(target) && isValidArrayIndex(key)) {
    target.length = Math.max(target.length, key);
    target.splice(key, 1, val);
    return val;
  }
}
```

要注意的是，判断`key`与数组的长度，如果大于数组的长度，要给数组“扩容”，然后通过改造后的`splice`方法来收集依赖

### `key`已经存在于`target`

```JS
function set(target, key, val) {
  // 处理数组的情况
  if (Array.isArray(target) && isValidArrayIndex(key)) {
    target.length = Math.max(target.length, key);
    target.splice(key, 1, val);
    return val;
  }

  // key 已存在
  if (key in target && !(key in Object.prototype)) {
    target[key] = val;
    return val;
  }
}
```

`key`已经存在`target`里，那么这个`key`实际上已经是响应式的了，这时候直接用`key`和`val`直接修改数据就好了

### 新增属性

```JS
function set(target, key, val) {
  // 处理数组的情况
  if (Array.isArray(target) && isValidArrayIndex(key)) {
    target.length = Math.max(target.length, key);
    target.splice(key, 1, val);
    return val;
  }

  // key 已存在
  if (key in target && !(key in Object.prototype)) {
    target[key] = val;
    return val;
  }

  // 新增属性
  const ob = target.__ob__;
  // target 是 Vue 实例，或者是 Vue 实例的根数据对象
  if (target._isVue || (ob && ob.vmCount)) {
    process.env.NODE_ENV !== 'production' ** console.warn('...');
    return val;
  }

  if (!ob) {
    target[key] = val;
    return val;
  }

  defineReactive(ob.value, key, val);
  ob.dep.notify();
  return val;
}
```

先获取`target`的`observer`实例，即`__ob__`属性。然后根据`_isVue`判断`target`是否是Vue实例，用`ob.vmCount`来判断是否是根数据对象（及根节点的数据对象`this.$data`），这两种情况都不需要处理

如果`target.__ob__`不存在，那么说明`target`不是响应式的，也不需要任何处理，直接赋值即可。

如果上面所有条件不满足，才证明用户是为一个响应式数据新增了一个属性，这是用`defineReactive`来将`ob.value`（即响应式数据对象）上添加`key`和`value`，并且将属性转换为getter/setter的形式，最后想`target`的依赖发送通知，并返回`val`

## `vm.$delete`

```JS
vm.$delete(target, key)
```

过程还是比较简单的：

```JS
function del(target, key) {
  // 先处理数组的情况
  if (Array.isArray(target) && isValidArrayIndex(key)) {
    // 使用改造后的数组的 splice 方法就可以做到响应式删除，自动向依赖发送更新
    target.splice(key, 1);
    return;
  }

  const ob = target.__ob__;

  // 如果是 Vue 实例或者是根数据对象，则不能使用这个方法
  if (target._isVue || (ob && ob.vmCount)) {
    process.env.NODE_ENV !== 'production' ** console.warn('...');
    return;
  }

  // 如果不是自身的属性，那么什么也不操作
  if (!Object.prototype.hasOwnProperty.call(target, key)) {
    return;
  }

  delete target[key];

  // 如果不是响应式数据，也就不需要向依赖发送通知
  if (!ob) {
    return;
  }
  ob.dep.notify();
}
```

# 虚拟DOM

## 为什么要引入虚拟DOM

Vue的依赖订阅机制，可以再一定程度上知道哪些状态发生了变化。在Vue1.0中并没有引入虚拟DOM，当状态发生变化，Vue知道哪些节点使用了这个状态，从而对这些节点进行更新操作，不需要对比

这样做的代价是，粒度太细，导致内存开销过大。

所以在Vue2.0中选择了中等粒度的解决方案，引入了虚拟DOM。组件级别是一个Watcher实例，当状态发生变化时，只能通知到组件，然后组件内部通过虚拟DOM来进行比对和渲染。

## Vue中的虚拟DOM

Vue中使用模板来描述状态与DOM间的映射关系。Vue通过编译将模板转换为渲染函数，执行渲染函数就可以得到虚拟节点树，使用这个虚拟节点树来渲染页面。

为了避免整体替换vnode带来的性能浪费，虚拟DOM在做虚拟节点映射到视图的过程中，会与旧的虚拟节点进行比对（diff），找出需要更新的节点来进行DOM操作

虚拟DOM完成的事情也就是：

1. 提供与真实DOM节点所对应的虚拟节点vnode
2. 将虚拟节点vnode和旧的虚拟节点oldVnode进行比对，然后更新视图

# VNode

vnode可以理解为节点描述对象，它描述了怎么样去创建真实的DOM节点，它只是一个普通的对象，是从VNode类实例化的对象。

vnode有不同类型：

- 注释节点
- 文本节点
- 元素节点
- 组件节点
- 函数式组件
- 克隆节点

## 注释节点

节点：

```HTML
<!-- 注释节点 -->
```

对应的vnode：

```JS
{
  text: '注释节点',
  isComment: true
}
```

创建过程：

```JS
export const createEmptyVNode = (text) => {
  const node = new VNode();
  node.text = text;
  node.isComment = true;
  return node;
};
```

## 文本节点

vnode：

```JS
{
  text: 'Hello Vue',
}
```

创建过程：

```JS
export const createTextVNode = (value) => {
  return new VNode(undefined, undefined, undefined, String(value));
};
```

## 克隆节点

克隆节点是将享有的节点属性复制到新节点中，让新创建的节点和被克隆节点的属性保持一致，实现克隆效果。作用是优化静态节点和插槽节点

```JS
export const cloneVNode = (vnode, deep) => {
  const cloned = new VNode(
    vnode.tag,
    vnode.data,
    vnode.children,
    vnode.text,
    vnode.elm,
    vnode.context,
    vnode.componentOptions,
    vnode.asyncFactory
  );

  cloned.ns = vnode.ns;
  cloned.isStatic = vnode.isStatic;
  cloned.key = vnode.key;
  cloned.isComment = vnode.isComment;
  cloned.isCloned = true;

  if (deep && vnode.children) {
    cloned.children = cloneVNode(vnode.children);
  }

  return cloned;
};
```

克隆节点就是将现有的节点的属性全部复制到新节点中，除了`isCloned`属性为`true`

## 元素节点

元素节点有四个属性：

- `tag`，节点名称，例如`ul`、`p`
- `data`，包含节点上的数据，`attrs`、`class`、`style`等
- `children`，子节点列表
- `context`，当前组件的Vue实例

元素节点：

```HTML
<p><span>Hello</span><span>Vue</span></p>
```

vnode:

```JS
{
  children: [VNode, Vnode],
  context: {...}
  data: {...},
  tag: 'p',
  ...
}
```

## 组件节点

组件节点和元素节点类似，有两个独有的属性：

- `componentOptions`，组件节点的选项参数，包含`propsData`、`tag`等
- `compontentInstance`，Vue实例

## 函数式组件

与组件节点类似，有特有的属性`functionalContext`和`functionalOptions`

# patch

patch的目的是修改DOM节点，渲染视图。它不是暴力替换节点，而是在现有的DOM上修改来达到渲染视图的目的，包含三个过程：

1. 创建新节点
2. 删除废弃节点
3. 更新已存在节点

整个patch的过程是:

1. 当oldVnode不存在时，直接使用vnode渲染视图
2. 当oldVnode和vnode都存在，但不是同一个节点时，使用vnode创建的新DOM元素替换旧的DOM元素
3. 当oldVnode和vnode都存在并且是同一个节点时，使用更详细的对比操作来局部更新真实DOM

![](http://image.oldzhou.cn/FjhcC9MMhk9dQJR9lNHE4pcIG1NV)

## 创建节点

只有元素节点、注释节点和文本节点会被创建并插入到DOM中

有`tag`属性就可以判定是一个元素节点，然后调用`document.createElement`方法创建真实的元素节点

然后调用`parentNode.appendChild`来将元素插入到父节点，如果这个父节点已经被渲染到视图，那么插入到这个父节点下的元素也会被渲染到视图

然后需要将元素节点的子节点也创建出来并插入到刚刚创建出的节点下面，这是一个递归过程，需要将vnode的children属性循环一遍，将每个子虚拟节点都执行一遍创建元素的逻辑。

如果vnode的`tag`属性不存在，并且`isComment`为`true`，那么认为是注释节点，调用`document.createComment`方法，创建注释节点。如果`isComment`为`false`，那么就调用`document.createTextNode`创建真是的文本节点

## 删除节点

```JS
function isDef(v) {
  return v !== undefined && v !== null;
}

const nodeOps = {
  parentNode(node) {
    return node.parentNode;
  },
  removeNode(node, child) {
    node.removeChild(child);
  }
};

function removeNode(el) {
  const parent = nodeOps.parentNode(el);
  if (isDef(parent)) {
    nodeOps.removeNode(parent, el);
  }
}
```

`nodeOps`是对节点操作的封装，目的是为了进行实现跨平台运行时API对接。

## 更新节点

### 静态节点

静态节点指的就是渲染到界面上后，无论日后状态如何变化，都不会发生改变的节点

如果新旧两个虚拟节点都是静态节点，如果是的话就可以直接跳过更新节点的过程。

### 新虚拟节点有文本属性

如果新生成的虚拟节点有`text`属性，那么无论之前的旧节点的子节点是什么，直接调用`setTextContext`方法（在浏览器环境下是[`node.textContext`](https://wiki.developer.mozilla.org/zh-CN/docs/Web/API/Node/textContent)方法）

### 新虚拟节点无文本属性

这时候新的虚拟节点是一个元素节点，根据是否有`children`属性，存在两种情况：

（1）新的虚拟节点有`children`

如果旧的虚拟节点也有`children`属性，那么需要对新、旧虚拟节点的`children`进行详细的diff

如果旧的虚拟节点没有`children`属性，那么旧 的虚拟节点可能是文本节点或者是空标签，如果是文本节点，那么就要将文本清空将其变为空标签，然后将新的虚拟节点的`children`创建为真实DOM元素插入进去

（2）新的虚拟节点没有`children`

这时候新的虚拟节点是一个空节点，那么会对旧的虚拟节点进行删除，达到视图中是空标签的目的

![](http://image.oldzhou.cn/FhMU5iQv9B7TnNNDA6z2lwuQYnD0)

## 更新子节点

更新子节点可以分为四种操作：

- 更新节点
- 新增节点
- 删除节点
- 移动节点

更新子节点要进行循环，先遍历新的子节点列表，每循环到一个新的子节点，就会去旧子节点列表中找和当前节点相同的节点。如果找不到，就进行新增；如果找到了就做更新操作；如果位置不同，就进行移动

### 更新策略

（1）创建子节点

如果在遍历oldChildren时，没有找到本次循环（外层循环）所指向的新子节点相同的节点，那么意味着这个新子节点是一个新增子节点，需要执行创建节点操作，并且**插入到oldChildren中所有没有处理的节点的前面**

没有插入到所有已处理后节点的后面，是因为有可能会有连续几个新增节点，如果都插入到已处理后的节点的后面，那么插入的顺序是相反的

例如，如果有两个新增节点，插入到oldChildRen的没有处理的节点前面，顺序正确：

![](http://image.oldzhou.cn/Fr0ytebuoer4n2eoZCBquc76b2hd)

如果插入到已处理的节点的后面，那么顺序是相反的：

![](http://image.oldzhou.cn/FiZjL_XRKiL0KYqSvnvetFfGZRzV)

（2）更新子节点

如果两个节点是同一个节点，并且位置相同，那么这时候只需要进行更新操作即可

（3）移动子节点

当新旧节点是同一个节点，但是位置不同时，会进行移动操作，调用的是`Node.insertBefore`方法

关键是得到将旧的节点移动到的位置，这个位置是当前所有未处理节点的第一个节点

> 新增和移动节点的位置，都是以未处理节点进行标定位置的

![](http://image.oldzhou.cn/FpXbMrMSvqR9BbHt4hA86oySIxSa)

（4）删除子节点

本质上是删除那些oldChildren中存在但是newChildren中不存在的节点

当newChildren中的所有节点都被循环了一遍后，也就是循环结束后，如果oldChildren中还有剩余没被处理的节点，那么这些节点就要被删除

### 优化策略

可以使用下面四种快速查找节点的方式，这四种方式可以涵盖大多数的节点变化的情况，从而来避免循环oldChildren来查找节点，提升执行速度

- 将『新前』与『旧前』比对
- 将『新后』与『旧后』比对
- 将『新后』与『旧前』对比
- 将『新前』与『旧后』对比

![](http://image.oldzhou.cn/ForjMZACaWehXBCzrYVaW9_5eDvn)

前两种快捷查找方式，不需要执行移动节点的操作，只需要更新节点即可

后两种方式，除了更新节点外，还需要执行移动节点的操作。

在将『新后』与『旧前』对比时，如果二者是通过一个节点，在真实DOM中除了做更新操作外，还需要将节点移动到oldChildren中**所有未处理节点的最后面**

![](http://image.oldzhou.cn/FpndwBHawiNvZ_LhhzZToBKuCrh4)

这样移动才能保证下一轮移动时顺序是正确的，即下一轮的新后会在上一轮新后的前面

第四种比对方式与与第三种是类似的，需要将节点移动到oldChildren中**所有未处理节点的最前面**

![](http://image.oldzhou.cn/FpLaebaATQeFLD9440aoEaiGY3gX)

要记住的就是，移动都是以oldChildren中未处理的节点为基准进行，所有已更新过的节点不用管。

如果上面4中比对方式都没有找到相同的节点，那么再去通过循环的方式去oldChildren中详细找一圈

### 判断节点是否被处理过

实际上，并不是通过节点本身的属性来表示节点是否被处理过，而是通过循环来控制，循环体内只包含未处理过的节点，通过这一特性来实现，将处理过的节点剔除的

通常的遍历，无论是从前到后，或是从后到前，自然而然就会实现上面说的效果，处理后的节点不会进入下一轮循环

但是由于上面提到的优化策略，在循环体内，有可能处理的是未处理节点中的第一个，也可能是最后一个，所以就不能是从前向后循环，而应该是**从两边向中间循环**

Vue使用了四个变量，来标识循环的起始位置：

- `oldStartIdx`
- `oldEndIdx`
- `newStartIdx`
- `newEndIdx`

在循环体内，每处理一个节点，就将下标向指定的方向移动一个位置。处理时一般是对两个节点进行操作，相当于一次处理两个节点，就会将新、就两个节点各向指定方向移动一个位置

`oldStartIdx`和`newStartIdx`只能向后移动，`oldEndIdx`和`newEndIdx`只能向前移动，当开始位置大于等于结束位置时，说明所有节点都遍历过了，结束循环

```JS
while(oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
  // 比对节点
}
```

如果新旧节点数量不一致时，循环不会覆盖所有节点，这样正是预期的结果，如果newChildren中有没遍历的节点（即下标在`newStartIdx`和`newEndIdx`之间的节点），那么说明这些节点都是需要新增的节点（即下标在`oldStartIdx`和`oldEndIdx`之间的节点），直接插入DOM就行。如果oldChilren中有未遍历的节点，那么这些都是要删除的节点，直接删除即可

这样可以减少一部分遍历次数，提升性能

### `key`

熏染列表时，推荐使用`key`，因为如果在列表循环时，如果设置了`key`，那么在oldChildren中循环相同节点时，就不需要通过遍历来查找节点，直接通过`key`就可以知道哦啊哦了

Vue是推荐在列表渲染时使用`key`，并且使用`key`来查找子节点的方式也是排在上面提到的四种快捷查找方式之后，我理解是由于使用`key`还是需要新建一个`key`与`index`索引关系的对象，而那四种快捷查找方式并不需要新建这一个对象，为所有的组件都应`key`来进行标识，开销太大。而在列表渲染时使用`key`可以有效的提升列表顺序变化导致的循环遍历
