# fullstack-dev-deploy 执行约束强化实现计划

> 本文件是 executing-plans 的输入契约。
> 执行者：请勿修改任务步骤顺序，如有疑问先与计划编写者确认。

**目标：** 收紧 `fullstack-dev-deploy` skill 的案例命名、Discovery 停机规则、资产门禁与验证摘要要求，并同步修正文档中的旧口径。  
**PRD 来源：** `docs/superpowers/specs/2026-04-19-fullstack-dev-deploy-execution-hardening-design.md`  
**创建日期：** 2026-04-19  
**预估任务数：** 6 个  

---

## 文件结构

| 操作 | 路径 | 职责说明 |
|------|------|--------|
| 修改 | `skills/fullstack-dev-deploy/SKILL.md` | 把主 skill 入口从 `biz/boss` 默认语义改为“真实服务名 + all”，并补强阶段门禁说明 |
| 修改 | `skills/fullstack-dev-deploy/references/phase-1-discovery.md` | 把 Discovery 输出改为通用 `projectProfile`，明确真实服务识别与停机条件 |
| 修改 | `skills/fullstack-dev-deploy/references/service-group-detection.md` | 把服务分组从固定业务名模板改为“实际服务名 + 可选聚合组”的识别规则 |
| 修改 | `skills/fullstack-dev-deploy/references/phase-2-dev.md` | 收紧 `dev.sh` 的参数口径、env 选择、治理与成功验证要求 |
| 修改 | `skills/fullstack-dev-deploy/references/phase-3-deploy.md` | 收紧 `deploy.sh` 的参数口径、双 compose 门禁与部署验证要求 |
| 修改 | `skills/fullstack-dev-deploy/references/testing-and-validation.md` | 把验证目标从 `all|biz|boss` 改为 `all + 真实服务名/聚合组`，并补足门禁判断 |
| 修改 | `skills/fullstack-dev-deploy/references/output-summary.md` | 让摘要字段反映真实服务识别、治理动作、假设项和验证结果 |
| 修改 | `docs/superpowers/specs/2026-04-18-fullstack-dev-deploy-design.md` | 修正旧设计文档中对固定业务分组、单 compose、根级 env 的旧表述 |
| 修改 | `docs/superpowers/specs/2026-04-18-fullstack-dev-deploy-governance-enhancement.md` | 与新硬门禁设计保持一致，避免治理增强文档与 skill 本体口径冲突 |

---

## 任务列表

### Task 1: 收紧主 skill 入口契约

> 更新 `SKILL.md`，让模型先学会“案例仅用于示范，运行时必须识别真实服务名”，并把硬门禁直接放在主文档里。

**关联文件：**
- 修改：`skills/fullstack-dev-deploy/SKILL.md`

- [ ] 写失败的检查：记录当前 `SKILL.md` 中仍把 `biz` / `boss` 当默认约定的段落，并用 `rg -n "默认约定|biz|boss" skills/fullstack-dev-deploy/SKILL.md` 确认旧口径仍存在
- [ ] 运行：`rg -n "默认约定|biz|boss" skills/fullstack-dev-deploy/SKILL.md`，确认能看到旧表述
- [ ] 实现最小修改：把主文档改为“主案例 + 扩展示例”“默认支持 all + 实际服务名”“聚合组按识别或确认产生”“未形成双 compose / app 自治 env / 摘要则未完成”
- [ ] 运行：`rg -n "实际服务名|双 compose|app 自治 env|未完成" skills/fullstack-dev-deploy/SKILL.md`，确认新门禁文案已落入主文档
- [ ] git commit -m "docs(fullstack-dev-deploy): harden skill contract"

---

### Task 2: 重写 Discovery 与服务分组 reference

> 让 Discovery 真正产出唯一项目画像，并把服务识别规则从固定业务名模板改成真实服务名优先。

**关联文件：**
- 修改：`skills/fullstack-dev-deploy/references/phase-1-discovery.md`
- 修改：`skills/fullstack-dev-deploy/references/service-group-detection.md`

