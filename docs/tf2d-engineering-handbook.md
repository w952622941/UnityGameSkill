# TF_2D 在线手游技术建设与故障解决 AI 复用手册

副标题：服务器权威战斗、北京 ECS、HTTPS/WSS、数据保护、压测优化、Android 真机诊断与“松杆/选技后小幅位移”修复全过程

版本：1.1　事实审计日期：2026-07-23（Asia/Shanghai）　适用项目：Unity 2022.3 LTS + ASP.NET Core + PostgreSQL + Docker

> [IMPORTANT] 本手册按“已由代码/自动化/运行记录证明”“已由 Android 真机证明”“研究建议/待上线”三类记录事实。其他 AI 必须先核对当前工作树、构建文件和运行环境，再引用数字。历史通过不等于当前提交通过。

## 0. 一页结论

### 0.1 到目前为止完成了什么

- 建立了 Unity 客户端、ASP.NET Core 后端、PostgreSQL、Docker/Compose、HTTPS/WSS、诊断、构建、压测和迁移文档组成的一体化开发基线。
- 将战斗从“客户端决定伤害、掉落和奖励”迁移为服务器权威：客户端只发送移动、选技、复活和退出意图；服务器以固定 20 Hz 推进真实状态，永久资产只由服务器结算。
- 建立 `AuthoritativeRealtime` 与 `VerifiedLocal` 两条明确分离的路径。前者逐 Tick 服务器权威；后者本地即时模拟、服务器完整回放验证，不能把本地战果直接当永久资产。
- 在阿里云北京 Integration ECS 上完成 Docker 化部署、独立数据库、Nginx、受信任 HTTPS/WSS、证书自动续期、备份/恢复验证、发布与回滚路径。
- 建立不进入 APK 的压测工具和模拟基准工具；修正“战斗提前终局导致伪长稳”的负载模型，在 4 核 8 GiB Integration 上完成 30 场长稳、35 场四轮和 40 场资源红线验证。
- 建立 Android 真机 DIAG、紧凑剪贴板报告、写后回读校验、逐帧 motion trace、构建 GUID/战斗 ID/相关 ID 关联链路。
- 解决了战斗不启动、技能弹窗不关闭、计时不动、摇杆残留、怪物/角色大幅瞬移、死亡不结算、继续战斗误开新局、技能选择恢复丢失、报告复制/导出不可信、松杆/选技后小幅位移等问题。
- 最新用户真机反馈：连续改变移动方向后松开摇杆可以原地保持；多次技能选择后未再观察到位移。r10 还加入了“零输入时保留最后逻辑朝向”的修复和 16/16 自动回归。

### 0.2 当前可复用的关键成果

| 领域 | 已形成的可复用能力 | 证据状态 |
|---|---|---|
| Android 构建 | 独立 acceptance 包名、ARM64、环境注入、构建后恢复默认开关、SHA-256/签名核验 | 已自动验证并在华为 API 29 真机反复安装 |
| 云端 | 标准 Linux + Docker/Compose + PostgreSQL + Nginx HTTPS/WSS | Integration 已运行验证；不是 production |
| 战斗权威 | 固定 Tick、确定性 RNG、版本固定、租约、序号、快照、重连、接管、不可变结算 | 代码、数据库/WSS 自动化和真机功能验证 |
| 数据安全 | 每日备份、每周隔离恢复、恢复成功后才清理、业务历史 report-only | 云端真实备份恢复已验证 |
| 性能 | 内存活动注册表、常驻模拟器、20/10 Hz 状态分离、有界 lane/持久化槽、池化编码、delta/checksum 复用、等价算法 | 规则/wire 等价与持续补位 ECS 压测验证；实现尚未进入 TF_2D main |
| 诊断 | 逐帧输入/权威/预测/Transform 同帧记录，紧凑 JSON，复制/文件写后校验 | 自动化与真机报告验证 |
| 反作弊 | 服务器事实、租约/序号、防重放、幂等账本、Outbox、风险事件边界 | 核心权威边界已实现；设备证明/RASP 属后续 |
| Excel 配置 | 仓库外 Excel、登记后编译、服务端/客户端白名单产物、确定性 hash | 编译器和隔离门禁已实现；真实业务表按需求新增 |

### 0.3 当前不应被误称为完成的事项

- staging/production 的权威战斗默认开关仍应保持关闭；真机门禁通过不等于 production 已批准。
- 固定自有域名、生产证书、正式账号/支付、完整运营后台、Play Integrity/App Attest、iOS、跨地区扩容尚未完成。
- 4 核 8 GiB 的 Integration ECS 不是生产容量承诺。Stage33 镜像通过 30 × 180 秒和 35 场资格轮 + 三轮重复，但 35 最差 API 峰值 84.63%、40 首轮峰值 98.86%；运营建议按 30 场，40/45/50 不认证。该实现仍在隔离 R10 未提交工作树，不得描述为 TF_2D `main` 当前能力。
- r10 的“保留朝向”有确定性失败测试、修复测试和 16/16 回归，但本次用户只明确反馈“不再位移”；朝向仍应在下一轮真机清单中单独签收。

## 1. 如何让另一个项目 AI 使用本手册

### 1.1 接管前的固定动作

1. 读取仓库根目录规则、领域词汇、ADR、当前迁移续接记录和测试手册。
2. 执行只读盘点：`git status --short`、当前分支/HEAD、项目版本、Unity 版本、构建产物是否真实存在。
3. 将事实分为四层：源码声明、自动化结果、云端实际状态、真机实际观察。四层冲突时，不得选自己喜欢的一层。
4. 保护脏工作树和 Unity `.meta` GUID；不重置、不覆盖、不把历史修改当作本轮修改。
5. 在任何修复前先保存失败证据：报告、时间、Build GUID、Battle ID、Correlation ID、设备/网络、稳定复现步骤。
6. 只对隔离 development/acceptance 环境操作；不把测试数据、临时 URL 或调试开关带进 production。

### 1.2 证据优先级

