---
name: project-root-cleanliness
description: 项目根目录只放标准 UE 项目文件，文档/工具/残留及时清理
metadata: 
  node_type: memory
  type: feedback
  originSessionId: d39ffac8-6499-4a2b-92b0-223eeca82082
  modified: 2026-08-11T17:17:44.529Z
---

项目根目录 `F:\UELibrary\KimmelRebirth\` 只保留标准 UE 项目文件（.uproject、.sln、Build.bat、Config/、Content/、Source/、Plugins/、Binaries/、Intermediate/、DerivedDataCache/、Saved/、.git/、.vs/、.gitignore、.vsconfig）。

**Why:** 用户不喜欢根目录有零散的非必要文件，影响项目整洁。

**How to apply:**
- 开发文档（Handoff_*.md、Design_*.md 等）→ 放 `F:\ClaudeLibrary\doc\`
- 仅用于解决某一次配置问题的工具脚本（如 configure_merchant_da.py）→ 解决完及时删除，不要留在根目录
- 编译残留（build_err.txt、build_out.txt、build_wrapper.bat 等）→ 及时清理
- Build.bat 这类长期有用的编译工具留在根目录没问题