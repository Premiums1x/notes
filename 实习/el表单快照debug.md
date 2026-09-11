---
title: "el表单快照debug"
date: 2026-08-19
section: "Debug"
source: http://120.77.152.123:8088/posts/el-debug
tags:
  - "Debug"
  - "博客迁移"
---
- 对日期选择器七天的选择范围：

watcher里写newVal、oldVal时，如果第一次进入就选择超出范围的时间，那么会拿默认值当oldVal。

若watcher中还写了符合某个条件时设置oldVal的逻辑，那么oldVal就会和默认值冲突。

- 拿到Form的ref直接调用组件的set方法，会导致文本DOM和日期选择器DOM的重置产生冲突。组件set方法遍历filedprop设置初始值。若日期选择器有tab栏做区分：不同条件下有不同的日期选择器规则，那么默认进入第一个tab栏时，会拿到当时的日期选择器初始值做initialData，若重置的事件回调仅调用组件的set方法，那么其他tab栏重置的值也会是这个initialData。
