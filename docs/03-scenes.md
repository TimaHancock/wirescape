# 03 — Scene catalog

Every scene consumes the same events (see [02-event-model.md](02-event-model.md)) and follows the shared signal mapping. Status values: **v1** = first release target, **next** = after v1, **idea** = parked.

| # | Scene | Status | Readable content | Build effort |
|---|---|---|---|---|
| 1 | Wire Rain | v1 | High | Low |
| 2 | Departures Board | v1 | High | Low–Med |
| 3 | Night Skyline | next | Med | Med |
| 4 | DNS Tree | next | High | Med |
| 5 | Hilbert IP Map | next | Low | Med |
| 6 | Solar System | idea | Low | Med |
| 7 | Radar Sweep | idea | Low | Low |
| 8 | Aquarium | idea | Low | High |
| 9 | Weather | idea | Low | High |
| 10 | Plinko | idea | Low | Med |
| 11 | Oscilloscope | idea | None | Low |
| 12 | Sequencer (with audio) | idea | None | Med |

---

## 1. Wire Rain (the seed concept)

Columns of falling text where **the characters are the traffic**.

- **Out** strings rise from the bottom and **In** strings fall from the top. Multicast drifts sideways at low opacity.
- **Labels** (DNS names, SNI, HTTP Host) fall as whole words. They flash white on spawn, then settle into the protocol color.
- **Encrypted payload** falls as dim hex pairs drawn from `payload_preview`, or random hex in privacy mode.
- **Plaintext printable runs** fall bright, as the "money shot" moments.
- Column speed scales with packet size. Column density scales with rate.
- **Lifecycle:** a TCP RST glitches the column (a brief horizontal jitter plus a red flash).
- **Overload:** above the spawn budget, sample rather than queue; labels get priority over hex.

## 2. Departures Board

A split-flap airport board.

- Two panels: **Departures** (outbound connections) and **Arrivals** (inbound-initiated).
- Row columns: time · destination (DNS name, SNI, or IP) · "airline" (protocol) · gate (remote port) · status.
- Status follows the flow lifecycle: `BOARDING` (SYN) → `IN FLIGHT` (established) → `LANDED` (FIN) / `CANCELLED` (RST) / `DELAYED` (RTT above threshold).
- New rows flip in letter by letter. Rows age out after idle.
- Optional: soft flap sound, off by default.

## 3. Night Skyline

- Each remote host or domain is a building. Height is the log of total bytes, and windows light up on packets.
- A highway at the bottom carries packets as cars, with Out in one lane and In in the other.
- Encrypted traffic gets tinted windows. Plaintext gets lit windows with the label readable on a sign or billboard.
- Process grouping places buildings into districts.

## 4. DNS Tree

- The domain hierarchy grows in real time, from root to TLD to registrable domain to subdomains.
- Branches grow when a name resolves, and leaves pulse when traffic hits that name (via the DNS cache).
- Over time it shows "everything this machine talks to". It's great for spotting one app calling forty subdomains.

## 5. Hilbert IP Map

- IPv4 space laid out on a Hilbert curve, one cell per /24. Cells glow on contact and decay over minutes.
- Clusters of cloud and CDN usage emerge. IPv6 gets a separate, simpler panel or is folded by prefix.

## 6–12. Parked ideas

- **Solar System:** the local host is the sun and remote hosts are planets. Orbit radius is RTT and size is volume. Packets are comets.
- **Radar Sweep:** hosts appear as blips by angle (IP hash or region) and distance (RTT). New hosts make a ping ripple.
- **Aquarium:** connections are creatures, with species by protocol. A handshake hatches, FIN swims off, and RST pops.
- **Weather:** DNS drizzle, download storms, ICMP lightning, LAN-chatter fog.
- **Plinko:** packets drop through pegs into port-range bins, building a live histogram.
- **Oscilloscope:** one trace per protocol, with packet size as amplitude.
- **Sequencer:** protocol is the instrument, port is the pitch, and In/Out map to stereo left/right.

## Adding a scene

Use `/new-scene <name>` in Claude Code. The checklist:
1. Add an entry to this table with its status.
2. Write a short mapping section here before coding.
3. Implement the Scene contract (see [01-architecture.md](01-architecture.md)).
4. Verify it with the reference replay captures in `samples/` and with `--demo`.
5. Add a screenshot or GIF to `docs/media/` and a devlog entry.
