# CloudFront / CDN Interview Questions

## Basics
Q: What is CloudFront?
Ans: AWS's managed CDN (Content Delivery Network) — it caches and serves content from edge locations close to users, reducing latency and offloading traffic from the origin.

Q: What is a CDN?
Ans: A globally distributed network of caching servers (edge locations) that store copies of content close to end users, reducing latency, origin load, and bandwidth cost compared to serving every request from a single origin.

Q: What is an Edge Location?
Ans: A physical CloudFront point-of-presence, geographically distributed worldwide, that caches content and serves it to nearby users with lower latency than reaching back to the origin.

Q: What is an Origin?
Ans: The source CloudFront pulls content from when it's not already cached — typically an S3 bucket, an ALB/EC2, or any custom HTTP server.

Q: What is a Distribution?
Ans: The CloudFront configuration object that ties an origin (or multiple origins) to cache behaviors, TLS settings, and edge delivery rules — it's what you actually deploy and get a `*.cloudfront.net` domain (or custom domain) for.

## Caching
Q: How does CloudFront caching work?
Ans: On a request, the nearest edge location checks if a valid (non-expired) copy of the object exists in its cache. If yes (cache hit), it's served directly from the edge. If not (cache miss), CloudFront fetches it from the origin, caches it at the edge per the object's TTL/cache behavior, and returns it to the client — subsequent requests for that object from nearby users are then served from cache.

Q: What is TTL in CloudFront?
Ans: Time To Live — how long (seconds) an object stays cached at an edge location before CloudFront re-validates or re-fetches it from the origin. Configurable via cache behaviors, or driven by the origin's `Cache-Control`/`Expires` headers.

Q: What are Cache Behaviors?
Ans: Path-pattern-based rules on a distribution that control how CloudFront handles requests matching that path — which origin to use, TTL settings, which headers/cookies/query strings to forward, allowed HTTP methods, and viewer protocol policy. Lets one distribution serve different content types differently (e.g., `/api/*` vs `/static/*`).

Q: What is a Price Class?
Ans: A setting that controls which edge locations (regions) CloudFront is allowed to use to serve your content, trading off global coverage against cost — e.g., "Use only US, Canada, and Europe" is cheaper than "Use all edge locations worldwide."

Q: Can CloudFront cache dynamic content?
Ans: Yes, though by default dynamic content (personalized responses, API responses) is often set to bypass caching. You can still cache it with short TTLs, cache-per-header/cookie/query-string forwarding rules, or use CloudFront Functions/Lambda@Edge to customize what gets cached and how.

Q: How do you reduce latency in CloudFront?
Ans: Cache more aggressively (higher TTLs where content allows), enable compression, use HTTP/2 and keep-alive to origin, minimize origin round-trips with longer cache behaviors, and place origins in regions closer to the bulk of your users.

Q: How do you reduce latency globally?
Ans: Use CloudFront with edge locations near users worldwide, choose a Price Class covering the regions your users are in, use Origin Shield to reduce origin fetches, enable Anycast-based routing, and pair with Route53 latency-based routing or Global Accelerator for the non-cacheable/dynamic portion of traffic.

## Security
Q: How do you enable HTTPS in CloudFront?
Ans: Attach an SSL/TLS certificate (via AWS Certificate Manager, issued in `us-east-1` for CloudFront) to the distribution and set the Viewer Protocol Policy to "HTTPS only" or "Redirect HTTP to HTTPS."

Q: Where is SSL terminated in CloudFront?
Ans: At the edge location — CloudFront terminates the viewer's TLS connection at the nearest edge. It can then optionally re-establish a separate TLS connection to the origin (origin protocol policy), so origin traffic can also be encrypted end-to-end.

Q: How does CloudFront integrate with WAF?
Ans: You attach an AWS WAF Web ACL directly to a CloudFront distribution. WAF inspects requests at the edge before they're processed/cached, blocking or rate-limiting malicious traffic (SQLi, XSS, bad bots, rate-based rules) before it ever reaches your origin.

Q: What is Geo Restriction?
Ans: A distribution-level setting that allows or blocks access based on the viewer's country (using an allowlist or denylist of country codes), useful for content licensing or compliance restrictions.

Q: Can CloudFront work with a private S3 bucket?
Ans: Yes — using Origin Access Control (OAC, the modern replacement for the older OAI), CloudFront is granted permission to read from a private S3 bucket, while the bucket itself blocks all direct public access. This forces all traffic through CloudFront.

Q: What is a Signed URL?
Ans: A URL with an attached signature and expiration (and optionally IP restriction) generated using a CloudFront key pair, granting time-limited access to a specific private object — used to share individual protected files.

Q: What is a Signed Cookie?
Ans: Similar to a signed URL but grants access to multiple restricted objects (e.g., an entire private video library) via cookies set on the client, so you don't need to sign every individual URL.

## Advanced Features
Q: What is CloudFront Behavior?
Ans: Same concept as Cache Behavior — a routing/caching rule scoped to a URL path pattern within a distribution.

Q: What happens if the origin is down?
Ans: Cached objects continue to be served from edge locations until their TTL expires. Once expired, CloudFront attempts to fetch from the origin and will return an error (e.g., 502/504) to the client unless Origin Failover is configured, or `Stale-While-Revalidate`/custom error responses are set up to serve stale content temporarily.

Q: What is Origin Failover?
Ans: A CloudFront feature where you configure an Origin Group with a primary and a secondary origin. If the primary origin returns specific failure status codes, CloudFront automatically retries the request against the secondary origin — providing origin-level high availability.

Q: What is Lambda@Edge?
Ans: A feature letting you run Lambda functions (Node.js/Python) at CloudFront edge locations, triggered at four points in the request/response cycle (viewer request, origin request, origin response, viewer response) — used for things like header manipulation, A/B testing, auth checks, or URL rewrites closer to the user. (CloudFront Functions is a lighter-weight, JS-only alternative for simpler, higher-throughput use cases.)

## Configuration
Q: How do you secure an S3 website using CloudFront?
Ans: Make the S3 bucket private (block all public access), create a CloudFront distribution with the bucket as origin using Origin Access Control so only CloudFront can read it, enforce HTTPS via the viewer protocol policy, and optionally attach WAF for additional protection.

Q: How do you configure CloudFront with S3?
Ans: Create a distribution, set the S3 bucket (or its website endpoint, for static site hosting with redirects/index docs) as the origin, configure OAC for private-bucket access, set cache behaviors/TTLs, and attach a certificate/custom domain as needed.

Q: How do you configure CloudFront with ALB?
Ans: Set the ALB's DNS name as a custom origin on the distribution, choose the origin protocol policy (HTTP/HTTPS to origin), configure cache behaviors (often with caching disabled or minimal for dynamic APIs), and attach the ACM certificate for the viewer-facing domain.

Q: Difference between CloudFront and ALB?
Ans: **CloudFront** is a global CDN — it caches and serves content from edge locations worldwide, primarily optimizing for latency and offload at the edge. **ALB** is a regional Layer 7 load balancer — it distributes incoming traffic across backend targets (EC2, ECS, Lambda) within a region/VPC and doesn't cache content. They're often used together: CloudFront in front of an ALB for global caching + edge security, with the ALB handling regional load balancing to the actual compute.
