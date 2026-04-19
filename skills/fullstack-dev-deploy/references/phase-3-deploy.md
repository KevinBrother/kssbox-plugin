# 阶段四：生成 docker deploy

## 目标

把目标项目统一收敛为单机 Docker / Compose 部署方案，输出固定入口 `scripts/deploy.sh [service...]`，并把部署资产收敛为双 compose 结构：

- `docker/infra.compose.yml`
- `docker/app.compose.yml`

## 必须输出的资产

- `scripts/deploy.sh`
- `docker/infra.compose.yml`
- `docker/app.compose.yml`
- `docker/<app>/Dockerfile`

必要时可补充：

- Docker 安装脚本
- 源配置辅助脚本
- 部署说明文档

## 统一入口约定

脚本入口固定为：

```bash
./scripts/deploy.sh [service...]
```

行为要求：

- 不传参数时默认 `all`
- `./scripts/deploy.sh backend`：部署 `backend` 对应服务集合
- `./scripts/deploy.sh h5 weapp`：部署多个真实服务
- 若 Discovery 或用户确认了聚合组，可额外支持 `./scripts/deploy.sh mobile` 这类高层服务集合

## 治理规则

在生成新入口前，必须先处理旧 deploy / docker 资产：

- 旧 `deploy.sh`、`release.sh`、`publish.sh` 若语义正确但路径不规范，执行 `migrate`
- 多个旧 compose 文件共同承担部署职责时，执行 `merge`
- 旧 Dockerfile 若语义正确但路径混乱，迁移到 `docker/<app>/Dockerfile`
- 已被双 compose 和统一入口完整覆盖的旧资产，执行 `delete`
- 无法判断语义或归属的旧 deploy / docker 资产，停止并追问用户

## 生成规则

1. 复用 Discovery 产出的 `serviceGroups`
2. 把 `all`、真实服务名与可选聚合组映射为具体服务集合
3. 判断现有 compose 与 Dockerfile 属于 infra 还是 app
4. 将基础设施收敛到 `docker/infra.compose.yml`
5. 将业务服务收敛到 `docker/app.compose.yml`
6. 为每个 app 生成或修正 `docker/<app>/Dockerfile`
7. 根据项目画像判断 infra 是否也由本仓库托管
8. 若最终仍只有单 compose 或 Docker 资产未收敛到标准目标，则直接视为未完成

## 双 compose 规则

### infra.compose.yml

用于基础依赖设施，例如：

- MySQL
- Redis
- MQ
- MinIO

这些服务通常会被 dev 使用，也可能在 deploy 时被联动或单独管理。

### app.compose.yml

用于项目自身 frontend / backend / app 服务的编排。

业务服务与 infra 的生命周期不同，不应被强行揉成一份 compose。

## deploy.sh 的最小职责

- 解析参数，空参默认 `all`
- 参数口径必须与 `scripts/dev.sh` 完全一致，并复用同一份服务映射
- 校验 Docker 与 Compose 是否可用
- 根据服务集合选择 app compose 中的目标服务
- 判断 infra 是否需要联动
- 触发 `docker compose build`
- 触发 `docker compose up -d`
- 输出本次构建和部署的服务集合

## Docker 化原则

- 所有构建必须在容器内完成
- 不依赖宿主机已有 Node、Go、Python 等语言运行时
- 优先使用多阶段构建
- `docker/<app>/Dockerfile` 必须与 app 技术栈匹配
- 如果目标项目已有 Dockerfile，可复用其意图，但最终资产仍统一收口到 `docker/`

## 宿主机要求

宿主机只要求：

- Docker
- Docker Compose

除此之外，不要求宿主机预装语言运行时。若缺少 Docker / Compose，应明确报错或生成安装引导，而不是继续执行。

## 部署后验证

部署完成后，不能只看到容器拉起，还要至少有一个 readiness 信号：

- healthcheck 成功
- 端口探测成功
- 容器健康状态正常
- 项目已有 smoke test 成功

## 输出要求

执行 `deploy.sh` 后，日志中必须清楚说明：

- 本次实际部署了哪些 app 服务
- 本次使用了哪些真实服务名或聚合组参数
- 是否联动了 `docker/infra.compose.yml`
- 使用了哪些 Dockerfile
- 哪些旧 deploy / docker 资产被 migrate / merge / delete
- 执行了哪些 `docker compose build` / `docker compose up -d` 动作
- 哪个 readiness 信号证明部署成功

## 禁止行为

- 把 infra 和 app 全部揉进一个唯一 compose 文件
- 依赖宿主机语言运行时来完成构建
- `scripts/deploy.sh` 和 `scripts/dev.sh` 使用不同的服务分组口径
- 最终仍保留单个 `docker-compose.yml` 作为唯一标准部署资产
- 发现已有 Dockerfile 或旧 compose 不规范，就直接忽略而不是说明如何收敛
