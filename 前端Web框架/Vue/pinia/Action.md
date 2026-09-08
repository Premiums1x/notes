相当于组件中的 method，通过 `defineStore()` 中的 `actions` 属性来定义，用以定义业务逻辑

action 也可通过 `this` 访问**整个 store 实例**，

- 与getter不同的点：
**`action` 可以是异步的**，你可以在它们里面 `await` 调用任何 API，以及其他 action


## 访问其他action：
使用另一个 store 的话，直接定义当前store在 _action_ 中调用。


## 使用选项式 API时，对action的访问
1. **`setup()` 返回 store 实例**，组件中通过this拿到所需store，通过`.`在store中直接取得action的一个对应方法。
2. **`mapActions` 拍平**，映射完后，直接在模板里写 `{{ increment() }}`，不用 `this.counterStore.increment()`。
跟 `mapState` 完全对称：

#### 对map映射的总结：
- `mapState` 把 state/getter 放进 `computed`，模板直接当属性用
- `mapActions` 把 action 放进 `methods`，模板直接当方法用
本质都是**省掉 `store.` 这个前缀**。

**误解**：**mapstate**不是应该放data中？
❌：`mapState` 虽然映射的是 state（数据），但必须放在 `computed` 里才能保持响应式。放 `data` 里就断开了连接，变成一次性快照了。

## 订阅action：监听action的执行：
通过 `store.$onAction()` 来监听 action 和它们的结果。

**`$onAction` 就是给 store 的 action 加**前后钩子**——每次 action 执行时自动触发你定义的回调**
```js
someStore.$onAction(({ name, store, args, after, onError }) => {
  // 这段代码在 action 执行前运行
  console.log(`"${name}" 开始执行`)
  
  after((result) => {
    // action 成功完成后运行
    console.log(`"${name}" 执行成功, 结果是:`, result)
  })
  
  onError((error) => {
    // action 报错时运行
    console.log(`"${name}" 执行失败:`, error)
  })
})

```

**主要用途：**:
1. **日志/埋点** — 记录每个 action 调用耗时、参数、结果
2. **错误追踪** — action 报错时统一捕获上报
3. **loading 状态** — action 开始时设 loading=true，完成后设 loading=false

**生命周期：**
- 在 `setup()` 里订阅 → 组件卸载时自动取消绑定
- 传第二个参数 `true` → 脱离组件生命周期，组件卸载后仍然存活
- 调用返回的 `unsubscribe()` → 手动取消

跟 `watch` 监听 state 变化不同，`$onAction` 监听的是**行为本身**（调用了哪个函数、传了什么参、执行了多久、成功了还是报错了），适合做跨 action 的统一拦截逻辑。
