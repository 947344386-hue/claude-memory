---
name: backup-before-edit-workflow
description: 改 KimmelRebirth/ClaudeCore 插件已存在的源码前，必须先备份到 StoneBetting 工作库，推 GitHub 后删备份
metadata: 
  node_type: memory
  type: feedback
  originSessionId: d39ffac8-6499-4a2b-92b0-223eeca82082
  modified: 2026-08-11T17:17:36.092Z
---

改 ClaudeCore 插件任何**已存在**的源码文件前，先把当前版本复制到工作库 `F:\ClaudeLibrary\KimmelRebirth\StoneBetting\`；改动推送到 GitHub 之后，删除那份备份。新建文件不需要备份。

**Why:** 用户在交接文档 `ClaudeCore_Handoff.md` 的研发规则第 7 条明确写了此工作流。文档未说明动机，推测用于改动前留快照以便回滚。代码目录与文档/备份目录是分开的：源码在 `F:\UELibrary\KimmelRebirth\Plugins\ClaudeCore\Source\ClaudeCore\`，备份落在本工作库 `F:\ClaudeLibrary\KimmelRebirth\StoneBetting\`。

**How to apply:** 每次要 Edit/Write 一个已存在的插件源文件时，先用 Bash `cp` 把原文件复制到 StoneBetting 目录（保持同名），再动手改；改动 commit 并 `git push origin main` 成功后，`rm` 掉 StoneBetting 里的备份。配合规则 6「Bug 先讨论再修」——确认方向后再备份+改。GitHub 仓库 `947344386-hue/KimmelRebirth`，只提交 `Plugins/ClaudeCore/Source/` 下的文件。