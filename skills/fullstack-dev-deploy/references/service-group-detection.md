# 服务分组识别

## 目标

稳定地把用户层入口 `all`、真实服务名和可选聚合组映射成具体 app 集合，供 `scripts/dev.sh`、`scripts/deploy.sh` 和资产治理阶段共用。

## 目标输出

```text
serviceGroups{
  all: [...],
  services: {
    backend: [...],
    h5: [...],
    weapp: [...]
  },
  groups: {
    admin: [...],
    mobile: [...]
  }
}
```

其中：

- `all` 必须覆盖所有可运行或可部署的 app
- `services` 记录目标项目真实存在的服务名
- `groups` 只记录已识别或已确认的聚合组；如果不存在稳定聚合语义，可以为空

## 识别优先级

按以下顺序判断服务映射：

1. 用户明确说明的服务名或服务集合
2. 目录命名、app 配置与旧脚本命名同时一致
3. 前后端配对关系明显，例如 `backend` + `h5`、`backend` + `weapp`
4. 旧启动脚本、旧部署脚本、旧 compose 服务名中的命名线索

只有第 2-4 条至少两处证据一致时，才允许自动推断。

## 推荐判断规则

### `all`

始终由全部 app 组成，是最容易确认的入口。

### 真实服务名

真实服务名通常直接来自目标项目，例如：

- `backend`
- `h5`
- `weapp`
- `admin-web`
- `gateway`
- `user-service`

识别线索包括：

- 目录名
- 现有脚本参数
- Compose 服务名
- README 中与实际配置相互印证的对外名称

### 可选聚合组

只有在以下条件满足时，才允许识别聚合组：

- 仓库中已有稳定的高层服务集合语义
- 同一组语义至少有两处证据一致
- 聚合组能稳定展开到具体 app 清单

例如：

```text
all     -> backend + h5 + weapp
services.backend -> backend
services.h5      -> h5
services.weapp   -> weapp
groups.mobile    -> h5 + weapp
```

## 与资产治理的关系

服务映射不能只服务于“新入口生成”，还必须能支撑旧资产治理：

- 旧 `run.sh`、`start.sh`、`deploy.sh` 的服务语义，要回写到 `serviceGroups`
- `convergencePlan` 中的 keep / migrate / merge / delete，必须复用这套映射口径
- 不允许旧脚本按一套服务语义，新入口按另一套语义

## 复杂项目处理

复杂项目允许内部映射到 app 级清单，但外部接口仍然优先暴露真实服务名，并可补充聚合组。例如：

```text
all                -> api-gateway + user-service + admin-web + portal-web
services.gateway   -> api-gateway
services.users     -> user-service
services.admin-web -> admin-web
services.portal    -> portal-web
groups.web         -> admin-web + portal-web
groups.backend     -> api-gateway + user-service
```

## 必须追问用户的情况

出现以下任一情况，停止自动推断并追问：

- 有多个同样合理的真实服务名候选
- 只有前端或只有后端能被识别，另一侧缺失且无法确认是否独立存在
- 某些 app 同时看起来属于多个聚合组，且证据冲突
- 仓库没有稳定聚合语义，但用户要求脚本暴露额外聚合组
- 旧 `dev` 与旧 `deploy` 对 `all` 或某个共享分组展开出不同 app 集合
- 旧脚本和旧 compose 的服务含义互相冲突

## 输出要求

最终摘要中必须显式列出：

- `all` 映射了哪些 app
- 识别出了哪些真实服务名
- 哪些聚合组被识别或确认
- 哪些映射是自动识别
- 哪些映射是按假设处理
- 哪些旧资产的语义被收敛到这套服务映射规则中
