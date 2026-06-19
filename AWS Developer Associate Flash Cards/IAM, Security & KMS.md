---
tags:
  - flashcards/aws-iam-security
---

# IAM, Security & KMS

Identity, policy evaluation, encryption, and secrets storage for DVA-C02.

---

What is the difference between IAM users, groups, and roles?
?
**Quick:** Users have long-term credentials, groups bundle permissions, roles are assumed for temporary credentials.<br>
**Deeper:** Users can be humans or apps with access keys (avoid for code). Groups attach policies to many users at once — you cannot nest groups or assign roles to groups. Roles are assumed via STS and return temporary credentials (15 min to 12 hours) — used by EC2 instance profiles, Lambda execution roles, and cross-account access. Exam best practice: never embed user keys; use roles.

How does IAM policy evaluation work?
?
**Quick:** Explicit Deny > Explicit Allow > Implicit Deny (default).<br>
**Deeper:** Order: 1) check SCPs (Organizations), 2) resource policies, 3) IAM permission boundaries, 4) session policies, 5) identity-based policies. Any explicit Deny anywhere wins. No Allow = implicit Deny. Exam trick: even if identity policy allows S3:GetObject, an SCP Deny or bucket policy Deny blocks it.

What is the difference between identity-based and resource-based policies?
?
**Quick:** Identity-based attach to IAM principals; resource-based attach to resources and include Principal field.<br>
**Deeper:** Identity-based (managed/inline) say what THIS USER can do. Resource-based (S3 bucket policy, SNS topic policy, KMS key policy, Lambda resource policy) say who can use THIS RESOURCE. Cross-account access usually needs both: identity policy in account A AND resource policy in account B (except for STS AssumeRole, which only needs the trust policy on the role).

What is STS AssumeRole and when do you use it?
?
**Quick:** Returns temporary credentials by assuming a role; used for cross-account, federation, and EC2.<br>
**Deeper:** Variants: AssumeRole (IAM principals), AssumeRoleWithSAML (SAML 2.0 IdP), AssumeRoleWithWebIdentity (OIDC/Cognito). Returns AccessKey, SecretKey, SessionToken, Expiration (default 1 hour, max 12). Exam trick: cross-account = role trust policy lists Account A's root, Account A's user has sts:AssumeRole on the role ARN. Web identity federation = mobile apps swapping ID tokens for AWS creds.

What is an EC2 instance profile?
?
**Quick:** A container that lets EC2 assume an IAM role automatically.<br>
**Deeper:** You attach a role to an EC2 via the instance profile (created automatically in console, manually in CLI/CFN). SDK fetches creds from the IMDS endpoint at 169.254.169.254. Use IMDSv2 (token-based) to prevent SSRF attacks. Exam gotcha: a role can only be in one instance profile, and one EC2 can have only one instance profile attached at a time.

What is the difference between AWS-managed and customer-managed KMS keys?
?
**Quick:** AWS-managed are free and auto-rotated; customer-managed give you full control.<br>
**Deeper:** AWS-managed keys (alias/aws/service) appear automatically per service per account, rotate yearly, free for storage. Customer-managed cost $1/month each, support custom key policies, manual or automatic (yearly) rotation, importing your own key material, and cross-account use. Exam trick: "audit who used the key" or "cross-account encryption" = customer-managed CMK.

How does KMS envelope encryption work?
?
**Quick:** A data key encrypts the data; the CMK encrypts the data key.<br>
**Deeper:** GenerateDataKey returns plaintext + encrypted data key; you encrypt your data with the plaintext key, store the encrypted data key alongside ciphertext, then discard the plaintext key. To decrypt, call KMS Decrypt on the encrypted data key. Exam trick: this pattern bypasses the KMS 4 KB per-API-call payload limit and offloads bulk encryption from KMS.

What are KMS key rotation rules?
?
**Quick:** Customer-managed keys can auto-rotate yearly; AWS-managed rotate yearly automatically; imported and asymmetric keys must be rotated manually.<br>
**Deeper:** Automatic rotation generates new key material yearly but the key ARN/ID stays the same — old ciphertexts can still be decrypted because KMS keeps prior backing keys. Exam gotcha: imported key material (BYOK), asymmetric keys, and custom key stores do NOT support automatic rotation — you must rotate manually by creating a new key and updating aliases.

What is the difference between Secrets Manager and SSM Parameter Store?
?
**Quick:** Secrets Manager auto-rotates and costs more; Parameter Store is cheaper and simpler.<br>
**Deeper:** Secrets Manager: $0.40/secret/month, native RDS/Redshift/DocumentDB rotation via Lambda, cross-region replication. Parameter Store: free for Standard tier (up to 10,000 params, 4 KB), Advanced tier paid (8 KB, expiration, more throughput). Both encrypt with KMS. Exam trick: "automatic database password rotation" = Secrets Manager; "store many config values cheaply" = Parameter Store.

What is the difference between Cognito User Pools and Identity Pools?
?
**Quick:** User Pools authenticate (sign-up/sign-in); Identity Pools authorize AWS access via temp credentials.<br>
**Deeper:** User Pool = your user directory, returns JWT (ID, access, refresh tokens), supports MFA, social IdPs. Identity Pool = federated identity broker that takes any auth token (User Pool, Google, Facebook, SAML) and exchanges it for STS temp credentials to call AWS services directly. Exam trick: "mobile app uploads photo directly to S3" = Identity Pool. "Manage app's user database" = User Pool. Often used together.

What is a permissions boundary?
?
**Quick:** An IAM policy that sets the MAX permissions an identity can have.<br>
**Deeper:** Attached to a user or role, the effective permissions are the intersection of identity policy AND boundary. Useful for delegating user creation safely — devs can create roles but only with permissions within the boundary. Exam trick: SCPs (Organizations) are similar but apply at the account level; permissions boundaries apply per identity.

What are the IAM credential best practices?
?
**Quick:** No root keys, enable MFA, rotate keys, prefer roles, use least privilege.<br>
**Deeper:** Lock away root account credentials and use it only for the few tasks requiring it. Enable MFA on root and all users. Rotate access keys every 90 days. Use roles for EC2/Lambda/cross-account instead of access keys. Use access advisor to find unused permissions and trim. Exam best practice favorite: "best way to grant EC2 access to S3" = instance profile with role, NOT access keys in user-data.

What is the IAM policy structure?
?
**Quick:** JSON with Version, Statement (Effect, Action, Resource, Condition, Principal).<br>
**Deeper:** Effect = Allow/Deny. Action = service:Operation (e.g., s3:GetObject), supports wildcards. Resource = ARN(s). Condition = key/operator/value (e.g., aws:SourceIp, aws:MultiFactorAuthPresent). Principal = required in resource policies, identifies who. Exam trick: NotAction and NotResource invert matching — use carefully, common cause of accidentally over-permissive policies.
