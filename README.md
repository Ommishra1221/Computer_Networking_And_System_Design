# Computer Networking & System Design — Study Workspace

This course follows one connected journey: a device joins a physical network, gets an address, finds a service through DNS, exchanges data, secures the connection, and reaches an application designed to handle real users and failures.

The five-part sequence follows the original workspace's chapter map for **Computer Networking & System Design For DevOps in OneShot** (4:19:58). The notes can be read directly; use the [lecture](https://youtu.be/xN3BKHji12I) for demonstrations or a topic that needs another explanation.

## Choose the amount of detail you need

Start with [Core Concepts](Computer_Networking_And_System_Design_Core_Concepts.md) for a complete connected first pass. Open the matching folder when you need more explanation, another diagram, or a practical observation. Use [Final Revision Notes](Computer_Networking_And_System_Design_Final_Revision_Notes.md) to recall the course once the ideas are familiar.

Within a folder, the study notes teach the concepts in order. The other files support that explanation instead of repeating it. There is no requirement to watch the whole video or finish every resource before moving on.

| Part | Lecture segment | The connection you will understand |
|---|---|---|
| [1. Internet & Physical Networking](01_Internet_And_Physical_Networking/README.md) | 00:00–01:09:00 | Devices, links, packets, performance, forwarding equipment, and where content lives |
| [2. IP Addressing, Subnets & DNS](02_IP_Addressing_Subnets_And_DNS/README.md) | 01:09:00–02:06:35 | Interface addresses, local versus routed delivery, prefix arithmetic, and name resolution |
| [3. Internet Communication & Transport Protocols](03_Internet_Communication_And_Transport_Protocols/README.md) | 02:06:35–02:49:20 | Layered delivery, ports, TCP/UDP, application protocols, and troubleshooting |
| [4. Internet Security](04_Internet_Security/README.md) | 02:49:20–03:30:07 | Connection policy, TLS trust, and public/private cloud network paths |
| [5. System Design Fundamentals](05_System_Design_Fundamentals/README.md) | 03:30:07–04:19:58 | Requirements, DevBoard's components, data, operations, and architecture choices |

## Each file has a different job

| File in each folder | Use it for |
|---|---|
| `README.md` | Orientation, prerequisites, and the few outcomes that matter most |
| `Part_0X_Study_Notes.md` | Connected teaching, worked examples, comparisons, and essential diagrams |
| `Part_0X_Diagrams.md` | Visual explanations of mechanisms and boundaries, with links to the teaching sections |
| `Part_0X_Additional_Resources.md` | Prioritised references and selected commands with interpretation |
| `Part_0X_Interview_Questions.md` | Essential fresher answers, followed by optional reasoning scenarios |

A useful stopping point is being able to explain the main diagram in your own words and answer the essential questions. If one link in your explanation is missing, revisit that specific section. Running a few diagnostic commands gives the vocabulary something concrete to describe.

## Lecture navigation

These timestamps are preserved from the existing course materials; chapter titles describe the lecture, not a required heading template for the notes.

| Time | Chapter | Part |
|---|---|---|
| 00:00 | Introduction | 1 |
| 01:40 | Topic Listing | 1 |
| 05:00 | What is the Internet & How Does It Work? | 1 |
| 24:45 | Data Transmission & Networking Terminologies | 1 |
| 33:55 | Physical Internet, Wi-Fi, Routers, Switches & Hubs | 1 |
| 57:00 | Data Centers, CDN, Edge Locations & Local Zones | 1 |
| 01:09:00 | Internet Addressing | 2 |
| 01:25:00 | Subnets & CIDR | 2 |
| 01:44:40 | All About DNS | 2 |
| 02:06:35 | How Communication Happens on the Internet | 3 |
| 02:37:10 | Types of Layer 4 (Transport Layer) Protocols | 3 |
| 02:49:20 | How Security Works on the Internet | 4 |
| 03:30:07 | System Design Fundamentals | 5 |
| 03:51:00 | Project Explanation with System Design | 5 |
| 04:14:25 | Types of System Design | 5 |
| 04:19:58 | End | — |

## Check that the course connects

- [ ] Explain a request from a laptop to an overseas service, distinguishing link delivery from routed IP delivery.
- [ ] Calculate an IPv4 block and trace a DNS cache miss without treating every lookup as a root query.
- [ ] Explain TCP/UDP choices, then distinguish transport success from application success.
- [ ] Draw public entry and private outbound paths, with the relevant access rules and TLS boundary.
- [ ] Draw DevBoard, justify its components, and describe a bottleneck, a failure, and a trade-off.

## Sources and technical care

The original workspace describes the lecture as Hindi/Hinglish with auto-generated captions. This revision uses the supplied local notes, diagrams, examples, and question sets; it is an explanatory adaptation rather than a verbatim transcript or a newly verified transcription of the video. Lecture examples remain useful anchors, while inaccurate or overbroad claims are corrected beside the relevant concept.

Optional supplements are identified where they extend the lecture. Detailed BGP, advanced IPv6 migration, Kubernetes internals, and large-scale distributed algorithms remain outside the core course. External resources have targeted reading suggestions; they are references, not an additional mandatory syllabus. A successful HTTP response alone would not establish a resource's technical accuracy, and this workspace does not claim every linked page was read or remains accessible.

The optional external networking-zine PDF linked in Part 3 was inaccessible during review because of retrieval size/connection limits. No local PDFs or image files are required: the teaching diagrams are embedded Mermaid or ASCII. Cloud counts, pricing, certificate lifetimes, and software behaviour can change, so the notes avoid treating recording-time values as permanent facts.
