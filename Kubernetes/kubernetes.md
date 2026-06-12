# Kubernetes Interview Questions

## Basics

1. What is Kubernetes?
2. Explain Kubernetes Architecture.
3. What is a Pod?
4. What is a Namespace?
5. What is a Service?
6. Types of Kubernetes Services?
7. Difference between Deployment and Service?

## Workload Controllers

8. What is a ReplicaSet?
9. Difference between Deployment, StatefulSet and DaemonSet?
10. Difference between ReplicaSet, Deployment and StatefulSet?
11. How do you perform a Kubernetes Deployment?
12. What is a Service Account?

## Configuration Management

13. What is a ConfigMap?
14. When would you use a ConfigMap?

## Probes

15. What is a Liveness Probe?
16. What is a Readiness Probe?
17. Difference between Liveness and Readiness probes?

## Scaling

18. How do you handle node autoscaling?
19. How do you scale microservices in Kubernetes?
20. What is HPA?

## Scheduling & Placement

21. What is Affinity?
22. What is Anti-Affinity?
23. What is Pod Anti-Affinity?
24. What are Taints and Tolerations?
25. How do taints and tolerations work?
26. Difference between Taints/Tolerations and Node Affinity?
27. How do you run specific microservices on specific nodes?

## Resource Management

28. What is a ResourceQuota?
29. Can we limit CPU and Memory at Namespace level?

## Networking

30. How does Pod-to-Pod communication work?
31. How do Pods communicate with external databases?
32. How do you block communication between pods in the same namespace?
33. How do you secure communication between microservices?
34. What is Kubernetes Network Policy?
35. On which resource is Network Policy applied?
36. How do you troubleshoot pod networking issues?
37. How do you troubleshoot DNS issues in Kubernetes?

## Deployment Strategies

38. What is Blue-Green deployment in Kubernetes?
39. What is Canary deployment?
40. What is Rolling deployment?
41. How do you perform zero-downtime deployments?
42. What are the types of deployment strategies?
43. What is the rollback strategy in Kubernetes?
44. Which type of deployment will you use for production and why?

## Security

45. How do you secure Kubernetes clusters?
46. How do you secure an EKS cluster?
47. What are Kubernetes security practices?
48. What is Kubernetes API Server use case?

## ArgoCD

49. What is an ArgoCD Project?
50. What are the steps to create an ArgoCD Project?
51. How does ArgoCD work?
52. What are ArgoCD CLI commands?
53. ArgoCD vs Flux — which would you choose and why?

## Ingress & Gateway

54. What is an Ingress Controller?
55. Is Ingress deprecated?
56. What is Gateway API?
57. Explain the ALB Ingress Controller workflow.
58. How does the ALB know which pods are healthy?

## Monitoring & Observability

59. How do you monitor Kubernetes nodes and pods?
60. How do you configure Prometheus to scrape all namespaces?
61. How do you update a container image in a Deployment?

## Troubleshooting

62. How do you troubleshoot CrashLoopBackOff?
63. Pod stuck in ContainerCreating state — what will you check?
64. Pods continuously crashing — how do you troubleshoot?
65. How would you troubleshoot multiple pod crashes over the last month without monitoring tools?
66. From where would you obtain one-month-old troubleshooting information?
67. CPU is spiking in production pods — what will you do?
68. If one pod shows a spike, how do you troubleshoot?
69. What happens if one node becomes unavailable?

## Upgrades & High Availability

70. How do you upgrade an EKS cluster version?
71. How do you upgrade CoreDNS, kube-proxy and VPC CNI?
72. How do you design a multi-AZ EKS cluster?
73. How do you ensure high availability in Kubernetes?
74. What is the purpose of the PodDisruptionBudget?
75. Your Keycloak deployment has 2 replicas. What happens if both pods are on the same node and that node fails?

## Helm

76. Why did you choose Helm over plain Kubernetes manifests?
77. How does Keycloak maintain sessions across multiple pods?
