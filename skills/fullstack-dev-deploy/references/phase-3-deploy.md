# 阶段三：生成 docker deploy

## 目标

把目标项目统一收敛为单机 Docker / Compose 部署方案，输出固定入口 `scripts/deploy.sh [service...]`，并把所有部署资产统一放在 `docker/` 目录下。

## 必须输出的资产

- `scripts/deploy.sh`
- `docker/docker-compose.yml`
- `docker/<app>/Dockerfile`
- `docker/deploy.env.example`

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
- `./scripts/deploy.sh biz`：部署 `biz` 对应服务集合
- `./scripts/deploy.sh boss`：部署 `boss` 对应服务集合
- `./scripts/deploy.sh biz boss`：部署多个服务集合

## 生成规则

1. 复用 Discovery 产出的 `serviceGroups`
2. 把 `all`、`biz`、`boss` 映射为具体服务集合
3. 为每个 app 生成或修正 `docker/<app>/Dockerfile`
4. 在 `docker/docker-compose.yml` 中统一编排服务
5. 把部署环境变量集中放进 `docker/deploy.env.example`

`docker/deploy.env.example` 是模板和契约文件，不应被 `deploy.sh` 直接当作生产运行时配置。

## deploy.sh 的最小职责

- 解析参数，空参默认 `all`
- 校验 Docker 与 Compose 是否可用
- 加载部署环境变量
- 读取实际部署环境文件，并遵循 `docker/deploy.env.example` 定义的变量契约
- 触发 `docker compose build`
- 触发 `docker compose up -d`
- 输出本次构建和部署的服务集合

## Docker 化原则

- 所有构建必须在容器内完成
- 不依赖宿主机已有 Node、Go、Python 等语言运行时
- 优先使用多阶段构建
- `docker/<app>/Dockerfile` 必须与 app 技术栈匹配
- 如果目标项目已有 Dockerfile，可复用其意图，但最终资产仍统一收口到 `docker/`

## Compose 组织原则

- `docker/docker-compose.yml` 是唯一默认编排入口
- 服务名与 Discovery 识别出的 app 名称保持一致
- `all`、`biz`、`boss` 的服务展开必须能复用到 compose 选择逻辑
- 不在本 skill 内扩展到 Kubernetes、Swarm、Nomad

## 宿主机要求

宿主机只要求：

- Docker
- Docker Compose

除此之外，不要求宿主机预装语言运行时。若缺少 Docker / Compose，应明确报错或生成安装引导，而不是继续执行。

## 输出要求

执行 `deploy.sh` 后，日志中必须清楚说明：

- 本次实际部署了哪些服务
- 使用了哪些 Dockerfile
- 执行了哪些 `docker compose build` / `docker compose up -d` 动作

## 禁止行为

- 把 Compose 文件散落到多个默认路径，而不统一到 `docker/docker-compose.yml`
- 依赖宿主机语言运行时来完成构建
- `scripts/deploy.sh` 和 `scripts/dev.sh` 使用不同的服务分组口径
- 发现已有 Dockerfile 不规范，就直接忽略而不是说明如何收敛
