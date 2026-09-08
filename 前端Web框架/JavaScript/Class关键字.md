# constructor 和 this.name = name 笔记

constructor 可以粗略理解成 Java 里的构造方法，用来在创建对象时做初始化。

但js和java的对象模型不一致，js中是通过constructor动态给对象新增属性，而java强制要求先定义对象属性，再用构造函数赋值。

`class Person { constructor(name) { this.name = name } }`

## constructor 的作用

- 创建对象时自动执行
- 接收传入的参数
- 用这些参数给当前对象初始化属性

## this.name = name 怎么理解

这句可以理解为：

**在当前对象上创建或设置一个 name 属性，并把传入的参数 name 赋给它。**

这里左右两边的 name 含义不同：

- this.name：对象的属性
- name：构造器接收的参数

所以它的本质是：

**把构造器参数保存到当前对象的属性上。**

## 要注意的一点

构造器里的参数不会自动变成对象属性。

只有写了：

`this.name = name`

这个参数才真正绑定到对象上。

## 一句话总结

**JS 中的 constructor 用来初始化对象，this.xxx = 参数 表示给当前对象动态添加或设置属性，并把参数值保存进去。**