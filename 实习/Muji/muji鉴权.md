机制：**基于 Axios 拦截器 + localStorage + HTTP 请求头（Headers）的无感携带与刷新机制**。

一、签发
1.  用户登录时，前端对密码进行 RSA 加密后调用登录接口。
2. 后端验证成功后，在响应中签发返回 `res.data.token`。
3. 前端拿到 token 后，调用 `setToken(res.data.token)`
不放Vuex：Vuex内存存储，F5刷新会丢失，不能持久化。

二、请求携带：
Axios请求拦截器：放在请求头中（无论业务接口是 `GET`、`POST` 还是 `DELETE`）

效果：后端网关/拦截器通过读取 Header 中的 `muji-rfid-ui-token` 鉴权，不需要每个接口都在请求体 JSON 里加一个 token 字段，干净解耦。

三、 Token 过期与无感自动刷新
1. response响应拦截器，某个请求返回 `code === -6`（`TOKEN_EXPIRED`，Token 已过期）时；前端会自动调用 `service.refreshToken()`（请求 `/v1/auth/refresh-token` 接口获取新 Token 并更新本地 `localStorage`）；
2. 利用reponse包装对象的config属性进行重发请求：`.request(response.config)`，`response.config` 记录了原请求的一切
	知识点：
	- Axios 在发起每一个请求时，都会把请求的方法、URL、携带的参数、Headers 全部保存在当前的 `config` 对象中。当请求失败进入响应拦截器时，这个配置会原封不动地挂在 `response.config` 上。
	- 重新调 `service.request` 会**再次完整走一遍 Axios 请求拦截器**

四、页面无感：
响应拦截器返回(重发请求)：
```
return service.request(response.config);
```
- 一个 Promise。原本调用的业务代码（比如 `pageDefectiveProduct().then(res => ...)`）一直处于挂起状态。
- 等重发请求成功后，**这个重发的成功结果会直接流转到原业务调用的 `.then` 里**！
- 页面代码根本不知道中途其实经历了：“失败 -> 刷新token -> 重新发送 -> 成功拿到数据”的整个过程。

### ！！两次请求间需要 `remainConfig = true`，免检标志。
前端向后端接口发送表单查询对象时，会将这个表单对象**序列化成 URL 键值对字符串**：
如：
`const data = { name: '张三', age: 18 }`

```
QS.stringify(data)

// 转换后的结果是字符串： "name=%E5%BC%A0%E4%B8%89&age=18"
```

若此时因token过期需refresh，那么拿到新token再请求时，**注意这里的查询表单对象已经是URL键值对字符串**，如果不拦截，请求拦截器会再次将其Stringfy**序列化成 URL 键值对字符串**。
	会将该URL**字符串**进行 `{ ... "字符串" }` 解构，然后在 JavaScript 里会把字符串拆成按索引排列的对象：

```js
// 解构一个字符串的结果：

{ 0: 'n', 1: 'a', 2: 'm', 3: 'e', 4: '=', 5: '%', ... }

然后再用 `QS.stringify` 一转，最终发给后端的请求体就变成了：


0=n&1=a&2=m&3=e&4=%3D&5=%25...（完全变成了面目全非的乱码！）
```

后端收到这样一堆由字符索引组成的参数，根本读不出 `name` 和 `age`，请求直接报 `400 Bad Request` 或系统异常！

因此加入`remainConfig`免检标志，就会直接使用最初转换好的字符串发送请求，而跳过请求拦截器的再一次`QS.stringify`