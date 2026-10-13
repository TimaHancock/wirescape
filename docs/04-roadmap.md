# 04 — Roadmap

Phases are sequential. Each has acceptance criteria that must pass before moving on. Tick boxes as work lands, and record anything surprising in a devlog entry.

## Phase 0 — Decide and scaffold
- [x] Settle language and stack: finish and accept [ADR-0002](decisions/ADR-0002-language-and-stack.md)
- [ ] Initialize the repo, build tooling, formatter, linter and test runner
- [ ] Fill in the **Commands** section of `CLAUDE.md`
- [ ] Capture or source 2–3 small reference `.pcap` files into `samples/` (see `samples/README.md`)

**Done when:** a hello-world binary builds and runs on the user's main OS, and `CLAUDE.md` lists real build, test and run commands.

## Phase 1 — Capture and decode to console
- [ ] `--list-ifaces`, `--iface`, `--filter`, `--replay <file>`, `--demo`
- [ ] L2–L4 decode and direction classification
- [ ] DNS, TLS SNI, HTTP Host decoding, plus printable-run extraction
- [ ] `--emit jsonl` prints `TrafficEvent`s (labels and metadata only, with no payload bytes)
- [ ] Unit tests against the reference pcaps

**Done when:** replaying a sample shows the expected DNS names and SNI hostnames, live capture shows names while browsing, and tests pass without admin rights.

## Phase 2 — Aggregator
- [ ] Flow table with lifecycle and idle eviction
- [ ] RTT from the handshake
- [ ] DNS cache (IP → name) applied to unlabeled flows
- [ ] Rolling rates
- [ ] Optional process attribution behind a flag

**Done when:** `--emit jsonl` also shows `FlowEvent`s, and a TLS flow without SNI gets a name from the DNS cache.

## Phase 3 — Renderer shell and Wire Rain
- [ ] Window, 60 fps loop, fullscreen toggle, HUD
- [ ] Scene registry and Scene contract
- [ ] Wire Rain scene per [03-scenes.md §1](03-scenes.md#1-wire-rain-the-seed-concept)
- [ ] Backpressure: drop and count, shown in the HUD

**Done when:** the budget in [01-architecture.md](01-architecture.md#backpressure-and-performance-budget) holds on a replay at `--speed 10`, and labels are readable as they fall.

## Phase 4 — Departures Board and scene switching
- [ ] Departures Board scene
- [ ] Hotkeys: `1`–`9` select a scene, `Tab` cycles, `H` toggles HUD, `P` toggles privacy mode, `F` toggles fullscreen, `Space` pauses
- [ ] Config file

**Done when:** you can switch scenes live without dropping capture.

## Phase 5 — Skyline and DNS Tree
- [ ] Night Skyline
- [ ] DNS Tree

## Phase 6 — Polish
- [ ] Privacy mode fully implemented (see [05-privacy-and-safety.md](05-privacy-and-safety.md))
- [ ] Themes
- [ ] Record a GIF or video of each scene for the README
- [ ] Packaging for the main OS, including Npcap guidance for Windows

## Stretch
- Hilbert IP map, Sequencer with audio, Solar System
- QUIC SNI extraction
- Screensaver mode
- A "Wireshark it" key that writes the last N seconds to a pcap. It must be explicit and opt-in.
