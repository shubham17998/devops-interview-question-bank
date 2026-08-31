# Kubernetes Interview Questions

## Basics

1. What is Kubernetes?
2. Explain Kubernetes Architecture.
3. What is a Pod?
4. What is a Namespace?
5. What is a Service?
6. Types of Kubernetes Services?
7. What is the difference between ClusterIP and NodePort?
8. Difference between Deployment and Service?
9. What is an Ingress in Kubernetes?
10. What is a Secret in Kubernetes?
11. What is a DaemonSet?
12. What is a Kubernetes Job?
13. What role does etcd play in Kubernetes? Can Kubernetes function without it?
14. What are the worker-node components, and what does each do?
15. What is imagePullPolicy?

## Workload Controllers

16. What is a ReplicaSet?
17. Difference between Deployment, StatefulSet and DaemonSet?
18. Difference between ReplicaSet, Deployment and StatefulSet?
19. How do you perform a Kubernetes Deployment?
20. What is a Service Account?
21. Can Kubernetes run in a single-node local environment?

## Storage

22. What are Persistent Volumes (PV) and Persistent Volume Claims (PVC)?

## Configuration Management

23. What is a ConfigMap?
24. When would you use a ConfigMap?
25. What is the difference between Secrets and ConfigMaps in Kubernetes?
26. How do you securely store credentials in Kubernetes?

## Probes

27. What is a Liveness Probe?
28. What is a Readiness Probe?
29. Difference between Liveness and Readiness probes?

## Scaling

30. How do you handle node autoscaling?
31. How do you scale microservices in Kubernetes?
32. What is HPA?
33. How does the Vertical Pod Autoscaler (VPA) work?
34. What is the difference between HPA and VPA?

## Scheduling & Placement

35. What is Affinity?
36. What is Anti-Affinity?
37. What is Pod Anti-Affinity?
38. What are Taints and Tolerations?
39. How do taints and tolerations work?
40. Difference between Taints/Tolerations and Node Affinity?
41. How do you run specific microservices on specific nodes?

## Resource Management

42. What is a ResourceQuota?
43. Can we limit CPU and Memory at Namespace level?

## Networking

44. How does Pod-to-Pod communication work?
45. How do Pods communicate with external databases?
46. How do you block communication between pods in the same namespace?
47. How do you secure communication between microservices?
48. What is Kubernetes Network Policy?
49. On which resource is Network Policy applied?
50. How does a Service know which Pods to route traffic to?
51. How do you troubleshoot pod networking issues?
52. How do you troubleshoot DNS issues in Kubernetes?

## Deployment Strategies

53. What is Blue-Green deployment in Kubernetes?
54. What is Canary deployment?
55. What is Rolling deployment?
56. How do you perform zero-downtime deployments?
57. What are the types of deployment strategies?
58. What is the rollback strategy in Kubernetes?
59. Which type of deployment will you use for production and why?

## Security

60. How do you secure Kubernetes clusters?
61. How do you secure an EKS cluster?
62. What are Kubernetes security practices?
63. What is Kubernetes API Server use case?
64. What is Role-Based Access Control (RBAC) in Kubernetes?
65. What is a Custom Resource Definition (CRD)?

## ArgoCD

66. What is an ArgoCD Project?
67. What are the steps to create an ArgoCD Project?
68. How does ArgoCD work?
69. What are ArgoCD CLI commands?
70. ArgoCD vs Flux — which would you choose and why?

## Ingress & Gateway

71. What is an Ingress Controller?
72. Is Ingress deprecated?
73. What is Gateway API?
74. Explain the ALB Ingress Controller workflow.
75. How does the ALB know which pods are healthy?

## Monitoring & Observability

76. How do you monitor Kubernetes nodes and pods?
77. How do you configure Prometheus to scrape all namespaces?
78. How do you update a container image in a Deployment?

## Troubleshooting

79. How do you troubleshoot CrashLoopBackOff?
80. Pod stuck in ContainerCreating state — what will you check?
81. Pods continuously crashing — how do you troubleshoot?
82. How would you troubleshoot multiple pod crashes over the last month without monitoring tools?
83. From where would you obtain one-month-old troubleshooting information?
84. CPU is spiking in production pods — what will you do?
85. If one pod shows a spike, how do you troubleshoot?
86. What happens if one node becomes unavailable?

## Upgrades & High Availability

87. How do you upgrade an EKS cluster version?
88. How do you upgrade CoreDNS, kube-proxy and VPC CNI?
89. How do you design a multi-AZ EKS cluster?
90. How do you ensure high availability in Kubernetes?
91. What is the purpose of the PodDisruptionBudget?
92. Your Keycloak deployment has 2 replicas. What happens if both pods are on the same node and that node fails?

## Helm

93. Why did you choose Helm over plain Kubernetes manifests?
94. What are Helm charts, and how do they simplify Kubernetes deployments?
95. What are the main components of Helm?
96. What is a Helm Chart, and what does it contain?
97. What is a Chart.yaml file, and what is its purpose?
98. What is a values.yaml file, and how does it work?
99. How does Keycloak maintain sessions across multiple pods?

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
