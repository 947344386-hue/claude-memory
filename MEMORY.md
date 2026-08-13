# Memory Index

## Feedback
- [备份工作流](feedback_backup_workflow.md) — 改 ClaudeCore 已存在源码前先备份到 StoneBetting\，推 GitHub 后删备份
- [代编译偏好](feedback_auto_compile.md) — 改完代码主动问是否代编译。稳定命令：`cd "F:/UELibrary/KimmelRebirth" && printf '2\r\n\r\n' | ./Build.bat`（全量=2 改.h/反射；只改.cpp 用 `1`）
- [引擎 API 优先](feedback_engine_api_first.md) — 涉及 UE 引擎子系统前先通读插件公共头文件
- [Tick 性能——禁止 FindComponent](feedback_perf_findcomponentbytick.md) — Tick 中不得调用 FindComponentByClass 等 O(N) 遍历
- [去重工具代码](feedback_dedup_utility_code.md) — 相同逻辑 ≥3 份必须抽成共享工具函数
- [根目录整洁](feedback_root_cleanliness.md) — 项目根只放标准 UE 文件，文档放 F:\ClaudeLibrary\doc\
- [全量大推方案](feedback_full_push_workflow.md) — UE 工程全量推 GitHub 方案
- [GitHub noreply 邮箱](feedback_github_noreply_email.md) — 推 GitHub 前 commit author 必须用 noreply 邮箱
- [UE 验证边界](feedback_ue_validation_boundary.md) — 优先做 C++ 业务逻辑；蓝图配置、关卡摆放和 PIE 默认由用户完成
- [Memory 自动推 GitHub](feedback_memory_auto_push.md) — 每次写 memory 后自动 commit+push 到 claude-memory 仓库
- [Git 推送走代理](feedback_git_push_proxy.md) — 本机推 GitHub 须 `-c http.proxy=http://127.0.0.1:10808`（一次性参数）
- [UE 资源路径由用户管理](feedback_user_manages_asset_paths.md) — 不擅自恢复、移动或重命名资产
- [C++ UI Widget 可见性陷阱](feedback_ui_cpp_widget.md) — BuildDefaultLayout 放 NativeOnInitialized，Visibility=4 根因与修复
- [赌石打包回写 canonical guide](feedback_doc_return_write.md) — 打包开发结束后回写引导文档

## Reference
- [UE 资产合作与自动搭建工作流](reference_ui_widget_pattern.md) — C++/Python/DA/MID 分层协作；UI 路线：C++ 整体回退，WBP 整体换皮
- [赌石玩法开发引导](kimmelrebirth_stonebetting.md) — StoneBetting canonical guide 指针
- [UE 5.6 Python 材质脚本经验](ue5-python-material-building.md) — pin 名、崩溃规避、两阶段构建
- [打包封装交接文档](kimmelrebirth-packaging-handoff.md) — 打包封装最新交接文档指针
- [项目开发术语统一](reference_terminology.md) — 开窗→擦石、切割→解石