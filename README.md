# munin-plugins-my
A set of Munin (http://munin-monitoring.org/) plugins for OpenConnect, I2Pd, XRay, Hysteria2, QBittorrent, Snowflake, etc.

# I2Pd
Munin plugin for I2Pd (https://github.com/PurpleI2P/i2pd)

Filename: i2pd_

Available symlinks: i2pd_traffic, i2pd_routers, i2pd_successrate, i2pd_status, i2pd_transits

Parameters: env.host (localhost), env.port (7650), env.password (itoopie)

![Routers](docs/i2pd_routers-day.png?raw=true "Routers")

![Status](docs/i2pd_status-day.png?raw=true "Status")

![Success rate](docs/i2pd_successrate-day.png?raw=true "Success rate")

![Traffic](docs/i2pd_traffic-day.png?raw=true "Traffic")

![Transit tunnels](docs/i2pd_transits-day.png?raw=true "Transit tunnels")

# OpenConnect VPN server
Munin plugin for OpenConnect VPN server (OCsev - https://gitlab.com/openconnect/ocserv)

Files: ocserv_, ocserv_users_ ocserv_vhosts

Available symlinks (from ocserv_): ocserv_traffic, ocserv_sessions, ocserv_bans

Parameters (ocserv_users): env.ocpasswd (/etc/ocserv/ocpasswd)

Parameters (ocserv_vhosts): env.config (/etc/ocserv/ocserv.conf)

Dependencies: jq, occtl

![Sessions](docs/ocserv_sessions-day.png?raw=true "Sessions")

![Sessions per vhost](docs/ocserv_sessions_vhosts-day.png?raw=true "Sessions per vhost")

![Sessions per user](docs/ocserv_users-day.png?raw=true "Sessions per user")

![Traffic](docs/ocserv_traffic-day.png?raw=true "Traffic")

Due to OCServ architecture specifics, it's not possible to retrieve real-time traffic data, so only the total traffic for each finished session is displayed.

# Snowflake (Tor pluggable transport)

Munin plugin for Tor pluggable transport named Snowflake (https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/snowflake)

Files: snowflake_

Available symlinks: snowflake_sessions, snowflake_traffic

Parameters: env.logfile (/var/log/snowflake.log)

![Sessions](docs/snowflake_sessions-day.png?raw=true "Sessions")

![Traffic](docs/snowflake_traffic-day.png?raw=true "Traffic")

# qBittorrent-nox

Munin plugin for qBittorrent-nox (https://github.com/qbittorrent/qBittorrent)

File: qbt_

Available symlinks: qbt_traffic, qbt_transfers

Parameters: env.host (localhost), env.port (55556)

![Transfers](docs/qbt_transfers-day.png?raw=true "Transfers (day)")

![Transfers](docs/qbt_transfers-week.png?raw=true "Transfers (week)")

![Traffic](docs/qbt_traffic-day.png?raw=true "Traffic")

# XRay-core

Munin plugin for XRay-core (https://github.com/XTLS/Xray-core). After some adaptation could also work with V2Ray-core.

File: xray_

Available symlinks: xray_user, xray_inbound, xray_outbound

Parameters: env.server (127.0.0.1:5002), env.config (/opt/xray/config.json), env.exclude_users (""), env.exclude_inbounds ("api"), env.exclude_outbounds ("block dns")

The plugin reads stats from Xray's metrics handler over HTTP (`http://${server}/debug/vars`), so you need a `metrics` inbound configured in Xray, e.g.:

```json
"metrics": { "tag": "metrics_out" },
"inbounds": [
  { "listen": "127.0.0.1", "port": 5002, "protocol": "dokodemo-door", "settings": { "address": "127.0.0.1" }, "tag": "metrics_in" }
],
"routing": { "rules": [ { "type": "field", "inboundTag": ["metrics_in"], "outboundTag": "metrics_out" } ] }
```

The `exclude_*` parameters accept space-separated substrings; any tag matching one of them is filtered out of the graph.

Dependencies: jq, md5sum, curl

![Inbounds traffic](docs/xray_inbound-day.png?raw=true "Inbounds traffic")

![Users traffic](docs/xray_user-day.png?raw=true "Users traffic")

# XRay Observatory

Companion Munin plugin that exposes data from Xray's Observatory subsystem (health and latency of probed outbounds). Independent from `xray_` - different subtree of the same `/debug/vars` endpoint.

File: xray_observe_

Available symlinks: xray_observe_alive, xray_observe_delay

Parameters: env.server (127.0.0.1:5002), env.config (/opt/xray/config.json)

Requires an `observatory` block in your Xray config with a `subjectSelector` listing the outbound tags to probe. The plugin uses those tag names directly as graph field names.

Dependencies: jq, curl

# Hysteria2

Munin plugin for Hysteria2 server (https://github.com/apernet/hysteria) - graphs traffic per user via the Traffic Stats API.

File: hyst_

Available symlinks: hyst_user

Parameters: env.server (127.0.0.1:9999), env.config (/opt/hy2_server.yaml), env.secret (required), env.exclude_users ("")

Requires the Traffic Stats API to be enabled in the Hysteria2 server config, e.g.:

```yaml
trafficStats:
  listen: 127.0.0.1:9999
  secret: your_random_secret_here
```

`env.secret` must match `trafficStats.secret` from your Hysteria2 config. The plugin reads the user list from the `auth.userpass` map in the same config file (so the YAML auth backend is assumed). Usernames are used verbatim as Munin field names, so they must match `[A-Za-z0-9_]` and not start with a digit.

The `exclude_users` parameter accepts a space-separated list of substrings; any username matching one of them is filtered out of the graph.

Dependencies: yq, jq, curl

# Network interface (if_)

A minimal `if_` clone that reads byte counters from `/sys/class/net/<iface>/statistics/`.

File: if_

Available symlinks: one per interface, e.g. `if_eth0`, `if_wg0`. `munin-node-configure --suggest` will auto-detect them (loopback is skipped).

Originally written as a workaround because [munin-node-c](https://github.com/munin-monitoring/munin-c) shipped without an `if_` plugin. That gap is now gone - I contributed a C implementation upstream in [munin-c#86](https://github.com/munin-monitoring/munin-c/pull/86) — so on systems running munin-node-c you should prefer the upstream plugin. This shell version is kept here for hosts that use an old munin-node-c version (or for quick testing).



