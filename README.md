# 🚀 Google Antigravity 中文完全指南

> **从零上手到深度定制：覆盖 Antigravity 2.0 桌面端、IDE、CLI (`agy`)、Python SDK、自定义扩展（Rules / Skills / Plugins / Hooks / MCP）与实战避坑。**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](./LICENSE)
[![Platform](https://img.shields.io/badge/Platform-Windows%20%7C%20macOS%20%7C%20Linux-brightgreen.svg)](#)
[![Docs](https://img.shields.io/badge/Target-Antigravity%202.0%2B-orange.svg)](https://antigravity.google/docs)

---

## 💡 为什么写这份指南？

目前开源社区中针对 Google Antigravity 的中文项目主要集中在**界面汉化补丁**或**零散的提示词收集**，缺少一份系统化、跟进 **Antigravity 2.0** 最新架构的实战手册。

本仓库致力于提供**一处就能查全**的中文实践参考：
- 🧭 **全景架构梳理**：讲透 Antigravity 2.0 桌面指挥中心、IDE、CLI (`agy`)、Python SDK 四大入口的定位与配合方式。
- 🧩 **扩展系统深潜**：完整覆盖 `AGENTS.md` / `GEMINI.md` 规则、渐进式披露 `SKILL.md`、插件打包 `plugin.json`、生命周期钩子 `hooks.json` 与 `MCP` 接入规范。
- 🌐 **全浏览器与外部生态实战**：不仅讲官方 Chrome / Browserbase 方案，还专门整理了 **Firefox（火狐浏览器）MCP 接入方案**。
- 🚧 **真实踩坑记录**：涵盖 Windows 终端沙盒（`exebox` 隔离桌面导致 GUI 窗口不显示）、MCP 旧版 SSE 协议不兼容、规则 Token 预算降级等官方文档容易略过的细节。
- 📦 **开箱即用模板库**：内置 [`templates/`](./templates) 目录，复制即用。

---

## 📚 目录导航

| 章节 | 核心内容 | 适合人群 |
| :--- | :--- | :--- |
| **[01. 快速入门与四大客户端](./docs/01-getting-started.md)** | 平台演进、App 2.0 / IDE / CLI / SDK 选型对比、安装与统一认证 | 新手必读 |
| **[02. 桌面端 2.0 与 IDE 高效工作流](./docs/02-app-and-ide-workflow.md)** | Slash 命令大全、`@` 上下文引用、Subagent 并行分身、定时任务、IDE 三大模式 | 日常使用者 |
| **[03. CLI (`agy`) 与 Python SDK 实战](./docs/03-cli-and-sdk.md)** | `agy` 终端交互命令、配置文件路径、`google-antigravity` Python SDK 编程调用 | 极客 / 自动化开发 |
| **[04. 自定义扩展系统（Rules / Skills / Plugins）](./docs/04-customizations.md)** | 目录发现机制、加载优先级、规则 24KB/20K Token 预算、编写高质量 Skill 与打包插件 | 进阶定制 |
| **[05. MCP 服务器与浏览器自动化（含 Firefox 方案）](./docs/05-mcp-and-browser.md)** | `mcp_config.json` 配置、Streamable HTTP 与 `mcp-remote` 桥接、Chrome 与 Firefox 控制对比 | 工具链扩展 |
| **[06. 权限引擎、终端沙盒与 Hooks 安全拦截](./docs/06-hooks-and-security.md)** | 四档执行策略、OS 级 Terminal Sandbox 原理、`hooks.json` 六大生命周期事件与契约 | 团队安全规范 |
| **[07. 高频踩坑与排错指南 (FAQ)](./docs/07-troubleshooting-faq.md)** | Windows 沙盒弹窗不可见、Firefox 连接、自定义不生效、Hook 参数重写注意事项 | 遇到问题时查阅 |

---

## 📦 开箱即用模板库 ([`templates/`](./templates))

* 📄 [`templates/AGENTS.md`](./templates/AGENTS.md) —— 适用于中文团队的项目级规范规则模板
* 🛠️ [`templates/skills/my-first-skill/SKILL.md`](./templates/skills/my-first-skill/SKILL.md) —— 符合官方最佳实践的 Skill 骨架模板
* 🔌 [`templates/mcp_config.json`](./templates/mcp_config.json) —— MCP 服务器配置示例（含 Stdio、Streamable HTTP 与 Firefox MCP）
* 🛡️ [`templates/hooks.json`](./templates/hooks.json) —— 工具调用拦截与安全审查 Hook 模板
* 📁 [`templates/skills.json`](./templates/skills.json) —— 团队通过 Git 仓库共享 Skill 的声明配置模板

---

## 🔗 相关官方与优质社区资源

### 官方入口
* 🌐 **官方主页与文档**：[antigravity.google/docs](https://antigravity.google/docs)
* 📰 **更新日志**：[antigravity.google/changelog](https://antigravity.google/changelog)
* 💻 **官方 CLI 仓库**：[google-antigravity/antigravity-cli](https://github.com/google-antigravity/antigravity-cli)
* 🐍 **官方 Python SDK**：[google-antigravity/antigravity-sdk-python](https://github.com/google-antigravity/antigravity-sdk-python)

### 社区生态补充（按定位各取所需）
* 🎨 **客户端界面汉化补丁**：如果你希望把软件界面菜单直接汉化，可参考 [iMissNan/antigravity-chinese](https://github.com/iMissNan/antigravity-chinese) 或 [cshitian/antigravity_chinese](https://github.com/cshitian/antigravity_chinese)
* 🌟 **英文社区技能库**：可通过 `npx antigravity-awesome-skills` 探索社区开源技能集合
* 📑 **社区资源导航**：[ZhangYu-zjut/awesome-Antigravity](https://github.com/ZhangYu-zjut/awesome-Antigravity)

---

## 🤝 参与贡献

欢迎提交 Issue 或 Pull Request 补充新的使用技巧、MCP 推荐或平台踩坑记录！

如果这份指南帮到了你，欢迎点个 **Star ⭐** 支持一下！

## 📄 开源协议

本项目基于 [MIT License](./LICENSE) 开源。
