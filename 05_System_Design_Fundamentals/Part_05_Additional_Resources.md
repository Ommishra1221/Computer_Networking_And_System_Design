# Part 05 — Selected Resources and Practical Commands

The [Study Notes](Part_05_Study_Notes.md) explain the course design without requiring you to read an external book. Use these selections to resolve a particular question or see a mechanism more clearly. **Core** targets support DevBoard directly. **Optional** targets extend it. Links identify the intended documentation, not a promise of continuing availability or an independently verified version of every page.

## Core reading: a small target for each question

| When you need more detail | Resource and exact target | Stop when you can explain |
|---|---|---|
| How can two services share a typed contract? | [gRPC introduction](https://grpc.io/docs/what-is-grpc/introduction/): read the service-definition and Protocol Buffers explanations, then the RPC lifecycle overview in [Core concepts](https://grpc.io/docs/what-is-grpc/core-concepts/) | The `.proto` contract, generated stubs, message encoding, unary versus streaming calls |
| Where do dashboard numbers originate? | [Prometheus overview](https://prometheus.io/docs/introduction/overview/): focus on the architecture and pull-based collection model | Who initiates a scrape, what is stored, and why Grafana is a separate query/display layer |
| What makes services meaningfully independent? | [Martin Fowler and James Lewis: Microservices](https://martinfowler.com/articles/microservices.html): read the introductory contrast with a monolith, business-capability organisation, and decentralised data management | Why splitting processes alone is insufficient, and which operational/data costs independence adds |
| How do the design trade-offs fit together? | [System Design Primer](https://github.com/donnemartin/system-design-primer): select “Performance vs scalability,” “Latency vs throughput,” and “Cache” | One bottleneck example, latency versus capacity, and the cache-aside freshness trade-off |

The Primer is a broad community reference, not a second compulsory syllabus. Use the listed targets rather than starting its complete interview index. The gRPC/Prometheus pages are implementation documentation; the microservices article explains the architectural reasoning. The local notes' SQL/NoSQL choice should still be judged from access patterns and required guarantees.

## Optional follow-up: choose the gap you actually have

| Gap | Resource | Focus and boundary |
|---|---|---|
| HPA versus VPA versus node capacity | [Kubernetes workload autoscaling](https://kubernetes.io/docs/concepts/workloads/autoscaling/) and [HPA](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) | Read the distinctions and CPU utilization example. Check current platform/version behaviour before applying manifests |
| Building one useful dashboard | [Grafana documentation](https://grafana.com/docs/) | Follow the introductory Prometheus-data-source and first-dashboard material. Aim for traffic, errors, and latency rather than many panels |
| Comparing a cloud reference design | [Google Cloud Architecture Center](https://cloud.google.com/architecture) | Choose a three-tier web application reference and identify public entry, private application/data, and failure boundaries. Product names are implementation choices |
| Seeing services in one repository | [Google Cloud Online Boutique](https://github.com/GoogleCloudPlatform/microservices-demo) | Inspect the service diagram and `src` layout. The useful lesson is microservices plus a monorepo, not that DevBoard needs the same number of services |
| Reviewing reliability and operating cost | [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/the-aws-well-architected-framework.html) | Read the Reliability and Cost Optimization design principles. Use them to ask what fails and what costs money |
| Going deeper after this course | [Designing Data-Intensive Applications](https://dataintensive.net/) | Chapter 1 on reliability/scalability/maintainability, then replication in the edition you use. It is optional long-form reading |

Patterns such as sagas, circuit breakers, and durable event publication become useful when a concrete failure requires them. The [microservices.io pattern catalogue](https://microservices.io/patterns/index.html) is a lookup reference for that stage, rather than a list to memorize now. CAP and multi-region designs also belong after the basic project explanation is comfortable.

## Inspect an HTTP contract

These commands make ordinary requests. `curl.exe` avoids the legacy PowerShell `curl` alias.

```powershell
# Windows: inspect response headers
curl.exe -I https://example.com/

# Adapt this URL to an already running local API
curl.exe -i http://localhost:8080/api/tasks
```

```bash
# Linux/macOS equivalents
curl -I https://example.com/
curl -i http://localhost:8080/api/tasks
```

A local API requiring a token might respond like this; the example is illustrative:

```http
HTTP/1.1 401 Unauthorized
Content-Type: application/json

{"error":"authentication_required"}
```

This is an application response, showing that an HTTP-speaking endpoint answered. It differs from connection refusal, a TLS validation failure, or a timeout. An authenticated request can still receive `403 Forbidden` because its caller lacks resource permission. Inspect method/path, status, content type, and body together. A server-identification header is implementation detail rather than the API's purpose.

For an API you own, inspecting the request in browser developer tools can be more convenient than copying credentials into shell history. Avoid sharing bearer tokens with screenshots/output.

## Compare observed latency with the intended request path

```powershell
# Windows: adapt to your local service
curl.exe -sS -o NUL -w 'status=%{http_code} total=%{time_total}s\n' http://localhost:8080/api/tasks
```

```bash
# Linux/macOS
curl -sS -o /dev/null -w 'status=%{http_code} total=%{time_total}s\n' http://localhost:8080/api/tasks
```

An illustrative output is `status=200 total=0.084s`. This measures one client-observed request, not p95 and not a diagnosis. Authentication failures can be fast while successful task queries are slow. A slow summary may include model work, queue waiting, network delay, and database time; use application metrics/traces to separate them.

For a background summary request, `202 Accepted` plus a job ID means accepted work, not completed generation. Its job-status API must distinguish pending, failed, and completed states and authorize access to that job.

## Observe autoscaling without assuming it is immediate

Use these only with an existing cluster/context you intend to inspect. Names and labels must match the deployed example.

```bash
kubectl get hpa
kubectl describe hpa devboard-api
kubectl get deployment devboard-api
kubectl get pods -l app=devboard-api
kubectl get hpa devboard-api -w
```

Stop the watch with Ctrl+C. Compare current replicas, target utilization, deployment readiness, and HPA events. Missing metrics or unset resource requests can prevent a CPU-utilization target from working as expected. A scale decision is not a ready pod: scheduling, image startup, and health checks take time.

For a simplified paper interpretation, four replicas at 90% against a 70% CPU target suggests `ceil(4 × 90/70) = 6` desired replicas before limits, missing data, and stabilization behaviour. HPA changes count; VPA concerns per-workload resource sizing. More pods do not establish that the database can handle their combined traffic.

## Read metrics before building extra infrastructure

With an instrumented service already exposing metrics:

```powershell
curl.exe http://localhost:8080/metrics
```

```bash
curl http://localhost:8080/metrics
```

An application might expose a counter such as `http_requests_total{status="200"} 1240`, but actual names/labels depend on instrumentation. A counter's cumulative value differs from requests per second. An illustrative PromQL query for an existing matching counter is:

```promql
sum(rate(http_requests_total[5m]))
```

This estimates a recent per-second rate from counter changes. Do not paste a metric name that the service does not expose and interpret an empty graph as healthy traffic. Percentile latency needs suitable duration instrumentation, such as histograms; it cannot be reconstructed from a request counter alone.

Use one concrete story: request volume rises, p95 rises, and database pool waits rise while API CPU stays modest. That is evidence to inspect data access/pool capacity, rather than immediately increase API CPU.

## Three short design checks

1. **Trace a task edit.** Name the token check, resource permission, durable write, cache invalidation, and notification path. State what the user sees if notification delivery fails after the write.
2. **Mark failure boundaries.** For API, task store, Redis, AI, notifications, and entry, distinguish unavailable from merely slower. Add one recovery choice for each critical dependency.
3. **Check the cost assumption.** With the lecture's assumed ₹1,000 payment, 2% fee, and 18% tax on the fee, compute `1000 − 20 − 3.60 = 976.40`. Then name the fixed and usage-based infrastructure costs. Provider prices need their own current quotation.

These are checks of your explanation, not a requirement to deploy a Kubernetes cluster or complete a large lab before moving on.

**Continue:** [Interview questions](Part_05_Interview_Questions.md) · [Study Notes](Part_05_Study_Notes.md) · [Diagrams](Part_05_Diagrams.md)

