# Flashcards — AWS Developer Associate (DVA-C02)

High-yield AWS knowledge for the DVA-C02 exam. Each note is its own subdeck (`flashcards/aws-<topic>`) for the Obsidian *Spaced Repetition* plugin.

## Decks
- [[Lambda & Serverless]] — runtime limits, concurrency, event sources, SAM
- [[S3 & Storage]] — storage classes, lifecycle, encryption, EFS vs EBS vs S3
- [[DynamoDB]] — keys, RCU/WCU math, GSI/LSI, streams, hot partitions
- [[API Gateway]] — REST vs HTTP vs WebSocket, authorizers, caching, throttling
- [[IAM, Security & KMS]] — policies, STS, envelope encryption, Secrets Manager vs SSM
- [[SQS, SNS & EventBridge]] — queues, fanout, event buses, when to pick each
- [[CloudFormation & SAM]] — template structure, intrinsic functions, change sets, CDK
- [[CodePipeline, CodeBuild & CodeDeploy]] — buildspec, appspec, blue/green, canary
- [[Monitoring — CloudWatch, X-Ray, CloudTrail]] — metrics, logs, tracing, audit
- [[ECS, EKS & Elastic Beanstalk]] — task definitions, Fargate, deployment policies
- [[Networking — VPC, Route 53, CloudFront]] — subnets, SG vs NACL, routing policies, OAC
- [[RDS, Aurora & ElastiCache]] — Multi-AZ vs replicas, Aurora variants, Redis vs Memcached

## How to use
- Pareto rule: these cover the most-tested 20%.
- The exam loves specific numbers (timeouts, sizes, limits) and "pick the right service for this scenario" — drill those.
- Add cards from your practice-test misses — those are worth 10x pre-made ones.
- Cards use the `Question`/`?`/`Answer` format used by Obsidian Spaced Repetition.
