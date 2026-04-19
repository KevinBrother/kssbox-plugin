# 镜像源与单一 docker env 契约

## 目标

为 `dev` 和 `deploy` 阶段提供一套统一、可覆盖的镜像源变量约定，并将所有运行时配置统一收敛到 `docker/.env.example -> docker/.env` 单一 env 契约。

## 设计原则

- 不把国内源写死成不可覆盖常量
- 本地脚本和 Docker build 使用同一套镜像源变量命名
- 优先通过环境变量注入，而不是把源写进脚本逻辑
- 云厂商内网源允许覆盖默认值
- 整个项目只维护一份 `docker/.env.example`
- 运行时只读取 `docker/.env`
- 自动生成的 env key 统一使用全大写蛇形命名

## 推荐镜像变量

- `NPM_REGISTRY`
- `PNPM_REGISTRY`
- `YARN_REGISTRY`
- `PIP_INDEX_URL`
- `GO_PROXY`
- `APT_MIRROR`
- `DOCKER_MIRROR`

如果目标项目使用其他生态，可以扩展变量，但命名和职责必须清晰。

## 单一 env 目标形态

整个项目只保留：

- `docker/.env.example`
- `docker/.env`

要求说明：

- `docker/.env.example` 是唯一模板文件
- `docker/.env` 由开发者或部署目标通过 `cp docker/.env.example docker/.env` 生成
- `dev.sh`、`deploy.sh`、Compose、Docker build 全部只读取 `docker/.env`
- 真实 `docker/.env` 不应提交到 git

### 变量分组建议

虽然采用单一 env 文件，但仍应通过注释分组保持可读性，例如：

- `# Server`
- `# Database`
- `# Cache`
- `# Frontend Build`
- `# Deployment`
- `# Mirrors`

### 命名规范

- 所有 env key 必须全大写
- 使用下划线分隔
- 同一职责只允许一个稳定命名

例如：

- `SERVER_PORT`
- `DATABASE_URL`
- `REDIS_URL`
- `NPM_REGISTRY`

禁止出现未说明职责差异的重复命名，例如：

- `PORT` 与 `SERVER_PORT`
- `APP_HOST` 与 `SERVER_HOST`

## 治理规则

在生成新 env 结构前，必须先处理旧 env 资产：

- 根目录 `.env*` 若承担项目运行时配置，迁移到 `docker/.env.example`
- app 内 `.env*` 若表达的是项目运行时配置，合并后收敛到 `docker/.env.example`
- `docker/deploy.env*` 的变量也必须并入 `docker/.env.example`
- 已被单一 env 契约完整覆盖的旧 env 文件，执行 `delete`
- 无法判断归属或同名变量互相冲突时，停止并追问用户

## dev 与 deploy 的共享规则

- `scripts/dev.sh` 只读取 `docker/.env`
- `scripts/deploy.sh` 只读取 `docker/.env`
- Compose 与 Docker build 只读取 `docker/.env`
- Dockerfile 与部署脚本使用同一套镜像源变量命名
- Docker 构建参数不能和本地脚本使用不同变量名

如果 `docker/.env` 缺失，脚本提示必须统一为：

```bash
✗ Missing env file: docker/.env
  Expected template: docker/.env.example
  Run: cp docker/.env.example docker/.env
```

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

- 重新引入 `.env.dev`、`.env.prod`、`.env.local` 或 app 内 `.env*` 作为主契约
- 继续保留 `docker/deploy.env.example` 作为独立主契约
- 把 `NPM_REGISTRY`、`PIP_INDEX_URL`、`GO_PROXY`、`APT_MIRROR`、`DOCKER_MIRROR` 写死成不可改
- `dev` 与 `deploy` 使用不同命名的变量表达同一个源
- 没问云厂商，就直接套某家云厂商默认值
- 把当前机器的私有配置直接泄露进模板
