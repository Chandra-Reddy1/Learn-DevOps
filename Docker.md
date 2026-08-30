# Docker — Top 70 Interview Questions
## DevOps Interview Preparation

> **Focus:** Docker basics → images → containers → Dockerfile → networking → volumes → registries → Docker Compose → security → optimization → troubleshooting → production scenarios.
>
> **Interview goal:** Be able to explain *what*, *why*, *how*, and *troubleshooting*, not just memorize commands.

---

# 1. Docker Fundamentals

### 1. What is Docker?
**Answer:** Docker is a platform for packaging and running applications in isolated, portable environments called containers.

### 2. Why do we use Docker?
**Answer:** Docker helps package an application together with its dependencies so it behaves consistently across development, testing, and deployment environments.

Common benefits:
- Portability
- Consistency
- Isolation
- Faster application startup
- Efficient resource usage
- Easier CI/CD
- Easier deployment

### 3. What is a container?
**Answer:** A container is an isolated process running on a host system with its own filesystem, process namespace, networking, and other isolated resources. Containers share the host's operating-system kernel.

### 4. What is a Docker image?
**Answer:** A Docker image is an immutable, layered package containing the filesystem and metadata needed to create and run a container.

### 5. What is the difference between an image and a container?
**Answer:**

```text
Docker Image
    |
    | docker run
    v
Container
```

- **Image:** Template/package used to create containers.
- **Container:** A running or stopped instance created from an image.

### 6. What is a Docker Engine?
**Answer:** Docker Engine is the software that provides the core functionality required to build and run containers. It includes the Docker daemon and related APIs/CLI components.

### 7. What is Docker CLI?
**Answer:** Docker CLI is the command-line interface used to communicate with Docker and perform operations such as building images, running containers, viewing logs, and managing networks.

### 8. What is Docker Hub?
**Answer:** Docker Hub is a hosted container registry service commonly used to store and distribute container images.

### 9. What is a container registry?
**Answer:** A container registry stores and distributes container images.

Examples:
- Docker Hub
- Amazon ECR
- Azure Container Registry
- Google Artifact Registry
- Harbor

### 10. What is the basic Docker workflow?
**Answer:**

```text
Application Code
      |
      v
Dockerfile
      |
      | docker build
      v
Docker Image
      |
      | docker push
      v
Container Registry
      |
      | docker pull
      v
Docker Host
      |
      | docker run
      v
Container
```

---

# 2. Essential Docker Commands

### 11. What does `docker pull` do?
**Answer:** Downloads an image from a container registry to the local Docker image store.

```bash
docker pull nginx:latest
```

### 12. What does `docker run` do?
**Answer:** Creates and starts a container from an image.

```bash
docker run nginx
```

If the image is not available locally, Docker normally attempts to pull it from a configured registry.

### 13. What does `docker ps` do?
**Answer:** Lists running containers.

```bash
docker ps
```

To list all containers, including stopped ones:

```bash
docker ps -a
```

### 14. What does `docker images` do?
**Answer:** Lists local Docker images.

```bash
docker images
```

### 15. What does `docker stop` do?
**Answer:** Gracefully requests a running container to stop.

```bash
docker stop mycontainer
```

### 16. What does `docker start` do?
**Answer:** Starts an existing stopped container.

```bash
docker start mycontainer
```

### 17. What does `docker restart` do?
**Answer:** Restarts a container.

```bash
docker restart mycontainer
```

### 18. What does `docker rm` do?
**Answer:** Removes a container.

```bash
docker rm mycontainer
```

A running container normally must be stopped first unless forced removal is used.

### 19. What does `docker rmi` do?
**Answer:** Removes a local Docker image.

```bash
docker rmi nginx:latest
```

### 20. What does `docker exec` do?
**Answer:** Executes a command inside a running container.

Example:

```bash
docker exec -it mycontainer /bin/sh
```

For images that contain Bash:

```bash
docker exec -it mycontainer /bin/bash
```

---

# 3. Dockerfile

