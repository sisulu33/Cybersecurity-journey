# Session 01: TCP, Wireshark, HTTP and DNS

Taken on macOS over a phone hotspot (client 172.20.10.3) with Wireshark/tshark and curl 8.16.0.
Evidence: `../captures/01-tcp-handshake-success-example-com.pcapng` and `../captures/02-syn-retransmission-backoff-failed.pcapng`.
Captures were trimmed with tshark to the relevant conversation before publishing.

## 1. The three-way handshake

| Step | Direction | Flags | Gap from previous packet |
|---|---|---|---|
| 1 | client to server | SYN (also CWR, ECE: ECN offered) | n/a |
| 2 | server to client | SYN, ACK (ECE: ECN accepted) | 17.332 ms |
| 3 | client to server | ACK | 0.220 ms |

**My explanation:** (states on each side, and why three steps, not two)

Before the connection, the server is in LISTEN state and the client has no state. When the client sends the SYN, it moves to SYN_SENT. The server receives it and moves to SYN_RECEIVED, and replies with SYN_ACK. When the client receives the SYN-ACK it replies with an ACK and moves into an ESTABLISHED state. The server moves to established and when the ACK arrives.

TCP numbers every byte, and each side picks a random starting number which is called the Initial sequence number(ISN). Each side needs two things ; to tell the other its own ISN, and to hear back that the other received it. Thats four messages in principle. The SYN-ACK is the middle two combined: the server acknowledges the client's ISN and announces its own in the packet, which makes three.

If it was only two steps the server would never learn if the client received its ISN.A delayed duplicate SYN from an old connection could also arrive and make the server open a connection no one wants.The third packet lets the client confirm that this connection is the one it actually started, or reset it if not.

In my capteure, the SYN-ACK arrived 17.3 ms after the syn, which is one network round trip.

## 2. Retransmission, RTO and exponential backoff

Failed connection to 34.223.124.45:80, same client port (61360) throughout, no SYN-ACK ever received.

| Attempt | Time (s) | Gap (s) |
|---|---|---|
| 1 (original SYN) | 7.287 | n/a |
| 2 | 8.288 | 1.000 |
| 3 | 9.288 | 1.000 |
| 4 | 10.289 | 1.001 |
| 5 | 11.290 | 1.001 |
| 6 | 12.291 | 1.001 |
| 7 | 14.292 | 2.001 |
| 8 | 18.293 | 4.001 |
| 9 | 26.294 | 8.001 |
| 10 | 42.295 | 16.001 |
| 11 | 74.295 | 32.000 |

Observation: five retries at a flat 1 s, then the gap doubles each time.

**My explanation:** (what RTO is, why TCP retransmits, the initial RTO default, why the wait doubles, and why this capture does not match the textbook exactly)

The RTO is how long the TCP waits for an acknowledgement before sendoing the packet again.A packet can be lost and tcp has to recover without help from the application. For a SYN,TCP has no measured round-trip time to base the timeout on so it uses an initial default.

If the retry also gets no answer the wait doubles. The reason is that silence usually means the network is congested or the host is down , and retrying at a fixed rate would add load to a path thats already struggling.

My data shows the doubling: 2, 4, 8, 16, then 32 seconds. The whole attempt spans about 67 seconds (7.287 s to 74.295 s) across 11 SYNs.(Observered on MacOS)

## 3. Reading Wireshark

**My explanation:** (the three panes, display filters vs capture filters, with examples I used)

The packet list (top) has one row per packet, with time, source, destination, protocol and a summary. The packet details pane (middle) is a collapsible tree of the selected packet, one section per protocol layer. The packet bytes pane (bottom) shows the raw hex and ASCII. Clicking a field in the middle pane highlights its bytes in the bottom pane.

Capture filters and display filters use different languages and apply at different times. A capture filter, such as tcp port 443, decides what gets recorded at all. Anything it excludes is gone permanently. A display filter, such as tcp.flags.syn==1 or tcp.port == 80, hides packets from view but keeps them in the file. That’s why you can switch from one display filter to another on the same capture, and why a capture filter that’s too narrow can cost you evidence.

## 4. Plaintext HTTP

Follow TCP Stream on the example.com connection: 75 bytes sent, 1037 bytes received, all readable.
Visible: Host, path, User-Agent (curl/8.16.0), the server's headers (Cloudflare, cache status) and the full HTML body.

**My explanation:** (what an observer on the path can see and what that means for real logins)

Follow TCP stream reassembles the packets of one connection into the conversation into the conversation. Mine showed 75 bytes sent and 1037 bytes received, all readable. Anyone who can see the treaffic, such as someone on the same hotspot or the isp or a router in between, can read the same thing: the site, the path, the tool used(curl/8.16.0) the server's headers and the whole page.

Nothing sensitive leaked here because example.com has no login. On a site that used plain HTTP, a login form would send the username and password in this same readable stream. That’s the problem HTTPS solves.

## 5. SYN flood

**My explanation:** (how it works, what resource it exhausts, why spoofed addresses make it hard to defend, and how it relates to the failed-connection capture)

A server that receives a SYN allocates a small amount of memory for. the half-open connection and waits, in SYN_RECEIVED, for the final ACK. That memory is limited. A SYN flood sends huge numbers of SYNs and never completes the handshake which fills the queue so real clients get refused.

Attackers usually spoof the source addresses. Then the SYN-ACKs go to innocent hosts that never answer, and the server can't tell the attackers from real users by address, so blocking by IP doesnt work. Defenses include SYN cookies which let the server avoid storing state until the ACK arrives, plus larger queues and rate limiting.

## 6. DNS over UDP

| Detail | Value |
|---|---|
| Transport | UDP, client port 64368 to port 53 |
| Query and answer | 2 packets, answered in 75.8 ms |
| Resolver | 172.20.10.1 (hotspot gateway) |
| Transaction ID | 0x6551 |
| Answer | 104.20.23.154, 172.66.147.243 |

**My explanation:** (why no handshake, what UDP trades away, the metadata problem, the poisoning problem)

DNS lookups are usually one small question and one small answer,so a three-packet setup would cost more than the lookup itself. Yours was two packets and took 75.8 ms in total.

UDP gives up reliability, ordering and connection state. If a query is lost, the application has to notice and retry. DNS does this itself, so the cost is handled one layer up.

There are two security problems. First, privacy: queries are plaintext, so your resolver and anyone on the path between you and it can see every domain you look up, even when the site itself uses HTTPS. Second, forgery: with no connection, a reply is accepted if it matches the query’s 16-bit transaction ID (0x6551 in yours) and the source port. An attacker who guesses those can get a fake answer accepted before the real one arrives. That’s cache poisoning. Randomizing the port makes the guess much harder, and DNSSEC lets the answer be cryptographically verified.
