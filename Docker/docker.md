# Docker Interview Questions

## Basics

Q: What is Docker?
Ans: A platform for building, shipping, and running applications in lightweight, isolated units called containers, which package an application with all its dependencies so it runs consistently across any environment.

Q: What is a Docker Image?
Ans: A read-only, layered template containing an application's filesystem and metadata (entrypoint, env vars, exposed ports) — the blueprint used to create running containers.

Q: Is a Docker image a static template?
Ans: Yes — an image itself is immutable; it never changes after being built. Running it creates a container with a thin writable layer on top, and any runtime changes happen in that container layer, not the underlying image.

Q: What is a Container?
Ans: A running instance of an image — an isolated process (or group of processes) with its own filesystem (image layers + a writable layer), network namespace, and resource limits, created from an image.

Q: Difference between Docker Image and Container?
Ans: An image is the static, immutable template (read-only layers). A container is a running (or stopped) instance created from that image, with an added writable layer for any runtime changes — many containers can be created from the same image.

Q: Difference between Dockerfile and Container?
Ans: A Dockerfile is a text file of build instructions used to *produce* an image. A container is a *running instance* of an already-built image. The Dockerfile never runs directly — it's compiled (via `docker build`) into an image, which is then run as a container.

Q: Difference between Docker Image and Layer?
Ans: An image is composed of multiple stacked, read-only **layers**, each corresponding to an instruction in the Dockerfile (e.g., each `RUN`/`COPY` creates a layer). Layers are cached and shared across images — an image is the full assembled stack; a layer is one filesystem diff within that stack.

Q: What is Containerd?
Ans: A container runtime that manages the container lifecycle (pulling images, creating/starting/stopping containers, managing storage/networking at a low level) — Docker itself uses containerd under the hood as its runtime engine.

Q: How does Docker work internally?
Ans: The Docker CLI talks to the Docker daemon (`dockerd`) via a REST API. The daemon delegates actual container execution to containerd, which uses `runc` (an OCI-compliant low-level runtime) to create containers using Linux kernel features — namespaces (isolation of process/network/mount/etc. views) and cgroups (resource limiting) — plus a union filesystem (overlay2) to assemble image layers.

Q: What happens internally when you run `docker run`?
Ans: Docker checks if the image exists locally (pulls it from a registry if not) → creates a new container filesystem from the image's layers plus a new writable layer → sets up namespaces (network, PID, mount, etc.) and cgroups for isolation/resource limits → configures networking (assigns an IP, connects to the specified network) → starts the specified process (CMD/ENTRYPOINT) as PID 1 inside the container.

Q: Which runtime is used by Docker?
Ans: `runc` — the default OCI-compliant low-level container runtime, invoked by containerd, which Docker uses under the hood.

Q: Docker vs Virtual Machine?
Ans: A **VM** virtualizes hardware — each VM runs its own full OS kernel via a hypervisor, giving strong isolation but higher overhead (slower startup, larger footprint). A **Docker container** shares the host's kernel and isolates processes via namespaces/cgroups — much lighter weight and faster to start, but with a slightly weaker isolation boundary since containers share the host kernel.

Q: When would you use Docker and when would you use a VM?
Ans: Use Docker for fast, lightweight, horizontally-scaled application deployment where kernel-level isolation isn't a hard requirement (most microservices/web apps). Use a VM when you need strong security/hardware-level isolation (multi-tenant untrusted workloads), need a different OS/kernel than the host, or need to run legacy/full-OS workloads that aren't containerized.

Q: How does Docker ensure isolation between containers?
Ans: Via Linux kernel **namespaces** (PID, network, mount, UTS, IPC, user — each giving a container its own isolated view of that resource) and **cgroups** (limiting/accounting CPU, memory, I/O per container) — combined with a separate root filesystem per container (from its image layers).

Q: What is Docker Hub?
Ans: Docker's default public container image registry — used to store, discover, and pull/push official and community/private images.

## Dockerfile Instructions

Q: What is a Dockerfile?
Ans: A text file of sequential instructions (`FROM`, `RUN`, `COPY`, `CMD`, etc.) that Docker reads to build an image layer by layer.

