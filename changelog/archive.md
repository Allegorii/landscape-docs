---
pageClass: changelog-page
---

# Older Releases

Landscape Router releases published before 2026-04-11. For the most recent six months see
[Version History](./); the presentation and marking rules are the same as on that page.

## v0.17.5 (2026-03-25)

Metric storage reworked for faster startup, with adjusted metric modes and DuckDB recovery.

::: details Detailed changes

- A missing `neigh_update` kprobe is handled gracefully instead of erroring out.
- Metric storage was restructured for faster startup; the connect write path and store
  backends were reworked, and global stats are cached by default.
- Metric modes were redone, and DuckDB recovery was hardened.
- Shutdown signals moved from ctrlc to tokio.
- Generated IDs and `update_at` defaults are aligned.
- eBPF handles and error context were refactored.
- Fixed DNS resolution for IPv6 link-local clients by deriving the address from the MAC.
  :::

## v0.17.3 (2026-03-17)

The gateway reverse proxy arrives, certificates gain EC keys and wildcard matching, plus rollback and time sync.

::: details Detailed changes

- A gateway reverse proxy with SNI support, including config, database, and UI management.
- The gateway supports HTTPS termination and passthrough routing, and host routing and the
  editor layout were reorganized.
- Without a certificate the gateway can still listen on 443 and blocks connections until one
  is ready.
- Certificate parsing supports EC private keys. A generated certificate mode was added,
  certificates can fall back to regeneration, and wildcard certificate matching was fixed.
- Common ACME DNS providers were integrated, and certificate domains for the gateway and API
  sync to dynamic DNS.
- DNS gained dynamic redirection, local redirect answers use route snapshots, and a DNS
  deadlock in certain scenarios was fixed.
- A database rollback subcommand.
- A time sync service.
- A global metrics on/off switch.
- A caller identity lookup API.
- Disabling the interface you are currently using no longer cuts you off immediately; it
  asks for confirmation first.
- NAT v3: shared maps initialize in one place and runtime bundles are reused. Fixed runtime
  restart handling, WAN scoping, IPv6 prefix matching, IPv4 timer cleanup, and ICMP
  handling.
- Fixed gateway shutdown cancellation.
- Housekeeping: threads and async tasks are named, and geo cache memory reads were
  optimized.
  :::

## v0.16.3 (2026-03-11)

Verdict queries accept multiple IPs, TXT geo IP import, and display fixes.

::: details Detailed changes

- Verdict queries accept multiple destination IPs, with input validation.
- The geo IP service deduplicates IP mark CIDRs.
- Cache consistency for verdict results is judged and shown correctly, especially for
  `keep_going` actions.
- In the DHCP assignment table, the name of an unknown device can be resolved from its MAC.
- TXT-format geo IP import, with a configurable key.
  :::

## v0.16.2 (2026-03-06)

Certificate account emails are masked, and metric cleanup is decoupled from aggregation.

::: details Detailed changes

- Certificate account emails are masked.
- Metric cleanup and aggregation were decoupled, several config parameters were added, and
  DuckDB thread management was adjusted.
  :::

## v0.16.1 (2026-03-05)

Certificate management takes shape (ACME accounts, DNS challenges, dynamic SNI resolution); DoH and a netlink module arrive.

::: warning Breaking changes
The UI language is fixed to `zh` instead of detecting the browser language.
:::

::: details Detailed changes

- A new `netlink` module collects interface parsing, link management, address handling, and
  event observation.
- Certificate management took shape: ACME account registration, verification and
  deactivation, certificate issuance with DNS challenges, account and order management,
  certificate details and download, cancellation, and status sync.
- A valid certificate can no longer change its ACME account, with a dedicated error message
  for it.
- Interface TLS certificates resolve dynamically by SNI.
- DNS over HTTPS (DoH) support.
- HTTPS-type DNS records, with IP hints filtered by IP version.
- DHCP and PPPoE clients gained exponential backoff retries.
- Fixed PTY session creation failures; the shell now tries several candidates. Terminal data
  is decoded with `TextDecoder`, and font preferences were updated.
- ICMPv6 option parsing and serialization were rewritten with `deku`.
- Reusing a pinned map now checks type, entry count, and flags so mismatched maps are not
  mixed up.
