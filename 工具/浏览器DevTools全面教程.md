# 浏览器 DevTools 全面教程：从接口调试到前端问题排查

## 1. DevTools 到底是什么？

浏览器 DevTools（开发者工具）可以简单理解为：

> 浏览器提供给开发者的一套“观察网页运行状态 + 直接干预网页运行”的工具。

平时我们在 IDE 里看到的是：

```text
源代码
↓
Vite / Webpack 构建
↓
浏览器执行
↓
页面最终运行
```

而 DevTools 看到的是：

```text
浏览器此时此刻真正运行的结果
```

所以很多问题，光看 IDE 里的代码是不够的。

例如：

- 接口到底有没有请求成功？
- 请求参数实际发了什么？
- 后端到底返回了什么？
- 为什么页面没有显示？
- 某个变量运行时到底是什么？
- CSS 为什么没生效？
- 按钮点了之后调用了哪个接口？
- 为什么接口请求两次？
- 为什么数据明明存在，字典却翻译失败？
- token 到底有没有放进请求头？
- 页面为什么越来越卡？

这些都非常适合通过 DevTools 排查。

---

# 2. 如何打开 DevTools

Windows 常用：

```text
F12
```

或者：

```text
Ctrl + Shift + I
```

打开 Console：

```text
Ctrl + Shift + J
```

检查页面元素：

```text
Ctrl + Shift + C
```

也可以：

```text
页面右键 → 检查
```

---

# 3. DevTools 最重要的几个面板

前端日常开发里最重要的是：

```text
Elements
Console
Sources
Network
Application
Performance
Memory
```

对于刚入门来说，可以先按优先级掌握：

```text
第一梯队：
Network
Console
Elements

第二梯队：
Sources
Application

第三梯队：
Performance
Memory
Lighthouse
```

其中日常业务开发大概 80% 的问题，都可以通过：

```text
Network + Console + Elements
```

解决。

---

# 4. Network：前端调接口最重要的工具

Network 可以理解为：

> 浏览器与服务器之间发生的所有网络通信记录。

例如 Vue 代码：

```js
axios.get('/api/user/list')
```

执行之后，浏览器真正发出去的 HTTP 请求，就能在 Network 中看到。

---

## 4.1 Network 到底在观察什么？

一次典型 Web 请求：

```text
前端代码
    ↓
axios / fetch
    ↓
浏览器
    ↓
HTTP 请求
    ↓
后端 Controller
    ↓
Service
    ↓
数据库
    ↓
后端返回 JSON
    ↓
浏览器
    ↓
axios 拿到 response
    ↓
Vue 更新页面
```

Network 观察的主要是：

```text
浏览器
    ↓
HTTP
    ↓
服务端
```

这一段。

因此当页面数据显示有问题时，可以先问：

> 后端到底有没有正确返回？

Network 就是最直接的证据。

---

# 5. Network 中一次请求应该看什么？

假设你调用：

```text
GET /api/user/list
```

点击 Network 中对应请求，一般会看到：

```text
Headers
Payload
Preview
Response
Initiator
Timing
```

这些非常重要。

---

# 6. Headers：检查请求基本信息

Headers 中最值得看的内容包括：

```text
Request URL
Request Method
Status Code
Request Headers
Response Headers
```

例如：

```text
Request URL:
http://localhost:8080/api/user/list

Request Method:
GET

Status Code:
200 OK
```

## 6.1 Request URL

可以检查：

```text
到底请求了哪个接口？
```

很多低级 bug 实际就是路径写错，或者环境变量配置错误。

---

# 7. HTTP 状态码要能快速判断

### 200

```text
请求成功
```

但注意：

```text
HTTP 200 ≠ 业务一定成功
```

有些后端会返回：

```json
{
  "code": 500,
  "message": "用户不存在"
}
```

HTTP 依然是 `200`，因此还需要看 Response。

### 400

```text
Bad Request
```

通常意味着请求参数有问题，例如缺字段、字段格式错误、参数无法反序列化。

### 401

```text
Unauthorized
```

通常和 token、登录状态、身份认证有关。

### 403

```text
Forbidden
```

一般表示已经知道你是谁，但你没有权限。

### 404

```text
Not Found
```

常见原因是接口路径错误、后端没有对应接口、代理配置错误。

### 500

```text
Internal Server Error
```

说明服务器执行过程中发生异常。

这种情况下继续盯着前端代码意义通常不大，应该：

```text
Network 看 Response
+
后端看日志
```

---

# 8. Payload：检查“前端到底发了什么”

