# Docker Interview Questions

## Basics

1. What is Docker?
2. What is a Docker Image?
3. Is a Docker image a static template?
4. What is a Container?
5. Difference between Docker Image and Container?
6. Difference between Dockerfile and Container?
7. Difference between Docker Image and Layer?
8. What is Containerd?
9. How does Docker work internally?
10. What happens internally when you run `docker run`?
11. Which runtime is used by Docker?
12. Docker vs Virtual Machine?
13. When would you use Docker and when would you use a VM?
14. How does Docker ensure isolation between containers?
15. What is Docker Hub?

## Dockerfile Instructions

16. What is a Dockerfile?
17. What is the purpose of a Dockerfile?
18. Difference between ADD and COPY?
19. Can we have multiple CMD instructions?
20. What Dockerfile instructions cannot be used multiple times?
21. What is ENTRYPOINT?
22. Difference between CMD and ENTRYPOINT?
23. How do CMD and ENTRYPOINT work together?
24. What is an entrypoint script and why would you use one?
25. Why do we use ARG in Dockerfile?
26. Difference between ARG and ENV?
27. What is Build Time vs Runtime?
28. What is the purpose of WORKDIR in a Dockerfile?
29. Why is it beneficial to copy dependency files (e.g., package.json, pom.xml) before copying the rest of the application code in a Dockerfile?
30. What is the purpose of .dockerignore?
31. What is the difference between package.json and package-lock.json in a Node.js project?

## Multi-Stage Builds

32. What is a Single-stage Dockerfile?
33. What is a Multi-stage Dockerfile?
34. Why do we use Multi-stage Builds?
35. How do you reduce Docker image size?
36. How do you minimize Docker image size?
37. What are Docker best practices?

## Build & Run

38. How do you containerize an application?
39. How do you build a Docker image?
40. Docker build command?
41. How do you run a Docker container?
42. Docker run command?
43. Difference between -p and -P?
44. How do you expose ports in Docker (EXPOSE vs -p)?
45. How do you pass environment variables to a container?
46. How do you access an application running in Docker?
47. How do you check container resource usage?
48. How do you inspect logs of a running container?
49. How do you debug a restarting container?
50. How do you troubleshoot a Docker container that keeps crashing?
51. What is the Docker container lifecycle (states)?
52. How do you delete all stopped containers?
53. How do you remove all unused Docker images, containers, and networks?
54. Can we create a Docker image without a Dockerfile?

## Docker Compose & Orchestration

55. What is Docker Compose?
56. How do you deploy multiple microservices using Docker Compose?
57. What is `depends_on` in Docker Compose?
58. Difference between Docker Swarm and Kubernetes?

## Networking

59. What are Docker network types?
60. What is Bridge Network?
61. What is Host Network?
62. What is Overlay Network?

## Volumes & Persistence

63. What are Docker Volumes?
64. Types of Docker Volumes?
65. What is a Bind Mount, and how does it differ from a Volume?
66. Can we persist Docker containers? If yes, how?
67. How do you share files between two containers running on different EC2 instances?

## Security & Troubleshooting

68. What are Container Exit Codes?
69. What is docker inspect?
70. How do you secure Docker containers?
71. If Trivy finds vulnerabilities in an image, how do you fix them?

## Practical Dockerfiles

72. Write a NodeJS Dockerfile.
73. Write an Nginx Dockerfile.
74. How would you create a Dockerfile to configure an Nginx web server?
75. What is the meaning of `daemon off;` and `-g` in Nginx?

## Windows Containers

76. Can .NET applications run on ECS?
77. What base image would you use for a .NET application?
78. How do you containerize Windows applications?

## Hands-on Exercises

### Exercise 1: Running Containers
**Objective:** Learn how to run, stop, and remove containers.
**Steps:**
1. Run a container using the latest nginx image.
2. List containers to confirm it's running.
3. Run another container using ubuntu:latest and attach to its terminal.
4. List containers again — how many are running now?
5. Stop the containers.
6. Remove the containers.

**Solution:**
```
docker run nginx:latest
docker ps
docker run -it ubuntu:latest /bin/bash
docker ps          # 2 running

docker stop $(docker ps -q)
docker rm $(docker ps -a -q)
```

### Exercise 2: Convert a Dockerfile to Multi-Stage
**Objective:** Practice converting a single-stage Dockerfile into a multi-stage build.
**Starting Dockerfile:**
```
FROM nginx
RUN apt-get update && apt-get install -y curl python build-essential nodejs && apt-get clean -y
RUN mkdir -p /my_app
ADD ./config/nginx/docker.conf /etc/nginx/nginx.conf
ADD app/ /my_cool_app
WORKDIR /my_cool_app
RUN npm install -g ember-cli bower
RUN npm install && bower install
RUN ember build --environment=prod
CMD ["nginx", "-g", "daemon off;"]
```
**Task:** Split this into a build stage (installs build tools, compiles the app) and a final stage (just nginx + the compiled output), then explain the benefits of doing so.

**Solution:**
```
FROM node:6 AS build
RUN npm install -g ember-cli bower
WORKDIR /my_cool_app
ADD app/ /my_cool_app
RUN npm install && bower install
RUN ember build --environment=prod

FROM nginx
ADD ./config/nginx/docker.conf /etc/nginx/nginx.conf
COPY --from=build /my_cool_app/dist /my_cool_app/dist
WORKDIR /my_cool_app
CMD ["nginx", "-g", "daemon off;"]
```
Multi-stage builds keep the final image small — build tools, source, and intermediate artifacts stay in the build stage and never end up in the shipped image.
