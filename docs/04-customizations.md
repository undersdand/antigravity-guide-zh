# 04. 自定义扩展系统全解（Rules / Skills / Plugins）

Antigravity 之所以强大，在于它支持通过本地配置文件把你的**团队规范、专业技能和外部工具**注入给 Agent。理解这套扩展系统的**发现路径、加载优先级与上下文预算**，是从普通用户迈向高手的必经之路。

---

## 一、核心概念一图分清

| 扩展类型 | 典型文件名/目录 | 何时加载（生效方式） | 最适合放什么 |
| :--- | :--- | :--- | :--- |
| **Rules（规则）** | `GEMINI.md`、`AGENTS.md` | **默认常驻**（按目录层级向上自动加载） | 代码规范、技术栈版本约束、禁用 API、项目背景常识 |
| **Skills（技能）** | `skills/<name>/SKILL.md` | **按需渐进式披露**（平时只加载名称和简介，触发时才读全文） | 多步骤专业 SOP、脚手架脚本、排错手册、垂直领域知识 |
| **Plugins（插件）** | `plugins/<name>/plugin.json` | **打包分发**（随开关启用/禁用） | 把相关的 Skills、Rules、Hooks 和 MCP 打包成一体分享给团队 |

---

## 二、目录发现机制与加载优先级

当同一名称的 Skill 或配置在多个地方出现时，Antigravity 严格按照 **从高到低** 的优先级覆盖：

1. **工作区本地发现（最高优先级）**：从当前工作目录一路向上追溯到仓库根目录（`.git` 所在目录），查找所有 `.agents/`（也兼容 `.agent/`、`_agents/`、`_agent/`）以及沿途的 `GEMINI.md` / `AGENTS.md`。
2. **工作区显式声明配置**：项目 `.agents/skills.json` 或 `.agents/plugins.json` 中指向的路径。
3. **全局本机配置**：`~/.gemini/config/` 下的配置（对本机所有项目生效）。
4. **系统内置（Built-in）**：官方默认自带的内置技能与插件。
5. **全局 JSON 显式配置（最低优先级）**。

> ✅ **去重保障**：哪怕同一个 Rule 文件通过父目录继承等多条路径被扫描到，系统都会按解析后的真实文件路径自动去重，单轮对话绝不会重复注入。

---

## 三、Rules（规则）：`AGENTS.md` 与 `GEMINI.md` 实战与预算限制

### 1. 怎么写才生效？
- 在项目根目录或任意子目录下放置 `AGENTS.md` 或 `GEMINI.md`。
- **注意格式**：独立的 `GEMINI.md` / `AGENTS.md` **不支持** YAML Frontmatter 头信息，它们对所在目录及其所有子目录**始终生效**。
- 当你在子目录 `frontend/` 下修改文件时，Agent 会同时生效根目录的 `AGENTS.md` 和 `frontend/AGENTS.md`。

### 2. ⚠️ 极易忽略的大小与 Token 预算限制
很多用户把几万字文档全塞进 `AGENTS.md`，结果发现 AI 根本没按要求做，原因就在官方的硬性预算保护机制：
- **单文件上限 `24 KB`（24,000 字节）**：包含 `@[label](path)` 展开的内容在内，超过 24KB 的部分会被按行截断！
- **总规则预算 `20,000 Tokens`（`defaultRulesBudget`）**：所有常驻和全局规则共享 2 万 Token 的总预算（内置插件自带规则豁免不计入）。
- **超预算自动降级**：一旦超过总预算，超出的规则文件会从“全文内联注入”被降级为“只给一个文件路径指针”，AI 需要觉得相关时才会自己去读。

**最佳实践**：保持 `AGENTS.md` 精炼（建议控制在几百行以内），把大块的操作指南、部署手册拆成 **Skill**！

---

## 四、Skills（技能）：渐进式披露的艺术