Q: What is the purpose of a Dockerfile?
Ans: To declaratively and reproducibly define how an image is built — starting base image, dependencies to install, files to copy in, environment/config, and the default command to run — so anyone (or any CI pipeline) can build an identical image from source.

Q: Difference between ADD and COPY?
Ans: `COPY` simply copies files/directories from the build context into the image. `ADD` does that too, but also supports auto-extracting local tar archives and fetching remote URLs directly into the image. Best practice is to prefer `COPY` unless you specifically need `ADD`'s extra behavior, since it's more predictable/explicit.

Q: Can we have multiple CMD instructions?
Ans: Yes, syntactically, but only the **last** `CMD` in the Dockerfile takes effect — all earlier ones are ignored, so there's effectively no benefit to having more than one.

Q: What Dockerfile instructions cannot be used multiple times?
Ans: Practically, `CMD` and `ENTRYPOINT` — only the last occurrence of each takes effect if specified multiple times (each simply overrides the previous one rather than combining).

Q: What is ENTRYPOINT?
Ans: Defines the fixed, main executable that always runs when the container starts — arguments passed via `docker run` or `CMD` are appended to it rather than replacing it (in exec form).

Q: Difference between CMD and ENTRYPOINT?
Ans: `CMD` provides default arguments/command that *can be fully overridden* by arguments passed to `docker run`. `ENTRYPOINT` defines the command that *always runs* — any `docker run` arguments (or `CMD`) are appended to it as arguments rather than replacing it.

