---
title: "Codex配置走代理：避免Reconnecting"
date: 2026-08-04
section: "Debug"
source: http://120.77.152.123:8088/posts/Codex%e9%85%8d%e7%bd%ae
tags:
  - "Debug"
  - "博客迁移"
---
1. 在`.codex`目录下新建一个`.env`文件（虚拟环境配置文件，全局生效），配置Codex要走的代理端口：

```
HTTP_PROXY=http://127.0.0.1:代理端口
HTTPS_PROXY=http://127.0.0.1:代理端口
ALL_PROXY=socks5://127.0.0.1:代理端口
http_proxy=http://127.0.0.1:代理端口
https_proxy=http://127.0.0.1:代理端口
all_proxy=http://127.0.0.1:代理端口
```

然后在`config.toml`文件末尾追加：

```
[shell_environment_policy]
include_only = ["PATH","path","HOME","USERPROFILE","TEMP","TMP","HTTP_PROXY","HTTPS_PROXY","ALL_PROXY","http_proxy","https_proxy","all_proxy"]
```
