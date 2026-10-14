# Containers and Kubernetes Standards

Legend: `[C]` Critical, `[H]` High, `[M]` Medium, `[L]` Low (default severity; adjust per [../severity.md](../severity.md)).

Covers `Dockerfile` / `Containerfile`, Docker Compose, Kubernetes manifests, Helm charts and Kustomize.

## Authorities
- Docker *Dockerfile best practices*
- CIS Docker Benchmark and CIS Kubernetes Benchmark
- Kubernetes **Pod Security Standards** (`restricted` profile)
- NSA/CISA Kubernetes Hardening Guide

## Tooling to look for and run
- `hadolint Dockerfile`
- `trivy config .` / `trivy image`
- `docker compose config`
- `kubeconform`, `kube-linter`
- `helm lint` and `helm template` (to render manifests before reviewing them)

## Dockerfile
- `[C]` Secrets in `ENV`, `ARG` or `COPY` instructions. They stay in the image layers. Use BuildKit secret mounts (`RUN --mount=type=secret`).
- `[H]` Running as root: no `USER` instruction for a non-root user.
- `[H]` Base images unpinned (`:latest`, or no tag). Pin a version tag, ideally with a `@sha256:` digest.
- `[H]` `curl ... | sh` or `ADD https://...` without checksum verification.
- `[M]` No `.dockerignore`, so `.git`, `.env` or `node_modules` end up in the build context and image.
- `[M]` Single-stage builds that ship compilers and dev dependencies. Use a multi-stage build.
- `[M]` Shell-form `CMD` / `ENTRYPOINT`, so the app isn't PID 1 and doesn't get `SIGTERM`. Use exec form `["app"]`.
- `[L]` `apt-get update` in a separate layer from `install`; no `--no-install-recommends`; package caches not cleaned; no `HEALTHCHECK` where the platform uses it.

## Docker Compose
- `[C]` `privileged: true`, or `/var/run/docker.sock` mounted into a container (gives root on the host).
- `[H]` Database or admin ports published on all interfaces (`"5432:5432"`) in shared or prod environments. Bind to `127.0.0.1` or remove the port.
- `[M]` Secrets as plain `environment:` values in committed files; `network_mode: host`.

## Kubernetes workloads
- `[C]` `privileged: true`, `hostPID`, `hostNetwork`, `hostIPC`, or `hostPath` volumes, especially of `/` or the container runtime socket.
- `[C]` RBAC granting `cluster-admin` or wildcard verbs or resources (`"*"`) to app service accounts; `secrets` `get` / `list` across the cluster.
- `[H]` Missing a `securityContext` with `runAsNonRoot: true`, `allowPrivilegeEscalation: false`, `capabilities.drop: ["ALL"]`, `readOnlyRootFilesystem: true` and `seccompProfile: RuntimeDefault`.
- `[H]` Secrets placed in `ConfigMap`s or plain manifests in Git. Use Sealed Secrets, External Secrets or SOPS.
- `[H]` No resource `requests` and `limits`. A memory limit is needed to prevent noisy neighbours and node out-of-memory kills.
- `[H]` No `readinessProbe`, so traffic reaches pods that aren't ready. A `livenessProbe` that checks dependencies causes cascading restarts.
- `[M]` `image: x:latest` or `imagePullPolicy: Always` used as a substitute for versioning.
- `[M]` `automountServiceAccountToken` not disabled for pods that don't call the API.
- `[M]` No `NetworkPolicy` in namespaces that rely on them.
- `[M]` Production deployments with 1 replica, no `PodDisruptionBudget`, or no anti-affinity or topology spread.

## Helm
- `[M]` Secure defaults left to whoever installs the chart: `securityContext` empty by default, and `values.yaml` holding real credentials.
- `[L]` Templates not quoting strings (`{{ .Values.x | quote }}`); missing `NOTES.txt` and chart version bumps.
