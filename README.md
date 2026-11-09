# Network_Monitor_UI

区域 IT 总部网络运营展示平台 —— 一个**单文件交付、离线可运行**的网络与安全指挥大屏。

## 特性

- **单文件交付**：`network-cockpit.html` 一个文件搞定；核心图表与离线地图断网可运行，Google 地图仅在启用时按需加载。
- **多分辨率自适应**：支持等比缩放 / 铺满 / 响应式重排 / 运行时切换四种适配模式，适配 1080p～4K 及超宽屏。
- **区域总部与专题视图**：按职责拆分为“区域总览 / 基础设施 / 网络运行 / 网络安全 / 内部规划”，同时支持自动巡屏和一键打开安全副屏。
- **领导决策摘要**：直接给出区域健康结论、影响站点与业务、P1 紧急事项、容量风险，以及当前责任组和处置建议。
- **自动巡屏**：单屏模式每 18 秒轮播区域总览、基础设施、网络运行和网络安全；内部规划不会出现在访客巡屏中，进入编辑态时自动暂停。
- **可编辑大屏**：所有展示文字均可修改，包括动态数值、按钮、页签、拓扑与地图 SVG 标签，并可单独调整字号、颜色和字重；同时支持模块显隐与排序、主题色与密度调节，以及新增、修改和删除自定义指标、排行或明细模块。
- **可编辑设备台账**：自行维护设备名称、类型、厂商、型号、区域、Zabbix 主机名，以及会话/入口/出口 item key；大屏以台账为准，不再靠设备名称猜测展示范围。
- **五站点互联**：为每个 Site 配置主备运营商、IPsec 在线/期望隧道数、SD-WAN 当前路径、时延、抖动和丢包 item key，网络运行页统一展示站点可用性与路径切换。
- **防火墙变更审计**：设备台账可绑定 Zabbix 日志/文本 item；安全页展示最近 24 小时修改记录、设备、用户与原始动作，未关联审批信息的变更标记为“待核验”。
- **外网源—目的地图**：以苏州为源点，在一张世界地图叠加青色出站与粉色入站链路，并同步展示出站/入站流量、应用 TOP、目的国家 TOP 和会话数。
- **ASA / FTD VPN 异常用户**：读取 Zabbix 中的 VPN 认证日志，识别 rejected / failed / denied 以及 ASA 常见认证失败消息，按设备、用户名和公网来源 IP 聚合失败次数。
- **多数据源接入**：
  - **Zabbix（默认）**：支持 API Token、账号密码换取 `sessionid`、手填 Session ID；默认界面仅展示 Zabbix 与设备台账，高级来源收起。
  - **Fortinet FortiAnalyzer**：JSON-RPC / REST 双模式，获取攻击与拦截数据，字段映射可配置。
  - **设备会话 REST**：汇总 NetFlow / IPFIX / 防火墙会话类数据，展示主要源、目的、设备、会话数与字节数。
  - **统一 REST 数据入口**：按字段结构自动甄别会话流、安全事件或性能指标；低置信度数据标记为未知，不自动覆盖现有面板。
  - 支持通过反向代理注入认证，规避跨域（CORS）。
- **双地图引擎**：Google Maps JavaScript API 按需加载，同时保留内置示意图与用户自导入底图作为离线回退。
- **流量走向**：会话 REST 优先、FortiAnalyzer 次优先、演示基线兜底，并可切换查看原有端口流量与丢包排行。
- **全球双向流量地图**：同一模块并列展示出站与入站世界地图，按私网/公网方向自动分流，支持会话自带经纬度或由企业 GeoIP REST 服务补全公网 IP 位置。
- **资产流量 TOP**：服务器、存储、交换机与 AP 分成四个独立组，按采集窗口累计流量显示 TOP 5；实时会话数据可依据 `asset_type` 或设备名称特征自动归组。

## 适用场景

