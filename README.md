# wirescape

*Working name.* A local desktop app for **watching the wire**: your machine's live network traffic rendered as ambient, readable visual scenes.

Most "Matrix rain" network toys use random glyphs that pulse with traffic volume. Here, **the characters are the traffic**. DNS names, TLS SNI hostnames, HTTP hosts and printable strings fall, flip and glow on screen, while protocol, direction and connection lifecycle decide how they move and look.

> **Status:** Phase 0, concept and design docs only. No code yet.

## Planned scenes
| Scene | What you see |
|---|---|
| **Wire Rain** | Decoded hostnames and strings rain down (inbound) or rise (outbound); encrypted bytes fall as dim hex, and plaintext flashes bright |
| **Departures Board** | Split-flap board of connections: destination, protocol, port, and status from BOARDING to IN FLIGHT to LANDED/CANCELLED |
| **Night Skyline** | Remote hosts as buildings, with windows lighting up per packet and traffic as cars on a two-lane highway |
| **DNS Tree** | The domain hierarchy growing live as names resolve |

More ideas are in [docs/03-scenes.md](docs/03-scenes.md).

## Principles
- **Local and passive.** One native app, no web server, never sends or modifies traffic.
- **Private by default.** Nothing is saved to disk, and a privacy mode masks names and IPs for screenshots.
- **Replayable.** Everything works from `.pcap` files or a demo generator without admin rights.

## Docs
| Doc | Covers |
|---|---|
| [00-concept](docs/00-concept.md) | Vision, pillars, prior art, scope |
| [01-architecture](docs/01-architecture.md) | Pipeline, Scene contract, performance budget, platform notes |
| [02-event-model](docs/02-event-model.md) | Event types and the shared signal → visual mapping |
| [03-scenes](docs/03-scenes.md) | Scene catalog and specs |
| [04-roadmap](docs/04-roadmap.md) | Phases and acceptance criteria |
| [05-privacy-and-safety](docs/05-privacy-and-safety.md) | Use boundaries and data handling rules |
| [decisions/](docs/decisions/) | Architecture decision records |
| [devlog/](docs/devlog/) | Session-by-session build log |

## Requirements (planned)
- Capture driver: **Npcap** (Windows) or **libpcap** (Linux/macOS)
- Live capture needs capture rights. Replay and demo modes don't.
- Only use live capture on machines and networks you own or are authorized to monitor.

## Working with Claude Code
`CLAUDE.md` holds session rules. Project slash commands:
- `/new-scene <name>`: spec, then implement, then verify a new scene
- `/devlog [title]`: write the session's devlog entry and tick roadmap boxes
