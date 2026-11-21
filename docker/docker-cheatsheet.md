# Docker Cheatsheet

Everyday commands for building, running, and debugging containers.

## 1. Images

```bash
docker images
docker pull alpine:3.20
docker build -t myapp:dev .
docker build --no-cache -t myapp:dev .
docker tag myapp:dev registry.example.com/myapp:1.0
docker push registry.example.com/myapp:1.0
docker rmi myapp:dev
```

Inspect:

```bash
docker image inspect alpine:3.20
docker history myapp:dev
```

## 2. Containers

```bash
docker run --rm -it alpine sh
docker run -d --name web -p 8080:80 nginx:alpine
docker ps
docker ps -a
docker logs -f web
docker exec -it web sh
docker stop web
docker start web
docker restart web
docker rm web
```

Useful flags:

- `--rm` — remove when it exits
- `-d` — detached
- `-it` — interactive TTY
- `-p host:container` — publish a port
- `-v host:container` — bind mount
- `--name` — stable name
- `--env-file .env` — environment variables
- `--network` — attach to a network

## 3. Volumes

```bash
docker volume create data
docker volume ls
docker volume inspect data
docker run --rm -v data:/data alpine sh -c 'echo hi > /data/x'
docker volume rm data
```

Bind mount vs volume:

- bind mount: `-v $(pwd):/app` — host path, good for development
- named volume: `-v data:/var/lib/postgresql/data` — managed by Docker, good for data

## 4. Networks

```bash
docker network ls
docker network create appnet
docker network inspect appnet
docker run -d --name db --network appnet postgres:16
docker run --rm --network appnet curlimages/curl http://db:5432
```

Containers on the same user-defined network can resolve each other by name.

## 5. Logs and debugging

```bash
docker logs --tail 100 web
docker logs -f --since 10m web
docker stats
docker top web
docker inspect web
docker diff web
```

Copy files:

```bash
docker cp web:/etc/nginx/nginx.conf .
docker cp ./index.html web:/usr/share/nginx/html/
```

## 6. Cleanup

```bash
docker system df
docker system prune
docker system prune -a --volumes
docker container prune
docker image prune
docker volume prune
docker network prune
```

`prune -a --volumes` is destructive. Check what will be removed first.

## 7. Compose

```bash
docker compose up -d
docker compose up --build
docker compose ps
docker compose logs -f
docker compose exec web sh
docker compose down
docker compose down -v
```

Minimal `compose.yaml`:

```yaml
services:
  web:
    build: .
    ports:
      - "8080:80"
    environment:
      APP_ENV: dev
    depends_on:
      - db
  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: example
    volumes:
      - pgdata:/var/lib/postgresql/data
volumes:
  pgdata:
```

## 8. Dockerfile tips

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
USER 10001
CMD ["python", "-m", "app"]
```

- pin base image versions
- copy dependency files before the source for better layer caching
- run as a non-root user
- use `.dockerignore` to keep the build context small
- avoid secrets in build args; they persist in image history

## 9. Security checks

```bash
docker scout cves myapp:dev
trivy image myapp:dev
docker inspect --format '{{.Config.User}}' myapp:dev
```

Check for:

- running as root
- privileged mode
- mounted Docker socket
- host network mode
- writable root filesystem
- secrets in environment variables or layers

## 10. Common fixes

**Port already in use**: `docker ps` and change the host port.

**Permission denied on a bind mount**: match the container UID/GID or use a named volume.

**Container exits immediately**: check `docker logs <name>` and the entrypoint.

**No space left**: `docker system df`, then prune carefully.

**DNS issues**: `docker run --rm alpine nslookup example.com`; check `/etc/resolv.conf`.
