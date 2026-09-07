# Production Scenarios Interview Questions

## Incident Troubleshooting
Q: Application down but EC2 healthy — what will you check?
Ans: The instance-level health check passing just means the OS/network is fine — check if the application process is actually running (`ps`/`systemctl status`), if it's listening on the expected port (`ss -tulnp`), application logs for a crash/startup error, whether it's failing a load balancer health check specifically (different endpoint/threshold than EC2 status checks), and recent deploys/config changes.

Q: Container running but application not accessible — what will you check?
Ans: Confirm the container is actually listening on the expected port inside the container (`docker exec ... ss -tulnp` or `netstat`), check port mapping (`-p host:container` matches what's expected), check the app's bind address (bound to `127.0.0.1` inside the container is unreachable from outside — must bind `0.0.0.0`), check container logs for startup errors, and check any network policy/security group in front of it.

Q: How do you troubleshoot a 502 Bad Gateway?
Ans: A 502 means the proxy/load balancer got an invalid response from the upstream — check if the backend/application is actually up and healthy, check the LB's target group health check status, check for a timeout mismatch (backend slower than LB's timeout), check backend logs for a crash coinciding with the 502, and verify network connectivity/security groups between the LB and backend.

Q: High latency in production — what will you check?
Ans: Check application-level metrics first (p95/p99 latency, error rate, is it isolated to one endpoint/service or system-wide), infrastructure metrics (CPU/memory/disk I/O saturation), database query performance (slow queries, connection pool exhaustion), downstream dependency latency, network issues (DNS resolution time, cross-AZ/region calls), and correlate with any recent deploy or traffic spike.

Q: Deployment caused outage — what will you do?
Ans: Roll back immediately to the last known-good version (fastest path to restore service), then investigate root cause afterward with the pressure off — check deployment logs, diff what changed, and only redeploy the fix once verified in a lower environment. Communicate status to stakeholders throughout.

Q: Application works in Dev but not in Prod — how do you troubleshoot?
Ans: Diff the environments systematically: configuration/environment variables, dependency/library versions, infrastructure differences (network rules, resource limits), data differences (prod data triggering an edge case dev data doesn't), and scale-related issues (works fine at dev's low traffic, breaks under prod load/concurrency).

Q: How do you handle a production outage?
Ans: Acknowledge and triage immediately, assemble the right people, mitigate first (rollback/failover/scale up — restore service before root-causing), communicate status regularly to stakeholders, then once stable, do a proper root cause analysis and follow up with a blameless postmortem and action items.

Q: How do you perform RCA?
Ans: Establish a clear timeline of events, gather evidence (logs, metrics, traces, deploy history) from the incident window, identify the proximate cause and then dig into contributing/root causes (often via "5 whys"), document findings in a blameless postmortem, and define concrete action items with owners to prevent recurrence.

