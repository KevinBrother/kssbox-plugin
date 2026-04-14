# 技术规范参考

> 本文件供 prd-writer skill 在生成第 5 章（API 设计/状态管理规格）和第 7.6 章（Harness Engineering 要求）时查阅。
> **不替代架构设计决策**——具体选型应由用户/团队在开放问题中确认，未确认前 PRD 里写"待定"。

---

## 一、认证方案

### 选项对比

| 方案 | 适用场景 | Token 有效期建议 | 主要注意点 |
|-----|--------|----------------|---------|
| **JWT（无状态）** | 前后端分离、无状态扩展、微服务 | Access Token 15-60 分钟；Refresh Token 7-30 天 | Refresh Token 需存 DB 以支持吊销；不要把敏感数据放 payload |
| **Session（有状态）** | 服务端渲染、对吊销要求高 | Session 有效期 1-7 天，滑动续期 | 需 Redis 等共享 Session 存储；横向扩展需 sticky session 或集中存储 |
| **OAuth2 + OIDC** | 第三方登录（Google / GitHub 等）、企业 SSO | 遵循 Provider 默认值 | 通常与 JWT 结合使用；需要处理 callback URL 安全验证 |

### PRD 中的写法

- **已确认方案**：在 5A 章的"认证方式"字段填写具体方案，在 7.2 安全要求里补充 Token 有效期和存储规则。
- **未确认方案**：5A 章填"待定，见第 10 章 Q1"，第 10 章开放问题列出上表三个选项，标注影响范围"所有需认证的 API"。

### 安全基线（无论选哪种方案都必须遵守）

- HTTPS 全站强制，禁止 HTTP 传输 Token
- Token 不存 localStorage（XSS 风险），推荐 `HttpOnly Cookie` 或内存
- 密码存储：bcrypt，cost factor ≥ 12
- 登录失败连续 5 次锁定账号或触发验证码

---

## 二、API 版本管理

### 版本策略选项

| 策略 | 示例 | 优点 | 缺点 |
|-----|-----|-----|-----|
| **URL 路径版本（推荐 MVP）** | `/api/v1/users` | 明确、易缓存、Swagger 易区分 | URL 冗长 |
| Header 版本 | `Accept: application/vnd.myapp.v1+json` | URL 干净 | 调试麻烦，需额外文档 |
| 查询参数版本 | `/api/users?version=1` | 简单 | 污染查询参数 |

**MVP 阶段建议**：URL 路径版本，`Base URL: /api/v1`。

### 版本管理规则

- v1 稳定后，破坏性变更（删字段、改类型、改行为）必须升版本为 `/api/v2`
- 向后兼容的变更（新增字段、新增端点）不需要升版本
- 版本生命周期：新版本发布后，旧版本至少维护 6 个月后才下线

---

## 三、Harness Engineering 要求

> 这是 AI 辅助开发的必要基础设施规范。所有通过 AI Agent 开发的项目**必须**在 M1 阶段完成这些文件的初始化。

### 3.1 仓库规范文件

M1（基础搭建阶段）必须创建以下文件：

| 文件 | 位置 | 核心内容 |
|-----|-----|--------|
| `AGENTS.md` | 项目根目录 | 架构层级、命名约定、禁止事项、测试要求、常见错误模式 |
| `docs/architecture.md` | docs/ | 模块边界图、单向依赖方向、关键设计决策 |
| `docs/conventions.md` | docs/ | 命名规范、文件组织规则、代码风格约定 |

**AGENTS.md 必须包含的章节**：

```
## Architecture Layers（架构层级，依赖方向不可逆向）
## Naming Conventions（命名约定）
## Prohibited Patterns（禁止事项）
## Testing Requirements（测试要求）
## Common Error Patterns（常见错误模式及修复方式）
```

### 3.2 分层架构强制要求

依赖方向（单向，不可逆向）：

```
Types → Config → Repository → Service → Runtime → UI
```

- Types/Config 层：纯数据结构，不引用任何业务层
- Repository 层：只做数据读写，不含业务逻辑
- Service 层：业务逻辑，不直接操作 UI 状态
- UI 层：只调用 Service，不直接操作 Repository

### 3.3 CI 强制检查

以下所有检查必须在 CI 中配置，**失败时阻断合并**：

| 检查项 | 工具参考 | 失败条件 |
|------|--------|--------|
| 类型检查 | TypeScript / mypy | 任意类型错误 |
| 代码风格 | ESLint / Ruff | 任意 lint 错误 |
| 模块边界 | dependency-cruiser / ArchUnit | 违反依赖方向 |
| 单元测试覆盖率 | Jest / pytest-cov | 核心业务逻辑覆盖率 < 80% |

> 具体工具选型不在 PRD 决策范围内，由开发团队在 M1 架构设计时确定，在第 10 章开放问题中列出候选。

### 3.4 人工节点（里程碑中必须标注）

| 节点 | 时机 | 内容 |
|-----|-----|-----|
| 架构设计确认 | M1 结束前 | 确认分层结构、依赖方向、CI 配置方案 |
| 数据模型 Schema 定稿 | Service 层开始前 | 数据库 Schema 评审，签字确认不再大改 |
| 架构一致性检查 | 每个 Milestone 结束后 | 0.5 天，检查 pattern drift，更新 AGENTS.md |

---

## 四、PRD 写作快查

| 章节 | 查阅本文哪部分 |
|-----|------------|
| 第 5A 章：认证方式 | 第一节：认证方案选项对比 |
| 第 5A 章：Base URL | 第二节：URL 路径版本策略 |
| 第 7.2 章：安全要求 | 第一节：安全基线 |
| 第 7.6 章：Harness Engineering | 第三节：完整要求 |
| 第 9 章：里程碑人工节点 | 第三节 3.4：人工节点 |
