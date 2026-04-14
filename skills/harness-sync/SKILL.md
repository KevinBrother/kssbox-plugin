---
name: harness-sync
description: 让项目的 Harness（AGENTS.md、架构文档、CI 配置、依赖规则）与项目实际状态保持同步。支持四种触发场景：新建项目时从 PRD 生成完整 Harness 文件集；功能迭代或重构后更新受影响的 Harness 文件；PRD 变更后将架构决策变化反哺到 Harness；定期检测 Harness 腐烂并修复。当用户说"帮我初始化 Harness"、"更新 AGENTS.md"、"新功能加了要更新架构文档"、"PRD 改了要同步"、"检查 Harness 是否过期"、"Harness 腐烂了"、"CI 规则需要更新"时，必须使用本 skill。
---

# Harness Sync Skill

## 什么是 Harness

Harness = 围绕 AI coding agent 的完整约束和引导系统，让 agent 输出可预测、可信任。

```
Agent = Model + Harness
Harness = Feedforward（导引）+ Feedback（传感）
```

- **Feedforward**：AGENTS.md、架构文档、命名约定 → 在 agent 行动前约束
- **Feedback**：linter、结构测试、CI 门禁 → 行动后自动检测并触发自我修正

Harness 不是一次性初始化，是随项目持续演进的活系统。

---

## 第一步：判断触发模式

读取用户输入，判断属于哪种场景，然后读对应的 reference 文件：

| 模式 | 触发信号 | 读取文件 |
|-----|--------|--------|
| 🆕 **INIT** 新建项目 | "初始化"、"新项目"、"建仓库"、有 PRD 没有现有 Harness | `references/mode-init.md` |
| 🔄 **UPDATE** 迭代更新 | "新功能"、"重构了"、"加了新模块"、"需求变了" | `references/mode-update.md` |
| 📋 **PRD-SYNC** PRD 变更 | "PRD 改了"、"需求调整"、"架构决策变了" | `references/mode-prd-sync.md` |
| 🔍 **AUDIT** 腾烂检测 | "检查 Harness"、"CI 开始误报"、"Harness 过期了"、"定期维护" | `references/mode-audit.md` |

**不确定时**：问用户一个问题——"你现在是在初始化新项目、更新已有 Harness、还是做定期检查？"

---

## Harness 文件集（所有模式的输出目标）

```
仓库根目录/
├── AGENTS.md                      ← AI agent 入职文档（最重要）
├── docs/
│   ├── architecture.md            ← 模块边界 + 依赖方向
│   ├── conventions.md             ← 命名规范 + 文件组织
│   └── decisions/
│       └── ADR-NNN-[topic].md     ← 架构决策记录
└── .github/
    └── workflows/
        └── ci.yml                 ← CI pipeline（含所有门禁）
```

依赖规则配置文件：根据项目已有技术栈放置对应位置，路径写入 `docs/architecture.md` 说明。

---

## 落盘与输出

所有生成/修改的文件：
- 直接写入项目仓库根目录对应路径（`AGENTS.md`、`docs/architecture.md` 等）
- 如果用户未指定仓库路径，先问清楚再写入
- 输出变更摘要：哪些文件是新建、哪些是更新、哪些无需改动

---

## 质量检查（所有模式执行完后都要过）

详见 `references/quality-check.md`

快速自检（输出前）：
- [ ] AGENTS.md 的禁止事项至少 3 条，且针对这个项目（不是通用废话）
- [ ] 分层架构依赖方向明确，且和 CI 依赖规则配置一致
- [ ] CI 配置里的四类检查都有：类型检查、linter、模块边界、覆盖率
- [ ] 所有文件互相不矛盾（AGENTS.md 说的层级 = architecture.md 画的层级 = dependency-cruiser 的规则）
- [ ] ADR 存在且记录了关键决策（至少技术栈选型）
