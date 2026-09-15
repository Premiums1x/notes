# Vue 组件库实战入门：从页面拼装到完整项目工程化

> 目标：不是“把组件库文档学一遍”，而是通过实际开发过程理解：
>
> **组件库到底怎么参与一个真实 Vue 项目的开发。**

本文以：

```text
Vue 3
Element Plus
Vue Router
Pinia
Axios
Vite
```

为主要技术栈。

项目案例以 **ToB 后台管理系统** 为主，同时穿插 ToC 项目的思路。

---

# 一、先理解：组件库到底解决什么问题？

假设我们完全不用组件库，现在要开发一个后台管理系统。

我们需要自己实现：

```text
按钮
输入框
下拉框
表单
表格
弹窗
分页
日期选择器
菜单
通知
加载动画
文件上传
树结构
……
```

而且还要处理：

```text
样式
交互
禁用状态
响应式
校验
浏览器兼容
键盘操作
可访问性
```

工作量非常大。

所以真实项目通常不会自己从零实现这些基础 UI。

而是：

```text
Vue
负责：
数据、状态、组件逻辑、页面逻辑

Element Plus
负责：
按钮、表单、表格、弹窗等 UI 基础设施
```

可以简单理解：

```text
组件库 = 已经做好的 UI 乐高积木
```

我们的工作是：

```text
选择组件
+
组合组件
+
绑定数据
+
加入业务逻辑
+
连接后端
```

---

# 二、第一阶段：先把一个组件库页面跑起来

不要一开始研究：

```text
Router
Pinia
Axios
权限系统
二次封装
Webpack
工程化
```

第一步只有一个目标：

> 先看到 Element Plus 的按钮出现在页面上。

---

# 三、创建 Vue 项目

创建：

```bash
npm create vue@latest
```

推荐初学阶段选择：

```text
Add TypeScript?      No
Add JSX?             No
Add Vue Router?      Yes
Add Pinia?           Yes
Add Vitest?          No
Add E2E Testing?     No
Add ESLint?          Yes
Add Prettier?        Yes
```

如果目前只想体验组件库，也可以暂时不选 Router 和 Pinia。

进入项目：

```bash
cd my-admin
npm install
npm run dev
```

项目基本结构：

```text
src/
├── assets/
├── components/
├── router/
├── stores/
├── views/
├── App.vue
└── main.js
```

---

# 四、安装 Element Plus

```bash
npm install element-plus
```

为了先理解，不急着研究按需加载。

直接全量引入：

```js
// main.js

import { createApp } from 'vue'
import ElementPlus from 'element-plus'
import 'element-plus/dist/index.css'

import App from './App.vue'

const app = createApp(App)

app.use(ElementPlus)

app.mount('#app')
```

然后：

```vue
<template>
  <el-button type="primary">
    新增
  </el-button>
</template>
```

运行：

```bash
npm run dev
```

看到 Element Plus 按钮。

到这里你已经完成：

```text
Vue 项目
    ↓
安装 Element Plus
    ↓
注册组件库
    ↓
使用组件
```

---

# 五、第一条组件库开发原则

以后看到一个 UI 需求，不要第一反应：

> 这个东西 HTML 怎么写？

而应该先想：

> 组件库里有没有现成组件？

比如需求：

```text
点击新增
弹出窗口
里面填写用户名
选择用户角色
点击确定保存
```

可以直接映射：

```text
新增按钮
→ el-button

弹窗
→ el-dialog

表单
→ el-form

输入框
→ el-input

角色选择
→ el-select
```

于是：

```text
业务需求
↓
拆 UI
↓
寻找组件库组件
↓
组合组件
↓
绑定 Vue 数据
```

这就是组件库开发的核心流程。

---

# 六、第二阶段：做第一个真实页面

我们现在做一个：

# 用户管理页面

页面结构：

