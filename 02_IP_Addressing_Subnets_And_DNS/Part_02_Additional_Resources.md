# Part 02 — Resources, Commands & Worked Examples

Use this reference alongside the [study notes](Part_02_Study_Notes.md). The subnet answers and sample DNS output are explanatory examples, not live observations.

## Reading targets

| Priority | Resource | Focus |
|---|---|---|
| Core | [Cloudflare: What is DNS?](https://www.cloudflare.com/learning/dns/what-is-dns/) | The resolver/root/TLD/authoritative roles, lookup sequence and caching |
| Core | [Cloudflare: DNS records](https://www.cloudflare.com/learning/dns/dns-records/) | Common records: A, AAAA, CNAME, MX, TXT, NS, SOA and PTR |
| Reference | [RFC 1918: Private address allocation](https://www.rfc-editor.org/rfc/rfc1918) | Section 3, the three private IPv4 ranges |
| Reference | [AWS: Subnet CIDR blocks](https://docs.aws.amazon.com/vpc/latest/userguide/subnet-sizing.html) | “Subnet sizing for IPv4”, reservations and non-overlap constraints |
| Optional tool | [CIDR.xyz](https://cidr.xyz/) | Change an IPv4 prefix and inspect aligned network/broadcast/host ranges |
| Optional | [RFC 3021: IPv4 /31 point-to-point links](https://www.rfc-editor.org/rfc/rfc3021) | Why both addresses can be endpoint addresses on these links |
| Optional | [RFC 1034: DNS concepts and facilities](https://www.rfc-editor.org/rfc/rfc1034) | Domain hierarchy, name-server responsibility and resolver behaviour |
| Optional definition | [MDN: DNS glossary](https://developer.mozilla.org/en-US/docs/Glossary/DNS) | Short web-oriented terminology check |

The earlier workspace also linked [DigitalOcean's IP/subnet/CIDR tutorial](https://www.digitalocean.com/community/tutorials/understanding-ip-addresses-subnets-and-cidr-notation-for-networking) and [RFC 4632 on CIDR](https://www.rfc-editor.org/rfc/rfc4632). During this revision, text retrieval exposed only the tutorial's site shell, and the RFC request was rate-limited. They remain optional pointers; the worked explanations here do not depend on having retrieved their full content. The interactive calculator's displayed ranges can check arithmetic, but it does not define a cloud platform's reservations.

## Read an interface and its route

```powershell
# Windows
ipconfig /all
route print
```

```bash
# Linux
ip -br addr
ip route

# macOS
ifconfig
netstat -rn
```

```text
Illustrative interface: 192.168.1.23/24
Network:                192.168.1.0/24
On-link example:        192.168.1.99
Off-link example:       192.168.2.10, normally reached through a gateway
```

The mask defines the basic local boundary. Check the actual route table when a host has multiple interfaces, VPN routes or explicit routes. A wrong mask can cause a host to ARP for a destination that should have been routed.

## Three subnet examples to understand, rather than memorise

| Input | Reasoning | Result in the traditional IPv4 model |
|---|---|---|
| `192.168.1.130/26` | Six host bits → 64-address blocks; .130 is in .128–.191 | Network `.128`, broadcast `.191`, hosts `.129–.190`, 62 hosts |
| `172.16.20.0/22` | Mask `255.255.252.0`; third-octet blocks advance by four | Network `172.16.20.0`, broadcast `172.16.23.255`, 1022 hosts |
| `192.168.5.200/28` | Four host bits → 16-address blocks; .200 is in .192–.207 | Network `.192`, broadcast `.207`, hosts `.193–.206`, 14 hosts |

For 500 ordinary hosts, `500 + 2 = 502`; round up to 512 addresses, giving nine host bits and `/23`. You obtain 510 usable hosts. For two endpoints, a traditional `/30` supplies two host addresses, while a supported `/31` point-to-point link uses both addresses and conserves space.

```bash
# Optional installed calculator
ipcalc 192.168.1.130/26
```

A calculator normally uses textbook IPv4 host rules. Standard AWS IPv4 subnets reserve five rather than two addresses, so `/24` commonly means 251 assignable addresses there. Do not use the calculator's host count as a universal platform guarantee.

## Query a DNS record

```powershell
# Windows
nslookup example.com
nslookup example.com 8.8.8.8
nslookup -type=AAAA example.com
nslookup -type=MX example.com
nslookup -type=NS example.com
```

```bash
# Linux/macOS/WSL with dig installed
dig example.com
dig example.com +short
dig AAAA example.com +short
dig MX example.com +short
dig NS example.com +short
dig @8.8.8.8 example.com
```

`nslookup`/ `dig` ask a DNS server; they do not fetch the website. An answer marked “non-authoritative” can be perfectly valid: it was obtained from a resolver rather than directly from the zone's authoritative server.

```text
Illustrative dig answer:
;; ANSWER SECTION:
www.example.com.  300  IN  A  203.0.113.10
```

| Field | Interpretation |
|---|---|
| `www.example.com.` | Fully qualified owner name, with explicit root dot |
| `300` | Remaining TTL in seconds as reported in this answer |
| `IN` | Internet class |
| `A` | IPv4-address record |
| `203.0.113.10` | Illustrative value from a documentation range |

The `SERVER` line identifies the responding DNS server, not necessarily the website's host. An MX answer returns mail-server names/priorities; a CNAME answer returns another name. A DNS answer does not establish that port 443 is open or that the application is healthy.

## See delegation and distinguish caches

```bash
dig +trace example.com
```

This performs an iterative delegation walk, showing root/TLD/authoritative responses when the network allows it. It is different from asking your normal recursive resolver for a final answer. Cached delegations mean ordinary browser lookups often skip much of this work.

A repeated lookup to the same caching resolver may show a decreasing TTL. It can also return a different value because the resolver refreshed the data or different backend caches answered. Browser/OS caching is separate from resolver caching.

```powershell
# Clear the Windows DNS client cache
ipconfig /flushdns
```

```bash
# Linux systems using systemd-resolved
sudo resolvectl flush-caches

# macOS cache command; browser/other services may have their own caches
sudo dscacheutil -flushcache
```

These commands do not erase your ISP/public resolver's cache. Do not expect a local flush to reset every DNS TTL to its initial value. A hosts-file entry, corporate resolver or browser encrypted-DNS setting can also change which resolution path an application uses.

For a migration, lower TTL before the cutover and wait out the old value. A low TTL added at the moment of change does not shorten copies already cached with the old longer TTL.

## Optional diagnostic patterns

For inconsistent answers, compare resolvers and then the authoritative servers discovered through NS lookup:

```bash
dig @8.8.8.8 example.com
dig @1.1.1.1 example.com
dig NS example.com +short

# Replace the placeholder with an actual authoritative nameserver from that lookup
dig @AUTH_SERVER_HOSTNAME example.com A
```

Differences can be intentional, especially for distributed delivery. They can also reveal a stale authoritative copy or a dead address among multiple answers. NXDOMAIN means the queried name does not exist; SERVFAIL means resolution failed and needs more investigation. DNSSEC problems are one possible SERVFAIL cause. `dig +dnssec` requests DNSSEC-related information; that option alone does not validate the entire trust chain.

To test a chosen HTTPS endpoint while keeping the hostname in the request and certificate checks:

```bash
# Illustrative syntax only: substitute a real endpoint IP
curl --resolve example.com:443:203.0.113.10 https://example.com/
```

The documentation IP above is not a real target. On Windows use `curl.exe` to select the executable. Testing `https://IP/` changes the hostname and can fail virtual hosting/certificate checks even when the actual site works.

For an existing dual-stack site, `curl -4 https://example.com/` and `curl -6 https://example.com/` select an address family. Success still depends on DNS records, local connectivity and endpoint support. Check both families' access rules rather than assuming IPv4 rules also protect IPv6.

Deep DNSSEC key management, IPv6 translation mechanisms, cloud pricing and private DNS administration are outside this part's core.

**Continue:** [Interview questions](Part_02_Interview_Questions.md) · [Part 3](../03_Internet_Communication_And_Transport_Protocols/README.md)
