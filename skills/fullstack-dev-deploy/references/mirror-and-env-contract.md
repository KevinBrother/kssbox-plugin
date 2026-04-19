# 镜像源与 app env 契约

## 目标

为 `dev` 和 `deploy` 阶段提供一套统一、可覆盖的镜像源变量约定，同时把环境变量模型收敛为 app 自治模式，而不是中心化 env 聚合文件。

## 设计原则

- 不把国内源写死成不可覆盖常量
- 本地脚本和 Docker build 使用同一套镜像源变量命名
- 优先通过环境变量注入，而不是把源写进脚本逻辑
- 云厂商内网源允许覆盖默认值
- app 自己维护运行时 env 模板，不由仓库根目录统一代管

## 推荐镜像变量

- `NPM_REGISTRY`
- `PNPM_REGISTRY`
- `YARN_REGISTRY`
- `PIP_INDEX_URL`
- `GO_PROXY`
- `APT_MIRROR`
- `DOCKER_MIRROR`

如果目标项目使用其他生态，可以扩展变量，但命名和职责必须清晰。

## app env 目标形态

每个 app 至少应具备：

- `<app>/.env.dev.example`
- `<app>/.env.prod.example`
- `<app>/.env.local.example`

要求说明：

- `.env.dev.example` 用于本地开发默认值
- `.env.prod.example` 用于部署环境契约说明
- `.env.local.example` 用于本地私有覆盖示例

真实运行时文件处理规则：

- `.env.dev`、`.env.prod`、`.env.local` 由开发者或部署目标从 example copy 生成
- 真实 `.env*` 不应提交到 git
- `dev.sh` 和 `deploy.sh` 只负责根据运行模式选择正确的 app env

### 单 app 多运行面

如果一个物理 app 目录同时承载多个真实服务名或运行面，例如同一个 `frontend/` 同时产出 `h5` 与 `weapp`：

- 仍以 app 目录作为 env 归属边界
- 先保留基础模板，例如 `frontend/.env.dev.example`、`frontend/.env.prod.example`、`frontend/.env.local.example`
- 只有在不同运行面确实需要差异变量时，才追加 service overlay，例如 `frontend/.env.h5.dev.example`、`frontend/.env.weapp.prod.example`
- service overlay 是补充层，不得替代 app 基础模板

### 部署编排 env 契约

只属于 Docker 构建、Compose 编排或部署目标选择的变量，可以单独收口到 `docker/` 下的 deploy env 契约，例如：

- `docker/deploy.env.example`

这类变量通常包括：

- 部署目标主机
- 对外暴露端口
- 构建开关
- compose profile 或发布批次参数

约束如下：

- deploy/orchestration env 不能替代 app 的 `.env.prod.example`
- app 运行时变量仍应归属到 app 自治 env
- `deploy.sh` 可以同时读取 deploy/orchestration env 和 app `.env.prod`
- 如果某个变量既影响编排又影响 app 运行时，必须明确拆分职责，不能偷懒只保留在中心化 deploy env 中

## 治理规则

在生成新 env 结构前，必须先处理旧 env 资产：

- 根目录 `.env.example` 若承担单 app 模板职责，可 `migrate` 到对应 app
- 多个旧 env 模板共同表达同一 app 的运行时配置时，可 `merge`
- 已被 app 自治 env 完整覆盖的中心化模板，执行 `delete`
- 旧 `docker/deploy.env*` 若承担部署编排职责，可保留为 `docker/deploy.env.example`；若混入 app 运行时变量，则必须拆分后再保留
- 无法判断归属的旧 env 模板，停止并追问用户

## dev 与 deploy 的共享规则

- `scripts/dev.sh` 读取与当前 app 对应的 `.env.dev` / `.env.local`
- `scripts/deploy.sh` 读取与当前 app 对应的 `.env.prod`
- 若存在 `docker/deploy.env`，仅用于部署编排或 Docker 构建层变量
- Dockerfile 与部署脚本使用同一套镜像源变量命名
- Docker 构建参数不能和本地脚本使用不同变量名

## 云厂商默认值

执行时要主动识别或询问用户当前使用的云厂商，并给出默认值，避免用户自己去查。默认值来源允许两种方式：

1. skill 内置常见云厂商预设
2. skill 运行时联网查询公开资料

无论用哪种方式，都必须遵守：

1. 先确认用户当前使用的云厂商或运行环境
2. 默认值只能作为初始建议，必须可覆盖
3. 最终写入生成结果时，要明确哪些值是默认值

## 国内源覆盖策略

当用户未指定源时：

- 可以给出常见国内源作为默认建议
- 不能默认假设某一家云厂商
- 不能因为当前机器已有 npm / pip 配置，就直接固化到目标仓库

当用户提供企业内网源时：

- 以内网源为最高优先级
- 继续保留公开源变量位，方便回退或多环境覆盖

## 禁止行为

- 重新引入根目录统一 `.env.example` 作为所有 app 的唯一 env 入口
- 把 `docker/deploy.env.example` 当成所有 app 运行时变量的唯一来源
- 把 `NPM_REGISTRY`、`PIP_INDEX_URL`、`GO_PROXY`、`APT_MIRROR`、`DOCKER_MIRROR` 写死成不可改
- `dev` 与 `deploy` 使用不同命名的变量表达同一个源
- 没问云厂商，就直接套某家云厂商默认值
- 把当前机器的私有配置直接泄露进模板
