---
name: dedup-utility-code
description: 相同逻辑超过 2 份即应抽取为共享工具函数
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 111e71e8-fe56-4ec9-9510-e45461fdc63b
  modified: 2026-08-11T17:17:36.327Z
---

**原则**：相同的代码片段出现 3 次及以上，必须抽取到公共工具类/工具函数中，禁止逐文件粘贴。

**为什么**：
- `IsLookedAtByPlayer()` 在 5 个交互站点（解石台、擦石台、修理站、升级站、出售台）中逐字复制——同样的 `Pawn->FindComponentByClass<UClcInteractionComponent>()` + `GetLookedAtActor() == this`，5 份完全相同。
- `FindComponentByClass` 调用隐含 O(N) 组件扫描——逐份复制意味着逐份独立消耗，没有共享缓存的可能。
- 交互站点的状态机骨架（Inactive/AwaitingStone/StoneOnBench 三态 + KeyPrompt 动态注册 + 按键边沿检测）在各站点中 80% 雷同，每个站点只差具体的 Enter/Exit/Execute 行为。

**如何应用**：
1. 任何函数在 ≥2 个文件中出现相同实现 → 立刻抽成工具函数（静态/全局/基类）
2. 状态机骨架考虑基类 `AClcInteractableStation` 统一：`EnterKey`/`ExitKey` 轮询、`KeyPrompt` 动态注册、`PlayerInRange` 缓存、`IsLookedAtByPlayer` 全部下沉
3. 写新站点前先 grep 已有站点看看有没有可复用的函数