---
name: ue-asset-collaboration-workflow
description: 跨项目 UE 资产合作流——C++ 运行时基线、可审计 Python 生成器、DA 内容注入、可选蓝图表现和分层验收
metadata:
  node_type: memory
  type: feedback
  originSessionId: cafda970-e47d-4931-908e-f3dd0ceb14d7
  modified: 2026-08-09T11:48:31.180Z
---

优先采用统一且可跨项目复用的 UE 资产合作流：**C++ 定义运行时契约并提供不依赖蓝图换皮的可运行基线；Unreal Python 只搭建可确定描述的资产结构；DataAsset 或项目现有的集中配置保存用户真实内容引用；Blueprint/WBP 只承担可选表现；最后按脚本、资产、编译、编辑器和 PIE 分层验收。**

**Why:** 用户认可此前零蓝图 UI 的合作方式，也认可 `M_StoneCutFace` 的材质合作方式：Claude 负责结构、参数、连线、运行时注入和可审计脚本；用户只维护真实美术引用、资源目录规划、视觉排版和最终 PIE 体感。工作流必须能换项目复用，同时不能因“自动化”省略资产所有权、API 版本、失败回滚、运行时注入和验收边界。

**How to apply:**

## 1. 适用范围与四层职责

这套方式适用于材质、Material Instance、DataAsset、简单配置资产、C++ 默认 UI，以及其他能够被稳定规格化、且当前 UE Editor API 确实支持的资产。地图、复杂动画、精细 WBP 排版和高度手工化的视觉资产不强行脚本化。

1. **C++ 运行时层**
   - 负责业务逻辑、生命周期、数据推送、资源加载边界和稳定参数契约。
   - UI 在没有 WBP 时也应能运行；材质运行时负责创建并缓存 MID，再把集中配置注入进去。
   - 必需的 Master Material 是明确的运行时依赖；用户尚未填写的可选贴图可以由中性默认值承接，但不能静默掩盖 Master 丢失。
   - 不在 Tick 中查资产、同步加载软引用或重复创建 MID；初始化一次并缓存。

2. **Unreal Python 资产搭建层**
   - 只负责确定性的资产外壳、节点、参数、连线、默认值、保存和验证。
   - 生成器不硬编码用户尚未确认的游戏贴图、动画、声音或 Widget Class。
   - 只允许重建明确由生成器拥有的资产；遇到用户手工资产或所有权不明的同路径资产必须停止，不能直接清空节点。

3. **DataAsset／集中配置层**
   - 用户真实贴图、动画、声音、Mesh、Widget Class 等引用集中放在 DA 或项目已有配置资产中。
   - C++ 从一个集中入口读取并注入，不让多个业务类重复硬编码路径和参数名。
   - 生成器可以创建初始结构或写明确授权的字段，但不得在每次重跑时覆盖用户已经填写的内容。

4. **Blueprint/WBP 可选表现层**
   - 负责换皮、排版、动画、音效、Mesh 选择和视觉调参。
   - 不复制 C++ 业务状态机，不承担必须存在才能运行的核心逻辑。
   - Python 对 WBP 的自动化只在当前 UE 版本 API 能稳定表达目标时使用；否则保留 C++ 默认 UI，让用户按需手工换皮。

## 2. 开工前先固定“资产契约”

任何脚本和 C++ 修改前，先形成一张最小但完整的契约表；不要边写节点边决定名字。

至少确认：

- 当前 UE 主次版本，以及所需 Python／Editor Scripting／目标插件是否启用。
- 目标资产的对象路径、资产类和路径所有者；资产目录由用户决定。
- 目标资产是“生成器拥有”还是“用户手工拥有”。
- 现有资产是否已被其他资产引用、是否在编辑器中加载或有未保存修改。
- C++ 消费者、创建时机、Outer／生命周期和材质槽位或 UI 所有者。
- 每个参数的精确名称、类型、默认值、Sampler Type、通道语义和运行时写入方。
- DA 的类、字段名、字段类型、软／硬引用策略和定位方式。
- 没有真实内容时的基线行为，以及编译、编辑器、PIE 各层的通过标准。