```text
用户管理

[用户名输入框] [状态选择框] [查询] [重置]

[新增用户]

------------------------------------
用户名 | 手机号 | 状态 | 操作
------------------------------------
张三   | ...   | 正常 | 编辑 删除
李四   | ...   | 禁用 | 编辑 删除
------------------------------------

           分页
```

拆解：

```text
查询区域
→ Form

按钮
→ Button

列表
→ Table

状态
→ Tag

分页
→ Pagination

新增编辑
→ Dialog
```

---

# 七、使用表单组件

```vue
<script setup>
import { reactive } from 'vue'

const query = reactive({
  username: '',
  status: ''
})

function search() {
  console.log(query)
}

function reset() {
  query.username = ''
  query.status = ''
}
</script>

<template>
  <el-form :inline="true" :model="query">

    <el-form-item label="用户名">
      <el-input
        v-model="query.username"
        placeholder="请输入用户名"
      />
    </el-form-item>

    <el-form-item label="状态">
      <el-select
        v-model="query.status"
        placeholder="请选择"
        style="width: 120px"
      >
        <el-option label="正常" value="normal" />
        <el-option label="禁用" value="disabled" />
      </el-select>
    </el-form-item>

    <el-form-item>
      <el-button type="primary" @click="search">
        查询
      </el-button>

      <el-button @click="reset">
        重置
      </el-button>
    </el-form-item>

  </el-form>
</template>
```

这里要注意：

Element Plus 并没有替代 Vue。

组件库提供的是：

```vue
<el-input>
<el-select>
<el-button>
```

真正的数据管理仍然是 Vue：

```js
const query = reactive({
  username: '',
  status: ''
})
```

通过：

```vue
v-model="query.username"
```

把组件和数据连接起来。

所以：

```text
Element Plus
负责 UI

Vue
负责状态和逻辑
```

---

# 八、加入表格

先准备假数据：

```js
const users = [
  {
    id: 1,
    username: '张三',
    phone: '13800000000',
    status: 'normal'
  },
  {
    id: 2,
    username: '李四',
    phone: '13900000000',
    status: 'disabled'
  }
]
```

表格：

```vue
<el-table :data="users">

  <el-table-column
    prop="username"
    label="用户名"
  />

  <el-table-column
    prop="phone"
    label="手机号"
  />

  <el-table-column label="状态">

    <template #default="{ row }">

      <el-tag
        :type="row.status === 'normal' ? 'success' : 'danger'"
      >
        {{ row.status === 'normal' ? '正常' : '禁用' }}
      </el-tag>

    </template>

  </el-table-column>

  <el-table-column label="操作">

    <template #default="{ row }">

      <el-button
        link
        type="primary"
        @click="editUser(row)"
      >
        编辑
      </el-button>

      <el-button
        link
        type="danger"
        @click="deleteUser(row)"
      >
        删除
      </el-button>

    </template>

  </el-table-column>

</el-table>
```

这里非常重要。

组件库：

```text
负责表格展示
```

而我们：

```text
负责 users 数据
负责 row 怎么处理
负责点击编辑后发生什么
负责点击删除后发生什么
```

---

# 九、加入 Dialog：开始形成 CRUD

新增按钮：

```vue
<el-button
  type="primary"
  @click="dialogVisible = true"
>
  新增用户
</el-button>
```

状态：

```js
import { ref, reactive } from 'vue'

const dialogVisible = ref(false)

const form = reactive({
  username: '',
  phone: '',
  status: 'normal'
})
```

弹窗：

```vue
<el-dialog
  v-model="dialogVisible"
  title="新增用户"
  width="500px"
>

  <el-form :model="form">

    <el-form-item label="用户名">
      <el-input v-model="form.username" />
    </el-form-item>

    <el-form-item label="手机号">
      <el-input v-model="form.phone" />
    </el-form-item>

    <el-form-item label="状态">

      <el-select v-model="form.status">

        <el-option
          label="正常"
          value="normal"
        />

        <el-option
          label="禁用"
          value="disabled"
        />

      </el-select>

    </el-form-item>

  </el-form>

  <template #footer>

    <el-button @click="dialogVisible = false">
      取消
    </el-button>

    <el-button
      type="primary"
      @click="submit"
    >
      确定
    </el-button>

  </template>

</el-dialog>
```

