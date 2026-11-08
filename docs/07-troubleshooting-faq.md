# 07. 高频踩坑与排错指南 (FAQ)

这一章汇总了在实际使用 Antigravity 2.0（尤其是 Windows 环境与深度自定义时）最容易踩中的坑与真实解法。

---

### Q1：Windows 下让 Agent 在终端运行脚本打开浏览器/软件，命令显示成功，屏幕上却什么都没弹出来？
* **真实原因**：你开启了 **Terminal Sandbox（终端沙盒）**。在 Windows 上，沙盒内的命令运行在一个隔离的隐藏桌面会话（如 `exebox-...`）中。脚本确实启动了程序，但窗口被画在了你看不见的沙盒桌面里，无法穿透到主屏幕！
* **解决办法**：
  1. **方式 A（使用宿主级工具）**：不要靠终端命令去“弹出窗口”，改用 **MCP 服务器**（如官方 `chrome-devtools-plugin`、`firefox-devtools-mcp`、`browser-control-mcp`）或内置 `/browser`。MCP 由 Antigravity 主进程拉起，不受终端沙盒桌面隔离影响。
  2. **方式 B（关闭沙盒/切换策略）**：在桌面端 **Settings** 中暂时关闭 Terminal Sandbox，或将执行策略调整为需要审批的非沙盒模式（注意安全性）。

---

### Q2：接入第三方远程 MCP 服务时，填了 `/sse` 地址却死活连不上？
* **真实原因**：Antigravity 遵循 MCP 新版 **Streamable HTTP** 规范，**不支持**旧版 HTTP+SSE（2024-11-05 规范）的 `/sse` 传输。
* **解决办法**：
  - 首先确认服务端是否提供了 `/mcp` 端点，如有则直接把 `url` 改为 `/mcp`。
  - 如果对方只提供 `/sse`，请在 `mcp_config.json` 里用 `mcp-remote` 作为 Stdio 桥接（具体配置见 [第 05 章](./05-mcp-and-browser.md)）。

---

### Q3：我在 `AGENTS.md` / `GEMINI.md` 写了很长的项目规范，为什么 AI 经常“失忆”或漏掉后面几条？
* **真实原因**：触发了规则预算限制！
  - 单个规则文件超过 **24 KB（24,000 字节）** 会被截断。
  - 所有常驻规则总预算超过 **20,000 Tokens** 时，多出的文件会被降级为“仅提供路径指针”，不会全文注入上下文。
* **解决办法**：把庞大的参考手册、详细流程说明从 `AGENTS.md` 拆出来，做成 **Skill**（放在 `.agents/skills/<name>/SKILL.md` 及 `references/` 下），只在必要时按需加载。

---

### Q4：写了 `SKILL.md` 放在 `.agents/skills/` 里，但对话时 Agent 从来不触发它？
* **常见原因排查**：
  1. **目录层级不对**：必须是 `<root>/skills/<skill-name>/SKILL.md`（扫描深度严格为一层，不能再嵌套多余目录或把 `.md` 直接扔在 `skills/` 根目录）。
  2. **缺少 YAML Frontmatter 或 `description` 写得太含糊**：Agent 只靠 `name` 和 `description` 来决定是否读取你的 Skill。请在 `description` 中明确写出触发场景与关键词（例如：`"Use when the user asks to..."`）。
  3. **当前没有激活工作区**：如果项目级的 `.agents/` 不在当前打开的 Workspace 路径内，Agent 扫描不到它。把常用 Skill 放进全局目录 `~/.gemini/config/skills/` 即可跨所有会话生效。

---

### Q5：自己打包了 Antigravity 原生插件，放在 `hooks/hooks.json` 的钩子为什么没运行？
* **真实原因**：路径规范差异。原生 Antigravity 插件（目录含 `plugin.json`）**严格从插件根目录加载 `<plugin_dir>/hooks.json`**，不会去扫描 `hooks/hooks.json` 子目录。只有通过 `agy plugin install` 导入纯 Gemini CLI 或 Claude Code 格式的扩展时才会自动中转路径。
* **解决办法**：直接把 `hooks.json` 移动到插件根目录。

---

### Q6：如何避免复杂任务中途遇到 Rate Limit（请求频率限制）或上下文爆掉？
* **拆分与委派**：遇到大范围代码检索时，让主 Agent 启动只读子智能体（Subagent）分担上下文，不要在主会话贴满几千行文件。
* **及时压缩**：对话超过 20~30 轮且已切换子话题时，输入 `/compact` 总结压缩历史上下文，或使用 `/fork`（CLI）另起分支实验。
* **选对思考档位**：简单的文件重命名、常规模板生成切到 Flash 系列或较低思考强度（`Low` / `Medium`），把 `High` 推理留给复杂架构设计与疑难排查（`/boost`）。
