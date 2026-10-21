# Session 03: Mapping a Wireshark packet onto the OSI model

Evidence: `../captures/01-tcp-handshake-success-example-com.pcapng`, frame 6 (the HTTP GET) and frame 1 (the DNS query).

## Frame 6: HTTP GET to example.com

| Wireshark line | OSI layer | Why |
|---|---|---|
| Frame 6: 141 bytes on wire, interface en0 | none (capture metadata) | Added by Wireshark: capture time, length, interface. It is not a protocol and adds no bytes to the packet. |
| Ethernet II, Src 92:de:6f:72:fd:80, Dst fe:66:cf:24:e3:64 | 2, Data Link | MAC addresses and framing for delivery on the local network |
| Internet Protocol Version 4, Src 172.20.10.3, Dst 104.20.23.154 | 3, Network | Logical addresses used to route the packet across networks |
| Transmission Control Protocol, 61396 to 80, Seq 1, Ack 1, Len 75 | 4, Transport | Ports, sequence numbers and acknowledgments give reliable, ordered delivery |
| Hypertext Transfer Protocol (GET / HTTP/1.1) | 7, Application | The request the program (curl) makes of the web server |

Byte check: 141 total - 14 (Ethernet) - 20 (IPv4) - 32 (TCP header) - 75 (HTTP data) = 0. Every byte belongs to a real protocol, and the Frame line adds none.

## Frame 1: DNS query

| Wireshark line | OSI layer | Why |
|---|---|---|
| Frame 1 | none (capture metadata) | Same as above |
| Ethernet II | 2, Data Link | Same role as in frame 6 |
| Internet Protocol Version 4, Dst 172.20.10.1 | 3, Network | Addressed to my hotspot's DNS resolver |
| User Datagram Protocol, port 64368 to 53 | 4, Transport | Connectionless: no handshake, no acknowledgments |
| Domain Name System (query) | 7, Application | The question "what is the address of example.com?" |

Compared with the GET, the same layers are present. Only the protocols differ: UDP replaces TCP at layer 4, and DNS replaces HTTP at layer 7.

## Which layers show nothing, and why

- **Layer 1 (Physical):** The signal (voltage, radio, light) is turned into bytes by the network card before Wireshark sees it. A capture has no physical layer to show.
- **Layers 5 and 6 (Session, Presentation):** No separate protocol appears for them. The OSI model is a reference model, and real TCP/IP traffic doesn't follow it exactly. The TCP/IP model has only four layers, with everything above transport folded into one application layer. In this traffic:
  - The job of layer 5, tracking a conversation, is done by the TCP connection (its handshake, sequence numbers and FIN close), and for the web by application-level things like cookies. HTTP itself is stateless.
  - The job of layer 6, data representation, shows up inside the application protocol. HTTP's `Content-Type: text/html; charset=utf-8` in the response is an example, and TLS encryption is often placed here in the model. In my HTTPS capture (session 02), TLS sat between TCP and HTTP and appeared as its own line.

## The destination MAC address

`fe:66:cf:24:e3:64` is not example.com's address. Layer 2 only delivers to the next hop on the local network, and that next hop is my hotspot's gateway (172.20.10.1). The destination IP (104.20.23.154) stays the same all the way to the server, but the MAC addresses are replaced at every hop. The DNS query goes to the same gateway MAC.
