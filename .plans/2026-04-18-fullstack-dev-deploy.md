# fullstack-dev-deploy 实现计划

> 本文件是 executing-plans 的输入契约。
> 执行者：请勿修改任务步骤顺序，如有疑问先与计划编写者确认。

**目标：** 交付一个新的 `fullstack-dev-deploy` skill，在任意全栈工程中按统一阶段生成 `scripts/dev.sh`、`scripts/deploy.sh`、`docker/` 部署资产、镜像源环境变量契约与验证规则。  
**PRD 来源：** `docs/superpowers/specs/2026-04-18-fullstack-dev-deploy-design.md`  
**创建日期：** 2026-04-18  
**预估任务数：** 7 个  

---

## 文件结构

| 操作 | 路径 | 职责说明 |
|------|------|--------|
| 新建 | `skills/fullstack-dev-deploy/SKILL.md` | 定义触发条件、阶段顺序、共享 Discovery、Dev/Deploy 路由、落盘与自检要求 |
| 新建 | `skills/fullstack-dev-deploy/references/phase-1-discovery.md` | 规定如何扫描任意全栈项目、提取 app 拓扑、命令、端口、已有资产 |
| 新建 | `skills/fullstack-dev-deploy/references/service-group-detection.md` | 规定如何把 `all`、`biz`、`boss` 映射为具体 app 集合，以及何时停下向用户追问 |
| 新建 | `skills/fullstack-dev-deploy/references/phase-2-dev.md` | 规定如何生成 `scripts/dev.sh [service...]`、默认 `all`、日志与启动成功验证 |
| 新建 | `skills/fullstack-dev-deploy/references/phase-3-deploy.md` | 规定如何生成 `scripts/deploy.sh [service...]`、`docker/docker-compose.yml`、`docker/<app>/Dockerfile` 与单机 Compose 部署流程 |
| 新建 | `skills/fullstack-dev-deploy/references/mirror-and-env-contract.md` | 规定镜像源环境变量契约、国内源覆盖策略、云厂商默认值确认逻辑 |
| 新建 | `skills/fullstack-dev-deploy/references/testing-and-validation.md` | 规定 `dev.sh` / `deploy.sh` 的验证层次、脚本检查、本机 Docker 验证与失败处理 |
| 新建 | `skills/fullstack-dev-deploy/references/output-summary.md` | 规定最终输出必须包含的识别结果、假设项、生成文件与验证摘要 |

---

## 任务列表

### Task 0: 初始化 skill 目录

> 建立 `fullstack-dev-deploy` 的目录骨架与 reference 文件集合，为后续内容填充提供稳定路径。

**关联文件：**
- 新建：`skills/fullstack-dev-deploy/SKILL.md`
- 新建：`skills/fullstack-dev-deploy/references/phase-1-discovery.md`
- 新建：`skills/fullstack-dev-deploy/references/service-group-detection.md`
- 新建：`skills/fullstack-dev-deploy/references/phase-2-dev.md`
- 新建：`skills/fullstack-dev-deploy/references/phase-3-deploy.md`
- 新建：`skills/fullstack-dev-deploy/references/mirror-and-env-contract.md`
- 新建：`skills/fullstack-dev-deploy/references/testing-and-validation.md`
- 新建：`skills/fullstack-dev-deploy/references/output-summary.md`

- [ ] 创建目录与占位文件：`mkdir -p skills/fullstack-dev-deploy/references`
- [ ] 建立初始文件集合，确保后续引用路径全部存在
- [ ] 运行：`find skills/fullstack-dev-deploy -maxdepth 2 -type f | sort`，确认 8 个目标文件都已创建
- [ ] git commit -m "chore(fullstack-dev-deploy): scaffold skill structure"

---

### Task 1: 编写主 skill 编排文档

> 让 `SKILL.md` 明确触发条件、阶段顺序、共享 Discovery、Dev/Deploy 路由和必须读取的 references。