Antigravity 为了不撑爆上下文窗口，对 Skill 采用 **Progressive Disclosure（渐进式披露）**：
- 对话开始时，系统**只把所有 Skill 的 `name` 和 `description` 放进提示词**。
- 只有当用户的请求匹配某个 Skill 的 `description` 时，Agent 才会自动去读取该 `SKILL.md` 的完整内容及附带文档。

### 1. 标准 Skill 目录结构
```text
skills/my-skill/
├── SKILL.md          # 必需：核心指令（带 YAML Frontmatter）
├── scripts/          # 可选：封装复杂命令的可执行脚本
├── references/       # 可选：大部头参考手册（需要时再读，省 Token）
├── resources/        # 可选：模板文件或静态资产
└── examples/         # 可选：标准输出范例
```

### 2. `SKILL.md` 编写黄金法则
文件顶部必须有 YAML Frontmatter：
```markdown
---
name: api-doc-generator
description: >-
  Generate OpenAPI 3.1 YAML documentation from FastAPI or Express route files.
  Use when the user asks to document API endpoints, export Swagger specs, or audit route schemas.
---

# API Documentation Generator

## Steps
1. Run the extraction script: [extract.py](./scripts/extract.py)
2. Verify schema validity against [references/openapi-rules.md](./references/openapi-rules.md)
```
**四个核心技巧**：
1. **`description` 是灵魂**：用第三人称清晰写出 **“这个技能做什么”** 和 **“什么时候必须触发它”**。
2. **大文档放 `references/`**：主 `SKILL.md` 保持精简，大段规范放进 `references/` 并用相对路径链接。
3. **复杂操作写成脚本放 `scripts/`**：能用确定性 Python/Shell 脚本一步跑完的动作，不要让模型每次临场拼一长串命令。
4. **必须包含自检步骤**：告诉 Agent 每步做完如何验证成功（例如检查输出文件、跑 dry-run）。

---

## 五、JSON 配置索引：如何跨目录与团队共享？

如果你不想把别人写好的 Skill 复制粘贴到每个项目，可以用 `.agents/skills.json` 或 `.agents/plugins.json` 挂载外部目录：

```json
{
  "inherits": [
    {
      "path": "/path/to/company-shared/skills.json",
      "include_only": ["security-audit", "db-migration"],
      "exclude": ["deprecated-.*"]
    }
  ],
  "entries": [
    {
      "path": "tools/agent-skills",
      "exclude": ["experimental-.*"]
    },
    {
      "path": "~/my-personal-skills"
    }
  ]
}
```
* **路径解析规则**：`/` 开头为绝对路径；`~/` 开头相对用户主目录；其他相对路径基于仓库根目录解析。
* **扫描深度**：每个 `entries` 目录仅向下一层扫描（即 `<path>/<skill-name>/SKILL.md`）。

---

## 六、Plugins（插件）：一键打包分发全套能力

当你想把 **Skills + Rules + Hooks + MCP 服务器** 一起打包发给同事或开源到 GitHub 时，做成 **Plugin** 是最规范的方式：

```text
plugins/team-dev-kit/
├── plugin.json       # 必需：插件清单（至少包含 {"name": "..."}）
├── mcp_config.json   # 可选：插件自带的 MCP 服务器
├── hooks.json        # 可选：插件自带的生命周期钩子（必须放在插件根目录！）
├── rules/
│   └── AGENTS.md     # 可选：插件启用时自动合并的规则
└── skills/
    └── my-skill/
        └── SKILL.md
```

> ⚠️ **原生插件踩坑提示**：原生 Antigravity 插件的 `hooks.json` **必须直接放在 `<plugin_dir>/hooks.json`** 根目录；只有通过 `agy plugin install` 导入旧版 Gemini CLI 或 Claude Code 扩展时，系统才会去转换 `hooks/hooks.json`。

### 如何开关某个插件？
插件可以在 `plugin.json` 中声明 `"disabled": true` 默认关闭；而用户自己在设置 UI 或通过 CLI（`plugin enable` / `plugin disable`）做出的开关决定，会保存在 `config.json` 中（且**用户偏好永远优先于插件默认值**，更新或重装插件也不会被覆盖）。