这一点非常重要。

比如代码：

```js
axios.post('/api/user', {
  name,
  age
})
```

你觉得发的是：

```json
{
  "name": "Tom",
  "age": 18
}
```

但不要只相信代码。

真正应该去：

```text
Network
→ 请求
→ Payload
```

看实际发送的数据。

可能实际变成：

```json
{
  "name": "",
  "age": null
}
```

这时候就说明问题发生在前端。

---

# 9. Preview 和 Response

## Response

表示：

> 服务端实际返回的原始数据。

例如：

```json
{
  "code": 200,
  "data": [
    {
      "id": 1,
      "status": "1"
    }
  ]
}
```

## Preview

只是浏览器把 Response 格式化、折叠、树形展示，方便阅读。

通常查看复杂 JSON 时 Preview 更舒服，复制原始响应时 Response 更直接。

---

# 10. Network 最重要的一个思维

以后出现“页面数据不对”，不要第一反应就在 Vue 页面里乱找。

先判断问题在哪一层。

例如页面应该显示：

```text
已完成
```

结果显示：

```text
99
```

第一步：

```text
Network → Response
```

看看服务器返回的是不是 `99`。

如果后端就是返回 `99`，说明前端至少真实拿到了 `99`，然后再检查：

```text
99 是否存在于前端字典？
```

这就是分层排查。

---

# 11. Console：直接操作浏览器中的 JavaScript

Console 本质上就是：

> 一个可以直接在当前网页环境中执行 JavaScript 的 REPL。

例如：

```js
1 + 1
```

会得到：

```text
2
```

你也可以：

```js
const arr = [1, 2, 3, 4]

arr.filter(item => item > 2)
```

得到：

```js
[3, 4]
```

所以：

> Console 中写 JavaScript 的基本逻辑和 IDE 完全一样。

---

# 12. Console 和 IDE 的真正区别

IDE 里面：

```js
const arr = [1, 2, 3]
```

代码需要保存、构建、浏览器加载后执行。

而 Console：

```js
const arr = [1, 2, 3]
```

是立即在当前网页运行环境执行。

所以 Console 非常适合：

```text
验证 JS 写法
查看数据
临时过滤数据
测试 API
排查运行时问题
```

---

# 13. 一个非常重要的概念：作用域

例如 Vue 组件：

```js
export default {
  data() {
    return {
      tableData: []
    }
  }
}
```

你在 Console 输入：

```js
tableData
```

可能得到：

```text
ReferenceError: tableData is not defined
```

为什么？

因为 `tableData` 属于 Vue 组件作用域，而 Console 默认执行环境更接近 `window` 全局作用域。

所以：

```text
组件中的变量
≠
window 上的变量
```

---

# 14. window 是理解 Console 的关键

浏览器全局对象：

```js
window
```

例如：

```js
window.location
window.document
window.localStorage
```

都可以直接访问。

而：

```js
window.xxx = 123
```

之后：

```js
xxx
```

也可以访问。

因此开发调试时可以临时写：

```js
window.tableData = this.tableData
```

然后 Console：

```js
tableData
```

就能访问。

甚至：

```js
tableData.filter(...)
```

这种方法适合临时调试，正式代码调试完应该删除。

---

# 15. Network 里的 Response 怎么拿到 Console？

假设 Network 看到：

```json
[
  {
    "id": 1,
    "status": "1"
  },
  {
    "id": 2,
    "status": "99"
  }
]
```

最简单的方法就是复制，然后在 Console 中赋值：

```js
const data = [
  {
    id: 1,
    status: '1'
  },
  {
    id: 2,
    status: '99'
  }
]
```

之后就可以：

```js
data.filter(...)
```

---

# 16. 用 Console 检查字典翻译失败的数据

比如接口：

```js
const data = [
  { id: 1, status: '1' },
  { id: 2, status: '2' },
  { id: 3, status: '99' }
]
```

字典：

```js
const dict = [
  { value: '1', label: '待处理' },
  { value: '2', label: '已完成' }
]
```

检查无法翻译的数据：

```js
const errorData = data.filter(item => {
  return !dict.some(dictItem => {
    return dictItem.value === item.status
  })
})

console.log(errorData)
```

得到：

```js
[
  { id: 3, status: '99' }
]
```

这个过程非常适合 Console，因为你只是临时验证数据，而不是要修改项目逻辑。

---

# 17. Console 中非常实用的数据处理方法

前端调试建议熟练这些：

```text
filter
map
find
findIndex
some
every
includes
Set
Object.keys
Object.values
Object.entries
```

