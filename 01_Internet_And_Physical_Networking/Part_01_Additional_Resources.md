# Part 01 — Resources & Command Reference

This companion supports the [study notes](Part_01_Study_Notes.md). Select a resource for the concept you need; there is no requirement to read every page. Example output below is illustrative, not a measurement of your connection.

## Targeted reading

| Priority | Resource | Read or inspect this portion | Purpose |
|---|---|---|---|
| Core | [Cloudflare: How does the Internet work?](https://www.cloudflare.com/learning/network-layer/how-does-the-internet-work/) | The Internet definition, packets and protocols | Reinforce the network-of-networks model |
| Core | [Cloudflare: What is a packet?](https://www.cloudflare.com/learning/network-layer/what-is-a-packet/) | “Why use packets?”, “What is a packet header?” and “Where do packet headers come from?” | Connect shared links, headers and encapsulation |
| Core | [Cloudflare: What is latency?](https://www.cloudflare.com/learning/performance/glossary/what-is-latency/) | “What causes Internet latency?” and “Network latency, throughput, and bandwidth” | Separate delay, capacity and achieved rate |
| Core | [Cloudflare: What is a CDN?](https://www.cloudflare.com/learning/cdn/what-is-a-cdn/) | The delivery/caching explanation and benefits | Understand why a nearby edge can avoid an origin round trip |
| Reference | [Wireshark User's Guide](https://www.wireshark.org/docs/wsug_html_chunked/) | “Capturing Live Network Data”, “Packet Details” and “Filtering Packets While Viewing” | Inspect headers rather than memorising their names |
| Optional | [AWS Global Infrastructure](https://aws.amazon.com/about-aws/global-infrastructure/) | Regional map, AZ grouping and edge/Local Zone footprint | Relate cloud locations to geography and failure boundaries |
| Optional | [Submarine Cable Map](https://www.submarinecablemap.com/) | One cable's landing points, route and owners | Make the overseas fibre path concrete |

The cable map is an interactive application; text retrieval does not expose its map details. Use it in a browser rather than treating a text fetch as evidence that a particular cable or route was inspected. Infrastructure counts and service availability can change.

The local notes refine common simplifications: speed tests report achieved throughput, RTT need not be exactly twice one fixed one-way delay, and not every physical component is an IP hop.

## Read the host's configuration

```powershell
# Windows
ipconfig /all
route print
arp -a
```

```bash
# Linux
ip -br addr
ip route
ip neigh

# macOS
ifconfig
netstat -rn
arp -a
```

Look for the interface's IP address and subnet mask/prefix, default gateway, DNS servers and MAC address. A routing table answers where a destination will be sent; the neighbour/ARP table holds learned local IP-to-link-address mappings.

```text
Illustrative configuration:
IPv4 address:    192.168.1.23
Subnet mask:     255.255.255.0
Default gateway: 192.168.1.1
DNS server:      192.168.1.1
```

The DNS entry may name the router because it forwards queries. That does not make the router authoritative for every Internet domain. A remote server's public IP normally will not appear in your ARP table: your next local destination is the gateway.

## Measure RTT and observed loss

```powershell
# Windows
ping -n 4 8.8.8.8
ping -n 4 example.com
```

```bash
# Linux/macOS
ping -c 4 8.8.8.8
ping -c 4 example.com
```

```text
Illustrative result:
Reply from 8.8.8.8: bytes=32 time=24ms TTL=117
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

`time=24ms` is a round trip. TTL is the remaining IPv4 hop limit in the received reply, not DNS cache time. Four packets give a small observation, not proof that a path never loses traffic. An ICMP timeout can reflect filtering or rate limits even while HTTPS works.

Compare the gateway with a remote destination to distinguish local access trouble from a wider path problem. A domain test also depends on name resolution; an IP-literal test avoids that lookup.

For achieved transfer rate, [Cloudflare Speed Test](https://speed.cloudflare.com/) is the browser tool used in the lecture. Compare its Mbps result with RTT/jitter/loss instead of reducing connection quality to one number.

## Interpret routed hops

```powershell
# Windows
tracert example.com
```

```bash
# Linux/macOS, when installed
traceroute example.com

# Optional repeated path observations, when mtr is installed
mtr -rw example.com
```

These tools send probes with increasing IP TTL/hop limits. Intermediate routers can return “time exceeded”; the destination response depends on the probe type.

```text
Illustrative trace:
1     2 ms     3 ms     2 ms    192.168.1.1
2     8 ms     9 ms     8 ms    ISP-router
3       *        *        *     Request timed out
4    24 ms    25 ms    24 ms    Later-router
```

Hop 3 did not answer those probes. Because later replies arrived, traffic can clearly pass beyond it. Do not interpret every intermediate timeout as end-to-end packet loss, or subtract adjacent RTTs as exact link delays: replies can take different paths. Some routed steps may be hidden, and geolocation labels are approximate.

## Inspect cache-related HTTP headers

```powershell
# Windows: curl.exe selects the actual curl executable
curl.exe -I https://example.com
```

```bash
# Linux/macOS
curl -I https://example.com
```

`-I` requests headers using HTTP HEAD. The target might not use a CDN or allow caching, so no specific cache header is guaranteed.

```text
Illustrative response fields:
Cache-Control: public, max-age=3600
Age: 120
ETag: "asset-v3"
```

The first line allows shared caching with a freshness lifetime of 3600 seconds. `Age` contributes to determining how old the response is; `ETag` supports validation. Vendor-specific `X-Cache` or `CF-Cache-Status` may explain a hit/miss. Sending `Cache-Control: no-cache` requests validation; it is not a universal promise to bypass every intermediary.

## Look inside one packet

In Wireshark, capture on the active interface while producing traffic, then apply the **display filter** `icmp`. Display filters select already captured packets; capture filters control what enters the capture.

An ordinary IPv4 ping exposes:

| Layer | Inspect |
|---|---|
| Link header, such as Ethernet II | Local source/destination link addresses |
| Internet Protocol Version 4 | Source IP, destination IP, TTL and protocol |
| ICMP | Echo request/reply type and identifiers |

The exact link header depends on the interface and capture mode. You do not need to decrypt HTTPS to inspect IP addresses or transport metadata. Later parts explain TCP/DNS filters and why encryption hides application content.

**Optional MTU illustration:** on Linux, `ping -M do -s 1472 8.8.8.8` uses 1472 ICMP data bytes plus an 8-byte ICMP and typical 20-byte IPv4 header, totalling 1500. It asks IPv4 not to fragment. Results depend on path MTU and whether ICMP is permitted; a failure alone does not diagnose the cause.

## Keep optional cloud work focused

The original lecture's EC2 demonstration establishes that a VM has a region, AZ, private address and physical hosting location. You can understand that without creating cloud resources. If inspecting an existing authorised environment, its console or configured CLI can show those properties. Older unauthenticated metadata commands may fail when IMDSv2 tokens are required; avoid treating them as a universal discovery method.

For this part, skip cloud pricing calculators, BGP policy, optical cable engineering and Wireshark dissector development. They do not improve the core explanation yet.

**Continue:** [Interview questions](Part_01_Interview_Questions.md) · [Part 2](../02_IP_Addressing_Subnets_And_DNS/README.md)
