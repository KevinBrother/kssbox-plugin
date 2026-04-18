# 阶段一：Discovery 共享扫描

## 目标

把任意全栈仓库收敛成一份统一“项目画像”，供后续 `dev` 和 `deploy` 阶段共用，避免两个阶段各自扫描导致判断不一致。

## 必须扫描的信息

1. app 列表：仓库中有哪些 frontend / backend 或其他可运行单元
2. 技术栈：Node、Go、Python、静态前端等
3. 包管理器：npm、pnpm、yarn、go mod、pip、poetry 等
4. 启动命令、构建命令、默认端口、依赖关系
5. 已有资产：脚本、Dockerfile、Compose、env 文件、镜像源配置
6. 服务分组候选：哪些 app 可能属于 `biz`、`boss`

## 推荐扫描顺序

1. 先看目录结构：`frontend/`、`backend/`、`apps/`、`services/`
2. 再看 app 级配置：`package.json`、`go.mod`、`pyproject.toml`、`requirements.txt`
3. 再看已有启动与部署资产：`scripts/`、`Dockerfile`、`docker-compose*.yml`、`.env*`
4. 最后看 README 或项目脚本中的命名线索，补齐命令和端口信息

## 输出格式

Discovery 结束后，逻辑上必须形成如下结构：

```text
apps[]
serviceGroups{all,biz,boss}
existingArtifacts
toolchains
envContracts
assumptions
```

字段说明：

- `apps[]`：每个 app 的名称、目录、类型、启动方式、构建方式、端口
- `serviceGroups`：`all`、`biz`、`boss` 对应的 app 清单
- `existingArtifacts`：已有脚本、Dockerfile、Compose、env 文件
- `toolchains`：语言生态、包管理器、构建依赖
- `envContracts`：生成物后续要读取的环境变量
- `assumptions`：自动推断但尚未得到用户确认的部分

## 扫描原则

- **先复用，再生成**：已有脚本和 Docker 资产先识别语义，再决定复用或替换
- **关注 app，而不是目录名**：目录结构可以不标准，只要能稳定识别 app 即可
- **以执行为中心**：启动命令、构建命令、端口优先级高于目录美观
- **一个事实来源**：Discovery 产物是后续所有阶段的唯一输入，不允许各阶段私自改口径

## 分流规则

### 1. 可自动识别

满足以下条件时直接继续：

- app 边界清晰
- 启动命令和构建命令可找到
- `biz` / `boss` / `all` 分组可直接确认

### 2. 可推断但有风险

满足以下情况时允许继续，但必须记录到 `assumptions`：

- app 边界基本清楚，但命名不统一
- `biz` / `boss` 可以从目录、脚本或端口中高概率推断
- 旧脚本语义存在但命名混乱

### 3. 无法安全识别

出现以下情况必须停下并追问用户：

- 无法确定哪些目录是独立 app
- 找不到可靠的启动命令或构建命令
- 无法判断 `biz`、`boss` 分组
- 发现多个互相冲突的 Docker / Compose 方案

## 禁止行为

- 看到目录名像业务模块，就直接当成 `biz` 或 `boss`
- 只凭 README 描述，不交叉验证实际脚本和配置
- Discovery 和 Dev / Deploy 使用不同的 app 名称
- 对关键字段识别失败时继续生成结果
