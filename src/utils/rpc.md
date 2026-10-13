# Monitor 数据兼容层

`src/utils/rpc.ts` 保留了原 Komari 客户端的 `KomariRpc`、`RpcClient` 和类型名称，供主题组件继续使用。当前分支的 `RpcClient.call()` 通过 [`dispatch()`](../monitor/transport.ts) 将受支持的方法映射到 Monitor REST 接口；方法名用于前端分发，不会作为 JSON-RPC 请求发送到 Hub。

本文描述当前 Monitor 适配版本。原 Komari RPC2 文档仅用于理解代码来源，不能作为本分支的接口支持清单。

## 请求地址与实时更新

普通数据请求使用当前站点的 `/api`，并携带同源登录凭据：

| 数据 | 实际接口 | 行为 |
| --- | --- | --- |
| 节点信息、最新状态、连通性检查 | `GET /api/nodes` | 映射为兼容类型；节点快照缓存 1.5 秒，并合并同时发起的读取 |
| 负载与 Ping 历史 | `GET /api/nodes/{id}/metrics?hours=…&points=…&series=…` | `series` 为 `metrics` 或 `ping`；相同请求缓存 15 秒，多节点查询最多并行 4 个请求 |
| 登录状态、站点设置 | `GET /api/me` 及主题配置接口 | 使用 [`getSharedApi()`](./api.ts) 的 `getMe()`、`getPublicSettings()`；不通过本兼容层的旧 RPC 方法读取 |
| 实时节点推送 | 同源 `/api/ws` | 由 [`src/utils/init.ts`](./init.ts) 单独建立 WebSocket，接收含 `nodes` 的快照，并继续使用 HTTP 轮询 |

`/api/ws` 使用与页面一致的主机，以及 HTTP 页面对应的 `ws:` 或 HTTPS 页面对应的 `wss:`。它独立于 `KomariRpc` 实例，不使用 RPC2 请求或响应格式。开发时 `/api` 及 WebSocket 由 Vite 代理到 `MONITOR_HUB` 指定的 Hub，默认目标为 `http://127.0.0.1:9911`。

## 受支持的方法

下表列出当前 `dispatch()` 实现的全部方法。可选参数用 `?` 表示；完整 TypeScript 类型以 [`rpc.ts`](./rpc.ts) 为准。

| `KomariRpc` 方法 | 内部分发方法 | 当前返回内容 |
| --- | --- | --- |
| `ping(signal?)` | `rpc.ping` | 成功读取节点快照后返回 `'pong'`，也可能使用缓存 |
| `getNodes()` | `common:getNodes` | 以节点 ID 字符串为键的 `Record<string, Client>` |
| `getNodesLatestStatus()` | `common:getNodesLatestStatus` | 以节点 ID 字符串为键的 `Record<string, NodeStatus>` |
| `getPublicNodesInformation()` | `public:getNodesInformation` | 节点信息数组 |
| `getNodeRecentStatus(uuid, limit?)` | `common:getNodeRecentStatus` | 默认最近 1 小时的负载历史及 `count`；当前忽略 `limit` |
| `getPublicClientRecentRecords(uuid, signal?)` | `public:getClientRecentRecords` | 默认最近 1 小时的负载历史数组 |
| `getPublicRecordsByUUID(params, signal?)` | `public:getRecordsByUUID` | 指定节点的负载历史及 `count` |
| `getRecords(params)` | `common:getRecords` | `type: 'load'` 或 `'ping'` 对应的历史记录及 `count`、`tasks`；此方法声明返回 `unknown` |
| `getLoadRecords(uuid?, hours?, loadType?, maxCount?, signal?)` | `common:getRecords` | 包含负载历史 `records` 数组的对象 |
| `getPingRecords(taskId?, hours?, maxCount?, signal?, uuid?)` | `common:getRecords` | Ping 历史 `records` 和任务信息 `tasks` |
| `getPublicPingRecords(params, signal?)` | `public:getPingRecords` | Ping 历史及 `count`、`tasks` |
| `getPublicPingTasks()` | `public:getPublicPingTasks` | 从最近 1 小时 Ping 历史提取任务，并合并同一任务关联的节点 ID |

`uuid` 是 Monitor 数字节点 ID 的字符串，例如 `'12'`，不是 Komari UUID。`getNodes()` 与 `getNodesLatestStatus()` 不接收节点筛选参数；读取单个节点时需从返回的映射中选取。历史查询的 `uuid` 不在当前可见节点列表中时，返回空记录。

历史查询默认 `hours = 1`、`points = 600`。`getRecords()` 支持 `max_count` 与 `maxCount`，前者优先；`getLoadRecords()`、`getPingRecords()` 的 `maxCount` 也映射到 Hub 的 `points` 参数。省略 `uuid` 会查询当前可见的全部节点，记录合并为一个数组，点数参数分别作用于每个节点。实际点数、汇总步长和可查询时长由 Hub 决定；页面根据 `/api/me` 的 `history_days` 生成可选时间范围，兼容层不会自动裁剪超出范围的调用参数。

`load_type` 与 `loadType` 仅保留在签名中，当前不会筛选单个指标。负载历史包含 CPU、内存、磁盘和上下行速率，以及 Hub 支持时提供的采样步长与峰值字段。`getNodeRecentStatus()` 和 `getPublicClientRecentRecords()` 也读取该历史接口，不代表最近一分钟的完整实时状态。

