# What is Podman?

Podman is a tool that runs containers — the same idea as Docker. A container is like a mini-computer inside your computer: it bundles your app and everything it needs (code, libraries, settings) into one package that runs anywhere.

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

1. It runs as the boss (`root`) — it has full power over your computer. If something attacks it, your whole system is at risk.
2. Single point of failure — if the manager crashes, it can mess up all your containers.
3. It's always running — even when you do nothing, it is eating memory.

## How Podman fixes it (in simple terms)

Podman has no manager. You ask Podman to run a container, and Podman directly starts it as a normal app on your system:

- You type `podman run nginx`
- Podman starts it directly (no middleman)

## You can even see your container as a normal process

```bash
ps aux | grep nginx    # your nginx container shows up just like any app
```

## Benefits

- Runs as **you**, not `root` → safer (a container can't mess with system files it doesn't own)
- No manager to crash → more reliable
- Nothing running in the background → less overhead

## Short summary

| | Docker | Podman |
|---|---|---|
| You type | `docker run app` | `podman run app` |
| Behind the scenes | Goes through a background manager | Starts directly |
| Who runs it | Root (boss) | You (normal user) |
| Goal | Same! | Same! |

> Note: Podman = Docker, but without the middleman — safer, simpler, and does the same job. Both use the same Dockerfile and images, so you don't need to change anything.
