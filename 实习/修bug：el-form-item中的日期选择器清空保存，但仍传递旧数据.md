---
title: "修bug：el-form-item中的日期选择器清空保存，但仍传递旧数据"
date: 2026-08-05
section: "Internship"
source: http://120.77.152.123:8088/posts/%e4%bf%aebuggggg
tags:
  - "Internship"
  - "博客迁移"
---
通过`$refs`，拿到对应el-form-item的dom，然后调用`.validate`进行字段校验。在传入的回调函数的参数中拿到校验结果，分别写校验成功、失败的逻辑。若成功则把已有表单数据对象`Object.assign`复制到`param`新数据对象,再对每个字段做处理，处理到日期选择器时的数组变量（存储前后两个日期）时，注意要存在且有数组长度为2，才赋值给`param`这个请求接口的表单数据，否则记得加上`else`，让表单里的两个日期参数为空。
