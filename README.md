
# kssbox-plugin

从 idea 到代码落地的全链路 AI 开发工具集。

## Skills 流程链路

```
         idea（一句话想法）
           │
           ▼
   ┌───────────────┐
   │product-strategy│  阶段一：产品策略规划
   │               │  竞品调研 · 蓝红海分析 · 推广策略 · 收费模式
   └───────┬───────┘
           │  输出：产品规划文档
           ▼
   ┌───────────────┐
   │  prd-writer   │  阶段二：PRD 撰写
   │               │  用户故事 · 功能规格 · 数据模型 · API 设计 · 验收标准
   └───────┬───────┘
           │  输出：完整 PRD 文档
           ▼
   ┌───────────────┐
   │ writing-plans │  阶段三：任务拆解
   │               │  将 PRD 拆为可执行的开发任务，输出 .plans/ 计划文件
   └───────┬───────┘
           │  输出：.plans/YYYY-MM-DD-<feature>.md
           ▼
   ┌───────────────┐
   │ harness-sync  │  阶段四：Harness 初始化/同步
   │               │  生成 AGENTS.md · 架构文档 · CI 配置 · 依赖规则
   └───────┬───────┘
           │  输出：项目 Harness 文件集
           ▼
   ┌───────────────┐
   │executing-plans│  阶段五：执行计划
   │               │  逐任务读取计划文件，编写代码，运行测试，交付可运行项目
   └───────────────┘
```

每个 skill 独立解耦，可单独触发使用，也可按上述链路串行推进。

## 安装 & 使用

```bash
# 全局链接 CLI
npm link

# 管理 skills
kssbox list                    # 查看所有可用 skills
kssbox add all                 # 安装全部
kssbox add harness-sync        # 安装单个
kssbox remove all              # 卸载全部
kssbox remove harness-sync     # 卸载单个
```

```bash
# 开发测试
claude --plugin-dir ./kssbox-plugin

# 发布安装
claude plugin marketplace add ./
claude plugin install kssbox@kssbox-dev
```

## TODO

- [ ] 自动循环迭代 skill

## 文档

<https://code.claude.com/docs/zh-CN/plugins>
