---
pageClass: changelog-page
---

# Version History

Release notes for the main project,
[Landscape Router](https://github.com/ThisSeanZhang/landscape). For the companion tool, see
[lkit Version History](./lkit).

Everything here is compiled from GitHub Releases and the commits between consecutive tags.
Every release has a short summary, with the detailed entries folded away behind a collapsible
block. Changes that may affect existing configuration, deployment, or day-to-day usage are
called out in a **Breaking changes** callout.

Headings marked <Badge type="warning" text="pre-release" /> are the ones GitHub flags as a
pre-release. A `-beta` suffix does not imply one (v0.6.8-beta shipped as a normal release), so the
GitHub flag is what counts.

Only releases from the **last six months** (after 2026-04-11) are listed here; older ones are in
[Older Releases](./archive).

## Getting and Upgrading

- Downloads: [Releases](https://github.com/ThisSeanZhang/landscape/releases). You need the
  `landscape-webserver` executable and the `static.zip` web assets.
- Versions look like `0.x.y`; releases before mid-2025 carry a `-beta` suffix.
- Upgrading means replacing the binary and the static files. The configuration directory
  stays where it is, and older data is migrated automatically when the program starts.
  See [Manual Deployment](../intro/manual-deployment) for the full steps.
- An exported configuration file can only be restored on the **same** version. To move
  across versions, restore it on the version that exported it first, then start the new
  one. See [Configuration File Guide](../configuration/).
- To go back, `landscape-webserver db rollback` downgrades the database interactively. See
  [System Configuration Export](../advanced/settings-export).

## v0.26.2 (2026-10-10)

The experimental DNS connection pool gains DoT, DoH, and DoQ upstreams with a toggle in the UI,
and geo site matchers move to SQLite storage reconciled by content hash.

::: details Detailed changes
- The experimental DNS connection pool supports DNS-over-TLS, DNS-over-HTTPS, and DNS-over-QUIC
  upstreams.
- The upstream configuration page gained a toggle for the experimental connection pool.
- The native upstream connection pool moved behind the `pool-native` build feature.
- The DNS rule lookup timeout is raised to 6 seconds to keep margin over the upstream budget.
- Geo site matchers are stored in SQLite and reconciled by content hash, unreferenced entries are
  garbage collected, and the last known good data is kept when a sync fails.
- Site rule match semantics are preserved in the SQL lookup with REGEXP and LIKE escaping, plus a
  guard against invalid patterns.
- Site rule semantics are pinned by shared golden cases that cover both the matcher and the SQL
  side.
- `geo_cache` splits into standalone db, ip, and site modules, with the domain matcher and site
  matcher registry moving in from the DNS crate.
- Shutdown phases are hardened against panics, and netlink address errors are handled gracefully.
- Housekeeping: the unused paste dependency is gone and workspace dependencies were bumped;
  llvm/curl/zstd dropped from the CI apt lists; IAPrefixMap runtime state moved into
  landscape-core's pd_prefix module; and config CLI e2e assertions were aligned with the
  `wan_links` init config format.
:::

## v0.26.0 (2026-10-09) <Badge type="warning" text="pre-release" />

WAN services move from per-interface to per-link management, with a new route tables page; static NAT and IPv6 PD follow suit.

::: warning Breaking changes
Per-interface WAN service endpoints are retired, and the per-interface WAN service arrays in
`landscape_init.toml` are no longer accepted. Use the unified WAN link configuration instead.
:::

::: details Detailed changes
- A new `wan_links` table gathers the WAN services that used to be scattered across
  interfaces: IPv4, pppd, PPPoE, and the section legs on each interface.
- `WanLinkService` treats a whole link as one unit instead of slicing it up by interface.
- The web UI now works against that unified link, so the old per-interface endpoints go away.
- A new `/wan-links` REST surface.
- Every section gets its own link status board. Which section the IPv4 session attaches to
  is decided by the v4 lease, and failures retry with backoff.
- Each link is given an auto-assigned, immutable `link_chain_id`, and fields for a future
  health check are reserved.
- A routing status API and a route tables page, so you no longer need the shell to inspect
  routes.
- Static NAT and WAN link updates support optimistic locking. Port validation now runs
  through `wan_links` and catches overlaps in both directions.
- IPv6 PD prefix state and events are keyed by link.
- WAN route ownership is keyed by `RouteOwner` (link UUID or netns name) rather than the
  interface name.
- A failed DHCPv6 PD socket now returns an error instead of panicking.
- The DHCPv6 Release message carries IA_PD, and the Server Unicast option is honored.
- DHCPv4 fixes renew and rebind ACK handling, the rebind ciaddr, and lease timer cleanup.
- PPP link activity is decided by `v4.enable` alone, so a delegated prefix cannot bring the
  session up by itself.
- Cross-validation of configuration may now fix values in place, so the fix actually lands
  in the database.
- Housekeeping: API view models, plus filtering and pagination for ddns, flow, lan_ipv6,
  and static NAT list queries.
:::

## v0.25.3 (2026-10-06)

Fixed the DHCPv6 fast-auth algorithm field; the reconfigure key is only sent in Reply messages.

::: details Detailed changes
- Fixed the DHCPv6 fast-auth algorithm field, and the reconfigure key is now only sent in
  Reply messages.
:::

## v0.25.2 (2026-10-03) <Badge type="warning" text="pre-release" />

Device information now comes from one directory, shared code moves into landscape-core, and DDNS and shutdown handling are more robust.

::: details Detailed changes
- Device information now comes from one place: results from the device list, DHCP, SLAAC,
  and ARP are merged into a single view, and that view arbitrates address conflicts.
- The directory broadcasts device events, and DDNS, NAT6, and others listen for them.
  Device IDs follow enrollment records.
- The devices page is fed by this directory, so the per-service DHCPv4/IPv6 status
  endpoints are no longer used.
- Device entries gained two extra records: an ARP presence trail and a DHCPv4 lease clock.
- Link-local IPv6 addresses are classified by source, and address tags and scrolling on the
  devices page were improved.
- Shared pieces moved into landscape-core: the route service, the TLS SNI resolver,
  certificate PEM helpers, and the geo cache engine.
- Flow WAN target sync became event-driven, and `tc_lan_dao` split into a standalone filter
  attached by the LAN IPv6 service.
- DDNS background loops shut down gracefully, and provider HTTP requests have timeouts.
- Critical IPs are preserved on shutdown so you do not lose access while services stop.
- Rollback boundaries from 0.21.5 through 0.25.1 were registered, so rollback can offer
  targets that match the current version.
:::

## v0.25.1 (2026-10-01)

Configuration writes move to ConfigStore with verify-before-save, and memory self-monitoring arrives.

::: details Detailed changes
- Every configuration module now goes through `ConfigStore`: writes are transactional,
  validation sits at the write door, and the old duplicate implementations are gone.
- Configuration writes now verify that the service can actually start before saving, so you
  no longer end up with saved config that fails to boot. Flows, static NAT, IPv6 PD, and
  others follow this path.
- Service state moved from watch channels to a structured-concurrency `ServiceHandle`.
  Singleton services have their own run state, and stopping and status reporting are
  deterministic.
- New memory self-monitoring: heap accounting per subsystem, labeled task spawning,
  per-minute memory metrics on disk, and a memory analytics page in the UI.
- Memory tracking is off by default and needs the `mem-track` build feature. When on, you
  also get a breakdown of how RSS is composed.
- This area was later renamed to the self-monitor domain, across APIs, crates, and the UI.
- Geo source files are stored under `geo/raw`, with offline restore and raw file download.
  Writes go to a temp file first, then get verified and atomically renamed.
- The `config` subcommand can generate `landscape_init.toml`, and set admin credentials,
  WAN-only mode, and static NAT flags.
- The topology is driven by whether `--wan-iface` / `--lan-iface` are present, and static
  NAT ports get validated.
- Fixed a subtle problem: the LAN ARP scan did not re-read the interface and could use a
  stale MAC.
- Housekeeping: dead store implementations removed and the file store renamed to geo_cache;
  config, error, and DTO types split by domain.
:::

## v0.25.0 (2026-09-28)

Geo reverse lookup, resource names, HTTP/2 and compression; admin credential comparison and login rate limiting were hardened.

::: details Detailed changes
- Geo gains reverse lookup APIs and a page, and refresh failures now report errors.
- Configurable resources can be given names.
- The backend enables HTTP/2 ALPN and response compression.
- UI theme settings are saved.
- Runtime interface addresses are readable through the API, and netlink handling was
  hardened.
- Build capabilities are reported, and the UI uses them to decide what to show.
- A `config` subcommand generates `landscape_init.toml`.
- Security: admin credentials are compared in constant time, login is rate limited, and the
  admin password is no longer written to logs.
- Upstream DNS gained an experimental connection pool flag, off by default.
- The topology view switched to a dagre layout with measured node heights.
- Pending PPPoE configs show as cards in the topology, and must be disabled before they can
  be deleted.
- The pppd service was rewritten as an async supervisor with unit tests.
- The metric ringbuf consumes on a 1-second fallback timer, fixing rare stalls when an epoll
  wakeup was lost.
- The NTP sync service was rewritten around an async client with testable traits.
- Fixed OpenAPI type mapping for `MacAddr` and port ranges.
- Removed leftover DuckDB code.
:::

## v0.24.3 (2026-09-10)

eBPF maps are pinned from Rust and split by domain; the legacy PPPoE client is gone and end-to-end smoke tests were added.

::: details Detailed changes
- GeoIP can be refreshed per source.
- Metric SQLite migrations run one transaction at a time, and a failed database connection
  is retried at startup.
- eBPF maps are pinned from the Rust side, init is split by domain, and the related types
  were reorganized into a per-domain tree.
- The DNS server is decoupled from eBPF, with the two connected through traits.
- eBPF map key and value types switched to zerocopy mirror types.
- The legacy PPPoE client was removed in favor of `wan_service`, along with fixes to status
  timing and eBPF init/teardown.
- eBPF state no longer lives in a process global but in an explicit `EbpfRuntime`, and PPPoE
  end-to-end clients each get their own map space.
- A map that cannot be opened, or a pin directory that is unavailable, now reports an error
  instead of panicking.
- Route BPF headers are split per version with unified route4/route6 naming, and tests are
  split by domain.
- New end-to-end smoke tests that need root; cross builds moved to cargo-zigbuild and CI
  jobs were merged.
- Log files are named `landscape.YYYY-MM-DD.log`.
- loongarch64 gateway support was enabled.
:::

## v0.24.2 (2026-08-31)

The metric backend moves from DuckDB to SQLite with per-minute DNS buckets, and IPv6 neighbor learning arrives.

::: warning Breaking changes
The metric backend moves from DuckDB to SQLite. DuckDB-only metric settings are removed,
metric config units are normalized to MB and seconds, and the accepted ranges for capacity
and retention changed, so it is worth reviewing your metric configuration after upgrading.
:::

::: details Detailed changes
- The metric backend now uses SQLite and is the default. DuckDB is gone, and the old duckdb
  mode is routed to the new store.
- The metric module is split into aggregation and persistence layers, with sinks and workers
  behind traits.
- Connect metrics gained pagination and totals, and omitting the limit returns the whole
  history.
- DNS metrics are pre-aggregated into 1-minute buckets for the dashboard, connect metrics
  became append-only, and bucket retention is configurable.
- Capacity caps were added: the store cleans up proactively when full and hot-reclaims on
  `SQLITE_FULL`.
- Metric config units are unified on MB and seconds with validated ranges.
- Metric config updates apply as one transaction and roll back on failure.
- Metric service lifecycle hardening: a lock for config switches, bounded reader shutdown,
  recovery from poisoned locks, and workers that flush before exiting.
- The ringbuf reader became a cancellable async loop, and `ConnectMetric` became
  zero-copy compatible with the eBPF event.
- Cold bucket writes are batched and pre-aggregated, and leftover WAL is replayed under a
  relaxed memory limit before the cold store opens.
- Connect and DNS history retention and the `conn_summaries` row cap are now enforced.
- New `tc_lan_dao` program learns IPv6 neighbors from DAD neighbor solicitations arriving on
  the LAN. On a cache miss, it also learns the IP-to-MAC mapping from inbound packets.
- Fixed neighbor learning storing the wrong MAC: it recorded the destination MAC instead of
  the source MAC.
- Static NATv6 can track devices that have no fixed IPv6 suffix.
- The PPPoE LCP echo interval is configurable.
- Fixed a SQLite write lock leak, and `SQLITE_BUSY` now retries.
- Housekeeping: tests dropped redundant global locks, fixed sleeps became bounded polling,
  XDP veth tests run in isolated netns pairs, and two-netns TC chain integration tests were
  added.
- Peer interface lookup skips non-directory entries under `/sys/class/net`.
:::

## v0.23.1 (2026-08-20) <Badge type="warning" text="pre-release" />

A transactional ConfigStore, with errors layered by database, boot, and service startup.

::: details Detailed changes
- A new `ConfigStore` provides transactional writes and returns what changed.
- Error types are layered: database, boot, and service startup each get their own kind, and
  API-layer errors can carry semantics and redaction through.
- API error types were consolidated into Geo, firewall, and DNS.
- DNS rule persistence and API error handling were improved.
- SLAAC addresses claimed through an NA without a TLLA option are now verified.
- DNS domains are normalized to ASCII.
- DHCPv6 interfaces are included in the enrolled device options.
:::

## v0.23.0 (2026-08-16)

The DNS server is rewritten as an engine plus matcher with allocation-free matching; redirect rules can pass metadata through.

::: warning Breaking changes
DNS source-address binding moves from rules to upstream configuration, and DNS and
destination IP rules can no longer be moved across flows.
:::

::: details Detailed changes
- The DNS server was rewritten as an engine plus matcher and rule components, split into
  separate cache, resolve, redirect, and snapshot chains.
- Domain matching moved from trie-rs to zerotrie with RegexSet, so matching no longer
  allocates.
- Local answers were extracted into a `LocalResolver` shared by every DNS path.
- Query paths and the rule lookup / cache write path were unified, and reverse lookup
  refresh semantics were corrected along the way.
- Matcher tests cover keywords, regexes, non-ASCII skips, and edge-case fallthrough.
- The resolver gained a TCP fallback, and upstream retry timing was tightened.
- DNS error codes from upstreams are passed through to clients, with response code mapping
  centralized in one place.
- Redirect rules gained a "pass metadata queries through" option, with a switch in the UI.
- The upstream bind pool is keyed by (dns mark, upstream), and source-address binding moved
  from rules into upstream config.
- Tencent DNS reconciliation reuses existing records and caps values at two.
- DNS reports a more accurate service status and degrades gracefully when startup fails.
- Moving DNS and destination IP rules across flows is blocked, so flow runtimes cannot read
  stale data.
- Geo: only genuinely invalid GeoKey sources are skipped, and rules without an effective
  domain are dropped. Applying geo values uses SHA256 to tell whether anything actually
  changed.
- The SLAAC liveness probe became two-stage, refreshing through solicited NAs.
- Architecture diagrams were added and screenshots moved into an assets directory.
:::

## v0.22.3 (2026-08-04)

DHCPv4 is unified on dhcproto; LAN IPv6 prefix planning and RA details were corrected.

::: details Detailed changes
- DHCPv4 message handling is unified on dhcproto, with the helpers moved into net_proto.
- Matching neighbor solicitations refresh the liveness of learned SLAAC addresses.
- Static parent prefix length in LAN IPv6 is limited to /56–/63.
- Router advertisements are allowed on PD prefix groups in SLAAC+DHCPv6 mode.
- /64 slot 0 is reserved for WAN.
- LAN IPv6 prefix conflict validation returns typed errors.
- The flow trace drawer batches verdicts per address family and guards against stale
  requests.
- The OpenAPI spec ships with each release.
- NAT restarts when the WAN IPv4 route address changes.
:::

## v0.22.2 (2026-07-31)

Code is repartitioned by business domain; static NAT catches up on conflict detection and flow rules can be named.

::: warning Breaking changes
RA lifetimes are derived from the global `ad_interval`, and IPv6 address lifetimes come from
a single global `preferred_lifetime`. The previous per-prefix-group lifetimes no longer
apply.
:::

::: details Detailed changes
- Code was repartitioned by domain into config, LAN, WAN, and system modules, with shared
  runtime code moving into `landscape-core`. This is structural; the exposed configuration
  is unchanged.
- Static and dynamic NAT now check port conflicts in both directions, with a real-time check
  API and debounced validation in the UI.
- Fixed UDP checksum encoding: the 0 that means "no checksum" stays as is, while a computed
  zero becomes 0xFFFF per RFC 768.
- The inner ICMP echo inside ICMP error packets is NATed too, with header checksums updated.
  DNAT and the scanner handle ICMP errors carrying minimal TCP quotes.
- Fixed wraparound when generating dynamic mappings, and rewrote timer resolution with
  tests.
- Connection tracking release is serialized, and egress resolvers skip entries that are
  being cleaned up.
- Timer values carry an `is_static` marker to identify static mappings.
- IPv6 static mappings share the WAN IID. Bit 63 of the IID splits WAN and LAN address
  space, and static NATv6 can track dynamic IPv6 addresses.
- IPv6 PD gained a prefix planner: group planning, expected prefix length tracking,
  re-selection after expiry, and color coding per group.
- RA and IPv6 address lifetimes are now global settings rather than per-group.
- Flow rules gained a dedicated `name` field.
- DHCP advertises the LAN suffix as option 15 / 119. LAN hostname config gained a toggle,
  validation, and a page, with old keys migrated automatically.
- DNS PTR lookups prefer the hostname of an enrolled device, LAN suffix matching covers the
  whole suffix, and the hardcoded `home.arpa` handling was replaced by generic suffix
  matching that still respects `.local` mDNS semantics.
- Hostname logic was pulled out of the DNS server and merged with DHCP hostname tracking,
  later renamed `lan_hostname` with a serde alias so old config still works.
- L2 broadcast and multicast MACs are checked, and broadcast/multicast IP checks were
  unified into a set of helpers.
- Neighbor solicitation support was added, including MAC extraction, and link-local
  recording was reworked.
- ICMP passes through in tproxy mode, and whether it does is now controlled by a flag
  instead of being hardcoded.
- Config services switched from file-level locking to per-section hashing.
- UI: a new About page listing GitHub contributors; i18n split into per-domain modules;
  charts moved from ApexCharts to Apache ECharts; fonts centralized into CSS variables; a
  shared duration formatter; and the sidebar hierarchy and route paths regrouped.
- DHCPv6 only includes the Authentication option when the client supports ReconfAccept.
- Fixed a WAN interface issue: a link-local address negotiated over IPv6CP was not applied
  to the interface.
- Gateway upstream TLS can now skip certificate verification, which helps with internal
  self-signed setups.
- PPPoE discovery can filter by AC-Name.
:::

## v0.21.4 (2026-07-02)

Fixed the LAN IPv6 route key collision, modularized the PPPoE client with tests, and fixed PD route installation.

::: details Detailed changes
- Fixed the LAN IPv6 route key collision by storing the real subnet index and dropping the
  WAN offset.
- On startup, static NA reservations get their IPv6 assignment records seeded.
- DNS record TTL is configurable, and Tencent DNS handling was hardened.
- The static NAT mapping service was split into IPv4 and IPv6.
- A PPPoE client module was added, restructured into two phases with classified errors.
- Fixed Ethernet frame parsing offsets in PPPoE discovery and LCP negotiation.
- Added PPPoE negotiation tests and fixed the runtime echo-request logic.
- PPPoE sessions now wire up eBPF TC/XDP and route setup as part of their lifecycle.
- LCP tracks the peer MRU and magic, handles PAP rejects, and logs more clearly.
- RA is sent immediately on startup and on prefix changes.
- Fixed IPv6 PD: the delegated prefix route is installed on both request and renew.
- Broadcast and link-local packets are allowed to enter the WAN interface under XDP.
:::

## v0.21.3 (2026-07-01)

The event hub lands, the DHCPv6 server is rewritten with Reconfigure support, and DDNS can resolve device IPv6 through PD.

::: warning Breaking changes
Static NAT6 config fields are renamed: `mapping_pair_ports` becomes `port_config` and
`device_id` becomes `device_ids`. The device management page no longer refreshes IPv6
suffixes automatically.
:::

::: details Detailed changes
- A new `EventHub` centralizes interface events, device events, and IPv4/IPv6 address
  assignment and expiry events, and pushes them to the UI.
- DHCPv4 allocation state is tracked separately, and device bindings update from device
  events.
- DHCPv6 allocation state is likewise tracked separately behind an async-safe mutex, with
  lease allocation and prefix sync reworked.
- Per-interface watch channels were replaced by a central IA prefix event bus where each
  consumer gets its own copy of the data.
- The DHCPv6 server was rewritten: NA / PD / SLAAC lease state is managed together, with
  ICMPv6 RA/NA handling, SLAAC tracking, periodic lease cleanup, and PD route management.
- DHCPv6 Reconfigure support (RFC 8415 §20.4).
- DUID-UUID clients fall back to looking up the MAC via the link-local address, and a
  missing MAC can be recovered from SLAAC NA conflicts.
- DHCPv6 validates suffix ownership in Confirm, and static bindings always refresh.
- DDNS can resolve the IPv6 of enrolled devices through PD delegation, sync multiple
  records, and filter by GUA/ULA. You can pick a LAN device as the source in the UI.
- Added a MAC-to-link-local mapping cache with periodic neighbor scans. IPv6 assignment
  events changed from a single address to per-MAC groups, with a refresh event added.
- LAN hostnames can take part in DNS resolution: the hostname reported by DHCP maps to IPv4.
- Static NAT6 port config became an enum (all ports, or a list) and targets support several
  devices with duplicate checks. A port wildcard fallback was added, and static NAT
  mappings are split into IPv4 and IPv6 sets in both the API and the UI.
- The device management page gained IPv6 suffix randomization and manual refresh, and
  automatic refresh was turned off.
- Geo: site configs support force refresh. An AdGuard rule conversion panel was added, with
  conservative parsing that understands negated rules.
- Log timestamps use the machine's timezone, and `--log-filter` adds keyword filtering.
- eBPF ARP/NDP sync switched to netlink instead of reading `/proc/net/arp`, and now handles
  IPv6 neighbors.
- Fixed the LAN IPv6 service missing an intermediate state transition, and an infinite loop
  when the prefix-change sender was dropped.
- DHCPv4 prefers static bindings when allocating, and NAK responses use the static IP too.
- Dependency upgrades: hickory-dns 0.25 → 0.26, and the pnpm config moved to a workspace.
- Housekeeping: threads and async tasks are named, modules keep migrating by domain, and
  rollback boundaries for 0.20.1 and 0.21.0 were registered.
:::

## v0.20.1 (2026-06-20)

A large part of the forwarding pipeline moves into XDP and unifies with TC; seven use-after-free bugs were fixed.

::: warning Breaking changes
Native XDP changes from default-on to opt-in via `--try-xdp`, falling back to TC when it is
not passed. The LAN static route `is_next_hop` field is replaced by `route_type`.
:::

::: details Detailed changes
- The XDP entry dispatch landed, including PPPoE stripping and tail calls, along with the
  XDP chain infrastructure and 3-level chain tests.
- XDP WAN and LAN routes were added, with caching, flow verdicts, and FIB fallback, covered
  by bidirectional integration tests.
- The bidirectional XDP chain applies MSS clamping on both LAN→WAN and WAN→LAN.
- A new XDP scanner classifies IPv4/IPv6, and MSS clamping switched to it with IPv6 support.
- XDP firewall was added, with integration tests covering IPv4/IPv6 blocking, direction, and
  unblocking.
- A new XDP NAT stage supports IPv4/IPv6 static and dynamic mappings.
- Fixed XDP checksums by using the `csum_fold` pipeline instead of assigning
  `bpf_csum_diff` directly.
- Fixed the byte order of the IPv4 fragment offset.
- Fixed seven use-after-free bugs, including in `SkbXdpBundle`, caused by struct field order
  not matching Rust's drop order.
- Fixed FIB fallback hijacking unknown IPs when `lan_map` missed.
- XDP and TC are unified: there is no route mode any more, both always run and hand off to
  each other. The route cache gained a "can XDP redirect" flag, and interfaces that cannot
  use XDP fall back to TC.
- Docker traffic handoff from XDP to TC works, with VLAN tagging.
- A TC chain architecture was introduced, turning firewall, MSS clamping, NAT, and PPPoE
  encapsulation into stages on the chain.
- The route mode can be chosen between TC and XDP from the command line.
- The LAN ingress TC chain was removed; LAN traffic now reaches the WAN egress chain through
  redirect.
- Fixed the WAN route type being skipped in the TC chain, and non-local WAN traffic returning
  the wrong status code.
- `is_next_hop` became `route_type`, with support for WAN-reachable routes.
- When the MAC lookup fails it falls back to `bpf_redirect_neigh`, and unmatched default
  flows are dropped.
- Removed the old v1/v2 route pipeline and the old firewall and MSS clamp modules.
- PPPoE supports XDP with an encapsulation stage, authentication supports CHAP MD5, and
  PPPoE was decoupled from the TC chain into its own attach.
- Static IPv6 link-local neighbor entries are written after PPPoE dial-up.
- A new ethtool module disables GRO when an interface comes up.
- Fixed the ethtool call type conversion on musl.
- DHCPv6 PD gives each client one pool block and uses that block's own prefix length in the
  response.
- On a neighbor advertisement, the IPv6 IP-MAC binding is written into the eBPF map.
- An mDNS local hostname responder and DNR advertisement support, and local DNS no longer
  leaks to upstream resolvers.
- Reverse DNS preflight handling was hardened.
- Docker: unified error types and removed dead code in event handling. Polling only happens
  on Docker-related pages, so an unavailable Docker no longer blocks periodic refresh.
- On shutdown the DHCPv4 server address and WAN static address are kept, so you can still
  reach the device.
- BPF programs were relicensed from Dual BSD/GPL to GPL.
- Housekeeping: test programs moved to their own directory and are no longer compiled into
  release builds.
:::

## v0.19.2 (2026-05-07)

WAN TC stages become a shared pipeline with a new LAN pipeline; PPPoE goes native and flows support weights.

::: warning Breaking changes
The deprecated `mac_binding_records` field is removed from `DHCPv4ServerConfig`; use
enrolled devices for MAC-to-IP bindings.
:::

::: details Detailed changes
- WAN TC stages were reworked into a shared tail-call pipeline, plus a separate LAN TC
  pipeline.
- Flow targets support weights distributed across slots, up to 16 targets.
- Resolved WAN target information is cached, so later packets skip the slot map lookup.
- Interface connection metrics were added.
- Fixed VLAN tagging on cached Docker routes.
- Fixed inconsistencies between the IPv4 and IPv6 WAN route paths, along with a few pointer
  and type problems.
- Route and tproxy packet parsing was rewritten, with L3 and L4 scanning split apart.
- Fixed MSS clamp edge handling.
- PPPoE was rewritten as a native implementation with a config flow and integration tests;
  the old dump parser is gone.
- DHCPv4 fixes the discover length field and uses unicast for the last two retries. ACKs set
  the server address and echo the client address per RFC 2131.
- Configurable DHCP options with an option editor; fixed custom option injection in NAKs.
- Removed the deprecated `mac_binding_records` from `DHCPv4ServerConfig`.
- Change-password support, with auth config reloaded on the fly.
- A custom DoH HTTP/2 listener.
- A versioned init config import flow.
- An AdGuard geosite source.
- A unified WAN candidates API, so DDNS can pick a PPP interface.
- NTP sync retries use exponential backoff.
- Fixed PPPoE termination on startup and the LAN pipeline drop order.
- DHCP lease acknowledgements prewarm the eBPF IP-MAC map, and Docker registration is more
  resilient.
- Config toggles moved next to modal titles.
:::

## v0.18.3 (2026-04-16)

Privacy masking, a fixed device binding flow, and steadier PPPoE validation and PPPD lifecycle.

::: details Detailed changes
- Sensitive fields are masked in the certificate and gateway views.
- Fixed the enrolled device binding flow.
- DHCP lease state syncs with binding hints.
- PPPoE interface validation and the update flow were hardened, and the PPPD lifecycle
  recovery is more stable.
:::

## v0.18.2 (2026-04-12)

DNS provider profiles and DDNS jobs, LAN IPv6 unified on prefix groups, and polished topology interactions.

::: details Detailed changes
- DNS provider profiles and DDNS jobs, shared between certificates and DDNS.
- DDNS is more reliable, record TTL is controllable, and source input now gives validation
  feedback.
- Route trace shows DNS filtering and cache check results, and DNS diagnostics can toggle
  caching at runtime.
- LAN IPv6 configuration is unified on prefix groups, with a clearer overview, inactive
  groups called out explicitly, and default static prefixes generated automatically.
- PD subnet 0 is reserved for the WAN IPv6 address.
- Gateway: clearer proxy error responses, dual-stack listening, and normalized trailing
  slash redirects.
- Fixed IPv6 route cache handling, and reorganized the route cache and route service layout.
- Fixed stale neighbor IP/MAC cache updates.
- DHCPv4 LAN routes are only advertised for enabled services, and DHCP zone cleanup moved
  inside the DHCP service.
- `accept_ra=2` is no longer forced onto PPP interfaces.
- Topology: selected nodes get focus, and panel interactions, layout, shortcuts, and service
  failure states were polished.
- Fixed IP ordering in the DHCPv4 status table.
:::

---

For full details on every tag, see the
[landscape releases](https://github.com/ThisSeanZhang/landscape/releases).
