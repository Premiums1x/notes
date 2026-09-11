---
title: "通过配置对象生成UI，需要套一层”拦截器“"
date: 2026-08-06
section: "Internship"
source: http://120.77.152.123:8088/posts/%e5%ad%90%e7%bb%84%e4%bb%b6computed%e8%ae%a1%e7%ae%97%e5%b1%9e%e6%80%a7%e5%bd%93%e2%80%9d%e6%8b%a6%e6%88%aa%e5%99%a8%e2%80%9c
tags:
  - "Internship"
  - "博客迁移"
---
使用Schema表单时，通常不是直接遍历用户传入的配置表单对象。

因为难免存在用户配置不到位情况：如在配置的时候，忘了给某个输入框声明初始的 value 字段：这会导致用户在页面上输入内容时报错或者无法响应。

而这就需要在遍历表单生成html模板前做一层”输入防范“， 用计算属性造一个 `formConfig` 的“拦截器/代理”。

它把传进来的 `formData` 遍历了一遍，一旦发现某个表单项没有自带 value 属性，它就贴心地用 `this.$set(temp[key], 'value', '')` 帮你补上一个默认的空值 ''。

总结： `:form-data` 就是传给 `props: ['formData']` 的。而 formConfig 只是 EastForm 内部为了容错和补全缺失状态，对 formData 进行的一层安全包装。最后真正在页面上循环生成的，是这层安全的 formConfig。