| 优先级 | 证据 | 使用方式 |
|---:|---|---|
| 1 | 当前源码、当前测试 XML/JSON、实际 APK/镜像 hash | 判断“现在有什么” |
| 2 | 云端只读检查、健康检查、容器版本、迁移表、真实恢复演练 | 判断“部署后实际是什么” |
| 3 | 指定 Build GUID 的真机报告/录屏 | 判断“玩家看到什么” |
| 4 | ADR、Runbook、历史压测摘要 | 解释设计与历史；不能覆盖新证据 |
| 5 | 聊天记忆和推测 | 只能生成待验证假设，不能生成根因结论 |

### 1.3 安全规则

- GitHub 文档不得包含私钥内容、数据库密码、完整连接字符串、云访问密钥、会话令牌、TLS 私钥或安全组真实授权地址。
- 本地新增软件不得安装到 C 盘；需安装时先与项目所有者确认目标位置。
- 压测只针对项目所有者授权的测试实例；压测库与 Integration 业务库隔离。
- 不允许为调试方便把 SSH、PostgreSQL、Redis 或 Docker API 长期开放到 `0.0.0.0/0`。
- 不允许用客户端上传的伤害、胜负、金币或存档余额修补无法证明的服务器结算。

## 2. 最终形成的技术架构

### 2.1 运行拓扑

```text
Android Unity 客户端
  ├─ HTTPS：登录、玩家数据、云存档、创建/查询/重连/退出、最终回执
  └─ WSS：一次性 ticket、控制租约、移动输入、可靠离散命令、权威快照
          ↓
Nginx/TLS（公网仅 80/443）
          ↓
TF2D API 容器（模块化单体，可切 API/实时模拟/Replay Worker 角色）
          ↓
PostgreSQL（会话、租约、命令、检查点、终局、资产账本、Outbox、证据）
```

### 2.2 两种战斗模式不可混淆

| 模式 | 玩家手感 | 权威事实 | 适用范围 | 主要成本 |
|---|---|---|---|---|
| AuthoritativeRealtime | 客户端预测/插值，持续 WSS | 服务器每 Tick 决定位置、敌人、伤害、掉落和结算 | 实时交互、高风险玩法、回滚通道 | 活跃战斗持续占 CPU/网络；受 RTT 影响 |
| VerifiedLocal | 本机固定 Tick 即时推进 | 服务器从 Tick 0 回放完整证据，验证后结算 | 普通单人 PvE、弱网体验 | 回放排队、证据存储、到账延迟、确定性要求 |

### 2.3 不变的信任边界

- 客户端可以预测画面，但不能决定资产事实。
- 服务器权威状态决定碰撞、拾取、伤害、死亡、掉落、胜负和永久结算。
- 每场战斗固定协议版本、模拟版本、内容版本/hash、随机种子和关卡；进行中的战斗不被热更新改规则。
- 重复请求、断线重试、双连接、旧租约和工作进程崩溃都不得产生第二次结算。

## 3. 从本地开发到 Android 验收

### 3.1 Android 工具链

使用 Unity Hub 为精确版本 `2022.3.62f3c1` 增加：Android Build Support、Android SDK & NDK Tools、OpenJDK。Unity 2022.3 LTS 使用与编辑器匹配的 NDK r23b/OpenJDK 11，不混用外部版本。Unity 的 External Tools 必须指向该编辑器随附工具链。

成功方法：构建脚本不硬编码默认 C 盘；`Tools/Resolve-UnityEditor.ps1` 优先读取 Unity Hub 的非系统盘安装根目录，也允许显式 `UNITY_EDITOR_PATH`。`cloudflared` 使用批准的 `E:\Tools\cloudflared\cloudflared.exe`。任何新安装位置变更都先获得项目所有者确认。

### 3.2 隔离验收构建

- 使用独立 product name 和 application identifier，避免覆盖正式安装。
- 仅在构建进程临时注入 acceptance API 和权威战斗开关。
- 构建结束后恢复 development/staging/production 仓库默认开关为 false。
- 输出 Build GUID、文件大小、SHA-256、ABI、API 地址和签名验证。
- 扫描 APK 条目，确保 LoadTest、SimulationBenchmarks、TestResults、Excel 源文件和 secrets 不存在。

### 3.3 本地 development 后端与 Quick Tunnel

首轮没有固定云端时采用：全新隔离数据库 → development API → `cloudflared tunnel --url http://127.0.0.1:5080` → 手机浏览器验证 `/health/ready` → 构建临时 Android APK。

Quick Tunnel 的正确定位是短时、有人看管的 development 验收。它不提供稳定地址或 SLA，每次停止地址失效；HTTP 往返也不适合作为正式实时战斗通道。测试结束按顺序关闭 Tunnel 和测试后端。它在已有 ECS 后仍可作为应急回退或本地复现通道，但不能替代正式固定 HTTPS/WSS。

## 4. 北京 ECS 的搭建与可迁移部署

### 4.1 选型与定位

Integration 基线为阿里云华北 2（北京）、通用算力 u1、4 vCPU、8 GiB、40 GiB ESSD Entry、100 Mbps 峰值、Alibaba Cloud Linux 3 x86_64。它用于开发联调、真机、弱网和压测，不承载正式玩家，也不是唯一数据副本。

### 4.2 成功部署顺序

1. 建实例后只读核对系统、CPU、内存、磁盘、时间同步、监听端口和防火墙。
2. 创建密钥对并验证 SSH；确认第二个会话仍可登录后再收紧配置。
3. SSH 安全组只允许受控出口 `/32`；删除 Linux 不需要的 3389；数据库 5432 不开放公网。
4. 安装 Docker Engine/Compose，启用开机自启和 `json-file` 日志轮转（10 MiB × 3）。
5. 建立 `/opt/tf2d`、`/srv/tf2d/postgres`、`/srv/tf2d/backups`、`/srv/tf2d/logs` 等可迁移目录。
6. 云端无法稳定访问 Docker Hub 时，在本地构建并验证镜像，`docker save` 后经 SSH 传输，云端 `docker load`。不配置来源不明的镜像加速器。
7. 使用独立 Integration 数据库与外部秘密文件启动 PostgreSQL、Migrator、API、Replay Worker。
8. API 只绑定回环地址；Nginx 对公网提供 80/443 并转发 WebSocket Upgrade。
9. 使用受信任证书；临时 IP 证书由 Certbot 固定镜像管理，systemd 每日两次续期，成功后验证并 reload Nginx。
10. 依次验证 live、ready、system-info、登录、云存档、战斗票据、WSS 初始快照、移动确认、防重放、退出结算、备份恢复。
11. 发布使用不可变 release 目录和 `/opt/tf2d/current` 软链接；回滚切回旧 release，不删除数据库。

