---
name: feedback_cook_dynamic_load_assets
description: 运行时字符串路径动态加载的资产必须配 DirectoriesToAlwaysCook，否则打包黑屏
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 31ccbcfe-c95f-4f41-a582-27934b93125f
  modified: 2026-08-13T17:03:25.980Z
---

打包独立 exe 黑屏（不崩、进程正常进主菜单但画面全黑）的根因之一：代码用字符串路径运行时 `LoadClass`/`LoadObject` 动态加载的资产（如 `LoadClass<UClcMainMenuWidget>(nullptr, TEXT("/Game/JadeBetting/UI/WBP_MainMenu.WBP_MainMenu_C"))`、`DeveloperSettings` 里的 `FSoftObjectPath` 配置如 `DA_StoneConfig`），编辑器 cook 依赖扫描扫不到这些字符串引用 → 资产不进包 → 运行时日志报 `SkipPackage: does not exist` → 菜单回退 C++ 裸布局 + 数据全空 → 黑屏。

**Why:** UE cook 默认只打包被静态引用链扫到的资产。字符串路径加载是运行时行为，cook 阶段看不到。`Manifest_UFSFiles_Win64.txt` 里能直接验证哪些资产进了包。

**How to apply:** 在 `Config/DefaultGame.ini` 加段，段名必须固定为 `[/Script/UnrealEd.ProjectPackagingSettings]`（`UProjectPackagingSettings::OverrideConfigSection` 强制回写旧名）。键：
```
+DirectoriesToAlwaysCook=(Path="/Game/JadeBetting")   ; 强制整目录进包
```
**关键坑**：`FDirectoryPath`/`FFilePath` 是结构体，ini 值**必须带 `(Path="...")` 括号**。写裸字符串 `+DirectoriesToAlwaysCook=/Game/JadeBetting` 会触发打包日志 `ImportText: Missing opening parenthesis` → 配置静默失效 → 资产/地图不进包 → 黑屏，且不会报错。`DirectoriesToAlwaysCook` 覆盖整个目录含 `Level/*.umap` 玩法关卡，比 `MapsToCook`（`FFilePath` 格式有 `LongPackageName`+`RelativeToGameContentDir` 冲突不确定）更可靠，优先用它。纯 ini 改动，无需编译，直接重新打包即生效。打包后用 `Manifest_UFSFiles_Win64.txt` 复核动态加载资产和玩法地图是否进了包。

关联：打包独立 exe 调试看 `F:\StoneJade\Windows\KimmelRebirth\Saved\Logs\` 和 `Saved\Crashes\`，不是编辑器日志。见 [[feedback_ui_cpp_widget]]（C++ 回退布局可见性）、[[kimmelrebirth-packaging-handoff]]。
