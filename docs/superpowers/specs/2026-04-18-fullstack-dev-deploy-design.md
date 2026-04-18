# fullstack-dev-deploy Skill 设计文档

## 背景

当前仓库缺少一个面向“任意全栈工程”的工程化 skill，用于统一本地开发入口和单机 Docker 部署入口。目标项目通常是全栈项目，可能包含微服务和微前端，常见形态是 `frontend/*` 与 `backend/*`，并可能出现 `biz`、`boss` 这样的业务分组，但 skill 不应把这些目录约定写死。

该 skill 的核心目标有两点：

1. 为任意工程统一生成本地开发入口：`scripts/run.sh [service...]`
2. 为任意工程统一生成 Docker 部署入口：`scripts/deploy.sh [service...]`

其中：

- 不传参数时默认 `all`
- 传 `biz`、`boss` 时，表示操作对应服务集合
- 对复杂项目，`biz`、`boss` 是用户层别名，内部可展开为具体 app 清单
- 部署必须完全不依赖宿主机已有 Node、Go、Python 等运行时，构建和部署都在 Docker 内完成
- 必须显式支持国内源、企业内网源、云厂商私有镜像源等环境差异

## 设计目标

### 目标内

- 自动探测目标项目结构，而不是强依赖固定目录
- 统一 dev 与 deploy 的外部使用体验
- 为目标项目生成或修正脚本、Dockerfile、Compose 文件、环境变量示例
- 让生成物具备可验证性，而不是只生成文件
- 对识别结果、假设项、生成结果给出清晰摘要

### 目标外

- 不覆盖 Kubernetes、Swarm、Nomad 等复杂编排
- 不假定目标项目已经符合某种脚手架结构
- 不要求宿主机安装语言运行时
- 不把国内源或某个云厂商源写死为不可覆盖默认值

## Skill 定位

建议 skill 名称使用 `fullstack-dev-deploy`。

它是一个**编排型 skill**，职责不是单纯提供建议，而是直接分析目标仓库并落盘生成工程资产。主文档只负责：

- 触发条件
- 输入输出定义
- 阶段路由
- 必须读取的 references
- 落盘与自检要求

复杂逻辑下沉到 references 中，保持 `SKILL.md` 可读且可执行。

## 总体架构

skill 分为三个执行层次：

1. `Discovery`：共享前置扫描阶段
2. `Dev`：本地开发入口规范化阶段
3. `Deploy`：Docker 化与单机部署阶段

整体执行顺序如下：

```text
读取用户需求
  -> 扫描目标仓库
  -> 生成项目画像
  -> 进入 Dev 阶段
  -> 进入 Deploy 阶段
  -> 输出识别结果、生成结果、假设项、验证结果
```

`Dev` 和 `Deploy` 都消费同一份“项目画像”，避免重复扫描和规则漂移。

## 统一外部契约

### Dev

统一入口：

```bash
./scripts/run.sh [service...]
```

行为约定：

- 空参等价于 `all`
- `./scripts/run.sh biz`：启动 `biz` 对应的前后端集合
- `./scripts/run.sh biz boss`：启动 `biz` 和 `boss` 对应的前后端集合

对于复杂项目，skill 可以在脚本内部把 `biz`、`boss` 展开为多个 frontend / backend app，但对用户仍保持统一入口。

### Deploy

统一入口：

```bash
./scripts/deploy.sh [service...]
```

行为约定：

- 空参等价于 `all`
- `./scripts/deploy.sh boss`：部署 `boss` 对应服务集合
- `./scripts/deploy.sh biz boss`：部署多个服务集合

部署阶段默认面向**单机 Docker / Docker Compose**。

## Discovery 设计

Discovery 负责把任意工程收敛成统一的中间抽象模型，供后续生成器和验证器使用。

### 需要探测的信息

- 目标仓库中有哪些 frontend / backend app
- 每个 app 的技术栈和类型
  - Node / pnpm / npm / yarn
  - Go / go mod
  - Python / pip / poetry
  - 纯静态前端
- 每个 app 的构建命令、启动命令、端口、依赖关系
- 现有脚本、Dockerfile、Compose、env 文件
- 哪些 app 属于 `biz`、哪些属于 `boss`
- 是否已存在镜像源、代理、私有源相关约定

### 中间抽象模型

Discovery 的结果在逻辑上应至少覆盖以下字段：

```text
apps[]
serviceGroups{all,biz,boss}
existingArtifacts
toolchains
envContracts
assumptions
```

说明：

- `apps[]`：每个应用的类型、目录、启动方式、构建方式、端口等
- `serviceGroups`：把用户层的 `all`、`biz`、`boss` 映射成具体 app 列表
- `existingArtifacts`：已有脚本、Dockerfile、Compose、env 文件
- `toolchains`：包管理器、语言生态、镜像源依赖
- `envContracts`：生成物需要读取的环境变量约定
- `assumptions`：自动推断但无法百分百确认的部分

### Discovery 的分流规则

1. **可自动识别**
   - 直接进入后续生成阶段
2. **可推断但有风险**
   - 继续生成，但必须记录“按假设处理”
3. **无法安全识别**
   - 停止对应阶段，并明确告诉用户缺少什么信息

skill 不允许在关键识别失败时静默生成“看起来完整”的结果。

## Dev 阶段设计

Dev 阶段的核心职责是：将不同项目的启动方式统一收敛到 `scripts/run.sh [service...]`。

### 生成内容

- `scripts/run.sh`
- 必要时更新 `.env.example`
- 必要时新增本地开发相关说明

### run.sh 的职责

