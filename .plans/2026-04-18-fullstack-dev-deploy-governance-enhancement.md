# fullstack-dev-deploy 治理增强实现计划

> 本文件是 executing-plans 的输入契约。
> 执行者：请勿修改任务步骤顺序，如有疑问先与计划编写者确认。

**目标：** 将现有 `fullstack-dev-deploy` skill 从“生成器”增强为“生成 + 资产治理器”，让它不仅能为空白项目生成标准 dev/deploy 资产，还能自动识别、迁移、合并、删除已有的 dev/docker/deploy/env 资产，并收敛到统一目标形态。  
**PRD 来源：** `docs/superpowers/specs/2026-04-18-fullstack-dev-deploy-governance-enhancement.md`  
**创建日期：** 2026-04-18  
**预估任务数：** 7 个  

---

## 文件结构

| 操作 | 路径 | 职责说明 |
|------|------|--------|
| 修改 | `skills/fullstack-dev-deploy/SKILL.md` | 把 skill 顶层定位升级为治理优先模式，并同步新的阶段顺序、双 compose、app 自治 env 与清理保证 |
| 修改 | `skills/fullstack-dev-deploy/references/phase-1-discovery.md` | 为 Discovery 增加 `assetInventory`、`convergencePlan`、现有资产盘点和语义识别规则 |
| 修改 | `skills/fullstack-dev-deploy/references/service-group-detection.md` | 确保服务分组规则与资产治理后的统一入口保持一致，避免旧脚本分组口径残留 |
| 修改 | `skills/fullstack-dev-deploy/references/phase-2-dev.md` | 增加现有 dev 资产收敛、infra/app 联动、旧入口迁移删除与 app 级 env 选择规则 |
| 修改 | `skills/fullstack-dev-deploy/references/phase-3-deploy.md` | 将单 compose 改成 `docker/infra.compose.yml + docker/app.compose.yml` 双 compose，并加入旧 deploy/docker 资产治理规则 |
| 修改 | `skills/fullstack-dev-deploy/references/mirror-and-env-contract.md` | 将中心化 env 契约改成 app 自治 env 模型，并保留镜像源与云厂商默认值规则 |
| 修改 | `skills/fullstack-dev-deploy/references/testing-and-validation.md` | 增加治理结果验证、旧资产清理验证、infra/app 双 compose 验证 |
| 修改 | `skills/fullstack-dev-deploy/references/output-summary.md` | 输出新增资产盘点、keep/migrate/merge/delete/generate 治理动作、最终保留资产与例外说明 |

---

## 任务列表

### Task 1: 重写主 skill 顶层编排

> 让 `SKILL.md` 明确这是一个治理优先的 skill，而不是只会生成新资产的生成器。

**关联文件：**
- 修改：`skills/fullstack-dev-deploy/SKILL.md`

- [ ] 写失败的测试：运行 `rg -n "assetInventory|convergencePlan|infra\\.compose|app\\.compose|\\.env\\.dev\\.example|清理" skills/fullstack-dev-deploy/SKILL.md`，确认治理增强相关关键词尚未完整出现
- [ ] 运行测试，确认红色
- [ ] 实现最小可用代码：重写顶层定位、阶段顺序、治理优先原则、双 compose 目标形态、app 自治 env 与清理保证
- [ ] 运行测试，确认绿色：`rg -n "assetInventory|convergencePlan|infra\\.compose|app\\.compose|\\.env\\.dev\\.example|清理" skills/fullstack-dev-deploy/SKILL.md`
- [ ] git commit -m "feat(fullstack-dev-deploy): add governance-first orchestration"

---

### Task 2: 增强 Discovery 与治理计划模型

**依赖：** Task 1 完成后才能开始

> 让 Discovery 不只识别 app 和分组，还能盘点旧资产并生成 keep/migrate/merge/delete/generate 的治理计划。

**关联文件：**
- 修改：`skills/fullstack-dev-deploy/references/phase-1-discovery.md`
- 修改：`skills/fullstack-dev-deploy/references/service-group-detection.md`

- [ ] 写失败的测试：运行 `rg -n "assetInventory|convergencePlan|keep|migrate|merge|delete|generate|旧脚本|旧 compose|旧 env" skills/fullstack-dev-deploy/references/phase-1-discovery.md skills/fullstack-dev-deploy/references/service-group-detection.md`，确认治理模型尚未完整出现
- [ ] 运行测试，确认红色
- [ ] 实现最小可用代码：补齐资产盘点范围、语义识别原则、治理动作模型、仅在语义无法判断时追问的边界，并确保服务分组与治理结果共用同一份项目画像
- [ ] 运行测试，确认绿色：`rg -n "assetInventory|convergencePlan|keep|migrate|merge|delete|generate|旧脚本|旧 compose|旧 env" skills/fullstack-dev-deploy/references/phase-1-discovery.md skills/fullstack-dev-deploy/references/service-group-detection.md`
- [ ] git commit -m "feat(fullstack-dev-deploy): add asset inventory and convergence rules"

