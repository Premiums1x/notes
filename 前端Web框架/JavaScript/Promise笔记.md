Promise 不是异步逻辑本身，而是**异步操作最终结果的代表对象**。

## Promise 表示什么

它主要记录两件事：

- 状态
- 成功值或失败原因

三种状态：

- pending：结果还没确定
- fulfilled：成功了
- rejected：失败了

这里的“状态”更准确地说，是**结果有没有确定**，不是任务内部执行到了哪一步。

## 创建 Promise 时传入的函数是什么意思

`new Promise((resolve, reject) => { // 启动异步任务 })`

这个函数会在 new Promise(...) 时**立刻执行**。

它的作用是：

- 启动异步任务
- 成功时调用 resolve(成功值)
- 失败时调用 reject(失败原因)

所以不是单纯“把逻辑包住”，而是：

**一边启动异步任务，一边让 Promise 表示这个任务未来的结果。**

## resolve 和 reject

- resolve(value)：表示成功，Promise 变成 fulfilled
- reject(reason)：表示失败，Promise 变成 rejected

注意：

- resolve 里传的是成功值
- reject 里传的是失败原因，不太适合叫“结果”

## 后续怎么用

可以通过：

- .then(...)
- .catch(...)
- await

来获取结果或继续处理。

## 最值得记住的一句话

**Promise 是异步结果的代表对象：在里面启动异步任务，用状态记录成功或失败，后续再通过 then/catch/await 处理结果**