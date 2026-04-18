# 阶段一：Discovery 共享扫描

## 目标

把任意全栈仓库收敛成一份统一“项目画像”，并同时产出资产盘点与治理计划，供后续 `dev` 和 `deploy` 阶段共用。

## 必须扫描的信息

1. app 列表：仓库中有哪些 frontend / backend 或其他可运行单元
2. 技术栈：Node、Go、Python、静态前端等
3. 包管理器：npm、pnpm、yarn、go mod、pip、poetry 等
4. 启动命令、构建命令、默认端口、依赖关系
5. 现有资产：脚本、Dockerfile、Compose、env 文件、镜像源配置
6. 服务分组候选：哪些 app 可能属于 `biz`、`boss`
7. 基础设施依赖：MySQL、Redis、MQ、MinIO 等是否由本仓库托管

## 推荐扫描顺序

1. 先看目录结构：`frontend/`、`backend/`、`apps/`、`services/`
2. 再看 app 级配置：`package.json`、`go.mod`、`pyproject.toml`、`requirements.txt`
3. 再看已有启动与部署资产：`scripts/`、`Dockerfile`、`docker-compose*.yml`、`compose*.yml`、`.env*`
4. 最后看 README 或项目脚本中的命名线索，补齐命令和端口信息

## 输出格式

Discovery 结束后，逻辑上必须形成如下结构：

```text
apps[]
serviceGroups{all,biz,boss}
existingArtifacts
assetInventory[]
convergencePlan[]
toolchains
assumptions
```

字段说明：

- `apps[]`：每个 app 的名称、目录、类型、启动方式、构建方式、端口
- `serviceGroups`：`all`、`biz`、`boss` 对应的 app 清单
- `existingArtifacts`：已有脚本、Dockerfile、Compose、env 文件
- `assetInventory[]`：所有现有资产的清单、语义、归属和现状
- `convergencePlan[]`：每个资产的 keep / migrate / merge / delete / generate 动作及原因
- `toolchains`：语言生态、包管理器、构建依赖
- `assumptions`：自动推断但尚未得到用户确认的部分

## 资产盘点范围

必须显式盘点：

- 本地启动脚本：`run.sh`、`start.sh`、`dev.sh`、`scripts/*`
- 部署脚本：`deploy.sh`、`release.sh`、`publish.sh`
- Dockerfile：根目录或各 app 下已有 Dockerfile
- Compose 文件：`docker-compose*.yml`、`compose*.yml`
- env 文件：根级旧模板、app 内 `.env*`、其他 `*.env*`
- Docker 安装与源配置脚本

## 资产盘点原则

- **不只看文件名**：必须结合内容和调用关系识别语义
- **先识别，再治理**：未识别语义前，不得先删后看
- **关注职责，不关注历史**：删除依据是职责已被标准资产完整吸收，而不是“文件很老”
- **一个事实来源**：Discovery 产物是后续所有阶段的唯一输入，不允许各阶段私自改口径

## 治理动作模型

每个资产最终都必须归入以下动作之一：

- `keep`：已符合目标形态，直接保留
- `migrate`：语义正确，但路径或命名不符合目标形态，需要迁移/重命名
- `merge`：多个旧资产共同承担未来一个标准资产的职责，需要合并收敛
- `delete`：已被标准资产完整覆盖，继续保留只会制造歧义
- `generate`：标准资产缺失，需要新建

## 分流规则

### 1. 可自动识别

满足以下条件时直接继续：

- app 边界清晰
- 启动命令和构建命令可找到
- `biz` / `boss` / `all` 分组可直接确认
- 现有资产的语义和归属可稳定判断

### 2. 可推断但有风险

满足以下情况时允许继续，但必须记录到 `assumptions`：

- app 边界基本清楚，但命名不统一
- `biz` / `boss` 可以从目录、脚本或端口中高概率推断
- 旧脚本、旧 compose 或旧 env 的语义大致清楚，但仍存在单点不确定

### 3. 无法安全识别

出现以下情况必须停下并追问用户：

- 无法确定哪些目录是独立 app
- 找不到可靠的启动命令或构建命令
- 无法判断 `biz`、`boss` 分组
- 无法判断某个旧脚本原本承担什么职责
- 无法判断某个旧 compose 属于 infra 还是 app
- 无法判断某个 env 文件属于哪个 app 或哪个环境

## 禁止行为

- 看到目录名像业务模块，就直接当成 `biz` 或 `boss`
- 只凭 README 描述，不交叉验证实际脚本和配置
- Discovery 和 Dev / Deploy 使用不同的 app 名称
- 对关键字段或旧资产语义识别失败时继续生成结果