Ping 记录的 `value` 单位为毫秒，Hub 返回空延迟时映射为 `-1`；`monitor_window_loss` 和 `monitor_bucket_loss` 分别表示窗口及采样桶的丢包率。提供非零 `task_id` / `taskId` 时按任务筛选记录，省略时返回全部线路；`tasks` 仍包含返回历史中的任务信息。

## 调用示例

```typescript
import { getSharedApi } from '@/utils/api'
import { getSharedRpc } from '@/utils/rpc'

const rpc = getSharedRpc()

const nodes = await rpc.getNodes()
const statuses = await rpc.getNodesLatestStatus()
const nodeId = Object.keys(nodes)[0]

if (nodeId) {
  const node = nodes[nodeId]
  const status = statuses[nodeId]
  console.log(node?.name, status?.online)

  // 当前返回对象的 records 字段为数组；loadType 留空，读取所有受支持指标。
  const { records: loadRecords } = await rpc.getLoadRecords(
    nodeId, 24, undefined, 600,
  )

  // 第五个参数是节点 ID；taskId 留空时查询该节点全部线路。
  const { records: pingRecords, tasks } = await rpc.getPingRecords(
    undefined, 1, 600, undefined, nodeId,
  )

  // 也可使用对象参数，避免混淆可选参数的位置。
  const history = await rpc.getPublicRecordsByUUID({
    uuid: nodeId,
    hours: 24,
  })
  const pingHistory = await rpc.getPublicPingRecords({
    uuid: nodeId,
    hours: 1,
  })

  console.log(loadRecords, pingRecords, tasks, history.count, pingHistory.count)
}

// 登录状态与站点设置使用已适配 Monitor 的 API 方法。
const api = getSharedApi()
const me = await api.getMe()
const settings = await api.getPublicSettings()
console.log(me.logged_in, settings.sitename)
```

Hub 决定当前用户可见的节点及字段。访客调用兼容层不会获得 Hub 未下发的节点 IP 或私有备注。

## 遗留配置与连接方法的限制

仍可创建 `new KomariRpc()` 或 `new RpcClient()`，但所有实例的普通调用都共享 `src/monitor/transport.ts` 的节点及历史缓存。

| 遗留选项或方法 | 当前行为 |
| --- | --- |
| `baseUrl`、`VITE_API_BASE` | 构造器仍保存旧 RPC2 地址，默认将 `VITE_API_BASE`（未设时为空字符串）与 `/rpc2` 拼接；普通 `call()` 不使用此地址，不能用它切换 Monitor Hub |
| `timeout` | 仍保存旧客户端超时值，默认 30 秒；当前 Monitor REST 请求使用 `AbortSignal.timeout(15000)`，不受此选项控制 |
| `useWebSocket`、`getClient().setTransport()` | 不改变普通 `call()` 的 REST 分发，也不控制页面的 `/api/ws` |
| `getClient().ensureWebSocketConnected()`、`ensureWebSocketConnectedWithPing()` | 会尝试建立遗留 RPC2 WebSocket；Monitor 页面初始化不使用这些方法，不应将其用作 `/api/ws` 的连接方式 |
| `close()`、`resetSharedRpc()` | 关闭该实例遗留的 RPC2 连接，后者同时清除共享 `KomariRpc` 实例；不关闭独立的 `/api/ws`，也不清空 transport 缓存 |
| 方法中的 `signal` | `call()` 仅在调用开始时检查是否已中止；当前不向 `dispatch()` 转交信号，因此调用中途 `abort()` 不会取消该历史请求 |

如需改变开发时连接的 Hub，请使用 `MONITOR_HUB`；生产请求使用部署站点的同源 `/api`。

## 错误处理与不支持的能力

表中未列出的旧 RPC 方法不受当前适配支持，即使 `KomariRpc` 类仍保留对应方法，也会抛出 `RpcError`，`code` 为 `-32601`，消息为 `Monitor 不提供此能力：<方法名>`。这包括旧 RPC 的版本、方法发现、帮助和站点设置查询，以及管理操作、审计、访客事件上报和通用指标查询。需要管理功能时使用 Monitor 的 `/admin/` 页面；站点设置由主题的 Monitor 配置接口处理。

REST 非成功响应抛出普通 `Error('Monitor API <状态码>')`，401 还会跳转到 `/admin/`；网络、超时和 JSON 解析错误直接向调用方传递，不能假设全部错误都是 `RpcError`。调用开始时已中止的信号会抛出 `DOMException`，名称为 `AbortError`。

```typescript
import { getSharedRpc, RpcError } from '@/utils/rpc'

try {
  const pong = await getSharedRpc().ping()
  console.log(pong)
}
catch (error) {
  if (error instanceof RpcError && error.code === -32601) {
    console.error('当前 Monitor 适配未实现此方法：', error.message)
  }
  else if (error instanceof Error) {
    console.error('Monitor 请求失败：', error.message)
  }
  else {
    console.error(error)
  }
}
```

## 代码与上游来源

- 当前支持清单与 HTTP 行为：[`src/monitor/transport.ts`](../monitor/transport.ts)。
- 兼容类、公开方法签名与错误类型：[`src/utils/rpc.ts`](./rpc.ts)。
- Monitor 实时更新与页面连接生命周期：[`src/utils/init.ts`](./init.ts)。
- 目标后端：[极简探针 Monitor](https://github.com/monitor-probe/monitor)。
- 原客户端协议参考：[Komari RPC2 接口文档](https://www.komari.wiki/dev/rpc.html)、[JSON-RPC 2.0 规范](https://www.jsonrpc.org/specification)。
