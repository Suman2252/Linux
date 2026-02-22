# Docker Cheatsheet

## Images
| Command | Description |
|---------|-------------|
| `docker pull image:tag` | Download image |
| `docker images` | List images |
| `docker build -t name:tag .` | Build from Dockerfile |
| `docker rmi image` | Remove image |
| `docker image prune -a` | Remove unused |
| `docker tag src dst` | Tag image |
| `docker push image:tag` | Push to registry |

## Containers
| Command | Description |
|---------|-------------|
| `docker run -d image` | Run detached |
| `docker run -it image bash` | Interactive shell |
| `docker run -d -p 8080:80 image` | Port mapping |
| `docker run -d -v vol:/data image` | Volume mount |
| `docker run --name web image` | Named container |
| `docker run --rm image` | Auto-remove |
| `docker ps` | Running containers |
| `docker ps -a` | All containers |
| `docker stop name` | Stop container |
| `docker start name` | Start container |
| `docker restart name` | Restart |
| `docker rm name` | Remove container |
| `docker exec -it name bash` | Exec into running |
| `docker logs -f name` | Follow logs |
| `docker inspect name` | Detailed info |

## Resource Limits
| Flag | Description |
|------|-------------|
| `--memory=512m` | Memory limit |
| `--cpus=1.5` | CPU limit |
| `--restart=unless-stopped` | Restart policy |

## Volumes
| Command | Description |
|---------|-------------|
| `docker volume create vol` | Create volume |
| `docker volume ls` | List volumes |
| `docker volume rm vol` | Remove volume |
| `-v vol:/path` | Named volume |
| `-v /host:/container` | Bind mount |
| `-v /host:/container:ro` | Read-only |

## Networks
| Command | Description |
|---------|-------------|
| `docker network create net` | Create network |
| `docker network ls` | List networks |
| `--network net` | Connect to network |
| `docker network inspect net` | Network details |

## Docker Compose
| Command | Description |
|---------|-------------|
| `docker compose up -d` | Start services |
| `docker compose down` | Stop + remove |
| `docker compose down -v` | + Remove volumes |
| `docker compose ps` | List services |
| `docker compose logs -f` | Follow logs |
| `docker compose build` | Build images |
| `docker compose exec svc bash` | Exec into service |
| `docker compose pull` | Pull latest |

## Cleanup
| Command | Description |
|---------|-------------|
| `docker system prune` | Remove unused |
| `docker system prune -a --volumes` | Remove ALL unused |
| `docker container prune` | Remove stopped |
| `docker system df` | Disk usage |
