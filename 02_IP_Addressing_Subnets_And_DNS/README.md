# Part 02 — IP Addressing, Subnets & DNS

**Lecture segment:** 01:09:00–02:06:35. Part 1 built the physical path. This part explains how interfaces are addressed, address space is organised, and human-readable names lead to services.

| File | Use it for |
|---|---|
| [Study notes](Part_02_Study_Notes.md) | Addressing → IPv4/IPv6 → subnets/CIDR → DNS → domain trust |
| [Visual companion](Part_02_Diagrams.md) | NAT, bit boundaries, aligned subnet blocks and DNS conversations |
| [Resources and commands](Part_02_Additional_Resources.md) | Targeted references, DNS output and a few checked subnet examples |
| [Interview questions](Part_02_Interview_Questions.md) | Eight essential answers and four optional scenarios |

Prioritise the worked subnet calculation and the DNS sequence. They explain mechanisms that later return in transport, firewall rules and system design. Optional sections retain useful operational depth without requiring a DNS administration course.

By the end, you should be able to:

- Explain why address uniqueness has a scope, and distinguish static/DHCP, public/private and loopback addressing.
- Read IPv4 and IPv6 notation and explain why NAT helped conserve public IPv4 space.
- Derive a subnet's network, broadcast, usable range and host count, including `/31`, `/32` and cloud-platform exceptions.
- Explain CIDR as classless allocation/routing as well as slash notation.
- Trace a DNS cache miss through a recursive resolver, root, TLD and authoritative servers.
- Choose common record types and explain caching, migrations and domain-name trust.

The notes preserve the instructor's phone-number and telephone-directory analogies, then give their limits. Subnetting does not create more addresses, `8.8.8.8` is a recursive resolver, and `bank.in` is below the TLD `.in`. A valid HTTPS connection does not establish that a lookalike hostname belongs to the intended brand.

These files interpret the course's Hindi/Hinglish auto-caption source rather than quoting it. **Optional supplement** sections provide supporting depth already present in the workspace.

**Course navigation:** [Previous: Internet & Physical Networking](../01_Internet_And_Physical_Networking/README.md) · [Course overview](../README.md) · [Compact whole-course guide](../Computer_Networking_And_System_Design_Core_Concepts.md) · [Next: Internet Communication & Transport Protocols](../03_Internet_Communication_And_Transport_Protocols/README.md)