参数名大小写视为契约的一部分。Python、Master Material、DA 注释和 C++ `FName` 必须完全一致；不要在多个业务类里散落相同字符串。

## 3. 可复用生成器的结构

生成器拆成两部分：

1. **通用引擎助手**：存在性检查、类检查、创建资产、连接断言、保存、重新加载和验证。
2. **项目规格 `SPEC`**：资产路径、类、生成器 ID、结构版本、材质属性、参数清单、节点和连接。

换项目时主要替换 `SPEC`，不要复制整份脚本再逐行修改。相同助手逻辑出现三次后，抽到工作库的共享模块；只有一两个脚本时不提前做大框架。

推荐规格至少包含：

```python
SPEC = {
    "tested_engine": "5.6",
    "asset_path": "/Game/Path/M_Master",
    "asset_kind": "Material",
    "generator_id": "project.asset_name",
    "schema_version": 1,
    "parameters": [],
    "connections": [],
}
```

生成器应具备以下性质：

- **确定性**：节点名、参数名、创建顺序和布局坐标稳定。
- **幂等**：第二次运行不产生重复节点，不改变资产身份。
- **所有权明确**：仅重建 allowlist 中、且确认由本生成器维护的资产。
- **失败不提交结果**：优先在独立 commandlet 中重建；所有检查通过后才调用 Save。若在已打开编辑器中运行，异常虽然没有写盘，却可能已把目标 package 改成 dirty 的半成品；必须立即撤销、重新加载或重启编辑器，确认恢复后才能执行 Save All，不能把 Python 异常当成自动事务回滚。
- **可只读验证**：最好提供 `VALIDATE_ONLY` 或等价模式，只检查对象类、参数、连接和版本，不修改资产。
- **版本可追踪**：记录脚本测试过的 UE 版本和生成器 schema 版本；跨 UE 主次版本先核对公共头文件、当前文档和 Python 暴露签名。
- **用户内容隔离**：生成器维护结构，用户维护 DA／MI／WBP 表现层；不要让用户手改下次会被生成器重建的节点图。

若当前版本支持可靠的资产 metadata，可记录生成器 ID 和 schema 版本；否则至少用脚本内的精确路径 allowlist、源控状态和交付说明界定所有权。

## 4. UI 路线：C++ 整体回退，WBP 整体换皮

