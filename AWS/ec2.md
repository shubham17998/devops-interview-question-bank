# EC2 Interview Questions

## Basics
1. Q: What is Amazon EC2?
   Ans: Elastic Compute Cloud — AWS's resizable virtual server service. You launch instances from an AMI, choose a size/family, attach storage and networking, and pay for compute time (or capacity commitment), without managing physical hardware.

2. Q: What are the different EC2 instance families?
   Ans: General Purpose (T, M — balanced compute/memory/network), Compute Optimized (C — high CPU-to-memory ratio), Memory Optimized (R, X, z1d — large RAM workloads like databases/caches), Storage Optimized (I, D, H1 — high I/O/throughput local storage), and Accelerated Computing (P, G, Inf — GPUs/inference/ML/graphics).

3. Q: What is the difference between burstable and non-burstable instances?
   Ans: **Burstable** (T-series) instances get a baseline CPU performance level and accumulate "CPU credits" during idle/low-usage periods, which can be spent to burst above baseline temporarily — cost-efficient for spiky, low-average workloads. **Non-burstable** instances (M, C, R, etc.) provide consistent, full CPU performance at all times with no credit system — better for steady, CPU-intensive workloads.

4. Q: How do CPU credits work in T-series instances?
   Ans: Each instance earns credits per hour based on its size/baseline, and one CPU credit equals one vCPU running at 100% for one minute. When usage is below baseline, unused credits accumulate (up to a cap); when usage exceeds baseline, credits are spent to allow bursting. `T3`/`T4g` instances also support "unlimited" mode, allowing sustained bursting beyond the credit balance for an extra charge.

5. Q: What happens when CPU credits are exhausted?
   Ans: In standard (non-unlimited) mode, the instance's CPU performance is throttled down to its baseline percentage — it can no longer burst until it accrues more credits, which can cause a visible performance drop under sustained load. In "unlimited" mode, the instance keeps bursting beyond baseline and you're billed for the extra vCPU-hours used.

6. Q: What is an AMI?
   Ans: Amazon Machine Image — a template containing the OS, application server, and application code (plus block device mapping) used to launch EC2 instances. AMIs can be AWS-provided, community-shared, marketplace, or custom-built from your own instances/snapshots.

7. Q: How do you create and use an AMI?
   Ans: Configure and test an instance the way you want it, then create an AMI from it (`Actions → Image and templates → Create image`, or `aws ec2 create-image`) — this snapshots its EBS volumes and registers a new AMI. You can then launch any number of new instances from that AMI, guaranteeing they start from the same baseline configuration.

8. Q: What is an Elastic Network Interface (ENI), and what role does it play in network connectivity?
   Ans: A virtual network interface that can be attached to an EC2 instance, carrying its own private IP(s), optional public/Elastic IP, MAC address, and security groups. ENIs let an instance have multiple network interfaces (e.g., for management vs. data traffic), and can be detached from one instance and re-attached to another to quickly move network identity/failover a workload.

## Pricing & Purchasing Options
9. Q: What are EC2 purchasing options?
   Ans: On-Demand, Reserved Instances, Savings Plans, Spot Instances, Dedicated Hosts, and Dedicated Instances.

10. Q: What are the EC2 pricing models?
    Ans: Pay-as-you-go (On-Demand), upfront/committed discounts (Reserved Instances, Savings Plans, 1 or 3 year terms), and spare-capacity discounted pricing (Spot Instances, up to ~90% off but reclaimable).

11. Q: Difference between On-Demand, Reserved, Spot, and Savings Plans?
    Ans: **On-Demand** — pay per second/hour with no commitment, most flexible, most expensive. **Reserved Instances** — commit to a specific instance type/region for 1-3 years for a significant discount. **Spot** — bid for spare AWS capacity at steep discounts, but instances can be reclaimed with a 2-minute warning — best for fault-tolerant/batch workloads. **Savings Plans** — commit to a $/hour spend for 1-3 years in exchange for a discount, more flexible than Reserved Instances since it applies across instance families/sizes (and even Fargate/Lambda for Compute Savings Plans).

## Instance Lifecycle
12. Q: Difference between Stop, Terminate, Reboot, and Hibernate?
    Ans: **Stop** — shuts down the instance, EBS root volume persists, no compute charges while stopped (EBS storage still billed), public IP is released unless Elastic IP. **Terminate** — permanently deletes the instance and (by default) its root EBS volume; cannot be restarted. **Reboot** — OS-level restart, keeps the same instance ID and IPs, no data loss. **Hibernate** — saves in-memory (RAM) state to the EBS root volume and stops the instance, so a subsequent start resumes the OS/apps from where they left off rather than a cold boot.

13. Q: What is EC2 Hibernate?
    Ans: A stop mode that persists the contents of RAM to the EBS root volume before shutting down, then restores that RAM state on next start — the instance boots up already in its previous running state instead of doing a fresh OS boot. Requires an encrypted root volume and appropriately sized volume/instance support.

14. Q: What is Instance Store?
    Ans: Temporary block-level storage physically attached to the host machine running the instance — very fast, but ephemeral: data is lost on stop, terminate, or underlying hardware failure (though it does survive a plain reboot).

