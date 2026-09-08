## 如 useStore() 在路由中的调用时机

### 原因

`useStore()` 必须在 `app.use(pinia)` 之后调用，否则拿不到 pinia 实例，直接报错。

### 失败 vs 成功

| 写法 | 调用时机 | 结果 |
|---|---|---|
| 在模块顶层：`const store = useStore()` | router.js 被 import 时立即执行（app 还没启动完） | ❌ pinia 未就绪 |
| 在守卫回调里：`router.beforeEach(() => { const store = useStore() })` | 导航发生时（app 早已启动完毕） | ✅ pinia 已就绪 |

### 一句话

**模块顶层是启动时跑，pinia 还没好；回调里是导航时跑，pinia 早就好了。** 把 `useStore()` 放在函数内部、延迟到用的时候再调，就不会出事。
