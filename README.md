# app-reference-implementations

Reference applications that consume the hardened base images from [platform-images/hardened-base](https://github.com/platform-images/hardened-base). Each app is a minimal but real service built with a multi-stage Dockerfile using the platform's distroless runtime images.

## Apps

| App | Runtime | Base image | Port |
|-----|---------|------------|------|
| `nodejs-api` | Express 4 | `ghcr.io/platform-images/nodejs-base` | 3000 |
| `python-api` | FastAPI + uvicorn | `ghcr.io/platform-images/python-base` | 8000 |
| `java-api` | Spring Boot 3 | `ghcr.io/platform-images/openjdk-base` | 8080 |
| `nginx-proxy` | nginx | `ghcr.io/platform-images/nginx-base` | 8080 |

Each app exposes:
- `GET /health` — liveness check
- `GET /api/items` — list items
- `GET /api/items/:id` — get item by ID (nodejs / python / java)

The nginx-proxy routes `/nodejs/`, `/python/`, `/java/` to the upstream services.

## Dockerfile pattern

Every app uses the same two-stage pattern:

```dockerfile
# Build stage — toolchain available (npm, pip, mvn)
# USER root overrides the nonroot default so package installs succeed
FROM ghcr.io/platform-images/<runtime>-base:1.0.0@sha256:<digest> AS builder
USER root
WORKDIR /app
# ... install dependencies ...

# Runtime stage — distroless, no shell, no package manager
FROM ghcr.io/platform-images/<runtime>-base:1.0.0-distroless
LABEL org.opencontainers.image.source="..."
LABEL org.opencontainers.image.licenses="Apache-2.0"
# ... copy built artefacts only ...
```

Base images are pinned to their SHA digest. Renovate automatically opens PRs to update these pins when a new base image version is published.

## CI pipeline

### PR checks (`pr-checks.yml`)

Runs on every pull request:

| Job | Tool | What it checks |
|-----|------|----------------|
| SAST | Semgrep | Source code security issues |
| SCA | Trivy (fs) | Known CVEs in dependency manifests |
| Build + image scan | Trivy (image) | Known CVEs in the built container image |

Only apps with changed files are checked. If no `apps/` files change, all apps are checked.

### Release (`release.yml`)

Triggers automatically when any `apps/*/Dockerfile` changes on main — which happens when a Renovate digest-bump PR merges. Can also be triggered manually.

For each changed app:

1. Build multi-arch image (`linux/amd64`, `linux/arm64`)
2. Push to GHCR tagged `latest` and `sha-<git-sha>`
3. Sign with Cosign (keyless, via GitHub OIDC)
4. Generate SPDX SBOM with Syft
5. Attest SBOM to the image manifest
6. Create GitHub Release

## Triggering a release manually

Go to **Actions → Release → Run workflow** and select the app (or `all`).

## Pulling an image

```bash
docker pull ghcr.io/platform-images/nodejs-api:latest
docker pull ghcr.io/platform-images/nodejs-api:sha-<git-sha>
```

## Verifying a signed image

```bash
cosign verify \
  --certificate-identity-regexp="https://github.com/platform-images/app-reference-implementations" \
  --certificate-oidc-issuer="https://token.actions.githubusercontent.com" \
  ghcr.io/platform-images/nodejs-api:latest
```

## Required secret

The workflows pull base images from the private `platform-images/hardened-base` packages. Add a secret named `GHCR_TOKEN` to this repo — a classic PAT with `write:packages` scope.

Settings → Secrets and variables → Actions → New repository secret.

## Repo structure

```
apps/
  nodejs-api/
    src/index.js
    package.json
    package-lock.json
    .dockerignore
    Dockerfile
  python-api/
    src/main.py
    requirements.txt
    Dockerfile
  java-api/
    src/main/java/com/platform/api/
      ApiApplication.java
      controller/ItemController.java
    pom.xml
    Dockerfile
  nginx-proxy/
    nginx.conf
    Dockerfile
.github/
  workflows/
    pr-checks.yml
    release.yml
renovate.json
```