## filter

筛选：

```js
data.filter(item => item.status === '1')
```

## map

提取字段：

```js
data.map(item => item.status)
```

## Set

去重：

```js
[...new Set(data.map(item => item.status))]
```

如果接口返回 3000 条数据，你想看看 status 总共有哪些值，这个写法非常实用。

---

# 18. 一个非常推荐的数据排查套路

面对几千条接口数据，不要人工找。

假设：

```js
data
```

是接口返回数组。

第一步：

```js
data.length
```

看看数量。

第二步：

```js
data.slice(0, 5)
```

看看结构。

第三步：

```js
data.map(item => item.status)
```

看看字段。

第四步：

```js
[...new Set(data.map(item => item.status))]
```

看所有可能值。

第五步：

```js
data.filter(item => !dict.some(d => d.value === item.status))
```

找异常。

这就是利用代码分析数据，而不是肉眼查看。

---

# 19. console.table()

这是非常值得掌握的。

普通：

```js
console.log(data)
```

如果数据是数组对象，会是一坨展开结构。

可以使用：

```js
console.table(data)
```

浏览器会把对象数组展示成表格。

调后台管理系统非常舒服。

---

# 20. Copy as fetch：非常实用

Network：

```text
找到接口
→ 右键
→ Copy
→ Copy as fetch
```

浏览器会生成类似：

```js
fetch("http://localhost:8080/api/list", {
  headers: {
    authorization: "Bearer xxx"
  }
})
```

直接粘到 Console 就能重新发送请求。

还可以改成：

```js
fetch("http://localhost:8080/api/list", {
  headers: {
    authorization: "Bearer xxx"
  }
})
  .then(res => res.json())
  .then(data => {
    console.log(data)
  })
```

这非常适合脱离页面代码单独测试接口。

---

# 21. Copy as fetch 的意义

假设：

```text
页面 → axios → interceptor → token → HTTP
```

过程很复杂。

但是你只想确认：

```text
这个接口到底能不能正常请求？
```

Copy as fetch 后直接 Console 执行，相当于把这次网络请求复刻出来。

---

# 22. Network 筛选请求

一个页面可能有几百条资源：

```text
JS
CSS
图片
字体
接口
WebSocket
```

Network 顶部一般可以筛：

```text
Fetch/XHR
JS
CSS
Img
Media
Font
WS
```

调接口时最常使用：

```text
Fetch/XHR
```

这样就不会被静态资源淹没。

---

# 23. Network 搜索

还可以直接搜索接口名称：

```text
user
order
list
query
```

非常适合大型后台管理系统。

---

# 24. Preserve log

Network 有个非常实用的选项：

```text
Preserve log
```

默认情况下页面刷新或跳转后，Network 记录可能被清空。

打开后，跳转页面仍然保留请求。

调登录、OAuth、页面跳转、重定向时尤其有用。

---

# 25. Disable cache

Network 还有：

```text
Disable cache
```

开发环境调试时非常实用。

可以避免浏览器缓存旧 JS、旧静态资源等导致：

```text
“我明明改代码了，怎么页面还是旧的？”
```

注意通常只有 DevTools 打开时才生效。

---

# 26. Network Throttling

可以模拟：

```text
Slow 3G
Fast 3G
Offline
```

用于测试弱网、加载状态、Skeleton、Loading、请求超时。

例如页面开发时网速太快，loading 一闪而过，可以开启 Slow 3G，真正观察 loading 是否合理、按钮是否重复点击、请求期间 UI 是否异常。

---

# 27. Initiator：这个请求是谁发的？

Network 的 `Initiator` 非常实用。

当你看到某个接口，却不知道到底哪段代码发出来的，可以看 Initiator。

通常可以定位到某个 JS 文件、某个方法、调用栈。

对于“为什么接口请求两次？”尤其好用。

---

# 28. Timing：接口慢到底慢在哪里？

Timing 可以查看：

```text
DNS
TCP
SSL
Request sent
Waiting
Content Download
```

其中常关注：

```text
Waiting (TTFB)
```

如果这里非常长，通常意味着服务端响应比较慢。

如果请求本身 5 秒，不要第一反应怪 Vue，先看 Timing。

---

# 29. Elements：调 HTML 和 CSS 的神器

Elements 主要观察：

```text
DOM
CSS
样式计算结果
```

你可以在 Elements 中查看真实 DOM。

---

# 30. HTML 和 DOM 的区别

HTML 是源代码描述，DOM 是浏览器解析 HTML 后建立的对象树。

