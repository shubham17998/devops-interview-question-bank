# Kubernetes Interview Questions

## Basics

1. Q: What is Kubernetes?
   Ans: An open-source container orchestration platform that automates deployment, scaling, self-healing, and management of containerized applications across a cluster of machines.

2. Q: Explain Kubernetes Architecture.
   Ans: A cluster has a **control plane** (API Server — the front door for all operations; etcd — the cluster's key-value data store; Scheduler — assigns pods to nodes; Controller Manager — runs reconciliation loops keeping actual state matching desired state) and **worker nodes** (kubelet — manages pods on that node; kube-proxy — handles Service networking rules; container runtime — actually runs containers).

3. Q: What is a Pod?
   Ans: The smallest deployable unit in Kubernetes — one or more tightly-coupled containers sharing the same network namespace (IP, port space) and storage volumes, always scheduled together on the same node.

4. Q: What is a Namespace?
   Ans: A logical partition within a cluster used to divide resources between teams/projects/environments, providing scope for names (so resources with the same name can exist in different namespaces) and a boundary for RBAC/ResourceQuota/NetworkPolicy application.

5. Q: What is a Service?
   Ans: A stable network abstraction (a fixed virtual IP and DNS name) that provides consistent access to a dynamic, changing set of pods, load-balancing traffic across them by label selector.

6. Q: Types of Kubernetes Services?
   Ans: ClusterIP (internal-only, default), NodePort (exposes a static port on every node's IP), LoadBalancer (provisions an external cloud load balancer), and ExternalName (maps a Service to an external DNS name, no proxying).

7. Q: What is the difference between ClusterIP and NodePort?
   Ans: ClusterIP is only reachable from within the cluster (internal virtual IP). NodePort additionally opens a static port (30000-32767 range) on every node's IP, making the service reachable from outside the cluster via `<any-node-ip>:<nodePort>`.

8. Q: Difference between Deployment and Service?
   Ans: A **Deployment** manages the actual running pods (desired replica count, rolling updates, self-healing). A **Service** provides a stable network endpoint to reach whatever pods currently exist/match its selector — they're complementary: a Deployment manages *what's running*, a Service manages *how to reach it*.

9. Q: What is an Ingress in Kubernetes?
   Ans: An API object defining HTTP(S) routing rules (host/path-based) into the cluster, implemented by an Ingress Controller (e.g., NGINX, ALB Ingress Controller) which provisions/configures the actual reverse proxy/load balancer.

10. Q: What is a Secret in Kubernetes?
    Ans: An object for storing sensitive data (passwords, tokens, keys), base64-encoded (not encrypted by default — encryption at rest must be separately enabled), consumable by pods as environment variables or mounted volumes, kept separate from ConfigMaps so it can be handled with tighter access controls.

11. Q: What is a DaemonSet?
    Ans: A controller that ensures exactly one copy of a pod runs on every (or a selected subset of) node in the cluster — used for node-level agents like log collectors, monitoring agents, or CNI/storage plugins.

12. Q: What is a Kubernetes Job?
    Ans: A controller that runs one or more pods to completion for a finite/batch task (not a long-running service) — retries on failure up to a configured limit, and tracks successful completions.

13. Q: What role does etcd play in Kubernetes? Can Kubernetes function without it?
    Ans: etcd is the cluster's single source of truth — a distributed, consistent key-value store holding all cluster state (objects, config, secrets). No — Kubernetes cannot function without it; if etcd is lost/corrupted with no backup, the cluster loses all knowledge of its desired state (though running pods may continue briefly, nothing can be scheduled/reconciled/updated).

14. Q: What are the worker-node components, and what does each do?
    Ans: **kubelet** — the primary node agent; ensures containers described in PodSpecs are running and healthy. **kube-proxy** — maintains network rules (iptables/IPVS) implementing Service virtual IPs and load balancing. **Container runtime** (containerd/CRI-O) — actually pulls images and runs containers.

15. Q: What is imagePullPolicy?
    Ans: A pod spec field controlling when the kubelet pulls a container image: `Always` (pull every time, even if cached), `IfNotPresent` (pull only if not already cached locally), `Never` (never pull, image must already exist locally). Defaults to `Always` if the tag is `latest`, otherwise `IfNotPresent`.

## Workload Controllers

16. Q: What is a ReplicaSet?
    Ans: A controller ensuring a specified number of identical pod replicas are running at all times, replacing any that fail/are deleted — typically managed indirectly through a Deployment rather than created directly.

17. Q: Difference between Deployment, StatefulSet and DaemonSet?
    Ans: **Deployment** — manages stateless, interchangeable pod replicas with rolling updates. **StatefulSet** — manages stateful pods needing stable, unique network identities and persistent storage per replica (e.g., databases), with ordered/predictable pod naming and scaling. **DaemonSet** — ensures one pod runs per (matching) node, for node-level agents.

18. Q: Difference between ReplicaSet, Deployment and StatefulSet?
    Ans: **ReplicaSet** is the low-level controller just maintaining replica count. **Deployment** wraps a ReplicaSet and adds declarative rolling updates/rollback history on top. **StatefulSet** is a separate controller type for workloads needing stable identity/storage per pod (ordered pod names, per-pod PVCs) — not built on ReplicaSet at all.

19. Q: How do you perform a Kubernetes Deployment?
    Ans: `kubectl apply -f deployment.yaml` (declarative) or imperatively `kubectl create deployment <name> --image=<image>` — Kubernetes then creates the underlying ReplicaSet and pods, and manages rolling updates on subsequent spec changes.

20. Q: What is a Service Account?
    Ans: An identity used by processes running inside pods to authenticate to the Kubernetes API server (distinct from a User Account, which is for human operators) — every pod runs as some Service Account (default, if none specified), which can be bound to RBAC Roles to grant it specific API permissions.

21. Q: Can Kubernetes run in a single-node local environment?
    Ans: Yes — tools like Minikube, kind (Kubernetes in Docker), k3s, or Docker Desktop's built-in Kubernetes let you run a fully functional single-node cluster locally for development/testing.

## Storage

22. Q: What are Persistent Volumes (PV) and Persistent Volume Claims (PVC)?
    Ans: A **PV** is a cluster-level storage resource (provisioned statically by an admin or dynamically via a StorageClass) representing actual backing storage (EBS, NFS, etc.), independent of any pod's lifecycle. A **PVC** is a namespaced request for storage by a user/pod, specifying size/access mode — Kubernetes binds it to a matching PV, and the pod mounts the PVC (abstracting away the underlying storage details).

## Configuration Management

23. Q: What is a ConfigMap?
    Ans: An API object for storing non-sensitive configuration data (key-value pairs, config files) separately from application images, consumable by pods as environment variables, command-line args, or mounted files.

24. Q: When would you use a ConfigMap?
    Ans: Whenever you need to inject environment-specific, non-sensitive configuration (feature flags, URLs, config file contents) into a pod without baking it into the container image — letting the same image run correctly across different environments/settings.

25. Q: What is the difference between Secrets and ConfigMaps in Kubernetes?
    Ans: Both store key-value configuration data consumable by pods the same way, but **Secrets** are intended for sensitive data (base64-encoded, with tighter default access/RBAC conventions and support for encryption at rest), while **ConfigMaps** are for non-sensitive plain configuration.

26. Q: How do you securely store credentials in Kubernetes?
    Ans: Native Secrets alone only base64-encode (not truly encrypt) data by default — use encryption-at-rest for etcd, restrict RBAC access tightly, and preferably sync credentials from an external, properly access-controlled secrets manager (AWS Secrets Manager/Vault) into Kubernetes Secrets via the External Secrets Operator, rather than storing raw secrets directly in manifests/Secrets objects.

## Probes

27. Q: What is a Liveness Probe?
    Ans: A periodic health check the kubelet performs on a container; if it fails repeatedly (past the configured threshold), the kubelet restarts the container — used to recover from a stuck/deadlocked process that's still running but no longer functioning.

28. Q: What is a Readiness Probe?
    Ans: A periodic check determining if a container is ready to accept traffic; if it fails, the pod is removed from the Service's endpoints (no traffic routed to it) without restarting the container — used for temporary "not ready yet" states (startup, warming a cache, temporary dependency unavailability).

29. Q: Difference between Liveness and Readiness probes?
    Ans: Liveness failure → container is **restarted** (assumes it's broken and needs a fresh start). Readiness failure → pod is **removed from Service load balancing** but left running (assumes it's temporarily not able to serve traffic, will recover on its own) — they answer different questions ("is it alive?" vs. "should it get traffic right now?").

## Scaling

30. Q: How do you handle node autoscaling?
    Ans: Deploy the **Cluster Autoscaler**, which watches for pods that can't be scheduled due to insufficient resources and automatically adds nodes (via the underlying cloud ASG/node group) to accommodate them, and removes underutilized nodes when their pods can be rescheduled elsewhere.

31. Q: How do you scale microservices in Kubernetes?
    Ans: Horizontal Pod Autoscaler (HPA) to scale replica count based on CPU/memory or custom metrics, Vertical Pod Autoscaler (VPA) to right-size individual pod resource requests, and Cluster Autoscaler to ensure enough node capacity exists for the scaled-out pods.

32. Q: What is HPA?
    Ans: Horizontal Pod Autoscaler — automatically adjusts a Deployment/StatefulSet's replica count based on observed metrics (CPU/memory utilization by default, or custom/external metrics via metrics adapters) relative to a target value.

33. Q: How does the Vertical Pod Autoscaler (VPA) work?
    Ans: VPA analyzes a pod's historical resource usage and automatically adjusts its CPU/memory *requests and limits* (rather than replica count) — in its default "Auto" update mode it evicts and recreates pods with the recommended new resource values (can't resize a running pod's resources in-place in most Kubernetes versions), or in "Off" mode just provides recommendations.

34. Q: What is the difference between HPA and VPA?
    Ans: HPA scales **out/in** — adjusts the *number* of pod replicas. VPA scales **up/down** — adjusts the *resource requests/limits* of existing pods. They address different scaling dimensions and generally shouldn't both actively manage the same metric on the same workload simultaneously (can conflict).

## Scheduling & Placement

35. Q: What is Affinity?
    Ans: A scheduling rule (Node Affinity or Pod Affinity) expressing a *preference or requirement* for where a pod should be scheduled, based on node labels (Node Affinity) or the presence of other pods (Pod Affinity) — more expressive than simple `nodeSelector`.

36. Q: What is Anti-Affinity?
    Ans: The inverse of Affinity — a rule expressing that a pod should *not* be scheduled near certain nodes/pods, commonly used to spread replicas of the same application across different nodes/zones for fault tolerance.

37. Q: What is Pod Anti-Affinity?
    Ans: Specifically, a rule preventing pods matching a given label selector from being scheduled onto the same node (or topology domain, e.g., zone) as each other — e.g., ensuring no two replicas of the same Deployment land on the same node.

38. Q: What are Taints and Tolerations?
    Ans: A **Taint** applied to a node repels pods from scheduling there unless the pod has a matching **Toleration** — the inverse of Affinity (a property of the node saying "don't schedule here unless explicitly permitted," rather than a property of the pod saying "prefer this node").

39. Q: How do taints and tolerations work?
    Ans: A taint (`key=value:effect`) on a node marks it as unsuitable for pods by default; effects include `NoSchedule` (won't schedule new pods without a matching toleration), `PreferNoSchedule` (soft version), and `NoExecute` (also evicts already-running pods lacking the toleration). A pod's `tolerations` field lists which taints it can tolerate, allowing the scheduler to place it on an otherwise-repelling node.

40. Q: Difference between Taints/Tolerations and Node Affinity?
    Ans: Taints/Tolerations work by **exclusion** — nodes repel pods by default, and only tolerating pods can land there (good for dedicating nodes to specific workloads). Node Affinity works by **attraction** — pods express a preference/requirement for certain node labels, but nothing stops other pods (without that affinity) from also landing there unless combined with taints.

41. Q: How do you run specific microservices on specific nodes?
    Ans: Label the target nodes (`kubectl label node <node> workload=special`), then use `nodeSelector` or Node Affinity in the pod spec to require that label — optionally combined with a matching taint/toleration pair to also prevent *other* workloads from landing on those dedicated nodes.

## Resource Management

42. Q: What is a ResourceQuota?
    Ans: A namespace-scoped object limiting the aggregate resource consumption (CPU, memory, object counts like number of pods/services) that can be used within that namespace — prevents one team/namespace from consuming the entire cluster's capacity.

43. Q: Can we limit CPU and Memory at Namespace level?
    Ans: Yes, via a `ResourceQuota` (aggregate limits for the whole namespace) and/or a `LimitRange` (default/min/max per-container CPU/memory within the namespace, applied automatically to pods that don't specify their own).

## Networking

44. Q: How does Pod-to-Pod communication work?
    Ans: Every pod gets its own cluster-wide routable IP (via the CNI plugin), and pods can communicate directly with each other's pod IP without NAT, regardless of which node they're on — the "flat network" model Kubernetes networking requires the CNI implementation to provide.

45. Q: How do Pods communicate with external databases?
    Ans: Directly via standard networking — if the database is reachable from the cluster's network (e.g., an RDS endpoint reachable from the VPC), pods connect using its DNS name/IP and port like any client, typically with the connection details supplied via a Secret/ConfigMap; if the DB is external to the VPC, a NAT Gateway or VPC Endpoint/peering may be needed depending on setup.

46. Q: How do you block communication between pods in the same namespace?
    Ans: Apply a `NetworkPolicy` with a pod selector matching the target pods and no allowed ingress rules from other pods in the namespace (a default-deny policy), then add specific allow rules only for the pods that should be permitted to talk — requires a CNI plugin that supports Network Policies (Calico, Cilium; the default AWS VPC CNI needs Calico or similar layered on top).

47. Q: How do you secure communication between microservices?
    Ans: Use Network Policies to restrict which services can talk to which, mutual TLS (often via a service mesh like Istio/Linkerd) for encrypted, authenticated service-to-service traffic, and RBAC/ServiceAccount-scoped permissions so a compromised service can't act beyond its intended scope.

48. Q: What is Kubernetes Network Policy?
    Ans: A namespaced resource defining allowed ingress/egress traffic rules for a set of pods (selected by label) — once any policy selects a pod, unlisted traffic is denied by default for that pod, enforced by the CNI plugin.

49. Q: On which resource is Network Policy applied?
    Ans: Pods — a NetworkPolicy selects pods via a `podSelector` and defines what traffic is allowed to/from that selected set.

50. Q: How does a Service know which Pods to route traffic to?
    Ans: Via **label selectors** — a Service's `selector` field matches pod labels; Kubernetes continuously watches for pods matching that selector and maintains them as the Service's Endpoints (or EndpointSlices), which kube-proxy uses to program the actual routing/load-balancing rules.

51. Q: How do you troubleshoot pod networking issues?
    Ans: Check pod status/events (`kubectl describe pod`), verify the pod has an IP and is Ready, test connectivity from inside the pod (`kubectl exec -it <pod> -- curl/ping <target>`), check Service endpoints are populated (`kubectl get endpoints`), check for restrictive NetworkPolicies, and check the CNI plugin's own health/logs on the node.

52. Q: How do you troubleshoot DNS issues in Kubernetes?
    Ans: Verify CoreDNS pods are running and healthy (`kubectl get pods -n kube-system -l k8s-app=kube-dns`), test resolution from a debug pod (`kubectl exec -it <pod> -- nslookup <service>.<namespace>.svc.cluster.local`), check the pod's `/etc/resolv.conf` for correct nameserver/search domains, and check CoreDNS logs/config (`Corefile`) for errors or misrouted forwarding.

## Deployment Strategies

53. Q: What is Blue-Green deployment in Kubernetes?
    Ans: Run the new version (Green) fully alongside the old version (Blue) as a separate Deployment, then switch the Service's selector (or Ingress) to point to Green all at once once validated — instant rollback by switching the selector back to Blue.

54. Q: What is Canary deployment?
    Ans: Gradually shift a small percentage of traffic to the new version alongside the stable version (via weighted routing at the Ingress/service-mesh level, or by running a small number of new-version replicas alongside many old-version ones), increasing the percentage as confidence grows, minimizing blast radius if something's wrong.

55. Q: What is Rolling deployment?
    Ans: The default Deployment update strategy — pods are replaced incrementally (controlled by `maxSurge`/`maxUnavailable`), maintaining availability throughout with a temporary mix of old and new versions.

56. Q: How do you perform zero-downtime deployments?
    Ans: Combine rolling updates with readiness probes (no traffic to a pod until ready), `maxUnavailable: 0` to never drop below full capacity, a PodDisruptionBudget, and graceful shutdown handling (`preStop` hook + `terminationGracePeriodSeconds`) so in-flight requests complete before a pod terminates.

57. Q: What are the types of deployment strategies?
    Ans: Rolling update, Blue-Green, Canary, Recreate (terminate all old pods, then create new ones — brief downtime), and Shadow (mirror production traffic to the new version without affecting real responses, for validation only).

58. Q: What is the rollback strategy in Kubernetes?
    Ans: `kubectl rollout undo deployment/<name>` reverts to the previous ReplicaSet revision (Deployments keep a configurable revision history), or target a specific earlier revision with `--to-revision=<n>`.

59. Q: Which type of deployment will you use for production and why?
    Ans: Depends on risk tolerance and resources: rolling updates for most routine, low-risk releases (efficient, built-in); canary for high-risk changes where gradual validation with real traffic reduces blast radius; blue-green when instant, guaranteed rollback matters more than resource efficiency (e.g., major version changes with schema implications).

## Security

60. Q: How do you secure Kubernetes clusters?
    Ans: Least-privilege RBAC, Network Policies, Pod Security Standards (non-root, no privilege escalation, read-only root filesystem), encrypted etcd at rest, restricted/private API server endpoint, image scanning, regular patching of control plane/nodes, and audit logging enabled.

61. Q: How do you secure an EKS cluster?
    Ans: IRSA/Pod Identity instead of node-wide IAM roles, restrict the API server endpoint (private or IP-allowlisted), enable control-plane audit logging to CloudWatch, RBAC least privilege, Network Policies, patched/managed node groups, and image scanning in CI before deploy.

62. Q: What are Kubernetes security practices?
    Ans: Least-privilege RBAC, Network Policies, Pod Security Standards/admission control, non-root containers, secrets via an external secrets manager, image scanning in CI, regular patching, and audit logging.

63. Q: What is Kubernetes API Server use case?
    Ans: The central management point of the cluster — the front-end for the Kubernetes control plane; every `kubectl` command, controller, and internal component interacts with cluster state exclusively through the API Server, which validates/authorizes requests and persists changes to etcd.

64. Q: What is Role-Based Access Control (RBAC) in Kubernetes?
    Ans: An authorization mechanism that grants permissions to users/ServiceAccounts based on assigned Roles (namespaced) or ClusterRoles (cluster-wide), each defining allowed verbs (get/list/create/delete) on specific resource types — bound to subjects via RoleBindings/ClusterRoleBindings.

65. Q: What is a Custom Resource Definition (CRD)?
    Ans: A mechanism to extend the Kubernetes API with your own custom resource types (beyond built-ins like Pod/Deployment) — paired with a custom controller/operator that watches and reconciles instances of that resource, enabling Kubernetes-native management of anything (databases, certificates, ArgoCD Applications, etc.).

## ArgoCD

66. Q: What is an ArgoCD Project?
    Ans: A logical grouping construct in ArgoCD that restricts what a set of Applications can do — which source repos, destination clusters/namespaces, and resource kinds are allowed — providing multi-tenancy boundaries within a single ArgoCD instance.

67. Q: What are the steps to create an ArgoCD Project?
    Ans: Define a `AppProject` custom resource (or via `argocd proj create`) specifying allowed source repositories, destination clusters/namespaces, allowed/denied resource kinds (cluster and namespace scoped), and optionally roles/RBAC policies scoped to that project — then create Applications referencing that project.

68. Q: How does ArgoCD work?
    Ans: ArgoCD continuously compares the desired state declared in a Git repo (manifests/Helm/Kustomize) against the live state of the target cluster; when they diverge, it reports "OutOfSync" and — automatically (if auto-sync is enabled) or on manual trigger — applies the changes needed to reconcile the cluster to match Git, implementing the GitOps pull-based model.

69. Q: What are ArgoCD CLI commands?
    Ans: Common ones: `argocd app create`, `argocd app sync`, `argocd app get <app>`, `argocd app diff <app>`, `argocd app history <app>`, `argocd app rollback <app> <revision>`, `argocd proj create`, `argocd login`.

70. Q: ArgoCD vs Flux — which would you choose and why?
    Ans: Both are GitOps controllers reconciling cluster state from Git. **ArgoCD** offers a richer, more polished Web UI, explicit Application/Project abstractions, and strong multi-cluster management — often preferred for visibility and easier onboarding of less CLI-comfortable teams. **Flux** is more lightweight, modular (separate controllers you compose), and integrates tightly with Kustomize/Helm natively as CNCF-graduated tooling — often preferred for a more "pure," minimal, automation-first GitOps setup. Choice generally comes down to whether a rich UI (ArgoCD) or a lighter, more composable toolchain (Flux) fits the team better.

## Ingress & Gateway

71. Q: What is an Ingress Controller?
    Ans: The actual implementation (NGINX Ingress Controller, AWS ALB Ingress Controller, Traefik, etc.) that watches Ingress resources and configures a real reverse proxy/load balancer accordingly — the Ingress object alone is just configuration; nothing happens without a controller to act on it.

72. Q: Is Ingress deprecated?
    Ans: Ingress itself isn't deprecated, but the newer **Gateway API** is the designated long-term evolution/replacement, offering a more expressive, role-oriented, extensible model — many new features/investments are going into Gateway API, though Ingress remains widely supported and used.

73. Q: What is Gateway API?
    Ans: A newer, more expressive Kubernetes API for traffic routing (superseding Ingress's limitations) — splitting configuration into GatewayClass, Gateway, and Route resources (HTTPRoute, TCPRoute, etc.), supporting richer routing/traffic-splitting semantics and clearer separation of infrastructure vs. application-team concerns.

74. Q: Explain the ALB Ingress Controller workflow.
    Ans: The controller watches Kubernetes Ingress resources; when one is created/updated, it provisions/configures an actual AWS ALB (and target groups) matching the Ingress's rules (host/path routing), registers pod IPs (or node ports, depending on target type) as ALB targets, and keeps the ALB's configuration continuously synced with the Ingress resource's desired state.

75. Q: How does the ALB know which pods are healthy?
    Ans: The ALB Ingress Controller configures target group health checks (HTTP path, interval, thresholds) pointing directly at pod IPs (in IP target-type mode) or node ports; the ALB itself performs these health checks independently and routes traffic only to targets currently passing them.

## Monitoring & Observability

76. Q: How do you monitor Kubernetes nodes and pods?
    Ans: Deploy `kube-state-metrics` (object-level state) and Node Exporter/cAdvisor (resource usage) with Prometheus scraping them (commonly via the kube-prometheus-stack Helm chart), visualized in Grafana, alerting on restarts, OOMKills, node pressure, and resource saturation.

77. Q: How do you configure Prometheus to scrape all namespaces?
    Ans: Use Kubernetes service discovery (`kubernetes_sd_configs`) with role `pod`/`endpoints` without restricting `namespaces.names` (leave empty/omitted to watch all namespaces), then filter which specific pods to scrape via `relabel_configs` based on annotations like `prometheus.io/scrape`.

78. Q: How do you update a container image in a Deployment?
    Ans: `kubectl set image deployment/<name> <container>=<new_image>` (triggers a rolling update automatically), or edit the manifest's image field and `kubectl apply -f` — Kubernetes handles the rolling replacement per the Deployment's update strategy.

## Troubleshooting

79. Q: How do you troubleshoot CrashLoopBackOff?
    Ans: `kubectl describe pod` for events, `kubectl logs <pod> --previous` (logs from the crashed instance, since the current one just restarted), check for OOMKills (exit code 137), missing config/secrets, failing liveness probes causing repeated restarts, or an application error on startup (bad config, missing dependency).

80. Q: Pod stuck in ContainerCreating state — what will you check?
    Ans: `kubectl describe pod` for events — common causes: image pull failure (bad image name, registry auth issue — `ImagePullBackOff`), a PVC that can't be bound (no matching PV/StorageClass issue), a missing ConfigMap/Secret referenced as a volume, or CNI/networking issues on the node preventing the sandbox from being created.

81. Q: Pods continuously crashing — how do you troubleshoot?
    Ans: Same as CrashLoopBackOff: check `kubectl logs --previous` for the actual error, `kubectl describe pod` for OOMKilled/exit codes/events, verify config/secrets/env vars are correct, check resource limits aren't too low, and check liveness probe configuration isn't too aggressive (killing an otherwise-healthy but slow-starting app).

82. Q: How would you troubleshoot multiple pod crashes over the last month without monitoring tools?
    Ans: Check `kubectl get events --sort-by='.lastTimestamp'` and pod restart counts (`kubectl get pods`) for patterns, review any retained logs (if log rotation/retention allows going back that far, or centralized logging exists even without dedicated "monitoring" dashboards), check node-level system logs (`journalctl`, `dmesg`) for OOM/kernel-level events correlating with crash timestamps, and check deployment history/annotations for changes that align with when crashes started.

83. Q: From where would you obtain one-month-old troubleshooting information?
    Ans: Centralized log aggregation (if logs were shipped off-cluster to ELK/Loki/CloudWatch before pod logs were rotated away locally — Kubernetes itself typically only retains recent logs per pod), cloud provider audit/activity logs (CloudTrail), and any historical metrics retained in a time-series database (Prometheus with long retention, or a managed metrics service) if configured with sufficient retention.

84. Q: CPU is spiking in production pods — what will you do?
    Ans: Check if it's isolated to specific pods (application-level — bad code path, traffic spike, no autoscaling) or cluster-wide (node-level saturation, noisy-neighbor); check HPA is configured and functioning if load-driven; check for a recent deploy correlating with the spike; profile the application if isolated to specific pods and not resolved by scaling.

85. Q: If one pod shows a spike, how do you troubleshoot?
    Ans: Check if only one replica or all replicas of that workload show the spike (isolated instance issue vs. workload-wide issue); check for uneven load distribution (a Service/load balancer routing disproportionately to one pod); exec into the pod to profile the actual process; check recent events/restarts for that specific pod.

86. Q: What happens if one node becomes unavailable?
    Ans: The control plane detects the node is unreachable (missed heartbeats past the node's configured timeout), marks it `NotReady`, and — after a grace period — evicts/reschedules the pods that were running on it onto other healthy nodes (assuming sufficient capacity and no StatefulSet/local-storage constraints preventing rescheduling elsewhere).

## Upgrades & High Availability

87. Q: How do you upgrade an EKS cluster version?
    Ans: Upgrade the control plane first (one minor version at a time), then roll managed node groups onto the new version, then upgrade add-ons (CoreDNS, kube-proxy, VPC CNI) to compatible versions.

88. Q: How do you upgrade CoreDNS, kube-proxy and VPC CNI?
    Ans: Via EKS managed add-ons: `aws eks update-addon --cluster-name <name> --addon-name <coredns|kube-proxy|vpc-cni> --addon-version <version>`, checking AWS's version compatibility matrix for the target Kubernetes version first.

89. Q: How do you design a multi-AZ EKS cluster?
    Ans: Spread node groups across at least 3 AZs, use topology spread constraints/pod anti-affinity to distribute workload replicas across AZs, ensure subnets exist in each AZ, and rely on the EKS control plane's already-multi-AZ managed design.

90. Q: How do you ensure high availability in Kubernetes?
    Ans: Run multiple replicas of each workload spread across nodes/AZs (via anti-affinity/topology spread constraints), use PodDisruptionBudgets to protect minimum availability during voluntary disruptions, run a multi-AZ/multi-master control plane (or use a managed offering like EKS which handles this), and use readiness probes + rolling updates for safe deployments.

91. Q: What is the purpose of the PodDisruptionBudget?
    Ans: Limits how many pods of a given workload can be voluntarily disrupted at once (e.g., during a node drain for maintenance/upgrade), ensuring a minimum number/percentage of replicas remain available — protects against a cluster-admin action (not application crashes) taking down too much capacity at once.

92. Q: Your Keycloak deployment has 2 replicas. What happens if both pods are on the same node and that node fails?
    Ans: Both replicas go down simultaneously — a full outage until Kubernetes reschedules them onto healthy nodes (which takes time and requires available capacity). This is exactly what Pod Anti-Affinity (or topology spread constraints) should prevent by ensuring replicas of the same workload never land on the same node in the first place.

## Helm

93. Q: Why did you choose Helm over plain Kubernetes manifests?
    Ans: Helm packages related manifests into a single reusable, versioned unit (a chart) with templating (parameterize per environment via `values.yaml`), release tracking/history (easy rollback with `helm rollback`), and a large ecosystem of ready-made charts for common software — reducing the copy-paste/manual-templating burden of managing raw YAML across environments.

94. Q: What are Helm charts, and how do they simplify Kubernetes deployments?
    Ans: A Helm chart is a packaged collection of templated Kubernetes manifests plus metadata and default configuration — installing/upgrading an entire application (potentially many resources) becomes a single `helm install`/`upgrade` command with environment-specific values substituted in, instead of manually applying/tracking many separate YAML files.

95. Q: What are the main components of Helm?
    Ans: The **Helm CLI** (client, runs locally/in CI), **Charts** (the packaged template + config), and (in Helm 3+) no separate server component — Helm stores release state directly as Secrets in the target cluster, tracking installed releases and their history.

96. Q: What is a Helm Chart, and what does it contain?
    Ans: A directory structure containing `Chart.yaml` (metadata), `values.yaml` (default configuration), a `templates/` directory (Kubernetes manifest templates using Go templating), and optionally subcharts (`charts/` for dependencies) and helper templates (`_helpers.tpl`).

97. Q: What is a Chart.yaml file, and what is its purpose?
    Ans: The chart's metadata file — name, version, description, dependencies, maintainers, and the Kubernetes API version it targets — required for every Helm chart.

98. Q: What is a values.yaml file, and how does it work?
    Ans: The default configuration values file for a chart, referenced inside templates via `{{ .Values.xxx }}` — users override these defaults per-environment/deployment by supplying their own `values.yaml` (or `--set` flags) at install/upgrade time, without modifying the chart's templates themselves.

99. Q: How does Keycloak maintain sessions across multiple pods?
    Ans: By externalizing session state instead of keeping it in-memory per-pod — Keycloak uses its backing database (and/or a distributed cache like Infinispan configured in clustered/JGroups mode) to share authentication session data across all replicas, so a user's session remains valid regardless of which pod handles a given request.

## Hands-on Exercises

### Exercise 1: Create a Pod
**Objective:** Learn how to create a Pod.
**Steps:**
1. Choose a container image (e.g., nginx).
2. Create a Pod in the default namespace using that image.
3. Verify the Pod is running.

**Solution:**
```
kubectl run nginx --image=nginx --restart=Never
kubectl get pods
```

### Exercise 2: Expose a Pod with a Service
**Objective:** Learn how to create a Service for a Pod.
**Steps:**
1. Create a Pod running nginx with a label.
2. Create a Service that selects that label.
3. Verify the app is reachable through the Service.

**Solution:**
```
kubectl run nginx --image=nginx --restart=Never --port=80 --labels="app=dev-nginx"

cat << EOF > nginx-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: dev-nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
EOF

kubectl apply -f nginx-service.yaml
```

### Exercise 3: Taints and Tolerations
**Objective:** Understand how taints control Pod scheduling.
**Steps:**
1. Check if a node in your cluster already has taints.
2. Taint a node with key `app`, value `web`, effect `NoSchedule`.
3. Verify the taint was applied, and explain what it does.
4. Run a Pod that can still be scheduled on that tainted node by adding a matching toleration.

**Solution:**
```
kubectl describe no <node-name> | grep -i taints
kubectl taint node <node-name> app=web:NoSchedule
# Any Pod without a matching toleration for "app=web" will not be scheduled on this node

kubectl run some-pod --image=redis
kubectl edit po some-pod
```
Add under `spec.tolerations`:
```
tolerations:
  - key: app
    operator: Equal
    value: web
    effect: NoSchedule
```
Save and exit — the Pod should now be schedulable on the tainted node.
