---
title: "Codex写入代码文件出现乱码"
date: 2026-07-09
section: "Debug"
source: http://120.77.152.123:8088/posts/codex%e4%b9%b1%e7%a0%81%e9%97%ae%e9%a2%98
tags:
  - "Debug"
  - "博客迁移"
---
建博客站时发现codex编程时会频发乱码现象，会把中文编译成火星文。

大概是codex（UTF-8格式）编写代码，然后例如终端写入文件时，终端的解码方式可能是GBK或者其他的与UTF-8不一致的编码格式，就导致写入的代码文件中中文解码异常。

目前尝试的解决方法：下载PowerShell 7，并设置为默认终端。
