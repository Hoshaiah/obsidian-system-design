---
tags:
  - flashcards/aws-s3
---

# S3 & Storage

S3 storage classes, security, lifecycle, and how S3/EFS/EBS compare for DVA-C02.

---

What are the main S3 storage classes and their use cases?
?
**Quick:** Standard, Standard-IA, One Zone-IA, Glacier Instant/Flexible/Deep Archive, Intelligent-Tiering.<br>
**Deeper:** Standard for frequent access; Standard-IA for infrequent but durable (multi-AZ); One Zone-IA for re-creatable data (single-AZ, ~20% cheaper); Glacier Instant (ms retrieval), Flexible (min-hr), Deep Archive (12 hr, cheapest); Intelligent-Tiering auto-moves objects between tiers based on access. Exam trick: One Zone-IA is the answer when "secondary copy" or "re-creatable" is mentioned.

What are the minimum storage durations for IA and Glacier classes?
?
**Quick:** IA classes 30 days; Glacier Flexible/Deep Archive 90/180 days minimum billing.<br>
**Deeper:** Standard-IA and One Zone-IA bill for 30 days minimum even if deleted earlier. Glacier Flexible Retrieval 90 days, Glacier Deep Archive 180 days. Exam gotcha: lifecycle policies cannot transition objects to IA classes until they are at least 30 days old, and objects must be at least 128 KB to avoid the small-object penalty.

How do S3 versioning and MFA Delete work?
?
**Quick:** Versioning preserves all object versions; MFA Delete requires MFA to permanently delete or suspend versioning.<br>
**Deeper:** Versioning is bucket-level, can be Enabled or Suspended (never disabled once enabled). Deletes create a delete marker rather than removing data. MFA Delete adds an extra factor for the bucket owner's root account only, and can ONLY be configured via the AWS CLI by the root account — never the console. Exam trick: MFA Delete + root account + CLI is the magic trio.

What are presigned URLs and what are their limits?
?
**Quick:** Time-limited URLs granting temporary access to S3 objects via the signer's credentials.<br>
**Deeper:** Default expiration 3,600 seconds; max 7 days when signed with IAM user credentials, max 36 hours with temporary STS credentials, max 12 hours with IAM role. The presigner must have the underlying GetObject/PutObject permission. Exam trick: signed URL expiration is capped by the credential type — a role-signed URL cannot last 7 days.

What is the S3 multipart upload threshold and why use it?
?
**Quick:** Recommended for files >100 MB; required for files >5 GB.<br>
**Deeper:** Max single PUT is 5 GB. Multipart supports up to 5 TB total, parallel parts (5 MB-5 GB each, last part can be smaller), and resumable uploads. Use lifecycle rule to abort incomplete multipart uploads after N days to save costs. Exam gotcha: max object size in S3 is 5 TB; max single PUT is 5 GB; minimum part size in multipart is 5 MB (except last part).

What are the four S3 server-side encryption options?
?
**Quick:** SSE-S3 (AES-256, AWS keys), SSE-KMS (KMS keys), SSE-C (customer-provided keys), DSSE-KMS (double encryption).<br>
**Deeper:** SSE-S3 is default (now mandatory for all new buckets). SSE-KMS adds audit trail via CloudTrail and supports customer-managed keys but costs API calls and is subject to KMS request quotas (can throttle high-volume workloads). SSE-C means you provide and manage the key on each request — S3 never stores it. Client-side encryption (CSE) encrypts before upload.

What is the difference between S3 bucket policies and ACLs?
?
**Quick:** Bucket policies are JSON resource policies; ACLs are legacy per-object/bucket grants.<br>
**Deeper:** AWS recommends disabling ACLs (Object Ownership: Bucket owner enforced is now the default). Bucket policies handle cross-account access, IP restrictions, and SSL enforcement. Exam trick: when a question mentions cross-account access to specific prefixes, the answer is a bucket policy, not an ACL.

How does S3 CORS work?
?
**Quick:** A bucket-level config that allows cross-origin browser requests.<br>
**Deeper:** Configure allowed origins, methods, headers, and max age on the bucket. Browser sends a preflight OPTIONS request; S3 must return matching CORS headers or the request fails. Exam gotcha: CORS errors in the browser console with an S3 static site behind CloudFront are usually fixed by adding the website endpoint origin to the bucket's CORS config.

What is S3 Object Lock and what modes exist?
?
**Quick:** WORM protection with Governance and Compliance modes plus Legal Hold.<br>
**Deeper:** Requires versioning. Governance mode allows users with s3:BypassGovernanceRetention to override; Compliance mode cannot be overridden even by root until retention expires. Legal Hold is indefinite and toggleable with s3:PutObjectLegalHold. Exam trick: Compliance mode + locked retention period = even root cannot delete. Used for SEC 17a-4, HIPAA, financial records.

How does S3 Transfer Acceleration work?
?
**Quick:** Uses CloudFront edge locations to speed up uploads via Amazon's backbone.<br>
**Deeper:** Clients upload to the nearest edge, which forwards over AWS's private network to the bucket region. Most useful for large objects from distant clients. Exam gotcha: incurs extra cost — only enable if S3 Transfer Acceleration Speed Comparison Tool shows benefit. Alternative for large uploads is multipart with parallel parts.

What S3 events can trigger Lambda/SQS/SNS/EventBridge?
?
**Quick:** Object created, removed, restored, replication, lifecycle, and tagging events.<br>
**Deeper:** Direct integrations: Lambda, SQS, SNS. EventBridge integration gives finer-grained filtering, archive/replay, and multiple targets per event. Exam gotcha: S3 event notifications are eventually consistent (typically seconds, occasionally minutes). For guaranteed ordering, use EventBridge plus a FIFO queue.

How do EFS, EBS, and S3 compare?
?
**Quick:** EBS = block (one EC2), EFS = NFS (many EC2s), S3 = object (HTTPS API).<br>
**Deeper:** EBS attaches to one EC2 (io2 supports multi-attach in same AZ). EFS is multi-AZ NFS, scales automatically, mounted by many EC2s/Lambdas/containers. S3 is HTTPS-only object storage. Exam trick: "shared file system across multiple EC2s in different AZs" = EFS; "shared between EC2 and Lambda" = EFS; "static website assets" = S3; "boot volume" = EBS.

What is S3 Intelligent-Tiering?
?
**Quick:** Auto-moves objects between frequent/infrequent/archive tiers based on access patterns.<br>
**Deeper:** Small monthly per-object monitoring fee, no retrieval fees for Frequent/Infrequent tiers. Archive Access and Deep Archive Access tiers are optional (opt-in) and add retrieval latency. Exam trick: "unpredictable access patterns, no operational overhead" — Intelligent-Tiering. No minimum duration, but small-object overhead means it's worse than Standard for objects under 128 KB.

How does S3 enforce SSL/TLS for requests?
?
**Quick:** Bucket policy denying requests where aws:SecureTransport is false.<br>
**Deeper:** Standard exam pattern: a Deny statement on s3:* when "aws:SecureTransport": "false". S3 does not refuse HTTP by default. Combine with public access block, default encryption, and versioning for a hardened bucket. Exam trick: "ensure all requests use HTTPS" almost always points to a bucket policy condition, not bucket settings.
