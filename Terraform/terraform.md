# Terraform Interview Questions

## Basics
1. What is Terraform?
2. What language does Terraform use?
3. What are the different Terraform file formats?
4. What is terraform fmt?
5. What is Terraform Validate?
6. Difference between Terraform Validate and Terraform Fmt?
7. What does terraform init do?
8. What is the purpose of the `terraform graph` command?
9. What are Terraform providers, and where are they downloaded from after `terraform init`?
10. Can multiple providers be used in a single Terraform configuration?
11. How do you auto-approve Terraform changes without an interactive prompt?

## State Management
12. What is Terraform state and why is it needed?
13. Where do you store Terraform State in production?
14. What is Terraform State Locking?
15. How does DynamoDB locking work?
16. What if only S3 backend is available?
17. What happens if Terraform state is accidentally deleted?
18. How would you handle a Terraform state corruption?
19. Difference between Terraform State and Workspace?
20. How do you migrate Terraform state to a new backend without recreating resources?
21. How does Terraform prevent duplicate resource creation?

## Drift
22. What is Terraform Drift?
23. Drift vs Refresh?
24. Example of Terraform Drift?
25. How do you handle Terraform Drift?
26. What would you do if Terraform drift is detected in production?
27. What happens if someone manually changes infrastructure managed by Terraform?
28. If EBS volume changed from 8GB to 20GB manually, what happens during Terraform Apply?
29. What will appear in the Terraform plan if you comment out a resource block that was previously applied?

## Refresh & Import
30. What is Terraform Refresh?
31. What is terraform apply -refresh-only?
32. What is terraform import and where do you use it?
33. What is a null resource in Terraform?

## Taint & Lifecycle
34. What is Terraform Taint?
35. Terraform taint vs untaint?
36. What is Untaint?
37. What is a Terraform Lifecycle Block?
38. What is prevent_destroy?
39. Does prevent_destroy apply to the entire configuration?
40. How do you delete a resource protected by prevent_destroy?

## Workspaces & Modules
41. What is Terraform Workspace?
42. How do Terraform workspaces work?
43. Does each workspace have a separate state file?
44. What happens if you modify one workspace state file?
45. What are Terraform Modules?
46. How do you structure Terraform modules?
47. How does a Terraform module differ from a template?

## Resources & Expressions
48. What is a Terraform Data Block?
49. What is depends_on?
50. Difference between count and for_each?
51. What are meta-arguments in Terraform?
52. What is indexing in Terraform (e.g., count.index)?
53. How do you create multiple EC2 instances with different configurations?

## Provisioners
54. What are Terraform Provisioners?
55. Types of Terraform Provisioners?
56. Difference between Provider and Provisioner?
57. Why do we use Provisioners?

## Secrets & Security
58. How do you pass secrets to Terraform?
59. How do you implement autoscaling using Terraform?
60. How do you secure Terraform execution from Jenkins?

## Rollback & Collaboration
61. How do you handle Terraform rollback?
62. How do multiple developers work on the same Terraform codebase?
63. How do you manage dependencies in Terraform?

## Practical
64. Write Terraform code to deploy an EC2 instance.

## Troubleshooting
65. Have you seen terraform plan pass but apply fail?
66. Have you seen terraform apply succeed but resources fail afterward?

## Hands-on Exercises

### Exercise 1: Launch an EC2 Instance
**Objective:** Write and apply Terraform configuration to launch an EC2 instance.
**Steps:**
1. Write a Terraform configuration for an EC2 instance.
2. Run the commands to apply it and create the instance.
3. Run `terraform apply` again — what happens?
4. Destroy the instance with Terraform.

**Solution:**
```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-west-2"
}

resource "aws_instance" "app_server" {
  ami           = "ami-0abcdef1234567890"
  instance_type = "t2.micro"

  tags = {
    Name = "ExampleAppServerInstance"
  }
}
```
```
terraform init
terraform validate
terraform plan      # Plan: 1 to add, 0 to change, 0 to destroy
terraform apply -auto-approve

# Running apply again: Apply complete! Resources: 0 added, 0 changed, 0 destroyed.
# Terraform compares actual infra to config and finds no drift.

terraform destroy -auto-approve
```

### Exercise 2: Create a Custom VPC with Subnets
**Objective:** Create a VPC with two subnets across different Availability Zones.
**Steps:**
1. Create a VPC with CIDR block `10.0.0.0/16`.
2. Create two subnets, e.g. `10.0.0.0/20` and `10.0.16.0/20`, each in a different AZ.
3. Apply the configuration and confirm both resources are tracked in state.

**Solution:**
```hcl
resource "aws_vpc" "my_custom_vpc" {
  cidr_block = "10.0.0.0/16"
  tags = { Name = "my_custom_vpc" }
}

resource "aws_subnet" "subnet_a" {
  vpc_id            = aws_vpc.my_custom_vpc.id
  cidr_block        = "10.0.0.0/20"
  availability_zone = "us-west-2a"
  tags = { Name = "Subnet A" }
}

resource "aws_subnet" "subnet_b" {
  vpc_id            = aws_vpc.my_custom_vpc.id
  cidr_block        = "10.0.16.0/20"
  availability_zone = "us-west-2b"
  tags = { Name = "Subnet B" }
}
```
```
terraform init
terraform apply -auto-approve
```
