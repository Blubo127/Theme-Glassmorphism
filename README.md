# Theme-Glassmorphism · Blubo127

![Glassmorphism 主题首页预览](preview.png)

这是由 [Blubo127](https://github.com/Blubo127) 维护的极简探针玻璃拟态主题仓库，基于 [Gongsc 的 Monitor 移植版本](https://github.com/Gongsc/Theme-Glassmorphism) 继续开发。移植版本源自 [sanrokamlan-prog/komari-theme-Glassmorphism](https://github.com/sanrokamlan-prog/komari-theme-Glassmorphism) v3.3.7，使用 [极简探针 Monitor](https://github.com/monitor-probe/monitor) 的 REST 与 WebSocket 接口提供数据，无需部署 Komari 服务器。

当前源码版本为 1.4.3。本仓库沿用 [MIT 许可](LICENSE)，保留 Tony Liu 的原始版权声明和上游来源；主题及 package 元数据中的维护者为 Blubo127。

## 项目地址与来源

| 用途 | 地址 |
| --- | --- |
| 当前维护仓库 | [Blubo127/Theme-Glassmorphism](https://github.com/Blubo127/Theme-Glassmorphism) |
| 安装包与版本发布 | [本仓库 Releases](https://github.com/Blubo127/Theme-Glassmorphism/releases) |
| 问题反馈（待启用） | [本仓库 Issues](https://github.com/Blubo127/Theme-Glassmorphism/issues) |
| Monitor 移植来源 | [Gongsc/Theme-Glassmorphism](https://github.com/Gongsc/Theme-Glassmorphism) |
| 原始 Komari 主题 | [sanrokamlan-prog/komari-theme-Glassmorphism](https://github.com/sanrokamlan-prog/komari-theme-Glassmorphism) |
| Monitor 后端与探针 | [monitor-probe/monitor](https://github.com/monitor-probe/monitor) |

本说明对应 [`diy` 分支](https://github.com/Blubo127/Theme-Glassmorphism/tree/diy)。截至 2026-10-12，GitHub 默认分支仍为 `main`；获取本分支源码时请明确选择 `diy`，后续安装已发布版本时可选择相应 tag。上游链接用于说明来源，Monitor 链接用于获取后端与探针。

仓库首页默认展示默认分支的 README。若希望首页展示这里的定制说明，可将 `diy` 设为默认分支，或将需要的修改合并到 `main`。

截至 2026-10-12，本仓库尚未启用 Issues；维护者可在仓库 Settings → General → Features 中启用 Issues，之后上述反馈入口与 package 元数据中的 issues 地址才可使用。

## 配置与开发文档

- [theme.json](theme.json) 定义主题元数据、设置字段和默认值；站点实际设置由 Hub 的 `/api/themes/glassmorphism/config` 保存与读取。
- [public/glass-config.json](public/glass-config.json) 是与默认值一致的配置导入示例，可通过主题设置面板导入并保存；页面不会自动读取这个文件作为站点配置。
- [Monitor 接口兼容层说明](src/utils/rpc.md) 记录当前主题支持的方法与传输方式。
- [安全检查报告](check_summary.md) 记录指定提交的审查结果和后续地址对齐说明。

## 主要功能

- **玻璃拟态外观**：提供翡翠、柔和、高对比、午夜和自定义五种配色，支持浅色、深色及按北京时间自动切换。背景默认使用 Bing 每日壁纸，也可设置图片或视频；关闭自定义背景时显示内置背景图。
- **地球与地图**：提供贴图地球、Cobe 点阵地球、平铺世界地图和点阵平铺地图四种样式，标注节点所在国家，支持自动旋转或隐藏。启用「地球显示城市」后，将根据节点名称识别城市并标注城市名称。
- **首页概览**：概览卡片提供多种预设方案，并支持自定义顺序；另提供公告（支持 Markdown）、快捷筛选（收藏、离线、高负载、即将到期等）、区域筛选、搜索及节点收藏功能。
- **节点分组**：支持 Monitor 原生分组，节点按后台设定的顺序排列，并提供「未分组」入口。切换分组后，概览、地球及工具面板仅统计当前分组。
- **节点卡片**：提供 mini、compact、comfortable、large 四种卡片尺寸及列表视图；可开启「离线节点置底」（默认关闭）。离线卡片以淡化样式显示，并标注「离线」。
- **多线路延迟**：普通卡片默认显示至多 3 条探测线路，左侧为最新延迟及历史色条，右侧为最近 1 小时的丢包率及历史色条；点击线路可查看延迟详情。「卡片延迟样式」可切换为延迟曲线，悬停或点按曲线可查看对应时段的延迟与丢包率。
- **节点详情**：包含资源负载图、延迟图，以及费用与剩余价值计算。
- **首页工具**：启用首页工具后，访客可对比最多四台节点；管理员登录后还可使用性价比分析、历史健康概览及 CSV / JSON 数据快照导出。
- **站点图标**：可在设置中指定页面左上角、浏览器标签页和 iOS 主屏幕使用的图标，支持填写 http(s) 图片地址、站内路径，或在主题设置面板中直接上传图片；上传时优先生成 180×180 PNG，体积超限时尝试 128×128 PNG 或 180×180 JPEG，通过校验后保存在站点配置中，无需另行托管。由于页面外壳为静态文件，首次加载时浏览器标签页会短暂显示内置图标。
- **站点设置**：管理员登录后可在页面中直接调整设置并实时预览，保存至 Hub 后对全部访客生效。设置亦可在后台主题卡片中修改，两处共用同一份配置，并支持导出、导入及恢复默认值。

下列 1.3.0–1.4.3 更新记录继承自 Gongsc 的 Monitor 移植版本，说明当前源码已有的功能，不代表本仓库已发布同名 Release。

## 1.4.3 更新内容

- 添加到 iOS 主屏幕后从后台切回时，不再闪现「连接错误」：单次请求失败只计数并在 1–2 秒内快速重试，连续 3 次失败才提示；任意一次数据更新成功即恢复正常。
- 页面恢复时若未收到可见性变化事件，会在页面重新显示或获得焦点、且数据已明显过期时立即刷新并重新连接。

## 1.4.2 更新内容

- 手机等设备将页面切到后台再切回时，不再误报「连接错误」：页面隐藏时暂停轮询与推送并中止在途请求，回到前台后立即刷新数据并重新连接。

## 1.4.1 更新内容

- 管理员登录后，节点卡片在公开备注之后一并显示私有备注，重复的项只显示一次。私有备注仅在登录后由 Hub 下发，访客无法看到。
- 平铺地图与点阵平铺地图的右侧图例改为与统计卡片一致的样式：顶部显示在线节点数、在线率与进度条，下方逐行列出地区及其节点数；部分节点离线的地区以「在线数/总数」标出。地区较多时改为两列，放不下时可在图例内滚动。地图左上角的在线/离线标签已移除。
- 「首页探测线路名称或 ID」默认留空，自动显示可用线路，不再预填浙江电信、浙江移动、浙江联通。

## 1.4.0 更新内容

- 适配极简探针 1.3.2 的历史保留天数：负载图与延迟图的时间范围按 Hub 设定的保留天数（`history_days`，1–365 天）生成，访客与管理员可查看的范围相同；保留期比某一档多不到四分之一时不再并列显示两档。旧版 Hub 仍按访客 7 天、管理员 90 天处理。
- 超过 7 天的范围由 Hub 按小时汇总，负载图按 Hub 返回的点距（`step`）绘制，长范围不再出现重复的阶梯点。
- CPU 图以虚线绘制区间内的最高占用（`cpu_max`），健康概览的 CPU 峰值同样依据该值计算。
- 节点卡片显示站长在后台填写的公开备注（`public_remark`），访客与管理员看到的内容一致。
- 价格按货币代码格式化，如 `US$5`、`HK$5.50`、`NT$12`；Hub 1.3.1 之前保存的非标准货币值按原样显示。数据快照中的其他币种不再被标为人民币。

## 1.3.1 更新内容

- 适配极简探针 1.3.1 的任意付款周期：以 `<n>m` 形式存储的周期（如 18 个月、5 年）将正确计算价格与剩余价值，并显示为「18 个月」「10 年」等。
- 适配任意三字母货币代码：费用统计按汇率接口提供的对应汇率折算；无汇率的币种不计入合计，不再按人民币计算。
- 延迟线路按后台延迟监控页设定的顺序显示。
- 新增「站点图标」设置，可指定页面左上角、浏览器标签页和 iOS 主屏幕使用的图标。
- 移除背景地址的 `local:` 前缀，该前缀指向 Monitor 不提供的路径；背景地址按填写原样使用。

## 1.3.0 更新内容

- 依据 Monitor 后端源码核对各项数据来源，移除 Hub 未提供数据的功能，包括 GPU、温度、自定义标签、城市与 ASN 信息、拓扑工具，以及节点详情中无有效数据的网络连接曲线、进程曲线和负载、Swap 历史曲线。相关设置项与预设方案一并移除，已保存的失效取值将自动恢复为默认值。
- 节点详情的负载图精简为 CPU、内存、磁盘和网络四项，并采用两列布局。
- 管理员登录后，节点详情页的「网络信息」区域将显示节点的 IPv4 与 IPv6 地址。地址取自 Hub 为管理面板选定的地址，位于 NAT 之后的节点显示其公网出口地址；访客无法查看。
- 主题不再将节点 IP 发送至第三方查询服务，服务商仅依据节点名称与备注识别。
- Hub 提供各时间区间内的峰值速率时，网络图将以虚线绘制上下行峰值，「近一天峰值」亦改为依据峰值计算。
- 流量配额直接采用 Hub 按计费方式计算的本周期用量 `month_used`，与后台及流量告警保持一致。

## 安装

安装需要 `theme.tar.gz` 主题包，包含构建后的页面和主题元数据。本仓库的版本下载入口为 [Releases](https://github.com/Blubo127/Theme-Glassmorphism/releases)。

截至 2026-10-12，本仓库尚未发布 Release 或 tag。推送代码到 `diy` 分支不会自动生成下载附件；当前请先准备下方“开发”所需的环境，再从该分支本地打包：

```sh
git clone --branch diy https://github.com/Blubo127/Theme-Glassmorphism.git
cd Theme-Glassmorphism
npm ci
npm run package
```

生成的安装包为 `release/theme.tar.gz`。在极简探针后台依次进入「主题」→「上传主题包」，选择这个文件并启用。打包命令还生成带版本名的包和 `release/theme.tar.gz.sha256` 校验文件；后续本仓库发布 Release 后，也可直接下载其中的 `theme.tar.gz` 安装。

`theme.json` 的项目 URL 指向本仓库，并随打包进入主题包。已经安装的旧包若仍显示上游地址，需要重新上传本仓库构建的包以更新元数据。在 Hub 支持“从 GitHub 更新”、且本仓库已有对应 Release 附件后，可通过主题卡片获取后续版本；仅更换本地 Git remote 或推送分支不会更新已安装包的元数据。

## 与原始 Komari 主题的差异

- **节点地址与位置**：Monitor 仅向已登录用户返回节点 IP。登录后，详情页显示 IPv4 与 IPv6 地址，搜索支持按 IP 匹配，数据导出包含地址信息；访客无法获取上述信息。地球按国家中心坐标定位节点；启用「地球显示城市」后，城市依据节点名称推断，以上位置均不代表机房的实际地理位置。服务商仅依据节点名称与备注识别（访客为公开备注，登录后另含私有备注），主题不会将节点 IP 发送至任何第三方。
- **已移除的功能**：原版中的 GPU、温度、自定义标签、后端提供的城市与 ASN 元数据、拓扑工具及审计日志依赖 Monitor 未提供的数据，均已移除。当前地球的城市标注根据节点名称和匹配规则推断。
- **历史图表**：历史数据仅包含 CPU、内存、磁盘、上下行速率及 Ping；负载、Swap、连接数和进程数仅提供实时值，不绘制历史曲线。Hub 返回各时间区间内的峰值速率（`net_rx_max` / `net_tx_max`）时，网络图以虚线绘制峰值，详情页的「近一天峰值」亦据此计算；较早版本的 Hub 不提供该数据，此时仅显示平均速率。
- **流量统计**：流量配额采用 Hub 依据节点计费方式（上行、下行、取较大值或求和）计算的本周期用量 `month_used`，与后台及流量告警保持一致；较早版本的 Hub 不提供该字段，此时主题按相同规则，依据本月上传量与下载量计算。累计流量为节点接入以来的总量。
- **外部服务**：访客信息条默认启用，该功能会请求第三方 IP 查询服务，可在设置中关闭；默认的 Bing 壁纸通过 `bing.biturl.top` 加载，可更换背景或关闭自定义背景；汇率与图标的加载同样依赖外部服务。
- **后台管理**：后台管理功能由极简探针的 `/admin/` 页面提供，主题包不包含 Komari 后台。

## 开发

建议使用 Node.js 24.15.0 或更新的 24.x 版本及 npm。`package.json` 的 `engines` 声明更宽，但当前锁定的开发依赖 `jsdoc-type-pratt-parser@9.2.2` 要求 Node.js `^22.22.2 || >=24.15.0`，旧版本安装会提示 `EBADENGINE`。GitHub Actions 发布工作流使用最新的 Node.js 22；本地使用 22.x 时需至少为 22.22.2。`npm test` 使用 Node 的实验性类型剥离参数。常用命令如下：

```sh
npm ci
npm run dev       # 启动开发服务器
npm run build     # 执行类型检查并构建
npm run lint      # 检查 src/monitor 目录
npm test          # 运行测试
npm run package   # 生成 release/theme.tar.gz
```

`npm run package` 还要求系统提供支持 `--format=ustar` 的 `tar` 命令；Windows 下建议在 WSL 中打包。

开发服务器默认将 `/api` 请求代理至 `127.0.0.1:9911`。如需连接已开启公开状态页的 Hub，在启动 Vite 时传入 `MONITOR_HUB`（以下为 Bash / WSL 示例）：

```sh
MONITOR_HUB=https://hub.example.com npm run dev
```

若无可用的 Hub，可在另一终端运行 `python3 scripts/demo-server.py` 提供演示数据；运行 `DEMO_AUTHED=1 python3 scripts/demo-server.py` 会模拟管理员登录状态，并为节点提供文档保留网段中的示例地址。演示服务只提供节点、历史数据和登录状态的 HTTP 接口，不实现 WebSocket 推送、主题配置保存或 `/admin/` 后台；这些功能需要连接真实 Hub 验证。

`preview.png` 为使用演示数据截取的 1440×900 首页截图，其中访客 IP 为示例地址。重新截图时，请勿记录真实的访客 IP。

当前 `vite.config.ts` 将页脚构建标识固定为 `monitor-port`；它是历史移植标记，不是实际 Git 提交哈希。需要追溯源码时，以发布 tag 或具体提交为准。

## 发布

发布入口是 [Blubo127/Theme-Glassmorphism 的 Actions](https://github.com/Blubo127/Theme-Glassmorphism/actions)。Fork 仓库首次使用时需确认已在 GitHub 启用 Actions；[发布工作流](.github/workflows/release.yml) 已声明 `contents: write` 权限，使用本仓库的 GitHub token 创建 Release。

发布新版本时，将 `theme.json`、`package.json`、`package-lock.json`（顶层及 `packages[""].version`）和 `vite.config.ts` 中的版本号统一修改为新版本号并提交。然后在要发布的提交上创建格式为 `x.y.z` 的 tag，并推送到本仓库。tag 不带 `v` 前缀，且必须与三个 JSON 文件的版本一致。例如，将版本统一改为 `1.4.4` 并提交后：

```sh
git tag 1.4.4
git push origin HEAD
git push origin 1.4.4
```

GitHub Actions 会检出该 tag，执行 `npm ci` 和 `npm run package`，并在本仓库创建附带 `theme.tar.gz` 的 Release。工作流发布目标由当前仓库决定；当前自动发布只上传 `theme.tar.gz`，带版本名的本地包及 SHA-256 文件不会自动成为 Release 附件。
