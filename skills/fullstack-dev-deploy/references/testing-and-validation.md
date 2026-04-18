# 测试与验证

## 目标

这个 skill 的交付标准不是“文件生成了”，而是“统一入口真实可用”。所有关键生成物都必须进入验证链路。

## 验证目标

1. `scripts/dev.sh` 参数解析正确
2. `scripts/dev.sh` 对 `all|biz|boss` 的展开正确
3. `scripts/dev.sh` 命中的启动命令正确
4. `scripts/dev.sh` 能证明服务已经启动成功
5. `scripts/deploy.sh` 参数解析正确
6. `scripts/deploy.sh` 能在本机 Docker / Compose 环境下执行
7. `docker/<app>/Dockerfile` 与 `docker/docker-compose.yml` 可工作

## 测试分层

### 第一层：静态校验

- 脚本文件存在
- 参数分支完整
- 环境变量读取逻辑存在
- 默认 `all` 行为存在

### 第二层：Dev 行为验证

- 空参是否等价于 `all`
- `biz` / `boss` 是否映射到正确 app 集合
- 启动命令是否命中目标 app
- 是否有端口、health、ready 或 smoke 信号证明启动成功

若目标仓库已有现成的 healthcheck、ready 或 smoke test，优先复用。

### 第三层：Deploy 本机验证

- `docker/<app>/Dockerfile` 能成功构建
- `docker/docker-compose.yml` 能按服务集合启动
- `scripts/deploy.sh` 能在本机 Docker 环境执行
- 若仓库已有 healthcheck / smoke test，纳入部署后验证

## 失败处理

- 第一次失败：修正生成物后重试
- 第二次仍失败且原因不明：立即停止，向用户报告
- 缺少 Docker / Compose 或测试前置条件：明确报错，不得伪造“验证通过”

## 必须验证的生成物

- `scripts/dev.sh`
- `scripts/deploy.sh`
- `docker/docker-compose.yml`
- `docker/<app>/Dockerfile`

## 禁止行为

- 只做静态检查，不做运行验证
- `dev.sh` 启动后不确认服务是否真的可用
- `deploy.sh` 只跑 build，不验证部署结果
- 把“用户手动看起来正常”当作唯一验证依据
