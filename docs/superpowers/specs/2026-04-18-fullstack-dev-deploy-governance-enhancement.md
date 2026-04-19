# fullstack-dev-deploy Skill 治理增强设计

## 背景

现有 `fullstack-dev-deploy` 设计已经覆盖了统一 `scripts/dev.sh`、`scripts/deploy.sh`、Docker 资产、镜像源契约和验证要求，但还不够强调“已有项目资产治理”。

目标项目往往不是空白仓库，而是已经存在：

- 旧的 dev 脚本，例如 `run.sh`、`start.sh`、`dev.sh`
- 旧的 deploy 脚本，例如 `deploy.sh`、`release.sh`
- 旧的 Docker 资产，例如散落的 Dockerfile、多个 compose 文件
- 旧的 env 资产，例如根级 `.env*`、脚本旁边的临时 env 文件

如果 skill 只会“新增标准资产”，而不会识别、迁移、合并、删除旧资产，最终仓库会长期存在新旧并存、入口重复、职责混乱的问题，不利于开发、维护和部署。

因此，本次增强的目标不是单纯补能力，而是把 `fullstack-dev-deploy` 升级成一个**生成 + 治理**型 skill。

## 新定位

增强后的 `fullstack-dev-deploy` 不再只是生成器，而是默认运行在**治理优先模式**下：

1. 如果目标项目没有相关资产，则按统一规则新建
2. 如果目标项目已有相关资产，则先识别语义，再按统一规则收敛
3. 最终仓库必须回到一套干净、单一、可维护的目标形态

默认原则如下：

- **符合目标形态**：保留
- **语义正确但形态不符合**：重命名、迁移、合并
- **重复、冗余、被标准资产覆盖**：删除
- **语义无法判断**：停止并追问用户

也就是说，增强后的 skill 默认不会问用户“要不要迁移”或“要不要删除”，而是按规则自动治理；只有在连旧资产原本职责都无法判断时，才向用户追问。

## 总体架构增强

原有设计中的 `Discovery -> Dev -> Deploy -> Validation` 链路，需要增强为：

```text
读取用户需求
  -> 扫描 app 结构
  -> 扫描现有资产
  -> 生成项目画像
  -> 生成资产治理计划
  -> 执行 Dev 收敛
  -> 执行 Deploy 收敛
  -> 清理冗余资产
  -> 输出治理摘要与验证结果
```

其中新增两个关键概念：

### 1. assetInventory

用于盘点目标仓库中所有与 `dev/docker/deploy/env` 相关的现有资产。

### 2. convergencePlan

用于描述每个资产应采取的治理动作：

- `keep`
- `migrate`
- `merge`
- `delete`
- `generate`

增强后，skill 的输出不只是“生成了什么”，还必须回答“保留了什么、迁移了什么、删除了什么”。

## Asset Inventory 设计

### 扫描范围

skill 需要显式盘点以下资产：

- 本地启动脚本：`run.sh`、`start.sh`、`dev.sh`、`scripts/*`
- 部署脚本：`deploy.sh`、`release.sh`、`publish.sh`
- Dockerfile：根目录或各 app 下已有 Dockerfile
- Compose 文件：`docker-compose*.yml`、`compose*.yml`
- env 文件：`.env*`、`*.env*`
- Docker 安装与源配置脚本

### 输出结构

逻辑上，Discovery 阶段的输出应扩展为：

```text
projectProfile{
  apps[]
  serviceGroups
  existingArtifacts
  assetInventory[]
  convergencePlan[]
  toolchains
  assumptions[]
}
```

其中：

- `assetInventory[]`：现有资产清单
- `convergencePlan[]`：每个资产的治理动作与原因

### 资产盘点原则

- 不只按文件名判断，必须结合内容和调用关系识别语义
- 不把“旧文件”当作删除理由，删除的唯一依据是职责已被标准资产完整吸收
- Discovery 与后续治理阶段必须共用同一份资产清单，不允许不同阶段各说各话

## 治理动作模型

### Keep

满足以下条件时保留：

- 已符合目标路径与命名
- 职责单一清晰
- 能直接纳入统一入口体系

例如：

- 已规范存在的 `scripts/dev.sh`
- 已符合规则的 `docker/<app>/Dockerfile`

### Migrate / Rename

满足以下条件时迁移或重命名：

- 语义正确
- 但路径、命名、目录层级不符合目标形态

例如：

- 根目录 `run.sh` 迁移为 `scripts/dev.sh`
- 根目录旧 `deploy.sh` 迁移为 `scripts/deploy.sh`

### Merge

满足以下条件时合并：

- 多个旧资产共同承担一个标准资产的职责
- 合并后仓库职责会更清晰

例如：

- 多个旧 compose 文件合并到标准 compose 结构
- 多个旧 wrapper 脚本收敛到统一入口

### Delete

满足以下条件时删除：

- 已被新标准资产完整覆盖
- 不再承担独立职责
- 继续保留只会制造维护歧义

例如：

- 被统一入口取代的旧 wrapper
- 已收敛完成的历史 compose 变体
- 冗余的旧 docker 启动脚本

