# 01. 快速入门与四大客户端

Google Antigravity 已经从早期的 AI 编程编辑器演进为一套 **以 Agent 为核心（Agent-First）的开发与任务编排平台**（Antigravity 2.0+）。整个生态由**四大入口（Surfaces）**组成，底层共享同一套模型能力、权限引擎与自定义配置。

---

## 一、四大客户端怎么选？

| 客户端形态 | 定位与特点 | 典型适用场景 |
| :--- | :--- | :--- |
| **1. Antigravity 2.0 桌面端 (App)** | 独立的 Electron 桌面指挥中心，不绑定单一代码编辑器，强调多 Agent 调度、后台任务与定时自动化 | 跨项目工作、后台长任务、定时自动巡检、日常科研/办公/全栈任务协同 |
| **2. Antigravity IDE** | 基于 VS Code 构建的 AI 原生集成开发环境，内置代码补全、行内指令与可视化 Diff 面板 | 深度编码、逐行重构、阅读大型仓库源码、依赖编辑器内联提示 |
| **3. Antigravity CLI (`agy`)** | 轻量级终端命令行工具，支持 TUI 交互界面与非交互脚本模式 | SSH 远程服务器、纯终端极客工作流、CI/CD 脚本或无桌面环境 |
| **4. Antigravity Python SDK** | 编程接口包（`pip install google-antigravity`），可直接在 Python 代码里拉起和控制 Agent | 搭建自己的自动化流水线、批量评测脚本、将 Agent 嵌入业务系统 |

> 💡 **补充说明**：除了官方独立桌面端和 IDE，官方还提供了适用于 **VS Code、JetBrains 全家桶（IntelliJ / PyCharm / WebStorm 等）与 Zed** 的 IDE 插件，均可在对应插件市场搜索安装。

---

## 二、安装与认证

### 1. 下载安装
1. 访问官方门户网站：**[https://antigravity.google](https://antigravity.google)**
2. 根据操作系统（Windows x64、macOS Apple Silicon/Intel、Linux）下载对应安装包：
   - **Antigravity 2.0 App**：独立安装即用。
   - **Antigravity IDE**：完整代码编辑器。
3. 如需使用 Python SDK：
   ```bash
   pip install google-antigravity
   ```
   > ⚠️ **注意**：Python SDK 依赖随 PyPI Wheel 包分发的本地编译运行时二进制文件，务必通过 `pip` 安装以确保二进制组件完整。

### 2. 统一登录认证（Unified Authentication）
Antigravity 支持一次登录、跨端共享凭证：
- **个人 Google 账号**（支持免费额度体验）
- **Gemini Enterprise** 企业账号
- **Google Cloud 账号（ADC 凭据）**

在桌面端或 IDE 首次打开按提示完成 Google 账号授权即可；CLI 端在首次运行 `agy` 时同样会引导完成登录认证。

---

## 三、目录与配置体系一眼看懂

不管你使用哪个端，记住这两个最核心的配置目录就够了：

1. **项目级配置目录（跟随代码仓库，推荐团队共享）**
   ```text
   <你的项目根目录>/
   ├── AGENTS.md / GEMINI.md   # 项目规则（进入项目即自动生效）
   └── .agents/                # 也支持 .agent/、_agents/、_agent/
       ├── rules/              # 分模块规则 (*.md)
       ├── skills/             # 项目专属技能 (<skill-name>/SKILL.md)
       ├── plugins/            # 项目插件
       ├── hooks.json          # 生命周期钩子
       ├── skills.json         # 声明引入外部路径的 Skills
       └── plugins.json        # 声明引入外部路径的 Plugins
   ```

2. **全局配置目录（仅本机有效，跨所有项目生效）**
   - **自定义扩展全局根目录**：`~/.gemini/config/`
     - 全局 Skills：`~/.gemini/config/skills/`
     - 全局 MCP 配置：`~/.gemini/config/mcp_config.json`
     - 全局插件配置：`~/.gemini/config/plugins/`
   - **CLI 专属设置文件**：`~/.gemini/antigravity-cli/settings.json`

   *(Windows 用户中 `~` 代表 `C:\Users\<你的用户名>`)*

---

## 四、首次使用：没有工作区怎么办？

在 Antigravity 2.0 桌面端中，**就算你还没有打开任何项目文件夹，也能直接聊天或让它帮你写代码**：
- 当你没有激活工作区（Active Workspace）时，建议让 Agent 在默认临时工程目录（如 `~/.gemini/antigravity/scratch/<项目名>`）下创建子目录，或者在左侧边栏 **Projects** 中直接打开本地文件夹作为工作区。
- 关联工作区后，Agent 会自动发现项目内的 `.agents/` 与 `AGENTS.md`，并把项目级的权限设置（Project-Level Settings）覆盖全局默认设置。