面向企业 / 园区网络管理员，统一展示多厂商设备（Cisco Catalyst / Nexus、Firepower、Palo Alto、Fortinet 等）的运行状态、安全态势与业务容量。

## 使用

直接用浏览器打开 `network-cockpit.html` 即可。Zabbix 地址与凭据在「自定义 → Zabbix」配置，要展示的设备在「自定义 → 设备台账」维护。

## Docker 部署

从 GitHub 拉取源码并启动：

```bash
git clone https://github.com/SRL-Development/Network_Monitor_UI.git
cd Network_Monitor_UI
docker compose up -d --build
```

浏览器打开：

```text
http://服务器IP:8765/
```

常用管理命令：

```bash
docker compose ps
docker compose logs -f
docker compose pull
docker compose up -d --build
docker compose down
```

修改宿主机端口：

```bash
DASHBOARD_PORT=8080 docker compose up -d --build
```

Windows PowerShell：

```powershell
$env:DASHBOARD_PORT=8080
docker compose up -d --build
```

每次推送 `main` 后，GitHub Actions 还会自动构建 `linux/amd64` 与 `linux/arm64` 镜像：

```bash
docker pull ghcr.io/srl-development/network_monitor_ui:latest
docker run -d --name network-monitor-ui --restart unless-stopped \
  -p 8765:80 ghcr.io/srl-development/network_monitor_ui:latest
```

健康检查地址为 `http://服务器IP:8765/healthz`。容器以只读文件系统运行，Nginx 缓存、运行目录和日志目录使用临时内存卷。

Zabbix 地址、设备台账、站点和布局配置保存在访问者浏览器的 `localStorage`，重建容器不会清除；但更换域名、端口或 HTTP/HTTPS 协议会被浏览器视为新的站点来源，需要重新配置或导入台账。Zabbix 跨域访问仍建议通过企业反向代理处理。

### 只有 Zabbix 时如何使用

1. 在「设备台账」登记实际设备，并填写与 Zabbix 完全一致的主机名。
2. 如果 Zabbix 已有当前连接数 / 会话数 item，把它的 `key_` 或唯一子串填入“会话总数 item key”。入口和出口流量同理。
3. 页面能取得会话总数时会标记为“Zabbix · 会话汇总”；只有接口流量和设备状态时会标记为“Zabbix · 无会话明细”。
4. Zabbix 没有逐条源 IP、目的 IP、字节数时，页面不会生成虚假的流向。需要该能力时，应先让设备向采集器输出 NetFlow/IPFIX/日志，或接入厂商管理平台，再从“高级数据源”提供会话明细。

ASA / FTD、F5、Fortinet 的模板与固件能力不同，先在 Zabbix 的“最新数据”中确认真实 item key，再填入台账。台账和指标映射保存在当前浏览器 `localStorage`，支持 JSON 导入/导出。

### 五个 Site 的 IPsec + SD-WAN

在「自定义 → 站点互联」逐个修改 5 个预置站点，填写：

- 站点名称、对应 Zabbix 主机名；
- 主线路与备线路名称；
- 当前在线 IPsec 隧道数 item key 与期望隧道数；
- SD-WAN 当前选中路径 item key；
- RTT 时延、抖动、丢包率 item key。

网络运行页按站点展示“主/备线路、IPsec 在线/期望、RTT、抖动、丢包、当前路径”。实时模式没有映射指标时会显示“待映射”，不会用演示值替代。

### 防火墙配置修改审计

只依靠普通 SNMP 轮询无法知道“谁修改了什么”。建议流程：

1. ASA/FTD、FortiGate、F5 等设备将配置审计日志发送到内部 syslog；
2. 由 Zabbix 的日志 item、文本 item 或 SNMP trap item 接收；
3. 在「设备台账 → 配置修改审计 item key」填写对应 key；
4. 安全页切换到“配置变更”，查看最近 24 小时历史；
5. 在 Zabbix 为“configuration changed / commit / policy deploy”等事件建立 trigger，并配置邮件、Teams 或短信媒介，才能在大屏关闭时仍主动通知。

