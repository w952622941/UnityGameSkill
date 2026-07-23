# Unity 网络游戏云服务器部署实战手册

本文给 AI 和项目负责人一条可复用、可验证、可迁移的实施路线：把本地 Docker 后端部署到一台 Linux 云服务器，为 Unity Android 客户端提供 HTTPS/WSS，并保留迁回本地或更换云厂商的能力。

它来自一次真实的 Unity 手机网游 Integration 部署与华为 Android 真机验收。文中的地址、实例 ID、密钥和密码全部使用占位符；任何项目使用前都必须重新发现自己的实际值。

## 1. 先定义边界

这套方案适合 development、acceptance、integration 或 staging。正式 production 仍应使用项目所有者控制的稳定域名、独立凭据、备份、监控和发布审批。

推荐拓扑：

```text
Unity Android
  -> HTTPS/WSS :443
  -> Nginx/Caddy TLS 网关
  -> 127.0.0.1:<API_PORT>
  -> Docker Compose API
  -> Docker 内网 PostgreSQL
```

不可破坏的约束：

- 公网通常只开放 SSH、80 和 443；API 原始端口只绑定回环，PostgreSQL 不映射公网端口。
- 客户端正式配置不硬编码 VM、容器或云厂商。临时 IP 证书只用于验收，生产切换稳定域名。
- 数据、密钥、发布文件和日志使用明确的持久目录；不能只留在可删除的容器层。
- 云服务器是可替换部署目标，不进入战斗、资产、存档或反作弊领域模型。
- Quick Tunnel 保留为临时回退，不能当生产实时战斗通道。

## 2. 准备信息与权限

在任何写操作前，由 AI 记录但不提交以下信息：

| 项目 | 示例占位符 | 安全要求 |
| --- | --- | --- |
| 云厂商/地域 | `<CLOUD>/<REGION>` | 选择接近主要测试设备的地域 |
| Linux | `<DISTRO>/<VERSION>` | 选择仍受支持的 64 位发行版 |
| 公网地址 | `<PUBLIC_IP>` | 不写入公共文档和业务代码 |
| SSH 用户/端口 | `<SSH_USER>:<SSH_PORT>` | 优先密钥登录 |
| 本地私钥 | `<LOCAL_KEY_PATH>` | 不读取、打印或上传私钥内容 |
| 实例规格 | `<VCPU>/<RAM>/<DISK>/<BANDWIDTH>` | 压测后再决定生产规格 |
| 环境 | `<integration>` | 与 production 数据和凭据隔离 |

### 2.1 试用 ECS 的起步规格

真实 Integration/压力测试选择了 4 vCPU、8 GiB、40 GiB ESSD Entry、峰值 100 Mbps（按流量）的个人免费试用实例。它的意义是提供足够余量同时测 API、PostgreSQL、WSS 和回放 Worker，不代表生产最终规格。

选择原则：

- 优先 Linux 64 位；Ubuntu LTS 或云厂商仍受支持的长期维护发行版均可。
- 2C2G 可做最小功能冒烟，但数据库、容器构建和回放并行时余量太小。
- 4C8G 适合作为第一台集成与分档压测机；依据 CPU、内存、Tick、回放和数据库 P95/P99 再降配或升配。
- 40 GiB 足够早期代码、镜像、测试数据库和日志，但必须监控镜像层、备份与日志增长。
- 峰值带宽不是稳定吞吐保证；按流量计费要设置预算和流量告警。
- 地域优先靠近主要玩家/真机，不能因为开发者正在使用 VPN 就选择 VPN 出口附近地域。

免费试用结束前要提前完成备份下载和本地恢复演练。免费额度、流量和到期释放规则会变化，购买前以云厂商当日页面为准。

首次 SSH 连接必须先核对主机指纹并使用独立 known-hosts 文件。第一轮只做只读盘点：

```bash
uname -a
cat /etc/os-release
nproc
free -h
df -h
timedatectl status
docker version
docker compose version
ss -lntp
```

检查安全组和主机防火墙。不要因为连不上就开放“全部 TCP”或数据库端口。

本地 VPN 可能改变浏览器、curl 和手机代理后的路由，但不会改变 ECS 自身的公网 IPv4。测试时分别记录 VPN 开/关；手机未经过电脑 VPN 时，不要把电脑测得的 RTT 当作手机 RTT。

