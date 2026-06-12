# Production Scenarios Interview Questions

## Incident Troubleshooting
1. Application down but EC2 healthy — what will you check?
2. Container running but application not accessible — what will you check?
3. How do you troubleshoot a 502 Bad Gateway?
4. High latency in production — what will you check?
5. Deployment caused outage — what will you do?
6. Application works in Dev but not in Prod — how do you troubleshoot?
7. How do you handle a production outage?
8. How do you perform RCA?
9. How do you debug a production DNS issue?
10. How do you identify whether the issue is application, service, cluster or infrastructure related?
11. How do you troubleshoot intermittent microservice failures?
12. One AWS region goes down — how do you handle failover?
13. CI/CD deployment failed in production — what steps do you take?

## Disaster Recovery & SRE
14. What is RTO?
15. What is RPO?
16. How do you handle production incidents?
17. What is your incident response process?
18. How do you perform rollback during a failed deployment?
19. How do you design for high availability?
20. How do you ensure disaster recovery readiness?
21. How would you build a disaster recovery solution across regions?

## Architecture Design
22. How would you handle sudden traffic spikes?
23. How would you migrate a Windows file server to AWS?
24. How would you secure a production AWS environment?
25. How would you optimize AWS costs?
26. How would you design a global application with low latency?
27. How would you build a serverless application?
28. How would you decouple microservices in AWS?
29. How would you build a secure static website on AWS?
30. How would you implement a CI/CD pipeline on AWS?
31. How would you connect multiple AWS accounts securely?
32. How would you monitor an enterprise AWS environment?

## EKS / Kubernetes Deep Dives
33. Walk me through your architecture. Why did you create two separate projects?
34. Why did you choose EKS over self-managed Kubernetes?
35. Explain IRSA. Why not just give the EC2 node an IAM role?
36. Why External Secrets Operator instead of putting secrets directly in Kubernetes Secrets?
37. Explain your blue-green strategy. How does the traffic switch actually happen?
38. What happens if the PostgreSQL pod crashes and loses data?
39. How do you handle secrets rotation?
40. Your NAT Gateway per AZ is expensive. How would you justify it?
41. If this was a real production system for a healthcare client what would you change?
42. How would you make this multi-tenant for multiple clients?
43. What is missing from your current architecture?
44. How do you ensure HIPAA compliance on this infrastructure?
45. Someone on your team accidentally committed an AWS secret key to GitHub. What do you do?
46. How would you implement least privilege for your EKS nodes?
47. How does your system handle a traffic spike say 10x normal load?
48. What is your strategy for zero downtime EKS cluster upgrades?
49. Your Keycloak deployment has 2 replicas. What happens if both pods are on the same node and that node fails?

## CI/CD Pipeline
50. How would you add a CI/CD pipeline to this project?
51. ArgoCD vs Flux which would you choose and why?
52. How do you handle a hotfix that needs to go to production immediately?
53. What if the CI pipeline itself goes down?
54. How long does your full pipeline take end to end?
55. How do you handle database migrations in the pipeline?
56. What is CICD architecture in production?

## Cost Optimization
57. How would you optimize the AWS cost of this infrastructure?
58. What are VPC Endpoints and why would you add them?
59. Blue-green costs 2x resources. How do you justify this to management?
60. Why gp3 over gp2 for storage?

## Deployment Strategy Deep Dives
61. What if the blue-green switch causes database schema issues?
62. How do you decide when to promote canary to 100%?
63. Have you used Shadow deployment?
64. Which ONE deployment strategy would you recommend for our projects?
65. What if the network team changes a subnet and recreates it. Does Project 2 break?
66. How does Project 2 authenticate to read the S3 state?
67. Can Project 2 accidentally modify Project 1 resources?
68. What if you need to pass a value that is not in Project 1 outputs?
69. How have you used the network terraform project inside the kubernetes terraform project?

## Debugging Real Systems
70. Production Keycloak is down. Walk me through how you would debug it?
71. How would you handle a Terraform state corruption?
72. The team wants to deploy to multiple AWS accounts. How would you restructure this?

## Behavioral / Leadership
73. Tell me about a time you had a production incident. How did you handle it?
74. How do you keep up with DevOps and cloud technologies?
75. Where do you see yourself in 3 years?
76. How do you handle a junior engineer who keeps breaking production?
77. How do you handle disagreement with a senior engineer on architecture?
78. How do you keep your team infrastructure knowledge up to date?
79. Why should we hire you?
80. How long did it take to resolve the incident?
81. How many users were impacted?
82. Did you do a post-mortem after the incident?
83. What would you do differently after that incident?
84. Was the issue caught in staging first?
