# Security / DevSecOps Interview Questions

## DevSecOps Concepts
1. What is DevSecOps?
   The practice of integrating security tooling and practices directly into the CI/CD pipeline and development workflow — SAST, DAST, dependency/image scanning, secrets detection — rather than treating security as a separate late-stage gate. "Security is everyone's job, built in from the start."

2. What is SAST?
   Static Application Security Testing — analyzes source code (without executing it) to find security vulnerabilities (injection flaws, insecure patterns, hardcoded secrets) early, typically run at commit/PR time (e.g., SonarQube, Semgrep, CodeQL).

3. What is DAST?
   Dynamic Application Security Testing — tests a running application from the outside (like an attacker would), sending malicious/malformed inputs to find vulnerabilities such as XSS, SQL injection, and misconfigurations that only manifest at runtime (e.g., OWASP ZAP, Burp Suite).

4. What is SBOM?
   Software Bill of Materials — a complete inventory of all components (libraries, dependencies, versions) that make up a software artifact, used to quickly identify exposure when a new vulnerability (e.g., Log4Shell) is disclosed in any dependency.

5. What security tools have you implemented in CI/CD?
   Representative stack: SAST (SonarQube/Semgrep) at PR time, dependency scanning (Dependabot/Snyk), container image scanning (Trivy/Grype) before push to registry, secrets scanning (gitleaks/GitGuardian) on every commit, and DAST (OWASP ZAP) against a staging environment before production release.

## Security Fundamentals
6. What is the Zero Trust security model?
   A security approach based on "never trust, always verify" — no user, device, or service is trusted by default regardless of whether it's inside or outside the network perimeter; every request is authenticated, authorized, and encrypted, with access granted per-request based on least privilege rather than implicit trust from network location.

7. What is the difference between Authentication and Authorization?
   **Authentication** verifies *who you are* (login, MFA, certificates). **Authorization** determines *what you're allowed to do* once authenticated (permissions, roles, policies) — authentication always happens first.

8. What is Multi-Factor Authentication (MFA)?
   Requiring two or more independent verification factors to authenticate — typically something you know (password), something you have (a hardware token/authenticator app code), and/or something you are (biometrics) — significantly reducing the risk of account compromise from a single leaked credential.

9. What is Single Sign-On (SSO)?
   An authentication scheme where a user logs in once with a central identity provider and gains access to multiple independent applications/services without re-authenticating to each one separately (e.g., via SAML or OIDC).

10. What is OAuth?
    An authorization framework (not authentication itself) that lets a user grant a third-party application limited access to their resources on another service, without sharing their password — using tokens issued by an authorization server. (OIDC is built on top of OAuth 2.0 to add an actual authentication/identity layer.)

11. What is the difference between Symmetric and Asymmetric encryption?
    **Symmetric** encryption uses the same single key for both encryption and decryption — fast, but the key must be securely shared between parties beforehand (e.g., AES). **Asymmetric** encryption uses a mathematically linked key pair — a public key (shared freely) to encrypt, and a private key (kept secret) to decrypt — solving the key-distribution problem at the cost of being computationally slower (e.g., RSA, ECC). In practice, TLS uses asymmetric crypto to securely exchange a symmetric session key, then symmetric crypto for the actual bulk data transfer.

12. What is Hashing, and how does it differ from encryption?
    Hashing is a one-way transformation that maps input data to a fixed-size output (a digest) — it's not meant to be reversed, used for integrity verification and password storage. Encryption is explicitly reversible (with the right key) — meant to protect confidentiality while still allowing the original data to be recovered.

13. What is the OWASP Top 10?
    A regularly updated industry-standard list of the ten most critical web application security risks (e.g., broken access control, cryptographic failures, injection, insecure design, security misconfiguration, vulnerable/outdated components), used as a baseline checklist for secure development and security testing.

14. What is Cross-Site Scripting (XSS)?
    A vulnerability where an attacker injects malicious client-side script into a web page viewed by other users (e.g., via an unescaped input field), which then executes in the victim's browser session — used to steal cookies/session tokens or perform actions as the victim. Prevented primarily by output encoding/escaping and Content Security Policy (CSP).

15. What is SQL Injection, and how do you prevent it?
    An attack where untrusted input is concatenated directly into a SQL query, letting an attacker manipulate the query's logic (e.g., `' OR '1'='1`) to bypass authentication or extract/modify data. Prevented by always using parameterized queries/prepared statements (never string-concatenating user input into SQL), input validation, and least-privilege database accounts.

