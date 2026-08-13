---
name: terminology
description: 项目开发术语统一——开窗→擦石、切割→解石（文案/文档/memory/UI 用词，C++ 标识符保留英文）
metadata: 
  node_type: memory
  type: reference
  originSessionId: 6f668379-1a01-4bf6-ad21-03e1cf824113
  modified: 2026-08-12T09:48:15.434Z
---

项目开发术语统一（2026-08-07 定，用户拍板）。**文案、文档、memory、UI 字符串统一用以下术语；C++ 类名/函数名保留英文（Opening/Cutting）作内部标识符。**

## 两条主线玩法
- **擦石** = 打磨揭示型开窗玩法（旧称"开窗"）。对应代码 `AClcJadeWorkbench`、`AClcOpeningStone`、`AClcOpeningTool`、`UClcOpeningMaskComponent`
- **解石** = 铡刀切片型切割玩法（旧称"切割"）。对应代码 `AClcCuttingTable`、`AClcCuttingStone`、`FClcStoneVoxelField3D`

## 衍生术语对照
| 旧 | 新 | 代码标识符（保留） |
|---|---|---|
| 开窗 | 擦石 | — |
| 开窗器 | 擦石器 | `AClcOpeningTool` |
| 开窗工作台 | 擦石台 | `AClcJadeWorkbench` |
| 切割 | 解石 | — |
| 切割台 | 解石台 | `AClcCuttingTable` |
| 切割石 | 解石载石 | `AClcCuttingStone` |
| 升级台 | 拓局专线 | `AClcToolUpgradeStation` |

## 不改的
- C++ 类名/函数名/UPROPERTY：`Opening`/`Cutting`/`Grind` 等英文标识符保留
- 材质/资产名保留