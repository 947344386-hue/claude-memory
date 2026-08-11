---
name: ask-to-compile-on-behalf
description: 改完 ClaudeCore 代码后主动询问是否代编译，需要则直接跑 Build.bat 而非让用户自己编译
metadata: 
  node_type: memory
  type: feedback
  originSessionId: d39ffac8-6499-4a2b-92b0-223eeca82082
  modified: 2026-08-11T17:16:37.257Z
---

改完 ClaudeCore 代码后，主动询问用户"要代编译吗"；用户说需要就直接跑 `F:\UELibrary\KimmelRebirth\Build.bat` 代编译，不要每次都让用户自己开终端跑。

**Why:** 用户明确说"下次你可以直接询问我是否需要代编译，需要的话直接帮我编译了就行"。用户希望减少自己手动编译的步骤。

**How to apply:** 代码改动完成、grep/逻辑自检后，主动问是否代编译。每次准备编译 KimmelRebirth 前必须先读取本记忆，不再现场试错调用方式。

稳定命令（已在本机验证）：

```bash
cd "F:/UELibrary/KimmelRebirth" && printf '2\r\n\r\n' | ./Build.bat
```

- 只改 `.cpp`：把 `2` 改为 `1`。
- 新增/删除 `.h/.cpp`、增加或修改 `UPROPERTY` / `UFUNCTION` / `UCLASS` 等反射结构：使用 `2`，让 UHT 和工程文件完整刷新。
- 必须使用 Git Bash 直接执行 `./Build.bat`；不要先套普通 `cmd.exe /c`，避免 `/c` 被路径转换或管道输入落到外层 cmd，出现 `'2' is not recognized`。
- 若因环境限制必须通过 cmd，已验证的兜底语法是 `cmd.exe //d //c "call F:\UELibrary\KimmelRebirth\Build.bat"`，但优先级低于直接 `./Build.bat`。
- Build.bat 会 `taskkill /F /IM UnrealEditor.exe` 和 `UnrealEditor-Cmd.exe`，执行前必须提醒用户保存蓝图、关卡和 DA，并取得确认。
- Git Bash 下输出 `timeout: invalid time interval '/t'` 是脚本中的 Windows `timeout` 被 Bash 同名程序截获；只要后续 UBT 正常运行并出现 `COMPILE SUCCESS`，该提示无害。
- 编译后必须检查实际输出：`COMPILE SUCCESS` 或 `Result: Succeeded` 才算通过；若出现 C++/UHT error，定位修复后重新编译，不能只看命令退出。

配合 [[backup-before-edit-workflow]]。