### 4.3 为什么 ECS 直连优于 Quick Tunnel

真机通过 Quick Tunnel 曾测得约 644 ms RTT，角色和敌人的插值压力明显；北京 ECS 直连 HTTPS 重复样本约 38–64 ms，某次真机报告为 43 ms。VPN 也可能改变本机出口与路由，但 Android 到北京 ECS 的实际延迟必须以设备报告判断，不能把电脑 VPN 状态直接当根因。正确做法是同时记录设备 RTT、快照间隔、丢包/断线、服务端 CPU和网络路径。

### 4.4 可迁移原则

- 客户端使用环境配置中的 HTTPS/WSS 地址，不在 gameplay、save 或 UI 中硬编码 IP、SSH、Docker 内网或厂商域名。
- 云服务器是可替换部署目标。换机器通过 PostgreSQL 逻辑备份、标准容器、外部 secrets、DNS/配置切换完成，不重写战斗规则。
- 释放旧服务器前必须停写、生成最终备份、校验 SHA-256、在全新本地数据库恢复并跑业务验收；旧环境在批准前保留回滚能力。

## 5. 数据库、备份、清理与回迁

### 5.1 已落地策略

- 每日 `pg_dump` 自定义压缩格式，附 SHA-256 和 manifest。
- 每周把最新备份恢复到临时数据库，读取迁移、战斗、资产等关键表。
- 只有最近恢复演练成功后才允许轮换旧备份；至少保留 14 份和 30 天内全部备份。
- Integration 业务历史默认 14 天；清理脚本默认 `--report`，实际删除必须显式 `TF2D_RETENTION_APPLY_CONFIRMED=YES`。
- 保留最终权威快照、最终 Tick 检查点、结算、资产账本、进度、权益、云存档、账号、反作弊证据和未确认 Outbox。

### 5.2 历史数据导致的性能问题

旧 Integration 库累计约 18 万检查点（约 494 MiB）和 22 万输入历史。同版本 15 场曾出现 0/15；全新压测库恢复 15/15，但 P95 仍可能越红线。结论：清理旧高频中间数据可以改善数据库扫描、索引和 I/O 压力，但不能代替模拟调度优化，也不能把“清空库”伪装成容量优化。

### 5.3 清理的安全顺序

1. 生成当日逻辑备份和 SHA-256。
2. 在隔离数据库实际恢复并运行校验。
3. 先执行保留脚本 report，人工核对目标只包括过期、已终局中间数据。
4. 显式执行 apply。
5. `VACUUM ANALYZE` 让空间可重用；需要归还操作系统空间时另开维护窗口评估 `pg_repack` 或 `VACUUM FULL`。
6. 清理后重新压测，记录数据库大小、P95、CPU和错误率，禁止只凭体感宣称改善。

## 6. 服务器权威战斗的具体实现

### 6.1 固定模拟与内容

- 纯 C# 模拟核心同时编译给后端和 Unity，固定 20 Hz、确定性 RNG、稳定实体 ID 和规范状态 hash。
- 当前 schema 14 内容包含 2 关、4 角色、17 种普通敌人、6 个 Boss、80 波、38 技能、5 类掉落和 600 项固定方向表。
- 玩家移动、敌人 AI、投射物、Boss 阶段、技能、掉落、经验、升级、进化、复活、终局和检查点均由权威模拟产生。

### 6.2 网络可靠性

- WSS Upgrade 使用短期一次性 ticket；ticket 防重放。
- 控制租约绑定玩家/战斗/连接；重连换租约，旧租约立即失效。
- 移动是 latest-state 邮箱：只保存最新方向，不把高频输入无限排队。
- 离散命令使用单调 sequence、有界队列和确认游标；重复幂等，倒序、跳号、旧租约、非法值拒绝。
- 服务端 20 Hz 模拟，普通快照 10 Hz；技能候选和终局快照立即发送。
- HTTP 保留用于生命周期、探活、重连和终局查询；WSS 不可用时不回退为客户端权威。

### 6.3 断线、接管、恢复

- 心跳/输入失效后由 `PlayerControlled` 进入 `ServerAutopilot`，仍运行同一权威模拟。
- 玩家在终局和恢复时限前可换取新租约并接管。
- “继续战斗”只调用无副作用 active 查询和 reconnect；找不到、关卡不符或已终局时明确失败，绝不调用 Start 创建新战斗。
- 恢复快照包含 Tick、输入游标、随机状态、实体、技能调度、掉落和待选技能。恢复到待选技能状态时仍显示同一候选。
- 第 14 个迁移持久化 `tick_schedule_origin` 与 `gameplay_paused_at`。选技/复活等待不推进玩法 Tick，Worker 重启后也不补算暂停期间墙钟时间。

### 6.4 终局和资产

- 死亡、胜利、主动退出和超时生成不可变 Terminal Fact。
- 结算、资产账本、关卡进度和 Outbox 在同一事务写入；唯一键保证重复 100 次仍只有一次业务效果。
- 终局已证明后立即显示结果 UI；资产刷新异步，不允许因第二次网络请求失败而把结算界面卡住。
- 主动退出与死亡 UI 语义分离；客户端本地存档余额只是镜像，服务器绝对余额覆盖它。

## 7. 反作弊、运营数据与 Excel 配置

### 7.1 已采用的反作弊核心

