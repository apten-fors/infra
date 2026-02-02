# Cluster Deployment Overview

This document provides a technical overview of the current Kubernetes cluster deployment, including system components, operators, and application workloads.

## Infrastructure & System Components

The cluster relies on the following core infrastructure components and controllers:

| Component | Implementation | Namespace | Description |
|-----------|----------------|-----------|-------------|
| **Ingress Controller** | **Ingress NGINX** | `ingress-nginx` | Manages external access to services (LoadBalancer IP: `217.195.194.204`). |
| **Certificates** | **cert-manager** | `cert-manager` | Automates management and issuance of TLS certificates. |
| **Redis Operator** | **Spotahome Redis Operator** | `default`* | Manages Redis Failover clusters via `redisfailovers.databases.spotahome.com` CRD. |

*> *Note: The Redis Operator controller runs in the cluster, attempting to manage Redis resources.*

---

## Application Workloads

The business logic is divided into two main namespaces: `tshawytscha-ai` (Web Platform) and `aptenbot` (Telegram Bots).

### 1. Web Platform (`tshawytscha-ai` namespace)
Hosts the AI-powered web application components.

- **Frontend**
  - **Deployment**: `tshawytscha-ai-front`
  - **Ingress**: Exposed via `tshawytscha.ai` (TLS enabled).
- **Backend**
  - **Deployment**: `tshawytscha-ai-back`

### 2. Telegram Bots (`aptenbot` namespace)
Hosts the Telegram bot ecosystem and its persistence layer.

- **Bot Deployments**
  - **`aptenbot`**: Main production bot instance.
  - **`aptenbot-vip`**: VIP instance of the bot.

- **Data Storage (Redis)**
  - Managed by the **Spotahome Redis Operator**.
  - **Resource**: `rfr-redisfailover` (CRD).
  - **Components**:
    - `rfr-redisfailover` (StatefulSet) - Redis Sentinel/Server nodes.
    - `rfs-redisfailover` (Deployment) - Sentinel management.
  - **Usage**: Shared state and data storage for both bot instances.

---

## Deployment Methodology (Legacy)

**Note:** This represents a legacy approach.
All components described above were deployed manually using imperative commands such as `helm install` or `kubectl apply -f <manifest>.yaml` directly into the cluster.
