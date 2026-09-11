---
title: "novaops连接服务器数据库"
date: 2026-08-25
section: "Projects"
source: http://120.77.152.123:8088/posts/novaops%e8%bf%9e%e6%8e%a5%e6%9c%8d%e5%8a%a1%e5%99%a8%e6%95%b0%e6%8d%ae%e5%ba%93
tags:
  - "Projects"
  - "博客迁移"
---
本地spring boot配置一份application.local.yml

后端启动时连接本地空闲的端口3307（MySQL，Qdrant在6333），然后在Mobaxterm上对3307和6333端口都开通一条ssh隧道，连接到服务器的对应端口

服务器的端口通过docker配置了服务器3307->(转发到)容器内监听的3306端口，6633 -> 6633、6634 -> 6634。

注：容器通常会获得的是一个自己的网络环境和 IP 地址；至于有没有端口，要看容器里面运行的程序有没有监听端口。如一个容器获得 172.18.0.2，里面的进程是MySQL，监听3306端口，这里的 3306 不是 Docker 分配给容器的，而是 MySQL 自己监听的。

注意配置几个yml读取的环境变量：

```
MySQL相关：
1. NOVAOPS_DB_PASSWORD
2. NOVAOPS_DB_URL
   NOVAOPS_DB_USERNAME
3. NOVAOPS_JWT_TOKEN

Qdrant向量库相关
4. QDRANT_BASE_URL
5. QDRANT_COLLECTION

LLM API相关
6. SILICONFLOW_API_KEY

Spring启用配置文件相关
7. SPRING_PROFILES_ACTIVE
```
