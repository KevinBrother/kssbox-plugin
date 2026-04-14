# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Skills 架构与执行规则

每个 skill **独立运行、互相解耦**，可单独触发使用。

结构：

```text
skill-name/
├── SKILL.md              # 主说明文件（含触发条件、执行流程）
└── references/           # 详细指南（阶段执行前必须先读取）
```

**核心执行原则**：

- 每个 skill 有明确的阶段流程，需严格按顺序执行
- 阶段开始前**必须先读取**对应的 reference 文件，再执行
- 多数 skill 有"落盘 + 自检"环节，不可跳过
- 文档不落盘、不自检，不算完成

**Skill 触发机制**：SKILL.md 头部的 YAML frontmatter 定义 `description`，Claude Code 根据关键词自动触发。

## Harness 概念

`harness-sync` skill 中的核心设计：

```text
Agent = Model + Harness
Harness = Feedforward（导引）+ Feedback（传感）
```

- **Feedforward**：AGENTS.md、架构文档、命名约定 → 行动前约束
- **Feedback**：linter、结构测试、CI 门禁 → 行动后检测并触发修正

Harness 文件集：`AGENTS.md`（最重要）、`docs/architecture.md`、`docs/conventions.md`、`docs/decisions/ADR-*.md`、`.github/workflows/ci.yml`

## Hooks 机制

`hooks/hooks.json` 定义 SessionStart hook，会话启动时自动执行 `session-start` 脚本，将 skills 目录软链接到 `~/.claude/skills/`。

## 开发操作

- **新增 skill**：在 `skills/` 下创建目录，包含 `SKILL.md` 和 `references/`，头部添加 YAML frontmatter
- **修改 skill**：直接编辑 Markdown 文件，无构建步骤
- **排除 skill**：在 `bin/kssbox` 的 `EXCLUDE` 数组添加名称
- **测试 skill**：`claude --plugin-dir ./kssbox-plugin`