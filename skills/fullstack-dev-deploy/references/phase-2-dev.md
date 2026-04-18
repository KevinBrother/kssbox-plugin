# 阶段二：生成 dev 入口

## 目标

把目标项目现有的启动方式统一收敛为一个入口：`scripts/dev.sh [service...]`，并确保脚本执行后能确认目标服务已经启动成功。

## 必须输出的资产

- `scripts/dev.sh`
- 必要时更新 `.env.example`
- 必要时补充本地开发说明

## 统一入口约定

脚本入口固定为：

```bash
./scripts/dev.sh [service...]
```

行为要求：

- 不传参数时默认 `all`
- `./scripts/dev.sh biz`：启动 `biz` 对应的前后端集合
- `./scripts/dev.sh boss`：启动 `boss` 对应的前后端集合
- `./scripts/dev.sh biz boss`：同时启动多个服务集合

## 生成规则

1. 先复用 Discovery 产出的 `serviceGroups`
2. 再把 `all`、`biz`、`boss` 展开为具体 app 清单
3. 为每个 app 选择正确的启动命令
4. 对复杂项目允许内部按 app 级执行，但外部接口保持不变
5. 启动前统一加载环境变量契约

## 参数解析要求

- 必须显式处理 `all`、`biz`、`boss`
- 空参必须等价于 `all`
- 重复参数要去重
- 非法参数不能静默忽略，必须明确报错

## 复杂项目处理

复杂项目允许内部映射为 app 级清单，例如：

```text
biz  -> frontend-biz + backend-biz
boss -> frontend-boss + backend-boss
all  -> 全部 app
```

如果只有前端或只有后端能识别出来，不要自己补全另一侧，必须停下追问用户。

## 环境变量约定

`scripts/dev.sh` 必须在启动前加载统一环境变量，例如：

- `.env`
- `.env.local`
- `.env.example` 中定义的镜像源和端口覆盖项

环境变量契约本身以 `mirror-and-env-contract.md` 为准，这里只负责消费。

## 启动成功验证

生成脚本后，不能只“发命令”，还要验证目标服务是否真的起来。至少要满足其一：

- 端口监听成功
- 项目已有 healthcheck 成功
- 项目已有 ready 信号成功
- 项目已有 smoke test 成功

如果仓库已有 health / ready / smoke 机制，优先复用，避免重复造轮子。

## 输出要求

执行 `dev.sh` 后，日志中必须清楚说明：

- 本次实际展开了哪些 app
- 每个 app 使用了什么启动命令
- 哪个检测信号证明启动成功

## 禁止行为

- 只生成 `scripts/dev.sh`，不验证启动成功
- 遇到未知参数时默认按 `all` 执行
- 为了统一入口而丢失已有 app 的关键启动参数
- 把 `biz`、`boss` 直接写死在脚本里，却不和 Discovery 结果同步