- The UI language is fixed to Chinese instead of probing the browser.
- IPv6 flow match length was updated, and DHCPv6 and ICMPv6 prefer link-local DNS.
  :::

## v0.15.0 (2026-03-01)

The DHCPv6 server lands, RA merges into LAN IPv6 with three modes, and PD route management is implemented.

::: warning Breaking changes
The previous ICMPv6 Router Advertisement UI components and services are removed. RA
configuration now lives in the LAN IPv6 server, where all three modes are selected.
:::

::: details Detailed changes

- The DHCPv6 server was implemented.
- RA configuration and PD management merged into the `ipv6/prefix` module with
  `IPv6PrefixRuntime`.
- The RA service became the LAN IPv6 server, supporting Slaac, Stateful, and SlaacDhcpv6
  modes.
- IPv6 source configuration is shared between DHCPv6 and RA, and DHCPv6 prefix delegation
  settings were simplified.
- IPv6 PD route management was implemented.
- ICMPv6 Router Advertisement UI components and services were removed.
- A new `LanOrUndefined` zone requirement, with WiFi AP mode restricted to non-WAN zones.
- Pinned eBPF maps check key and value sizes, so a silent layout mismatch on reuse is caught
  instead of going unnoticed.
- A `from_addr = 0` fallback was added to egress lookups for static mappings to the local
  machine.
- NAT tests were split into IPv4/IPv6 submodules covering static and dynamic cases.
- The UI adopted i18n, with components moving to translated strings.
  :::

## v0.14.3 (2026-02-26)

Connection tracking keys use the allocated ICMP ID, and shutdown also handles SIGTERM.

::: details Detailed changes

- Connection tracking keys now use the allocated ICMP ID, so egress and ingress match up.
- Shutdown handling covers SIGTERM in addition to SIGINT.
  :::

## v0.14.2 (2026-02-26)

Graceful service shutdown, config validation wired into the API, and optimistic locking for updates.

::: details Detailed changes

- Services shut down gracefully.
- Service state is unified behind `WatchService`, and log levels and state transitions were
  straightened out.
- Service configuration is validated against the interface type and zone requirements.
- Validation for NAT, DHCPv4, PPPoE, MSS Clamp, Wi-Fi, and IPv6 RA configuration was wired
  into the API.
- Configuration updates use optimistic locking based on timestamps to detect conflicts.
- PPPD config supports a custom PPPoE plugin.
- Fixed upstream DNS updates failing on quick fill because the timestamp was missing.
  :::

## v0.14.0 (2026-02-24)

OpenAPI generation and a reference UI with the frontend on generated clients; firewall rule management is removed.

::: warning Breaking changes
Firewall rule management is removed from the web UI and the server API. The API also
requires global JWT Bearer authentication on everything except the login endpoint.
:::

::: details Detailed changes

- OpenAPI generation and a reference UI were introduced, with configuration models and API
  routes annotated through utoipa.
- The frontend switched to generated API clients and schemas, with a request instance that
  shares interceptors.
- The API requires global JWT Bearer authentication, except for the login endpoint.
- Firewall rule management was removed from both the web UI and the server API.
- API errors are structured: each kind has a unique ID and supports multiple languages.
- The `landscape-types` package was published, and type definitions were consolidated in one
  place.
- Metric views show where an IP is located, and route trace shows devices more clearly.
- The API reference page is customized with Scalar plus a home button, and its assets are
  served locally rather than from a CDN.
- DuckDB metric storage became optional, falling back to a no-op when disabled.
- CI was split: aarch64 gets its own build, and the OpenAPI spec is exported from CI to
  generate frontend types.
- DHCPv4 LAN route management was improved.
  :::

## v0.13.1 (2026-02-23)

Flow tracing arrives with IPv6 support, along with a firewall blacklist and connection direction.

::: details Detailed changes

- Static NAT reports connection metrics.
- MAC masking in privacy mode went from first-and-last segment to the first two and last two.
- Connections gained a direction field.
- Flow tracing arrived, with IPv6 support.
- Connection direction visualization improved: arrows are dynamic, and direction filters are
  translated.
