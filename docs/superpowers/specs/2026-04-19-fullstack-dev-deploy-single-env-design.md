# fullstack-dev-deploy 单一 env 契约重构设计

## 背景

当前 `fullstack-dev-deploy` skill 已经支持生成和治理 `scripts/dev.sh`、`scripts/deploy.sh`、Docker 资产与 env 资产，但 env 设计采用了“app 自治 + 多环境文件”的路线：

- `<app>/.env.dev.example`
- `<app>/.env.prod.example`
- `<app>/.env.local.example`
- 必要时还允许 service overlay

这一设计在真实项目中带来了两类问题：

1. **使用成本过高**：目标项目被重构后，用户需要维护多套 `.env.dev` / `.env.prod` / `.env.local`，与“快速可运行”的目标相冲突。
2. **脚本体验不稳定**：以 `high-chat` 为例，当前生成的 `scripts/dev.sh` 在缺失 env 文件时提示不够聚焦，而且在提前退出路径上出现 `PIDS[@]: unbound variable`，说明脚本退出与清理模型也需要收敛。

用户希望本 skill 改为更接近 `coze-studio` 的模型：

- 项目只维护**一份**运行时 env 模板
- 该模板位于 `docker/.env.example`
- 实际运行时一律 `cp docker/.env.example docker/.env`
- 前端与后端都通过进程环境变量获取配置

本次设计的目标不是微调文案，而是把 `fullstack-dev-deploy` 的 env 哲学从“按 app 分散维护”切换为“按项目集中维护”。

## 设计目标

重构后的 `fullstack-dev-deploy` 应满足：

1. 默认只生成 `docker/.env.example`
2. 实际运行时只要求 `docker/.env`
3. `scripts/dev.sh`、`scripts/deploy.sh`、Compose、Docker build 全部围绕同一份 env 契约工作
4. 所有自动生成的 env key 必须采用**全大写蛇形命名**
5. 旧的根级 env、app 内 env、`docker/deploy.env*` 都进入治理流程，最终统一收敛
6. 缺失 env 时，脚本必须明确提示**具体缺失路径**和**推荐复制命令**
7. 提前失败时，`scripts/dev.sh` 的清理逻辑不能再触发 `PIDS[@]: unbound variable`

## 非目标

本次重构不做以下事情：

- 不扩展到 Kubernetes、Helm 或其他部署模型
- 不重新引入按 app 或按环境拆分的 `.env.dev/.env.prod/.env.local`
- 不允许 service-specific env overlay 成为默认路径
- 不为少数特殊项目保留多 env 作为主规范，只能在未来必要时讨论例外机制

## 参考模型：coze-studio

通过对 `coze-dev/coze-studio` 的公开仓库进行分析，可提炼出以下模式：

1. `docker/.env.example` 是唯一模板
2. 运行时复制为 `docker/.env`
3. `docker/docker-compose.yml` 使用 `.env` 作为统一 env source
4. 服务端容器通过挂载或进程环境读取同一份 env
5. 前端不再要求开发者维护独立的前端 env 文件，而是通过编排层或进程环境获得配置

`fullstack-dev-deploy` 不需要逐字照搬 `coze-studio` 的目录结构，但应采用同样的核心原则：**一份项目级 env 契约供全部运行面共享**。

## 总体架构调整

### 新的 env 契约

重构后的目标形态：

```text
docker/
  .env.example
  .env              # 本地或服务器运行时复制生成，不入 git
  infra.compose.yml
  app.compose.yml
scripts/
  dev.sh
  deploy.sh
```

其中：

- `docker/.env.example`：唯一模板文件
- `docker/.env`：唯一实际运行时 env 文件
- `scripts/dev.sh`：只检查并读取 `docker/.env`
- `scripts/deploy.sh`：只检查并读取 `docker/.env`
- `docker/infra.compose.yml` 与 `docker/app.compose.yml`：只通过 `docker/.env` 获取配置

### 命名规范

自动生成或重构后的 env key 必须遵守：

- 全部使用全大写
- 使用下划线分隔
- 同一职责只允许一个稳定命名

示例：

- `SERVER_PORT`
- `DATABASE_URL`
- `REDIS_URL`
- `NPM_REGISTRY`
- `WEAPP_APP_ID`

禁止生成含糊或重复职责的命名组合，例如：

- `PORT` + `SERVER_PORT`
- `APP_HOST` + `SERVER_HOST`
- `MYSQL_URL` + `DATABASE_URL` 在无明确职责区分时同时存在

## Discovery 与治理模型调整

### 旧 env 资产盘点范围

Discovery 阶段必须显式扫描：

- 根目录 `.env*`
- app 目录内的 `.env*`
- `docker/deploy.env*`
- 其他被旧脚本或 compose 显式引用的 env 文件

### 新的治理目标

所有旧 env 资产都要收敛到：

- `docker/.env.example`
- `docker/.env`（仅运行时）

### 治理动作

每个旧 env 资产必须落入以下动作之一：

- `migrate`：语义正确，但位置不对，迁移到 `docker/.env.example`
- `merge`：多个旧文件共同表达项目运行时配置，需要合并为一份模板
- `delete`：已被统一模板完整吸收，继续保留会制造歧义
- `keep`：只允许用于真正不应被合并的非 env 资产；对 env 资产应极少出现
- `stop-and-ask`：同名变量在多个来源中存在冲突，且无法确定优先级时停止并询问用户

