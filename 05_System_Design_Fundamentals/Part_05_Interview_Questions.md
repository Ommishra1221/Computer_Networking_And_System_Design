# Part 05 — Interview Questions

Use these eight essentials for fresher backend, Java, software-engineering, and introductory system-design interviews. Explain the decision and one trade-off in your own words. The five scenarios are optional follow-up, not another question bank.

The [Study Notes](Part_05_Study_Notes.md) teach the full project; the [Diagrams](Part_05_Diagrams.md) supply the flows to draw while answering.

## Eight essential questions

### 1. What is system design, and where do you start?

System design turns requirements into component responsibilities, interfaces, data choices, and plans for load, failure, security, operation, and cost. Start with the user actions and constraints, rather than choosing products first.

For DevBoard, creating tasks and requesting AI summaries are functional requirements. Expected peak traffic, latency targets, availability, and per-user isolation are non-functional constraints. Define measurable assumptions: daily users alone do not determine requests per second.

### 2. Explain DevBoard's tiers and one request through them.

React presents the task board. The Go API checks identity, permissions, input, and business rules. SQL holds users while a document-oriented NoSQL store holds tasks/summaries. A Python service calls Ollama for AI work.

For a task edit, the UI calls the API through entry routing. The API verifies the token and task permission, commits the update, invalidates related cache entries, and emits a notification event. The response and WebSocket notification are separate paths. No direct browser-to-database access is part of this design.

### 3. Why use REST/JSON at one boundary and gRPC at another?

REST-style HTTP/JSON suits browser-facing resource APIs and straightforward inspection. gRPC suits internal Go-to-Python calls through typed schemas, generated code, compact messages, and unary/streaming interfaces.

Both expose APIs. REST is not restricted to JSON or HTTP/1.1, and gRPC is not universally faster. Browser access generally needs gRPC-Web or adapters. Choose from the client's needs and operational trade-offs.

### 4. How would you choose SQL or NoSQL?

Use relationships, invariants, transactions, query patterns, expected load, and the team's operational knowledge. SQL fits DevBoard's users and unique-email constraint; documents fit its demonstrated nested task/summary representation. SQL could also store those tasks.

NoSQL covers several models and does not imply eventual consistency or absent transactions. Two stores add cross-store consistency, backup, migration, and operational work.

### 5. How does JWT authentication work, and what does it not establish?

The auth service verifies credentials and issues a signed JSON Web Token. The API verifies its signature, permitted algorithm, expiry, issuer, and audience as appropriate, then separately checks permission for the requested project/task.

A typical signed payload is readable. A stolen bearer token can be used until expiry/revocation, so token handling and lifetime matter. Local verification reduces per-request auth-service dependence, but does not eliminate authorization checks or guarantee operation during every auth outage.

### 6. How would you cache DevBoard task lists?

Use cache-aside: authorize project access, look up a project-scoped key, query the task store on a miss, and cache the result with a suitable TTL. After a successful write, invalidate affected entries.

TTL and deletion reduce stale data but do not eliminate races, such as an old read refilling the cache. Define acceptable freshness. Redis failure needs bounded database fallback, not an assumption of unlimited database capacity.

### 7. How do a load balancer, horizontal scaling, and vertical scaling differ?

A load balancer distributes traffic among healthy targets. Horizontal scaling adds instances; vertical scaling adds resources to an instance. API replicas work more flexibly when required per-client state is external/shared and any healthy replica can serve the request.

Health checks, state handling, and downstream capacity matter. Kubernetes HPA adjusts replica count; VPA concerns resource sizing according to policy. More API replicas do not automatically fix a saturated database, and two smaller machines cannot necessarily run one indivisible large job.

### 8. Compare monoliths, microservices, and monorepos.

A monolith is an application deployed as one unit, potentially with strong internal modules and an external database. Microservices are independently deployable capabilities that communicate through network interfaces. They can scale/release independently but add partial failures, latency, data coordination, and operations.