设备日志通常只能证明发生了修改以及账号/动作；“是否经过授权”需要额外接入审批记录。页面只呈现“已授权 / 待核验”等管理结论，不展示执行团队的流程流水。

### ASA / FTD VPN 异常认证用户

在「自定义 → 设备台账」为 ASA / FTD 填写“VPN 认证日志 item key”。安全页进入“风险与防火墙变更 → VPN 异常用户”后，页面读取最近 24 小时 Zabbix 日志历史，并按“设备 + 用户名 + 公网来源 IP”聚合。

默认识别 `rejected`、`failed`、`denied`、`invalid`、`lockout`、`timeout`，以及 ASA `113005`、`113015`、`716039`、`611102`、`109006` 等常见认证失败消息。大屏用于展示和排序；建议同时在 Zabbix 对短时间内失败次数建立 trigger，才能主动告警密码喷洒或暴力尝试。

外网源—目的地图需要逐条会话的源/目的 IP、方向和字节数。GeoIP 未配置或 IP 无法定位时，页面只统计为“待定位”，不会生成虚假国家或落点。

### 苏州外网流向与五站点流量

外网地图优先使用会话数据中的 `app/application/service`、`dstcountry/dst_country`、源/目的 IP、方向、会话数和字节数。FortiGate / FortiAnalyzer 流量日志可以提供 `app`、`appcat`、`dstcountry`、`sentbyte`、`rcvdbyte`；F5 GeoIP 或其它地理数据源也可提供目的国家。只有国家名称、没有坐标时，页面对常见国家使用内置中心点落图；无法识别的国家仍标记为“待定位”。

“5 站点 · IPsec + SD-WAN”模块可以切换到“到苏州流量”。在「自定义 → 站点互联」为每个站点配置采集设备、站点到苏州上行 bps item 和苏州到站点下行 bps item。

双 IPsec 的推荐口径：

1. 以苏州 Hub 侧真正终结两条 IPsec 的 SD-WAN/防火墙设备作为主采集点；
2. 分别采集两条隧道接口、VTI 或 SD-WAN member 的流量计数；
3. 在 Zabbix 建立 calculated item，将两条隧道的 bps 求和后映射到本页面；
4. 当前活动路径、隧道在线数、时延、抖动、丢包单独映射；
5. 分支侧设备用于交叉核对，不建议把两端流量再次相加，否则会重复统计；
6. 不要使用物理 WAN 总接口作为站点专线流量，因为其中可能包含普通 Internet 出口流量。

### 页面组织

- **区域总览**：区域健康结论、业务可用率、P1 事项、容量风险、风险对象、业务影响与责任、总部/数据中心/分支站点和流量，适合集团来访或地区管理层一眼判断。
- **基础设施**：服务器、存储、交换机、AP 的健康度和流量 TOP，以及容量水位、设备拓扑、站点分布和硬件维护事项。
- **网络运行**：骨干吞吐、网络拓扑、站点地图、链路质量、会话与全球出入站流量、运营商和 WAN 质量。
- **网络安全**：安全评分、攻击趋势、攻击类型和高风险对象。
- **内部规划**：备份与灾备、RPO/RTO、生命周期、容量预测和规划优先级；带内部标识且不参与自动巡屏。
- **双屏展示**：在专题视图点击“双屏展示”，当前窗口切到网络运行，同时打开网络安全副屏。
- **自动巡屏**：点击“自动巡屏”后按区域总览 → 基础设施 → 网络运行 → 网络安全循环展示；再次点击即可停止，也可使用 `?view=paged&tour=1` 直接启动。

### 自定义模块

在「自定义 → 模块管理」填写标题，选择单值指标 / TOP 排行 / 明细列表、数据通道和放置区域，然后点击“新增模块”。列表中的“编辑”可修改已有模块；双击大屏模块标题也可直接改名。