Elements 看到的是：

> 当前浏览器中的 DOM 状态。

所以 Vue 动态生成的元素，即使原始 `index.html` 没有，也会出现在 Elements。

---

# 31. 检查元素

最常用：

```text
Ctrl + Shift + C
```

然后点页面元素。

DevTools 会自动定位对应 DOM。

特别适合检查：

```text
这个按钮为什么位置不对？
这个 div 为什么宽度这么大？
这个 Element UI 组件到底生成了什么 HTML？
```

---

# 32. Styles

选中元素后可以看到 `Styles`。

你可以直接修改、关闭、新增 CSS。

这些修改只是临时作用于当前浏览器，不会修改源代码。

---

# 33. 为什么 CSS 不生效？

Elements 可以直接告诉你。

如果某个样式被划掉，说明被别的规则覆盖了。

你可以据此理解 CSS 权重、继承、覆盖、`!important` 实际是怎么工作的。

---

# 34. Computed

Elements 里的 `Computed` 展示的是：

> 浏览器最终计算出来的样式。

所以可以理解为：

```text
Styles = CSS 规则来源
Computed = 最终结果
```

---

# 35. Box Model

Elements 中经常可以看到：

```text
margin
border
padding
content
```

这就是 CSS 盒模型：

```text
┌──────── margin ────────┐
│ ┌───── border ───────┐ │
│ │ ┌── padding ─────┐ │ │
│ │ │    content     │ │ │
│ │ └────────────────┘ │ │
│ └────────────────────┘ │
└────────────────────────┘
```

页面出现“怎么多了 20px？”时，看 Box Model 往往一眼能发现。

---

# 36. Elements 可以临时改页面

可以直接修改文字、HTML 或 CSS。

非常适合快速试 UI，而不是不断在 IDE 改、保存、切页面、看效果、再改。

可以先 DevTools 调满意，再回代码修改。

---

# 37. Sources：真正的代码调试

Sources 可以理解为：

> 浏览器中的代码调试器。

你可以：

```text
查看 JS
打断点
单步执行
查看变量
查看调用栈
```

---

# 38. debugger

代码里可以写：

```js
function submit() {
  const data = getData()

  debugger

  sendRequest(data)
}
```

浏览器执行到 `debugger` 会暂停。

然后你可以检查：

```text
data 到底是什么？
当前 this 是谁？
函数参数是什么？
```

---

# 39. Breakpoint

也可以直接在 Sources 点击某一行设置断点。

程序运行到这行时自动暂停。

这对于复杂逻辑非常重要。

---

# 40. 单步调试

暂停后通常有几个按钮：

```text
Resume
Step over
Step into
Step out
```

- Resume：继续执行
- Step over：执行下一行，不进入函数内部
- Step into：进入调用函数内部
- Step out：从当前函数执行出去

---

# 41. Call Stack

Call Stack 就是调用栈。

例如：

```text
click
↓
handleSubmit
↓
validateForm
↓
request
```

你可以看到当前代码为什么会执行到这里。

调复杂业务非常重要。

---

# 42. Console 和断点配合

程序暂停之后，Console 会非常强大。

当前作用域里的变量可以直接在 Console 中访问、计算和验证。

因为程序暂停后，Console 可以进入当前执行上下文。

这比单纯 `console.log()` 强很多。

---

# 43. Application：浏览器存储

Application 主要检查：

```text
Local Storage
Session Storage
Cookies
IndexedDB
Cache
Service Worker
```

前端业务开发最常用：

```text
Local Storage
Session Storage
Cookies
```

---

# 44. LocalStorage

例如：

```js
localStorage.setItem('token', 'abc123')
```

Application → Local Storage → 当前域名，可以直接看到数据。

因此登录问题经常这样排查：

```text
登录成功了吗？
↓
token 保存了吗？
↓
Application 看 token
↓
Network 看请求有没有携带 token
```

---

# 45. SessionStorage

和 LocalStorage 类似。

区别简单理解：

```text
localStorage：
关闭页面之后通常还存在

sessionStorage：
当前标签页会话生命周期
```

---

# 46. Cookies

Application 也可以查看 Cookie，以及：

```text
Domain
Path
Expires
HttpOnly
Secure
SameSite
```

这些对登录认证非常重要。

---

# 47. 登录问题的一套标准排查流程

比如“明明登录成功，进入首页又显示未登录”。

可以按顺序检查：