1. 服务器拥有所有可产生利益的事实。
2. 每条命令绑定玩家、战斗、租约、连接和序号，并做白名单语义校验。
3. 客户端位置、命中、伤害、敌人死亡、掉落、金币、胜负永不作为事实字段。
4. 重复命令/结算幂等，资产变化写不可变账本。
5. 客户端日志、root/调试、设备完整性均标为风险信号，不能单独永久封禁。
6. 风控优先 observe、限速、限制高价值功能、临时 hold 和人工复核。

### 7.2 客户端投射物方案的结论

“客户端造成伤害时上报，服务器再验证”不能带来数量级降耗。服务器若保持相同反作弊能力，仍要核验技能、冷却、发射 Tick、轨迹、位置、碰撞、穿透、重复命中、伤害和目标状态，还增加乱序、重放和伪造攻击面。可接受的是：客户端先播特效、上报候选命中作为查询提示；服务器仍独立决定是否扣血，并且客户端没上报时也不能漏伤害。

### 7.3 运营/分析边界

- 事务 Outbox 与业务状态同事务提交，消费者按事件 ID 去重。
- 事件至少区分 `server_fact`、`provider_fact`、`client_claim`、`operator_action`。
- 每条战斗、结算和风险事件携带 app/build/protocol/simulation/content 版本及 correlation/trace。
- 运营后台和 BI 只读可重建投影，不能直接修改余额、结算或封禁事实；补偿走带原因、审批和幂等键的管理命令。

### 7.4 Excel 配置链

```text
仓库外 Excel → schema registry 校验/编译 → server-config.json + client-config.json
            → SHA-256 manifest → 测试/审批 → 不可变发布 → 新战斗锁定版本
```

- 当前不预建领域工作表；功能开发需要时才登记。
- 未登记的非空工作表、宏、外链、未知列、重复主键、错误类型全部拒绝。
- 每列显式声明 server/client；服务端私有数值不能进入客户端产物。
- Excel 源文件不复制到 Assets、APK、AAB 或 Git；预检脚本扫描隔离。

## 8. 压测工具、工作负载与容量结论

### 8.1 工具边界与负载修正

负载发生器位于 `Backend/tools/TF2D.LoadTest`，模拟基准/AOI Profiler 位于 `Backend/tools/TF2D.SimulationBenchmarks`。它们在开发电脑运行，不与被测 ECS 抢 CPU，也不在 Unity `Assets` 下，不会进入 APK。

一个虚拟玩家代表一场独立权威战斗和一条 WSS：创建独立身份、登录、开战、选技能、10 Hz 移动输入、接收 10 Hz 普通快照、真实刷怪/技能/投射物/掉落，并执行终局/结算或到时退出。服务端仍为 20 Hz。

Stage19–27 旧文件虽然命名为 `30 × 180 秒`，但战斗约 60–86 秒终局后没有补位，因此只用于诊断，不再作为长稳容量证据。Stage28 起使用测试专用持续补位：终局后创建替补战斗直到统一截止时间；快照超时、5xx 和数据库错误不隐藏重试。

### 8.2 门槛

- 绿：成功率 100%，P95 `< 180 ms`，API CPU 平均 `< 70%`、峰值 `< 85%`，内存 `< 80%`。
- 红：Tick 倒退、租约/快照/数据库错误、5xx、容器重启/OOM，或任一业务/资源门槛失败；立即停止上探。
- 进入 45/50 场的额外门槛：40 场三轮全部绿色，且 API 峰值 `<= 75%`，否则不执行。

### 8.3 Stage33 严格阶梯

| 档位 | 成功 | P95 / P99 | API CPU 平均/峰值 | 判定 |
|---|---:|---:|---:|---|
| 30 × 180 秒 | 30/30 | 150.15 / 158.50 ms | 51.83% / 84.11% | 绿色，达到目标 |
| 35 × 120 秒资格轮 | 35/35 | 150.32 / 159.75 ms | 52.91% / 79.18% | 绿色 |
| 35 重复 r1 | 35/35 | 150.25 / 159.84 ms | 51.73% / 82.04% | 绿色 |
| 35 重复 r2 | 35/35 | 150.32 / 160.41 ms | 52.46% / 78.12% | 绿色 |
| 35 重复 r3 | 35/35 | 149.90 / 160.53 ms | 54.06% / 84.63% | 绿色但余量薄 |
| 40 × 120 秒首轮 | 40/40 | 150.48 / 165.61 ms | 57.73% / **98.86%** | 红色，停止 |

40 场虽然业务成功和 P95 正常，但 API 峰值越线；因此没有执行 40 的后两轮和 45/50。当前相同模型下，30 场是运营规划目标，35 场是余量很薄的实验上限，40 场不认证。不同内容、时长、网络、数据库和机器必须重测。

## 9. 零规则变更优化与 AOI

### 9.1 早期已实施优化

1. 预排序技能定义，移除每 Tick/每敌人重复排序和迭代器分配。
2. 最近敌人用稳定单次线性扫描，距离相同按实体 ID 决胜。
3. 同阶段复用敌人碰撞快照；只有集合变化才重建。
4. 投射物重叠检测移除 LINQ/临时 HashSet，保持稳定命中顺序。
5. 范围/近战/光环/随机/地雷/射线使用复用列表和手动扫描。
6. 依赖实体 ID 单调有序不变量，查找改二分、快照直接复制。
7. 20 Hz 玩法不变，普通快照 10 Hz；技能候选/终局立即发送。
8. exact-base prefix/suffix delta + 协商 gzip；保留旧 JSON 兼容。
9. 复用序列化缓冲、每客户端只保留最新待发快照、减少热路径日志。
10. 当前输入和可靠命令缓存；每 10 Tick 持久化检查点，终局/flush 立即写。

### 9.2 R10 Stage19–33 后续方法

