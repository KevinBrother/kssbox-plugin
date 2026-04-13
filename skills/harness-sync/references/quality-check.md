# 质量检查：所有模式通用

任何模式执行完生成文件后，必须过以下检查。

---

## 内部一致性检查（最重要）

Harness 文件之间必须互相一致，否则 agent 会读到矛盾信息，输出不可预测。

**检查矩阵**：

| 检查项 | 文件 A | 文件 B | 一致性要求 |
|------|------|------|---------|
| 层级名称 | AGENTS.md | architecture.md | 完全相同，不能一个叫 "repo" 一个叫 "repository" |
| 禁止事项 | AGENTS.md | dependency-cruiser | 每条禁止事项都有对应的机械规则 |
| 目录结构 | conventions.md | architecture.md | 目录名和层级图一致 |
| 技术栈 | ADR-001 | ci.yml | CI 工具链和选定技术栈匹配 |
| 覆盖率要求 | AGENTS.md（测试要求） | ci.yml | 数字一致 |

---

## AGENTS.md 质量检查

- [ ] 禁止事项：每条都是"不得做X"格式，有具体原因，不是"写好代码"
- [ ] 禁止事项：至少 3 条，覆盖最常见的层级违规
- [ ] 不确定时怎么做：有明确的人工节点，不是"自行判断"
- [ ] 测试要求：有具体数字（覆盖率百分比）
- [ ] 已知常见错误：INIT 时可以为空，UPDATE/AUDIT 时必须有内容

---

## 依赖规则检查

- [ ] dependency-cruiser（或同类工具）的规则覆盖了所有"禁止"的跨层依赖
- [ ] 每条规则有 `comment` 字段说明为什么禁止
- [ ] 没有过于宽泛的规则（如 `from: {path: "src"}` 这种几乎禁止所有的规则）

---

## CI 检查

- [ ] 四类门禁全部存在：类型检查、linter、模块边界、覆盖率
- [ ] 所有门禁失败时 exit code 非零（阻断合并）
- [ ] 没有 `|| true`、`continue-on-error: true`、`--max-warnings` 被设成很大的数

---

## ADR 检查

- [ ] 技术栈选型（ADR-001）必须存在
- [ ] 每个重要的架构决策都有 ADR（不遗漏）
- [ ] ADR 状态字段已更新（不是全部停在"草稿"）

---

## 落盘前的文件完整性

- [ ] 所有文件无未替换的占位符（如 `[产品名]`、`[填写]`）
- [ ] 所有文件路径正确（符合上述目录结构）
- [ ] INIT 模式：六个文件全部生成
- [ ] UPDATE/PRD-SYNC 模式：变更摘要说明了哪些文件改了哪些没改
- [ ] AUDIT 模式：健康报告有具体问题描述和修复建议

---

## 输出自检报告格式

```
## Harness 文件生成完成

模式：[INIT / UPDATE / PRD-SYNC / AUDIT]

| 文件 | 状态 | 备注 |
|-----|-----|-----|
| AGENTS.md | ✅ 新建 | 5条禁止事项，覆盖全部层级违规 |
| docs/architecture.md | ✅ 新建 | 6层架构，依赖方向明确 |
| docs/conventions.md | ✅ 新建 | |
| docs/decisions/ADR-001-tech-stack.md | ✅ 新建 | |
| .github/workflows/ci.yml | ✅ 新建 | 4类门禁，均为阻断级别 |
| .dependency-cruiser.cjs | ✅ 新建 | 4条禁止规则，与 AGENTS.md 一一对应 |

内部一致性：✅ 通过（层级名称、禁止事项、覆盖率数字均一致）

下一步建议：
1. 将这些文件放入仓库根目录
2. 确认 CI pipeline 权限配置
3. 在 README 中加入"AI 开发者请先读 AGENTS.md"的提示
```