- 解析参数，空参默认 `all`
- 把 `all`、`biz`、`boss` 展开为具体 app 集合
- 为每个 app 选择正确的启动命令
- 支持一次启动一个或多个服务集合
- 启动前加载统一环境变量契约
- 输出明确日志，说明本次实际启动了哪些 app

### 复杂项目处理

对于复杂项目，外部接口仍然只有一个 `run.sh`，但内部允许维护从服务别名到具体 app 的映射。例如：

```text
boss -> frontend-boss + backend-boss
biz  -> frontend-biz + backend-biz
all  -> 全部 app
```

如果识别不到分组关系，skill 应停止并要求用户补充映射，而不是胡乱猜测。

## Deploy 阶段设计

Deploy 阶段负责让目标项目在单机 Docker / Compose 上完成构建和部署。

### 生成内容

- `scripts/deploy.sh`
- `docker-compose.yml`
- 各 app 的 Dockerfile（缺失则创建，不规范则修正）
- `deploy.env.example`
- 必要时新增辅助脚本，例如 Docker 安装或镜像源配置脚本

### deploy.sh 的职责

- 解析参数，空参默认 `all`
- 把 `all`、`biz`、`boss` 展开为具体服务集合
- 校验 Docker 与 Compose 是否可用
- 加载部署环境变量
- 触发 `docker compose build`
- 触发 `docker compose up -d`
- 输出本次构建和部署的服务集合

### Docker 化原则

- 所有构建在容器内完成
- 不依赖宿主机已有语言运行时
- 优先使用多阶段构建
- 每个 app 的 Dockerfile 要与其技术栈匹配
- 若目标项目已有 Dockerfile，可在保留已有意图的前提下修正为统一约定

## 镜像源与环境变量契约

这是本 skill 的横切能力，`Dev` 和 `Deploy` 都必须遵守。

### 设计原则

- 不把国内源写死成不可覆盖常量
- 统一生成“可覆盖的环境变量契约”
- 本地脚本和 Docker build 都从同一套变量读取镜像源配置
- 允许云厂商内网源在目标环境中覆盖默认值

### 推荐的环境变量

- `NPM_REGISTRY`
- `PNPM_REGISTRY`
- `YARN_REGISTRY`
- `PIP_INDEX_URL`
- `GO_PROXY`
- `APT_MIRROR`
- `DOCKER_MIRROR`

必要时可按项目实际生态扩展，但原则保持一致：**通过环境变量配置源，而不是在脚本中写死。**

### 文件形态

建议至少生成：

- `.env.example`
- `deploy.env.example`

其中应明确说明：

- 哪些变量用于本地开发
- 哪些变量用于 Docker 构建与部署
- 哪些变量可被云厂商或企业内网源覆盖

## 测试与验证设计

这个 skill 的验收标准不是“文件生成成功”，而是“统一入口真实可用”。

### 验证目标

1. `run.sh` 参数解析正确
2. `run.sh` 对 `all|biz|boss` 的服务展开正确
3. `run.sh` 命中的启动命令正确
4. `deploy.sh` 参数解析正确
5. `deploy.sh` 能在本机 Docker / Compose 环境下正确构建与部署
6. 生成的 Dockerfile 与 Compose 配置可工作

### 测试分层

#### 第一层：脚本静态校验

- 脚本存在
- 参数分支完整
- 环境变量读取逻辑存在
- 默认 `all` 行为存在

#### 第二层：Dev 行为验证

- 空参是否等价于 `all`
- `biz` / `boss` 是否映射到正确 app 集合
- 生成的启动命令是否命中目标 app
- 若目标仓库已有测试、健康检查或 smoke test，优先复用

#### 第三层：Deploy 本机验证

- Dockerfile 能成功构建
- `docker-compose.yml` 能按服务集合启动
- `scripts/deploy.sh` 能在本机 Docker 环境执行
- 若已有 healthcheck / smoke test，则纳入部署后验证

### 对生成物的要求

所有生成的关键脚本都必须纳入验证链路：

- `scripts/run.sh`
- `scripts/deploy.sh`

不能出现“脚本已生成，但未被验证”的交付结果。

## 落盘清单

默认情况下，skill 应直接改造目标仓库并生成以下文件：

- `scripts/run.sh`
- `scripts/deploy.sh`
- `docker-compose.yml`
- 每个 app 对应的 Dockerfile
- `.env.example`
- `deploy.env.example`

按项目实际情况，可补充：

- Docker 安装脚本
- 源配置辅助脚本
- 部署说明文档

## 输出摘要要求

skill 执行结束时，必须输出结构化摘要，至少覆盖：

- 识别到的 app 列表
- `all`、`biz`、`boss` 分别映射了哪些 app
- 新建了哪些文件
- 修改了哪些文件
- 哪些地方按假设处理
- 哪些验证已执行

## References 建议拆分

建议将主 skill 下沉为以下 reference 文件：

- `references/phase-1-discovery.md`
- `references/phase-2-dev.md`
- `references/phase-3-deploy.md`
- `references/mirror-and-env-contract.md`
- `references/testing-and-validation.md`
- `references/output-summary.md`

如果需要强化“复杂工程识别”，可再补：

- `references/service-group-detection.md`

## 设计结论

`fullstack-dev-deploy` 应被设计为一个“先识别、后统一、再验证”的编排型 skill：

- 先通过 Discovery 生成统一项目画像
- 再基于项目画像生成 `run.sh` 与 `deploy.sh`
- 同时生成 Dockerfile、Compose、环境变量示例
- 最终用脚本测试与本机 Docker 验证来证明交付可用

这样既能适配任意全栈项目，又能给用户稳定统一的入口体验，并兼顾国内源、云厂商内网源和复杂项目结构。
