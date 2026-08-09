---
name: ue-asset-collaboration-workflow
description: UE UI、材质和 DataAsset 的统一合作流——C++ 可运行基线、Unreal Python 搭资产、DA 注入真实引用、蓝图可选换皮
metadata: 
  node_type: memory
  type: feedback
  originSessionId: cafda970-e47d-4931-908e-f3dd0ceb14d7
  modified: 2026-08-09T11:32:15.961Z
---

优先采用统一的 UE 资产合作流：**C++ 定义运行时契约并提供可运行基线，Unreal Python 确定性创建或重建资产外壳，DataAsset 保存用户的真实内容引用，蓝图/WBP 仅作为可选美化层，最后按“脚本→资产→编译→编辑器/PIE”分层验证。**

**Why:** 用户认可此前零蓝图 UI 的合作方式，也认可本次 `M_StoneCutFace` 的材质方式：Claude 负责把结构、参数、连线和运行时注入链路搭好；用户不需要手工重复搭节点，只维护自己的贴图引用、视觉排版和最终 PIE 体感。这样既能自动化，又不会把用户资产硬编码进公共模板。

**How to apply:**

## 1. 先划分四层职责

1. **C++ 运行时层**
   - 负责业务逻辑、生命周期、数据推送和参数名契约。
   - UI 要有无需 WBP 也能工作的默认布局；材质要有 MID 创建和参数注入入口。
   - 资产缺少可选美化时，核心功能仍应可运行。

2. **Unreal Python 资产搭建层**
   - 用 Unreal Python 创建或幂等重建 WBP 配套资产、材质图、Material Instance 或 DataAsset 初始结构。
   - 脚本负责固定节点、参数、连接、默认值和保存，不负责绑定用户尚未确认的美术资产。
   - 已存在资产优先保留资产身份并重建内部结构，不随意删除再创建，以免破坏现有引用。

3. **DataAsset 内容引用层**
   - 用户的实际贴图、动画、声音、Widget Class 等内容引用集中放进 DA；运行时由 C++ 注入 MID、Widget 或 Actor。
   - DA 路径继续由 DeveloperSettings 或当前项目的集中配置管理，不在多个业务类里重复硬编码。
   - 配 DA 字段时使用 Unreal Python 的 `set_editor_property`，保存后再做只读审计。

4. **蓝图/WBP 可选表现层**
   - UI 默认用 C++ 布局即可完整运行；需要换皮时再建 WBP 子类，并通过可选同名控件覆盖。
   - 材质主图只保留参数和通用连线；用户以后只需维护 DA 中的真实纹理引用。
   - 蓝图主要承担排版、Mesh、动画、音效和视觉参数，不复制 C++ 业务逻辑。

## 2. UI 路线

- C++ Widget 提供默认布局和完整交互，蓝图不存在时也能使用。
- 可选控件采用 `BindWidgetOptional`；若 WBP 提供同名控件则使用 WBP，未提供则继续用 C++ 默认控件。
- 默认布局只在 Widget 没有蓝图 Root 时创建，避免覆盖设计师布局。
- 事件绑定需要可重复构造而不叠加；Widget 销毁时解绑并清理引用。
- 常驻 HUD 由 LocalPlayerSubsystem 管单例和生命周期；某个 Actor/工作站专属的临时 UI 由该业务所有者管理。
- Widget Class 未配置时回退到 C++ Widget 类，确保 clone→编译后不依赖额外 WBP 才能运行。

核心骨架：

```cpp
UPROPERTY(BlueprintReadWrite, meta = (BindWidgetOptional))
TObjectPtr<UTextBlock> TitleText;

void UMyWidget::NativeOnInitialized()
{
    Super::NativeOnInitialized();
    BuildDefaultLayout();
}

void UMyWidget::BuildDefaultLayout()
{
    if (!WidgetTree || WidgetTree->RootWidget) return;
    TitleText = WidgetTree->ConstructWidget<UTextBlock>(
        UTextBlock::StaticClass(), TEXT("TitleText"));
    WidgetTree->RootWidget = TitleText;
}
```

- `ConstructWidget` 的对象名必须和 `BindWidgetOptional` 属性名一致。
- `NativeConstruct` 中先 `RemoveDynamic` 再 `AddDynamic`；`NativeDestruct` 中解绑。
- Subsystem 创建时优先配置的 `TSubclassOf`，为空则回退 `UMyWidget::StaticClass()`，再 `CreateWidget + AddToViewport`。

## 3. 材质路线

- 先在 C++/DA 中确定稳定参数契约，再由 Python 搭 Master Material，避免脚本和运行时参数名分叉。
- Master Material 只创建参数、混合节点和材质属性连接，不直接引用用户的游戏贴图。
- Texture Parameter 必须有能正常编译的中性默认纹理；真实 BaseColor、Normal、ORM 等由 DA 在运行时覆盖。
- ORM 约定、Sampler Type、sRGB 和 mask 通道必须在资产、DA 注释和运行时代码中保持一致。
- 生成脚本必须：
  1. 先判断资产是否存在；
  2. 不存在时通过 AssetTools + 对应 Factory 创建；
  3. 已存在时保留资产对象并清理后重建内部图；
  4. 对每条表达式连接和材质属性连接检查返回值；
  5. Layout、Recompile、Save；
  6. 重复执行一次验证幂等性。
- 不把“资产文件已经出现”等同于“材质可用”；仍需检查参数、连线、默认纹理和实际 MID 注入。

UE 5.6 创建/重建材质的正确骨架：