C++ Widget 提供默认布局和完整交互；`TSubclassOf` 未配置时回退原生 Widget 类。常驻 HUD 由 LocalPlayerSubsystem 管理实例和生命周期，某个 Actor／工作站专属的临时 UI 由该业务所有者管理。

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
    if (!WidgetTree || WidgetTree->RootWidget)
    {
        return;
    }

    TitleText = WidgetTree->ConstructWidget<UTextBlock>(
        UTextBlock::StaticClass(), TEXT("TitleText"));
    WidgetTree->RootWidget = TitleText;
}
```

必须明确以下细节：

- **没有 WBP Root**：C++ 创建完整默认 WidgetTree，并直接给 `TObjectPtr` 成员赋值。
- **WBP 已提供 Root**：`BuildDefaultLayout` 整体跳过，WBP 的 WidgetTree 完全接管布局。
- `BindWidgetOptional` 只会绑定 WBP 中同名控件；WBP 未提供的可选控件会保持空，不会逐控件回退成 C++ 默认控件。
- C++ 自建控件依靠直接赋值，不依赖自动绑定；对象名与属性名保持一致是为了 WBP 契约、调试和迁移一致性，不是 C++ 指针赋值的必要条件。
- WBP 中缺少后逻辑无法工作的控件，应改为 `BindWidget` 或在初始化时明确失败；真正可选的控件才使用 `BindWidgetOptional` 并做空指针分支。
- `NativeOnInitialized` 适合一次性默认树搭建；`NativeConstruct` 可能多次发生，动态委托先 `RemoveDynamic` 再 `AddDynamic`；`NativeDestruct` 解绑并清理外部弱引用。
- 多控件布局先创建根 Panel，再添加子控件并设置 Slot、锚点、对齐和 Padding，不能只创建控件而遗漏布局槽属性。
- Subsystem／Actor 用 `UPROPERTY(Transient) TObjectPtr<>` 持有 Widget；结束时 `RemoveFromParent` 并清引用。
- UI 数据优先由事件或显式刷新推送，不用 Tick 轮询业务对象。

因此，这里的回退粒度是“整个 C++ 默认布局”与“整个 WBP 布局”二选一，而不是在一个已有 WBP Root 中自动混入缺失的 C++ 控件。

## 5. 材质路线：Master 结构与真实内容解耦

先确定参数契约，再由 Python 搭 Master Material。Master 只包含通用参数、混合逻辑和材质属性连接；用户实际纹理由 DA 在运行时注入 MID。

每个纹理参数至少记录：

- 参数名和用途。
- Texture2D／Cube 等类型。
- `SAMPLERTYPE_COLOR`、`SAMPLERTYPE_NORMAL` 或 `SAMPLERTYPE_MASKS`。
- 使用 RGB、R、G、B、A 中哪个输出。
- 对应 DA 字段和 C++ 注入常量。
- 缺省纹理及其通道语义。

UE 5.6 已验证的材质创建／重建骨架：

```python
assets = unreal.EditorAssetLibrary
materials = unreal.MaterialEditingLibrary
mat_path = SPEC["asset_path"]
package_path, asset_name = mat_path.rsplit("/", 1)

mat = assets.load_asset(mat_path) if assets.does_asset_exist(mat_path) else None
if mat:
    if not isinstance(mat, unreal.Material):
        raise RuntimeError("Unexpected asset class: {}".format(mat_path))
    require_generator_owned(mat, SPEC)
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

`require_generator_owned` 是项目助手：所有权不匹配时直接中止。不能把“资产存在”自动解释为“允许清空”。

UE 5.6 关键 API 细节：

- `MaterialFactoryNew` 没有 `create_asset`；必须调用 `AssetTools.create_asset`。
- 第三个参数是 `unreal.Material` 类，第四个参数是 `unreal.MaterialFactoryNew()` 实例。
- 不存在时先用 `does_asset_exist` 判断，再 `load_asset`；直接加载预期不存在的路径会在 commandlet 日志中留下 Error，甚至使最终退出失败。
- 枚举成员使用全大写形式，例如 `MD_SURFACE`、`BLEND_OPAQUE`、`MSM_DEFAULT_LIT`、`SAMPLERTYPE_NORMAL`、`MP_BASE_COLOR`。
- 表达式连接签名为 `connect_material_expressions(source, source_output, target, target_input)`，不能传 tuple。
- 材质属性连接签名为 `connect_material_property(expression, output_name, material_property)`；表达式默认输出通常传 `""`，纹理通道则显式传 `"RGB"`、`"R"`、`"G"`、`"B"` 或 `"A"`。
- 每条连接都检查布尔返回值，失败立即抛错；不能只看资产文件是否生成。
- 明确设置 Material Domain、Blend Mode、Shading Model、Two Sided 等目标属性，不依赖编辑器当前默认值。
- 最后依次执行 Layout、Recompile、结构验证和 Save；只有全部通过才保存。

Texture Parameter 不能保持空纹理，否则材质可能编译失败。可用的编译占位包括：

```text
BaseColor: /Engine/EngineResources/WhiteSquareTexture.WhiteSquareTexture
Normal:    /Engine/EngineMaterials/DefaultNormal.DefaultNormal
Masks:     /Engine/EngineMaterials/DefaultDiffuse_TC_Masks.DefaultDiffuse_TC_Masks
```

