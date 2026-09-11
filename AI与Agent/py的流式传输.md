---
title: "py的流式传输"
date: 2026-07-22
section: "Agent"
source: http://120.77.152.123:8088/posts/Agent-stream
tags:
  - "Agent"
  - "博客迁移"
---
`yield chunk` 表示把当前这一小段内容“产出给调用这个函数的人”，然后暂停函数；等对方继续读取时，再回来处理下一个 chunk。 这会让包含它的函数变成一个生成器函数。 假设客户端拿yield，直接使用一段yield是没用的，要在循环中不断拿yield。
