---
name: kimmelrebirth-stonebetting
description: KimmelRebirth 赌石玩法的 canonical 开发引导与 session 路由指针
metadata: 
  node_type: memory
  type: reference
  originSessionId: 6f668379-1a01-4bf6-ad21-03e1cf824113
  modified: 2026-08-11T10:21:25.152Z
---

KimmelRebirth 的赌石玩法 canonical guide 位于 `F:\ClaudeLibrary\KimmelRebirth\StoneBetting\赌石玩法开发引导.md`。

**必须读取时机：** 任务涉及 KimmelRebirth、StoneBetting、JadeBetting、赌石、原石、擦石、解石、切石、玉石材质、摊位、商人、鹰眼、背包或回收商时，先完整读取该 guide，再读专项文档和当前代码；任务结束回写当前状态、已知偏差、验证结果和变更记录。

**稳定约束：** 玩法运行逻辑进入 `F:\UELibrary\KimmelRebirth\Plugins\ClaudeCore\Source\ClaudeCore\`；BP 主要负责排版与视觉；DataAsset 路径以 `UClcDeveloperSettings` 和当前资产为准；改已有 ClaudeCore 源码前先备份到 `F:\ClaudeLibrary\KimmelRebirth\StoneBetting\`；编译使用 `F:\UELibrary\KimmelRebirth\Build.bat`；不要误碰无关未跟踪/未提交文件。文档写完推 GitHub 前 commit author 用 `251538000+947344386-hue@users.noreply.github.com`。

**打包开发规则：** 每次打包相关开发任务结束后必须回写 canonical guide 的"当前状态表"、"变更记录"和"当前待确认项"；跨 session 长线追踪参见 [[feedback_doc_return_write]]。每次编译完不要自动 push，等用户喊大推再推。

**事实优先级：** 当前 ClaudeCore 源码 > 当前 JadeBetting 资产/DeveloperSettings > 已验证运行表现 > 规则书/设计稿/旧交接文档。完整数据流、入口类、资产索引、验证清单和历史文档定位只维护在 canonical guide，避免记忆与文档分叉。

## 打包当前状态（2026-08-11 session）

### 已完成

| Phase | 内容 | 文件 |
|-------|------|------|
| 0 | GameInstance + Session类型 | ClcSessionTypes.h, ClcGameInstance.h/.cpp, DefaultEngine.ini 注册 GameInstanceClass |
| 1 | 主菜单界面 | ClcMainMenuSubsystem.h/.cpp, ClcMainMenuWidget.h/.cpp, Map_MainMenu + BP_MainMenuGameMode + WBP_MainMenu（用户建） |
| 2 | 存档系统 | ClcSaveManagerSubsystem.h/.cpp, Backpack/ToolDurability 各加 RestoreFromSaveData/SetSessionConfig |

编译：COMPILE SUCCESS（0 error 0 warning）

### 已知问题

| 问题 | 状态 |
|------|------|
| PIE 从 Map_MainMenu 点"开始新游戏"报 `GameInstance 不是 UClcGameInstance` | 已改 `GetLocalPlayer()->GetGameInstance()`，待用户 PIE 验证日志打出实际类型名 |
| 删档按钮无存档时不隐藏 | 已修复（`DeleteSaveButton` 与 `NoSavesText` 互斥），编译通过待验证 |
| `ContinueGame`/`QuitGame` 里的 `GetGameInstance` 旧写法未修正 | 下次一起改 |

### 待验证（用户 PIE）
1. 删档按钮互斥是否正确
2. 开始新游戏后 GameInstance 类型日志是什么

### 后续 Phase（下次 session）

| Phase | 内容 | 预计新增文件 |
|-------|------|------------|
| 3 | 任务/Quest系统 | ClcQuestTypes.h, ClcQuestConfig.h, ClcQuestSubsystem.h/.cpp, ClcQuestHudWidget.h/.cpp, DA_QuestConfig |
| 4 | 摊位价值档位管理器 | ClcStallTierConfig.h, ClcStallValueTierManager.h/.cpp, DA_StallTierConfig |
| 5 | 游戏流程整合（关卡转换+加载画面） | ClcGameFlowManager.h/.cpp, ClcLoadingScreenWidget.h/.cpp, DefaultEngine.ini 改 GameDefaultMap |
| 6 | 打磨与全链路验证 | 纯修改 |

### 规划文档
`C:\Users\mengwenbo\.claude\plans\bright-toasting-mitten.md`

### 当前未提交变更
无（本 session 最后一次编译修复后尚未 git add/commit）
