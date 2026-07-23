# Unity Project Bootstrap Knowledge

这是一个给 AI 和团队成员复用的 Unity 项目启动知识库。内容来自 TF_2D 的真实执行与真机验证：Git 基线、Unity 目录重构、美术资源管理，以及手机网游的客户端/服务器架构、确定性战斗、反作弊、结算、运营数据和云迁移。

## 知识地图

```text
unity-project-bootstrap-knowledge/
  AGENTS.md                              # AI 总入口与不可违反的规则
  docs/
    index.md                             # 分类导航和执行顺序
    new-unity-project-runbook.md         # Unity Git 基线与目录重构
    online-game-architecture.md          # 客户端/服务器总体架构与模式选择
    unity-editor-connected-development.md # Editor 本地资源/平台适配 + 仓库外后端
    unity-client-networking.md           # Unity 输入、表现、恢复、Android 与 DIAG
    online-game-incident-response.md      # 在线故障同帧取证、确定性复现与修复闭环
    game-server-authority.md             # 权威服务器、协议、反作弊、账本与性能
    verified-local-battle.md             # 本地即时模拟 + 服务器重放验证
    data-operations-and-cloud.md          # Excel、运营数据、部署与云迁移
    cloud-game-server-deployment-runbook.md # ECS/Docker/HTTPS/WSS 实战部署与排障
    server-performance-concurrency.md     # 服务端降耗、容量、过载保护，以及 TF_2D 零规则变更实战
    svn-art-repository-runbook.md        # 本地 SVN 美术大资源库
    art-and-lfs-policy.md                 # 美术资源与 Git LFS 策略
    tf2d-case-study.md                    # TF_2D 真实案例复盘
    tf2d-engineering-handbook.md          # TF_2D 完整工程、云端、压测与故障实证手册
  checklists/
    preflight.md                         # Git 基线执行前检查
    verification.md                      # Git 基线执行后验证
    online-game-acceptance.md            # 真机、弱网、结算与安全验收
    svn-art-repository.md                # SVN 美术库检查清单
  prompts/
    new-project-bootstrap.md             # Git 基线/重构提示词
    build-unity-online-game-foundation.md # 手机网游基础架构提示词
    diagnose-unity-online-game-incident.md # 在线游戏故障诊断提示词
    setup-local-svn-art-repository.md    # SVN 美术库提示词
  templates/                             # 可复制的 Git/Unity 模板
```

## AI 先读什么

| 场景 | 先读 |
| --- | --- |
| 新 Unity 项目要进入 GitHub | `AGENTS.md` + `docs/new-unity-project-runbook.md` |
| Git/UGit 推送失败但 GitHub API 仍可用 | `docs/environment-setup.md`：先分离诊断，再使用受控发布回退 |
| 要设计手机网游客户端/服务器 | `docs/online-game-architecture.md` |
| 只有客户端工程，要在 Editor 连接已部署后端运行 | `docs/unity-editor-connected-development.md` |
| 单人 PvE 操作被高延迟拖慢 | `docs/verified-local-battle.md` |
| 要做实时战斗、结算和反作弊 | `docs/game-server-authority.md` |
| 要处理摇杆、重连、Android 或 DIAG | `docs/unity-client-networking.md` |
| 在线问题反复修补仍无法定位根因 | `docs/online-game-incident-response.md` |
| 要接 Excel、运营后台或迁移云服务器 | `docs/data-operations-and-cloud.md` |
| 要实际搭建云游戏服务器、HTTPS/WSS 或排障 | `docs/cloud-game-server-deployment-runbook.md` |
| 要降低服务端消耗、提高并发或做容量规划 | `docs/server-performance-concurrency.md`：通用方法 + TF_2D Stage33 实战 |
| 要查看 TF_2D 完整实证、压测数据和修复证据 | `docs/tf2d-engineering-handbook.md` |
| 要建立本地 SVN 美术资源库 | `docs/svn-art-repository-runbook.md` |
| 要直接给 AI 一段可执行提示词 | `prompts/` |

## 核心原则

- 先提交 Unity 原始 Git 基线，再做目录重构。
- 保留 `.meta` GUID，不提交 Unity 生成目录和构建产物。
- 客户端负责输入与即时表现，永久奖励由服务器模拟或重放校验后结算。
- 普通单人 PvE 可使用 Verified Local 消除 RTT 手感影响；PVP/多人/高风险玩法采用服务器实时权威。
- 固定 Tick、确定性 RNG、版本化内容、幂等终局和不可变资产账本是战斗可信的基础。
- 移动输入使用“最新状态邮箱”，不积压每个摇杆采样点；生命周期结束必须归零。
- 生产实时通道使用固定域名和持久 WSS；Quick Tunnel 只用于临时验收。
- Excel 源文件在项目外，经严格校验生成客户端/服务器不同产物，并用内容哈希固定战斗规则。
- 运营统计通过事务 Outbox 消费权威事实，分析系统不能直接修改资产。
- 所有结论以自动化测试、Release 基准、Android 真机和弱网证据为准。
- 运动、恢复和弹窗问题要在同一帧记录输入、权威、预测、Transform 与 UI/暂停状态；先重现真实数量级，再做最小修复。
- 性能优化先建立真实负载基线；优先修复热点与无界资源，再做缓存、异步和扩容，最后才考虑内核、NUMA、分片等高风险深调。
- 实时战斗优先做“零规则变更”降耗：少查数据库、常驻模拟状态、复用编码缓冲、减少重复校验和计算；不得用降 Tick、降输入/快照频率或减少实体伪造容量。
- 长稳压测必须在战斗终局后持续补位，并把业务成功率与 CPU/GC/数据库等资源门槛分别判定；任一红线都停止上探。

## 快速入口

让 AI 建立手机网游基础架构：

```text
请读取 AGENTS.md 和 prompts/build-unity-online-game-foundation.md，先审计现状并制定分阶段计划，再按 checklists/online-game-acceptance.md 验收。
```

让 AI 执行新项目 Git 基线：

```text
请读取 AGENTS.md，然后按 docs/new-unity-project-runbook.md 执行。
```

让 AI 建立只有客户端工程的 Editor 联网开发模式：

```text
请读取 AGENTS.md 和 docs/unity-editor-connected-development.md，先审计启动、平台、资源、环境和后端依赖，再按文档分阶段实施并验证。
```

让 AI 建立本地 SVN 美术库：

```text
请按 docs/svn-art-repository-runbook.md 建立本地 SVN 美术大资源库。不要预设业务目录结构，目录由我以后定义。
```

让 AI 优化服务器容量：

```text
请读取 AGENTS.md 和 docs/server-performance-concurrency.md。先固定玩家体验、反作弊和持久化不变量，再建立终局持续补位的容量基线并定位前三个瓶颈。优先实施零规则变更的 P0/P1 改造；每次只改一个变量，给出风险、正确性验证、资源门槛和回滚，任一门槛变红就停止上探。
```

让 AI 诊断在线游戏故障：

```text
请读取 AGENTS.md、docs/online-game-incident-response.md 和 prompts/diagnose-unity-online-game-incident.md。先固定构建/设备/网络/战斗身份并建立同帧证据，再写确定性失败测试和最小修复；不要先猜根因或调阈值。
```