现在一个最小后台页面已经成型：

```text
查询
+
表格
+
新增
+
编辑
+
删除
+
弹窗
```

这就是后台系统最典型的 CRUD 页面。

---

# 十、什么时候开始加入 Router？

现在假设项目增加页面：

```text
用户管理
商品管理
订单管理
系统设置
```

如果全部写在：

```text
App.vue
```

显然不合理。

这时引入：

```text
Vue Router
```

---

# 十一、Router 解决什么问题？

让：

```text
/user
```

显示：

```text
用户管理
```

让：

```text
/product
```

显示：

```text
商品管理
```

让：

```text
/order
```

显示：

```text
订单管理
```

所以 Router 可以粗暴理解成：

```text
URL
↓
找到对应 Vue 页面组件
↓
渲染
```

---

# 十二、配置 Router

目录：

```text
src/
├── views/
│   ├── UserView.vue
│   ├── ProductView.vue
│   └── OrderView.vue
│
└── router/
    └── index.js
```

router：

```js
import {
  createRouter,
  createWebHistory
} from 'vue-router'

const routes = [
  {
    path: '/user',
    component: () => import('../views/UserView.vue')
  },
  {
    path: '/product',
    component: () => import('../views/ProductView.vue')
  },
  {
    path: '/order',
    component: () => import('../views/OrderView.vue')
  }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

export default router
```

在：

```text
main.js
```

注册：

```js
import router from './router'

app.use(router)
```

然后：

```vue
<router-view />
```

Router 会根据当前 URL：

```text
/user
/product
/order
```

决定这里显示什么。

---

# 十三、开始做后台 Layout

后台一般长这样：

```text
┌──────────────────────────────┐
│ Header                       │
├──────────┬───────────────────┤
│          │                   │
│ Sidebar  │     RouterView    │
│          │                   │
│          │                   │
└──────────┴───────────────────┘
```

使用 Element Plus：

```vue
<template>

  <el-container class="layout">

    <el-aside width="200px">

      <el-menu router>

        <el-menu-item index="/user">
          用户管理
        </el-menu-item>

        <el-menu-item index="/product">
          商品管理
        </el-menu-item>

        <el-menu-item index="/order">
          订单管理
        </el-menu-item>

      </el-menu>

    </el-aside>

    <el-container>

      <el-header>
        后台管理系统
      </el-header>

      <el-main>
        <router-view />
      </el-main>

    </el-container>

  </el-container>

</template>
```

现在：

```text
Element Plus
负责整个后台 UI 框架

Router
负责页面切换
```

组件库开始和 Vue 生态配合起来了。

---

# 十四、什么时候需要 Pinia？

假设登录后获得用户：

```js
{
  id: 1,
  username: 'admin',
  token: 'xxxxx',
  avatar: '...'
}
```

这些数据：

```text
Header 要用
Sidebar 要用
用户中心要用
权限系统要用
请求接口也要用 token
```

如果全部通过：

```text
props
emit
```

层层传递，就非常麻烦。

于是需要：

```text
Pinia
```

---

# 十五、Pinia 可以粗暴理解成什么？

普通组件数据：

```text
属于某个组件
```

Pinia：

```text
属于整个应用
```

适合保存：

```text
当前用户
token
权限
购物车
主题
语言
全局设置
```

---

# 十六、创建 user store

```text
src/
└── stores/
    └── user.js
```

```js
import { defineStore } from 'pinia'
import { ref } from 'vue'

export const useUserStore = defineStore(
  'user',
  () => {

    const token = ref('')

    const userInfo = ref(null)

    function login(data) {
      token.value = data.token
      userInfo.value = data.user
    }

    function logout() {
      token.value = ''
      userInfo.value = null
    }

    return {
      token,
      userInfo,
      login,
      logout
    }
  }
)
```

