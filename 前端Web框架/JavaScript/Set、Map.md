Set 和 Map 都是 **ES6（ECMAScript 2015）** 新引入的数据结构。

可以简单记：
- Set：类似“值的集合”，**成员不能重复**
- Map：类似“键值对集合”，比普通对象更适合做映射关系

## Set

`const s = new Set([1, 2, 2, 3]) console.log(s) // {1, 2, 3}`

特点：

- 自动去重
- 常用于数组去重

## Map

`const m = new Map() m.set("name", "Tom") console.log(m.get("name")) // Tom`

特点：

- 存键值对
- 键不一定非得是字符串，**任何值都可以当键**

比如：

`const m = new Map() const obj = {} m.set(obj, "hello")`

## 和以前的对象/数组区别

- Array：适合按顺序存一组值
- Object：适合存以字符串为键的属性
- Set：适合存不重复的值
- Map：适合存真正的键值映射

## 一句话总结

**是的，Set 和 Map 都是 ES6 新增的数据结构：Set 用来存不重复的值，Map 用来存键值对。**