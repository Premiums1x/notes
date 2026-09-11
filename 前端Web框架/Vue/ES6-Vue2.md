---
title: "ES6-Vue2"
date: 2026-08-10
section: "Internship"
source: http://120.77.152.123:8088/posts/template&script-ES6-With-Vue2
tags:
  - "Internship"
  - "博客迁移"
---
ES 新语法：

- `??` :空值合并：a ?? b 仅在 a 是 null/undefined 时取 b。和 || 不同，0、''、false 不会触发 ??。
- `?.`:可选链：a?.b 在 a 为 null/undefined 时直接返回 undefined，不会报错。例如 options.find(...)?.name，找不到就返回 undefined 而不是抛错。

这两个新语法写在`Template`和`script`里大不相同：

1. 写在 `<script>` 里（完全安全，不会出错）:

- `<script>` 里的代码会直接交给 `Babel`（JS 编译器）处理。你的项目配置了相应的 Babel 插件（@babel/plugin-proposal-optional-chaining），Babel 认识这些新语法，并在打包时完美地把它们转换成了所有浏览器都能看懂的、带 if 判断的兼容性代码（老语法）。所以运行极其稳定。

2. 写在 `<template>` 里（非常危险，容易导致白屏打不开） Vue 2 中，`<template>`并不是直接交给 Babel 处理的，而是交给`vue-template-compiler`这个模板编译器处理。

- 生成了带有`with(this)` 的怪异代码：模板编译器会把你的 HTML 解析成一段带有`with(this) { ... }`的原生 JavaScript 渲染函数。
- 漏网之鱼（逃过了 Babel 的转换）：在很多 `Vue CLI` 的`Webpack` 配置中，为了性能，这段由模板生成的渲染函数代码没有完整经过`Babel`的二次深度转译（或者`Babel` 在面对 `with(this)` 作用域时无法正确转换新语法），导致原生的 `??` 和`?.`字符被原封不动地打包进了最终代码里。
- 浏览器直接白屏/点不开：当用户的浏览器（尤其是稍微老一点的 Chrome，或微信/钉钉内置的旧版浏览器内核）执行到这个组件时，它根本不认识原生的`??`。JS 引擎会直接抛出 `SyntaxError: Unexpected token '?'`语法致命错误。此时 Vue 组件的渲染流程瞬间中断，整个组件直接表现出来的就是页面空白，或者点击 Tab 没有任何反应（因为里面的组件已经没法正常渲染）。

总结：

Vue 2 环境下， 新语法（?. 和 ??）可在 `<script>` 里面用，但绝对不要出现在 `<template> 的双大括号 {{ }}` 或者`v-if / v-show` 里面。

如果在模板里确实需要这种逻辑判断，最好用老式的 || 或者是直接在 `<script>` 里写一个 computed 计算属性，把算好的结果再丢给模板去渲染，这样就能避免这种诡异的白屏问题。
