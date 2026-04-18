# 阶段三：生成 dev 入口

## 目标

把目标项目现有的启动方式统一收敛为一个入口：`scripts/dev.sh [service...]`，并在统一入口下治理旧 dev 资产、联动基础设施、按 app 选择环境变量，最终确认目标服务已经启动成功。

## 必须输出的资产

- `scripts/dev.sh`
- 各 app 自己的 `.env.dev.example`
- 各 app 自己的 `.env.local.example`（如该 app 需要本地私有覆盖）
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

## 治理规则

在生成新入口前，必须先处理旧 dev 资产：

- 旧 `run.sh`、`start.sh`、旧 `dev.sh` 若语义正确但路径不规范，执行 `migrate`
- 多个 wrapper 脚本共同承担启动职责时，执行 `merge`
- 已被 `scripts/dev.sh` 完整覆盖的旧入口，执行 `delete`
- 无法判断语义的旧 dev 资产，停止并追问用户

## 生成规则

1. 先复用 Discovery 产出的 `serviceGroups`
2. 把 `all`、`biz`、`boss` 展开为具体 app 清单
3. 判断每个 app 所依赖的基础设施
4. 若 infra 缺失，优先联动 `docker/infra.compose.yml`
5. 为每个 app 选择正确的启动命令
6. 对复杂项目允许内部按 app 级执行，但外部接口保持不变
7. 启动前按 app 加载运行时环境变量

## 参数解析要求

- 必须显式处理 `all`、`biz`、`boss`
- 空参必须等价于 `all`
- 重复参数要去重
- 非法参数不能静默忽略，必须明确报错

## app 自治 env

`scripts/dev.sh` 不再依赖中心化 `.env.example`，而是按 app 读取运行时环境。

每个 app 至少应具备：

- `<app>/.env.dev.example`
- `<app>/.env.local.example`（如需要）

实际运行时：

- `.env.dev`、`.env.local` 由开发者从 example copy 生成
- 真实 `.env*` 不应提交到 git

## 复杂项目处理

复杂项目允许内部映射为 app 级清单，例如：

```text
biz  -> frontend-biz + backend-biz
boss -> frontend-boss + backend-boss
all  -> 全部 app
```

如果只有前端或只有后端能识别出来，不要自己补全另一侧，必须停下追问用户。

## 启动成功验证

生成脚本后，不能只“发命令”，还要验证 infra 和 app 是否真的起来。至少要满足：

- 依赖的 infra 可用
- app 端口监听、healthcheck、ready 信号或 smoke test 至少有一个成功

如果仓库已有 health / ready / smoke 机制，优先复用，避免重复造轮子。

## 输出要求

执行 `dev.sh` 后，日志中必须清楚说明：

- 本次实际展开了哪些 app
- 哪些旧 dev 资产被 migrate / merge / delete
- 是否联动了 `docker/infra.compose.yml`
- 每个 app 使用了什么启动命令
- 每个 app 选择了哪类 env 模式
- 哪个检测信号证明启动成功

## 禁止行为

- 只生成 `scripts/dev.sh`，不治理旧 dev 入口
- 遇到未知参数时默认按 `all` 执行
- 为了统一入口而丢失已有 app 的关键启动参数
- 继续依赖根级 `.env.example` 作为所有 app 的统一 env 入口
- 把 `biz`、`boss` 直接写死在脚本里，却不和 Discovery 结果同步
