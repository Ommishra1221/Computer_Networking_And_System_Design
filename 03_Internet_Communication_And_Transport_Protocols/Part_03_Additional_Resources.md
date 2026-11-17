# Part 03 — Resources and Practical Commands

Use these references to clarify a specific mechanism after reading the [study notes](Part_03_Study_Notes.md). They are optional reading, not a second required course. Commands below inspect behaviour rather than creating a lab project. Outputs are illustrative and can differ by OS, server and negotiated protocol.

## Focused references

| Reference | Read for this purpose |
|---|---|
| [RFC 9293 — TCP](https://www.rfc-editor.org/rfc/rfc9293) | Primary specification: introduction, header, connection establishment/close and state diagram. Do not begin with the entire specification. |
| [RFC 768 — UDP](https://www.rfc-editor.org/rfc/rfc768) | Primary specification: short header format and the delivery guarantees UDP does not provide. |
| [MDN — Overview of HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview) | Request/response messages, components and HTTP's stateless semantics. |
| [MDN — HTTP guides](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Overview) | Look up a specific question about caching, cookies, headers or HTTP versions. |
| [Cloudflare — OSI model](https://www.cloudflare.com/learning/ddos/glossary/open-systems-interconnection-model-osi/) | Explanatory seven-layer overview. Treat TLS/session mappings as conceptual, not rigid program boundaries. |
| [Cloudflare — TCP/IP](https://www.cloudflare.com/learning/ddos/glossary/tcp-ip/) | Explanatory connection between the deployed suite, IP and TCP. |
| [Cloudflare — TLS](https://www.cloudflare.com/learning/ssl/transport-layer-security-tls/) | Explanatory security layer behind HTTPS, continued in Part 4. |
| [Wireshark User's Guide](https://www.wireshark.org/docs/wsug_html_chunked/) | Capture on the correct interface, display filters and following a connection. |
| [OpenSSL s_client manual](https://docs.openssl.org/master/man1/openssl-s_client/) | Meaning of SNI, certificate display and verification flags. Installed versions can differ from this reference. |

The original resources also link [Julia Evans's networking zine PDF](https://jvns.ca/networking-zine.pdf). It is optional and **was not reviewed during this rewrite**: retrieval exceeded the web tool's 10 MiB limit, and the local download attempt could not connect. The local study notes and diagrams supply the required course coverage without depending on it.

## Read an HTTP exchange with curl

On Windows PowerShell, use `curl.exe` to select the real curl program instead of an older PowerShell alias. On Linux/macOS, use `curl`.

```powershell
curl.exe -v https://example.com
curl.exe -I https://example.com
curl.exe -I http://example.com
```

```text
> GET / HTTP/1.1
> Host: example.com
< HTTP/1.1 200 OK
< Content-Type: text/html
```

In verbose output, `>` denotes a request line/header sent by curl and `<` a received response line/header. The example assumes HTTP/1.1; a different negotiated version changes the presentation. `-I` requests headers using HEAD rather than downloading a normal GET body. Plain HTTP may redirect to HTTPS, but a redirect is server policy, not a guaranteed response from every site.

Curl can show the HTTP messages because it is a TLS endpoint and has decrypted them. This does not mean an ordinary on-path capture can read HTTPS content. An HTTP 200 establishes that one request succeeded, not that every application workflow is healthy.

## Separate DNS, TCP, TLS and first-byte time

```bash
# Linux/macOS, fresh connection with no redirect-following flag
curl -s -o /dev/null -w 'dns=%{time_namelookup} connect=%{time_connect} tls=%{time_appconnect} first=%{time_starttransfer} total=%{time_total}\n' https://example.com
```

On Windows, the same arguments can be used with `curl.exe` and `-o NUL` instead of `/dev/null`.

```text
dns=0.020 connect=0.080 tls=0.140 first=0.200 total=0.230
```

These are cumulative seconds from the start, not independent phase durations. Here DNS took about 20 ms, TCP setup another 60 ms, TLS another 60 ms, the time from TLS completion to first response byte another 60 ms, and the remaining download about 30 ms. That first-byte interval includes request transmission and response travel as well as server processing. Caches, proxies, reuse, redirects and HTTP/3 can change interpretation.

## Observe connection state and listening addresses

```powershell
# Windows
netstat -ano | findstr TCP
netstat -ano | findstr LISTENING
Test-NetConnection example.com -Port 443
```

```bash
# Linux
ss -tan
sudo ss -tulpn
nc -vz example.com 443
```

```text
TCP  127.0.0.1:8080       0.0.0.0:0          LISTENING    1234
TCP  192.168.1.23:54321   203.0.113.10:443   ESTABLISHED  5678
```

The first listener is IPv4 loopback-only. The second row represents an established TCP connection; port 54321 is the client's temporary endpoint. PID identifies the local process. Documentation addresses in examples are placeholders, not live targets.

A successful TCP port test proves that a handshake completed. A refusal often means no listener or active rejection. A timeout may indicate filtering, routing trouble, loss or an unavailable destination. Neither result by itself proves an application or certificate problem.

## Use Wireshark to connect diagrams to packets

Capture your own test traffic on the active interface, make one curl request and stop the capture. Display filters are applied after capture; they do not themselves create traffic.

| Display filter or action | Evidence to inspect |
|---|---|
| `tcp.flags.syn == 1` | SYN and SYN-ACK packets identifying new connections |
| Select the connection, then Follow → TCP Stream | Isolate one flow and find its final handshake ACK |
| `tcp.stream eq 3` | Example stream filter only: use the stream number in your own capture |
| `tcp.analysis.retransmission` | Suspected retransmissions, interpreted with capture completeness in mind |
| `dns` | Visible plain DNS queries/replies if the client uses them |
| `tls` | TLS handshake/protected records when recognised by the dissector |

Expand a packet's link, IP and transport sections to see encapsulation. Wireshark commonly displays relative TCP sequence numbers, so the handshake can appear as sequence 0 and acknowledgement 1 rather than the raw initial numbers. Capture loss can resemble network loss; a diagnostic flag is evidence to investigate, not an infallible verdict.

HTTPS application data remains encrypted unless the capture is supplied with appropriate session secrets. Encrypted DNS may not appear as plain `dns`. A fresh client request also may use an existing connection in a browser, so not every page load produces a new TCP/TLS handshake.

## Check DNS with both supported transports

```bash
# dig is commonly available on Linux/macOS or through suitable tools/WSL
# Ordinary DNS default query and explicit TCP query
dig example.com A
dig example.com A +tcp
```

Both can return the same DNS record. UDP avoids TCP connection establishment for a small exchange; TCP can be selected directly and is used when a truncated UDP answer requires retry, among other cases. The DNS answer's TTL concerns caching, not IP hop lifetime. Windows `nslookup example.com` is useful when dig is unavailable; Part 2 covers its interpretation.

## Inspect a TLS endpoint

```bash
# Linux/macOS shell, with OpenSSL installed
openssl s_client -connect example.com:443 -servername example.com -verify_hostname example.com -verify_return_error </dev/null
```

```text
Certificate chain
 0 ... server certificate ...
 1 ... intermediate certificate ...
Verification: OK
```

The output is illustrative. `-servername` supplies SNI, while `-verify_hostname` explicitly asks for hostname validation and `-verify_return_error` makes verification errors fail the handshake. Trust-store configuration affects the result. Displaying a certificate is not the same as validating it.

To print fields from the leaf certificate in a Unix-style shell:

```bash
openssl s_client -connect example.com:443 -servername example.com </dev/null 2>/dev/null | openssl x509 -noout -subject -issuer -dates -ext subjectAltName
```

That pipeline is for **inspection**; it suppresses connection diagnostics and does not establish that every verification check succeeded. A browser certificate viewer or `curl.exe -vI https://example.com` is often simpler on Windows.

## Keep the first pass focused

You can explain this part without memorising TCP algorithm variants, socket-programming APIs, TLS key schedules, QUIC internals or HTTP header compression. Retain the mechanisms: envelopes, endpoints, byte recovery, datagram trade-offs and observable protocol steps. Use the [interview scenarios](Part_03_Interview_Questions.md#optional-applied-scenarios) if you want a little more depth after that foundation.