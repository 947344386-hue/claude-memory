---
name: kimmelrebirth-stonebetting
description: KimmelRebirth 赌石玩法的 canonical 开发引导与 session 路由指针
metadata: 
  node_type: memory
  type: reference
  originSessionId: 6f668379-1a01-4bf6-ad21-03e1cf824113
  modified: 2026-08-10T14:56:03.377Z
---

KimmelRebirth 的赌石玩法 canonical guide 位于 `F:\ClaudeLibrary\KimmelRebirth\StoneBetting\赌石玩法开发引导.md`。

**必须读取时机：** 任务涉及 KimmelRebirth、StoneBetting、JadeBetting、赌石、原石、擦石、解石、切石、玉石材质、摊位、商人、鹰眼、背包或回收商时，先完整读取该 guide，再读专项文档和当前代码；任务结束回写当前状态、已知偏差、验证结果和变更记录。

**稳定约束：** 玩法运行逻辑进入 `F:\UELibrary\KimmelRebirth\Plugins\ClaudeCore\Source\ClaudeCore\`；BP 主要负责排版与视觉；DataAsset 路径以 `UClcDeveloperSettings` 和当前资产为准；改已有 ClaudeCore 源码前先备份到 `F:\ClaudeLibrary\KimmelRebirth\StoneBetting\`；编译使用 `F:\UELibrary\KimmelRebirth\Build.bat`；不要误碰无关未跟踪/未提交文件。文档写完推 GitHub 前 commit author 用 `251538000+947344386-hue@users.noreply.github.com`。

**事实优先级：** 当前 ClaudeCore 源码 > 当前 JadeBetting 资产/DeveloperSettings > 已验证运行表现 > 规则书/设计稿/旧交接文档。

**事实优先级：** 当前 ClaudeCore 源码 > 当前 JadeBetting 资产/DeveloperSettings > 已验证运行表现 > 规则书/设计稿/旧交接文档。完整数据流、入口类、资产索引、验证清单和历史文档定位只维护在 canonical guide，避免记忆与文档分叉。