**依赖：** Task 1 完成后才能开始

- [ ] 写失败的检查：用 `rg -n "serviceGroups\\{all,biz,boss\\}|frontend-biz|frontend-boss|只使用 `all`、`biz`、`boss`" skills/fullstack-dev-deploy/references/{phase-1-discovery,service-group-detection}.md` 标记旧模板化语句
- [ ] 运行上述 `rg`，确认仍存在固定 `biz/boss` 模板
- [ ] 实现最小修改：把 Discovery 输出改成通用 `projectProfile`；把服务分组规则改成“至少 all + 实际服务名，可选聚合组”；把必须停下追问的条件写清楚
- [ ] 运行：`rg -n "projectProfile|实际服务名|聚合组|必须停下追问" skills/fullstack-dev-deploy/references/{phase-1-discovery,service-group-detection}.md`，确认新规则已写入
- [ ] git commit -m "docs(fullstack-dev-deploy): generalize discovery grouping"

---

### Task 3: 收紧 dev / deploy reference 的资产门禁

> 把 dev 和 deploy 阶段都绑定到同一套真实服务映射与最终资产形态，不再允许“能跑但未收敛”的结果过关。

**关联文件：**
- 修改：`skills/fullstack-dev-deploy/references/phase-2-dev.md`
- 修改：`skills/fullstack-dev-deploy/references/phase-3-deploy.md`

**依赖：** Task 2 完成后才能开始

- [ ] 写失败的检查：用 `rg -n "./scripts/(dev|deploy)\\.sh biz|./scripts/(dev|deploy)\\.sh boss|all`、`biz`、`boss`|把 `biz`、`boss` 直接写死" skills/fullstack-dev-deploy/references/phase-{2-dev,3-deploy}.md` 标出旧参数口径
- [ ] 运行上述 `rg`，确认 dev / deploy reference 仍以固定业务分组为主要示例
- [ ] 实现最小修改：把参数规则改为“all + 实际服务名 + 可选聚合组”；补充旧资产只能作为治理输入、双 compose 与 app env 是强门禁、dev / deploy 必须共用同一份服务映射
- [ ] 运行：`rg -n "实际服务名|可选聚合组|双 compose|同一份服务映射|app 自治 env" skills/fullstack-dev-deploy/references/phase-{2-dev,3-deploy}.md`，确认新门禁已覆盖两阶段
- [ ] git commit -m "docs(fullstack-dev-deploy): tighten dev deploy gates"

---

### Task 4: 收紧验证与摘要 reference

> 让“验证通过”和“结构化摘要齐全”成为完成条件，而不是附带说明。

**关联文件：**
- 修改：`skills/fullstack-dev-deploy/references/testing-and-validation.md`
- 修改：`skills/fullstack-dev-deploy/references/output-summary.md`

**依赖：** Task 3 完成后才能开始

- [ ] 写失败的检查：用 `rg -n "all\\|biz\\|boss|`biz`|`boss`" skills/fullstack-dev-deploy/references/{testing-and-validation,output-summary}.md` 确认验证与摘要仍绑定旧分组名
- [ ] 运行上述 `rg`，确认旧服务分组口径仍存在
- [ ] 实现最小修改：把验证目标改成“all + 真实服务名 / 聚合组”；把摘要字段改成“识别到的真实 app 与服务集合、治理动作、假设项、验证结果”；补上缺少摘要即未完成的语义
- [ ] 运行：`rg -n "真实服务名|聚合组|治理动作|假设项|验证结果" skills/fullstack-dev-deploy/references/{testing-and-validation,output-summary}.md`，确认输出口径已更新
- [ ] git commit -m "docs(fullstack-dev-deploy): harden validation summary"

---

### Task 5: 同步修正旧设计文档

> 把两份 4 月 18 日设计文档改到与当前硬门禁方案一致，避免后续再从旧 spec 抄出错误口径。

**关联文件：**
- 修改：`docs/superpowers/specs/2026-04-18-fullstack-dev-deploy-design.md`
- 修改：`docs/superpowers/specs/2026-04-18-fullstack-dev-deploy-governance-enhancement.md`

