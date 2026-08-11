---
name: ui-cpp-widget-visibility
description: C++ 默认 UI Widget 的 BuildDefaultLayout 必须放 NativeOnInitialized，Visibility 陷阱与对齐 TeleportMenuWidget 模式
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 09752910-6d85-4268-a84e-b89941441bb5
  modified: 2026-08-11T17:15:09.610Z
---

**C++ 构建的默认 UI Widget（无 WBP 换皮）必须严格遵守以下规则，否则 Widget 在 PIE 中不可见（Collapsed 状态）且不会报错。**

## 核心规则

1. **`BuildDefaultLayout` 放 `NativeOnInitialized`，不放 `NativeConstruct`**
   - `NativeOnInitialized` 在 Widget 树初始化时调一次，此时 WidgetTree 完全就绪
   - `NativeConstruct` 时序更靠后，`GetVisibility()` 在此阶段仍返回 CDO 默认值（`Collapsed`=4），`SetVisibility` 调用无效
   - 参照：`UClcTeleportMenuWidget`（已验证可工作）

2. **根控件用 `UBorder` + `SetBrushColor` + `SetContent`，不要用 `UOverlay`**
   - `UBorder` 做根自动撑满全屏，`SetBrushColor` 设暗色背景
   - `UOverlay` 做根不自动撑满，子控件尺寸为 0×0 时什么都不渲染

3. **不要手动调 `SetVisibility`**
   - UE 的 UserWidget 默认 Visibility 是 `Visible`（不是 Collapsed）
   - 如果 `GetVisibility()` 返回 4（Collapsed），说明另有原因（CDO 状态、时序问题），不要用 `SetVisibility` 强改——修正时序即可
   - 在 KimmelRebirth 项目中，把 `BuildDefaultLayout` 从 `NativeConstruct` 移到 `NativeOnInitialized` 解决了 Visibility=4 问题

4. **按钮绑定：`NativeConstruct` 里 `RemoveDynamic` + `AddDynamic`**
   - 参照 TeleportMenuWidget，防止重复绑

5. **构造函数设 `SetIsFocusable(true)`**
   - 菜单打开时收到 Esc 关闭事件（`NativeOnKeyDown` → `ResumeGame`）

6. **按钮用 `SetContent` 设文字，不用 `AddChild`**
   - `Btn->SetContent(LabelWidget)` 而非 `Btn->AddChild(LabelWidget)`

## 已验证模板

```cpp
// .h
protected:
    virtual void NativeOnInitialized() override;
    virtual void NativeConstruct() override;
    // BindWidgetOptional 控件声明...

// .cpp 构造函数
SetIsFocusable(true);

// .cpp NativeOnInitialized
void UMyWidget::NativeOnInitialized()
{
    Super::NativeOnInitialized();
    BuildDefaultLayout();
}

// .cpp NativeConstruct
void UMyWidget::NativeConstruct()
{
    Super::NativeConstruct();
    // RemoveDynamic + AddDynamic 绑按钮
    if (MyButton) { MyButton->OnClicked.RemoveDynamic(...); MyButton->OnClicked.AddDynamic(...); }
}

// .cpp BuildDefaultLayout
void UMyWidget::BuildDefaultLayout()
{
    if (!WidgetTree || WidgetTree->RootWidget) return;
    UBorder* Root = WidgetTree->ConstructWidget<UBorder>(...);
    Root->SetBrushColor(FLinearColor(0.02f, 0.03f, 0.05f, 0.92f));
    WidgetTree->RootWidget = Root;
    // ... 构建内容 ...
}
```

## 相关案例

2026-08-12 PauseMenu 连续 4 轮编译调试菜单不显示，根因：
1. BuildDefaultLayout 在 NativeConstruct 里调 → Visibility=4 无法修正
2. 根控件用 UOverlay → DimBg 尺寸 0×0
3. 反复尝试 SetVisibility 强改 → 无效
4. 最终对齐 TeleportMenuWidget 模式（NativeOnInitialized + UBorder 根）→ 一次通过