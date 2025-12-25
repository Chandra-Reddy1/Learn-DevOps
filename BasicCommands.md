# Essential Docker Commands for Daily Use

## Image Management

### Build an Image
```bash
# Build from Dockerfile in current directory
docker build -t my-app:latest .

# Build with a specific Dockerfile
docker build -f Dockerfile.dev -t my-app:dev .

# Build without cache
docker build --no-cache -t my-app:latest .
```

### List Images
```bash
# List all images
docker images

# List images with specific name
docker images my-app

# Show all images including intermediate
docker images -a
```

### Remove Images
```bash
# Remove a specific image
docker rmi image_name:tag

# Remove image by ID
docker rmi abc123def456

# Force remove
docker rmi -f image_name

# Remove all unused images
docker image prune

# Remove all images
docker rmi $(docker images -q)
```

### Pull/Push Images
```bash
# Pull image from Docker Hub
docker pull python:3.11-slim

# Push image to Docker Hub
docker push username/my-app:latest

# Tag an image
docker tag my-app:latest username/my-app:v1.0
```

---

## Container Management

### Run Containers
```bash
# Basic run
docker run image_name

# Run with custom name
docker run --name my-container my-app

# Run in detached mode (background)
docker run -d my-app

# Run with port mapping
docker run -p 8080:80 my-app

# Run with environment variables
docker run -e DB_HOST=localhost -e DB_PORT=5432 my-app

# Run with volume mount
docker run -v /host/path:/container/path my-app

# Run interactively with terminal
docker run -it my-app /bin/bash

# Run and remove container after exit
docker run --rm my-app

# Combined example
docker run -d --name web-app -p 8080:80 -v $(pwd):/app my-app
```

### List Containers
```bash
# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# List container IDs only
docker ps -q

# Show last created container
docker ps -l
```

### Stop/Start/Restart Containers
```bash
# Stop a running container
docker stop container_name

# Stop multiple containers
docker stop container1 container2

# Stop all running containers
docker stop $(docker ps -q)

# Start a stopped container
docker start container_name

# Restart a container
docker restart container_name
```

### Remove Containers
```bash
# Remove a stopped container
docker rm container_name

# Force remove a running container
docker rm -f container_name

# Remove all stopped containers
docker container prune

# Remove all containers
docker rm $(docker ps -aq)
```

---

## Container Interaction

### Execute Commands in Running Container
```bash
# Run a command in running container
docker exec container_name ls /app

# Open interactive bash shell
docker exec -it container_name bash

# Run as specific user
docker exec -u root container_name apt-get update
```

### View Logs
```bash
# View container logs
docker logs container_name

# Follow logs in real-time
docker logs -f container_name

# Show last 100 lines
docker logs --tail 100 container_name

# Show logs with timestamps
docker logs -t container_name
```

### Inspect Container
```bash
# View detailed container info
docker inspect container_name

# Get specific info (e.g., IP address)
docker inspect -f '{{.NetworkSettings.IPAddress}}' container_name
```

### Copy Files
```bash
# Copy from container to host
docker cp container_name:/path/in/container /path/on/host

# Copy from host to container
docker cp /path/on/host container_name:/path/in/container
```

### Monitor Resources
```bash
# View resource usage stats
docker stats

# Stats for specific container
docker stats container_name

# Display only once (no streaming)
docker stats --no-stream
```

---

## Docker Compose Commands

### Basic Operations
```bash
# Start services defined in docker-compose.yml
docker-compose up

# Start in detached mode
docker-compose up -d

# Build images before starting
docker-compose up --build

# Stop services
docker-compose down

# Stop and remove volumes
docker-compose down -v

# View running services
docker-compose ps

# View logs
docker-compose logs

# Follow logs
docker-compose logs -f

# Logs for specific service
docker-compose logs -f web
```

### Service Management
```bash
# Start specific service
docker-compose start service_name

# Stop specific service
docker-compose stop service_name

# Restart service
docker-compose restart service_name

# Execute command in service
docker-compose exec service_name bash

# Scale service
docker-compose up -d --scale web=3
```

---

## System Cleanup

### Clean Up Everything
```bash
# Remove unused data (containers, networks, images)
docker system prune

# Remove everything including volumes
docker system prune -a --volumes

# Show disk usage
docker system df
```

### Specific Cleanup
```bash
# Remove stopped containers
docker container prune

# Remove unused images
docker image prune

# Remove unused volumes
docker volume prune

# Remove unused networks
docker network prune
```

---

## Network Commands

### Manage Networks
```bash
# List networks
docker network ls

# Create network
docker network create my-network

# Remove network
docker network rm my-network

# Connect container to network
docker network connect my-network container_name

# Disconnect container from network
docker network disconnect my-network container_name

# Inspect network
docker network inspect my-network
```

---

## Volume Commands

### Manage Volumes
```bash
# List volumes
docker volume ls

# Create volume
docker volume create my-volume

# Remove volume
docker volume rm my-volume

# Inspect volume
docker volume inspect my-volume

# Remove unused volumes
docker volume prune
```

---

## Useful Aliases (Add to .bashrc or .zshrc)

```bash
# Docker aliases
alias dps='docker ps'
alias dpsa='docker ps -a'
alias di='docker images'
alias dstop='docker stop $(docker ps -q)'
alias drm='docker rm $(docker ps -aq)'
alias drmi='docker rmi $(docker images -q)'
alias dprune='docker system prune -af'
alias dlog='docker logs -f'
alias dexec='docker exec -it'

# Docker Compose aliases
alias dc='docker-compose'
alias dcu='docker-compose up -d'
alias dcd='docker-compose down'
alias dcl='docker-compose logs -f'
alias dcps='docker-compose ps'
```

---

## Quick Reference Cheatsheet

| Task | Command |
|------|---------|
| Build image | `docker build -t name .` |
| Run container | `docker run -d -p 8080:80 name` |
| List running | `docker ps` |
| Stop container | `docker stop name` |
| Remove container | `docker rm name` |
| View logs | `docker logs -f name` |
| Execute command | `docker exec -it name bash` |
| Clean up | `docker system prune -a` |

---

## Common Workflow Example

```bash
# 1. Build your application
docker build -t my-app:latest .

# 2. Run the container
docker run -d --name my-app-container -p 8080:80 my-app:latest

# 3. Check if it's running
docker ps

# 4. View logs
docker logs -f my-app-container

# 5. Access the container
docker exec -it my-app-container bash

# 6. Stop the container
docker stop my-app-container

# 7. Remove the container
docker rm my-app-container

# 8. Clean up unused resources
docker system prune
```

---

## Tips for Daily Use

1. **Always name your containers** - Use `--name` flag for easier management
2. **Use docker-compose for multi-container apps** - Much easier than managing multiple `docker run` commands
3. **Clean up regularly** - Run `docker system prune` to free up disk space
4. **Use .dockerignore** - Exclude unnecessary files from your image
5. **Tag your images properly** - Use meaningful tags like `v1.0`, `latest`, `dev`
6. **Check logs first** - When debugging, always check `docker logs` first
7. **Use volumes for persistent data** - Never store important data inside containers
8. **Keep images small** - Use slim base images and multi-stage builds