1. Network → 登录接口，看登录是否成功。
2. Response 看有没有 token。
3. Application 看 token 有没有存。
4. Network → 后续接口 → Request Headers，看有没有 `Authorization: Bearer xxx`。
5. 看接口是否返回 401 / 403。

这样问题基本就被定位了。

---

# 48. Performance：页面为什么卡？

Performance 用来分析：

```text
页面性能
JS 执行
渲染
布局
FPS
长任务
```

例如页面滚动特别卡、点击按钮卡 2 秒、表格加载几千条数据后卡死，可以录制 Performance。

然后看：

```text
Main Thread
Long Task
Rendering
Scripting
```

---

# 49. 前端性能问题基本可以分成几类

```text
网络慢
JS 计算慢
DOM 太多
频繁重绘
频繁重排
资源太大
内存泄漏
```

对应工具：

```text
Network
Performance
Memory
```

---

# 50. Memory：排查内存泄漏

Memory 是比较进阶的工具。

例如打开关闭弹窗 100 次，页面越来越卡，内存越来越高，可能存在：

```text
事件监听未解绑
定时器未清理
DOM 引用残留
大型对象长期被引用
```

可以通过 Heap Snapshot 等工具分析。

初级开发阶段知道它是干什么的即可。

---

# 51. Console 常见日志

最普通：

```js
console.log(data)
```

还可以：

```js
console.warn(data)
console.error(data)
console.table(data)
```

例如：

```js
console.warn('字典匹配失败', item)
```

---

# 52. 不要滥用 console.log

很多新人调 bug 会：

```js
console.log(1)
console.log(2)
console.log(3)
console.log(4)
```

更推荐：

```js
console.log('接口返回结果:', response)
console.log('格式化前的数据:', data)
console.log('匹配失败的数据:', errorData)
```

调试信息应该描述“这个值是什么”。

---

# 53. Console 最适合干什么？

推荐：

```text
临时验证 JavaScript
分析接口数据
验证数据结构
测试 filter/map
检查 window
检查 localStorage
测试 DOM
测试 API
```

不推荐把大量正式业务逻辑都写进 Console。

Console 更像一个实验室 / 调试台。

---

# 54. DevTools 中最重要的思想：不要猜

比如“为什么页面没有数据？”

错误排查方式：

```text
是不是 Vue 有 bug？
是不是后端有 bug？
是不是组件库有问题？
```

正确方式：

```text
Network 有没有请求？
↓
请求成功了吗？
↓
参数是什么？
↓
Response 是什么？
↓
前端有没有拿到？
↓
数据处理之后变成什么？
↓
DOM 有没有渲染？
```

逐层确认。

---

# 55. 一个典型问题应该怎样排查？

假设表格“状态”列应该显示“已完成”，结果显示为空。

## Step 1：Network

看接口：

```json
{
  "status": "3"
}
```

说明后端返回了 `3`。

## Step 2：检查字典

字典：

```js
[
  { value: '1', label: '待处理' },
  { value: '2', label: '处理中' }
]
```

发现没有 `3`，问题定位。

如果字典有：

```js
{ value: 3, label: '已完成' }
```

接口却是：

```js
'3'
```

那么：

```js
3 === '3'
```

结果是：

```js
false
```

这时候问题就是类型不一致，而不是数据不存在。

---

# 56. Console 可以快速验证

直接：

```js
3 === '3'
```

得到：

```js
false
```

然后：

```js
String(3) === String('3')
```

得到：

```js
true
```

这种小问题完全没必要改代码、启动项目、刷新页面再看，Console 十秒就能验证。

---

# 57. 字典问题的推荐 Console 调试方案

接口数组：

```js
const data = ...
```

字典：

```js
const dict = ...
```

先看接口到底有哪些值：

```js
[...new Set(data.map(item => item.status))]
```

再看字典有哪些值：

```js
dict.map(item => item.value)
```

然后找不匹配：

```js
data.filter(item => {
  return !dict.some(dictItem => {
    return String(dictItem.value) === String(item.status)
  })
})
```

如果只想看异常值：

```js
[
  ...new Set(
    data
      .filter(item => {
        return !dict.some(dictItem => {
          return String(dictItem.value) === String(item.status)
        })
      })
      .map(item => item.status)
  )
]
```

可能直接得到：

```js
['8', '99']
```

这比翻几千条 Response 高效得多。

---

# 58. 前端开发中一个非常实用的分层排查模型

以后出现 bug，可以按照：