## 3. 推荐目录

目录可以调整，但必须在部署记录中固定：

```text
/opt/<game>/releases/<release-id>/    # 不可变发布文件
/etc/<game>/<environment>.env         # 环境变量与秘密，权限 0600
/etc/<game>/gateway/                  # TLS 网关配置
/etc/<game>/tls/                      # ACME 账户与证书
/srv/<game>/data/postgresql/          # 数据库持久数据
/srv/<game>/backups/                  # 逻辑备份
/srv/<game>/logs/                     # 应用和网关日志
/opt/<game>/tools/                    # 续期、备份与运维脚本
```

发布目录按编号新增，不覆盖旧发布。环境秘密与发布文件分离，以便相同镜像部署到不同环境。

## 4. Docker/Compose 部署顺序

### 4.1 固定发布身份

每次发布记录：

- Git 提交和未提交差异摘要
- API、迁移器和数据库镜像标签/摘要
- Compose 文件摘要
- 数据库迁移版本
- 协议、模拟器和内容版本
- 回滚到哪个发布目录

不要使用无法追踪的 `latest` 作为应用发布身份。基础镜像也应固定版本。

### 4.2 Docker Hub 不通时的可靠路径

真实部署中，云主机访问 Docker Hub 经常超时，但本地开发机可以正常拉取和构建。已验证的替代路径：

1. 在本地基于当前源码构建 Release 镜像。
2. 运行本地自动化测试和容器健康检查。
3. `docker save` 导出应用、迁移器和固定数据库镜像。
4. 计算 SHA-256。
5. 通过 SSH/SCP 上传到新发布目录。
6. 云端再次核对 SHA-256，再执行 `docker load`。
7. 禁止临时改用未知第三方镜像源绕过失败。

```powershell
docker save <api-image> <migrator-image> <postgres-image> -o <IMAGE_TAR>
Get-FileHash <IMAGE_TAR> -Algorithm SHA256
scp -i <LOCAL_KEY_PATH> <IMAGE_TAR> <SSH_USER>@<PUBLIC_IP>:/tmp/
```

### 4.3 数据库先行

1. 创建独立环境数据库和凭据。
2. 启动 PostgreSQL，等待健康。
3. 运行版本化迁移器；迁移失败则停止，不手工改生产表“修好”。
4. 启动 API。
5. API 只绑定 `127.0.0.1:<API_PORT>`。

Compose 必须包含：健康检查、`restart: unless-stopped`、持久卷、资源边界、`no-new-privileges`，并通过外部环境变量注入密码。

### 4.4 功能开关必须匹配客户端

真实故障：Android 验收包同时启用了 Authoritative Realtime 和 Verified Local，但 Integration Compose 把 Verified Local 固定为 `false`。HTTPS 健康检查仍返回 200，客户端却无法开局并得到 `503 verified_battles_unavailable`。

预防方法：

- 不只检查 Compose 文件，要检查运行容器实际收到的环境变量。
- 为每种客户端验收模式执行一次真实“创建战斗”请求。
- 重新创建容器后再次检查镜像标签，防止 Compose 默认标签引入发布漂移。

```bash
docker inspect <api-container> --format '{{range .Config.Env}}{{println .}}{{end}}' \
  | grep -E 'AUTHORITATIVE|VERIFIED|REPLAY'
docker inspect <api-container> --format '{{.Config.Image}} {{.Image}}'
```

## 5. 从 Quick Tunnel 迁移到直连 HTTPS/WSS

Quick Tunnel 很适合第一次真机验收，因为无需域名和证书配置。但随机域名、额外边缘跳转和每次 HTTP 往返会带来高延迟，不适合实时战斗。

已验证的渐进顺序：

1. 保留 Quick Tunnel 作为回退。
2. 云端 API 继续只绑定回环地址。
3. 安装 Nginx/Caddy 到系统标准位置，配置文件和日志放入约定目录。
4. 安全组只新增 TCP 80/443，来源为公网时使用 `0.0.0.0/0`；不要开放 API/数据库端口。
5. 先用 ACME staging 验证 80 端口和挑战流程。
6. 再申请生产证书。
7. 配置 443 TLS 反向代理与 WebSocket Upgrade。
8. 从外网验证证书链、HTTPS 和 WSS。
9. 生成独立验收 APK；真机通过前不关闭 Quick Tunnel。

