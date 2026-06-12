# Kubernetes Interview Questions

## Basics
1. What is Kubernetes?
2. What is a Pod?
3. What is a Namespace?
4. What is a Service?
5. Types of Kubernetes Services?
6. Difference between Deployment and Service?

## Workload Controllers
7. Difference between Deployment, StatefulSet and DaemonSet?
8. Difference between ReplicaSet, Deployment and StatefulSet?
9. What is a Service Account?

## Probes
10. What is a Liveness Probe?
11. What is a Readiness Probe?
12. Difference between Liveness and Readiness probes?

## Scaling
13. How do you handle node autoscaling?
14. How do you scale microservices in Kubernetes?
15. What is HPA?

## Scheduling & Placement
16. What is Affinity?
17. What is Anti-Affinity?
18. What is Pod Anti-Affinity?
19. What are Taints and Tolerations?
20. How do taints and tolerations work?
21. Difference between Taints/Tolerations and Node Affinity?
22. How do you run specific microservices on specific nodes?

## Resource Management
23. What is a ResourceQuota?
24. Can we limit CPU and Memory at Namespace level?

## Networking
25. How does Pod-to-Pod communication work?
26. How do Pods communicate with external databases?
27. How do you block communication between pods in the same namespace?
28. How do you secure communication between microservices?
29. What is Kubernetes Network Policy?
30. On which resource is Network Policy applied?
31. How do you troubleshoot pod networking issues?
32. How do you troubleshoot DNS issues in Kubernetes?

## Deployment Strategies
33. What is Blue-Green deployment in Kubernetes?
34. What is Canary deployment?
35. What is Rolling deployment?
36. How do you perform zero-downtime deployments?
37. What are the types of deployment strategies?
38. What is the rollback strategy in Kubernetes?
39. Which type of deployment will you use for production and why?

## Security
40. How do you secure Kubernetes clusters?
41. How do you secure an EKS cluster?
42. What are Kubernetes security practices?
43. What is Kubernetes API Server use case?

## ArgoCD
44. What is an ArgoCD Project?
45. What are the steps to create an ArgoCD Project?
46. How does ArgoCD work?
47. What are ArgoCD CLI commands?
48. ArgoCD vs Flux — which would you choose and why?

## Ingress & Gateway
49. Is Ingress deprecated?
50. What is Gateway API?

## Monitoring & Observability
51. How do you monitor Kubernetes nodes and pods?
52. How do you configure Prometheus to scrape all namespaces?
53. How do you update a container image in a Deployment?

## Troubleshooting
54. How do you troubleshoot CrashLoopBackOff?
55. Pod stuck in ContainerCreating state — what will you check?
56. Pods continuously crashing — how do you troubleshoot?
57. CPU is spiking in production pods — what will you do?
58. If one pod shows a spike, how do you troubleshoot?
59. What happens if one node becomes unavailable?

## Upgrades & High Availability
60. How do you upgrade an EKS cluster version?
61. How do you upgrade CoreDNS, kube-proxy and VPC CNI?
62. How do you design a multi-AZ EKS cluster?
63. How do you ensure high availability in Kubernetes?
64. What is the purpose of the PodDisruptionBudget?
65. Your Keycloak deployment has 2 replicas. What happens if both pods are on the same node and that node fails?

## Helm
66. Why did you choose Helm over plain Kubernetes manifests?
67. How does the ALB know which pods are healthy?
68. How does Keycloak maintain sessions across multiple pods?
