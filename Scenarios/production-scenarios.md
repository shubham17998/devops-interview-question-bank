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
14. Kubernetes pods are Running but users receive 503 errors — what will you check?
15. Your database performs well initially, but after a certain month you notice slowness — how do you troubleshoot and fix it?

## Disaster Recovery & SRE
16. What is RTO?
17. What is RPO?
18. How do you handle production incidents?
19. What is your incident response process?
20. How do you perform rollback during a failed deployment?
21. How do you design for high availability?
22. How do you ensure disaster recovery readiness?
23. How would you build a disaster recovery solution across regions?
24. A Terraform script bypasses validation and deletes a production database during peak business hours. What would be your action plan to contain the blast radius?

## Architecture Design
25. How would you handle sudden traffic spikes?
26. How would you migrate a Windows file server to AWS?
27. How would you secure a production AWS environment?
28. How would you optimize AWS costs?
29. How would you design a global application with low latency?
30. How would you build a serverless application?
31. How would you decouple microservices in AWS?
32. How would you build a secure static website on AWS?
33. How would you implement a CI/CD pipeline on AWS?
34. How would you connect multiple AWS accounts securely?
35. How would you monitor an enterprise AWS environment?
36. How would you design a secure and highly available 3-tier infrastructure using AWS services?
37. Given two front-end services and 15 back-end services in a GitLab repository, how would you deploy them in Kubernetes?
38. Explain the complete request flow from a browser to a backend Kubernetes pod.
39. How would you ensure even Pod distribution across Nodes in a Kubernetes cluster (e.g., 6 Pods across 3 Nodes, 2 per Node)?
40. A Kubernetes Node has reached its Pod capacity. What would you check and do?

## EKS / Kubernetes Deep Dives
41. Walk me through your architecture. Why did you create two separate projects?
42. Why did you choose EKS over self-managed Kubernetes?
43. Explain IRSA. Why not just give the EC2 node an IAM role?
44. Why External Secrets Operator instead of putting secrets directly in Kubernetes Secrets?
45. Explain your blue-green strategy. How does the traffic switch actually happen?
46. What happens if the PostgreSQL pod crashes and loses data?
47. How do you handle secrets rotation?
48. Your NAT Gateway per AZ is expensive. How would you justify it?
49. If this was a real production system for a healthcare client what would you change?
50. How would you make this multi-tenant for multiple clients?
51. What is missing from your current architecture?
52. How do you ensure HIPAA compliance on this infrastructure?
53. Someone on your team accidentally committed an AWS secret key to GitHub. What do you do?
54. How would you implement least privilege for your EKS nodes?
55. How does your system handle a traffic spike say 10x normal load?
56. What is your strategy for zero downtime EKS cluster upgrades?
57. Your Keycloak deployment has 2 replicas. What happens if both pods are on the same node and that node fails?

## CI/CD Pipeline
58. How would you add a CI/CD pipeline to this project?
59. ArgoCD vs Flux which would you choose and why?
60. How do you handle a hotfix that needs to go to production immediately?
61. What if the CI pipeline itself goes down?
62. How long does your full pipeline take end to end?
63. How do you handle database migrations in the pipeline?
64. What is CICD architecture in production?

## Cost Optimization
65. How would you optimize the AWS cost of this infrastructure?
66. What are VPC Endpoints and why would you add them?
67. Blue-green costs 2x resources. How do you justify this to management?
68. Why gp3 over gp2 for storage?

## Deployment Strategy Deep Dives
69. What if the blue-green switch causes database schema issues?
70. How do you decide when to promote canary to 100%?
71. Have you used Shadow deployment?
72. Which ONE deployment strategy would you recommend for our projects?
73. What if the network team changes a subnet and recreates it. Does Project 2 break?
74. How does Project 2 authenticate to read the S3 state?
75. Can Project 2 accidentally modify Project 1 resources?
76. What if you need to pass a value that is not in Project 1 outputs?
77. How have you used the network terraform project inside the kubernetes terraform project?

## Debugging Real Systems
78. Production Keycloak is down. Walk me through how you would debug it?
79. How would you handle a Terraform state corruption?
80. The team wants to deploy to multiple AWS accounts. How would you restructure this?

## Behavioral / Leadership
81. Introduce yourself and walk through your DevOps experience.
82. Tell me about a time you had a production incident. How did you handle it?
83. Describe a challenge you faced as a DevOps engineer and how you overcame it.
84. Would you prefer to work individually or as part of a team?
85. Tell me about a time you learned and implemented something from scratch.
86. How would you handle a situation where you're not getting help from your team members?
87. Name 5 AWS services you have used, and what are their use cases?
88. How do you keep up with DevOps and cloud technologies?
89. Where do you see yourself in 3 years?
90. How do you handle a junior engineer who keeps breaking production?
91. How do you handle disagreement with a senior engineer on architecture?
92. How do you keep your team infrastructure knowledge up to date?
93. Why should we hire you?
94. How long did it take to resolve the incident?
95. How many users were impacted?
96. Did you do a post-mortem after the incident?
97. What would you do differently after that incident?
98. Was the issue caught in staging first?
99. What is your biggest achievement?