### 全局文字与字体编辑

打开「自定义 → 模块管理」后，大屏中的所有文字会进入可选择状态：单击选择文字，双击直接修改内容，按 Enter 保存、Esc 取消。选中后还可以设置字号、颜色和字重；手工修改会保存在当前浏览器，并优先于实时数据展示。可恢复当前文字、恢复全部默认值，或导出文字覆盖配置作为备份。

### 自动数据分流

在「自定义 → 数据接入 → 统一 REST 数据入口」配置一个 GET JSON 接口。系统按字段特征识别：

- `src/dst + bytes/sessions` → 会话流量；
- `severity/message/action/attack` → 安全事件；
- `name/key + value/lastvalue` → 性能指标。

类型得分相同、缺少关键计数字段或结构无法识别时，结果保持为 `unknown`，不会错误覆盖当前大屏。专用会话接口优先级仍高于统一入口。

网络运行页的「资产分类流量 TOP」会进一步把会话记录归入服务器、存储、交换机或 AP。推荐数据源同时提供 `asset_name` 和 `asset_type`；`asset_type` 可使用 `server`、`storage`、`switch`、`ap`，系统也会用常见设备命名（如 `SRV-`、`PowerStore`、`Core-`、`AP-`）辅助识别。无法可靠归组时不会把记录硬塞入错误类别，该组继续显示“演示基线”。

### 全球会话流向与 GeoIP

在网络运行页的「主要设备会话与流量走向」点击“全球流向”，即可查看出站和入站两张世界地图。系统将“私网源 → 公网目的”识别为出站，将“公网源 → 私网目的”识别为入站；也可以由数据源直接提供 `direction: inbound/outbound`。

定位方式按以下顺序使用：

1. 会话记录直接提供源/目的国家和经纬度；
2. 在「自定义 → 数据接入 → IP 地理定位」配置企业内部 GeoIP REST 地址模板，例如 `https://geo.internal/lookup?ip={ip}`；
3. 仍无法定位时只统计为“待定位”，不会生成虚假落点。

GeoIP 默认关闭，避免把企业会话 IP 发送给未经授权的第三方。推荐通过同源反向代理接入内部 GeoIP/MaxMind 服务。

### Google Maps

在「自定义 → 地图与数据」填写 Maps JavaScript API Key 并启用。Google Maps Platform 要求项目启用计费；生产 Key 应限制为大屏站点的 HTTP Referrer，并仅授权 Maps JavaScript API。建议通过 Web 服务器访问页面，不要以 `file://` 作为生产部署方式。

### 会话 REST 数据格式

接口使用 GET，返回数组，或使用 `data/result/items/rows/sessions` 包装。以下字段名均支持常见别名：

```json
{
  "device": "FW-02",
  "asset_name": "Core-01",
  "asset_type": "switch",
  "src": "10.20.1.10",
  "dst": "203.0.113.20",
  "direction": "outbound",
  "dst_country": "新加坡",
  "dst_lat": 1.3521,
  "dst_lng": 103.8198,
  "service": "HTTPS",
  "sessions": 128,
  "bytes_in": 1048576,
  "bytes_out": 2097152
}
```

## 安全与运行说明

- Token、密码与 sessionid 仅写入当前标签页的 `sessionStorage`，关闭标签页后清除；接口地址、用户名、布局与展示配置仍保存在 `localStorage`。
- 生产环境建议通过 HTTPS + 同源反向代理访问 Zabbix、FortiAnalyzer 和会话接口，并由代理注入认证信息。
- 页面会阻止重叠轮询，并在标签页进入后台时暂停自动刷新，恢复可见后立即同步一次。
- 支持键盘操作主要按钮，并尊重系统“减少动态效果”设置。

> 注：Google Maps 的使用范围、跨境访问与地图测绘合规需由部署单位按所在地区法规评估；不满足条件时可始终使用离线地图。
