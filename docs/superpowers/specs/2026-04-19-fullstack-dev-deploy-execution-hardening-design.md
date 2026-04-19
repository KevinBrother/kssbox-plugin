# fullstack-dev-deploy Skill 执行约束强化设计

## 背景

`fullstack-dev-deploy` 已经具备较完整的治理目标，但实际落到目标仓库时，仍可能出现“生成了能跑的脚本，却没有严格收敛到目标资产形态”的偏差。

这类偏差主要表现为：

- 直接沿用根级 `.env.example`，没有收敛到 app 自治 env
- 继续使用单个 `docker-compose.yml`，没有形成 `infra + app` 双 compose
- 对旧脚本、旧 compose、旧 Dockerfile、旧 env 只做兼容，没有完成治理闭环
- 在 app 边界、资产语义不够确定时，没有停下来追问
- 缺少结构化验证摘要，导致“生成完成”和“规范完成”混淆

本设计的目标不是新增功能，而是把现有 skill 从“规则较强的生成器”强化为“带硬门禁的治理型 skill”。

## 设计目标

### 目标内

- 让案例示例更好地引导模型，但不把案例名带入真实项目
- 强化 Discovery 的结构化产物，避免后续阶段各自发挥
- 明确哪些情况必须停下追问用户
- 把最终资产形态设为硬门禁，而不是建议项
- 把验证和输出摘要设为交付门禁，而不是可选项

### 目标外

- 不改变该 skill 面向任意全栈项目的定位
- 不把服务分组写死成某几个业务名
- 不扩展到 Kubernetes 或其他更复杂编排系统

## 设计总览

强化后的 `fullstack-dev-deploy` 采用以下约束：

1. **案例名只用于示范，不可复用为真实项目服务名**
2. **运行时必须识别目标仓库中的真实 app 名称**
3. **Discovery 必须产出固定结构的项目画像**
4. **语义不确定时必须停下追问**
5. **最终资产未收敛到目标形态，则视为未完成**
6. **未完成验证与摘要输出，则视为未完成**

## 一、案例与命名规则

### 1.1 案例结构

skill 中的案例按两层组织：

1. **主案例**：单前端 + 单后端
2. **扩展示例**：多前端 + 多服务端

这样既能给模型一个最常见的默认心智模型，也能提醒它考虑微前端、微服务和多构建产物场景。

### 1.2 案例名的使用边界

案例里的命名只是结构示意，不是运行时接口模板。

例如：

```text
frontend + backend
frontend-h5 + backend-api
frontend-h5 + frontend-weapp + backend
web-admin + web-portal + api-gateway + user-service
```

这些名字只用于向模型说明“可能存在多个 app、多个前端、多个服务端”，不允许被直接照搬到目标项目里。

### 1.3 运行时命名规则

真正执行 skill 时，必须扫描目标仓库并识别实际 app 名称和实际服务边界，例如：

- `backend`
- `h5`
- `weapp`
- `admin-web`
- `gateway`
- `user-service`

默认统一入口规则为：

- 支持 `all`
- 支持实际识别出的服务名
- 如果仓库或用户需求中存在更高层服务集合语义，可在确认后额外支持聚合组

禁止把案例中出现的名字直接当作真实项目参数接口。

## 二、Discovery 与停机规则

### 2.1 Discovery 的唯一事实来源地位

Discovery 不再只是“扫描一下目录”，而必须形成后续阶段共用的唯一项目画像。后续 `dev`、`deploy`、验证、治理摘要都只能消费同一份 Discovery 结果。

### 2.2 必须产出的结构

Discovery 结束后，逻辑上必须形成如下结构：

```text
projectProfile{
  apps[]
  serviceGroups
  existingArtifacts
  assetInventory[]
  convergencePlan[]
  assumptions[]
}
```

其中：

- `apps[]`：真实 app 列表、目录、类型、构建命令、启动命令、端口、依赖
- `serviceGroups`：至少包含 `all`；其他组只有在可稳定识别或用户确认时才存在
- `existingArtifacts`：已有脚本、Dockerfile、Compose、env 文件
- `assetInventory[]`：每个旧资产的语义、归属、当前职责
- `convergencePlan[]`：每个资产的 `keep / migrate / merge / delete / generate`
- `assumptions[]`：可继续推进但尚未完全确认的推断项

### 2.3 必须停下追问的情况

出现以下任一情况，必须停止自动推进并追问用户：

