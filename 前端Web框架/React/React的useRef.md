---
title: "React的useRef"
date: 2026-07-27
section: "React"
source: http://120.77.152.123:8088/posts/useRef
tags:
  - "React"
  - "博客迁移"
---
useState：数据驱动：展示业务数据，用于value能被useState的State所管理的原生DOM元素。

useRef：一些DOM元素无法被State管理，需要直接拿到原生DOM进行后续操作，通过Ref抓取。

JS变量、State、Ref的对比：

| 存储方式 | 页面重新渲染后，值还在吗？ | 修改它的值，会触发页面重新渲染吗？ | 典型用途 |
| --- | --- | --- | --- |
| **普通 JS 变量** `let a = 0` | ❌ **不在** （组件函数重新执行，被重置为 0） | ❌ **不会** | 临时的计算中间值 |
| **State** `const [val, setVal] = useState()` | ✅ **在** （永久保留直到卸载） | ✅ **会！** （调用 `setVal` 会让组件重新渲染页面） | 驱动页面展示的业务数据 （输入框内容、弹窗开关等） |
| **Ref** `const val = useRef()` | ✅ **在** （永久保留直到卸载） | ❌ **不会！** （修改 `ref.current = 新值` 是静默的） | • 抓取 DOM 元素 • 存定时器 ID (Timer) • 记下某个不需要显示在页面上的临时状态 |
