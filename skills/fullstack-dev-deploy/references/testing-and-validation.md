# 测试与验证

## 目标

这个 skill 的交付标准不是“文件生成了”，而是“治理后的统一入口真实可用，且旧资产已正确收敛”。所有关键生成物都必须进入验证链路。

## 验证目标

1. `scripts/dev.sh` 参数解析正确
2. `scripts/dev.sh` 对 `all + 真实服务名 + 已确认聚合组` 的展开正确
3. `scripts/dev.sh` 命中的启动命令正确
4. `scripts/dev.sh` 能证明 infra 和 app 已启动成功
5. `scripts/deploy.sh` 参数解析正确
6. `scripts/deploy.sh` 能在本机 Docker / Compose 环境下执行
7. `docker/infra.compose.yml` 与 `docker/app.compose.yml` 结构正确
8. `docker/<app>/Dockerfile` 与双 compose 可工作
9. 旧 dev / deploy / docker / env 资产已按治理动作收敛

## 测试分层

### 第一层：静态校验

- 脚本文件存在
- 参数分支完整
- `docker/.env.example` 存在
- `scripts/dev.sh` 与 `scripts/deploy.sh` 都指向 `docker/.env`
- `docker/infra.compose.yml` 与 `docker/app.compose.yml` 同时存在
- 默认 `all` 行为存在
- env key 命名符合全大写蛇形规范

### 第二层：治理校验

- 旧 `run.sh`、`start.sh`、旧 `dev.sh` 是否已被 keep / migrate / merge / delete
- 旧 `deploy.sh`、旧 compose、旧 Dockerfile 是否已按规则收敛
- 根目录 `.env*`、app 内 `.env*`、`docker/deploy.env*` 是否已迁移、合并或删除
- 最终保留资产是否与 `convergencePlan` 一致

### 第三层：Dev 行为验证

- 空参是否等价于 `all`
- 真实服务名与已确认聚合组是否映射到正确 app 集合
- 是否按需要联动 `docker/infra.compose.yml`
- 缺失 `docker/.env` 时是否明确提示具体路径和复制命令
- `docker/.env` 存在但关键变量缺失时是否明确列出变量名
- 启动命令是否命中目标 app
- 是否有端口、health、ready 或 smoke 信号证明启动成功
- 提前退出时是否避免 `PIDS[@]: unbound variable`

若目标仓库已有现成的 healthcheck、ready 或 smoke test，优先复用。

### 第四层：Deploy 本机验证

- `docker/<app>/Dockerfile` 能成功构建
- `docker/app.compose.yml` 能按服务集合启动
- 若 infra 由本仓库托管，`docker/infra.compose.yml` 能被正确联动
- `scripts/deploy.sh` 能在本机 Docker 环境执行
- 缺失 `docker/.env` 时是否明确提示具体路径和复制命令
- 部署后必须至少有一个 readiness 信号：healthcheck、端口探测、容器健康状态或 smoke test

## 失败处理

- 第一次失败：修正生成物后重试
- 第二次仍失败且原因不明：立即停止，向用户报告
- 缺少 Docker / Compose 或测试前置条件：明确报错，不得伪造“验证通过”
- 无法判断旧资产语义：停止，不得继续自动治理
- 若缺少治理摘要或无法说明假设项：直接判定未完成，不得声称交付

## 必须验证的生成物

- `scripts/dev.sh`
- `scripts/deploy.sh`
- `docker/infra.compose.yml`
- `docker/app.compose.yml`
- `docker/<app>/Dockerfile`
- `docker/.env.example`

## 禁止行为

- 只做静态检查，不做治理验证与运行验证
- `dev.sh` 启动后不确认 infra 或 app 是否真的可用
- `deploy.sh` 只跑 build，不验证部署结果
- 没核对旧资产去向，就声称“已完成收敛”
- 把“用户手动看起来正常”当作唯一验证依据
- 未列出真实服务名、聚合组、治理动作和验证结果，就声称“验证通过”
- 缺失 `docker/.env` 时不提示路径和复制命令
- 提前失败时仍出现 `PIDS[@]: unbound variable`
