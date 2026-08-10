---
name: ue5-python-material-building
description: UE 5.6 Python 材质脚本核心踩坑经验——pin 名、节点 API、崩溃规避
metadata: 
  node_type: memory
  type: reference
  originSessionId: 13730bae-c1a1-428c-ae73-5c54845695a4
  modified: 2026-08-10T11:30:26.642Z
---

# UE 5.6 Python 材质构建核心经验

**Why:** 多次试错后沉淀，避免每次从零排雷。涵盖 `MaterialEditingLibrary` pin 名匹配、节点创建 API 差异、材质编译器崩溃原因和规避手段。

**How to apply:** 每次写 UE Python 材质脚本前先读这份经验，尤其是 `_connect()` 必须用真实 pin 名、纹理参数必须设默认贴图、Noise 节点注意 `Position` pin 可选性。

## Pin 名

- `connect_material_expressions(frm, frm_pin, to, to_pin)` 的所有 4 个参数必须精确匹配。空字符串 `""` 通常只对单输出/单输入节点有效，多 pin 节点（`Step.X`/`Step.Y`、`Lerp.A`/`Lerp.B`/`Lerp.Alpha`、`Add.A`/`Add.B`）必须显式指定。
- `ComponentMask` 的输入 pin 名不是空字符串——在 UE 5.6 中 `get_material_expression_input_names` 返回 `[""]`（空名称），但 `connect_material_expressions` 用 `""` 有时连不上。稳定方案：始终用 `get_material_expression_input_names` 查询后取 `[0]`。
- `Noise` 节点的 `Position` 输入 pin 在 UE 5.6 Python 绑定中存在但可选——无连接时默认用 WorldPosition。连接 `TexCoord` 到 `Position` 需显式指定 `to_pin="Position"`。
- `TextureSampleParameter2D` 的 UV 输入 pin 叫 `"UVs"`（注意大写 U、复数 s）。

## API 存在性

| API | UE 5.6 状态 |
|---|---|
| `MaterialEditingLibrary.get_material_expression_input_names` | ✅ 存在 |
| `MaterialEditingLibrary.get_material_expression_output_names` | ❌ 不存在于 `MaterialEditingLibrary` |
| `unreal.PythonMaterialLib` | ❌ 模块不存在 |
| `connect_material_property(expr, "", MP_XXX)` | ✅ 第二个参数留空即可 |
| `NoiseFunction.NOISEFUNCTION_SIMPLEX_TEX` | ✅ 存在，比 `NOISEFUNCTION_SIMPLEX` 质量高 |

## 崩溃原因

- **TextureSampleParameter2D 无默认贴图**：`_tex()` 创建纹理参数节点后如果 `texture` 属性为 `None`，材质编译器遍历 sampler 时 crash（`EXCEPTION_ACCESS_VIOLATION`）。**必须**给每个纹理参数设默认贴图（`/Engine/EngineMaterials/DefaultWhiteGrid`、`/Engine/EngineMaterials/DefaultNormal`），运行时 `InjectIntoMID` 会覆盖。
- Python 语法错误（如 positional arg after keyword arg）在 UE Python 解释器中会中断整个脚本执行，没有 fallback。

## 噪声节点

- `MaterialExpressionNoise` 两个实例：一个扰动 UV（`scale=32, levels=4, turbulence=True`），一个调制粗糙度（`scale=64, levels=3`）
- `output_min=-1.0, output_max=1.0` 让噪声值以 0 为中心，可直接用作偏移
- 扰动公式：`TexCoord + Noise * NoiseUVStrength`（Strength 默认 0.08，范围 0~0.5）

## 编译命令行

- `UnrealEditor-Cmd.exe` 路径：`"C:/Program Files/Epic Games/UE_5.6/Engine/Binaries/Win64/UnrealEditor-Cmd.exe"`
- 运行单个 Python 脚本：`-ExecCmds="py F:/path/to/script.py"`
- 无头模式：`-stdin -unattended -NoSplash -NullRHI`
- 该命令启动耗时长（>300s），不适合自动化诊断；最简单方式是在编辑器内手动执行 `py`

## 两阶段构建策略

- **第一阶段**（v4）：最简核心——VertexColor → Step → Lerp → 6 纹参 → 属性输出。验证不崩溃。
- **第二阶段**（v5）：在稳定基底上加 Noise UV 扰动 + Noise Roughness 调制 + VertexColor.R 杂质分驱。
- 核心原则：**每加一个节点类型先在编辑器里手动测 pin 名，再写进脚本。**

## 相关文件

- `F:\UELibrary\KimmelRebirth\Tools\build_m_stone_cut_face.py` — 最终 v5 脚本
- `F:\UELibrary\KimmelRebirth\Plugins\ClaudeCore\Source\ClaudeCore\Public\Data\ClcJadeTextureConfig.h` — `InjectIntoMID` 定义（6 个纹理参数名）
- `F:\UELibrary\KimmelRebirth\Plugins\ClaudeCore\Source\ClaudeCore\Private\Actors\ClcCuttingStone.cpp` — `ApplyVoxelColorsToSection` + `SampleVoxelColor`（VertexColor 数据源）
