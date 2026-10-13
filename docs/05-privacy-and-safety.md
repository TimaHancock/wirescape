# 05 — Privacy and safety

This app looks at network traffic, so the guardrails are part of the design, not an afterthought.

## Use boundaries
- **Run it on machines and networks you own or are authorized to monitor.** On a work or client network, capturing other people's traffic usually needs explicit authorization even if you're the admin.
- The app is **passive and read-only**. It never sends, injects or modifies packets. Code that writes to the network is out of scope; don't add it.
- On a switched network the app normally sees only the local host's traffic plus broadcast and multicast. Promiscuous mode stays **off by default**.

## Data handling rules (enforced in code review)
1. **Nothing persists by default.** No packet bytes, labels or flow logs go to disk unless the user turns on an explicit, clearly named option.
2. **Payload bytes never go to logs.** Debug and trace logging may include protocol, sizes, ports and flags. It must never include `payload_preview` or label text unless a dedicated `--log-content` flag is set.
3. `payload_preview` is capped at 64 bytes and lives only in memory, discarded once the visual is spawned.
4. `--emit jsonl` outputs metadata and labels only, never payload bytes.
5. Reference captures in `samples/` must be synthetic or scrubbed. Real captures stay out of git (`.gitignore` covers `*.pcap*` outside `samples/`).

## Privacy mode (`P` hotkey / `--privacy`)
For screenshots, streams and demos:
- Hostnames and labels are replaced with stable pseudonyms per session, e.g. `host-7f3a.example`. This keeps structure (the same host keeps the same name) while hiding identity.
- IP addresses are masked to the /16 for IPv4, or the /32 prefix for IPv6.
- `payload_preview` is emptied, so hex rain becomes random hex.
- Process names are hidden.
- The HUD shows a visible "PRIVACY" badge, so it's obvious which mode is on screen.

## Least privilege
- Prefer granting capture capability to the binary over running the whole app elevated (see platform notes in [01-architecture.md](01-architecture.md#platform-notes)).
- If the app must start elevated, drop privileges after opening the capture handle where the platform allows it.
- Replay and demo modes never require elevation.
