# CLAUDE.md

Guidance for Claude Code sessions in this repo. Read this first, every session.

## What this is
**wirescape** (working name) is a local desktop app that captures live network traffic and renders it as ambient visual "scenes". The visuals are built from **real decoded content** (DNS names, TLS SNI, HTTP hosts, printable strings), and **protocol, direction and lifecycle** drive how that content looks and moves. Think "watching the wire", not a packet table.

## Current status
- **Phase 0:** stack decided, with no code yet.
- [ADR-0002](docs/decisions/ADR-0002-language-and-stack.md) is **Accepted**: **Windows only, Go + gopacket/gopacket + Ebitengine v2.** Next comes the scaffold (module, tooling, hello-world window and spike), which needs Go installed on the dev machine.

## Read order
1. `docs/00-concept.md` covers the why, the design pillars and scope.
2. `docs/01-architecture.md` covers the pipeline, the Scene contract and the perf budget.
3. `docs/02-event-model.md` covers the event types and the **shared signal → visual mapping**.
4. `docs/03-scenes.md` is the scene catalog and specs.
5. `docs/04-roadmap.md` lists phases and acceptance criteria. Work the current phase only.
6. `docs/05-privacy-and-safety.md` lists the **hard rules**.
7. `docs/decisions/` holds ADRs; `docs/devlog/` holds session history.

## Hard rules
- **Passive only.** Never write code that sends, injects or modifies packets.
- **No payload bytes in logs or on disk.** See the data handling rules in `docs/05-privacy-and-safety.md`. This includes test fixtures, snapshots and devlog entries.
- **Never commit real captures.** Only synthetic or scrubbed files go in `samples/`.
- **Everything must run in replay or demo mode without elevation.** Tests use `--replay samples/...` or `--demo`, never live capture.
- **Never run live capture or elevate privileges (sudo, setcap, admin) yourself.** Ask the user to do it.
- **Keep layers decoupled.** Scenes only see `Event` and `WorldState`. Capture must never block on downstream stages; drop and count instead.
- **Stay on the shared mapping.** New scenes reuse the signal → visual vocabulary in `02-event-model.md` unless an ADR changes it.

## Architectural decisions: research first, the user decides
- **The user makes every major architectural decision.** This covers language/stack, core libraries, threading or concurrency model, data structures that are hard to change, the event model or Scene contract, persistence or storage, packaging and distribution, platform support, and dropping or reshaping a feature. Never pick one silently or "for now"; stop and ask.
- **Research before presenting options.** Before asking, do and write down initial research. Check current versions, release dates and maintenance status (recent commits, open issues, last release). Check license and platform support, especially Windows/Npcap. Note known pitfalls. Check fit against the perf budget in `01-architecture.md` and the hard rules above. Don't rely on general knowledge alone; verify it against current sources and cite them.
- **Present 2–4 options with trade-offs and a recommendation**, then wait for the user's choice. Record the research, options and outcome in an ADR (status **Proposed** until the user confirms, then **Accepted**).
- Small, easy-to-reverse implementation details (local naming, helper structure, test layout) don't need this. When unsure whether something counts as major, ask.

## How to work here
- Work in small, testable steps that map to roadmap checkboxes.
- Before implementing a scene, write or confirm its mapping section in `docs/03-scenes.md` and get the user's OK. `/new-scene <name>` walks through this.
- When a non-trivial choice comes up (a library, a data structure that's hard to change, a dropped feature), record it as an ADR from `docs/decisions/ADR-template.md`.
- End each session with `/devlog`. It writes `docs/devlog/<date>-<slug>.md` and ticks roadmap boxes.
- Parsing untrusted bytes: bounds-check everything, never panic or crash on malformed packets, and fuzz parsers once the stack supports it.

## Commands
_TBD in Phase 0, once ADR-0002 is accepted. Fill in:_
- Build:
- Run (demo): `... --demo`
- Run (replay): `... --replay samples/dns-and-tls.pcapng`
- Run (live, needs capture rights): `... --iface <name>`
- Test:
- Lint / format:

## Definition of done (any change)
- Builds and passes tests with no elevation.
- Matches the relevant roadmap acceptance criteria.
- Docs updated if behavior, events or mappings changed.
- No payload bytes or real hostnames/IPs in code, tests, logs or docs.