15. Q: Difference between Instance Store and EBS?
    Ans: **Instance Store** is directly attached, high-performance, but ephemeral (tied to the instance's lifecycle/host) and not resizable independently. **EBS** is network-attached persistent block storage that survives stop/start and instance termination (unless configured to delete-on-termination), can be resized, snapshotted, and reattached to a different instance.

## Connectivity
16. Q: How do you connect to an EC2 instance?
    Ans: SSH (Linux, port 22) or RDP (Windows, port 3389) using a key pair; or without exposing those ports directly, via **AWS Systems Manager Session Manager**, which requires only an IAM role/SSM agent and no open inbound ports at all.

17. Q: What do you do if port 22 is closed and you cannot SSH into an EC2 instance?
    Ans: Use **EC2 Instance Connect** (browser-based, requires port 22 open but can use temporary short-lived keys) if available, or preferably **SSM Session Manager**, which doesn't need port 22 open at all — just the SSM agent running and an appropriate IAM role. If neither is available, check/update the security group to allow port 22 from your IP, or use the EC2 Serial Console for direct console access as a last resort.

18. Q: How do you access an EC2 instance that does not have a public IP?
    Ans: Via a bastion host/jump box in a public subnet with access to the private instance; a VPN or AWS Direct Connect connection into the VPC; or, most simply, AWS Systems Manager Session Manager, which reaches the instance over the SSM agent's outbound connection to the AWS Systems Manager service — no public IP, bastion, or open inbound ports required.

## Auto Scaling
19. Q: How do you update an application in an Auto Scaling Group without downtime?
    Ans: Create a new Launch Template version with the updated AMI/user data, then trigger an **Instance Refresh**, which replaces instances gradually (respecting a configurable minimum healthy percentage) while the ASG keeps serving traffic from the remaining healthy instances throughout the rollout.

20. Q: What is Instance Refresh in ASG?
    Ans: A native ASG feature that rolls out a new Launch Template version to all instances in the group by gradually terminating old instances and launching new ones in batches, governed by a minimum healthy percentage and optional warm-up time — avoiding a big-bang replacement.

21. Q: What are Launch Templates?
    Ans: The modern, versioned way to define an EC2 instance's launch configuration (AMI, instance type, key pair, security groups, user data, etc.) for use by an ASG or manual launches — supports multiple versions and mixing instance types.

22. Q: What are Launch Configurations?
    Ans: The older, legacy way to define launch parameters for an ASG — immutable (cannot be edited, only replaced) and single-instance-type only, since deprecated in favor of Launch Templates.

23. Q: Difference between a Launch Template and a Launch Configuration?
    Ans: Launch Templates support versioning, mixed instance types/purchase options (On-Demand + Spot in one ASG), and get ongoing feature updates from AWS. Launch Configurations are immutable, single-version, single instance type, and are the legacy/frozen option — AWS recommends Launch Templates for all new ASGs.

24. Q: What is an Auto Scaling Group?
    Ans: A logical group of EC2 instances that AWS automatically scales in/out based on defined policies, health checks, and desired/min/max capacity — ensuring a target number of healthy instances are always running and replacing unhealthy ones automatically.

25. Q: What are ASG scaling policies?
    Ans: Rules that decide when/how an ASG changes capacity: Target Tracking, Step Scaling, Simple Scaling, and Scheduled Scaling.

26. Q: What is Target Tracking Scaling?
    Ans: A scaling policy where you specify a target value for a metric (e.g., "keep average CPU at 50%"), and ASG automatically adds/removes instances to keep the metric near that target — similar to a thermostat.

27. Q: What is Step Scaling?
    Ans: A scaling policy that adds/removes capacity in defined step increments based on the magnitude of a CloudWatch alarm breach — e.g., CPU 50-70% adds 1 instance, CPU >70% adds 3 instances, giving finer-grained response than simple scaling.

28. Q: What is Simple Scaling?
    Ans: The original scaling policy type — a single scaling adjustment triggered by a CloudWatch alarm, followed by a cooldown period before another scaling activity can occur. Largely superseded by Step and Target Tracking policies which react faster/more precisely.

29. Q: What is Scheduled Scaling?
    Ans: Scaling actions that occur at specific, predictable date/times (e.g., scale up every weekday at 8am, scale down at 8pm) — useful for known traffic patterns rather than reactive metric-based scaling.

30. Q: What metrics can trigger Auto Scaling?
    Ans: CloudWatch metrics like average CPU utilization, network in/out, ALB request count per target, or any custom CloudWatch metric (e.g., SQS queue depth via a custom metric) can back a scaling policy's alarm.

31. Q: What is Cooldown Period?
    Ans: A configurable time window after a scaling activity during which further scaling actions (triggered by Simple Scaling alarms) are suppressed, preventing the ASG from over-reacting to the same spike multiple times before its effect is reflected in metrics.

32. Q: What is Warm-up Time?
    Ans: The time given to a newly launched instance to initialize/reach a ready state before it's counted toward Target Tracking/Step Scaling aggregate metrics or an Instance Refresh's health checks — prevents scaling from double-triggering on instances that haven't fully started yet.

33. Q: What are Lifecycle Hooks?
    Ans: Hooks that let you pause an instance in a `Pending` (before InService) or `Terminating` (before actual termination) state to run custom actions — e.g., installing software before it goes live, or draining connections/uploading logs before it's terminated.

34. Q: How does ASG replace unhealthy instances?
    Ans: The ASG continuously runs health checks (EC2 status checks and/or ELB health checks if attached). If an instance is marked unhealthy, the ASG terminates it and launches a replacement automatically to maintain the desired capacity.

35. Q: How do you scale applications based on SQS queue depth?
    Ans: Publish the queue's `ApproximateNumberOfMessagesVisible` as a CloudWatch metric (native SQS metric), then create a Target Tracking or Step Scaling policy on the consumer ASG using that metric (often normalized as messages-per-instance) as the trigger — so worker capacity scales with backlog size.

36. Q: How would you design a highly available and scalable application using ASG?
    Ans: Spread the ASG across multiple Availability Zones behind an ALB, use a Launch Template with health checks and lifecycle hooks for graceful startup/shutdown, configure target tracking scaling on a relevant metric, set sensible min/max/desired capacity, and use Instance Refresh for zero-downtime deployments.

37. Q: How do you create 10 EC2 instances with incrementing names/values (e.g., web-0, web-1, web-2)?
    Ans: With Terraform: `count = 10` on the resource, and `tags = { Name = "web-${count.index}" }`. With the AWS CLI, loop in a shell script calling `aws ec2 run-instances` and `aws ec2 create-tags --resources <id> --tags Key=Name,Value=web-$i` for each iteration.

38. Q: How would you terminate 9 out of 10 running EC2 instances and leave exactly one running?
    Ans: List running instance IDs (`aws ec2 describe-instances --filters Name=instance-state-name,Values=running --query 'Reservations[].Instances[].InstanceId'`), pick one to keep, then terminate the rest: `aws ec2 terminate-instances --instance-ids <9 ids>` (e.g., via a shell loop/`tail`/`grep -v` excluding the one to keep).

## Security & IAM
39. Q: How do you securely provide AWS permissions to an EC2 instance?
    Ans: Attach an IAM role to the instance via an Instance Profile — the instance then retrieves short-lived, auto-rotated credentials from the Instance Metadata Service, avoiding the need for stored access keys.

40. Q: What is an EC2 Instance Profile, and how is it used to grant permissions to an EC2 instance?
    Ans: A container object that wraps an IAM role and is what's actually attached to an EC2 instance (the console does this for you automatically when you "attach a role," but the CLI/API requires creating and associating the instance profile explicitly). The instance uses it to obtain temporary credentials for that role via IMDS.

41. Q: Why should IAM Roles be used instead of Access Keys?
    Ans: Roles provide short-lived, automatically rotated credentials with nothing to hardcode, leak, or manually rotate, removing the most common source of credential-leak incidents, and are easier to audit and revoke.

42. Q: What is the EC2 Instance Metadata Service, and how can you use it from within an instance?
    Ans: A local HTTP endpoint (`http://169.254.169.254/latest/meta-data/`) reachable only from inside the instance, exposing instance details (instance ID, AMI, IAM role credentials, user data, network config, etc.). Applications running on the instance query it (ideally IMDSv2, which requires a session token, to prevent SSRF-based credential theft) to retrieve temporary IAM role credentials or self-discover configuration.

43. Q: What is EC2 User Data, and how can you use it to automate instance configuration during launch?
    Ans: A script or cloud-init directive passed at launch time that the instance executes automatically on first boot — commonly used to install packages, pull configuration, register with a service, or run bootstrap automation, without needing to log in manually.

## Disaster Recovery
44. Q: How can you perform disaster recovery for EC2 in another region?
    Ans: Regularly copy AMIs (and EBS snapshots) to the DR region, keep Infrastructure as Code (Terraform/CloudFormation) ready to stand up the environment there, replicate data (cross-region RDS read replica, S3 cross-region replication), and use Route53 health checks with failover routing to redirect traffic to the DR region if the primary fails. The specific RTO/RPO target determines whether this is a cold, warm, or hot standby setup (pilot light, warm standby, or active-active).

## Hands-on Exercises

### Exercise 1: Attach an IAM Role to a Running Instance
**Objective:** Grant an EC2 instance AWS API access via an IAM role instead of access keys.
**Requirements:** A running EC2 instance with no IAM role attached (AWS CLI commands from it currently fail), and an IAM role with the `IAMReadOnlyAccess` policy (create it if it doesn't exist).
**Steps:**
1. Attach the role to the instance.
2. Verify you can now run AWS CLI commands from inside the instance.

**Solution (Console):**
1. Go to EC2 → select the instance.
2. Actions → Security → Modify IAM Role.
3. Choose the role with `IAMReadOnlyAccess` and save.
4. From the instance, run `aws iam list-users` — it should now succeed.