```python
assets = unreal.EditorAssetLibrary
materials = unreal.MaterialEditingLibrary
mat_path = "/Game/Path/M_Master"
package_path, asset_name = mat_path.rsplit("/", 1)

mat = assets.load_asset(mat_path) if assets.does_asset_exist(mat_path) else None
if mat:
    materials.delete_all_material_expressions(mat)
else:
    mat = unreal.AssetToolsHelpers.get_asset_tools().create_asset(
        asset_name,
        package_path,
        unreal.Material,
        unreal.MaterialFactoryNew(),
    )
if not mat:
    raise RuntimeError("Failed to create {}".format(mat_path))
```

关键 API 约束：

- `MaterialFactoryNew` 本身没有 `create_asset`；必须由 `AssetTools.create_asset` 创建，第三参数是 `unreal.Material` 类，第四参数是 `unreal.MaterialFactoryNew()` 实例。
- UE Python 枚举使用全大写成员，如 `MD_SURFACE`、`BLEND_OPAQUE`、`MSM_DEFAULT_LIT`、`SAMPLERTYPE_NORMAL`、`MP_BASE_COLOR`。
- 表达式连接签名是 `connect_material_expressions(source, source_output, target, target_input)`。
- 材质属性连接签名是 `connect_material_property(expression, output_name, material_property)`；通常 `output_name` 传空字符串。
- BaseColor/Normal/ORM 可分别使用 `/Engine/EngineResources/WhiteSquareTexture.WhiteSquareTexture`、`/Engine/EngineMaterials/DefaultNormal.DefaultNormal`、`/Engine/EngineMaterials/DefaultDiffuse_TC_Masks.DefaultDiffuse_TC_Masks` 作中性默认纹理。
- ORM 的 Texture Sample 使用 `SAMPLERTYPE_MASKS`；Normal 使用 `SAMPLERTYPE_NORMAL`，不能一律当 Color。
- 结束时执行 `layout_material_expressions`、`recompile_material`、`save_asset(..., only_if_is_dirty=False)`。

## 4. DataAsset 路线

- 创建或配置 DA 使用 Unreal Python，而不是依赖不具备 DA 字段编辑能力的外部 MCP。
- 基本步骤：加载 DA → `set_editor_property` 写字段/数组/对象引用 → 保存 → 重新加载或审计实际值。

```python
asset = unreal.EditorAssetLibrary.load_asset("/Game/Path/DA_Config")
asset.set_editor_property("field_name", value)
unreal.EditorAssetLibrary.save_asset(
    "/Game/Path/DA_Config", only_if_is_dirty=False)
```

- `FText` 用 `unreal.Text("文本")`；对象引用先 `unreal.load_asset`；数组直接传 Python list 或对应 Unreal struct 列表。
- 复杂内容引用优先使用软引用；运行时集中加载和注入，避免 Master Material、Widget 或多个 Actor 各自持有重复硬引用。
- 用户的内容资产路径由用户规划；除非明确要求，不移动、重命名或“修复”现有资产位置。

## 5. 脚本执行与存放

- 编辑器已打开且适合交互执行时，用 Output Log 的 Python 命令运行脚本；不要使用已废弃的 `exec(open(...).read())` 形式。
- 需要无人值守、可重复验证时，用 UE 的 PythonScript commandlet，并以日志中的脚本成功和 `0 error(s), 0 warning(s)` 为通过标准。
- commandlet 从外部创建资产后，已打开的编辑器可能需要刷新 Content Browser 或重启才能识别。
- 可复用脚本放工作库；一次性脚本用完清理。不要把 Python 工具、日志或缓存散落在 UE 项目根目录。

## 6. 统一验证阶梯

1. **脚本静态检查**：语法正确，无临时缓存残留。
2. **UE 脚本验证**：创建/重建成功，连接断言全部通过，日志无 Error/Warning。
3. **幂等验证**：同一脚本第二次运行仍成功，不改变资产身份、不新增重复节点。
4. **C++ 编译验证**：运行时参数、DA 类型或 Widget C++ 有变更时按改动类型编译；编译前提醒用户保存编辑器内容。
5. **编辑器目视验证**：确认材质图、WBP 绑定、DA 引用和资产加载状态。
6. **PIE 体验验证**：确认真实贴图、UI 排版、交互和恢复链路；默认由用户完成，除非用户明确要求代做。

任何一级未完成，都要明确写成“资产已生成但待编译/待 PIE”，不能笼统宣称功能全部完成。

## 7. 选择策略

- **纯业务逻辑或状态机**：C++。
- **无需美化也必须可用的 UI**：C++ 默认 Widget。
- **确定性的资产结构、节点和初始配置**：Unreal Python。
- **用户实际美术引用和可调内容**：DataAsset。
- **视觉换皮、排版、动画和音效**：可选 Blueprint/WBP。
- **运行时变化的材质参数**：MID，由 C++ 从 DA 注入。

## 8. 与其他长期规则配合

- 改已有 ClaudeCore 源码前遵循 [[backup-before-edit-workflow]]。
- 脚本与缓存清理遵循 [[project-root-cleanliness]]。
- 引擎资产 API 先查公共接口，遵循 [[engine-api-first]]。
- 用户负责资产路径与最终视觉规划，遵循 [[user-manages-ue-asset-paths]]。
- 编译和 PIE 边界遵循 [[ask-to-compile-on-behalf]] 与 [[feedback-ue-validation-boundary]]。
- KimmelRebirth 赌石资产的当前实现细节以 [[kimmelrebirth-stonebetting]] 和当前源码/资产为准，不把阶段性文件状态固化在本记忆中。
