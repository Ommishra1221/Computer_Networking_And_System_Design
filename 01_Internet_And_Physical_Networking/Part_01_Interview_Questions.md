# Part 01 — Interview Questions

Eight essential questions cover the fresher-level core. Answer naturally rather than reciting a paragraph. The optional scenarios test whether the same concepts help you reason about a real problem.

Use the [study notes](Part_01_Study_Notes.md) for explanations and the [visual companion](Part_01_Diagrams.md) when an answer needs a sketch.

## Essential questions

### 1. How do a network, the Internet and the Web differ?

A network connects devices so they can exchange data. The Internet connects independently operated networks using common protocols. The Web is one service using that infrastructure, primarily through HTTP/HTTPS. Wi-Fi is a local access link; email, DNS and SSH also use the Internet without being the Web.

### 2. What is a packet, and why divide a transfer into packets?

An IP packet contains an IP header and payload. Smaller units let many flows share links, rather than letting one large transfer occupy the medium indefinitely. Higher protocols such as TCP can recover missing pieces. A TCP segment is carried inside the IP packet; a link-layer frame carries that packet across a link.

### 3. How do bandwidth, throughput, latency, jitter and loss affect users?

Bandwidth is link capacity; throughput is the rate actually achieved. Latency is delay, while jitter is its variation and loss means packets fail to arrive. Large downloads need throughput; calls and games also need low, stable delay and tolerable loss. `ping` normally reports RTT, the outward and return journey combined.

### 4. Compare a hub, switch and router.

A hub repeats signals without interpreting addresses. A layer-2 switch learns source MAC-to-port mappings and forwards using destination MACs, flooding when required within its VLAN. A router forwards IP packets between networks using a routing table. A home router commonly bundles switching, Wi-Fi, DHCP and NAT as well.

### 5. Why does a remote packet use the gateway's MAC address?

The IP destination remains the remote endpoint, but the current link needs a reachable local destination. The host resolves the gateway's MAC using ARP and sends the frame there. The router removes that framing, chooses the next IP hop and creates new framing. MAC addresses support local delivery; IP addresses support routed delivery.

### 6. What makes a data centre more than a room of servers?

It provides computing, storage and network equipment together with power, backup power, cooling, physical security and resilient connectivity. A rented cloud VM still runs on physical hardware there. Location affects user latency, while the design of facility and application dependencies affects availability.

### 7. How does a CDN reduce origin load, and when is the origin contacted?

A CDN caches suitable responses at distributed edge locations. Fresh cache hits can be answered locally. Misses, expiry/revalidation, eviction or a forwarding policy can require origin contact. Shared assets are good candidates; personalised responses need safe policies. A CDN can forward dynamic requests but cannot remove database execution time.

### 8. Distinguish a region, AZ, Local Zone and edge location.

A region is a geographic deployment area. An AZ is an isolated data-centre group within it. A Local Zone extends selected regional services into another metro for nearby compute. An edge location is part of a distributed delivery footprint, often caching content. AZ redundancy helps only when the workload and dependencies actually span failure domains.

## Optional scenarios

### 9. A 1 Gbps user reports an eight-second page load, while the server reports 200 ms. Where do you look?

Separate DNS, TCP/TLS setup, request/response transfer, application time and browser rendering. Compare local versus remote RTT, jitter and end-to-end loss. Inspect request count, asset size and caching. The server's 200 ms measures only part of the journey. A CDN, fewer round trips or smaller assets may help different portions; more bandwidth does not remove distance.

An intermediate `traceroute` timeout alone does not identify the fault: routers can decline probe replies while forwarding normally.

### 10. How would you deliver an Oregon-hosted learning platform worldwide?

Use a CDN for shared images, scripts, PDFs and video segments, with controlled cache freshness and versioned asset names. Dynamic account/progress operations reach regional application services. Range requests help media seeking. Monitor user-facing latency, errors, cache hit ratio and origin traffic. The CDN reduces repeated origin work and potentially transfer cost, while multi-region application state adds complexity.

### 11. Two application replicas run in different AZs. Is the service highly available?

Only if the load balancer spans healthy locations, detects failures, and the surviving capacity can handle traffic. Database access, session state and other dependencies must also survive. During detection/failover, some requests may fail. In-memory state confined to one replica and a single-AZ database can defeat the purpose of distributing application instances.

### 12. What changes when a cable cut isolates the primary region?

Avoid depending on one network path where possible, and use another region if the required recovery objectives justify it. A standby, replicated data, health-based traffic switching and degraded/read-only operation can help. DNS failover is limited by caches. State the target recovery time (**RTO**) and tolerable data-loss window (**RPO**); active-active deployment costs more coordination than a standby.

The aim is to explain which dependency fails and how service recovers, not to promise that more regions prevent every outage.

**Continue:** [Part 2 — IP Addressing, Subnets & DNS](../02_IP_Addressing_Subnets_And_DNS/README.md)
