# EC2 Interview Questions

## Basics
1. What is Amazon EC2?
2. What are the different EC2 instance families?
3. What is the difference between burstable and non-burstable instances?
4. How do CPU credits work in T-series instances?
5. What happens when CPU credits are exhausted?
6. What is an AMI?
7. How do you create and use an AMI?
8. What is an Elastic Network Interface (ENI), and what role does it play in network connectivity?

## Pricing & Purchasing Options
9. What are EC2 purchasing options?
10. What are the EC2 pricing models?
11. Difference between On-Demand, Reserved, Spot, and Savings Plans?

## Instance Lifecycle
12. Difference between Stop, Terminate, Reboot, and Hibernate?
13. What is EC2 Hibernate?
14. What is Instance Store?
15. Difference between Instance Store and EBS?

## Connectivity
16. How do you connect to an EC2 instance?
17. What do you do if port 22 is closed and you cannot SSH into an EC2 instance?
18. How do you access an EC2 instance that does not have a public IP?

## Auto Scaling
19. How do you update an application in an Auto Scaling Group without downtime?
20. What is Instance Refresh in ASG?
21. What are Launch Templates?
22. What are Launch Configurations?
23. Difference between a Launch Template and a Launch Configuration?
24. What is an Auto Scaling Group?
25. What are ASG scaling policies?
26. What is Target Tracking Scaling?
27. What is Step Scaling?
28. What is Simple Scaling?
29. What is Scheduled Scaling?
30. What metrics can trigger Auto Scaling?
31. What is Cooldown Period?
32. What is Warm-up Time?
33. What are Lifecycle Hooks?
34. How does ASG replace unhealthy instances?
35. How do you scale applications based on SQS queue depth?
36. How would you design a highly available and scalable application using ASG?
37. How do you create 10 EC2 instances with incrementing names/values (e.g., web-0, web-1, web-2)?
38. How would you terminate 9 out of 10 running EC2 instances and leave exactly one running?

## Security & IAM
39. How do you securely provide AWS permissions to an EC2 instance?
40. What is an EC2 Instance Profile, and how is it used to grant permissions to an EC2 instance?
41. Why should IAM Roles be used instead of Access Keys?
42. What is the EC2 Instance Metadata Service, and how can you use it from within an instance?
43. What is EC2 User Data, and how can you use it to automate instance configuration during launch?

## Disaster Recovery
44. How can you perform disaster recovery for EC2 in another region?

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