- 用启动恢复 + 内存活动注册表 + 低频 ID 对账，替代每 25 ms 完整数据库扫描。
- 每场战斗常驻确定性模拟器；分离 20 Hz 私有运行态和 10 Hz 展示态。
- 使用稳定逻辑 lane、有界 2 个模拟槽和 2 个检查点写槽；同 battleId 单写。
- 原子合并心跳/控制字段与模拟状态，禁止旧 Tick 覆盖新 Tick。
- 移动热路径只处理必要小字段；驻留状态校验可靠命令；开启有界 SQL auto-prepare。
- 心跳数据库循环与 WSS 快照循环拆分，避免数据库抖动阻塞画面。
- canonical JSON、实时信封和 gzip 使用池化缓冲；delta 选择、前帧/当前帧 checksum 复用保持 wire bytes 等价。
- 方向候选凸包和实时投射物集合减少重复遍历，检查点与终局哈希保持一致。

真实 2,877 个连续 checkpoint 对上，delta 快速选择的编码/payload 与旧实现 0 差异，耗时约 545.3→263.6 ms、分配约 54.30→28.65 MB。高密度 40 场 30 秒窗仍约分配 2.46 GB、GC pause 220.9 ms，说明 presentation 构造、持久化、移动事务与 GC 的叠加仍是 40+ 的限制。

这些方法没有降低 Tick、快照/输入频率，没有减少刷怪/技能/投射物，也没有改变动作、客户端预测、租约、防重放、恢复和结算。更完整的可复用步骤见 `server-performance-concurrency.md`。

### 9.3 AOI 结论

| 敌人数 | 全遍历 | 空间哈希 | 十字链表 | 最优 |
|---:|---:|---:|---:|---|
| 50 | 11.67 ms | 7.06 ms | 4.18 ms | 十字链表 |
| 100 | 18.21 ms | 10.09 ms | 7.03 ms | 十字链表 |
| 250 | 45.40 ms | 19.77 ms | 16.51 ms | 十字链表 |
| 500 | 98.52 ms | 34.88 ms | 37.20 ms | 空间哈希 |
| 1000 | 186.13 ms | 64.33 ms | 89.89 ms | 空间哈希 |

十字链表算法正确且低密度快，但本项目每 Tick 大量移动、生成/死亡频繁；高密度时维护 X/Y 两条有序链成本超过空间哈希。正式实现采用自适应空间哈希：敌人少于 256 保留全遍历，达到阈值后只对服务端投射物碰撞做格子粗筛，再交给原精确几何/伤害逻辑。1000 敌人时空间哈希约为全遍历总耗时的 34.6%，候选检查从 1600 万降至约 34.5 万。

## 10. 故障解决总表

| 症状 | 根因/事实 | 成功方法 | 验收 |
|---|---|---|---|
| 云存档 checksum 冲突 | 本地与云端历史不一致 | 保护本地存档、禁自动上传；隔离新数据库建立 revision 1 | 报告显示 Player session and cloud save ready |
| `persistentDataPath` 主线程异常 | 后台线程调用 Unity API | 主线程先解析绝对路径，Worker 只序列化/文件 I/O，捕获线程异常 | 真机无该异常，保存继续更新 |
| 战斗场景计时/摇杆/刷怪全停 | 权威启动/选技状态与本地 timeScale/场景桥未完成交接 | 明确 Bootstrap → snapshot → initial offer → command applied → resume 状态机 | 弹窗关闭、计时、移动、刷怪、技能全部恢复 |
| 松杆仍沿旧方向 | Android 取消/失焦和旧异步请求可能保留方向 | 所有 PointerUp/EndDrag/Cancel/失焦/后台/禁用/弹窗/销毁统一 `ResetToZero`；latest-state mailbox | 连续变向松杆原地保持 |
| 怪物和角色大距离瞬移 | Quick Tunnel RTT、高快照空洞、客户端直接追权威位置、卡帧放大纠偏 | ECS 直连；预测/插值；单帧纠偏上限；普通快照 10 Hz + 紧急快照 | 真机大幅瞬移消失；快照门槛可监控 |
| 选技弹窗关后瞬移 | 暂停仍用 unscaled time 回收位置；370.8 ms 卡帧放大 | 暂停冻结表现；单帧纠偏 ≤0.25；零输入先发、选技后发；应用游标确认后恢复 | 专用 9/9、完整 119/119；真机大幅问题消失 |
| 死亡不结算 | 终局 UI 等待额外资产请求/重复生命周期 | 终局事实先显示 UI，资产异步；终局幂等且只处理一次 | Terminal flow 自动测试和真机结算 |
| 杀进程后“继续”重开新战斗 | 恢复入口用 Start 探测状态 | active 无副作用查询 + reconnect；恢复失败不创建新局 | 同一 Battle ID/Tick 恢复；恢复失败有稳定错误码 |
| 恢复到技能选择但面板缺失 | 待选候选/暂停状态没有完整恢复到 UI | 检查点持久化 offer 与暂停时钟；恢复快照直接播候选 | 同一战斗在原技能选择状态恢复 |
| Copy report/Export file 假成功 | 旧代码不回读剪贴板或文件 | 紧凑 JSON；写后逐字/长度校验；文件存在和字节数校验；必要时安全分段 | DIAG 显示 copied/saved and verified |
| 松杆/选技后仍小幅位移 | 零输入时 `stationary_reconciliation` 把约 1 单位权威误差表现为 0.12558 单帧移动 | 逐帧取证；玩家控制期零输入 `stationary_hold`，只记录误差；Autopilot 才跟随权威 | 红测复现 → 15/15；r9 真机确认无位移 |
| 选技后总朝右 | 面板/视觉重建把 scale.x 恢复为默认 +1，零输入分支不恢复逻辑朝向 | `ResolveFacingScaleX` 用最后 `LookDirection` 恢复 x，只保留 y/z；权威接管使用位移朝向 | 红测 -1→+1；修复 16/16；r10 待单独真机签收 |
| “180 秒”压测后半段资源很低 | 战斗约 60–86 秒终局后未补位，名义并发不等于持续并发 | 终局持续补位到统一截止时间；每秒记录目标/实际活跃场数 | Stage28 起采用 sustain；Stage19–27 降级为诊断证据 |
| 增加模拟 lane 后尾延迟/峰值恶化 | 数据库、连接、GC 和持久化争用取代了原队头阻塞 | 稳定逻辑 lane + 有界全局槽；同 battleId 单写；失败候选回滚 | Tick 单调；35 三轮绿色；不靠无限线程/连接 |
| 40 场 40/40 且 P95 正常但机器接近打满 | 业务成功与资源安全不是同一个门槛；热窗内编码/持久化/移动叠加 | 独立资源门槛，API 峰值 98.86% 即判红并停止 | 未执行 40 后两轮及 45/50；规划目标保持 30 场 |
| 编码优化候选看似省对象却增加分配 | 值类型敌人快照在真实高密度路径发生更多复制/装箱 | 用基准决定保留/回滚；改用池化 buffer、等价 delta/checksum 快路 | 值类型候选约 +16% 分配后回滚；2,877 对 wire 0 差异 |