### 21. What is a Dockerfile?
**Answer:** A Dockerfile is a text file containing instructions used to build a Docker image.

Example:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8080

CMD ["python", "app.py"]
```

### 22. What is `FROM` in a Dockerfile?
**Answer:** `FROM` specifies the base image for the build.

```dockerfile
FROM ubuntu:24.04
```

A Dockerfile normally starts with `FROM`, except for special cases such as certain parser directives or `ARG` usage before `FROM`.

### 23. What is `RUN`?
**Answer:** `RUN` executes commands during image build time and creates a new image layer.

Example:

```dockerfile
RUN apt-get update && apt-get install -y curl
```

### 24. What is `CMD`?
**Answer:** `CMD` defines the default command or arguments executed when a container starts.

Example:

```dockerfile
CMD ["python", "app.py"]
```

### 25. What is `ENTRYPOINT`?
**Answer:** `ENTRYPOINT` configures the main executable of a container.

Example:

```dockerfile
ENTRYPOINT ["python", "app.py"]
```

### 26. What is the difference between `CMD` and `ENTRYPOINT`?
**Answer:**

**ENTRYPOINT:**
- Defines the main executable
- Makes the container behave like a specific executable

**CMD:**
- Provides default command or default arguments
- Is easier to override with normal `docker run` command arguments

A common pattern is:

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```

### 27. What is `COPY`?
**Answer:** `COPY` copies files from the Docker build context into the image.

```dockerfile
COPY . /app
```

### 28. What is `ADD`?
**Answer:** `ADD` can copy files into the image and has additional behavior, such as automatic extraction of local tar archives. It can also support certain URL-related behavior depending on usage.

For ordinary file copying, `COPY` is generally clearer and preferred.

### 29. What is `WORKDIR`?
**Answer:** `WORKDIR` sets the working directory for subsequent Dockerfile instructions and the default working directory for the container.

```dockerfile
WORKDIR /app
```

### 30. What is `EXPOSE`?
**Answer:** `EXPOSE` documents the port that the application is expected to listen on inside the container. It does **not** publish the port to the host by itself.

Example:

```dockerfile
EXPOSE 8080
```

To publish it:

```bash
docker run -p 8080:8080 myapp
```

---

# 4. Docker Image Layers and Build

### 31. What is a Docker image layer?
**Answer:** Docker images are built from layers. Many Dockerfile instructions create filesystem layers, and Docker can reuse unchanged layers during subsequent builds.

### 32. Why are Docker image layers useful?
**Answer:** Layers provide:
- Build-cache reuse
- Faster builds
- Storage efficiency
- Image composition

### 33. What is Docker build cache?
**Answer:** Docker can reuse previously built layers when the relevant Dockerfile instruction and build inputs have not changed.

Good Dockerfile ordering can improve cache efficiency.

### 34. How do you build a Docker image?
**Answer:**

```bash
docker build -t myapp:1.0 .
```

Here:
- `-t` assigns a name/tag
- `.` is the build context

### 35. What is a Docker build context?
**Answer:** The build context is the set of files available to the Docker build process. In:

```bash
docker build -t myapp .
```

the current directory is the build context.

### 36. Why should we use `.dockerignore`?
**Answer:** `.dockerignore` excludes unnecessary files from the build context.

Example:

```text
.git
node_modules
*.log
.env
```

This can:
- Reduce build context size
- Improve build speed
- Prevent unnecessary files from being sent to the builder
- Reduce accidental inclusion of sensitive files

### 37. What is a multi-stage Docker build?
**Answer:** A multi-stage build uses multiple `FROM` instructions so build tools and intermediate files can be separated from the final runtime image.

Example:

```dockerfile
FROM golang:1.24 AS builder
WORKDIR /src
COPY . .
RUN go build -o app .

FROM alpine:3.22
COPY --from=builder /src/app /app
CMD ["/app"]
```

### 38. Why are multi-stage builds useful?
**Answer:** They can produce smaller and cleaner runtime images by excluding compilers, source code, and other build-time dependencies.