Q: How do CMD and ENTRYPOINT work together?
Ans: `ENTRYPOINT` sets the fixed executable, and `CMD` supplies default arguments to it — e.g., `ENTRYPOINT ["python"]` + `CMD ["app.py"]` runs `python app.py` by default, but `docker run myimage other.py` runs `python other.py` (CMD's default is overridden, ENTRYPOINT stays fixed).

Q: What is an entrypoint script and why would you use one?
Ans: A shell script set as the `ENTRYPOINT` (instead of the app binary directly) that performs setup work — waiting for a dependency, running migrations, templating config from env vars — before finally `exec`-ing the real application process. Used to handle runtime initialization logic that can't be expressed in the Dockerfile's build-time instructions.

Q: Why do we use ARG in Dockerfile?
Ans: `ARG` defines a build-time-only variable (passed via `docker build --build-arg`), used to parameterize the build itself (e.g., a version number, a build flag) — it does not persist into the running container's environment.

Q: Difference between ARG and ENV?
Ans: `ARG` is only available during the image *build* (not in the running container, unless explicitly passed into an `ENV` in the Dockerfile). `ENV` sets an environment variable that persists into the built image and is available at container *runtime*.

Q: What is Build Time vs Runtime?
Ans: **Build time** is when `docker build` executes the Dockerfile's instructions to produce an image (e.g., installing packages, `ARG` values apply here). **Runtime** is when a container created from that image is actually running (e.g., `ENV` values, the process started by `CMD`/`ENTRYPOINT`, apply here).

Q: What is the purpose of WORKDIR in a Dockerfile?
Ans: Sets the working directory for any subsequent `RUN`, `CMD`, `ENTRYPOINT`, `COPY`, `ADD` instructions (and for the container's shell if you exec into it) — cleaner and more portable than `cd`-ing manually inside `RUN` commands.

Q: Why is it beneficial to copy dependency files (e.g., package.json, pom.xml) before copying the rest of the application code in a Dockerfile?
Ans: Docker caches each layer; if the dependency manifest hasn't changed, the (often slow) dependency-install layer is reused from cache even if application source code changed. Copying dependency files first and running the install step before copying the rest of the source means routine code changes don't invalidate/re-run the expensive dependency-install step.

Q: What is the purpose of .dockerignore?
Ans: Like `.gitignore`, but for the build context — excludes files/directories (node_modules, .git, build artifacts, secrets) from being sent to the Docker daemon during build, reducing build context size/time and preventing unwanted files from accidentally being copied into the image.

Q: What is the difference between package.json and package-lock.json in a Node.js project?
Ans: `package.json` declares the project's dependencies with permitted version ranges (e.g., `^1.2.0`). `package-lock.json` records the exact resolved version of every installed dependency (and sub-dependency) actually installed, ensuring reproducible installs across machines/builds regardless of what new versions might otherwise satisfy the ranges in `package.json`.

## Multi-Stage Builds

Q: What is a Single-stage Dockerfile?
Ans: A Dockerfile with only one `FROM` instruction — everything (build tools, source, compiled output) ends up in the same final image, often bloating it with tools/artifacts only needed during build, not at runtime.

Q: What is a Multi-stage Dockerfile?
Ans: A Dockerfile with multiple `FROM` instructions (stages), where later stages can selectively `COPY --from=<stage>` specific artifacts from earlier stages — letting you use a heavy "build" stage for compiling, then discard everything except the final compiled output in a lean final stage.

Q: Why do we use Multi-stage Builds?
Ans: To keep the final image small and secure by excluding build-only tooling, source code, and intermediate artifacts — only what's needed to actually run the application ends up in the shipped image.

Q: How do you reduce Docker image size?
Ans: Use multi-stage builds, choose a minimal base image (alpine, distroless, slim variants), combine/minimize `RUN` layers, clean up package manager caches within the same layer they were created in, avoid copying unnecessary files (use `.dockerignore`), and avoid installing dev-only dependencies in the final image.

Q: How do you minimize Docker image size?
Ans: Same techniques as reducing image size — multi-stage builds, minimal base images, layer consolidation, `.dockerignore`, pruning unnecessary dependencies/files, and using tools like `docker image inspect`/`dive` to identify and eliminate bloated layers.

Q: What are Docker best practices?
Ans: Use minimal base images; leverage multi-stage builds; order Dockerfile instructions from least-to-most frequently changing (for cache efficiency); use `.dockerignore`; run as a non-root user; pin specific image tags/digests (avoid `latest`); keep one process/concern per container; use `HEALTHCHECK`; and scan images for vulnerabilities before deploying.

## Build & Run

Q: How do you containerize an application?
Ans: Write a Dockerfile specifying a suitable base image, copy in dependency manifests and install dependencies, copy application code, expose the needed port, and set the startup command — then build (`docker build`) and run (`docker run`) it.

Q: How do you build a Docker image?
Ans: `docker build -t <name>:<tag> <path-to-context>` (e.g., `docker build -t myapp:1.0 .`).

Q: Docker build command?
Ans: `docker build -t myapp:latest .` — `-t` tags the resulting image, `.` specifies the build context (directory containing the Dockerfile and files to copy in).

Q: How do you run a Docker container?
Ans: `docker run <image>` (add `-d` for detached/background, `-p host:container` to publish a port, `--name` to name it, `-e` for environment variables).

Q: Docker run command?
Ans: `docker run -d -p 8080:80 --name myapp myimage:latest`.

Q: Difference between -p and -P?
Ans: `-p host_port:container_port` maps a *specific* host port to a container port. `-P` (capital) automatically publishes *all* ports the image `EXPOSE`s to random, ephemeral host ports.

Q: How do you expose ports in Docker (EXPOSE vs -p)?
Ans: `EXPOSE` in the Dockerfile is purely **documentation/metadata** — it doesn't actually publish anything by itself. Actual port publishing (making a container port reachable from the host) happens at `docker run` time via `-p host:container` (or `-P` to auto-map all `EXPOSE`d ports).

Q: How do you pass environment variables to a container?
Ans: `docker run -e KEY=value myimage`, or `--env-file .env` to load multiple variables from a file, or `environment:` in a Docker Compose file.

Q: How do you access an application running in Docker?
Ans: Via the published port on the host (`http://localhost:<host_port>` if run with `-p host:container`), or by `docker exec -it <container> <shell>` to get an interactive shell inside the container itself.

Q: How do you check container resource usage?
Ans: `docker stats` (live CPU/memory/network/I/O usage per container), or `docker stats <container_name>` for a specific one.

Q: How do you inspect logs of a running container?
Ans: `docker logs <container>` (add `-f` to follow/tail in real time, `--tail N` for the last N lines).

Q: How do you debug a restarting container?
Ans: Check `docker logs <container>` for the crash reason first; `docker inspect <container>` for exit code/restart count; if it exits too fast to inspect while running, override the entrypoint to drop into a shell instead (`docker run -it --entrypoint sh myimage`) to poke around manually.

Q: How do you troubleshoot a Docker container that keeps crashing?
Ans: Check `docker logs` for the actual error/stack trace, `docker inspect` for the exit code (e.g., 137 = OOM-killed, 1 = general app error), verify required environment variables/config/volumes are correctly provided, check resource limits (is it being OOM-killed?), and try running the image interactively with a shell entrypoint to reproduce and debug manually.

Q: What is the Docker container lifecycle (states)?
Ans: Created → Running → Paused (optional) → Stopped/Exited → Removed. A container can also go directly from Running to Exited (on crash or normal exit), and Docker's restart policies (`--restart`) can automatically transition it from Exited back to Running.

Q: How do you delete all stopped containers?
Ans: `docker container prune` (prompts for confirmation; add `-f` to skip it).

Q: How do you remove all unused Docker images, containers, and networks?
Ans: `docker system prune` (add `-a` to also remove all unused images, not just dangling ones, and `--volumes` to include unused volumes too).

Q: Can we create a Docker image without a Dockerfile?
Ans: Yes — `docker commit <container> <new_image_name>` creates an image from a running/stopped container's current state, capturing any manual changes made inside it. (Not generally recommended for reproducible builds — a Dockerfile is preferred for anything beyond quick, throwaway experimentation.)

## Docker Compose & Orchestration

Q: What is Docker Compose?
Ans: A tool for defining and running multi-container applications using a single YAML file (`docker-compose.yml`), specifying services, networks, and volumes — start/stop the whole stack with one command (`docker compose up`/`down`).

Q: How do you deploy multiple microservices using Docker Compose?
Ans: Define each microservice as a separate `service` in `docker-compose.yml` (image/build context, ports, environment, dependencies via `depends_on`), put them on a shared Compose network so they can reach each other by service name, then run `docker compose up -d` to start the whole stack together.

Q: What is `depends_on` in Docker Compose?
Ans: Declares startup ordering between services — Compose starts the depended-upon service first. Note that by default it only waits for the container to *start*, not for the application inside to be actually *ready* (use a healthcheck + `condition: service_healthy` for true readiness-based ordering).

Q: Difference between Docker Swarm and Kubernetes?
Ans: **Docker Swarm** is Docker's built-in, simpler orchestrator — easier to set up, fewer features, smaller ecosystem. **Kubernetes** is far more feature-rich (advanced scheduling, auto-scaling, self-healing, huge ecosystem/CRDs/operators) but has a steeper learning curve and more operational complexity — Kubernetes is the dominant choice for production at scale today.

## Networking

Q: What are Docker network types?
Ans: Bridge (default, isolated per-host virtual network), Host (shares the host's network namespace directly), Overlay (multi-host networking for Swarm/clusters), None (no networking), and Macvlan (assigns a container its own MAC address, appearing as a physical device on the network).

Q: What is Bridge Network?
Ans: The default network driver — creates a private internal network on the host; containers on the same bridge network can communicate with each other by container name/IP, and reach the outside world via NAT through the host.

Q: What is Host Network?
Ans: The container shares the host's network namespace directly — no network isolation/NAT, the container uses the host's IP and ports directly (better performance, but no port mapping needed/possible and reduced isolation).

Q: What is Overlay Network?
Ans: A network driver spanning multiple Docker hosts (used in Swarm/multi-host setups), letting containers on different physical machines communicate as if on the same local network, using an encapsulation protocol (VXLAN) under the hood.

## Volumes & Persistence

Q: What are Docker Volumes?
Ans: Docker-managed persistent storage that lives outside the container's writable layer (stored under Docker's storage area on the host, or a remote/plugin-backed location), surviving container removal and shareable between containers.

Q: Types of Docker Volumes?
Ans: **Named volumes** (Docker-managed, referenced by name, most portable/recommended), **anonymous volumes** (Docker-managed but unnamed, auto-generated ID, cleaned up more easily lost track of), and **bind mounts** (map a specific host path directly into the container — not "managed" by Docker, tied to host filesystem layout).

Q: What is a Bind Mount, and how does it differ from a Volume?
Ans: A bind mount maps an arbitrary, specific path on the host filesystem directly into the container — full control over host location, but tightly coupled to that host's directory structure (less portable). A (named) Volume is managed entirely by Docker in its own storage area, independent of host filesystem layout, and more portable across environments/hosts.

Q: Can we persist Docker containers? If yes, how?
Ans: Container *state* itself isn't meant to persist (containers are designed to be ephemeral/disposable) — but *data* can be persisted independently of the container's lifecycle using volumes or bind mounts, so data survives even if the container is removed and recreated.

Q: How do you share files between two containers running on different EC2 instances?
Ans: Containers on different hosts can't share a local Docker volume directly — use a shared network filesystem (EFS mounted on both instances, then bind-mounted into each container), a shared object store (S3), or an application-level file transfer (SCP/rsync, or an API) between them.

## Security & Troubleshooting

Q: What are Container Exit Codes?
Ans: Numeric codes a container returns on exit: `0` = success, `1` = general application error, `137` = SIGKILL (often OOM-killed by the kernel), `143` = SIGTERM (graceful termination requested), `126`/`127` = command not executable/not found. Useful for quickly diagnosing why a container stopped.

Q: What is docker inspect?
Ans: `docker inspect <container|image|volume|network>` returns detailed low-level JSON metadata — config, mounts, network settings, state, exit code, environment variables — useful for debugging.

Q: How do you secure Docker containers?
Ans: Run as a non-root user inside the container, use minimal base images to reduce attack surface, scan images for vulnerabilities (Trivy), avoid mounting the Docker socket into containers, drop unnecessary Linux capabilities (`--cap-drop`), use read-only root filesystems where possible, keep the Docker daemon/host patched, and avoid `--privileged` mode unless absolutely necessary.

Q: If Trivy finds vulnerabilities in an image, how do you fix them?
Ans: Update the affected package/base image to a patched version if available, rebuild and re-scan to confirm; if no fix exists yet and the CVE isn't actually exploitable in context, document and explicitly suppress it (`.trivyignore`) with justification rather than ignoring it silently.

## Practical Dockerfiles

Q: Write a NodeJS Dockerfile.
Ans:
```dockerfile
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY . .

FROM node:20-alpine
WORKDIR /app
COPY --from=build /app .
EXPOSE 3000
CMD ["node", "index.js"]
```

Q: Write an Nginx Dockerfile.
Ans:
```dockerfile
FROM nginx:alpine
COPY ./dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

Q: How would you create a Dockerfile to configure an Nginx web server?
Ans: Start `FROM nginx:alpine` (or another Nginx base), `COPY` in a custom `nginx.conf` and/or site config to `/etc/nginx/conf.d/`, `COPY` static content into the configured web root, `EXPOSE 80` (and 443 if TLS-terminated at Nginx), and let the base image's default `CMD ["nginx", "-g", "daemon off;"]` run the server.

Q: What is the meaning of `daemon off;` and `-g` in Nginx?
Ans: Nginx normally daemonizes (forks into the background); `daemon off;` keeps it running in the foreground, which is required in Docker since the container exits once its main (PID 1) process exits — a backgrounded Nginx would let the container think it's done and stop immediately. `-g` passes a global directive (like `daemon off;`) on the command line without needing to edit the config file.

## Windows Containers

Q: Can .NET applications run on ECS?
Ans: Yes — via EC2 launch type using Windows container instances, or Windows Fargate — both support running .NET (Framework or Core/cross-platform) containers.

Q: What base image would you use for a .NET application?
Ans: For .NET (Core/5+, cross-platform): official `mcr.microsoft.com/dotnet/aspnet` (runtime) or `mcr.microsoft.com/dotnet/sdk` (for build stages) Linux-based images. For legacy .NET Framework (Windows-only): `mcr.microsoft.com/dotnet/framework/aspnet` Windows Server Core-based images.

Q: How do you containerize Windows applications?
Ans: Use a Windows-based base image (`mcr.microsoft.com/windows/servercore` or `mcr.microsoft.com/dotnet/framework/*` for .NET Framework apps), build/run on a Windows Docker host (Windows containers require a Windows kernel — they cannot run on a Linux host), and follow the same Dockerfile pattern (copy app, set entrypoint) adapted to Windows paths/shell (PowerShell/cmd) syntax.

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