**依赖：** Task 4 完成后才能开始

- [ ] 写失败的检查：用 `rg -n "biz|boss|docker/docker-compose\\.yml|\\.env\\.example" docs/superpowers/specs/2026-04-18-fullstack-dev-deploy-*.md` 标出仍然过时的表述
- [ ] 运行上述 `rg`，确认旧 spec 仍写着固定业务分组、单 compose 或根级 env 的旧设计
- [ ] 实现最小修改：把旧 spec 统一改成“案例仅示范、运行时识别真实服务名、最终收敛双 compose 与 app 自治 env、验证与摘要是门禁”
- [ ] 运行：`rg -n "真实服务名|双 compose|app 自治 env|验证摘要|未完成" docs/superpowers/specs/2026-04-18-fullstack-dev-deploy-*.md`，确认旧 spec 已与新设计对齐
- [ ] git commit -m "docs(specs): align fullstack deploy docs"

---

### Task 6: 端到端自检与回归核对

> 对照新 spec 和 high-chat 暴露出来的问题，确认 skill 文案已经能拦住同类偏差。

**关联文件：**
- 修改：`skills/fullstack-dev-deploy/SKILL.md`
- 修改：`skills/fullstack-dev-deploy/references/phase-1-discovery.md`
- 修改：`skills/fullstack-dev-deploy/references/service-group-detection.md`
- 修改：`skills/fullstack-dev-deploy/references/phase-2-dev.md`
- 修改：`skills/fullstack-dev-deploy/references/phase-3-deploy.md`
- 修改：`skills/fullstack-dev-deploy/references/testing-and-validation.md`
- 修改：`skills/fullstack-dev-deploy/references/output-summary.md`
- 修改：`docs/superpowers/specs/2026-04-18-fullstack-dev-deploy-design.md`
- 修改：`docs/superpowers/specs/2026-04-18-fullstack-dev-deploy-governance-enhancement.md`

**依赖：** Task 5 完成后才能开始

- [ ] 写失败的检查：列出 `high-chat` 暴露的 4 个失败信号——固定案例名误导、未形成双 compose、未形成 app env、缺少治理摘要，并为每项准备对应 `rg` 检查语句
- [ ] 运行检查：`rg -n "实际服务名|双 compose|app 自治 env|治理动作|验证摘要|必须停下追问" skills/fullstack-dev-deploy/SKILL.md skills/fullstack-dev-deploy/references/*.md docs/superpowers/specs/2026-04-18-fullstack-dev-deploy-*.md`
- [ ] 实现最小修补：若自检时仍发现遗漏口径，立即补齐相关 Markdown，直到 4 个失败信号都有明确拦截文案
- [ ] 运行最终核对：`git diff --stat && rg -n "实际服务名|双 compose|app 自治 env|治理动作|验证摘要|必须停下追问" skills/fullstack-dev-deploy/SKILL.md skills/fullstack-dev-deploy/references/*.md docs/superpowers/specs/2026-04-18-fullstack-dev-deploy-*.md`
- [ ] git commit -m "docs(fullstack-dev-deploy): finalize hardening pass"

---

## 验收标准

- [ ] `skills/fullstack-dev-deploy/SKILL.md` 不再把 `biz` / `boss` 当默认服务语义，而是明确“all + 实际服务名 + 可选聚合组”
- [ ] Discovery、dev、deploy、validation、summary 五类文档都写清楚“无法确定语义时必须停下追问”
- [ ] skill 文档明确把双 compose、app 自治 env、治理动作摘要、验证摘要设为完成门禁
- [ ] 两份 2026-04-18 的旧设计文档已同步到新口径，没有再和 skill 本体打架

---

## 技术债（本次不处理）

- 本次只修正规则文档与设计文档，不直接对 `high-chat` 仓库重新执行一次 fullstack-dev-deploy 改造
- 本次不处理此前误提交中夹带的其他文档提交历史，只保证后续实现按新计划推进
