---
name: ue5-python-material-building
description: UE 5.6 Python 材质脚本核心踩坑经验——pin 名、节点 API、崩溃规避
metadata: 
  node_type: memory
  type: reference
  originSessionId: 13730bae-c1a1-428c-ae73-5c54845695a4
  modified: 2026-08-11T17:18:26.674Z
---

# UE 5.6 Python 材质构建核心经验

## Pin 名
- `connect_material_expressions(frm, frm_pin, to, to_pin)` 的所有 4 个参数必须精确匹配。空字符串 `""` 通常只对单输出/单输入节点有效
- `TextureSampleParameter2D` 的 UV 输入 pin 叫 `"UVs"`（注意大写 U、复数 s）

## 崩溃原因
- **TextureSampleParameter2D 无默认贴图**：材质编译器遍历 sampler 时 crash。必须给每个纹理参数设默认贴图
- Python 语法错误（如 positional arg after keyword arg）在 UE Python 解释器中会中断整个脚本执行

## 编译命令行
- `UnrealEditor-Cmd.exe` 路径：`"C:/Program Files/Epic Games/UE_5.6/Engine/Binaries/Win64/UnrealEditor-Cmd.exe"`
- 运行单个 Python 脚本：`-ExecCmds="py F:/path/to/script.py"`

## 两阶段构建策略
- 第一阶段：最简核心——验证不崩溃
- 第二阶段：在稳定基底上加细节
- 核心原则：每加一个节点类型先在编辑器里手动测 pin 名，再写进脚本

## 相关文件
- `F:\UELibrary\KimmelRebirth\Tools\build_m_stone_cut_face.py` — 最终 v5 切面材质脚本
- `F:\UELibrary\KimmelRebirth\Tools\build_m_cut_aim_line.py` — 瞄准线 Decal 材质脚本