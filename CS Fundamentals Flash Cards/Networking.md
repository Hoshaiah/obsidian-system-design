---
tags:
  - flashcards/networking
---

# Networking

The stack from Ethernet frames to HTTPS, with focus on what affects application performance and reliability.

---

What are the OSI and TCP/IP models and why does the TCP/IP model dominate in practice?
?
**Quick:** OSI has 7 layers (physical to application); TCP/IP collapses them into 4 (link, internet, transport, application) and matches real protocols.<br>
**Deeper:** OSI is a teaching tool; nobody builds clean 7-layer stacks. TCP/IP's pragmatic 4 layers (Ethernet/IP/TCP/HTTP) reflect how the Internet actually runs. Useful interview frame: identify the layer of a problem (a "DNS issue" is application-layer; a "MTU issue" is link/internet) to know which tool to reach for (curl vs traceroute vs tcpdump).

What's the difference between TCP and UDP, and when do you pick UDP?
?
**Quick:** TCP is reliable, ordered, connection-oriented, congestion-controlled; UDP is none of these but is low-overhead.<br>
**Deeper:** Pick UDP when latency matters more than completeness: real-time video/voice, gaming, DNS, QUIC. TCP head-of-line blocking and 3-way handshake are unacceptable for these. Modern stacks (QUIC/HTTP3) rebuild reliability on top of UDP at the application layer to avoid kernel TCP's limitations.

How does the TCP three-way handshake work and why does it exist?
?
**Quick:** SYN -> SYN-ACK -> ACK establishes initial sequence numbers and confirms both sides can send and receive.<br>
**Deeper:** Sequence number randomization defends against blind injection attacks. The handshake adds 1 RTT before any data, which is why TCP Fast Open and 0-RTT (in TLS 1.3 / QUIC) exist. SYN floods exploit the half-open state on the server; SYN cookies are a mitigation that avoids storing state until the third packet.

What is TCP congestion control and what algorithm does Linux use by default?
?
**Quick:** Congestion control regulates send rate to avoid overwhelming the network; modern Linux defaults to CUBIC, with BBR available.<br>
**Deeper:** CUBIC grows the congestion window aggressively then backs off on packet loss, treating loss as congestion. BBR (Google) models the bottleneck bandwidth and RTT directly and ignores loss — much better on lossy or buffer-bloated links. Misconfigured buffers cause bufferbloat: huge latency without packet loss because queues fill before any loss signal appears.

How does DNS resolution work end to end?
?
**Quick:** A client recursive resolver walks from root -> TLD -> authoritative servers, caching results by TTL.<br>
**Deeper:** Most clients hit a configured resolver (8.8.8.8, 1.1.1.1, ISP), which does the recursion. Cached answers return in microseconds; cold lookups can take 100s of ms. DNS over HTTPS/TLS encrypts queries to prevent ISP surveillance and tampering. TTL is a critical operational lever: low TTLs enable fast failover but increase resolver load. CNAME chains and ANAME records can multiply lookups.

What is the difference between HTTP/1.1, HTTP/2, and HTTP/3?
?
**Quick:** HTTP/1.1 is text and one-request-at-a-time per connection; HTTP/2 multiplexes streams over one TCP connection; HTTP/3 runs over QUIC/UDP for better loss recovery.<br>
**Deeper:** HTTP/2 solved head-of-line blocking at the HTTP layer but TCP-level HOL blocking remained — one lost packet stalls all streams. HTTP/3 fixes this by giving each stream its own loss recovery in QUIC. HTTP/2 also added HPACK header compression. Server push has been deprecated. For latency-critical apps, HTTP/3 is meaningfully better on mobile networks.

How does TLS work at a high level?
?
**Quick:** Asymmetric crypto to exchange a symmetric session key, then symmetric crypto for bulk encryption; server identity proven via certificate chain.<br>
**Deeper:** TLS 1.3 reduced the handshake to 1-RTT (0-RTT for resumption) by dropping legacy ciphers and using ECDHE for forward secrecy. Cert chain: leaf -> intermediates -> root CA in the OS trust store. SNI lets one IP serve many domains; ESNI/ECH now hides which domain you're contacting. mTLS adds client cert authentication, common in zero-trust networking.

What's the difference between a forward proxy, reverse proxy, and load balancer?
?
**Quick:** Forward proxy fronts clients; reverse proxy fronts servers; load balancer is a reverse proxy that distributes among backends.<br>
**Deeper:** Forward proxies (corporate egress, browser proxies) hide clients and enforce policy. Reverse proxies (nginx, Envoy) terminate TLS, cache, route, and load balance. L4 load balancers route by TCP/UDP; L7 routes by HTTP host/path/headers. Modern service meshes use sidecar proxies (Envoy) for observability, mTLS, and traffic shaping.

What does NAT do and why does it complicate things?
?
**Quick:** NAT rewrites private IP:port to a public IP:port so many devices share one IP; complicates inbound connections and P2P.<br>
**Deeper:** Stateful mapping in the NAT means inbound packets without an existing flow get dropped, breaking servers behind NAT. Solutions: STUN (discover public address), TURN (relay through a server), ICE (negotiate the best path), UPnP/PCP (request port mapping). IPv6 was supposed to eliminate NAT but adoption is slow.

What are well-known port ranges and why does it matter?
?
**Quick:** 0-1023 well-known (require root on Unix), 1024-49151 registered, 49152-65535 ephemeral.<br>
**Deeper:** Listening on ports below 1024 requires elevated privileges or capabilities (CAP_NET_BIND_SERVICE), affecting container design. Ephemeral port exhaustion is a common production issue: each outbound TCP connection consumes a local port for the TIME_WAIT period (~60s). Heavy outbound traffic to a small set of remote IP:port pairs can exhaust the 4-tuple space.

What is the difference between idempotent and safe HTTP methods?
?
**Quick:** Safe = no side effects (GET, HEAD); idempotent = same effect if repeated (GET, PUT, DELETE); POST is neither.<br>
**Deeper:** Idempotency lets clients retry without fear, which is critical for unreliable networks. Real APIs make POST idempotent via an Idempotency-Key header (Stripe pattern). Caches and CDNs assume GET/HEAD are safe; breaking that assumption (state-changing GETs) leads to subtle bugs and security holes like CSRF.

What happens when you type google.com into a browser?
?
**Quick:** DNS lookup, TCP/QUIC connect, TLS handshake, HTTP request, server response, render.<br>
**Deeper:** This canonical interview question tests breadth: browser cache check, OS resolver cache, recursive DNS, ARP for the gateway MAC, TCP handshake (or 0-RTT QUIC), TLS handshake with SNI + cert validation, HTTP request, possibly through CDN edge, server-side routing/DB/cache, HTML parsing, subresource fetches, JS execution, paint. Each stage has failure modes and optimizations worth discussing.
