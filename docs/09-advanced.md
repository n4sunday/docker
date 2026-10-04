# 🚀 09 · Advanced Docker

> **Level:** Advanced · [← 08 · Debugging](./08-debugging.md) · [10 · CI/CD & Orchestration →](./10-cicd-and-orchestration.md)

## ⚙️ BuildKit

BuildKit is the default builder in modern Docker. It runs independent stages in parallel and
unlocks extra Dockerfile features. Enable the latest syntax at the top of your Dockerfile:

```dockerfile
# syntax=docker/dockerfile:1
```

### 📦 Cache mounts

Keep package-manager caches **between builds** without baking them into the image:

```dockerfile
# syntax=docker/dockerfile:1
FROM node:22-alpine
WORKDIR /app
COPY package*.json ./
RUN --mount=type=cache,target=/root/.npm \
    npm ci
COPY . .
```

Works the same for `pip` (`/root/.cache/pip`), `apt` (`/var/cache/apt`), Go (`/root/go/pkg/mod`)…

### 🔑 Build secrets

Use a secret during the build **without** leaving it in any layer:

```dockerfile
# syntax=docker/dockerfile:1
RUN --mount=type=secret,id=npmrc,target=/root/.npmrc \
    npm ci
```

```sh
docker build --secret id=npmrc,src=$HOME/.npmrc -t app .
```

## 🌍 Multi-platform images (amd64 + arm64)

Build one tag that works on Intel servers **and** Apple Silicon / AWS Graviton:

```sh
docker buildx create --use --name multi
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t <your-docker-id>/app:1.0 \
  --push .
```

```sh
docker buildx imagetools inspect <your-docker-id>/app:1.0   # see the platforms
```

## 📏 Resource limits

```sh
docker run --memory 512m --memory-swap 512m --cpus 1.5 --pids-limit 200 my-app
docker update --memory 1g --memory-swap 1g <c>   # change limits on a running container
```

```yaml
# compose.yaml
services:
  api:
    deploy:
      resources:
        limits:
          cpus: "1.5"
          memory: 512M
```

> If a container goes over its memory limit, the kernel kills it → exit code `137`.

## 🧟 PID 1 & signals

The first process in a container is **PID 1**, which has special duties: forward signals and
reap zombie processes. Most apps (and `npm`) aren't written for that.

```sh
docker run --init my-app       # adds a tiny init (tini) as PID 1
```

```yaml
services:
  api:
    init: true
    stop_grace_period: 30s     # time between SIGTERM and SIGKILL
```

## 📜 Logging drivers

By default logs are JSON files that **grow forever**. Rotate them:

```sh
docker run --log-opt max-size=10m --log-opt max-file=3 my-app
```

```yaml
services:
  api:
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
```

Other drivers ship logs elsewhere: `local`, `syslog`, `journald`, `fluentd`, `awslogs`, `gcplogs`…

## 🏪 Running your own registry

```sh
docker run -d -p 5000:5000 --name registry -v registry-data:/var/lib/registry registry:2

docker tag my-app localhost:5000/my-app:1.0
docker push localhost:5000/my-app:1.0
docker pull localhost:5000/my-app:1.0
```

Hosted alternatives: GitHub Container Registry (`ghcr.io`), AWS ECR, Google Artifact Registry,
Azure ACR.

## 🖥️ Docker contexts — control remote hosts

```sh
docker context create prod --docker "host=ssh://user@my-server"
docker context use prod
docker ps                     # 👈 now lists containers on my-server
docker context use default
```

## ✅ Checkpoint

- [ ] I use BuildKit cache mounts and build secrets
- [ ] I can publish a multi-platform image
- [ ] My containers have memory limits, an init process and rotated logs

---

[← 08 · Debugging](./08-debugging.md) · [🏠 Home](../README.md) · [10 · CI/CD & Orchestration →](./10-cicd-and-orchestration.md)
