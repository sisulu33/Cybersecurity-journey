# Session 02: HTTPS compared with HTTP

`curl --http1.1 -v https://example.com` on macOS, captured with tshark (`-f "tcp port 443 or udp port 53"`).
Evidence: `../captures/03-https-example-com-tls13.pcapng`, compared with `../captures/01-tcp-handshake-success-example-com.pcapng`.
Negotiated: TLS 1.3, TLS_AES_256_GCM_SHA384, ALPN http/1.1 (from curl -v).

## What the wire shows

| Frame | From | TLS record size | Note |
|---|---|---|---|
| 301 | client | 512 | Client Hello, SNI example.com readable |
| 303 | server | 122 + 1 | Server Hello, change-cipher-spec |
| 305 | server | 3859 | encrypted handshake messages (Certificate etc.) |
| 309 | client | 92 | the HTTP request: 75 bytes + 17 bytes TLS overhead |
| 312 | server | 948 | the encrypted response |

## What changed vs HTTP

| | HTTP | HTTPS |
|---|---|---|
| IPs, ports, timing, TCP handshake | visible | visible |
| DNS lookup | plaintext | plaintext |
| Site name | Host header, DNS | DNS and SNI |
| Method, path, headers, page | readable | encrypted |
| Certificate | n/a | encrypted in TLS 1.3 |
| Message sizes | visible | visible |

## My explanation

**What changed:**

HTTPS adds a TLS handhsake between the TCP handshake and the HTTP request. The client hello goes out in plain text, and after the server hello both sides derive encryption keys. Everything after that is ciphertext: the request, response and in TLS 1.3 even the server's certificate. In HTTP capture, follow TCP Stream showed GET /HTTP/1.1, teh headers and the full page. In the HTTPS capture, the same stream showed none of those strings, and a grep for them returned 0. My 75-byte request became a 92-byte encrypted record, which is the same bytes plus 17 bytes of TLS overhead. 

**What did not change, and what an observer can still learn:**

TCP,IP and DNS behave exactly as before. The SYN,SYN-ACK and ACK are visible with the same ECN flags, and so are the IP addresses, ports and timing. The DNS lookup for example.com was still plain text, and the Client Hello still carried example.com as the SNI, so an observer learns which site i visited from two places. Record sizes are also visible, which means they can tell how much data moved even though they cant read it. HTTPS hides the content of the conversation butnot the fact of it.

**What I cannot conclude from Follow TCP Stream:**

Follow TCP Stream showed unreadable bytes because wireshark doesnt have session keys, not because there is nothing there to read.Unreadable bytes tell me the content is encrypted from the point of view of someone without keys, and nothing more. They don’t prove the connection is safe, because encryption says nothing about who is on the other end. That comes from certificate validation, which curl reported as "SSL certificate verify ok". They also don’t prove HTTPS hides everything, since the metadata above stayed visible.
