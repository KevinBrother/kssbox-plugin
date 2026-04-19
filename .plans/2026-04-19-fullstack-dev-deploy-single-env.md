# fullstack-dev-deploy 单一 env 契约重构实现计划

> 本文件是 executing-plans 的输入契约。
> 执行者：请勿修改任务步骤顺序，如有疑问先与计划编写者确认。

**目标：** 将 `fullstack-dev-deploy` skill 从“app 自治多 env”重构为 `docker/.env.example -> docker/.env` 的单一 env 契约，并同步修正文档中的命名规范、治理规则、脚本行为要求与验证标准。  
**PRD 来源：** `docs/superpowers/specs/2026-04-19-fullstack-dev-deploy-single-env-design.md`  
**创建日期：** 2026-04-19  
**预估任务数：** 6 个  

---

## 文件结构

| 操作 | 路径 | 职责说明 |
|------|------|--------|
| 修改 | `skills/fullstack-dev-deploy/SKILL.md` | 更新 skill 总契约、输出资产、质量要求，改为单一 `docker/.env` 模型 |
| 修改 | `skills/fullstack-dev-deploy/references/phase-1-discovery.md` | 将 Discovery 的 env 盘点与治理目标改为统一收敛到 `docker/.env.example` |
| 修改 | `skills/fullstack-dev-deploy/references/service-group-detection.md` | 确保服务分组文档不再暗含按 app 拆 env 的前提 |
| 修改 | `skills/fullstack-dev-deploy/references/phase-2-dev.md` | 把 dev 阶段改为只检查 `docker/.env`，并明确缺失提示、复制命令与 cleanup 安全要求 |
| 修改 | `skills/fullstack-dev-deploy/references/phase-3-deploy.md` | 把 deploy 阶段改为统一读取 `docker/.env`，删除 `docker/deploy.env*` 依赖 |
| 修改 | `skills/fullstack-dev-deploy/references/mirror-and-env-contract.md` | 重写 env 契约、命名规范、旧 env 治理规则与 `coze-studio` 风格模型 |
| 修改 | `skills/fullstack-dev-deploy/references/testing-and-validation.md` | 把验证标准改为单一 env 模型，覆盖缺失 env 与 `PIDS[@]` 二次错误场景 |
| 修改 | `skills/fullstack-dev-deploy/references/output-summary.md` | 输出摘要中新增 env 收敛与复制指引信息 |

---

## 任务列表

### Task 1: 重写 skill 总契约

> 让 `SKILL.md` 的默认目标形态从“app 自治 env”切换为 `docker/.env.example -> docker/.env`，作为后续 references 的唯一上位约束。

**关联文件：**
- 修改：`skills/fullstack-dev-deploy/SKILL.md`

- [ ] 写失败的检查：用 `rg "app 自治 env|\\.env\\.dev\\.example|\\.env\\.prod\\.example|\\.env\\.local\\.example" skills/fullstack-dev-deploy/SKILL.md` 记录旧契约仍存在
- [ ] 运行检查，确认红色（仍能搜到旧 env 契约）
- [ ] 实现最小改动让检查通过：把目标形态、输出资产、阶段目标、质量要求改成单一 `docker/.env` 模型，并补充“env key 全大写蛇形命名”
- [ ] 运行：`rg "docker/\\.env\\.example|docker/\\.env|全大写|蛇形" skills/fullstack-dev-deploy/SKILL.md`
- [ ] git commit -m "docs(fullstack-dev-deploy): adopt single env contract"

---

### Task 2: 收敛 Discovery 与服务分组前提

> 让 Discovery/服务分组阶段把旧 env 识别和治理统一指向 `docker/.env.example`，并移除所有按 app 拆 env 的默认前提。

**依赖：** Task 1 完成后才能开始

**关联文件：**
- 修改：`skills/fullstack-dev-deploy/references/phase-1-discovery.md`
- 修改：`skills/fullstack-dev-deploy/references/service-group-detection.md`

- [ ] 写失败的检查：用 `rg "\\.env\\.dev|\\.env\\.prod|\\.env\\.local|app 内 \\.env|app 自治" skills/fullstack-dev-deploy/references/{phase-1-discovery.md,service-group-detection.md}` 记录旧模型残留
- [ ] 运行检查，确认红色
- [ ] 实现最小改动让检查通过：把 env 盘点、收敛目标、冲突处理与输出字段统一改成 `docker/.env.example` / `docker/.env`
- [ ] 运行：`rg "docker/\\.env\\.example|docker/\\.env|merge|migrate|delete" skills/fullstack-dev-deploy/references/{phase-1-discovery.md,service-group-detection.md}`
- [ ] git commit -m "docs(fullstack-dev-deploy): align discovery with single env model"

---

### Task 3: 重写 dev 阶段规则

> 把 dev 阶段改成只检查 `docker/.env`，并明确缺失提示、复制命令、关键变量缺失提示以及 cleanup 安全要求。

**依赖：** Task 2 完成后才能开始

**关联文件：**
- 修改：`skills/fullstack-dev-deploy/references/phase-2-dev.md`

- [ ] 写失败的检查：用 `rg "\\.env\\.dev|\\.env\\.local|overlay|service overlay|app 读取运行时环境" skills/fullstack-dev-deploy/references/phase-2-dev.md` 记录旧 dev env 规则仍存在
- [ ] 运行检查，确认红色
- [ ] 实现最小改动让检查通过：将 dev 阶段改为只依赖 `docker/.env`，补充 `Missing env file: docker/.env` / `Expected template: docker/.env.example` / `Run: cp ...` 的报错格式，并把 `PIDS[@]` 提前退出 bug 写入明确约束
- [ ] 运行：`rg "docker/\\.env|Expected template: docker/\\.env\\.example|PIDS\\[@\\]|全大写" skills/fullstack-dev-deploy/references/phase-2-dev.md`
- [ ] git commit -m "docs(fullstack-dev-deploy): rewrite dev phase for single env"

