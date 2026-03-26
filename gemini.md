# AI Context: K3s Setup & Ops

This directory is the root of a comprehensive development infrastructure platform. Use this context when helping with architectural changes, networking, or overall system flow.

## 🏗️ Architecture Stack
- **Cluster**: K3s (Single node)
- **Ingress**: Traefik (Bundled with K3s)
- **SSL**: cert-manager with Let's Encrypt (HTTP-01 solver).
- **Registry**: Zot (Private container registry).
- **Secrets**: Infisical (Self-hosted) + Infisical K8s Operator.
- **CI/CD**: GitHub Actions pushing to private Zot and deploying to K3s.

## 🔒 Secret Management Architecture
We use a "pull-based" secret model:
1. **Infisical** is the source of truth.
2. **GitHub Actions** fetches secrets from Infisical at build time using a Machine Identity.
3. **K3s Cluster** uses the Infisical Operator to sync secrets from Infisical into native K8s Secrets.
4. **Pods** mount these secrets as environment variables using `envFrom`.

## 🛠️ Operational Workflow
- **Infrastructure Changes**: Modified in `server-setup/` and applied manually via `kubectl`.
- **Application Changes**: Committed to `github-project/` and deployed automatically via GitHub Actions.
- **Secret Changes**: Updated via the Infisical Web UI; synced automatically to the cluster within 60s.
