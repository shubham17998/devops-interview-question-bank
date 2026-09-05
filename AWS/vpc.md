# VPC Interview Questions

## Basics
1. What is a VPC?
   Virtual Private Cloud — an isolated, logically-separated virtual network within AWS where you define your own IP address range, subnets, route tables, and gateways, giving you full control over networking for your resources.

2. How many VPCs can be created per region by default?
   5 per region by default (a soft limit — increasable via a service quota increase request).

3. How many subnets can be created in a VPC?
   200 per VPC by default (also a soft, increasable limit).

4. What is CIDR?
   Classless Inter-Domain Routing — notation (e.g., `10.0.0.0/16`) specifying an IP range by a base address and a prefix length indicating how many leading bits are fixed (network) versus variable (host).

5. How do you calculate CIDR ranges?
   The prefix length (`/n`) determines the number of host addresses: `2^(32-n)` total addresses for IPv4. E.g., `/24` = 2^8 = 256 addresses, `/16` = 2^16 = 65,536 addresses. Subnetting a VPC's CIDR into smaller subnet CIDRs means increasing the prefix length (borrowing host bits for additional network bits) — e.g., splitting a `/16` into four `/18`s.

6. Why does AWS reserve 5 IP addresses in every subnet?
   In every subnet, AWS reserves: the network address (first IP), the VPC router address (second IP), the DNS server address (third IP), a future-use reserved address (fourth IP), and the broadcast address (last IP) — even though AWS VPCs don't support broadcast, it's still reserved for consistency.

## Subnets
7. What is a Public Subnet?
   A subnet whose route table has a route to an Internet Gateway, allowing its resources (with a public IP) to send/receive traffic directly to/from the internet.

8. What is a Private Subnet?
   A subnet with no direct route to an Internet Gateway — its resources have no direct internet exposure and (if outbound internet access is needed) rely on a NAT Gateway/Instance in a public subnet instead.

9. How do you configure a Public Subnet?
   Create the subnet within a VPC, attach an Internet Gateway to the VPC, add a route in the subnet's route table (`0.0.0.0/0 → igw-xxxx`), and enable auto-assign public IP on the subnet (or assign Elastic IPs to instances) so they get a routable public address.

10. How do you configure a private subnet?
    Create the subnet, and do NOT route `0.0.0.0/0` to an Internet Gateway in its route table. If outbound internet access is still needed, route `0.0.0.0/0` to a NAT Gateway located in a public subnet instead.

11. Where do we use private subnets?
    For resources that shouldn't be directly internet-reachable — application/database tiers, internal services, worker nodes — anything that should only be reached via an internal load balancer, bastion, or through the app tier itself.

12. Can a private subnet receive inbound traffic?
    Not directly from the internet (no route from an IGW). It can receive traffic from within the VPC (e.g., from a public-subnet load balancer or bastion), from peered VPCs, from on-premises via VPN/Direct Connect, or return traffic for connections it initiated outbound (e.g., via NAT Gateway, which is stateful).

