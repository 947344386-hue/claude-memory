---
name: reference-ui-workflow
description: C++ 默认布局 + Subsystem 管 Widget 生命周期 + Python 建资产，蓝图不建也能用的 UI 开发工作流
metadata: 
  node_type: memory
  type: reference
  originSessionId: 933be537-2d5a-49c9-865f-b9d1b0fe0485
  modified: 2026-07-30T10:40:23.927Z
---

**传给新 session 的一句话：**

> "用 ClcTeleportMenuWidget / ClcKeyPromptWidget 的模式做 UI：C++ 写 Widget 基类，NativeOnInitialized 里调 BuildDefaultLayout() 用 WidgetTree->ConstructWidget<T>() 构造所有控件（Button/ProgressBar/Slider 等），NativeConstruct 里 AddDynamic 绑按钮事件，BindWidgetOptional 属性让蓝图可替换；用 ULocalPlayerSubsystem 管理 Widget 单例（CreateWidget + AddToViewport + RemoveFromParent）；资产用 Python 脚本 + UnrealEditor-Cmd.exe 批量建；不想建蓝图也能用 C++ 默认布局。具体参考 Plugins/ClaudeCore/Source/ClaudeCore/Public/Tools/Teleport/UI/ClcTeleportMenuWidget.* 和 Subsystems/ClcKeyPromptSubsystem.*。"

**核心文件参考：**
- Widget 基类（按钮+默认布局）：`Plugins/ClaudeCore/Source/ClaudeCore/Public/Tools/Teleport/UI/ClcTeleportMenuWidget.*`
- 列表项（按钮+文字）：`Plugins/ClaudeCore/Source/ClaudeCore/Public/Tools/Teleport/UI/ClcTeleportEntryWidget.*`
- 提示条（纯文本+Canvas 锚点）：`Plugins/ClaudeCore/Source/ClaudeCore/Public/UI/ClcKeyPromptWidget.*`
- Subsystem 管生命周期：`Plugins/ClaudeCore/Source/ClaudeCore/Public/Subsystems/ClcKeyPromptSubsystem.*`
- 资产创建 Python：`F:\ClaudeLibrary\doc\SetupTeleportAssets.py`（已删，可参考本 session 对话中的脚本内容）
- Enhanced Input 绑定：`ClcTeleportSubsystem.cpp` 的 `EnsureInputBinding()`

**关键时序：**
1. `NativeOnInitialized` → `BuildDefaultLayout()`（`WidgetTree->RootWidget == nullptr` 才生成，BP 已有布局就跳过）
2. `NativeConstruct` → 按钮 `OnClicked.AddDynamic`
3. `Subsystem` 懒创建：`CreateWidget` → `AddToViewport` → 数据刷新
4. `Subsystem::Deinitialize` → `RemoveFromParent` + 清理
5. `BindWidgetOptional` 的 UPROPERTY：蓝图同名控件自动绑定，不建也不报错