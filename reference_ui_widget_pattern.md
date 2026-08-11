---
name: ue-asset-collaboration-workflow
description: 跨项目 UE 资产合作流——C++ 运行时基线、可审计 Python 生成器、DA 内容注入、可选蓝图表现和分层验收
metadata:
  node_type: memory
  type: reference
  originSessionId: cafda970-e47d-4931-908e-f3dd0ceb14d7
  modified: 2026-08-11T17:18:26.704Z
---

## UI 路线：C++ 整体回退，WBP 整体换皮

C++ Widget 提供默认布局和完整交互；`TSubclassOf` 未配置时回退原生 Widget 类。常驻 HUD 由 LocalPlayerSubsystem 管理实例和生命周期。

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
    // 构建 C++ 默认布局...
}
```

必须明确以下细节：

- **`NativeOnInitialized` 适合一次性默认树搭建；`NativeConstruct` 可能多次发生**，动态委托先 `RemoveDynamic` 再 `AddDynamic`
- **没有 WBP Root**：C++ 创建完整默认 WidgetTree
- **WBP 已提供 Root**：`BuildDefaultLayout` 整体跳过，WBP 的 WidgetTree 完全接管
- `BindWidgetOptional` 只会绑定 WBP 中同名控件；WBP 未提供的可选控件保持空
- WBP 中缺少后逻辑无法工作的控件，应改为 `BindWidget` 或在初始化时明确失败
- 多控件布局先创建根 Panel，再添加子控件并设置 Slot、锚点、对齐和 Padding
- Subsystem／Actor 用 `UPROPERTY(Transient) TObjectPtr<>` 持有 Widget；结束时 `RemoveFromParent` 并清引用
- UI 数据优先由事件或显式刷新推送，不用 Tick 轮询业务对象

**回退粒度是"整个 C++ 默认布局"与"整个 WBP 布局"二选一**，而不是在一个已有 WBP Root 中自动混入缺失的 C++ 控件。

**参见** [[feedback_ui_cpp_widget]]（PauseMenu 踩坑修复——Visibility=4 根因与修复）。