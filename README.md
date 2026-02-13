# GitOps Infrastructure with ArgoCD

Public reference repository demonstrating a full GitOps setup: **ArgoCD** (App of Apps pattern) + **Vault** (Bank-Vaults operator) + **External Secrets Operator** + **Envoy Gateway** + **DragonflyDB** + **cert-manager**.

Active cluster: **xavier**

## App of Apps hierarchy

```
the-apps-of-apps.yaml                    # bootstrap — apply this to init the cluster
  └── apps-of-apps/apps-xavier.yaml      # points to apps/xavier/ (recursive)
        ├── infrastructure/               # cert-manager, Vault, ESO, DragonflyDB, Envoy Gateway
        ├── observability/                # Metrics Server
        └── workloads/bots/              # aptenbot, aptenbot-vip
```

All Applications use automated sync with `selfHeal: true`. ArgoCD watches `main` branch — push and it syncs.

## Secrets flow

```
Vault (Bank-Vaults operator, Raft storage)
  ↓ AppRole auth
External Secrets Operator (ClusterSecretStore "eso-ro")
  ↓ ExternalSecret CRDs
Kubernetes Secrets
```

`secrets/` directory contains ExternalSecret manifests that reference Vault paths — not raw secrets.

## Repository structure

```
the-apps-of-apps.yaml          # root ArgoCD Application
apps-of-apps/                   # ArgoCD Applications and AppProjects
apps/
    xavier/                     # active cluster
        infrastructure/         # core platform components
        observability/          # monitoring
        workloads/              # application deployments
secrets/
    xavier/                     # ExternalSecret manifests per namespace
static/
    xavier/                     # Helm values, static manifests
    vault/                      # Vault CR with Kustomize overlays
```

## Security considerations

This is a public reference repository. The following trade-offs are intentional and accepted for this setup. **For production, you should address these:**

- **Vault TLS is disabled** (`tls_disable: true`) — in-cluster traffic is unencrypted. In production, enable TLS on the Vault listener.
- **Vault unseal threshold is 1/1** — a single key is generated and the root token is stored in a Kubernetes secret. In production, use a higher threshold (e.g. 3/5) and an auto-unseal mechanism (AWS KMS, GCP CKMS, etc.).
- **ArgoCD AppProjects use wildcards** — `*` for resources, namespaces, and source repos. In production, scope these down to only what each project needs.
- **Vault RBAC policy is broad** — `allow_secrets` grants full CRUD on `secret/*`. In production, create granular per-service read-only policies.
- **Metrics server uses `--kubelet-insecure-tls`** — acceptable for self-managed clusters without signed kubelet certs, but not ideal for production.
