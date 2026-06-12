# Terraform Interview Questions

## Basics
1. What is Terraform?
2. What language does Terraform use?
3. What are the different Terraform file formats?
4. What is terraform fmt?
5. What is Terraform Validate?
6. Difference between Terraform Validate and Terraform Fmt?
7. What does terraform init do?

## State Management
8. What is Terraform State?
9. What is Terraform state and why is it needed?
10. Where do you store Terraform State in production?
11. What is Terraform State Locking?
12. How does DynamoDB locking work?
13. What if only S3 backend is available?
14. What happens if Terraform state is accidentally deleted?
15. How would you handle a Terraform state corruption?
16. Difference between Terraform State and Workspace?

## Drift
17. What is Terraform Drift?
18. Drift vs Refresh?
19. Example of Terraform Drift?
20. How do you handle Terraform Drift?
21. What would you do if Terraform drift is detected in production?
22. What happens if someone manually changes infrastructure managed by Terraform?
23. If EBS volume changed from 8GB to 20GB manually, what happens during Terraform Apply?

## Refresh & Import
24. What is Terraform Refresh?
25. What is terraform apply -refresh-only?
26. What is Terraform Import?
27. What is terraform import and where do you use it?

## Taint & Lifecycle
28. What is Terraform Taint?
29. Terraform taint vs untaint?
30. What is Untaint?
31. What is a Terraform Lifecycle Block?
32. What is prevent_destroy?
33. Does prevent_destroy apply to the entire configuration?
34. How do you delete a resource protected by prevent_destroy?

## Workspaces & Modules
35. What is Terraform Workspace?
36. How do Terraform workspaces work?
37. Does each workspace have a separate state file?
38. What happens if you modify one workspace state file?
39. What are Terraform Modules?
40. How do you structure Terraform modules?

## Resources & Expressions
41. What is a Terraform Data Block?
42. What is depends_on?
43. Difference between count and for_each?
44. How do you create multiple EC2 instances with different configurations?

## Provisioners
45. What are Terraform Provisioners?
46. Types of Terraform Provisioners?
47. Difference between Provider and Provisioner?
48. Why do we use Provisioners?

## Secrets & Security
49. How do you pass secrets to Terraform?
50. How do you implement autoscaling using Terraform?
51. How do you secure Terraform execution from Jenkins?

## Rollback & Collaboration
52. How do you handle Terraform rollback?
53. How do multiple developers work on the same Terraform codebase?
54. How do you manage dependencies in Terraform?

## Troubleshooting
55. Have you seen terraform plan pass but apply fail?
56. Have you seen terraform apply succeed but resources fail afterward?
