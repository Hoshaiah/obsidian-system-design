---
tags:
  - flashcards/aws-api-gateway
---

# API Gateway

REST vs HTTP vs WebSocket APIs, integrations, auth, and caching for DVA-C02.

---

What are the three API Gateway types and when do you use each?
?
**Quick:** REST API (full features), HTTP API (cheap/fast), WebSocket API (bidirectional).<br>
**Deeper:** REST API: usage plans, API keys, request validation, caching, WAF, private endpoints — most features, highest cost. HTTP API: ~70% cheaper, lower latency, JWT auth, CORS — fewer features (no caching, no usage plans). WebSocket: stateful, $connect/$disconnect/$default routes. Exam trick: "cheapest option for a simple proxy to Lambda" = HTTP API; "API keys and usage plans" = REST API.

What is the API Gateway integration timeout?
?
**Quick:** 29 seconds max for REST and HTTP APIs.<br>
**Deeper:** Even if Lambda allows 15 minutes, API Gateway cuts the connection at 29 seconds. Exam gotcha: long-running tasks must be async — return 202 with a job ID, do the work in the background (Step Functions, SQS+Lambda), and poll or notify. WebSocket APIs are not bound by this since they're streaming.

What is the difference between Lambda proxy and non-proxy integration?
?
**Quick:** Proxy passes raw request to Lambda; non-proxy uses mapping templates to transform.<br>
**Deeper:** Proxy integration (AWS_PROXY) sends the full event {headers, body, queryStringParameters, ...} and Lambda must return {statusCode, headers, body}. Non-proxy lets you write VTL mapping templates to reshape requests/responses. Exam trick: proxy = simplest, default, most common; non-proxy = "transform XML to JSON without changing the backend."

What are the four API Gateway authorizer types?
?
**Quick:** IAM, Cognito User Pools, Lambda (token/request), and (HTTP API only) JWT.<br>
**Deeper:** IAM uses SigV4 — best for internal/service-to-service. Cognito authorizer validates user pool tokens. Lambda authorizer (formerly custom) runs your code returning an IAM policy — most flexible, can cache by token for performance. HTTP API has built-in JWT authorizer for OIDC providers. Exam trick: "authenticate via Google/Facebook" = Cognito; "custom OAuth flow" = Lambda authorizer.

How do usage plans and API keys work?
?
**Quick:** API keys identify clients; usage plans enforce throttle and quota per key.<br>
**Deeper:** Usage plans tie keys to stages and define rate (RPS), burst, and daily/weekly/monthly quotas. API keys are NOT auth — they're for metering and identification only. Exam gotcha: "rate limit per customer" = usage plans with API keys; pair with a real authorizer for actual security.

How does API Gateway caching work?
?
**Quick:** Stage-level cache (REST only), 0.5 GB to 237 GB, TTL up to 1 hour.<br>
**Deeper:** Default TTL 300 seconds, max 3,600. Per-method cache key overrides via query strings or headers. Cache invalidation via header Cache-Control: max-age=0 requires IAM permission. Exam gotcha: HTTP APIs do NOT support caching — only REST APIs do. Use CloudFront in front of HTTP API if you need edge caching.

What are the API Gateway throttle limits?
?
**Quick:** Account-level 10,000 RPS / 5,000 burst per region; per-method configurable.<br>
**Deeper:** When throttled, API Gateway returns 429 Too Many Requests. Throttling order: per-client (API key usage plan) → per-method → per-stage → account. Exam gotcha: 429 means client should back off; 504 from API Gateway often means Lambda timeout (vs 502 = bad Lambda response format).

What is API Gateway stage and deployment?
?
**Quick:** Deployments are immutable snapshots; stages (dev/prod) point to a deployment.<br>
**Deeper:** Changes to the API don't take effect until you "Deploy API" to a stage. Stage variables ($stageVariables.foo) can parameterize Lambda function names, endpoints, etc. — useful for prod-vs-dev routing without changing the API. Canary deployments split traffic between current and new versions at the stage level (REST API only).

What CORS configuration does API Gateway need?
?
**Quick:** Enable CORS to respond to browser preflight OPTIONS requests.<br>
**Deeper:** Console "Enable CORS" creates an OPTIONS method with Mock integration returning Access-Control-Allow-* headers. For Lambda proxy, you also need to return CORS headers in the Lambda response (proxy passes through). HTTP API has simpler CORS config in the API settings. Exam trick: CORS errors despite "CORS enabled" usually mean Lambda isn't returning the headers in proxy integration.

What are mapping templates and when do you need them?
?
**Quick:** VTL (Velocity Template Language) snippets that transform requests/responses in non-proxy integrations.<br>
**Deeper:** Use $input.json('$'), $input.params(), $context.identity, etc. Common in HTTP integrations to translate REST to a SOAP backend, or to reshape between API contract and Lambda's internal format. Exam gotcha: mapping templates apply per Content-Type — if request comes in as application/xml but template is for application/json, you'll get 415 Unsupported Media Type.

What are the three API Gateway endpoint types?
?
**Quick:** Edge-optimized (CloudFront), Regional, and Private (VPC only).<br>
**Deeper:** Edge-optimized routes through CloudFront edge — good for global clients. Regional terminates in one region — better for in-region or when you already have your own CloudFront. Private exposes the API only inside a VPC via interface endpoint — for internal microservices. Exam trick: "API for our internal services in VPC, no public internet" = Private endpoint with VPC endpoint policy.

How does API Gateway integrate with WebSockets?
?
**Quick:** Routes by $connect, $disconnect, $default, and custom route keys.<br>
**Deeper:** Connection ID is given on $connect; use the @connections API (POST to /@connections/{id}) to push messages from the backend to clients. Store connection IDs in DynamoDB to track active clients. Exam trick: "broadcast to all connected users" — Lambda iterates DynamoDB and posts to each connection via the management API.

What are common API Gateway error codes you should know?
?
**Quick:** 400 bad request, 401/403 auth, 429 throttled, 502 bad integration, 504 integration timeout.<br>
**Deeper:** 401 = authorizer denied (no/invalid token); 403 = WAF or resource policy denied; 429 = throttled by usage plan or account limit; 502 = backend returned malformed response (e.g., Lambda non-JSON); 504 = integration timed out (Lambda took >29s). Exam favorite: distinguishing 502 (bad response) from 504 (timeout).
