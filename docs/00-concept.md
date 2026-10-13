# 00 — Concept

## One-liner

A local desktop app that captures live traffic from the machine's network interface and renders it as ambient, readable visual "scenes". The goal is that you are **watching the wire**, not reading a packet table.

## The problem it plays with

Wireshark makes traffic *legible* but not *glanceable*. Existing "Matrix rain" network toys make traffic *glanceable* but not *legible*, because their falling characters are random glyphs whose color or density reflects activity. This project sits between the two. The visuals are built from **real decoded content**: DNS names, TLS SNI hostnames, HTTP hosts and headers, and printable strings. **Protocol, direction and connection lifecycle** change how that content moves and looks.

## Design pillars

1. **Real content, not decoration.** If a character or label appears on screen, it came from the wire, or is an honest representation of encrypted bytes, such as dim hex.
2. **Behavior encodes meaning.** Protocol, direction, size, latency and lifecycle each map to a visual property. Someone who learns the mappings should be able to "read" the screen.
3. **Scenes, one engine.** One capture → decode → aggregate pipeline feeds many interchangeable visual scenes.
4. **Ambient by default.** It runs full-screen, unattended, and looks good while idle (background chatter) and under load (big downloads).
5. **Local and private.** There is no network service, no cloud, and nothing persisted by default. See [05-privacy-and-safety.md](05-privacy-and-safety.md).

## What makes it different (prior art, as of Oct 2026)

| Project | What it does | Gap this project fills |
|---|---|---|
| ByteFall (github.com/sanif/bytefall) | Matrix rain of outbound traffic, grouped and colored by domain and process | Rain color/intensity tracks volume; this project renders the decoded content itself |
| netrain (Rust crate, terminal UI) | Matrix-style pcap monitor with a packet log and threat hints | Rain columns are random characters |
| NetRain (Windows screensaver) | Code rain colored by TCP vs UDP | Protocol shown by color only; no content |
| wiregraph, net-glimpse | Host/edge graph visualizers | Topology view, not "watching the wire" |

These are small hobby projects; they are listed for inspiration, not as dependencies.

## Out of scope

- Packet injection, modification, or anything active on the network
- Deep forensic analysis. Wireshark already exists; we can link out to it.
- Decrypting TLS. Encrypted traffic is shown *as* encrypted.
- A web or browser front end. Decided in [ADR-0001](decisions/ADR-0001-local-native-app.md).

## Working name

`wirescape` is a placeholder. Check for name collisions before publishing anything.
