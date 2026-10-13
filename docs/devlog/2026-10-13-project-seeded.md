# 2026-10-13 — Project seeded

**Phase:** 0 · **Session goal:** turn the idea into a documented seed folder for Claude Code sessions.

## What I set out to do
Capture the "watching the wire" concept: a local app that visualizes live NIC traffic, with visuals built from real decoded content and behavior driven by protocol and direction.

## What happened
- Checked prior art. Matrix-rain network toys exist (ByteFall, netrain, NetRain), but they use random or decorative glyphs. Rendering the decoded content itself looks like open ground.
- Brainstormed a dozen scene concepts and picked **Wire Rain** and **Departures Board** for v1, with **Skyline** and **DNS Tree** next.
- Dropped the browser front end in favor of a single local app ([ADR-0001](../decisions/ADR-0001-local-native-app.md)).

## Decisions
- ADR-0001 accepted: local native app.
- ADR-0002 proposed: Rust stack recommended, not yet accepted.

## Next session
- [ ] Decide ADR-0002 (language/stack, target OS)
- [ ] Phase 0 scaffold
- [ ] Gather reference pcaps
