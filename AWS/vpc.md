# VPC Interview Questions

## Basics
1. What is a VPC?
2. How many VPCs can be created per region by default?
3. How many subnets can be created in a VPC?
4. What is CIDR?
5. How do you calculate CIDR ranges?
6. Why does AWS reserve 5 IP addresses in every subnet?

## Subnets
7. What is a Public Subnet?
8. What is a Private Subnet?
9. How do you configure a Public Subnet?
10. How do you configure a private subnet?
11. Where do we use private subnets?
12. Can a private subnet receive inbound traffic?

## Gateways & Routing
13. What is a Route Table?
14. What is an Internet Gateway?
15. How do you configure an Internet Gateway?
16. What is a NAT Gateway?
17. How do you configure a NAT Gateway?
18. Difference between NAT Gateway and Internet Gateway?
19. What is the difference between NAT Gateway, NAT Instance, Egress-Only Internet Gateway, and Internet Gateway?
20. Why must a NAT Gateway be in a public subnet?

## Security
21. What is a Security Group?
22. What is NACL?
23. What is the difference between Security Group and NACL?
24. What happens if port 80 is blocked in NACL but allowed in Security Group?
25. What is the difference between a stateful and a stateless firewall?
26. What is VPC Flow Log?
27. What information does VPC Flow Log provide?

## Connectivity
28. What is VPC Peering?
29. What are the limitations of VPC Peering?
30. If VPC A is peered with VPC B, and VPC B is peered with VPC C, can A communicate with C?
31. Difference between VPC Peering and Transit Gateway?
32. What is Transit Gateway?
33. Can we connect 2 VPCs and 1 datacenter together?
34. How do you connect an on-premises data center to AWS?
35. Difference between VPN and Direct Connect?

## VPC Endpoints
36. What is a VPC Endpoint?
37. What is a Gateway Endpoint?
38. What is an Interface Endpoint?
39. How do you provide private access to DynamoDB from a VPC?
40. What is AWS PrivateLink?

## Load Balancers
41. What is Elastic Load Balancing?
42. What are the types of Load Balancers?
43. Difference between ALB and NLB?
44. Difference between ALB and Classic Load Balancer?
45. Which load balancer provides static IPs?
46. What layer does ALB operate on?
47. What layer does NLB work on?
48. What is Path-Based Routing?
49. What is Host-Based Routing?
50. What are Target Groups?
51. What are Health Checks?
52. What is Cross-Zone Load Balancing?
53. What are Sticky Sessions?
54. How do you make private EC2 instances accessible from the internet?
55. What is SSL Termination?
56. Difference between CloudFront and ALB?
57. How do you design a VPC and network architecture?
58. How would you design network architecture for a multi-tier application?

## Hands-on Exercises

### Exercise: Security Group Inbound Rules
**Objective:** Understand how Security Group rules control reachability.
**Requirements:** An EC2 instance running a web app, with a Security Group allowing inbound HTTP.
**Steps:**
1. List the Security Groups in your account/region.
2. Remove the HTTP inbound rule.
3. Try to access the app — what happens?
4. Add the rule back.
5. Try again — can you access it now?

**Solution (CLI):**
```
aws ec2 describe-security-groups

aws ec2 revoke-security-group-ingress \
    --group-name someHTTPSecurityGroup \
    --protocol tcp --port 80 --cidr 0.0.0.0/0
# App is now unreachable — the request times out with no HTTP rule allowing it in

aws ec2 authorize-security-group-ingress \
    --group-name someHTTPSecurityGroup \
    --protocol tcp --port 80 --cidr 0.0.0.0/0
# App is reachable again
```