组件里：

```js
import { useUserStore } from '@/stores/user'

const userStore = useUserStore()

console.log(userStore.userInfo)
```

任何页面都可以访问。

---

# 十七、到这里项目结构开始变得真实

```text
src/

├── api/
│   ├── user.js
│   ├── product.js
│   └── order.js
│
├── components/
│   ├── SearchForm.vue
│   └── Pagination.vue
│
├── layout/
│   └── AdminLayout.vue
│
├── router/
│   └── index.js
│
├── stores/
│   ├── user.js
│   └── app.js
│
├── views/
│   ├── user/
│   │   └── index.vue
│   ├── product/
│   │   └── index.vue
│   └── order/
│       └── index.vue
│
├── App.vue
└── main.js
```

---

# 十八、下一阶段：接真实后端

安装 Axios：

```bash
npm install axios
```

创建：

```text
src/utils/request.js
```

```js
import axios from 'axios'

const request = axios.create({
  baseURL: '/api',
  timeout: 10000
})

export default request
```

用户 API：

```text
src/api/user.js
```

```js
import request from '@/utils/request'

export function getUserList(params) {
  return request({
    url: '/users',
    method: 'get',
    params
  })
}

export function createUser(data) {
  return request({
    url: '/users',
    method: 'post',
    data
  })
}
```

页面：

```js
const users = ref([])

async function loadUsers() {

  const res = await getUserList(query)

  users.value = res.data
}
```

组件：

```vue
<el-table :data="users">
```

整个逻辑：

```text
页面加载
↓
调用 getUserList()
↓
Axios 请求后端
↓
后端查询数据库
↓
返回 JSON
↓
users.value = res.data
↓
Element Plus 表格重新渲染
```

这就是一个真正的前后端项目。

---

# 十九、组件库开发真正要掌握什么？

不是背 API。

真正要掌握的是：

```text
看到需求
↓
拆成 UI 模块
↓
找到对应组件
↓
看组件文档
↓
复制基础示例
↓
修改成业务需求
↓
绑定 Vue 数据
↓
加入事件
↓
接接口
```

例如：

```text
需要树形组织架构
```

查：

```text
Tree
```

需要：

```text
上传头像
```

查：

```text
Upload
```

需要：

```text
选择日期区间
```

查：

```text
DatePicker
```

需要：

```text
抽屉详情页
```

查：

```text
Drawer
```

组件库非常适合：

> **用到什么学什么。**

---

# 二十、不要一开始疯狂二次封装

这是后台项目常见误区。

刚开始：

```vue
<el-table>
<el-form>
<el-dialog>
```

写得很直接。

做久之后，有人开始封装：

```text
CommonTable
SearchForm
ConfigForm
ConfigTable
DynamicDialog
```

最后页面变成：

```vue
<CommonTable
  :columns="columns"
  :config="config"
  :formatter="formatter"
  :rules="rules"
/>
```

结果：

```text
原来 Element Plus 很简单

↓ 一顿封装

新人完全看不懂
```

正确方式应该是：

```text
第一次重复
→ 先写

第二次重复
→ 继续观察

第三第四次重复
→ 确认模式稳定

再封装
```

原则：

> **抽象来自重复，而不是想象。**

---

# 二十一、什么时候适合封装？

比如每个后台列表都有：

```text
查询
重置
分页
loading
```

可以把分页：

```vue
<AppPagination />
```

做成公共组件。

或者统一：

```text
状态标签
```

封装：

```vue
<StatusTag :status="row.status" />
```

这种通常比较安全。

但不要轻易把整个：

```text
Form
+
Table
+
Dialog
```

封成万能配置引擎。

除非你的业务确实大量重复。

---

# 二十二、完整 ToB 后台项目案例

假设开发：

# 外卖平台管理后台

模块：

```text
登录

首页 Dashboard

用户管理

商家管理

商品管理

订单管理

优惠券管理

评价管理

系统设置
```

