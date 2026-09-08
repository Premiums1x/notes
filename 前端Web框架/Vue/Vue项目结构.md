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