其中 Masks 路径只保证提供非空的引擎占位纹理；若项目要求严格的 ORM 中性值，应使用满足实际通道约定的专用默认纹理或常量，例如 AO=1、Metallic=0，并为 Roughness 选择项目需要的默认值。

还要审计真实纹理的导入设置：BaseColor 通常使用 sRGB；Normal 和 ORM／Mask 通常关闭 sRGB，且使用对应压缩和 Sampler Type。除非用户明确授权，不自动改用户纹理的导入属性，只报告不匹配项。

若材质使用 Vertex Color 或其他顶点 mask，必须把每个通道的语义写入契约。顶点色会在三角形内部插值；需要硬边界时，不能只依赖稀疏顶点上的 0／1 值，应从网格密度、拆顶点、独立几何或其他 mask 方案解决。

## 6. MID 与 C++ 注入路线

运行时只通过一个集中入口创建和填充 MID：

1. 加载必需的 Master Material。
2. 用稳定 Outer 创建一次 MID，并缓存到拥有它的 Actor／Component／Subsystem。
3. 从 DA／集中配置加载真实内容。
4. 通过统一的 `InjectIntoMID` 或等价函数设置全部参数。
5. 将 MID 赋给正确的 Mesh 材质槽、Procedural Mesh section 或 Widget Brush。
6. 重建、恢复存档或生成新 section 时，确认新载体继续使用同一个契约和正确材质槽。

参数名集中定义为 `FName` 常量或单一契约表，不在多个 `.cpp` 中重复字符串。软引用只在明确的初始化边界加载一次；若同步加载会造成可见卡顿，改为项目已有的异步预载机制，但不在 Tick 中补救式加载。

MID 创建 API 要分清：

```cpp
CutFaceMID = UMaterialInstanceDynamic::Create(
    MasterMaterial, this, TEXT("CutFaceMID"));
Mesh->SetMaterial(MaterialSlotIndex, CutFaceMID);
```

- `UMaterialInstanceDynamic::Create` 的参数是 Source Material、Outer、可选名称，适合先创建再赋给动态 section／slot。
- `UPrimitiveComponent::CreateDynamicMaterialInstance` 的第一个参数是 `int32 ElementIndex`，不是 Master Material；调用成功时会给对应槽创建并设置 MID。
- MID 必须由拥有者用 `UPROPERTY(Transient) TObjectPtr<UMaterialInstanceDynamic>` 等 GC 可见引用持有；只放局部裸指针不可靠。
- Procedural Mesh 新建或重排 section 后要重新核对 `MaterialSlotIndex`，不能假设 cap 永远是固定 section；恢复存档和多次处理路径也要覆盖。

Master Material 缺失属于配置错误，应有明确日志或失败状态；用户内容贴图暂未配置时，可继续使用 Master 中性默认值，并在验收状态中写明 DA 尚未填充。

## 7. DataAsset 路线：结构可生成，用户内容不覆盖

配置已有 DA 的基本模式：

```python
assets = unreal.EditorAssetLibrary
asset_path = "/Game/Path/DA_Config"

if not assets.does_asset_exist(asset_path):
    raise RuntimeError("Missing DataAsset: {}".format(asset_path))

asset = assets.load_asset(asset_path)
asset.set_editor_property("field_name", value)
assets.save_asset(asset_path, only_if_is_dirty=False)
```

类型细节：

- `FText` 使用 `unreal.Text("文本")`。
- 对象引用先加载目标 UObject；类引用按字段类型提供对应 UClass。
- 枚举使用当前 Python 暴露的枚举成员。
- 数组传 Python list；结构体数组先创建正确的 Unreal struct，再逐字段赋值。
- `TSoftObjectPtr`／`TSoftClassPtr` 的具体赋值形式要按当前 UE 版本和字段暴露类型验证，保存后必须重新加载审计序列化结果。
- Python 属性名使用实际暴露给 Python 的名称，通常是 snake_case；不要直接猜 C++ 成员拼写。