组件库使用：

```text
Dashboard
→ Card
→ Statistic
→ Table

用户管理
→ Form
→ Table
→ Dialog
→ Pagination

商品管理
→ Upload
→ Input
→ Select
→ Table
→ Dialog

订单管理
→ Table
→ Tag
→ Drawer
→ DatePicker

系统设置
→ Form
→ Switch
→ Tabs
```

Vue Router：

```text
/dashboard
/users
/products
/orders
/settings
```

Pinia：

```text
用户登录状态
token
用户权限
全局主题
菜单状态
```

Axios：

```text
登录
查询订单
新增商品
删除商品
修改状态
```

于是：

```text
Element Plus
负责 UI

Vue
负责页面逻辑

Router
负责页面组织

Pinia
负责全局状态

Axios
负责前后端通信
```

这几个东西组合起来，就是一个典型 Vue 管理后台。

---

# 二十三、ToC 项目有什么不同？

比如：

# 电商网站

页面：

```text
首页
商品列表
商品详情
购物车
订单
个人中心
```

Pinia 会非常重要。

比如购物车：

```text
商品详情页
→ 加入购物车

Header
→ 显示购物车数量

购物车页
→ 展示商品

结算页
→ 获取购物车
```

购物车状态明显属于：

```text
多个组件共享状态
```

所以适合 Pinia：

```js
const cart = ref([])

function addProduct(product) {
  cart.value.push(product)
}
```

---

# 二十四、组件库是不是适合所有 ToC 项目？

不是。

Element Plus 特别适合：

```text
后台
管理系统
企业系统
工具系统
中后台
```

因为这些系统重视：

```text
效率
一致性
数据展示
表单
操作
```

而 ToC 页面：

```text
电商首页
音乐应用
社交平台
品牌官网
```

通常设计要求更强。

这时可能：

```text
组件库
+
自定义组件
+
大量自定义 CSS
```

甚至使用更偏设计系统的组件方案。

所以：

```text
ToB
组件库占比可能很高

ToC
组件库更多作为基础能力
```

---

# 二十五、当项目越来越大，会出现什么问题？

一开始：

```text
10 个组件
10 个页面
```

问题不明显。

后来：

```text
100 个组件
50 个页面
几十个依赖
大量图片
大量接口
```

就会出现：

```text
启动慢
打包慢
首屏慢
JS 太大
页面重复请求
组件频繁渲染
代码难维护
```

于是进入：

# 工程化和性能优化阶段

---

# 二十六、第一类优化：路由懒加载

不要：

```js
import UserView from '@/views/UserView.vue'
import ProductView from '@/views/ProductView.vue'
import OrderView from '@/views/OrderView.vue'
```

全部一次性加载。

使用：

```js
{
  path: '/user',
  component: () => import('@/views/UserView.vue')
}
```

这样：

```text
访问 /user
↓
才加载 UserView
```

而不是：

```text
进入网站
↓
所有页面全部加载
```

这叫：

```text
Code Splitting
代码分割
```

---

# 二十七、第二类优化：组件按需加载

早期为了学习方便：

```js
app.use(ElementPlus)
```

整个组件库一次引入。

项目较大时可以使用按需自动导入。

安装：

```bash
npm install -D unplugin-vue-components unplugin-auto-import
```

Vite：

```js
import { defineConfig } from 'vite'

import AutoImport from 'unplugin-auto-import/vite'

import Components from 'unplugin-vue-components/vite'

import {
  ElementPlusResolver
} from 'unplugin-vue-components/resolvers'

export default defineConfig({

  plugins: [

    AutoImport({
      resolvers: [
        ElementPlusResolver()
      ]
    }),

    Components({
      resolvers: [
        ElementPlusResolver()
      ]
    })

  ]

})
```

这样：

```vue
<el-button />
```

使用到什么组件，就自动导入什么。

---

# 二十八、第三类优化：不要无意义全局状态

Pinia 不是：

