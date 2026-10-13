# ADR-0001 — Local native app, no web front end

- **Status:** Accepted
- **Date:** 2026-10-13

## Context
The first idea was a local capture backend streaming to a browser page over a WebSocket, because browsers can't read the NIC directly. That adds a local server, a port, and a second runtime, all for an app that is meant to sit full-screen on one machine.

## Decision
Build a single local desktop application that captures, decodes and renders in-process. There is no HTTP server, no WebSocket, and no browser.

## Consequences
- Simpler security story: no listening port and no cross-origin concerns.
- Rendering uses a native graphics library (picked in ADR-0002) instead of Canvas/WebGL.
- Sharing a live view remotely is not supported. Screenshots or recordings (with privacy mode) cover demos.
- `--emit jsonl` remains as a debug and integration seam, so a separate viewer could still be built later without changing this decision.
