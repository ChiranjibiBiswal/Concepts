# What is Podman?

Podman is a tool that runs containers — the same idea as Docker. A container is like a mini-computer inside your computer: it bundles your app and everything it needs (code, libraries, settings) into one package that runs anywhere.

Podman is OCI-compliant, Apache-2.0 licensed, and built by Red Hat (the engine itself is the `libpod` project).

## Both do the same thing

```bash
docker run nginx
podman run nginx
```

## The problem Docker has (in simple terms)

Docker has a manager (daemon) — a program always running in the background. Every command you type goes through this manager:

- You type `docker run nginx`
- Manager receives it
- Manager creates the container

Problems with this manager:

1. It runs as the boss (`root`) by default — it has full power over your computer. If something attacks it, your whole system is at risk.
2. Single point of failure — if the manager crashes, your containers go down with it (unless it was started with `dockerd --live-restore`).
3. It's always running — even when you do nothing, it is eating memory.
4. Being in the `docker` group is effectively being root — anyone who can talk to the daemon can mount `/` into a container and take over the host. This is the escalation path Podman removes entirely.

> Note: **rootless Docker exists** (since 19.03, via `dockerd-rootless.sh`), so the "runs as root" point applies to the *default* setup, not as an absolute rule.

## How Podman fixes it (in simple terms)

Podman has no manager always running. You ask Podman to run a container, and Podman directly starts it as a normal app on your system:

- You type `podman run nginx`
- Podman starts it directly (no middleman)

This is **daemonless**, not "no daemon at all." Podman can still expose an on-demand REST API for tools that expect Docker (`podman system service`). The difference is that it is **socket-activated** — it starts when a client connects and stops when idle, so nothing sits in memory all day.

> Platform note: this is **Linux** behaviour. On macOS and Windows, Podman runs inside a small Linux VM (`podman machine init && podman machine start`), so there is still a VM and its API service underneath.

## You can even see your container as a normal process

```bash
ps aux | grep nginx
```

You'll see **two** entries: `conmon` (the small per-container monitor Podman keeps) and `nginx` itself as its child process — unlike Docker, where containers are hidden behind the daemon:

```
user  12345  ... conmon --name web-nginx ...
user  12346  ... nginx: master process nginx -g daemon off;
```

## Benefits

- Runs as **you**, not `root` → safer (a container can't mess with system files it doesn't own)
- No manager to crash → more reliable
- Nothing running in the background → less overhead
- No root-equivalent group to be added to → smaller blast radius
- Same CLI, same Dockerfiles, same OCI images → easy migration

## Short summary

| | Docker | Podman |
|---|---|---|
| You type | `docker run app` | `podman run app` |
| Behind the scenes | Always-on background daemon | Starts directly (daemonless) |
| Who runs it | Root by default | You (rootless) by default |
| REST API | Always available | On-demand, socket-activated |
| Docker CLI | Native | Drop-in (`podman-docker` or alias) |
| Compose | Native | `podman compose` / via socket |
| Kubernetes YAML | Needs kubectl/kompose | `generate kube` / `play kube` built in |
| Boot / systemd | Restart policies | Quadlet, native systemd units |
| macOS / Windows | Docker Desktop (VM) | `podman machine` (VM) |
| License | Apache-2.0 | Apache-2.0 |
| Goal | Same! | Same! |

## Pods — and why it's called Podman

A **pod** is a group of containers sharing one network namespace (same IP, same ports) — exactly the Kubernetes idea. This is where the name comes from.

```bash
podman pod create --name web -p 8080:80
podman run -d --pod web --name web-nginx nginx
podman run -d --pod web --name web-api my-api:local
podman pod ps
```

Pods can be exported to Kubernetes YAML and replayed:

```bash
podman generate kube web > web.yaml   # pod -> Kubernetes manifest
podman play kube web.yaml             # Kubernetes manifest -> running containers
```

That makes Podman a decent local bridge to Kubernetes without a cluster.

## Drop-in Docker replacement

```bash
alias docker=podman          # quick and dirty
# or, for a permanent drop-in: install the `podman-docker` package
```

```bash
podman compose up -d         # Podman 4.7+, wraps docker-compose or podman-compose
```

For IDEs, Testcontainers, or anything that speaks the Docker API, point them at the socket:

```bash
systemctl --user enable --now podman.socket
export DOCKER_HOST=unix:///run/user/$(id -u)/podman/podman.sock
```

## Running on boot (systemd + Quadlet)

Docker needs a third-party tool for this. Podman is a first-class systemd citizen.

> `podman generate systemd` is **deprecated** (4.4) and **removed** (5.0). Use **Quadlet** instead.

```ini
# ~/.config/containers/systemd/myapp.container
[Unit]
Description=My Node app

[Container]
Image=docker.io/library/nginx:latest
PublishPort=8080:80
Volume=%h/html:/usr/share/nginx/html:z

[Install]
WantedBy=default.target
```

```bash
systemctl --user daemon-reload
systemctl --user start myapp.service
loginctl enable-linger $USER    # keep it running after logout
```

## Rootless gotchas (these bite real users)

- **Ports below 1024** can't be bound without a sysctl:
  ```bash
  sudo sysctl net.ipv4.ip_unprivileged_port_start=0
  ```
- **Sub-UID/GID ranges** are required — check `/etc/subuid` and `/etc/subgid`, then:
  ```bash
  podman unshare cat /proc/self/uid_map   # verify the mapping
  ```
- **cgroups v2** is required for rootless CPU/memory limits.
- **Networking** is slower: rootless uses `slirp4netns` or `pasta`, not a real bridge. Create networks with `podman network create`.
- **SELinux** (Fedora/RHEL) needs a volume label — `:z` (shared) or `:Z` (private):
  ```bash
  podman run -d -v $(pwd)/html:/usr/share/nginx/html:z nginx
  ```
- **Image names must be fully qualified** by default (`docker.io/library/nginx`, not `nginx`) unless `/etc/containers/registries.conf` sets `unqualified-search-registries`.
- If something gets stuck after a config change: `podman system migrate`.

## Ecosystem

Podman doesn't do everything alone — three daemonless OCI tools split the job:

| Tool | Job |
|---|---|
| **Podman** | run / manage containers and pods |
| **Buildah** | build images (this is what `podman build` uses under the hood) |
| **Skopeo** | inspect, copy, and sign images without pulling them |

Plus **Podman Desktop** — the GUI alternative to Docker Desktop.

Build note: `podman build` understands Dockerfiles, but it uses **Buildah**, not BuildKit. Most things carry over (`RUN --mount=type=cache`, heredocs, `--secret`), but BuildKit-only flags and `buildx` features may not.

## Everyday commands

```bash
podman info                 # daemonless, so this is your only "status" check
podman ps -a                # containers
podman images               # images
podman system df            # disk usage
podman system prune -a      # cleanup
podman auto-update          # pull + restart (label: io.containers.autoupdate=registry)
```

## Takeaway

> **Note:** Podman = Docker without the always-on middleman — on **Linux** it runs rootless, so it's safer and lighter. Same CLI, same Dockerfiles, same OCI images, so migration is usually just an `alias`. Trade-offs: rootless port binding, SELinux volume labels, and rootless networking need small tweaks, and BuildKit-specific build features don't all carry over.
