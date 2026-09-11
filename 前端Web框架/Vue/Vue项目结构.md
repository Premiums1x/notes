### main.js
此文件起“入口”作用，**浏览器需要一个明确的起点来运行你的应用**，而 Vue 应用也需要一个地方去“创建并挂载”到页面上。

书写代码步骤一般为：
1.创建Vue实例对象： ```
```
createApp(App)
```

2.然后把Vue应用全局都会应用的东西挂载到应用实例上：
如：
```
- 注册路由：`app.use(router)`
    
- 注册状态管理：`app.use(pinia)`
    
- 注册 UI 组件库/插件：`app.use(ElementPlus)` 之类
    
- 注册全局组件：`app.component('Xxx', Xxx)`
    
- 注册全局指令：`app.directive('focus', ...)`
    
- 设置全局属性：`app.config.globalProperties.$xxx = ...`
    
- 引入全局样式：`import './styles/index.css'`
```

3.挂载到页面的某个 DOM 节点
```
app.mount('#app')
```
作用是： 把 Vue 应用渲染到 `index.html` 里的 `<div id="app"></div>`

补充：也可以**作为webpack的entry指向的文件**
- Webpack 的 `entry` 指向这个 `main.js`，从它开始把 `App.vue`、路由、组件、样式、图片等一路追踪打包。

---

## Vue 原理复习（博客笔记）

> 来源：http://120.77.152.123:8088/posts/vue%e5%a4%8d%e4%b9%a0 ｜ 原发布日期：2026-09-02

main.js是入口，创建、管理Vue实例，Vue是在内存中提供JS逻辑实时管理、更新DOM

浏览器 -> Vite服务器 -> 返回index.html -> 发现\<div #app>是空的，继续向下找到\<script src="./main.js">，就找到js执行逻辑创建Vue实例 -> 注册根组件 -> Vue 根据 template / 状态维护 DOM

HTML文本语言描述结构 到 浏览器 HTML parse 到 内存中创建维护的DOM对象树 到 Vue来维护、动态实时更新DOM对象

HTML静态写死 / DOM实时更新（Vue的原理） + CSS = 渲染 UI

Router:

```
createRouter()创建路由实例
app.use()，在Vue应用中使用

需要router-view路由出口：即根据当前什么路由展示什么组件，
匹配规则定义：routes.js
[
  {
    path:"",
    component:xxx

   }
]
```

Pinia：

```
defineStore
↓
state
↓
多个组件拿到同一个 Store
↓
一个组件修改
↓
其他组件响应式更新

以及：
state / getters / **actions**
```