**关联文件：**
- 修改：`skills/fullstack-dev-deploy/SKILL.md`

- [ ] 写失败的检查：运行 `rg -n "phase-1-discovery|phase-2-dev|phase-3-deploy|mirror-and-env-contract|testing-and-validation|output-summary" skills/fullstack-dev-deploy/SKILL.md`，确认关键阶段引用尚未完整出现
- [ ] 运行检查，确认红色
- [ ] 实现最小内容让检查通过：补齐 YAML frontmatter、触发描述、严格阶段顺序、共享 Discovery、`scripts/dev.sh` / `scripts/deploy.sh` 统一入口、默认 `all`、落盘与自检要求
- [ ] 运行：`rg -n "phase-1-discovery|phase-2-dev|phase-3-deploy|mirror-and-env-contract|testing-and-validation|output-summary" skills/fullstack-dev-deploy/SKILL.md`，确认绿色
- [ ] git commit -m "feat(fullstack-dev-deploy): add skill orchestration"

---

### Task 2: 编写 Discovery 与服务分组规则

**依赖：** Task 1 完成后才能开始

> 让 skill 能先识别任意项目结构，再安全地把 `all`、`biz`、`boss` 映射成具体 app 集合。

**关联文件：**
- 修改：`skills/fullstack-dev-deploy/references/phase-1-discovery.md`
- 修改：`skills/fullstack-dev-deploy/references/service-group-detection.md`

- [ ] 写失败的检查：运行 `rg -n "apps\\[\\]|serviceGroups|existingArtifacts|assumptions|biz|boss|all" skills/fullstack-dev-deploy/references/phase-1-discovery.md skills/fullstack-dev-deploy/references/service-group-detection.md`，确认中间抽象模型和分组规则尚未完整出现
- [ ] 运行检查，确认红色
- [ ] 实现最小内容让检查通过：写清扫描对象、项目画像字段、已有资产识别、可自动识别/可推断/无法安全识别的分流，以及无法可靠分组时的追问规则
- [ ] 运行：`rg -n "apps\\[\\]|serviceGroups|existingArtifacts|assumptions|biz|boss|all" skills/fullstack-dev-deploy/references/phase-1-discovery.md skills/fullstack-dev-deploy/references/service-group-detection.md`，确认绿色
- [ ] git commit -m "feat(fullstack-dev-deploy): add discovery and grouping rules"

---

### Task 3: 编写 Dev 阶段规则

**依赖：** Task 2 完成后才能开始

> 让 skill 能为任意项目生成统一的 `scripts/dev.sh [service...]`，并验证目标服务已实际拉起。

**关联文件：**
- 修改：`skills/fullstack-dev-deploy/references/phase-2-dev.md`

- [ ] 写失败的检查：运行 `rg -n "scripts/dev\\.sh|default.*all|biz|boss|启动成功|ready|health" skills/fullstack-dev-deploy/references/phase-2-dev.md`，确认统一入口、默认参数与启动验证要求尚未完整出现
- [ ] 运行检查，确认红色
- [ ] 实现最小内容让检查通过：写清参数解析、服务集合展开、复杂项目的 app 级映射、环境变量加载、日志输出，以及通过端口/healthcheck/ready 信号验证服务已启动
- [ ] 运行：`rg -n "scripts/dev\\.sh|all|biz|boss|启动成功|ready|health" skills/fullstack-dev-deploy/references/phase-2-dev.md`，确认绿色
- [ ] git commit -m "feat(fullstack-dev-deploy): add dev phase guidance"

---

### Task 4: 编写 Deploy 阶段规则

**依赖：** Task 2 完成后才能开始

> 让 skill 能生成 `scripts/deploy.sh [service...]`、`docker/` 目录下的 Compose 与 Dockerfile，并约束所有构建在容器内完成。

**关联文件：**
- 修改：`skills/fullstack-dev-deploy/references/phase-3-deploy.md`

