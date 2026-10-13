Scaffold a new visual scene named: $ARGUMENTS

1. Read `docs/02-event-model.md` (shared signal mapping) and `docs/03-scenes.md`.
2. If the scene isn't in the catalog yet, add a row and write its mapping section first: direction, protocol, volume, lifecycle, readable content, encrypted vs plaintext, and how it handles overload. Show me the mapping and wait for my OK before writing code.
3. Implement it against the Scene contract in `docs/01-architecture.md`. Keep it in its own module and register it in the scene registry.
4. The scene must only read `Event` and `WorldState`, never capture or OS APIs.
5. Verify it with `--demo` and at least one reference pcap from `samples/` at `--speed 10`, checking the HUD drop count and frame rate.
6. Update the scene's status in `docs/03-scenes.md`, and remind me to run `/devlog`.