## 11. 深度案例：不猜测地解决小幅位移

### 11.1 为什么此前的修复不够

先后做过暂停冻结、卡帧单帧限幅、输入归零、位置死区。这些能消除“大跳”，但玩家仍能看到小幅移动。旧 DIAG 只给复制时的静态值：摇杆为 0、revision 已确认、WSS RTT 正常、服务器位置与渲染位置有差。它无法证明位移发生在哪一帧、由哪个分支产生。

成功转折不是继续调阈值，而是停止猜测，建立同帧因果链。

### 11.2 motion trace 记录内容

- UTC ticks、Unity frame、unscaled time、帧耗时、timeScale 及 owner。
- server tick、快照年龄/间隔、local/sent/accepted input revision、最近移动方向、技能 pending/applied sequence。
- 摇杆原始值、进入预测器的输入、是否 ServerAutopilot。
- 权威位置、帧起点、纠偏候选、预测结果、分支、误差、死区、领先/单帧限幅标记。
- Transform 应用前后、visible delta、预测器 delta、外部 Transform 偏移。
- 事发前 90 帧、事发后 120 帧，总上限 480；未触发时保留最近 90 帧。

自动捕获条件只描述事实：玩家控制期、timeScale > 0、输入为零但 Transform 位移 > 0.0001，或帧起点 Transform 与上帧预测结果不一致。捕获器不预先宣判根因。

### 11.3 真实证据和确定性复现

r8 报告显示：输入为 0，预测分支为 `stationary_reconciliation`；渲染位置约 `(0.263,-2.830)`，权威位置约 `(0.9,-3.6)`，误差接近 1 单位。把该帧数值写进回归测试后，旧代码产生 `0.125583336` 单位单帧移动，与手机报告的 `0.1255836` 相符。至此才能确认：不是摇杆残留、不是外部脚本、不是网络 RTT，而是预测器静止纠偏分支本身。

### 11.4 最终策略

```text
若 presentationPaused：冻结画面位置
否则若有本地输入：执行本地预测，并允许领先上限
否则若 PlayerControlled：stationary_hold，只记录与权威位置误差，不移动 Transform
否则若 ServerAutopilot：限速 authority_reconciliation
碰撞、拾取、伤害、反作弊、终局始终读取服务器权威状态
```

关键点：这不是把客户端变成权威，也没有修改速度/伤害/刷怪。它只规定“玩家已经松手时，表现层不能自己继续移动”。服务器仍继续计算真实状态；重新输入时画面从当前表现位置进行受限预测，接管期则可跟随服务器。

### 11.5 红—绿—回归—真机

1. 红测：用真实轨迹重放，必须得到约 0.12558 的失败位移。
2. 绿测：实现 `stationary_hold`，同一输入必须位移为 0。
3. 回归：ReleaseTrace 15/15；验证本地预测、Autopilot、暂停、单帧限幅、死区不倒退。
4. 构建 r9，记录 GUID/hash/签名并扫描包体。
5. 真机连续变向、松杆、多次选技；用户确认角色能原地保持且不再出现位移。

### 11.6 适用于其他项目的判断表

| 证据 | 结论 |
|---|---|
| 输入非零、revision 随后发送/接受 | 查输入生命周期/邮箱，不改插值 |
| 输入为零，prediction delta = visible delta | 预测/纠偏分支是直接原因 |
| Transform 帧起点偏离上帧预测，prediction 本帧不动 | 有其他 Unity 写入者，查动画/物理/脚本 |
| 暂停恢复首帧很大但 frame limit 生效 | 大 deltaTime 只是诱因，继续查最终 delta 来源 |
| 权威/渲染有误差但 visible delta 为 0 | 只是状态差，不是玩家看见的位移 |

## 12. 深度案例：技能选择后朝右

### 12.1 复现

逻辑最后朝左，视觉节点在弹窗/重建后回到默认 `scale.x=+1`，此时本地输入和 visible delta 均为 0。旧 `ShouldAnimate` 正确返回 false，但没有把最后逻辑朝向重新应用，因此角色静止却朝右。

### 12.2 成功方法

- 先写确定性红测：previousLookDirection.x = -1、currentScaleX = +1、input/delta = 0，预期输出 -1，旧实现实际 +1。
- 增加纯策略 `ResolveFacingScaleX`：玩家控制且零输入时，若最后逻辑 x 有效则恢复其符号；有输入时使用输入朝向；Autopilot 使用权威渲染 delta。
- 只修改视觉节点 x，保留原 y/z scale；只有实际移动才更新 `LookDirection`，避免零向量覆盖最后方向。
- 单测转绿后跑完整 16/16 回归，再构建 r10。

## 13. DIAG 与报告传输的成功设计

### 13.1 报告必须回答的问题

- 我装的是哪个包：app version、Build GUID、Unity、平台/设备/OS。
- 连到哪里：environment、API base URL、backend version、last status/latency/correlation。
- 哪场战斗：mode、battle ID、stage、tick、phase、control lease、entry intent。
- 输入是否真的停：joystick raw、input、sent/accepted revision、zero reason。
- 服务器和画面各在哪：authority/rendered/prediction/Transform、快照年龄、分支。
- 技能/暂停是否完成：offer、pending/applied sequence、timeScale owner、pause reason。
- 终局/验证/结算到哪一步：terminal、settlement、evidence cursor、first divergence。

