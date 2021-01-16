---
title: Vue3.0-01 API学习
top: false
date: 2020-08-03 10:40:31
updated: 2020-08-03 10:40:31
tags:
- Vue
- Vue3
categories: Vue
---

Vue3已经了Beta阶段，主要API都固化了，学习了一下主要的API，搭建了一个[项目](https://github.com/duola8789/vue3-learning/)，算是尝鲜吧。

<!-- more -->

# 升级到Vue3

## 升级VueCLI

VueCLI需要在4.3.1以上才可以支持Vue3

```BASH
npm update -g @vue/cli

vue -V
@vue/cli 4.4.6
```

## 创建项目

```BASH
vue create vue3-learning
vue add vue-next # 添加 vue3 插件升级为 vue3
```

在创建项目时选择手动添加配置，选择vue-router和Vuex，这样创建完的项目各个插件也都会升级为支持Vue3的版本

```JS
{
  "dependencies": {
    "core-js": "^3.6.5",
    "vue": "^3.0.0-beta.1",
    "vue-router": "^4.0.0-alpha.6",
    "vuex": "^4.0.0-alpha.1"
  }
}
```

## 创建Vue实例

```JS
import {createApp} from 'vue';
import App from './App.vue';
import router from './router';
import store from './store';

createApp(App)
  .use(router)
  .use(store)
  .mount('#app');
```

## 创建Router

```JS
import {createRouter, createWebHistory} from 'vue-router';

const router = createRouter({
  history: createWebHistory(process.env.BASE_URL),
  routes
});

export default router;
```

创建路由的方式与以前有所变化，路由模式除了Hash模式（`createWebHashHistory`）和History模式（`createWebHistory`），还多了带缓存的History路由（`createMemoryHistory`）

使用路由跳转的API也有所变化：

```JS
import {useRouter} from 'vue-router';

export default {
  setup() {
    const router = useRouter();
    const goHome = () => router.push('/');
    return {goHome}
  }
}
```

关于router的具体变化，后面再单独学习

## 创建Store

```JS
import {createStore} from 'vuex';

export default createStore({
  state: {
    count: 0
  },
  mutations: {
    changeCount(state, isAdd) {
      state.count = isAdd ? state.count + 1 : state.count - 1;
    }
  },
  actions: {},
  modules: {}
});
```

使用：

```JS
import {useStore} from 'vuex';

export default {
  setup() {
    const store = useStore();
    const storeState = store.state;
    return {storeState}
  }
}
```

可以看出来，Vuex和vue-router，API都有了一些变化，与React Hooks的API很类似，但是基本原理没有太大变化

# `setup`

`setup`函数是新的Composition API的入口点

## 调用时机

Props初始化后就会调用`setup`函数，在`beforeCreate`钩子前被调用

## 返回值

`setup`返回的对象的属性将被合并到组件模板的渲染上下文，也可以返回一个渲染函数

## 参数

接受`props`作为第一个参数，使用的时候，需要首先声明`props`：

```JS
export default {
  props: {
    name: String,
  },
  setup(props) {
    console.log(props.name)
  },
}
```

`props`是响应式的，前提是不对`props`进行解构，解构后会失去响应性

`setup`第二个参数是上下文对象，从2.x中的`this`选择性地暴露出一些属性，例如`attrs`/`slots`/`emit`等，可以解构取值不会失去响应性

## `this`的用法

`this`在`setup`中不可用

# 响应式系统API

## `reactive`


```JS
const obj = reactive({ count: 0 })
```

返回一个普通对象的响应式代理，响应式转换是深层的，会影响对象内部嵌套的属性，基于ES2015的Proxy实现，返回的代理对象不等于原始对象，避免使用原始对象

经过试验，Vue3中可以通过修改数组下标来响应式的更改数组成员的值了

## `ref`

`ref`的引入是为了以变量形式传递响应式的值而不再依赖访问`this`:

```JS
const count = ref(0)
```

接受一个参数，返回一个响应式可改变的`ref`对象，`ref`对象拥有一个指向内部值的单一属性`.value`

`ref`主要目的是保证基本类型值的响应式，如果传入的参数不是基本类型，会调用`reative`方法进行深层响应式转换

`ref`使用时：

- `ref`的返回值`setup`中返回应用在模板中时，会自动解构，不需要书写`.value`
- `ref`作为`reactive`对象的属性被修改或访问时，也会自动解构，不需要书写`.value`
- 通过`Array`、`Map`等原声集合类中范围`ref`时，不会自动解构，需要使用`.value`获取值

## `reactive` VS `ref`

使用`ref`和`reactive`的区别可以通过如何撰写编撰的JavaScript逻辑比较

```JS
// 风格 1: 将变量分离
let x = 0
let y = 0

function updatePosition(e) {
  x = e.pageX
  y = e.pageY
}

// --- 与下面的相比较 ---

// 风格 2: 单个对象
const pos = {
  x: 0,
  y: 0,
}

function updatePosition(e) {
  pos.x = e.pageX
  pos.y = e.pageY
}
```

使用`ref`就是将将风格（1）转换为使用`ref`，让基础类型值也具有响应性，使用`reactive`和风格（2）一致

只使用`reactive`的问题是，使用组合函数的时候必须始终保持对这个组合函数返回对象的引用以保持响应性，这个对象不能够被解构或者展开

```JS
// 组合函数：
function useMousePosition() {
  const pos = reactive({
    x: 0,
    y: 0,
  })

  // ...
  return pos
}

// 消费者组件
export default {
  setup() {
    // 这里会丢失响应性!
    const { x, y } = useMousePosition()
    return {
      x,
      y,
    }

    // 这里会丢失响应性!
    return {
      ...useMousePosition(),
    }

    // 这是保持响应性的唯一办法！
    // 你必须返回 `pos` 本身，并按 `pos.x` 和 `pos.y` 的方式在模板中引用 x 和 y。
    return {
      pos: useMousePosition(),
    }
  },
}
```

解决方法时使用`toRefs`将响应式对象的每个对象都转换为响应的`ref`：

```JS
function useMousePosition() {
  const pos = reactive({
    x: 0,
    y: 0,
  })

  // ...
  return toRefs(pos)
}

// x & y 现在是 ref 形式了!
const { x, y } = useMousePosition()
```

目前阶段可以从下面两种风格二选其一：

（1）如果在普通的JavaScript中声明基础变量类型与对象变量时一样区别使用`ref`和`reacitve`，也就是说如果声明响应式的基础类型使用`ref`，如果声明响应式对象变量使用`reactive`

（2）全部使用`reactive`，然后在组合函数返回对象时使用`toRefs`

目前（2020.08.01）官方对`ref`和`reactive`的最佳实践还没有建议，自己选择更适合自己的风格使用，我会选择风格1使用。

## `computed`

```JS
const count = ref(1)
const plusOne = computed(() => count.value + 1)
```

传入一个`getter`函数，也可以传入一个拥有`get`和`set`函数的对象

## `readonly`

```JS
const original = reactive({ count: 0 })

const copy = readonly(original)
```

传入一个对象（普通或者响应式对象）或`ref`，返回原始对象的深层的制度代理

## `watchEffect`

> 与React的`useEffect`非常类似

```JS
const count = ref(0)

watchEffect(() => console.log(count.value))
```

立即执行传入的函数，并响应式追踪依赖

### 停止监听

在`setup`中或生命周期钩子中被调用是，会被链接到组件的生命周期，在组件写在时自动停止

返回值是一个函数，可以手动来停止真挺

### 清除副作用

传入的函数中，可以接受`onInvalidate`函数作为入参，用来注册清理失效时的回调，在下面的情况中：

- 副作用即将重新执行
- 侦听器被停止

```JS
watchEffect((onInvalidate) => {
  const token = performAsyncOperation(id.value)
  onInvalidate(() => {
    // id 改变时 或 停止侦听时
    // 取消之前的异步操作
    token.cancel()
  })
})
```

### 执行时机

`watchEffect`会在组件初始运行时同步打印出来，在监听状态变化后，会在组件更新后执行副作用

在`watchEffect`中访问DOM，需要在`onMounted`钩子中进行

可以通过传递第二个参数的`flush`属性来改变副作用函数的执行时机：

- `post`，默认，在组件更新后执行
- `sync`，同步运行
- `pre`，在组件更新前执行

### 调试

在第二个参数中传入`onTrack`和`onTrigger`来调试

```JS
watchEffect(
  () => {
    /* 副作用的内容 */
  },
  {
    onTrigger(e) {
      debugger
    },
  }
)
```

`onTrack`在依赖被追踪时被调用，`onTrigger`在依赖变更导致副作用被触发时调用

仅在开发模式下生效

## `watch`

```JS
// 侦听一个 getter
const state = reactive({ count: 0 })
watch(
  () => state.count,
  (count, prevCount) => {
    /* ... */
  }
)

// 直接侦听一个 ref
const count = ref(0)
watch(count, (count, prevCount) => {
  /* ... */
})

// 侦听多个数据源
watch([fooRef, barRef], ([foo, bar], [prevFoo, prevBar]) => {
  /* ... */
})
```

与2.X的`watch`完全相同，与`watchEffect`相比的不同点：

- 懒执行
- 将依赖提取为第一个参数，更明确哪些状态的改变会重新运行副作用
- 可以访问侦听状态变化前后的值

# 生命周期

生命周期钩子函数只能在`setup`期间同步使用，在组件卸载时，生命周期内部创建的侦听器和计算状态也会被自动删除

与Vue2.x相比，`beforeCreated`和`created`被删除了，对应的逻辑在`setup`内部完成，其他的生命周期钩子都改为了`onXxx`的形式（`beforeDestoryed`改为了`onBeforeUnmount`，`destroyed`改为了`onUnmounted`）

![](http://image.oldzhou.cn/FkSOv8v0no7J-Hp3VxfrL-GF-qoj)

两个新增的调试钩子函数`onRenderTracked`和`onRenderTriggered`：

```JS
export default {
  onRenderTriggered(e) {
    debugger
    // 检查哪个依赖性导致组件重新渲染
  },
}
```
> 目前（2020.07.29）在Demo尝试调用这两个钩子函数，没有生效，不知道是Bug还是我调用的姿势不对

# 依赖注入

使用`provide`和`inject`实现依赖注入，与2.x版本中基本一致，只能在`setup`中使用

```JS
import { provide, inject } from 'vue'

const ThemeSymbol = Symbol()

const Ancestor = {
  setup() {
    provide(ThemeSymbol, ref('dark'))
  },
}

const Descendent = {
  setup() {
    const theme = inject(ThemeSymbol, ref('light') /* optional default value */)
    return {
      theme,
    }
  },
}
```

使用`ref`传值可以保证`provided`和`injected`之间值的响应性

# 模板Refs

Vue2.x中的`ref`原本是用于获取DOM的， Vue3中`ref`不仅可以响应化数据，也可以实现获取DOM的功能

```HTML
<template>
  <div ref="root"></div>
</template>

<script>
  import { ref, onMounted } from 'vue'

  export default {
    setup() {
      const root = ref(null)

      onMounted(() => {
        // 在渲染完成后, 这个 div DOM 会被赋值给 root ref 对象
        console.log(root.value) // <div/>
      })

      return {
        root,
      }
    },
  }
</script>
```

在`setup`中声明一个`ref`并返回，在模板中声明`ref`并且值与返回的`ref`相同，这时在渲染初始化后（`onMounted`）就可以获取分配的DOM或组件实例

在`v-for`中使用时，需要使用3.0新增的函数形的`ref`，为`ref`赋值：

```HTML
<template>
  <div v-for="(item, i) in list" :ref="el => { divs[i] = el }">
    {{ item }}
  </div>
</template>

<script>
  import { ref, reactive, onBeforeUpdate } from 'vue'

  export default {
    setup() {
      const list = reactive([1, 2, 3])
      const divs = ref([])

      // 确保在每次变更之前重置引用
      onBeforeUpdate(() => {
        divs.value = []
      })

      return {
        list,
        divs,
      }
    },
  }
</script>
```

# 响应式系统工具集

## `isRef`

判断值是否是`ref`对象

## `isProxy`

判断一个对象是否是由`reactive`或者`readonly`创建的代理

## isReactive

判断一个对象是否是由`reactive`创建的代理。

如果这个代理是由`readonly`创建的，但是又被`reactive`创建的另一个代理包裹了一层，那么同样也会返回`true`

## `isReadonly`

判断一个对象是否是由`readonly`创建的代理。

## `unref`

用来快速返回`ref`的值，如果参数是`ref`，返回它的`value`，否则返回参数本身。它是`val = isRef(val) ? ref.value : ref`的语法糖

## `toRef`

为`reactive`对象的属性创建一个`ref`，这个`ref`可以被传递并且保持响应性

```JS
const state = reactive({
  foo: 1,
  bar: 2,
})

const fooRef = toRef(state, 'foo')

fooRef.value++
console.log(state.foo) // 2

state.foo++
console.log(fooRef.value) // 3
```

当需要将一个Prop中的属性作为`ref`传给组合逻辑函数时，可以使用`toRef`

```JS
export default {
  setup(props) {
    useSomeFeature(toRef(props, 'foo'))
  },
}
```

> 目前还没有发现这种情况的实际场景

## `toRefs`

把一个响应式对象转换为普通对象，该普通对象的每个属性都是一个`ref`，与原来的响应式对象一一对应

```JS
const state = reactive({
  foo: 1,
  bar: 2,
})

const stateAsRefs = toRefs(state)
/*
stateAsRefs 的类型如下:

{
  foo: Ref<number>,
  bar: Ref<number>
}
*/

// ref 对象 与 原属性的引用是 "链接" 上的
state.foo++
console.log(stateAsRefs.foo) // 2

stateAsRefs.foo.value++
console.log(state.foo) // 3
```

当从一个组合逻辑中返回响应式对象时，用`toRefs`是很有效的，它可以让消费组件可以解构或者扩展返回的对象，而不是去响应性

```JS
function useFeatureX() {
  const state = reactive({
    foo: 1,
    bar: 2,
  })

  // 对 state 的逻辑操作

  // 返回时将属性都转为 ref
  return toRefs(state)
}

export default {
  setup() {
    // 可以解构，不会丢失响应性
    const { foo, bar } = useFeatureX()

    return {
      foo,
      bar,
    }
  },
}
```

# 高级响应式系统API

## `customRef`

用来自定义`ref`，可以显示依赖追踪和触发响应，接受一个函数，函数的两个参数是用于追踪的`track`和触发响应式的`trigger`，返回一个带有`get`和`set`属性的对象

可以使用自定义`ref`来实现带防抖功能的`v-model`


```JS
function useDebouncedRef(value, delay = 200) {
  let timeout
  return customRef((track, trigger) => {
    return {
      get() {
        track()
        return value
      },
      set(newValue) {
        clearTimeout(timeout)
        timeout = setTimeout(() => {
          value = newValue
          trigger()
        }, delay)
      },
    }
  })
}

export default {
  setup() {
    return {
      text: useDebouncedRef('hello'),
    }
  },
}
```
> 个人还不知道有什么好的应用场景，实际上上面的例子不通过`custromRef`来实现，可能灵活度还更大

## `markRaw`

显示标记一个对象永远不会转换为响应式dialing，返回这个对象本身

```JS
const foo = markRaw({})
console.log(isReactive(reactive(foo))) // false

// 如果被 markRaw 标记了，即使在响应式对象中作属性，也依然不是响应式的
const bar = reactive({ foo })
console.log(isReactive(bar.foo)) // false
```

它的作用是用来覆盖`reactive`为默认深层的特性，主要用来提升性能，比如一些值复杂的第三方类库的实例或者Vue组件对象，或者一个元素数量庞大，但是数据不可变，跳过Proxy也可以提升性能

这种标识只停留在根级别，`markRaw`对象的属性如果被`reactive`处理，仍然会返回一个响应式对象，并且导致原始值与Proxy值不同

```JS
const foo = markRaw({
  nested: {},
})

const bar = reactive({
  // 尽管 `foo` 己经被标记为 raw 了, 但 foo.nested 并没有
  nested: foo.nested,
})

console.log(foo.nested === bar.nested) // false
```

## `shallowReactive`

只为某个对象的私有（第一层）属性创建浅层次的响应式代理，不会对深层属性做深层次、递归地响应式代理

```JS
onst state = shallowReactive({
  foo: 1,
  nested: {
    bar: 2,
  },
})

// 变更 state 的自有属性是响应式的
state.foo++

// ...但不会深层代理
isReactive(state.nested) // false
state.nested.bar++ // 非响应式
```

## `shallowReadonly`

与`shallowReactive`类似，只为对象的私有（第一层）属性创建浅层的只读响应代理

## `shallowRef`

创建一个`ref`，将会追踪它的`.value`更改操作，但是不会对变更后的`.value`做响应式代理转换

```JS
const foo = shallowRef({})

// 更改对操作会触发响应
foo.value = {}

// 但上面新赋的这个对象并不会变为响应式对象
isReactive(foo.value) // false
```

注意，如果每次都为`foo.value`重新赋值，那么仍然会触发响应式改动。上面说的“不会变为响应式对象”指的是更改`value`的某个属性不会触发响应式改动

## `toRaw`

返回`reactive`或者`readonly`方法转换为响应式代理的普通对象。用于临时读取，访问不会被代理、跟踪，写入时不会触发更改。不建议一致持有原始对象的引用。

# 组合式API

## 更多的灵活性来自更多的自我克制

组合式API的初衷就是为了实现更有组织的代码，实现更灵活的逻辑提取与复用，在代码中会出现更多的、零碎的函数模块，在不同的位置、不同的组件间进行重复调用

它可以避免Vue2.x时代逻辑复用的几种主要形式（Mixin/HOS/SLOT）的弊端，带来了比较明显的好处：

![](http://image.oldzhou.cn/Fk7zo_E1Y0zYdzdz9xVr-RHH2NYy)

但是它在提到了代码质量的上限的同时，降低了下线，`setup`中会出现大量面条式的代码，避免这种糟糕情况的关键就是，将逻辑更合理的划分为单独的函数，将`setup`作为一个入口，在其中进行不同组合函数的调用。

## 与React Hooks比较

Vue3的基于函数的组合式API受到了React Hooks的启发，在很多思维模型方面与React Hooks很类似，提供了同等级别的逻辑组合能力，但是也有着比较明显的不同，组合式API的`setup`函数只会被调用一次，也就意味着使用组合式API时：

1. 不需要考虑调用顺序，可以用在条件语句中（React Hooks不可以）
2. 不会再每次渲染时重复执行，降低垃圾回收的压力（React Hooks每次渲染都会重复执行）
3. 不存在内联处理函数导致子组件永远更新的问题，也不需要`useCallback`（React Hooks需要用`useCallback`进行性能优化）
4. 不存在忘记记录依赖的问题，也不需要`useEffecr`和`useMemo`并传入依赖数组以捕获过时的变量，Vue的自动以来可以确保侦听器和计算值总是准确无误的（React Hooks需要手动记录依赖）

# 参考

- [Vue组合式API](https://composition-api.vuejs.org/zh/api/)
- [组合式 API 征求意见稿](https://composition-api.vuejs.org/zh/#%E6%A6%82%E8%BF%B0)