```text
① 用户操作
↓
② DOM / 事件
↓
③ Vue 业务逻辑
↓
④ 请求参数
↓
⑤ HTTP 请求
↓
⑥ 后端返回
↓
⑦ 前端数据处理
↓
⑧ Vue 响应式更新
↓
⑨ DOM 渲染
↓
⑩ CSS 展示
```

去判断问题发生在哪一层。

DevTools 基本覆盖整个过程。

---

# 59. 不同问题对应哪个 DevTools 工具？

- 接口没数据：`Network`
- 请求参数不对：`Network → Payload`
- 后端返回不对：`Network → Response`
- 请求为什么重复：`Network → Initiator`、`Sources → breakpoint`
- 变量为什么变成 undefined：`Console`、`Sources`、`debugger`
- CSS 不生效：`Elements → Styles`
- 元素位置不对：`Elements`、`Computed`、`Box Model`
- token 不对：`Application`、`Network → Request Headers`
- 页面特别慢：`Network`、`Performance`
- 页面越来越卡：`Memory`、`Performance`

---

# 60. 一个值得养成的习惯：先观察，后改代码

很多新人：

```text
看到 bug
↓
马上改代码
↓
试一下
↓
没好
↓
继续乱改
```

这是效率比较低的。

更合理的是：

```text
看到 bug
↓
DevTools 获取证据
↓
确定是哪一层
↓
确定具体原因
↓
再修改代码
```

例如表格没数据，不要立刻到处修改 `this.tableData`。

先看 `Network Response`。

如果 Response 本来就是 `[]`，那前端怎么改都不可能凭空变出数据。

---

# 61. DevTools 不是“调 bug 才打开”

它其实应该是前端开发日常的一部分。

例如你写完一个功能，可以顺手检查：

```text
Network：
有没有重复请求？

Payload：
参数是不是预期格式？

Response：
后端数据结构是什么？

Console：
有没有 error / warning？

Elements：
DOM 是否合理？

Application：
有没有乱存东西？
```

DevTools 不只是“出问题 → 修问题”，还是理解自己的程序到底怎样运行。

---

# 62. 初级前端最值得掌握的 DevTools 能力

不需要一开始就研究非常高级的 Performance 分析。

## 第一阶段

熟练：

```text
Network 找接口
看 Headers
看 Payload
看 Response
看状态码
```

## 第二阶段

能够：

```text
Console 操作数组
filter
map
find
some
Set
console.table
```

## 第三阶段

能够：

```text
Elements 调 CSS
查看 DOM
查看盒模型
检查 CSS 覆盖关系
```

## 第四阶段

掌握：

```text
debugger
breakpoint
Scope
Call Stack
```

## 第五阶段

开始掌握：

```text
Application
Performance
Memory
```

这个顺序比从头把 Chrome DevTools 文档全部读一遍有效得多。

---

# 63. 为什么建议在实际业务里学 DevTools？

因为 DevTools 本身就是一个高度实践型工具。

比如这次真实需求：

> 接口数据很多，不确定字典是否全部能够匹配。

这里就同时涉及：

```text
Network
HTTP Response
Console
JS 数组
filter
some
Set
作用域
```

通过一个真实问题，就把这些概念串起来了。

这比单独学习“今天背 filter、明天背 Network、后天背 HTTP”更容易真正理解。

---

# 64. DevTools 学习心得：把浏览器当成“正在运行的程序”

这是理解 DevTools 最重要的一点。

IDE 中看到的是：

```text
程序应该怎么运行
```

DevTools 中看到的是：

```text
程序实际上怎么运行
```

两者是不同视角。

例如你的代码写：

```js
status: 1
```

不代表运行时一定是 `1`。

它可能经过接口之后变成 `'1'`、`null`、`undefined`。

所以调试应该相信：

```text
运行时证据
```

而不是仅仅相信：

```text
“我的代码看起来应该没问题。”
```

---

# 65. DevTools 学习心得：Console 是一个实验室

Console 不应该只理解成“打印 `console.log` 的地方”。

它还是一个 JavaScript 实验室。

例如不知道 `filter` 怎么写，直接：

```js
[1, 2, 3].filter(item => item > 1)
```

不知道 `Set` 怎么去重，直接：

```js
[...new Set([1, 1, 2, 2, 3])]
```

这种“小实验”对于学习 JavaScript 非常高效。

---

# 66. DevTools 学习心得：Network 是前后端之间的真相层

做全栈项目时特别值得记住：

```text
前端说：我发了。
后端说：我没收到。
```

不要争，看 Network。

```text
前端说：后端返回错了。
后端说：我返回是对的。
```

不要争，看 Network Response。

