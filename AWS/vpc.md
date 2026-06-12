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
19. NAT Gateway vs NAT Instance?
20. What is the difference between NAT Gateway and NAT Instance?
21. Why must a NAT Gateway be in a public subnet?

## Security
22. What is a Security Group?
23. What is NACL?
24. What is the difference between Security Group and NACL?
25. What happens if port 80 is blocked in NACL but allowed in Security Group?
26. What is VPC Flow Log?
27. What information does VPC Flow Log provide?

## Connectivity
28. What is VPC Peering?
29. What are the limitations of VPC Peering?
30. Difference between VPC Peering and Transit Gateway?
31. What is Transit Gateway?
32. Can we connect 2 VPCs and 1 datacenter together?
33. How do you connect an on-premises data center to AWS?
34. Difference between VPN and Direct Connect?

## VPC Endpoints
35. What is a VPC Endpoint?
36. What is a Gateway Endpoint?
37. What is an Interface Endpoint?
38. How do you provide private access to DynamoDB from a VPC?
39. What is AWS PrivateLink?

## Load Balancers
40. What is Elastic Load Balancing?
41. What are the types of Load Balancers?
42. Difference between ALB and NLB?
43. Difference between ALB and Classic Load Balancer?
44. Which load balancer provides static IPs?
45. What layer does ALB operate on?
46. What layer does NLB work on?
47. What is Path-Based Routing?
48. What is Host-Based Routing?
49. What are Target Groups?
50. What are Health Checks?
51. What is Cross-Zone Load Balancing?
52. What are Sticky Sessions?
53. How do you make private EC2 instances accessible from the internet?
54. What is SSL Termination?
55. Difference between CloudFront and ALB?
56. How do you design a VPC and network architecture?
57. How would you design network architecture for a multi-tier application?