生成器与内容填充分开：

- Schema 脚本负责创建 DA 资产外壳或验证字段契约。
- Content 脚本只写用户明确授权的字段。
- 幂等重跑不得把已填写的真实引用重置为默认值。
- 保存后重新加载，逐项核对对象路径、数组长度、结构字段和空引用数量。
- DA 的定位优先走 DeveloperSettings 或项目现有集中配置，不把 DA 路径散落到多个业务 Actor。

## 8. 脚本执行、编辑器并发与文件存放

编辑器交互执行：

```text
py "F:/WorkLibrary/script.py"
```

不要使用已废弃的：

```python
exec(open(...).read())
```

无人值守验证使用当前 UE 版本的 PythonScript commandlet：

```text
UnrealEditor-Cmd.exe Project.uproject -run=PythonScript -Script="F:/WorkLibrary/script.py" -unattended -nosplash -nullrhi
```

`-nullrhi` 只用于不需要实际渲染输出的资产操作；若目标 API 或验证依赖渲染，移除该参数。

执行前后遵守：

- 先保存编辑器中的蓝图、关卡、DA 和目标资产。
- 最安全的方式是不要让已打开编辑器与 commandlet 同时修改同一个 package。
- 若必须在编辑器开着时由 commandlet 写资产，目标 package 不能有未保存修改；执行后刷新或重启编辑器，防止旧的内存 package 在后续保存时覆盖磁盘新版本。
- 脚本静态检查不要遗留 `__pycache__`；若工具生成缓存，验证后清理。
- 可复用脚本放工作库或项目批准的 Tools 目录；一次性脚本和日志用完删除，不散落 UE 项目根目录。
- commandlet 以退出码、目标脚本成功标记和日志 Summary 共同判断；任何目标相关 Error 都算失败。无关的引擎启动 Warning 单独记录，不能用它掩盖脚本错误，也不能把无关 Warning 误报成资产逻辑失败。

## 9. 完整验证阶梯

### 0. 预检

- UE 版本、插件、路径和资产类已确认。
- 目标资产所有权明确。
- 编辑器目标 package 无未保存修改。
- Git 工作区已审计，知道本次允许变化的路径。

### 1. Python 静态检查

- 语法通过。
- 所有常量、枚举、参数名和路径已定义。
- 没有临时缓存或测试输出残留。

### 2. UE 脚本执行

- 创建或加载了正确类的资产。
- 所有节点／字段写入和连接断言通过。
- 未保存半成品。
- 日志无目标相关 Error。

### 3. 资产结构审计

- 对象路径和资产类正确。
- 参数名、类型、Sampler、默认纹理和通道语义与契约一致。
- 节点数量、关键连接、材质属性、DA 字段和数组内容符合规格。
- 真实纹理导入设置的不匹配项已报告。

### 4. 幂等验证

- 第二次运行仍成功。
- 资产对象身份和引用不变。
- 不新增重复节点、参数或数组项。
- 用户填写的 DA 内容不被重置。

### 5. C++ 编译验证

- 参数契约、DA 类型、Widget 或运行时注入代码变更后完成对应编译。
- 编译前按项目规则提醒用户保存编辑器内容。
- 只有实际出现成功结果才算通过，不以命令启动成功代替编译成功。

### 6. 编辑器目视验证

- Content Browser 能识别资产，无 stale package。
- 材质编辑器中参数、连线和属性正确。
- WBP 的必需／可选控件符合绑定契约。
- DA 显示真实引用，保存并重开后仍存在。

### 7. PIE／运行时验证

