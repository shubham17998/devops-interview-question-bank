# AWS Core Interview Questions

1. Q: How do you secure workloads on AWS?
   Ans: Least-privilege IAM (roles over static keys), encrypt data at rest and in transit, use private subnets/Security Groups to minimize exposure, enable logging/monitoring (CloudTrail, GuardDuty, VPC Flow Logs), patch OS/runtimes regularly, scan images/dependencies for vulnerabilities, and put WAF/Shield in front of internet-facing services.

2. Q: What is the difference between EC2 and S3?
   Ans: EC2 is compute — virtual servers you run applications/processes on. S3 is object storage — a place to durably store and retrieve files/data, with no compute of its own. They're complementary, not comparable substitutes: an EC2 instance might read/write objects to/from S3.

3. Q: How do you design a scalable multi-AZ deployment?
   Ans: Spread compute across at least 2-3 Availability Zones behind a load balancer, use an Auto Scaling Group to add/remove capacity based on demand, use a Multi-AZ database (RDS Multi-AZ or a distributed datastore), and keep the application stateless so any instance in any AZ can serve any request.

4. Q: How do you reduce AWS cloud costs?
   Ans: Right-size resources based on actual usage, use Savings Plans/Reserved Instances for steady baseline load and Spot for fault-tolerant workloads, apply S3 lifecycle policies, clean up unused resources (idle EIPs, orphaned volumes/snapshots), auto-scale to match demand, and regularly review Cost Explorer/Trusted Advisor recommendations.

5. Q: What is AWS Organizations?
   Ans: A service for centrally managing multiple AWS accounts — consolidated billing, Organizational Units (OUs) for grouping accounts, Service Control Policies (SCPs) for org-wide guardrails, and delegated administration of other services across accounts.

6. Q: What is AWS Identity Center?
   Ans: (Formerly AWS SSO) A service providing centralized single sign-on across multiple AWS accounts and business applications — users log in once and access assigned accounts/roles via permission sets, backed by an identity source (built-in, AD, or external IdP).

7. Q: What is the difference between IAM, AWS Identity Center and AWS Organizations?
   Ans: IAM manages identities/permissions *within a single account*. AWS Organizations manages/governs *multiple accounts* as a unit (billing, SCPs, OUs). AWS Identity Center provides centralized human *sign-on* across those multiple accounts, layered on top of Organizations.

8. Q: What is AWS Cost Optimization?
   Ans: The ongoing practice of ensuring AWS spend matches actual need — right-sizing, using the appropriate pricing model (On-Demand/Reserved/Spot/Savings Plans) per workload, eliminating waste (idle/unused resources), and continuously monitoring cost trends against budgets.

9. Q: How do you automatically stop/start EC2 instances at specific times?
   Ans: Use EventBridge Scheduler (or a cron-based EventBridge rule) to trigger a Lambda function (or Systems Manager Automation document) on a schedule that calls `ec2:StopInstances`/`ec2:StartInstances` on tagged instances — commonly used to shut down dev/test environments outside business hours to save cost.

10. Q: What is CloudWatch?
    Ans: AWS's native monitoring and observability service — collects metrics, logs, and events from AWS resources/applications, and supports dashboards, alarms, and automated actions.

11. Q: What is CloudTrail?
    Ans: A service that records API activity across your AWS account(s) — who did what, when, and from where — used for auditing, compliance, and security investigation (distinct from CloudWatch, which tracks operational health/performance).

12. Q: What is AWS WAF?
    Ans: Web Application Firewall — inspects and filters HTTP(S) requests to protect web applications (behind CloudFront, ALB, or API Gateway) from common exploits like SQL injection, XSS, and bad bots, using configurable rules.

13. Q: What is Auto Scaling Group?
    Ans: A logical group of EC2 instances that AWS automatically scales in/out based on defined policies and health checks, maintaining a target number of healthy instances and replacing unhealthy ones automatically.

