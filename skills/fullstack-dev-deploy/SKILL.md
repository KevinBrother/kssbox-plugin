---
name: fullstack-dev-deploy
description: 为任意全栈工程生成统一的 dev 和 docker deploy 方案，自动识别项目结构，产出 `scripts/dev.sh`、`scripts/deploy.sh`、`docker/` 部署资产与镜像源环境变量契约。当用户说"给项目加 dev 脚本"、"做 docker deploy"、"统一本地启动和部署"、"生成 dev.sh 和 deploy.sh"、"给全栈项目补 docker 化部署"时，必须使用本 skill。
---

# fullstack-dev-deploy Skill

## 适用场景

这个 skill 面向“任意全栈工程”的工程化改造，目标是统一两类入口：

1. 本地开发入口：`scripts/dev.sh [service...]`
2. 单机 Docker 部署入口：`scripts/deploy.sh [service...]`

默认约定：

- 不传参数时默认 `all`
- 传 `biz`、`boss` 时，表示操作对应服务集合
- 对复杂项目，`biz`、`boss` 是用户层别名，内部可展开为具体 app 清单
- 所有构建在 Docker 内完成，不依赖宿主机已有 Go、Node、Python 等运行时

---

## 执行流程（严格按顺序）

```text
阶段一：Discovery 共享扫描      → 详见 references/phase-1-discovery.md
         服务分组识别           → 详见 references/service-group-detection.md
阶段二：生成 dev 入口           → 详见 references/phase-2-dev.md
阶段三：生成 docker deploy      → 详见 references/phase-3-deploy.md
阶段四：镜像源与环境变量契约     → 详见 references/mirror-and-env-contract.md
阶段五：验证与输出摘要          → 详见 references/testing-and-validation.md
                                  输出格式：references/output-summary.md
```

**在每个阶段开始前，先读对应的 reference 文件，再执行。**
**阶段五不可跳过，生成物不验证、不输出摘要，不算完成。**

---

## 前提检查

| 情况 | 处理方式 |
|------|--------|
| 用户已给出目标项目路径 | 直接进入阶段一 |
| 用户只描述想法，未给出仓库路径 | 先问目标项目根目录在哪里 |
| 目标项目不是全栈仓库 | 告知本 skill 更适合 fullstack 项目，确认是否继续 |
| 目标仓库已有 dev/deploy 资产 | 允许复用，但必须统一到本 skill 的入口契约 |

---

## 阶段一：Discovery 共享扫描

先读取：

- `references/phase-1-discovery.md`
- `references/service-group-detection.md`

核心目标：

- 扫描 frontend / backend app、语言生态、包管理器、构建和启动命令
- 识别已有脚本、Dockerfile、Compose、env 文件
- 把 `all`、`biz`、`boss` 映射为具体 app 集合
- 形成统一“项目画像”，供后续 Dev 和 Deploy 阶段共用

**如果无法可靠识别 app 分组或启动方式，必须停下追问用户。**

---

## 阶段二：生成 dev 入口

读取 `references/phase-2-dev.md`。

核心目标：

- 为目标项目生成或更新 `scripts/dev.sh [service...]`
- 统一 `all` / `biz` / `boss` 的参数行为
- 对复杂项目允许内部展开到 app 级映射
- 在脚本中加载统一环境变量契约
- 生成后必须验证目标服务确实已经启动成功

---

## 阶段三：生成 docker deploy

读取 `references/phase-3-deploy.md`。

核心目标：

- 生成或更新 `scripts/deploy.sh [service...]`
- 在 `docker/` 目录下统一放置部署资产
- 生成 `docker/docker-compose.yml`
- 为各 app 生成 `docker/<app>/Dockerfile`
- 生成 `docker/deploy.env.example`
- 构建与部署必须完全在 Docker / Compose 内完成

---

## 阶段四：镜像源与环境变量契约

读取 `references/mirror-and-env-contract.md`。

核心目标：

- 统一本地开发和 Docker 构建共享的环境变量契约
- 支持国内源、企业内网源、云厂商私有源
- 先询问或识别云厂商，再给出可覆盖的默认值
- 不把任何镜像源硬编码成唯一方案

---

## 阶段五：验证与输出摘要

同时读取：

- `references/testing-and-validation.md`
- `references/output-summary.md`

核心目标：

- 验证 `scripts/dev.sh` 参数解析、服务映射和启动成功
- 验证 `scripts/deploy.sh`、`docker/docker-compose.yml`、`docker/<app>/Dockerfile` 可在本机 Docker 环境工作
- 输出识别结果、假设项、生成文件、验证结果摘要

---

## 输出资产

默认目标是直接改造目标仓库，生成或更新：

- `scripts/dev.sh`
- `scripts/deploy.sh`
- `docker/docker-compose.yml`
- `docker/<app>/Dockerfile`
- `.env.example`
- `docker/deploy.env.example`

必要时可补充：

- Docker 安装脚本
- 源配置辅助脚本
- 部署说明文档

---

## 质量要求

输出前自检：

- [ ] Discovery、Dev、Deploy 三阶段共用同一份项目画像，没有重复扫描导致的规则漂移
- [ ] `scripts/dev.sh` 与 `scripts/deploy.sh` 都支持默认 `all`
- [ ] Docker 相关资产统一收口在 `docker/` 目录
- [ ] 云厂商默认值先确认再写入，且始终允许覆盖
- [ ] 最终输出包含识别结果、假设项、生成文件和验证摘要

---

## 特殊情况

| 情况 | 处理方式 |
|------|--------|
| 仓库结构不是 `frontend/*` + `backend/*` | 继续扫描，只要能稳定识别 app 即可 |
| 已有 `run.sh`、`start.sh` 或旧 deploy 脚本 | 不直接保留旧入口，统一收敛到 `dev.sh` / `deploy.sh` |
| 识别到多个可能的 `biz` / `boss` 分组 | 列出候选映射，向用户确认后再继续 |
| 宿主机未安装 Docker / Compose | 记录缺失并根据 reference 生成安装或引导步骤 |
| 目标项目已有 Kubernetes 配置 | 忽略，不扩展到 K8s，本 skill 只处理单机 Docker / Compose |
