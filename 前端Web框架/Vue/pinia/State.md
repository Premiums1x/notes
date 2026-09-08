state 被定义为一个返回初始状态的函数：```
```
state: () =>{
	return{
	}
}
```

State对TS的兼容：
可以用一个接口定义 state，并添加 `state()` 的返回值的类型。
接口类型**把数据的"模样"声明好，再让 Pinia 按这个模样组装数据**。


## 访问State：
- 通过 `store` 实例访问 state，直接对其进行读写。
- state 必须提前"登记"所有属性，不能临时加塞，否则不会成为响应式属性。
- 确实需要一个"可能之后才赋值的属性"时，先在 `state()` 里声明它，初始值给 `null` 或 `undefined`占位。

重置state：
- 选项式风格写法时：调用 store 的 `$reset()` 方法将 state 重置为初始值：`store.$reset()`
- setup函数式写法时：需在定义store实例时手动创建自己的 `$reset()` 方法：
`function $reset() { count.value = 0 }`

### Vue2写法的选项式API可通过mapState，把 store 里的属性摊开到组件的 `computed` 里

读取：映射完从 `this` 上读，不可写，否则报错
```js
1.数组写法，按原名映射

2.对象写法，映射时改名

3.支持访问this，TS无法推断类型
```

修改：通过this访问、修改
用 `mapWritableState()` 作为代替。但不能像 `mapState()` 那样传递一个函数：

## 批量变更State：调用 `$patch` 方法。
1. 它允许你传入一个 `state` 的补丁对象在同一时间更改多个属性。
2. 接受一个函数来实现难以用补丁对象实现的变更。（对集合元素的改动）


## 替换state：
1. **不能完全替换掉** store 的 state，因为那样会破坏其响应性。
例：想通过=赋值，来用一个新对象覆盖掉整个state：`store.$state = { count: 24 }`,
对这种情况，Pinia 做拦截：
- 虽然写的是 `=` 赋值，但它**不会真正替换**底层的响应式对象
- 内部被转成了 `$patch({ count: 24 })`，只做**合并更新**，
- 即只改 `count` 这一个属性，state 里其他属性保持不变。

2. 设置pinia实例的state值来为所有**store 批量设置初始 state**：
```js
pinia.state.value = {
  counter: { count: 10 },       // counter store 的 state
  user: { name: '张三', age: 18 } // user store 的 state
}
```


## 订阅state：
通过 store 的 `$subscribe()` 方法侦听 state 及其变化。
可通过watch监听pinia上的state，以达到只要在 `pinia.state` 这棵树里有变化，就去作出对应操作(前提是deep:true，以监听嵌套对象的深层变化)

#### 底层实现：
Vue的watch，可以传入与 `watch()` 相同的选项，可实现每次state变化就立即触发订阅

## 取消订阅：
_state subscription_ 会默认被绑定到添加它们的组件上，组件卸载跟着消失，若想将订阅和组件分离开，在`$subscribe`订阅时：将 `{ detached: true }` 作为第二个参数