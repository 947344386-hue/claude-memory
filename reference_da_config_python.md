---
name: da-config-via-python
description: 配 UE DataAsset 字段的标准路径——写 unreal Python 脚本，编辑器 py 命令执行
metadata: 
  node_type: memory
  type: reference
  originSessionId: d39ffac8-6499-4a2b-92b0-223eeca82082
---

**Why:** VibeUE MCP（已删除）验证了它没有 DataAsset 字段配置能力（manage_asset 仅 7 个 action、Bridge.cpp TCP 分发无 DA 命令、execute_python_code 未接 TCP）。但验证过程中发现了真正可行的标准路径。

**How to apply:**
1. 写 unreal Python 脚本，用 `unreal.load_asset(path)` 加载 DA 实例
2. 设字段：`asset.set_editor_property("FieldName", value)`
   - FText → `unreal.Text("文本")`
   - 对象引用 → `unreal.load_asset("/Game/Path/To/Asset")`
   - 数组 → 直接传 Python list
3. 保存：`unreal.EditorAssetLibrary.save_asset(path)`
4. 执行：编辑器按 `~` 开命令台，输入 `py "脚本路径"`（前提：Edit→Plugins 启用 Python Scripting 插件）
5. 脚本放 `F:\UELibrary\KimmelRebirth\` 目录下跑，用完删（一次性工具不留根目录）

范例：`F:/UELibrary/KimmelRebirth/configure_merchant_da.py`（已删，配过商人 4 通道 DA）。

关联 [[backup-before-edit-workflow]] [[root-cleanliness]]