```text
什么都往里面塞
```

例如：

```text
弹窗是否打开
```

如果只属于某个页面：

```js
const dialogVisible = ref(false)
```

就应该放组件自己里面。

如果所有数据都塞 Pinia：

```text
store 越来越大
依赖混乱
状态来源不清晰
```

判断标准：

```text
只有当前组件使用
→ local state

父子组件共享
→ props / emit

多个无直接关系组件共享
→ Pinia
```

---

# 二十九、第四类优化：避免重复请求

比如：

```text
用户列表
```

每切换回来就请求一次。

是否应该缓存，要看数据实时性。

例如：

```text
菜单配置
用户信息
地区列表
系统字典
```

通常变化不频繁。

可以缓存：

```text
Pinia
LocalStorage
SessionStorage
```

但：

```text
订单状态
库存
支付结果
```

实时性高，不应该随便缓存。

---

# 三十、第五类优化：大型列表

如果一页展示：

```text
10000 条数据
```

DOM 节点非常多。

浏览器会卡。

正确做法通常是：

```text
后端分页
```

例如：

```http
GET /users?page=1&pageSize=20
```

后端：

```text
数据库只查 20 条
```

前端：

```text
只渲染 20 条
```

如果业务确实需要超长列表：

```text
Virtual List
虚拟列表
```

只渲染屏幕可见内容。

---

# 三十一、Vue 层面的性能优化

不要看到：

```text
computed
watch
ref
reactive
```

就乱用。

例如一个数据可以直接：

```js
const total = ref(10)
```

就不要为了“高级”写复杂 watch 链。

常见原则：

```text
能直接计算
→ computed

需要监听变化并产生副作用
→ watch

基础响应式值
→ ref / reactive
```

组件也不要拆得过碎。

比如：

```text
UserName.vue
UserPhone.vue
UserStatus.vue
```

这种通常没有意义。

更合理：

```text
UserForm.vue
UserTable.vue
UserDetail.vue
```

按业务模块拆。

---

# 三十二、Vite 是什么？

现代 Vue 项目通常使用：

```text
Vite
```

开发时：

```bash
npm run dev
```

实际上启动的是：

```text
Vite 开发服务器
```

负责：

```text
处理 Vue 文件
处理 ES Module
处理 CSS
HMR 热更新
开发环境模块加载
```

生产：

```bash
npm run build
```

则会：

```text
分析依赖
↓
处理源码
↓
压缩
↓
代码分割
↓
生成生产文件
```

输出：

```text
dist/
```

部署到：

```text
Nginx
CDN
静态服务器
```

即可。

---

# 三十三、那 Webpack 是什么？

Webpack 本质上是：

```text
模块打包工具
```

假设项目：

```text
main.js
↓
App.vue
↓
User.vue
↓
user.js
↓
axios
↓
CSS
↓
图片
```

Webpack 会从入口开始：

```text
分析 import
↓
建立依赖关系
↓
处理模块
↓
生成 bundle
```

可以粗略理解：

```text
项目代码

↓ Webpack

浏览器可以高效加载的生产资源
```

---

# 三十四、新 Vue3 项目为什么更常用 Vite？

传统 Webpack 开发模式更接近：

```text
启动
↓
先分析大量模块
↓
构建 bundle
↓
浏览器运行
```

项目越大，开发启动和更新可能越慢。

Vite 在开发环境充分利用：

```text
ES Modules
```

按需提供模块。

所以通常：

```text
开发服务器启动快
热更新快
配置相对简单
```

因此现代 Vue3 新项目：

```text
优先 Vite
```

---

# 三十五、那为什么还要学 Webpack？

因为企业中还有大量：

```text
Vue2
Vue CLI
老 Vue3 项目
历史管理后台
微前端系统
定制构建项目
```

这些项目很多仍然使用：

```text
Webpack
```

所以你至少应该理解：

```text
entry
output
loader
plugin
devServer
code splitting
tree shaking
cache
```