```text
后端说：你参数传错了。
前端说：我代码明明写对了。
```

不要争，看 Payload。

所以 Network 很大程度上就是：

> 前后端联调时的客观证据。

---

# 67. DevTools 学习心得：尽量缩小问题范围

好的调试不是一上来就“找到 bug”，而是不断把问题范围缩小。

例如：

```text
页面有问题
↓
数据展示有问题
↓
status 列有问题
↓
接口返回 status = 99
↓
字典没有 99
```

到这里问题已经非常具体。

而 DevTools 的最大价值，就是帮助你不断排除可能性。

---

# 68. 一个成熟一些的排障思路

以后遇到前端问题，可以先问自己五个问题：

```text
1. 用户到底做了什么？
2. 浏览器到底发了什么？
3. 服务端到底回了什么？
4. JavaScript 到底处理成了什么？
5. DOM 最终到底渲染了什么？
```

分别对应：

```text
事件
Network
Network
Console / Sources
Elements
```

基本涵盖绝大部分前端业务问题。

---

# 69. 一个非常适合日常工作的 DevTools 排查模板

以后出现 bug，可以直接按这个顺序走：

```text
① Console
有没有报错？

② Network
有没有对应请求？

③ Headers
URL、Method、Status 是否正确？

④ Payload
请求参数是否正确？

⑤ Response
返回数据是否正确？

⑥ Console / debugger
JS 处理结果是否正确？

⑦ Elements
DOM 是否正确生成？

⑧ Styles
是不是 CSS 导致看不见？

⑨ Application
如果涉及登录，再检查 token / Cookie。

⑩ Performance
如果功能正确但很卡，再查性能。
```

不要每次从头到尾机械执行，而是根据问题选择对应层级。

---

# 70. 最终应该形成的能力

真正熟悉 DevTools 后，你面对：

```text
“这个功能怎么不对？”
```

脑子里不应该只是：

```text
我去翻代码看看。
```

而应该自动形成：

```text
这是 UI 问题？
↓
数据问题？
↓
请求问题？
↓
后端问题？
↓
状态问题？
↓
类型问题？
```

然后选择对应工具：

```text
接口 → Network
数据 → Console
业务执行 → Sources
DOM/CSS → Elements
缓存/token → Application
卡顿 → Performance
内存 → Memory
```

这时候 DevTools 就不再是几个工具面板，而真正变成了一套观察和定位 Web 应用运行问题的方法。

---


# 71. Debug 实战复盘：页面一直 Loading，但 Network 根本没有请求

这是一次实习项目里很“哭笑不得”，但也很值得记住的 Debug 经历。

## 71.1 项目里的接口封装方式

项目中不会在 Vue 组件里直接写 axios，而是先对 axios 做统一封装。

大致调用链是：

~~~text
Vue 组件
↓
导入 api 目录下暴露的方法
↓
api/对应业务模块.js
↓
request(...)
↓
axios
↓
请求拦截器 / 响应拦截器
↓
HTTP 请求
~~~

例如供应商接口：

~~~js
import request from "@/utils/request"

// 获取全部供应商
export function getSupplierList(data) {
  return request({
    url: '/v1/supplier/list',
    method: 'post',
    data
  })
}
~~~

其中 **request** 是项目底层基于 axios 封装的请求方法，统一处理请求、响应拦截等逻辑。

组件中再导入：

~~~js
import { getSupplierList } from '@/api/supplier'
~~~

然后在业务方法里组装参数并调用：

~~~js
const params = {
  // ...
}

getSupplierList(params)
~~~

---

## 71.2 当时的现象

当天下午写一个功能时，流程大概是：

~~~text
组装请求参数
↓
调用 api 方法
↓
等待接口结果
~~~

结果页面一直处于 Loading / 转圈状态。

打开浏览器 DevTools 后发现一个非常关键的现象：

> Network 里根本没有出现对应的接口请求。

当时其实已经隐约意识到：

~~~text
可能是“调用接口的方法”这里出了问题
~~~

但脑子还是顺着业务代码往下钻，误以为：

~~~text
是不是参数组装过程卡住了？
是不是某段参数处理逻辑有问题？
~~~

于是排查重点放错了地方。

---

## 71.3 真正的原因

最后发现并不是参数的问题。

真正的问题非常简单：

> api 模块中实际 export 的方法名，和组件中导入 / 调用的方法名没有对应上。

也就是说，请求调用链实际上连 **request(...)** 这一层都没有正常走到。

因此浏览器自然不会产生 HTTP 请求，Network 中也就什么都看不到。

