# 阶段一：Discovery 共享扫描

## 目标

把任意全栈仓库收敛成一份统一 `projectProfile`，并同时产出资产盘点与治理计划，供后续 `dev`、`deploy`、验证和摘要阶段共用。

## 必须扫描的信息

1. app 列表：仓库中有哪些 frontend / backend 或其他可运行单元
2. 技术栈：Node、Go、Python、静态前端等
3. 包管理器：npm、pnpm、yarn、go mod、pip、poetry 等
4. 启动命令、构建命令、默认端口、依赖关系
5. 现有资产：脚本、Dockerfile、Compose、env 文件、镜像源配置
6. 真实服务名：目标项目实际暴露给用户的服务名，例如 `backend`、`h5`、`weapp`
7. 可选聚合组：是否存在稳定的高层服务集合语义，例如 `all` 之外的 `admin`、`mobile`、`gateway`
8. 一个 app 目录是否承载多个运行面或多个对外服务，例如同一个 `frontend/` 同时产出 `h5` 与 `weapp`
9. 基础设施依赖：MySQL、Redis、MQ、MinIO 等是否由本仓库托管

## 推荐扫描顺序

1. 先看目录结构：`frontend/`、`backend/`、`apps/`、`services/`
2. 再看 app 级配置：`package.json`、`go.mod`、`pyproject.toml`、`requirements.txt`
3. 再看已有启动与部署资产：`scripts/`、`Dockerfile`、`docker-compose*.yml`、`compose*.yml`、`.env*`
4. 最后看 README 或项目脚本中的命名线索，补齐命令、端口和对外服务名

## 输出格式

Discovery 结束后，逻辑上必须形成如下结构：

```text
projectProfile{
  apps[]
  serviceGroups
  existingArtifacts
  assetInventory[]
  convergencePlan[]
  toolchains
  assumptions[]
}
```

字段说明：

- `apps[]`：每个 app 的名称、目录、类型、启动方式、构建方式、端口、依赖
- `serviceGroups`：至少包含 `all`；同时记录真实服务名；若存在稳定聚合语义，再追加聚合组
- `existingArtifacts`：已有脚本、Dockerfile、Compose、env 文件
- `assetInventory[]`：所有现有资产的清单、语义、归属和现状
- `convergencePlan[]`：每个资产的 keep / migrate / merge / delete / generate 动作及原因
- `toolchains`：语言生态、包管理器、构建依赖
- `assumptions[]`：自动推断但尚未得到用户确认的部分

补充约束：

- `apps[]` 记录物理 app 边界，例如 `frontend`、`backend`
- `serviceGroups` 记录对外暴露的真实服务名，例如 `h5`、`weapp`
- 如果一个 app 目录承载多个运行面，必须在 Discovery 中显式记录“一个 app → 多个服务”的关系，后续阶段不得私自拆成多个假 app

## 资产盘点范围

必须显式盘点：

- 本地启动脚本：`run.sh`、`start.sh`、`dev.sh`、`scripts/*`
- 部署脚本：`deploy.sh`、`release.sh`、`publish.sh`
- Dockerfile：根目录或各 app 下已有 Dockerfile
- Compose 文件：`docker-compose*.yml`、`compose*.yml`
- env 文件：根级旧模板、app 内 `.env*`、`docker/deploy.env*`、其他 `*.env*`
- Docker 安装与源配置脚本

## 资产盘点原则

- **不只看文件名**：必须结合内容和调用关系识别语义
- **先识别，再治理**：未识别语义前，不得先删后看
- **关注职责，不关注历史**：删除依据是职责已被标准资产完整吸收，而不是“文件很老”
- **一个事实来源**：`projectProfile` 是后续所有阶段的唯一输入，不允许各阶段私自改口径
- **案例名不能入侵项目画像**：案例中的 `frontend`、`backend`、`app1` 等示意名，不能直接替代真实项目服务名

## 治理动作模型

每个资产最终都必须归入以下动作之一：

- `keep`：已符合目标形态，直接保留
- `migrate`：语义正确，但路径或命名不符合目标形态，需要迁移/重命名
- `merge`：多个旧资产共同承担未来一个标准资产的职责，需要合并收敛
- `delete`：已被标准资产完整覆盖，继续保留只会制造歧义
- `generate`：标准资产缺失，需要新建

## env 收敛目标

env 相关资产的默认目标形态必须统一为：

- `docker/.env.example`
- `docker/.env`

Discovery 结束时，`convergencePlan` 必须明确：

- 哪些旧 env 文件将迁移到 `docker/.env.example`
- 哪些旧 env 文件将被合并后再写入 `docker/.env.example`
- 哪些旧 env 文件将被删除
- 哪些 env key 会被标准化为全大写蛇形命名

## 分流规则

### 1. 可自动识别

满足以下条件时直接继续：

- app 边界清晰
- 启动命令和构建命令可找到
- `all` 与真实服务名可直接确认
- 若存在聚合组，其证据来源一致
- 现有资产的语义和归属可稳定判断

### 2. 可推断但有风险

满足以下情况时允许继续，但必须记录到 `assumptions[]`：

- app 边界基本清楚，但命名不统一
- 真实服务名可以从目录、脚本、端口或现有入口中高概率推断
- 聚合组语义基本清楚，但仍存在单点不确定
- 一个 app 目录承载多个运行面，但共享关系基本清楚
- 旧脚本、旧 compose 或旧 env 的语义大致清楚，但仍存在单点不确定

### 3. 无法安全识别

出现以下情况必须停下并追问用户：

- 无法确定哪些目录是独立 app
- 找不到可靠的启动命令或构建命令
- 无法确认真实服务名
- 无法判断某个候选聚合组是否真的存在
- 无法判断一个 app 目录是否承载多个运行面
- 旧 `dev` / `deploy` 入口对 `all` 或共享分组的含义不一致
- 无法判断某个旧脚本原本承担什么职责
- 无法判断某个旧 compose 属于 infra 还是 app
- 无法判断某个 env 文件是否应收敛到 `docker/.env.example`

## 禁止行为

- 看到案例里的名字，就直接当成目标项目服务名
- 只凭 README 描述，不交叉验证实际脚本和配置
- Discovery 和 Dev / Deploy 使用不同的 app 名称或服务映射
- 对关键字段或旧资产语义识别失败时继续生成结果
