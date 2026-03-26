# Server Setup Guide

This directory contains all configuration files that must be applied **directly on your K3s server**.
Follow each phase in order.

---

## Prerequisites

- A running K3s server (`curl -sfL https://get.k3s.io | sh -s -`)
- DNS A Record `*.bmd` pointing to `YOUR_SERVER_IP`
- `kubectl` access to your cluster

---

## Phase 0: Firewall & Port Configuration

Before installing K3s, you MUST ensure your server's firewall allows traffic on the following ports. If you are using a cloud provider (AWS, DigitalOcean, etc.), update your **Security Group** or **Firewall Rules** accordingly.

| Port | Protocol | Purpose |
|---|---|---|
| **80** | TCP | HTTP Traffic (Traefik Ingress) |
| **443** | TCP | HTTPS Traffic (Traefik Ingress) |
| **6443** | TCP | K3s API Server (Required for `kubectl` and CI/CD) |

### On the Server (Ubuntu/Debian):
If you are using `ufw`, run these commands:
```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 6443/tcp
sudo ufw reload
```

---

## Phase 1: K3s Registry Mirror

Configure K3s to route image pulls to your private Zot registry.

```bash
# Copy registries.yaml to the K3s config directory
sudo cp registries.yaml /etc/rancher/k3s/registries.yaml

# Restart K3s to apply
sudo systemctl restart k3s
```

---

## Phase 2: Create Namespace

```bash
kubectl create namespace dev
```

---

## Phase 3: Install Cert-Manager (SSL)

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.0/cert-manager.yaml

# Wait for cert-manager to be ready before proceeding
kubectl wait --namespace cert-manager \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/instance=cert-manager \
  --timeout=120s
```

Then apply the ClusterIssuer:

> ⚠️ **IMPORTANT:** Open `cluster-issuer.yaml` and replace `your-email@ambrizal.net` with your real email before applying.

```bash
kubectl apply -f cluster-issuer.yaml
```

---

## Phase 4: Deploy Zot (Private Container Registry)

```bash
kubectl apply -f zot-infra.yaml

# Verify Zot is running
kubectl get pods -n dev -l app=zot
```

Zot will be available at: `https://registry.bmd.ambrizal.net`

---

## Phase 5: Deploy PostgreSQL (App Database)

```bash
kubectl apply -f postgres-infra.yaml

# Verify Postgres is running
kubectl get pods -n dev -l app=postgres
```

---

## Phase 6: Deploy Infisical (Secret Manager)

> ⚠️ **IMPORTANT:** Before applying, open `infisical-infra.yaml` and replace the placeholder values:
>
> | Field | How to Generate |
> |---|---|
> | `ENCRYPTION_KEY` | `openssl rand -hex 16` |
> | `AUTH_SECRET` | `openssl rand -base64 32` |
> | `POSTGRES_PASSWORD` (infisical-postgres) | Choose a strong password |
> | `DB_CONNECTION_URI` | Must match the password you set above |

```bash
kubectl apply -f infisical-infra.yaml

# Verify all Infisical pods are running
kubectl get pods -n dev -l app=infisical
kubectl get pods -n dev -l app=infisical-postgres
kubectl get pods -n dev -l app=infisical-redis
```

Infisical will be available at: `https://secrets.bmd.ambrizal.net`

After it's live:
1. Open `https://secrets.bmd.ambrizal.net` and create your admin account.
2. Create a new project (e.g., `my-app`).
3. Add your secrets: `REGISTRY_USER`, `REGISTRY_PASS`, `KUBECONFIG_DATA`.
4. Create a **Machine Identity** and note the `CLIENT_ID` and `CLIENT_SECRET`.

---

---

## Phase 7: Install Infisical Kubernetes Operator

The operator watches for `InfisicalSecret` resources and syncs secrets from Infisical
into native Kubernetes Secrets — so your pods can use them as environment variables.

```bash
# Install the Infisical operator into your cluster
kubectl apply -f https://raw.githubusercontent.com/Infisical/infisical/main/k8-operator/config/install/install.yaml

# Verify the operator is running
kubectl get pods -n infisical-operator-system
```

---

## Phase 8: Sync Secrets to Kubernetes

> ⚠️ **IMPORTANT:** Open `infisical-secret-sync.yaml` and replace:
> - `REPLACE_WITH_INFISICAL_CLIENT_ID` — from your Infisical Machine Identity
> - `REPLACE_WITH_INFISICAL_CLIENT_SECRET` — from your Infisical Machine Identity
> - `my-app` — with your actual Infisical project slug
> - `prod` — with your target environment (`dev` / `staging` / `prod`)

```bash
kubectl apply -f infisical-secret-sync.yaml

# Verify the InfisicalSecret resource and the synced K8s Secret
kubectl get infisicalsecret -n dev
kubectl get secret backend-secrets -n dev
```

Once synced, all secrets you add to Infisical (e.g. `DB_HOST`, `DB_PASS`, `API_KEY`)
will automatically appear as environment variables inside your pods via `envFrom`.

**To add a new env var to your backend service:**
1. Log in to `https://secrets.bmd.ambrizal.net`
2. Navigate to your project → environment
3. Add the key/value pair
4. The operator re-syncs within **60 seconds** — no redeployment needed

---

## Final Directory Structure

```
server-setup/
├── README.md                    ← You are here
├── registries.yaml              ← Phase 1: K3s registry mirror
├── cluster-issuer.yaml          ← Phase 3: Let's Encrypt SSL issuer
├── zot-infra.yaml               ← Phase 4: Zot private container registry
├── postgres-infra.yaml          ← Phase 5: PostgreSQL database
├── infisical-infra.yaml         ← Phase 6: Infisical secret manager
├── infisical-operator.yaml      ← Phase 7: Infisical K8s operator (reference)
└── infisical-secret-sync.yaml   ← Phase 8: Sync Infisical secrets → K8s Secret
```

## Secret Flow Summary

```
Infisical Dashboard
       │
       │  (Infisical Operator polls every 60s)
       ▼
K8s Secret: backend-secrets  (namespace: dev)
       │
       │  (envFrom: secretRef)
       ▼
backend-deployment pods → DB_HOST, DB_PASS, API_KEY, etc. available as env vars
```