### 5.1 没有域名时的临时 IP 证书

Let’s Encrypt 已支持短期 IP 地址证书。它适合临时 Integration/压力测试，但有效期只有约 6 天，必须自动续期。Certbot 需使用支持 `--ip-address` 的当前版本。

先使用 staging：

```bash
certbot certonly --staging --standalone \
  --preferred-profile shortlived \
  --ip-address <PUBLIC_IP>
```

staging 成功后去掉 `--staging` 申请可信证书。证书路径通常为：

```text
/etc/letsencrypt/live/<PUBLIC_IP>/fullchain.pem
/etc/letsencrypt/live/<PUBLIC_IP>/privkey.pem
```

如果 CA 报 `Timeout during connect`，优先检查安全组 80，而不是反复申请证书。真实案例正是安全组未开放 80；开放后 staging 和 production 均一次通过。

### 5.2 Nginx 的关键配置

以下为结构示例，路径和上游端口必须替换：

```nginx
map $http_upgrade $connection_upgrade {
    default upgrade;
    '' close;
}

server {
    listen 443 ssl;
    server_name <PUBLIC_IP_OR_DOMAIN>;

    ssl_certificate     <FULLCHAIN_PATH>;
    ssl_certificate_key <PRIVATE_KEY_PATH>;
    ssl_protocols TLSv1.2 TLSv1.3;

    location / {
        proxy_pass http://127.0.0.1:<API_PORT>;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
        proxy_read_timeout 3600s;
        proxy_send_timeout 3600s;
        proxy_buffering off;
    }
}
```

在公网网关增加连接数、请求速率和请求体大小限制，但要根据登录、存档、证据上传与 WSS 特性分路由调优，不能用一个过严阈值误伤合法客户端。

### 5.3 短期证书自动续期

推荐 systemd timer 每日执行两次，并增加随机延迟。续期脚本顺序：

1. 运行 `certbot renew --no-random-sleep-on-renew`。
2. `nginx -t` 校验配置。
3. `systemctl reload nginx`。

Timer 已经负责随机化时，不要让 Certbot 再随机休眠，否则人工演练可能看似“卡死”。必须执行一次 `renew --dry-run --no-random-sleep-on-renew`，并确认模拟续期成功。

Certbot 在容器内运行时不能直接执行宿主机 `systemctl`。正确边界是：容器完成续证并退出，宿主机脚本再校验和重载 Nginx。

## 6. 分层验收：健康 200 不等于游戏可用

按以下顺序执行，每层失败就停止并定位：

1. **进程层**：数据库/API/网关运行，重启策略正确。
2. **端口层**：公网只有预期端口，API 回环绑定，数据库内网可见。
3. **TLS 层**：证书链可信、SAN 覆盖目标地址、有效期和续期正常。
4. **HTTP 层**：`/health/live`、`/health/ready`、系统版本。
5. **玩家层**：游客登录、云存档读写、资产读取。
6. **实时战斗层**：创建战斗、一次性 WSS 票据、初始快照、移动确认、票据重放拒绝。
7. **Verified Local 层**：开局、证据分批上传、完整回放、丢失/乱序/篡改拒绝、唯一结算。
8. **持久层**：数据库中存在唯一终局、账本和 Outbox；客户端 UI 成功不算持久成功。
9. **真机层**：Build GUID、实际 API、RTT、Tick、输入、死亡/胜利、结算和重启后资产一致。

真实验收结果显示：直连后手机 API 请求约 43–101ms，而此前 Quick Tunnel 可达约 400–600ms。这个结果只证明该设备、网络和地域的本次路径，不可替代多地域和弱网测试。

## 7. 故障表