### Generate

满足以下条件时新建：

- 标准资产缺失
- 仓库需要该资产才能达到目标形态

例如：

- 缺失的 `scripts/dev.sh`
- 缺失的 `docker/<app>/Dockerfile`

## 目标形态增强

增强后的目标形态不再是“单 compose + 中心化 env”，而是更贴近真实多服务项目。

### 统一入口

- `scripts/dev.sh`
- `scripts/deploy.sh`

### Docker 资产

应拆成两层：

- `docker/infra.compose.yml`
- `docker/app.compose.yml`
- `docker/<app>/Dockerfile`

这样可以把基础设施和业务服务分开治理。

#### infra.compose.yml

用于基础依赖设施，例如：

- MySQL
- Redis
- MQ
- MinIO

这些服务通常会被 dev 使用，也可能在 deploy 时被联动或单独管理。

#### app.compose.yml

用于项目自身 frontend / backend / app 服务的编排。

业务服务与 infra 的生命周期不同，不应被强行揉成一份 compose。

## Dev / Deploy 行为增强

### Dev

`scripts/dev.sh [service...]` 在增强后应具备：

1. 识别目标 app 所依赖的 infra
2. 若 infra 缺失，优先拉起 `docker/infra.compose.yml`
3. 启动目标 app
4. 验证 infra 可用且 app 成功启动

也就是说，dev 阶段不只是“跑脚本”，而是会先收敛旧 dev 资产，再在统一入口下运行。

### Deploy

`scripts/deploy.sh [service...]` 在增强后应具备：

1. 以 `docker/app.compose.yml` 为主要部署入口
2. 根据项目画像判断 infra 是否也由本仓库托管
3. 若 infra 由本仓库托管，则按规则联动
4. 若 infra 属于外部托管，则只验证其可达性，不强行部署
5. 验证 app compose 构建、启动与 readiness

## Env 契约增强

### 原则

不再采用中心化 env 聚合文件，而采用**app 自治 env**。

### 标准形态

每个 app 自己维护自己的 env 模板，例如：

- `<app>/.env.dev.example`
- `<app>/.env.prod.example`
- `<app>/.env.local.example`

实际运行时：

- 真实 `.env`、`.env.local` 由开发者或部署目标从 example copy 生成
- 真实 `.env*` 不应提交到 git

### 技术含义

- `dev.sh` 和 `deploy.sh` 只负责根据运行模式选择正确的 app env
- 不再维护一个巨大的根级 env 汇总表
- 多服务项目的变量职责更清晰，不互相污染

### 治理要求

增强后的 skill 必须识别现有 env 文件的归属：

- 是某个 app 的私有 env
- 是全局历史遗留 env
- 是临时/冗余/重复 env

再决定：

- 保留
- 迁移到 app 内
- 合并
- 删除

## 输出摘要增强

执行结束时，除了原有的 app 识别结果和验证结果，还必须额外输出：

### 1. 资产盘点摘要

- 识别到了哪些旧脚本
- 识别到了哪些旧 Docker 资产
- 识别到了哪些旧 env 文件

### 2. 治理动作摘要

对每个资产说明其动作：

- `keep`
- `migrate`
- `merge`
- `delete`
- `generate`

并说明理由。

### 3. 最终目标形态摘要

明确说明最终仓库保留了哪些标准资产：

- `scripts/dev.sh`
- `scripts/deploy.sh`
- `docker/infra.compose.yml`
- `docker/app.compose.yml`
- `docker/<app>/Dockerfile`
- 各 app 自己的 `.env.*.example`

### 4. 假设项与例外

如果某些旧资产未被删除，必须说明：

- 为什么保留
- 是否还承担独立职责
- 后续应如何处理

## 清理保证

增强后的 skill 需要显式承诺：

- 不让新旧入口长期并存
- 不让职责已被吸收的旧文件继续残留
- 不让多个 compose 入口持续误导维护者
- 不让中心化 env 设计重新污染多服务项目

如果最终仍保留旧资产，必须在输出摘要中解释原因。

## 追问边界

增强后的 skill 默认按规则自动治理，只有以下情况才追问用户：

- 无法判断某个旧脚本原本承担什么职责
- 无法判断某个旧 compose 属于 infra 还是 app
- 无法判断某个 env 文件属于哪个 app 或哪个环境
- 多个候选语义互相冲突，继续自动治理会有高风险

也就是说，用户不需要确认“要不要删”，用户只需要在**语义无法识别**时提供补充信息。

## 设计结论

这轮增强的核心不是增加更多模板，而是把 `fullstack-dev-deploy` 升级为一个真正的**项目资产治理型 skill**：

- 对空白项目，能从 0 到 1 建立标准入口
- 对已有项目，能自动识别旧资产并完成强收敛
- 对多服务项目，能区分 infra 与 app 的 compose 生命周期
- 对 env，采用 app 自治、真实 `.env*` 不入 git 的治理模型

最终交付的不是“又多一套文件”，而是一套**被治理过、足够简洁、便于开发维护与部署**的目标仓库形态。
