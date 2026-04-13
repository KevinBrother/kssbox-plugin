# 模式：UPDATE 迭代更新

功能迭代、重构、新增模块后，更新受影响的 Harness 文件。

## 触发场景

- 新增了一个功能模块（新的 Service、新的数据库表、新的 API）
- 重构了某层的结构（如把 repository 层拆成多个子模块）
- 发现了新的 agent 常见错误，需要加进 AGENTS.md
- 第三方依赖变化（新增/替换集成）
- 团队约定了新的命名规范

## 执行流程

### 第一步：理解变更范围

问用户（或从提供的信息里提取）：
1. 变更了什么？（新增/修改/删除了什么模块/功能）
2. 是否涉及新的依赖关系？（新模块依赖谁，被谁依赖）
3. 是否有新的禁止事项需要记录？

### 第二步：影响分析

根据变更内容，判断哪些 Harness 文件需要更新：

| 变更类型 | 需要更新的文件 |
|--------|------------|
| 新增模块/层级 | architecture.md、AGENTS.md、dependency-cruiser 规则、CI（如需新测试类型） |
| 重构层级结构 | architecture.md、AGENTS.md、dependency-cruiser 规则 |
| 新命名规范 | conventions.md、AGENTS.md |
| 新禁止事项 | AGENTS.md（禁止事项节）、dependency-cruiser（如可机械执行） |
| 新第三方集成 | architecture.md（外部依赖节）、ADR（新增一条）、AGENTS.md |
| 架构重大决策变更 | ADR（新增一条）、所有受影响文件 |

### 第三步：生成 diff

不要重写整个文件，只输出**需要变更的部分**，格式：

```
文件：docs/architecture.md
变更类型：新增内容
位置：模块层级图 → 在 service 层后新增

+ notification/           ← 新增：通知服务模块
+   notification.service.ts
+   email.adapter.ts      ← 适配器模式，隔离第三方邮件 SDK
```

```
文件：AGENTS.md
变更类型：新增禁止事项

禁止事项第 5 条（新增）：
+ 5. 不得在 Service 层直接调用邮件 SDK（如 Resend），
+    必须通过 notification/email.adapter.ts 访问，
+    以便未来切换邮件服务商时只改适配器。
```

### 第四步：同步检查

更新完成后，检查内部一致性：
- [ ] AGENTS.md 的禁止事项 ↔ dependency-cruiser 规则（说禁止的，都有规则执行）
- [ ] architecture.md 的层级 ↔ AGENTS.md 的模块职责速查（描述一致）
- [ ] 新增的 ADR ↔ AGENTS.md 的项目概述（如有技术变更）

## 输出格式

```
## Harness 更新摘要

**变更原因**：[一句话]

| 文件 | 操作 | 变更内容摘要 |
|-----|-----|-----------|
| AGENTS.md | 更新 | 新增禁止事项第5条 |
| docs/architecture.md | 更新 | 新增 notification 模块说明 |
| docs/decisions/ADR-003-email-adapter.md | 新建 | 邮件适配器选型决策 |
| .dependency-cruiser.cjs | 更新 | 新增 service→email-sdk 禁止规则 |

**无需更新**：conventions.md、ci.yml
```
