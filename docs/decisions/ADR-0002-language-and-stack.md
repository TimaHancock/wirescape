# ADR-0002 — Language and stack

- **Status:** Accepted (2026-10-13)
- **Date:** 2026-10-13 (research verified against crates.io, the Go proxy, NuGet, PyPI and GitHub on this date)

## Context
We need four things: packet capture (libpcap/Npcap), fast protocol parsing, a 60 fps 2D renderer that can draw thousands of glyphs per frame, and threads with bounded channels that support non-blocking drop-and-count. Windows is likely the primary OS (to be confirmed). Replay and demo must work without Npcap or elevation where possible.

Dev machine (2026-10-13): Windows 11, Npcap runtime installed, Python 3.11, git. **No Rust, Go or .NET toolchain installed yet.**

### Findings that apply to every option
- **TLS SNI needs light TCP reassembly.** Chrome and Firefox now send post-quantum key shares (X25519MLKEM768) by default, which pushes ClientHello to about 1.8 KB, so it often spans 2 TCP segments. Every stack needs a small per-flow buffer of the first few KB, or it will miss SNI. Neither gopacket nor tls-parser does this for us.
- **QUIC SNI** stays a stretch goal, because it sits inside protected Initial packets.
- Some capture libraries expose send/inject APIs. Rule: **we never call them** (passive only).

## Options

### A. Rust
| Role | Crate | Latest | Health |
|---|---|---|---|
| Live capture | `pcap` | 2.5.0 (2026-08-15) | Active. **Windows build needs the Npcap SDK** (on `LIB`) today; unreleased main loads `wpcap.dll` at runtime and removes this. |
| Replay | `pcap-parser` | 0.17.0 (2025-07-25) | Pure Rust, stable, low activity. Replay needs no Npcap. |
| L2–L4 | `etherparse` | 0.21.0 (2026-07-21) | Active, zero-alloc |
| DNS | `simple-dns` | 0.12.0 (2026-07-26) | Active, 0 open issues |
| SNI | hand-written (~100 lines) | — | `tls-parser` 0.12.2 (2024-09) is slow-moving; keep it as a test oracle only |
| Render | `wgpu` 30 + `glyphon` 0.12 | 2026-08 / 2026-07 | Best text throughput (one atlas, one draw). More boilerplate. |
| Render (alt) | `macroquad` | 0.4.16 (2026-07-30) | Fastest to prototype. One maintainer, 347 open issues, some open text bugs. |
| Channels | `crossbeam-channel` | 0.5.17 (2026-09-05) | `try_send` → `Full`, plus `len()` for metrics |

- **Pros:** most performance headroom, with no GC. The best text-rendering path available (wgpu+glyphon). Strong typed parsing.
- **Cons:** steepest learning curve and slow compiles. Today it needs the Npcap SDK at build time. **cargo-fuzz doesn't work on native Windows**, so fuzzing would run in WSL2 or Linux CI, with proptest locally. Its renderer choice is a trade-off: macroquad is quick but carries bus-factor risk, while wgpu is solid but more work.
- **Avoid:** `pnet` (stale, needs SDK), `dns-parser` (abandoned since 2018), `notan` (low activity).

### B. Go
| Role | Library | Latest | Health |
|---|---|---|---|
| Capture, replay, L2–L7 | `github.com/gopacket/gopacket` (community fork) | v1.7.4 (2026-10-01) | Active, 8 open issues, BSD-3. v1.7.x turned crafted-input decoder panics into errors. |
| Render | Ebitengine | v2.10.5 (2026-10-06) | Very active. text/v2 uses glyph atlases and batched draws. |
| Channels | built in | — | Buffered chan + `select { default: }` = drop-and-count |

- **Windows capture without cgo or the SDK:** `pcap_windows.go` loads `wpcap.dll` at runtime. It cross-compiles to a single exe, and replay (`pcapgo`, which reads pcapng) needs no Npcap.
- **Pros:** the simplest Windows toolchain. Single-binary output. DNS and TLS layers are built in. Native `go test -fuzz` works on Windows, so we can fuzz locally. Fast to learn.
- **Cons:** GC means we must watch allocation churn (use `DecodingLayerParser` and pooled buffers, never `NewPacket` per packet). The built-in SNI parse is minimal (single segment, first entry), so we'd likely write our own as in Rust. Ebitengine's text throughput with many distinct strings is unbenchmarked.
- **Avoid:** `github.com/google/gopacket` (last tag 2020, effectively abandoned).
- **Not yet verified:** the claim that Ebitengine needs no cgo on Windows. Check it during the scaffold spike.

### C. C# / .NET
- **Capture:** SharpPcap 6.3.1 (2025-06), active, MIT. It reads pcapng through libpcap, so **replay also needs the Npcap DLL**.
- **Parsing:** PacketDotNet 1.4.8 (2025-04) is slow-moving and has **no DNS or TLS parser**, so we'd hand-write both.
- **Render:** Raylib-cs 8.1.0 (2026-09), or MonoGame 3.8.6 (2026-10) with FontStashSharp.
- **Pros:** excellent Windows tooling.
- **Cons:** more parsers to hand-write. NativeAOT single-file is possible but fiddly (native DLLs sit beside the exe, plus trimming warnings). Less natural on Linux/macOS.

### D. Python
- **Libraries:** scapy 2.8.0 is active but **GPL-2.0** and runs at a few thousand pps per core, so it can't meet the 10k pps budget. dpkt is stale (2022). pygame-ce is active. arcade has had no release in 12 months.
- **Cons:** the GIL contends across the capture, parse and render threads. PyInstaller onefile is slow to start and often trips antivirus.
- **Verdict:** OK for a throwaway visual spike. Not viable for the real app.

## Recommendation
Rust and Go both fit, and the research narrowed the gap that the original draft assumed.

- **Recommended: Go (gopacket fork + Ebitengine)** if Windows is primary. It has the least toolchain friction on Windows (no SDK, no cgo), produces a single exe, fuzzes natively on Windows, and gets us to the first visible scene fastest.
- **Choose Rust (pcap + pcap-parser + etherparse + simple-dns + wgpu/glyphon + crossbeam)** if you want maximum performance headroom and the best text renderer, or you already know or want to learn Rust. Accept the SDK step and WSL for fuzzing.
- Prior familiarity with either language should outweigh these margins.

## Decision
Decided with the user on 2026-10-13:
- **Target OS:** Windows only. Keep code portable (no gratuitous Windows-only APIs outside the capture and platform layer), but don't test or package other OSes yet.
- **Stack:** **Go** with `github.com/gopacket/gopacket` (capture via runtime-loaded Npcap, `pcapgo` for replay, layers for L2–L4 and DNS), a hand-written bounds-checked SNI parser with small per-flow TCP prefix reassembly, **Ebitengine v2** for rendering, and buffered channels with `select default` for drop-and-count.
- The user is new to both Go and Rust. Go's gentler learning curve was a deciding factor.

## Consequences
- Builds and cross-compiles without cgo or the Npcap SDK. Live capture still needs the Npcap runtime (already installed on the dev machine).
- Replay and demo run with no Npcap and no elevation.
- We must keep allocations low on the hot path: `DecodingLayerParser`, pooled buffers, no per-packet `gopacket.NewPacket`. Profile with `pprof` against `--speed 10` replays.
- Parsers are fuzzed with native `go test -fuzz` on Windows.
- To verify in the Phase 0 spike: Ebitengine builds without cgo on Windows, and text/v2 throughput with thousands of distinct glyph strings per frame.
- Linux/macOS support would be a new ADR.
