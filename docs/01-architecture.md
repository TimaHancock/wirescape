# 01 — Architecture

## Pipeline

```
 NIC / .pcap file
       │
       ▼
┌──────────────┐  raw frames   ┌──────────────┐  TrafficEvent  ┌──────────────┐  WorldState + events  ┌──────────────┐
│   Capture    │ ────────────▶ │    Decode    │ ─────────────▶ │  Aggregate   │ ─────────────────────▶ │   Renderer   │
│ live | replay│  (bounded ch) │ L2–L4 + apps │  (bounded ch)  │ flows, DNS,  │   (per-frame snapshot) │  active Scene│
└──────────────┘               └──────────────┘                │ hosts, rates │                        └──────────────┘
                                                               └──────────────┘
```

Each stage runs on its own thread. Stages talk through **bounded channels**. The renderer is the only thing on the main/UI thread.

## Stages

### Capture
- **Live:** libpcap on Linux/macOS, Npcap on Windows. The interface is chosen by `--iface`, or the default route interface if omitted. An optional BPF filter can be set with `--filter`.
- **Replay:** reads a `.pcap`/`.pcapng` at real-time speed, or `--speed N`. **Replay is a first-class mode.** It is how development, testing, demos and CI work without admin rights.
- **Synthetic:** `--demo` generates plausible fake traffic, so the app can be shown with zero capture at all.
- Emits `RawFrame { ts, linktype, bytes }`.

### Decode
- L2–L4: Ethernet/802.1Q, ARP, IPv4/IPv6, TCP, UDP, ICMP/ICMPv6.
- Application, best-effort and cheap:
  - **DNS** (UDP/TCP 53), **mDNS** (5353), **LLMNR** (5355): query names and answer records
  - **TLS ClientHello**: SNI hostname and ALPN (h2/http/1.1); marks the flow encrypted
  - **QUIC Initial**: flagged as QUIC/HTTP3. SNI extraction from QUIC is a stretch goal because the Initial packet is protected.
  - **HTTP/1.x**: request line, Host, User-Agent
  - **SSDP / NetBIOS-NS**: device and service names
  - **Everything else**: protocol is guessed by port, with printable ASCII runs (≥ 4 chars) extracted from the payload
- **Direction** is classified by comparing addresses against the capture host's interface addresses: `In`, `Out`, `Local` (both ends local), `Broadcast/Multicast`.
- Emits `TrafficEvent` (see [02-event-model.md](02-event-model.md)).

### Aggregate
Holds the slow-changing state that scenes need and that raw packets do not carry:
- **Flow table** keyed by 5-tuple: lifecycle (SYN → established → FIN/RST), byte and packet counts, first and last seen
- **RTT estimate** from the SYN → SYN/ACK gap
- **DNS cache**: IP → most recent name resolved through it. Many flows have no SNI, so this lets scenes label them.
- **Host table**: per remote host totals, protocols seen, and label
- **Rates**: rolling packets/sec and bytes/sec, overall and per protocol
- **Process attribution (optional)**: maps local sockets to process names via the OS socket table. It's best-effort, platform-specific and polled on an interval.
- Emits `FlowEvent`s (opened, established, closed, reset) alongside the passed-through `TrafficEvent`s, and exposes a read-only `WorldState` snapshot.

### Renderer
- Owns the window, the frame loop (target 60 fps) and input.
- Hosts exactly one active `Scene` at a time. Hotkeys switch scenes.
- Each frame:
  1. Drain up to `MAX_EVENTS_PER_FRAME` events and pass each to `scene.on_event()`.
  2. Call `scene.update(dt, &world)`.
  3. Call `scene.draw()`.
- Overlay HUD (toggle): interface, pps, bps, dropped-event count, active scene, privacy mode indicator.

## The Scene contract

Language-neutral shape. Translate it to whatever ADR-0002 picks.

```
trait Scene {
    fn name(&self) -> &str;
    fn on_event(&mut self, ev: &Event, world: &WorldState);   // cheap; spawn/queue visuals only
    fn update(&mut self, dt: f32, world: &WorldState);        // animate
    fn draw(&self, gfx: &mut Gfx);                            // render only, no state changes
    fn on_resize(&mut self, w: u32, h: u32);
}
```

Rules:
- Scenes **never** touch capture, decode or the OS. They only see `Event` and `WorldState`.
- Scenes must stay smooth at 10k events/sec. They do this by sampling, coalescing or budgeting spawns, never by blocking.
- Each scene lives in its own module or file and is registered in one place.

## Backpressure and performance budget

| Item | Target |
|---|---|
| Render | 60 fps on an integrated GPU at 1080p |
| Ingest | ≥ 10,000 packets/sec without UI stutter |
| Capture → screen latency | < 100 ms |
| Memory | Bounded; flow and host tables evict on idle timeout and an LRU cap |

When channels are full, the **decode → aggregate** and **aggregate → renderer** stages **drop and count** rather than block. Capture must never block on downstream stages. The HUD shows the drop count, so overload is visible instead of silent.

## Configuration

A single config file (format per ADR-0002, probably TOML), with CLI flags overriding it. It covers interface, BPF filter, default scene, theme, privacy mode, and per-scene options.

## Platform notes

- **Windows:** needs Npcap installed. Linking against it usually requires the Npcap SDK. Live capture needs admin unless Npcap was installed with admin-only access turned off. Check the Npcap license terms before redistributing anything that bundles it.
- **Linux:** run as root, or grant the binary `cap_net_raw,cap_net_admin` with `setcap` rather than running the whole app as root.
- **macOS:** BPF devices need permission (root, or the ChmodBPF approach Wireshark uses).
