---
name: feedback-ue-validation-boundary
description: UE 功能开发优先做业务逻辑，蓝图配置、关卡摆放和 PIE 体验验证默认交给用户
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 933be537-2d5a-49c9-865f-b9d1b0fe0485
  modified: 2026-08-11T17:18:26.483Z
---

UE 功能开发时，主要投入 C++ 业务逻辑和必要的编译检查；蓝图细节配置、关卡摆放、测试地图搭建和 PIE 体验验证默认由用户完成，除非用户明确要求代做。

**Why:** 用户认为自动搭测试地图、配置蓝图和跑完整 PIE 会把时间浪费在验证上，希望助手集中处理业务逻辑。

**How to apply:** 实现完成后说明需要用户在编辑器配置/验证的项目，最多做不扩张范围的静态资产检查；不要主动创建临时测试关卡或长时间驱动编辑器验证。