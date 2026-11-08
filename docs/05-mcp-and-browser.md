# 05. MCP 服务器与浏览器自动化（含 Firefox 方案）

**MCP (Model Context Protocol)** 让 Antigravity 能够安全地连接本地数据库、企业内部系统、浏览器或任何第三方 API。

---

## 一、配置 MCP 服务器（`mcp_config.json`）

### 1. 配置文件放哪里？
- **全局生效**：`~/.gemini/config/mcp_config.json`
- **随插件生效**：`plugins/<plugin_name>/mcp_config.json`（仅在插件启用时加载，且工具会自动命名空间隔离防冲突）
- **在界面查看状态**：点击聊天区右上角或侧边栏的 **`Additional Options (...) > MCP Servers`**，即可检查当前连接的 MCP 服务与工具列表。

### 2. 支持的两种传输协议与格式

Antigravity 支持 **Stdio（本地子进程）** 和 **Streamable HTTP（远程服务）** 两种方式：

```json
{
  "mcpServers": {
    "sqlite-helper": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-sqlite", "/path/to/db.sqlite"],
      "env": {
        "DB_READONLY": "true"
      }
    },
    "remote-team-mcp": {
      "url": "https://mcp.example.com/mcp"
    }
  }
}
```

> ⚠️ **重点避坑：旧版 HTTP+SSE 协议已不再支持！**
> Antigravity 仅支持 MCP 规范（2025-03-26）定义的 **Streamable HTTP**（端点通常为 `/mcp` 而非 `/sse`）。如果你要接入的远程服务目前只提供旧版 HTTP+SSE（`/sse`），请不要直接填 `url`，而是改用 `mcp-remote` 作为本地 Stdio 桥接器：
> ```json
> {
>   "mcpServers": {
>     "legacy-sse-bridge": {
>       "command": "npx",
>       "args": ["-y", "mcp-remote", "https://legacy-server.example.com/sse"]
>     }
>   }
> }
> ```

---

## 二、浏览器自动化：Chrome 与 Firefox 完整对照

很多新手想让 Antigravity“打开某个网站、自动点按钮、查资料或截图测试页面”。目前主要有以下几条路线：

### 1. 官方与市场内置方案（针对 Chrome / 云端浏览器）
- **内置 `/browser` 工作流**：对话中输入 `/browser` 触发内置浏览流程。
- **Chrome DevTools 插件**（市场 ID：`antigravity-plugins-official/plugin/chrome-devtools-plugin`）：基于 Puppeteer 与 Chrome DevTools Protocol (CDP)，在本地 Chrome 进行点击、自动化与性能调试。
- **Browserbase 插件**（市场 ID：`antigravity-plugins-partners/plugin/browse`）：提供本地或云端浏览器操作。

> **权限说明**：在设置中可通过 **Browser Allowlist** 限制 Agent 仅能访问特定域名，防止意外跳转风险。

### 2. 🦊 如果你的主力浏览器是 Firefox（火狐）怎么办？
官方内置插件主要面向 Chrome / Chromium。**如果你日常用 Firefox，可以通过配置开源 MCP 服务器直接接管火狐**（在 `mcp_config.json` 中配置，由 Antigravity 宿主直接拉起，**不会受终端沙盒窗口隐藏影响**）：

| 开源项目 | 接入原理 | 是否能用你日常的火狐登录态 | 最适合的场景 |
| :--- | :--- | :---: | :--- |
| **[`eyalzh/browser-control-mcp`](https://github.com/eyalzh/browser-control-mcp)** | 本地 MCP Server + Firefox 扩展 | ✅ **完美支持**（扩展就在你火狐里） | 日常使用：开关/整理标签页、读历史记录、按域名授权后提取网页文本（安全性高） |
| **[`mozilla/firefox-devtools-mcp`](https://github.com/mozilla/firefox-devtools-mcp)**（Mozilla 官方） | WebDriver BiDi 原生双向协议 | ⚠️ 默认开新窗口，支持 `--connect-existing` 连已有实例 | 功能最全：点击、填表、上传文件、读控制台错误与网络请求（需 Node.js v20.19+） |
| **`@playwright/mcp --browser firefox`**（微软） | Playwright Firefox 内核 | ❌ 独立干净环境，无日常 Cookie | 端到端 UI 自动化测试、干净环境复现问题 |

#### 示例：在 `~/.gemini/config/mcp_config.json` 中接入 Mozilla 官方 Firefox MCP
```json
{
  "mcpServers": {
    "firefox-devtools": {
      "command": "npx",
      "args": [
        "-y",
        "@mozilla/firefox-devtools-mcp@latest"
      ]
    }
  }
}
```
配置保存并重启 Antigravity 后，在 **MCP Servers** 面板即可看到 Firefox 相关工具已就绪。

> 🔒 **安全提醒**：不要轻易让自动化工具连接存放了网银、核心生产后台 Cookie 的主 Firefox 配置文件；如果是做复杂点击测试，建议新建一个专用 Firefox Profile，或使用带“按域名逐一授权”机制的扩展版 MCP。