### 39. How do you reduce Docker image size?
**Answer:**
- Use a suitable minimal base image
- Use multi-stage builds
- Remove unnecessary packages
- Avoid copying unnecessary files
- Use `.dockerignore`
- Combine appropriate package-manager operations
- Keep only runtime dependencies in the final image

### 40. What is image tagging?
**Answer:** A tag is a human-readable reference associated with an image.

Examples:

```text
myapp:1.0.0
myapp:2026-08-30
myapp:latest
```

For production, immutable version tags or digests are generally safer than relying only on `latest`.

---

# 5. Container Lifecycle

### 41. What happens internally when you run `docker run`?
**Answer:**

Conceptually:

```text
docker run
    |
    +--> Find image locally
    |
    +--> Pull image if required
    |
    +--> Create container
    |
    +--> Configure filesystem/network/resources
    |
    +--> Start container process
```

### 42. What is the main process of a container?
**Answer:** A container normally exists as long as its main process (PID 1 inside the container) is running. If that process exits, the container stops unless another mechanism keeps it running.

### 43. Why does a Docker container immediately exit?
**Answer:** Usually the container's main process exits.

For example:

```bash
docker run ubuntu
```

may exit because the default command finishes immediately.

To troubleshoot:

```bash
docker ps -a
docker logs <container>
docker inspect <container>
```

### 44. What is the difference between `docker run` and `docker start`?
**Answer:**

**`docker run`:**
- Creates a new container from an image
- Starts it

**`docker start`:**
- Starts an existing stopped container

### 45. What is the difference between `docker exec` and `docker run`?
**Answer:**

**`docker run`:** Creates and starts a new container.

**`docker exec`:** Runs a command inside an already running container.

### 46. How do you view container logs?
**Answer:**

```bash
docker logs <container>
```

Follow logs:

```bash
docker logs -f <container>
```

### 47. How do you inspect a container?
**Answer:**

```bash
docker inspect <container>
```

This returns detailed metadata such as configuration, network information, mounts, environment configuration, and runtime state.

### 48. How do you check resource usage of containers?
**Answer:**

```bash
docker stats
```

It displays runtime resource information such as CPU, memory, network, and block I/O usage.

### 49. How do you copy files between host and container?
**Answer:**

Host → container:

```bash
docker cp file.txt mycontainer:/app/file.txt
```

Container → host:

```bash
docker cp mycontainer:/app/output.txt .
```

### 50. What is a restart policy?
**Answer:** A restart policy tells Docker when to automatically restart a container.

Examples:

```bash
docker run --restart=always myapp
docker run --restart=unless-stopped myapp
```

Common policies include:
- `no`
- `on-failure`
- `always`
- `unless-stopped`

---

# 6. Docker Networking

### 51. What is Docker networking?
**Answer:** Docker networking provides communication between containers, the host, and external networks.

Common Docker network drivers include:
- bridge
- host
- none
- overlay
- macvlan

### 52. What is the default bridge network?
**Answer:** Docker provides a default `bridge` network. Containers attached to it can communicate using Docker's networking behavior, but user-defined bridge networks generally provide better service discovery and isolation characteristics.

### 53. What is a user-defined bridge network?
**Answer:** A user-defined bridge network is a custom bridge network that provides better control and automatic DNS-based container name resolution between connected containers.

Example:

```bash
docker network create app-net

docker run -d --name db --network app-net postgres
docker run -d --name app --network app-net myapp
```

The application can typically reach the database using:

```text
db
```

as the hostname on that network.

### 54. What is host networking?
**Answer:** With host networking, the container shares the host's network namespace rather than getting a separate container network namespace.

Example:

```bash
docker run --network host nginx
```

This reduces network isolation and changes how port publishing works.

### 55. What is port mapping?
**Answer:** Port mapping publishes a container port through a host port.

Example:

```bash
docker run -p 8080:80 nginx
```

Meaning:

```text
Host port 8080
      |
      v
Container port 80
```

### 56. What is the difference between `EXPOSE` and `-p`?
**Answer:**