### 输出要求

Discovery 产出的 `convergencePlan` 必须明确：

- 哪些旧 env 文件将被迁移
- 哪些将被合并
- 哪些将被删除
- 哪些变量名会被标准化为全大写形式

## dev 阶段设计调整

### 读取模型

`scripts/dev.sh` 只允许依赖：

- `docker/.env`

不得再生成或读取：

- `.env.dev`
- `.env.local`
- `<app>/.env.dev`
- `<app>/.env.local`
- 任意 service overlay env

### 缺失 env 文件时的行为

若 `docker/.env` 不存在，必须明确输出：

```bash
✗ Missing env file: docker/.env
  Expected template: docker/.env.example
  Run: cp docker/.env.example docker/.env
```

如果 env 文件存在但关键变量缺失，则继续报：

```bash
✗ Missing required env keys: DATABASE_URL REDIS_URL
```

### 清理与提前退出

`scripts/dev.sh` 必须修复提前退出路径上的清理问题：

- 在尚未启动任何子进程时退出，cleanup 不能访问未初始化的 PID 集合
- 只有确实启动过子进程，才输出“Shutting down dev services...”
- 缺失 env 文件时不应再出现二次错误

### 启动日志要求

`scripts/dev.sh` 的输出至少要说明：

- 实际目标服务
- 使用的 env 文件路径（`docker/.env`）
- 缺失文件时的复制命令
- 若启动失败，是 env 缺失、关键变量缺失还是 readiness 失败

## deploy 阶段设计调整

### 读取模型

`scripts/deploy.sh`、`docker/infra.compose.yml`、`docker/app.compose.yml`、Docker build 都统一读取：

- `docker/.env`

不得再依赖：

- `docker/deploy.env`
- `docker/deploy.env.example`
- app 内 `.env.prod`

### 变量职责

由于现在使用项目级单一 env，必须在 `docker/.env.example` 内通过注释分组来区分职责，而不是通过拆文件区分职责。

推荐分组：

- `# Server`
- `# Database`
- `# Cache`
- `# Frontend Build`
- `# Deployment`
- `# Mirrors`

### 迁移要求

旧 `docker/deploy.env*` 中的变量需要：

- 如果是编排级变量，迁入 `docker/.env.example`
- 如果是 app 运行时变量，也迁入 `docker/.env.example`
- 如果存在命名不规范，统一改为全大写蛇形命名

## 输出摘要调整

执行摘要必须新增 env 收敛信息：

- 旧 env 文件清单
- migrate / merge / delete 动作
- 最终保留的 env 资产：`docker/.env.example`
- 本地实际运行文件：`docker/.env`
- 是否发现命名标准化行为
- 缺失 env 时推荐的复制命令

## 验证策略调整

### 静态验证

生成后必须验证：

- 仓库中不再残留作为主契约的 `.env.dev*` / `.env.prod*` / `.env.local*`
- `docker/.env.example` 存在
- `scripts/dev.sh` 与 `scripts/deploy.sh` 都引用 `docker/.env`
- env key 命名符合全大写蛇形规范

### 行为验证

至少需要覆盖以下行为：

1. 删除 `docker/.env` 后运行 `scripts/dev.sh`
   - 必须提示缺失的是 `docker/.env`
   - 必须提示模板是 `docker/.env.example`
   - 必须提示复制命令
   - 不能出现 `PIDS[@]: unbound variable`

2. 删除 `docker/.env` 后运行 `scripts/deploy.sh`
   - 必须提示缺失的是 `docker/.env`
   - 不得继续执行 compose 或 build

3. `docker/.env` 存在但关键变量缺失
   - 必须明确列出缺失变量名

### 成功标准

本次重构完成的标志是：

- skill 不再默认生成多环境 env 结构
- `high-chat` 这类项目重新套用 skill 后，只剩 `docker/.env.example`
- 运行前只需执行 `cp docker/.env.example docker/.env`
- `dev.sh` / `deploy.sh` 的 env 报错信息准确、可执行、无二次异常

## 风险与约束

### 风险

1. 单一 env 模型会降低“按 app 隔离配置”的表达能力
2. 某些复杂项目可能把 dev 与 deploy 变量混在同一份文件中，导致模板较长
3. 旧项目变量命名不规范时，批量标准化可能引入兼容性问题

### 应对

1. 明确本 skill 优先追求统一、可维护、低学习成本
2. 通过注释分组而不是拆文件来维持可读性
3. Discovery 必须记录变量重命名映射，避免无说明迁移

## 实施边界

本次设计仅要求修改 `fullstack-dev-deploy` skill 包及其 references，不要求在本次设计阶段直接改动 `high-chat` 仓库。

后续实现时，应至少覆盖以下文件：

- `skills/fullstack-dev-deploy/SKILL.md`
- `skills/fullstack-dev-deploy/references/phase-1-discovery.md`
- `skills/fullstack-dev-deploy/references/phase-2-dev.md`
- `skills/fullstack-dev-deploy/references/phase-3-deploy.md`
- `skills/fullstack-dev-deploy/references/mirror-and-env-contract.md`
- `skills/fullstack-dev-deploy/references/testing-and-validation.md`
- `skills/fullstack-dev-deploy/references/output-summary.md`

必要时还应补充或调整与执行 hardening 相关的设计文档与实现计划。
