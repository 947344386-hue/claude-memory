---
name: memory-auto-push
description: memory 目录变更后自动推 GitHub，保证跨主机传承
metadata: 
  node_type: memory
  type: feedback
  originSessionId: f404e089-1958-4eb9-ba91-c8dccd2e36b7
  modified: 2026-07-31T06:24:50.383Z
---

每次写入或修改 memory 文件后，自动 commit 并 push 到 [947344386-hue/claude-memory](https://github.com/947344386-hue/claude-memory)。

**Why:** 用户在公司和家里两台主机之间切换，memory 需保持同步。仓库根目录即 memory 目录本身，clone 后直接可用。

**How to apply:**
- 写入/更新任何 memory 文件后，在同一个会话里执行 `git add` + `git commit` + `git push`
- commit author 用 `251538000+947344386-hue@users.noreply.github.com`（避免 QQ 邮箱被 GH007 拦）
- 仓库路径：`C:\Users\mengwenbo\.claude\projects\F--ClaudeLibrary\memory\`

**新主机首次 setup：**
```bash
git clone https://github.com/947344386-hue/claude-memory.git "C:/Users/<用户名>/.claude/projects/<项目名>/memory"
```

**注意：** memory 中部分条目包含硬编码的绝对路径（如 `F:\UELibrary\KimmelRebirth`、`F:\ClaudeLibrary`），换主机后需按实际盘符/目录调整。但工作流约定（编译命令格式、Git 规范、UI 模式、备份策略等）是跨主机通用的，换机后只需更新路径引用即可继续生效。