| 症状 | 根因 | 已验证解决办法 |
| --- | --- | --- |
| Docker Hub 拉取超时 | 云端到仓库网络不稳定 | 本地构建/验证，`docker save` + SHA-256 + SCP + `docker load` |
| ACME `Timeout during connect` | 安全组未开放 80 | 只新增 TCP 80，先 staging，再 production |
| 证书演练长时间无输出 | Certbot renew 随机休眠 | Timer 负责抖动；演练和脚本加 `--no-random-sleep-on-renew` |
| 容器 deploy-hook 找不到 `systemctl` | 容器与宿主机职责混淆 | 容器续证，宿主机校验并 reload Nginx |
| HTTPS 200，但战斗无法创建 | 服务功能开关与客户端模式不一致 | 检查容器实际 Env；修 Compose；重建 API；真实创建战斗 |
| Compose 重建后镜像标签变化 | 未传发布标签，使用默认标签并触发构建 | 显式设置 release tag，`--no-build` 重建，再核对 image ID |
| 客户端仍走旧通道 | 旧 APK 内含临时验收地址 | 生成独立验收 APK，核对 Build GUID 和 DIAG 的实际入口 |
| 延迟仍高 | VPN、地域、Tunnel 边缘或每 Tick HTTP | 分别测直连/VPN 开关；实时使用持久 WSS；记录 P50/P95/P99 |

## 8. 安全与反作弊

- TLS 只保护传输，不能让客户端变可信。
- 实时模式接受输入意图，服务端决定伤害、掉落、死亡和结算。
- Verified Local 接受证据，服务器用相同版本、种子和规则完整回放后结算。
- 使用一次性票据、租约轮换、单调序号、防重放、幂等键、消息尺寸和速率上限。
- 数据库结算、资产账本和 Outbox 在同一事务中提交。
- DIAG 可以包含 Build、Tick、延迟、拒绝码和关联 ID，但不能包含令牌、密码或私钥。

## 9. 压测前后做什么

功能验收通过后再压测。使用 Release 镜像，分档增加并发并记录：

- API/WSS 连接数、请求率、错误率、重连率
- Tick P50/P95/P99、回放/结算 P50/P95/P99
- CPU、内存、磁盘 IO、网络、PostgreSQL 连接/锁/慢查询
- 单战斗快照字节、证据量、队列长度和最长战斗时间

停止条件必须预先定义，例如错误率、P99、CPU、内存、数据库锁等待或队列超过阈值。不要用“服务器没崩”作为容量结论。

如果一场战斗会在计时窗口内终局，负载工具必须补充新战斗直到统一截止时间；否则名义上的长稳测试会提前空载。详细的零规则变更优化、业务/资源双门槛和 TF_2D Stage33 案例见 `server-performance-concurrency.md`。

## 10. 备份、回滚与迁回本地

1. 使用 `pg_dump` 生成带时间戳的逻辑备份。
2. 计算 SHA-256，并下载到 Git 之外的本地备份目录。
3. 在隔离数据库实际恢复，核对迁移数量和关键记录。
4. 保留旧发布目录、Compose、镜像摘要和环境变量名清单。
5. 新版本异常时切回旧镜像/配置；涉及写入时按明确的数据一致性方案回滚。
6. 云服务器到期前停止新写入，做最终备份并恢复到本地 Docker PostgreSQL。
7. 本地依次验证健康、登录、存档、资产、战斗、回放、结算、重连和幂等。
8. 项目负责人明确批准后，才释放云资源。

本地已有内容不需要删除。迁移是“备份、恢复、验证、切换”，不是复制容器数据目录，更不是先删除旧环境。

## 11. AI 完成交付时必须报告

- 改了哪些本地与云端文件，哪些只存在于服务器
- 当前发布标签、镜像摘要、迁移数量和服务状态
- TLS 类型、到期时间、续期 Timer 和 dry-run 结果
- HTTPS/WSS、两种战斗模式、结算和数据库持久化证据
- 延迟样本的设备、网络、地域和统计口径
- Quick Tunnel/旧环境是否仍可回退
- 未完成的域名、监控、备份、压测和生产门槛
- 用户下一步只需执行的真机验收动作

只有这些证据齐全，才能说“Integration 部署成功”；它仍不等于 production 已上线。

## 12. 官方参考

- [Let’s Encrypt：Certbot 申请短期/IP 地址证书](https://letsencrypt.org/2026/03/11/shorter-certs-certbot.html)
- [Let’s Encrypt：短期和 IP 地址证书正式可用](https://letsencrypt.org/2026/01/15/6day-and-ip-general-availability.html)
- [Nginx：WebSocket 反向代理](https://nginx.org/en/docs/http/websocket.html)
- [Docker：镜像 save/load 命令索引](https://docs.docker.com/reference/cli/docker/image/)
- [Docker：容器环境变量与 env-file](https://docs.docker.com/reference/cli/docker/container/run)
