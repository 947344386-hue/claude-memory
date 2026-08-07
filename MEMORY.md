# Memory Index

## Feedback
- [备份工作流](feedback_backup_workflow.md) — 改 ClaudeCore 已存在源码前先备份到 StoneBetting\，推 GitHub 后删备份
- [代编译偏好](feedback_auto_compile.md) — 改完代码主动问是否代编译。稳定命令（每次必读，别再试错）：Git Bash 里 `cd "F:/UELibrary/KimmelRebirth" && printf '2\r\n\r\n' | ./Build.bat`（全量=2 改.h/反射/新增文件；只改.cpp 用 `'1\r\n\r\n'`）。⚠️别套 `cmd /c`（会报 `'2' is not recognized`）；必须先让用户保存编辑器（Build.bat 会 taskkill UnrealEditor）；`timeout: invalid time interval '/t'` 无害
- [引擎 API 优先](feedback_engine_api_first.md) — 涉及 UE 引擎子系统前先通读插件公共头文件，别手写图形学基础算法
- [Tick 性能——禁止 FindComponent](feedback_perf_findcomponentbytick.md) — Tick 中不得调用 FindComponentByClass/GetAllActorsWithInterface 等 O(N) 遍历；一次缓存全程复用
- [去重工具代码](feedback_dedup_utility_code.md) — 相同逻辑 ≥3 份必须抽成共享工具函数，禁止逐文件粘贴
- [根目录整洁](feedback_root_cleanliness.md) — 项目根只放标准 UE 文件，文档放 F:\ClaudeLibrary\doc\，一次性工具用完删，编译残留及时清
- [全量大推方案](feedback_full_push_workflow.md) — UE 工程全量推 GitHub：排除可重生构建产物(DDC/Intermediate/Saved/Binaries)+dormant 插件，纳入 Source/ 保证 clone→编译→不丢引用直接跑
- [GitHub noreply 邮箱](feedback_github_noreply_email.md) — 推 GitHub 前提交 author 必须用 `251538000+947344386-hue@users.noreply.github.com`，QQ 邮箱会被 GH007 拦
- [UE 验证边界](feedback_ue_validation_boundary.md) — 优先做 C++ 业务逻辑；蓝图配置、关卡摆放和 PIE 默认由用户完成
- [Memory 自动推 GitHub](feedback_memory_auto_push.md) — 每次写 memory 后自动 commit+push 到 claude-memory 仓库，保证跨主机同步；路径相关记忆换机需更新
- [UE 资源路径由用户管理](feedback_user_manages_asset_paths.md) — 不擅自恢复、移动或重命名 KimmelRebirth 资产，资源目录规划由用户负责

## Reference
- [赌石玩法开发引导](kimmelrebirth_stonebetting.md) — 涉及 KimmelRebirth 赌石/原石/擦石/商人/鹰眼/背包/回收商时，先读 StoneBetting canonical guide 并在结束时维护
- [DA 配置——unreal Python 脚本](reference_da_config_python.md) — 配 DataAsset 字段走 unreal Python 脚本 + 编辑器 py 命令，VibeUE MCP 无此能力已删除
- [UE C++ Widget 标准模式](reference_ui_widget_pattern.md) — 零蓝图 UI：BuildDefaultLayout + BindWidgetOptional + Subsystem 单例，跨项目可复用
