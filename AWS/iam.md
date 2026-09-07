# IAM Interview Questions

## Basics
Q: What is IAM?\
Ans: AWS Identity and Access Management — the service that controls authentication (who can sign in) and authorization (what they're allowed to do) across AWS resources, via users, groups, roles, and policies.

Q: What is IAM Role?\
Ans: An identity with a set of permissions that isn't tied to a specific person — it's assumed temporarily by a trusted entity (an EC2 instance, a Lambda function, another AWS account, a federated user) and grants temporary credentials rather than long-lived ones.

Q: What is IAM Policy?\
Ans: A JSON document that defines permissions — a set of Allow/Deny statements specifying which actions can be performed on which resources, under what conditions. Policies are attached to users, groups, or roles.

Q: What is the difference between IAM User, Group, Role, and Policy?\
Ans: **User** — a persistent identity for a person or service, with long-term credentials. **Group** — a collection of users, used to apply the same policies to all of them at once. **Role** — a temporary identity assumed by trusted entities (no permanent credentials). **Policy** — the actual JSON permission document attached to a user, group, or role defining what it can do.

Q: What is the difference between IAM Role and IAM Policy?\
Ans: A Role is an identity that can be assumed and grants temporary credentials; a Policy is the document that defines what permissions that identity (or a user/group) has. A role has policies attached to it — the role is "who," the policy is "what they can do."

## Policies
Q: What are Managed Policies?\
Ans: Standalone, reusable IAM policies (AWS-managed, provided by AWS for common use cases, or customer-managed, created by you) that can be attached to multiple users/groups/roles. Changes to a managed policy automatically apply everywhere it's attached.

Q: What are Inline Policies?\
Ans: A policy embedded directly inside a single user/group/role, with a strict one-to-one relationship — it can't be attached to any other identity and is deleted when its parent identity is deleted.

Q: Difference between Inline Policies and Managed Policies?\
Ans: Managed policies are reusable, standalone, and can be attached to many identities (easier to maintain and audit centrally). Inline policies are tightly coupled to one identity, useful when you need a strict one-to-one permission set that must never be accidentally reused or shared.

Q: What actions can be used other than S3:GetObject and S3:PutObject?\
Ans: Many others depending on need, e.g.: `s3:ListBucket`, `s3:DeleteObject`, `s3:GetBucketLocation`, `s3:PutObjectAcl`, `s3:GetObjectVersion`, `s3:CopyObject` (implemented as Get+Put), `s3:PutBucketPolicy`, `s3:GetBucketPolicy`, `s3:CreateBucket`, `s3:DeleteBucket` — IAM policies should grant only the specific actions actually required (least privilege), rather than wildcarding `s3:*`.

## Access Keys & Users
Q: How many Access Keys can an IAM User have?\
Ans: Up to 2 active access keys at a time — this limit exists specifically to support zero-downtime key rotation (generate a new key, update apps to use it, then deactivate/delete the old one).

Q: What are Access Keys and Secret Keys?\
Ans: A long-term credential pair for programmatic (CLI/SDK/API) access — the **Access Key ID** identifies the credential, and the **Secret Access Key** is used to cryptographically sign requests. They should be avoided in favor of IAM roles wherever the workload runs on AWS infrastructure, since they're long-lived and risk leaking.

## Organizations & SCPs
Q: What are Service Control Policies (SCPs)?\
Ans: Organization-level policies (used with AWS Organizations) that set the maximum available permissions for accounts within an OU or the whole org. SCPs don't grant permissions themselves — they act as a guardrail/filter on top of whatever IAM permissions exist within the account.

Q: How do SCPs work in AWS Organizations?\
Ans: Applied to the root, an OU, or individual accounts, SCPs restrict what IAM principals in that account can do — even a full-admin user in the account cannot exceed what the SCP allows. They're evaluated alongside IAM policies; the *intersection* of SCP allowances and IAM policy allowances is what's actually permitted.

Q: How do you restrict EC2 instance types across multiple AWS accounts?\
Ans: Attach an SCP at the OU (or account) level with a `Deny` on `ec2:RunInstances`, conditioned on `ec2:InstanceType` not being in the allowed list (using a `StringNotEquals` or `ForAnyValue` condition) — this blocks launching disallowed instance types organization-wide, regardless of what IAM permissions exist inside each account.

Q: Can an SCP have both Allow and Deny statements?\
Ans: Yes. However, since the default FullAWSAccess SCP already allows everything unless restricted, most practical SCPs are either an explicit allow-list (replacing FullAWSAccess) or a set of targeted Deny statements layered on top of it — Deny always wins over Allow in policy evaluation.

Q: What is AWS Organizations?\
Ans: A service for centrally managing multiple AWS accounts — consolidated billing, organizational units (OUs) for grouping accounts, SCPs for centralized guardrails, and delegated administration of other AWS services across accounts.

Q: What is AWS Identity Center?\
Ans: (Formerly AWS SSO) — a service providing centralized single sign-on across multiple AWS accounts and business applications, letting users log in once and access assigned accounts/roles via permission sets, backed by an identity source (built-in directory, Active Directory, or an external IdP).

Q: What is the difference between IAM, AWS Identity Center and AWS Organizations?\
Ans: **IAM** manages identities and permissions *within a single AWS account*. **AWS Organizations** manages and governs *multiple accounts* as a unit (billing, SCPs, OUs). **AWS Identity Center** provides centralized human user *sign-on* across those multiple accounts (mapping workforce identities to IAM roles in each account), sitting on top of Organizations.

## Cross-Account Access
Q: How do you provide cross-account S3 access?\
Ans: Either (a) attach a bucket policy on the S3 bucket granting the other account's principal (user/role ARN) access, or (b) create an IAM role in the bucket's account with the needed S3 permissions and a trust policy allowing the other account to assume it, then have users in the other account `sts:AssumeRole` into it.

Q: How do you provide S3 access to 10 users from another account?\
Ans: Rather than granting each of the 10 users direct access, create one IAM role in the bucket-owning account with a trust policy allowing the other account (or a specific group/role there) to assume it, and grant the role S3 permissions via policy. Users in the other account assume that single role — far more scalable and easier to audit/revoke than 10 separate cross-account grants.

Q: Bucket Policy vs IAM Role for cross-account access?\
Ans: A **bucket policy** is resource-based — attached to the S3 bucket itself, good for simple, direct grants to specific external accounts/principals without requiring role assumption. An **IAM role** is identity-based and requires the accessing principal to explicitly `AssumeRole` — better for scenarios needing temporary credentials, auditability (CloudTrail shows who assumed what), and centralized permission management for many external users.

Q: What is the best scalable approach for multi-user S3 access?\
Ans: Use IAM roles (not per-user policies or per-user bucket policy entries): group users, attach one role per access pattern/tier, and have users assume the appropriate role. This centralizes permission changes to one place, supports temporary credentials, and scales to any number of users without bucket policy sprawl.

## Best Practices
Q: How do you manage IAM permissions following least privilege?\
Ans: Start with no permissions and add only what's explicitly needed; use IAM Access Analyzer and CloudTrail to identify actually-used permissions and trim unused ones; prefer scoped custom policies over broad AWS-managed ones like `AdministratorAccess`; use conditions (source IP, MFA, tags) to further restrict; grant access via roles/temporary credentials rather than standing user permissions; and review/rotate permissions regularly.

Q: Why should IAM Roles be used instead of Access Keys?\
Ans: Roles issue short-lived, automatically-rotated temporary credentials with no secret to store, leak, or manually rotate — removing the biggest source of credential-leak incidents (hardcoded/committed access keys). They're also easier to audit (tied to an assumed session) and can be revoked instantly by changing the role's trust/permission policy.

Q: How do you securely provide AWS permissions to an EC2 instance?\
Ans: Attach an IAM role via an **Instance Profile** to the EC2 instance. The instance retrieves temporary, automatically-rotated credentials from the Instance Metadata Service (IMDS) — no access keys are ever stored on the instance. This is strictly preferred over embedding access keys in user data, config files, or environment variables.