没必要刚开始就研究 Webpack 源码。

---

# 三十六、Webpack / Vite 优化到底在优化什么？

主要四件事：

```text
开发速度

打包速度

最终 bundle 体积

浏览器加载速度
```

---

# 三十七、Tree Shaking

假设一个库：

```js
export function A() {}

export function B() {}

export function C() {}
```

项目只：

```js
import { A } from './utils'
```

那么生产构建希望：

```text
A 保留
B 删除
C 删除
```

这类移除未使用代码的优化叫：

```text
Tree Shaking
```

---

# 三十八、代码分割

假设：

```text
首页 1MB
用户页 1MB
订单页 1MB
```

如果打一个：

```text
3MB main.js
```

用户访问首页：

```text
还得下载订单页面代码
```

没必要。

因此：

```text
首页 chunk
用户 chunk
订单 chunk
```

访问什么加载什么。

Router 懒加载：

```js
() => import('./User.vue')
```

就是非常典型的代码分割入口。

---

# 三十九、第三方依赖拆分

例如：

```text
Vue
Element Plus
Axios
ECharts
业务代码
```

如果全部塞一个 JS：

```text
main.js
```

文件会很大。

生产构建通常会拆成不同 chunk：

```text
vendor
UI library
chart library
business
```

这样：

```text
缓存效果更好
并行加载更合理
```

---

# 四十、静态资源优化

项目里还有：

```text
PNG
JPG
SVG
字体
视频
```

常见优化：

```text
压缩图片
使用 WebP / AVIF
SVG 图标
CDN
懒加载
合理尺寸
```

例如商品列表：

不要加载：

```text
4000 × 4000
```

的大图，然后显示成：

```text
100 × 100
```

这是纯浪费带宽。

---

# 四十一、项目后期最重要的不是“性能”，而是维护性

真实项目长期开发后最大的问题通常是：

```text
没人敢改
```

原因：

```text
目录混乱
命名混乱
重复代码
组件过度封装
API 到处写
状态到处存
魔法数字
没有统一规范
```

所以工程化不只是：

```text
Webpack
```

更重要的是：

```text
项目结构
代码规范
职责划分
```

---

# 四十二、推荐的中型项目结构

```text
src/

├── api/
│   ├── user.js
│   ├── product.js
│   └── order.js
│
├── assets/
│
├── components/
│   ├── common/
│   └── business/
│
├── layouts/
│   └── AdminLayout.vue
│
├── router/
│   └── index.js
│
├── stores/
│   ├── user.js
│   └── app.js
│
├── utils/
│   ├── request.js
│   ├── date.js
│   └── auth.js
│
├── views/
│   ├── login/
│   ├── user/
│   ├── product/
│   └── order/
│
├── App.vue
└── main.js
```

---

# 四十三、各目录职责

## views

完整业务页面。

例如：

```text
用户管理
订单管理
商品管理
```

---

## components

可复用组件。

例如：

```text
状态标签
上传组件
业务表格
搜索组件
```

---

## api

集中维护接口。

不要：

```vue
axios.get(...)
```

到处散落。

应该：

```js
getUsers()
createUser()
deleteUser()
```

---

## stores

全局状态。

例如：

```text
用户
权限
主题
购物车
```

---

## utils

纯工具能力。

例如：

```text
日期格式化
权限判断
金额格式化
请求封装
```

---

# 四十四、权限系统如何逐渐加入？

后台系统一般：

```text
管理员
运营
客服
商家
```

不同角色看到不同菜单。

登录后后端返回：

```js
{
  roles: ['admin'],
  permissions: [
    'user:view',
    'user:create',
    'user:delete'
  ]
}
```

放 Pinia：

```js
userStore.permissions
```

然后按钮：

```vue
<el-button
  v-if="hasPermission('user:create')"
>
  新增
</el-button>
```

路由也可以：

```js
{
  path: '/users',
  meta: {
    permission: 'user:view'
  }
}
```

Router 守卫：

