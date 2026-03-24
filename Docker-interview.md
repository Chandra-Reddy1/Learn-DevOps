# Docker Interview Questions & Answers (Top 20)

---

## 1. What is Docker?

Docker is a containerization platform used to package applications and their dependencies into containers.

---

## 2. What is a container?

A lightweight, isolated environment that runs an application with all required dependencies.

---

## 3. What is a Docker image?

A read-only template used to create containers.

---

## 4. What is a Dockerfile?

A script with instructions to build a Docker image.

---

## 5. Difference between image and container?

- Image → blueprint  
- Container → running instance of image  

---

## 6. What is Docker Hub?

Public registry to store and share Docker images.

---

## 7. What is a Docker registry?

A storage system for Docker images (e.g., Docker Hub, AWS ECR).

---

## 8. What is a layer in Docker?

Each instruction in Dockerfile creates a layer, improving caching and efficiency.

---

## 9. What is COPY vs ADD?

- COPY → simple file copy  
- ADD → supports URL and extraction  

---

## 10. What is CMD vs ENTRYPOINT?

- CMD → default command (can override)  
- ENTRYPOINT → fixed command  

---

## 11. What is Docker volume?

Used to persist data outside the container.

---

## 12. What is bind mount?

Maps a host directory to container.

---

## 13. What is Docker network?

Enables communication between containers.

Types:
- bridge
- host
- overlay

---

## 14. What is port mapping?

Exposes container port to host:
e.g., -p 8080:80

---

## 15. What is Docker Compose?

Tool to define and run multi-container applications using YAML.

---

## 16. What is multi-stage build?

Uses multiple stages in Dockerfile to reduce final image size.

---

## 17. What is Docker cache?

Reuses layers to speed up builds.

---

## 18. What is container lifecycle?

- create
- start
- stop
- restart
- delete

---

## 19. What is Docker daemon?

Background service that manages containers.

---

## 20. What is difference between Docker and VM?

- Docker → lightweight, shares OS kernel  
- VM → heavy, separate OS  

---

# Bonus (High-Impact DevOps Questions)

---

## 21. How to optimize Docker image?

- Use minimal base image (alpine)
- Use multi-stage build
- Remove unnecessary files
- Combine RUN commands

---

## 22. How to secure Docker container?

- Use non-root user
- Scan images
- Limit permissions

---

## 23. What is Docker logging?

- docker logs
- Central logging (ELK, CloudWatch)

---

## 24. How to handle environment variables?

- Use ENV in Dockerfile
- Pass at runtime

---

## 25. What is health check in Docker?

Checks container health using commands.

---

## 26. What is difference between RUN, CMD, ENTRYPOINT?

- RUN → build time  
- CMD → runtime (default)  
- ENTRYPOINT → fixed execution  

---

## 27. What is Docker swarm?

Native container orchestration tool (less used vs Kubernetes)

---

## 28. What is dangling image?

Unused image without tag.

---

## 29. How to reduce container startup time?

- Smaller image
- Fewer dependencies

---

## 30. How Docker fits in CI/CD?

- Build image → push to registry → deploy to Kubernetes/ECS

---
# Docker Core Interview Explanations

---

# 1. Dockerfile → Image → Container Flow

## Step-by-Step Explanation

### Dockerfile
- A Dockerfile contains instructions to build an image
- Example steps:
  - Define base image (e.g., node, python, alpine)
  - Install dependencies
  - Copy application code
  - Define startup command

---

### Image
- Built using Dockerfile:
  docker build -t my-app .
- Image is a read-only template
- Stored in registry (Docker Hub / ECR)

---

### Container
- Running instance of image:
  docker run -d -p 8080:80 my-app
- Executes application in isolated environment

---

## Flow Summary

Dockerfile → docker build → Image → docker run → Container

---

## One-Line Interview Answer

"We define application setup in a Dockerfile, build it into an image, and run that image as a container to execute the application."

---

# 2. Real Use Case (CI/CD with EKS)

## Scenario
Deploy application automatically using CI/CD pipeline.

---

## Flow

1. Developer pushes code to GitHub

2. CI Pipeline (Jenkins / GitHub Actions)
- Builds Docker image
- Runs tests

3. Push to Registry
- Push image to AWS ECR

4. Deployment to EKS
- Update Kubernetes deployment YAML
- Apply using kubectl or ArgoCD

5. Application Runs
- Pods pull image from ECR
- Application is deployed in EKS

---

## Flow Summary

Code → Build → Docker Image → ECR → EKS Deployment → Pods Running

---

## One-Line Interview Answer

"We use Docker in CI/CD to build images, push them to ECR, and deploy them to EKS, enabling consistent and automated application deployment."

---

# 3. Image Optimization

## Techniques

### Use Lightweight Base Image
- Prefer alpine or slim images

---

### Multi-Stage Build
- Build dependencies in one stage
- Copy only required files to final image

---

### Reduce Layers
- Combine commands:
  RUN apt-get update && apt-get install -y ...

---

### Remove Unnecessary Files
- Delete temp/cache files

---

### Use .dockerignore
- Avoid copying unnecessary files

---

## Example

Bad:
- Large image with unused files

Good:
- Optimized image with only required binaries

---

## One-Line Interview Answer

"We optimize Docker images by using lightweight base images, multi-stage builds, and removing unnecessary files to reduce size and improve performance."

---

# Common Mistakes

- Confusing image and container  
- Not mentioning registry (ECR)  
- Ignoring optimization  
- Giving only theoretical answers  

---

# Final Tip

Always explain in order:
1. How image is built (Dockerfile)
2. Where it is stored (ECR)
3. Where it runs (container / EKS)

