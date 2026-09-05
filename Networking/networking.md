# Networking Interview Questions

## OSI & TCP/IP Models
1. Q: What are the OSI layers, and how does each function?
   Ans:
   1. **Physical** — raw bit transmission over a medium (cables, radio, voltages).
   2. **Data Link** — framing, MAC addressing, error detection on a local network segment (switches, Ethernet).
   3. **Network** — logical addressing and routing between networks (IP, routers).
   4. **Transport** — end-to-end delivery, reliability, ordering (TCP, UDP).
   5. **Session** — establishes/manages/terminates sessions between applications.
   6. **Presentation** — data translation/encryption/compression (TLS, encoding).
   7. **Application** — the protocols applications actually talk (HTTP, DNS, SMTP).

2. Q: Between the OSI and TCP/IP models, which one is better to implement and why?
   Ans: TCP/IP is what's actually implemented in real networks — it's simpler (4 layers vs 7), was designed alongside the protocols it describes (IP, TCP, UDP), and maps directly to how the internet works. OSI is more of a theoretical/teaching reference model — it's useful for reasoning about where a problem lives (e.g., "is this L3 or L4?"), but no stack strictly implements all seven distinct layers in practice.

3. Q: What exactly is the TCP/IP model?
   Ans: A 4-layer practical networking model: **Network Access** (physical + data link), **Internet** (IP — addressing and routing), **Transport** (TCP/UDP), and **Application** (HTTP, DNS, FTP, etc.). It's the model the actual internet protocol suite is built on.

4. Q: What is the difference between TCP and UDP? Where would you use each?
   Ans: **TCP** is connection-oriented, reliable, ordered, and flow/congestion-controlled (three-way handshake, retransmissions, ACKs) — use it when correctness and completeness matter more than latency: HTTP/HTTPS, databases, file transfer, SSH. **UDP** is connectionless, unreliable, unordered, but low-overhead and fast — use it where speed matters more than guaranteed delivery, or the application handles reliability itself: DNS queries, video/voice streaming, gaming, metrics/telemetry.

## IP Addressing
5. Q: What is CIDR, and why is it used?
   Ans: Classless Inter-Domain Routing — a notation (`10.0.0.0/16`) that specifies an IP range by base address and prefix length (number of fixed network bits). It replaced rigid class-based addressing (A/B/C) with flexible, arbitrarily-sized subnets, which reduced IP address wastage and shrank global routing tables via route aggregation.

6. Q: How many bytes are there in IPv4 and IPv6 addresses?
   Ans: IPv4 = 4 bytes (32 bits). IPv6 = 16 bytes (128 bits).

7. Q: What differentiates a logical address from a physical address?
   Ans: A **logical address** (IP address) is software-assigned, hierarchical, and can change (e.g., DHCP-issued) — it identifies a device's location on a network for routing. A **physical address** (MAC address) is burned into the network interface hardware, is flat (not hierarchical), and is used for delivery within a single local network segment (Layer 2).

8. Q: How do a MAC address and an IP address differ in function?
   Ans: The IP address gets a packet routed across networks to the right destination network/host (Layer 3). The MAC address handles the "last hop" delivery within a local segment — once a router determines the next hop, ARP resolves the destination IP to a MAC address so the frame can actually be delivered on that Ethernet segment (Layer 2).

## Load Balancing & Traffic
9. Q: What is a Network Gateway?
   Ans: A node that acts as an entry/exit point between two different networks, translating or routing traffic between them — e.g., a home router acting as the gateway between a LAN and the internet, or an Internet Gateway in a VPC.

10. Q: What are the different types of load balancers?
    Ans: **Layer 4 (Network/Transport)** — routes based on IP/port, very fast, no payload inspection (e.g., AWS NLB). **Layer 7 (Application)** — routes based on HTTP content (host, path, headers), supports SSL termination, content-based routing (e.g., AWS ALB, NGINX). Also categorized by placement: hardware LBs, software/cloud LBs, DNS-based (Route53), and global anycast LBs (CloudFront, Global Accelerator).

11. Q: What is the difference between a Load Balancer, a Reverse Proxy, and an Ingress?
    Ans: A **Load Balancer** distributes traffic across multiple backend instances for scalability/availability. A **Reverse Proxy** sits in front of one or more servers and forwards client requests to them — it may or may not load-balance, but often also does caching, SSL termination, or request rewriting (e.g., NGINX, HAProxy). An **Ingress** is Kubernetes-specific: an API object that defines HTTP(S) routing rules into the cluster, which is implemented by an Ingress Controller — effectively a reverse proxy/load balancer configured declaratively via Kubernetes resources.

## DNS
12. Q: What is DNS, and what is it used for?
    Ans: The Domain Name System — a distributed, hierarchical naming system that translates human-readable domain names (example.com) into machine-usable data, most commonly IP addresses, so clients can locate services without memorizing numeric addresses.

13. Q: What is the DNS resolution sequence when you look up a URL like www.example.com?
    Ans: Browser/OS checks its local cache → if not cached, queries a configured recursive resolver (e.g., ISP or 8.8.8.8) → resolver queries a **root server** for the TLD server → queries the **.com TLD server** for the authoritative name servers of example.com → queries **example.com's authoritative name server** for the `www` A/CNAME record → the answer is returned to the client and cached per its TTL along the way.

14. Q: What is a Domain Name Registrar?
    Ans: An accredited organization (e.g., GoDaddy, Namecheap, Route53 Domains) through which you register and manage ownership of a domain name, and configure which name servers are authoritative for it.

15. Q: In a FQDN like www.example.com, what are the root, top-level domain, second-level domain, and domain?
    Ans: Root = the implicit trailing `.` after `com` (the DNS root zone). Top-level domain (TLD) = `com`. Second-level domain = `example`. Domain = `example.com`. `www` is a subdomain/hostname within that domain.

16. Q: What is a DNS record, and what types of DNS records exist (A, AAAA, CNAME, PTR, MX, NS)?
    Ans: A DNS record is an entry in a zone file mapping a name to data. **A** — hostname to IPv4. **AAAA** — hostname to IPv6. **CNAME** — hostname to another hostname (alias). **PTR** — IP to hostname (reverse DNS). **MX** — mail exchange server(s) for a domain. **NS** — authoritative name servers for a zone.

17. Q: What is TTL in DNS?
    Ans: Time To Live — the number of seconds a resolver/client may cache a DNS record before re-querying the authoritative server for a fresh answer. Balances propagation speed (lower TTL) against query load/cost (higher TTL).

18. Q: Does DNS use TCP or UDP?
    Ans: Primarily **UDP** (port 53) for standard queries, since it's fast and low-overhead. It falls back to **TCP** for large responses that exceed UDP's size limits (e.g., DNSSEC, zone transfers) or when the truncation bit is set.

19. Q: Can DNS be used for load balancing? How?
    Ans: Yes — by returning multiple records for the same name (round-robin DNS), or via smarter routing policies like Route53's Weighted, Latency-based, Geolocation, or Multi-value Answer routing, so different clients (or repeated queries) are directed to different backend endpoints without a dedicated load balancer.

20. Q: What is a DNS zone?
    Ans: A distinct, administratively separate portion of the DNS namespace whose records are managed together and served by a specific set of authoritative name servers — e.g., the hosted zone for `example.com` in Route53.