- 无法确定哪些目录是独立 app
- 无法确定某个旧脚本是 dev、deploy 还是其他用途
- 无法确定某个 compose 文件属于 infra 还是 app
- 无法确定某个 env 文件属于哪个 app 或哪个环境
- 只能看到案例示意名，无法映射到真实项目名称
- 多个候选分组或多个候选服务边界同时成立，且证据冲突

这条规则的核心是：**不能用案例语义替代项目语义，不能用猜测代替资产识别。**

## 三、最终资产门禁

### 3.1 最终目标形态

skill 可以自适应分析任意全栈项目，但最终资产形态必须收敛到标准目标：

- `scripts/dev.sh`
- `scripts/deploy.sh`
- `docker/infra.compose.yml`
- `docker/app.compose.yml`
- `docker/<app>/Dockerfile`
- `<app>/.env.dev.example`
- `<app>/.env.prod.example`
- `<app>/.env.local.example`

### 3.2 旧资产的处理原则

如果目标仓库中已有如下资产：

- 根级 `docker-compose.yml`
- 散落的 Dockerfile
- 根级 `.env.example`
- 多个旧启动脚本或旧部署脚本

那么这些资产只能作为治理输入进入 `assetInventory` 和 `convergencePlan`，不能直接作为最终标准资产保留下来，除非它们已经完全符合目标形态。

### 3.3 硬门禁判定

出现以下任一结果，都应直接判定为“未按 skill 完成”：

- 最终仍只有单个 compose，没有拆出 `infra.compose.yml` 与 `app.compose.yml`
- 最终仍依赖根级统一 `.env.example`，没有 app 自治 env
- 新入口已生成，但旧资产没有被说明如何 keep / migrate / merge / delete / generate
- `dev` 与 `deploy` 使用了不同的 app 命名口径或服务映射口径

这意味着：**可以自适应项目结构，但不能自适应降低最终资产标准。**

## 四、验证与输出纪律

### 4.1 Dev 验证门禁

`scripts/dev.sh` 不仅要存在，还必须验证：

- 空参是否等价于 `all`
- `all` 与真实服务名是否展开正确
- 依赖 infra 是否已正确联动
- app 是否具备可证明的成功启动信号

成功信号至少满足其一：

- 端口监听成功
- healthcheck 成功
- ready 信号成功
- 现有 smoke test 成功

### 4.2 Deploy 验证门禁

`scripts/deploy.sh` 不仅要存在，还必须验证：

- `docker/infra.compose.yml` 与 `docker/app.compose.yml` 均可工作
- `docker/<app>/Dockerfile` 能参与正确构建
- 部署后的容器或服务具备 readiness 信号

### 4.3 输出摘要门禁

最终输出必须显式包含：

1. 识别到的 app 列表
2. `all` 与其他实际服务集合映射
3. 旧资产的治理动作
4. 最终保留的标准资产
5. 新建、修改、删除的文件
6. 按假设继续执行的地方
7. 已执行的验证与结果

如果缺少上述摘要，即使文件已经生成，也不应视为完整交付。

## 五、对案例文案的直接修改建议

### 5.1 示例风格

正文中的示例建议改成：

- 主案例：单前端 + 单后端
- 扩展示例：多前端 + 多服务端

### 5.2 示例说明语句

应增加类似下面的明确说明：

> 示例中的服务名仅用于说明仓库可能存在多个前端和多个服务端，不代表目标项目的真实参数接口。执行时必须识别目标项目中实际存在的服务名称，并据此生成统一入口和治理计划。

### 5.3 服务集合规则

应把 `all / biz / boss` 从“默认约定”降级为“示例性服务集合写法”，改成更通用的描述：

> 统一入口默认支持 `all`，并支持目标项目中实际识别出的服务名或服务集合名。若仓库中不存在稳定的聚合语义，则至少保留 `all + 实际服务名`。

这样既保留了统一入口思想，也避免模型把少数业务词误当成固定模板。

## 六、预期效果

强化后，skill 应具备以下行为特征：

- 会先识别真实项目服务名，而不是照搬案例名
- 会先判断旧资产语义，再决定是否迁移、合并或删除
- 在无法安全识别时会停下追问，而不是继续猜
- 不会把“生成了几个脚本”误当成“治理已完成”
- 最终必须交付统一入口、双 compose、app 自治 env、验证结果和治理摘要

这能显著降低在真实仓库中出现“表面统一、实际未收敛”的情况。
