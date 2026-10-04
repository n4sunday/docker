# 🛡️ 07 · Best Practices & Security

> **Level:** Advanced · [← 06 · Docker Compose](./06-docker-compose.md) · [08 · Debugging →](./08-debugging.md)

## ✅ Image checklist

| | Practice | Why |
| :-: | :-- | :-- |
| 📌 | Pin base image versions (`node:22-alpine`, not `node`) | Reproducible builds |
| 🪶 | Use small bases (`-slim`, `-alpine`, distroless) | Faster pulls, smaller attack surface |
| 🏗️ | Use multi-stage builds | Ship only what runs — no compilers or dev deps |
| ⚡ | Order layers from least → most frequently changed | Fast rebuilds |
| 🙈 | Add a `.dockerignore` | Smaller context, no leaked `.env` or `.git` |
| 🧹 | Clean package caches in the same `RUN` | Smaller layers |
| 🎯 | One main process per container | Easier scaling, logging and restarts |
| 🏷️ | Tag with versions or git SHAs, not just `latest` | Know exactly what is deployed |

## 👤 Don't run as root

By default, processes in a container run as **root**. If an attacker escapes your app, they
are root inside the container.

```dockerfile
# Official Node images already include a "node" user
USER node

# Other images: create one
RUN addgroup -S app && adduser -S app -G app
USER app
```

Check it:

```sh
docker run --rm my-app whoami   # should NOT print "root"
```

## 🔐 Harden containers at run time

```sh
docker run \
  --read-only \
  --tmpfs /tmp \
  --cap-drop ALL \
  --security-opt no-new-privileges \
  --memory 256m --cpus 0.5 \
  my-app
```

| Flag | Effect |
| :-- | :-- |
| `--read-only` | Root filesystem is read-only |
| `--tmpfs /tmp` | …except a scratch directory in memory |
| `--cap-drop ALL` | Drop all Linux capabilities (add back only what's needed with `--cap-add`) |
| `--security-opt no-new-privileges` | Block privilege escalation via setuid binaries |
| `--memory` / `--cpus` | Limit resources so one container can't starve the host |

> [!CAUTION]
> Avoid `--privileged` and mounting `/var/run/docker.sock` — both effectively give the
> container full control of the host.

## 🤫 Secrets

| ❌ Don't | ✅ Do |
| :-- | :-- |
| `ENV API_KEY=abc123` in the Dockerfile | Pass at run time: `-e API_KEY` / `env_file` |
| `COPY .env .` | Keep `.env` in `.dockerignore` |
| `ARG NPM_TOKEN` for private packages | [BuildKit build secrets](./09-advanced.md#-build-secrets) |
| Commit secrets to git | Use your platform's secret manager (K8s Secrets, GitHub Actions secrets…) |

## 🔍 Scan images for vulnerabilities

```sh
# Docker Scout (built into Docker Desktop)
docker scout quickview my-app
docker scout cves my-app

# Trivy (open source)
trivy image my-app
```

Run a scanner in CI and rebuild images regularly to pick up base-image security patches.

## 📐 12-Factor friendly containers

- ⚙️ **Config** via environment variables, not baked into the image
- 📜 **Logs** to `stdout` / `stderr` — Docker collects them (`docker logs`)
- 🛑 **Graceful shutdown** — handle `SIGTERM` and close connections
- 🔁 **Stateless** — keep state in databases / volumes so containers can be replaced anytime

```js
// Node.js graceful shutdown
process.on("SIGTERM", () => {
  server.close(() => process.exit(0));
});
```

## ✅ Checkpoint

- [ ] My production images are multi-stage, pinned and run as non-root
- [ ] No secrets end up in image layers
- [ ] Images are scanned in CI

---

[← 06 · Docker Compose](./06-docker-compose.md) · [🏠 Home](../README.md) · [08 · Debugging →](./08-debugging.md)
