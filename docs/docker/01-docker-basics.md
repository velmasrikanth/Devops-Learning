# Docker — Basics

## What is Docker
Docker is a platform to package applications and dependencies into lightweight, portable containers that run consistently across environments.

## Key concepts
- Image: Read-only template with application and environment.
- Container: Runnable instance of an image (isolated process).
- Dockerfile: Text file that defines how to build an image.
- Registry: Remote storage (Docker Hub, private registry) for images.
- Volume: Persistent storage mounted into containers.
- Network: Container communication layers (bridge, host, overlay).

## Simple Dockerfile
```dockerfile
# Dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install --production
COPY . .
CMD ["node", "index.js"]
EXPOSE 3000
```

## Common commands
- Build: docker build -t myapp:1.0 .
- Run: docker run --rm -d --name myapp -p 3000:3000 myapp:1.0
- List containers: docker ps
- List images: docker images
- Stop/remove: docker stop myapp && docker rm myapp
- Remove image: docker rmi myapp:1.0
- Logs: docker logs -f myapp
- Exec into running container: docker exec -it myapp sh
- Pull/push: docker pull imagename, docker push repo/imagename

## Volumes and persistence
- Create/run with volume: docker run -v myvol:/data ...
- Bind mount host dir: -v /host/path:/container/path
- Use volumes for databases and data that must outlive containers.

## Networking
- Default bridge: isolated network with NAT.
- Expose and map ports: -p hostPort:containerPort
- Create custom bridge: docker network create mynet
- Use overlay networks for multi-host Swarm/Kubernetes uses.

## Docker Compose (multi-container)
docker-compose.yml minimal example:
```yaml
version: "3.8"
services:
    web:
        build: .
        ports:
            - "3000:3000"
        volumes:
            - .:/app
    redis:
        image: redis:7-alpine
```
Commands: docker-compose up -d, docker-compose down

## Best practices
- Keep images small (use slim/alpine base images).
- Multi-stage builds to separate build/runtime.
- Pin base image versions.
- Avoid storing secrets in images; use environment variables/secret managers.
- Use .dockerignore to exclude files from build context.

## Security basics
- Run minimal processes and non-root user in containers.
- Scan images for vulnerabilities.
- Limit container privileges (no --privileged, use seccomp/AppArmor profiles).

## Useful resources
- Official docs: https://docs.docker.com
- Dockerfile reference and command docs