**`EXPOSE`:**
- Documents intended container port
- Does not publish it

**`-p`:**
- Publishes/maps a container port to a host port

Example:

```dockerfile
EXPOSE 80
```

does not make the application reachable from the host by itself.

---

# 7. Docker Volumes and Storage

### 57. Why do we need Docker volumes?
**Answer:** Containers are designed to be replaceable, and data written to the container's writable layer is tied to that container's lifecycle. Volumes provide persistent storage independent of a specific container.

### 58. What is a Docker volume?
**Answer:** A Docker volume is Docker-managed persistent storage that can be mounted into one or more containers.

Example:

```bash
docker volume create app-data

docker run \
  -v app-data:/var/lib/myapp \
  myapp
```

### 59. What is a bind mount?
**Answer:** A bind mount maps a specific host filesystem path into a container.

Example:

```bash
docker run \
  -v /host/config:/app/config \
  myapp
```

### 60. Volume vs bind mount — what is the difference?
**Answer:**

**Volume:**
- Managed by Docker
- Better suited to Docker-managed persistent application data
- Host path is abstracted

**Bind mount:**
- Explicitly maps a host path
- Useful when the host path itself matters, such as local development or specific host configuration

---

# 8. Docker Compose

### 61. What is Docker Compose?
**Answer:** Docker Compose is a tool for defining and running multi-container applications using a YAML configuration file.

Example:

```yaml
services:
  app:
    image: myapp:1.0
    ports:
      - "8080:8080"

  db:
    image: postgres:17
    environment:
      POSTGRES_PASSWORD: example
```

### 62. Why do we use Docker Compose?
**Answer:** It simplifies running applications consisting of multiple related containers, such as:

```text
Application
    |
    +--> Frontend
    +--> Backend
    +--> Database
    +--> Redis
```

### 63. How do you start a Compose application?
**Answer:**

```bash
docker compose up -d
```

### 64. How do you stop a Compose application?
**Answer:**

```bash
docker compose down
```

`down` normally removes the containers and network created by Compose, while named volumes are not removed unless you explicitly request volume removal.

---

# 9. Docker Security

### 65. Why should containers not normally run as root?
**Answer:** Running applications as root inside a container increases the impact of a container compromise. Use a non-root user where practical.

Dockerfile example:

```dockerfile
RUN useradd -r -u 10001 appuser
USER appuser
```

### 66. How do you secure Docker images?
**Answer:**
- Use trusted/minimal base images
- Keep images patched
- Scan images for vulnerabilities
- Avoid embedding secrets
- Run as non-root
- Minimize installed packages
- Pin important dependencies
- Use signed/verified images where appropriate
- Use immutable image references in production where practical

### 67. Should secrets be stored in a Dockerfile?
**Answer:** No.

Bad:

```dockerfile
ENV DB_PASSWORD=mysecret
```

Secrets can become part of image configuration/history and may be exposed.

Use an appropriate secret-management mechanism instead, such as:
- CI/CD secret store
- Docker secrets where applicable
- Cloud secret manager
- Kubernetes Secrets with appropriate security controls

---

# 10. Production Troubleshooting and Scenarios

### 68. A container is running but the application is not reachable. How would you troubleshoot it?
**Answer:**

Use a systematic process:

```text
1. Check container status
       |
       v
2. Check application logs
       |
       v
3. Check application is listening on expected port
       |
       v
4. Check Docker port mapping
       |
       v
5. Check Docker network
       |
       v
6. Test connectivity from host/container
       |
       v
7. Check host firewall/security rules
       |
       v
8. Check application configuration
```

Useful commands:

```bash
docker ps
docker logs <container>
docker inspect <container>
docker port <container>
docker exec -it <container> sh
```

### 69. A Docker container keeps restarting. How would you troubleshoot it?
**Answer:**

First identify why the main process is exiting.

Check:

```bash
docker ps -a
docker logs <container>
docker inspect <container>
docker events
```

