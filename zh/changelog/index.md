---
pageClass: changelog-page
---

# 版本变化

主项目 [Landscape Router](https://github.com/ThisSeanZhang/landscape) 的版本更新.
配套管理工具 landscape-kit 的变更见 [lkit 版本变化](./lkit).

内容依据 GitHub Releases 与相邻 tag 之间的提交整理而成. 每个版本给一句摘要, 详细条目收在
折叠区里, 展开即可查看; 可能影响现有配置、部署方式或使用习惯的改动, 会单独用
**需要注意的变更** 提示标出.

版本标题上带 <Badge type="warning" text="预发布" /> 标注的, 是 GitHub 上勾了 pre-release 的版本.
版本号里带 `-beta` 的不一定是(比如 v0.6.8-beta 就按正式版发布), 以 GitHub 的标记为准.

页面只保留**近半年**发布的版本(2026-04-11 之后); 更早的版本移到 [历史版本](./archive).

## 获取与升级

- 下载地址: [Releases](https://github.com/ThisSeanZhang/landscape/releases),
  需要 `landscape-webserver` 可执行文件和 `static.zip` 前端静态资源.
- 版本号形如 `0.x.y`, 2025 年中之前的版本带 `-beta` 后缀.
- 升级只需要替换二进制和静态资源, 配置目录保持不变, 旧版本的数据会在程序启动时自动迁移.
  完整步骤见 [手工部署](../intro/manual-deployment).
- 导出的配置文件只能用于**相同**版本的恢复. 跨版本时需要先在导出时的版本完成恢复,
  再启动新版本. 见 [配置文件介绍](../configuration/).
- 需要回退时可以用 `landscape-webserver db rollback` 交互式降级数据库,
  见 [系统配置导出](../advanced/settings-export).

## v0.26.2 (2026-10-10)

DNS 的实验性连接池补齐了 DoT / DoH / DoQ 上游, 并在界面上可以开关; Geo 的站点匹配器改为 SQLite 存储,
按内容哈希同步.

::: details 详细变化
- DNS 实验性连接池支持 DNS-over-TLS、DNS-over-HTTPS 和 DNS-over-QUIC 上游.
- 上游配置页面新增实验性连接池的开关.
- 原生上游连接池挪到 `pool-native` 编译特性之后.
- DNS 规则查找的超时提到 6 秒, 给上游留出余量.
- Geo 的站点匹配器改由 SQLite 存储, 并按内容哈希同步; 不再被引用的条目会回收,
  同步失败时沿用上一次可用的数据.
- 站点规则的匹配语义在 SQL 查找里保持一致: 用 REGEXP 与 LIKE 转义, 并对非法模式做保护.
- 站点规则的语义由共享的 golden 用例统一约束匹配器和 SQL 两侧.
- `geo_cache` 拆成 db / ip / site 三个独立模块, 域名匹配器与站点匹配器注册表从 DNS 模块移进来.
- 关机的各个阶段做了加固, 不再因为 panic 中断; netlink 的地址错误改为优雅处理.
- 内部整理: 移除不再使用的 paste 依赖并升级工作区依赖; CI 的 apt 依赖去掉 llvm/curl/zstd;
  IAPrefixMap 的运行状态下移到 landscape-core 的 pd_prefix 模块;
  config CLI 的 e2e 断言对齐 `wan_links` 的 init 配置格式.
:::

## v0.26.0 (2026-10-09) <Badge type="warning" text="预发布" />

WAN 服务从“按接口”改为“按链路”统一管理, 新增路由表页面; 静态 NAT 与 IPv6 PD 跟着适配.

::: warning 需要注意的变更
按接口划分的 WAN 服务接口已下线, `landscape_init.toml` 中原有的按接口 WAN 服务数组也不再被接受,
请改用统一的 WAN 链路配置.
:::

::: details 详细变化
- 新增 `wan_links` 表, 把散落在各个接口上的 WAN 服务(IPv4、pppd、PPPoE 以及接口连接段)收拢到一处.
- 配套引入 `WanLinkService`, 一条链路就是一个整体, 不再按接口切分.
- 前端的 WAN 配置改用这条统一链路, 原来的按接口配置随之下线.
- 新增 `/wan-links` 系列接口.
- 每个连接段都有自己的链路状态面板; IPv4 会话挂到哪个段由 v4 租约决定, 失败会退避重试.
- 每条链路自动分配一个不可改的 `link_chain_id`, 并预留了健康检查的字段和配置段落.
- 新增路由状态接口和路由表页面, 路由不用再靠命令行查.
- 静态 NAT 和 WAN 链路的增删改支持乐观锁; 端口校验改走 `wan_links`, 而且会双向检查重叠.
- IPv6 PD 的前缀状态和事件改为按链路记录.
- WAN 路由的归属改看 `RouteOwner`(链路 uuid 或 netns 名), 不再看接口名.
- DHCPv6 PD 套接字创建失败时返回错误, 不再 panic.
- DHCPv6 的 Release 报文补上 IA_PD, 并支持 Server Unicast 选项.
- DHCPv4 修正续租的 ACK 处理、REBIND 里的 ciaddr, 并清理租约计时器.
- 修正 PPP 链路的活动判定: 只看 `v4.enable`, 单有 PD 不会把会话拉起来.
- 配置交叉校验时允许顺手修正内容, 让修正结果真正写进库.
- 内部整理: 接口层引入视图模型, ddns / flow / lan_ipv6 / 静态 NAT 的列表查询支持过滤与分页.
:::

## v0.25.3 (2026-10-06)

修正 DHCPv6 快速认证的算法字段, Reconfigure 密钥只在 Reply 报文中下发.

::: details 详细变化
- 修正 DHCPv6 快速认证的算法字段; Reconfigure 的密钥只在 Reply 报文里下发.
:::

## v0.25.2 (2026-10-03) <Badge type="warning" text="预发布" />

设备信息改为由统一目录提供, 公共代码往 landscape-core 收拢, DDNS 和停机流程更稳.

::: details 详细变化
- 设备信息的来源从“主机名注册表”换成统一的“设备目录”: 设备本身、DHCP、SLAAC、ARP 的发现结果
  合并成一份视图, 地址冲突由它统一裁定.
- 设备目录会广播设备事件, DDNS、NAT6 等都改成监听事件; 设备 id 以登记信息为准.
- 前端的设备页改由这份目录提供数据, 各服务原先各自的 DHCPv4/IPv6 状态接口不再使用.
- 设备条目上多了两项记录: ARP 出现轨迹和 DHCPv4 租约时钟.
- IPv6 的 link-local 地址会区分来源, 设备页的地址标签和滚动体验也做了优化.
- 路由服务、TLS SNI 解析、证书 PEM 处理、Geo 缓存引擎等公共部分抽到 landscape-core.
- 分流 WAN 目标的同步改成事件驱动; `tc_lan_dao` 拆成独立的过滤器, 由 LAN IPv6 服务挂载.
- DDNS 的后台循环支持优雅退出, 访问服务商的 HTTP 请求加了超时.
- 停机时先保住关键 IP, 避免关服务的过程中失联.
- 补齐 0.21.5 到 0.25.1 的回滚边界, 回滚时才能选到当前版本可用的目标.
:::

## v0.25.1 (2026-10-01)

配置写入统一到 ConfigStore 并改成“先验证服务再落盘”; 新增按子系统的内存自监控.

::: details 详细变化
- 所有配置模块统一走 `ConfigStore`, 写入带事务语义, 校验放在写入口, 不再有新旧两套实现并存.
- 配置写入改成“先确认服务能起来, 再落盘”, 避免配置写成功而服务起不来; 分流、静态 NAT、
  IPv6 PD 等都改走这条路径.
- 服务状态从 watch 通道换成结构化并发的 `ServiceHandle`, 单例服务有独立的运行状态,
  停止和状态上报都是确定的.
- 新增内存自监控: 按子系统统计堆内存, 任务派生带标签, 内存指标按分钟落盘, 前端有内存分析页.
- 内存跟踪默认不开, 需要 `mem-track` 编译特性; 开启后还能看到 RSS 的组成拆解.
- 这套能力后来统一改名 self-monitor, 接口、crate 和前端一起改.
- Geo 源文件落盘到 `geo/raw`, 支持离线恢复和下载原始文件; 写入改成先写临时文件, 校验后原子改名.
- `config` 子命令可以生成 `landscape_init.toml`, 也能设置管理员凭据、WAN-only 模式
  和静态 NAT 相关开关.
- 拓扑由是否存在 `--wan-iface` / `--lan-iface` 决定, 并校验静态 NAT 端口.
- 修掉一个隐蔽问题: LAN 的 ARP 扫描没有重新读接口, 可能拿到过期的 MAC.
- 内部整理: 清掉废弃的 store 实现, 文件存储改名 geo_cache; 配置、错误和 DTO 类型按域拆分.
:::

## v0.25.0 (2026-09-28)

新增 Geo 反查、资源命名、HTTP/2 与响应压缩; 管理员凭据比较和登录限流做了加固.

::: details 详细变化
- Geo 新增反查接口和界面, 刷新失败会返回错误.
- 可配置的资源可以起名字了.
- 后端启用 HTTP/2 ALPN 和响应压缩.
- 界面主题设置会保存下来.
- 接口的运行时地址可以通过接口读到, netlink 的处理也做了加固.
- 新增构建能力上报, 前端据此决定显示哪些功能.
- 新增 `config` 子命令用来生成 `landscape_init.toml`.
- 安全方面: 管理员凭据改用常量时间比较, 登录加限流, 日志里不再输出管理员密码.
- 上游 DNS 新增实验性连接池开关, 默认关闭.
- 拓扑图改用 dagre 布局, 节点高度用实测值.
- 待生效的 PPPoE 配置会在拓扑里显示成卡片, 而且必须先禁用才能删除.
- pppd 服务重写成异步 supervisor, 并补了单元测试.
- 指标 ringbuf 增加 1 秒兜底消费, 修掉偶发丢失 epoll 唤醒后卡住的问题.
- NTP 同步服务用异步客户端重写, 抽成可测试的 trait.
- 修正 OpenAPI 里 `MacAddr` 和端口范围的类型映射.
- 清掉 DuckDB 的残留代码.
:::

## v0.24.3 (2026-09-10)

eBPF map 改为 Rust 侧 pinning 并按域拆模块; 移除旧 PPPoE 客户端, 补上端到端冒烟测试.

::: details 详细变化
- GeoIP 支持按来源单独刷新.
- 指标的 SQLite 迁移逐个事务执行; 启动时连不上数据库会重试.
- eBPF 的 map 改为 Rust 侧 pinning, 初始化按域拆模块, 相关类型也重组成按域的目录树.
- DNS 服务器与 eBPF 解耦, 中间用 trait 衔接.
- eBPF map 的键值类型换成 zerocopy 镜像类型.
- 移除旧的 PPPoE 客户端, 统一走 `wan_service`; 顺带修了状态时序和 eBPF 初始化/退出问题.
- eBPF 状态不再挂在进程全局, 改为显式的 `EbpfRuntime`;
  PPPoE 的端到端客户端各自使用独立的 map 空间.
- map 打不开或 pin 目录不可用时只报错, 不再 panic.
- 路由的 BPF 头文件按版本拆分, route4/route6 命名统一, 测试也按域拆开.
- 新增需要 root 权限的端到端冒烟测试; 跨平台构建换成 cargo-zigbuild, CI 任务合并.
- 日志文件名改为 `landscape.YYYY-MM-DD.log`.
- 开启 loongarch64 的网关支持.
:::

## v0.24.2 (2026-08-31)

指标后端从 DuckDB 换成 SQLite, DNS 指标按分钟预聚合; 新增 IPv6 邻居学习.

::: warning 需要注意的变更
指标后端从 DuckDB 换成 SQLite. 仅 DuckDB 可用的指标设置被移除, 指标配置的单位统一为 MB 与秒,
容量、保留期等参数的可选范围也有调整, 升级后建议重新确认一遍指标配置.
:::

::: details 详细变化
- 指标后端改用 SQLite 并成为默认; DuckDB 后端下线, 原来的 duckdb 模式会转到新存储.
- 指标模块按“聚合”与“落盘”分层, sink 和 worker 都改成 trait.
- 连接指标支持分页和总数, 不传 limit 可以查全量历史.
- DNS 指标按 1 分钟预聚合, 直接给仪表盘用; 连接指标改成追加写; 桶的保留期可配置.
- 新增容量上限, 写满时主动清理, 存储报 `SQLITE_FULL` 时会热回收.
- 指标配置的单位统一成 MB 和秒, 并对取值范围做校验.
- 改指标配置时整批事务生效, 失败回滚.
- 指标服务的生命周期加固: 切换配置加锁, 读端有界关闭, 锁中毒能恢复, worker 退出前先落盘.
- ringbuf 的读取改成可取消的异步循环; `ConnectMetric` 改成与 eBPF 事件零拷贝兼容的结构.
- 冷桶写入改成批量预聚合; 打开冷存储前先在放宽的内存限制下重放残留的 WAL.
- 连接与 DNS 历史的保留期、`conn_summaries` 的行数上限开始强制生效.
- 新增 `tc_lan_dao` 程序, 从 LAN 入向的 DAD 邻居请求里学习 IPv6 邻居;
  缓存未命中时也会从入向包学习 IP 与 MAC 的对应关系.
- 修正邻居学习写错 MAC 的问题: 原来记的是目的 MAC, 应该是源 MAC.
- 静态 NATv6 可以跟踪没有固定 IPv6 后缀的设备.
- PPPoE 的 LCP echo 间隔可以配置.
- 修掉 SQLite 写锁泄漏, 遇到 `SQLITE_BUSY` 会重试.
- 内部整理: 测试去掉多余的全局锁, 固定 sleep 换成有界轮询, XDP veth 测试放到独立 netns 里跑,
  新增两 netns 的 TC 链集成测试.
- 定位对端接口时跳过 `/sys/class/net` 里的非目录项.
:::

## v0.23.1 (2026-08-20) <Badge type="warning" text="预发布" />

新增带事务的 ConfigStore, 错误类型按数据库、启动、服务分层.

::: details 详细变化
- 新增 `ConfigStore`, 写入带事务, 并能把本次改动的内容返回出来.
- 错误类型分层: 数据库、启动、服务启动各自一类, 接口层的错误可以带上语义和脱敏信息透传.
- 接口错误类型收敛成 Geo、防火墙、DNS 三类.
- 完善 DNS 规则的持久化和接口的错误处理.
- 通过 NA(不带 TLLA)声明的 SLAAC 地址也会校验.
- DNS 域名统一按 ASCII 规范化.
- 已登记设备的选项里纳入 DHCPv6 接口.
:::

## v0.23.0 (2026-08-16)

DNS 服务器重写为引擎加匹配器的结构, 匹配不再分配内存; 重定向支持元数据透传.

::: warning 需要注意的变更
DNS 的源地址绑定从规则移到上游配置. 另外不再允许把 DNS 规则与目的 IP 规则跨 flow 移动.
:::

::: details 详细变化
- DNS 服务器按“引擎 + 匹配器 + 规则组件”重写, 拆成缓存、解析、重定向、快照几条独立的链.
- 域名匹配从 trie-rs 换成 zerotrie 加 RegexSet, 匹配过程不再产生额外内存分配.
- 本地应答抽成 `LocalResolver`, 所有 DNS 流程共用一份.
- 统一了 DNS 的查询路径和规则查找/缓存写入路径, 反查刷新的语义也顺手修正.
- 补了匹配器测试, 覆盖关键字、正则、非 ASCII 跳过和边界穿透.
- 解析器新增 TCP fallback, 上游重试的时序收紧.
- 上游返回的 DNS 错误码会透传给客户端, 响应码映射集中到一处.
- 重定向规则新增“元数据查询透传”选项, 前端有对应开关.
- 上游的绑定地址池按 (dns mark, upstream) 维护, 源地址绑定从规则挪到上游配置里.
- 腾讯 DNS 的同步会复用已有记录, 记录值最多两条.
- DNS 服务上报的状态更准确, 启动失败时优雅降级.
- 禁止把 DNS 规则和目的 IP 规则跨 flow 移动, 避免流运行时读到过期数据.
- Geo: 只有无效的 GeoKey 来源会被跳过, 没有有效域名的规则会被丢弃;
  应用 Geo 值时会用 SHA256 判断是否真的变了.
- SLAAC 存活探测改成两阶段, 通过主动发 NA 请求来刷新.
- 新增架构图, 截图挪到 assets 目录.
:::

## v0.22.3 (2026-08-04)

DHCPv4 统一到 dhcproto; LAN IPv6 的前缀规划与 RA 细节修正.

::: details 详细变化
- DHCPv4 的报文处理统一到 dhcproto, 相关 helper 收进 net_proto.
- 收到匹配的邻居请求时, 会刷新已学习的 SLAAC 地址活跃度.
- LAN IPv6 里静态父前缀的长度限制为 /56–/63.
- SLAAC+DHCPv6 模式下允许在 PD 前缀组上发 RA.
- 为 WAN 保留 /64 的 0 号槽位.
- LAN IPv6 的前缀冲突校验会返回带类型的错误.
- 分流追踪抽屉按协议族批量处理判决, 并加了过期请求保护.
- OpenAPI 规范随 Release 一起发布.
- WAN 的 IPv4 路由地址变化时, 会重启 NAT 服务.
:::

## v0.22.2 (2026-07-31)

代码按业务域重新分区; 静态 NAT 补上双向端口冲突检测, 分流规则支持命名.

::: warning 需要注意的变更
RA 的生存期改由全局 `ad_interval` 推导, IPv6 地址的生存期统一到全局 `preferred_lifetime`,
原来按前缀组单独配置的生存期不再生效.
:::

::: details 详细变化
- 代码按域重新分区, 分成配置、LAN、WAN、系统几个模块, 公共运行时代码收进 `landscape-core`.
  这是结构调整, 对外配置没有变化.
- 静态 NAT 和动态 NAT 之间会双向检查端口冲突, 并新增实时检查接口, 前端做防抖校验.
- 修正 UDP 校验和为 0 的编码: 表示“不校验”的 0 保持原样, 计算得出的 0 按 RFC 768 编成 0xFFFF.
- ICMP 错误报文里内层的 ICMP echo 也会做 NAT, 头校验和同步更新;
  DNAT 和扫描器能处理带最小 TCP 引用的 ICMP 错误.
- 修正动态映射生成的环绕问题, 定时器解析逻辑重写并补了测试.
- 连接跟踪的释放改成串行; egress 解析器不再处理处于清理阶段的连接项.
- 定时器值新增 `is_static` 标记, 便于识别静态映射.
- IPv6 静态映射共用 WAN 的 IID; 用 IID 第 63 位划分 WAN 和 LAN 的地址分配空间;
  静态 NATv6 可以跟踪动态 IPv6 地址.
- IPv6 PD 新增前缀规划器: 支持分组规划、记录期望前缀长度、过期后重新选组,
  规划单元按组配色.
- RA 和 IPv6 地址的生存期统一到全局字段, 不再按前缀组单独配置.
- 分流规则新增独立的 `name` 字段.
- DHCP 会把 LAN 后缀作为 option 15 / 119 下发; LAN 主机名的配置新增开关、校验和界面,
  旧的配置键会自动迁移.
- DNS 的 PTR 反查优先返回已登记设备的主机名; LAN 后缀改成整段匹配;
  原先写死的 `home.arpa` 换成通用的后缀匹配, 同时保住 `.local` 的 mDNS 语义.
- 主机名相关的逻辑从 DNS 服务器里抽出来, 和 DHCP 主机名跟踪合并, 后续改名 `lan_hostname`
  (保留 serde 别名, 旧配置仍可用).
- 二层广播/多播的 MAC 会做检查; 广播和多播 IP 的判断统一到一组 helper.
- 新增邻居请求(NS)支持, 能从中取出 MAC; link-local 的记录方式也重做了.
- tproxy 模式下放行 ICMP, 是否放行改由命令行或环境变量控制.
- 配置服务从文件级加锁改成按段落哈希.
- 界面: 新增 About 页面展示 GitHub 贡献者; i18n 按域拆成多个模块;
  图表库从 ApexCharts 换成 Apache ECharts; 字体统一到 CSS 变量; 抽出了时长格式化组件;
  侧边栏层级和路由路径重新划分.
- DHCPv6 只在客户端支持 ReconfAccept 时才带上 Authentication 选项.
- 修正 WAN 接口的一个问题: IPv6CP 协商出 link-local 地址后没有设置到接口上.
- 网关的上游 TLS 现在可以跳过证书校验, 方便内部自签名环境.
- PPPoE 发现可以按 AC-Name 过滤.
:::

## v0.21.4 (2026-07-02)

修正 LAN IPv6 路由键冲突; PPPoE 客户端模块化并补测试; PD 路由安装修正.

::: details 详细变化
- 修正 LAN IPv6 路由的键冲突: 改为存真实的子网索引, 去掉 WAN 偏移.
- 服务器启动时, 会为静态 NA 预留的地址补上 IPv6 分配记录.
- DNS 记录的 TTL 可以配置, 腾讯 DNS 的处理也做了加固.
- 静态 NAT 映射服务拆成 IPv4 和 IPv6 两个.
- 新增 PPPoE 客户端模块, 按两阶段设计重构, 错误也做了分类.
- 修正 PPPoE 发现和 LCP 协商里以太网帧的解析偏移.
- 补上 PPPoE 协商层的测试, 修正运行时的 echo-request 逻辑.
- PPPoE 的会话生命周期里接入 eBPF TC/XDP 和路由设置.
- LCP 里跟踪对端的 MRU 和 magic, 处理 PAP 拒绝, 日志也更清楚.
- 启动和前缀变化时会立刻发一次 RA.
- 修正 IPv6 PD: 请求和续期时都会安装委派前缀路由.
- XDP 下允许广播和链路本地包进入 WAN 接口.
:::

## v0.21.3 (2026-07-01)

事件中心落地, DHCPv6 服务器重写并支持 Reconfigure; DDNS 可按 PD 解析设备 IPv6.

::: warning 需要注意的变更
静态 NAT6 的配置字段改名: `mapping_pair_ports` 变为 `port_config`, `device_id` 变为 `device_ids`.
设备管理页不再自动刷新 IPv6 后缀, 需要手动刷新.
:::

::: details 详细变化
- 新增事件中心 `EventHub`, 统一分发接口事件、设备事件、IPv4/IPv6 地址分配与过期事件,
  并推给前端.
- DHCPv4 的地址分配状态独立出来, 设备绑定改由设备事件驱动更新.
- DHCPv6 的分配状态也独立成结构, 用异步安全的互斥保护, 租约分配和前缀同步重做.
- 按接口的 watch 通道换成集中的 IA 前缀事件总线, 每个消费方拿到自己那份数据.
- DHCPv6 服务器重写: NA / PD / SLAAC 的租约状态统一管理, 补上 ICMPv6 RA/NA 处理、
  SLAAC 跟踪、周期性租约清理和 PD 路由管理.
- 支持 DHCPv6 Reconfigure(RFC 8415 §20.4).
- DUID-UUID 客户端会回退到用 link-local 地址查 MAC, 缺失的 MAC 也能从 SLAAC 的 NA 冲突里补上.
- DHCPv6 在 Confirm 里校验后缀归属; 静态绑定每次都刷新.
- DDNS 可以通过 PD 委派解析已登记设备的 IPv6, 支持多记录同步和 GUA/ULA 过滤,
  前端可以选择 LAN 设备作为来源.
- 新增 MAC 与 link-local 地址的映射缓存, 周期性扫描邻居;
  IPv6 分配事件从单个地址改成按 MAC 分组, 并新增刷新事件.
- LAN 主机名可以参与 DNS 解析: DHCP 上报的主机名会映射到 IPv4.
- 静态 NAT6 的端口配置改成枚举(全部端口或指定端口), 目标支持多个设备并做重复校验;
  新增端口通配回退; 静态 NAT 映射在前端和接口层都拆成 IPv4/IPv6 两套.
- 设备管理页新增 IPv6 后缀随机生成和手动刷新, 并停止了自动刷新.
- Geo: site 配置支持强制刷新; 新增 AdGuard 规则转换的说明面板, 解析时保守处理并支持否定规则.
- 日志时间戳带上本机时区, 新增 `--log-filter` 关键字过滤.
- eBPF 的 ARP/NDP 同步改用 netlink, 不再读 `/proc/net/arp`, 并支持 IPv6 邻居.
- 修掉 LAN IPv6 服务缺少中间状态转换、以及前缀变化时消息发送端被丢弃导致的死循环.
- DHCPv4 分配时优先使用静态绑定, NAK 响应也用静态 IP.
- 升级依赖: hickory-dns 0.25 → 0.26; pnpm 配置迁移到 workspace.
- 内部整理: 线程和异步任务命名; 模块继续按域迁移; 补齐 0.20.1 与 0.21.0 的回滚边界.
:::

## v0.20.1 (2026-06-20)

转发管线大面积搬进 XDP 并与 TC 统一; 修掉 7 处 use-after-free.

::: warning 需要注意的变更
原生 XDP 从默认开启改为需要显式传入 `--try-xdp`, 未指定时会走 TC.
LAN 静态路由的 `is_next_hop` 字段换成了 `route_type`.
:::

::: details 详细变化
- XDP 入口分发落地, 支持 PPPoE 剥离和 tail call, 并搭好 XDP 链的基础设施和三级链测试.
- 新增 XDP 的 WAN 路由和 LAN 路由, 带缓存、分流判决和 FIB 回退, 并补了双向集成测试.
- 双向 XDP 链在 LAN→WAN 和 WAN→LAN 两条路径上都做 MSS clamp.
- 新增 XDP 扫描器做 IPv4/IPv6 分类; MSS clamp 改用扫描器并支持 IPv6.
- 新增 XDP 防火墙, 集成测试覆盖 IPv4/IPv6 的拦截、方向和放行场景.
- 新增 XDP 的 NAT 阶段, 支持 IPv4/IPv6 的静态和动态映射.
- 修正 XDP 校验和: 改用 `csum_fold` 管线, 不再直接给 `bpf_csum_diff` 赋值.
- 修正 IPv4 分片偏移的字节序问题.
- 修掉 `SkbXdpBundle` 等 7 处 use-after-free, 原因是结构体字段顺序和 Rust 的析构顺序对不上.
- 修正 `lan_map` 未命中时 FIB 回退会劫持未知 IP 的问题.
- XDP 和 TC 统一: 不再区分路由模式, 两者始终一起工作, 中间靠交接衔接;
  路由缓存新增“能否 XDP 重定向”标记, 非 XDP 接口回退到 TC.
- 打通 XDP 到 TC 的 Docker 流量交接, 并带上 VLAN 标记.
- 引入 TC 链架构, 把防火墙、MSS clamp、NAT、PPPoE 封装都做成链上的阶段.
- 路由模式可以通过命令行在 TC 和 XDP 之间选择.
- 移除 LAN 入向的 TC 链, LAN 流量改为通过重定向走 WAN 出向链.
- 修正 WAN 路由类型在 TC 链里被错误跳过、以及非本地 WAN 流量返回状态码不对的问题.
- `is_next_hop` 换成 `route_type`, 并支持 WAN 可达路由.
- MAC 查不到时回退到 `bpf_redirect_neigh`, 未匹配的默认流会被丢弃.
- 移除旧的 v1/v2 路由管线, 以及旧的防火墙和 MSS clamp 模块.
- PPPoE 支持 XDP, 新增封装阶段; 认证支持 CHAP MD5; PPPoE 从 TC 链里解耦成独立的挂载.
- PPPoE 拨号后写入静态的 IPv6 link-local 邻居项.
- 新增 ethtool 模块, 接口拉起时关闭 GRO.
- 修正 musl 下 ethtool 调用的类型转换问题.
- DHCPv6 PD 改为每个客户端分一个池块, 响应里用该块自己的前缀长度.
- 收到邻居通告时, 把 IPv6 的 IP-MAC 绑定写进 eBPF map.
- 新增 mDNS 本地主机名应答, 支持 DNR 广告; 并阻止本地 DNS 泄漏到上游解析器.
- 反查 DNS 的预检处理加固.
- Docker: 统一错误类型, 清掉事件处理里的死代码; 只在与 Docker 相关的页面轮询,
  避免 Docker 不可用时阻塞定时刷新.
- 停机时保留 DHCPv4 服务器地址和 WAN 静态地址, 保证还能连上设备.
- BPF 程序的许可证从 Dual BSD/GPL 改为 GPL.
- 内部整理: 测试程序移到独立目录, release 构建不再编译它们.
:::

## v0.19.2 (2026-05-07)

WAN TC 阶段重构为共享管线并新增 LAN 管线; PPPoE 换成原生实现, 分流支持权重.

::: warning 需要注意的变更
`DHCPv4ServerConfig` 中已废弃的 `mac_binding_records` 字段被移除, MAC 与 IP 的绑定请改用已登记设备.
:::

::: details 详细变化
- WAN 的 TC 阶段重构成共享的 tail-call 管线, 另加一条独立的 LAN TC 管线.
- 分流目标支持权重, 按槽位分配流量, 最多 16 个目标.
- 已解析的 WAN 目标信息会缓存在 LAN 缓存里, 后续包不用再查槽位表.
- 新增接口的连接指标.
- 修正缓存过的 Docker 路由 VLAN 标记不对的问题.
- 修正 IPv4 和 IPv6 的 WAN 路由路径不一致, 以及若干指针和类型上的问题.
- 路由和 tproxy 的包解析重写, L3 与 L4 的扫描分开.
- 修正 MSS clamp 的边界处理.
- PPPoE 换成原生实现, 补上配置流程和集成测试, 移除旧的 dump 解析器.
- DHCPv4 修正 discover 报文的长度字段, 最后两次重试改走单播;
  ACK 按 RFC 2131 设置服务器地址并回显客户端地址.
- 新增可配置的 DHCP 选项和选项编辑器; 修正 NAK 里自定义选项的注入问题.
- 从 `DHCPv4ServerConfig` 里移除废弃的 `mac_binding_records`.
- 新增修改密码功能, 认证配置支持热加载.
- 新增自定义 DoH 的 HTTP/2 监听.
- 新增带版本号的 init 配置导入流程.
- 新增 AdGuard geosite 源.
- 新增统一的 WAN 候选接口, 供 DDNS 选择 PPP 接口.
- NTP 同步重试改为指数退避.
- 修正 PPPoE 启动时的终止流程, 以及 LAN 管线的丢弃顺序.
- DHCP 租约确认后会把 IP-MAC 对应关系预热进 eBPF map; Docker 注册更稳.
- 配置开关挪到弹窗标题旁边.
:::

## v0.18.3 (2026-04-16)

隐私字段打码; 修正设备绑定流程; PPPoE 校验与 PPPD 生命周期更稳.

::: details 详细变化
- 证书和网关视图里的敏感字段做了打码.
- 修正已登记设备的绑定流程.
- DHCP 租约状态与绑定提示同步.
- PPPoE 的接口校验和更新流程加固, PPPD 的生命周期恢复更稳.
:::

## v0.18.2 (2026-04-12)

新增 DNS 服务商档案与 DDNS 任务; LAN IPv6 配置统一到前缀组; 拓扑交互打磨.

::: details 详细变化
- 新增 DNS 服务商配置档案和 DDNS 任务, 证书和 DDNS 复用同一份档案.
- DDNS 更可靠, 可以控制记录 TTL, 来源输入有了校验提示.
- 路由追踪能看到 DNS 过滤和缓存检查的结果; DNS 诊断支持运行时开关缓存.
- LAN IPv6 的配置统一到前缀组, 总览更清楚, 未启用的分组会明确标出,
  并会自动生成默认的静态前缀.
- 为 WAN 的 IPv6 地址保留 PD 的 0 号子网.
- 网关: 代理的错误响应更清楚, 监听双栈地址, 尾斜杠跳转规范化.
- 修正 IPv6 路由缓存的处理, 路由缓存和路由服务的结构也重新整理.
- 修正邻居 IP/MAC 缓存更新不及时的问题.
- DHCPv4 的 LAN 路由只对已启用的服务下发; DHCP 的区域清理移进 DHCP 服务内部.
- 不再强制给 PPP 接口设置 `accept_ra=2`.
- 拓扑图: 选中节点会聚焦; 面板交互、布局、快捷键和服务异常状态都做了打磨.
- 修正 DHCPv4 状态表里 IP 排序不对的问题.
:::

---

主项目的完整变更可以直接查看
[landscape Releases](https://github.com/ThisSeanZhang/landscape/releases).
