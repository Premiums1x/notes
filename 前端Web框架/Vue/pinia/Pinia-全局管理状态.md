## 创建pinia实例并挂载到vue应用实例上：

- Vue3：
在创建好Vue的应用实例app，基础上：
```
const pinia = createPinia()
app.use(pinia)
```

- Vue2:安装插件，创建pinia实例，并在创建Vue应用实例时的配置对象中，的底部，传入所创建的pinia实例
```
Vue.use(PiniaVuePlugin)
const pinia = createPinia()
new Vue({ el: '#app',
 // 其他配置... 
 // ... // 请注意，同一个`pinia'实例 
 // 可以在同一个页面的多个 Vue 应用中使用。
 pinia, })
```

## Store:
保存状态和业务逻辑的实体, 承载全局状态，每个组件都可以读取和写入它，包含三部分内容：
- state
- getter
- action
相当于组件中的 `data`、 `computed` 和 `methods`。


## 定义Store：
- 引入pinia库的defineStore工厂函数
- 开始定义
```js
import { defineStore } from 'pinia'

defineStore('名称（唯一标识）', {
  state: () => ({
    数据
  }),
  getters: { },
  actions: {
    fn() { }
  }
})
```

或者用 setup 函数写法：
```js
defineStore('名称（唯一标识）', () => {
  const 数据 = ref(...)          // → state
  const 计算值 = computed(...)   // → getter
  function fn() { }              // → action
  return { 数据, 计算值, fn }
})
```

关键点：

| 选项式（第二个参数是对象） | setup 函数式（第二个参数是函数）    |
| ------------- | ---------------------- |
| `state` 工厂    | `ref()` / `reactive()` |
| `getters`     | `computed()`           |
| `actions`     | 普通 `function`          |
*注：用setup函数时，ref()对应state，computed()对应getter，function()对应action。*

  - 存到变量后导出

## 使用Store：
- import所需Store后，调用如：`const counter = useCounterStore()`，这样才将定义的store实例化
- 直接通过 `.`访问state和方法

一旦 store 被实例化，你可以直接访问在 store 的 `state`、`getters` 和 `actions` 中定义的任何属性。


## 解构Store：
`store` 是一个用 `reactive` 包装的对象，**我们不能对它进行解构**，否则拿到的变量的值失去响应性。
*注，上述对state和getter成立，因为这两者是数据需要保持响应式，函数不会变所以随便解构，故可解构actions！*


**从 store 中提取属性时保持其响应性，使用 `storeToRefs()`**
`const { name, doubleCount } = storeToRefs(store)`
  *先处理，再解构*
  