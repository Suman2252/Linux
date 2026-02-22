# 🐳 Chapter 26: Docker & Containers

<p align="center">
  <img src="https://img.shields.io/badge/Level-Expert-red?style=for-the-badge" alt="Expert">
  <img src="https://img.shields.io/badge/Chapter-26%20of%2034-blue?style=for-the-badge" alt="Chapter 26">
</p>

---

## 📑 Table of Contents

- [Containers vs VMs](#containers-vs-vms)
- [Installing Docker](#installing-docker)
- [Image Management](#image-management)
- [Container Management](#container-management)
- [Dockerfile — Building Images](#dockerfile--building-images)
- [Docker Networking](#docker-networking)
- [Docker Volumes](#docker-volumes)
- [Docker Compose](#docker-compose)
- [Container Security](#container-security)
- [Podman — Docker Alternative](#podman--docker-alternative)
- [Practice Exercises](#-practice-exercises)

---

## Containers vs VMs

```
  Virtual Machine                    Container
┌──────────────────┐           ┌──────────────────┐
│    Application   │           │    Application   │
├──────────────────┤           ├──────────────────┤
│    Libraries     │           │    Libraries     │
├──────────────────┤           ├──────────────────┤
│   Guest OS       │           │ Container Engine │
├──────────────────┤           │   (shared host   │
│   Hypervisor     │           │    kernel)       │
├──────────────────┤           ├──────────────────┤
│   Host OS        │           │   Host OS        │
└──────────────────┘           └──────────────────┘
  Heavy (GBs)                    Lightweight (MBs)
  Minutes to start               Seconds to start
```

---

## Installing Docker

```bash
# Ubuntu/Debian
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker $USER        # Run without sudo
newgrp docker                         # Apply group change

# Verify
docker --version
docker run hello-world
```

---

## Image Management

```bash
# Pull images
docker pull ubuntu:22.04
docker pull nginx:latest
docker pull python:3.12-slim

# List images
docker images
docker image ls

# Search Docker Hub
docker search nginx

# Remove images
docker rmi nginx:latest
docker image prune                    # Remove dangling images
docker image prune -a                 # Remove ALL unused images

# Inspect
docker inspect nginx
docker history nginx                  # Show image layers
```

---

## Container Management

```bash
# Run containers
docker run nginx                      # Foreground
docker run -d nginx                   # Detached (background)
docker run -d -p 8080:80 nginx        # Port mapping (host:container)
docker run -d --name web nginx        # Named container
docker run -it ubuntu bash            # Interactive terminal
docker run --rm ubuntu echo "hi"      # Auto-remove after exit

# Manage running containers
docker ps                             # Running containers
docker ps -a                          # All containers (including stopped)
docker stop web                       # Graceful stop
docker kill web                       # Force stop
docker start web                      # Start stopped container
docker restart web                    # Restart

# Interact with containers
docker exec -it web bash              # Shell into running container
docker exec web ls /etc/nginx/        # Run command
docker logs web                       # View logs
docker logs -f web                    # Follow logs
docker logs --tail 50 web             # Last 50 lines

# Resource limits
docker run -d --memory=512m --cpus=1.5 nginx

# Copy files
docker cp file.txt web:/tmp/          # Host → container
docker cp web:/etc/nginx/nginx.conf ./  # Container → host

# Cleanup
docker rm web                         # Remove stopped container
docker container prune                # Remove all stopped
docker system prune -a --volumes      # Nuclear cleanup
```

---

## Dockerfile — Building Images

```dockerfile
# Dockerfile
FROM python:3.12-slim

# Metadata
LABEL maintainer="sovon@example.com"
LABEL version="1.0"

# Set working directory
WORKDIR /app

# Install dependencies (cached layer)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY . .

# Environment variables
ENV FLASK_ENV=production
ENV PORT=5000

# Expose port
EXPOSE 5000

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:5000/health || exit 1

# Run as non-root
RUN useradd -m appuser
USER appuser

# Default command
CMD ["python", "app.py"]
```

### Build & Run

```bash
docker build -t myapp:1.0 .
docker run -d -p 5000:5000 myapp:1.0
```

### Multi-Stage Build (Smaller Images)

```dockerfile
# Build stage
FROM golang:1.21 AS builder
WORKDIR /app
COPY . .
RUN CGO_ENABLED=0 go build -o server .

# Runtime stage
FROM alpine:3.19
COPY --from=builder /app/server /server
EXPOSE 8080
CMD ["/server"]
```

---

## Docker Networking

```bash
# List networks
docker network ls

# Create network
docker network create mynet

# Connect containers
docker run -d --name db --network mynet postgres
docker run -d --name app --network mynet myapp
# 'app' can reach 'db' by hostname: postgres://db:5432

# Network types
# bridge   — Default, isolated per bridge
# host     — Share host network (no isolation)
# none     — No networking
# overlay  — Multi-host (Swarm)
```

---

## Docker Volumes

```bash
# Named volumes (managed by Docker)
docker volume create mydata
docker run -d -v mydata:/var/lib/mysql mysql

# Bind mounts (host directory)
docker run -d -v /host/path:/container/path nginx
docker run -d -v $(pwd)/html:/usr/share/nginx/html nginx

# Read-only
docker run -d -v mydata:/data:ro nginx

# List & cleanup
docker volume ls
docker volume prune
```

---

## Docker Compose

Define multi-container apps in YAML.

```yaml
# docker-compose.yml
services:
  web:
    build: .
    ports:
      - "8080:80"
    environment:
      - DB_HOST=db
    depends_on:
      db:
        condition: service_healthy
    volumes:
      - ./app:/var/www/html
    restart: unless-stopped

  db:
    image: postgres:16
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: secret
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
      interval: 5s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

volumes:
  pgdata:
```

```bash
docker compose up -d              # Start all services
docker compose ps                 # List services
docker compose logs -f web        # Follow web logs
docker compose down               # Stop and remove
docker compose down -v            # Also remove volumes
docker compose build              # Rebuild images
```

---

## Container Security

```bash
# Run as non-root (in Dockerfile)
USER 1000:1000

# Read-only filesystem
docker run --read-only --tmpfs /tmp nginx

# Drop capabilities
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE nginx

# No new privileges
docker run --security-opt=no-new-privileges nginx

# Scan for vulnerabilities
docker scout cves myimage:latest
```

---

## Podman — Docker Alternative

```bash
# Podman is rootless by default (more secure)
sudo apt install podman

# Drop-in Docker replacement (same CLI!)
podman pull nginx
podman run -d -p 8080:80 nginx
podman ps
podman stop <id>

# Generate systemd service from container
podman generate systemd --new --name myweb > ~/.config/systemd/user/myweb.service
systemctl --user enable myweb
```

---

## 🏋️ Practice Exercises

1. **Run**: Pull and run an Nginx container, access it in browser
2. **Dockerfile**: Create a Dockerfile for a simple Python/Node app
3. **Multi-stage**: Build a Go binary using multi-stage builds
4. **Compose**: Create a docker-compose.yml with web + database
5. **Networks**: Create a custom network and connect two containers
6. **Volumes**: Persist a database using named volumes
7. **Security**: Run a container with `--cap-drop=ALL`
8. **Cleanup**: Use `docker system prune` to reclaim space

---

<p align="center">
  <a href="../25-kernel-compilation-modules/README.md">← Previous: Kernel Compilation</a> · <a href="../README.md">🏠 Home</a> · <a href="../27-kubernetes-orchestration/README.md">Next: Kubernetes & Orchestration →</a>
</p>