Q: How do you debug a production DNS issue?
Ans: Verify the record itself is correct (`dig` against the authoritative name servers directly, bypassing local cache), check TTL/propagation, check health-check status if using failover/weighted routing (an unhealthy target won't be returned), test resolution from multiple locations/resolvers to rule out a localized caching issue, and check the client's local resolver/cache as a possible red herring.

Q: How do you identify whether the issue is application, service, cluster or infrastructure related?
Ans: Work from the outside in: check infrastructure health first (nodes/instances up, resource saturation), then cluster-level (Kubernetes control plane health, scheduling issues), then service-level (is the Service routing correctly, are endpoints populated), then application-level (logs, error rates, is the specific pod/instance's code failing) — isolating which layer shows the anomaly narrows down the actual fault domain.

Q: How do you troubleshoot intermittent microservice failures?
Ans: Look for patterns correlated with load (does it happen only under high concurrency — connection pool exhaustion, rate limiting), timing (does it correlate with a downstream dependency's own intermittent issues, a cache expiry, a scheduled job), or specific instances (is it isolated to particular pods/nodes — hardware/network flakiness); add distributed tracing if not already present, since intermittent issues are often a timing/race condition invisible in simple logs.

Q: One AWS region goes down — how do you handle failover?
Ans: If designed for multi-region: Route53 health checks detect the primary region's failure and failover routing shifts traffic to the standby/secondary region automatically (assuming data replication — cross-region RDS replica, S3 CRR — keeps that region ready); if not pre-architected for multi-region, this becomes a disaster recovery event requiring manual failover/rebuild, which is exactly why RTO/RPO requirements should drive proactive multi-region design ahead of time.

Q: CI/CD deployment failed in production — what steps do you take?
Ans: Check the pipeline logs for the exact failure point/reason, determine if it's safe to simply retry (transient issue) or needs a fix (real regression), ensure the environment was left in a consistent state (partial deploy rolled back automatically, or needs manual rollback), and communicate status while resolving.

Q: Kubernetes pods are Running but users receive 503 errors — what will you check?
Ans: Check if the pods are actually **Ready** (not just Running — a failing readiness probe means "Running" pods are excluded from Service endpoints), check `kubectl get endpoints` to confirm the Service has healthy backends registered, check Ingress/ALB target group health checks, and check for a mismatch between the Service's selector and the pods' actual labels.

Q: Your database performs well initially, but after a certain month you notice slowness — how do you troubleshoot and fix it?
Ans: Likely a data-growth-related issue: check for missing/degraded indexes as table size grew, check query plans for full table scans that used to be fast on small data, check for table/index bloat needing vacuum/maintenance, check connection pool exhaustion under growing load, and consider partitioning, read replicas, or archiving old data if the table has simply outgrown its original design.

## Disaster Recovery & SRE
Q: What is RTO?
Ans: Recovery Time Objective — the maximum acceptable time to restore service after a disruption; drives how much automation/standby infrastructure you need to invest in (a low RTO requires hot standby, a higher RTO tolerates cold/manual recovery).

Q: What is RPO?
Ans: Recovery Point Objective — the maximum acceptable amount of data loss, measured in time (e.g., "we can lose at most 5 minutes of data"); drives how frequently you must replicate/back up data.

Q: How do you handle production incidents?
Ans: Detect (monitoring/alerting) → triage/assemble responders → mitigate first (rollback, failover, scale) → communicate status → stabilize → root cause analysis → blameless postmortem with concrete follow-up actions.

Q: What is your incident response process?
Ans: A defined on-call/escalation path, a clear incident commander role once severity crosses a threshold, a communication channel/status page for stakeholders, a mitigate-first mindset (restore service before deep-diving root cause), and a mandatory postmortem with tracked action items after resolution.

Q: How do you perform rollback during a failed deployment?
Ans: Re-deploy the last known-good artifact/version immediately (`kubectl rollout undo`, redeploy previous container tag, revert Terraform to prior applied state) rather than trying to forward-fix under pressure — investigate root cause only after service is restored.

Q: How do you design for high availability?
Ans: Eliminate single points of failure at every layer — multi-AZ compute and data, redundant load balancers, health-checked auto-healing (ASG/Kubernetes self-healing), stateless application tiers so any instance can serve any request, and graceful degradation for non-critical dependencies.

Q: How do you ensure disaster recovery readiness?
Ans: Regularly test actual failover (not just assume backups/replicas work), maintain up-to-date runbooks, automate recovery where possible (IaC to rebuild infra, automated failover routing), replicate data continuously to the DR target meeting your RPO, and periodically run full DR drills/game days.

Q: How would you build a disaster recovery solution across regions?
Ans: Choose a DR strategy matching RTO/RPO needs: backup & restore (cheapest, slowest), pilot light (minimal standby infra, scaled up on failover), warm standby (scaled-down but running copy, fast failover), or active-active (full duplicate, near-zero RTO/RPO, most expensive) — replicate data continuously (cross-region RDS replica, S3 CRR), keep infra-as-code ready to provision the DR region, and use Route53 health-check-based failover routing to redirect traffic automatically.

Q: A Terraform script bypasses validation and deletes a production database during peak business hours. What would be your action plan to contain the blast radius?
Ans: Immediately halt any further Terraform runs across the team to prevent compounding the damage; assess what's actually recoverable (automated backups/snapshots, point-in-time recovery, cross-region replica promotion) and begin restoration immediately; communicate impact/ETA to stakeholders; once stabilized, root-cause how validation was bypassed (missing `prevent_destroy`, no plan review/approval gate, or a state mismatch) and add guardrails — mandatory plan review for prod, `prevent_destroy` on critical resources, and removing any path that allows apply without review.

## Architecture Design
Q: How would you handle sudden traffic spikes?
Ans: Auto Scaling (horizontal, reactive to load) combined with pre-warmed capacity if the spike is predictable, a CDN/cache in front to absorb read-heavy load, queue-based buffering for write-heavy/async work so backend systems aren't overwhelmed directly, and circuit breakers/graceful degradation for non-critical features to protect core functionality under extreme load.

Q: How would you migrate a Windows file server to AWS?
Ans: Use AWS FSx for Windows File Server as the target (native SMB support, AD integration), and AWS DataSync (or Storage Gateway for ongoing hybrid access) to migrate the actual file data with minimal downtime — cut over DNS/mount points once the sync is complete and validated.

Q: How would you secure a production AWS environment?
Ans: Least-privilege IAM everywhere (roles over keys), MFA enforced, encrypted data at rest/in transit, private subnets for anything not needing direct internet exposure, WAF/Shield on internet-facing endpoints, GuardDuty/CloudTrail for detection and audit, Config/SCPs for guardrails, and regular patching/vulnerability scanning.

Q: How would you optimize AWS costs?
Ans: Right-size instances based on actual utilization, use Reserved Instances/Savings Plans for steady-state baseline capacity and Spot for fault-tolerant workloads, apply S3 lifecycle policies, clean up unused resources (idle EIPs, unattached EBS volumes, old snapshots), use Auto Scaling to match capacity to actual demand, and review Cost Explorer/Trusted Advisor recommendations regularly.

Q: How would you design a global application with low latency?
Ans: CloudFront/CDN at the edge for static content, multi-region deployment with Route53 latency-based routing directing users to their nearest region, regional data replication (with a strategy for consistency tradeoffs), and Global Accelerator for non-cacheable dynamic traffic needing optimized routing over AWS's backbone.

Q: How would you build a serverless application?
Ans: API Gateway as the entry point, Lambda for compute (stateless, event-driven functions), DynamoDB for storage (fits Lambda's stateless/scaling model well), S3 for static assets/file storage, and SQS/SNS/EventBridge for async decoupling between components — all scaling automatically without managing servers.

Q: How would you decouple microservices in AWS?
Ans: Use SQS for point-to-point asynchronous work queues, SNS (often fanning out to multiple SQS queues) for pub/sub broadcast events, and EventBridge for more complex event routing/filtering across services — so producing services don't need to know about or wait on consuming services directly.

Q: How would you build a secure static website on AWS?
Ans: S3 (private bucket) behind CloudFront with Origin Access Control, HTTPS enforced via ACM certificate + viewer protocol policy, WAF attached for additional protection, and Route53 for custom domain DNS.

Q: How would you implement a CI/CD pipeline on AWS?
Ans: CodeCommit/GitHub for source, CodeBuild (or Jenkins/GitHub Actions) for build/test, an artifact store (ECR for images, S3/CodeArtifact for packages), CodeDeploy (or a Kubernetes-native tool like ArgoCD) for deployment with a chosen strategy (rolling/blue-green/canary), and CloudWatch for pipeline/deployment monitoring and automated rollback triggers.

Q: How would you connect multiple AWS accounts securely?
Ans: AWS Organizations for centralized management with SCPs as guardrails, AWS Identity Center (SSO) for centralized human access, cross-account IAM roles (assumed via STS) for service-to-service access instead of shared credentials, and Transit Gateway for network-level connectivity between accounts' VPCs where needed.

Q: How would you monitor an enterprise AWS environment?
Ans: Centralize CloudTrail logs (org-wide trail to a dedicated logging account), CloudWatch for metrics/alarms across accounts (or a cross-account observability solution), GuardDuty/Security Hub for centralized security findings, and a unified dashboard (Grafana pulling from CloudWatch, or a third-party observability platform) aggregating the whole estate.

Q: How would you design a secure and highly available 3-tier infrastructure using AWS services?
Ans: Public subnets host an internet-facing ALB (with WAF); a private application-tier subnet runs the app on an Auto Scaling Group across multiple AZs, reachable only from the ALB; a private database-tier subnet runs RDS Multi-AZ, reachable only from the app tier — each tier scoped with tight Security Groups, spanning at least 2-3 AZs, with NAT Gateways per AZ for the app tier's outbound needs.

Q: Given two front-end services and 15 back-end services in a GitLab repository, how would you deploy them in Kubernetes?
Ans: Package each service as its own Deployment + Service (ideally via Helm charts or Kustomize overlays for consistency), use a GitOps controller (ArgoCD/Flux) watching the GitLab repo to auto-deploy on merge, route external traffic to the 2 front-ends via Ingress, and keep the 15 back-end services internal (ClusterIP), with each pipeline stage (per-service CI in GitLab) building/pushing its own image tag that the GitOps manifests reference.

Q: Explain the complete request flow from a browser to a backend Kubernetes pod.
Ans: Browser resolves the domain via DNS (Route53) → request hits CloudFront/WAF (if used) → routed to an ALB (provisioned via the ALB Ingress Controller from a Kubernetes Ingress resource) → ALB forwards to a healthy target (pod IP, in IP target-type mode) based on Ingress routing rules → kube-proxy/Service networking isn't in this specific path if using IP-mode ALB targets directly to pods; if instead going through a Kubernetes Service (ClusterIP/NodePort) as an intermediate hop, kube-proxy's iptables/IPVS rules load-balance to one of the matching pod IPs → the pod's container processes the request and returns the response back along the same path.

Q: How would you ensure even Pod distribution across Nodes in a Kubernetes cluster (e.g., 6 Pods across 3 Nodes, 2 per Node)?
Ans: Use `topologySpreadConstraints` on the pod spec with `maxSkew: 1`, `topologyKey: kubernetes.io/hostname`, and `whenUnsatisfiable: DoNotSchedule` — this instructs the scheduler to actively balance replicas evenly across nodes, rejecting placements that would skew distribution beyond the allowed threshold.

Q: A Kubernetes Node has reached its Pod capacity. What would you check and do?
Ans: Check the node's `kubectl describe node` for its max-pods limit (often tied to ENI/IP limits for the AWS VPC CNI) and current allocation; short-term, cordon it and let the scheduler place new pods elsewhere (assuming capacity exists); longer-term, add more nodes (Cluster Autoscaler), use larger instance types (more ENIs/IPs available for more pods), or switch to prefix-delegation mode on the VPC CNI to increase pods-per-node capacity.

## EKS / Kubernetes Deep Dives
Q: Walk me through your architecture. Why did you create two separate projects?
Ans: A common rationale: separating the networking/foundational Terraform project (VPC, subnets, Transit Gateway) from the application/Kubernetes Terraform project decouples their lifecycles — network topology changes rarely and is often owned by a platform/infra team, while application infrastructure changes frequently and is owned by product teams; splitting them limits blast radius (an app-infra change can't accidentally touch network config) and allows independent state files/access control per project.

Q: Why did you choose EKS over self-managed Kubernetes?
Ans: AWS manages control-plane availability, patching, and etcd for you, removing significant operational burden and a major source of production risk, while still giving native integration with IAM (IRSA), VPC networking, and other AWS services — at the cost of a per-cluster control-plane fee and less low-level control over the control plane itself.

Q: Explain IRSA. Why not just give the EC2 node an IAM role?
Ans: Without IRSA, every pod on a node inherits the node's instance-profile permissions — any workload sharing that node could access anything the node's role allows, violating least privilege. IRSA uses OIDC federation so each pod's ServiceAccount can be mapped to its own distinct IAM role via a short-lived, projected token exchanged through AWS STS — scoping AWS permissions per-workload instead of per-node.

Q: Why External Secrets Operator instead of putting secrets directly in Kubernetes Secrets?
Ans: Native Kubernetes Secrets are only base64-encoded (not truly encrypted by default) and managed manually per cluster. External Secrets Operator syncs from a properly access-controlled, audited, rotating secrets manager (AWS Secrets Manager/Vault) into Kubernetes Secrets automatically, keeping the actual source of truth in a purpose-built system while pods still consume it the normal way.

Q: Explain your blue-green strategy. How does the traffic switch actually happen?
Ans: Two full environments (target groups/deployments) run simultaneously — Blue (current) serving live traffic and Green (new version) fully deployed and validated but not yet receiving production traffic; the switch happens at the routing layer — updating the ALB listener rule (or Service selector, in a Kubernetes-native approach) to point to Green's target group all at once — making the cutover near-instant, with Blue kept warm briefly for immediate rollback if needed.

Q: What happens if the PostgreSQL pod crashes and loses data?
Ans: If PostgreSQL is running with a properly configured PersistentVolume (backed by durable storage like EBS, not ephemeral pod storage), data survives the pod crash — Kubernetes reschedules the pod and it remounts the same PVC. True data loss would require the underlying storage itself to fail; mitigated by regular automated backups/snapshots and (for critical systems) a managed RDS instance or a replicated PostgreSQL setup instead of a single unmanaged pod.

Q: How do you handle secrets rotation?
Ans: Automate rotation via the secrets manager's native rotation feature where possible (e.g., AWS Secrets Manager rotation Lambdas for RDS credentials), support a brief dual-validity window (old and new both valid) during the transition to avoid downtime, and ensure consumers (via External Secrets Operator syncing on an interval, or a restart/reload trigger) pick up the new value promptly.

Q: Your NAT Gateway per AZ is expensive. How would you justify it?
Ans: A single NAT Gateway (in one AZ) is a cross-AZ single point of failure — if that AZ has an issue, every other AZ's outbound traffic breaks too, undermining the very multi-AZ HA design the rest of the architecture provides. One NAT Gateway per AZ keeps each AZ's outbound path independent, matching the availability guarantee of the rest of the architecture — the added cost buys real resilience, not just redundancy theater.

Q: If this was a real production system for a healthcare client what would you change?
Ans: Ensure every PHI-handling component runs on HIPAA-eligible services under a signed BAA, tighten encryption/key management (customer-managed KMS keys), add detailed audit logging on all PHI access paths, strengthen network segmentation between PHI and non-PHI workloads, formalize incident response/breach notification procedures, and add regular third-party compliance audits.

Q: How would you make this multi-tenant for multiple clients?
Ans: Depending on isolation requirements: separate AWS accounts per tenant (strongest isolation, via AWS Organizations), separate namespaces with strict RBAC/NetworkPolicy/ResourceQuota per tenant within a shared cluster (lighter-weight, lower isolation), or a hybrid (dedicated clusters for larger/more sensitive tenants, shared cluster for smaller ones) — chosen based on compliance requirements and cost tolerance.

Q: What is missing from your current architecture?
Ans: A representative honest answer: no chaos-engineering/DR game-day testing performed yet, no formal cost-anomaly alerting, limited automated security scanning coverage, and no fully automated multi-region failover — all reasonable next investments as the system matures past MVP.

Q: How do you ensure HIPAA compliance on this infrastructure?
Ans: Only use HIPAA-eligible AWS services under a signed BAA, encrypt PHI at rest and in transit everywhere, enforce strict least-privilege IAM with full audit logging (CloudTrail) on PHI access, isolate PHI workloads network-wise, and meet required backup/retention/recovery objectives.

Q: Someone on your team accidentally committed an AWS secret key to GitHub. What do you do?
Ans: Immediately deactivate the exposed IAM key in AWS (don't wait on git history cleanup — assume it's already scraped), check CloudTrail for unauthorized activity during the exposure window, issue a replacement (or migrate to an IAM role instead), scrub the secret from git history if the repo is public/sensitive, and add automated secret-scanning to prevent recurrence.

Q: How would you implement least privilege for your EKS nodes?
Ans: Keep node instance-role permissions minimal (just what's needed for cluster operation — ECR pull, CNI, logs), and grant application-level AWS access per-workload via IRSA/Pod Identity instead of node-wide roles, so no pod inherits more than it explicitly needs.

Q: How does your system handle a traffic spike say 10x normal load?
Ans: HPA scales pod replicas based on load, Cluster Autoscaler adds nodes to accommodate the extra pods, the ALB/Ingress distributes across the growing pool, and — depending on architecture — a cache/CDN layer absorbs read-heavy spikes and a queue buffers write-heavy bursts so downstream systems (databases) aren't directly overwhelmed by the full spike.

Q: What is your strategy for zero downtime EKS cluster upgrades?
Ans: Upgrade the control plane first (brief API disruption only, workloads unaffected), then perform a rolling node replacement (new node group on the upgraded version, gradually cordon/drain old nodes respecting PodDisruptionBudgets so pods reschedule onto new nodes without dropping below minimum availability), then remove old nodes.

Q: Your Keycloak deployment has 2 replicas. What happens if both pods are on the same node and that node fails?
Ans: Both go down together — a full Keycloak outage until Kubernetes reschedules them elsewhere. This is prevented by Pod Anti-Affinity (or topology spread constraints) ensuring the two replicas are never co-located on the same node.

## CI/CD Pipeline
Q: How would you add a CI/CD pipeline to this project?
Ans: Define stages matching the delivery flow: build → unit test → static/security scan → build & scan container image → push to registry → deploy to a lower environment automatically → integration/smoke test → manual approval gate → deploy to production (rolling/blue-green) → post-deploy health check, wired into a chosen tool (GitHub Actions/Jenkins/GitLab CI) triggered on merge to main.

Q: ArgoCD vs Flux which would you choose and why?
Ans: ArgoCD for a richer UI and easier multi-team/multi-cluster visibility; Flux for a lighter-weight, more modular, automation-first setup tightly integrated with Kustomize/Helm — the choice depends on whether the team values a polished UI (ArgoCD) or minimal footprint/composability (Flux) more.

Q: How do you handle a hotfix that needs to go to production immediately?
Ans: Branch directly off the current production tag/commit (not the potentially-ahead main branch, to avoid pulling in unrelated unreleased changes), apply the minimal fix, run it through an expedited but still-real CI pipeline (tests/scans not skipped), and deploy through the same pipeline with an expedited approval — never bypass the pipeline entirely just because it's urgent.

Q: What if the CI pipeline itself goes down?
Ans: Have a documented manual deployment runbook as a fallback for critical/emergency fixes, prioritize restoring the CI system quickly (treat it as production infrastructure with its own on-call/monitoring), and avoid single-point-of-failure CI architecture where practical (e.g., self-hosted runners with redundancy, or a managed SaaS CI with its own uptime SLA).

Q: How long does your full pipeline take end to end?
Ans: Answer with a concrete, representative figure and its breakdown when asked in an interview — e.g., "~12 minutes: 3 min build, 4 min tests, 2 min image build/scan, 3 min deploy + health check" — and be ready to explain what's been done to optimize it (caching, parallelization) and what the next bottleneck to address is.

Q: How do you handle database migrations in the pipeline?
Ans: Run migrations as a distinct, ordered pipeline step (before or as part of the deploy stage), write migrations to be backward-compatible with the currently-running old application version during rolling deployments (expand-contract pattern: add new columns/tables first, deploy code that can use either, migrate data, then remove old columns in a later release), and always have a tested rollback/down-migration path.

Q: What is CICD architecture in production?
Ans: Source control triggers a pipeline that builds, tests, and scans the code; produces a versioned, immutable artifact (container image); deploys that same artifact progressively through environments (dev → staging → prod) using a consistent deployment strategy per environment; with monitoring/health checks gating promotion and automated or manual rollback on failure — the core principle being the exact same artifact that passed tests is what reaches production, never rebuilt per-environment.

## Cost Optimization
Q: How would you optimize the AWS cost of this infrastructure?
Ans: Right-size compute based on actual utilization, use Savings Plans/Reserved Instances for steady baseline load and Spot for fault-tolerant workloads, apply S3 lifecycle policies, clean up unused resources (idle EIPs, orphaned EBS volumes/snapshots), consolidate/reduce NAT Gateway or CloudWatch costs where safe to do so, and continuously review Cost Explorer/Trusted Advisor findings.

Q: What are VPC Endpoints and why would you add them?
Ans: Private connections from a VPC to supported AWS services (S3, DynamoDB, and many others via PrivateLink) that avoid routing through a NAT Gateway/internet — added both for security (traffic never leaves the AWS network) and cost (avoids NAT Gateway data-processing charges for that traffic).

Q: Blue-green costs 2x resources. How do you justify this to management?
Ans: Frame it against the cost of an outage or failed rollback: blue-green buys near-instant rollback and zero mixed-version risk during a release, which for a high-traffic/high-revenue-impact service is often far cheaper than the cost of extended downtime or a botched release — and the 2x cost is typically only incurred briefly during the cutover window, not as steady-state spend, if the old environment is torn down promptly after validation.

Q: Why gp3 over gp2 for storage?
Ans: gp3 decouples IOPS/throughput from volume size (gp2's performance scaled with size, forcing over-provisioning just to get more IOPS) — gp3 lets you provision exactly the IOPS/throughput needed independently, and is generally cheaper per-GB than gp2 for equivalent performance.

## Deployment Strategy Deep Dives
Q: What if the blue-green switch causes database schema issues?
Ans: This is why blue-green deployments need schema changes handled via the expand-contract pattern — both old and new application versions must be able to work against the same database schema during the transition window, since a database (unlike the app tier) typically isn't duplicated/switched atomically alongside the traffic cutover.

Q: How do you decide when to promote canary to 100%?
Ans: Define objective promotion criteria upfront (error rate, latency percentiles, business metrics) compared against the stable baseline over a sufficient traffic sample/time window — promote automatically (or with sign-off) only once the canary meets those thresholds, and roll back automatically if it breaches them at any point during the canary period.

Q: Have you used Shadow deployment?
Ans: Shadow (mirroring/dark) deployment sends a copy of real production traffic to the new version in parallel, without its responses affecting real users — useful for validating a new version's behavior/performance under genuine production load and data patterns before it ever serves live traffic, particularly valuable for high-risk changes where even a small percentage of real canary traffic is too risky.

Q: Which ONE deployment strategy would you recommend for our projects?
Ans: Rolling updates as the default for most routine, low-risk releases (efficient, no extra infrastructure cost), reserving canary for higher-risk changes needing gradual, metric-gated validation — a single blanket recommendation without knowing the specific risk profile/traffic pattern would be answering the wrong question; the right strategy depends on rollback speed needed vs. resource cost tolerance for that specific service.

Q: What if the network team changes a subnet and recreates it. Does Project 2 break?
Ans: Yes, likely — if Project 2 (the Kubernetes/app Terraform project) references the subnet by ID via a `terraform_remote_state` data source, recreating the subnet changes its ID, and Project 2's next plan would show resources needing replacement (or fail entirely) since they're tied to a now-nonexistent subnet ID — this is exactly the kind of tight coupling that argues for careful, versioned outputs and coordination between the two projects' change processes.

Q: How does Project 2 authenticate to read the S3 state?
Ans: Via an IAM role/policy granting `s3:GetObject` on the specific state file path in Project 1's state bucket (and `dynamodb:GetItem` on the lock table, if applicable) — read-only access to the state artifact itself, used through a `terraform_remote_state` data source pointing at that S3 key.

Q: Can Project 2 accidentally modify Project 1 resources?
Ans: Not if properly separated — Project 2 only *reads* Project 1's state via `terraform_remote_state` (a read-only data source) and has its own separate state file; as long as Project 2's provider credentials/IAM permissions don't also grant write access to Project 1's actual resources, and Project 2 never declares resources with the same addresses as Project 1's, there's no path for it to modify Project 1's infrastructure.

Q: What if you need to pass a value that is not in Project 1 outputs?
Ans: Add the needed value as a new output in Project 1's configuration (if you control that project) and apply it — outputs are just declared attributes, so this is generally low-risk; if you can't modify Project 1, fall back to a data source lookup for that value directly (e.g., a tag-based `aws_subnet` data source query) instead of depending on Project 1's outputs at all.

Q: How have you used the network terraform project inside the kubernetes terraform project?
Ans: The Kubernetes/app project reads the network project's outputs (VPC ID, subnet IDs, security group IDs) via a `terraform_remote_state` data source pointing at the network project's state file, then references those values directly in its own resources (e.g., `data.terraform_remote_state.network.outputs.private_subnet_ids`) — keeping the two projects' lifecycles independent while still composing them together.

## Debugging Real Systems
Q: Production Keycloak is down. Walk me through how you would debug it?
Ans: Check pod status first (`kubectl get pods` — CrashLoopBackOff, Pending, etc.), then logs (`kubectl logs --previous` if crashed), check its database connectivity (Keycloak is stateless itself but depends heavily on its backing DB — a DB outage looks like a Keycloak outage), check resource limits/OOMKills, check recent deploys/config changes, and check the Ingress/load balancer's health check status to confirm the issue is actually at Keycloak and not the routing layer in front of it.

Q: How would you handle a Terraform state corruption?
Ans: Restore from the most recent backup (S3 versioning or the local `.tfstate.backup`), or rebuild resource-by-resource via `terraform import` if no backup exists, then run `terraform plan` to confirm the reconstructed state matches real infrastructure with no unexpected diffs before making any further changes.

Q: The team wants to deploy to multiple AWS accounts. How would you restructure this?
Ans: Adopt AWS Organizations with a clear account structure (e.g., per environment: dev/staging/prod, or per team), use SCPs for org-wide guardrails, separate Terraform state per account/environment, use cross-account IAM roles (assumed via STS, or federated through Identity Center) instead of duplicating credentials, and use a Transit Gateway if cross-account network connectivity is needed.

## Behavioral / Leadership
Q: Introduce yourself and walk through your DevOps experience.
Ans: Structure it as: current role and focus area, the core stack/tools you work with day-to-day, one or two concrete projects/achievements that show impact, and what you're looking for next — kept concise (60-90 seconds) and tailored to the role you're interviewing for.

Q: Tell me about a time you had a production incident. How did you handle it?
Ans: Use the STAR structure: Situation (what broke, impact/scope), Task (your role in the response), Action (what you specifically did to triage/mitigate/resolve), Result (time to resolution, what changed afterward to prevent recurrence) — be specific and honest about what went wrong, not just what went right.

Q: Describe a challenge you faced as a DevOps engineer and how you overcame it.
Ans: Pick a real, specific technical or organizational challenge (a tough migration, a stubborn production issue, getting buy-in for a process change), explain the obstacles and your reasoning for the approach taken, and the concrete outcome/lesson learned.

Q: Would you prefer to work individually or as part of a team?
Ans: A balanced, honest answer acknowledging DevOps work is inherently collaborative (cross-team by nature — dev, ops, security) while also requiring focused independent work (writing IaC, debugging), rather than picking an extreme.

Q: Tell me about a time you learned and implemented something from scratch.
Ans: Pick a concrete example (a new tool/platform adopted, a from-scratch pipeline/architecture) and walk through how you approached learning it (docs, POC, incremental rollout) and the outcome it enabled.

Q: How would you handle a situation where you're not getting help from your team members?
Ans: Try to understand why first (are they overloaded, is the ask unclear, is there a process gap), communicate directly and specifically about what help is needed and by when, escalate to a manager only if direct communication doesn't resolve it, and focus on making it easy for others to help (clear context, well-scoped asks).

Q: Name 5 AWS services you have used, and what are their use cases?
Ans: Representative example: **EC2** (general compute), **S3** (object storage/static hosting), **RDS** (managed relational databases), **EKS** (managed Kubernetes for container orchestration), **CloudWatch** (monitoring/alerting) — tailor the actual list to services genuinely used in your own experience.

Q: How do you keep up with DevOps and cloud technologies?
Ans: Representative approach: following release notes/changelogs for core tools, hands-on experimentation with new features in a sandbox account, reading engineering blogs from major cloud providers and companies at scale, and community engagement (conferences, forums, following practitioners).

Q: Where do you see yourself in 3 years?
Ans: Frame around growth in scope/impact relevant to the role — deeper technical specialization (e.g., platform engineering, SRE) or growing into more architectural/leadership responsibility — tied to what the specific company/role could realistically offer.

Q: How do you handle a junior engineer who keeps breaking production?
Ans: Address it as a coaching/process problem, not just an individual failing — pair on the next few changes to understand the actual gap (skill, process, tooling), check if guardrails are insufficient (should this class of mistake even be possible — better CI checks, required review, safer defaults), and give direct, specific, supportive feedback.

Q: How do you handle disagreement with a senior engineer on architecture?
Ans: Present your reasoning with concrete tradeoffs/data rather than just an opinion, genuinely listen to their reasoning in return, look for the underlying concern behind the disagreement (often both approaches address different valid priorities), and be willing to defer if their reasoning is sound — while advocating for a documented decision (an ADR) so the tradeoff is visible for future reference either way.

Q: How do you keep your team infrastructure knowledge up to date?
Ans: Documentation-as-you-go (README/runbooks updated alongside changes, not after), architecture decision records for significant choices, regular knowledge-sharing sessions/demos, and rotating on-call so knowledge isn't siloed in one person.

Q: Why should we hire you?
Ans: Tie specific, relevant experience directly to the role's actual needs (not a generic pitch) — concrete examples of solving similar problems before, and genuine enthusiasm for the specific problems this team/role deals with.

Q: How long did it take to resolve the incident?
Ans: Answer with the specific, honest timeline from your own experience when asked in an interview, broken into detection time, triage time, and resolution time — this level of specificity is what makes the answer credible.

Q: How many users were impacted?
Ans: Answer with the actual scope/blast radius from your own experience — being able to quantify impact (percentage of users, specific customer segments, revenue impact) demonstrates you understood the incident's real severity, not just the technical fix.

Q: Did you do a post-mortem after the incident?
Ans: Describe the actual blameless postmortem process used — timeline reconstruction, root cause (not just proximate cause), and concrete tracked action items — since "yes, and here's what changed as a result" is a much stronger answer than just "yes."

Q: What would you do differently after that incident?
Ans: Speak to concrete follow-up actions taken (added monitoring/alerting that would have caught it sooner, added a guardrail that would have prevented it entirely, improved the runbook/on-call process) rather than a vague "be more careful."

Q: Was the issue caught in staging first?
Ans: Answer honestly from your experience — if not, that's a legitimate opening to discuss what gap in staging (data realism, load, config parity with prod) let it slip through, and what was done afterward to close that gap.

Q: What is your biggest achievement?
Ans: Pick a concrete, quantifiable outcome (cost saved, uptime improved, incident frequency reduced, migration completed) that's genuinely relevant to the role you're interviewing for, and be ready to explain your specific contribution to it.
