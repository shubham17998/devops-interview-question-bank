# Docker Interview Questions

## Basics

1. What is Docker?
2. What is a Docker Image?
3. Is a Docker image a static template?
4. What is a Container?
5. Difference between Docker Image and Container?
6. Difference between Dockerfile and Container?
7. What is Containerd?
8. How does Docker work internally?
9. Which runtime is used by Docker?
10. Docker vs Virtual Machine?
11. When would you use Docker and when would you use a VM?

## Dockerfile Instructions

12. What is a Dockerfile?
13. What is the purpose of a Dockerfile?
14. Difference between ADD and COPY?
15. Can we have multiple CMD instructions?
16. What Dockerfile instructions cannot be used multiple times?
17. What is ENTRYPOINT?
18. Difference between CMD and ENTRYPOINT?
19. How do CMD and ENTRYPOINT work together?
20. Why do we use ARG in Dockerfile?
21. Difference between ARG and ENV?
22. What is Build Time vs Runtime?

## Multi-Stage Builds

23. What is a Single-stage Dockerfile?
24. What is a Multi-stage Dockerfile?
25. Why do we use Multi-stage Builds?
26. How do you reduce Docker image size?
27. How do you minimize Docker image size?
28. What are Docker best practices?

## Build & Run

29. How do you containerize an application?
30. How do you build a Docker image?
31. Docker build command?
32. How do you run a Docker container?
33. Docker run command?
34. Difference between -p and -P?
35. How do you access an application running in Docker?
36. How do you debug a restarting container?
37. How do you troubleshoot a Docker container that keeps crashing?
38. How do you delete all stopped containers?
39. Can we create a Docker image without a Dockerfile?

## Docker Compose & Orchestration

40. What is Docker Compose?
41. Difference between Docker Swarm and Kubernetes?

## Networking

42. What are Docker network types?
43. What is Bridge Network?
44. What is Host Network?
45. What is Overlay Network?

## Volumes & Persistence

46. What are Docker Volumes?
47. Types of Docker Volumes?
48. Can we persist Docker containers? If yes, how?
49. How do you share files between two containers running on different EC2 instances?

## Security & Troubleshooting

50. What are Container Exit Codes?
51. What is docker inspect?
52. How do you secure Docker containers?
53. If Trivy finds vulnerabilities in an image, how do you fix them?

## Practical Dockerfiles

54. Write a NodeJS Dockerfile.
55. Write an Nginx Dockerfile.
56. How would you create a Dockerfile to configure an Nginx web server?
57. What is the meaning of `daemon off;` and `-g` in Nginx?

## Windows Containers

58. Can .NET applications run on ECS?
59. What base image would you use for a .NET application?
60. How do you containerize Windows applications?
