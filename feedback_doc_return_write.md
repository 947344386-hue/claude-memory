---
name: doc-return-write
description: 赌石打包开发必须回写 canonical guide 的规则
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 5aea570f-c1c6-4807-9d93-9f191d52749a
  modified: 2026-08-11T17:17:36.363Z
---

每次赌石 Demo 打包相关开发任务结束后，必须回写 `F:\ClaudeLibrary\KimmelRebirth\StoneBetting\赌石玩法开发引导.md` 的以下部分：
- **状态表（第2节）**：更新模块的代码/资产/验证状态
- **变更记录（第9节）**：按模板记录目标、入口类/函数、资产、备份位置、Build.bat 结果、端到端验证、已知偏差/后续
- **当前待确认项（第10节）**：增删待确认条目

**Why:** 打包是多 Phase 长期目标，跨多个 session，必须靠 canonical guide 追踪进度和事实。
**How to apply:** 每个 Phase 完成后，在 guide 末尾新增一条变更记录，并更新状态表中相应模块的行。