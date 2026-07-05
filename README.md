# Lab 2 — Wireshark & Network Analysis

**Tools:** Wireshark (Free) · nslookup · tshark
**Environment:** Local Machine
**Category:** Network Analysis / Troubleshooting

---

## Overview

This lab covers packet-level network analysis using Wireshark — capturing live traffic, applying display filters, reading TCP handshakes, identifying DNS queries, and reconstructing full conversations from individual packets.

Every network issue an org faces — a service that's unreachable, a slow app, a suspected breach — eventually comes down to "what's actually happening on the wire." This lab builds the foundational skill of answering that question directly from packet captures instead of guessing.

| Role | How this applies |
|---|---|
| Network Engineer | Diagnose connectivity issues by seeing exactly where packets are dropped or delayed |
| SOC Analyst | Identify malicious traffic patterns, extract indicators of compromise from captures |
| Cloud Security Engineer | Same mental model transfers directly to reading Azure Network Watcher / VPC flow logs |
| Help Desk | Prove a reported network issue is real, and determine if it's client-side or server-side |

---

## Skills Demonstrated

- Capturing live traffic on an active network interface
- Writing and applying Wireshark display filters to isolate relevant traffic in large captures
- Reading and validating a TCP three-way handshake (SYN → SYN-ACK → ACK)
- Identifying DNS queries/responses and matching transaction IDs
- Spotting cleartext credentials in HTTP traffic (and why HTTPS matters)
- Following a full TCP stream to reconstruct a client-server conversation
- Saving, exporting, and reopening `.pcapng` captures for later analysis
- Command-line packet capture with `tshark`

---

## Key Concepts Covered

- **Packets & protocols** — how data is broken into packets, headers vs. payload
- **TCP three-way handshake** — SYN / SYN-ACK / ACK and what a missing SYN-ACK or an RST tells you
- **DNS** — how name resolution works, A records vs. AAAA/CNAME/MX
- **HTTP vs. HTTPS** — why unencrypted traffic exposes credentials in plaintext
- **Promiscuous mode** — what Wireshark's NIC capture mode does and doesn't show you on a switched network
- **Display filters vs. capture filters** — why display filters are the right tool for post-capture analysis

---

## Walkthrough

### Exercise A — Capture a DNS Lookup
Started a capture, ran `nslookup google.com` from a separate terminal, then filtered on `dns` in Wireshark to find the query and matching response packet. Verified the A record IP in the response matched what `nslookup` returned in the terminal.

### Exercise B — Watch the TCP Three-Way Handshake
Resolved `example.com`'s IP via `nslookup`, then filtered with `tcp and ip.addr == [IP]` while loading the site over HTTP. Identified the SYN → SYN-ACK → ACK sequence and confirmed what each flag combination means for connection state.

### Exercise C — Spot Cleartext Credentials (HTTP)
*Educational exercise, run only against a local test login form.* Filtered on `http.request.method == POST` and inspected the HTML Form URL Encoded layer to see submitted credentials in plaintext — direct evidence of why HTTPS is non-negotiable for anything handling auth data.

### Exercise D — Follow a Full TCP Stream
Right-clicked an HTTP packet → **Follow → TCP Stream** to reassemble the full client-server conversation from individual packets, distinguishing the browser's request (red) from the server's response (blue).

---

## Reference: Display Filters Used

| Filter | What it shows |
|---|---|
| `dns` | All DNS queries and responses |
| `http` | Unencrypted HTTP traffic |
| `tcp` | All TCP traffic |
| `tcp.flags.syn == 1` | TCP SYN packets (connection attempts) |
| `tcp.flags.reset == 1` | TCP RST packets (refused/reset connections) |
| `icmp` | Ping / reachability traffic |
| `ip.addr == x.x.x.x` | All traffic to/from a specific IP |
| `tcp.port == 443` | HTTPS traffic |
| `http.request` | HTTP GET/POST requests only |

---

## Verification

- [x] Captured a DNS query/response pair with matching transaction IDs
- [x] Identified all three packets of a TCP handshake and explained each flag
- [x] Filtered live by IP, port, and protocol without reference material
- [x] Followed a TCP stream and read a full HTTP request/response as a conversation
- [x] Saved a `.pcapng` capture, closed Wireshark, reopened it, and confirmed all packets persisted

---

## Repo Contents

```
/captures
  dns-lookup.pcapng
  tcp-handshake.pcapng
  tcp-stream-follow.pcapng
/screenshots
  dns-query-response.png
  three-way-handshake.png
  follow-tcp-stream.png
README.md
```

---

## What I Learned

Seeing a DNS query and its response side by side — with matching transaction IDs — makes name resolution click in a way that reading about it never does. The same goes for the TCP handshake: once you've watched SYN → SYN-ACK → ACK happen in real traffic, "connection refused" errors in the field stop being a mystery and start being a pattern you recognize instantly.

## What I'd Do Differently

Next time I'd capture on an Azure VM alongside the local capture, to compare what cloud-hosted traffic looks like vs. local traffic — and start correlating Wireshark's packet-level view with what shows up in Azure Network Watcher for the same connection.
