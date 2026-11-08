---
name: my-first-skill
description: >-
  Standard starter template for a custom Antigravity skill.
  Use when demonstrating how to structure SKILL.md instructions, trigger descriptions,
  and verification steps.
---

# My First Custom Skill

请将此目录复制到 `.agents/skills/<your-skill-name>/`（项目级）或 `~/.gemini/config/skills/<your-skill-name>/`（全局），并替换为你自己的工作流规范。

## 执行步骤（Steps）

1. **前置检查**：确认工作目录与前置依赖已就绪。
2. **核心处理**：
   - 将复杂、固定的操作写进 `./scripts/` 下的脚本，不要每次现场拼长命令。
   - 如需加载大段背景规范，放在 `./references/` 并在此处通过相对链接引用，以节省 Token。
3. **结果校验（Validation）**：
   - 列出产出物路径。
   - 运行验证命令或检查输出日志，确认无报错后再汇报完成。
