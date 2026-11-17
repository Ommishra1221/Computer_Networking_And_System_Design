# Part 05 — System Design Fundamentals

This part turns the earlier networking concepts into a working application design. It follows the lecture from design fundamentals and product examples through **DevBoard**, then ends with application and repository organisation.

**Lecture segment:** 03:30:07–04:19:58 in [Computer Networking & System Design For DevOps in OneShot](https://youtu.be/xN3BKHji12I).

You can read before or after watching. Parts 1–4 help with DNS, TCP/UDP, TLS, addressing, and access rules; revisit those when an architecture arrow feels unclear.

## Choose the file that serves your next step

| File | Use it for |
|---|---|
| [Study Notes](Part_05_Study_Notes.md) | Connected explanations, project decisions, trade-offs, and complete request paths |
| [Diagrams](Part_05_Diagrams.md) | Requirements, tiers, login, reads/writes, AI jobs, scaling, metrics, and failure boundaries |
| [Resources and commands](Part_05_Additional_Resources.md) | Small external reading targets and interpretable HTTP/metrics/autoscaling examples |
| [Interview questions](Part_05_Interview_Questions.md) | Eight essential answers and five optional scenarios |

Start with the notes for understanding, a diagram for a specific flow, or the interview essentials for revision. No deployment lab is required to follow the course.

## The learning sequence

The Book Kida, YouTube, Netflix-clone, and teaching-platform examples uncover the work behind a visible page. DevBoard then supplies a concrete project: React UI, Go API, SQL users, document tasks/summaries, JWT identity, Redis cache, Python/Ollama AI, and WebSocket notifications.

Entry routing/load balancing, Kubernetes, and Prometheus/Grafana complete the operating picture. The closing distinction is that monolith/microservices describe application deployment, while monorepo describes where source code lives.

**Source supplements** preserve useful production checks—queues, retries, backups, bottlenecks, quotas, tracing, and cost—without turning them into prerequisites or asserting that the demonstrated project implements them all.

## Ready-to-explain outcomes

- [ ] Turn user behaviour and measurable constraints into justified components.
- [ ] Draw DevBoard and trace a read, task edit, login, and summary request.
- [ ] Explain REST/gRPC and SQL/NoSQL choices without blanket rules.
- [ ] Separate token verification, resource authorization, cache freshness, and live delivery.
- [ ] Describe one bottleneck, one failure boundary, and useful monitoring evidence.
- [ ] Compare application architecture and repository organisation with a concrete trade-off.

For each arrow, name the protocol, permission, carried data, and failure outcome. For each box, name the requirement that justifies it.

**Previous:** [Part 4 — Internet Security](../04_Internet_Security/README.md)  
**Next:** [Course revision](../Computer_Networking_And_System_Design_Final_Revision_Notes.md) · [Course home](../README.md)

