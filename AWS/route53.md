# Route53 Interview Questions

## Basics
Q: What is Route53?\
Ans: Route53 is AWS's managed DNS service. It handles domain registration, DNS resolution (translating domain names to IP addresses/endpoints), health checking, and traffic routing (weighted, latency-based, failover, geolocation, etc.).

Q: What is a Hosted Zone?\
Ans: A Hosted Zone is a container for DNS records for a specific domain (e.g., `example.com`). A **public** hosted zone answers queries from the public internet; a **private** hosted zone answers queries only from one or more VPCs.

Q: What DNS records are available in Route53?\
Ans: A, AAAA, CNAME, MX, TXT, NS, SOA, SRV, PTR, CAA, NAPTR, and the Route53-specific **Alias** record.

## DNS Record Types
Q: What is an A record?\
Ans: Maps a domain/subdomain name directly to an IPv4 address (e.g., `app.example.com` → `203.0.113.10`).

Q: What is a CNAME record?\
Ans: Maps a domain/subdomain name to another domain name (an alias to a hostname), not directly to an IP. CNAMEs cannot be used at the zone apex (root domain).

Q: What is a TXT record?\
Ans: Stores arbitrary text data associated with a domain — commonly used for domain ownership verification, SPF/DKIM/DMARC email authentication records, and other machine-readable metadata.

Q: What is an Alias record?\
Ans: A Route53-specific record type that maps a name to an AWS resource (ALB, CloudFront distribution, S3 website endpoint, another Route53 record, etc.) using the target's underlying IPs, which AWS updates automatically. Unlike CNAME, Alias records can be used at the zone apex and Route53 doesn't charge for Alias queries to AWS resources.

Q: Difference between Alias and CNAME?\
Ans:
| | Alias | CNAME |
|---|---|---|
| Zone apex (root domain) | Supported | Not supported |
| Target | AWS resources (ALB, CloudFront, S3, etc.) | Any hostname |
| Cost | Free for queries to AWS targets | Standard query pricing |
| Health check integration | Native, evaluates target health | No |

Q: What is TTL?\
Ans: Time To Live — how long (in seconds) a DNS resolver is allowed to cache a record before it must re-query Route53. Lower TTL means faster propagation of changes but more queries (and cost); higher TTL reduces load but slows change propagation.

## Routing Policies
Q: What Route53 routing policies are available?\
Ans: Simple, Weighted, Latency-based, Failover, Geolocation, Geoproximity (with Traffic Flow), and Multi-value Answer.

Q: Difference between Weighted and Latency Routing?\
Ans: **Weighted** routing splits traffic across multiple endpoints by a proportion you assign (e.g., 90/10 for canary releases), regardless of where the requester is. **Latency-based** routing sends each requester to whichever region/endpoint gives them the lowest network latency, based on measured latency between AWS regions and the requester.

Q: What is Failover Routing?\
Ans: An active-passive routing policy: Route53 sends traffic to a primary endpoint and continuously checks its health. If the primary fails its health check, Route53 automatically starts routing to the designated secondary (standby) endpoint.

## Health Checks
Q: What are Route53 Health Checks?\
Ans: Route53 can periodically probe an endpoint (HTTP/HTTPS/TCP, or an existing CloudWatch alarm) to determine if it's healthy. Unhealthy endpoints are automatically taken out of rotation for Failover, Weighted, and Multi-value routing policies, enabling automatic failover without manual DNS changes.

## DNS Troubleshooting
Q: How do DNS resolution and Route53 work?\
Ans: A client's resolver queries the DNS hierarchy: root servers → TLD servers → the domain's authoritative name servers (Route53's name servers for that hosted zone), which return the matching record. Route53 then applies its routing policy (if multiple records/health checks are configured) to decide which answer(s) to return. The result is cached by intermediate resolvers per the record's TTL.

Q: What should you check when DNS resolution fails?\
Ans:
- Verify the domain's registrar NS records point to Route53's assigned name servers (`dig NS example.com`).
- Confirm the record exists in the correct hosted zone and has the expected value (`dig A app.example.com`).
- Check TTL — a stale cached record may still be in effect at the resolver/client.
- For Alias/health-check-based records, check the target resource's health check status — an unhealthy target won't be returned.
- Rule out client-side DNS cache or a misconfigured `/etc/resolv.conf` / local resolver.
- Use `dig +trace` to walk the full resolution chain and see exactly where it breaks.

## Scenarios
Q: You want to redirect traffic from x.company.in to company.in/x. How would you achieve it?\
Ans: Route53 alone can't rewrite paths — it only resolves names to endpoints. The common pattern: point `x.company.in` (via an A/Alias record) at a small redirect target — either an S3 bucket configured for static website redirect rules, or a CloudFront distribution / ALB with a redirect rule/Lambda@Edge function — that issues an HTTP 301/302 to `https://company.in/x`. The actual redirect logic lives at that target, not in Route53 itself.