```text
准备进入页面
↓
检查登录
↓
检查权限
↓
允许 / 拒绝
```

一个简单后台系统就逐渐演化成企业级系统了。

---

# 四十五、组件库开发的学习路线

不要：

```text
Element Plus 文档
从第一页看到最后一页
```

推荐：

## 第一阶段

只会：

```text
Button
Input
Select
Form
Table
Dialog
Pagination
```

做一个 CRUD 页面。

---

## 第二阶段

加入：

```text
Router
```

实现：

```text
多页面后台
```

---

## 第三阶段

加入：

```text
Pinia
```

实现：

```text
登录状态
用户信息
权限
```

---

## 第四阶段

加入：

```text
Axios
```

实现：

```text
真实后端 CRUD
```

---

## 第五阶段

继续组件库：

```text
Upload
Tree
Drawer
Tabs
DatePicker
Cascader
Message
MessageBox
```

---

## 第六阶段

学习项目结构：

```text
API
components
views
stores
utils
layouts
```

---

## 第七阶段

进入工程化：

```text
Vite
Webpack
环境变量
构建
代码分割
Tree Shaking
缓存
静态资源优化
```

---

# 四十六、一个完整练习项目

建议最终练：

# 外卖后台管理系统

## 阶段 1

只写：

```text
用户管理
```

假数据即可。

实现：

```text
查询
表格
分页
新增
编辑
删除
```

---

## 阶段 2

加入：

```text
商品
订单
商家
```

Router 管页面。

---

## 阶段 3

加入 Pinia：

```text
登录
token
用户信息
```

---

## 阶段 4

连接：

```text
Spring Boot
Node.js
Mock API
```

都可以。

---

## 阶段 5

实现：

```text
角色权限
菜单权限
按钮权限
```

---

## 阶段 6

优化：

```text
路由懒加载
组件按需加载
图片优化
接口缓存
分页
bundle 分析
```

---

## 阶段 7

构建：

```bash
npm run build
```

生成：

```text
dist/
```

部署到：

```text
Nginx
```

一个完整前端项目的生命周期基本就跑通了。

---

# 四十七、开发过程中你真正应该形成的思维

以后拿到需求：

```text
做一个订单管理页面
```

不要立刻写代码。

先拆：

```text
页面结构是什么？

需要哪些组件？

哪些是页面自己的状态？

哪些是全局状态？

哪些数据来自接口？

哪些组件值得复用？

哪些页面需要 Router？

哪些数据适合 Pinia？

是否需要权限？

数据量大不大？

是否需要分页？

是否需要懒加载？
```

然后再动手。

---

# 四十八、最终理解

组件库不是：

```text
学完才能开发的技术
```

它更像：

```text
开发过程中不断查阅的工具箱
```

真实开发过程应该是：

```text
需求
↓
页面拆解
↓
寻找组件
↓
拼 UI
↓
Vue 管状态和交互
↓
Router 管页面
↓
Pinia 管共享状态
↓
Axios 接后端
↓
抽取真正重复的组件
↓
性能优化
↓
Vite / Webpack 构建
↓
部署维护
```

最开始：

```text
Button
+
Input
+
Table
```

最后会逐渐长成：

```text
组件库
+
Vue
+
Router
+
Pinia
+
Axios
+
权限系统
+
工程化
+
性能优化
+
构建部署
```

这就是一个 Vue 项目从：

> **“把组件拼出来”**

逐渐演变成：

> **“可长期维护的真实业务系统”**

的完整过程。

---

# 一句话学习原则

> **不要先学完整个组件库再做项目，而是在真实页面中用到什么组件就查什么组件；当页面逐渐复杂，再自然引入 Router、Pinia、Axios、权限、封装和工程化。**

最终目标不是记住 Element Plus 有多少组件。

而是做到：

> **看到一个业务需求，能够迅速判断应该用哪些组件、状态放在哪里、页面如何组织、接口如何接入，以及项目变大以后该如何维护和优化。**