- 无 WBP／无真实贴图时的 C++／中性默认基线可运行。
- 填入真实 DA 内容后，MID、UI 或目标资产正确生效。
- 验证核心路径、恢复／重建路径和至少一个边界场景。
- 检查是否重复创建 Widget／MID、是否在 Tick 加载资源、是否写错材质 section／slot。

状态汇报必须分层，例如：

- `脚本和资产结构已验证`。
- `C++ 已编译，编辑器目视待确认`。
- `资产已生成但 DA 尚未填真实引用`。
- `PIE 待用户验证`。

不能用“资产已创建”代替“游戏内全链路完成”。

## 10. Source Control、回滚和交付

- 修改前后都检查 Git 状态，只允许预期的源码、生成脚本和 `.uasset` 变化。
- `.uasset` 是二进制，代码评审无法可靠阅读内部 diff；把生成脚本／声明式规格视为可审计源，同时保留资产结构验证结果。资产与生成器应在同一版本交付，避免只有二进制没有重建依据。
- 生成器对已有资产做破坏性重建前，确认源码管理或其他可靠回滚点存在；所有权不清时不动。
- 只 stage 明确相关文件，检查自动变化的 Config、token、凭据、缓存和构建产物。
- 项目仓库默认不自动 commit／push，等待用户明确提出大推；memory 中有意义的长期工作流更新按 memory 同步规则单独提交推送。
- 改已有 ClaudeCore 源码仍执行专用备份规则；项目尚未推送成功前保留备份。

每次交付至少说明：

- 生成或修改了哪些资产和脚本。
- 参数／字段契约以及用户需要填写的 DA 内容。
- 哪些资产内部由生成器拥有、哪些区域允许用户编辑。
- 已完成到验证阶梯的哪一级。
- 尚未完成的编译、编辑器或 PIE 项目。
- 当前 Git 状态、是否提交／推送，以及回滚点位置。

## 11. 常见错误清单

- 把 Factory 当成 Asset Class，或把 Factory 类当成实例。
- 调用不存在的 `MaterialFactoryNew.create_asset`。
- 使用旧版枚举大小写或旧版函数签名。
- 给连接函数传 tuple，而不是四个独立参数。
- 直接 `load_asset` 一个预期不存在的路径，制造 commandlet Error。
- Texture Parameter 无默认纹理，或 Normal／ORM 错用 Color sampler。
- 只创建了资产文件，没有验证节点连接和材质属性。
- 资产存在就无条件清空表达式，覆盖用户手工材质。
- 重跑脚本时覆盖用户已填写的 DA 引用。
- 编辑器持有旧 package，commandlet 写完后又被旧内存版本覆盖。
- 误以为 WBP 缺少某个可选控件时会自动使用 C++ 同名控件。
- MID 已创建但没有赋到正确 slot／section，或恢复路径重新生成 Mesh 后丢失材质。
- 把“脚本成功”“资产生成”“C++ 编译”“PIE 生效”混成一个完成状态。

## 12. 与其他长期规则配合

- 改已有 ClaudeCore 源码前遵循 [[backup-before-edit-workflow]]。
- 脚本、日志和缓存遵循 [[project-root-cleanliness]]。
- 涉及 UE 引擎系统先读公共 API，遵循 [[engine-api-first]]。
- 重复生成器助手达到三份再抽共享工具，遵循 [[dedup-utility-code]]。
- 初始化加载并缓存，不把资产查询放进 Tick，遵循 [[perf-findcomponentbytick]]。
- 用户负责资产路径与最终视觉规划，遵循 [[user-manages-ue-asset-paths]]。
- 编译和 PIE 边界遵循 [[ask-to-compile-on-behalf]] 与 [[feedback-ue-validation-boundary]]。
- 有意义的工作流变更同步遵循 [[memory-auto-push]]。
- 具体项目当下的资产路径和实现状态以当前源码、资产和项目 canonical guide 为准，不固化进这条跨项目工作流。
