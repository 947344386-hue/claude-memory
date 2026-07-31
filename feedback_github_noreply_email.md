---
name: github-noreply-email
description: 推 KimmelRebirth 到 GitHub 前提交 author 必须用 noreply 邮箱，QQ 邮箱会被 GH007 拦截
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 29a9c2fc-f773-4d2a-8030-3b8d7451aa80
  modified: 2026-07-29T11:46:17.443Z
---

GitHub 账号 `947344386-hue`（数字 ID `251538000`）开了"Block command line pushes that expose my email"。git 全局配置的 `94744386@qq.com`（实际 `947344386@qq.com`）是注册为隐私的已验证邮箱，每次 `git push` 都会被远端以 **GH007** 拒绝（"Your push would publish a private email address"）。

**Why:** GitHub 邮箱隐私保护拦截 author 为账号私有邮箱的提交；noreply 邮箱不触发该保护且仍归属到账号。

**How to apply:** 推送 KimmelRebirth 前，确保提交 author 用 noreply 邮箱：
```
251538000+947344386-hue@users.noreply.github.com
```
- 已 commit 但 author 是 QQ 邮箱：`git config user.email "251538000+947344386-hue@users.noreply.github.com"`（本仓库 local，2026-07-29 已设）再 `git commit --amend --reset-author --no-edit`；commit 未推送时安全，无需 force push。
- 未 commit：先设 `git config user.email ...` 再 commit。
- 全局 config 仍是 QQ 邮箱，新 clone 或其他仓库需重设（可考虑改 global）。
- user.name `Kimmel` 不受邮箱保护影响，无需改。
- 数字 ID 查询：`curl -s https://api.github.com/users/947344386-hue | grep '"id"'`。

关联 [[feedback-full-push-workflow]]（全量推方案）与 [[backup-before-edit-workflow]]（推后删备份）。
