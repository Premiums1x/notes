```
git config --global http.https://github.com.proxy http://127.0.0.1:7890
```

底层转换为：
```
[http "https://github.com"]

proxy = http://127.0.0.1:7890
```
1. ** `http.`**：
    - 这是 Git 内部的**网络模块名称**。在 Git 的规范中，不论是 HTTP 还是 HTTPS 传输，底层配置项都是统一以 `http.` 开头的。
2. **中间的 `https://github.com`**：
    - 这是**网址匹配规则（过滤器）**。告诉 Git：“**只有在访问以 `https://github.com` 开头的仓库时**，这条规则才生效”。
3. **后面的 `.proxy`**：
    - 代表**代理选项**。
4. **最后的 `http://127.0.0.1:7890`**：
    - 代理服务器的地址（`127.0.0.1` 代表本机，`7890` 是你本地代理软件监听的端口）。