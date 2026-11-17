# Part 04 — Interview Questions

Eight essential questions cover the fresher-level concepts. Four optional scenarios add practical judgement. Explain the control, the permitted path and its limitation; use the linked notes for port lists and configuration details.

## Eight essential questions

### 1. What does a firewall control, and how do ingress and egress differ?

A firewall permits or blocks traffic using policy, often considering addresses, ports, protocol and state. A host firewall runs inside the OS; a network firewall filters elsewhere along the path. Ingress enters the boundary being discussed; egress leaves it. Replies to outbound-initiated connections are ingress packets. Stateful filtering recognises permitted replies, while stateless filtering evaluates each packet independently. Egress restrictions can limit attacker callbacks or data theft. Firewalls reduce reachable attack surface, but do not repair application vulnerabilities or replace authentication.

### 2. Compare AWS security groups and network ACLs.

| Property | Security group | Network ACL |
|---|---|---|
| Association | Interface/resource | Subnet |
| State | Stateful | Stateless |
| Rules | Allow only | Allow and deny |
| Evaluation | Permissions combine | Numbered order, first match |
| Replies | Tracked replies permitted | Both directions need permission |

Multiple attached SGs combine permissions; an SG has no explicit deny that overrides another SG's allow. NACLs can add subnet-level restrictions, and a broad SG allow cannot bypass them. Inspect actual rules rather than assuming every ordinary/default group has identical initial permissions.

### 3. How would you allow a load balancer, app and database to communicate safely?

Allow public HTTPS to the load balancer, the app's service port from the load balancer's SG, and the database port from the app's SG. Permit necessary newly initiated outbound connections; tracked replies are automatic. SG references follow eligible resource membership rather than hardcoded instance IPs. They do not create routes, copy rules or identify logged-in users. Restrict administration separately. `0.0.0.0/0` means every IPv4 address: public HTTPS may be deliberate, while unnecessary public SSH/database exposure increases risk.

### 4. nginx works on localhost but not from another machine. How do you investigate?

Check the process and listener with `systemctl`/`ss`. A loopback bind accepts local traffic only; a wildcard bind still needs network access. Check host firewall, SG source/protocol/port, NACL directions, public entry point/address and route table. Then test remotely with a port probe and `curl -v`. Local success narrows the fault without proving the firewall is responsible. Successful TCP still leaves TLS/HTTP unresolved. A bare-IP HTTPS test can differ because of certificate names, SNI and virtual-host configuration.

### 5. What does TLS provide, and what does a client validate?

TLS provides confidentiality, integrity and normally server authentication. A certificate binds a public key to names; the server proves possession of the matching private key. The client checks hostname, dates, allowed use, trust chain and handshake proof. Revocation handling varies by client. In a normal full TLS 1.3 exchange, hello messages exchange key shares before protected server-authentication messages and Finished confirmation. Do not postpone all key establishment until after certificate validation. SSL is obsolete, despite the informal phrase "SSL certificate."

### 6. Why does allowing port 443 not create HTTPS or prevent phishing?

A network rule permits traffic; the service still needs a TLS listener, suitable certificate/private key and correct configuration. A valid certificate secures the named endpoint, not its intentions: a phishing domain can obtain one. TLS does not fix application bugs or authorise users. Failures can involve names, dates, intermediates, trust or the client clock. Automate renewal and monitor expiry instead of assuming a fixed lifetime. HTTP content is protected, while destination addressing and some metadata can remain visible.

### 7. What makes a subnet public, and how do IGW and NAT differ?

A public subnet has an internet-gateway route; direct public IPv4 instance access additionally needs suitable addressing, permissions and a listener. Private subnets lack a direct IGW route but may use public NAT for outbound access: private app → NAT in public subnet → IGW → Internet. Replies follow existing mappings; unsolicited users do not enter through NAT. NAT translates addresses, while SGs/NACLs filter. A DB sharing a NAT-routed app subnet shares that route; local-only data routing requires a separate subnet arrangement.

### 8. Explain defence in depth and shared responsibility.

Combine identity controls, network placement/filtering, host hardening, application authentication/authorisation, data protection/backups and monitoring. Each covers gaps in others: private placement does not stop a compromised internal app, and HTTPS does not replace input validation. The provider secures its underlying infrastructure; the customer secures identities, configuration, code and data according to the service model. Least privilege reduces a failure's blast radius. Leaked credentials, public databases, excessive permissions and missing logs/backups remain risks despite a firewall.

## Optional applied scenarios

### 9. Sketch a three-AZ deployment and choose where TLS ends.

Match the lecture with non-overlapping public/private subnet pairs across three AZs, public load-balancer nodes and controlled private app/data paths. Separate local-only data routing requires additional data subnets. A public NAT per AZ reduces cross-zone egress dependence but costs more; private endpoints can serve supported cloud dependencies. Redundancy must include application and data services. Terminate TLS at the load balancer for centralised certificates/HTTP routing, re-encrypt to protect the backend hop, or pass encrypted traffic through to backend termination. Each adds different visibility/configuration trade-offs. Public-facing HTTPS alone does not prove encryption across every internal hop.

### 10. An access key was committed publicly. What do you do?

Treat it as compromised: promptly revoke/deactivate and replace it, or move to an appropriate short-lived role. Investigate audit logs for misuse, new identities, policy changes, data access, resource creation and billing anomalies. Contain affected resources, remove persistence, recover with least privilege and follow the organisation's incident process. Removing the key from the latest commit, rewriting history or rotating without investigation is incomplete: somebody may already have copied it or used it to create another access path. Add secret scanning, appropriate permissions, credential lifecycle controls and monitoring to reduce recurrence.

### 11. How would you respond to DDoS at different layers?

Identify the exhausted resource. Volumetric attacks saturate bandwidth and often require upstream/provider absorption. SYN floods target connection-establishment state and need transport/upstream controls. HTTP floods or expensive requests consume application capacity and may need WAF/rate controls, caching, query limits and graceful degradation. A CDN can absorb suitable cacheable traffic, but caching does not solve every attack. Autoscaling may help legitimate demand while also increasing attack costs; it is not a complete defence. Monitor traffic, errors and saturation, and maintain an incident path to the provider. Match protection to the mechanism rather than one generic "security" setting.

### 12. How do you restrict egress without treating an internal network as trusted?

Inventory required destinations and protocols, including DNS, updates, APIs and monitoring. Permit necessary paths, consider private endpoints, and use a suitable proxy/application-aware control for domain-based policy. A plain NAT gateway does not provide a domain allow-list. Review denied traffic and roll changes out carefully. Zero trust means private location alone grants no application trust: authenticate/authorise requests, limit service permissions and protect communications. Segmentation still matters. Cross-VPC connectivity/references have specific supported cases and restrictions; verify the actual topology rather than assuming internal resources are automatically connected or trusted. Choose an auditable private administration path separately.

For port conventions, bind examples, UFW/container caveats and routes, use the [notes](Part_04_Study_Notes.md), [traffic diagrams](Part_04_Diagrams.md) and [command evidence](Part_04_Additional_Resources.md).