---
name: git-push-proxy
description: KimmelRebirth 推 GitHub 直连 443 失败，须走本机 127.0.0.1:10808 代理（一次性 -c 参数，不改全局配置）
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 93003e61-3cc7-4394-b305-34905ac69b21
  modified: 2026-08-13T09:10:42.820Z
---

推 GitHub（github.com 443）在本机直连会失败（`Connection was reset` / `Could not connect to server`）。实测可用代理端口 **10808**（curl 隧道 200），其他常见端口（7890/7897/10809/1080/2080/33210）不可用或响应异常。

**How to apply:** 推送时用一次性参数，不要写进 git config：

```
git -c http.proxy=http://127.0.0.1:10808 push origin main
```

**Why:** 2026-08-13 全量大推时直连两次失败，端口探测后 10808 成功。一次性 `-c` 不污染仓库/全局配置，换网络环境后直连仍可工作。

**注意:** 若以后网络环境变化（换代理软件/直连恢复），重新探测：`curl -sI --connect-timeout 3 -x http://127.0.0.1:<port> https://github.com`，认 `HTTP/1.1 200 Connection established`。
