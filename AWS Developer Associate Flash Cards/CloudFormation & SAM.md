---
tags:
  - flashcards/aws-cloudformation
---

# CloudFormation & SAM

Templates, intrinsic functions, drift, change sets, and SAM/CDK for DVA-C02.

---

What are the sections of a CloudFormation template?
?
**Quick:** AWSTemplateFormatVersion, Description, Metadata, Parameters, Mappings, Conditions, Transform, Resources, Outputs.<br>
**Deeper:** Only Resources is required. Parameters take user input at deploy time. Mappings are static lookup tables (e.g., region → AMI). Conditions toggle resource creation. Transform invokes macros like AWS::Serverless (SAM) or AWS::Include. Outputs export values for cross-stack reference or display. Exam gotcha: parameter limit is 200 per template; resource limit is 500.

What does the Ref intrinsic function return?
?
**Quick:** For a parameter, its value; for a resource, the resource's logical default identifier.<br>
**Deeper:** Returns vary by resource: !Ref of an EC2 returns the instance ID, !Ref of an S3 bucket returns the bucket name, !Ref of a parameter returns the parameter value. Exam trick: !Ref MyBucket gives the name, but !GetAtt MyBucket.Arn gives the ARN — pick the right one.

What does Fn::GetAtt do?
?
**Quick:** Returns a specific attribute of a resource (e.g., ARN, DNS name, IP).<br>
**Deeper:** Syntax: !GetAtt LogicalId.Attribute. Each resource type has documented attributes — examples: !GetAtt MyDB.Endpoint.Address, !GetAtt MyLambda.Arn. Exam trick: cross-stack references typically use !GetAtt for non-default attributes and Outputs + !ImportValue across stacks.

What is Fn::Sub used for?
?
**Quick:** String substitution with variables, similar to template literals.<br>
**Deeper:** !Sub "arn:aws:s3:::${BucketName}/${AWS::AccountId}/*" substitutes parameters, pseudo-parameters (AWS::Region, AWS::StackName), and resource attributes. Cleaner than nested !Join calls. Exam trick: Sub is preferred over Join for readability; many newer template examples have abandoned Join entirely.

What is Fn::ImportValue and how do exports work?
?
**Quick:** Imports a value Exported by another stack in the same region/account.<br>
**Deeper:** Exporter declares Outputs with an Export.Name; importer uses !ImportValue ExportName. Exports must be region-unique. Exam gotcha: you cannot delete a stack whose exports are imported elsewhere, and you cannot modify the value of an active export — refactor by deprecating the export, removing imports, then changing it.

What is a CloudFormation change set?
?
**Quick:** A preview of changes a template update will make before executing it.<br>
**Deeper:** Generated via CreateChangeSet, shows Add/Modify/Remove and whether resources will be Replaced (data-destructive!). You then Execute or Delete the change set. Exam best practice: always review change sets in prod to avoid unexpected resource replacement (e.g., changing a property that forces replacement of an RDS instance).

What is CloudFormation drift detection?
?
**Quick:** Compares deployed resources to the template to find out-of-band changes.<br>
**Deeper:** Run on stack or individual resources; reports IN_SYNC, MODIFIED, DELETED, or NOT_CHECKED. Doesn't auto-remediate. Exam trick: drift is detection-only; to fix, either update the resource manually back, or update the stack with the actual state. Common cause: someone tweaks a resource via console.

What is a nested stack?
?
**Quick:** A child stack created as a resource (AWS::CloudFormation::Stack) within a parent.<br>
**Deeper:** Used to break large templates into reusable components — e.g., a VPC nested stack, a security-group nested stack. Pass parameters from parent, get Outputs back. Exam trick: nested stacks for reusable components within a project; cross-stack exports (ImportValue) for sharing between unrelated stacks.

What is a stack policy?
?
**Quick:** A JSON document that protects specific resources from accidental updates.<br>
**Deeper:** Default policy allows all updates. Stack policy can Deny Update:Replace or Update:* on resources by logical ID. Override at update time with a temporary stack policy. Exam trick: protect prod DBs by denying Update:Replace on the DB resource in the stack policy — different from termination protection (which protects the whole stack from deletion).

What does SAM transform do?
?
**Quick:** Expands AWS::Serverless::* shortcuts into full CloudFormation resources.<br>
**Deeper:** Add Transform: AWS::Serverless-2016-10-31 at the top. AWS::Serverless::Function generates Lambda, IAM role, log group, API Gateway, event source mappings. AWS::Serverless::Api generates API Gateway + stages. AWS::Serverless::HttpApi for v2. Exam gotcha: SAM is a CloudFormation macro — you can still include raw CFN resources in a SAM template.

What are the key SAM CLI commands?
?
**Quick:** sam init, sam build, sam local invoke, sam package, sam deploy, sam sync.<br>
**Deeper:** sam build compiles deps (per runtime); sam local invoke / start-api / start-lambda for local testing via Docker; sam package uploads to S3 and rewrites local paths; sam deploy creates change set + executes; sam sync is fast dev loop that updates only changed Lambda code. Exam trick: sam deploy --guided walks first-timers through stack name, region, capabilities.

What is the difference between CloudFormation and CDK?
?
**Quick:** CFN is YAML/JSON; CDK is real code (TS/Python/Java/C#) that synthesizes to CFN.<br>
**Deeper:** CDK gives loops, conditionals, abstractions (Constructs), and reusable libraries. cdk synth produces CFN; cdk deploy runs the synthesized CFN. CDK uses CFN under the hood — all resources are CFN resources, drift and change sets work identically. Exam trick: CDK constructs at L1 are raw CFN, L2 are higher-level patterns, L3 are full patterns (e.g., ECS app behind an ALB).

What are CFN stack capabilities?
?
**Quick:** CAPABILITY_IAM, CAPABILITY_NAMED_IAM, CAPABILITY_AUTO_EXPAND.<br>
**Deeper:** Required when a template creates IAM resources or uses transforms. CAPABILITY_IAM acknowledges resources may create IAM; CAPABILITY_NAMED_IAM required when resources have explicit names; CAPABILITY_AUTO_EXPAND for SAM/macro transforms. Exam gotcha: deploying a SAM template needs --capabilities CAPABILITY_IAM CAPABILITY_AUTO_EXPAND.

What are CloudFormation pseudo-parameters?
?
**Quick:** Built-in variables like AWS::Region, AWS::AccountId, AWS::StackName.<br>
**Deeper:** Available everywhere via !Ref or !Sub. Common ones: AWS::Region, AWS::AccountId, AWS::StackName, AWS::Partition (aws, aws-us-gov, aws-cn), AWS::URLSuffix, AWS::NotificationARNs, AWS::NoValue (omit a property). Exam trick: !Sub "arn:${AWS::Partition}:s3:::..." works in all partitions, while hardcoded "arn:aws:..." fails in GovCloud/China.