- Geo IP and Geo Site gained a direct source type.
- Flow rules detect duplicate and conflicting match entries.
- The firewall blacklist arrived, with a description field for each IP entry.
- Domain handling supports IDNA (Punycode) in both backend and frontend.
- Network tools can extract domains, and flows have default labels.
  :::

## v0.13.0 (2026-02-22)

IP-MAC bindings become enrolled devices, with a unified privacy mode and QUIC socket marking.

::: warning Breaking changes
The `conn_realtime` and `conn_summaries` tables and their related logic are removed. Flow
rule match entries use enrolled devices, and that input no longer accepts IPv6 for now.
:::

::: details Detailed changes

- IP-MAC binding management: pick an interface, validate against DHCP ranges, prevent
  duplicates, and select interfaces dynamically in the UI.
- A unified privacy mode, with consistent masking for IPs and MACs. The DHCP UI uses
  privacy-aware device names and supports quick binding.
- IP/MAC bindings migrated to enrolled devices.
- Flow rule match entries can pick enrolled devices, and LAN IPv4 input toggles between
  picking a device and typing it in.
- QUIC socket marking was added, DoT/DoQ default ports were adjusted, and the default
  upstream options were tidied up.
- DHCP service conflict detection was redone.
- DNS statistics breakdowns are shown on hover.
- The `conn_realtime` and `conn_summaries` tables and their logic were removed, and query
  parameterization improved.
  :::

## v0.12.7 (2026-02-18)

A big pass on connection metrics and DNS: realtime rates, history, a dashboard, moka and negative caching, plus batched DuckDB writes.

::: details Detailed changes

- Realtime connection rates and packet statistics.
- Historical connection metrics, with live charts switching to cumulative values.
- Connection history supports advanced filtering and sorting, with an 800ms debounce on
  queries.
- Connections gained a status field, and history queries moved to a summary table.
- The backend provides global connection statistics and the frontend aggregates live
  metrics.
- Charts support cumulative and delta modes.
- Metric history uses a virtual list, with custom time ranges, sorting by duration, and
  chart downsampling.
- Historical source/destination IP metrics, with finer filtering and tuple search.
- Global metrics can be refreshed manually, and connection time and duration display
  improved.
- NAT connection metrics gained a CPU ID.
- DNS results are classified as local, blocked, cache hit, or error.
- A DNS dashboard with advanced filtering, and a history table with paging, search,
  filtering, and sorting.
- DNS caching moved from `LruCache` to moka, with configurable cache settings.
- DNS negative caching, with a configurable TTL.
- Latency statistics exclude local answers and cache hits, with hover text explaining
  latency and hit rate.
- DNS error responses set the RA and AA flags correctly.
- Removed the reuseport DNS server implementation.
- DuckDB: metrics and connection info are written in batches, with configurable flush
  intervals and batch sizes.
- Metric retention, batch size, and flush interval are configurable, and retention can be
  set per granularity.
- DuckDB was upgraded to 1.4.4, the database file is versioned, and live and on-disk
  connections use separate pools.
- Queries compute byte and packet deltas with the lag function, and metric cleanup moved to a
  background thread.
- Metric views are split into connection and DNS sets, with components and routes organized
  in subdirectories.
- UI configuration management: language, timezone, and theme are all configurable with
  conflict detection, and times display in the preferred timezone.
- Metric configuration gained APIs, types, and frontend state.
- JWT tokens refresh automatically, and the login page redirect loop after a 401 is fixed.
- UI state management was reworked: username logic is centralized, interface node state
  persists, and there is a logout action.
- Visited route history, with tabs and pinning.
- SPA routing and a 404 page, and configuration views split into separate cards.
- Fixed misaligned memory access on RISC-V.
- A lightweight DNS status card.
- Network interfaces are configured automatically on first start.
- CPU softIRQ balancing extends to all interface queues.
- A unified network protocol codec module.
- The DHCPv4 client uses raw packet sockets.
- Reduced the frequent IO caused by fsync (a temporary measure).
- Housekeeping: i18n for configuration and error pages, and a formatting check in CI for
  Rust and frontend code.
  :::

## v0.11.3 (2026-01-29)

A button to clear the LAN route cache, more multicast filtering, and a validation scope fix.

::: details Detailed changes