---

### Task 4: 重写 deploy 与 env 契约

> 把 deploy 阶段和 env 契约统一到 `docker/.env`，并废除 `docker/deploy.env*` 与多环境 env 文档模型。

**依赖：** Task 3 完成后才能开始

**关联文件：**
- 修改：`skills/fullstack-dev-deploy/references/phase-3-deploy.md`
- 修改：`skills/fullstack-dev-deploy/references/mirror-and-env-contract.md`

- [ ] 写失败的检查：用 `rg "docker/deploy\\.env|\\.env\\.prod|\\.env\\.local|app 自治 env|每个 app" skills/fullstack-dev-deploy/references/{phase-3-deploy.md,mirror-and-env-contract.md}` 记录旧 deploy/env 契约残留
- [ ] 运行检查，确认红色
- [ ] 实现最小改动让检查通过：将 deploy、compose、Docker build 的 env 来源统一改为 `docker/.env`，补充全大写蛇形命名规范、注释分组建议，以及旧 env 文件迁移/合并/删除规则
- [ ] 运行：`rg "docker/\\.env\\.example|docker/\\.env|全大写|蛇形|NPM_REGISTRY|DATABASE_URL" skills/fullstack-dev-deploy/references/{phase-3-deploy.md,mirror-and-env-contract.md}`
- [ ] git commit -m "docs(fullstack-dev-deploy): rewrite deploy and env contract"

---

### Task 5: 重写验证与摘要

> 让验证和输出摘要围绕单一 env 模型展开，覆盖缺失 env 路径、复制指令、关键变量缺失与 cleanup 二次错误场景。

**依赖：** Task 4 完成后才能开始

**关联文件：**
- 修改：`skills/fullstack-dev-deploy/references/testing-and-validation.md`
- 修改：`skills/fullstack-dev-deploy/references/output-summary.md`

- [ ] 写失败的检查：用 `rg "\\.env\\.dev|\\.env\\.prod|\\.env\\.local|app 自治|最终保留.*\\.env\\.dev" skills/fullstack-dev-deploy/references/{testing-and-validation.md,output-summary.md}` 记录旧验收模型残留
- [ ] 运行检查，确认红色
- [ ] 实现最小改动让检查通过：把验证与摘要改成只认 `docker/.env.example` / `docker/.env`，并要求输出旧 env 文件治理动作、缺失路径、复制命令与 `PIDS[@]` 不再触发
- [ ] 运行：`rg "docker/\\.env\\.example|docker/\\.env|Missing env file|cp docker/\\.env\\.example docker/\\.env|PIDS\\[@\\]" skills/fullstack-dev-deploy/references/{testing-and-validation.md,output-summary.md}`
- [ ] git commit -m "docs(fullstack-dev-deploy): update validation and summary for single env"

---

### Task 6: 一致性收口与 smoke 验证

> 清除 skill 包内残留的旧 env 术语，确认 `fullstack-dev-deploy` 仍能被插件识别，并保证文档之间口径一致。

**依赖：** Task 5 完成后才能开始

**关联文件：**
- 修改：`skills/fullstack-dev-deploy/SKILL.md`
- 修改：`skills/fullstack-dev-deploy/references/phase-1-discovery.md`
- 修改：`skills/fullstack-dev-deploy/references/service-group-detection.md`
- 修改：`skills/fullstack-dev-deploy/references/phase-2-dev.md`
- 修改：`skills/fullstack-dev-deploy/references/phase-3-deploy.md`
- 修改：`skills/fullstack-dev-deploy/references/mirror-and-env-contract.md`
- 修改：`skills/fullstack-dev-deploy/references/testing-and-validation.md`
- 修改：`skills/fullstack-dev-deploy/references/output-summary.md`

- [ ] 写失败的检查：运行 `rg "\\.env\\.dev|\\.env\\.prod|\\.env\\.local|docker/deploy\\.env|app 自治 env|每个 app 至少应具备" skills/fullstack-dev-deploy --glob '**/*.md'`
- [ ] 运行检查，确认红色
- [ ] 实现最小改动让检查通过：统一全包术语，保留只与“旧资产待治理”相关的负例表述，消除正向旧模型描述
- [ ] 运行：`claude -p --plugin-dir . '列出当前插件目录中的 skill 名称，只返回名称，每行一个。'`
- [ ] git commit -m "docs(fullstack-dev-deploy): finalize single env refactor"

---

## 验收标准

- [ ] `skills/fullstack-dev-deploy/` 内的默认 env 契约已统一为 `docker/.env.example -> docker/.env`
- [ ] skill 文档明确要求 env key 全部使用全大写蛇形命名
- [ ] 文档明确要求 `dev.sh` 缺 env 时提示 `docker/.env` 路径和复制命令，并禁止 `PIDS[@]: unbound variable` 二次错误
- [ ] `fullstack-dev-deploy` 仍能通过 `claude -p --plugin-dir .` 被识别

---

## 技术债（本次不处理）

- `high-chat` 现有生成物的实际脚本修复不在本计划范围内；待 skill 重构完成后，可另起计划对目标仓库重新执行并验证生成结果
- 本次不处理已提交的 `.plans/2026-04-19-fullstack-dev-deploy-hardening.md` 与其他暂存 skill 修改的提交拆分问题
