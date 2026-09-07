# EKS Interview Questions

## EKS Basics
Q: What is EKS?\
Ans: Amazon Elastic Kubernetes Service — a managed Kubernetes control plane service. AWS runs, patches, and scales the control plane (API server, etcd) across multiple AZs for you, while you manage worker nodes (or use Fargate to avoid managing nodes entirely) and workloads.

Q: What are the components of EKS?\
Ans: The managed **control plane** (API server, etcd, scheduler, controller manager — run by AWS), **worker nodes** (self-managed, managed node groups, or Fargate), the **VPC CNI plugin** for pod networking, **IAM integration** (via `aws-auth` ConfigMap or EKS access entries, and IRSA), **CoreDNS** for cluster DNS, and `kube-proxy` for service networking.

Q: Difference between EKS and ECS?\
Ans: **EKS** runs standard Kubernetes — portable across clouds, larger ecosystem (Helm, operators, CRDs), steeper learning curve. **ECS** is AWS's own proprietary container orchestrator — simpler to learn, tightly integrated with AWS services, but not portable outside AWS. Choose EKS for portability/Kubernetes-ecosystem needs, ECS for simplicity within an all-AWS stack.

Q: How does EKS integrate with IAM?\
Ans: Cluster access is controlled by mapping IAM principals to Kubernetes RBAC via EKS access entries (or the legacy `aws-auth` ConfigMap). Workload-level AWS permissions are granted via **IRSA** (IAM Roles for Service Accounts) or EKS Pod Identity, letting individual pods assume a specific IAM role rather than inheriting the node's role.

Q: What are Worker Nodes?\
Ans: The EC2 instances (or Fargate pods) that actually run your Kubernetes pods — they run the kubelet, container runtime, and kube-proxy, and register with the EKS control plane.

Q: What is Fargate in EKS?\
Ans: A serverless compute option for EKS where you don't provision or manage EC2 worker nodes at all — each pod runs in its own isolated Fargate micro-VM, and you're billed per pod based on vCPU/memory requested.

Q: What is a Node Group in EKS?\
Ans: A managed set of EC2 instances (via a Managed Node Group, backed by an ASG) that AWS provisions and can automatically update/scale/drain, following an EKS-optimized AMI and Launch Template you configure.

Q: Why choose EKS over self-managed Kubernetes?\
Ans: AWS manages control plane availability, patching, and etcd backups/scaling for you — removing significant operational burden and single points of failure. You also get native integration with IAM, VPC networking, ALB/NLB, EBS/EFS CSI drivers, and AWS support — at the cost of less control over control-plane internals and a per-cluster hourly control-plane fee.

## Kubernetes Workloads
Q: What is a Kubernetes Pod?\
Ans: The smallest deployable unit in Kubernetes — one or more tightly-coupled containers that share network namespace (same IP/port space) and storage volumes, always scheduled together on the same node.

Q: What is a Kubernetes Deployment?\
Ans: A controller that manages a ReplicaSet of Pods, providing declarative updates (rolling updates, rollbacks), self-healing (replacing failed pods), and scaling for stateless applications.

## Deployment Strategies
Q: What is a Rolling Update in Kubernetes?\
Ans: The default Deployment update strategy — new-version pods are gradually created and old-version pods gradually terminated, controlled by `maxSurge`/`maxUnavailable`, so the app remains available with a mix of old/new versions during rollout.

Q: What is a Blue-Green Deployment?\
Ans: Two full, independent environments (Blue = current, Green = new version) run simultaneously; traffic is switched from Blue to Green all at once (via a Service selector change, Ingress, or load balancer) once Green is validated — enabling instant rollback by switching back.

Q: Difference between Rolling and Blue-Green Deployment?\
Ans: Rolling updates gradually replace instances in place with a mix of both versions live during the transition, using fewer resources but with mixed-version exposure. Blue-Green runs two complete environments simultaneously and cuts over all traffic at once — instant rollback, no mixed-version window, but doubles resource cost during the switch.

Q: How do you achieve zero-downtime deployments?\
Ans: Combine rolling updates (or blue-green/canary) with readiness probes (so traffic isn't sent to a pod until it's actually ready), a PodDisruptionBudget, `preStop` hooks/graceful termination for connection draining, and `maxUnavailable: 0` to guarantee full capacity is maintained throughout the rollout.

