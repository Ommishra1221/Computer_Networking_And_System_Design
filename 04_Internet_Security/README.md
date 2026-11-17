# Part 04 — Internet Security

The lecture's nginx-on-EC2 demonstration provides the thread: a running application is initially unreachable, a security-group rule permits HTTP, HTTPS needs additional configuration, and VPC routing separates public entry points from private services.

**Lecture:** 02:49:20–03:30:07 in [Computer Networking & System Design For DevOps in OneShot](https://youtu.be/xN3BKHji12I?t=10160).

Use the notes for the explanation, diagrams for traffic paths, and commands for evidence. The optional depth keeps selected production topics available without turning a first pass into a cloud-security certification course.

| File | Purpose |
|---|---|
| [Study notes](Part_04_Study_Notes.md) | Ingress/egress, filtering, ports, TLS, UFW, VPC routes and defence in depth |
| [Diagrams](Part_04_Diagrams.md) | Rule decisions, least-privilege tiers, TLS validation and separate inbound/outbound VPC paths |
| [Resources and commands](Part_04_Additional_Resources.md) | Selected documentation, representative output and service-access diagnosis |
| [Interview questions](Part_04_Interview_Questions.md) | Eight essential answers and four optional applied scenarios |

After this part, you should be able to distinguish SGs from NACLs, diagnose an unreachable service, explain what TLS validates, and draw public/private routes without treating NAT as a firewall. You should also be able to say which controls remain necessary after opening a port or enabling HTTPS.

[Previous: Internet Communication & Transport Protocols](../03_Internet_Communication_And_Transport_Protocols/README.md) · [Next: System Design Fundamentals](../05_System_Design_Fundamentals/README.md) · [Course workspace](../README.md)