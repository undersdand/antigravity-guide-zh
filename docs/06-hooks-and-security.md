# 06. 权限引擎、终端沙盒与 Hooks 安全拦截

让 AI 能在本地跑命令行是一把双刃剑。Antigravity 提供了一套从**权限规则、OS 级终端沙盒到生命周期钩子（Hooks）**的三层防护体系。

---

## 一、四大执行策略预设与沙盒（Terminal Sandbox）

在桌面端 **Settings** 或 CLI 的 `/config` 中，可以切换工具执行策略：

| 策略预设 | 终端沙盒状态 | 执行行为说明 |
| :--- | :---: | :--- |
| **Request Review（请求审查）** | 关闭 | 每次执行命令前都弹窗询问用户确认（最保守） |
| **Default（默认）** | 开启 | 沙盒允许范围内的命令自动放行；跳出沙盒的操作需用户审批 |
| **Auto（自动）** | 开启 | 沙盒内自动放行；沙盒外安全、可逆的操作也允许自动运行 |
| **Turbo / Always Proceed** | 关闭 | 所有命令无提示直接执行（⚠️ 仅建议在一次性隔离容器或非敏感环境使用） |

### 沙盒底层是怎么工作的？
Antigravity 的 Terminal Sandbox **不依赖厚重的 Docker 或虚拟机**，而是直接调用操作系统原生隔离机制：
- **macOS**：Apple Seatbelt (`sandbox-exec`)
- **Linux**：`cgroups` + `seccomp` 命名空间
- **Windows**：独立桌面会话隔离（如 `exebox-...` 桌面）

在沙盒内，命令默认拥有**当前项目文件夹、临时目录及构建缓存的读写权限**，但像 `~/.ssh`、关键敏感目录以及未授权的网络请求都会被隔离限制。

---

## 二、细粒度权限配置（`/permissions` 与 Settings）

除了全局策略，还可以按 **Global（全局）** 和 **Project（项目级，优先级高于全局）** 精细控制：
- **Tool Execution Policy**：控制命令审批策略
- **Non-Workspace File Access**：`allow` / `ask` / `deny`（是否允许读写项目以外的目录）
- **Internet Access Policy**：`allow` / `ask` / `deny`（是否允许联网）
- **Command Allowlist / Denylist**：支持精确命令或正则规则（如 `command(git)`、`command(regex:git .*)`）
- **Shell 组合命令保护**：凡是包含管道 `|`、重定向 `>` 或子 Shell `$(...)` 的复杂命令，除非精确匹配白名单，否则系统会自动降级为弹窗人工确认。

---

## 三、Lifecycle Hooks（`hooks.json`）深度指南

如果你希望在 Agent 执行某一步之前做**硬性安全审计**、执行后**自动跑代码格式化**、或阻止 Agent 在测试通过前停下，可以使用 `hooks.json`。

### 1. 放置位置
- 工作区：`<project>/.agents/hooks.json`
- 原生插件：`<plugin_dir>/hooks.json`

### 2. 支持的六大生命周期事件

| 事件名 | 触发时机 | 是否需要 `matcher` | 典型用途 |
| :--- | :--- | :---: | :--- |
| **`SessionStart`** | 新对话启动时触发一次 | 否（扁平数组） | 注入欢迎提示或当前项目环境摘要 |
| **`PreToolUse`** | 工具执行**前** | **是**（按工具名正则匹配） | 拦截危险命令（如 `rm -rf`）、重写命令参数、强制弹窗审批 |
| **`PostToolUse`** | 工具执行**后** | **是**（按工具名正则匹配） | 编辑文件后自动运行 `eslint --fix` / `prettier` 或记录审计日志 |
| **`PreInvocation`** | 每次请求大模型**前** | 否（扁平数组） | 注入临时提醒（`ephemeralMessage`）或直接终止请求 |
| **`PostInvocation`** | 工具结果返回、推理循环继续前 | 否（扁平数组） | 检查输出并强制继续循环（`force_continue`）或终止 |
| **`Stop`** | Agent 准备结束本轮响应时 | 否（扁平数组） | 若后台测试还在跑或未满足验收条件，返回 `continue` 要求它继续干活 |

### 3. `hooks.json` 配置示例
```json
{
  "safety-gate": {
    "PreToolUse": [
      {
        "matcher": "run_command",
        "hooks": [
          {
            "type": "command",
            "command": "python ./scripts/check_cmd.py",
            "timeout": 10
          }
        ]
      }
    ]
  },
  "auto-format": {
    "PostToolUse": [
      {
        "matcher": "edit_file|write_file",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write .",
            "timeout": 20
          }
        ]
      }
    ]
  }
}
```

### 4. 输入输出契约核心要点
* Hook 脚本**从 `stdin` 读取 JSON**，**向 `stdout` 输出 JSON**。
* **⚠️ 所有 JSON 字段均使用驼峰命名（camelCase）**：如 `conversationId`、`workspacePaths`、`transcriptPath`、`toolCall`、`stepIdx`、`lastUserInput`。
* **`PreToolUse` 返回字段**：
  * `"decision"`（必填）：`"allow"`（自动放行）、`"deny"`（直接拦截报错）、`"ask"`（弹窗询问，受“总是允许”缓存影响）、`"force_ask"`（每次必弹窗，忽略缓存）。
  * `"overwrite"`（可选）：对工具入参做**浅层顶层覆盖合并**（Shallow Merge）。例如把原本的 `"CommandLine"` 替换成安全参数再真正执行。
* **`Stop` 返回字段**：返回 `{"decision": "continue", "reason": "单元测试尚未全部通过，请修复后再结束。"}` 即可强制把 Agent 抓回来继续干活。

### 5. 当前限制注意
- 目前 `type` 仅支持 `"command"`（本地 Shell 脚本，Unix 用 `sh -c`，Windows 用 `cmd /c`，工作目录为 `hooks.json` 所在目录，默认超时 `30` 秒）。
- Hooks 为**同步阻塞执行**，因此务必保持 Hook 脚本轻量快速，避免拖慢整个 Agent 响应节奏。
