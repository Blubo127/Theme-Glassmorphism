# 仓库安全检查报告

- 检查日期：2026-10-09（北京时间）
- 项目仓库：[Blubo127/Theme-Glassmorphism](https://github.com/Blubo127/Theme-Glassmorphism)
- Git 分支：`diy`
- 检查提交：`0b2f707526e9140c6197ee657fb0c5ad00a5f78e`（版本 1.4.3）
- 检查时 origin：`https://github.com/Blubo127/Theme-Glassmorphism`
- 方法：源码静态审查、关键数据流追踪、锁定依赖来源及公开安全公告查询；对可疑路径进行独立复核。

报告主体记录上述提交在检查时的状态，代码证据链接固定到该提交，不代表后续分支已重新通过安全审查。项目地址与文档对齐记录见末尾附录。

## 1. 结论：未发现明确后门，但有外联和需要处理的安全风险

**在本次检查的当前分支源码中，未发现主动读取服务器凭据、SSH 密钥、环境配置或业务文件，并将它们发送到作者服务器的实现；也未发现反向 Shell、隐蔽远程命令执行或用于隐蔽上传服务器数据的预置地址。这个结果是“未发现证据”，不能作为百分之百安全保证。**

本项目是 Vue/Vite 编写的 Monitor 前端主题。正常部署后主要代码在访问者浏览器运行，通过本站 `/api` 与 `/api/ws` 获取监控数据。它不是完整的 Monitor Hub 或服务器探针 Agent，因此本报告不能证明底层后端、探针或主题包解压安装过程安全。

需要注意的具体问题如下：

| 项目 | 本次判断 | 对使用者的影响 |
| --- | --- | --- |
| 恶意后门、服务器数据主动外传 | 当前源码未发现明确证据 | 未发现把节点指标、私有备注、服务器文件或本站认证信息主动上传给作者的代码路径 |
| 默认第三方请求 | 已确认；中等隐私风险 | 访客定位、壁纸、图标、汇率会从访问者浏览器联系外部服务，第三方能看到请求来源 IP 等信息 |
| 远程 SVG 内容信任 | 已确认依赖渲染路径；条件型供应链风险 | 图标服务的 SVG 内容未经安全清理进入本站 DOM；若响应遭篡改，可能影响浏览器会话 |
| Ping 图表 HTML 注入/XSS | 未转义插值已确认；潜在高影响，利用前提未验证 | 如果攻击者能控制探测线路名称，可能在浏览器中执行脚本，以用户会话权限读取或操作本站 API |
| 锁定依赖漏洞 | 4 个依赖、5 条公开公告 | 涉及 DoS、条件型命令注入及原型污染后的代码执行；不能直接等同于主题线上可被远程控制 |
| 后端权限、发行包与源码一致性 | 尚未验证 | 必须进一步核对实际部署的 Hub/Agent、登录 Cookie、API 返回字段和最终主题包 |

**使用建议：先处理图表的未转义 HTML，按需要关闭或本地化外部服务，再在隔离环境处理依赖和构建。若服务器管理信息敏感，应核验后端匿名接口返回内容，并使用与本次检查提交对应的构建产物。**

## 2. 已确认的网络请求：发到哪里、发送什么

下表中的自动第三方请求均由浏览器发起。第三方通常能看到访问者的公网出口 IP、请求时间及浏览器提供的 HTTP 元信息，可能包含 Referer。现代浏览器默认跨站 Referer 通常仅包含本站 origin，但实际行为取决于浏览器和部署响应头，不能保证永远如此。如果浏览器运行在服务器上，请求来源也可能是该服务器的出口 IP。

| 目的地 | 触发条件 | 实际请求/数据 | 代码证据 |
| --- | --- | --- | --- |
| `ipwho.is`，失败后依次尝试 `ipapi.co/json/`、`api.ip.sb/geoip` | “显示访客信息”默认开启；页面初始化完成后自动查询 | 固定 GET，由对端依据请求来源 IP 返回访客位置；没有节点 IP、指标、密码或请求正文 | [theme.json](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/theme.json#L178)、[Header.vue](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/src/components/Header.vue#L98)、[VisitorInfo.vue](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/src/components/VisitorInfo.vue#L39) 和第 97、129、154 行 |
| `bing.biturl.top` | 自定义背景默认开启，默认类型为 `bing` | 加载壁纸图片，参数为分辨率、市场、索引和日期；代码未附加节点数据。该域名不是微软官方域名，最终图片重定向目的地未在线验证 | [theme.json](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/theme.json#L548)、[app.ts](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/src/stores/app.ts#L1161)、[Background.vue](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/src/components/Background.vue#L17) 和第 93、120 行 |
| `api.iconify.design`，备用为 `api.simplesvg.com`、`api.unisvg.com` | 渲染未在本地注册的 `<Icon>` 时 | GET `/{prefix}.json?icons=...` 获取 SVG 图标；外部服务可获图标名称和请求元信息；备用顺序随机 | [iconify.ts](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/src/utils/iconify.ts#L2)、[main.ts](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/src/main.ts#L12)；另已核对锁定的 `@iconify/vue@4.3.0` 包源码 |
| `open.er-api.com/v6/latest/CNY`，失败后尝试 `api.frankfurter.app/latest?from=CNY` | 首页挂载、当天汇率缓存缺失时自动查询 | 固定 CNY 汇率 GET；没有上传节点价格、账单或财务明细，计算在本地完成 | [HomeView.vue](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/src/views/HomeView.vue#L157)、[financeHelper.ts](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/src/utils/financeHelper.ts#L79) 和第 384、453、478 行 |
| `www.cpubenchmark.net/cpu_lookup.php?cpu=...` | 用户点击 CPU Mark 外链 | CPU 型号作为 URL 编码后的查询参数发送；链接使用 `noopener noreferrer` | [cpuBenchmark.ts](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/src/utils/cpuBenchmark.ts#L207)、[InstanceDetail.vue](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/src/views/InstanceDetail.vue#L570) |
| 管理员填写的背景、视频、图标、公告图片主机 | 配置站外 URL 后，在展示时加载 | 请求对应资源。公告图片支持站外 URL，包括 `//host/...`；没有发现默认隐藏追踪图片 | [Background.vue](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/src/components/Background.vue#L93)、[useSiteIcon.ts](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/src/composables/useSiteIcon.ts#L37)、[MarkdownRenderer.vue](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/src/components/MarkdownRenderer.vue#L99) 和第 119 行 |
| `github.com` | 用户点击页脚项目链接 | 普通外部页面跳转，使用 `noopener noreferrer` | [Footer.vue](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/src/components/Footer.vue#L37) 和第 52 行 |

当前节点数据的主要路径是 [transport.ts](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/src/monitor/transport.ts#L10) 的同源 `/api/nodes`、`/api/nodes/:id/metrics` 等 GET，以及 [init.ts](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/src/utils/init.ts#L36) 的同站 `/api/ws`。主题配置读取和保存使用同源 `/api/themes/glassmorphism/config`，见 [config.ts](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/src/monitor/config.ts#L18)、第 73、107 行。携带本站 Cookie 访问本站 API 是正常认证行为，未发现将该 Cookie 明文取出并附加给上述第三方请求的代码。

代码保留了旧版通用 API/RPC 客户端和 `VITE_API_BASE` 配置能力，但不能据此认定它们都在当前主题中运行。当前 `RpcClient.call()` 在 [rpc.ts](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/src/utils/rpc.ts#L716) 委托 Monitor 的 `dispatch()`，主要活跃接口已经改为同源调用；开发代理 `MONITOR_HUB` 的风险见第 5 节。

### Iconify 还存在远程内容信任边界

额外下载了锁文件指定的 `@iconify/vue@4.3.0` npm 包，验证其 SHA512 与锁文件 integrity 一致；只读取源码，没有安装或执行。包内 `dist/iconify.mjs:606` 配置上述三个默认 API，`:727` 构造图标 GET，`:182` 的校验只检查图标 body 为字符串及元数据类型，没有清理 SVG 的事件属性或外部引用。`:1631` 把处理后的 body 赋给 `innerHTML`，`:1633` 渲染为 SVG。ID 替换和几何变换不等于安全清理。

因此，图标服务是页面信任的内容供应方。若这些 API 的内容被恶意修改或篡改，SVG 事件处理器等可能在本站上下文产生 XSS；当前没有证据表明实际图标响应含恶意代码。它属于条件型供应链风险，不能当作已经发现的后门。将图标本地化并阻止远程回退可以消除这个信任边界；经过验证的 CSP 可进一步限制行内事件脚本。

### 怎样减少外联

1. 在实际生效的主题设置中将 `visitorInfoEnabled` 设为 `false`。
2. 将 `backgroundEnabled` 设为 `false`，或使用本站 `/images/...` 图片；图标、公告图片和视频也使用本站资源。
3. 将实际使用的 Iconify 图标本地注册或打包，避免按需获取远程 SVG。
4. 若需要完全禁止自动第三方请求，需修改汇率加载逻辑，使用固定或本站汇率；当前没有独立的汇率网络请求关闭开关。
5. 在部署环境以浏览器 Network 面板复核，包括缓存未命中、查询失败时的备用服务和图片重定向；可以使用经过验证的 CSP 限制 `connect-src`、`img-src` 和 `media-src`。同源 WebSocket、`data:` 图标及当前内联样式/脚本需要兼容处理，不建议直接套用未经测试的策略。

**只关闭访客信息和 Bing 背景并不能消除全部外联，Iconify 与汇率查询仍可能发生。** 实际设置由 Hub 配置和 `theme.json` 默认值决定；仅修改仓库中的 `public/glass-config.json` 不能保证线上行为改变，当前配置加载入口见 [config.ts](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/src/monitor/config.ts#L82)。

## 3. 重点代码问题：Ping 图表存在条件型 XSS

**位置：[PingChart.vue](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/src/components/PingChart.vue#L643)。** 自定义 ECharts tooltip 把 `item.seriesName` 直接插入返回的 HTML，没有做 HTML 实体转义。

数据流可以从源码追踪：

```text
本站 /api/nodes/:id/metrics 返回的 probes 线路名称
  → src/monitor/transport.ts:84-86 构造 task.name
  → src/services/history.service.ts:174 返回 records/tasks
  → src/components/PingChart.vue:337、345 接收 tasks
  → src/components/PingChart.vue:591 作为图表 series.name
  → src/components/PingChart.vue:643 原样拼入 tooltip HTML
```

仓库没有设置 `tooltip.renderMode: 'richText'`。额外只读核对锁定的 ECharts 6.1.0 公开源码：`TooltipModel.js:68` 默认模式为 `auto`，`lib/util/model.js:668` 在 DOM 浏览器下解析为 `html`，`TooltipView.js:585` 使用 formatter 返回值，`TooltipHTMLContent.js:315` 把字符串写入 `el.innerHTML`。因此 CanvasRenderer 不会自动消除这个 HTML 注入点。

这部分复核读取的是公开 CDN 的指定版本文件：[TooltipModel.js](https://cdn.jsdelivr.net/npm/echarts@6.1.0/lib/component/tooltip/TooltipModel.js)、[model.js](https://cdn.jsdelivr.net/npm/echarts@6.1.0/lib/util/model.js)、[TooltipView.js](https://cdn.jsdelivr.net/npm/echarts@6.1.0/lib/component/tooltip/TooltipView.js)、[TooltipHTMLContent.js](https://unpkg.com/echarts@6.1.0/lib/component/tooltip/TooltipHTMLContent.js)。没有校验完整 ECharts tarball 的 integrity；它们作为依赖行为的补充证据，与本仓库中未转义拼接的直接证据分开理解。

可复核的无害示例是线路名称 `<b>audit-marker</b>`：第 643 行会生成含 `<span ...><b>audit-marker</b></span>` 的 HTML，按该渲染路径会解释为标签，而不是显示原始名称文字。本次未在浏览器运行载荷，也未对实际 Hub 发起利用请求。

利用需要攻击者能影响线路名称、可信 API 响应或数据传输，且曲线有非空延迟值并触发 tooltip。本仓库没有后端任务编辑和过滤实现，**尚未证明匿名用户能控制这个字段**；后端是否拒绝 HTML、部署 CSP 是否阻止事件脚本也未验证。因此这是已确认的未转义 HTML 插值及条件型 XSS 风险，不能写成已确认匿名入侵。

如果成功在管理员浏览器中执行脚本，脚本可能调用同源 API 读取登录后可见的节点地址、私有备注等，或执行用户有权限的操作，再发送给攻击者。HttpOnly 能保护 Cookie 明文读取，但不能阻止同源脚本使用已有会话发请求。这是浏览器会话风险，**不是当前已确认的服务器操作系统命令执行**，也不是作者恶意植入的证据。

建议对名称做 HTML 实体转义，或改用 DOM 节点并设置 `textContent`。对同类自定义 tooltip 也应统一转义外部字符串，并在后端验证线路名称。修复后用上述标签名称及包含引号、`<`、`>`、`&` 的名称确认显示为普通文字。

对照检查：[LoadChart.vue](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/src/components/LoadChart.vue#L409) 也构造 HTML，但当前 series 名称为代码常量；真实地球城市标签在 [NodeEarthRealisticGlobe.vue](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/src/components/NodeEarthRealisticGlobe.vue#L111) 使用 `textContent`。公告 Markdown 文本用 Vue 插值并限制链接协议，未找到同样的脚本注入路径。

## 4. 锁定依赖的公开漏洞

2026-10-09 22:50:01（北京时间）向 npm 公共 bulk advisory API 查询了锁文件中的包名和版本，并通过 GitHub 公告 API 复核了下面 5 条公告。查询仅发送依赖名称、版本，没有发送源代码、仓库路径、业务数据、环境变量或凭据。

**以下等级是公开公告等级，不是本项目已经证实可被远程利用的等级。** 锁文件有 799 个依赖包条目、709 个不同包名、758 个包名/版本组合。结果为 4 个依赖涉及 5 条公告；这不是 `npm audit` 完整依赖树风险统计，也不是恶意包检测。

| 锁定版本与证据 | 公告等级及编号 | 攻击条件与当前项目关联 | 公告修复版本 |
| --- | --- | --- | --- |
| `braces@3.0.3`；[package-lock.json](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/package-lock.json#L4787)，开发依赖 | High；[GHSA-vfj7-8cjw-p6xm](https://github.com/advisories/GHSA-vfj7-8cjw-p6xm) | 极深嵌套的攻击者可控 brace pattern 导致栈溢出 DoS；由 chokidar、micromatch 引入。当前未启用 `vite-plugin-static-copy` 这条引入链，但未证明所有构建链均不可达 | 公告未列出修复版本，不应假设已修复 |
| `shell-quote@1.10.0`；[package-lock.json](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/package-lock.json#L9965)，开发依赖 | Critical；[GHSA-pqg4-j6r4-53mv](https://github.com/advisories/GHSA-pqg4-j6r4-53mv) | `quote()` 同一次调用中，comment token 后有带换行的攻击者可控字符串，并将结果交给 shell 执行；由 `npm-run-all2` 引入，当前 scripts 未使用它 | `1.11.0` |
| `source-map-js@1.2.1`；[package-lock.json](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/package-lock.json#L10019)，锁文件未标记 dev | High；[GHSA-68fv-2mgg-jv7q](https://github.com/advisories/GHSA-68fv-2mgg-jv7q) | 处理不可信 indexed source map 时，超大 section offset line 可阻塞事件循环；由 Vue 编译器、PostCSS、Tailwind 等引入，未发现公开 API 让匿名访客提交 source map 的实现 | `1.2.2` |
| `tinypool@2.1.0`；[package-lock.json](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/package-lock.json#L10462)，开发依赖 | Critical；[GHSA-5gmw-xhrv-c9v3](https://github.com/advisories/GHSA-5gmw-xhrv-c9v3) | 需要已有 `Object.prototype` 原型污染，污染 `execArgv` 或 `env` 后影响 Worker，可能加载攻击者代码；由 `eslint-plugin-format → oxfmt` 引入，当前 lint 用 oxlint | 该条为 `2.1.1` |
| 同上 | Critical；[GHSA-85c8-ppgw-ccpr](https://github.com/advisories/GHSA-85c8-ppgw-ccpr) | 需要原型污染 `filename`，且使用 `pool.run(task, options)` 传第二参数对象；公告明确不传第二参数的调用不受这一条影响 | `2.1.2`；同时修复两条应至少升级至该版本 |

锁文件的 dev 分类不能证明依赖是否进入最终浏览器 bundle，也不能证明服务器存在可利用入口。本次没有构建产物分析或动态利用验证，不能把这些公告直接解释成“安装主题就会泄露服务器密钥”。

建议删除不用的开发依赖、升级对应上游依赖或在验证兼容性后固定安全版本，再更新锁文件并重新查询公告。`oxfmt@0.35.0` 精确依赖 `tinypool@2.1.0`，见 [package-lock.json](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/package-lock.json#L9020)，单纯执行常规更新不一定消除它。对于尚无公告修复版本的 braces，可移除不必要的引入链并限制不可信 pattern 输入。不要在生产服务器直接执行未经验证的自动强制依赖修复。

公告查询入口：`https://registry.npmjs.org/-/npm/v1/security/advisories/bulk`。公告内容与依赖版本会变化，上表仅记录检查时的结果。

## 5. 安装、构建及发布过程

当前脚本未发现隐蔽数据读取或外传：

- [package.json](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/package.json#L13) 定义 Vite 开发/构建、Vue 类型检查、oxlint 和本地测试；根包没有 `preinstall`、`install`、`postinstall` 或 `prepare`。
- [vite.config.ts](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/vite.config.ts#L5) 仅启用 Vue 和 Tailwind 插件；第 8 行开发服务绑定 `127.0.0.1`，API 代理默认为 `http://127.0.0.1:9911`。`MONITOR_HUB` 可覆盖目标，使用时必须确认指向自己控制的 Hub；该变量不是预置的作者收集地址。
- [scripts/package.mjs](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/scripts/package.mjs#L4) 读取主题元数据、复制 `theme.json`、LICENSE、dist 和预览图，使用参数数组调用本地 `tar` 并计算 SHA-256。没有网络请求或凭据读取；清理范围是固定 release 目录下的主题目录。
- [scripts/demo-server.py](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/scripts/demo-server.py#L1) 生成固定演示数据，绑定 `127.0.0.1:9911`，只读取 `DEMO_AUTHED` 标志；没有读取真实服务器信息，也没有对外请求。
- [.github/workflows/release.yml](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/.github/workflows/release.yml#L16) 执行 checkout、setup-node、`npm ci`、构建打包，再用 GitHub token 发布 Release。未发现将 token 交给其他主机的实现。Actions 使用 `@v4` 可变 tag，固定完整 commit SHA 可减少供应链变更风险。

全部锁定依赖的 `resolved` 地址来自 `https://registry.npmjs.org`，且存在 integrity；未发现 Git、file、普通 HTTP 或不明注册源。**正规来源和完整性哈希只能帮助固定下载内容，不能证明包本身无恶意代码。**

锁文件标记有安装脚本的依赖是 `vue-demi@0.14.10`（第 1580 行）、`esbuild@0.28.2`（第 5668 行）和平台可选包 `fsevents@2.3.3`（第 6996 行）。本次未安装它们，也未审查这三个包全部安装脚本；运行 `npm ci` 仍存在第三方代码执行面。因此，构建应使用独立的低权限环境，不携带生产服务器 SSH 密钥或业务凭据；依赖安装与构建完成后，生产环境按项目设计只部署静态主题产物。

在原审查提交中，origin 是 `Blubo127/Theme-Glassmorphism`，而当时的 [README.md](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/README.md#L63)、[theme.json](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/theme.json#L7) 的发行链接指向 `Gongsc/Theme-Glassmorphism`；README 还声明基于另一份 Komari 原版源码移植。该地址问题的后续修正见附录。**不同仓库、不同 tag 或预构建 Release 包不能自动继承这份审查结论。** 应记录实际下载来源、版本和校验值，并核对它与本次提交的关系。本次没有下载或比对发行主题包。

## 6. 敏感数据、访客指纹及后端权限

### 访客指纹代码存在，但当前正常初始化不会启用

[visitorFingerprint.ts](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/src/utils/visitorFingerprint.ts#L186) 包含平台、语言、时区、屏幕、WebGL 显卡、硬件并发数等浏览器特征及站点范围 SHA-256 指纹。WebRTC 使用空 `iceServers`，没有外部 STUN/TURN；地址进入本地派生哈希，不能把它描述为原始地址上传。

当前 [app.ts](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/src/stores/app.ts#L825) 要求 `publicSettings.visitor_audit_enabled === true` 才启用，Monitor 适配的 [api.ts](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/src/utils/api.ts#L412) 返回值没有提供该字段，所以正常初始化为关闭。[rpc.ts](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/src/utils/rpc.ts#L716) 实际走的 [transport.ts](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/src/monitor/transport.ts#L94) 也不支持 `public:recordVisitorEvent`，最终拒绝该方法。**没有证据表明当前分支正在向第三方上报浏览器指纹。** 以后如果恢复该功能，需要重新审查采集开关、具体接收地址和发送字段。

### 本地存储与导出

源码中的 localStorage 主要保存主题偏好、分组/收藏/对比节点 ID、Ping 缓存、汇率和费用估算设置；sessionStorage 保留随机访客审计会话 ID。未检出读取 `document.cookie`、硬编码私钥/访问密钥、把认证 token 保存到本地存储或打印密码的路径。

快照导出在 [snapshot.service.ts](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/src/services/snapshot.service.ts#L62) 使用 Blob 和浏览器下载，不是上传。导出可能包含登录后可见的节点地址及费用信息，见 [SnapshotExportPanel.vue](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/src/components/SnapshotExportPanel.vue#L158)，下载文件应按其实际敏感程度保存。CSV 单元格对公式前缀做了处理，见 [csv.ts](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/src/utils/csv.ts#L7)。

### 隐藏字段不等于后端保密

“未登录隐藏价格”控制页面展示；[mapping.ts](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/src/monitor/mapping.ts#L47) 仍会把 API 已返回的价格、币种、付款周期存入浏览器状态。如果价格不能对匿名访客公开，需要 Hub 在匿名响应中不下发它，而不能只隐藏卡片。

节点 IP 和私有备注同样依赖 Hub 不向匿名用户返回；本前端会使用收到的字段，见 [mapping.ts](https://github.com/Blubo127/Theme-Glassmorphism/blob/0b2f707526e9140c6197ee657fb0c5ad00a5f78e/src/monitor/mapping.ts#L7)、第 36、50 行。仅凭 README 或前端的登录显示条件，无法验证实际部署的 `/api/nodes`、历史接口和 WebSocket 是否正确过滤。当前证据不足以断言它们存在匿名泄露。

应在实际环境分别以匿名与管理员会话核验这些接口、主题配置 PUT 权限、Cookie 的 HttpOnly/Secure/SameSite、CSRF 和 WebSocket Origin 检查。该核验属于后端及部署检查，本仓库未提供实现。

## 7. 本次检查范围与未验证项

本次对当前提交的 476 个 Git 跟踪文件进行文件清单与模式检查，其中有 164 个 TS/Vue/Python/MJS/CSS/HTML 源文件；对网络调用、配置入口、认证、存储、HTML 渲染、导出、构建脚本和 CI 进行了重点人工追踪。285 个 SVG 未检出脚本、事件属性、`javascript:`、`foreignObject` 或外部 href 引用；SVG 标准命名空间 URL 不属于网络外传。

本仓库自有源码未检出 `sendBeacon`、XHR、`eval`、`new Function`、下载任意 JS 后执行、反向 Shell、云元数据读取或服务器敏感路径读取的实现。已单独记录第三方 Iconify 的远程 SVG `innerHTML` 渲染风险。搜索结果用于辅助审查，未命中并不能证明不存在其他形式的漏洞。除有针对性核对的公开依赖源码外，没有逐行审计所有第三方依赖。

本次未启动应用、未安装或执行依赖、未运行仓库构建/测试脚本，未执行浏览器攻击载荷，也未抓取实际部署流量。检查中仅写入此报告，没有修改业务代码、配置或锁文件。

未验证的范围包括：最终 dist/Release 包是否与源码一致、第三方图片重定向和线上响应内容、全部依赖/原生二进制的恶意行为、完整 Git 历史、Monitor Hub 与 Agent 源码、生产服务器权限和网络出口策略。公开公告查询也不能发现所有未知漏洞或恶意供应链变更。

这份报告可作为当前源码的风险判断依据；对于“整个服务器是否会泄密”，仍需要结合实际构建产物和后端部署验证。

## 附录：项目地址与文档对齐（2026-10-10，2026-10-12 复核）

本次以当前 Git origin `https://github.com/Blubo127/Theme-Glassmorphism` 对齐了项目入口：README 的安装包下载链接、`theme.json` 的项目 URL、页面主题链接，以及 package 元数据中的 repository/homepage/issues。主题与 package 的维护者为 Blubo127，Gongsc 保留在 package 的 contributors 中；README 区分当前维护仓库、Monitor 移植来源和原始 Komari 主题来源，LICENSE 保留 Tony Liu 的原始 MIT 版权声明。

第 5 节记录的旧安装链接与 origin 不一致属于原审查时的事实，以上文档和元数据问题已在本次修正。已安装的旧主题包仍持有原元数据，需要重新构建并上传本仓库的包。

2026-10-12 通过 GitHub API 复核，本仓库默认分支为 `main`，当前检查分支为 `diy`，尚无 Release 或 tag，Issues 也未启用。因此 README 补充了从 `diy` 本地打包和通过本仓库 Actions 发布的步骤，标明继承的上游更新历史与反馈入口状态；修改下载链接本身不会创建安装包或发布版本。

同时将 `src/utils/rpc.md` 改为当前 Monitor 兼容层说明，并把 `public/glass-config.json` 配置导入示例与 `theme.json` 默认值同步。这次调整不构成新的完整安全审计；上述 XSS、远程 SVG 信任和依赖漏洞仍需单独处理。
