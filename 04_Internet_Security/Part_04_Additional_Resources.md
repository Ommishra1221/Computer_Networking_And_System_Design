# Part 04 — Resources and Practical Commands

Use these references to answer specific questions from the [study notes](Part_04_Study_Notes.md). The commands are short examples rather than a required exercise sequence. Read-only diagnosis can be done on your machine or an authorised endpoint; configuration examples belong on a host you intend to administer.

## Focused references

| Reference | Reading target |
|---|---|
| [AWS — Security groups](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html) | Stateful tracking, allow rules, combined permissions, defaults and references. Distinguish SG behaviour from NACL behaviour. |
| [AWS — Network ACLs](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html) | Subnet scope, numbered rule order and stateless return-traffic permissions. |
| [AWS — NAT gateways](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html) | Public NAT Internet-egress pattern, address translation and connectivity types. The course diagram specifically uses a public NAT gateway. |
| [AWS — Subnets](https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html) | Subnet/AZ relationship and route-based public/private classification. |
| [IANA — Service and port registry](https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml) | Look up a conventional service/transport assignment. A registered number does not force the actual listener to use that protocol. |
| [Ubuntu — UFW](https://help.ubuntu.com/community/UFW) | Defaults, source-specific allows, status and deletion. Confirm SSH access before enabling it remotely. |
| [Certbot](https://certbot.eff.org/) | Obtain installation instructions matching your OS/server, then understand issuance, challenge validation and renewal. |
| [Cloudflare — Firewalls](https://www.cloudflare.com/learning/security/what-is-a-firewall/) | Explanatory overview of packet, stateful and application filtering. |
| [Cloudflare — TLS](https://www.cloudflare.com/learning/ssl/transport-layer-security-tls/) | Explanatory connection between encryption, certificates and handshake authentication. |
| [OWASP Top 10](https://owasp.org/www-project-top-ten/) | Optional application-security context: network reachability controls do not fix application weaknesses. |

These are references, not a claim that every page was newly verified or returned a particular HTTP status on a particular date. Provider features, software flags and certificate policies can change; consult the applicable documentation when administering a real deployment.

## Diagnose the listener before changing rules

```powershell
# Windows: show listeners and test a remote TCP endpoint
netstat -ano | findstr LISTENING
Test-NetConnection example.com -Port 443
```

```bash
# Linux: show TCP/UDP sockets and owning processes
sudo ss -tulpn
sudo systemctl status nginx
curl -I http://127.0.0.1
```

```text
LISTEN 0 511 127.0.0.1:8080 0.0.0.0:* users:(("java",pid=1234,fd=6))
LISTEN 0 511 0.0.0.0:80    0.0.0.0:* users:(("nginx",pid=5678,fd=7))
```

The Java example accepts local IPv4 traffic only. nginx listens on all local IPv4 interfaces, but a route and security permissions are still required. The text is representative output, not a report of services on your actual machine.

From another authorised machine, test the intended destination and port:

```bash
nc -vz your-server.example 443
curl -vI https://your-server.example
```

Use your actual host in place of the example. A TCP success precedes TLS/HTTP validation. "Connection refused" commonly means an active rejection or no listener; timeout has several possible causes, including filtering and routing. For HTTPS virtual hosting, requesting a bare IP can fail because hostname/SNI/certificate behaviour differs, so it is not a complete replacement for testing the name.

## Interpret certificate evidence

```bash
# OpenSSL, Linux/macOS shell
openssl s_client -connect example.com:443 -servername example.com -showcerts -verify_hostname example.com -verify_return_error </dev/null
```

Inspect the names, dates, chain and verification outcome. SNI selects the intended site; explicit hostname verification checks the requested name. The relevant CA must be in the client's trust configuration. `-showcerts` shows what the server sends, not a guarantee that the chain is trusted.

```text
Possible error                 Evidence to inspect
expired / not yet valid        certificate dates and client clock
hostname mismatch              requested name and certificate SANs
unable to get local issuer     intermediate chain and trusted CA store
wrong certificate              SNI, listener and virtual-host configuration
```

**SAN**, Subject Alternative Name, lists certificate names. Revocation handling is client-dependent. Renewal needs monitoring and automation; do not rely on a fixed issuer lifetime in study notes. On Windows, `curl.exe -vI https://example.com` and the browser's certificate viewer provide useful checks without a Unix pipeline.

## Read a least-privilege rule table

For the application path, ask which component initiates each connection:

| Initiator | Destination | Protocol/port | Corresponding inbound permission |
|---|---|---|---|
| Public user | Load balancer | TCP 443 | `alb-sg` from intended public clients |
| Load balancer | App | TCP 8080 | `app-sg` from `alb-sg` |
| App | MySQL | TCP 3306 | `db-sg` from `app-sg` |
| Admin access mechanism | Managed host | As chosen | Restricted administrative path |
| App | External dependency | Only required protocols/ports | External endpoint's policy |

Outbound permissions are required for **new** connections. Tracked replies do not need independent SG return rules. NACLs are stateless and do need corresponding permissions. Group references do not create routes or replace authentication.

A read-only AWS CLI view can help inspect actual rules in your own account:

```bash
aws ec2 describe-security-groups --group-ids sg-YOURGROUP --output json
```

Replace the placeholder and use the intended account/region. Inspect inbound and outbound permissions, both IPv4 and IPv6 ranges, transport, port ranges and referenced groups. A world-open HTTPS entry point can be deliberate; world-open SSH or a database needs a much stronger justification. Avoid treating every `0.0.0.0/0` rule as the same risk regardless of service and direction.

## UFW examples with the bind and network path kept separate

The following changes a Linux host's policy. It is included to explain the lecture's commands; use the correct administrative source for an actual host and preserve required access before enabling it.

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow from 203.0.113.10 to any port 22 proto tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
sudo ufw status verbose
```

`203.0.113.10` is a documentation placeholder. `allow OpenSSH` is an alternative application-profile allow, usually broader than a source-specific rule. `sudo ufw status numbered` shows rule numbers; `sudo ufw delete allow 80/tcp` removes the matching allow. Inspect before deletion, since a changed policy can terminate needed access.

Published container ports need separate verification: container runtimes can create forwarding rules that do not behave like ordinary host-input traffic. Check the runtime's binding and firewall integration rather than assuming UFW automatically covers every container path.

## Connect CIDR planning to actual routes

The lecture's example uses `10.0.0.0/16` with three public/private subnet pairs:

```text
AZ a: public 10.0.1.0/24, private 10.0.11.0/24
AZ b: public 10.0.2.0/24, private 10.0.12.0/24
AZ c: public 10.0.3.0/24, private 10.0.13.0/24

Public:      local VPC route + 0.0.0.0/0 -> IGW
Private app: local VPC route + 0.0.0.0/0 -> public NAT, if needed
Data-only:   optional separate subnets with no Internet default route
```

No two listed subnets overlap. A `/24` has 256 total IPv4 addresses, but provider reservations reduce assignable addresses. A NAT-routed private subnet permits an outbound routing path; its database does not magically lack that route if it shares the subnet. Separate data routing is an optional refinement.

For multi-AZ resilience, consider whether each zone has its own required NAT/dependency path. More components can improve isolation and availability but also add cost and operating work. Infrastructure deletion must account for dependent resources; the lecture's disposable-console cleanup is not a universal cascade-deletion rule.

## Optional production context

Further reading may cover IAM policy authoring, private endpoints, WAF rules, certificate pinning/mTLS, hybrid networking and incident response. First be able to explain one allowed connection, one denied connection, one certificate failure and one NAT-routed request. Those four explanations support the [essential interview questions](Part_04_Interview_Questions.md) without requiring additional cloud exercises.