Q: Which deployment strategy allows fastest rollback?\
Ans: Blue-Green — rollback is just re-pointing traffic back to the still-running old environment (near-instant), versus a rolling update rollback which has to progressively roll pods back to the previous version.

## IAM & Security
Q: What is IRSA?\
Ans: IAM Roles for Service Accounts — a mechanism that lets a specific Kubernetes ServiceAccount (and therefore the pods using it) assume a distinct IAM role via OIDC federation with the EKS cluster's OIDC provider, without needing node-wide IAM permissions or static credentials.

Q: Explain IRSA. Why not just give the EC2 node an IAM role?\
Ans: Without IRSA, every pod scheduled on a node inherits that node's instance-profile IAM permissions — meaning any workload on a shared node can access anything the node's role allows, violating least privilege. IRSA uses OIDC federation so the EKS control plane issues each pod's ServiceAccount a projected, short-lived token that AWS STS exchanges for temporary credentials scoped to a specific IAM role — so each workload only gets exactly the permissions it needs, independent of what other pods share the node.

Q: How do you secure an EKS cluster?\
Ans: Use IRSA/Pod Identity instead of node-wide IAM roles; restrict the API server endpoint (private endpoint or IP allowlisting); enable and review audit logs (control plane logging to CloudWatch); apply Kubernetes RBAC with least privilege; use Network Policies to restrict pod-to-pod traffic; keep node AMIs and Kubernetes version patched; scan images for vulnerabilities; and use security groups/NACLs to restrict node network exposure.

Q: What are Kubernetes security practices?\
Ans: Least-privilege RBAC, Network Policies, Pod Security Standards/admission controls, running containers as non-root with read-only root filesystems, image scanning in CI, secrets managed via a vault/External Secrets Operator rather than plain Kubernetes Secrets, regular patching, and audit logging enabled.

Q: How would you implement least privilege for your EKS nodes?\
Ans: Avoid attaching broad IAM permissions to the node instance role at all — nodes should only need permissions for cluster operation (ECR pull, CNI, CloudWatch logs). Application-level AWS access is granted per-workload via IRSA/Pod Identity so no pod inherits more than it explicitly needs, and node security groups restrict traffic to only what's required for cluster/pod communication.

## Networking
Q: How does pod networking work in EKS?\
Ans: EKS uses the **VPC CNI plugin**, which assigns each pod a real routable IP address directly from the VPC's subnet CIDR (via ENIs/secondary IPs attached to the node) — meaning pods communicate using native VPC networking rather than an overlay network, and can be reached directly by other VPC resources subject to security groups.

Q: What is a CNI Plugin?\
Ans: Container Network Interface — the standard Kubernetes uses to configure pod networking. EKS's default is the AWS VPC CNI, which assigns VPC-native IPs to pods; alternatives (Calico, Cilium) can be used for overlay networking or advanced network policy features.

Q: What is CoreDNS?\
Ans: The cluster's internal DNS server, running as a Deployment in `kube-system`, that resolves Kubernetes Service names (e.g., `my-svc.my-namespace.svc.cluster.local`) to ClusterIPs so pods can discover each other by name.

## ECS
Q: What is ECS?\
Ans: Elastic Container Service — AWS's native container orchestration service, supporting EC2 or Fargate launch types, with tasks, services, and task definitions instead of Kubernetes' pods/deployments.

Q: What are ECS Launch Types?\
Ans: **EC2** — you manage the underlying EC2 instances that run tasks. **Fargate** — serverless, AWS manages the compute and you just specify vCPU/memory per task.

Q: What is Fargate?\
Ans: A serverless compute engine for containers (usable with both ECS and EKS) — you specify resource requirements per task/pod and AWS provisions and manages the underlying infrastructure, with no EC2 instances to patch or scale yourself.

Q: What is a Task Definition?\
Ans: ECS's blueprint for running containers — analogous to a Kubernetes Pod spec — defining container image(s), CPU/memory, networking mode, IAM roles (execution and task role), volumes, and logging configuration.