## Gateways & Routing
13. What is a Route Table?
    A set of rules (routes) associated with a subnet (or the VPC's main route table by default) that determines where network traffic from that subnet is directed, based on destination CIDR.

14. What is an Internet Gateway?
    A horizontally-scaled, highly available VPC component that enables communication between resources in the VPC and the public internet — it performs 1:1 NAT for instances with public IPs and is the target for `0.0.0.0/0` routes in public subnets.

15. How do you configure an Internet Gateway?
    Create the IGW, attach it to the VPC, then add a route in the relevant subnet's route table pointing `0.0.0.0/0` to the IGW.

16. What is a NAT Gateway?
    A managed AWS service that lets instances in a private subnet initiate outbound connections to the internet (or other VPCs), while remaining unreachable for inbound connections initiated from the internet — a managed, highly-available implementation of NAT.

17. How do you configure a NAT Gateway?
    Create a NAT Gateway in a public subnet, assign it an Elastic IP, then add a route in the private subnet's route table pointing `0.0.0.0/0` to the NAT Gateway's ID.

18. Difference between NAT Gateway and Internet Gateway?
    An Internet Gateway allows bidirectional internet access for resources with public IPs. A NAT Gateway allows only outbound-initiated internet access for private-subnet resources (no public IP needed on the instance) — inbound connections from the internet cannot reach the instance through it.

19. What is the difference between NAT Gateway, NAT Instance, Egress-Only Internet Gateway, and Internet Gateway?
    **NAT Gateway** — AWS-managed, highly available, scales automatically, IPv4 only. **NAT Instance** — a self-managed EC2 instance running NAT software, cheaper but you manage patching/scaling/HA yourself. **Egress-Only Internet Gateway** — the IPv6 equivalent of a NAT Gateway (stateful, outbound-only for IPv6, since IPv6 doesn't use NAT). **Internet Gateway** — full bidirectional internet access, used for public subnets.

20. Why must a NAT Gateway be in a public subnet?
    Because the NAT Gateway itself needs a route to the Internet Gateway to actually forward the private subnet's outbound traffic onto the internet — it needs an Elastic IP and internet reachability to perform that translation, which only a public subnet's routing provides.

## Security
21. What is a Security Group?
    A stateful, instance/ENI-level virtual firewall — you define allow rules only (no explicit deny); return traffic for an allowed connection is automatically permitted regardless of outbound rules.

22. What is NACL?
    Network Access Control List — a stateless, subnet-level firewall that evaluates numbered rules in order and supports both Allow and Deny rules; because it's stateless, return traffic must be explicitly allowed by a corresponding rule.

23. What is the difference between Security Group and NACL?
    | | Security Group | NACL |
    |---|---|---|
    | Level | Instance/ENI | Subnet |
    | State | Stateful (return traffic auto-allowed) | Stateless (must allow both directions) |
    | Rules | Allow only | Allow and Deny |
    | Evaluation | All rules evaluated | Rules evaluated in numbered order, first match wins |

24. What happens if port 80 is blocked in NACL but allowed in Security Group?
    Traffic is blocked. NACLs are evaluated first at the subnet boundary (both inbound and outbound must pass), so a Deny at the NACL level stops the traffic before it ever reaches the instance's Security Group evaluation — both layers must allow the traffic for it to succeed.

25. What is the difference between a stateful and a stateless firewall?
    A **stateful** firewall tracks connection state and automatically allows return traffic for an established/allowed connection without a separate rule (e.g., Security Groups). A **stateless** firewall evaluates every packet independently against rules with no memory of prior packets, so both the request and its response must be explicitly permitted (e.g., NACLs).

26. What is VPC Flow Log?
    A feature that captures IP traffic metadata (not payload) flowing to/from network interfaces in a VPC, subnet, or ENI, and delivers it to CloudWatch Logs or S3 for network monitoring, troubleshooting, and security analysis.

27. What information does VPC Flow Log provide?
    Source/destination IP and port, protocol, packet/byte counts, action (ACCEPT/REJECT), start/end time, and the interface/ENI involved — enough to answer "who talked to whom, on what port, and was it allowed."

## Connectivity
28. What is VPC Peering?
    A direct, private network connection between two VPCs (in the same or different accounts/regions) that lets resources communicate using private IPs as if on the same network, with traffic routed over AWS's backbone rather than the public internet.

29. What are the limitations of VPC Peering?
    No transitive peering (see Q30), no overlapping CIDR blocks between peered VPCs, a connection must be explicitly added to route tables on both sides, and it doesn't scale well for many-VPC topologies (requires a full mesh of individual peering connections) — which is exactly what Transit Gateway solves.

30. If VPC A is peered with VPC B, and VPC B is peered with VPC C, can A communicate with C?
    No. VPC Peering is not transitive — A would need its own direct peering connection to C.

31. Difference between VPC Peering and Transit Gateway?
    VPC Peering is a point-to-point, non-transitive connection between exactly two VPCs — it doesn't scale well past a handful of VPCs. Transit Gateway is a central hub that many VPCs (and on-prem VPNs/Direct Connect) attach to, enabling transitive routing between all attached networks through one place, much more manageable at scale.

32. What is Transit Gateway?
    A regional network transit hub that connects multiple VPCs, VPNs, and Direct Connect connections through a single gateway, using route tables to control which attached networks can reach each other — replacing complex peering meshes.

33. Can we connect 2 VPCs and 1 datacenter together?
    Yes — attach both VPCs and a Site-to-Site VPN (or Direct Connect) connection to the on-prem datacenter to a Transit Gateway, which then routes traffic between all three based on its route tables.

34. How do you connect an on-premises data center to AWS?
    Via a Site-to-Site VPN (IPsec tunnel over the internet, quick to set up) or AWS Direct Connect (a dedicated private network connection, lower latency/more consistent bandwidth, longer to provision) — often both together for DX as primary with VPN as failover.

35. Difference between VPN and Direct Connect?
    **VPN** is encrypted, runs over the public internet, quick/cheap to set up, but variable latency/bandwidth. **Direct Connect** is a dedicated physical network link to AWS, offering consistent low latency and higher bandwidth, but requires physical provisioning (longer lead time) and costs more.

## VPC Endpoints
36. What is a VPC Endpoint?
    A way to privately connect a VPC to supported AWS services without traversing the public internet or requiring a NAT Gateway/IGW — traffic stays entirely within the AWS network.

37. What is a Gateway Endpoint?
    A VPC Endpoint type (only for S3 and DynamoDB) implemented as a route table target — you add a route to the endpoint's prefix list, and traffic to that service is routed privately without needing an ENI.

38. What is an Interface Endpoint?
    A VPC Endpoint implemented as an ENI with a private IP in your subnet (powered by AWS PrivateLink), used for most other AWS services (and third-party/custom services) — the service is reached by resolving its private DNS name to that ENI's IP.

39. How do you provide private access to DynamoDB from a VPC?
    Create a **Gateway Endpoint** for DynamoDB and add a route to it in the relevant subnets' route tables — traffic to DynamoDB then stays on the AWS network without needing a NAT Gateway or internet access at all.

40. What is AWS PrivateLink?
    The underlying technology behind Interface Endpoints — it lets you privately expose a service (your own, or a third-party's) to consumers' VPCs via ENIs, without exposing it to the public internet, using their own VPC's private IP space.

## Load Balancers
41. What is Elastic Load Balancing?
    AWS's managed load balancing service (ALB, NLB, and the legacy CLB) that automatically distributes incoming traffic across multiple targets (EC2, containers, IPs, Lambda) in one or more AZs, with built-in health checking and scaling.

42. What are the types of Load Balancers?
    Application Load Balancer (Layer 7, HTTP/HTTPS), Network Load Balancer (Layer 4, TCP/UDP/TLS, extreme performance and static IPs), and Classic Load Balancer (legacy, Layer 4/7 hybrid, deprecated for new use).

43. Difference between ALB and NLB?
    ALB operates at Layer 7 — routes based on HTTP content (host/path/headers), supports SSL termination, WAF integration, path/host-based routing. NLB operates at Layer 4 — routes based on IP/port only, ultra-low latency, millions of requests/sec, supports static/Elastic IPs, preserves source IP.

44. Difference between ALB and Classic Load Balancer?
    ALB supports Layer 7 features (host/path-based routing, native container/Lambda targets, WebSockets, HTTP/2) that CLB lacks; CLB is a legacy load balancer operating at both Layer 4 and (limited) Layer 7, generally being phased out in favor of ALB/NLB.

45. Which load balancer provides static IPs?
    Network Load Balancer (and Classic Load Balancer in EC2-Classic, historically) — ALB does not have static IPs by default (though it can sit behind a Global Accelerator or an NLB for a static-IP frontend if truly needed).

46. What layer does ALB operate on?
    Layer 7 (Application layer — HTTP/HTTPS).

47. What layer does NLB work on?
    Layer 4 (Transport layer — TCP/UDP/TLS).

48. What is Path-Based Routing?
    An ALB routing rule that sends requests to different target groups based on the URL path (e.g., `/api/*` → API target group, `/images/*` → static-content target group), all under the same domain/listener.

49. What is Host-Based Routing?
    An ALB routing rule that sends requests to different target groups based on the `Host` header (e.g., `api.example.com` vs `app.example.com`), letting one ALB serve multiple domains/subdomains with different backends.

50. What are Target Groups?
    A grouping of registered targets (EC2 instances, IPs, Lambda functions, or ECS tasks) that a load balancer routes requests to, each with its own health check configuration and routing rules pointing to it.

51. What are Health Checks?
    Periodic probes (HTTP, HTTPS, or TCP) a load balancer sends to registered targets to determine if they're healthy; unhealthy targets are automatically removed from active rotation until they pass health checks again.

52. What is Cross-Zone Load Balancing?
    A setting that lets a load balancer distribute traffic evenly across targets in **all** enabled AZs, rather than only distributing evenly within each AZ's own targets — prevents uneven load when AZs have different numbers of healthy targets.

53. What are Sticky Sessions?
    A load balancer feature (session affinity) that routes all requests from a given client to the same backend target for the duration of a session (via a cookie), useful for stateful applications that store session data locally on one instance.

54. How do you make private EC2 instances accessible from the internet?
    Place a load balancer (ALB/NLB) in public subnets in front of them — the LB is internet-facing and forwards to the private instances' target group, so the instances themselves never need public IPs or direct internet routes.

55. What is SSL Termination?
    Decrypting incoming HTTPS/TLS traffic at a specific point (e.g., the load balancer or CloudFront) rather than at the backend server, offloading the CPU cost of encryption/decryption from application servers and centralizing certificate management.

56. Difference between CloudFront and ALB?
    CloudFront is a global CDN operating at edge locations worldwide, focused on caching and latency reduction. ALB is a regional Layer 7 load balancer distributing traffic to backend targets within a region — commonly used together, CloudFront in front of an ALB.

57. How do you design a VPC and network architecture?
    Choose a non-overlapping CIDR range sized for growth; create public subnets (for load balancers/NAT gateways) and private subnets (for app/data tiers) spread across at least 2-3 AZs for HA; route public subnets to an IGW and private subnets to per-AZ NAT Gateways; use Security Groups for least-privilege instance-level rules and NACLs as a coarse subnet-level backstop; add VPC Endpoints for AWS services accessed privately; and connect to on-prem/other VPCs via Transit Gateway/VPN/Direct Connect as needed.

58. How would you design network architecture for a multi-tier application?
    Public subnets host only the internet-facing ALB/NAT Gateways; an application-tier private subnet holds app servers/containers, reachable only from the ALB; a database-tier private subnet (often with even tighter Security Group rules, allowing only the app tier) holds RDS/databases with no route to the internet at all; each tier spans multiple AZs for HA, with Security Groups scoped tier-to-tier (ALB→app on app port, app→db on db port only) rather than broad CIDR-based rules.

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
