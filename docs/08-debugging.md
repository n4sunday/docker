# 🐞 08 · Debugging & Troubleshooting

> **Level:** Advanced · [← 07 · Best Practices & Security](./07-best-practices-and-security.md) · [09 · Advanced Docker →](./09-advanced.md)

## 🧰 The debugging toolbox

| Command | What it tells you |
| :-- | :-- |
| `docker ps -a` | Is it running? What's the exit code? |
| `docker logs -f --tail 100 <c>` | 📜 Latest output, live |
| `docker logs --since 10m <c>` | Output from the last 10 minutes |
| `docker inspect <c>` | 🔍 Full config: env, mounts, networks, IP, health |
| `docker exec -it <c> sh` | 🐚 Shell into a running container |
| `docker stats` | 📊 Live CPU / memory / network usage |
| `docker top <c>` | Processes inside the container |
| `docker diff <c>` | Files changed since the container started |
| `docker events` | Real-time stream of daemon events |
| `docker system df` | 💽 Disk usage by images, containers, volumes, cache |

`docker inspect` + Go templates = quick answers:

```sh
docker inspect -f '{{.State.ExitCode}}' <c>
docker inspect -f '{{json .State.Health}}' <c>
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <c>
```

## 🚨 Common problems

<details>
<summary>💥 <b>Container exits immediately</b></summary>

The main process finished or crashed — a container lives only as long as its PID 1.

```sh
docker ps -a                 # look at the STATUS / exit code
docker logs <c>              # read the error
docker run -it --entrypoint sh <image>   # open a shell instead of the normal command
```

| Exit code | Usually means |
| :-: | :-- |
| `0` | Process finished normally (nothing kept it running) |
| `1` | Application error — check the logs |
| `126` / `127` | Command not executable / not found (typo in `CMD`?) |
| `137` | Killed with `SIGKILL` — often **out of memory** (`OOMKilled` in `docker inspect`) |
| `143` | Stopped with `SIGTERM` (`docker stop`) |

</details>

<details>
<summary>🔌 <b>"Connection refused" from the browser</b></summary>

- Did you publish the port? `docker port <c>`
- Is the app listening on `0.0.0.0`, not `127.0.0.1`? Inside a container, `localhost` means the container itself.
- Right port order? It's `-p <host>:<container>`

</details>

<details>
<summary>🌐 <b>Containers can't reach each other</b></summary>

- Are they on the same **user-defined** network? `docker network inspect <net>`
- Use the **service/container name** and the **container** port, not `localhost` or the host port.

</details>

<details>
<summary>🔒 <b>Permission denied on mounted files</b></summary>

The container user's UID doesn't match the file owner on the host.

```sh
docker run --user "$(id -u):$(id -g)" -v $(pwd):/app my-app
```

</details>

<details>
<summary>🐢 <b>Builds are slow</b></summary>

- Add a `.dockerignore` (is `node_modules` being sent as build context?)
- Copy dependency manifests before source code
- Use [BuildKit cache mounts](./09-advanced.md#-cache-mounts)
- `docker build --progress=plain .` shows which step is slow

</details>

<details>
<summary>💽 <b>Disk is full</b></summary>

```sh
docker system df
docker system prune          # safe-ish cleanup
docker builder prune         # clear build cache
docker image prune -a        # remove all unused images
```

</details>

## 🔬 Debugging a minimal image (no shell)

Distroless / scratch images have no `sh`. Attach a toolbox container that shares the
target's namespaces:

```sh
docker run -it --rm \
  --pid container:<c> \
  --network container:<c> \
  nicolaka/netshoot
```

## ✅ Checkpoint

- [ ] I can find out why a container exited
- [ ] I can inspect env, mounts and networks of a running container
- [ ] I know the usual suspects for networking and permission issues

---

[← 07 · Best Practices & Security](./07-best-practices-and-security.md) · [🏠 Home](../README.md) · [09 · Advanced Docker →](./09-advanced.md)
