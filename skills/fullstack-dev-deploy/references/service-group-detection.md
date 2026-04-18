# 服务分组识别

## 目标

稳定地把用户层入口 `all`、`biz`、`boss` 映射成具体 app 集合，供 `scripts/dev.sh`、`scripts/deploy.sh` 和资产治理阶段共用。

## 目标输出

```text
serviceGroups{
  all: [...],
  biz: [...],
  boss: [...]
}
```

其中：

- `all` 必须覆盖所有可运行或可部署的 app
- `biz`、`boss` 可以是空候选，但不能在未确认时假装确定

## 识别优先级

按以下顺序判断分组：

1. 用户明确说明的服务映射
2. 目录命名与旧脚本命名同时一致
3. 前后端配对关系明显，例如 `frontend-biz` + `backend-biz`
4. 旧启动脚本、旧部署脚本、旧 compose 服务名中的分组线索

只有第 2-4 条至少两处证据一致时，才允许自动推断。

## 推荐判断规则

### `all`

始终由全部 app 组成，是最容易确认的分组。

### `biz`

可以从以下线索识别：

- 目录名包含 `biz`
- 旧脚本命令中有 `biz`
- 旧 compose 或服务配置中有 `biz`
- 前后端 app 可以稳定配对到 `biz`

### `boss`

可以从以下线索识别：

- 目录名包含 `boss`
- 旧脚本命令中有 `boss`
- 旧 compose 或服务配置中有 `boss`
- 前后端 app 可以稳定配对到 `boss`

## 与资产治理的关系

服务分组不能只服务于“新入口生成”，还必须能支撑旧资产治理：

- 旧 `run.sh`、`start.sh`、`deploy.sh` 的服务语义，要回写到 `serviceGroups`
- `convergencePlan` 中的 keep / migrate / merge / delete，必须复用这套分组口径
- 不允许旧脚本按一套 `biz` / `boss` 语义，新入口按另一套语义

## 复杂项目处理

复杂项目允许内部是 app 级清单，例如：

```text
all  -> frontend-biz + backend-biz + frontend-boss + backend-boss
biz  -> frontend-biz + backend-biz
boss -> frontend-boss + backend-boss
```

对用户暴露的接口不变，仍然只使用 `all`、`biz`、`boss`。

## 必须追问用户的情况

出现以下任一情况，停止自动推断并追问：

- 有多个 `biz` 候选或多个 `boss` 候选
- 只有前端或只有后端能被识别，另一侧缺失
- 某些 app 同时看起来属于 `biz` 和 `boss`
- 仓库没有 `biz` / `boss` 命名，但用户仍要求这两个入口
- 旧脚本和旧 compose 的分组含义互相冲突

## 输出要求

最终摘要中必须显式列出：

- `all` 映射了哪些 app
- `biz` 映射了哪些 app
- `boss` 映射了哪些 app
- 哪些映射是自动识别
- 哪些映射是按假设处理
- 哪些旧资产的语义被收敛到这套分组规则中
