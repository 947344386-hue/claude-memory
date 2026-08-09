---
name: terminology
description: 项目开发术语统一——开窗→擦石、切割→解石（文案/文档/memory/UI 用词，C++ 标识符保留英文）
metadata: 
  node_type: memory
  type: reference
  originSessionId: 6f668379-1a01-4bf6-ad21-03e1cf824113
  modified: 2026-08-09T11:32:48.052Z
---

项目开发术语统一（2026-08-07 定，用户拍板）。**文案、文档、memory、UI 字符串统一用以下术语；C++ 类名/函数名保留英文（Opening/Cutting）作内部标识符，材质资产名也保留。**

## 两条主线玩法
- **擦石** = 打磨揭示型开窗玩法（旧称"开窗"）。玩家用擦石器在擦石台上打磨皮壳揭示内部。对应代码 `AClcJadeWorkbench`（擦石台）、`AClcOpeningStone`（擦石载石）、`AClcOpeningTool`（擦石器）、`UClcOpeningMaskComponent`（擦石遮罩）。
- **解石** = 铡刀切片型切割玩法（旧称"切割"）。玩家在解石台上俯视角左右移动原石、铡刀落下切片，小块折金币、大块继续。对应代码 `AClcCuttingTable`（解石台）、`AClcCuttingStone`（解石载石）、`FClcStoneVoxelField3D`（3D 体素内部场）。

## 衍生术语对照
| 旧 | 新 | 代码标识符（保留） |
|---|---|---|
| 开窗 | 擦石 | — |
| 开窗器 | 擦石器 | `AClcOpeningTool` |
| 手电开窗器 | 手电擦石器 | `AClcCombinedTool` |
| 开窗工作台 / 工作台 | 擦石台 | `AClcJadeWorkbench` |
| 开窗遮罩 / 遮罩组件 | 擦石遮罩 | `UClcOpeningMaskComponent` |
| 开窗进度 / 已开窗 | 擦石进度 / 已擦石 | RuntimeData opened 字段 |
| 开窗面 / 开窗材质 | 擦石面 / 擦石材质 | `M_StoneOpening`（资产名保留） |
| 切割 | 解石 | — |
| 切割台 / 铡刀台 | 解石台 | `AClcCuttingTable` |
| 切割石 | 解石载石 | `AClcCuttingStone` |
| 铡刀 | 解石刀 | `BladeMesh` / `EClcRepairableTool::Blade` |
| 切割次数 / 切一刀 | 解石次数 / 解一刀 | — |

## 不改的
- C++ 类名/函数名/UPROPERTY：`Opening`/`Cutting`/`Grind` 等英文标识符保留（重命名风险大、动 BP/反射）。
- 材质/资产名 `M_StoneOpening`/`M_StoneShell` 等保留（uasset 名）。
- 纯几何技术描述（mesh 切分/三角形裁剪/cap 三角化）可保留"切分/裁剪"用词，与玩法层"解石"区分。

阶段属性 `EClcStonePhase{Unworked,Windowed,Cut}`：Windowed=已擦石、Cut=已解石（代码枚举名保留，文档用中文术语）。

链接：[[kimmelrebirth-stonebetting]]、[[ue-asset-collaboration-workflow]]。