---

### Task 3: 增强 Dev 阶段为治理型入口

**依赖：** Task 2 完成后才能开始

> 让 `phase-2-dev` 不只描述生成 `scripts/dev.sh`，还描述如何收敛旧 dev 资产、联动 infra，并按 app 加载 env。

**关联文件：**
- 修改：`skills/fullstack-dev-deploy/references/phase-2-dev.md`

- [ ] 写失败的测试：运行 `rg -n "旧 dev|run\\.sh|start\\.sh|migrate|delete|infra\\.compose|\\.env\\.dev\\.example|\\.env\\.local\\.example" skills/fullstack-dev-deploy/references/phase-2-dev.md`，确认治理增强要求尚未完整出现
- [ ] 运行测试，确认红色
- [ ] 实现最小可用代码：写清旧 dev 资产迁移/删除规则、infra 依赖联动、按 app 选择 dev/local env、以及统一入口下的启动成功验证
- [ ] 运行测试，确认绿色：`rg -n "旧 dev|run\\.sh|start\\.sh|migrate|delete|infra\\.compose|\\.env\\.dev\\.example|\\.env\\.local\\.example" skills/fullstack-dev-deploy/references/phase-2-dev.md`
- [ ] git commit -m "feat(fullstack-dev-deploy): add dev asset governance"

---

### Task 4: 增强 Deploy 阶段为双 compose 治理模型

**依赖：** Task 2 完成后才能开始

> 让 `phase-3-deploy` 从单 compose 设计升级为 infra/app 双 compose，并收敛旧 deploy/docker 资产。

**关联文件：**
- 修改：`skills/fullstack-dev-deploy/references/phase-3-deploy.md`

- [ ] 写失败的测试：运行 `rg -n "infra\\.compose|app\\.compose|旧 deploy|旧 Dockerfile|旧 compose|migrate|merge|delete|readiness" skills/fullstack-dev-deploy/references/phase-3-deploy.md`，确认双 compose 与治理要求尚未完整出现
- [ ] 运行测试，确认红色
- [ ] 实现最小可用代码：把目标形态改为 `docker/infra.compose.yml + docker/app.compose.yml`，写清旧 compose / Dockerfile / deploy 脚本的 keep/migrate/merge/delete 规则，以及 deploy 后 readiness 验证
- [ ] 运行测试，确认绿色：`rg -n "infra\\.compose|app\\.compose|旧 deploy|旧 Dockerfile|旧 compose|migrate|merge|delete|readiness" skills/fullstack-dev-deploy/references/phase-3-deploy.md`
- [ ] git commit -m "feat(fullstack-dev-deploy): add dual-compose deploy governance"

---

### Task 5: 将 env 契约改成 app 自治模型

**依赖：** Task 3 完成后才能开始

> 让 env 规则不再中心化汇总，而是每个 app 自己维护 `.env.dev/.prod/.local` 模板，真实 `.env*` 不入 git。

**关联文件：**
- 修改：`skills/fullstack-dev-deploy/references/mirror-and-env-contract.md`

- [ ] 写失败的测试：运行 `rg -n "\\.env\\.dev\\.example|\\.env\\.prod\\.example|\\.env\\.local\\.example|不应提交到 git|app 自治|旧 env" skills/fullstack-dev-deploy/references/mirror-and-env-contract.md`，确认 app 自治 env 模型尚未完整出现
- [ ] 运行测试，确认红色
- [ ] 实现最小可用代码：移除中心化 env 口径，改写为 app 自治 env 模型，保留镜像源/云厂商默认值规则，并加入旧 env 文件的迁移/删除判定
- [ ] 运行测试，确认绿色：`rg -n "\\.env\\.dev\\.example|\\.env\\.prod\\.example|\\.env\\.local\\.example|不应提交到 git|app 自治|旧 env" skills/fullstack-dev-deploy/references/mirror-and-env-contract.md`
- [ ] git commit -m "feat(fullstack-dev-deploy): adopt app-owned env model"

---

### Task 6: 增强验证与输出摘要

**依赖：** Task 4、Task 5 完成后才能开始