Then investigate:
- Application crash
- Invalid configuration
- Missing environment variables
- Dependency/service failure
- Permission issues
- Health-check behavior
- Resource limits
- Restart policy

Do not simply disable the restart policy; identify the underlying failure.

### 70. Design a production-grade Docker CI/CD deployment architecture.
**Answer:**

A strong design is:

```text
Developer
   |
   v
Git Repository
   |
   v
CI Pipeline
   |
   +--> Checkout
   +--> Build
   +--> Unit Tests
   +--> Security / Dependency Scan
   +--> Docker Build
   +--> Image Scan
   |
   v
Container Registry
   |
   | Immutable version/tag or digest
   v
Deployment Platform
   |
   +--> Staging
   |
   +--> Smoke Tests
   |
   +--> Approval / Automated Gate
   |
   +--> Production
   |
   v
Health Checks
   |
   +--> Success
   |
   +--> Failure --> Rollback
```

Production principles:
- Build once and promote the same image across environments.
- Store images in a controlled registry.
- Prefer immutable version tags or image digests.
- Scan images before deployment.
- Use minimal runtime images.
- Run containers as non-root where possible.
- Do not embed secrets in images.
- Apply CPU/memory limits appropriate to the platform.
- Implement health checks.
- Use centralized logging and monitoring.
- Have a rollback strategy.
- Keep base images and dependencies patched.
- Use a proper orchestrator such as Kubernetes/ECS when application scale and availability requirements justify it.

---

# High-Priority Questions to Master First

If your interview is in the next few days, prioritize these:

## Docker Fundamentals
1. What is Docker?
2. Why Docker?
3. Container vs image
4. Docker Engine
5. Docker workflow
6. Docker registry
7. Docker Hub

## Commands
8. `docker pull`
9. `docker run`
10. `docker ps`
11. `docker exec`
12. `docker logs`
13. `docker inspect`
14. `docker stop` vs `docker start`
15. `docker run` vs `docker exec`

## Dockerfile
16. Dockerfile
17. `FROM`
18. `RUN`
19. `CMD`
20. `ENTRYPOINT`
21. CMD vs ENTRYPOINT
22. `COPY`
23. `ADD`
24. `WORKDIR`
25. `EXPOSE`

## Images
26. Image layers
27. Build cache
28. Build context
29. `.dockerignore`
30. Multi-stage builds
31. Image size optimization
32. Image tagging

## Containers
33. Container lifecycle
34. Main container process
35. Why containers exit
36. Restart policies
37. Resource monitoring

## Networking
38. Docker networking
39. Bridge network
40. User-defined bridge
41. Host network
42. Port mapping
43. `EXPOSE` vs `-p`

## Storage
44. Volumes
45. Bind mounts
46. Volume vs bind mount

## Compose
47. Docker Compose
48. Compose services
49. `docker compose up`
50. `docker compose down`

## Security
51. Non-root containers
52. Image security
53. Secrets
54. Vulnerability scanning

## Troubleshooting
55. Application not reachable
56. Container restarting
57. Container exits immediately
58. Image build failure
59. Docker network issue
60. Disk/resource issue

## Production
61. CI/CD with Docker
62. Registry
63. Immutable images
64. Build once/promote
65. Image scanning
66. Health checks
67. Logging/monitoring
68. Resource limits
69. Rollback
70. Docker + Kubernetes/ECS architecture

---

# Essential Docker Command Cheat Sheet

```bash
# Images
docker images
docker pull nginx:latest
docker build -t myapp:1.0 .
docker rmi myapp:1.0

# Containers
docker run -d --name myapp myapp:1.0
docker ps
docker ps -a
docker start myapp
docker stop myapp
docker restart myapp
docker rm myapp

# Debugging
docker logs myapp
docker logs -f myapp
docker inspect myapp
docker stats
docker exec -it myapp sh
docker port myapp

# Copy files
docker cp file.txt myapp:/app/
docker cp myapp:/app/output.txt .

# Networks
docker network ls
docker network create app-net
docker network inspect app-net

# Volumes
docker volume ls
docker volume create app-data
docker volume inspect app-data

# Registry
docker login
docker tag myapp:1.0 registry.example.com/myapp:1.0
docker push registry.example.com/myapp:1.0
docker pull registry.example.com/myapp:1.0

# Compose
docker compose up -d
docker compose ps
docker compose logs
docker compose down
```

