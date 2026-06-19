---
tags:
  - flashcards/security
---

# Security Fundamentals

The authn/authz model, crypto primitives, and common web attack patterns every senior engineer should reason about.

---

What's the difference between authentication and authorization?
?
**Quick:** Authentication proves who you are; authorization determines what you can do.<br>
**Deeper:** Authn happens once (login, token issuance); authz happens on every request. Conflating them is a common source of bugs — e.g., an API correctly authenticates a user but skips checking whether they own the resource they're modifying (IDOR). Modern stacks: OIDC/SAML for authn, RBAC/ABAC/policy engines (OPA, Cedar) for authz.

What's the difference between hashing and encryption?
?
**Quick:** Hashing is one-way (no key, no reverse); encryption is two-way (key recovers plaintext).<br>
**Deeper:** Hash passwords — never encrypt them. Encrypt data you need to read later (DB fields, files, tokens). Don't confuse with HMAC (keyed hash for integrity). Common mistake: storing passwords with a fast hash like MD5 or SHA-256. Use slow, salted KDFs (bcrypt, scrypt, Argon2) designed to resist GPU brute force.

What's the difference between symmetric and asymmetric encryption?
?
**Quick:** Symmetric uses one shared key (fast); asymmetric uses a public/private key pair (slow, but solves key distribution).<br>
**Deeper:** Symmetric (AES) is used for bulk data. Asymmetric (RSA, ECC) is used to exchange or sign — usually to bootstrap a symmetric session key (TLS, age, PGP). Never roll your own; use vetted libraries with authenticated encryption (AES-GCM, ChaCha20-Poly1305) to prevent tampering as well as confidentiality.

Why do we salt password hashes and what is a pepper?
?
**Quick:** Salt is a unique random value per password preventing rainbow tables; pepper is a global secret stored separately for defense in depth.<br>
**Deeper:** Salt forces attackers to brute force each hash individually instead of precomputing one rainbow table for all users. Salts are stored alongside the hash; peppers are kept in HSM/secrets manager, so a DB-only leak doesn't enable offline cracking. Modern KDFs (Argon2id) handle salting and slow hashing automatically; just configure the cost parameters.

What is SQL injection and how do you prevent it?
?
**Quick:** Attacker injects SQL syntax through user input; prevent with parameterized queries (prepared statements), never string concatenation.<br>
**Deeper:** Even ORMs can be vulnerable when developers fall back to raw SQL. Defense in depth: least-privilege DB users, query allow-lists, WAF rules, input validation. Parameterized queries also help the query planner (plan reuse) so they're a perf win too. Stored procedures alone don't help if they internally concatenate.

What is XSS and what are the categories?
?
**Quick:** Cross-Site Scripting injects attacker JS into a page; types are reflected (URL), stored (DB-persisted), and DOM-based (client-side).<br>
**Deeper:** Defenses: context-aware output encoding (HTML, attribute, JS, URL), CSP headers restricting script origins, HttpOnly cookies (so stolen XSS can't read auth tokens), Trusted Types in modern browsers. Modern frameworks (React, Vue) escape by default; the bugs creep in via `dangerouslySetInnerHTML` and friends.

What is CSRF and how is it mitigated?
?
**Quick:** Attacker's site triggers state-changing requests on yours using the victim's cookies; mitigate with anti-CSRF tokens or SameSite cookies.<br>
**Deeper:** Browsers automatically send cookies on cross-origin requests, so a hidden form on attacker.com could POST to your bank. Defenses: same-site cookies (Lax/Strict now the default), CSRF tokens validated server-side, double-submit cookies, Origin/Referer header checks. APIs using bearer tokens (not cookies) are immune by construction.

What is JWT and what are its common pitfalls?
?
**Quick:** JSON Web Token is a signed (sometimes encrypted) token carrying claims; pitfalls include the `alg: none` bug, weak secrets, and no easy revocation.<br>
**Deeper:** JWTs are stateless — that's the benefit (no DB lookup) and the curse (can't easily revoke before expiry). Common bugs: accepting unsigned tokens (`alg: none`), accepting HS256 when expecting RS256 (algorithm confusion), storing them in localStorage (XSS-readable). Short-lived access tokens + refresh tokens minimize the revocation problem.

What is OAuth 2.0 and how does it differ from OIDC?
?
**Quick:** OAuth 2.0 is an authorization framework for delegated access; OIDC adds an identity layer (ID token) on top for authentication.<br>
**Deeper:** OAuth issues access tokens that let one service act on a user's behalf at another. OIDC adds the ID token (a signed JWT proving identity claims) for SSO/login. The authorization code flow with PKCE is the modern standard for web/mobile. Common mistake: using OAuth access tokens as identity proof; the spec says they're opaque to the client.

What are the OWASP Top 10 categories at a high level?
?
**Quick:** A rotating list of the most critical web vulns — broken access control, crypto failures, injection, insecure design, misconfig, vulnerable components, authn failures, data integrity, logging failures, SSRF.<br>
**Deeper:** Broken access control (#1 in 2021) covers IDOR, missing function-level auth, and authorization logic bugs. The list shifts emphasis over time: SSRF moved up as cloud metadata endpoints became targets. Use the Top 10 as a checklist during threat modeling and security reviews, not as the only framework.

What is TLS and how does the handshake achieve confidentiality and authenticity?
?
**Quick:** Asymmetric handshake exchanges a session key with forward secrecy; symmetric encryption (AEAD) protects messages; server certificate proves identity.<br>
**Deeper:** TLS 1.3 strips legacy options, defaults to ECDHE (forward secrecy) and AEAD ciphers (AES-GCM, ChaCha20-Poly1305), and reduces handshake to 1-RTT (0-RTT for session resumption). mTLS adds client cert authn for zero-trust. Cert validation traverses a chain to a trusted root; pinning further restricts to expected certs (but is rarely worth the ops cost).

What is SSRF and why does it matter in cloud environments?
?
**Quick:** Server-Side Request Forgery tricks your server into making attacker-controlled requests, often to internal services.<br>
**Deeper:** Classic attack: a feature lets users provide a URL the server fetches; attacker points it at 169.254.169.254 (AWS metadata endpoint) to steal IAM credentials. Defenses: deny internal IP ranges, use IMDSv2 (requires session token), egress proxies/allow-lists, network segmentation. SSRF was central to the 2019 Capital One breach.

What is the principle of least privilege and how does it apply at multiple layers?
?
**Quick:** Give every actor (user, service, process) only the permissions they need; minimizes blast radius of any compromise.<br>
**Deeper:** Apply at every layer: DB users with table-level grants, service IAM roles scoped to required actions, OS users without sudo, containers as non-root with read-only filesystems, network policies that deny by default. Combined with short-lived credentials and audit logging, this turns most breaches from catastrophic into contained.