16. What is CSRF (Cross-Site Request Forgery), and how do you handle it?
    An attack that tricks an authenticated user's browser into unknowingly submitting a malicious request to a site they're logged into (exploiting the browser's automatic cookie inclusion). Mitigated with anti-CSRF tokens (unique per-session/per-request tokens validated server-side), the `SameSite` cookie attribute, and checking the `Origin`/`Referer` header on state-changing requests.

17. What is a DDoS attack, and how do you mitigate it?
    A Distributed Denial of Service attack floods a target with traffic from many sources simultaneously to exhaust its resources and make it unavailable to legitimate users. Mitigated with AWS Shield (Standard/Advanced), WAF rate-based rules, CDN/edge caching to absorb volumetric traffic, auto-scaling to absorb load, and rate limiting/IP reputation filtering at the edge.

18. What is a Man-in-the-Middle (MITM) attack?
    An attack where an attacker secretly intercepts (and potentially alters) communication between two parties who believe they're communicating directly with each other — mitigated primarily by enforcing TLS/HTTPS everywhere, certificate pinning, and mutual TLS for sensitive service-to-service communication.

## CI/CD Pipeline Security
19. How do you secure a CI/CD pipeline?
    Store all secrets in a dedicated secrets manager (not in code/config); use short-lived, scoped credentials (OIDC federation to cloud providers) instead of static keys; scan code (SAST), dependencies, and images at every build; sign and verify artifacts; restrict who can modify pipeline definitions/approve production deploys; isolate build environments (ephemeral runners); and audit/log all pipeline executions.

20. What DevSecOps tools have you used?
    Representative examples: SonarQube/Semgrep (SAST), Trivy/Grype (image scanning), OWASP ZAP (DAST), gitleaks/GitGuardian (secrets scanning), Snyk/Dependabot (dependency scanning), HashiCorp Vault/AWS Secrets Manager (secrets management), and OPA/Conftest for policy-as-code.

21. What is GitGuardian?
    A secrets-detection platform that continuously scans repositories (and can hook into commits/PRs in real time) for accidentally committed secrets — API keys, credentials, certificates — alerting immediately so they can be revoked/rotated before being exploited.

22. What would you do if a developer accidentally committed a secret?
    Immediately revoke/rotate the exposed credential at the source (not just remove it from the repo — the commit history still has it, and it may already be scraped by bots). Then remove it from git history (`git filter-repo`/BFG Repo-Cleaner) if the repo is public or history-sensitive, audit for any unauthorized use of the credential during the exposure window, and add/verify pre-commit secret-scanning hooks to prevent recurrence.

23. Someone on your team accidentally committed an AWS secret key to GitHub. What do you do?
    1. Immediately deactivate/delete the exposed IAM access key in AWS (don't wait to clean up git history first — assume it's already been scraped).
    2. Check CloudTrail for any unauthorized activity using that key during the exposure window.
    3. Issue a new key (or better, migrate that use case to an IAM role) and update the application.
    4. Remove the secret from git history if needed.
    5. Add secret-scanning (GitGuardian/gitleaks) as a pre-commit hook or CI gate to prevent recurrence.

24. How do you handle secrets in CI/CD?
    Store them in a secrets manager (Vault, AWS Secrets Manager, GitHub/GitLab CI secrets), inject at runtime as environment variables/files rather than baking into images or committing to code, scope access to only the jobs/environments that need them, prefer short-lived OIDC-federated credentials over static keys, and ensure logs mask/never print secret values.

25. How do you implement end-to-end secure CI/CD?
    Shift security left at every stage: pre-commit secret scanning, SAST on every PR, dependency/SBOM scanning, signed commits, least-privilege pipeline credentials via OIDC, image scanning before registry push, image signing (Cosign) and verification before deploy, policy-as-code (OPA) gating infrastructure changes, DAST against staging, and audit logging of every deploy — with production deploys gated behind passing all of the above plus manual approval for high-risk changes.

## Secrets Management
26. How do you encrypt secrets?
    Use envelope encryption via a managed KMS (AWS KMS, GCP KMS) — a data encryption key encrypts the secret, and a master key (held in the KMS, never exported) encrypts the data key — or a dedicated secrets manager (Vault, AWS Secrets Manager) that handles encryption/decryption/access control transparently.

27. How do you rotate credentials and API keys?
    Automate rotation via the secrets manager's native rotation feature (e.g., AWS Secrets Manager rotation Lambdas for RDS credentials) on a schedule; for manually-managed keys, generate a new key, update all consumers to use it, verify, then revoke the old key — never rotate by revoking first (causes an outage).

28. How do you rotate secrets without downtime?
    Support two valid credentials simultaneously during the transition: issue the new secret alongside the still-valid old one, roll out consumers to the new secret gradually, confirm all consumers have switched (via logs/metrics), then revoke the old secret — this "dual validity" window is what avoids an outage.

29. Why External Secrets Operator instead of putting secrets directly in Kubernetes Secrets?
    Native Kubernetes Secrets are only base64-encoded (not encrypted) by default and are duplicated/managed manually per cluster. External Secrets Operator instead syncs secrets from an external, properly access-controlled and audited secrets manager (AWS Secrets Manager, Vault) into Kubernetes Secrets automatically — keeping the actual source of truth in a purpose-built system with rotation, versioning, and fine-grained access control, while still letting pods consume them the normal Kubernetes way.

## Image & Code Scanning
30. How do you implement image scanning?
    Run a scanner (Trivy, Grype, or a registry-native scanner like ECR scanning) against the built container image as a CI pipeline step before it's pushed/deployed, failing the build on critical/high vulnerabilities above a defined threshold, and re-scan images periodically post-deployment since new CVEs are discovered after the fact.

31. How do you implement code scanning (SAST)?
    Integrate a SAST tool (SonarQube, Semgrep, CodeQL) as a required CI check on every PR, configure quality/security gates that block merge on new critical findings, and track/triage existing findings (technical debt) separately from new-code gates so legacy issues don't block unrelated work.

32. What is the difference between a SonarQube Quality Gate and a Quality Profile?
    A **Quality Profile** defines *which rules* are checked for a given language (the ruleset itself — code smells, bugs, vulnerabilities to look for). A **Quality Gate** defines the *pass/fail conditions* based on the analysis results (e.g., "0 new bugs, coverage ≥ 80%, no new critical vulnerabilities") — the Profile determines what's checked, the Gate determines what counts as passing.

33. What is a Sonar Runner/Scanner?
    The client-side tool (`sonar-scanner`) that runs in your build/CI pipeline, analyzes the codebase according to the configured Quality Profile, and uploads the results to the SonarQube server for gate evaluation and dashboard display.

34. How do you manage artifact signing?
    Sign build artifacts/container images cryptographically (e.g., Cosign for container images, GPG for packages) as a pipeline step after build, store the signature alongside the artifact, and enforce signature verification before deployment (admission controller policy in Kubernetes, or a pipeline gate) so only artifacts built by the trusted pipeline can run.

35. If Trivy finds vulnerabilities in an image, how do you fix them?
    Check if a patched version of the vulnerable package/base image is available and bump to it; if the vulnerability is in a base OS package, update to a newer/patched base image; if no fix exists yet and the vulnerability isn't actually exploitable in your context, document and suppress it explicitly (with justification, via `.trivyignore`) rather than silently ignoring; then rebuild and re-scan to confirm resolution.

## Cloud & Infrastructure Security
36. How do you secure cloud workloads?
    Apply least-privilege IAM everywhere (roles over static keys), encrypt data at rest and in transit, use security groups/NACLs and private subnets to minimize exposure, enable logging/monitoring (CloudTrail, VPC Flow Logs, GuardDuty), patch OS/runtime regularly, scan images/dependencies, and use WAF/Shield for internet-facing services.

37. How do you secure Kubernetes workloads?
    Least-privilege RBAC, Network Policies to restrict pod-to-pod traffic, Pod Security Standards (non-root, read-only root filesystem, no privilege escalation), image scanning before deploy, secrets via External Secrets Operator rather than plain Secrets, and admission controllers (OPA/Gatekeeper, Kyverno) to enforce policy at deploy time.

38. How do you manage IAM permissions following least privilege?
    Start from zero permissions and grant only what's demonstrably needed, using IAM Access Analyzer/CloudTrail to identify actually-used actions and trim unused grants over time; prefer scoped custom policies over broad managed policies; use conditions (MFA, source IP, tags) for additional restriction; and review permissions periodically.

39. How do you restrict manual changes to infrastructure?
    Enforce that all infrastructure changes go through IaC + CI/CD (Terraform via a pipeline, GitOps for Kubernetes) rather than console/CLI access; restrict direct console/CLI write access via IAM/SCPs (read-only for most engineers); use drift detection to catch and alert on any out-of-band manual changes; and require PR review for all infra changes.

40. How do you perform vulnerability management?
    Continuously scan infrastructure, images, and dependencies for known CVEs; maintain an SBOM to quickly assess exposure when new CVEs are disclosed; triage findings by severity/exploitability/exposure; track remediation SLAs by severity; and patch systematically rather than reactively.

41. What is Fail2Ban?
    A log-parsing intrusion prevention tool that monitors logs (e.g., SSH auth logs) for repeated failed login attempts and automatically bans the offending IP (via firewall rules) for a configurable period — a lightweight defense against brute-force attacks.

## AWS WAF & Shield
42. What attacks can WAF prevent?
    SQL injection, cross-site scripting (XSS), bad bots/scrapers, known malicious IP patterns, and — via rate-based rules — basic application-layer (L7) DDoS/brute-force patterns, by inspecting and filtering HTTP requests before they reach the origin.

43. What is Rate Limiting in WAF?
    A rule type that tracks the number of requests from a given IP (or other key) over a rolling time window and automatically blocks/challenges that source once it exceeds a defined threshold — used to throttle abusive clients or brute-force/scraping attempts.

44. How do you block a malicious IP?
    Add an IP-set-based Deny rule in WAF (fastest, edge-level), or a Security Group/NACL deny rule at the network layer, or a Fail2Ban ban at the host level — WAF is preferred for internet-facing HTTP services since it blocks before the request ever reaches your infrastructure.

45. How do you block traffic from specific countries?
    Use a **Geo Match** rule in AWS WAF (or CloudFront's Geo Restriction feature), specifying the country codes to block/allow.

46. What is AWS Shield?
    AWS's managed DDoS protection service. **Shield Standard** is automatic and free, protecting against common network/transport-layer (L3/L4) DDoS attacks for all AWS customers. **Shield Advanced** is a paid tier adding more sophisticated attack detection/mitigation, cost protection during attacks, and 24/7 access to the AWS DDoS Response Team (DRT).

47. Difference between WAF and Shield?
    **WAF** operates at Layer 7, filtering HTTP requests based on rules (SQLi, XSS, rate limiting, geo-blocking) — it's about inspecting request *content*. **Shield** operates primarily at Layer 3/4, protecting against volumetric/protocol DDoS attacks that try to overwhelm network capacity — it's about absorbing/mitigating traffic *volume*. They're complementary and often used together.

48. What is KMS?
    AWS Key Management Service — a managed service for creating and controlling encryption keys used to encrypt data across AWS services (S3, EBS, RDS, etc.), with fine-grained key policies, automatic key rotation, and full audit logging via CloudTrail.

49. How do you encrypt data in AWS?
    At rest: enable encryption on the storage service itself (S3 SSE-S3/SSE-KMS, EBS encryption, RDS encryption) using KMS-managed or customer-managed keys. In transit: enforce TLS/HTTPS for all client-server and service-to-service communication (ACM certificates on ALB/CloudFront, TLS for database connections).

## Kubernetes Networking Security
50. How do you secure Kafka?
    TLS for encryption in transit (client-broker and broker-broker), SASL (SCRAM/Kerberos) or mutual TLS for authentication, Kafka ACLs for topic-level authorization, network isolation (private subnets/security groups), and encrypted disks for data at rest.

51. How do Network Policies work in Kubernetes?
    A Network Policy is a namespaced resource that selects pods (via labels) and defines allowed ingress/egress rules (by pod selector, namespace selector, IP block, and port) — once any Network Policy selects a pod, all traffic not explicitly allowed is denied by default (implemented by the cluster's CNI plugin, which must support Network Policies — e.g., Calico, Cilium).

## Compliance
52. How do you ensure HIPAA compliance on this infrastructure?
    Encrypt PHI at rest and in transit everywhere, sign a Business Associate Agreement (BAA) with AWS and use only HIPAA-eligible services, enforce strict least-privilege IAM and audit logging (CloudTrail) on all access to PHI, isolate PHI-handling workloads in dedicated accounts/VPCs, enable detailed monitoring/alerting for anomalous access, and ensure backup/DR meets required retention and recovery objectives.

53. If this was a real production system for a healthcare client what would you change?
    Ensure every component handling PHI runs on HIPAA-eligible AWS services under a signed BAA; tighten encryption (at rest and in transit) and key management (customer-managed KMS keys with strict key policies); add detailed audit logging and alerting on all data access; enforce stricter network segmentation between PHI and non-PHI workloads; formalize incident response and breach-notification procedures; and add regular third-party security/compliance audits.