> 让验证规则和输出摘要覆盖资产治理结果，而不只是覆盖新资产生成结果。

**关联文件：**
- 修改：`skills/fullstack-dev-deploy/references/testing-and-validation.md`
- 修改：`skills/fullstack-dev-deploy/references/output-summary.md`

- [ ] 写失败的测试：运行 `rg -n "assetInventory|keep|migrate|merge|delete|generate|infra\\.compose|app\\.compose|旧资产|最终保留资产" skills/fullstack-dev-deploy/references/testing-and-validation.md skills/fullstack-dev-deploy/references/output-summary.md`，确认治理结果验证与摘要字段尚未完整出现
- [ ] 运行测试，确认红色
- [ ] 实现最小可用代码：补齐治理后验证、旧资产清理验证、infra/app 双 compose 验证，以及资产盘点/治理动作/最终保留资产/例外说明的输出要求
- [ ] 运行测试，确认绿色：`rg -n "assetInventory|keep|migrate|merge|delete|generate|infra\\.compose|app\\.compose|旧资产|最终保留资产" skills/fullstack-dev-deploy/references/testing-and-validation.md skills/fullstack-dev-deploy/references/output-summary.md`
- [ ] git commit -m "feat(fullstack-dev-deploy): add governance validation and summary"

---

### Task 7: 全量术语与 smoke 自检

**依赖：** Task 3、Task 4、Task 5、Task 6 完成后才能开始

> 确保所有文档对治理优先、双 compose、app 自治 env、旧资产清理的表达一致，并确认 skill 可被插件目录加载。

**关联文件：**
- 修改：`skills/fullstack-dev-deploy/SKILL.md`
- 修改：`skills/fullstack-dev-deploy/references/phase-1-discovery.md`
- 修改：`skills/fullstack-dev-deploy/references/service-group-detection.md`
- 修改：`skills/fullstack-dev-deploy/references/phase-2-dev.md`
- 修改：`skills/fullstack-dev-deploy/references/phase-3-deploy.md`
- 修改：`skills/fullstack-dev-deploy/references/mirror-and-env-contract.md`
- 修改：`skills/fullstack-dev-deploy/references/testing-and-validation.md`
- 修改：`skills/fullstack-dev-deploy/references/output-summary.md`

- [ ] 写失败的测试：运行 `rg -n "infra\\.compose|app\\.compose|assetInventory|convergencePlan|\\.env\\.dev\\.example|\\.env\\.prod\\.example|\\.env\\.local\\.example|keep|migrate|merge|delete|generate" skills/fullstack-dev-deploy/SKILL.md skills/fullstack-dev-deploy/references/*.md`，在增强落地前确认关键术语未全面一致
- [ ] 运行测试，确认红色
- [ ] 实现最小可用代码：统一术语、修正交叉引用、消除“旧资产保守兼容”残留表述，并确保治理模式在所有文档中表达一致
- [ ] 运行测试，确认绿色：`rg -n "infra\\.compose|app\\.compose|assetInventory|convergencePlan|\\.env\\.dev\\.example|\\.env\\.prod\\.example|\\.env\\.local\\.example|keep|migrate|merge|delete|generate" skills/fullstack-dev-deploy/SKILL.md skills/fullstack-dev-deploy/references/*.md`
- [ ] 手动 smoke：`claude -p --plugin-dir . '列出当前插件目录中的 skill 名称，只返回名称，每行一个。'`，确认 `fullstack-dev-deploy` 可被发现
- [ ] git commit -m "feat(fullstack-dev-deploy): finalize governance enhancement"

---

## 验收标准

- [ ] `fullstack-dev-deploy` 明确从“生成器”升级为“生成 + 资产治理器”，并且只在语义无法识别时追问
- [ ] Discovery 文案明确包含 `assetInventory` 与 `convergencePlan` 两个新增概念
- [ ] Docker 目标形态明确为 `docker/infra.compose.yml + docker/app.compose.yml + docker/<app>/Dockerfile`
- [ ] env 文案明确为 app 自治模型：每个 app 自己维护 `.env.dev/.prod/.local` 模板，真实 `.env*` 不入 git
- [ ] 输出摘要明确包含 keep / migrate / merge / delete / generate 治理动作以及最终保留资产
- [ ] 至少完成一次插件目录 smoke，确认增强后的 skill 仍可被发现

---

## 技术债（本次不处理）

- 原始设计 spec 与旧计划文件暂不同步到增强版口径
- README、`bin/kssbox` 暴露链路与安装说明仍不纳入这次增强范围
