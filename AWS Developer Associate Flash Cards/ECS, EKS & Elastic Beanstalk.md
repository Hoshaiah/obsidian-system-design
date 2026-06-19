---
tags:
  - flashcards/aws-containers
---

# ECS, EKS & Elastic Beanstalk

Containers, orchestration, and PaaS for DVA-C02.

---

What is an ECS task definition?
?
**Quick:** A JSON spec for one or more containers — image, CPU, memory, ports, env, IAM.<br>
**Deeper:** Includes task-level CPU/memory (Fargate requires this), container definitions, volumes, network mode (awsvpc, bridge, host, none), task role + execution role. Revisions are immutable; updates create a new revision. Exam trick: awsvpc is required for Fargate and gives each task its own ENI + private IP.

What is the difference between EC2 and Fargate launch types?
?
**Quick:** EC2 manages your own instances; Fargate is serverless containers.<br>
**Deeper:** EC2 mode: cluster of EC2s you patch/scale, tasks pack onto them, cheaper for steady high utilization, supports GPU/large memory hosts. Fargate: AWS manages compute, pay per task vCPU/memory/second, no instance management, slightly more expensive but operationally trivial. Exam trick: "minimize operational overhead" = Fargate; "tight cost control / GPU" = EC2.

What is the difference between task role and execution role?
?
**Quick:** Task role = perms FOR your container code; execution role = perms FOR ECS agent (pull image, push logs).<br>
**Deeper:** Execution role needs ecr:GetAuthorizationToken, ecr:BatchGetImage, logs:CreateLogStream + PutLogEvents, and access to Secrets Manager/SSM for env injection. Task role is what AWS SDK calls inside the container use. Exam favorite gotcha: "container can't write to DynamoDB" — fix the TASK role, not the execution role.

What is ECR?
?
**Quick:** Managed private Docker image registry.<br>
**Deeper:** Authenticate with aws ecr get-login-password. Supports image scanning (basic and enhanced via Inspector), lifecycle policies (delete untagged images after N days), cross-region replication, immutable tags. Exam trick: "vulnerability scan container images" = ECR enhanced scanning; basic only scans on push.

What is ECS Service Auto Scaling?
?
**Quick:** Application Auto Scaling adjusts service desired count based on metrics.<br>
**Deeper:** Target tracking (CPU, memory, ALB request count per target), step scaling, scheduled scaling. Separate from EC2 Auto Scaling Group (which scales the cluster instances themselves). Exam trick: in EC2 launch type, you may need to scale BOTH the service and the ASG; Fargate only scales the service.

How does an ECS service integrate with an ALB?
?
**Quick:** Service registers tasks with a target group; ALB routes to them.<br>
**Deeper:** awsvpc network mode + IP target group (each task gets a private IP). Service auto-deregisters on scale-in. Path/host-based routing on ALB lets multiple services share a load balancer. Exam trick: NLB for high-perf TCP/static IP, ALB for HTTP with content-based routing.

What is Elastic Beanstalk?
?
**Quick:** PaaS that deploys web apps to managed EC2 + ALB + ASG + RDS infrastructure.<br>
**Deeper:** Supports Node, Python, Ruby, Java, Go, .NET, PHP, Docker. Free service — pay only for underlying resources. Environments: Web Server (handles HTTP) or Worker (SQS-backed). Saved configurations and application versions are reusable. Exam trick: "deploy code with minimum infra knowledge" = Beanstalk.

What are the Elastic Beanstalk deployment policies?
?
**Quick:** All-at-once, Rolling, Rolling with additional batch, Immutable, Traffic Splitting.<br>
**Deeper:** All-at-once = downtime, fastest. Rolling = no extra capacity, partial downtime/reduced capacity. Rolling with additional batch = adds batch first then rolls, no capacity loss. Immutable = new ASG with new instances, swap then terminate old — safest, slowest. Traffic splitting = canary on weighted ALB. Exam trick: "zero downtime, safe rollback" = Immutable.

What are .ebextensions?
?
**Quick:** YAML/JSON config files in .ebextensions/ that customize the EB environment.<br>
**Deeper:** Configure packages, sources, files, commands, container_commands, services, option_settings. Run during deploy. For Amazon Linux 2 platforms, prefer the Platform Hooks mechanism (.platform/hooks). Exam gotcha: option_settings can configure ASG, ELB, env variables without console clicks — common exam scenario.

What is Beanstalk Worker environment?
?
**Quick:** Background job processor — pulls SQS messages and POSTs to a local endpoint.<br>
**Deeper:** SQS daemon on each instance polls an SQS queue (auto-created by EB or user-specified), POSTs message body to localhost:80. Periodic tasks via cron.yaml. Exam trick: "decouple web tier from heavy processing" = web environment + worker environment + SQS in between.

What is ECS Service Discovery / Cloud Map?
?
**Quick:** Auto-registers ECS tasks in Route 53 so services find each other by DNS.<br>
**Deeper:** Cloud Map namespace + service maps service names to current task IPs. Supports A and SRV records. Alternative to running your own service registry. Exam trick: "microservices need to discover each other without ALB" = ECS Service Discovery.

What is the difference between ECS and EKS?
?
**Quick:** ECS is AWS-native container orchestrator; EKS is managed Kubernetes.<br>
**Deeper:** ECS is simpler, deeper AWS integration (IAM per task, no extra control plane cost). EKS is portable (vanilla Kubernetes) but adds $0.10/hr per cluster control plane and requires kubectl/helm knowledge. Both support Fargate. Exam trick: "portability across cloud providers" = EKS; "minimum learning curve and operational overhead with AWS" = ECS.

What is Fargate Spot?
?
**Quick:** Discounted Fargate capacity that can be interrupted.<br>
**Deeper:** Up to ~70% discount. AWS gives a 2-minute SIGTERM warning before interruption. Use capacity providers to mix FARGATE and FARGATE_SPOT (e.g., 80% spot). Best for fault-tolerant batch and stateless services. Exam trick: similar concept to EC2 Spot but for serverless containers.

What is the ECS task placement strategy?
?
**Quick:** Algorithm to choose which container instance runs a new task.<br>
**Deeper:** binpack (densely pack to fewest instances), random, spread (distribute across attribute like AZ). Combine with placement constraints (distinctInstance, memberOf). EC2 launch type only — Fargate doesn't expose placement. Exam trick: "spread tasks across AZs for HA" = spread strategy on attribute:ecs.availability-zone.