Q: What is an ECS Service?\
Ans: A controller that maintains a desired number of running Task instances, handles rolling deployments, integrates with a load balancer's target group, and replaces unhealthy tasks — analogous to a Kubernetes Deployment + Service.

Q: What is ECS Execution Role?\
Ans: The IAM role ECS itself uses on your behalf to perform actions needed to *launch* the task — pulling the container image from ECR, and writing logs to CloudWatch/pulling secrets from Secrets Manager referenced in the task definition.

Q: What is ECS Task Role?\
Ans: The IAM role assumed by the application code *running inside* the container, granting it permissions to call other AWS APIs (e.g., read from S3, write to DynamoDB) — analogous to IRSA in EKS.

Q: Difference between ECS Execution Role and Task Role?\
Ans: The Execution Role is used by the ECS agent/infrastructure to set up the task (pull image, fetch secrets, push logs). The Task Role is used by your application code at runtime to call AWS APIs. They serve different actors and should be scoped independently.

Q: How do ECS tasks access DynamoDB securely?\
Ans: Attach a Task Role with the specific DynamoDB permissions needed (e.g., `dynamodb:GetItem`, `dynamodb:PutItem` scoped to the specific table ARN) — the application's AWS SDK automatically picks up these temporary credentials from the container's metadata endpoint, no static keys needed.

Q: Can .NET applications run on ECS?\
Ans: Yes — .NET (including .NET Framework on Windows containers, and cross-platform .NET on Linux containers) runs fine on ECS with either EC2 (Windows or Linux instances) or Fargate (Linux and Windows Fargate are both supported) launch types.

## Operations
Q: How do you create an EKS cluster?\
Ans: Via `eksctl create cluster`, Terraform (`aws_eks_cluster` + node group resources), CloudFormation, or the AWS Console — all of which provision the managed control plane, then a node group (managed, self-managed, or Fargate profile) is attached so pods have somewhere to run.

Q: How do you access EKS from CLI?\
Ans: Install `kubectl` and the AWS CLI, then run `aws eks update-kubeconfig --name <cluster-name> --region <region>` to generate a kubeconfig entry that uses IAM-based authentication (via the `aws eks get-token`/`aws-iam-authenticator` exec plugin) for cluster access.

Q: How do you expose an application running on EKS to the internet?\
Ans: Create a Kubernetes Service of type `LoadBalancer` (provisions an NLB/CLB) or, more commonly for HTTP(S) apps, an Ingress resource backed by the AWS Load Balancer Controller, which provisions and configures an ALB routing to the appropriate pods.

Q: How do you upgrade an EKS cluster version?\
Ans: Upgrade the control plane first (`aws eks update-cluster-version` or via Terraform/console, one minor version at a time), then upgrade managed node groups (rolling replacement of nodes onto the new version's AMI), and finally upgrade cluster add-ons (CoreDNS, kube-proxy, VPC CNI) to versions compatible with the new Kubernetes version.

Q: How do you upgrade CoreDNS, kube-proxy and VPC CNI?\
Ans: As managed EKS add-ons, update via `aws eks update-addon --cluster-name <name> --addon-name <coredns|kube-proxy|vpc-cni> --addon-version <version>` (or through Terraform's `aws_eks_addon` resource/console), checking AWS's compatibility matrix for the versions supported by your cluster's Kubernetes version first.

Q: How do you design a multi-AZ EKS cluster?\
Ans: Spread worker nodes/node groups across at least 3 Availability Zones, use topology spread constraints/pod anti-affinity so replicas of a workload land across AZs, ensure the VPC has subnets in each AZ used, and rely on the EKS control plane's built-in multi-AZ redundancy (already spread across AZs by AWS automatically).

Q: What is your strategy for zero downtime EKS cluster upgrades?\
Ans: Upgrade the control plane (brief API server disruption only, workloads keep running), then perform a rolling replacement of node groups (create a new node group on the upgraded AMI/version, cordon and drain old nodes gradually with PodDisruptionBudgets respected, let workloads reschedule onto new nodes, then remove old nodes) — ensuring readiness probes and PDBs prevent any window where all replicas of a service are unavailable simultaneously.
