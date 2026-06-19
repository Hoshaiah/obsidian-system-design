---
tags:
  - flashcards/aws-codepipeline
---

# CodePipeline, CodeBuild & CodeDeploy

CI/CD on AWS, deployment strategies, and buildspec/appspec for DVA-C02.

---

What is the structure of a buildspec.yml?
?
**Quick:** version + phases (install/pre_build/build/post_build) + artifacts + env + cache.<br>
**Deeper:** Phases run in order, fail-fast unless on-failure: CONTINUE is set. env exposes variables (including secrets-manager and parameter-store integration). artifacts paths declare what to upload. cache speeds up subsequent builds. Exam gotcha: buildspec.yml must be at the repo root by default — override with the Build project's buildspec path setting.

What is the structure of an appspec.yml?
?
**Quick:** Hooks and resources for CodeDeploy — varies by EC2/Lambda/ECS.<br>
**Deeper:** EC2/On-Prem: files (source/destination), permissions, hooks (BeforeInstall, AfterInstall, ApplicationStart, ValidateService). Lambda: hooks (BeforeAllowTraffic, AfterAllowTraffic). ECS: resources (TaskDef, ContainerName) + hooks (BeforeAllowTestTraffic, AfterAllowTraffic, etc.). Exam trick: appspec.yml MUST be at the root of the deployment bundle; rename mistakes = deploy fails immediately.

What are CodeDeploy in-place vs blue/green deployments?
?
**Quick:** In-place updates existing instances; blue/green spins up parallel fleet and switches.<br>
**Deeper:** In-place (EC2 only) deploys to current instances, faster + cheaper but downtime during deploy. Blue/green creates fresh green fleet, runs validation, then routes traffic from blue → green (ALB swap), keeps blue as rollback. Lambda and ECS deployments are ALWAYS blue/green (or canary/linear) — only EC2 supports in-place.

What are Lambda/ECS canary, linear, and all-at-once deployments?
?
**Quick:** Canary = jump %, linear = gradual %, all-at-once = 100% immediately.<br>
**Deeper:** Canary10Percent5Minutes shifts 10% then 100% after 5 min. Linear10PercentEvery1Minute shifts 10% increments. AllAtOnce flips immediately. CodeDeploy uses Lambda aliases (weighted) or ECS service updates. Exam trick: failed CloudWatch alarms during the shift = auto-rollback. PreTraffic/PostTraffic hooks run validation Lambdas.

What are CodeDeploy lifecycle hooks for EC2?
?
**Quick:** ApplicationStop, BeforeInstall, AfterInstall, ApplicationStart, ValidateService (+ DownloadBundle/Install built-in).<br>
**Deeper:** Order: ApplicationStop → DownloadBundle → BeforeInstall → Install → AfterInstall → ApplicationStart → ValidateService. Blue/green adds BeforeBlockTraffic, AfterBlockTraffic (on old), BeforeAllowTraffic, AfterAllowTraffic (on new). Hook scripts referenced in appspec.yml; failed hook = deployment fails. Exam trick: ValidateService is where you run smoke tests.

What are CodePipeline stages and actions?
?
**Quick:** Stages contain actions like Source, Build, Test, Deploy, Approval, Invoke.<br>
**Deeper:** Source actions: CodeCommit, GitHub, S3, ECR, Bitbucket. Build/Test: CodeBuild, Jenkins. Deploy: CodeDeploy, CFN, ECS, Beanstalk, S3, Lambda. Manual approval pauses for human OK. Invoke: Lambda for custom logic. Exam trick: artifacts flow between stages via S3 (artifact store); CodePipeline does NOT run code itself.

How does CodeDeploy roll back?
?
**Quick:** Manual rollback or auto on CloudWatch alarm / deployment failure.<br>
**Deeper:** Configure auto-rollback "when a deployment fails" and/or "when alarm thresholds are met". Rollback is actually a new deployment of the previous successful revision. For blue/green, rollback is a fast traffic-switch back to blue (still running). Exam best practice: always pair canary/linear with alarms on error rates.

What does CodeBuild use for compute and what's the timeout?
?
**Quick:** Docker containers (managed or custom); default 60 min, max 8 hours.<br>
**Deeper:** Compute types: BUILD_GENERAL1_SMALL/MEDIUM/LARGE/2XLARGE/XLARGE. Build images include Amazon Linux 2 with prebaked toolchains, or bring your own image from ECR/Docker Hub. Pay per build-minute. Exam trick: for VPC resources (private RDS, internal services), enable VPC config on the project; default builds have no VPC access.

What is CodeArtifact?
?
**Quick:** Managed artifact repository for npm/pip/Maven/NuGet/RubyGems packages.<br>
**Deeper:** Organize into domain → repositories. Upstream other repos like npmjs.com or PyPI through CodeArtifact for caching + access control. Authenticate via aws codeartifact login. Exam trick: a single domain pulls and caches packages from public registries — reduces external dependency risk and audits internal package consumption.

How do CodePipeline + CodeBuild + CodeDeploy fit together?
?
**Quick:** Pipeline orchestrates stages; Build compiles code; Deploy delivers it.<br>
**Deeper:** Typical flow: CodeCommit → CodeBuild (compile, test, package) → CodeDeploy (release with strategy). Pipeline manages artifact passing through S3 and stage transitions. CodeBuild can also do tests/lint; CodeDeploy handles rollout strategy and rollback. Exam trick: question gives a scenario — match each tool to one job.

What environment variable sources does CodeBuild support?
?
**Quick:** Plaintext env, Secrets Manager, and Parameter Store.<br>
**Deeper:** In buildspec env block: variables (plaintext), parameter-store (SSM, requires GetParameters perm), secrets-manager (requires secretsmanager:GetSecretValue). Never put secrets in plaintext env. Exam gotcha: the CodeBuild service role must have permissions to read SSM/Secrets Manager values referenced.

What is CodeCommit and how does auth work?
?
**Quick:** Managed Git hosting; HTTPS auth via git-credential-helper or SSH via IAM-uploaded keys.<br>
**Deeper:** Repos are private to your account by default. Auth options: HTTPS with IAM credentials (helper signs requests), HTTPS git-remote-codecommit (GRC) using AWS creds, or SSH with public keys uploaded to IAM users. Exam trick: cross-account access via IAM roles; never use long-lived access keys for Git when temp credentials are available.

What CloudWatch metrics can trigger CodeDeploy rollback?
?
**Quick:** Any alarm — typically error rates, latency, or custom app metrics.<br>
**Deeper:** Configure deployment group with one or more CloudWatch alarms. If alarm fires during deployment (or within the alarm period after), CodeDeploy rolls back. Common: Lambda Errors, ALB 5XXCount, ECS Service unhealthy host count, custom app KPIs. Exam trick: alarms must be in ALARM state to trigger rollback — INSUFFICIENT_DATA does not.
