---
name: feedback_cook_dynamic_load_assets
description: 运行时字符串路径动态加载的资产必须配 DirectoriesToAlwaysCook，否则打包黑屏
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 31ccbcfe-c95f-4f41-a582-27934b93125f
  modified: 2026-08-13T17:20:26.547Z
---

打包独立 exe 黑屏（不崩、进程正常进主菜单但画面全黑）的根因之一：代码用字符串路径运行时 `LoadClass`/`LoadObject` 动态加载的资产（如 `LoadClass<UClcMainMenuWidget>(nullptr, TEXT("/Game/JadeBetting/UI/WBP_MainMenu.WBP_MainMenu_C"))`、`DeveloperSettings` 里的 `FSoftObjectPath` 配置如 `DA_StoneConfig`），编辑器 cook 依赖扫描扫不到这些字符串引用 → 资产不进包 → 运行时日志报 `SkipPackage: does not exist` → 菜单回退 C++ 裸布局 + 数据全空 → 黑屏。

**Why:** UE cook 默认只打包被静态引用链扫到的资产。字符串路径加载是运行时行为，cook 阶段看不到。`Manifest_UFSFiles_Win64.txt` 里能直接验证哪些资产进了包。

**How to apply:** 在 `Config/DefaultGame.ini` 加段，段名必须固定为 `[/Script/UnrealEd.ProjectPackagingSettings]`（`UProjectPackagingSettings::OverrideConfigSection` 强制回写旧名）。两条都要：
```
+DirectoriesToAlwaysCook=(Path="/Game/JadeBetting")          ; 兜底动态加载的普通资产/WBP/DA
+MapsToCook=(FilePath="/Game/.../Level/Map_X")               ; .umap 玩法关卡——必须显式列！
```
**关键坑 1**：`FDirectoryPath`/`FFilePath` 是结构体，ini 值**必须带 `(Path="...")`/`(FilePath="...")` 括号**。写裸字符串会触发打包日志 `ImportText: Missing opening parenthesis` → 配置静默失效 → 资产/地图不进包 → 黑屏，且不报错。

**关键坑 2**：`DirectoriesToAlwaysCook` 只 cook 目录内被引用的**普通资产**，**不会把 `.umap` 当 cook 入口**。玩法关卡（不被默认地图 `GameDefaultMap` 引用的 `.umap`）必须靠 `MapsToCook` 显式列出，否则开始新游戏 `Browse` 到该地图 → `SkipPackage: does not exist` → `TravelFailure` → 回退主菜单 → 黑屏。引擎源码 `CookOnTheFlyServer.cpp:8346` 确认：GUI 打包无 `-Map` 命令行、项目无 `[AlwaysCookMaps]` ini 段时，`MapsToCook` 是地图入口的唯一来源。`FilePath` 值为**包路径不带扩展名**，格式 `+MapsToCook=(FilePath="/Game/JadeBetting/Level/Map_JadePlayTest")`（引擎注释 `CookOnTheFlyServer.cpp:8249` 示范）。

纯 ini 改动，无需编译，直接重新打包即生效。打包后用 `Manifest_UFSFiles_Win64.txt` 复核：动态加载资产 + 玩法地图都应出现在清单里。

关联：打包独立 exe 调试看 `F:\StoneJade\Windows\KimmelRebirth\Saved\Logs\` 和 `Saved\Crashes\`，不是编辑器日志。见 [[feedback_ui_cpp_widget]]（C++ 回退布局可见性）、[[kimmelrebirth-packaging-handoff]]。