A monorepo is one repository for multiple projects, compatible with either architecture. Client-server roles and distributed deployment also overlap with these choices. A modular monolith can suit a small team until a measured scaling, release, or ownership need justifies extraction.

## Optional scenarios

### A. AI summaries take eight seconds and make the application feel slow.

Confirm whether generation is blocking the request or exhausting shared capacity. Accept authorized work into a durable queue, return `202 Accepted` with a job ID, and let workers persist results for polling/notification. Bound model concurrency, add timeouts, and make redelivered jobs idempotent.

Cache summaries against the project's relevant data version if useful. Quotas, input limits, and cost alarms protect expensive AI calls. Streaming can improve perceived progress, but does not remove total compute cost or capacity limits.

### B. Task reads become slow as traffic grows. What do you inspect first?

Correlate request latency with query time, slow queries, pool waits, active connections, CPU, and cache misses. In Java/JPA, inspect N+1 queries and fetch patterns. Fix access patterns and useful indexes before adding infrastructure.

Cache hot reads if freshness permits, then consider read replicas for read-heavy load while accepting possible lag. Sharding may help a genuinely larger partitionable workload, but adds key-selection and cross-shard costs. Measure the bottleneck before applying a scaling product.

### C. A task was saved, but the user did not receive its live notification.

The durable write and event delivery are separate operations. Preserve the committed task, and let reconnecting clients refetch state. WebSocket maintains a live connection; it does not store offline messages.

Reliable delivery needs durable event recording/retry or reconciliation. With multiple notification replicas, distribute events to the instance holding the relevant socket. Decide whether the requirement is “eventually show current task state” or “deliver every notification,” because the second needs more machinery.

### D. What would you monitor, and what happens if a dependency fails?

Monitor traffic, latency percentiles, errors, and saturation, then task-store query/pool time, cache hit ratio, AI latency/failures, and queue backlog. Prometheus pulls metric samples; Grafana queries data sources for dashboards. Logs explain events and traces connect dependency calls.

One failed API replica should lose traffic through health checks. AI/notifications should degrade independently through deadlines and resource limits. Redis fallback must fit DB capacity. A sole task store or sole entry can remove service; appropriate redundancy and recoverable backups need an explicit plan.

### E. A user is deleted in SQL, but task cleanup in the other store fails.

A single local SQL transaction does not make an independent store update atomic. Use a deliberate deletion state/soft delete, retryable cleanup, and a reconciliation process that detects leftovers. Authorize reads so deleted users do not retain access during cleanup.

An event-driven cleanup or a saga can coordinate eventual completion; each step needs idempotency and failed-work handling. These are optional production extensions. Do not claim a separate-store operation is atomic without an actual supported distributed-transaction mechanism.

## Short follow-up notes worth keeping

- **Gateway policy:** entry routing and load balancing do not automatically provide every authentication/rate-limit function. Configure suitable supported features.
- **Auth failure:** existing tokens may still work only while trusted verification keys and required authorization information remain available.
- **Replication versus backup:** replicas support availability/read scale; backups preserve recoverable state when deletion/corruption also reaches replicas.
- **CAP:** the consistency/availability trade-off applies during a network partition. It is not a general instruction to choose any two properties in every situation.
- **Cost:** use the lecture's payment example as assumed arithmetic, then discuss fixed hosting and usage-based delivery/model costs.
- **Repository choice:** shared tooling and atomic changes favour a monorepo; stronger isolation may favour separate repositories, with coordination costs.

A project answer should connect **requirements → user journey → boundaries/contracts → data/security → measured bottleneck → failure/recovery → monitoring and cost**. For each component, explain what user promise it fulfils.

**Continue:** [Course revision](../Computer_Networking_And_System_Design_Final_Revision_Notes.md) · [Study Notes](Part_05_Study_Notes.md) · [Resources](Part_05_Additional_Resources.md)

