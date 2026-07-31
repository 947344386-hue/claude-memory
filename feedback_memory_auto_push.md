---
name: memory-auto-push
description: memory 目录变更后自动推 GitHub，保证跨主机传承
metadata: 
  node_type: memory
  type: feedback
  originSessionId: f404e089-1958-4eb9-ba91-c8dccd2e36b7
  modified: 2026-07-31T06:47:02.409Z
---

**重大操作习惯或新开发合作流**写入 memory 后，commit 并 push 到 [947344386-hue/claude-memory](https://github.com/947344386-hue/claude-memory)。日常小修小补不推，攒到有意义的变更再推。

**Why:** 用户在公司和家里两台主机之间切换，memory 需保持同步。但频繁推送噪音大，只有新的工作流约定、模式沉淀、跨项目可复用的经验才值得跨主机传承。

**什么该推：** 新的编译/备份/UI 模式、Git 规范约定、开发合作流变更、跨项目可复用的参考条目
**什么不该推：** 日常小修正、临时笔记、路径更新、单次任务的细节记录

**How to apply:**
- 写入有意义的 memory 后，在同一个会话里执行 `git add` + `git commit` + `git push`
- commit author 用 `251538000+947344386-hue@users.noreply.github.com`（避免 QQ 邮箱被 GH007 拦）
- 仓库路径：`C:\Users\mengwenbo\.claude\projects\F--ClaudeLibrary\memory\`

**新主机首次 setup：**
```bash
git clone https://github.com/947344386-hue/claude-memory.git "C:/Users/<用户名>/.claude/projects/<项目名>/memory"
```

**注意：** memory 中部分条目包含硬编码的绝对路径（如 `F:\UELibrary\KimmelRebirth`、`F:\ClaudeLibrary`），换主机后需按实际盘符/目录调整。但工作流约定（编译命令格式、Git 规范、UI 模式、备份策略等）是跨主机通用的，换机后只需更新路径引用即可继续生效。