### 13.2 为什么“按钮提示成功”不够

Android 剪贴板和共享存储可能失败，应用切后台还可能被系统杀死。最终实现：剪贴板写后立刻读回并逐字比较；文件写后检查存在、长度和字节数。常用报告采用 compact JSON 并限制在单次安全负载内；超限才按小于 90,000 字符的连续片段复制，拼接后必须还原原 JSON。

### 13.3 最小报告与取证报告分层

- 默认 compact：构建/环境/战斗摘要、最新错误、有限 breadcrumbs、事件计数；便于一次粘贴。
- incident trace：仅在检测到可见异常时保留前后帧，不持续无限增长。
- full export：详细记录只在文件导出成功并回读验证后使用。
- 永远不记录密码、私钥、令牌、数据库连接串或完整 ticket。

## 14. 可复用的 AI 故障诊断流程

### 14.1 十步闭环

1. 把用户描述改写为可观察断言，不先写根因。
2. 固定 Build GUID、设备、系统、网络、战斗 ID和操作步骤。
3. 只读检查现有代码/测试/日志，列出最多 3 个可区分假设。
4. 为每个假设定义同一帧必须记录的变量和互斥判据。
5. 先增加诊断，不调整阈值或规则。
6. 在真机采集一次可还原数据；若报告传输不可靠，先修报告链。
7. 将真实数值写成确定性红测；红测必须按观察方式失败。
8. 做最小修复，保持服务器权威、规则 hash 和环境隔离。
9. 跑专用绿测、邻近回归、完整门禁，生成独立验收包。
10. 让用户在同一真机执行简短清单；只有用户观察和数据均通过才关闭问题。

### 14.2 常见错误做法

- 只看到“延迟高”就加插值时长；可能把摇杆残留掩盖成更慢漂移。
- 只看到服务器/渲染位置有差就强制 snap；会把正常网络误差变成肉眼瞬移。
- 用更大死区消除一切纠偏；会让 Autopilot/远端表现不再跟权威。
- 在客户端上报命中和奖励以省服务器；破坏反作弊且验证成本仍在。
- 清空旧数据库后宣称代码优化成功；必须用全新库和保留策略分别验证。
- 只跑单元测试就声称 Android 手感通过；触摸取消、后台、帧率和厂商差异只能由真机门禁证明。

## 15. 发布、回滚和验收 SOP

### 15.1 代码门禁

1. 后端架构/玩法测试。
2. Unity EditMode 与 RealtimeBattleReliability。
3. PostgreSQL 迁移、生命周期、备份恢复。
4. N0–N4 弱网/故障注入。
5. Excel self-test、仓库策略、Excel/APK 隔离。
6. 同一输入规范状态 hash 等价。

### 15.2 云端门禁

- 固定 release/tag、镜像 digest、配置/内容 hash。
- migrations 全部成功；API/PostgreSQL/Replay Worker 健康。
- HTTPS 证书链、WSS Upgrade、ticket 防重放、移动确认、重连租约、退出结算。
- 发布后 1 场校准探针；最新 post-deploy 记录 `resume-offer-r4` 为 1/1、P95 152.7 ms。
- 异常时切换 `/opt/tf2d/current` 到已知稳定 release；数据库不删除、不重建。

### 15.3 Android 门禁

- 核对文件真实存在、Build GUID、SHA-256、v1/v2 签名、ARM64、独立包名。
- 依次验证：初始技能、计时、摇杆、刷怪/自动技能、连续选技、死亡结算、主动退出、第二/三场隔离。
- 断网 5 秒、恢复、Wi-Fi/蜂窝切换、杀进程继续、待选技能恢复。
- 连续 10 分钟观察瞬移、GC/卡顿、发热、内存和诊断异常。

## 16. 当前构建与证据索引

### 16.1 最新移动修复构建

| 构建 | 用途 | Build GUID | SHA-256 | 自动化/真机 |
|---|---|---|---|---|
| r9 `TF2D-stationary-hold-20260721-r9.apk` | 玩家控制零输入原地保持 | 见对应 manifest/构建记录 | `CB78438E...E19EFFA` | ReleaseTrace 15/15；真机确认无位移 |
| r10 `TF2D-r10.apk`（manifest 原名 facing-preservation） | 保留最后逻辑朝向 | `5741b7b9efda426ca765e8081c3fc406` | `47B782B6...155C590` | Facing 16/16；用户确认不再位移，朝向待独立签收 |

完整 SHA-256：r9 `CB78438E364FC2965F68B441E5382A4CAFDFACB93D1C91CBEA3578BEEE19EFFA`；r10 `47B782B682BE5A2A30EBFE0055D7CA33C058FFBDF5686D2F9391DC91A155C590`。

### 16.2 核心源码入口

- `AuthoritativePlayerPrediction.cs`：预测、暂停、stationary_hold、Autopilot 纠偏、逐帧 trace。
- `PlayerBehavior.cs`：把输入/权威模式传入预测器，应用位置、动画和朝向。
- `AuthoritativeMovementMailbox.cs`：latest-state、零方向顺序屏障、modal presentation policy。
- `AuthoritativeBattleSessionService.cs`：Start/Resume 分离、WSS ticket/timeout、租约和命令。
- `DiagnosticHub.cs`、`DiagnosticOverlay.cs`：报告、motion trace、复制/导出校验。
- `BattleSimulationRunner.cs`、`AuthoritativeBattleSimulation.cs`：20 Hz 权威模拟和暂停。
- `PostgresBattleSessionStore.cs`、迁移 0014：持久暂停时钟和检查点。
- `EnemySnapshotSpatialHash.cs`、`ServerProjectileSimulator.cs`：AOI 粗筛与精确碰撞。

### 16.3 重要仓库文档

