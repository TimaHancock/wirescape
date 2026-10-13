# samples/

Small reference captures for tests, replay and demos. Every feature must work against these **without admin rights**.

## Rules
- **Only synthetic or scrubbed captures are committed here.** Never commit a raw capture from a real machine or network.
- Keep each file small (under ~2 MB).
- Document each file in the table below.

## Suggested reference set
| File | Contents | Used for |
|---|---|---|
| `dns-and-tls.pcapng` | A handful of DNS lookups, each followed by a TLS ClientHello carrying SNI | Label extraction, DNS-cache naming |
| `plain-http.pcapng` | Plain HTTP/1.1 requests to a local test server | HTTP Host and request-line labels |
| `lan-chatter.pcapng` | ARP, mDNS, SSDP, LLMNR | Multicast/broadcast visuals |
| `burst.pcapng` | Thousands of small UDP packets in a few seconds | Backpressure and drop counting |

## How to make them safely
- Generate traffic against a local test VM or a lab network, not your daily-driver browsing.
- Alternatively, generate packets programmatically (e.g., scapy writing a pcap) so nothing real is ever captured.
- Use Wireshark/`editcap`/`tcprewrite` to rewrite IPs, then check the file in Wireshark before committing.