- [ ] 写失败的检查：运行 `rg -n "scripts/deploy\\.sh|docker/docker-compose\\.yml|docker/.*/Dockerfile|docker compose build|docker compose up -d|宿主机" skills/fullstack-dev-deploy/references/phase-3-deploy.md`，确认部署入口和 Docker 目录约定尚未完整出现
- [ ] 运行检查，确认红色
- [ ] 实现最小内容让检查通过：写清默认 `all`、服务集合展开、Docker/Compose 可用性校验、`docker/` 目录结构、单机 Compose 流程、多阶段构建与“宿主机不依赖语言运行时”的约束
- [ ] 运行：`rg -n "scripts/deploy\\.sh|docker/docker-compose\\.yml|docker/.*/Dockerfile|docker compose build|docker compose up -d|不依赖宿主机" skills/fullstack-dev-deploy/references/phase-3-deploy.md`，确认绿色
- [ ] git commit -m "feat(fullstack-dev-deploy): add deploy phase guidance"

---

### Task 5: 编写镜像源与云厂商默认值契约

**依赖：** Task 3 完成后才能开始

> 让 skill 统一读取可覆盖的镜像源环境变量，并在执行时询问云厂商后给出默认值。

**关联文件：**
- 修改：`skills/fullstack-dev-deploy/references/mirror-and-env-contract.md`

- [ ] 写失败的检查：运行 `rg -n "NPM_REGISTRY|PIP_INDEX_URL|GO_PROXY|APT_MIRROR|DOCKER_MIRROR|云厂商|默认值|可覆盖" skills/fullstack-dev-deploy/references/mirror-and-env-contract.md`，确认环境变量契约与云厂商默认值逻辑尚未完整出现
- [ ] 运行检查，确认红色
- [ ] 实现最小内容让检查通过：写清通用环境变量、Docker 与本地共享的读取规则、国内源覆盖原则、先询问云厂商再给默认值、可内置或联网查询默认值但不得硬编码成唯一方案
- [ ] 运行：`rg -n "NPM_REGISTRY|PIP_INDEX_URL|GO_PROXY|APT_MIRROR|DOCKER_MIRROR|云厂商|默认值|可覆盖" skills/fullstack-dev-deploy/references/mirror-and-env-contract.md`，确认绿色
- [ ] git commit -m "feat(fullstack-dev-deploy): add mirror and env contract"

---

### Task 6: 编写验证规则与输出摘要

**依赖：** Task 3 完成后才能开始

> 让 skill 明确 `dev.sh` / `deploy.sh` 的验证层次、失败分流和最终输出摘要格式。

**关联文件：**
- 修改：`skills/fullstack-dev-deploy/references/testing-and-validation.md`
- 修改：`skills/fullstack-dev-deploy/references/output-summary.md`

- [ ] 写失败的检查：运行 `rg -n "静态校验|Dev 行为验证|Deploy 本机验证|假设项|新建了哪些文件|修改了哪些文件|哪些验证已执行" skills/fullstack-dev-deploy/references/testing-and-validation.md skills/fullstack-dev-deploy/references/output-summary.md`，确认验证层次与摘要字段尚未完整出现
- [ ] 运行检查，确认红色
- [ ] 实现最小内容让检查通过：写清脚本静态校验、`dev.sh` 启动验证、`deploy.sh` 本机 Docker 验证、失败时的停止规则，以及最终必须输出的识别结果、映射关系、假设项、生成文件与验证摘要
- [ ] 运行：`rg -n "静态校验|Dev 行为验证|Deploy 本机验证|假设项|新建了哪些文件|修改了哪些文件|哪些验证已执行" skills/fullstack-dev-deploy/references/testing-and-validation.md skills/fullstack-dev-deploy/references/output-summary.md`，确认绿色
- [ ] git commit -m "feat(fullstack-dev-deploy): add validation and summary rules"