- `Docs/Runbooks/AuthoritativeBattleMigrationResume.md`
- `Docs/Architecture/RealtimeBattleReliabilityPlan.md`
- `Docs/Testing/AuthoritativePlayerMotionTrace-2026-07-21.md`
- `Docs/Testing/DurablePauseAndAoiEvaluation-2026-07-21.md`
- `Docs/Testing/EcsRealtimeZeroRulePass1Benchmark-2026-07-20.md`
- `Docs/Testing/EcsRealtimeBreakpointBenchmark-2026-07-21.md`
- `Docs/Runbooks/AliyunIntegrationDeployment-2026-07-20.md`
- `Docs/Runbooks/PostgresBackupAndRetention.md`
- `Docs/Security/AntiCheatArchitectureResearch.md`
- `Docs/Runbooks/ExcelConfigurationPipeline.md`

## 17. 未完成风险与下一步

| 风险/事项 | 当前状态 | 下一步门禁 |
|---|---|---|
| r10 朝向真机签收 | 自动 16/16，用户未明确单独描述朝向 | 左朝向进入/退出技能弹窗 3 次并录屏/DIAG |
| WSS 终局关闭握手 | 部分报告出现 remote closed without close handshake | 把正常终局与异常关闭分类，补协议关闭码测试 |
| 生产域名/证书 | 当前 Integration 临时 IP HTTPS | 自有域名、DNS、正式证书、迁移/回滚演练 |
| 单机 40+ 场 | Stage33 的 40 场首轮业务成功但 API 峰值 98.86%，45/50 未执行 | 先优化 presentation 分配、持久化和移动事务峰值，再从 30 场长稳重新走门槛；不得外推 100 场 |
| 运营后台 | 事件/Outbox 边界已定义 | 风险时间线、结算/账本、案件、RBAC、审计 |
| Android 兼容矩阵 | 华为 API 29 为主要真机 | 小米/OPPO/vivo/三星/Google，多 API/刷新率/图形 API |
| iOS | 后置 | 安装 iOS Build Support、签名账号、App Attest ADR与真机 |
| production 开关 | 默认关闭 | 完整自动门禁 + 真机签收 + 所有者明确批准 |

## 18. 可直接交给其他 AI 的执行提示词

```text
你接手的是一个 Unity 在线手游。先读取仓库规则、ADR、当前续接记录和本手册；
不要依赖聊天记忆。先执行只读盘点并保护脏工作树。

目标：复现并解决“松开摇杆或技能弹窗关闭后，角色仍有小幅位移/错误朝向”。
约束：客户端不是权威；不能修改伤害、速度、Tick、刷怪、掉落或结算规则；
不能把调试工具、压测工具、Excel 源文件或 secrets 放进 APK；默认环境开关必须恢复关闭。

执行：
1. 固定 Build GUID、设备、网络、Battle ID和复现步骤。
2. 同帧记录摇杆原始值、实际输入、sent/accepted revision、server tick、快照年龄、
   权威位置、预测器分支/候选/结果、Transform 前后、timeScale owner、modal sequence。
3. 只按可观测条件捕获前后帧，不预设根因。
4. 将真实轨迹写成确定性红测；证明旧代码产生与真机相同数量级位移。
5. 玩家控制期零输入时画面必须 stationary_hold；只有 ServerAutopilot 可无输入跟随权威位置。
6. 零输入时用最后逻辑 LookDirection 恢复视觉朝向，保留 y/z scale。
7. 跑专用绿测、完整回归、弱网、数据库和预检；验证规范状态 hash 不变。
8. 构建独立 ARM64 acceptance APK，核对 SHA-256/签名/包体隔离并恢复默认开关。
9. 让项目所有者在指定真机连续变向松杆和完成三次技能选择；用用户观察和 DIAG 共同签收。
```

## 19. 最终验收清单

- [ ] 当前工作树和实际构建产物已核对，没有引用不存在的 APK。
- [ ] development/staging/production 默认权威开关关闭。
- [ ] 玩家控制期零输入 visible delta 为 0；服务器权威误差只记录。
- [ ] ServerAutopilot 仍能受限速跟随权威位置。
- [ ] 左/右最后逻辑朝向经过技能弹窗后保持。
- [ ] 初始技能、计时、摇杆、刷怪、自动技能、升级、Boss、死亡/退出/结算正常。
- [ ] 继续战斗恢复同一 Battle ID/Tick；待选技能恢复同一候选。
- [ ] Copy/Export 只有回读验证成功才提示成功，报告无 secrets。
- [ ] API/PostgreSQL/Replay Worker/HTTPS/WSS 健康，ticket/租约/序号/防重放通过。
- [ ] 备份已实际恢复；清理只在恢复成功后执行。
- [ ] 压测工具、基准、TestResults、Excel 源文件不在 APK。
- [ ] 性能容量结论引用同一版本、同一数据库状态和明确工作负载。
- [ ] 长稳压测在战斗终局后持续补位；业务成功和 CPU/GC/数据库/重启/OOM 分别判门槛。
- [ ] 性能优化没有放宽服务器权威、租约、序号、防重放、检查点恢复和幂等结算。
- [ ] 真机签收至少覆盖华为 Android 10/API 29；production 仍需多机型矩阵和所有者批准。

## 20. 核心经验总结

最成功的技术选择不是某一个阈值，而是把“事实”和“表现”分开：服务器决定规则与资产，客户端负责立即反馈；诊断系统把输入、权威状态、预测决策和 Transform 放在同一帧；测试把真机数据原样变成红测；发布系统把测试环境和正式环境隔离；数据库把备份恢复放在删除之前；压测把工具放在客户端包之外并用明确战斗标准测量。

本次小幅位移最终能关闭，关键是没有继续猜测“可能是网络、摇杆或插值”，而是记录到 `stationary_reconciliation` 在零输入帧产生了与真机相同的 0.12558 单位位移，再以 `stationary_hold` 只改变表现策略、不改变服务器权威规则。这个“先取证、再红测、最小修复、完整回归、真机签收”的闭环，才是其他项目最值得复制的成功方法。
