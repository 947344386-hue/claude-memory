---
name: user-manages-ue-asset-paths
description: KimmelRebirth 的 UE 资源目录、移动和重命名由用户规划，除非明确要求否则不要代为调整
metadata: 
  node_type: memory
  type: feedback
  originSessionId: e9477b97-86db-471e-a85c-fecea88ac0cb
  modified: 2026-08-11T17:18:26.558Z
---

KimmelRebirth 的 UE 资源路径、资产移动和重命名由用户负责规划；即使 Git 显示旧路径删除、新路径新增，也不要擅自恢复、移动或改名。

**Why:** 用户明确表示会保证资源路径符合自己的规划，不需要 Claude 管理资源路径。

**How to apply:** 修改 C++、配置或文档时避开用户的资产整理；只有用户明确要求处理路径时才操作。需要引用资产时先读取当前 DA/源码事实，不根据 Git 的删除/新增状态自行"修复"资产位置。