问题实际发生的位置是：

~~~text
组件业务方法
↓
API 方法引用 / 调用   ← 问题在这里
↓
request
↓
axios
↓
HTTP
~~~

而我当时却在重点检查：

~~~text
请求参数
~~~

相当于还没真正“发车”，就在研究车里装的货有没有问题。

---

## 71.4 这次 Debug 最大的教训

以后看到：

~~~text
页面一直 Loading
+
Network 没有任何对应请求
~~~

第一反应不应该是：

~~~text
后端是不是挂了？
接口参数是不是错了？
request 拦截器是不是有问题？
~~~

因为如果 Network 中连请求都没有，说明问题很可能发生在 HTTP 请求产生之前。

应该优先排查：

~~~text
① 点击事件 / 生命周期有没有真正触发？
↓
② 业务方法有没有执行？
↓
③ 参数组装代码有没有执行完成？
↓
④ API 方法是否正确 import？
↓
⑤ import 的方法名和 export 的方法名是否一致？
↓
⑥ 调用的是否真的是预期 API 方法？
↓
⑦ 代码是否执行到了 request(...)？
↓
⑧ 最后才进入 axios / 拦截器 / HTTP 请求
~~~

核心判断是：

> **Network 没请求，就先查“为什么代码没有走到发请求这一步”，而不是先查请求发出去之后会发生什么。**

---

## 71.5 一个更成熟的排查方法

以后遇到类似代码：

~~~js
async function submit() {
  const params = buildParams()

  await getSupplierList(params)
}
~~~

页面卡住，同时 Network 没请求，可以快速加几个断点或日志：

~~~js
async function submit() {
  console.log('1. submit 已进入')

  const params = buildParams()
  console.log('2. 参数组装完成', params)

  console.log('3. 准备调用 API', getSupplierList)

  const res = await getSupplierList(params)

  console.log('4. API 返回', res)
}
~~~

或者直接使用 Sources 打断点。

如果只看到：

~~~text
1
2
~~~

却没有继续执行，就说明问题范围已经被压缩到了 API 调用附近。

如果 Network 还是没有请求，就继续向调用链下钻：

~~~text
组件
↓
api 方法
↓
request
↓
axios
~~~

而不是直接跳到：

~~~text
Payload
Response
后端接口
数据库
~~~

---

## 71.6 这次经历补充了一个很重要的分层思想

以前容易把：

~~~text
“接口有问题”
~~~

理解成一个整体。

实际上至少可以拆成：

~~~text
调用业务方法
↓
调用 API 方法
↓
进入 request 封装
↓
axios 创建请求
↓
请求拦截器
↓
浏览器发 HTTP
↓
后端收到请求
↓
后端返回
↓
响应拦截器
↓
组件拿到结果
~~~

而 Network 只会从：

~~~text
浏览器真正开始进行网络通信
~~~

之后给你证据。

所以：

~~~text
Network 有请求
→ 可以继续查 URL / Method / Payload / Response / 后端

Network 没请求
→ 应该向前查 JS 调用链
~~~

这两个排查方向要明确区分。

---

## 71.7 给自己记一句

> **没有请求，就不要先调“请求内容”；先确认请求代码到底有没有被执行。**

这次问题虽然只是一个 API 方法名没有对上，但它提醒我：Debug 最容易浪费时间的地方，往往不是不知道某个技术，而是已经看到了关键证据，却没有顺着证据及时调整排查方向。

---

# 总结

对于前端开发来说，DevTools 最核心的价值可以总结成一句话：

> IDE 帮你写程序，DevTools 帮你理解程序实际上是怎么运行的。

最值得优先掌握的是：

```text
Network
    → 请求到底发生了什么

Console
    → JavaScript 和数据到底是什么

Elements
    → 页面最终到底变成了什么

Sources
    → 代码到底是怎么一步步执行的

Application
    → 浏览器到底保存了什么
```

而日常排障最值得形成的习惯是：

```text
不要猜
↓
观察运行时
↓
获取证据
↓
缩小范围
↓
定位问题
↓
最后再修改代码
```

例如这次“接口数据与字典翻译不匹配”的问题，一个比较成熟的思路就是：

```text
Network
↓
确认接口真实 Response

Console
↓
把数据拿出来

map + Set
↓
观察接口到底有哪些状态值

filter + some
↓
筛选字典无法匹配的数据

检查 number / string
↓
排除类型问题

最后回到代码
↓
决定应该修接口、修字典还是修 formatter
```

这其实已经是一套非常典型的前端调试流程了。
