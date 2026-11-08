# 03. Antigravity CLI (`agy`) 与 Python SDK 实战

对于习惯留在终端里的开发者，或需要把 Agent 集成进自动化流水线的团队，Antigravity 提供了与桌面端共享底层引擎的 **CLI (`agy`)** 和 **Python SDK**。

---

## 一、Antigravity CLI (`agy`) 使用指南

### 1. 基础启动与退出
* **启动交互式终端 (TUI)**：在任意项目目录下运行
  ```bash
  agy
  ```
* **查看所有命令行参数与子命令**：
  ```bash
  agy --help
  ```
* **退出 CLI**：连按两次 `Ctrl + D`，或者输入 `/exit`、`/quit`。

### 2. CLI 配置文件路径
`agy` 的全局配置保存在：
```text
~/.gemini/antigravity-cli/settings.json
```

### 3. CLI TUI 内部斜杠命令速查
进入 `agy` 的终端交互界面后，输入 `/help` 可查看全部命令，最常用的包括：

| 类别 | 命令 | 说明 |
| :--- | :--- | :--- |
| **会话管理** | `/resume` | 列出并切换回之前的历史对话记录 |
| | `/rewind` 或 `/undo` | 将对话历史与状态回滚到之前的检查点 |
| | `/fork` 或 `/branch` | 从当前对话分叉出一条新分支，做平行实验 |
| | `/clear` | 清空当前对话重新开始 |
| **模型与监控** | `/model` | 切换当前使用的 Gemini 模型 |
| | `/effort <level>` | 调整模型思考/推理强度（如 `low`、`medium`、`high`） |
| | `/context` | 可视化查看当前 Token 占用与上下文窗口消耗 |
| | `/agents` | 打开子智能体监控面板，查看后台并发任务状态 |
| **配置与权限** | `/config` 或 `/settings` | 打开全屏设置菜单，调整工作区策略与界面样式 |
| | `/permissions` | 打开权限管理 TUI 面板，增删 `allow` / `deny` / `ask` 细粒度规则 |
| | `/plugin` | 管理插件、Hooks 钩子与 MCP 服务器 |

---

## 二、Antigravity Python SDK 编程接入

通过官方开源的 Python SDK（[google-antigravity/antigravity-sdk-python](https://github.com/google-antigravity/antigravity-sdk-python)），你可以在 Python 脚本中以代码形式创建、控制、监听 Agent。

### 1. 安装
```bash
pip install google-antigravity
```

### 2. 快速上手：异步流式调用 Agent

> ⚠️ **安全默认原则**：`LocalAgentConfig` 默认以**只读模式**运行。只有显式传入 `capabilities=CapabilitiesConfig()` 时，Agent 才会获得修改文件或执行命令等写权限工具。

```python
import asyncio
import sys
from google.antigravity import Agent, LocalAgentConfig, CapabilitiesConfig

async def main():
    config = LocalAgentConfig(
        system_instructions="你是一个熟悉代码库导航与分析的资深技术专家。",
        capabilities=CapabilitiesConfig(),  # 显式开启写操作工具（如 edit_file, run_command）
    )

    # 使用异步上下文管理器管理 Agent 生命周期
    async with Agent(config) as agent:
        response = await agent.chat("帮我写一个统计当前目录代码行数的 Python 脚本。")

        # 实时流式输出响应 Token
        async for token in response:
            sys.stdout.write(token)
            sys.stdout.flush()
        print()

if __name__ == "__main__":
    asyncio.run(main())
```

### 3. 进阶用法：实时拦截思考过程与工具调用

在编写自动化评测、监控面板或自定义流水线时，可以分别订阅模型的思考流（`thoughts`）和强类型工具调用流（`tool_calls`）：

```python
# 1. 实时监听 Agent 推理过程
async for thought in response.thoughts:
    print(f"[思考] {thought}")

# 2. 实时监听 Agent 触发了哪些工具及参数
async for call in response.tool_calls:
    print(f"[执行工具] {call.name} 参数: {call.args}")
```

### 4. 一行代码在终端拉起交互式循环

```python
import asyncio
from google.antigravity import Agent, LocalAgentConfig, CapabilitiesConfig
from google.antigravity.utils.interactive import run_interactive_loop

async def main():
    config = LocalAgentConfig(capabilities=CapabilitiesConfig())
    async with Agent(config) as agent:
        await run_interactive_loop(agent)

asyncio.run(main())
```

---

## 三、如何选择 CLI 还是 Python SDK？
- **日常在远程服务器写代码 / 修 Bug / 人机交互**：直接用 `agy`。
- **做批处理任务（如每天夜间自动跑 50 个仓库的代码审查）、嵌入后端服务或科研自动化流**：用 Python SDK，配合 `response.tool_calls` 可以精确审计每一步执行。
