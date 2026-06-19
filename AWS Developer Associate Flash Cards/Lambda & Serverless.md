---
tags:
  - flashcards/aws-lambda
---

# Lambda & Serverless

Lambda runtime limits, concurrency, event sources, and SAM essentials for DVA-C02.

---

What is the maximum Lambda function timeout?
?
**Quick:** 15 minutes (900 seconds).<br>
**Deeper:** Default is 3 seconds. Exam trick: if a workload needs to run longer than 15 minutes, the answer is almost always Fargate, Step Functions, or Batch — not Lambda. API Gateway adds its own 29-second integration timeout limit on top of Lambda, so even a 15-minute Lambda will fail behind API Gateway after 29 seconds.

What are Lambda memory and /tmp size limits?
?
**Quick:** Memory 128 MB to 10,240 MB (10 GB); /tmp 512 MB to 10,240 MB.<br>
**Deeper:** Memory is allocated in 1 MB increments and CPU scales linearly with memory. /tmp defaults to 512 MB and is configurable up to 10 GB. Exam gotcha: increasing memory often makes functions faster AND cheaper because duration drops faster than cost rises — use AWS Lambda Power Tuning to find the sweet spot.

What are Lambda payload size limits (sync vs async)?
?
**Quick:** 6 MB synchronous (request and response), 256 KB asynchronous.<br>
**Deeper:** Sync invokes (RequestResponse) cap at 6 MB each way. Async invokes (Event) and event source mappings cap at 256 KB. Exam trick: for large payloads, drop the file in S3 and pass the S3 key in the event. Deployment package is 50 MB zipped / 250 MB unzipped (including layers), or 10 GB via container image.

What is the difference between a Lambda version and an alias?
?
**Quick:** Versions are immutable snapshots; aliases are mutable pointers to versions.<br>
**Deeper:** Publishing a version freezes code + config and gives it a number ($LATEST is mutable). Aliases (like prod, dev) point to a version and can do weighted routing for canary deploys (e.g., 90% to v1, 10% to v2). Exam gotcha: you cannot point an alias at $LATEST for weighted routing — you must publish numbered versions.

What is the difference between reserved and provisioned concurrency?
?
**Quick:** Reserved guarantees/caps concurrent executions; provisioned pre-warms execution environments.<br>
**Deeper:** Reserved concurrency reserves a slice of the account's 1,000-default concurrency pool for one function AND limits it to that number (throttles excess). Provisioned concurrency keeps N instances initialized and ready, eliminating cold starts — costs money even when idle. Exam trick: cold starts are the symptom, provisioned concurrency is the answer.

What causes Lambda cold starts and how do you mitigate them?
?
**Quick:** Initializing a new execution environment; mitigate with provisioned concurrency or smaller packages.<br>
**Deeper:** Cold starts happen on first invoke, after scaling, or after ~15 minutes idle. Mitigations: provisioned concurrency, smaller deployment packages, fewer dependencies, languages with faster init (Go/Node beat Java/.NET), and keeping VPC config minimal. SnapStart (Java/.NET/Python) snapshots initialized state for fast restore.

How does Lambda VPC configuration work?
?
**Quick:** Lambda attaches Hyperplane ENIs into your subnets to reach private resources.<br>
**Deeper:** Specify subnets + security groups; Lambda needs a NAT Gateway in a private subnet to reach the internet (it has no public IP). Exam gotcha: you do NOT need VPC config to call AWS APIs — Lambda already has internet by default. Only add VPC when you must reach RDS, ElastiCache, or other VPC-only resources. Use VPC endpoints to avoid NAT costs.

What are the three Lambda invocation models?
?
**Quick:** Synchronous, asynchronous, and event source mapping (poll-based).<br>
**Deeper:** Sync (API Gateway, ALB, Cognito) — caller waits and handles errors. Async (S3, SNS, EventBridge) — Lambda retries twice automatically, can route failures to a DLQ or Destinations. Poll-based (SQS, Kinesis, DynamoDB Streams) — Lambda service polls the source and invokes the function in batches.

What is the difference between Lambda DLQ and Destinations?
?
**Quick:** DLQ catches only failures; Destinations route on success OR failure to multiple targets.<br>
**Deeper:** DLQ (SQS or SNS) only catches async invoke failures after retries exhausted — failure-only, one target. Destinations (newer, preferred) support SQS, SNS, Lambda, or EventBridge as targets for both success AND failure outcomes, and capture richer context. Exam trick: Destinations are the modern answer; DLQ is still valid but lower fidelity.

What are Lambda layers and what are their limits?
?
**Quick:** Reusable .zip archives of libraries/runtime; max 5 layers per function.<br>
**Deeper:** Layers ship shared dependencies separately from function code, extracted to /opt at runtime. Total unzipped size (function + all layers) must stay under 250 MB. Exam gotcha: max 5 layers per function — the question often tries to trick you with "6 layers."

How does Lambda handle environment variables and secrets?
?
**Quick:** Env vars are key/value pairs encrypted at rest with KMS; use Secrets Manager or SSM for secrets.<br>
**Deeper:** Total env var size limit is 4 KB. Lambda encrypts env vars with an AWS-managed KMS key by default; use a customer-managed key for envelope encryption in transit during console display. Exam best practice: don't put secrets in env vars — pull from Secrets Manager (auto-rotation) or SSM Parameter Store (cheaper) at init.

What does SAM provide on top of CloudFormation?
?
**Quick:** A simplified transform for serverless resources that expands to CloudFormation.<br>
**Deeper:** SAM templates declare AWS::Serverless::Function, ::Api, ::Table, etc., which the SAM transform expands into full CFN. SAM CLI commands: sam init, sam build, sam local invoke (local testing), sam package, sam deploy, sam sync (fast dev loop). Exam gotcha: SAM is a superset of CloudFormation — any CFN resource is valid in a SAM template.

What is the Lambda execution role?
?
**Quick:** The IAM role Lambda assumes to call AWS services on your behalf.<br>
**Deeper:** Must include the trust policy allowing lambda.amazonaws.com to AssumeRole, and permissions for CloudWatch Logs (CreateLogGroup, CreateLogStream, PutLogEvents) at minimum. Exam trick: "function is running but no logs appear" usually means the execution role is missing CloudWatch Logs permissions. Resource-based policies on Lambda control WHO can invoke it; execution role controls what the function CAN DO.
