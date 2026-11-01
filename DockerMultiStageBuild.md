# Docker Multi Stage Build

## package.json

```json
{
  "name": "Service name",
  "version": "1.0.0",
  "description": "Service description",
  "main": "index.js",
  "private": true,
  "packageManager": "pnpm@9.12.3",
  "engines": {
    "node": ">=22.0.0",
    "pnpm": ">=9.0.0"
  },
  "scripts": {
    "dev": "NODE_ENV=development nodemon index.js",
    "stage": "NODE_ENV=development node index.js",
    "prod": "NODE_ENV=production node index.js"
  },
  "author": "Chiranjibi Biswal",
  "dependencies": {
    "@google-cloud/firestore": "^8.0.0",
    "@google-cloud/secret-manager": "^6.3.0",
    "agentkeepalive": "^4.5.0",
    "axios": "^1.19.0",
    "cassandra-driver": "^4.6.3",
    "cors": "^2.8.6",
    "dotenv": "^16.3.1",
    "express": "^5.2.1",
    "express-validator": "^6.14.0",
    "helmet": "^8.1.0",
    "json-bigint": "^1.0.0",
    "moment-timezone": "^0.5.34",
    "pg": "^8.8.0"
  },
  "devDependencies": {}
}
```

## .dockerignore file

```.dockerignore
Dockerfile
.dockerignore
node_modules
npm-debug.log
pnpm-debug.log
.scannerwork
.git
.gitignore
.github
.env
.env.*
environment/.env.*
*.md
.vscode
.idea
coverage
tests
docker-compose*.yml
```

## Docker Multi-Stage File

```Dockerfile
# syntax=docker/dockerfile:1

# -------- 🔨 Stage 1: Builder --------
FROM node:22.14.0-alpine AS builder

# Installed via npm directly rather than corepack — corepack's "prepare"
# step fetches from registry.npmjs.org and does not honor our internal
# registry/proxy config, which fails on locked-down build agents.
# Version is pinned explicitly here AND in package.json's "packageManager"
# field for documentation purposes — keep both in sync when upgrading.
RUN npm i -g pnpm@9.12.3

WORKDIR /usr/src/app

# 🧠 Copy dependency manifests first to leverage Docker layer caching
COPY package.json pnpm-lock.yaml ./

# ⬇️ Install production dependencies using a deterministic lockfile.
# The cache mount persists pnpm's content-addressable store across builds,
# so re-installs are fast even when this layer's cache is otherwise cold.
RUN --mount=type=cache,id=pnpm-store,target=/root/.local/share/pnpm/store \
    pnpm install --prod --frozen-lockfile

# 📂 Copy application source. .dockerignore keeps .git, .env*, tests, docs,
# and any local node_modules out of the build context entirely.
COPY . .


# -------- 🚀 Stage 2: Runtime --------
# 🛡️ Google's distroless Node 22 image, :nonroot variant — no shell, no package
# manager, and runs as an unprivileged user by default (smaller attack surface).
FROM gcr.io/distroless/nodejs22-debian12:nonroot AS runtime

WORKDIR /usr/src/app

# 🌍 Safe-by-default. Override per environment at the orchestration layer
# (Kubernetes manifest, ECS task definition, docker run -e) — never by
# editing this file per environment.
ENV NODE_ENV=production

# 📋 Copy the built application (source + prod-only node_modules) from the builder stage.
COPY --from=builder --chown=nonroot:nonroot /usr/src/app /usr/src/app

# 💬 Inform Docker/orchestrators which port the app listens on.
EXPOSE 8080

# ▶️ Equivalent to: node index.js
# No Docker HEALTHCHECK here — distroless has no shell to run one. Use a
# Kubernetes liveness/readiness probe against an HTTP endpoint instead.
CMD ["index.js"]
```

## Note

Environment/secret files are never copied into the Docker image — `.dockerignore`
excludes them from the build context entirely, so the builder stage never
contains them.

In Kubernetes, they're mounted into the running pod at deploy time via the
**Secrets Store CSI Driver**, backed by **GCP Secret Manager**, and authenticated
with **Workload Identity** (no credential file, no `GOOGLE_APPLICATION_CREDENTIALS`
env var needed). The mounted file lands at the same `environment/` path the app
already reads locally via `dotenv` — no code changes required between local,
staging, and production.

### Workload Identity, simply: the pod's built-in ID card

* Every pod runs as a Kubernetes identity (KSA).
* That KSA is linked to a Google Cloud identity (GSA) that has permission to the secret.
* When the pod asks for the secret, Google checks "which pod is this," sees the link, and trusts it — no password, no key file.

That trust is the "proof."

For local testing only, the same file is supplied via a bind mount instead:

```bash
docker run -d --name service-test -p 8080:8080 \
  -v $(pwd)/environment:/usr/src/app/environment:ro \
  -e NODE_ENV=production \
  <service-name>:local
```
