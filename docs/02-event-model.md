# 02 — Event model and signal mapping

This is the contract between the pipeline and the scenes. Keep it small. Scenes should be able to do everything they need from these types plus `WorldState`.

## TrafficEvent (one per decoded packet)

| Field | Type | Notes |
|---|---|---|
| `ts` | timestamp | Capture timestamp, not render time |
| `direction` | `In \| Out \| Local \| Multicast` | From local interface addresses |
| `l4` | `Tcp \| Udp \| Icmp \| Other` | |
| `app` | `Dns \| Mdns \| Llmnr \| Tls \| Quic \| Http \| Ssh \| Ssdp \| NetBios \| Arp \| Icmp \| Unknown` | Best-effort; port guess as a fallback |
| `local` / `remote` | addr + port | `remote` is the far end, whatever the direction |
| `flow_id` | u64 | Stable hash of the 5-tuple, direction-independent |
| `bytes` | u32 | Wire length |
| `tcp_flags` | optional bitset | SYN/ACK/FIN/RST/PSH |
| `encrypted` | bool | TLS/QUIC/SSH, or a high-entropy payload on an unknown port |
| `labels` | list of `Label` | Zero or more readable strings from this packet |
| `payload_preview` | optional, ≤ 64 bytes | **In-memory only, never logged or persisted.** Used by scenes that show hex or ASCII. Empty in privacy mode. |

### Label

| Field | Type | Example |
|---|---|---|
| `kind` | `DnsQuery \| DnsAnswer \| Sni \| HttpHost \| HttpRequestLine \| UserAgent \| DeviceName \| Printable` | `Sni` |
| `text` | string (sanitized: printable, length-capped at 128) | `login.microsoftonline.com` |

## FlowEvent (from the aggregator)

`Opened(flow_id)`, `Established(flow_id, rtt_ms)`, `Closed(flow_id)`, `Reset(flow_id)`, `Idle(flow_id)`.

## WorldState (read-only snapshot per frame)

- `flows`: lookup by `flow_id`, giving lifecycle, bytes in/out, label, protocol and process
- `hosts`: per remote host, giving resolved name, totals and protocols
- `dns_cache`: IP → name
- `rates`: overall and per-protocol pps/bps over 1 s and 10 s windows
- `local_addrs`, `iface_name`, `drop_count`

## Signal → visual mapping (shared vocabulary)

Scenes don't have to use all of these, but when a scene does express a signal, it should use the shared mapping so users only learn it once.

| Signal | Source | Default visual meaning |
|---|---|---|
| Direction | `direction` | **Out** rises or moves left→right. **In** falls or moves right→left. Multicast drifts or fogs. |
| Protocol | `app` | Color from the theme palette (below), plus shape or species per scene |
| Volume | `bytes`, rates | Size, brightness, height, density |
| Latency | `rtt_ms` | Distance, orbit radius, delay |
| Lifecycle | `FlowEvent` | Opened = spawn/birth. Established = solidify. Closed = graceful exit. Reset = pop, flash or glitch. |
| Readable content | `labels` | Text that actually appears: signs, leaves, falling strings |
| Encrypted vs plaintext | `encrypted` | Encrypted is dim, tinted or hex. Plaintext is bright, clear, and briefly highlighted. |
| Owning process | `WorldState.flows[].process` | Grouping: district, column cluster, school |

### Default protocol palette (theme can override)

| App | Role |
|---|---|
| DNS / mDNS / LLMNR | cyan |
| TLS | green |
| QUIC | teal |
| HTTP (plaintext) | white / amber highlight |
| SSH | violet |
| SSDP / NetBIOS / ARP (LAN chatter) | grey-blue |
| ICMP | yellow |
| Unknown | dim grey |

The palette must stay distinguishable for common color-vision deficiencies. Scenes should never rely on color alone; pair it with motion or shape.