---

# Docker Troubleshooting Framework

When an interviewer gives you a Docker production problem, avoid jumping directly to a command.

Use:

```text
1. Identify the container/image
          |
          v
2. Check current state
   docker ps -a
          |
          v
3. Check logs
   docker logs
          |
          v
4. Inspect configuration
   docker inspect
          |
          v
5. Check networking
          |
          v
6. Check ports
          |
          v
7. Check environment/configuration
          |
          v
8. Check resources
          |
          v
9. Reproduce the issue
          |
          v
10. Fix root cause and verify
```

---

# Important Interview Comparisons

## Image vs Container

```text
Image = Template / Package
Container = Running or stopped instance
```

## Dockerfile vs Docker Image

```text
Dockerfile
    |
    | docker build
    v
Docker Image
```

## Volume vs Bind Mount

```text
Volume
  |
  +--> Docker-managed storage

Bind Mount
  |
  +--> Specific host path
```

## `CMD` vs `ENTRYPOINT`

```text
ENTRYPOINT = Main executable
CMD        = Default command/arguments
```

## `docker run` vs `docker start`

```text
docker run
  = Create + Start new container

docker start
  = Start existing stopped container
```

## `EXPOSE` vs `-p`

```text
EXPOSE
  = Documentation/metadata

-p
  = Actual host-to-container port publishing
```

---

# Final Interview Preparation Strategy

For every important Docker question, practice answering in this order:

```text
1. What is it?
2. Why do we use it?
3. How does it work?
4. Give a command/example
5. What can go wrong?
6. How would you troubleshoot it?
```

For scenario questions, always show a sequence.

For example:

> **"The container is running but the application is not accessible. What will you do?"**

A strong answer should sound like:

```text
First I will check whether the container is actually running
and inspect its status.

Then I will check the application logs and verify that the
application is listening on the expected port.

Next I will check the Docker port mapping using docker ps
or docker port.

Then I will inspect the Docker network and test connectivity
from the host or another container.

Finally I will check firewall/security rules and application
configuration.

I will identify the root cause first rather than simply
restarting the container.
```

---

# Final Must-Know Docker Concepts

```text
Docker
 |
 +-- Docker Engine
 |
 +-- Docker CLI
 |
 +-- Image
 |     +-- Layers
 |     +-- Tags
 |     +-- Digest
 |
 +-- Container
 |     +-- Process
 |     +-- Filesystem
 |     +-- Network
 |     +-- Resources
 |
 +-- Dockerfile
 |     +-- FROM
 |     +-- RUN
 |     +-- COPY
 |     +-- WORKDIR
 |     +-- EXPOSE
 |     +-- CMD
 |     +-- ENTRYPOINT
 |
 +-- Network
 |     +-- Bridge
 |     +-- Host
 |     +-- Overlay
 |
 +-- Storage
 |     +-- Volume
 |     +-- Bind Mount
 |
 +-- Registry
 |     +-- Docker Hub
 |     +-- ECR
 |     +-- ACR
 |     +-- Artifact Registry
 |
 +-- Compose
 |
 +-- Security
 |
 +-- CI/CD
 |
 +-- Troubleshooting
```

## The most important production principle

A good Docker interview answer should show that you understand:

```text
Code
  ↓
Build
  ↓
Test
  ↓
Secure
  ↓
Build Image
  ↓
Scan Image
  ↓
Push Immutable Image
  ↓
Deploy
  ↓
Health Check
  ↓
Monitor
  ↓
Rollback if required
```

Do not treat Docker as only a collection of commands. In a DevOps interview, the interviewer is usually testing whether you understand how containers behave in the complete software-delivery lifecycle.
