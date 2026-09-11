---
title: "BFF中间层"
date: 2026-07-31
section: "Frontend"
source: http://120.77.152.123:8088/posts/Backend-for-Frontend
tags:
  - "Frontend"
  - "博客迁移"
---
BFF 简单理解为：

一个懂前端页面需求、负责帮前端调用和整理后端数据的**中间层**。

注重于帮前端整合所需资源，即聚合调用接口，返回前端所需数据格式，具体后端业务逻辑不是其关注的点。

*需要和API网关做区分*： API网关更偏向基础设施，而BFF更偏向前端的业务需求。

`API Gateway`:

```
认证
限流
路由转发
日志
负载均衡
黑白名单
```

但`API Gateway 和 BFF`也可以同时用在一个架构里：

```
前端
 ↓
API Gateway
 ↓
BFF
 ↓
各个微服务
```

小型项目中，BFF 和 API 网关也可能合并到一个服务里, 而不作明确区分。
