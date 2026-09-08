# Vue Router 基础

## route 与 router

`route`：当前路由信息对象，表示当前 URL 被解析后的结果，包括路径、参数、query、匹配到的路由记录等。

`router`：路由器实例，负责管理路由表和导航行为。

常用访问方式：

```js
// Vue3 组合式 API
const route = useRoute()
const router = useRouter()
```

Vue2 / 选项式 API：

```js
this.$route
this.$router
```

## 创建并注册路由

```js
const router = createRouter({
  history: createWebHistory(),
  routes: [
    { path: '/home', component: Home }
  ]
})

createApp(App).use(router).mount('#app')
```

`app.use(router)` 的作用：

- 注册 `<RouterView>` 和 `<RouterLink>`
- 添加 `$route` 和 `$router`
- 启用 `useRoute()` 和 `useRouter()`
- 解析初始路由

`<RouterView>`：显示当前路由匹配到的组件。

`<RouterLink>`：生成跳转链接。

## 动态路由参数

```js
{ path: '/users/:id', component: User }
```

这些路径都会匹配：

```txt
/users/1
/users/100
/users/zhangsan
```

对应参数：

```js
route.params.id
```

注意：`params` 里的值通常是字符串。

如果同一个组件只是参数变了，组件不会重新创建，需要监听参数变化：

```js
watch(
  () => route.params.id,
  (newId, oldId) => {
    // 根据新 id 重新请求数据
  }
)
```

也可以用路由守卫：

```js
onBeforeRouteUpdate((to, from) => {
  // 使用 to.params.id
})
```

## 路由匹配规则

可选参数：

```js
{ path: '/users/:id?' }
```

正则限制参数：

```js
{ path: '/:orderId(\\d+)' }
```

可重复参数：

```js
{ path: '/:chapters+' }
```

可重复参数匹配多段路径时，`params` 会变成数组。

`sensitive: true`：区分大小写。

`strict: true`：严格匹配尾部 `/`。

## 嵌套路由

父组件里有自己的 `<router-view>` 时，需要配置 `children`。

```js
{
  path: '/user/:id',
  component: User,
  children: [
    { path: '', name: 'user-home', component: UserHome },
    { path: 'profile', component: UserProfile }
  ]
}
```

要点：

- `children` 里的 `path: ''` 是默认子路由
- 访问 `/user/:id` 时，会显示父组件和默认子组件
- 子路由路径不要以 `/` 开头，否则会变成根路径

## 命名路由

```js
{ path: '/user/:username', name: 'profile', component: User }
```

跳转时可以不用写死路径：

```vue
<RouterLink :to="{ name: 'profile', params: { username: 'erina' } }">
  User profile
</RouterLink>
```

等价于访问：

数据流向：**正常访问（URL → params）：**
```
/user/123  →  Vue Router 解析  →  params: { userID: '123' }  →  传给组件
```

**`router-link` 跳转（params → URL）：**
```
{ name: 'user', params: { userID: '123' } }  →  Vue Router 拼接路径  →  /user/123  →  跳转
```

正常是"从 URL 里解出参数"，编程跳转是"我们直接告诉路由对象参数值，让它去拼出 URL 再跳"。本质就是**手动给路由参数赋值**

```txt
/user/erina
```

路由 `name` 必须唯一。

## 编程式导航

`<RouterLink :to="...">` 本质上相当于调用：

```js
router.push(...)
```

常见写法：

```js
router.push('/home')

router.push({
  name: 'profile',
  params: { username: 'erina' }
})
```

`push` 会新增历史记录，可以后退。

`replace` 会替换当前记录，不能回退到原页面：

```js
router.replace('/home')

router.push({ path: '/home', replace: true })
```

历史前进/后退：

```js
router.go(1)
router.go(-1)
router.forward()
router.back()
```

## 命名视图

默认情况下，一个路由渲染一个组件，用 `component`。

如果同一个路由要渲染多个区域，需要多个命名 `<router-view>`：

```vue
<router-view />
<router-view name="sidebar" />
```

路由配置改成 `components`：

```js
{
  path: '/layout',
  components: {
    default: Main,
    sidebar: Sidebar
  }
}
```

## 重定向与别名

重定向：访问 A，实际跳到 B。

```js
{ path: '/home', redirect: '/' }
```

也可以重定向到命名路由，或者使用函数返回重定向目标：

```js
{
  path: '/users/:id/posts',
  redirect: to => to.path.replace(/posts$/, 'profile')
}
```

这里的 `to` 是目标路由对象，不是单纯的 path 字符串。

别名：访问 A 和访问 B 效果一样，但 URL 不一定改变。

```js
{ path: '/user', component: User, alias: '/member' }
```

如果原路径有参数，别名也要带对应参数。

## props 传参

不要让组件强依赖 `$route.params`，更推荐把路由参数作为 `props` 传给组件。

```js
{ path: '/user/:id', component: User, props: true }
```

组件里直接接收：

```js
defineProps(['id'])
```

三种模式：

```js
props: true
props: { fixed: true }//固定props
props: route => ({ query: route.query.q })//函数模式
```

注意：`props: true` 只会传 `params`。

如果一个路由渲染多个命名视图，需要分别给每个组件配置 props。

## RouterLink 激活样式

当前路由匹配某个 `<RouterLink>` 时，会自动加 class：

```txt
router-link-active
router-link-exact-active
```

区别：

- `router-link-active`：相关匹配，父级也算
- `router-link-exact-active`：完全匹配
- `query` 不参与 active 判断

局部改类名：

```vue
<RouterLink active-class="active" exact-active-class="exact-active" />
```

全局改类名：

```js
createRouter({
  history: createWebHistory(),
  routes,
  linkActiveClass: 'active',
  linkExactActiveClass: 'exact-active'
})
```

## 三种 History 模式

Hash 模式：

```js
createWebHashHistory()
```

- URL 带 `#`，例如 `/#/user/100`
- `#` 后面的内容不会发送给服务器
- 部署简单，刷新不容易 404

HTML5 模式：

```js
createWebHistory()
```

- URL 正常，例如 `/user/100`
- 更推荐用于正式项目
- 部署时服务器要配置 fallback 到 `index.html`
- 否则刷新子路由可能 404

Memory 模式：

```js
createMemoryHistory()
```

- 不依赖浏览器 URL
- 主要用于 SSR、Node 环境或测试
- 普通前端项目很少用

一句话：Hash 模式部署简单；HTML5 模式 URL 好看但要配服务器回退；Memory 模式普通项目基本不用。