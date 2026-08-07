---
name: engine-api-first
description: 涉及引擎系统操作前，先通读对应插件的公共头文件，确认是否有现成 API
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 111e71e8-fe56-4ec9-9510-e45461fdc63b
  modified: 2026-08-07T13:02:47.277Z
---

**原则**：在基于 UE 引擎子系统做开发之前，必须先读完对应插件的所有公共头文件（`.h`），确认引擎是否已有现成 API。

**为什么**：
- `ProceduralMeshComponent` 插件除了 `UProceduralMeshComponent` 本身，还有 `UKismetProceduralMeshLibrary` 提供工具函数——`SliceProceduralMesh`、`CopyProceduralMeshFromStaticMeshComponent`、`GetSectionFromStaticMesh` 等。我们手写了 200+ 行的 Sutherland-Hodgman 三角形裁剪 + cap 串链三角化代码，但引擎自 UE 4.x 起就用 `FGeomTools` 做了同样的事，且 UV/法线插值、碰撞生成都比手写版本成熟。
- 结果是剖面材质填充 bug 始终无法根除（手写 `ChainSegments` 的边段匹配逻辑有边界条件漏洞），而引擎 API 一步到位。

**如何应用**：
1. 涉及 `UProceduralMeshComponent` → `grep Slice/Copy/Get` 读 `KismetProceduralMeshLibrary.h` 和 `ProceduralMeshComponent.h`
2. 涉及任何引擎内置系统（物理、渲染、AI、动画等）→ 先到对应 `Engine/Plugins/` 或 `Engine/Source/Runtime/` 下列出公共头文件，确认 API 存量
3. 不要在匿名 namespace 里手写图形学基础算法（多边形裁剪、三角化、边段串链）→ 引擎的 `FGeomTools` 已经做了
4. 读到符合需求的 API 后看透签名和源码，确认边界语义（正侧保留、cap 材质分配、双面处理），再决定是否使用

**相关案例**：`[[kimmelrebirth_stonebetting]]` 中 `AClcCuttingStone` 的手动切割代码被 `SliceProceduralMesh` 替代后，删除了 ~200 行匿名 namespace 函数，剖面 cap 由引擎 `FGeomTools` 生成。
