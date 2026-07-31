---
name: ui-widget-pattern
description: C++ Widget 默认布局 + BindWidgetOptional + Subsystem 单例的标准模式，可跨项目复用
metadata: 
  node_type: memory
  type: reference
  originSessionId: f404e089-1958-4eb9-ba91-c8dccd2e36b7
  modified: 2026-07-31T06:14:12.336Z
---

## 问题
UE UMG 蓝图做 UI 需要 C++ 父类 → 建 WBP 蓝图 → 摆控件 → 连线绑定，链条长、跨机器迁移易丢蓝图资产。目标是：**不建蓝图也能跑完整 UI**，蓝图只做可选美化。

## 核心模式

### 1. Widget 端：C++ 默认布局 + BindWidgetOptional

**参照**：`ClcTeleportMenuWidget`（`Plugins/ClaudeCore/Source/ClaudeCore/Public/Tools/Teleport/UI/`）、`ClcKeyPromptWidget`（`Public/UI/`）

**关键步骤**：

```cpp
// .h —— 三件套
UCLASS()
class UMyWidget : public UUserWidget
{
    GENERATED_BODY()
public:
    UMyWidget(const FObjectInitializer& ObjectInitializer);  // UE5 规范
protected:
    virtual void NativeOnInitialized() override;  // → BuildDefaultLayout
    virtual void NativeConstruct() override;      // → AddDynamic 绑按钮
    virtual void NativeDestruct() override;       // → 解绑+清弱引用

    // BindWidgetOptional：BP 可提供同名控件覆盖 C++ 默认
    UPROPERTY(BlueprintReadWrite, meta = (BindWidgetOptional))
    TObjectPtr<UTextBlock> MyText;
    UPROPERTY(BlueprintReadWrite, meta = (BindWidgetOptional))
    TObjectPtr<UButton> MyButton;
private:
    void BuildDefaultLayout();
};
```

```cpp
// .cpp —— BuildDefaultLayout 构造全控件
void UMyWidget::NativeOnInitialized()
{
    Super::NativeOnInitialized();
    BuildDefaultLayout();
}

void UMyWidget::BuildDefaultLayout()
{
    // WidgetTree 未就绪 或 BP 已提供 RootWidget → 跳过
    if (!WidgetTree || WidgetTree->RootWidget) return;

    // 根控件
    UBorder* Root = WidgetTree->ConstructWidget<UBorder>(UBorder::StaticClass(), TEXT("DefaultRoot"));
    WidgetTree->RootWidget = Root;

    // 子控件 —— 名字必须与 BindWidgetOptional 一致
    MyText = WidgetTree->ConstructWidget<UTextBlock>(UTextBlock::StaticClass(), TEXT("MyText"));
    Root->SetContent(MyText);
}

void UMyWidget::NativeConstruct()
{
    Super::NativeConstruct();
    if (MyButton)  // RemoveDynamic + AddDynamic 幂等
    {
        MyButton->OnClicked.RemoveDynamic(this, &UMyWidget::HandleClick);
        MyButton->OnClicked.AddDynamic(this, &UMyWidget::HandleClick);
    }
}

void UMyWidget::NativeDestruct()
{
    if (MyButton) MyButton->OnClicked.RemoveDynamic(this, &UMyWidget::HandleClick);
    // 清弱引用...
    Super::NativeDestruct();
}
```

**核心要点**：
- `WidgetTree->ConstructWidget<T>(T::StaticClass(), TEXT("Name"))` 构造控件，名字与 `BindWidgetOptional` UPROPERTY 名一致 → 自动绑定
- `WidgetTree->RootWidget` 非空 → BP 已提供布局，`BuildDefaultLayout` 跳过（BP 胜）
- BP 可整体替换布局，也可逐控件覆盖（同名即绑定）
- `BindWidgetOptional`（非 `BindWidget`）：控件可选，未提供时 C++ 默认生效
- `NativeConstruct` 里 `RemoveDynamic` + `AddDynamic` 幂等绑定（防重复构造）
- 构造函数用 `const FObjectInitializer&`（UE5 规范）
- `TObjectPtr<T>` 替代裸指针（UE5 规范）

### 2. Subsystem 端：ULocalPlayerSubsystem 管 Widget 单例

**参照**：`UClcKeyPromptSubsystem`（`Public/Subsystems/ClcKeyPromptSubsystem.h`）

**适用场景**：全局持久 Widget（如 HUD 提示、按键提示），非会话级 Widget（如工作台 HUD 由工作台 Actor 自己管）。

```cpp
UCLASS()
class UMySubsystem : public ULocalPlayerSubsystem
{
    // Widget 单例
    UPROPERTY(Transient) TObjectPtr<UMyWidget> Widget;
    // BP 可设 Widget 类，不设则 fallback C++ 类
    UPROPERTY(EditAnywhere) TSubclassOf<UMyWidget> WidgetClass;

    void CreateWidget()
    {
        if (!WidgetClass)
            WidgetClass = UMyWidget::StaticClass();  // C++ 默认布局
        Widget = CreateWidget<UMyWidget>(PC, WidgetClass);
        Widget->AddToViewport(zOrder);
    }
};
```

### 3. HUDWidgetClass fallback 模式

Widget 类槽位（`TSubclassOf<T>`）可在 BP Details 设，不设则自动 fallback 到 C++ 类，保证不建蓝图也能跑：

```cpp
TSubclassOf<UMyWidget> WidgetClass = HUDWidgetClass;
if (!WidgetClass)
    WidgetClass = UMyWidget::StaticClass();
HUDWidget = CreateWidget<UMyWidget>(PC, WidgetClass);
```

## 收益
- **零蓝图依赖**：C++ 编译完即完整 UI，无需建 WBP、摆控件、连线
- **跨机器迁移**：无 .uasset 依赖，纯 C++ 代码，git clone + 编译 = 完整功能
- **渐进增强**：想美化时建 WBP 选父类，逐控件覆盖（不改 C++）
- **内存更小**：不建蓝图 Widget 不打包冗余 UMG 蓝图资产

## 与本项目相关文件
- `ClcTeleportMenuWidget`（传送菜单，C++ 默认布局 + BindWidgetOptional）
- `ClcKeyPromptWidget`（按键提示，C++ 默认布局 + Subsystem 单例）
- `ClcVendorHUD`（回收台 HUD，C++ 默认布局 + HUDWidgetClass fallback）