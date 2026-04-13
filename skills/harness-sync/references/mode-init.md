# 模式：INIT 新建项目

从 PRD 生成完整的 Harness 文件集。

## 输入要求

必须有以下信息（从 PRD 或用户处获取）：
- 产品名称和技术栈
- 分层架构（PRD 第 7.6 章）
- MVP 功能列表（PRD 第 6 章）
- 第三方集成（PRD 第三方集成需求）
- CI 要求（PRD 第 7.6 章）

如果用户没有提供 PRD，问：
1. 技术栈是什么？（Next.js / Node+React / Python+FastAPI / 其他）
2. 主要分几层？各层叫什么名字？
3. 有哪些核心模块？

---

## 生成顺序（串行，有依赖）

```
Step 1: 生成 architecture.md    ← 先定层级，后续所有文件基于它
Step 2: 生成 AGENTS.md          ← 基于 architecture.md
Step 3: 生成 conventions.md     ← 基于 architecture.md + 技术栈
Step 4: 生成 ADR-001            ← 记录技术栈选型决策
Step 5: 生成 CI 配置             ← 基于技术栈 + architecture.md 的层级
Step 6: 生成依赖规则配置          ← 基于 architecture.md 的层级
```

---

## 各文件生成规范

### Step 1: `docs/architecture.md`

必须包含：

**模块层级图**（文字版，清晰表达依赖方向）：
```
[层级名] ──依赖→ [层级名] ──依赖→ [层级名]

示例（全栈 Web）：
types/schemas → config → repository → service → api-handlers → ui
```

**每层职责说明**：
| 层级 | 目录 | 职责 | 不负责 |
|-----|-----|-----|------|
| Types | `src/types/` | 类型定义、接口、枚举 | 任何业务逻辑 |
| Config | `src/config/` | 环境变量、常量 | 数据访问 |
| Repository | `src/repository/` | 数据库读写、ORM 操作 | 业务规则 |
| Service | `src/service/` | 业务逻辑、跨模块协调 | HTTP 细节 |
| API Handlers | `src/api/` | 路由、请求/响应处理 | 业务逻辑 |
| UI | `src/components/` | 渲染、用户交互 | 直接访问数据库 |

**严格禁止的依赖**（必须列出）：
- UI 层不得直接 import Repository 层
- API Handlers 不得包含业务逻辑（只做参数校验和调用 Service）
- [根据项目具体情况补充]

---

### Step 2: `AGENTS.md`

这是 AI agent 每次打开仓库都会读的文件。必须**具体**，不能写通用废话。

结构：

```markdown
# [产品名] — Agent 指南

## 项目概述
[2-3 句话，说明这个项目是什么，用什么技术栈]

## 架构层级
[直接引用 architecture.md 的层级图，或简化版]

依赖方向：types → config → repository → service → api → ui
违反此规则的 import 会被 CI 的依赖检查拦截。

## 模块职责速查
[每层一行，说明放什么、不放什么]

## 命名约定
- 文件名：[规则，如 kebab-case]
- 函数名：[规则，如 camelCase，动词开头]
- 类名：[规则，如 PascalCase]
- 数据库表名：[规则，如 snake_case 复数]
- API 路由：[规则，如 /api/v1/resources 复数名词]

## 禁止事项（重要）
这些事情 agent 绝对不能做：
1. [具体禁止项，如：UI 组件中直接调用 Prisma/数据库]
2. [具体禁止项，如：在 Service 层处理 HTTP request/response 对象]
3. [具体禁止项，如：在 Repository 层写业务规则]
4. [具体禁止项，如：跳过 Service 层在 API Handler 里直接写业务逻辑]
5. [根据项目特点补充]

## 测试要求
- Service 层：单元测试覆盖率 ≥ 80%，必须 mock Repository 层
- Repository 层：集成测试，连接测试数据库
- API Handler：端到端测试覆盖核心流程
- UI 组件：[视项目情况]

## 不确定时怎么做
遇到以下情况，停下来等人工确认，不要自行决定：
- 需要新增一个层级或模块
- 发现现有架构无法满足需求，需要重构
- 第三方依赖需要替换
- 安全相关的实现细节

## 已知的常见错误
[随项目演进持续更新]
- [错误1：agent 之前犯过的错，怎么正确做]
```

---

### Step 3: `docs/conventions.md`

```markdown
# 命名与文件组织约定

## 目录结构
[画出项目的标准目录树，越具体越好]

## 命名规范
[详细规则，包括正面示例和反面示例]

## 错误处理约定
- 如何定义自定义错误类
- 错误在哪层捕获、在哪层抛出
- 日志记录规范

## 测试文件组织
- 测试文件放在哪里（与源文件同目录 vs 独立 __tests__ 目录）
- 测试文件命名规则
- Mock 文件组织

## 环境变量
- 命名规则（如全大写 SNAKE_CASE）
- 在哪里访问（只通过 config 层）
- .env.example 必须和 .env 保持同步
```

---

### Step 4: `docs/decisions/ADR-001-tech-stack.md`

ADR（Architecture Decision Record）格式：

```markdown
# ADR-001: 技术栈选型

**日期**：[日期]
**状态**：已确认

## 背景
[为什么需要做这个决策]

## 决策
选用 [技术栈]，原因：
- [原因1]
- [原因2]

## 后果
**正面**：
- [好处1]

**负面/权衡**：
- [代价1，如何应对]

## 备选方案
- [方案A]：放弃原因 [...]
- [方案B]：放弃原因 [...]
```

---

### Step 5: CI 配置（`.github/workflows/ci.yml`）

根据技术栈生成对应的 CI，必须包含四类门禁：

**JS/TS 项目示例结构**：
```yaml
name: CI
on: [push, pull_request]

jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4

      # 门禁1：类型检查
      - name: Type check
        run: npx tsc --noEmit

      # 门禁2：代码风格
      - name: Lint
        run: npx eslint . --max-warnings=0

      # 门禁3：模块边界
      - name: Dependency rules
        run: npx dependency-cruiser src --config .dependency-cruiser.cjs

      # 门禁4：测试覆盖率
      - name: Test with coverage
        run: npx jest --coverage --coverageThreshold='{"global":{"lines":80}}'
```

**Python 项目**（用 Ruff + mypy + import-linter + pytest-cov）

所有门禁失败均阻断合并（不是 warning）。

---

### Step 6: 依赖规则配置

**JS/TS：`.dependency-cruiser.cjs`**

根据 architecture.md 的层级，生成对应的 forbidden rules：

```javascript
module.exports = {
  forbidden: [
    {
      name: "no-ui-to-repository",
      comment: "UI 层不得直接访问 Repository 层，必须通过 Service 层",
      severity: "error",
      from: { path: "src/components" },
      to: { path: "src/repository" }
    },
    {
      name: "no-api-to-repository",
      comment: "API Handler 不得直接访问 Repository 层",
      severity: "error",
      from: { path: "src/api" },
      to: { path: "src/repository" }
    },
    // 根据架构层级自动生成对应规则
  ]
}
```

规则必须和 AGENTS.md 的"禁止事项"一一对应——AGENTS.md 说禁止的，这里就有规则来机械执行。