14. Q: How do you design a highly available AWS architecture?
    Ans: Eliminate single points of failure: multi-AZ compute (ASG across AZs) and data (RDS Multi-AZ or a distributed datastore), redundant load balancers, health-checked self-healing, and — for the highest tier — multi-region with Route53 failover routing.

15. Q: How do you recover a deleted RDS database?
    Ans: If deletion protection/automated backups were enabled, restore from the latest automated snapshot or a specific point-in-time (`RestoreDBInstanceToPointInTime`) up to your backup retention window — this creates a new instance from that backup point (RDS doesn't support "undelete" in place). Without backups/snapshots, the data is unrecoverable — which is why enabling deletion protection and automated backups on production databases matters.

16. Q: What are the types of AWS Load Balancers?
    Ans: Application Load Balancer (Layer 7, HTTP/HTTPS), Network Load Balancer (Layer 4, TCP/UDP/TLS), and the legacy Classic Load Balancer.

17. Q: What is the difference between ALB and NLB?
    Ans: ALB operates at Layer 7 (content-based routing, SSL termination, WAF integration). NLB operates at Layer 4 (ultra-low latency, millions of requests/sec, static IPs, preserves source IP).

18. Q: How do you configure HTTP to HTTPS redirect in ALB?
    Ans: Add a listener rule on the ALB's HTTP (port 80) listener with a "Redirect" action targeting HTTPS (port 443, status code 301), so all HTTP traffic is redirected before reaching any target group.

19. Q: Does AWS provide SSL certificates? What is AWS Certificate Manager (ACM)?
    Ans: Yes — ACM provisions and manages free, auto-renewing SSL/TLS certificates for use with AWS services like ALB, CloudFront, and API Gateway (domain-validated certs at no extra cost when used with these integrated services).

20. Q: How do you attach an ACM certificate to ALB?
    Ans: On the ALB's HTTPS (443) listener, select the ACM certificate from the listener's default SSL certificate configuration (certificate must be in the same region as the ALB, except for CloudFront which requires `us-east-1`).

21. Q: How do you attach an external certificate to ALB?
    Ans: Import the external certificate (and its private key/chain) into ACM (`aws acm import-certificate`) first, then select that imported certificate on the ALB's HTTPS listener the same way as a native ACM-issued certificate.

22. Q: How do you reduce CloudWatch costs?
    Ans: Reduce log retention periods, filter out noisy/unnecessary logs before ingestion, use metric filters instead of shipping full logs where only counts/patterns are needed, avoid high-cardinality custom metric dimensions, and use Logs Insights queries sparingly (billed per GB scanned).

23. Q: How do you integrate application metrics with Prometheus?
    Ans: Instrument the application with a Prometheus client library exposing a `/metrics` endpoint, then have Prometheus scrape it directly (via static config or Kubernetes service discovery) — for AWS-native metrics, use the CloudWatch exporter to translate CloudWatch metrics into a Prometheus-scrapable format if you want everything in one Prometheus/Grafana stack.

## Storage & Access
24. Q: How do you give temporary access to a private S3 object?
    Ans: Generate a pre-signed URL (`aws s3 presign`, or the SDK's `getSignedUrl`), embedding a signature and expiration so the object is accessible via that link for a limited time without making it public.

25. Q: How do you rotate secrets weekly using AWS Secrets Manager?
    Ans: Enable automatic rotation on the secret, specifying a rotation Lambda function (AWS provides templates for RDS/DocumentDB/Redshift, or write a custom one for other secret types) and a rotation schedule (`rotationRules: { automaticallyAfterDays: 7 }`) — Secrets Manager then invokes the Lambda weekly to generate and update the credential.

26. Q: How does an EC2 instance securely access S3?
    Ans: Via an IAM role attached through an instance profile — the instance's SDK/CLI automatically retrieves short-lived, auto-rotated credentials scoped to that role from the Instance Metadata Service, without any hardcoded access keys.

## Databases
27. Q: What is RDS?
    Ans: Relational Database Service — AWS's managed relational database offering, handling provisioning, patching, backups, and (optionally) Multi-AZ failover for engines like MySQL, PostgreSQL, MariaDB, Oracle, and SQL Server.

28. Q: What database engines does RDS support?
    Ans: MySQL, PostgreSQL, MariaDB, Oracle, Microsoft SQL Server, and Amazon Aurora (MySQL- and PostgreSQL-compatible).

29. Q: What is Multi-AZ?
    Ans: An RDS high-availability feature that synchronously replicates data to a standby instance in a different AZ; if the primary fails, RDS automatically fails over to the standby (same endpoint, updated DNS) with minimal downtime.

30. Q: What is a Read Replica?
    Ans: An asynchronously-replicated, read-only copy of an RDS database, used to offload read traffic from the primary and improve read scalability — can also be promoted to a standalone writable instance if needed (e.g., for cross-region DR).

31. Q: Difference between Multi-AZ and Read Replica?
    Ans: Multi-AZ is for **availability** — a synchronous standby, not readable, used purely for automatic failover. Read Replicas are for **read scalability** — asynchronous, readable copies (can have multiple), not automatically failed-over-to (though promotable manually).

32. Q: How do you improve database read performance?
    Ans: Add read replicas to offload read traffic, add appropriate indexes, use a caching layer (ElastiCache) in front of frequent reads, optimize slow queries, and consider partitioning/sharding for very large datasets.

33. Q: How do you design a highly available database architecture?
    Ans: RDS Multi-AZ for automatic failover, read replicas (potentially cross-region) for read scaling and additional DR options, automated backups with a tested restore process, and connection pooling/retry logic in the application to handle brief failover interruptions gracefully.

34. Q: What is DynamoDB?
    Ans: AWS's fully managed, serverless NoSQL key-value/document database — single-digit millisecond latency at virtually any scale, with on-demand or provisioned capacity modes.

35. Q: What is a Partition Key?
    Ans: The attribute DynamoDB uses to determine which physical partition an item is stored on (via hashing) — choosing a partition key with high cardinality/even access distribution is critical to avoiding "hot partitions."

36. Q: What is a Sort Key?
    Ans: An optional second component of a DynamoDB primary key (used alongside the partition key) that determines the order items with the same partition key are stored in, enabling range queries within a partition (e.g., all orders for a customer, sorted by date).

37. Q: Difference between DynamoDB and RDS?
    Ans: DynamoDB is a fully managed, serverless NoSQL key-value/document store — scales horizontally with minimal operational overhead, schema-flexible, best for high-throughput/simple-access-pattern workloads. RDS is managed relational SQL — supports complex joins/transactions/ad-hoc queries, requires more careful capacity planning/vertical scaling, best for workloads needing relational integrity and complex querying.

38. Q: What is DynamoDB DAX?
    Ans: DynamoDB Accelerator — a fully managed, in-memory caching layer for DynamoDB that can reduce read latency from milliseconds to microseconds for read-heavy workloads, without requiring application logic changes beyond pointing at the DAX endpoint.

## Storage
39. Q: Difference between S3, EBS, and EFS?
    Ans: **S3** — object storage, accessed over HTTP/API, not mountable as a filesystem, virtually unlimited scale. **EBS** — block storage, attached to a single EC2 instance at a time (in the same AZ), like a virtual hard drive. **EFS** — a managed, elastic NFS file system, mountable by many EC2 instances simultaneously across AZs.

40. Q: What is EBS?
    Ans: Elastic Block Store — persistent, network-attached block storage volumes for EC2 instances, functioning like a virtual hard disk that can be resized, snapshotted, and (when detached) reattached to a different instance.

41. Q: What are EBS volume types?
    Ans: gp2/gp3 (general-purpose SSD), io1/io2 (provisioned IOPS SSD, for the highest-performance needs), st1 (throughput-optimized HDD), and sc1 (cold HDD, lowest cost for infrequent access).

42. Q: Difference between gp2, gp3, io1, io2, st1, and sc1?
    Ans: **gp2** — SSD, performance scales with volume size. **gp3** — SSD, IOPS/throughput provisioned independently of size, generally cheaper than gp2 for equivalent performance. **io1/io2** — provisioned IOPS SSD for the highest, most consistent performance needs (databases); io2 offers higher durability/max IOPS than io1. **st1** — HDD, optimized for high-throughput sequential workloads (big data, log processing), not for random I/O. **sc1** — HDD, lowest cost, for infrequently accessed data.

43. Q: Which EBS volume supports 64,000 IOPS?
    Ans: io2 Block Express (and io1/io2 in general support up to 64,000 IOPS on Nitro-based instances, with io2 Block Express supporting even higher limits) — gp3 tops out lower (16,000 IOPS) by comparison.

44. Q: What is EFS?
    Ans: Elastic File System — a fully managed, elastic NFS file system that can be mounted concurrently by many EC2 instances (and Lambda, ECS/EKS) across multiple AZs, growing/shrinking automatically as data is added/removed.

45. Q: When should EFS be used?
    Ans: When multiple compute instances/containers need to read/write a shared file system concurrently (e.g., shared config, CMS uploads, home directories in a multi-instance web farm) — anything needing genuine shared-filesystem semantics that EBS (single-attach) can't provide.

46. Q: What is FSx?
    Ans: A family of fully managed third-party file systems on AWS — FSx for Windows File Server (native SMB/AD-integrated), FSx for Lustre (high-performance computing workloads), FSx for NetApp ONTAP, and FSx for OpenZFS.

47. Q: What is FSx for Windows?
    Ans: A fully managed native Windows file system supporting the SMB protocol and Active Directory integration — used for lifting-and-shifting Windows file server workloads to AWS without re-architecting for NFS/EFS.

48. Q: When should FSx be preferred over EFS?
    Ans: When the workload specifically needs Windows-native SMB support and AD integration (EFS is Linux/NFS-only), or needs a specialized high-performance file system like Lustre for HPC/ML workloads that EFS isn't designed for.

## Messaging
49. Q: What is SQS?
    Ans: Simple Queue Service — a fully managed message queuing service that decouples producers and consumers, letting components communicate asynchronously via messages without needing to be online/available at the same time.

50. Q: What are SQS Standard and FIFO queues?
    Ans: **Standard** queues offer at-least-once delivery, best-effort ordering, and nearly unlimited throughput. **FIFO** queues guarantee exactly-once processing and strict ordering (within a message group), at the cost of lower throughput limits.

51. Q: Difference between Standard and FIFO queues?
    Ans: Standard: higher throughput, best-effort ordering, possible duplicate delivery. FIFO: strict first-in-first-out ordering, exactly-once processing, lower throughput ceiling (though "high throughput mode" for FIFO raises this significantly) — choose FIFO when message order/dedup correctness matters more than raw throughput.

52. Q: What is Visibility Timeout?
    Ans: The period after a consumer receives a message during which SQS hides that message from other consumers, assuming it's being processed — if the consumer doesn't delete it within that window (e.g., it crashed), the message becomes visible again for redelivery.

53. Q: What is a Dead Letter Queue?
    Ans: A separate queue that messages are automatically moved to after failing processing a configured number of times (exceeding `maxReceiveCount`) — isolates poison/unprocessable messages so they don't block the main queue indefinitely, and lets you inspect/reprocess them separately.

54. Q: What is Message Retention?
    Ans: The configurable duration (default 4 days, up to 14) SQS retains a message if it isn't deleted by a consumer — after this period, unprocessed messages are automatically discarded.

55. Q: What is SNS?
    Ans: Simple Notification Service — a fully managed pub/sub messaging service that broadcasts a single published message to multiple subscribers (email, SMS, Lambda, SQS, HTTP endpoints) simultaneously.

56. Q: What is a Topic in SNS?
    Ans: A named communication channel that publishers send messages to and subscribers subscribe to — SNS delivers each published message to every current subscriber of that topic.

57. Q: What is Fan-Out Architecture?
    Ans: A pattern where a single SNS topic publishes a message that's simultaneously delivered to multiple SQS queues (subscribers), letting several independent downstream services process the same event in parallel without the publisher knowing/caring how many consumers exist.

58. Q: Difference between SNS and SQS?
    Ans: SNS is push-based pub/sub — one message broadcast to many subscribers immediately. SQS is a pull-based point-to-point queue — messages wait until a consumer polls and retrieves them, and are processed once (per queue) rather than broadcast.

59. Q: How do SNS and SQS work together?
    Ans: SNS often fans out a published message to multiple SQS queues subscribed to the topic — combining SNS's broadcast/fan-out with SQS's durable, poll-based, retry-capable consumption, so each downstream service gets its own durable queue of events to process at its own pace.

60. Q: When should SNS be used?
    Ans: When an event needs to be broadcast to multiple independent consumers simultaneously (fan-out), or for direct notifications (email/SMS/push) — anywhere the "one event, many recipients, delivered immediately" pattern fits.

61. Q: When should SQS be used?
    Ans: When you need reliable, durable, asynchronous point-to-point processing with built-in retry/redelivery semantics and buffering against consumer downtime/slowness — decoupling a producer from a single consumer (or a consumer group) that processes at its own pace.

62. Q: How do you prioritize paid users over free users using SQS?
    Ans: Use separate queues per priority tier (a "paid" queue and a "free" queue), with consumers configured to poll the paid queue more frequently/with more workers (or exclusively first, falling back to the free queue only when the paid queue is empty) — SQS itself has no native per-message priority field, so priority is implemented at the queue-topology/consumer-behavior level.

## Lambda
63. Q: What is AWS Lambda?
    Ans: A serverless compute service that runs your code in response to events without provisioning or managing servers — you pay only for actual compute time consumed, and it scales automatically with the number of concurrent invocations.

64. Q: What are Lambda Triggers?
    Ans: Event sources that invoke a Lambda function — S3 events, API Gateway requests, DynamoDB Streams, SQS/SNS messages, EventBridge rules/schedules, and many others.

65. Q: What is Serverless Architecture?
    Ans: An application design where you write code (functions) without managing the underlying servers — the cloud provider handles provisioning, scaling, and availability; you pay per invocation/execution rather than for idle capacity.

66. Q: How does Lambda integrate with S3?
    Ans: S3 can be configured to publish event notifications (`s3:ObjectCreated:*`, etc.) directly to a Lambda function, automatically invoking it whenever a matching object event occurs — e.g., triggering image processing whenever a file is uploaded.

67. Q: How does Lambda integrate with API Gateway?
    Ans: API Gateway can be configured with a Lambda proxy integration, forwarding incoming HTTP requests directly to a Lambda function as its event payload, and returning the function's response back to the client — enabling fully serverless REST/HTTP APIs with no servers to manage.

## Monitoring
68. Q: What are CloudWatch Metrics?
    Ans: Time-ordered numeric data points representing a resource's behavior over time (CPUUtilization, RequestCount, etc.) — published automatically by AWS services, or as custom application metrics.

69. Q: What are CloudWatch Alarms?
    Ans: Watchers on a metric that transition between OK/ALARM/INSUFFICIENT_DATA states based on a threshold over an evaluation period, capable of triggering actions (SNS notification, Auto Scaling, EC2 recovery).

70. Q: How do you send alerts from CloudWatch?
    Ans: Create a CloudWatch Alarm on the relevant metric and set its action to publish to an SNS topic, which fans out to subscribers (email, SMS, Lambda, PagerDuty, Slack via a subscriber Lambda).

71. Q: Difference between CloudWatch and CloudTrail?
    Ans: CloudWatch monitors operational performance/health (metrics, logs, alarms). CloudTrail records API activity/audit history (who did what, when) for governance and security investigation.

72. Q: What is AWS X-Ray?
    Ans: A distributed tracing service that tracks requests as they travel across multiple services/components, producing a service map and per-segment latency breakdown to pinpoint bottlenecks/errors in distributed architectures.

## Migration
73. Q: What is DMS?
    Ans: Database Migration Service — a managed service for migrating databases to AWS (or between databases), supporting both one-time migrations and continuous replication (change data capture) for minimal-downtime cutovers, including cross-engine migrations via the Schema Conversion Tool.

74. Q: How do you migrate SQL Server to RDS with minimal downtime?
    Ans: Use AWS DMS with change data capture (CDC) enabled: perform a full initial load to the RDS target while the source stays live, then continuously replicate ongoing changes until the target has fully caught up, and finally cut over application traffic to RDS during a brief final sync window — minimizing downtime to just the cutover moment rather than the full migration duration.

75. Q: What is AWS DataSync?
    Ans: A managed data transfer service for moving large amounts of data between on-premises storage (or other clouds) and AWS storage services (S3, EFS, FSx), handling network optimization, scheduling, and data integrity verification automatically.

76. Q: When should DataSync be used?
    Ans: For online, ongoing, or one-time bulk data transfers over a network connection (not physically shipping media) — migrating file shares to AWS, or maintaining an ongoing hybrid-cloud data sync between on-prem storage and AWS storage.

77. Q: What is Snowball?
    Ans: A physical data transport appliance AWS ships to you — you load large volumes of data onto it locally, then ship it back to AWS for import, bypassing network bandwidth limits entirely for very large one-time data migrations.

78. Q: When should Snowball be used?
    Ans: When the data volume is so large that transferring it over the network would take impractically long (or cost too much in bandwidth) — a rule of thumb often cited: if a transfer would take more than about a week over your available network bandwidth, physical transport is likely faster/cheaper.

79. Q: Difference between DataSync and Snowball?
    Ans: DataSync moves data over a network connection (online, ongoing or one-time, requires adequate bandwidth). Snowball physically ships a device to transport data offline (for very large one-time transfers where network transfer would be impractically slow).

## Security
80. Q: What attacks can WAF prevent?
    Ans: SQL injection, cross-site scripting (XSS), bad bots/scrapers, known malicious IP-based patterns, and (via rate-based rules) basic application-layer DDoS/brute-force patterns.

81. Q: What is Rate Limiting in WAF?
    Ans: A rule type tracking request counts from a given IP (or other key) over a rolling window, automatically blocking/challenging a source once it exceeds a defined threshold.

82. Q: How do you block a malicious IP?
    Ans: Add an IP-set Deny rule in WAF (edge-level, fastest for internet-facing HTTP services), or a Security Group/NACL deny rule at the network layer.

83. Q: How do you block traffic from specific countries?
    Ans: Use a Geo Match rule in AWS WAF (or CloudFront's Geo Restriction feature) specifying the country codes to block.

84. Q: What is AWS Shield?
    Ans: AWS's managed DDoS protection service. Shield Standard is automatic/free, protecting against common L3/L4 DDoS attacks. Shield Advanced is a paid tier adding advanced detection/mitigation, cost protection, and 24/7 access to the DDoS Response Team.

85. Q: Difference between WAF and Shield?
    Ans: WAF operates at Layer 7, filtering HTTP request content (SQLi, XSS, rate limiting). Shield operates primarily at Layer 3/4, protecting against volumetric/protocol DDoS attacks — complementary, often used together.

86. Q: What is KMS?
    Ans: Key Management Service — a managed service for creating and controlling encryption keys used across AWS services, with fine-grained key policies, automatic rotation, and full CloudTrail audit logging.

87. Q: How do you encrypt data in AWS?
    Ans: At rest: enable encryption on the storage service (S3 SSE-S3/SSE-KMS, EBS encryption, RDS encryption) using KMS-managed or customer-managed keys. In transit: enforce TLS/HTTPS for all client-server and service-to-service communication.
