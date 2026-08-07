---
name: perf-findcomponentbytick
description: 不要在 Tick 中频繁调用 FindComponentByClass；缓存到成员变量
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 111e71e8-fe56-4ec9-9510-e45461fdc63b
  modified: 2026-08-07T13:02:57.382Z
---

**原则**：任何 Actor/Component 的 `Tick` / `TickComponent` 中不得调用 `FindComponentByClass`、`GetComponentByClass`、`FindObject`、`GetAllActorsOfClass`、`GetAllActorsWithInterface` 等 O(N) 遍历操作。必须在上层容器（`BeginPlay`、`OnTriggerBeginOverlap`、构造函数）做一次查询并缓存到成员变量。

**为什么**：
- 在 KimmelRebirth 赌石项目中，5 个交互站点（解石台、擦石台、修理站、升级站、出售台）各自在 `Tick` 中调用 `Pawn->FindComponentByClass<UClcInteractionComponent>()`，且全部是 `TickInterval=0.0`（每帧）。5 个站点 × 60fps = 300 次/秒的 O(N) 组件扫描，造成每帧不必要的 CPU 消耗。
- 组件的 `Pawn` 引用在 `CachePlayerRefs()` 时就已经拿到了——完全可以在那时做一次查找并缓存 `TWeakObjectPtr<UClcInteractionComponent>`，后续 Tick 直接读。
- `ClcInteractionComponent::UpdateInteraction` 的 `GetAllActorsWithInterface` 虽然做了 1s 缓冲，但也是全局遍历——如果场景中交互物数量可控，应该在 `BeginPlay` 注册到 component 的数组里，不需要每 N 秒重新扫全场景。

**如何应用**：
1. 所有 `Tick` 用到的引用都应该在非 Tick 路径（`BeginPlay`、`OnTriggerBeginOverlap`、`CachePlayerRefs`）一次性获取并缓存
2. `FindComponentByClass` 只在初始化路径用，不在 Tick 用
3. `GetAllActorsWithInterface` 规模可控时改用注册/注销模式；规模不可控时加大缓存间隔（至少 5s）并取 Min 而非逐帧
4. 无独立 Tick 需求的站点（如修理站只在玩家瞄中时轮询按键）把 `TickInterval` 提到 0.1s 以上

**相关案例**：`[[kimmelrebirth_stonebetting]]` 所有交互站点共用一个 `UClcInteractionComponent`（挂在 Player Pawn 上），球扫结果通过 `GetLookedAtActor()` 返回——一次球扫供全部站点消费。每帧 5 次 `FindComponentByClass` 完全冗余。
