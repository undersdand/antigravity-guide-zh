# 02. 桌面端 2.0 与 IDE 高效工作流

这一章整理了 **Antigravity 2.0 桌面端** 和 **Antigravity IDE** 中最常用、也最能拉开效率差距的核心功能。

---

## 一、Antigravity 2.0 桌面端界面结构

Antigravity 2.0 采用双栏/三栏协同布局，核心分为三大区域：

1. **左侧导航栏（Left-hand Sidebar）**
   - **New Conversation**：开启新对话（每个对话拥有独立的上下文与任务轨迹）。
   - **Projects**：切换、管理不同本地代码仓库或工作区。
   - **Scheduled Tasks（定时任务）**：查看、管理 Cron 定时循环任务与一次性延迟提醒任务，状态分为 *Scheduled / Always on / Paused*，带完整运行日志。
   - **Skills & Customizations**：直观查看当前已加载的 Skills、Rules、Plugins 和 MCP 服务器。
   - **Settings**：配置模型、思考深度（Low / Medium / High）、工具执行策略与安全沙盒。

2. **主对话画布（Chat Canvas）**
   - **`/` 斜杠命令**：输入 `/` 呼出工作流快捷入口。
   - **`@` 上下文引用**：输入 `@` 可精准挂载文件、文件夹、历史对话、终端会话、Rules 规则或指定的 MCP 工具。
   - **多媒体输入**：支持直接拖入/粘贴图片、PDF、音频（如 MP3/WAV）及 Jupyter Notebook（`.ipynb`），甚至支持在 UI 截图中框选特定区域留言批注。

3. **辅助面板（HTML Auxiliary Pane）**
   - 集中展示 **Subagents（子智能体）**、**Background Tasks（后台命令）**、**Artifacts（产物文档/计划）**、**Files Changed（文件改动 Git Diff）** 以及 **Terminals（终端输出）**。

---

## 二、2.0 桌面端斜杠命令（Slash Commands）速查

在对话输入框输入 `/` 即可选择以下内置高频工作流（注意：它们与 CLI 终端里的 `/config` 等控制命令有所区别）：

| 命令 | 核心作用 | 什么时候用最合适 |
| :--- | :--- | :--- |
| **`/plan`** | 进入分步规划模式，先输出可执行计划让你审阅再动手 | 需求复杂、跨多文件重构，怕 AI 一上来就改乱代码 |
| **`/grill-me`** | 交互式“拷问”对齐模式，通过多轮追问帮你理清模糊需求与设计决策 | 想法还很模糊，想让 AI 扮演资深架构师反向帮你收敛方案 |
| **`/boost`** | 深度思考、多视角规划与严格验证模式 | 攻克高难度 Bug、复杂算法或严谨的架构研究 |
| **`/teamwork`** | 组织多个自主子智能体团队协同作业 | 大型项目拆解，前端、后端、测试同时推进 |
| **`/goal`** | 开启超长程自主闭环模式，彻底达成目标前不轻易停下 | 需要过夜跑实验、批量处理或长链路排查的任务 |
| **`/schedule`** | 创建定时循环任务（Cron）或单次倒计时提醒 | 每日早报摘要、定期巡检网站/仓库更新 |
| **`/browser`** | 启动网页浏览与交互工作流 | 需要打开网页、搜索资料或与 Web 应用交互 |
| **`/learn`** | 当你纠正了 AI 或摸索出一套复杂配置后，将其固化为持久化规则/技能 | 避免下次开新对话再重复踩同一个坑 |
| **`/compact`** | 压缩、总结当前过长的对话上下文，释放上下文窗口 | 对话几十轮后变慢或快触及上下文上限时 |

---

## 三、Subagents（子智能体）与后台并行机制

Antigravity 2.0 最强的设计之一，是主 Agent 不会把所有脏活累活都塞进自己的上下文里，而是会**动态拉起 Subagents**：

- **上下文隔离**：派一个只读 `research` 子智能体去翻遍整个代码库或查几篇论文，它只需要把提炼好的结论汇报给主 Agent，主会话完全不会被几十个文件源码撑爆。
- **异步响应式唤醒（Reactive Wakeup）**：不管是子智能体还是后台耗时终端命令，运行期间主界面**不会卡死**，系统会在后台任务完成时自动唤醒主 Agent 汇报结果。
- **并行独立工作区**：主 Agent 创建子智能体时，可以选择 `inherit`（共用工作区）、`branch`（新建隔离分支副本）或 `share`（类似 `git worktree` 共享仓库目录但独立分支），多人并行改代码互不打架。

---

## 四、Antigravity IDE 三大核心交互模态

如果你同时使用 **Antigravity IDE**（基于 VS Code 的完整代码编辑器），它额外提供编辑器内原生的三大交互方式：

1. **被动预测：Antigravity Tab（代码补全与跳转）**
   - **Autocomplete & Supercomplete**：不仅能补全光标后的代码，还能在浮窗中预测整段增删改 Diff。
   - **Tab to Jump**：预测你下一步最可能编辑的位置，按 `Tab` 直接跳过去。
   - **Tab to Import**：使用了新模块时自动在文件顶部补齐 `import`。
   - **快捷键**：`Tab` 全部采纳，`Esc` 取消，`⌘ + →`（macOS）/ `Ctrl + →`（Windows/Linux）逐词采纳。

2. **精准指令：Inline Command（`⌘ + I` / `Ctrl + I`）**
   - 选中一段代码按快捷键，输入修改要求（如“转成异步写法”、“补充边界校验”），AI 只改选区，不碰其他文件；不选代码直接按快捷键则在光标处生成新代码。

3. **深度协作：Sidebar Chat + 内联能力**
   - **Inline Code Lenses**：函数/类上方直接出现 `Refactor`、`Write Tests`、`Explain Code` 等快捷按钮。
   - **Visual Diff Overlays**：编辑器内直接高亮红绿修改对比，支持逐块 Accept / Reject。
   - **Diagnostic Auto-Fix**：遇到编译报错或 Lint 红线时，直接从错误提示一键触发 Agent 自动修复。

---

## 五、实用工作流小技巧

* **Artifact 计划审阅**：遇到大改动时，先让 Agent 生成 Markdown 计划 Artifact，上面带有反馈按钮（`Proceed`），改满意了再让它批量执行。
* **Remote Agents（云端远程代理）**：支持将任务放到云端项目运行（通过 `antigravity.google.com/remote` 访问管理）。
* **Voice Mode**：新版桌面端集成 Gemini Live 语音交互，适合边口述思路边让 Agent 拆解任务。