---

### Task 7: 全量串联自检

**依赖：** Task 4、Task 5、Task 6 完成后才能开始

> 确认主文档与 references 互相不打架，且 skill 文案足以指导执行者稳定产出。

**关联文件：**
- 修改：`skills/fullstack-dev-deploy/SKILL.md`
- 修改：`skills/fullstack-dev-deploy/references/phase-1-discovery.md`
- 修改：`skills/fullstack-dev-deploy/references/service-group-detection.md`
- 修改：`skills/fullstack-dev-deploy/references/phase-2-dev.md`
- 修改：`skills/fullstack-dev-deploy/references/phase-3-deploy.md`
- 修改：`skills/fullstack-dev-deploy/references/mirror-and-env-contract.md`
- 修改：`skills/fullstack-dev-deploy/references/testing-and-validation.md`
- 修改：`skills/fullstack-dev-deploy/references/output-summary.md`

- [ ] 写失败的检查：运行 `rg -n "phase-1-discovery|phase-2-dev|phase-3-deploy|mirror-and-env-contract|testing-and-validation|output-summary" skills/fullstack-dev-deploy/SKILL.md && test -f skills/fullstack-dev-deploy/references/phase-1-discovery.md && test -f skills/fullstack-dev-deploy/references/service-group-detection.md && test -f skills/fullstack-dev-deploy/references/phase-2-dev.md && test -f skills/fullstack-dev-deploy/references/phase-3-deploy.md && test -f skills/fullstack-dev-deploy/references/mirror-and-env-contract.md && test -f skills/fullstack-dev-deploy/references/testing-and-validation.md && test -f skills/fullstack-dev-deploy/references/output-summary.md`，在实现前确认引用或内容不完整
- [ ] 运行检查，确认红色
- [ ] 实现最小修正让检查通过：统一术语、修正交叉引用、补齐遗漏约束，并确保 `dev.sh` / `deploy.sh`、`docker/` 目录、云厂商默认值、验证摘要在各文件中表达一致
- [ ] 运行：`rg -n "phase-1-discovery|phase-2-dev|phase-3-deploy|mirror-and-env-contract|testing-and-validation|output-summary" skills/fullstack-dev-deploy/SKILL.md && test -f skills/fullstack-dev-deploy/references/phase-1-discovery.md && test -f skills/fullstack-dev-deploy/references/service-group-detection.md && test -f skills/fullstack-dev-deploy/references/phase-2-dev.md && test -f skills/fullstack-dev-deploy/references/phase-3-deploy.md && test -f skills/fullstack-dev-deploy/references/mirror-and-env-contract.md && test -f skills/fullstack-dev-deploy/references/testing-and-validation.md && test -f skills/fullstack-dev-deploy/references/output-summary.md`，确认绿色
- [ ] 手动 smoke：`claude --plugin-dir ./kssbox-plugin`，确认新 skill 可被发现并能作为独立 skill 读取（若本地安装了 claude CLI）
- [ ] git commit -m "feat(fullstack-dev-deploy): finalize skill package"

---

## 验收标准

- [ ] `skills/fullstack-dev-deploy/` 下的 `SKILL.md` 与 7 个 reference 文件齐全，路径与引用一致
- [ ] skill 文案明确包含 Discovery、Dev、Deploy、镜像源契约、验证规则、输出摘要五类核心约束
- [ ] `scripts/dev.sh [service...]`、`scripts/deploy.sh [service...]`、默认 `all`、`docker/` 目录约束、云厂商默认值确认逻辑在文档中表达一致
- [ ] 至少完成一次仓库内自检，确认 references 文件存在且主 skill 的交叉引用可解析

---

## 技术债（本次不处理）

- README、`bin/kssbox` 暴露链路与安装说明暂不纳入首版计划
- 若后续需要更强的 smoke 测试，可再补真实触发样例和自动化校验脚本
