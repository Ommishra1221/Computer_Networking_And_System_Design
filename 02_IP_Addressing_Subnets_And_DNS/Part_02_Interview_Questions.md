# Part 02 — Interview Questions

Eight essential questions cover the fresher-level mechanisms. Four optional scenarios connect them to deployment and troubleshooting. Use the [study notes](Part_02_Study_Notes.md) for explanation and the [visual companion](Part_02_Diagrams.md) for subnet/DNS sketches.

## Essential questions

### 1. What does an IP address identify, and why can homes reuse private addresses?

An IP address is assigned to a network interface within an addressing scope. A device can have several addresses. Separate private networks can reuse RFC 1918 ranges because those addresses are not routed across the public Internet. NAT/PAT commonly maps outbound conversations through a public IPv4 address and reverses the mapping for replies.

Private ranges are `10.0.0.0/8`, `172.16.0.0/12` and `192.168.0.0/16`.

### 2. Compare IPv4 and IPv6.

IPv4 uses 32-bit addresses written as decimal octets; IPv6 uses 128-bit addresses written in hexadecimal groups, with zero compression. IPv6 has a much larger space, uses Neighbor Discovery instead of ARP, and has no broadcast. Both use network prefixes. IPv6 is not automatic encryption, and dual stack runs both protocols together.

### 3. How do subnet masks and CIDR relate to forwarding?

A mask separates network and host bits. In a basic LAN, matching masked network addresses indicate direct local delivery; otherwise the host normally uses a gateway. The actual route table selects the route. CIDR describes flexible prefix lengths and supports efficient classless allocation and route aggregation. A longer IPv4 prefix means fewer host bits and a smaller block.

### 4. Calculate the network, broadcast and host range for `192.168.10.50/26`.

Six host bits give 64 addresses. The mask is `255.255.255.192`; aligned blocks begin at `.0`, `.64`, `.128` and `.192`. The address is in `.0–.63`, so the network is `192.168.10.0`, broadcast `.63`, and traditional hosts `.1–.62`: 62 usable addresses.

### 5. How do you size a subnet for 500 hosts, and when does “subtract two” fail?

Allow at least 502 total addresses, round up to 512, and use nine host bits: `/23`, giving 510 traditional hosts. Supported `/31` point-to-point links use both addresses, while `/32` identifies one host/address route. Cloud platforms can reserve extra addresses: standard AWS IPv4 `/24` subnets normally provide 251 assignable addresses.

### 6. Explain a cold DNS lookup and recursive versus iterative queries.

After local caches/hosts configuration miss, the stub asks a recursive resolver. The resolver checks caches, then follows root → TLD → authoritative referrals as needed. It caches the returned record and answers the client. The client asks for recursion; the hierarchy walk receives iterative answers/referrals. `8.8.8.8` is a recursive resolver, not a TLD server.

### 7. Which DNS records would you use for addresses, aliases, email and authority?

A maps a name to IPv4; AAAA maps it to IPv6. CNAME aliases another name, not an IP or URL. MX names mail servers with priorities; TXT carries text such as verification/email-policy data; NS identifies authoritative nameservers. SOA holds zone administrative data, and PTR supports reverse naming. Ordinary CNAME data cannot coexist with other ordinary data at the same name.

### 8. Why does a DNS change work for some users before others?

Resolvers and clients can retain different cached answers until their TTLs expire. Before a move, lower TTL early enough for the old value to expire, prepare the new endpoint, switch the record and retain the old endpoint during the transition. An OS cache flush does not flush the upstream resolver. DNS TTL is seconds, unlike IP's hop-limiting TTL.

## Optional scenarios

### 9. Plan addressing for an application deployed across three AZs.

Start with a non-overlapping VPC block and allocate aligned subnets per AZ/tier with room for growth and rolling replacements. For example, `10.20.0.0/16` can contain public `10.20.1.0/24`–`10.20.3.0/24`, application `10.20.11.0/24`–`10.20.13.0/24`, and data `10.20.21.0/24`–`10.20.23.0/24`.

In AWS, each subnet occupies one AZ. Routes and filters determine public/private access. Overlap with a network you later connect can make routing ambiguous or prevent peering, so plan ranges before integration.

### 10. A domain works intermittently for different users. How do you investigate DNS?

Compare answers from relevant resolvers and every authoritative server. Check TTLs/recent changes, multiple returned IPs, cached NXDOMAIN, hosts/VPN configuration and whether the failure concerns only IPv4 or IPv6. A stale authoritative copy or dead address can cause intermittent symptoms. SERVFAIL needs investigation; DNSSEC validation failure is one possible reason. `dig +dnssec` alone does not prove successful validation.

DNS-based balancing is coarse: caches and client address selection can delay failover and distribute traffic unevenly.

### 11. How would you introduce IPv6 without breaking IPv4 clients?

Start with dual-stack support, often at the CDN/load balancer. Publish AAAA only when the IPv6 target, listeners, routes, firewall rules and monitoring work. Retain A records and working IPv4 support. Check applications storing/parsing IP addresses and compare errors by family. Running IPv6 does not automatically copy IPv4 security policy or encrypt traffic.

### 12. Does DNSSEC or a valid HTTPS certificate stop a fake banking website?

Neither prevents an attacker registering a different lookalike domain. DNSSEC authenticates DNS data for the queried name; TLS authenticates the connection's intended hostname under certificate trust and encrypts transport. Users still need the correct hostname. Restricted registration policies, domain/certificate monitoring and email authentication can help, but they address different parts of the problem.

`bank.in` is below the TLD `in`; a fake domain does not need to bypass the DNS hierarchy.

**Continue:** [Part 3 — Internet Communication & Transport Protocols](../03_Internet_Communication_And_Transport_Protocols/README.md)
