# 📄 03 · Dockerfile Deep Dive

> **Level:** Beginner → Intermediate · [← 02 · Images](./02-images.md) · [04 · Data & Volumes →](./04-data-and-volumes.md)

## 📋 All the instructions

| Instruction | Purpose | Example |
| :-- | :-- | :-- |
| `FROM` | Base image (starts a new stage) | `FROM node:22-alpine AS build` |
| `WORKDIR` | Set & create the working directory | `WORKDIR /app` |
| `COPY` | Copy files from the build context | `COPY package*.json ./` |
| `ADD` | Like `COPY`, plus URLs and auto-extracting tar files | `ADD app.tar.gz /opt/` |
| `RUN` | Run a command at **build** time | `RUN npm ci` |
| `CMD` | Default command at **run** time (easy to override) | `CMD ["node", "server.js"]` |
| `ENTRYPOINT` | Fixed executable at run time | `ENTRYPOINT ["nginx"]` |
| `ENV` | Environment variable (build **and** run time) | `ENV NODE_ENV=production` |
| `ARG` | Build-time variable only | `ARG VERSION=1.0` |
| `EXPOSE` | Document which port the app listens on | `EXPOSE 3000` |
| `USER` | Run following steps / the container as this user | `USER node` |
| `VOLUME` | Declare a mount point for persistent data | `VOLUME /data` |
| `LABEL` | Add metadata | `LABEL org.opencontainers.image.source="https://github.com/n4sunday/docker"` |
| `HEALTHCHECK` | How Docker checks the app is healthy | see below |
| `SHELL` | Change the shell used by shell-form commands | `SHELL ["/bin/bash", "-c"]` |

> [!NOTE]
> `EXPOSE` does **not** publish the port — you still need `-p` when running.

## 🐚 Shell form vs exec form

```dockerfile
CMD npm start              # shell form → runs as: /bin/sh -c "npm start"
CMD ["npm", "start"]       # exec form  → runs npm directly ✅
```

Prefer **exec form** for `CMD` and `ENTRYPOINT`: your process becomes PID 1 and receives
signals like `SIGTERM` directly, so `docker stop` shuts it down gracefully.

## 🎯 `CMD` vs `ENTRYPOINT`

```dockerfile
FROM alpine
ENTRYPOINT ["ping"]
CMD ["-c", "3", "localhost"]
```

```sh
docker run pinger                 # → ping -c 3 localhost
docker run pinger -c 1 google.com # → ping -c 1 google.com  (CMD replaced)
docker run --entrypoint sh -it pinger   # override the entrypoint
```

| | `CMD` | `ENTRYPOINT` |
| :-- | :-- | :-- |
| Role | Default command / default arguments | The executable |
| Override with | Arguments after the image name | `--entrypoint` |
| Use when | General-purpose images | The image *is* a single tool |

## 🔧 `ARG` vs `ENV`

```dockerfile
ARG NODE_VERSION=22
FROM node:${NODE_VERSION}-alpine

ARG APP_VERSION=dev          # only during the build
ENV APP_VERSION=${APP_VERSION}   # copy it into the runtime env
ENV PORT=3000
```

```sh
docker build --build-arg APP_VERSION=1.2.3 -t app .
docker run -e PORT=8080 app      # ENV can be overridden at run time
```

> [!CAUTION]
> Never pass secrets via `ARG` or `ENV` — they are stored in the image history.
> Use [build secrets](./09-advanced.md#-build-secrets) instead.

## 🙈 `.dockerignore`

Everything in the build folder (the **build context**) is sent to the daemon. Exclude what you
don't need — builds get faster and you avoid leaking files into the image.

```gitignore
node_modules
dist
.git
.env
*.log
Dockerfile*
docker-compose*.yml
```

## ❤️ `HEALTHCHECK`

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
  CMD wget -qO- http://localhost:3000/health || exit 1
```

`docker ps` will then show `(healthy)` or `(unhealthy)`, and Compose can wait for it
([see 06](./06-docker-compose.md#-startup-order-with-healthchecks)).

## ⚡ Cache-friendly ordering

```dockerfile
FROM node:22-alpine
WORKDIR /app

# 1️⃣ Rarely changes → cached
COPY package.json package-lock.json ./
RUN npm ci

# 2️⃣ Changes on every commit
COPY . .

CMD ["npm", "start"]
```

Rules of thumb:

- Once a layer changes, **every layer after it** is rebuilt
- Put the slowest, least-changing steps first
- Combine related `RUN` commands and clean up in the **same** layer:

```dockerfile
RUN apt-get update \
 && apt-get install -y --no-install-recommends curl \
 && rm -rf /var/lib/apt/lists/*
```

## 🏗️ A production-ready Node.js Dockerfile

```dockerfile
# syntax=docker/dockerfile:1

# ---------- deps ----------
FROM node:22-alpine AS deps
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci --omit=dev

# ---------- build ----------
FROM node:22-alpine AS build
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci
COPY . .
RUN npm run build

# ---------- runtime ----------
FROM node:22-alpine
ENV NODE_ENV=production
WORKDIR /app
COPY --from=deps  --chown=node:node /app/node_modules ./node_modules
COPY --from=build --chown=node:node /app/dist ./dist
COPY --chown=node:node package.json ./
USER node
EXPOSE 3000
HEALTHCHECK CMD wget -qO- http://localhost:3000/health || exit 1
CMD ["node", "dist/server.js"]
```

✅ pinned base · ✅ multi-stage · ✅ prod-only deps · ✅ non-root · ✅ healthcheck

## ✅ Checkpoint

- [ ] I know when to use `CMD` vs `ENTRYPOINT`, `ARG` vs `ENV`
- [ ] My Dockerfiles use exec form and cache-friendly ordering
- [ ] I have a `.dockerignore` in every project

---

[← 02 · Images](./02-images.md) · [🏠 Home](../README.md) · [04 · Data & Volumes →](./04-data-and-volumes.md)
