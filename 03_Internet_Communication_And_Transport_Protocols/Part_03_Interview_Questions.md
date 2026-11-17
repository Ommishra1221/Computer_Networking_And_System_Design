# Part 03 — Interview Questions

Eight essential questions cover the fresher-level foundation. Explain the mechanism in your own words. The four optional scenarios add practical judgement; use the linked notes for implementation detail.

## Eight essential questions

### 1. How do OSI and TCP/IP describe the same communication?

OSI has Application, Presentation, Session, Transport, Network, Data link and Physical layers. TCP/IP groups these into Application, Transport, Internet and Network access. OSI is reference vocabulary; TCP/IP describes the deployed protocol suite. A conventional switch forwards at layer 2, a router at layer 3 and a hub repeats layer 1 signals. Real products combine functions. TLS has presentation/session-like responsibilities but sits above TCP in the familiar HTTP/1.1–2 stack.

### 2. Explain encapsulation, and distinguish an IP address, port and socket.

Application bytes receive a transport header, then an IP header, then a link envelope: segment/datagram → packet → frame. The receiving endpoint reverses this. An IP identifies an interface in a network; a port helps select a TCP/UDP endpoint; a socket is an OS communication endpoint. Routers replace local framing and decrement packet lifetime. Ordinary routing preserves IP endpoints, while NAT changes them. The remote server does not receive the laptop's original local-link frame.

### 3. Explain the TCP three-way handshake and why it is not authentication.

The client sends SYN with sequence `x`. The listening server replies SYN-ACK with its own `y` and ACK `x+1`. The client acknowledges `y+1`. Both directions now have synchronised initial sequence numbers; the third message confirms that the client received the server's SYN. This establishes transport state and bidirectional communication. It does not prove a user identity, establish a trusted business or encrypt the stream. Higher-layer security and authentication do those jobs.

### 4. What does TCP guarantee, and how does it recover from loss?

TCP presents an ordered byte stream using sequence numbers, acknowledgements and retransmission. ACK 1100 means byte 1100 is expected next; later bytes can wait in a buffer until a gap is filled. TCP recovers from network packet loss rather than preventing it. Permanent failure can stop progress, and a transport ACK does not prove an application committed a transaction. Flow control protects receiver buffers; congestion control limits network pressure. TCP does not preserve application message boundaries.

### 5. When would you choose TCP or UDP?

Choose TCP's ordered byte stream for complete documents, SSH, database connections and ordinary HTTP/1.1–2 APIs. UDP provides separate datagrams without its own handshake, retransmission, ordering, flow or congestion control. It suits workloads whose applications handle retries or prioritise timely data, such as DNS, games and interactive media. UDP's lower overhead does not guarantee a faster application. QUIC adds reliable streams above UDP, while buffered video can use HTTP/TCP. Protocol choice follows requirements, not a universal speed ranking.

### 6. What belongs in HTTP messages, and why does idempotency matter?

An HTTP/1.1 request has a method, target, headers and optional body; its response has a status, headers and optional body. HTTP is stateless, while cookies/sessions/tokens provide application state. Idempotency means repeating an operation has the same intended effect as doing it once, even if responses differ. GET, PUT and DELETE have idempotent semantics; POST/PATCH require case-specific reasoning. Retried business POSTs may need server-supported idempotency keys. Recognise 2xx success, 3xx redirection/cache validation, 4xx request-side and 5xx server/upstream responses.

### 7. How does HTTPS differ from HTTP?

HTTPS protects HTTP using TLS: confidentiality, integrity and normally server authentication. The client validates certificate names, dates and the trusted chain, and verifies the server's handshake proof. This does not automatically authenticate a user or prove honest intentions. HTTP/1.1–2 use HTTP → TLS → TCP → IP; HTTP/3 uses QUIC/UDP with integrated TLS security. Port 443 is a convention, not a certificate. Certificate errors can reflect expiry, wrong names, missing intermediates, untrusted issuers or an incorrect clock.

### 8. Walk through a URL request, then explain how you would diagnose failure.

Resolve the name, select a route, establish transport and security, exchange HTTP, process the request and render the response. Frames carry packets hop by hop throughout. Caches/reused connections can skip setup; rendering often triggers more requests. Diagnose interface/IP/gateway → DNS → port → TLS → HTTP/logs/dependencies using configuration tools, `nslookup`/`dig`, port tests and `curl`. DNS supports UDP and TCP, not UDP alone. Failed ICMP does not prove a host is down; successful TCP does not prove a working API.

## Optional applied scenarios

### 9. Design communication for chat messages, attachments and calls.

A long-lived WebSocket over TCP supports bidirectional chat. Add message IDs and stored synchronisation state so clients can reconnect and recover missed messages; TCP alone cannot provide offline delivery or application deduplication. Treat typing/presence as ephemeral. Upload attachments over HTTPS to suitable storage and share references; use mobile push for offline notifications. Interactive voice/video commonly uses UDP-based media with jitter buffers, adaptive bitrate, loss concealment, congestion handling and limited timely retransmission. Signalling can use HTTPS/WebSocket, and real-time implementations may fall back to other transports. Separate workloads by their delivery requirements.

### 10. A service is slow although server CPU is low. What would you inspect?

Separate DNS, connection, TLS, first-byte and download timing. Curl's timestamps are cumulative: subtract adjacent milestones to estimate phases, and remember first-byte time includes network effects. Inspect retransmissions, receive windows, dependency latency, connection churn and large-packet/MTU failures. Bandwidth × RTT estimates the data needed in flight to fill a path. Pooling/reuse often removes repeated setup cost. TIME-WAIT normally handles repeated FINs and old duplicates after active close; it is not inherently a fault. Investigate small-write delays or `TCP_NODELAY` only with evidence, and distinguish network loss from incomplete captures.

### 11. Compare HTTP/1.1, HTTP/2 and HTTP/3 without assuming every version is faster.

| Version | Transport/concurrency | Limit to remember |
|---|---|---|
| HTTP/1.1 | TCP, default persistent connections | Ordered exchanges can delay others |
| HTTP/2 | TCP, multiplexed streams | Missing TCP bytes can stall streams |
| HTTP/3 | QUIC/UDP, independent reliable streams | Congestion and network constraints remain |

HTTP/1.1 does not require a `Connection: keep-alive` header for default persistence. QUIC adds security, loss recovery and congestion handling above UDP; connection IDs can support path changes. Its streams avoid one shared ordered TCP byte stream, although one lost packet can contain data from multiple streams. Performance depends on workload and network conditions. Compression and handshake optimisations belong in [optional depth](Part_03_Study_Notes.md#udp-leaves-delivery-policy-to-the-application).

### 12. Does HTTPS stop a SYN flood?

No. A SYN flood targets TCP establishment before TLS. Many incomplete handshakes can consume server connection state; mitigations include SYN cookies, appropriate connection handling, rate controls and upstream DDoS protection. Certificates solve endpoint authentication and protected communication, a different problem. Likewise, completing a TCP handshake does not authorise application access. The general lesson is to identify the resource being exhausted and place the control where it can act: application authentication cannot repair an already saturated link, and encryption cannot guarantee transport availability. Layered defences remain necessary.

For methods/status examples, close states, protocol versions and commands, use the [notes](Part_03_Study_Notes.md), [mechanism diagrams](Part_03_Diagrams.md) and [interpreted commands](Part_03_Additional_Resources.md).