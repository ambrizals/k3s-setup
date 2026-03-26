# AI Context: Server Infrastructure Setup

This directory contains the "ground floor" of the development environment. Use this context when helping with storage, networking, or core service configuration.

## 📂 Key Components
- **`registries.yaml`**: Critical K3s configuration for private registry mirroring.
- **`zot-infra.yaml`**: Deployment for the Zot registry. Requires a 50Gi PVC.
- **`infisical-infra.yaml`**: Full stack for the secret manager (App + Postgres + Redis).
- **`infisical-operator.yaml`**: Installation instructions for the K8s secrets operator.
- **`infisical-secret-sync.yaml`**: The bridge between Infisical and native Kubernetes Secret objects.

## ⚠️ Important Configuration Notes
- **Namespaces**: All core services are strictly deployed in the `dev` namespace.
- **Ingress Domains**: 
    - Registry: `registry.bmd.ambrizal.net`
    - Secrets: `secrets.bmd.ambrizal.net`
- **PVCs**: Using the default K3s (Local Path) storage class.
- **SSL**: Uses the `letsencrypt-prod` ClusterIssuer. Ensure `cluster-issuer.yaml` has a valid email before use.
