---
tags:
  - flashcards/aws-networking
---

# Networking — VPC, Route 53, CloudFront

VPC subnets, security, DNS routing, and CDN essentials for DVA-C02.

---

What is the difference between a public and private subnet?
?
**Quick:** Public has a route to an Internet Gateway; private does not.<br>
**Deeper:** Public subnet: route table 0.0.0.0/0 → IGW; instances need public IPs. Private subnet: no IGW route; use NAT Gateway in a public subnet for outbound internet. Exam trick: "private subnet has internet access via NAT" = outbound only — inbound from internet still blocked.

What is the difference between security groups and NACLs?
?
**Quick:** Security groups are stateful per-ENI; NACLs are stateless per-subnet.<br>
**Deeper:** SG: allow rules only, stateful (return traffic auto-allowed), applied to ENIs, evaluate all rules. NACL: allow + deny, stateless (must allow both directions), applied at subnet, evaluated in order. Default SG denies inbound and allows all outbound. Default NACL allows all both ways. Exam trick: NACLs are blunt instruments — common answer when you need to block a specific IP range.

What is the difference between Gateway and Interface VPC endpoints?
?
**Quick:** Gateway endpoints are free for S3/DynamoDB only; Interface endpoints are paid ENIs for most services.<br>
**Deeper:** Gateway endpoints add a route table entry — no cost, no bandwidth charges. Interface endpoints (PrivateLink) are ENIs in your subnets — billed hourly + per GB, support most AWS services. Exam trick: "private subnet access S3 without NAT" = Gateway endpoint (free); "private subnet access SQS/SNS/KMS without internet" = Interface endpoint.

What is a NAT Gateway and how does it differ from a NAT Instance?
?
**Quick:** NAT Gateway is managed and HA per AZ; NAT instance is self-managed EC2.<br>
**Deeper:** NAT Gateway: AWS-managed, up to 45 Gbps, automatic scaling, $/hour + GB. NAT instance: cheaper for small, you patch + manage HA, can act as bastion. Exam best practice: one NAT Gateway per AZ for HA (cross-AZ traffic costs money + single point of failure). Deprecated direction: NAT instance.

What are the Route 53 routing policies?
?
**Quick:** Simple, Weighted, Latency, Failover, Geolocation, Geoproximity, Multi-Value Answer, IP-based.<br>
**Deeper:** Simple = single record, no logic. Weighted = % split across records. Latency = lowest-latency region for the user. Failover = active/passive with health checks. Geolocation = continent/country routing. Geoproximity = bias by distance + bias slider (requires Traffic Flow). Multi-Value = up to 8 healthy records returned. Exam trick: failover ≠ latency — failover is HA across regions, latency optimizes performance.

How do Route 53 health checks work?
?
**Quick:** Periodic checks against endpoints, with optional alarming and failover triggering.<br>
**Deeper:** Three types: endpoint (HTTP/HTTPS/TCP from 15+ AWS health checkers), CloudWatch alarm-based, calculated (combine multiple checks with AND/OR). Standard interval 30s, fast 10s ($). Exam trick: for private resources, you can't use endpoint checks directly — use CloudWatch alarm checks instead.

What is the difference between Route 53 Alias and CNAME?
?
**Quick:** Alias is AWS-only and free, works at zone apex; CNAME is DNS standard but cannot be at apex.<br>
**Deeper:** Alias points to AWS resources (ALB, CloudFront, S3 website, API Gateway) — free queries, supports zone apex (example.com). CNAME points to any DNS name but cannot exist at zone apex per DNS spec. Exam favorite: "point example.com (apex) to CloudFront" = Alias record, NOT CNAME.

What is CloudFront and how do its origins work?
?
**Quick:** CDN that caches content at 600+ edge locations; origins can be S3, ALB, EC2, HTTP endpoint, MediaStore.<br>
**Deeper:** Behaviors map URL path patterns to origins. Origin groups enable failover between primary and secondary origins. Origin Shield adds a central caching layer for higher cache hit ratio. Exam trick: "global low-latency content delivery" = CloudFront; "regional ALB caching" = no — use CloudFront in front of ALB.

What is Origin Access Control (OAC)?
?
**Quick:** Restricts S3 bucket access so only CloudFront can fetch objects.<br>
**Deeper:** Replaces deprecated Origin Access Identity (OAI). Uses SigV4 signing on origin requests; bucket policy allows the CloudFront distribution ARN. Required for SSE-KMS encrypted buckets (OAI didn't support KMS). Exam trick: "make S3 bucket private but serve via CloudFront" = OAC + restrictive bucket policy. New deployments should use OAC over OAI.

What are CloudFront signed URLs and signed cookies?
?
**Quick:** Signed URLs grant access to one file; signed cookies grant access to many files.<br>
**Deeper:** Signed URLs ideal for per-file downloads (e.g., individual videos). Signed cookies for protecting a set of files (e.g., HLS streams, premium content area) — no need to rewrite URLs. Both can include expiration, IP whitelist, date restrictions. Exam trick: video streaming with many segments = signed cookies; one-off downloads = signed URLs.

What is the difference between CloudFront cache policy and origin request policy?
?
**Quick:** Cache policy controls what's in the cache key; origin request policy controls what's forwarded.<br>
**Deeper:** Cache policy: TTLs, query strings/headers/cookies included in the cache key (affects hit rate). Origin request policy: which query strings/headers/cookies forward to origin but NOT in cache key. Exam trick: include auth headers in origin request policy (not cache policy) so each user sees the same cached file but origin still gets the token.

What are VPC peering and Transit Gateway?
?
**Quick:** Peering = 1:1 VPC connection; Transit Gateway = hub-and-spoke for many VPCs + on-prem.<br>
**Deeper:** Peering: non-transitive (A↔B and B↔C does NOT mean A↔C), no transitive routing, works cross-region/account. Transit Gateway: central router, simplifies many-VPC topologies, supports VPN/Direct Connect attachments, costs more. Exam trick: 50+ VPCs to interconnect = Transit Gateway; 2-3 VPCs simple peering.

What are the VPC sizing rules?
?
**Quick:** VPC CIDR /16 to /28; AWS reserves 5 IPs per subnet.<br>
**Deeper:** Reserved per subnet: .0 (network), .1 (VPC router), .2 (DNS), .3 (future), and .255 (broadcast). So a /28 has 16 - 5 = 11 usable IPs. Subnets are AZ-bound — one subnet, one AZ. Exam trick: "why is my subnet showing fewer IPs than expected" = the 5 reserved.

What are common DNS record types you should know for Route 53?
?
**Quick:** A, AAAA, CNAME, MX, TXT, NS, SOA, ALIAS, PTR.<br>
**Deeper:** A = IPv4, AAAA = IPv6, CNAME = alias to DNS name (not apex), MX = mail, TXT = arbitrary (SPF, DKIM, domain verification), NS = nameservers, SOA = zone authority. ALIAS is AWS-specific. Exam trick: TXT for domain verification (SES, ACM DNS validation, Google Workspace).
