# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

GitOps infrastructure repository for Kubernetes cluster **xavier**, managed by **ArgoCD** using the **App of Apps pattern**. This is a declarative YAML-only repo — there are no build, lint, or test commands. Changes are pushed to `main` and ArgoCD auto-syncs them to the cluster.

## Directory Layout

- **`the-apps-of-apps.yaml`** — Root ArgoCD Application (bootstrap entry point), points to `apps-of-apps/` with recursive directory scanning
- **`apps-of-apps/`** — ArgoCD Application and AppProject definitions that reference per-cluster app directories
- **`apps/xavier/`** — ArgoCD Application manifests organized by concern:
  - `infrastructure/` — cert-manager, DragonflyDB, Envoy Gateway, External Secrets Operator, Vault
  - `observability/` — Metrics Server
  - `workloads/bots/` — aptenbot Telegram bot deployments
- **`secrets/`** — ExternalSecret CRDs referencing Vault paths (NOT raw secrets — never commit secret values here)
- **`static/`** — Static Kubernetes manifests, Helm values files, and Kustomize overlays referenced by ArgoCD Applications

## ArgoCD Hierarchy

```
the-apps-of-apps.yaml
  └── apps-of-apps/apps-xavier.yaml  (project: management)
        └── apps/xavier/
              ├── infrastructure/*    (project: xavier)
              ├── observability/*     (project: xavier)
              └── workloads/bots/*    (project: default)
```

## Key Conventions

- **App manifest naming**: Infrastructure apps prefixed with `app-` (e.g., `app-cert-manager.yaml`). Workloads use the name directly (e.g., `aptenbot.yaml`).
- **Sync policies**: All apps use `automated` sync with `selfHeal: true`. Infrastructure apps set `prune: false` for safety.
- **Common syncOptions**: `CreateNamespace=true`, `ApplyOutOfSyncOnly=true`
- **Multi-source pattern**: Bot workloads use ArgoCD multi-source — one source for the Helm chart (OCI), one `ref: values` source for values files from this repo, one source for secrets
- **Secrets flow**: Vault → External Secrets Operator (ClusterSecretStore `eso-ro`) → Kubernetes Secrets. ExternalSecrets reference Vault paths like `secret/data/aptenbot`.

## Infrastructure Stack

- **Vault** (Bank-Vaults operator) — Raft storage, Kustomize-based config in `static/vault/`
- **External Secrets Operator** — Authenticates to Vault via AppRole
- **DragonflyDB** (Redis-compatible) — Used by aptenbot workloads, config in `static/xavier/dragonflydb/`
- **Envoy Gateway** — Ingress/gateway layer
- **cert-manager** — TLS certificate management

## Workloads

**aptenbot** / **aptenbot-vip** — AI-powered Telegram bots in namespace `aptenbot`. Helm charts and container images hosted on Gcore OCI registry. Values files in `static/xavier/aptenbot/`.

## Cluster Notes

- **xavier** — Active cluster, all current deployments
- **voldemort** — Decommissioned cluster (`apps/voldemort/README.md` documents its former state)
