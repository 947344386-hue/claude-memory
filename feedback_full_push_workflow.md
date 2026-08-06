---
name: feedback-full-push-workflow
description: UE 工程全量大推 GitHub 的必备方案——排除可重生构建产物+dormant 插件，保证 clone→编译→不丢引用直接跑
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 9305f694-8007-4ec1-bb38-da1892ed3443
  modified: 2026-08-06T08:48:59.939Z
---

把整个 UE 工程（如 KimmelRebirth）推 GitHub、保证"clone→UE 打开→编译→不丢引用直接跑"的必备方案：**推整个工程，.gitignore 只排除可重生构建产物 + IDE 产物 + 已验证零引用的 dormant 插件**。

**Why:** 精简推送（只挑玩法文件夹）会漏 `Source/` 游戏模块——`.uproject` 引用它，漏了项目打不开；且难穷举所有被引用资产，易断引用。全量推则任何 uasset 引用都不断，仓库是完整可运行工程。

**How to apply:**
1. `.gitignore` 排除**可重生构建产物**：`Binaries/`/`Intermediate/`/`Saved/`/`DerivedDataCache/`/`LocalDerivedDataCache/`（DDC 尤其要排除，常 GB 级）+ IDE 产物 `*.sln`/`.vsconfig`。
2. **必纳入**：`Content/`（全量资产，含用户下载的 Fab/Marketplace 资产）+ `Source/`（游戏模块，.uproject 引用）+ `Config/` + 活跃插件 + 根文件（.uproject、Build.bat）。用户下载的内容资产不是"无关文件"——clone 后缺了就跑不起来，必须推。canonical guide 的"不要纳入无关未跟踪文件"特指构建产物（Binaries/Intermediate/Saved/DDC），不是内容资产。
3. dormant 插件（如 VibeUE）排除前**必须 grep 全项目 + 看 .uproject 插件列表**确认零引用——有引用就纳入，别排除。
4. 推前核验：`find -size +50M` 确认无单文件 >50MB（GitHub 100MB 硬限，留余量）；`du -sh` 确认排除构建产物后体量可接受（repo <1GB 软建议、<5GB 硬限）。
5. 无 LFS：单文件 <50MB 直接入库即可（UE .uasset 二进制直接 commit，与历史一致）；若日后 repo 过大再考虑 `git lfs migrate`（会改写历史）。
6. 前提：克隆方装对应 UE 版本（KimmelRebirth 是 5.6）；首次打开 UE 弹"是否编译"→点是。
7. `git add -A && git commit && git push origin main`；大推送用后台跑，中断可重试（已传对象不重传）。**推前必扫 `Config/DefaultEngine.ini`**：编辑器跑过后 AFS 段 `[/Script/AndroidFileServerEditor.AndroidFileServerRuntimeSettings]`（含 `SecurityToken=`）会**重生**——KimmelRebirth 是 Windows 项目用不上 AFS，token 不进公开仓库（同 `a3f780d` 清理）。`git add -A` 后 `git restore --staged Config/DefaultEngine.ini` 排除；或推前 `git diff Config/DefaultEngine.ini | grep -iE 'token|secret'` 自查。
8. 推 HTTPS 远端需凭证（credential helper / token），凭证已缓存则非交互成功；否则会卡在 Username 提示——先用 `git config --global credential.helper manager` 或配 PAT。

参考 [[feedback-backup-workflow]]（改源前备份）与 [[kimmelrebirth-stonebetting]]（赌石 canonical guide）。本方案在 2026-07-23 KimmelRebirth 全量推送落地。