- A button to clear the LAN route cache.
- More multicast addresses are filtered.
- Fixed a validation that should only apply to the LAN (#155).
  :::

## v0.11.2 (2026-01-25)

NAT connection state management and multi-port static NAT, plus CPU temperature monitoring and a WebShell rewrite.

::: details Detailed changes

- The log banner shows the version, and the UI warns when versions do not match.
- IPv4 and IPv6 NAT handling were separated.
- NAT gained connection state management, reporting v4/v6 connection info and v4 metrics.
- Connection tracking moved from the firewall to NAT, and firewall metric reporting was
  removed along with it.
- IPv4 static NAT mappings use a dedicated BPF map and Rust logic.
- A WAN mark identifier.
- The initial ARP sync frequency was optimized.
- LAN packets destined for another IP now look up and update the MAC address before being
  redirected.
- Static NAT supports multi-port mappings.
- CPU temperature monitoring (#114); the system info cards were redone; the CPU model is
  hidden in privacy mode; and the NaN% display when swap is 0 was fixed.
- WebShell became a global floating/docked terminal.
- `rust_bindings` became the standalone `landscape-types` package.
- UI: GeoIP/GeoSite search is easier to use (#147), results sort by key and name filtering
  became exact matching. MAC input is auto-formatted, and port input is range-validated.
- Fixed an unchecked-value null pointer, NAT connection state tracking, the prefix length
  when NAT maps to the local machine, and handler behavior in routing mode.
- Link-local multicast addresses are skipped when forwarding.
- `redirect_pkg_handler` supports several tproxy instances.
- Config can be saved when CPU balancing is set to 0, and stopped services show a red
  button.
  :::

## v0.10.3 (2026-01-12)

Topology remembers node positions and can be unlocked, empty DNS results can be redirected, and IP-MAC cache and Docker L2 issues are fixed.

::: details Detailed changes

- The CPU soft load settings component was improved.
- The topology remembers node positions and gained an unlock button.
- The IP-MAC cache listener was updated.
- Empty DNS results can be redirected, which is handy for blocking access.
- Housekeeping: NAT tests were separated from the main program, and dead service code and
  duplicate definitions were removed.
- Fixed the IP-MAC cache updating late, and the L2 protocol being wrong when sending to
  Docker containers.
  :::

## v0.10.1 (2026-01-04)

Fixed neighbor cache updates missing a flag check.

::: details Detailed changes

- Fixed an incorrect neighbor cache update caused by a missing flag check.
  :::

## v0.10.0 (2025-12-24)

LAN static routing arrives, WAN-to-LAN forwarding is reworked, and eBPF reads skb data directly.

::: details Detailed changes

- Basic LAN static routing, with a next-hop flag on route entries.
- WAN-to-LAN forwarding was reworked.
- The WAN cache now carries MAC information.
- eBPF optimizations: read skb data directly instead of copying byte by byte, and use branch
  prediction hints.
- Dependencies, frontend dependencies, and the node version were upgraded.
  :::

## v0.9.5 (2025-12-17)

Work on the V3 NAT begins, fragment handling and the route program are reworked, and the UI shows IPv6 neighbors.

::: details Detailed changes

- Work on the V3 NAT began (only partially complete at the time).
- Fixed IPv6 assignment being read incorrectly by the firewall; reporting is now on a timer.
- eBPF optimizations: direct skb data access, and recovering the packet digest from the
  control block.
- The fragment type enum was redefined and fragment reading logic fixed, with dead code
  removed.
- The route program file was renamed to `flow_route.bpf.c`.
- The UI can show IPv6 neighbor information.
  :::

## v0.9.4 (2025-12-06)

Fixed the firewall service failing to start under a newer verifier.

::: details Detailed changes

- Fixed the firewall service failing to start under a newer verifier.
  :::

## v0.9.3 (2025-12-01)

Fixed IPv6 fragment header handling, and the UI warns about DHCPv4 and network config problems.

::: details Detailed changes

- Fixed IPv6 fragment header handling.
- The UI warns about DHCPv4 service and network configuration problems.
- PPP interface IPs are added to the machine's own IP list.
- Added firewall and WAN IP forwarding test demos.
  :::

## v0.9.2 (2025-11-21)

Fixed NAT eBPF loading on RISC-V and moved the UI to a virtual list.

::: details Detailed changes

- Fixed NAT eBPF programs failing to load on RISC-V.
- The UI uses a virtual list for faster rendering.
- Housekeeping: CI and docs updated, dead code removed.
  :::

## v0.9.1 (2025-11-06)

A packet redirector handler mode, Docker network discovery, and several Docker fixes.

::: details Detailed changes

- A packet redirector handler mode.
- Docker network creation is observed and added to the LAN information.
- Fixed Docker image pulls, the wrong MAC field in Docker LAN info, and a panic when the
  Docker socket could not be reached.
- A copy button for destination IP rules.
  :::

## v0.8.1 (2025-10-26)

Upstream DNS addresses are validated, IPv6 routes are no longer deleted needlessly, and PPPoE supports an AC Name.

::: details Detailed changes

- Upstream DNS IP addresses are validated.
- The IPv6 static mapping implementation was updated.
- Fixed IPv6 routes being deleted when the route information had not changed.
- PPPoE connections support setting an AC Name.
  :::

## v0.8.0 (2025-10-10)

A route forwarding cache, source-MAC flow matching, and IPv6 NPT and TPROXY handling.

::: details Detailed changes

- A route forwarding cache.
- Flows can match by source MAC address.
- The DHCPv4 service scans ARP on a timer and shows the results, along with client
  hostnames.
- IPv6 prefix translation (NPT) for egress, and the feature was enabled.
- Internal ICMPv6 RAs support multiple source prefixes.
- Once prefixes are configured, static IPs are applied to the interface.
- IPv6 TPROXY handling was added (that traffic may still be filtered by netfilter rules).
- IPv6 static NAT mapping was completed.
- IPv6 port reuse is checked.
- Housekeeping: packet information retrieval was abstracted, and the L3 offset type went
  from int to u32.
  :::

## v0.7.1 (2025-09-26)

Container restart policies and image pull progress, WebShell, and a batch of compatibility fixes.

::: details Detailed changes

- Containers support a restart policy.
- Docker image pulling with realtime progress.
- Custom DoH addresses.
- A button for quickly adding DNS rules.
- WebShell.
- IP inputs and GEO import rules support IPv6.
- Docker containers support a custom entrypoint.
- The IP-MAC cache was removed.
- Fixes: compatibility with systems lacking nl80211, command failures from a missing `TERM`
  variable, redirect domains still querying upstream when IPv6 was not configured, wrong log
  path and server port configuration, and Debian 13 security policy blocking DHCP from
  getting an IP.
- Housekeeping: an incremental paste button, frontend styling, the remaining services moved
  into the app with an upgraded service management interface, and bollard and libbpf
  upgraded.
  :::

## v0.7.0 (2025-09-09)

Flows match by IP prefix, musl support, and more information in network views.

::: details Detailed changes

- Flows can match by IP prefix instead of exact IP only.
- Fixed the display of flow match rules.
- Network info shows the DHCPv6 delegated prefix and the IP assigned by DHCPv4.
- musl support.
- A button to delete a bridge.
- Sensitive information is hidden on several pages in privacy mode.
- Fixed the RA service failing to restart because a link-local address was missing.
- The default upstream IP changed to 1.0.0.1.
- Housekeeping: better error handling and logging for UnixListener, and frontend styling
  adjustments.
  :::

## v0.6.9 (2025-09-05)

The DNS service and server-side management take shape: a dispatcher, port reuse, and dedicated redirect and upstream config storage.

::: warning Breaking changes
The DNS upstream `resolve_mode` field is replaced by `upstream_id`, and upstream
configuration moves out of rules into a dedicated table.
:::

::: details Detailed changes

- A DNS dispatcher.
- The DNS service switched to port reuse (several workers sharing one port), introducing
  `ReusePortChainDnsServer`.
- DNS redirect configuration got storage and a page; upstream configuration became its own
  table with its own page.
- Upstreams are referenced by ID instead of by "mode".
- Fixes: dynamic NAT1 admission, DNS redirect rule update notifications, the upstream popover
  not refreshing when the rule ID changed, and cached record TTL not being shortened during
  reloads.
- GEO serialization moved from JSON to bincode, with restrictions on geo names.
- ID and time generation moved to the backend.
  :::

## v0.6.8-beta (2025-08-19)

Static NAT mappings and flow rules take shape, and a forwarding loop from two WAN NICs in one subnet is fixed.

::: details Detailed changes

- Static NAT mappings arrived, with a config service and a UI page. Rule processing order
  was adjusted, and IPv4 and IPv6 static NAT config was merged into a single record.
- Flows: the internal mark became a struct so DNS marks can be held as bits; the action field
  was split out with a port-reuse flag; the L4 protocol went from a single value to a list so
  several protocols can be matched; and a default flow card was added.
- Fixed a forwarding loop caused by two WAN NICs in the same subnet.
- With forwarding enabled, PPP interfaces can receive IPv6 addresses from RAs.
- Fixed DNS targets being looked up with the wrong flow ID, and the current flow missing a
  default flow ID.
- A rollback subcommand.
- Fixes: an address range ending in 0.0.0.0 is treated as unset, requested IPs must fall
  inside the assignable range, and fields were not properly reset when switching mark types.
- Subnet matching for non-default NICs, a reworked WAN IP cache map structure, and ICMP ID
  reuse.
  :::

## v0.6.7-beta (2025-07-24)

IPv6 forwarding arrives, flow rules gain priority, and API responses and error codes are unified.

::: details Detailed changes

- IPv6 forwarding with eBPF TC.
- Flow rules gained a priority field, and the default priority was fixed.
- Fixed `FLOW_DIRECT` behaving incorrectly on the LAN side.
- Docker containers are scanned at startup.
- IPv6 link-local addresses are generated from the MAC.
- The ICMPv6 RA interface setting became a dropdown.
- Geo can be updated from files, with cleanup on force update that no longer wipes existing
  data first.
- API response formats and system error codes were unified.
- Fixes: flow ID assignment and use, config file checks when adding, init config export, one
  map's value size, the route flag being 0 when sending NAs, IPv6 address printing, empty PD
  options when the RA config is empty, CTCC failing to get an IPv6 prefix, interface zone
  changes, and using the wrong service store.
- Untagged container traffic can enter the flow logic, and Docker traffic is directly
  connected by default.
- DHCP decline messages are handled.
- Container scanning uses periodic socket reporting.
- Housekeeping: configuration initialization options, the flow_id filter disabled on export,
  and CI for building the flow edge.
  :::

## v0.6.0-beta (2025-07-09)

An IP route service is defined and wired into DHCPv4 and WAN config; Docker containers can be routing targets.

::: details Detailed changes

- An IP route service was defined and wired into the DHCPv4 server and WAN IP configuration.
- A LAN forwarding BPF program, an IP-MAC cache to help redirection, and abstracted WAN
  selection logic.
- Routes got database storage and UI components, and route changes update the route target
  BPF map.
- Docker containers can be routing targets.
- Flow IDs moved from hardcoded values to database lookups.
- GeoSite configuration supports attribute parsing and filtering.
- Metric cleanup time was set to 10 minutes, and polars and duckdb became optional at build
  time.
- Interface configuration switched from typing to selecting, and the WAN card's LAN IP was
  added to the checks.
- A ppp adapter.
- Fixed a DHCP Inform message handling error.
  :::

## v0.5.5-beta (2025-06-23)

Configuration moves into the database, with geo management pages, HTTPS, and config export and restore.

::: warning Breaking changes
Configuration moves out of files and into the database, which changes where settings live.
Reading `landscape.toml` and config export/restore are introduced at the same time.
:::

::: details Detailed changes

- Configuration moved from files into the database, one area at a time: interfaces, the
  DHCPv4 service, DNS, IPv6PD, firewall, flows, MSS clamp, NAT, pppd, ICMPv6 RA, IP mode,
  WiFi, and the various rule sets.
- A geo site service and geo ip management page, with DNS rules referencing geo keys.
- HTTPS support, with http and https on separate listeners and a redirect between them.
- Configuration can be exported and restored, `landscape.toml` can be read, and geo
  configuration can be exported and restored too.
- The UI shows the version, and there is a one-click privacy display.
- DNS rule behavior checks, and a timeout for domain checks.
- The DHCPv4 service checks for address range conflicts, and offers return the lease time.
- An armv7 build target, and the aarch64 build image gets a cross toolchain.
- Fixes: the UI freezing when editing a mask with the keyboard (#63), a `WifiMode`
  deserialization error, overlong generated ppp interface names, serialization
  inconsistencies, a wrong geoip IP key, DNS flow settings updating late, subdomain prefix
  matching, the RA service hanging on a missing link-local address, an MSS clamp calculation
  error, and slow domain matcher initialization.
- Metrics moved to duckdb and polars.
- Frontend refactor (fixes #44), and TypeScript types generated for configuration.
- DHCPv4 migrated to the new service management framework.
- Buttons for adding a default destination IP rule and for copying and pasting config.
  :::

## v0.4.12-beta (2025-05-16)

The firewall lands in three parts, flows and metrics follow, and an AP service and ECMP configuration arrive.

::: warning Breaking changes
The firewall function built into NAT is removed; that role is taken over by the standalone
firewall feature.
:::

::: details Detailed changes

- The firewall landed in three parts, with a UI and enums for protocols and ICMP types.
- A hostapd AP service.
- ECMP routing configuration.
- System token export.
- IP configuration for hardware monitoring restarts.
- A button to hide offline devices in the topology.
- The DHCPv4 client and server were reworked.
- DHCPv4 MAC-to-IP static bindings.
- Proto files were split into their own library, and TypeScript types are generated with
  ts-rs.
- Flows landed in three parts: first dividing by tos and IP, then deciding by target WAN IP.
- How DNS marks are stored in eBPF was changed.
- Connection metric reporting, a metric service and API, and a connection metrics page.
- System info shows memory, swap, CPU usage, and load average.
- XPS/RPS settings.
- A database crate and a default database path.
- The MSS clamp eBPF program with a UI entry point, removing the MSS clamping script check
  from pppd.
- DoH and DoT configuration in DNS.
- Fixes: deleting flow records, missing record returns, DNS redirect returning mismatched
  records, invalid access on arrch64, a memcpy compile failure on Arch Linux, memory
  alignment of the timer struct, the `CloudFlareMode` enum, and a DHCPv4 ClientIdentifier
  option error.
- The firewall function inside NAT was removed.
  :::

## v0.4.2-beta (2025-03-13) <Badge type="warning" text="pre-release" />

A DHCPv6 client and ICMPv6 RS/RA arrive, and IPv6 PD gets service pages.

::: details Detailed changes

- A DHCPv6 client.
- ICMPv6 RS/RA support, with editable RA flags.
- IPv6 PD testing was completed, and IPv6 PD and ICMPv6 PD got service pages.
- A unified ServiceManager was abstracted, with service state represented by WatchService.
- An interface address observer.
- Logs can be written to a file.
- Fixed the DHCP server missing options and the ICMPv6 RA service not stopping.
- NAT temporarily allows port ranges.
- Some error and parameter messages were reworded.
  :::

## v0.3.3 (2025-02-27)

Static IP configuration, forwarding the router own listening ports in NAT mode, and docs and CI.

::: details Detailed changes

- Static IP configuration.
- NAT mode can forward the router's own listening ports directly.
- How DNS caching and IP marking work was adjusted, in two parts.
- Fixed DNS failing to write the marker record, and empty hostnames in DHCP requests.
- A documentation page and docs CI, plus a CI config file.
  :::

## v0.3.0 (2025-02-18)

The project is reorganized, BPF maps are split by prefix, and Docker image management and domain-to-IP resolution arrive.

::: details Detailed changes

- The project directory structure was reorganized and build scripts improved.
- BPF maps are split into separate spaces by prefix.
- A Docker image base manager.
- The i18n foundation.
- Initialization from a pre-configured file.
- Resolving a chosen domain name to a specific IP.
- curl triggers were replaced by a link observer.
- NAT static mappings.
- Fixed DHCP client and server IP responses.
- Styling adjustments for some components.
- Known limitation: some TProxy servers do not work with `bpf_sk_assign` on Linux 6.1.y
  because `SO_REUSEPORT` is enabled.
  :::

## 0.2.0-beta (2025-01-08) <Badge type="warning" text="pre-release" />

The first release.

---

For full details on every tag, see the
[landscape releases](https://github.com/ThisSeanZhang/landscape/releases).
